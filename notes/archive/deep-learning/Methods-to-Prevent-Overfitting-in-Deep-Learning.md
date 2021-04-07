---
title: "Methods to Prevent Overfitting in Deep Learning"
date: 2019-03-20T09:55:04+11:00
draft: false
categories: ["Deep Learning"]
---
# Methods to Prevent Overfitting in Deep Learning

## Overfitting

Overfitting refers to that when a model fits the training data well but cannot predict the test data correctly, we may say that the model lacks the ability of generalization. It is important to figure out how it happens, and how we can prevent overfitting from the very beginning. 

## Detect Overfitting


The simplest way to detect overfitting is to split the dataset into two parts: the training set for training the model, and the test set for testing the accuracy of the model on a dataset that it has never seen before. Of course, we will also partition part of the training set to be the validation set for fine-tuning hyper-parameters. Note that it is necessary to shuffle all the data before splitting.

Through this split, we check the performance of the model to gain insight on how the training process goes, and detect overfitting.

During training, we may see the process:

<img src="https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Deep-Learning/0.png">

The accuracy on the training set has reached a very high ratio, but the accuracy on the validation set still remains not high enough, or even a little bit low.

If we plot the accuracy and loss of the training set and the validation set into one figure, we will see:

<img src="https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Deep-Learning/1.png">

It is obvious that the accuracy of the validation data is much  lower than that of the training data.

Although overfitting is frustrating, there are plenty of methods to prevent it from happening.

## Prevent Overfitting

Here are some practical methods to prevent overfitting during training deep neural networks:

### 1. Regularization

Regularization is the most-used method to prevent overfitting in Machine Learning. It constrains the learning of the model by adding a regularization term. Typical regularization is to explicitly add regularization terms in the objective function, e.g. L1 and L2 regularization terms. 

In deep learning, there are two commonly-used regularization methods: **Batch Normalization** and **Dropout**.

#### Batch Normalization

<!-- Batch Gradient Descent -->

<!-- In deep learning, since using the whole training set to do train the model, i.e. Batch Gradient Descent, requires a large amount of memory and a very long time, we divide the dataset to mini-batches to train the model. Batch Normalization is calculated based on mini-batch.

Mini-Batch Gradient Descent is a method to group scattered data. The term "mini-batch" refers to a small group of data, that is the number of samples per round of optimization. It divides the data into mini-batches and updates the parameters mini-batch by mini-batch. The data in one mini-batch together determine the direction of the gradient descent, which reduces the randomness when updating parameters using Stochastic Gradient Descent. In the meanwhile, the number of samples in a mini-batch is much smaller than that in an entire dataset, so the amount of calculation has dropped a lot compared with using Batch Gradient Descent. -->

As the training progresses, the parameters in the deep neural network are constantly updated. On the one hand, parameters are changing slightly during the training. Due to the activation functions in each layer, these slight changes are amplified as the number of layers deepens, and the input distribution of each layer changes; on the other hand, with the change of parameters, former layers need to adapt to these distribution changes, which makes it more difficult to train the model. These are called Internal Covariate Shift.

Batch Normalization is proposed to solve these two problems. We perform normalization on each layer separately, making the features of each layer have a mean of 0 and a variance of 1, which lets the value of each layer propagate in the effective range. we can also add a linear transformation operation, so that the data can restore the ability of expression.

The intuition of Batch Normalization is not to prevent over-fitting or prevent the gradient from vanishing or exploding but to increase the robustness by normalizing the parameter. This constraint also improves the structural rationality of the system, which brings a series of improvements, e.g. accelerate convergence, prevent over-fitting, etc.

#### Dropout

When we are training the model, we can set a probability P for eliminating a node in the deep neural network. For each node, there is a probability of (1-P) for keeping it and a probability of P for dropping it. Then we perform forward-propagation and backpropagation-propagation on this much-diminished deep neural network.

At each training step of a mini-batch dataset, the process of dropout creates a different deep neural network by randomly removing some units regarding the probability P. The process of dropout is similar to use ensemble learning on many different deep neural networks, each trained with a separate mini-batch dataset but share some context in the process of training.

In ensemble learning, since each classifier has been trained separately, it has learned different aspects of the dataset and their mistakes are different. Combining them helps to produce a more accurate classifier, which is less prone to overfitting. We can view dropout as a form of ensemble learning, and this is why it can prevent overfitting.

### 2. Data Augmentation

In Deep Learning, collecting data is an effective way to enhance the training but also a tedious and intricate process. In fact, for the majority of image recognition problems, we cannot get as much data as we expect. In order to obtain more data, **data augmentation** techniques is a very efficient method to improve the result.

For Convolutional Neural Networks, some common image augmentation techniques are listed:

- mirroring/flipping (on vertical or horizontal axis)
- rotating
- cropping
- scaling
- warping
- color shifting
- adding noise

There are two ways to do data augmentation: **Offline Augmentation** and **Online Augmentation**.

#### Offline Augmentation

**Offline Augmentation** is to perform all augmentations in advance, before training, and save all the augmented data in memory. In fact, it will increase, or multiple, the size of the dataset. This method fits small datasets well.

#### Online Augmentation

Another option, **Online Augmentation**, is to perform these augmentations on a mini-batch dataset right before they are fed to the model. This method is more suitable for large datasets because we do not have enough memory for staging all the augmented data when its scale is too enormous, so what we do is to perform mini-batch augmentation before feeding the current batch of data into the model.

### 3. Early Stopping

Early Stopping is a trade-off between training epochs and validation accuracy. At the end of each epoch, compare current validation accuracy of this epoch with the best validation accuracy. If the accuracy on the validation set decreases or does not reach the best one for more than 10 consecutive epochs, we stop the training, and we think the accuracy is no longer improved.

### 4. Simplify The Model

If we have done all the methods above to prevent overfitting but the performance is still bad, we may consider simplifying our model. The model may be too complicated for our dataset to fit and we can try to reduce the complexity of the model in some ways, e.g. reduce the number of layers, remove some neurons, etc.