---
title: "A Neural Network Approach to Wisconsin Diagnosis Breast Cancer (WDBC)"
date: 2018-01-22T21:48:18Z
draft: true
tags: ["R", "Statistical Learning"]
---
(*The full code of the application can by found on <a href="https://github.com/christospitsi/wdbc-ann">github</a>*)

## 1. Introduction
The recent advances in machine learning methods and computational power, in conjunction with the development of highly sophisticated tools and frameworks, have provided us with a powerful toolbox with which we can predict with high accuracy and train algorithms effectively and quickly.

Medical practice is one of the fields in which the aforementioned methods are widely used for diagnostic purposes. A practitioner can use a machine learning tool in order to conclude whether an abnormal condition such as breast cancer is present and judge on the seriousness of the condition.

Breast cancer may be detected via a careful study of clinical history, physical examination, and imaging with either mammography or ultrasound. However, deﬁnitive diagnosis of a breast mass can only be established through ﬁne-needle aspiration (FNA) biopsy, core needle biopsy, or excisional biopsy. Among these methods, FNA is the easiest and fastest method of obtaining a breast biopsy. [5]

In this project, it is examined the usage of Artificial Neural Networks on the Wisconsin Diagnosis Breast Cancer (WDBC) data set, in order to classify a mass as benign or malignant, based on 30 features extracted from FNA biopsies.

The neural network approach was followed since it has been shown that it performs quite well on problems of image recognition and they have been widely used on applications of computer vision.

## 2. A Neural Network Approach to Wisconsin Diagnosis Breast Cancer (WDBC)
An artificial neural network consists of a number of interconnected processors, called neurons (nodes). The nodes are connected by weighted links passing signals from one node to another [6], dependent on the activation function.

Specifically, for this application, a Multilayer Perceptron (MLP) model has been built, using as an input the features of the data set, and as an output the classification i.e. whether a mass is malignant or benign.

The application was developed using ```R``` interface to ```Keras``` framework [2][3] and ```TensorFlow``` [1] backend engine in order to create and train a sequential model which uses the backpropagation algorithm. ```Keras``` is a high-level neural networks API, written in Python and capable of running on top of TensorFlow software library.

### 2.1 Data and preprocessing
The data used for model training and testing consist of 30 features extracted from the image captured during FNA biopsies for each nucleus. The features used are:

* 	radius (mean of distances from centre to points on the perimeter)
* 	texture (standard deviation of gray-scale values)
* 	perimeter
* 	area
* 	smoothness (local variation in radius lengths)
* 	compactness (perimeter^2 / area - 1.0)
* 	concavity (severity of concave portions of the contour)
* 	concave points (number of concave portions of the contour)
* 	symmetry
* 	fractal dimension (“coastline approximation” - 1)

The mean, standard error, and “worst” or largest (mean of the three largest values) of these features were computed for each image, resulting in 30 features. [7]

The data set was split into training and test sets (roughly 80% and 20% respectively) and converted to arrays in order to be used by ```Keras```.

A close look to the original data set revealed that the features have different order of magnitude. In order to avoid a situation where features with higher values have greater contribution to the outcome of the model, each feature was normalized using R build-in scale() function.  scale() function transforms each value by subtracting the mean value of the feature and dividing it by the standard deviation.

With regard to the output of the model, the M and B categories corresponding to Malignant” and Benign” masses, where converted to 1 and 0 respectively.

### 2.2 Model structure
The initial model built was a Multilayer Perceptron Model with one input layer with 30 nodes (one per feature), one hidden layer with 32 nodes (layer1) and the output layer with 1 node. The value of the output node represents the prediction of the value, as described in section 2.1.

Although the initial model had 32 nodes on its hidden layer, after multiple experiments with different layer structures it became apparent that the model with a 16-node hidden layer perform better than the one with 32 nodes. (see section 3.2 for details)

### 2.3 Model parameters
*	Rectified Linear Unit (ReLU) was selected as the **activation function** for the hidden layer since it resulted to slightly higher accuracy compared to Sigmoid. With regard to the output layer, a sigmoid activation function was used. Although the softmax function is probably the most common choice for classification problems, this application deals with a binary classification, thus, softmax (and ReLU) activation functions were proved unsuitable for the output layer.

