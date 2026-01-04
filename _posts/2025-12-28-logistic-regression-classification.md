---
layout: post
title: "ML Foundations: Logistic Regression for Classification"
date: 2025-12-28 13:30:00 +0545
categories: [Python, Machine Learning]
tags: [logistic-regression, classification, machine-learning, python, data-science]
---

# Machine Learning Foundations: Logistic Regression for Classification

## Introduction

In our previous post, we explored Linear Regression, a powerful tool for predicting continuous values like house prices. But what if our prediction isn't a number on a spectrum? What if we need to answer a "yes" or "no" question?

- Is this email spam or not?
- Is this transaction fraudulent or legitimate?
- Will this customer churn or stay?

These are **classification** problems, and they require a different approach. Welcome to **Logistic Regression**, a foundational algorithm for binary classification (predicting one of two possible outcomes). Despite its name, Logistic Regression is used for classification, not regression.

This guide will walk you through the intuition and practical application of Logistic Regression, building directly on the concepts of Gradient Descent we've already learned.

In this post, we'll cover:
1.  Why Linear Regression isn't suitable for classification.
2.  The **Sigmoid Function**, which maps any value to a probability between 0 and 1.
3.  How to interpret this probability using a **Decision Boundary**.
4.  A new cost function for classification: **Log Loss (Binary Cross-Entropy)**.
5.  A practical example using Python and Scikit-Learn.

## 1. From Linear Regression to a Probability

Let's start with what we know: the equation for a line from Linear Regression.

`z = θ₁x₁ + θ₂x₂ + ... + θₙxₙ + θ₀`

Here, `z` can be any real number, from negative infinity to positive infinity. This is great for predicting prices, but it's problematic for classification. How do you map a value of, say, 47.3 to "spam" or "not spam"?

We could try to set a threshold (e.g., if `z > 0.5`, predict "spam"), but the unbounded nature of `z` makes it sensitive to outliers and hard to interpret. We need something better. We need a probability.

This is where the **Sigmoid Function** (also called the **Logit Function**) comes in. It's a beautiful S-shaped function that takes any real number `z` and "squashes" it into a value between 0 and 1.

**Sigmoid Function:** `h(z) = 1 / (1 + e⁻ᶻ)`

