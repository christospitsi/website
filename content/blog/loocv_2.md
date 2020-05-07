---
title: "Cross validation on a simulated dataset"
date: 2017-10-30T10:06:06Z
tags: ["R", "Statistical Learning"]
draft: true
---

For the purposes of this exercise,
we generate the simulated data using the below equation:

``` r
set.seed(1)
x <- rnorm(100)
y <- x -2*x^2 + rnorm(100)
```

Our data set consists of 100 values for x (n = 100) and one predictor (p = 1) The function for the calculation of *y* is of the below form:

Y = β<sub>0</sub> + β<sub>1</sub>X + β<sub>2</sub>X<sup>2</sup> + ϵ.
 In our example, β<sub>0</sub> = 0, β<sub>1</sub> = 1, β<sub>2</sub> = −2
 and the error is a randomly generated number.

On the following plot of X against Y we can see that the points seem to have a *quadratic* shape which was what we expected since we used a quadratic equation to calculate the y values.

``` r
plot(x, y, col = "coral2",
    xlab = "X", ylab = "Y",
    type = "p", pch = 20, cex.axis = 0.8, cex.lab = 0.8)
```
<center><img align = "middle" src="/images/unnamed-chunk-5-1.png" style="width: 90%; height: 90%;"/></center>

### LOOCV

We will now calculate the LOOCV test errors for the following models
using least squares:

i. Y = β<sub>0</sub> + β<sub>1</sub>X + ϵ</br>
ii. Y = β<sub>0</sub> + β<sub>1</sub>X + β<sub>2</sub>X<sup>2</sup> + ϵ</br>
iii. Y = β<sub>0</sub> + β<sub>1</sub>X + β<sub>2</sub>X<sup>2</sup> + β<sub>3</sub>X<sup>3</sup> + ϵ</br>
iv. Y = β<sub>0</sub> + β<sub>1</sub>X + β<sub>2</sub>X<sup>2</sup> + 
β<sub>3</sub>X<sup>3</sup> + β<sub>4</sub>X<sup>4</sup> + ϵ

We create a data frame with all values of x and y and
a vector `ls.error` which will store the error values for the different models.
Finally,
using a for loop and function `cv.glm()` from `boot` library,
we calculate the LOOCV estimates for the test MSE of each model.

We can see on the generated plot with the calculated errors that
there is a sharp drop between the linear and quadratic fits
(models i. and ii. respectively).
Moreover, it appears that the use of higher order polynomials
does not improve the accuracy of the model (models iii. and iv.)

``` r
library(boot)
set.seed(1)
df <- data.frame(y, x)

ls.error <- rep(0, 4)

for (i in 1:4){
  glm.fit = glm(y~poly(x, i), data = df)
  ls.error[i] = cv.glm(df, glm.fit)$delta[1]
}
ls.error
```

    ## [1] 7.2881616 0.9374236 0.9566218 0.9539049

``` r
plot(ls.error, col = "coral2",
    xlab = "Polynomial Order",
    ylab = "Test MSE",
    type = "p", pch = 20,
    xaxp = c(1, 4, 1))

axis(1, at = seq(2, 3))
lines(ls.error, type = "o", col = "coral2")
```

<center><img align = "middle" src="/images/loocv.err1-1.png" style="width: 90%; height: 90%;"/></center>

### LOOCV and randomness

As we explained earlier,
*Leave-One-Out Cross Validation (LOOCV)*
keeps a single observation to be used as validation set.
This procedure is repeated n times,
using each time a different observation for testing.

In general, the seed value in R has an effect on all functions
which use some kind of randomness to perform their calculations.
In our calculations above,
we defined at the beginning of the function the seed to be 1.

Function `cv.glm()` estimates the test error
using individual observations as validation set,
one at a time,
picked in random order based on the seed value thus
a different seed will have as a result a different picking order.

We now re-run the above code using a different `set.seed` argument.
As we can see, the estimated test MSEs for all models are the same
regardless the seed value since
we use all observations as validation sets and
the order does not matter.

It is worth noting that a different seed
would have an effect on the test MSE calculation
if we used other cross-validation approaches such as *Validation Set*
where we don't use all observations to test the trained models.

``` r
library(boot)
set.seed(5)
df <- data.frame(y, x)

ls.error <- rep(0, 4)

for (i in 1:4){
  glm.fit = glm(y ~ poly(x, i), data = df)
  ls.error[i] = cv.glm(df, glm.fit)$delta[1]
}
ls.error
```

    ## [1] 7.2881616 0.9374236 0.9566218 0.9539049

The calculated error values show that
the most accurate model with the smallest *LOOCV error* (0.9374)
was the one shown below.
This confirms our expectations since this model approaches the form of the real equation used to generate the simulated data set.

Y = β<sub>0</sub> + β<sub>1</sub>X + β<sub>2</sub>X<sup>2</sup> + ϵ

Statistical significance of the coefficient estimates

We now print the fourth column
in order to assess the *P-Values* for the coefficients of each model.
As we can see, the P-Values for the calculated β<sub>0</sub>,
β<sub>1</sub> and β<sub>2</sub> are sufficiently small so
we can claim that they are significant whereas
the values for the rest of the coefficients
have relatively large values ( &gt;0.05), thus they are insignificant.

``` r
library(boot)
set.seed(1)
df <- data.frame(y, x)

glm.fit = glm(y ~ poly(x, 1), data = df)
summary(glm.fit)$coefficients[, 4]  
```

    ##  (Intercept)   poly(x, 1)
    ## 3.953542e-08 1.923846e-02

``` r
glm.fit = glm(y ~ poly(x, 2), data = df)
summary(glm.fit)$coefficients[, 4]
```

    ##  (Intercept)  poly(x, 2)1  poly(x, 2)2
    ## 2.656229e-29 4.184810e-09 4.584330e-44

``` r
glm.fit = glm(y ~ poly(x, 3), data = df)
summary(glm.fit)$coefficients[, 4]
```

    ##  (Intercept)  poly(x, 3)1  poly(x, 3)2  poly(x, 3)3
    ## 4.995066e-29 4.971565e-09 1.216703e-43 7.843990e-01

``` r
glm.fit = glm(y ~ poly(x, 4), data = df)
summary(glm.fit)$coefficients[, 4]
```

    ##  (Intercept)  poly(x, 4)1  poly(x, 4)2  poly(x, 4)3  poly(x, 4)4
    ## 5.169227e-29 4.590732e-09 1.593826e-43 7.836207e-01 1.930956e-01

*This analysis is based on Exercise 5.8 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
