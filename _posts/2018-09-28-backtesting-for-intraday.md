---
layout: post
comments: true
title: "Backtesting for Intraday Execution"
excerpt: "Intraday execution involves buying or selling a certain quantity of shares in a given time period. Backtesting is really important in trying to improve execution algorithms. This post explores a backtesting for a simplified scenario."
date:   2018-09-28 12:00:00
mathjax: false
image_url: "/assets/intraday_backtesting/execution_problem.jpg"
---

* TOC
{:toc}

**Intraday execution** involves buying or selling a certain quantity of shares in a given time period. For example, you want to buy 1000 shares of AMZN stock today. You have the entire day to buy. If your goal is to a get a good price on average, what would be your strategy to buy?

![Execution Problem]({{ site.url }}/assets/intraday_backtesting/execution_problem.jpg)

## Simple Methods to Execute Our Order

There are many ways to go about this. A simple method is to simply divide your 1000 sized order into 100 sized 10 orders - and execute each of those orders at a fixed time interval. This is commonly referred to as **TWAP execution**.

Another method can be to wait for the stock price to go down for a few cents and then buy all 1000 shares in a single go. This can be done either through an aggressive order (an aggressive limit order or a market order) or you can simply enter a passive limit order and wait for it to get executed in some time. If we can get this low price to buy, it's certainly a very good thing for us. However, there is a risk that the prices can continue to go up the entire day. In that case, we may end up buying a much higher price later in the day.

You can come up with many such strategies (or algorithms) to buy 1000 shares. But, the question is: How do you know if your execution algorithm is any good? What if it's based on a bunch of hypotheses that don't hold up in a real situation? Are you willing to bet on it?

## Backtesting

The best tool we have to be confident up to a certain degree is to **backtest** our execution algorithm very well. If we can see how our algorithm performed in various situations in the past, we can be more confident about using it in real situations.

I am going to describe one way to backtest execution algorithms. It involves a number of assumptions. If any assumption doesn't work, you would likely not get a good backtest result. i.e. your backtest will differ significantly from what the real buy/sell price would have been.

Challenges in backtesting execution algorithms:
* By placing orders and buying/selling shares, you're affecting the market. You can't fully understand how the other participants in the market will react to your orders.
* 

Assumptions:
* We will avoid shares that do not trade much. Illiquid securities can behave very differently to your orders. A single order/trade can make a lot of effects there.
* We will cap the order size to less than 1% of the average volume in the given time period. For institutions, this is a very big assumption. You often have to buy/sell quite a lot - and the order size can be larger than 1%.
* We have access to timestamped tick data for the last few years.

## A simple backtesting logic

We're going to implement a very simple backtesting logic in python. But, here's the two line summary: "Backtester maintains the list of buy and sell orders waiting to be executed. On each market event, Backtester checks if any outstanding buy/sell orders would have gotten executed at this point in time and assigns appropriate trade for that buy/sell order."

NOTE: Usable minimal backtester would be more complex than what we will do here today. My goal is to highlight various nuances, but not cover all of them.

A common way to set up our backtesting is to have an event based setup. Each market update event is passed to the execution algorithm as well as the backtester. On each event, execution algorithm decides whether to send an order, modify an existing limit order or cancel an existing limit order. On each event, backtester decides whether to assign a fill to the list of live orders or not.

So, the backtester has inputs from (1) Execution algorithm and (2) Market (in the form of market events).

Let's break our backtester stages into 2 parts:
1. Maintain bids and asks
2. Process each market event to assign fills

## 1. Maintain bids and asks

```python
class Backtester(object):
    """
    Backtester tries to act as a proxy for the real exchange.
    Execution algorithms can send orders and expect trades in response to them.
    """

    def __init__(self):
        self._bids = []
        self._asks = []
```

However, maintaining a list of buy and sell orders is more than simply creating empty lists of `bids` and `asks`. We will add `send_order`, `cancel_order` and `modify_order` methods to complete this first part. We will also need a way to represent our order - so, we will add `Order` class.

Let's start with the `Order` class.
```python
class Order(object):
    def __init__(self, buysell, size, price, order_id):
        self.buysell = buysell
        self.size = size
        self.price = price
        self.order_id = order_id
```

Here's how we will handle `send_order` event. Execution algorithm would call this function to send a limit order to the backtester. For simplicity, I am skipping other order types. In `send_order`, we will simply create a new Order object.

```python
    def send_order(self, buysell, order_id, size, price):
        """
        Execution Algorithm uses the send_order function to send limit orders
        to the exchange/backtester.
        """
        order = Order(buysell, size, price, order_id)
        if buysell == 'BUY':
            self._bids.append(order)
        else:
            self._asks.append(order)
```

`cancel_order` tries to see if the order we're supposed to cancel is in our list or not. If it's there, we will cancel it.

