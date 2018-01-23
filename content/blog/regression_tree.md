---
title: "Predicting sales using regression trees and bagging"
date: 2017-12-14T12:15:54Z
tags: ["R", "Statistical Learning"]
draft: true
---

`Carseats` is a simulated data set
containing sales of child car seats at 400 different stores and various predictors
(please see <a href="https://www.rdocumentation.org/packages/ISLR/versions/1.2/topics/Carseats">documentation</a> for more details).

We will examine how classification trees, bagging and random forest models
can be applied to predict the sales of this product.

We load the necessary libraries and then we split the data set into a training and a test set

``` r
library(tree)
library(randomForest)
library(ISLR)

train <- sample(1:nrow(Carseats), 200)
Carseats.test <- Carseats[-train,]
test <- Carseats[-train, "Sales"]
```

We fit a regression tree and then we plot it as shown below

``` r
tree.carseats <- tree(Sales~., data = Carseats, subset = train)
plot(tree.carseats)
text(tree.carseats, pretty = 0, cex = 0.6)
```

<img src="/images/car_2-1.png" style="display: block; margin: auto;" />

The resulted tree has 18 terminal nodes (leaves) and
the most important predictors appear to be *ShelveLoc* and *Price*.
In general, the predicted *Sales* tend to be higher when the carseat shelve location
is *Good* (right-hand branch).

Below we calculate the predicted *Sales* and then the test error rate.

``` r
tree.pred <- predict(tree.carseats, Carseats.test)
tree.error <- mean((tree.pred - test)^2)
sprintf("The test error rate of the unpruned tree is: %f%%",
        tree.error) #Test set MSE
```

    ## [1] "The test error rate of the unpruned tree is: 4.148897%"

### Optimal level of tree complexity

Using the `cv.tree()` function we can determine the optimal level of the tree complexity.

``` r
cv.sales <- cv.tree(tree.carseats)
cv.sales
```

    ## $size
    ##  [1] 18 17 16 15 14 12 11 10  9  8  7  6  5  4  3  1
    ##
    ## $dev
    ##  [1] 1094.537 1085.178 1089.409 1069.294 1069.294 1075.806 1044.469
    ##  [8] 1039.212 1039.212 1041.308 1043.459 1064.162 1119.046 1120.344
    ## [15] 1270.012 1560.273
    ##
    ## $k
    ##  [1]      -Inf  15.48181  15.53599  18.69038  18.74886  21.05038  23.79480
    ##  [8]  25.78579  26.01210  30.10435  32.74801  53.28569  72.33061  78.19599
    ## [15] 141.73781 251.22901
    ##
    ## $method
    ## [1] "deviance"
    ##
    ## attr(,"class")
    ## [1] "prune"         "tree.sequence"

The optimal size of the tree is, as determined below, a tree with 10 terminal nodes.

``` r
which.min(cv.sales$dev) #Index of the tree with the smallest error
```

    ## [1] 8

``` r
cv.sales$size[which.min(cv.sales$dev)] #Tree size with smaller error
```

    ## [1] 10

We now create the pruned tree and calculate the test error rate using this new model.

As we can see,
the new test error rate is 4.81 which is higher from the test error rate of the unpruned tree.
This means that the model became less complex thus more interpretable,
at the expense of accuracy.

``` r
prune.carseats <- prune.tree(tree.carseats,
                             best = cv.sales$size[which.min(cv.sales$dev)])

tree.prune <- predict(prune.carseats, Carseats.test)
prune.error <- mean((tree.prune - test)^2)
sprintf("The test error rate of the pruned tree is: %f%%",
        prune.error)
```

    ## [1] "The test error rate of the pruned tree is: 4.819708%"

### Bagging

We will now use the *bagging* approach in order to analyse the data.

*Bagging* is simply a special case of *random forest* in which
we use all the predictors hence the *mtry* parameter in our model will be equal to 10.

We calculate the test error rate using the same methods as before.
As we can see, the error rate has been decreased significantly.

``` r
bag.carseats <- randomForest(Sales ~ ., data = Carseats, subset = train,
                           mtry = 10, importance = TRUE)

bag.pred <- predict(bag.carseats, Carseats.test)
bag.error <- mean((bag.pred - test)^2)
sprintf("The test error rate of the bagging model is: %f%%",
        bag.error)
```

    ## [1] "The test error rate of the bagging model is: 2.604369%"

The `importance()` function returns the most important parameters.
In our example, the most important parameters are *Price* and *ShelveLoc*.

``` r
importance(bag.carseats)
```

    ##                %IncMSE IncNodePurity
    ## CompPrice   14.4124562    133.731797
    ## Income       6.5147532     74.346961
    ## Advertising 15.7607104    117.822651
    ## Population   0.6031237     60.227867
    ## Price       57.8206926    514.802084
    ## ShelveLoc   43.0486065    319.117972
    ## Age         19.8789659    192.880596
    ## Education    2.9319161     39.490093
    ## Urban       -3.1300102      8.695529
    ## US           7.6298722     15.723975

### Random forests

We will now use the *random forests* method to analyze the data.

First we create a model using sqrt(p) parameters.
The most important parameters on the generated model appear to be *ShelveLoc*, *Price* and *Age*.

``` r
    forest.carseats1 <- randomForest(Sales ~., data = Carseats,
                                     subset = train,
                                     mtry = sqrt(10),
                                     ntree = 500, importance = TRUE)

    importance(forest.carseats1)
```

    ##                %IncMSE IncNodePurity
    ## CompPrice    7.5233429     127.36625
    ## Income       4.3612064     119.19152
    ## Advertising 12.5799388     138.13567
    ## Population  -0.2974474     100.28836
    ## Price       37.1612032     383.12126
    ## ShelveLoc   30.3751253     246.19930
    ## Age         16.0261885     197.44865
    ## Education    1.7855151      63.87939
    ## Urban       -1.3928225      16.01173
    ## US           5.6393475      32.85850

In order to determine the effect of the number of the parameters used on a random forest method,
we create all possible models and then plot the errors.

As we can see,
in general,
when we use more parameters the test error rate tends to decrease.

``` r
a <- matrix(nrow = 10) #Vector to store test errors

for (i in 1:10) {
    forest.carseats <- randomForest(Sales ~., data = Carseats,
                                    subset = train,
                                    mtry = i, ntree = 500,
                                    importance = TRUE)

    yhat.carseats <- predict(forest.carseats, Carseats.test)
    a[i] <- mean((yhat.carseats - test)^2)
}
plot(a, type = "b",
     ylab = "Test Error Rate", xlab = "Number of Parameters")
```
<center><img align = "middle" src="/images/car_11-1.png" style="width: 80%; height: 80%;"/></center>

*This analysis is based on Exercise 8.4.8 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
