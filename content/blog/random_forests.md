---
title: "Random forests in all shapes and sizes"
date: 2017-12-02T11:28:37Z
tags: ["R", "Statistical Learning"]
draft: true
---
As presented in <a href="/blog/a-few-hello-world-r-commands-and-boston-housing-prices/">previous post</a>, ```Boston``` data set
provides information regarding the housing values in the suburbs of Boston.

In this post, we will use *random forest* models of different sizes and
taking under consideration different number of predictors,
in order to predict the median house value.

As we can see on the generated plot, the error rate for a model with small number of trees (less than 100) is quite high.
Moreover, by using more trees, the accuracy tends to increase until a certain point (~ 200 trees) where it stabilises.

With regard to the number of predictors used,
it appears that the most accurate model is the one which uses p/2 parameters
whereas the models which use all the parameters (m = p) have the highest error rates.

``` r
library(MASS)
library(randomForest)

set.seed(1)
train <- sample(1:nrow(Boston), nrow(Boston) / 2) #Indexes of training set
boston.test <- Boston[-train, "medv"] #Test Set

a <- matrix(nrow = 500, ncol = 3) #Stores test errors
x <- 1

  for (i in c(13, 13/2, sqrt(13))) { #Number of predictors per tree
    for (n in 1:500) {               #Number of trees
    bag.Boston <- randomForest(medv ~., data = Boston, subset = train,
                             mtry = i, ntree = n, importance = TRUE)

    yhat.bag <- predict(bag.Boston, newdata = Boston[-train, ])
    a[n,x] <- mean((yhat.bag - boston.test)^2) #Test Error
    }
    x <- x + 1
  }

plot(a[,1], type = "l", col = "coral2", ylim = c(10, 20),  
     ylab = "Test MSE", xlab = "Number of Trees")

lines(a[,2], type = "l", col = "blue")
lines(a[,3], type = "l", col = "green")

legend(300, 20, c("m = p", "m = p/2", "m = sqrt(p)"),
       lty = c(1, 1, 1), col = c("Coral2", "Blue", "Green"))
```

<img src="/images/unnamed-chunk-1-10.png" style="display: block; margin: auto;" />

*This analysis is based on Exercise 8.4.7 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
