# Pairs trading simulator viewed as part of a high frequency trading system

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

Animation can be run with parameters at your own choice, by setting the parmeters "corr", "mean", "std", and "num_points" in the main file. It is interesting to see how my strategy performs when the stocks have low correlation, such as 0.2, or even 0 correlation, although this is not supposed to happen in practice because the stock should track the etf.
