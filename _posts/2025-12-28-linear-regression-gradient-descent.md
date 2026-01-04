---
layout: post
title: "ML Foundations: Linear Regression and Gradient Descent"
date: 2025-12-28 12:30:00 +0545
categories: [Python, Machine Learning]
tags: [linear-regression, gradient-descent, machine-learning, python, data-science]
---

# Machine Learning Foundations: Linear Regression and Gradient Descent

## Introduction

Welcome to our foundational series on Machine Learning! This series is for everyone—whether you're a student preparing for an exam, a teacher designing a course, or a seasoned developer venturing into the world of AI. We'll prioritize clear concepts and practical code over dense academic theory.

Today, we're starting with a cornerstone algorithm: **Linear Regression**. Imagine you have a scatter plot of data—say, house sizes versus their prices. Linear regression is the technique we use to draw the "best-fit" straight line through that data to predict the price for a new, unseen house size.

But how do we find that "best" line? That's where **Gradient Descent** comes in. It's an optimization algorithm, our mathematical GPS, that guides us to the ideal parameters for our line to minimize prediction errors.

In this post, we'll explore:
1.  The simple math behind a straight line.
2.  How we measure the "error" of our line using a **Cost Function**.
3.  The intuition behind **Gradient Descent**.
4.  The three main "flavors" of Gradient Descent: **Batch**, **Stochastic**, and **Mini-Batch**.
5.  A practical implementation from scratch with Python, and then the easy way with Scikit-Learn.

## 1. The Goal: Modeling a Simple Line

At its heart, linear regression is about finding the parameters of a line. You might remember the equation from school:

`y = mx + c`

In machine learning, we often write it with slightly different notation:

`h(x) = θ₁x + θ₀`

- `h(x)` is our hypothesis or predicted value (e.g., predicted house price).
- `x` is our input feature (e.g., house size).
- `θ₁` (theta 1) is the **weight** or **slope** (`m` in the old equation). It determines how much the price increases per square foot.
- `θ₀` (theta 0) is the **bias** or **y-intercept** (`c`). It's the base price of the house, even if its size was zero.

Our entire goal is to find the optimal values for `θ₁` and `θ₀` that create the best-fitting line.

## 2. Measuring Error: The Cost Function

How do we define the "best" line? It's the line where the difference between the predicted prices (`h(x)`) and the actual prices (`y`) is as small as possible. This difference is called the **error** or **residual**.

We need a single number to quantify the total error across all our data points. For this, we use a **Cost Function**. The most common one for linear regression is the **Mean Squared Error (MSE)**.

**MSE Formula:** `J(θ₀, θ₁) = (1 / 2m) * Σ(h(xᵢ) - yᵢ)²`

Let's break it down:
- `m` is the number of data points (e.g., number of houses).
- `Σ` is the sum over all data points (from `i=1` to `m`).
- `h(xᵢ) - yᵢ` is the error for a single data point.
- `(...)²`: We square the error. This does two things: it ensures all errors are positive, and it penalizes larger errors more heavily.
- `(1 / 2m)`: We average the squared errors. The `2` is just a mathematical convenience that simplifies the derivative later on.

Our goal is now refined: **find the `θ₀` and `θ₁` that minimize the cost function `J`**.

## 3. Finding the Best Line: Gradient Descent

Imagine the cost function `J` as a 3D bowl. The two horizontal axes are `θ₀` and `θ₁`, and the vertical axis is the cost (MSE). We are standing somewhere on the surface of this bowl, and our goal is to walk to the very bottom—the point of minimum cost.

How do you walk downhill in the fog? You feel the slope under your feet and take a step in the steepest downward direction. That's exactly what Gradient Descent does.

The "slope" of the cost function is called the **gradient**. It's a vector that points in the direction of the steepest ascent. To go downhill, we just move in the opposite direction of the gradient.

We do this iteratively with two main steps:
1.  Calculate the gradient of the cost function with respect to each parameter (`θ₀` and `θ₁`).
2.  Update each parameter by taking a small step in the negative gradient direction.

**The Update Rule:**
`θⱼ := θⱼ - α * (∂J / ∂θⱼ)`

- `θⱼ` is the parameter we are updating (`θ₀` or `θ₁`).
- `:=` means we are updating the value.
- `α` (alpha) is the **learning rate**. It's a small number (e.g., 0.01) that controls how big of a step we take. Too big, and we might overshoot the minimum. Too small, and the algorithm will be too slow.
- `(∂J / ∂θⱼ)` is the partial derivative of the cost function `J` with respect to the parameter `θⱼ`—this is the gradient!

We repeat this update step for both `θ₀` and `θ₁` until the cost stops decreasing significantly, meaning we've reached the bottom.

## 4. The Flavors of Gradient Descent

The key difference between the types of Gradient Descent is *how much data* we use to calculate the gradient in each step.

Let's create some sample data to see them in action.

