---
title: "Machine Learning Algorithm Implementation (2): Logistic Regression"
date: 2019-02-01T14:00:16+11:00
draft: true
categories: ["Machine Learning"]
markup: mmark
---
Click [here](https://github.com/ZintrulCre/Machine-Learning/blob/master/Logistic%20Regression.ipynb) to see the implementation in Jupyter Notebook

# Logistic Regression

- Goal
    - implementing L2-regularised logistic regression, using `scipy` and `numpy` library
    - apply polynomial basis expansion and L2 regularisation.
- Implementation Methods
    - hill-climbing algorithm
- Verification
    - compare our output to the output of using library `sklearn`.
    
## 1. Import Library

Firstly we should import the relevant libraries (`numpy`, `matplotlib`, etc.).

```
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
```

## 2. Binary classification data

Firstly, we generate some binary classification data using the `make_circles` function from `sklearn`.

```
from sklearn.datasets import make_circles
X, Y = make_circles(n_samples=300, noise=0.1, factor=0.7, random_state=90051)
plt.plot(X[Y==0,0], X[Y==0,1], 'o', label = "y=0")
plt.plot(X[Y==1,0], X[Y==1,1], 's', label = "y=1")
plt.legend()
plt.xlabel("$x_0$")
plt.ylabel("$x_1$")
plt.show()
```

![1](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-2/1.png)

In preparation for fitting and evaluating a logistic regression model, we randomly partition the data into train/test sets using the `train_test_split` function from `sklearn`.

```
from sklearn.model_selection import train_test_split
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.33, random_state=90051)
print("Training set has {} instances. Test set has {} instances.".format(X_train.shape[0], X_test.shape[0]))
```

## 2. Logistic regression objective function

The logistic regression models the distribution of the binary class $$y$$ *conditional* on the feature vector $$\mathbf{x}$$ as

$$
y | \mathbf{x} \sim \mathrm{Bernoulli}[\sigma(\mathbf{w}^T \mathbf{x} + b)]
$$

where

- $$\mathbf{w}$$ is the weight vector
- $$b$$ is the bias term
- $$\sigma(z) = 1/(1 + e^{-z})$$ is the logistic function.

To simplify the notation, we collect the parameters $$\mathbf{w}$$ and $$b$$ into a single vector $$\mathbf{v} = [b, \mathbf{w}]$$.

Fitting this model amounts to choosing $$\mathbf{v}$$ that minimises the sum of cross-entropies over the instances ($$i = 1,\ldots,n$$) in the training set

$$
f_\mathrm{cross-ent}(\mathbf{v}; \mathbf{X}, \mathbf{Y}) = - \sum_{i = 1}^{n} \left\{ y_i \log \sigma(\mathbf{w}^T \mathbf{x}_i + b) + (1 - y_i) \log (1 - \sigma(\mathbf{w}^T \mathbf{x}_i + b)) \right\}
$$

Often a regularisation term of the form $$f_\mathrm{reg}(\mathbf{w}; \lambda) = \frac{1}{2} \lambda \mathbf{w}^T \mathbf{w}$$ is added to the objective to penalize large weights in orderto prevent overfitting. Note that $$\lambda \geq 0$$ controls the strength of the regularisation term.

Putting this together, our goal is to minimise the following objective function with respect to $$\mathbf{w}$$ and $$b$$:

$$
f(\mathbf{v}; \mathbf{X}, \mathbf{Y}, \lambda) = f_\mathrm{reg}(\mathbf{w}; \lambda) + f_\mathrm{cross-ent}(\mathbf{v}; \mathbf{X}, \mathbf{Y})
$$

If we replace $$\mathbf{w}$$ with $$\mathbf{v}$$ in the regularisation term, we'd be penalising large $$b$$ because a large bias may be required for some data sets, and restricting the bias doesn't help with generalisation. So only $$\mathbf{w}$$ is included in $$f_\mathrm{reg}$$.

We use the BFGS algorithm to solve this minimisation problem. BFGS is a "hill-climbing" algorithm like gradient descent, however it additionally makes use of second-order derivative information by approximating the Hessian. It converges in fewer iterations than gradient descent. It's convergence rate is superlinear whereas gradient descent is only linear.

We use the function `fmin_bfgs` from `scipy`. The algorithm requires two functions as input:

1. a function that evaluates the objective $$f(\mathbf{v}; \ldots)$$
2. a function that evalutes the gradient $$\nabla_{\mathbf{v}} f(\mathbf{v}; \ldots)$$.

Firstly we write a function to compute $$f(\mathbf{v}; \ldots)$$.

```
from scipy.special import expit # logistic function

# v: parameter vector
# X: feature matrix
# Y: class labels
# Lambda: regularisation constant
def obj_fn(v, X, Y, Lambda):
    prob_1 = expit(np.dot(X,v[1::]) + v[0])
    reg_term = 1 * Lambda * np.dot(v[1::].T,v[1::])
    cross_entropy_term = - np.dot(Y, np.log(prob_1)) - np.dot(1. - Y, np.log(1. - prob_1))
    return reg_term + cross_entropy_term
```

Now for the gradient, we use the following result:

$$
\nabla_{\mathbf{v}} f(\mathbf{v}; \ldots) = \left[\frac{\partial f(\mathbf{w}, b;\ldots)}{\partial b}, \nabla_{\mathbf{w}} f(\mathbf{w}, b; \ldots) \right] = \left[\sum_{i = 1}^{n} \sigma(\mathbf{w}^T \mathbf{x}_i + b) - y_i, \lambda \mathbf{w} + \sum_{i = 1}^{n} (\sigma(\mathbf{w}^T \mathbf{x}_i + b) - y_i)\mathbf{x}_i\right]
$$

```
# v: parameter vector
# X: feature matrix
# Y: class labels
# Lambda: regularisation constant
def grad_obj_fn(v, X, Y, Lambda):
    prob_1 = expit(np.dot(X, v[1::]) + v[0])
    grad_b = np.sum(prob_1 - Y)
    grad_w = Lambda * v[1::] + np.dot(prob_1 - Y, X)
    return np.insert(grad_w, 0, grad_b)
```

## 3. Solving the minimization problem using BFGS

Now that we've implemented functions to compute the objective and the gradient, we can plug them into `fmin_bfgs`.

We define a function `logistic_regression` which calls `fmin_bfgs` and returns the optimal weight vector.

```
from scipy.optimize import fmin_bfgs

# X: feature matrix
# Y: class labels
# Lambda: regularisation constant
# v_initial: initial guess for parameter vector
def logistic_regression(X, Y, Lambda, v_initial, disp=True):
    # Function for displaying progress
    def display(v):
        print('v is', v, 'objective is', obj_fn(v, X, Y, Lambda))
    
    return fmin_bfgs(f=obj_fn, fprime=grad_obj_fn, 
                     x0=v_initial, args=(X, Y, Lambda), disp=disp, 
                     callback=display)
```
```
Lambda = 1
v_initial = np.zeros(X_train.shape[1] + 1)
v_opt = logistic_regression(X_train, Y_train, Lambda, v_initial)

# Function to plot the data points and decision boundary
def plot_results(X, Y, v, trans_func = None):
    # Scatter plot in feature space
    plt.plot(X[Y==0,0], X[Y==0,1], 'o', label = "y=0")
    plt.plot(X[Y==1,0], X[Y==1,1], 's', label = "y=1")
    
    # Compute axis limits
    x0_lower = X[:,0].min() - 0.1
    x0_upper = X[:,0].max() + 0.1
    x1_lower = X[:,1].min() - 0.1
    x1_upper = X[:,1].max() + 0.1
    
    # Generate grid over feature space
    x0, x1 = np.mgrid[x0_lower:x0_upper:.01, x1_lower:x1_upper:.01]
    grid = np.c_[x0.ravel(), x1.ravel()]
    if (trans_func is not None):
        grid = trans_func(grid) # apply transformation to features
    arg = (np.dot(grid, v[1::]) + v[0]).reshape(x0.shape)
    
    # Plot decision boundary (where w^T x + b == 0)
    plt.contour(x0, x1, arg, levels=[0], cmap="Greys", vmin=-0.2, vmax=0.2)
    plt.legend()
    plt.show()
    
plot_results(X, Y, v_opt)
```

![2](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-2/2.png)

It's not a good fit because logistic regression is a linear classifier, and the data is not linearly seperable.

Now we calculate the accuracy of this model:

$$\hat{y} = \begin{cases}1, &\mathrm{if} \ p(y = 1|\mathbf{x}) \geq \tfrac{1}{2}, \\0, &\mathrm{otherwise}.\end{cases}$$

## 4. Adding polynomial features

Ordinary logistic regression does poorly on this data set because the data is not linearly separable in the $$x_0,x_1$$ feature space.

We can get around this problem using basis expansion. In this case, we'll augment the feature space by adding polynomial features of degree 2. In other words, we replace the original feature matrix $$\mathbf{X}$$ by a transformed feature matrix $$\mathbf{\Phi}$$ which contains additional columns corresponding to $$x_0^2$$, $$x_0 x_1$$ and $$x_1^2$$.

There is a built-in function `preprocessing.PolynomialFeatures` from `sklearn` to add polynomial featrues, but we can implement the function `add_quadratic_features` by ourselves.

```
# X: original feature matrix
def add_quadratic_features(X):
    return np.c_[X, X[:,0]**2, X[:,0]*X[:,1], X[:,1]**2]

Phi_train = add_quadratic_features(X_train)
Phi_test = add_quadratic_features(X_test)
```

Now we apply our custom logistic regression function again on the augmented feature space.

```
Lambda = 1
v_initial = np.zeros(Phi_train.shape[1] + 1) # fill in a vector of zeros of appropriate length
v_opt = logistic_regression(Phi_train, Y_train, Lambda, v_initial)
plot_results(X, Y, v_opt, trans_func=add_quadratic_features)
```

![3](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-2/3.png)

This time we should get a better result for the accuracy on the test set.

```
from sklearn.metrics import accuracy_score
Y_test_pred = ((np.dot(Phi_test, v_opt[1::]) + v_opt[0]) >= 0)*1 # fill in
accuracy_score(Y_test, Y_test_pred)
```

We've chosen the regularisation constant as $$\lambda = 1$$, but it's possible to choose an optimal value for $$\lambda$$ by applying cross-validation.

If we try to set $$\lambda$$ to a small value  (say $$10^{-3}$$) or switched off entirely, we will risk overfitting. We can observe that the accuracy on the test set reduces slightly with $$\lambda = 10^{-3}$$ vs. $$\lambda = 1$$.*

## 5. Verification using scikit-learn

Now that we have some insight into the optimisation problem behind logistic regression, we should feel confident in using the built-in implementation in `sklearn`.
The `sklearn` implementation handles floating point underflow/overflow more carefully than we have done, and uses faster numerical optimisation algorithms.

```
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(solver='lbfgs')
clf.fit(Phi_train, Y_train)
```

```
from sklearn.metrics import accuracy_score
Y_test_pred = clf.predict(Phi_test)
accuracy_score(Y_test, Y_test_pred)
```