--- 
layout: post
title: "ML Foundations: A Deeper Dive into Decision Trees"
date: 2025-12-28 21:30:00 +0545
categories: [Python, Machine Learning]
tags: [decision-trees, entropy, information-gain, overfitting, machine-learning]
---

# ML Foundations: A Deeper Dive into Decision Trees

## Introduction

In a previous post, we introduced Decision Trees and Random Forests, highlighting their intuitive, flowchart-like structure. We learned that trees make decisions by asking a series of questions and that Random Forests build a powerful model by averaging the predictions of many individual trees.

But this leaves us with two critical, "under the hood" questions:
1.  How does a tree *quantitatively decide* which feature to split on and where to split it? What makes one question "better" than another?
2.  If a single tree is prone to overfitting, what are the specific mechanisms we can use to control its growth and make it more general?

This guide will take a deeper dive into the mechanics of Decision Trees, focusing on the mathematics of splitting and the art of pruning.

We will explore:
1.  **Entropy:** A measure of impurity or uncertainty in a group of data points.
2.  **Information Gain:** The metric used to pick the best possible split.
3.  **Overfitting:** Why it happens in trees.
4.  **Pruning:** The techniques used to control overfitting and create a more robust model.

## 1. How a Tree Chooses the Best Split

Recall that the goal of a Decision Tree is to create splits that result in the "purest" possible child nodes—nodes where the data points ideally all belong to a single class.

To do this, the algorithm needs a mathematical way to measure the "impurity" of a node. If it can measure impurity, it can then calculate how much a particular split *reduces* that impurity. This reduction is what we're trying to maximize.

The most common metric for measuring impurity is **Entropy**.

## 2. Measuring Impurity: Entropy

In information theory, **Entropy** is a measure of the uncertainty or randomness in a system.

- A dataset that is perfectly "pure" (all data points belong to the same class) has **zero entropy**. There is no uncertainty.
- A dataset that is evenly split between classes has the **maximum possible entropy**. There is maximum uncertainty.

The formula for Entropy is:

`Entropy(S) = - Σ pᵢ * log₂(pᵢ)`

Where:
- `S` is the set of data points in a node.
- `pᵢ` is the proportion (or probability) of data points belonging to class `i` in that node.

Let's build some intuition. Consider a node with 10 data points:
- **Case 1 (Pure Node):** 10 points are "Class A".
  - `p_A = 10/10 = 1.0`. `p_B = 0`.
  - `Entropy = - (1.0 * log₂(1.0)) = 0`. Perfect purity, zero uncertainty.
- **Case 2 (Impure Node):** 5 points are "Class A", 5 are "Class B".
  - `p_A = 5/10 = 0.5`. `p_B = 5/10 = 0.5`.
  - `Entropy = - (0.5 * log₂(0.5) + 0.5 * log₂(0.5)) = 1.0`. Maximum impurity and uncertainty.

## 3. The Splitting Criterion: Information Gain

Now that we can measure entropy, we can define our splitting criterion: **Information Gain**.

**Information Gain** is simply the reduction in entropy achieved by splitting a dataset on a particular attribute. The algorithm calculates the Information Gain for every possible split and chooses the split that yields the highest value.

The formula is:

`InformationGain(S, A) = Entropy(S) - Σ (|Sᵥ| / |S|) * Entropy(Sᵥ)`

Where:
- `Entropy(S)` is the entropy of the parent node (before the split).
- The second term is the **weighted average entropy** of the child nodes.
  - `|Sᵥ| / |S|` is the proportion of data points that go into child node `v`.
  - `Entropy(Sv)` is the entropy of that child node.

In simple terms: **Information Gain = Entropy(parent) - Weighted Average Entropy(children)**.

The algorithm is greedy: at each node, it picks the split with the highest immediate Information Gain and repeats the process.

***A Note on Gini Impurity:*** Scikit-Learn uses a slightly different metric called **Gini Impurity** by default. It is computationally faster than Entropy because it doesn't involve a logarithm calculation. In practice, both metrics usually produce very similar trees.

## 4. The Enemy: Overfitting

