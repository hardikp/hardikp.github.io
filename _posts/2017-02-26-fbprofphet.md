---
layout: post
comments: true
title:  "Prophet - Time series prediction"
excerpt: ""
date:   2017-02-26 23:00:00
mathjax: false
---

### Brief history

Time series prediction is one of the most common statistical problems over the last 30-50 years. [AutoRegressive (AR)](https://en.wikipedia.org/wiki/Autoregressive_model) models and [Moving Average (MA)](https://en.wikipedia.org/wiki/Moving-average_model) models were some of the earliest models to be used for time series prediction.

[AutoRegressive Moving Average (ARMA)](https://en.wikipedia.org/wiki/Autoregressive%E2%80%93moving-average_model) and [AutoRegressive Integrated Moving Average (ARIMA)](https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average) models became more popular over the years.

[statsmodels](http://www.statsmodels.org/stable/tsa.html) is a great library for the above time series models.

### Prophet
Prophet uses [stan]() models for prediction.

### Show me the code!
```python
from fbprophet import Prophet
import pandas as pd

# Read the SPY_notional.csv file
# Prophet expects the days/time to be in the 'ds' column and the values in the 'y' column.
df = pd.read_csv('SPY_notional.csv')
df = df.rename(columns={'Unnamed: 0': 'ds', 'SPY': 'y'})

# Fit the Prophet model
df['cap'] = 3.5 * 1e10
m = Prophet(growth='logistic')
m = Prophet()
m.fit(df)

# Predict the notional volume for the next 60 days
future = m.make_future_dataframe(periods=60)
future['cap'] = 3.5 * 1e10
forecast = m.predict(future)
forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail()

# Plot
m.plot(forecast)
```

<img src="/assets/prophet.png">
