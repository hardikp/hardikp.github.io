---
layout: post
comments: true
title: "Machine Learning for Intraday Stock Price Prediction 2: Neural Networks"
excerpt: "This is the second of a series of posts on the task of applying machine learning for intraday stock price/return prediction. Price prediction is extremely crucial to most trading firms. People have been using various prediction techniques for many years. We will explore those techniques as well as recently popular algorithms like neural networks. In this post, we will focus on applying neural networks on the features derived from market data."
date:   2017-10-19 22:00:00
mathjax: true
---
This is the second of a series of posts on the task of applying machine learning for intraday stock price/return prediction. Price prediction is extremely crucial to most trading firms. People have been using various prediction techniques for many years. We will explore those techniques as well as recently popular algorithms like neural networks. In this post, we will focus on applying neural networks on the features derived from market data.

You can also check out the [first](/2017/10/03/intraday-stock-price-prediction-1) post.

* TOC
{:toc}

## Feed Forward Neural Networks

The goal is to predict the price change of a security in the next 5 min. We can use a simple feed-forward neural network (FNN) for this. The loss function for the FNN remains the same. The loss function remains the same as what we saw in the linear regression section in the last post.

$$J(\theta) = \frac{1}{2m}\sum_{i=1}^m (h_{\theta}(x^{(i)}) - y^{(i)})^2 + \alpha ||\theta||_2^2$$

