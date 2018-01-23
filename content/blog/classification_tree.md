---
title: "Customer behaviour and classification trees"
date: 2017-12-18T12:39:29Z
tags: ["R", "Statistical Learning"]
draft: true
---

In this example we are going to use `OJ` dataset which contains
1070 purchases where the customer either purchased Citrus Hill or Minute Maid Orange Juice and
a number of characteristics of the customer and product.
(please see <a href="https://www.rdocumentation.org/packages/ISLR/versions/1.2/topics/OJ">documentation</a>
for more details)


Before we examine the possible models,
we create as usual a training set with 800 observations and the test set
with the remaining observations from the `OJ` data set.

``` r
library(ISLR)
library(tree)

set.seed(1)
train <- sample(1:nrow(OJ), 800)
OJ.test <- OJ[-train, ]
```

We now fit a tree to the training data with *Purchase* as the response and the other variables as predictors.

``` r
tree.OJ <- tree(Purchase~., data = OJ, subset = train)
summary(tree.OJ)
```

    ##
    ## Classification tree:
    ## tree(formula = Purchase ~ ., data = OJ, subset = train)
    ## Variables actually used in tree construction:
    ## [1] "LoyalCH"       "PriceDiff"     "SpecialCH"     "ListPriceDiff"
    ## Number of terminal nodes:  8
    ## Residual mean deviance:  0.7305 = 578.6 / 792
    ## Misclassification error rate: 0.165 = 132 / 800

Using the summary function we can produce summary statistics about the tree.
As we can see, the tree has 8 terminal nodes and the training error rate is 16.5%.

In addition to the tree statistics,
by typing the name of the tree we can get a detailed text output which describes
the tree structure and the various parameters for each node/branch.

``` r
tree.OJ
```

    ## node), split, n, deviance, yval, (yprob)
    ##       * denotes terminal node
    ##
    ##  1) root 800 1064.00 CH ( 0.61750 0.38250 )  
    ##    2) LoyalCH < 0.508643 350  409.30 MM ( 0.27143 0.72857 )  
    ##      4) LoyalCH < 0.264232 166  122.10 MM ( 0.12048 0.87952 )  
    ##        8) LoyalCH < 0.0356415 57   10.07 MM ( 0.01754 0.98246 ) *
    ##        9) LoyalCH > 0.0356415 109  100.90 MM ( 0.17431 0.82569 ) *
    ##      5) LoyalCH > 0.264232 184  248.80 MM ( 0.40761 0.59239 )  
    ##       10) PriceDiff < 0.195 83   91.66 MM ( 0.24096 0.75904 )  
    ##         20) SpecialCH < 0.5 70   60.89 MM ( 0.15714 0.84286 ) *
    ##         21) SpecialCH > 0.5 13   16.05 CH ( 0.69231 0.30769 ) *
    ##       11) PriceDiff > 0.195 101  139.20 CH ( 0.54455 0.45545 ) *
    ##    3) LoyalCH > 0.508643 450  318.10 CH ( 0.88667 0.11333 )  
    ##      6) LoyalCH < 0.764572 172  188.90 CH ( 0.76163 0.23837 )  
    ##       12) ListPriceDiff < 0.235 70   95.61 CH ( 0.57143 0.42857 ) *
    ##       13) ListPriceDiff > 0.235 102   69.76 CH ( 0.89216 0.10784 ) *
    ##      7) LoyalCH > 0.764572 278   86.14 CH ( 0.96403 0.03597 ) *

For instance, the line for terminal node 20 shows that under the branch `SpecialCH < 0.5`,
60.89 observations have been classified as `MM` which is the most occurring class on that node.
Moreover, the deviance is 70 and the *class proportions* of `CH` and `MM` are 15.7 and 84.2 respectively.

On the graphical presentation of the tree below,
we can see that it has 7 *internal nodes* and 8 *terminal nodes*.
The most important predictor appears to be the *LoyalCH*
since it have been used to split the data on the first node and
then again on the next two internal nodes.
Additionally, all splits on the right branch have been performed in order to
increase the node *purity* since all terminal nodes have been classified as `CH`.

``` r
plot(tree.OJ)
text(tree.OJ, pretty = 0, cex = 0.7)  
```

<img src="/images/OJ_4-1.png" style="display: block; margin: auto;" />

We will now predict the response on the test data and generate the confusion matrix.

``` r
OJ.pred <- predict(tree.OJ, OJ.test, type = "class")
table(OJ.pred, OJ.test[ ,"Purchase"])
```

    ##        
    ## OJ.pred  CH  MM
    ##      CH 147  49
    ##      MM  12  62

As we see on the confusion matrix above, the correct predictions percentage is:
(62 + 147)/(62 + 147 + 12 + 49) = 77%,
hence the (test) classification error rate is 23%.

### Optimal tree size

We will now apply the `cv.tree()` function
in order to determine the optimal tree size.

On the plotted error rate and the `cv.OJ` printed parameters,
we see that the trees with 8, 5 and 2 terminal nodes have 156 cross-validatiod errors each
which is the lowest cross-validation error rate.

``` r
cv.OJ <- cv.tree(tree.OJ, FUN = prune.misclass)
cv.OJ
```

    ## $size
    ## [1] 8 5 2 1
    ##
    ## $dev
    ## [1] 156 156 156 306
    ##
    ## $k
    ## [1]       -Inf   0.000000   4.666667 160.000000
    ##
    ## $method
    ## [1] "misclass"
    ##
    ## attr(,"class")
    ## [1] "prune"         "tree.sequence"

``` r
plot(cv.OJ$size, cv.OJ$dev, type = "b",
     ylab = "Test Error Rate", xlab = "Tree Size")
```

<img src="/images/OJ_7-1.png" style="display: block; margin: auto;" />

### Pruned trees

We apply the ```prune.misclass()``` method in order to produce pruned trees with 5 terminal nodes.


``` r
prune.OJ <- prune.misclass(tree.OJ, best = 5)
summary(prune.OJ)
```

    ##
    ## Classification tree:
    ## snip.tree(tree = tree.OJ, nodes = 3:4)
    ## Variables actually used in tree construction:
    ## [1] "LoyalCH"   "PriceDiff" "SpecialCH"
    ## Number of terminal nodes:  5
    ## Residual mean deviance:  0.8256 = 656.4 / 795
    ## Misclassification error rate: 0.165 = 132 / 800

We will now predict the response on the test data and produce the confusion matrix,
using the pruned tree model.

``` r
prune.OJ.pred <- predict(prune.OJ, OJ.test, type = "class")
table(prune.OJ.pred, OJ.test[ ,"Purchase"])
```

    ##              
    ## prune.OJ.pred  CH  MM
    ##            CH 147  49
    ##            MM  12  62

As we see on the confusion matrix above,
the classification error rate of the pruned tree is identical to the error rate
of the unpruned tree and equal to 23%.

*This analysis is based on Exercise 8.4.9 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
