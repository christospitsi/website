---
title: "Support vector machines and customer behaviour classification"
date: 2018-01-04T13:25:28Z
tags: ["R", "Statistical Learning"]
draft: true
---

### SVM TEST 

In this post we will re-examine the `OJ` data set,
by applying this time a *support vector machines* (SVM) model.
In addition to the `ISLR` library,
we import `e1071` library which
provides us with the necessary SVM functions and methods.

After splitting the data into training and test sets,
we fit a SVM classifier to the training data using `cost = 0.01`,
with `Purchase` as the response and all the other variables as predictors.

As we can see, the classifier has a total of 432 support vectors,
215 of which lie on the `CH` side (or the wrong side of the margin) and
217 on the `MM` side (or the wrong side of the margin).

``` r
library(ISLR)
library(e1071)

set.seed(1)
train <- sample(1:nrow(OJ), 800)
OJ.train <-OJ[train, ]
OJ.test <- OJ[-train, ]

svmfit <- svm(Purchase~., data = OJ, subset = train,
                kernel = "linear", cost = 0.01)

summary(svmfit)
```

    ##
    ## Call:
    ## svm(formula = Purchase ~ ., data = OJ, kernel = "linear", cost = 0.01,
    ##     subset = train)
    ##
    ##
    ## Parameters:
    ##    SVM-Type:  C-classification
    ##  SVM-Kernel:  linear
    ##        cost:  0.01
    ##       gamma:  0.05555556
    ##
    ## Number of Support Vectors:  432
    ##
    ##  ( 215 217 )
    ##
    ##
    ## Number of Classes:  2
    ##
    ## Levels:
    ##  CH MM

Using the confusion matrices for the training and test data,
we calculate the errors using the below function.

``` r
error <- function(x){
  return((((x[2] + x[3])/sum(x))*100))
}

pred.train <- predict(svmfit, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 439  78
    ##         MM  55 228

``` r
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ## [1] "The training error rate is: 16.625000%"

``` r
pred.test <- predict(svmfit, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 141  31
    ##        MM  18  80

    ## [1] "The test error rate is: 18.148148%"

### Optimal Cost

Using the `tune()` function we can find the optimal `cost` among the below values.
`Cost` controls the bias-variance trade-off of the model.

The calculated optimal `cost` value is 0.05.

``` r
tune.out <- tune(svm, Purchase~., data = OJ.train,
                 kernel = "linear",
                 ranges = list(cost = c(0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10)))

tune.out$best.model$cost
```

    ## [1] 0.05

We can now calculate the training and test error rates using a SVM and the optimal cost.
As we can see, the training error rate has slightly decreased
whereas the test error rate is the same.

``` r
svmfit.2 <- svm(Purchase~., data = OJ, subset = train,
                kernel = "linear",
                cost = tune.out$best.model$cost,
                scale = FALSE)

