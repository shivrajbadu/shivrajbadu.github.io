---
layout: post
title: "Python DSA Series 06: An Introduction to Graphs"
date: 2025-12-25 14:20:00 +0545
categories: [Python, DSA]
tags: [python, dsa, graphs, data-structures, algorithms, bfs, dfs]
---

# Python DSA Series 06: An Introduction to Graphs

We've climbed the hierarchy of Trees, but what if you need to represent a structure with more complex connections, like a social network, a city road map, or the internet itself? For this, we need **Graphs**, one of the most flexible and powerful data structures in computer science.

## What is a Graph?

A graph is a data structure consisting of a set of **vertices** (or nodes) and a set of **edges** that connect these vertices. Unlike trees, graphs do not have a root node or a strict parent-child hierarchy. Any node can be connected to any other node.

### Core Terminology

-   **Vertex (or Node):** The fundamental entity in a graph.
-   **Edge (or Link):** The connection between two vertices.
-   **Undirected Graph:** Edges have no direction. A connection between A and B is the same as a connection between B and A (e.g., a Facebook friendship).
-   **Directed Graph (Digraph):** Edges have a direction. A connection from A to B does not imply a connection from B to A (e.g., following someone on Twitter).
-   **Unweighted Graph:** Edges have no associated value or cost.
-   **Weighted Graph:** Each edge has a numerical weight, representing a cost, distance, or capacity (e.g., the distance between two cities on a map).

![Graph Types Diagram](https://i.imgur.com/3m0YV8s.png)

## How to Represent a Graph

Before you can work with a graph, you need a way to store it in memory. There are two primary methods:

### 1. Adjacency List

This is the most common way to represent a graph. You use a dictionary (hash map) where each key is a vertex, and the value is a list of all vertices it's connected to.

**Python Example (for the undirected graph above):**
```python
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D', 'E'],
    'C': ['A', 'F'],
    'D': ['B'],
    'E': ['B', 'F'],
    'F': ['C', 'E']
}
```
-   **Pros:** Space-efficient for **sparse graphs** (graphs with relatively few edges). Iterating over all neighbors of a vertex is very efficient.
-   **Cons:** Checking for the existence of a specific edge between two vertices `(u, v)` is slightly slower (O(k), where k is the number of neighbors of `u`).

### 2. Adjacency Matrix

An adjacency matrix is a 2D array (a list of lists in Python) of size V x V, where V is the number of vertices. `matrix[i][j] = 1` if there is an edge from vertex `i` to `j`, and `0` otherwise. For weighted graphs, the entry would be the weight of the edge.

-   **Pros:** Checking for a specific edge `(u, v)` is an O(1) operation.
-   **Cons:** Consumes a lot of space (O(V²)), which is inefficient for sparse graphs.

For most problems, especially in interviews, the **adjacency list** is the preferred representation.

## Graph Traversal Algorithms

Just like with trees, we need ways to visit all the nodes in a graph.

### 1. Depth-First Search (DFS)

DFS for a graph works similarly to DFS for a tree. It explores as far as possible down one path before backtracking. The key difference is that you must keep track of **visited nodes** to avoid getting stuck in infinite loops in graphs with cycles.

**Python Implementation (using recursion):**
```python
def dfs(graph, start_node, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(start_node)
    print(start_node, end=" ")

    for neighbor in graph[start_node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)

# Example Usage:
# dfs(graph, 'A')
# Possible Output: A B D E F C 
```

### 2. Breadth-First Search (BFS)

BFS explores the graph level by level. It's excellent for finding the shortest path in an unweighted graph. It uses a **Queue** to keep track of nodes to visit.

**Python Implementation (using a queue):**
```python
from collections import deque

def bfs(graph, start_node):
    visited = {start_node}
    queue = deque([start_node])

    while queue:
        vertex = queue.popleft()
        print(vertex, end=" ")

        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

# Example Usage:
# bfs(graph, 'A')
# Output: A B C D E F
```

### Shortest Path with BFS

A fantastic property of BFS is that the first time it reaches a target node, it has found one of the shortest paths from the start node. This is because it explores nodes at distance 1, then distance 2, and so on. You can augment the BFS algorithm to keep track of path distances.

## Conclusion

Graphs are the ultimate tool for modeling networks. They are more flexible than trees but also introduce new challenges, like cycles.

-   **Adjacency Lists** are the standard way to represent graphs unless they are very dense.
-   **DFS** is great for exploring a graph's structure, finding paths, or detecting cycles.
-   **BFS** is the go-to algorithm for finding the shortest path in an unweighted graph.

Understanding how to represent and traverse graphs is fundamental to solving a wide range of problems, from network routing to puzzle solving.

In our final posts, we'll shift our focus from data structures to pure **Algorithms**, covering the essential searching and sorting techniques that every developer should know.

{% include inarticle-adsense.html %}

## Suggested Reading
- [Graphs in Python (Real Python)](https://www.realpython.com/python-graph-data-structures/)
- [Breadth-First Search and Depth-First Search (GeeksforGeeks)](https://www.geeksforgeeks.org/breadth-first-search-or-bfs-for-a-graph/)
- [Graph Algorithms Course (freeCodeCamp)](https://www.youtube.com/watch?v=09_LlHjoEiY)
