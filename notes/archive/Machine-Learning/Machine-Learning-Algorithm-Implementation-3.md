---
title: "Machine Learning Algorithm Implementation (3): Perceptron"
date: 2019-02-05T00:58:34+11:00
draft: true
categories: ["Machine Learning"]
markup: mmark
---
Click [here](https://github.com/ZintrulCre/Machine-Learning/blob/master/Perceptron.ipynb) to see the implementation in Jupyter Notebook

# Perceptron

- Goal
    - implement the perceptron (a building block of neural networks)
    - to assess how the perceptron behaves in two distinct scenarios (separable vs. non-separable data)

- Verification
    - compare our output to the output of using `LogisticRegression` function from library `sklearn`.
    
## 1. Import Library

Firstly we should import the relevant libraries (`numpy`, `matplotlib`, etc.).

```
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
```

## 2. Synthetic data set

We'll use the `make_classification` function from `sklearn` to generate a synthetic binary classification data set.

The main advantage of using synthetic data is that we have complete control over the distribution. 
In particular, we'll be varying the **degree of separability** between the two classes by adjusting the `class_sep` parameter below.

Now we generate a data set that is almost linearly separable (with `class_sep = 2`).

We use `-1` in place of `0` for the negative class, and `1` for the positive class.
This is for making the gradient descent update simpler to implement.

```
class_sep = 2 # set the `class_sep` to 2 firstly
# class_sep = 0.5 # adjust the `class_sep` to 0.5 later

from sklearn.datasets import make_classification
X, Y = make_classification(n_samples=200, n_features=2, n_informative=2, 
                           n_redundant=0, n_clusters_per_class=1, flip_y=0,
                           class_sep=class_sep, random_state=1)
Y[Y==0] = -1 # encode "negative" class using -1 rather than 0
plt.plot(X[Y==-1,0], X[Y==-1,1], "o", label="Y = -1")
plt.plot(X[Y==1,0], X[Y==1,1], "o", label="Y = 1")
plt.legend()
plt.xlabel("$x_0$")
plt.ylabel("$x_1$")
plt.show()
```

![1](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/1.png)

We can see that this data is linearly seperable.

In preparation for training and evaluating a perceptron on this data, we randomly partition the data into train/test sets.

```
from sklearn.model_selection import train_test_split
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.33, random_state=90051)
print("Training set has {} instances. Test set has {} instances.".format(X_train.shape[0], X_test.shape[0]))
```

## 3. Definition of the perceptron

A perceptron is a binary classifier which maps an input vector $$\mathbf{x}$$ to a binary ouput given by

![0](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/0.png)

where $$s(\mathbf{x}; \mathbf{w}, b) = \mathbf{w} \cdot \mathbf{x} + b$$. 
$$\mathbf{w}$$ is a vector of weights (one for each feature) and $$b$$ is the bias term.

Now we start by implementing the weighted sum function $$s(\mathbf{x}; \mathbf{w}, b)$$.

```
def weighted_sum(X, w, b):
    """
    Returns an array containing the weighted sum s(x) for each instance x in the feature matrix
    
    Arguments:
    X : numpy array, shape: (n_instances, n_features)
        feature matrix
    w : numpy array, shape: (n_features,)
        weights vector
    b : float
        bias term
    """
    return np.dot(X, w) + b

def predict(X, w, b):
    """
    Returns an array containing the predicted binary labels (-1/1) for each instance in the feature matrix
    
    Arguments:
    X : numpy array, shape: (n_instances, n_features)
        feature matrix
    w : numpy array, shape: (n_features,)
        weights vector
    b : float
        bias term
    """
    return np.where(weighted_sum(X, w, b) >= 0, 1, -1)
```

## 4. Perceptron training algorithm

We're now going to implement the perceptron training algorithm.

The algorithm is essentially an application of **sequential gradient descent** (a.k.a. **online stochastic gradient descent**) to minimise the following empirical loss:

$$L[\mathbf{w}, b] = \frac{1}{n} \sum_{i = 1}^{n} \max(0, -y \cdot s(\mathbf{x}; \mathbf{w}, b))$$

It's **sequential** in that the weights/bias are updated for each training instance—one at a time.
After iterating through all of the training instances, we say that we've completed an **epoch**.
Typically, multiple epochs are required to get close to the optimal solution.

Now we write a function called `train` which implements sequential gradient descent. The function should implement the following pseudocode.

> repeat $$n_\mathrm{epochs}$$ times

> >   for each $$(\mathbf{x}, y)$$ pair in the training set

> > > if the model prediction $$\hat{y} = f(\mathbf{x})$$ and $$y$$ differ, make a weight update

> return $$\mathbf{w}$$ and $$b$$

Note that the weight update in the inner-most loop is given by $$\mathbf{w} \gets \mathbf{w} + \eta y  \mathbf{x}$$ and $$b \gets b + \eta y$$.

```
def train(X, Y, n_epochs, w, b, eta=0.1):
    """
    Returns updated weight vector w and bias term b
    
    Arguments:
    X : numpy array, shape: (n_instances, n_features)
        feature matrix
    Y : numpy array, shape: (n_instances,)
        target class labels relative to X
    n_epochs : int
        number of epochs (full sweeps through X)
    w : numpy array, shape: (n_features,)
        initial guess for weights vector
    b : float
        initial guess for bias term
    eta : positive float
        step size (default: 0.1)
    """
    for t in range(n_epochs):
        for i in range(X.shape[0]):
            yhat = predict(X[i,:], w, b)
            if yhat != Y[i]:
                w += eta * Y[i] * X[i,:]
                b += eta * Y[i]
    return w, b
```

Test our implementation by running it for 5 epochs. We can get the following result for the weights and bias term:
`w = [ 0.26746342 -0.96011853]; b = -0.2`

```
# Initialise weights and bias to zero
w = np.zeros(X.shape[1]); b = 0.0

w, b = train(X_train, Y_train, 5, w, b)
print("w = {}; b = {}".format(w, b))
```

## 5. Evaluation

Now that we've trained the perceptron, let's see how it performs.

Now we plot the data (training and test sets) along with the decision boundary (which is defined by $$\{\mathbf{x}: s(\mathbf{x}; \mathbf{w}, b) = 0$$\}).

```
def plot_results(X_train, Y_train, X_test, Y_test, score_fn, threshold = 0):
    # Plot training set
    plt.plot(X_train[Y_train==-1,0], X_train[Y_train==-1,1], "o", label="Y=-1, train")
    plt.plot(X_train[Y_train==1,0], X_train[Y_train==1,1], "o", label="Y=1, train")
    plt.gca().set_prop_cycle(None) # reset colour cycle

    # Plot test set
    plt.plot(X_test[Y_test==-1,0], X_test[Y_test==-1,1], "x", label="Y=-1, test")
    plt.plot(X_test[Y_test==1,0], X_test[Y_test==1,1], "x", label="Y=1, test")

    # Compute axes limits
    border = 1
    x0_lower = X[:,0].min() - border
    x0_upper = X[:,0].max() + border
    x1_lower = X[:,1].min() - border
    x1_upper = X[:,1].max() + border

    # Generate grid over feature space
    resolution = 0.01
    x0, x1 = np.mgrid[x0_lower:x0_upper:resolution, x1_lower:x1_upper:resolution]
    grid = np.c_[x0.ravel(), x1.ravel()]
    s = score_fn(grid).reshape(x0.shape)

    # Plot decision boundary (where s(x) == 0)
    plt.contour(x0, x1, s, levels=[0], cmap="Greys", vmin=-0.2, vmax=0.2)

    plt.legend()
    plt.xlabel("$x_0$")
    plt.ylabel("$x_1$")
    plt.show()
    
plot_results(X_train, Y_train, X_test, Y_test, lambda X: weighted_sum(X, w, b))
```

![2](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/2.png)

The decision boundary is acceptable for `class_sep=2`.

To evaluate the perceptron quantitatively, we use the error rate (proportion of misclassified instances).
The error rate is a reasonable evaluation measure to use for this data since the classes are well balanced.

```
def evaluate(X, Y, w, b):
    """
    Returns the proportion of misclassified instances (error rate)
    
    Arguments:
    X : numpy array, shape: (n_instances, n_features)
        feature matrix
    Y : numpy array, shape: (n_instances,)
        target class labels relative to X
    w : numpy array, shape: (n_features,)
        weights vector
    b : float
        bias term
    """
    return np.mean(predict(X, w, b) != Y)

print(evaluate(X_train, Y_train, w, b))
```

The `evaluate` function computes the error rate on the training data, but it is not a good idea in general.

Compute the error rate on the test set instead.

```
print(evaluate(X_test, Y_test, w, b))
```

The error on the test set is actually smaller than the error on the training set. This suggests we may not have a problem with overfitting (but it's difficult to say for such a small sample size).

Now we examine how the train/test error rates vary as a function of the number of epochs.
Note that careful tuning of the learning rate is needed to get sensible behaviour.
Setting $$\eta(t) = \frac{1}{1+t}$$ where $$t$$ is the epoch number often works well.

```
w_hat = np.zeros(X_train.shape[1]); b_hat = 0
n_epochs = 100

# Initialize arrays to store errors for each epoch
train_error = np.empty(n_epochs)
heldout_error = np.empty(n_epochs)

for t in range(n_epochs):
    # here we use a learning rate, which decays with each epoch
    eta = 1./(1+t)
    w_hat, b_hat = train(X_train, Y_train, 1, w_hat, b_hat, eta=eta)    
    train_error[t] = evaluate(X_train, Y_train, w_hat, b_hat)
    heldout_error[t] = evaluate(X_test, Y_test, w_hat, b_hat)

plt.plot(train_error, label = 'Train error')
plt.plot(heldout_error, label = 'Test error')
plt.legend()
plt.xlabel('Epoch, $t$')
plt.ylabel('Error')
plt.show()
```

![3](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/3.png)

The train/test errors are quite close. This suggests we may not have a problem with overfitting.

The model is relatively stable and converges quite rapidly for `class_sep=2`. However, The model is highly unstable for `class_sep=0.5` if we adjust the `class_sep` to 0.5.

```
plot_results(X_train, Y_train, X_test, Y_test, lambda X: weighted_sum(X, w_hat, b_hat))
```

![4](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/4.png)

## 6. Repeat with class overlap

By now we've probably concluded that the perceptron performs well on the data with `class_sep=2`, which is to be expected as it's roughly linearly separable. 

However, the perceptron can actually fail on non-linearly separable data if we re-generate the synthetic data set with `class_sep=0.5` and repeat the program.

## 7. Comparison with logistic regression

We know that the perceptron is not robust to binary classification problems with overlapping classes. 
But how does logistic regression fare in this case?

Run the code block below to fit a logistic regression model using `sklearn`. 
We may wish to switch off regularisation (alter the `C` parameter) for a fairer comparison with the perceptron.

```
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression()
clf.fit(X_train, Y_train)
```

Let's plot the decision boundary.

```
plot_results(X_train, Y_train, X_test, Y_test, lambda X: clf.decision_function(X))
```

![5](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-3/5.png)

The logistic regression classifier seems to produce a more reasonable "approximate" solution in the presence of class overlap. The support vector machine was designed to address this problem.

The error rate is lower for logistic regression (around 0.23). The error rate for the perceptron fluctuates, but never seems to reach a comparably low value.

```
1.0 - clf.score(X_test, Y_test)
```