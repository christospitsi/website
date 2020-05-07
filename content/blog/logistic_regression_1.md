---
title: "S&P 500 stock exchange - Logistic regression model"
date: 2017-09-08T00:09:00+02:00
tags: ["R", "Statistical Learning"]
draft: true
---

For the purposes of this post we will use the ```Weekly``` dataset, included in the ```ISLR``` library. The dataset consists of percentage returns
for the S&P 500 stock index between 1990 and 2010.

In the data frame, there are 1089 observations and the following variables:

-   *year*: Year that the observation was recorded
-   *lag1*: Percentage return for previous week
-   *lag2*: Percentage return for 2 weeks previous
-   *lag3*: Percentage return for 3 weeks previous
-   *lag4*: Percentage return for 4 weeks previous
-   *lag5*: Percentage return for 5 weeks previous
-   *volume*: Volume of shares traded (average number of daily shares traded in billions)
-   *today*: Percentage return for this week
-   *direction*: A factor with levels Down and Up indicating whether the market had a positive or negative return on a given week

Below we can see a numerical summary for all predictors:

``` r
library(ISLR)

## Warning: package 'ISLR' was built under R version 3.4.2

sprintf("The Weekly data frame has %d observations and %d predictors",
        nrow(Weekly), ncol((Weekly)))

## [1] "The Weekly data frame has 1089 observations and 9 predictors"          
```

``` r
summary(Weekly)
```

```r
##       Year           Lag1               Lag2               Lag3         
##  Min.   :1990   Min.   :-18.1950   Min.   :-18.1950   Min.   :-18.1950
##  1st Qu.:1995   1st Qu.: -1.1540   1st Qu.: -1.1540   1st Qu.: -1.1580
##  Median :2000   Median :  0.2410   Median :  0.2410   Median :  0.2410
##  Mean   :2000   Mean   :  0.1506   Mean   :  0.1511   Mean   :  0.1472
##  3rd Qu.:2005   3rd Qu.:  1.4050   3rd Qu.:  1.4090   3rd Qu.:  1.4090
##  Max.   :2010   Max.   : 12.0260   Max.   : 12.0260   Max.   : 12.0260
##       Lag4               Lag5              Volume       
##  Min.   :-18.1950   Min.   :-18.1950   Min.   :0.08747  
##  1st Qu.: -1.1580   1st Qu.: -1.1660   1st Qu.:0.33202  
##  Median :  0.2380   Median :  0.2340   Median :1.00268  
##  Mean   :  0.1458   Mean   :  0.1399   Mean   :1.57462  
##  3rd Qu.:  1.4090   3rd Qu.:  1.4050   3rd Qu.:2.05373  
##  Max.   : 12.0260   Max.   : 12.0260   Max.   :9.32821  
##      Today          Direction
##  Min.   :-18.1950   Down:484  
##  1st Qu.: -1.1540   Up  :605  
##  Median :  0.2410             
##  Mean   :  0.1499             
##  3rd Qu.:  1.4050             
##  Max.   : 12.0260
```
It appears that all percentage return predictors (`Lag1-5` and `Today`) have identical minimum and maximum values, also known as *trading curbs*. Those limits were introduced to prevent both speculative gains and dramatic losses within a small time frame.

Additionally, we can calculate all pairwise correlations coefficients among the predictors, with the exception of `Direction` which is qualitative.

``` r
cor(Weekly[, -9])

##               Year         Lag1        Lag2        Lag3         Lag4
## Year    1.00000000 -0.032289274 -0.03339001 -0.03000649 -0.031127923
## Lag1   -0.03228927  1.000000000 -0.07485305  0.05863568 -0.071273876
## Lag2   -0.03339001 -0.074853051  1.00000000 -0.07572091  0.058381535
## Lag3   -0.03000649  0.058635682 -0.07572091  1.00000000 -0.075395865
## Lag4   -0.03112792 -0.071273876  0.05838153 -0.07539587  1.000000000
## Lag5   -0.03051910 -0.008183096 -0.07249948  0.06065717 -0.075675027
## Volume  0.84194162 -0.064951313 -0.08551314 -0.06928771 -0.061074617
## Today  -0.03245989 -0.075031842  0.05916672 -0.07124364 -0.007825873
##                Lag5      Volume        Today
## Year   -0.030519101  0.84194162 -0.032459894
## Lag1   -0.008183096 -0.06495131 -0.075031842
## Lag2   -0.072499482 -0.08551314  0.059166717
## Lag3    0.060657175 -0.06928771 -0.071243639
## Lag4   -0.075675027 -0.06107462 -0.007825873
## Lag5    1.000000000 -0.05851741  0.011012698
## Volume -0.058517414  1.00000000 -0.033077783
## Today   0.011012698 -0.03307778  1.000000000
```

As expected, the percentage return predictors are not corellated with any of the other predictors and the only association appears to be between `Year` and `Volume`. The association is strongly positive and it can be seen graphically on the below plot:

