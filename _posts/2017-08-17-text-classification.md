---
layout: post
comments: true
title:  "Practical Text Classification for Production Systems"
excerpt: "This post is about using a relatively simple yet powerful text classification model for
a production text classificaiton system. Other topics like deployment, testing for out-of-sample texts are
also discussed - they are often not the sexiest aspects, but it makes sense to discuss them in this post."
date:   2017-08-17 23:00:00
mathjax: false
---

I had to create a text classification system few months ago. Unfortunately, I had never done any text processing
and didn't know anything about NLP. Fortunately, it's relatively easy to create a simple text classifier by modifying
the state of the art models. This post is about using a relatively simple yet powerful text classification model for
a production system. Other topics like deployment, testing for out-of-sample texts are
also discussed - they are often not the sexiest aspects, but it makes sense to discuss them in this post.

## **Data**

I didn't have any data to start with. I did not know how to get the data either - after a lot of back and forth it was
decided to work on manually generating the dataset. The following advice from Richard Socher is
pertinent in this case.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Rather than spending a month figuring out an unsupervised machine learning problem, just label some data for a week and train a classifier.</p>&mdash; Richard (@RichardSocher) <a href="https://twitter.com/RichardSocher/status/840333380130553856">March 10, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

The dataset consisted of input texts and corresponding class numbers.
```
text | class_id
t1   | 1
t2   | 1
t3   | 2
t4   | 2
t5   | 2
```

In order to make sure random text inputs do not creep into the valid classes, I added a special `class_id 0` for all
**negative cases**. For example, _'What is fake news?'_ and _'How much time does it take to drive to airport?'_ were
part of the negative samples.

## **Word vectors**
A week into this task, I realized the pre-trained [GloVe word vectors](https://nlp.stanford.edu/projects/glove/)
from the Stanford website are not very useful to
me. Important words like `roboadvisor`, `S&P500` were missing from that set. Additionally, terms like `risk`,
`bond`, `swap`, `interest`, `liquid`, `trade` and `market` have a different meaning in Finance.
I also wanted to capture the finance
specific relationship between the words so that the classification model can do a better job on the finance specific sentences.

So I decided to start from scratch. I scraped the web for finance specific content. In the end, I had about 100MB of raw text. I used the [GloVe C code](https://nlp.stanford.edu/software/GloVe-1.2.zip)
from the Stanford website to generate
the word vectors.

We can do a sanity check for the word vectors.
```python
import numpy as np
import torch

# Load word vectors
# Downloaded from https://github.com/hardikp/fnlp/releases/download/v0.0.4/glove.37M.50d.zip
word_vector_path = 'glove.37M.50d.txt'
embeddings_index = {}
f = open(word_vector_path)
for line in f:
    values = line.split(' ')
    word = values[0]
    coefs = np.asarray(values[1:], dtype='float32')
    embeddings_index[word] = coefs
f.close()

def get_word(w):
    return torch.from_numpy(embeddings_index[w])

def closest(d, n=10):
    all_dists = [(w, torch.dist(d, get_word(w))) for w in embeddings_index]
    return sorted(all_dists, key=lambda t: t[1])[:n]

# In the form w1 : w2 :: w3 : ?
def analogy(w1, w2, w3, n=5, filter_given=True):
    print('\n[%s : %s :: %s : ?]' % (w1, w2, w3))

    # w2 - w1 + w3 = w4
    closest_words = closest(get_word(w2) - get_word(w1) + get_word(w3))

    # Optionally filter out given words
    if filter_given:
        closest_words = [t for t in closest_words if t[0] not in [w1, w2, w3]]

    return closest_words[:n]
```

Let's check if the closest words make sense.
```
# Check the closest words
In [3]: closest(get_word('stock'), 5)
Out[3]:
[('stock', 0.0),
 ('market', 3.1340246200561523),
 ('exchange', 3.162646532058716),
 ('shares', 3.349428176879883),
 ('stocks', 3.3590168952941895)]

In [4]: closest(get_word('google'), 5)
Out[4]:
[('google', 0.0),
 ('facebook', 2.7423112392425537),
 ('microsoft', 3.0939431190490723),
 ('apple', 3.184936285018921),
 ('twitter', 3.452094554901123)]

# Check analogies
In [10]: analogy('stock', 'price', 'bond')

[stock : price :: bond : ?]
Out[10]:
[('yield', 4.4601311683654785),
 ('coupon', 4.605474948883057),
 ('maturity', 4.728261947631836),
 ('spot', 4.933423042297363),
 ('premium', 5.085546016693115)]
```

Both the closest words and the analogies make sense for us to proceed with a simple neural network model.

## Model
I noticed [this simple Bag of Words model](https://github.com/Smerity/keras_snli)
by Stephen Merity using keras. It was remarkably simple in its architecture.
I just modified that to use a different type of dataset.

The final model code is really simple:
```python
inp = Input(shape=(MAX_LEN, ), dtype='int32')
out = Embedding(
        VOCAB, config.word_vector_dim, weights=[embedding_matrix],
        input_length=MAX_LEN, trainable=TRAIN_EMBED)(out)
out = TimeDistributed(Dense(config.hidden_size, activation=config.activation))(out)

# Bag of Words layer - Sum up the sequence
out = keras.layers.core.Lambda(lambda x: K.sum(x, axis=1), output_shape=(config.hidden_size, ))(out)
out = BatchNormalization()(out)

for i in range(3):
    out = Dense(config.hidden_size, activation=config.activation, kernel_regularizer=l2(L2))(out)
    out = Dropout(config.dropout_p)(out)
    out = BatchNormalization()(out)

out = Dense(y_train.shape[1], activation='softmax')(out)
model = Model(inputs=[inp], outputs=out)
model.compile(optimizer=config.optimizer, loss='categorical_crossentropy', metrics=['accuracy'])
```

## Deployment

The preliminary results looked good enough to move this to a production system.

To move this to web facing interface, I basically just dumped the trained keras model into a file, uploaded it as
a [LARGEBLOB in mysql](https://dev.mysql.com/doc/refman/5.7/en/blob.html). Additionally, the word to index mapping
is also required when serving a web request.
```
mysql> DESCRIBE models;
+---------------+-----------+------+-----+
| Field         | Type      | Null | Key |
+---------------+-----------+------+-----+
| day           | date      | NO   | PRI |
| classifier    | longblob  | NO   |     |
| word_to_index | longblob  | NO   |     |
| created_at    | timestamp | YES  |     |
+---------------+-----------+------+-----+
```
The web apis were configured to use this model as a singleton class.

## **Testing for out of sample texts**

There was one problem which I struggled with for more than a month. Even after adding sufficient negative sample cases,
I wasn't entirely sure how the model behaved against completely random input sentences. So, I came up with this simple
yet effective empirical strategy:
* Randomly generate input sentences using the vocabulary, combine them into a sentence and feed that to the model.
  For example, if the randomly selected words are `world`, `da`, `commission` and `veda`, my random sentence would be
  `world da commission veda`. I would generate thousands of such random sentences of different lengths and test that
  against my model to check if it's okay at discarding irrelevant items. If the fraction of sentences matching against
  one of the labels is higher than certain threshold, I have a problem in my model that needs to be fixed.
  Otherwise, I can confident at deploying it in production.
