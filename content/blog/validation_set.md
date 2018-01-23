---
title: "The credit card problem - Validation set approach"
date: 2017-10-10T22:17:53Z
tags: ["R", "Statistical Learning"]
draft: true
---

*Default* is a simulated data set from ISLR library with information on ten thousand customers. We are interested in predicting whether a customer will default using the following predictors:

-   *balance*: The average balance that the customer has remaining on their credit card after making their monthly payment
-   *income*: Income of the customer.

We fit a logistic regression model which uses *income* and *balance* to predict *default* and then we print a summary with all the calculated values.

``` r
library(ISLR)
default.fits1 <- glm(default ~ income + balance,
                 data = Default, family = binomial)

summary(default.fits1)
```

    ##
    ## Call:
    ## glm(formula = default ~ income + balance, family = binomial,
    ##     data = Default)
    ##
    ## Deviance Residuals:
    ##     Min       1Q   Median       3Q      Max  
    ## -2.4725  -0.1444  -0.0574  -0.0211   3.7245  
    ##
    ## Coefficients:
    ##               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept) -1.154e+01  4.348e-01 -26.545  < 2e-16 ***
    ## income       2.081e-05  4.985e-06   4.174 2.99e-05 ***
    ## balance      5.647e-03  2.274e-04  24.836  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 *** 0.001 ** 0.01 * 0.05 '.' 0.1 ' ' 1
    ##
    ## (Dispersion parameter for binomial family taken to be 1)
    ##
    ##     Null deviance: 2920.6  on 9999  degrees of freedom
    ## Residual deviance: 1579.0  on 9997  degrees of freedom
    ## AIC: 1585
    ##
    ## Number of Fisher Scoring iterations: 8


We will use the validation set approach in order to estimate the test error rate of the model. In this approach, we randomly split the data into a training set and a validation set. We then use the two sets to train and test the model. We will now apply this method by creating a general function `glm.error(x)`, with argument *x* being the size of the training set.

The function first creates the training set using the `sample()` function, the total number of observations (10000) and x as argument, performs logistic regression and then calculates the probabilities vector `probs` for the training set. Finally, it creates the validation set `def.test` which comprises of the observations we didn't use for the training of the model and then calculates the test error of the method.

``` r
glm.error <- function(x) {

#Creates training sample     
set.seed(5)
train <- sample(10000, x)

#Logistic Regression (income,balance)
default.fits <- glm(default ~ income + balance,
                    family = binomial, data = Default, subset = train)

#Creates vector with predictions
probs <- predict(default.fits, Default[-train, ], type = "response")
glm.pred <- rep("No", 10000-x)
glm.pred[probs > 0.5] <- "Yes"

#Creates validation set
def.test <- Default[-train, ]

#Calculates and prints test error
test.error <- (1 - mean(glm.pred == def.test$default))*100
sprintf("Test error with a training sample of %d observations: %g%%",
        x, test.error)
}
```

Using the above function, we perform logistic regression four times, using each time different training set size. As we can see, by using a bigger training set, the generated model performs better and has a smaller test error.

``` r
glm.error(1000)
```

    ## [1] "Test error with a training sample of 1000 observations: 2.61111%"

``` r
glm.error(4000)
```

    ## [1] "Test error with a training sample of 4000 observations: 2.46667%"

``` r
glm.error(6000)
```

    ## [1] "Test error with a training sample of 6000 observations: 2.275%"

``` r
glm.error(7000)
```

    ## [1] "Test error with a training sample of 7000 observations: 2.2%"

### Logistic regression model

We now fit a logistic regression model which predicts the probability of *default* using predictor *student* in addition to *income* and *balance*. For the same training and test sets, the generated results show that predictor student didn't improve the test error since the new model performs slightly worse for all sample sizes.

``` r
glm.stud <- function(x) {

#Creates training sample     
set.seed(5)
train <- sample(10000, x)

#Logistic Regression (income, balance, student)
default.fits <- glm(default ~ income + balance + student,
                    family = binomial, data = Default, subset = train)

#Creates vector with predictions
probs <- predict(default.fits, Default[-train, ], type = "response")
glm.pred <- rep("No", 10000-x)
glm.pred[probs > 0.5] <- "Yes"

#Creates validation set
def.test <- Default[-train, ]

#Calculates and prints test error
test.error <- (1 - mean(glm.pred == def.test$default))*100
sprintf("Test error with a training sample of %d observations: %g%%",
        x, test.error)
}
```

``` r
glm.stud(1000)
```

    ## [1] "Test error with a training sample of 1000 observations: 2.63333%"

``` r
glm.stud(4000)
```

    ## [1] "Test error with a training sample of 4000 observations: 2.58333%"

``` r
glm.stud(6000)
```

    ## [1] "Test error with a training sample of 6000 observations: 2.325%"

``` r
glm.stud(7000)
```

    ## [1] "Test error with a training sample of 7000 observations: 2.3%"

*This analysis is based on Exercise 5.5 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