``` r
plot(Weekly$Year, Weekly$Volume, col = "coral2",
    xlab = "Year",
    ylab = "Average number of daily shares traded (billions)",
    type = "p", pch = 20, cex.axis = 0.8, cex.lab = 0.8)
```

<img src="/images/unnamed-chunk-1-1.png" width="100%" style="display: block; margin: auto;" />

We can see below the plots for all combinations of the predictors. The scatterplots between `Today` and `Lag1-5` predictors are very similar and there is no indication of an association between them.

``` r
pairs(Weekly)
```

<img src="/images/unnamed-chunk-2-1.png" width="80%" style="display: block; margin: auto;" />

### Logistic Regression Model

We now fit a logistic regression model with `Direction` as the response and the five lag variables plus `Volume` as predictors, using the `glm()` and `summary` functions as shown below:

``` r
weekly.fit <- glm(Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 + Volume,
                 data = Weekly, family = binomial)
summary(weekly.fit)

##
    ## Call:
    ## glm(formula = Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 +
    ##     Volume, family = binomial, data = Weekly)
    ##
    ## Deviance Residuals:
    ##     Min       1Q   Median       3Q      Max  
    ## -1.6949  -1.2565   0.9913   1.0849   1.4579  
    ##
    ## Coefficients:
    ##             Estimate Std. Error z value Pr(>|z|)   
    ## (Intercept)  0.26686    0.08593   3.106   0.0019 **
    ## Lag1        -0.04127    0.02641  -1.563   0.1181   
    ## Lag2         0.05844    0.02686   2.175   0.0296 *
    ## Lag3        -0.01606    0.02666  -0.602   0.5469   
    ## Lag4        -0.02779    0.02646  -1.050   0.2937   
    ## Lag5        -0.01447    0.02638  -0.549   0.5833   
    ## Volume      -0.02274    0.03690  -0.616   0.5377   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ##
    ## (Dispersion parameter for binomial family taken to be 1)
    ##
    ##     Null deviance: 1496.2  on 1088  degrees of freedom
    ## Residual deviance: 1486.4  on 1082  degrees of freedom
    ## AIC: 1500.4
    ##
    ## Number of Fisher Scoring iterations: 4
```

The last column of the results shows the *P-Values* for each variable. For variables `Lag1`, `Lag3`, `Lag4`, `Lag5` and `Volume`, we can accept the *null hypothesis* that there is no relationship between them and `Direction`.

With regards to variable `Lag2`, the *P-Value* is equal to 0.0296 which is smaller than the *P-Value* associated with a 95 percent confidence (a = 0.05). In theory, based on those results, we should reject the null hypothesis Ho that there is no relation between `Lag2` and `Direction`, however, it must be noted that since *P-Value* is very close to the limit value of 0.05, we may as well accept the hypothesis.

The positive coefficient for predictor `Lag2` suggests that if the market had a positive return two weeks ago, then it is more likely that it will move upwards this week.

### Testing and Confusion Matrix
We can now use `predict()` function and the generated model to predict the probability that the market will go up at the end of the week, given values of the predictors.

Since we used the whole dataset to train our model, the probabilities will be calculated using the training data.

``` r
weekly.prd <- predict(weekly.fit, type = "response")
contrasts(Weekly$Direction)
```

    ##      Up
    ## Down  0
    ## Up    1

We have created vector `weekly.prd`, the elements of which are the calculated probabilities. We know that these values correspond to the probability of the market going up on a given week because the `contrasts()` function indicates that `R` has created a dummy variable, assigning value 1 to `Up`.

In order to compare the predicted values to the actual direction of the market, first we convert the calculated probabilities to "Up" (for values greater than 0.5) and "Down" (for values lower than 0.5).

``` r
weekly.prd.dir <- rep("Up", 1089)
weekly.prd.dir[weekly.prd < 0.5] <- "Down"
```

Using the `table()` function, we can create the *confusion matrix* and determine how many of the predictions were actually correct:

``` r
table(weekly.prd.dir, Weekly$Direction)
```

    ##               
    ## weekly.prd.dir Down  Up
    ##           Down   54  48
    ##           Up    430 557

The results above can be interpreted as follows:

-   Element \[1,1\] = 54: Indicates how many times Predicted Value = "Down" AND Real Value = "Down" (Correct prediction)
-   Element \[1,2\] = 48: Indicates how many times Predicted Value = "Up" AND Real Value = "Down" (Incorrect prediction)
-   Element \[2,1\] = 430: Indicates how many times Predicted Value = "Down" AND Real Value = "Up" (Incorrect prediction)
-   Element \[2,2\] = 557: Indicates how many times Predicted Value = "Up" AND Real Value = "Up" (Correct prediction)

We can calculate the overall fraction of correct predictions by using the `mean()` function. The result shows that out model made a correct prediction 56.1% of the time.

``` r
mean(weekly.prd.dir == Weekly$Direction)

## [1] 0.5610652
```

*This analysis is based on Exercise 4.10 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