```python
import numpy as np

# Generate some sample data
X = 2 * np.random.rand(100, 1)
y = 4 + 3 * X + np.random.randn(100, 1) # y = 4 + 3x + noise

# Add x0 = 1 to each instance (for the bias term θ₀)
X_b = np.c_[np.ones((100, 1)), X]
```

### A. Batch Gradient Descent

In Batch GD, we calculate the gradient using the **entire dataset** for every single step.

- **Pros:** It's a deterministic and stable path towards the minimum. For a convex cost function like MSE, it's guaranteed to converge to the global minimum.
- **Cons:** It is incredibly slow and memory-intensive for large datasets, as you have to load and process all the data for each step.

```python
# --- Batch Gradient Descent Implementation ---
learning_rate = 0.1
n_iterations = 1000
m = 100 # Number of data points

theta = np.random.randn(2, 1) # Random initialization [θ₀, θ₁]

for iteration in range(n_iterations):
    gradients = (2/m) * X_b.T.dot(X_b.dot(theta) - y)
    theta = theta - learning_rate * gradients

print("Batch GD Theta:", theta)
```

### B. Stochastic Gradient Descent (SGD)

In SGD, we do the opposite. We calculate the gradient using just **one randomly selected data point** at each step.

- **Pros:** It's very fast and uses very little memory. The randomness can also help it jump out of local minima in non-convex problems.
- **Cons:** The path to the minimum is very noisy and erratic. It never truly "settles" at the minimum but bounces around it.

```python
# --- Stochastic Gradient Descent Implementation ---
n_epochs = 50
t0, t1 = 5, 50  # Learning schedule hyperparameters

def learning_schedule(t):
    return t0 / (t + t1)

theta = np.random.randn(2, 1)  # Random initialization

for epoch in range(n_epochs):
    for i in range(m):
        random_index = np.random.randint(m)
        xi = X_b[random_index:random_index+1]
        yi = y[random_index:random_index+1]
        
        gradients = 2 * xi.T.dot(xi.dot(theta) - yi)
        eta = learning_schedule(epoch * m + i)
        theta = theta - eta * gradients

print("SGD Theta:", theta)
```

### C. Mini-Batch Gradient Descent

This is the most common and practical approach. We take a small, random subset of the data—a "mini-batch"—to calculate the gradient at each step.

- **Pros:** It's the best of both worlds. It's more stable than SGD but far more efficient than Batch GD. It also benefits from hardware optimizations for matrix operations.
- **Cons:** It introduces a new hyperparameter to tune: the `batch_size`.

```python
# --- Mini-Batch Gradient Descent Implementation ---
n_iterations = 50
minibatch_size = 20

theta = np.random.randn(2,1) # random initialization
t = 0

for epoch in range(n_iterations):
    shuffled_indices = np.random.permutation(m)
    X_b_shuffled = X_b[shuffled_indices]
    y_shuffled = y[shuffled_indices]
    for i in range(0, m, minibatch_size):
        t += 1
        xi = X_b_shuffled[i:i+minibatch_size]
        yi = y_shuffled[i:i+minibatch_size]
        
        gradients = 2/minibatch_size * xi.T.dot(xi.dot(theta) - yi)
        eta = learning_schedule(t)
        theta = theta - eta * gradients

print("Mini-Batch GD Theta:", theta)
```

## 5. The Easy Way: Scikit-Learn

While it's crucial to understand the mechanics, in practice, you'll almost always use a library like Scikit-Learn.

```python
from sklearn.linear_model import LinearRegression, SGDRegressor

# For LinearRegression (uses an exact analytical solution called Ordinary Least Squares)
lin_reg = LinearRegression()
lin_reg.fit(X, y)
print("Scikit-Learn LinearRegression (θ₀, θ₁):", lin_reg.intercept_, lin_reg.coef_)

# For SGDRegressor (uses Gradient Descent)
sgd_reg = SGDRegressor(max_iter=1000, tol=1e-3, penalty=None, eta0=0.1)
sgd_reg.fit(X, y.ravel()) # .ravel() is needed to flatten y
print("Scikit-Learn SGDRegressor (θ₀, θ₁):", sgd_reg.intercept_, sgd_reg.coef_)
```

## Conclusion

You've just learned the fundamentals of one of the most important concepts in machine learning!

- **Linear Regression** is about finding a line to model the relationship between variables.
- The **Cost Function (MSE)** tells us how good our line is.
- **Gradient Descent** is the optimizer that minimizes the cost by iteratively adjusting the line's parameters.
- **Batch GD** is slow but steady, **SGD** is fast but erratic, and **Mini-Batch GD** is the practical compromise that powers modern machine learning.

Understanding these core ideas is essential, as they are the foundation for many more complex algorithms, including neural networks.

{% include inarticle-adsense.html %}

## Suggested Reading

- **"Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow" by Aurélien Géron:** Chapter 4 provides an excellent, in-depth look at these concepts.
- **StatQuest with Josh Starmer on YouTube:** Search for his videos on Linear Regression and Gradient Descent for fantastic visual explanations.
- **Andrew Ng's Machine Learning Course (Coursera):** A classic and comprehensive introduction to the theory.
