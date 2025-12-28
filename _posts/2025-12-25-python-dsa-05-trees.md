---
layout: post
title: "Python DSA Series 05: Introduction to Trees"
date: 2025-12-25 14:15:00 +0545
categories: [Python, DSA]
tags: [python, dsa, trees, binary-search-trees, data-structures, algorithms]
---

# Python DSA Series 05: Introduction to Trees

We've covered linear data structures where elements follow in a sequence. Now, we venture into the world of **hierarchical data structures** with Trees. Trees are incredibly important and are used to represent everything from your computer's file system to the structure of a company's hierarchy.

## What is a Tree?

A tree is a non-linear data structure that consists of **nodes** connected by **edges**. It simulates a hierarchical structure. Unlike natural trees, a computer science tree is typically drawn with the root at the top.

### Core Terminology

-   **Node:** The fundamental part of a tree that holds data.
-   **Root:** The topmost node in the tree. A tree has only one root.
-   **Edge:** The link between two nodes.
-   **Parent:** A node that has an edge to a child node.
-   **Child:** A node that has an edge from a parent node.
-   **Sibling:** Nodes that share the same parent.
-   **Leaf:** A node with no children.
-   **Height of a Tree:** The length of the longest path from the root to a leaf.
-   **Depth of a Node:** The length of the path from the root to that node.

![Tree Terminology Diagram](https://i.imgur.com/hV9pA2a.png)

## Binary Trees

The most common type of tree is a **Binary Tree**. In a binary tree, each node can have at most **two** children: a `left` child and a `right` child.

### Python Implementation of a Node

```python
class TreeNode:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None
```

## Tree Traversal Algorithms

"Traversing" a tree means visiting every node exactly once. This is a critical concept and a frequent source of interview questions. There are two main approaches: Depth-First Search (DFS) and Breadth-First Search (BFS).

### 1. Depth-First Search (DFS)

DFS explores as far as possible down one branch before backtracking. It's often implemented using recursion, which naturally uses a stack (the call stack).

There are three main types of DFS traversal:

**a) In-order Traversal (Left, Root, Right)**
Visits the left subtree, then the root node, then the right subtree. For a Binary Search Tree, this traversal visits the nodes in ascending order.

```python
def inorder_traversal(root):
    if root:
        inorder_traversal(root.left)
        print(root.key, end=" ")
        inorder_traversal(root.right)
```

**b) Pre-order Traversal (Root, Left, Right)**
Visits the root node first, then the left subtree, then the right subtree. Useful for creating a copy of a tree.

```python
def preorder_traversal(root):
    if root:
        print(root.key, end=" ")
        preorder_traversal(root.left)
        preorder_traversal(root.right)
```

**c) Post-order Traversal (Left, Right, Root)**
Visits the left subtree, then the right subtree, and finally the root node. Useful for deleting nodes from a tree (you delete children before the parent).

```python
def postorder_traversal(root):
    if root:
        postorder_traversal(root.left)
        postorder_traversal(root.right)
        print(root.key, end=" ")
```

### 2. Breadth-First Search (BFS)

BFS explores the tree level by level. It visits all the nodes at a certain depth before moving on to the next level. BFS is implemented using a **Queue**.

```python
from collections import deque

def bfs_traversal(root):
    if not root:
        return
    
    queue = deque([root])
    
    while queue:
        node = queue.popleft()
        print(node.key, end=" ")
        
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
```

## Binary Search Trees (BST)

A Binary Search Tree is a special type of binary tree that imposes a crucial ordering property:
> For any given node `N`, all values in its **left subtree** are less than `N.key`, and all values in its **right subtree** are greater than `N.key`.

This property makes searching for elements incredibly efficient.

### Time Complexity of BST Operations

| Operation | Average Case | Worst Case |
| :--- | :--- | :--- |
| **Search** | **O(log n)** | O(n) |
| **Insert** | **O(log n)** | O(n) |
| **Delete** | **O(log n)** | O(n) |

The average case is O(log n) because with each comparison, you eliminate about half of the remaining nodes. The worst case of O(n) occurs when the tree is **unbalanced** (e.g., a sorted list is inserted), making it degenerate into a linked list.

### Python Implementation of BST Operations

```python
# Insert a key in a BST
def insert(root, key):
    if root is None:
        return TreeNode(key)
    else:
        if root.key == key:
            return root # Or handle duplicates as needed
        elif root.key < key:
            root.right = insert(root.right, key)
        else:
            root.left = insert(root.left, key)
    return root

# Search for a key in a BST
def search(root, key):
    if root is None or root.key == key:
        return root
    
    if root.key < key:
        return search(root.right, key)
    
    return search(root.left, key)
```

## Conclusion

Trees are the go-to structure for representing any kind of hierarchical data.
-   **Binary Trees** are the most common form.
-   **Tree Traversal** (DFS and BFS) are fundamental algorithms for processing trees.
-   **Binary Search Trees (BSTs)** are an enhancement that provides O(log n) search, insert, and delete operations, making them highly efficient for dynamic, ordered data.

The key weakness of a simple BST is the danger of it becoming unbalanced, which degrades performance to O(n). Advanced trees like AVL Trees and Red-Black Trees solve this by automatically rebalancing themselves, but the core principles of the BST remain.

Next, we'll look at an even more flexible data structure: **Graphs**, which can represent complex networks like social connections or road maps.

{% include inarticle-adsense.html %}

## Suggested Reading
- [Tree Traversal (Wikipedia)](https://en.wikipedia.org/wiki/Tree_traversal)
- [Binary Search Tree (GeeksforGeeks)](https://www.geeksforgeeks.org/binary-search-tree-data-structure/)
- [Introduction to Algorithms (CLRS)](https://www.amazon.com/Introduction-Algorithms-3rd-MIT-Press/dp/0262033844) - The definitive guide to algorithms and data structures.
