---
layout: post
title: "Faster Tetrachoric Correlations"
date: 2013-11-07 16:48
comments: true
categories: [R, correlations]
---

## What are tetra- and polychoric correlations?

Polychoric correlations estimate the correlation between two theorized normal distributions given two ordinal variables.  In psychological research, much of our data fits this definition.  For example, many survey studies used with introductory psychology pools use Likert scale items.  The responses to these items typically range from 1 (Strongly disagree) to 6 (Strongly agree).  However, we don't *really* think that person's relationship to the item is actually polytomous.  Instead, it's an imperfect approximation.

Similarly, tetrachorics are special cases of polychoric crrelations when the variable of interest is dichotomous. The participant may have gotten the item either correct (i.e., 1) or incorrect (i.e., 0), but the underlying knowledge that led to the items' response is probably a continuous distribution.

When you have polytomous rating scales but want to disattenuate the correlations to more accurately estimate the correlation betwen the latent continuous variables, one way of doing this is to use a tetrachoric or polychoric correlation coefficient.

## The problem

At the [SAPA Project](https://sapa-project.org), the majority of our data is polytomous.  We ask you the degree to which you like to go to lively parties to estimate your score on latent *extraversion*.  Presently, we use `mixed.cor()`, which calls a combination of the `tetrachoric()` and `polychoric()` functions in the `psych` package (Revelle, W., 2013).

However, each time we build a new dataset from the website's SQL server, it takes *hours*.  And that's **if** everything goes correctly.  If there's an error in the code or a bug in a new function, it may take hours to hit the error, wasting your day.

After a bit of profiling, it was revealed that much of our time building the SAPA dataset was used estimating the tetrachoric and polychoric correlation coefficients.  When you do this for 250,000+ participants for 10,000+ variables, it takes *a long time*.  So Bill and I thought about how we could speed them up and feel others may benefit from our optimization.

<!-- more -->

### Optimizing the existing function for a 300% speed up

If we call `psych:::tetraBinBvn`, we see that most of the time spent computer the tetrachorics is using `pmvnorm` to estimate each of the 4 probabilities of the tetrachoric:

```r
psych:::tetraBinBvn
function (rho, rc, cc) 
{
    row.cuts <- c(-Inf, rc, Inf)
    col.cuts <- c(-Inf, cc, Inf)
    P <- matrix(0, 2, 2)
    R <- matrix(c(1, rho, rho, 1), 2, 2)
    for (i in 1:2) {
        for (j in 1:2) {
            P[i, j] <- pmvnorm(lower = c(row.cuts[i], col.cuts[j]), 
                upper = c(row.cuts[i + 1], col.cuts[j + 1]), 
                corr = R)
        }
    }
    P
}
<environment: namespace:psych>
```

{% img http://gradstudents.wcas.northwestern.edu/~jaf502/images/tetra.png %}

The function says for each row *i* and each column *j*, optimize a probability based on the lower and upper bounds of the row cuts and column cuts.  **But wait!**  Since the sum of the probabilities in the tetrachoric 2x2 matrix add to 1, why not just use a bit of subtraction to figure it out?  That would avoid waiting on `pvnorm` to compute each cell.

This solution resulted in a speed up of `tetrachor()` by a factor of 3!  See our benchmarks below!

Unfortunately, our trick doesn't work for the `polychor()` function as well, since there are more degrees of freedom. However, we can eliminate calling `pvnorm()` for 11 cells in a hypothetical 6x6 matrix as we only need to find the values for 25 and can use subtraction to speed up the estimation of the remaining 11.  This results in a less impressive speed up for `polychor()` that I nonetheless hope will benefit researchers.


## Testing our result


### Generate our hypothetical dataset

```r
library(psych)
data <- sim.irt(nvar = 50, n = 500)
```


### Profiling old code

```r
system.time(old <- polychoric(data$items))
```

```
##    user  system elapsed 
##  16.733   0.097  17.286
```

### Profiling new code

```r
system.time(new <- polychoric(data$items))
```

```
##    user  system elapsed 
##   6.336   0.035   6.537
```


### Differences between new and old results

```r
round(new$rho - old$rho, 2)
```

```
##       V1 V2    V3 V4 V5 V6 V7   V8 V9 V10 V11 V12 V13 V14 V15 V16 V17 V18
## V1  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V2  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V3  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V4  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V5  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V6  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V7  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V8  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V9  0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
## V10 0.00  0  0.00  0  0  0  0 0.00  0   0   0   0   0   0   0   0   0   0
...
```