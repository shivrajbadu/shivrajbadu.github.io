---
layout: post
title: "ML Foundations: Unsupervised Learning and K-Means Clustering"
date: 2025-12-28 18:30:00 +0545
categories: [Python, Machine Learning]
tags: [unsupervised-learning, clustering, k-means, machine-learning, python]
---

# ML Foundations: Unsupervised Learning and K-Means Clustering

## Introduction

Throughout our machine learning series, we've focused on **Supervised Learning**. In supervised learning, our data is labeled: we have input features (`X`) and the correct output answers (`y`). Our goal is to train a model that can map from `X` to `y`.

But what if we don't have any labels? What if we're given a dataset and simply asked to "find something interesting" in it? This is the domain of **Unsupervised Learning**. The goal is not to predict a known outcome but to discover hidden patterns, structures, and groupings within the data itself.

The most common type of unsupervised learning is **clustering**, where we aim to group similar data points together. And the most famous clustering algorithm is **K-Means**.

In this final guide of our foundations series, we'll explore:
1.  The difference between Supervised and Unsupervised Learning.
2.  The intuition behind K-Means clustering.
3.  How the K-Means algorithm works step-by-step.
4.  The importance of choosing the right number of clusters (`K`) using the Elbow Method.
5.  A practical example with Scikit-Learn.

## 1. Supervised vs. Unsupervised Learning

|                    | **Supervised Learning**                               | **Unsupervised Learning**                             |
| ------------------ | ----------------------------------------------------- | ----------------------------------------------------- |
| **Goal**           | Predict a known outcome (label).                      | Discover hidden patterns or structures.               |
| **Input Data**     | Labeled data (features `X` and labels `y`).           | Unlabeled data (only features `X`).                   |
| **Example Tasks**  | Classification (spam/not spam), Regression (price).   | Clustering (customer segments), Anomaly Detection.    |
| **Example Algos**  | Linear Regression, SVMs, Random Forests.              | K-Means Clustering, Hierarchical Clustering, PCA.     |

Unsupervised learning is often used in exploratory data analysis to understand a dataset before you might attempt a supervised learning task. For example, you could use clustering to identify different types of customers (customer segmentation) and then build a separate supervised model to predict the purchasing behavior of each segment.

## 2. The Intuition of K-Means

Imagine you have a scatter plot of unlabeled data points. K-Means tries to find a user-defined number of clusters (`K`) in this data.

The core idea is simple: **A good cluster is one where the data points are packed closely together, and the clusters themselves are far apart.**

K-Means represents each cluster by its center point, called the **centroid**. The algorithm works by assigning each data point to its nearest centroid and then updating the centroid's position based on the points assigned to it. This process is repeated until the cluster assignments no longer change.

## 3. The K-Means Algorithm Step-by-Step

Let's say we want to find `K=3` clusters in our data.

1.  **Initialization:** Randomly place `K` centroids on the data plot. These are your initial guesses for the cluster centers.

2.  **Assignment Step:** For each data point, calculate its distance to every centroid. Assign the data point to the cluster of the **closest centroid**.

3.  **Update Step:** For each cluster, calculate the mean of all the data points assigned to it. Move the centroid of that cluster to this new mean position.

4.  **Repeat:** Repeat the **Assignment Step** and the **Update Step** until the centroids stop moving significantly. When this happens, the algorithm has **converged**, and the final cluster assignments represent the result.

This iterative process is guaranteed to converge, but it may find a local optimum, not the global best solution. The quality of the final result can depend heavily on the initial random placement of the centroids. To combat this, Scikit-Learn's implementation runs the algorithm multiple times with different random initializations (`n_init`) and chooses the best result.

## 4. How to Choose the Right `K`? The Elbow Method

The biggest challenge in K-Means is that you have to choose the number of clusters, `K`, beforehand. How do you know if you should look for 3 customer segments or 5?

