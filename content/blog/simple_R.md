---
title: "A few 'Hello World' R commands and Boston housing prices"
date: 2017-08-27T21:06:10Z
tags: ["R", "Statistical Learning"]
draft: true
---
The ```Boston``` data set, part of the ```MASS``` library,
contains information regarding the housing values
in the suburbs of Boston.

The dimensions of the data frame are shown below:

``` r
library (MASS)

dim(Boston)
## [1] 506  14
```
The 506 rows of the dataset correspond to 506 observations,
each of which has 14 predictors (number of columns).

Using ```pairs()``` function,
we generate scatterplots for all combinations between predictors
in order to try to figure out any possible associations.

<img src = "/images/unnamed-chunk-3-1.png" width="90%" style="display: block; margin: auto;" />

The below scatterplot indicates a possible association between the median value of owner-occupied homes and the percentage of the lower socioeconomic status of the population predictors. The association is to be expected since a lower class means lower income, thus the members of that class cannot afford homes in expensive areas.

``` r
plot(Boston$medv, Boston$lstat, col = "deepskyblue2",
    xlab = "Median value of owner-occupied homes ($10.000)",
    ylab = "% lower status of the population",
    type = "p", pch = 20, cex.axis = 0.8, cex.lab = 0.8)
```
<img src = "/images/unnamed-chunk-4-1.png" width="90%" style = "display: block; margin: auto;" />

The correlation between the two parameters is -0.73, indication of a relatively strong negative linear relationship between the two predictors.

``` r
cor(Boston$lstat, Boston$medv)
## [1] -0.7376627
```

Another negative association between predictors can be seen below between the average number of room per dwelling and the percentage of lower status of the population. The data confirm that more rooms means bigger dwelling size and as a consequence a more expensive one. Members of the population in the category of "Lower Status" tend to live in smaller, more affordable homes.

<img src = "/images/unnamed-chunk-6-1.png" width="90%" style = "display: block; margin: auto;" />

The correlation between those two predictors is -0.61

``` r
cor(Boston$rm,Boston$lstat)
## [1] -0.6138083
```

### Per capita crime

In order to examine which predictors are associated with the per capita crime, we calculate the correlation between the per capita crime and all the other predictors.

As can be seen below, first we pass the dataset into a new matrix `Mtrx_Boston` in order to manipulate the data. Then, by using an iteration, we calculate the correlation between the first column (per capita crime) and all other columns. The calculated values are stored in a new matrix `x_cor`.

``` r
Mtrx_Boston <- Boston
x_cor <- matrix(nrow = 1, ncol = 13)

for (n in 2:ncol(Mtrx_Boston)) {
   x_cor[n-1] = c(cor(Mtrx_Boston[, 1],Mtrx_Boston[, n]))
}
x_cor
##            [,1]      [,2]        [,3]      [,4]       [,5]      [,6]
## [1,] -0.2004692 0.4065834 -0.05589158 0.4209717 -0.2192467 0.3527343
##            [,7]      [,8]      [,9]     [,10]      [,11]     [,12]
## [1,] -0.3796701 0.6255051 0.5827643 0.2899456 -0.3850639 0.4556215
##           [,13]
## [1,] -0.3883046
```

As it can be seen from the results, the highest correlation with the per capita crime is stored in elements [8], [9] and [12] of the above vector, values which correspond to predictors `rad` (index of accessibility to radial highways), `tax` (full-value property-tax rate per $10.000) and `lstat` (lower status of the population (%)) respectively.

On Plot 1 we can see that, although the rad predictor has numeric values, it is more accurate to treat it as an ordinal variable thus we cannot extract any safe conclusions based on its correlation with per capita crime.

<img src="/images/unnamed-chunk-9-1.png" width="90%" style="display: block; margin: auto;" />

On the second plot (Plot 2) we see that a significant amount of suburbs have a specific, high property-tax rate (namely 666) hence these points have a major effect on the correlation calculation.

<img src="/images/unnamed-chunk-10-1.png" width="100%" style="display: block; margin: auto;" />

Finally, Plot 3 shows that there is a relationship between the percentage of the lower status of the population and per capita crime.

<img src="/images/unnamed-chunk-11-1.png" width="100%" style="display: block; margin: auto;" />

### Extreme values of Crime Rates/Tax Rates/Teacher-Pupil Ratio