![Sigmoid Function](https://upload.wikimedia.org/wikipedia/commons/8/88/Logistic-curve.svg)

- If `z` is a large positive number, `e⁻ᶻ` approaches 0, so `h(z)` approaches 1.
- If `z` is a large negative number, `e⁻ᶻ` approaches infinity, so `h(z)` approaches 0.
- If `z` is 0, `e⁻ᶻ` is 1, so `h(z)` is 0.5.

By plugging our linear equation into the sigmoid function, we get the Logistic Regression hypothesis:

`h(x) = 1 / (1 + e⁻⁽ᶻ⁾)` where `z` is our linear equation.

The output `h(x)` is now a probability. For example, if `h(x) = 0.85`, we are 85% confident that the input `x` belongs to the positive class (e.g., the email is spam).

## 2. Making a Decision: The Decision Boundary

Now that we have a probability, making a prediction is straightforward. We can set a simple threshold:

- If `h(x) ≥ 0.5`, predict class 1 (e.g., "spam").
- If `h(x) < 0.5`, predict class 0 (e.g., "not spam").

From the sigmoid curve, we know that `h(x) = 0.5` happens exactly when the input to the function, `z`, is 0.

So, the **decision boundary**—the line that separates our two classes—is simply the line where `z = 0`.

`θ₁x₁ + θ₂x₂ + ... + θ₀ = 0`

The job of our learning algorithm is to find the parameters `θ` that define the perfect decision boundary to separate our data.

## 3. A New Way to Measure Error: Log Loss

We can't use the Mean Squared Error (MSE) cost function from Linear Regression. If we plug our new sigmoid hypothesis into the MSE formula, we get a wavy, non-convex function with many local minima. Gradient Descent would likely get stuck and not find the best solution.

We need a cost function that is convex and correctly penalizes the model for being confidently wrong. This function is called **Log Loss**, or **Binary Cross-Entropy**.

**Cost for a single example:**
`Cost(h(x), y) = -y * log(h(x)) - (1 - y) * log(1 - h(x))`

Let's build intuition for this:

- **Case 1: The actual class is `y = 1`**
  - The formula simplifies to `Cost = -log(h(x))`.
  - If our prediction `h(x)` is close to 1 (correct), `log(h(x))` is close to 0, so the cost is low.
  - If our prediction `h(x)` is close to 0 (incorrect), `log(h(x))` approaches negative infinity, so the cost becomes very high.

- **Case 2: The actual class is `y = 0`**
  - The formula simplifies to `Cost = -log(1 - h(x))`.
  - If our prediction `h(x)` is close to 0 (correct), `1 - h(x)` is close to 1, `log(1 - h(x))` is close to 0, and the cost is low.
  - If our prediction `h(x)` is close to 1 (incorrect), `1 - h(x)` is close to 0, `log(1 - h(x))` approaches negative infinity, and the cost becomes very high.

This is exactly what we want! The cost function heavily penalizes predictions that are both confident and wrong. The total cost `J(θ)` is just the average of this cost over all training examples.

## 4. Optimization with Gradient Descent

Now that we have a new cost function, how do we minimize it? The same way as before: **Gradient Descent**!

The overall process is identical:
1.  Initialize random values for the parameters `θ`.
2.  Calculate the gradient of the Log Loss cost function.
3.  Update the parameters using the rule: `θⱼ := θⱼ - α * (∂J / ∂θⱼ)`.
4.  Repeat for a number of iterations.

The flavors of Gradient Descent (Batch, Mini-Batch, SGD) all apply here just as they did for Linear Regression. The only thing that changes is the formula for the gradient itself, which is derived from the Log Loss function.

## 5. Practical Example with Scikit-Learn

Let's use Scikit-Learn to build a classifier to detect whether a flower is of the *Iris-Virginica* species based on its petal width.

```python
import numpy as np
from sklearn import datasets
from sklearn.linear_model import LogisticRegression
import matplotlib.pyplot as plt

# Load the Iris dataset
iris = datasets.load_iris()
X = iris["data"][:, 3:]  # Petal width
y = (iris["target"] == 2).astype(int)  # 1 if Iris-Virginica, else 0

# Train a logistic regression model
log_reg = LogisticRegression()
log_reg.fit(X, y)

# Let's see the model's predictions over a range of petal widths
X_new = np.linspace(0, 3, 1000).reshape(-1, 1)
y_proba = log_reg.predict_proba(X_new)

# The decision boundary is where probability is 0.5
decision_boundary = X_new[y_proba[:, 1] >= 0.5][0]

plt.figure(figsize=(8, 4))
plt.plot(X[y==0], y[y==0], "bs")
plt.plot(X[y==1], y[y==1], "g^")
plt.plot(X_new, y_proba[:, 1], "g-", label="Iris-Virginica probability")
plt.plot(X_new, y_proba[:, 0], "b--", label="Not Iris-Virginica probability")
plt.plot([decision_boundary, decision_boundary], [-0.02, 1.02], "k:", linewidth=2)
plt.xlabel("Petal width (cm)")
plt.ylabel("Probability")
plt.legend()
plt.show()

print(f"Decision Boundary at: {decision_boundary[0]:.2f} cm")
# The model predicts a flower is Iris-Virginica if its petal width is > 1.66 cm
```

The code above trains a model and then visualizes the S-shaped probability curve and the decision boundary it learned.

## Conclusion

Logistic Regression is the go-to algorithm for binary classification tasks. It elegantly builds upon the concepts of Linear Regression by adding one key ingredient: the **Sigmoid function**.

- It takes a linear combination of inputs and squashes the result into a **probability** between 0 and 1.
- It uses a **Decision Boundary** to convert that probability into a final class prediction.
- It is trained using a **Log Loss** cost function and **Gradient Descent** to find the best parameters.

Mastering Logistic Regression is a critical step in your machine learning journey. It's not only powerful on its own but also serves as a fundamental building block for more complex models like neural networks.

{% include inarticle-adsense.html %}

## Suggested Reading

- **"Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow" by Aurélien Géron:** Chapter 4 also covers Logistic Regression in great detail.
- **StatQuest with Josh Starmer on YouTube:** His video on Logistic Regression is one of the best visual explanations available.
- **Andrew Ng's Machine Learning Course (Coursera):** Provides a deep and intuitive dive into the mathematics of Log Loss and classification.
