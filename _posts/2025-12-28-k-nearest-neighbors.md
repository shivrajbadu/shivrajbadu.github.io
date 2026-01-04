---
layout: post
title: "ML Foundations: K-Nearest Neighbors (KNN)"
date: 2025-12-28 19:30:00 +0545
categories: [Python, Machine Learning]
tags: [knn, k-nearest-neighbors, classification, machine-learning, python]
---

# ML Foundations: K-Nearest Neighbors (KNN)

## Introduction

Welcome back to our machine learning foundations series! Today, we're looking at one of the most intuitive and straightforward algorithms in the entire field: **K-Nearest Neighbors (KNN)**.

The core idea behind KNN is beautifully simple: **"An object is most likely to be like its neighbors."** To classify a new, unseen data point, the algorithm looks at the other data points that are closest to it in the feature space and takes a vote.

KNN is often called a **"lazy learner"** or an **"instance-based"** algorithm. This is because it doesn't learn a "model" in the traditional sense during a training phase. Instead, it simply memorizes the entire training dataset. The real work happens during prediction time.

In this guide, we'll explore:
1.  The step-by-step logic of the KNN algorithm.
2.  The critical choice of `K` and its effect on the model.
3.  How "distance" is measured using different metrics.
4.  The pros and cons of this simple yet powerful algorithm.

## 1. The KNN Algorithm: A Step-by-Step Guide

Imagine you have a dataset of dogs and cats, plotted based on their height and weight. Now, a new animal arrives, and you want to classify it. Here's how KNN would work:

1.  **Choose the number of neighbors (`K`).** Let's say we pick `K=5`.
2.  **Calculate Distances.** For the new animal, calculate its distance to *every single animal* in your dataset.
3.  **Find the Neighbors.** Identify the top `K` (in our case, 5) animals that are closest to the new one. These are its "nearest neighbors."
4.  **Vote!** Look at the labels of these 5 neighbors. If, for example, 4 of them are dogs and 1 is a cat, the algorithm takes a majority vote. The new animal is classified as a **dog**.

That's it! The same logic applies to regression problems, but instead of a vote, KNN would take the average of the values of its neighbors (e.g., to predict the price of a house based on its neighbors' prices).

## 2. The All-Important `K`: How Many Neighbors to Ask?

The choice of `K` has a dramatic effect on the model's performance and is a critical hyperparameter you need to tune.

- **A small `K` (e.g., `K=1`)** makes the model highly sensitive to noise. The decision boundary will be very complex and jagged, trying to perfectly classify every single point. This leads to **high variance** and **overfitting**. Your model learns the training data's noise instead of the underlying pattern.

- **A large `K`** makes the model's decision boundary much smoother and more robust to outliers. However, if `K` is too large, the model becomes too generalized. It might consider neighbors from other classes, leading to misclassifications. This leads to **high bias** and **underfitting**.

![KNN K Value](https://i.stack.imgur.com/4N7A3.png)
*Image credit: "An Introduction to Statistical Learning"*

**Rule of Thumb:**
- Start with a small, odd value for `K` (e.g., 3, 5, 7) to avoid ties in binary classification.
- Use cross-validation to test different values of `K` and find the one that produces the best-performing model on your data.

## 3. How Do We Measure "Distance"?

Since the entire algorithm is based on "closeness," the way we measure distance is fundamental.

**Crucial Prerequisite: Feature Scaling**
Before we even talk about metrics, it's vital to understand that **you must scale your features before using KNN**. If you have one feature that ranges from 0-1000 (like salary) and another that ranges from 0-10 (like years of experience), the salary feature will completely dominate the distance calculation. Your model will effectively ignore the years of experience. Use `StandardScaler` or `MinMaxScaler` from Scikit-Learn to bring all features to a comparable scale.

Here are the most common distance metrics:

1.  **Euclidean Distance (L2 Norm):** This is the most common metric and the default in Scikit-Learn. It's the straight-line distance between two points in space.
    `distance = sqrt(Σ(aᵢ - bᵢ)²)`

2.  **Manhattan Distance (L1 Norm):** Also known as "city block" distance. It's the sum of the absolute differences between the coordinates of two points. Imagine moving along a grid.
    `distance = Σ|aᵢ - bᵢ|`

3.  **Minkowski Distance:** This is the generalized form of the above two. It's Euclidean when the parameter `p=2` and Manhattan when `p=1`.

## 4. Pros and Cons of KNN

### Pros
- **Simple and Intuitive:** Easy to understand and explain.
- **No Training Phase:** As a "lazy learner," it's very fast to get started since it just stores the data.
- **Versatile:** Works for classification, regression, and naturally handles multi-class problems.
- **Non-linear:** Can learn complex, non-linear decision boundaries.

### Cons
- **Computationally Expensive:** The prediction step is slow for large datasets because it must compute the distance to every single training point.
- **High Memory Usage:** It needs to store the entire dataset in memory.
- **Curse of Dimensionality:** Performance degrades significantly as the number of features (dimensions) increases. In high-dimensional space, the concept of "distance" becomes less meaningful as points become sparsely distributed.
- **Sensitive to Irrelevant Features:** Features that are not useful for prediction will still be included in the distance calculation, adding noise to the model.

## 5. Practical Example with Scikit-Learn

Let's use KNN to classify different types of cancer based on medical measurements.

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

# 1. Load the data
cancer = load_breast_cancer()
X, y = cancer.data, cancer.target

# 2. Split the data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 3. Scale the features! This is mandatory for KNN.
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 4. Train a KNN model
knn = KNeighborsClassifier(n_neighbors=5) # Let's start with K=5
knn.fit(X_train_scaled, y_train)
y_pred = knn.predict(X_test_scaled)
print(f"Accuracy with K=5: {accuracy_score(y_test, y_pred):.3f}")

# 5. Find the optimal K
k_range = range(1, 31)
k_scores = []
for k in k_range:
    knn_cv = KNeighborsClassifier(n_neighbors=k)
    scores = cross_val_score(knn_cv, X_train_scaled, y_train, cv=10, scoring='accuracy')
    k_scores.append(scores.mean())

plt.plot(k_range, k_scores)
plt.xlabel('Value of K for KNN')
plt.ylabel('Cross-Validated Accuracy')
plt.show()

# Find the K with the highest score
best_k = k_range[np.argmax(k_scores)]
print(f"The optimal K is: {best_k}")
```
The plot from this code will help you visually identify the "elbow" or the `K` value that gives the highest cross-validated accuracy, providing a robust way to choose your hyperparameter.

## Conclusion

K-Nearest Neighbors is a fantastic algorithm to have in your toolkit. Its simplicity makes it a great baseline model to compare against more complex methods. While it may not be the top performer for large, high-dimensional datasets due to its computational cost, its intuitive, non-parametric nature makes it a valuable tool for many classification and regression problems.

Remember the key takeaways:
- KNN works by a majority vote of its `K` closest neighbors.
- The choice of `K` is a trade-off between bias and variance.
- **Feature scaling is not optional; it's a requirement.**

{% include inarticle-adsense.html %}

## Suggested Reading

- **"An Introduction to Statistical Learning" by James, Witten, Hastie, and Tibshirani:** Provides a very clear, foundational explanation of KNN.
- **Scikit-Learn Documentation:** The user guide on Nearest Neighbors is a great resource for practical implementation details.
- **"Hands-On Machine Learning with Scikit-Learn, Keras & TensorFlow" by Aurélien Géron:** Offers practical advice and context for using KNN in a larger ML workflow.
