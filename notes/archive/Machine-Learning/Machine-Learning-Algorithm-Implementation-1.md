---
title: "Machine Learning Algorithm Implementation (1): Linear Regression"
date: 2019-01-30T14:38:43+11:00
draft: true
categories: ["Machine Learning"]
markup: mmark
---
Click [here](https://github.com/ZintrulCre/Machine-Learning/blob/master/Linear%20Regression.ipynb) to see the implementation in Jupyter Notebook

# Linear regression

- Goal
    - fit a linear model, using only the `numpy` library
    - visualize the output, using the `matplotlib` library
- Implementation Methods
    - iterative updates (coordinate descent)
    - linear algebra
- Verification
    - compare our output to the output of using library `sklearn`.

## 1. Import Library

Firstly we should import the relevant libraries (`numpy`, `matplotlib`, etc.).

```
import numpy as np
import matplotlib.pyplot as plt
import io
```

## 2. Formula

The linear model can be expressed as:

$$
y = w_0 + \sum_{j = 1}^{m} w_j x_j = \mathbf{w} \cdot \mathbf{x}
$$

where

- $$y$$ is the **target variable**
- $$\mathbf{x} = [x_1, \ldots, x_m]$$ is a vector of **features** ($$x_0 = 1$$)
- $$\mathbf{w} = [w_0, \ldots, w_m]$$ are the **weights**.

To fit the model, we should **minimise** the empirical risk with respect to $$\vec{w}$$.

In this case, this amounts to minimising the sum of squared residuals (square loss):

$$SSR(\mathbf{w}) = \sum_{i=1}^{n}(y_i - \mathbf{w} \cdot \mathbf{x}_i)^2$$

For simplicity, we only consider the case $$m = 1$$ (i.e. only one feature excluding the intercept).

## 3. Import Data set

Let's use some data of the gold medal race times for marathon winners from 1896 to 2012 of Olympics.

The code block below reads the data into a numpy array of floats, and prints the result.

```
# CSV file with variables YEAR,TIME
csv ="""
1896,4.47083333333333
1900,4.46472925981123
1904,5.22208333333333
1908,4.1546786744085
1912,3.90331674958541
1920,3.5695126705653
1924,3.8245447722874
1928,3.62483706600308
1932,3.59284275388079
1936,3.53880791562981
1948,3.6701030927835
1952,3.39029110874116
1956,3.43642611683849
1960,3.2058300746534
1964,3.13275664573212
1968,3.32819844373346
1972,3.13583757949204
1976,3.07895880238575
1980,3.10581822490816
1984,3.06552909112454
1988,3.09357348817
1992,3.16111703598373
1996,3.14255243512264
2000,3.08527866650867
2004,3.1026582928467
2008,2.99877552632618
2012,3.03392977050993
"""

# Read into a numpy array as floats
olympics = np.genfromtxt(io.BytesIO(csv.encode()), delimiter=",")
print(olympics)
```

Now we take the race time as the **target variable** $$y$$ and the year of the race as the only **feature** $$x = x_1$$.

```
x = olympics[:, 0:1]
y = olympics[:, 1:2]
```

Plot $$y$$ and $$x$$.

```
plt.plot(x, y, 'rx')
plt.ylabel("y (Race time)")
plt.xlabel("x (Year of race)")
plt.show()
```

![1](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-1/1.png)


We can see that a linear model could be a decent fit for this data.

## 4. Iterative solution (coordinate descent)

Expanding out the sum of square residuals for this simple case (where $$\mathbf{w}=[w_0, w_1]$$) we have:

$$SSR(w_0, w_1) = \sum_{i=1}^{n}(y_i - w_0 - w_1 x_i)^2$$

Let's start with an initial guess for the slope $$w_1$$ (which is clearly negative from the plot).

```
w1 = -0.5
```

Then using the maximum likelihood update, we can get the following estimate for the intercept $$w_0$$

$$w_0 = \frac{\sum_{i=1}^{n}(y_i-w_1 x_i)}{n}$$

```
def update_w0(x, y, w1):
    return (y - w1*x).mean()

w0 = update_w0(x, y, w1)
print(w0)
```

Similarly, we can update $$w_1$$ based on this new estimate of $$w_0$$

$$w_1 = \frac{\sum_{i=1}^{n} (y_i - w_0) \times x_i}{\sum_{i=1}^{n} x_i^2}$$

```
def update_w1(x, y, w0):
    return ((y - w0)*x).sum()/(x**2).sum()

w1 = update_w1(x, y, w0)
print(w1)
```

Now let's examine the quality of fit for these values for the weights $$w_0$$ and $$w_1$$. We create a vector of "test" values `x_test` and a function to compute the predictions according to the model.

```
x_test = np.arange(1890, 2020)[:, None]

def predict(x_test, w0, w1): 
    return w0 + w1 * x_test

y_test = predict(x_test, w0, w1)
print(y_test)
```

Now plot the test predictions with a blue line on the same plot as the data.

```
def plot_fit(x_test, y_test, x, y): 
    plt.plot(x_test, y_test, 'b-')
    plt.plot(x, y, 'rx')
    plt.ylabel("y (Race time)")
    plt.xlabel("x (Year of race)")
    plt.show()

plot_fit(x_test, predict(x_test, w0, w1), x, y)
```

![2](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-1/2.png)

We can compute the sum of square residuals $$SSR(w_0,w_1)$$ on the training set to measure the goodness of fit.

```
def compute_SSR(x, y, w0, w1): 
    return ((y - w0 - w1*x)**2).sum()
    
print(compute_SSR(x, y, w0, w1))
```

It's obvious that the fit isn't very good from the plot. 

We should repeat the alternating parameter updates many times before the algorithm converges to the optimal weights.

```
for i in np.arange(10000):
    w1 = update_w1(x, y, w0) 
    w0 = update_w0(x, y, w1) 
    if i % 500 == 0:
        print("Iteration #{}: SSR = {}".format(i, compute_SSR(x, y, w0, w1)))
print("Final estimates: w0 = {}; w1 = {}".format(w0, w1))
```

Let's try plotting the result again.

```
plot_fit(x_test, predict(x_test, w0, w1), x, y)
```

![3](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-1/3.png)

## 5. Linear algebra solution

it's possible to solve for the optimal weights $$\mathbf{w}^\star$$ analytically. The solution is

$$\mathbf{w}^* = \left[\mathbf{X}^\top \mathbf{X}\right]^{-1} \mathbf{X}^\top \mathbf{y}$$

where

$$\mathbf{X} = \begin{pmatrix} 1 & x_1 \\ 1 & x_2 \\ \vdots & \vdots \\ 1 & x_n \end{pmatrix} \quad \text{and} \quad \mathbf{y} = \begin{pmatrix} y_1 \\ y_2 \\ \vdots \\ y_n\end{pmatrix}$$

We construct $$\mathbf{X}$$ in the code block below. It's important to include the $$x_0 = 1$$ column for the bias (intercept).

```
X = np.hstack((np.ones_like(x), x))
print(X)
```

Although we can express $$\mathbf{w}^\star$$ explicitly in terms of the matrix inverse $$(\mathbf{X}^\top \mathbf{X})^{-1}$$, this isn't an efficient way to compute $$\mathbf{w}$$ numerically. It is better instead to solve the following system of linear equations

$$\mathbf{X}^\top\mathbf{X} \mathbf{w}^\star = \mathbf{X}^\top\mathbf{y}$$

This can be done in numpy using the function linalg.solve

```
w = np.linalg.solve(np.dot(X.T, X), np.dot(X.T, y))
print(w)
```

Plotting this solution

```
w0, w1 = w
plot_fit(x_test, predict(x_test, w0, w1), x, y)
```

![4](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Machine-Learning/Machine-Learning-Algorithm-Implementation-1/4.png)

We should verify that the sum of squared residuals $$SSR(w_0, w_1)$$, matches or beats the earlier iterative result.

```
print(compute_SSR(x, y, w0, w1))
```

The error we computed above is the **training error**. It doesn't assess the model's generalization ability, it only assesses how well it's performing on the given training data. We can assess the generalization ability of models using held-out evaluation data.

## 6. Verification using scikit-learn

Now we can use the functionality in `sklearn` to solve linear regression problems to do the verification.

Use the `LinearRegression` module to fit a linear regression model.

```
from sklearn.linear_model import LinearRegression
lr = LinearRegression().fit(x, y)

lr.intercept_ # bias weight

lr.coef_ # weights

lr.score(x,y)
```