If we let a Decision Tree grow without any constraints, it will continue to split until every leaf node is 100% pure. This means it will create a specific path or rule for every single data point in the training set.

The result is an overly complex tree that has perfectly memorized the training data, including its noise and outliers. This model will have 100% accuracy on the data it was trained on, but it will fail miserably when it encounters new, unseen data because it hasn't learned the general underlying pattern. This is **overfitting**.

![Overfitting Tree](https://d33wubrfki0l68.cloudfront.net/622152b634a1c123a48581f554504333395a137d/19269/static/c07f3a2c35529855529d7496518f3d16/06448/overfitting.png)
*A simple, generalized boundary (left) vs. a complex, overfit boundary (right).*

## 5. Controlling Overfitting: Pruning

To combat overfitting, we must constrain the tree's growth. This process is called **pruning**. There are two main approaches:

### Pre-Pruning (Early Stopping)
This is the most common approach. We stop the tree from growing too deep by setting limits *before* training. Scikit-Learn provides several hyperparameters for this:

- `max_depth`: The maximum allowed depth of the tree. This is one of the most effective ways to prevent overfitting. A smaller `max_depth` leads to a more general model.
- `min_samples_split`: The minimum number of data points a node must contain before it is allowed to be split. A higher value prevents the model from learning from very small, specific groups of data.
- `min_samples_leaf`: The minimum number of data points that must exist in a leaf node after a split. This ensures that every final decision is based on a reasonably sized group of samples.
- `max_leaf_nodes`: Limits the total number of terminal nodes in the tree.

Let's see how `max_depth` affects the model.

```python
from sklearn.datasets import make_moons
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score

X, y = make_moons(n_samples=500, noise=0.3, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)

# An unconstrained, overfit tree
tree_overfit = DecisionTreeClassifier(random_state=42)
tree_overfit.fit(X_train, y_train)
print(f"Overfit Tree Train Accuracy: {accuracy_score(y_train, tree_overfit.predict(X_train)):.3f}")
print(f"Overfit Tree Test Accuracy: {accuracy_score(y_test, tree_overfit.predict(X_test)):.3f}")
# Train accuracy will be very high, test accuracy will be lower.

# A constrained, pruned tree
tree_pruned = DecisionTreeClassifier(max_depth=4, random_state=42)
tree_pruned.fit(X_train, y_train)
print(f"\nPruned Tree Train Accuracy: {accuracy_score(y_train, tree_pruned.predict(X_train)):.3f}")
print(f"Pruned Tree Test Accuracy: {accuracy_score(y_test, tree_pruned.predict(X_test)):.3f}")
# Train and test accuracies will be much closer, indicating better generalization.
```

### Post-Pruning
This technique involves growing the full, overfit tree first and then removing (pruning) branches that are not statistically significant. While powerful, this is less common in Scikit-Learn's `DecisionTreeClassifier` and is more complex to implement.

## Conclusion

Understanding these "under the hood" mechanics is key to mastering tree-based models.

- Decision Trees build themselves by greedily choosing splits that provide the highest **Information Gain**—the biggest reduction in **Entropy**.
- The primary weakness of a single tree is its tendency to **overfit** by memorizing the training data.
- We combat this by **pruning** the tree, using hyperparameters like `max_depth` and `min_samples_leaf` to constrain its growth and force it to learn more general patterns.

This brings us full circle to **Random Forests**. A Random Forest works so well because it is an ensemble of many deep, unpruned (or lightly pruned) Decision Trees. Each tree is overfit to a different random subset of the data. By averaging their predictions, the individual errors and overfitting tendencies cancel each other out, resulting in a powerful, robust, and highly accurate model.

{% include inarticle-adsense.html %}

## Suggested Reading

- **"Elements of Statistical Learning" by Hastie, Tibshirani, and Friedman:** For a deep, mathematical treatment of these topics.
- **"Machine Learning" by Tom M. Mitchell:** A classic textbook that provides a foundational chapter on Decision Tree learning, including the ID3 algorithm.
- **StatQuest with Josh Starmer on YouTube:** His videos on Entropy and Gini Impurity are exceptionally clear visual explanations.
