# Estimating experimental power
A lot of the measure's I work with on a day to day basis are binomial.
In this case, I am trying to estimate what proportion of Windows Firefox users are on a given version of Windows.
The first question I ask myself is, "how many observations do I need?"

It's silly to query over the entire population, because the computation will slow down your iteration speed.
On the other hand, using a sample of 100 windows users will only give you a rough estimate of the actual distribution.
How can we formalize how many clients we should sample?

Here's some quick and dirty R code to get a confidence interval for a binomial test of size N.
We always test at p=50% because this is where the variance is going to be the widest.

```R
library(binom)

get.confint <- function(N) {
    binom.confint(floor(0.5*N), N, method="agresti-coull")
}
```

Testing some sample sizes shows that a 10k sample shoud keep our estimates within +/-1% of the true value:

```
> get.confint(100)

         method  x   n mean     lower     upper
         1 agresti-coull 50 100  0.5 0.4038315 0.5961685

> get.confint(1000)

         method   x    n mean     lower     upper
         1 agresti-coull 500 1000  0.5 0.4690696 0.5309304

get.confint(10000)

         method    x     n mean     lower     upper
         1 agresti-coull 5000 10000  0.5 0.4902021 0.5097979
```

Intuitively, this makes sense.
The sample variance of the mean should be sqrt(0.5^2/N), which is 0.005 at N=10k.
With +/- 2*sigma, we get a clean +/-0.01.

This snippet is more useful when dealing with probabilities closer to 1 or 0 with smaller sample size.
In this case, we can get accurate intervals for strance occurances.