*	The **optimization method** set in ```Keras``` compiler was Adam, an algorithm for first-order gradient-based optimization of stochastic objective functions, based on adaptive estimates of lower-order moments [4]. The learning rate of the optimizer, after a few experimentations with different values, was set to 0.0003.

* Finally, it is worth mentioning the initializers used on all models. The **biases** for all models were arbitrarily initialized to ‘1’ since it was observed that when the biases were initialized to ‘zeros’, the model performed slightly worse. Additionally, the **weights** were arbitrarily initialized to have mean = 0 and standard deviation = 0.1 in all models examined in section 3.

### 2.4 Cross validation
A 20% of the training set was used as validation data. The model used this fraction of the training set to evaluate the loss and accuracy on this data at the end of each epoch. As a consequence, the validation set was not used for training purposes.

## 3. Experiments
The model described in section 2 was trained for 20 epochs. The below plot shows how accuracy and loss values change, for both training and validation sets.
As we can see, the accuracy of the model increases significantly during the first 5 epochs, after which stabilizes. After epoch 5, the loss values continue to drop, and the accuracy keeps increasing, however at a lower rate.

<center><img align = "middle" src="/images/wdbc_1.png" style="width: 70%; height: 70%;"/></center>

### 3.1 Activation functions
It was tested how a different activation function changes the learning ability/speed and accuracy of the neural network, keeping all other parameters constant. The hidden layer of the neural network for this experiment consisted of 32 nodes.

The ReLU activation function has output 0 if the input is negative. In all other cases, the output equals to the input.
The mathematical equations below show the expressions for both activation functions.

On the below plot we can see how the loss decreases as the training progresses. It is depicted that the curves have similar shape, however the loss corresponding to ReLU function seems to decrease slightly faster but most importantly, it settles at a lower value that the Sigmoid curve.

<center><img align = "middle" src="/images/wdbc_2.png" style="width: 70%; height: 70%;"/></center>

Following the loss for each function, it was examined how the training accuracy of the model changes as the training progresses. On the below plot we can see the values for both activation functions.

The sigmoid curve follows the familiar “S” shape and, as mentioned on section 2.3, results on slightly lower accuracy. The ReLU curve has the expected shape based on its function and at the end of the training phase has achieved slightly higher accuracy.

<center><img align = "middle" src="/images/wdbc_3.png" style="width: 70%; height: 70%;"/></center>

The results on the table below show the testing accuracy for both models. We can observe that the majority of the errors for both models are false negatives i.e. observations labeled as “Benign” while their real value is “Malignant”, however, as determined during the training process, the model using the ReLU activation function has higher overall accuracy.
The table below table shows the type of classification errors in addition to the overall accuracy. It appears that the model using ReLU activation function generates only Type II (false negative) errors whereas the model using sigmoid generates both Type I (false positive) and Type II (false negative) errors.

|Activation Function   | Benign(%)     | Malignant(%)  | Overall  |
|:--------------------:|:-------------:|:-------------:| :-------:|
|        Sigmoid       | 98            | 84            |   92     |
|         ReLU         | 100           | 84            |   93     |


### 3.2 Architecture
In addition to the changes due to different activation function, it was examined how neural networks learn and perform when the number of the nodes on the hidden layer is changed and all other parameters kept constant.

For that purpose, models with 4, 8, 16 and 64 nodes on the hidden layer were created and compared to the one already examined in the previous chapter with 32 nodes on the hidden layer. The same training weights and biases were used for all models and each training phase lasted for 20 epochs.

<center><img align = "middle" src="/images/wdbc_4.png" style="width: 70%; height: 70%;"/></center>

The bar plot above describes the findings regarding the testing accuracy for the above models. It appears that the model with the worst accuracy is the one with 8 nodes on its hidden layer whereas the models with 16 and 64 nodes have the highest accuracy.

