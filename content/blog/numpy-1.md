---
title: "Introduction to NumPy"
date: 2018-02-06T22:42:26Z
tags: ["Python", "Numpy"]
draft: true
---

`NumPy` library provides us with a plethora of array manipulation functions, much more efficient than the standard Python ones.
Let us begin by generating vectors u = -10, -9, -8, ...0 and v = -0.1, 0.4, 0.9...4.9.


```python
import numpy as np

u = np.arange(-10, 1, 1, dtype = float)
v = np.arange(-0.1, (-0.1 + 11*0.5), 0.5, dtype = float)
u,v

```
    (array([-10.,  -9.,  -8.,  -7.,  -6.,  -5.,  -4.,  -3.,  -2.,  -1.,   0.]),
     array([-0.1,  0.4,  0.9,  1.4,  1.9,  2.4,  2.9,  3.4,  3.9,  4.4,  4.9]))



Common mathematical operators can be applied as long as shown below:


```python
print((u + v),'\n', (u*v))
```

    [-10.1  -8.6  -7.1  -5.6  -4.1  -2.6  -1.1   0.4   1.9   3.4   4.9]
     [  1.   -3.6  -7.2  -9.8 -11.4 -12.  -11.6 -10.2  -7.8  -4.4   0. ]


We now increase all elements of vector u by 1,
subtract 20% from v elements and create vector w, containing all the numbers from u and v.


```python
u = u + 1
v = v - abs(0.2*v)
w = np.concatenate([u,v])
```

We can print the elements stored by indexing `w` as shown below. The returned values correspond to the 5th, 9th and 21st elements.


```python
'%.2f'%np.round(w[5],2), '%.2f'%np.round(w[9],2), '%.2f'%np.round(w[21],2)
```

    ('-4.00', '0.00', '3.92')



It is worth noting that the indices start from 0.
Thus, although the length of `w` is 22,
the below indexing returns an error since the last element is w[21].


```python
w[22]
```


    ---------------------------------------------------------------------------

    IndexError                             Traceback (most recent call last)

    <ipython-input-54-da15fbae8f3b> in <module>()
    ----> 1 w[22]


    IndexError: index 22 is out of bounds for axis 0 with size 22


We will now create the matrix below. First, we create an empty table and fill it with zeros and then, using the `for` loop, we generate the values.

|       |     |     |      |    |
|:----------:|:---------:|:--------:|:---------:|:------: |
| 1     | 3      | 5   |  7    |9        |
| 11    | 13      | 15   |  17    |19       |
| 21    | 23      | 25   |  27    |29       |
| 31    | 33      | 35   |  37    |39       |



```python
B = np.zeros((4,5))
x = 1

for i in range (0, 4):
    B[i,:] = np.linspace(x, x + 8, 5)
    x = x + 10  
B
```




    array([[  1.,   3.,   5.,   7.,   9.],
           [ 11.,  13.,  15.,  17.,  19.],
           [ 21.,  23.,  25.,  27.,  29.],
           [ 31.,  33.,  35.,  37.,  39.]])



### Basic Statistics

Suppose we have the following observations of variable x.

10, 2, 15, 6, 4, 9, 12, 11, 3, 0, 12, 10, 9, 7, 11, 10, 8, 5, 10, 6

We create vector `x` using the `Numpy` array function.



```python
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline  

x = np.array([10, 2, 15, 6, 4, 9, 12, 11, 3, 0, 12, 10, 9,
              7, 11, 10, 8, 5, 10, 6])
```

The standard `len()` Python function returns the length of the array whereas Numpy function `sum()` calculates the sum of all array elements.


```python
len(x)
```
    20

```python
np.sum(x)
```
    160

Numpy library includes functions for common measures of centrality and other statistical parameters. The standard deviation can be calculated as the square root of variance.


```python
#Mean, Median, Variance, Standard deviation
np.mean(x), np.median(x), '%.2f'% np.var(x), '%.2f'% np.sqrt(np.var(x))

```
    (8.0, 9.0, '13.80', '3.71')

We now set a seed for reproducibility and then create a vector y of random variables, with the same length and mean as x and standard deviation 1.

```python
np.random.seed(42)
y = np.random.normal(np.mean(x), 1, len(x))
y
```

    array([ 8.49671415,  7.8617357 ,  8.64768854,  9.52302986,  7.76584663,
            7.76586304,  9.57921282,  8.76743473,  7.53052561,  8.54256004,
            7.53658231,  7.53427025,  8.24196227,  6.08671976,  6.27508217,
            7.43771247,  6.98716888,  8.31424733,  7.09197592,  6.5876963 ])

In order to use to `corrcoef()` and `cov()` functions to calculate the correlation and
covariance matrices between x and y,
we vertically stack the vectors and then call the respective methods.  


```python
print ("Correlation:\n", np.corrcoef(np.vstack([x,y])),"\n")
print ("Covariance\n", np.cov(np.vstack([x,y])))
```

    Correlation:
     [[ 1.          0.05431314]
     [ 0.05431314  1.        ]]

    Covariance
     [[ 14.52631579   0.19873151]
     [  0.19873151   0.92165457]]


We now create vectors z and u and fill them with random normal variables. Both vectors are of length 20, mean 8 and standard deviation 1 and 10 respectively.

Finally we plot x,z and u vectors on the same scatter plot.


```python
z = np.random.normal(8,1,20)
u = np.random.normal(8,10,20)

plt.scatter(x,z)
plt.scatter(x,u)
plt.xlabel("x")
plt.ylabel("z/u")
```

<center><img src="/images/output_26_1.png" style="width: 70%; height: 70%" display: block; margin: auto;" /></center>
