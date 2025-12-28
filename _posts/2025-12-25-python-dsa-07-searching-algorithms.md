---
layout: post
title: "Python DSA Series 07: Essential Searching Algorithms"
date: 2025-12-25 14:25:00 +0545
categories: [Python, DSA]
tags: [python, dsa, algorithms, searching, binary-search, linear-search]
---

# Python DSA Series 07: Essential Searching Algorithms

We've explored many ways to *store* data. Now, let's focus on two fundamental algorithms for *finding* it. Searching is a core task in programming, and understanding the trade-offs between different search algorithms is crucial for writing efficient code.

In this post, we'll cover the two most essential searching algorithms: Linear Search and Binary Search.

## 1. Linear Search: The Simple Scan

Linear search is the most straightforward search algorithm. It sequentially checks each element of a collection until a match is found or the whole collection has been searched.

-   **Concept:** Start at the beginning and go one by one.
-   **When to use:** When the data is **unsorted**, or when the collection is very small.

### Python Implementation

```python
def linear_search(data, target):
    """
    Searches for a target in a list using linear search.
    Returns the index of the target if found, otherwise returns -1.
    """
    for i in range(len(data)):
        if data[i] == target:
            return i
    return -1

# Example
my_list = [4, 2, 7, 1, 9, 5]
print(f"Element 7 found at index: {linear_search(my_list, 7)}") # Output: 2
print(f"Element 6 found at index: {linear_search(my_list, 6)}") # Output: -1
```

### Time Complexity

-   **Best Case:** O(1) - The element is the first one in the list.
-   **Worst Case:** **O(n)** - The element is the last one, or not in the list at all.
-   **Average Case:** O(n)

Linear search is simple but can be very slow for large datasets.

---

## 2. Binary Search: The Power of Divide and Conquer

Binary search is a much faster and more powerful algorithm, but it comes with one critical prerequisite: **the data must be sorted.**

It works by repeatedly dividing the search interval in half.

-   **Concept:** Compare the target with the middle element. If they don't match, the half in which the target cannot lie is eliminated, and the search continues on the remaining half.

### Algorithm Steps

1.  Find the middle element of the sorted collection.
2.  If the middle element is the target, the search is over.
3.  If the target is **less than** the middle element, repeat the search on the **left half**.
4.  If the target is **greater than** the middle element, repeat the search on the **right half**.
5.  Continue until the element is found or the interval is empty.

### Python Implementation

```python
def binary_search(sorted_data, target):
    """
    Searches for a target in a sorted list using binary search.
    Returns the index of the target if found, otherwise returns -1.
    """
    left, right = 0, len(sorted_data) - 1

    while left <= right:
        mid = (left + right) // 2 # Find the middle index

        if sorted_data[mid] == target:
            return mid
        elif sorted_data[mid] < target:
            left = mid + 1 # Search the right half
        else:
            right = mid - 1 # Search the left half
            
    return -1

# Example
my_sorted_list = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
print(f"Element 23 found at index: {binary_search(my_sorted_list, 23)}") # Output: 5
print(f"Element 15 found at index: {binary_search(my_sorted_list, 15)}") # Output: -1
```

### Time Complexity

-   **Best Case:** O(1) - The element is the middle one.
-   **Worst Case:** **O(log n)** - The search continues until the interval is of size 1.
-   **Average Case:** O(log n)

The O(log n) complexity is what makes binary search so powerful. For a list of 1 million items, linear search might take 1 million comparisons, while binary search would take only about 20!

## Comparison: Linear vs. Binary Search

| Feature | Linear Search | Binary Search |
| :--- | :--- | :--- |
| **Time Complexity** | O(n) | **O(log n)** |
| **Prerequisite** | None | **Data must be sorted** |
| **Implementation** | Simple | Moderately complex |
| **Use Case** | Unsorted or small lists | Large, sorted lists |

## Python's `bisect` Module

Python's standard library has a built-in module for working with sorted lists called `bisect`. It provides functions to perform binary searches, which is often safer and faster than writing your own.

```python
import bisect

my_sorted_list = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]

# bisect_left finds the insertion point for 23, which is its index
index = bisect.bisect_left(my_sorted_list, 23)

# Check if the element is actually at that index
if index < len(my_sorted_list) and my_sorted_list[index] == 23:
    print(f"Element 23 found at index: {index}") # Output: 5
else:
    print("Element not found")
```

## Conclusion

Choosing the right search algorithm is a classic example of a space-time trade-off.
-   **Linear Search** is simple and works on any data, but it's slow.
-   **Binary Search** is exponentially faster but requires the overhead of sorting the data first.

If you're searching the same collection multiple times, the initial cost of sorting is often well worth the O(log n) search speed you gain afterward.

This leads us perfectly into our final topic: **Sorting Algorithms**. How do we get our data into the sorted order required for binary search? We'll find out in the next post.

{% include inarticle-adsense.html %}

## Suggested Reading
- [Binary Search (Wikipedia)](https://en.wikipedia.org/wiki/Binary_search)
- [Python's bisect Module Documentation](https://docs.python.org/3/library/bisect.html)
- [Searching Algorithms in Python (Real Python)](https://realpython.com/python-search-algorithms/)
