# Pairs trading simulator viewed as part of a high frequency trading system

-------------------
Important EDIT: 

My simulations do not take into account: 

1. real world covariates (signal/noise ratio is very small in finance so this impacts simulations a lot since I am making this ratio much bigger than it would really be)
2. market impact (say for example someone executes an large buy order which dramatically drives the price up, unless maybe this order is split into smaller chunks aka iceberg), or to put it differently in terms of execution price, the celebrated saying that "the price you observe is not necessarily the price you're gonna get"
3. short selling margin

This is why, in `strategy-on-real-data.ipynb` (based on data/ideas from https://www.kaggle.com/datasets/yekahaaagayeham/stocks-listed-on-nifty-500-july-2021/versions/9?resource=download), I attempt to test my basic strategy on a pair of stocks using real data, rather than synthetic (simulated) data. That file still does not get around all the 3 items above, but at least it is real data as opposed to simulated data, which means I dealt with item 1. If the market impact caused by the orders in the strategy is small, then item 2 is dealt with as well. I am not considering the short selling margin (item 3) here, leaving it for future work - although my risk limits in `display_animation.ipynb` could allow for a margin as far as I am concerned. Any other practicalities that I omitted are welcome!

Another question is related to the exact reason why mean-reversion appears. Since correlation and cointegration are not the same thing (one can find an example where two time series are highly correlated but not cointegrated, and two time series which are cointegrated that have close to zero correlation), it means that probably the reason is not a high correlation. I think that detecting a pair is fundamentally hard, since mean-reversion has a lot to do with how much intrinsically two time series relate to each other, as opposed to high correlation - therefore I suspect mean-reversion is not caused by high correlation but further evidence is required for this claim, instead cointegration is more relevant as well as intrinsic factors. If we have say a future (ES) and an ETF (SPY) tracking the same index (S&P), then their prices should be roughly the same (if the interest rate r is relatively small making e^(rt) ~ 1). So supposing that SPY is abnormally more expensive than ES, the market is expected to short sell SPY and long ES, pushing the prices close to each other again. In terms of cointegration, this just means that the ratio mean-reverts to 1. But this has nothing to do with correlation, but with the fact that the two are tracking the same index and implicitly the market exploitation of this. Correlation is also not reliable as one can look at short term correlation (which can be negative) or long term correlation (which can be positive) for the same two time series: ((20,10), (20 + 5, 10 - 1), (25 - 1 = 20 + 5, 9 + 5 = 10 + 4), (24 + 5 = 25 + 5, 14 - 1 = 9 + 4), etc) is an example where the correlation is negative  (when one moves up the other goes down)  if you only look at the next time step (LHSs), but it is positive if you look two steps ahead (RHSs). Correlation is also not constant over time: take say two stocks, one lead, one lag, and any lead's movement is a trading signal for the lag, but after this has been exploited, the lead will become the lag and the lag lead. This remains to be investigated further: what exactly causes mean-reversion and use this to find pairs in a systematic way (for now we can test for cointegration which is fine)

Another way is to just take say A, B, find the beta by regressing A on B, so A = a + beta * B, and trade based on the spread A - beta * B. Under normality assumption (wrong), this spread, i.e. the epsilon in linear regression, is assumed to have mean 0, so this spread would revert around the mean 0, and so just short A, long B when this spread is above 2 * sigma, and long A, short B when this is less than - 2 * sigma. The nice thing about this is the stationarity of the spread resulting from the linear regression assumption(not necessarily met), that epsilon is a mean-zero normal with variance sigma^2, making this another way to make the pair strategy work. 

## Comparisons between the two strategies (numbers computed by me, not previously done in the kaggle article)

My strategy:
- horizon: long
- large-volume buy/sell orders since I enter positions less frequently (potentially having high market impact)
- limits value of the short position by always being just half of the total available investment amount
- Sharpe: 0.060
- maximal drawdown: 0%
- Dolar PnL at the end: ~ (6/5) * (2 * value of first short sell trade)

Strategy from Kaggle article:
- horizon: short
- small-volume buy/sell orders since it enters positions much more frequently, each time volume being 1 (market impact is significantly smaller due to iceberg-like orders)
- does not take into account short sell margin at all
- Sharpe: 0.053
- maximal drawdown: -91%  
- Dolar PnL at the end: 819

It seems that the two strategies are comparable, and I would choose mine if I were to trade mid-to-long term, and the one from the Kaggle article if I were to do a short term strategy.

Note that the daily Sharpe 0.06 corresponds to roughly a yearly Sharpe of 1 (multiplying 0.06 by $\sqrt{250}$), which is a reasonable Sharpe ratio, considering that in practice 0.3-0.4 is quite good a metric both for Sharpe but also for R^2 (too-good-to-be-true measures often give a sign of look-ahead bias, which can be hidden in situations like linearly-interpolating as a way to impute data in place of NaNs!)

-------------------


In this repository I am myself implementing a simulator of the pairs trading strategy. 

The system is envisioned as being a high frequency trading system, hence I want the readers to imagine that there is an HFT system that does all the hidden work .

To understand how to develop such a system, I read chapter 5 of the book 'All About High Frequency Trading'.

I heavily commented each relevant line of code with the corresponding thing that happens behind the scenes.

## Details about the HFT system that I envisioned when coding this - followed terminology from the book

At step 'i' of the animation, the exchange sends information to the listener with regards to the current prices of the stocks.

The pricer calculates the ratio of the stocks, once it receives information about them from the listener. It also calculates current mean and standard deviation of the ratios seen so far.

Once the new market data is sent to the traders, they can now submit orders back to the exchange by using the pairs trading strategy.

The cycle continues for a finite specified number of steps in my program: Exchange -> Listener -> Pricer -> Trader -> Exchange.

The latency due to the message passing between these 4 major components is assumed to be 0 in my program.


## Details about the pairs trading strategy 

The pairs trading strategy assumes that we have two reasonably highly correlated stocks.

We have to take the ratio of the two stocks and use mean reversion.

When we see a peak in the ratio graph, we short on the numerator and long on the denominator WITH THE SAME DOLLAR AMOUNT.

When we see a trough in the ratio graph, we long on the numerator and short on the denominator WITH THE SAME DOLLAR AMOUNT.

### Intuition:
    in either case, we expect to gain more from the better position than we lose from the worse position, so overall there is a profit.

Why does this work?

WLOG we are in the first case:

    a0 / b0 is a "peak"
    
    Choose a1 / b1 "close" to the "mean".
    
    a0 * m = b0 * n
    
    Net position: -m * (a1 - a0) + n * (b1 - b0) =
                = n * b1 - m * a1 
                > 0 because n / m = a0 / b0 > a1 / b1

## Experimenting yourself

Animation can be run with parameters at your own choice, by setting the parmeters "corr", "mean", "std", and "num_points" in the main file. It is interesting to see how my strategy performs when the stocks have low correlation, such as 0.2, or even 0 correlation, although the latter is not supposed to happen in practice because the stock should track the etf.
This can be researched further by trying different ways of generating the stock movements: for example, allowing more oscillatory movement in the stocks, or allowing frequent timeframes of alternating increase/decrease in the trend rather than only an increasing trend.
