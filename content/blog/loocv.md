---
title: "S&P 500 stock exchange - A 'for loop' LOOCV approach"
date: 2017-10-18T23:24:14Z
tags: ["R", "Statistical Learning"]
draft: true
---
The Leave-one-out cross validation test error can be estimated using the ```R``` build-in
function ```cv.glm```, however, it is worth exploring an alternative solution,
using functions ```glm```, ```predict.glm``` and a for loop.
For the purposes of this exercise,
we will revisit the percentage returns for the S&P 500 stock index included in
<a href="/blog/sp-500-stock-exchange---logistic-regression-model">Weekly dataset.</a>

We first fit a logistic regression model which predicts *Direction*
using *Lag1* and *Lag2* variables and the whole dataset as training set.

``` r
weekly.fits <- glm(Direction ~ Lag1 + Lag2, family = binomial, data = Weekly)
summary(weekly.fits)
```

    ##
    ## Call:
    ## glm(formula = Direction ~ Lag1 + Lag2, family = binomial, data = Weekly)
    ##
    ## Deviance Residuals:
    ##    Min      1Q  Median      3Q     Max  
    ## -1.623  -1.261   1.001   1.083   1.506  
    ##
    ## Coefficients:
    ##             Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)  0.22122    0.06147   3.599 0.000319 ***
    ## Lag1        -0.03872    0.02622  -1.477 0.139672    
    ## Lag2         0.06025    0.02655   2.270 0.023232 *  
    ## ---
    ## Signif. codes:  0 *** 0.001 ** 0.01 * 0.05 '.' 0.1 ' ' 1
    ##
    ## (Dispersion parameter for binomial family taken to be 1)
    ##
    ##     Null deviance: 1496.2  on 1088  degrees of freedom
    ## Residual deviance: 1488.2  on 1086  degrees of freedom
    ## AIC: 1494.2
    ##
    ## Number of Fisher Scoring iterations: 4


We now fit a logistic regression model which predicts *Direction*
using *Lag1* and *Lag2* and all but the first observation as training set.

The generated model predicts that the first observation will go "Up",
based on the fact that P(Direction = "Up" | Lag1, Lag2) = 0.571 which is greater than 0.5.
The predicted value is incorrect since the real value for than particular observation is -0.27
which means that the market moved downwards.

``` r
loocv.fits <- glm(Direction ~ Lag1 + Lag2,
                  family = binomial,
                  data = Weekly[-1,])

loocv.prob <- predict(loocv.fits, Weekly[1, ], type = "response")
loocv.prob
```

    ##         1
    ## 0.5713923

``` r
Weekly[1, 8]
```

    ## [1] -0.27

### For-loop LOOCV

The below loop is executed n times.
Each time,
the algorithm fits a logistic regression model and
estimates the error using validation set approach, each time with a different observation as validation set.
The average of those errors is, by definition, the LOOCV test estimation.

The LOOCV estimated test MSE is 44.99%, a result which shows that our model is slightly better than random guessing.

``` r
prob.dir <- rep("Up", 1089)
prob.err <- rep(0, 1089)
prob.loop <- rep(0, 1089)

for (n in 1:nrow(Weekly)) {

   loocv.loop <- glm(Direction ~ Lag1 + Lag2,
                  family = binomial,
                  data = Weekly[-n, ])
    prob.loop[n] <- predict(loocv.loop, Weekly[n, ], type = "response")

    if (prob.loop[n] < 0.5) {
            prob.dir[n] <- "Down"
    }

    if (prob.dir[n] == Weekly[n, 9]) {
            prob.err[n] <- 1
    }

err <- (1 - mean(prob.err))*100
}

sprintf("LOOCV estimate for test MSE: %g%%",
        err)
```

    ## [1] "LOOCV estimate for test MSE: 44.9954%"

*This analysis is based on Exercise 5.7 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
