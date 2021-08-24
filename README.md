# Pairs-trading-simulator
You can view a nice animation of the pairs trading strategy by choosing whichever parameters you wish.

The pairs trading strategy assumes that we have two reasonably highly correlated stocks.

We have to take the ratio of the two stocks and use mean reversion.

When we see a peak in the ratio graph, we short on the numerator and long on the denominator WITH THE SAME DOLLAR AMOUNT.

When we see a trough in the ratio graph, we long on the numerator and short on the denominator WITH THE SAME DOLLAR AMOUNT.

Intuition:
    in either case, we expect to gain more from the better position than we lose from the worse position, so overall there is a profit.

Why does this work?

WLOG we are in the first case:

    a0 / b0 is a "peak"
    
    Choose a1 / b1 "close" to the "mean".
    
    a0 * m = b0 * n
    
    Net position: -m * (a1 - a0) + n * (b1 - b0) =
                = n * b1 - m * a1 
                > 0 because n / m = a0 / b0 > a1 / b1
