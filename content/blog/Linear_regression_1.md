---
title: "A simple linear regression model using simulated data"
date: 2017-09-04T22:52:49Z
tags: ["R", "Statistical Learning"]
draft: true
---

In this exercise we create some simulated data and fit a simple linear regression model.
The simulated data are generated using the ```rnorm()``` function with mean 0 and variance 1. The generated data are stored in
vector X.

```{r}
set.seed(1)

x <- rnorm(100, 0, 1)
```

In addition to the data, we generate vector ```eps``` which
will represent the error of the model.  

```{r}
eps <- rnorm(100, 0, sqrt(0.25))
```
We create vector y based on the below linear equation in which β0 = -1 and β1 = 0.5.
The length of y is equal to the length of x vector.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
y = -1 + 0.5X + ε

```{r}
y = -1 + 0.5 * x + eps
length(y)

## [1] 100
```

We can see below the plot of ```x``` and ```y``` values. As expected, we observe a linear relationship between them although not perfectly linear duto to the addition of the error ```eps```.

```{r}
plot(x, y, col = "deepskyblue2",
     xlab = "x", ylab = "y",
     type = "p", pch = 20, cex.axis = 1, cex.lab = 1)
```
<img src="/images/unnamed-chunk-27-1.png" width="100%" style="display: block; margin: auto;" />

### Least Squares model

We fit a least squares linear model using the `lm.fit` function. The calculated parameters are quite close to the actual parameters (-1 and 0.5)

``` r
lm.fit = lm(y~x)
lm.fit
##
## Call:
## lm(formula = y ~ x)
##
## Coefficients:
## (Intercept)            x  
##     -1.0188       0.4995
```
We can compare the generated model to the actual equation by plotting the least squares line
compared to the population regression line. As we can see, the two line are quite close to
each other.

<img src="/images/unnamed-chunk-29-1.png" width="100%" style="display: block; margin: auto;" />

*This analysis is based on Exercise 3.13 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
