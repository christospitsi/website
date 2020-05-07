---
title: "Statistics 101"
date: 2017-08-21T19:52:49Z
tags: ["R", "Statistical Learning"]
draft: true
---
### Mean, Median and Mode

A few introductory words about
the very basic definitions and ```R``` commands
of measures of central tendency and spread in statistics.

Suppose we ask a group of 10 students
how many brothers and sisters they have and
the number obtained are as follows:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
(2&nbsp;&nbsp;3&nbsp;&nbsp;0&nbsp;
  &nbsp;5&nbsp;&nbsp;2&nbsp;&nbsp;1
  &nbsp;&nbsp;1&nbsp;&nbsp;0&nbsp;&nbsp;3
  &nbsp;&nbsp;3)

The *mean* of our data is the arithmetic average of the observations whereas the *median* is the middle value when the observations are ranked in order.

Those values can be calculated simply by creating a vector with the observations and then calling the respective build-in ```R``` commands as shown below:

```r
x <- c(2, 3, 0, 5, 2, 1, 1, 0, 3, 3)
mean(x)
## [1] 2
median(x)
## [1] 2
```  
The *mode* of our data is the most frequently occurring observation. The most frequent observation in our example is 3 which appears three times.

Since there is no built-in R command for mode calculation, we have to create a simple function.

For this purpose, we will use ```table()``` function which returns a table with the frequency of each observation and `sort()` which sorts the frequencies from high to low. `names()[1]` will then return the first element of the sorted vector, which is our most frequent value.

``` r
mode <- function(x) {
  names(sort(table(x), decreasing = TRUE))[1]
}

mode(x)
## [1] "3"
```

### Variance and Standard deviation

Now, let's assume that the age of the students asked
are as follows:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
(23&nbsp;&nbsp;25&nbsp;&nbsp;18&nbsp;
  &nbsp;45&nbsp;&nbsp;30&nbsp;&nbsp;21
  &nbsp;&nbsp;22&nbsp;&nbsp;19&nbsp;&nbsp;29
  &nbsp;&nbsp;35)

*Variance* and *standard deviation* are measures of spread and describe
how much the observations vary from the mean value.
We can calculate those measures of spread,
between the numbers of siblings and theirs age
using the ```R``` build-in functions ```var()``` and ```sd()```.

``` r
var(x)
## [1] 2.444444
sd(x)
## [1] 1.563472
```
```R``` calculates the *unbiased* values of variance and standard deviation
where the *Bessel's correction* has been applied.

### Covariance and correlation

The *covariance* measures how much the movement in one variable predicts
the movement in a corresponding variable.
In our example,
variable ```x``` is the number of siblings
and ```y``` the age of the students.

In order to calculate those values,
we create the vectors for ```x``` and ```y``` variables
and then call the respective functions

``` r
y <- c(23, 25, 18, 45, 30, 21, 22, 19, 29, 35)

message('The covariance coefficient is: ',cov(x,y))
## The covariance coefficient is: 11.8888888888889

message('The correlation coefficient is: ',cor(x,y))
## The correlation coefficient is: 0.91169712523063
```

Since the calculated correlation coefficient (0.91) is positive and near 1,
we can safely state that there is a very strong positive relationship between ```x``` and ```y```.

Although there is a strong relationship between the number of siblings and the age of a student,
we cannot conclude that there is causation between the two variables since the probability of having a certain amount of siblings is independent from age.