pred.train <- predict(svmfit.2, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 437  75
    ##         MM  57 231

    ## [1] "The training error rate is: 16.500000%"

``` r
pred.test <- predict(svmfit.2, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 143  33
    ##        MM  16  78

    ## [1] "The test error rate is: 18.148148%"

### Radial kernel

We will now use a classifier with a radial kernel and cost = 0.01.
The new model has 617 support vectors,
306 of which lie on the `CH` side (or the wrong side of the margin) and
311 on the `MM` side (or the wrong side of the margin).

``` r
svmfit.rad <- svm(Purchase ~ ., data = OJ, subset = train,
                kernel = "radial", cost = 0.01)
summary(svmfit.rad)
```

    ##
    ## Call:
    ## svm(formula = Purchase ~ ., data = OJ, kernel = "radial", cost = 0.01,
    ##     subset = train)
    ##
    ##
    ## Parameters:
    ##    SVM-Type:  C-classification
    ##  SVM-Kernel:  radial
    ##        cost:  0.01
    ##       gamma:  0.05555556
    ##
    ## Number of Support Vectors:  617
    ##
    ##  ( 306 311 )
    ##
    ##
    ## Number of Classes:  2
    ##
    ## Levels:
    ##  CH MM

We calculate the training and test error rates.

``` r
pred.train <- predict(svmfit.rad, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 494 306
    ##         MM   0   0

    ## [1] "The training error rate is: 38.250000%"

``` r
pred.test <- predict(svmfit.rad, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 159 111
    ##        MM   0   0

    ## [1] "The test error rate is: 41.111111%"

We now use the `tune()` function to find the optimal cost and
we create the new model using the calculated cost value.
Based on the results for the training and test error rates,
the classifier with radial kernel performs better than the linear one.

``` r
tune.out.rad <- tune(svm, Purchase ~ ., data = OJ.train,
                 kernel = "radial",
                 ranges = list(cost = c(0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10)))

svmfit.rad.2 <- svm(Purchase~., data = OJ, subset = train,
                kernel = "radial",
                cost = tune.out.rad$best.model$cost)

pred.train <- predict(svmfit.rad.2, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 453  77
    ##         MM  41 229

    ## [1] "The training error rate is: 14.750000%"

``` r
pred.test <- predict(svmfit.rad.2, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 143  29
    ##        MM  16  82

    ## [1] "The test error rate is: 16.666667%"

### Polynomial kernel

We now repeat the above methodology
using a support vector machine with a polynomial kernel and `degree` = 2.

The model with polynomial kernel has 620 support vectors,
306 of which lie on the `CH` side (or the wrong side of the margin)
and 314 on the `MM` side (or the wrong side of the margin).

``` r
svmfit.poly <- svm(Purchase~., data = OJ, subset = train,
                  kernel = "polynomial", cost = 0.01,
                  degree = 2)

summary(svmfit.poly)
```

    ##
    ## Call:
    ## svm(formula = Purchase ~ ., data = OJ, kernel = "polynomial",
    ##     cost = 0.01, degree = 2, subset = train)
    ##
    ##
    ## Parameters:
    ##    SVM-Type:  C-classification
    ##  SVM-Kernel:  polynomial
    ##        cost:  0.01
    ##      degree:  2
    ##       gamma:  0.05555556
    ##      coef.0:  0
    ##
    ## Number of Support Vectors:  620
    ##
    ##  ( 306 314 )
    ##
    ##
    ## Number of Classes:  2
    ##
    ## Levels:
    ##  CH MM

We calculate the training and test error rates.

``` r
pred.train <- predict(svmfit.poly, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 494 306
    ##         MM   0   0

    ## [1] "The training error rate is: 38.250000%"

``` r
pred.test <- predict(svmfit.poly, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 159 111
    ##        MM   0   0

    ## [1] "The test error rate is: 41.111111%"

We now create a new model based on the optimal `cost`.

``` r
tune.out.poly <- tune(svm, Purchase~., data = OJ.train,
                     kernel = "polynomial",
                     degree = 2,
                     ranges = list(cost = c(0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10)))

svmfit.poly.2 <- svm(Purchase~., data = OJ, subset = train,
                    kernel = "polynomial",
                    degree = 2,
                    cost = tune.out.poly$best.model$cost)
```

Finally, we calculate the training and test error rates.

``` r
pred.train <- predict(svmfit.poly.2, OJ.train)
(train.mtx <- table(pred.train, OJ.train[,"Purchase"]))
sprintf("The training error rate is: %f%%", error(train.mtx))
```

    ##           
    ## pred.train  CH  MM
    ##         CH 450  72
    ##         MM  44 234

    ## [1] "The training error rate is: 14.500000%"

``` r
pred.test <- predict(svmfit.poly.2, OJ.test)
(test.mtx <- table(pred.test, OJ.test[,"Purchase"]))
sprintf("The test error rate is: %f%%", error(test.mtx))
```

    ##          
    ## pred.test  CH  MM
    ##        CH 140  31
    ##        MM  19  80

    ## [1] "The test error rate is: 18.518519%"

Based on the resulted test error rates, the model with the highest accuracy
appears to be the one with the radial kernel.

*This analysis is based on Exercise 9.7.8 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