In the below boxplot, we observe that although the vast majority of the suburbs have a crime per capita rate close to zero, there are a few extreme values (outliers).

We can pass the boxplot to a new variable and then have a closer look to the minimum, maximum and the outliers of the boxplot using the `stat` function.

``` r
crim_box <- boxplot(Boston$crim, col = "deepskyblue2", main = "Boston Crime",
                   ylab = "per capita crime", type = "p", pch = 20,
                   cex.axis = 0.8, cex.lab = 0.8)
```

<img src="/images/unnamed-chunk-12-1.png" width="100%" style="display: block; margin: auto;" />

``` r
crim_box$stats
##         [,1]
## [1,] 0.00632
## [2,] 0.08199
## [3,] 0.25651
## [4,] 3.67822
## [5,] 8.98296
```

The results show that the top whisker is 8.98296 thus the outliers are the suburbs with crime rate higher than this value.

``` r
nrow(Boston[Boston$crim > 8.98296,])
## [1] 66
```

The whole range of crime rate predictor as calculated below is [0.00632, 8.9762] however, by eliminating the outliers, the resulted range is the one between the top and bottom whiskers i.e. [0.00632, 8.98296]

``` r
range(Boston$crim)
## [1]  0.00632 88.97620
```

With regard to the tax rates, we can see on the boxplot below that there are no outliers and the predictor ranges between the top and bottom whiskers i.e. [187, 711].

<img src="/images/unnamed-chunk-15-1.png" width="100%" style="display: block; margin: auto;" />

``` r
    ##      [,1]
    ## [1,]  187
    ## [2,]  279
    ## [3,]  330
    ## [4,]  666
    ## [5,]  711
    ## [1] 187 711
```

With regard to the pupils per teachers ratio, there are two outliers as shown on the below plot and the predictor ranges between [12.6, 22.0]

<img src="/images/unnamed-chunk-16-1.png" width="100%" style="display: block; margin: auto;" />
``` r
    ##       [,1]
    ## [1,] 13.60
    ## [2,] 17.40
    ## [3,] 19.05
    ## [4,] 20.20
    ## [5,] 22.00
    ## [1] 12.6 22.0
```

### Suburbs by Charles river

As calculated below, there are 35 suburbs by Charles river.

``` r
nrow(Boston[Boston$chas == 1,])
## [1] 35
```

### Median pupil-teacher ratio

The median pupil-teacher ratio is 19.05:

``` r
median(Boston$ptratio)
## [1] 19.05
```

### Suburb with lowest median value of owner occupied homes

Using `Which.min` function which returns the index of the observation with the lowest `medv`, we get all predictors for that particular observation.

``` r
Boston[which.min(Boston$medv),]
##     crim zn indus chas   nox    rm age    dis rad tax ptratio black
##  38.3518  0  18.1    0 0.693 5.453 100 1.4896  24 666    20.2 396.9
##  lstat medv
##  30.59    5
```

Using the `summary()` function, we can get the range for all predictors and compare them to the values of the above suburb.

``` r
summary(Boston)
##       crim                zn             indus            chas        
##  Min.   : 0.00632   Min.   :  0.00   Min.   : 0.46   Min.   :0.00  
##  1st Qu.: 0.08204   1st Qu.:  0.00   1st Qu.: 5.19   1st Qu.:0.00  
##  Median : 0.25651   Median :  0.00   Median : 9.69   Median :0.00  
##  Mean   : 3.61352   Mean   : 11.36   Mean   :11.14   Mean   :0.06  
##  3rd Qu.: 3.67708   3rd Qu.: 12.50   3rd Qu.:18.10   3rd Qu.:0.00  
##  Max.   :88.97620   Max.   :100.00   Max.   :27.74   Max.   :1.00  
##       nox               rm             age              dis        
##  Min.   :0.3850   Min.   :3.561   Min.   :  2.90   Min.   : 1.130  
##  1st Qu.:0.4490   1st Qu.:5.886   1st Qu.: 45.02   1st Qu.: 2.100  
##  Median :0.5380   Median :6.208   Median : 77.50   Median : 3.207  
##  Mean   :0.5547   Mean   :6.285   Mean   : 68.57   Mean   : 3.795  
##  3rd Qu.:0.6240   3rd Qu.:6.623   3rd Qu.: 94.08   3rd Qu.: 5.188  
##  Max.   :0.8710   Max.   :8.780   Max.   :100.00   Max.   :12.127  
##       rad              tax           ptratio          black       
##  Min.   : 1.000   Min.   :187.0   Min.   :12.60   Min.   :  0.32  
##  1st Qu.: 4.000   1st Qu.:279.0   1st Qu.:17.40   1st Qu.:375.38  
##  Median : 5.000   Median :330.0   Median :19.05   Median :391.44  
##  Mean   : 9.549   Mean   :408.2   Mean   :18.46   Mean   :356.67  
##  3rd Qu.:24.000   3rd Qu.:666.0   3rd Qu.:20.20   3rd Qu.:396.23  
##  Max.   :24.000   Max.   :711.0   Max.   :22.00   Max.   :396.90  
##      lstat            medv      
##  Min.   : 1.73   Min.   : 5.00  
##  1st Qu.: 6.95   1st Qu.:17.02  
##  Median :11.36   Median :21.20  
##  Mean   :12.65   Mean   :22.53  
##  3rd Qu.:16.95   3rd Qu.:25.00  
##  Max.   :37.97   Max.   :50.00
```

