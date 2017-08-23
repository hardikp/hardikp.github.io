---
layout: post
comments: true
title: "Deep learning networks for stock market analysis and prediction"
excerpt: "In this paper, Deep learning techniques are applied to the financial market data directly rather than using any text/alternative data sources. This has been a relatively tricky dataset for any non-linear machine learning technique because of the extremely high noise-to-signal ratio. The authors use a relatively high-frequency dataset sampled at every 5 minutes. They consider 38 stocks from Korea KOSPI."
date:   2017-08-22 18:00:00
mathjax: true
---

This post summarizes the following paper that try to predict the stock movement using neural networks:
* [Deep learning networks for stock market analysis and prediction](http://download.xuebalib.com/xuebalib.com.32109.pdf): Methodology, data representations, and case studies

Table of Contents
* TOC
{:toc}

## **Deep learning networks for stock market analysis and prediction**

### <span style="color:#e08d60">Summary</span>
The authors consider 2 main problems:
1. Predicting intraday stock returns only using the intraday market data
1. Predicting covariance matrix using the predicted stock returns

### <span style="color:#e08d60">Dataset</span>
Dataset consists of **38 stocks** from Korea KOSPI with prices **sampled every 5 minutes**.
The date range for the data collection
is from 2010-01-04 to 2014-12-30. First 80% of the sample (from 2010-01-04 to 2013-12-24) is taken for training.
At each timestamp, the algorithm has access to last 10 log returns for each stock. Log return is computed as
$$r_t = \ln(S_t/S_{t-\Delta{t}})$$, where $$S_t$$ is the stock price at time $$t$$, and $$\Delta{t}$$ is 5 minutes.
The sample contains a total of 1239 trading days and 73,041 five-minute returns (excluding the first ten returns each
day) for each stock.

### <span style="color:#e08d60">Data Preprocessing</span>
The authors explore various preprocessing techniques. Preprocessed data is fed into the neural network in the prediction stage.
* **RawData**: No proprocessing. Raw returns in a 38 * 10 sized vector.
* **PC200**: PCA with output dimension 200.
* **PC380**: PCA with output dimention 380.
* **AE400**: Sparse Autoencoder with output dimension 400. (The autoencoder has 1-hidden layer with size 400.)
* **AE800**: Sparse Autoencoder with output dimention 800.

### <span style="color:#e08d60">Intraday Stock Return Prediction Approaches</span>
A neural network with 2 hidden layers is compared against a [univariate autoregressive model](http://www.statsmodels.org/dev/generated/statsmodels.tsa.ar_model.AR.html#statsmodels.tsa.ar_model.AR) with 10 lagged variables.
Sizes of the hidden layers are 200 and 100 respectively. Since this a regression model, the final output is a scalar.

$$h_1 = ReLU(W_1u_t + b_1)$$<br>
$$h_2 = ReLU(W_2h_1 + b_2)$$<br>
$$\hat{r}_{i,t+1} = W_3h_2 + b_3$$

### <span style="color:#e08d60">Stock Return Results</span>

| Method        | NMSE   |
| ------------- | ------ |
| AR(10)        | 0.9655 |
| ANN (RawData) | 0.9937 |
| DNN (RawData) | **0.9629** |
| DNN (PCA380)  | 0.9660 |
| DNN (RBM400)  | 0.9702 |
| DNN (AE400)   | 0.9638 |

NMSE is the normalized Mean Squared Error defined as

$$NMSE = \frac{1}{N} \frac{\sum_{n=1}^N (r_{t+1}^n - \hat{r}_{t+1}^n)^2}{var(r_{t+1}^n)}$$

where $$var()$$ is the variance.

### <span style="color:#e08d60">Comments</span>
Results are certainly a bit underwhelming. However, that's not really surprising. My own experiments with the US equities
intraday data have been similar. I was using an LSTM model to predict the out-of-sample intraday stock returns.
It was better than linear regression in some stocks, but a bit worse in other stocks.

The underlying problem with the higher frequency intraday data is the significant amount of noise built into the data.
Simply increasing the model capacity by using a neural network is not going to fix that issue.
