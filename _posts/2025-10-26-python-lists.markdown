---
layout: post
title: "A Quick Note on Python Lists"
date: 2025-10-26 19:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

Lists are one of the most versatile and commonly used data structures in Python. A list is a collection of items which are ordered and changeable. In Python, lists are written with square brackets.

### Creating Lists

Creating a list is as simple as placing different comma-separated values between square brackets.

```python
# A list of strings
fruits = ["apple", "banana", "cherry"]

# A list of numbers
numbers = [1, 2, 3, 4, 5]

# A list with mixed data types
mixed_list = ["apple", 3, True]
```

### Accessing Items

You can access the items of a list by referring to the index number.

```python
fruits = ["apple", "banana", "cherry"]

print(fruits[0])  # 'apple'
print(fruits[1])  # 'banana'
```

### List Methods

Python has a large set of built-in methods that you can use on lists.

- `append()`: Adds an element at the end of the list.
- `insert()`: Adds an element at the specified position.
- `remove()`: Removes the specified item.
- `pop()`: Removes the element at the specified position.
- `sort()`: Sorts the list.

```python
fruits = ["apple", "banana", "cherry"]

fruits.append("orange")
print(fruits)  # ['apple', 'banana', 'cherry', 'orange']

fruits.insert(1, "grape")
print(fruits)  # ['apple', 'grape', 'banana', 'cherry', 'orange']

fruits.remove("banana")
print(fruits)  # ['apple', 'grape', 'cherry', 'orange']
```

### List Slicing

You can get a range of items in a list using slicing.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Get the items from index 2 to 5 (not included)
print(numbers[2:5])  # [3, 4, 5]

# Get the items from the beginning to index 4 (not included)
print(numbers[:4])  # [1, 2, 3, 4]

# Get the items from index 5 to the end
print(numbers[5:])  # [6, 7, 8, 9, 10]
```

### Conclusion

Lists are a fundamental data structure in Python, and their flexibility makes them incredibly useful in a wide range of applications. Whether you're storing a collection of items, managing a queue, or just grouping related data, lists are an essential tool in any Python programmer's toolkit.
