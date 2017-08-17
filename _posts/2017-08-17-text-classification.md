---
layout: post
comments: true
title:  "Practical Text Classification for Question Answering"
excerpt: ""
date:   2017-08-17 23:00:00
mathjax: false
---

I was tasked to create a question-answer application few months. Unfortunately, I had never done any text processing
and didn't know anything about NLP. Fortunately, it's relatively easy to create a simple text classifier by modifying
the state of the art models. This post is about using a relatively simple yet powerful text classification model for
a production question-answer system. Other topics like deployment, testing for out-of-sample texts are
also discussed - they are often not the sexiest aspects, but it makes sense to discuss them in this post.

## **Data**

I didn't have any data to start with. I did not know how to get the data either - after a lot of back and forth it was
decided to work on manually generating the question-answer dataset ourselves. This advice from Richard Socher is
pertinent in this case.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Rather than spending a month figuring out an unsupervised machine learning problem, just label some data for a week and train a classifier.</p>&mdash; Richard (@RichardSocher) <a href="https://twitter.com/RichardSocher/status/840333380130553856">March 10, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

The dataset consisted of input texts and corresponding class numbers.
```
question | class_id
q1       | 1
q2       | 1
q3       | 2
q4       | 2
q5       | 2
```

In order to make sure random text inputs do not creep into the valid classes, I added a special `class_id 0` for all
**negative cases**. For example, _'What is fake news?'_ and _'How much time does it take to drive to airport?'_ were
part of the negative samples.

## **Word vectors**
A week into this task, I realized the pre-trained GloVe word vector from the Stanford website are not very useful to
us. Important words like `qplum`, `roboadvisor`, `S&P500` were missing from that set. Additionally, terms like `risk`,
`bond`, `swap`, `interest`, `liquid`, `trade` and `market` have a different meaning in Finance.
I also wanted to capture the finance
specific relationship between the words so that the classification model can better answer the questions on qplum.

So I decided to start from scratch. I scraped the web for finance specific content. I also scraped the qplum website
content. In the end, I had about 100MB of raw text. I used the GloVe C code from the Stanford website to generate
the word vectors.

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

The preliminary results looked good enough to move this to a production system. At the time, the production question
answer system was purely hardcoded. If the questions matched with the hardcoded questions, they would
answer appropriately. In all other cases, they would fail.

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
  one of the labels is higher than certain threshold, I have a problem in my model. Otherwise, I can confident at
  deploying it in production.
