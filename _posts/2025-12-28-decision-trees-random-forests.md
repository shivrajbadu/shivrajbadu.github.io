---
layout: post
title: "ML Foundations: Decision Trees and Random Forests"
date: 2025-12-28 16:30:00 +0545
categories: [Python, Machine Learning]
tags: [decision-trees, random-forests, ensemble-learning, machine-learning, python]
---

# ML Foundations: Decision Trees and Random Forests

## Introduction

So far in our series, we've explored models that are fundamentally mathematical, like Linear and Logistic Regression. Now, we're going to look at a family of models that are far more intuitive and work much like the human brain does: **Decision Trees**.

A Decision Tree is a simple, flowchart-like structure that asks a series of questions about your data to arrive at a decision. They are one of the most interpretable models in machine learning, often called "white-box" models because you can clearly see and understand the logic behind their predictions.

But the real power comes when we combine many Decision Trees into an **ensemble**. This leads us to **Random Forests**, one of the most popular and effective machine learning algorithms in use today.

In this guide, we will cover:
1.  The intuition behind Decision Trees and how they learn.
2.  The pros and cons of single Decision Trees (and their tendency to overfit).
3.  How **Random Forests** use "the wisdom of the crowd" to build a better model.
4.  A practical example using Scikit-Learn.

## 1. The Intuition of a Decision Tree

Imagine you're trying to decide if you should play tennis today. You might ask a series of questions:
- Is the weather outlook sunny, overcast, or rainy?
  - If it's rainy, you don't play.
  - If it's sunny, you might ask: Is the humidity high?
    - If yes, you don't play.
    - If no, you play.
  - If it's overcast, you play.

You've just built a Decision Tree!

![Decision Tree Example](https://upload.wikimedia.org/wikipedia/commons/e/eb/Decision_Tree.jpg)

A Decision Tree in machine learning works the same way. The algorithm learns a hierarchy of questions to ask about the features of your data to predict a target value.

- **Root Node:** The first question that splits the entire dataset.
- **Internal Nodes:** Subsequent questions that split the data further.
- **Leaf Nodes:** The final "decision" or prediction.

### How Does a Tree Learn?

The algorithm's goal is to find the best questions (splits) that create the "purest" possible leaf nodes. A pure node is one where all the data points belong to a single class (e.g., all "Play Tennis" or all "Don't Play Tennis").

To do this, the algorithm searches through every feature and every possible split point for that feature. It then picks the split that results in the greatest reduction in "impurity." The two most common measures of impurity are:

1.  **Gini Impurity:** A measure of how often a randomly chosen element from the set would be incorrectly labeled. A Gini score of 0 is perfect purity.
2.  **Entropy (Information Gain):** A measure of disorder or uncertainty. The algorithm seeks to maximize "information gain," which is equivalent to minimizing entropy.

The algorithm is greedy: it picks the best possible split at the current step and then repeats the process on the resulting subsets. This continues until it reaches a stopping condition, such as a maximum depth or a minimum number of samples per leaf.

## 2. The Pros and Cons of a Single Tree

**Pros:**
- **Highly Interpretable:** You can visualize the tree and explain its logic to non-technical stakeholders.
- **No Feature Scaling Needed:** The question-based nature of trees means they are not sensitive to the scale of features.
- **Handles Non-linear Relationships:** They can capture complex relationships that linear models cannot.

**Cons:**
- **Prone to Overfitting:** If you let a tree grow deep enough, it can create a specific rule for every single data point in the training set. It will have 100% accuracy on the training data but will fail to generalize to new, unseen data.
- **Instability:** Small changes in the training data can lead to a completely different tree structure.

The overfitting problem is the biggest weakness of single Decision Trees. This is where Random Forests come in.

## 3. The Wisdom of the Crowd: Random Forests

A Random Forest is an **ensemble learning** method. Instead of relying on a single, complex Decision Tree, it builds hundreds or thousands of simple, slightly different Decision Trees and then aggregates their predictions. For classification, it takes a majority vote; for regression, it takes the average.

This "wisdom of the crowd" approach is incredibly powerful and overcomes the overfitting problem of single trees.

But how does it ensure the trees are different from each other? It uses two key techniques:

1.  **Bagging (Bootstrap Aggregating):** Each tree in the forest is trained on a different random sample of the training data, drawn *with replacement*. This means some data points may be used multiple times in a single tree's training set, while others may not be used at all.
2.  **Feature Randomness:** When splitting a node, the algorithm doesn't search through *all* the features for the best split. Instead, it selects a *random subset of features* and only considers them for the split.

These two sources of randomness ensure that the individual trees are diverse. While each individual tree might be a weak learner and slightly wrong in its own way, the collective errors cancel each other out, leading to a much more robust and accurate final prediction.

## 4. Practical Example with Scikit-Learn

Let's use the well-known wine dataset to classify wine into one of three cultivars based on its chemical properties.

```python
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Load data
wine = load_wine()
X, y = wine.data, wine.target

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# --- 1. Train a single Decision Tree ---
# We'll limit its depth to prevent massive overfitting
single_tree = DecisionTreeClassifier(max_depth=4, random_state=42)
single_tree.fit(X_train, y_train)
y_pred_tree = single_tree.predict(X_test)
print(f"Single Decision Tree Accuracy: {accuracy_score(y_test, y_pred_tree):.3f}")

# --- 2. Train a Random Forest ---
# n_estimators is the number of trees in the forest
random_forest = RandomForestClassifier(n_estimators=100, random_state=42, n_jobs=-1)
random_forest.fit(X_train, y_train)
y_pred_forest = random_forest.predict(X_test)
print(f"Random Forest Accuracy: {accuracy_score(y_test, y_pred_forest):.3f}")
```
In most cases, you will find that the Random Forest model is more accurate and reliable than the single Decision Tree.

### Feature Importance
A great bonus of Random Forests is their ability to calculate the relative importance of each feature in making predictions.

```python
import pandas as pd
import matplotlib.pyplot as plt

importances = random_forest.feature_importances_
feature_names = wine.feature_names
forest_importances = pd.Series(importances, index=feature_names)

fig, ax = plt.subplots()
forest_importances.sort_values().plot.barh(ax=ax)
ax.set_title("Feature importances using Random Forest")
ax.set_ylabel("Features")
fig.tight_layout()
plt.show()
```
This helps you understand which features are most influential in your model.

## Conclusion

Decision Trees provide an intuitive and powerful way to model complex data. While single trees are prone to overfitting, they are the building blocks for one of the most successful machine learning algorithms: the **Random Forest**.

- **Decision Trees** learn a hierarchy of if/else questions to make predictions and are highly interpretable.
- **Random Forests** build an ensemble of diverse trees, using bagging and feature randomness to create a robust model that corrects for the errors of individual trees.
- This "wisdom of the crowd" approach makes Random Forests a go-to algorithm for many tabular data problems, delivering high accuracy without requiring extensive feature engineering or scaling.

{% include inarticle-adsense.html %}

## Suggested Reading

- **"An Introduction to Statistical Learning" by James, Witten, Hastie, and Tibshirani:** Chapter 8 provides a fantastic and accessible explanation of tree-based methods.
- **StatQuest with Josh Starmer on YouTube:** His videos on Decision Trees and Random Forests are legendary for their clarity.
- **Scikit-Learn Documentation:** The user guides for both Decision Trees and Random Forests are excellent practical resources.