In addition to the testing accuracy, the below bar plot shows the training accuracy of the above models. The model with the 64 nodes appears to have very high training accuracy (98%). It is worth pointing out that the model with 16 nodes performs worst on the training data compared to the test data whereas the opposite behaviour is observed for the 32-node model. Finally, we can see that the 8-node model has been trained rather poorly since its training accuracy is below 90%.

<center><img align = "middle" src="/images/wdbc_5.png" style="width: 70%; height: 70%;"/></center>

In order to have a better perception of the accuracy of each model and the type of wrong classifications, the test classification accuracy for each class was calculated in addition to the overall accuracy presented on the “Testing Accuracy” graph. The following table (Table 2) sums up the results for all different architectures during the test phase.

|**Architecture (Nodes)** | **Benign(%)** | **Malignant(%)**| **Overall**  |
|:-----------------------:|:-------------:|:---------------:| :-----------:|
|         4               | 100           | 84              |   93         |
|         8               | 86            | 100             |   92         |
|         16              | 92            | 100             |   96         |
|         32              | 100           | 84              |   93         |
|         64              | 100           | 89              |   96         |


The data presented show that each architecture produces only one kind of error; either Type I (false positive) or Type II (false negative). More specifically, 4, 32, and 64 node architectures do only false negative predictions, whereas all wrong predictions of 8 and 16 node architectures are false positives.

Although both 16 and 64-node models have the same overall accuracy, given that the 16-node model has a 100% accuracy rate on classifying the “Malignant” tumours, we can state that this is the best model amongst the ones examined in this project.

## 4. Conclusions

### 4.1 Summary
In this project it was presented how a neural network can be used in order to classify masses as “Benign” or “Malignant”, based on the Breast Cancer Wisconsin data set. It was examined how different activation functions, learning rates and network architectures have an effect on the network accuracy and speed.

Based on the testing accuracy of the best performing models (~96% for the models with 16 and 64-node hidden layers), it can be said that the neural network performed fairly well. The training time for each model using a standard home computer was below 10 seconds, a fact which reinforces the point made in the introductory section that the computational power available to everyone today can be used to built very efficient and accurate models.

### 4.2 Future work
Although the models of this application produced fairly good predictions, there are different approaches that can be examined in the future which can produce even better results. As stated in the description of the dataset [7], the best predictive accuracy obtained using one separating plane had an estimated accuracy 97.5%. The neural network approach described in this project can be modified to a model which may produce accuracy of that level.

As described in section 2, the neural network developed consists of one hidden layer, in addition to the input and output ones. In the future, different layer architectures can be examined, with more than one hidden layer and different layer configurations in terms of nodes per layer. Although the data set is fairly small and probably performs better using simpler architectures, a more complex one may produce better results.

In addition to the architecture, the network parameters can be modified as well. The application presented in this project used fixed and arbitrarily chosen biases and weights, as described in section 2.3. In the future, a more comprehensive list of values of biases, weights and optimizer parameters can be examined which can have as a result a more accurate model.

## 5. Bibliography
[1] Martín Abadiet et al. TensorFlow: Large-scale machine learning on heterogeneous systems, 2015. Software available from tensorflow.org.

[2] Chollet François, Allaire JJ, available online at https://Keras.rstudio.com/ and on Github https://github.com/rstudio/Keras

[3] Chollet François and others, Keras, 2015, GitHub, available online at https://github.com/Keras-team/Keras

[4] Diederik P. Kingma, Jimmy Ba, Adam: A Method for Stochastic Optimization, 2005, available online at https://arxiv.org/abs/1412.6980

[5] Tingting Mu, Asoke K. Nandi. Breast cancer diagnosis from fine-needle aspiration using supervised compact hyperspheres and establishment of confidence of malignancy. Department of Electrical Engineering and Electronics, The University of Liverpool, 2008

[6] Michael Negnevitsky. Artificial Intelligence, A Guide to Intelligent Systems, 2nd edn. Pearson Education Limited, Second Edition, 2005

[7] UCI Machine Learning Repository, available online at https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+(Diagnostic), last accessed 14/01/2018