While $$h(x)$$ was a linear model in the last post, it is a feed forward neural network in this case. I am also use the `l2_penalty` for stability. I used the following [PyTorch](http://pytorch.org/) code to train the network.

```python
class FNN(nn.Module):
    def __init__(self, num_features):
        super(FNN, self).__init__()
        self.fc1 = nn.Linear(num_features, 100)
        self.bn1 = nn.BatchNorm1d(100)
        self.fc2 = nn.Linear(100, 20)
        self.bn2 = nn.BatchNorm1d(20)
        self.fc3 = nn.Linear(20, 1)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.bn1(self.fc1(x)))
        x = F.dropout(x, training=self.training)
        x = self.relu(self.bn2(self.fc2(x)))
        x = F.dropout(x, training=self.training)
        return self.fc3(x)
```

Some of things I learned while optimizing the above model:
* Normalized inputs are a must. I used the [RobustScaler](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html#sklearn.preprocessing.RobustScaler) class from the scikit-learn library to normalize the inputs.
* BatchNormalization and Dropout significantly help improve the generalization error. `BatchNormalization` ensures the activations in the hidden layers are normalized as well.
* Normalizining the target (Y) also helps.
* Early stopping based on a validation set is generally a good idea. Neural Networks are generally very good at learning dataset. So, we need to stop when the validation error stops decreasing. This [paper](https://arxiv.org/abs/1611.03530) from Google Brain discusses the generalization errors in much more detail.

## A simple Joint Many-Task Model to predict for all the securities together

The ability to model multiple tasks together is a really good advantage of using a neural network. Our hypothesis is that the feature vectors contain enough information to be able to predict multiple securities. Therefore, we can directly predict the price change for all the securities together. We will still use the same MSE loss function for optimization.

```python
class FNN(nn.Module):
    def __init__(self, num_features, num_securities):
        super(FNN, self).__init__()
        self.fc1 = nn.Linear(num_features, 100)
        self.bn1 = nn.BatchNorm1d(100)
        self.fc2 = nn.Linear(100, 50)
        self.bn2 = nn.BatchNorm1d(50)
        self.fc3 = nn.Linear(50, num_securities)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.bn1(self.fc1(x)))
        x = F.dropout(x, training=self.training)
        x = self.relu(self.bn2(self.fc2(x)))
        x = F.dropout(x, training=self.training)
        return self.fc3(x)
```

The training for the Joint Many-Task model would be similar to the normal FNN model.

```python
criterion = nn.MSE()
fnn = FNN(num_features, num_securities)

l2_penalty = 1e-3
learning_rate = 1e-3
optimizer = torch.optim.Adam(fnn.parameters(), lr=1e-3, weight_decay=l2_penalty)
for n in range(num_epochs):
    for i, batch in enumerate(train_loader):
        inputs, targets = Variable(batch[0]), Variable(batch[1])

        optimizer.zero_grad()
        outputs = fnn(inputs)
        loss = criterion(outputs, targets)
        loss.backward()
        optimizer.step()
```

## LSTM Model

Since Recurrent Neural Network (RNN) models are supposed to be good at the predicting sequences, financial prediction models like the one we are exploring might be well suited for them. One problem with a vanilla RNN is that of [vanishing gradient](http://neuralnetworksanddeeplearning.com/chap5.html#the_vanishing_gradient_problem). Long Short Term Memory (LSTM) networks are better at dealing with this problem.

However, we know that even LSTMs have their own limitations - they are often good at shorter sequences, till the length of 10-30. So, I tried the following model with sequence length of 15.

```python
class LSTM(nn.Module):
    def __init__(self, input_size, hidden_size, dropout=0.2):
        super(LSTM, self).__init__()

        self.hidden_size = hidden_size
        self.rnn = nn.LSTM(
            input_size=input_size,
            hidden_size=hidden_size,
            num_layers=1,
            dropout=dropout,
            bidirectional=False, )
        self.fc1 = nn.Linear(hidden_size, hidden_size)
        self.bn1 = nn.BatchNorm1d(hidden_size)
        self.fc2 = nn.Linear(hidden_size, 1)
        self.relu = nn.ReLU()

    def forward(self, x):
        batch_size = x.size()[0]
        seq_length = x.size()[1]

        x = x.view(seq_length, batch_size, -1)

        # We need to pass the initial cell states
        h0 = Variable(torch.zeros(seq_length, batch_size, self.hidden_size))
        c0 = Variable(torch.zeros(seq_length, batch_size, self.hidden_size))
        outputs, (ht, ct) = self.rnn(x, (h0, c0))

        out = outputs[-1]  # We are only interested in the final prediction
        out = self.bn1(self.fc1(out))
        out = self.relu(out)
        out = F.dropout(out, training=self.training)
        out = self.fc2(out)
        return out
```

## Joint Many-Task using LSTM

Similar to the FNN case, we can model all the securities together. We just need to make the following change in our LSTM model for that.

```python
self.fc2 = nn.Linear(hidden_size, num_securities)
```

## Using intraday models for trading

An important task after this is to convert the predicted price change into action. A popular method is to send a limit buy order if the prediction signal from the model is more than certain threshold. If the signal falls below the threshold after some time, we can choose to keep or cancel the order. Similarly, send a limit sell order if the prediction signal is below a certain threshold on the negative side.

One common method is to have a continuous stream of market data updates coming in. This would include every single relevant market data update happening. For example, you're trading AAPL stock and your model includes AAPL, MSFT, GOOGL, FB and AMZN, you might want to continuously stream each new/cancel order event as well as all the trades happening real time. You can compute the features realtime and feed that to the neural network. The model would output a single 5 min prediction for AAPL. It's important to keep in mind that these predictions are often wrong. However, you want to utilize the correct predicts well - but, we won't know which ones are correct beforehand.

Anyway, once you have a predicted signal, you can use a threshold to check if the signal is strong enough. Send a new order in the direction of the prediction. The following would be the rough python code for such a system:

```python
class TradingSystem(object):
    def trading_logic(self, signal):
        # Check if there is a BUY signal
        if signal > self.threshold and self.current_risk() < self.risk_threshold:
            # Send a new BUY order at the current bid price
            self.send_new_order('B', self.current_bid_price)

            # Cancel existing sell orders - since we have a BUY signal
            self.cancel_sell_orders()
        
        elif signal < -self.threshold and self.current_risk() > -self.risk_threshold:
            # Send a new SELL order at the current ask price
            self.send_new_order('S', self.current_ask_price)

            # Cancel existing buy orders - since we have a SELL signal
            self.cancel_buy_orders()
        
        else:
            # We may want to keep the existing orders
            # or we can cancel them as well
            self.cancel_all_orders()
```

At this point, it's important to note that the above function has certain parameters that affect the trading. More specifically, `threshold` is the most important parameter there. It's often a good practice to iterate through the threshold and other parameters to see what works out well in the PNL space.

The above function might get called 100k-1M times a day for an HFT system. A relatively lower frequency trading system might be able to utilize better pipelining of feature computation as well as more complex models.
