---
layout: post
comments: true
title: "Selected Papers on Applying Deep Learning for Stock Prediction"
excerpt: ""
date:   2017-08-18 18:00:00
mathjax: true
---

This post summarizes 2 papers/articles that try to predict the stock movement using neural networks:
* Deep Learning for Event-Driven Stock Prediction by Ding et al. ([pdf](https://www.ijcai.org/Proceedings/15/Papers/329.pdf))
* Predicting Stock Market Movement with Deep RNNs ([pdf](https://bcourses.berkeley.edu/files/70257274/download))

Table of Contents:
* TOC
{:toc}

## **Deep Learning for Event-Driven Stock Prediction**

### Summary
The aim is to be able to predict the price movement using long-term and short-term events, as reported in the news.
It is framed as a classification problem. The approach is to first learn event embeddings from the news events.
Embeddings for long-term and short-term events are then considered together in a feed-forward network to predict the
final class.

### Dataset
They gathered about 10 million news events from Reuters and Bloomberg with timestamps ranging from Oct 2006 to Nov 2013.

Each event has the form **$$E = (O_1, P, O_2)$$**, where $$P$$ is the action, $$O_1$$ is the actor and $$O_2$$
is the object on which the action is performed.
For example, in the event "Google Acquires Smart Thermostat Maker Nest For for $3.2 billion.", Actor is _Google_,
Action is _acquires_ and the Object is _Nest_.

<img src="/assets/stocks-dl-papers-1/dataset1.png" style="width: 500px;">

### Event Embeddings

<img src="/assets/stocks-dl-papers-1/event_embeddings.png" style="width: 400px;">

They convert the news events into event embeddings. They do this using a 2-step process: first converting
$$O_1$$ and $$P$$ to $$R_1$$ and $$O_2$$ and $$P$$ to $$R_2$$. $$R_1$$ is computed by:

$$R_1 = f(O_1^TT_1^{[1:k]}P + W\begin{bmatrix}
        O_1 \\
        P \\
        \end{bmatrix} + b)$$

In the second step, $$R_1$$ and $$R_2$$ are combined into the event embedding vector.

### Model Description

The news events are grouped into 3 categories:
* **long-term events**: Events over the past month
* **mid-term events**: Events over the past week
* **short-term events**: Events on the past day of the stock price change

In each case, they combine the events within each day by averaging the event embedding vectors. For **long-term** and
**mid-term** sequences, they apply a convolutional layer of width 3 to extract localized features. Max pooling is
applied to get the final dominant feature from each of them. Ultimately, they end up with feature vectors
$$V^C = (V^l, V^m, V^s)$$.

<img src="/assets/stocks-dl-papers-1/model1.png" style="width: 400px;">

This feature vector is fed into a feed-forward neural network with one hidden layer and one output layer. This can be
represented as:
$$y_{cls} = f(net_{cls}) = \sigma(W_3^T \cdot Y)$$ and $$Y = \sigma(W_2^T \cdot V^C)$$.

### Results
<img src="/assets/stocks-dl-papers-1/results1.png" style="width: 400px;">

* **WB-NN**: word embeddings input and standard neural network prediction model
* **WB-CNN**: word embeddings input and convolutional neural network prediction model
* **E-CNN**: structured events tuple input and convolutional neural network prediction model
* **EB-NN**: event embeddings input and standard neural network prediction model
* **EB-CNN**: event embeddings input and convolutional neural network prediction model

### Comments

The event embedding aspect of the paper seems promising. It looks like a nice structured way to represent the events
that can be used as an input to more concrete machine learning problems in finance.

While the authors have not released any code, there is a github [repo available online](https://github.com/WayneDW/Sentiment-Analysis-in-Event-Driven-Stock-Price-Movement-Prediction) that takes inspiration from this
paper - even though the main idea of event embedding is missing in the linked code.


## **Predicting Stock Market Movement with Deep RNNs**

I believe this is a project report, rather than a paper.

### Dataset

Dataset consists of top 25 news headlines on the "/r/worldnews" subreddit from 2008-08-08 to 2016-07-01.
Each example is labeled as 0 or 1 depending on Dow Jones Industrial Average (DJIA) Adjusted Close value increased or decreased.
(Data source - [https://www.kaggle.com/aaron7sun/stocknews](https://www.kaggle.com/aaron7sun/stocknews))

### Model Description

All 25 headlines are concatenated into a single paragraph. [tf-idf](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)
feature vector is extracted on this paragraph.

3 different models are considered here: 3-layer GRU, 3-layer LSTM and 12-layer GRU.

<img src="/assets/stocks-dl-papers-1/model4.png" style="width: 150px;"/>

Hyperparameters considered:
* Dropout: 0.25, 0.5, 0.75
* Hidden size in each GRU layer: 32, 64, 128
* Activation functions: softmax, tanh, sigmoid, ELU
* Weight Initialization: uniform, gaussian, xavier
* Learning rates: 1e-3 and Adaptive rates

### Results

<img src="/assets/stocks-dl-papers-1/results4.png">

SVM is slightly outperforming 12-layer GRU, which is better than 3-layer LSTM and 3-layer GRU.

The author has put up the code [here](https://github.com/jvpoulos/drnns-prediction).

### Comments

Given that this was a course project, the quality of the experimentation and the writing is understandably lacking.
However, it seems this a relatively small datasets lacking the inherent predictability. And SVM seems to be able to
better utilize whatever predictive power this "processed" dataset (i.e. tf-idf feature vectors) has.