One of the most common techniques is the **Elbow Method**.
1.  Run the K-Means algorithm for a range of different `K` values (e.g., from 1 to 10).
2.  For each `K`, calculate the **Within-Cluster Sum of Squares (WCSS)**. This is the sum of the squared distances between each data point and its assigned centroid. It's a measure of how compact the clusters are. In Scikit-Learn, this is available as the `inertia_` attribute after fitting the model.
3.  Plot the WCSS for each value of `K`.

The plot will typically look like an arm. As `K` increases, the WCSS will decrease because the clusters are getting smaller and more compact. The "elbow" of the arm—the point where the rate of decrease sharply slows down—is considered the optimal `K`. This is the point of diminishing returns, where adding another cluster doesn't significantly improve the compactness of the clusters.

![Elbow Method](https://www.oreilly.com/library/view/statistics-for-machine/9781788295758/assets/e7c2b2a8-353a-45b3-8037-55b483a73a12.png)

## 5. Practical Example with Scikit-Learn

Let's use K-Means to find clusters in a synthetic dataset.

```python
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans

# 1. Generate synthetic data
X, y = make_blobs(n_samples=300, centers=4, cluster_std=0.60, random_state=0)

# 2. Use the Elbow Method to find the optimal K
wcss = []
for i in range(1, 11):
    kmeans = KMeans(n_clusters=i, init='k-means++', max_iter=300, n_init=10, random_state=0)
    kmeans.fit(X)
    wcss.append(kmeans.inertia_)

plt.plot(range(1, 11), wcss)
plt.title('The Elbow Method')
plt.xlabel('Number of clusters')
plt.ylabel('WCSS')
plt.show()
# The elbow is clearly at K=4, as expected.

# 3. Apply K-Means with the optimal K
kmeans = KMeans(n_clusters=4, init='k-means++', max_iter=300, n_init=10, random_state=0)
y_kmeans = kmeans.fit_predict(X)

# 4. Visualize the clusters
plt.scatter(X[y_kmeans == 0, 0], X[y_kmeans == 0, 1], s=100, c='red', label='Cluster 1')
plt.scatter(X[y_kmeans == 1, 0], X[y_kmeans == 1, 1], s=100, c='blue', label='Cluster 2')
plt.scatter(X[y_kmeans == 2, 0], X[y_kmeans == 2, 1], s=100, c='green', label='Cluster 3')
plt.scatter(X[y_kmeans == 3, 0], X[y_kmeans == 3, 1], s=100, c='cyan', label='Cluster 4')

# Plot the centroids
plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], s=300, c='yellow', label='Centroids')
plt.title('Clusters of data')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.show()
```

## Conclusion

Unsupervised Learning opens up a new dimension of machine learning where we can find insights without being guided by pre-existing labels. **K-Means Clustering** is a simple yet powerful algorithm for partitioning data into distinct groups.

- **Unsupervised Learning** finds patterns in unlabeled data.
- **K-Means** aims to create compact clusters by assigning data points to the nearest **centroid** and then updating the centroid's position.
- The **Elbow Method** provides a heuristic for finding the optimal number of clusters, `K`.
- **Important Note:** Since K-Means is based on distance, it's crucial to **scale your features** (e.g., using `StandardScaler`) before applying the algorithm, just as we discussed in our Data Preprocessing post.

This concludes our foundational series! We've journeyed from predicting values with Linear Regression to classifying them with Logistic Regression and SVMs, building robust models with Random Forests, evaluating them properly, and now, discovering hidden patterns with K-Means. With these fundamentals, you are well-equipped to tackle a wide range of machine learning problems.

{% include inarticle-adsense.html %}

## Suggested Reading

- **"Python Data Science Handbook" by Jake VanderPlas:** Has an excellent chapter on K-Means and clustering in general.
- **Scikit-Learn Documentation:** The user guide on clustering provides a deep dive into K-Means and other clustering algorithms.
- **StatQuest with Josh Starmer on YouTube:** His video on K-Means is a fantastic and easy-to-follow explanation of the algorithm.
