---
title: "Predicting vehicle gas consumption"
date: 2017-09-22T21:48:18Z
draft: true
tags: ["R", "Statistical Learning"]
---

The *Auto* data set, included in the ISLR library provides us with details for 392 vehicles and the following variables for each observation:

-   *Mpg*: Miles per gallon
-   *cylinders*: Number of cylinders (4-8)
-   *displacement*: Engine displacement (cu. inches)
-   *horsepower*: Engine power
-   *weight*: Vehicle weight (lbs.)
-   *acceleration*: Time to accelerate from 0 to 60 mph (sec.)
-   *year*: Model Year (modulo 100)
-   *origin*: Origin of the car (1. American, 2. European, 3. Japanese)
-   *name*: Vehicle name

Based on the *mpg* variable, we create variable *mpg01* which is equal to 1 if *mpg* is above the median *mpg* value and equal 0 if *mpg* value is below the median.

First we create a vector with size equal to the number of observations and fill it with 0s. Following that, we change 0s to 1s based on the above condition. Finally, we create a new data frame *df* which consists of the original data set plus the new variable *mpg01*.

``` r
library(ISLR)
mpg01 <- rep("0", nrow(Auto))
mpg01[Auto$mpg > (median(Auto$mpg))] <- "1"
df <- data.frame(Auto, mpg01)
```

Based on the aforementioned, variable *mpg01* equals to 1 when *mpg* is above the median value which means that the car has relatively low gas consumption since it does more miles per gallon. We expect a car to have low consumption when it is not heavy and it is not a "sports" car i.e. it doesn't have high *horsepower* neither many *cylinders*. As we can see below, the data confirm our predictions about those variables.

Firstly, we investigate whether there is association between *horsepower* and low gas consumption. On the below plot we can see all values of variable *horsepower* conditionally coloured based on consumption. Observations with mpg = 1 (low consumption) are in blue colour whereas the ones with mpg = 0 (high consumption) are in red. It is apparent that all vehicles with low consumption are the ones with low horsepower and the two variables are indeed associated.

``` r
plot(df$horsepower, col = ifelse(df$mpg01 == 1, "blue", "red"),
     type = "p", pch = 20, cex.axis = 1, cex.lab = 1,
     main = "Horsepower",
     ylab = "Horsepower")

legend(250, 225, c("Low Consumption", "High Consumption"),
       pch = c(20, 20), col = c("Blue", "Red"))
```

<img src="/images/horsepower_mpg-1.png" width="80%" style="display: block; margin: auto;" />

With regard to the cylinder predictor, as we can see on the below boxplot, all vehicles with 4 cylinders (with the exception of four outliers) have low gas consumption whereas the vehicles with 6-8 cylinders have all high consumption and as a consequence low miles per gallon. The result confirms our expectations that fast cars (with many cylinders) have high consumption.

``` r
boxplot(cylinders~mpg01, data = df,
        xlab = "mpg01", ylab = "Cylinders", col = "Red")
```

<img src="/images/cyl_box-1.png" width="80%" style="display: block; margin: auto;" />

Finally, the below plot confirms that heavier vehicles have higher consumption:

``` r
plot(df$weight, col = ifelse(df$mpg01 == 1, "blue", "red"),
     type = "p", pch = 20, cex.axis = 1, cex.lab = 1,
     main = "Weight",
     ylab = "Vehicle Weight")

legend(255, 5280, c("Low Consumption", "High Consumption"),
       pch = c(20, 20), col = c("Blue", "Red"))
```

<img src="/images/weight_mpg-1.png" width="80%" style="display: block; margin: auto;" />

### Logistic regression model

We now split the data into training and test set in order to predict the consumption of a car, using the above features as predictors.
The logistic regression model is performed on the training set and then we calculate the test error.

``` r
set.seed(1)
train <- sample(nrow(df), nrow(df) / 2) #Creates training set

mpg01.fits <- glm(mpg01 ~ horsepower + cylinders + weight,
                    family = binomial, data = df, subset = train)

#Predicts probabilities
glm.probs <- predict(mpg01.fits, df[-train, ], type = "response")

#Converts probabilities to boolean
glm.probs.b <- ifelse(glm.probs > 0.5, "1", "0")

df.test <- df[-train, ] #Test set

table(glm.probs.b, df.test$mpg01) #Prints confusion matrix
```

    ##            
    ## glm.probs.b  0  1
    ##           0 78  6
    ##           1 15 97

``` r
#Test error calculation
test.error <- (1 - mean(glm.probs.b == df.test$mpg01))*100  
sprintf("Test error: %g%%", test.error)
```

    ## [1] "Test error: 10.7143%"

*This analysis is based on Exercise 4.11 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