As we can see, the suburb with the lowest median value of owner occupied homes compared to the rest of the observations has:

-   Remarkably high crime rate
-   The minimum observed proportion of residential land zoned for lots over 25,000 sq.ft.
-   High proportion of non-retail business acres per town (equal to 3rd quartile)
-   High nitrogen oxides concentration (above 3rd quartile)
-   Low number of rooms (below 1st quartile)
-   The maximum proportion of owner-occupied units built prior to 1940 (100%)
-   Very low weighted mean of distances to five Boston employment centres (below 1st quartile)
-   Maximum index of accessibility to radial highways
-   Very high tax rate (equal to 3rd quartile)
-   Very high pupil-teacher ratio (equal to 3rd quartile)
-   The maximum proportion of blacks by town
-   Remarkably high percentage of lower status of population (close to maximum observed value)

### Rooms per dwelling

There are 64 suburbs which average more than seven rooms per dwelling whereas there are 13 which average more than 8:

``` r
nrow(Boston[Boston$rm > 7, ])
## [1] 64
nrow(Boston[Boston$rm > 8, ])
## [1] 13
```

As we can see the suburbs which average more than eight rooms per dwelling have very low crime rate, very high median value of owner occupied homes and low percentage of lower class population.

``` r
Boston[Boston$rm > 8, ]
##     crim zn indus chas    nox    rm  age    dis rad tax ptratio  black
##  0.12083  0  2.89    0 0.4450 8.069 76.0 3.4952   2 276    18.0 396.90
##  1.51902  0 19.58    1 0.6050 8.375 93.9 2.1620   5 403    14.7 388.45
##  0.02009 95  2.68    0 0.4161 8.034 31.9 5.1180   4 224    14.7 390.55
##  0.31533  0  6.20    0 0.5040 8.266 78.3 2.8944   8 307    17.4 385.05
##  0.52693  0  6.20    0 0.5040 8.725 83.0 2.8944   8 307    17.4 382.00
##  0.38214  0  6.20    0 0.5040 8.040 86.5 3.2157   8 307    17.4 387.38
##  0.57529  0  6.20    0 0.5070 8.337 73.3 3.8384   8 307    17.4 385.91
##  0.33147  0  6.20    0 0.5070 8.247 70.4 3.6519   8 307    17.4 378.95
##  0.36894 22  5.86    0 0.4310 8.259  8.4 8.9067   7 330    19.1 396.90
##  0.61154 20  3.97    0 0.6470 8.704 86.9 1.8010   5 264    13.0 389.70
##  0.52014 20  3.97    0 0.6470 8.398 91.5 2.2885   5 264    13.0 386.86
##  0.57834 20  3.97    0 0.5750 8.297 67.0 2.4216   5 264    13.0 384.54
##  3.47428  0 18.10    1 0.7180 8.780 82.9 1.9047  24 666    20.2 354.55
##  lstat medv
##   4.21 38.7
##   3.32 50.0
##   2.88 50.0
##   4.14 44.8
##   4.63 50.0
##   3.13 37.6
##   2.47 41.7
##   3.95 48.3
##   3.54 42.8
##   5.12 50.0
##   5.91 48.8
##   7.44 50.0
##   5.29 21.9
```

*This analysis is based on Exercise 2.10 from "An Introduction to Statistical Learning" book by Robert Tibshirani and Trevor Hastie*