Note: In reality, the exchange takes its time to receive the cancel order request and respond with a delay. It's crucial to incorporate that in our backtester, but I have skipped it for simplicity purposes.

```python
    def cancel_order(self, buysell, order_id):
        """
        Cancel an existing limit order.
        """
        if buysell == 'BUY':
            for order in self._bids:
                if order.order_id == order_id:
                    self._bids.remove(order)
                    break
        
        else:
            for order in self._asks:
                if order.order_id == order_id:
                    self._asks.remove(order)
                    break
```

`modify_order` will try to modify an existing order to the new size and new price. For simplicity, we will assume we don't have partially executed orders. i.e. a 100 sized order is either fully executed and deleted from our `_bids` and `_asks` lists or it's not executed at all.

```python
    def modify_order(self, buysell, order_id, new_size, new_price):
        """
        Modify an existing limit order.
        """
        if buysell == 'BUY':
            for order in self._bids:
                if order.order_id == order_id:
                    order.size = new_size
                    order.price = new_price
                    break
        
        else:
            for order in self._asks:
                if order.order_id == order_id:
                    order.size = new_size
                    order.price = new_price
                    break
```

## 2. Process each market event to assign fills

We will process each market event to check if any of our open orders would have have been traded as a result of this event. Each event consists of [bid_size, bid_price, ask_price, ask_size]. For simplicity, we're only considering the top levels. `bid_price` indicates the highest price for a buy order. `ask_price` indicates the lowest price for a sell order.

Let's consider what conditions would cause a trade. We want to be more conservative here. Example: Current `bid_price` is 100, current `ask_price` is 102. We'll denote this market as `[100 * 102]`.
* If we have a buy limit order with price 100: **NO TRADE**. There is no sell order at price 100. Lowest sell order price is 102.
* If we have a buy limit order with price 102: **TRADE** - because there is a sell order at the same price in the market.
* If we have a sell limit order with price 100: **TRADE** - because there is a buy order at the same price in the market.
* If we have a sell limit order with price 102: **NO TRADE**. There is no buy order at price 102. Highest buy order price is 100.

We can use this insight to handle the fills/trades in our backtester. Here's the code for that.

```python
    def on_market_update(self, timestamp, bid_size, bid_price, ask_price, ask_size):
        """
        This is called whenever there is a new market update.
        NOTE: We're ignoring trade messages for simplicity.
        """
        bids_to_delete = []
        for index, order in enumerate(self._bids):
            # Check for a trade condition
            # Example: bid order price = 99, market = [95 * 99]
            # 99 priced order would get matched against 99 ask_price from the market.
            if order.price >= ask_price:
                self.notify_trade(order)

                # We will delete this later in this function
                bids_to_delete.append(index)
        
        # Delete executed bids
        for index in bids_to_delete[::-1]:
            self._bids.remove(self._bids[index])
        
        asks_to_delete = []
        for index, order in enumerate(self._asks):
            # Check for a trade condition
            # Example: ask order price = 99, market = [100 * 102]
            # 99 priced order would get matched against 100 bid_price from the market.
            if order.price <= bid_price:
                # Trade
                self.notify_trade(order)

                # We will delete this later in this function
                asks_to_delete.append(index)
        
        # Delete executed asks
        for index in asks_to_delete[::-1]:
            self._asks.remove(self._asks[index])
```

## Possible Improvements

1. When execution algorithms send an order, it's not immediately received by the exchange. There is a delay. So, it's usually a good idea to add an appropriate delay in the `send_order`, `cancel_order` and `modify_order` functions. One way to do this is to have a `request_queue` that maintains current send/cancel/modify requests and considers them after the appropriate delay has been passed.
1. This example only uses limit orders. Other types of orders (Market, Fill or Kill, Stop, Stop Limit,...) can be handled with a little extra effort.
1. We're assuming the order gets completely filled or it doesn't get filled at all. This doesn't have to be as binary. Partial execution support can be added by expanding the `Order` class with an additional `size_executed` field. `size_executed` would have to be properly tracked for both cancel/modify order events as well as market update events.
1. We're only filling orders when the price advances beyond the limit order price. This is a conservative approach to estimating when the trade would happen. A better approach involves tracking the position of our order in the bid/ask queue. We can track how much size is before our order and how much size is after our order. That way we can check if our order would have been executed at the current level. An even better approach is to track individual orders (if we have order information) in the backtesting - it's as accurate as it can get.
1. The trading pattern differs significantly based on the type of the security (stocks, ETFs, options, futures, currencies), liquidity, minimum price increment, whether there is an underlying (Futures, ETFs, options) and many other factors. While this makes it hard to write execution algorithm, it also impacts backtesting. We can penalize the execution/trade more if the stock is illiquid and the total trade size is more than a certain % of the average daily volume. We can also incorporate other parameters in a similar way.
