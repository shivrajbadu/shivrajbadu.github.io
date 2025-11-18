---
layout: post
title: "A Quick Note on Python Tuples"
date: 2025-10-26 20:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

A tuple is a collection of items which is ordered and **unchangeable**. In Python, tuples are written with round brackets.

### Creating Tuples

Creating a tuple is similar to creating a list, but with round brackets.

```python
# A tuple of strings
fruits = ("apple", "banana", "cherry")

# A tuple of numbers
numbers = (1, 2, 3, 4, 5)

# A tuple with mixed data types
mixed_tuple = ("apple", 3, True)

# A tuple with one item (note the trailing comma)
my_tuple = ("apple",)
```

### Accessing Items

You can access the items of a tuple by referring to the index number, just like with lists.

```python
fruits = ("apple", "banana", "cherry")

print(fruits[0])  # 'apple'
print(fruits[1])  # 'banana'
```

### Immutability

The key difference between tuples and lists is that tuples are immutable, meaning you cannot change, add, or remove items after the tuple has been created.

```python
fruits = ("apple", "banana", "cherry")

# This will raise a TypeError
# fruits[0] = "grape"
```

### Tuple Methods

Tuples have only two built-in methods:

- `count()`: Returns the number of times a specified value occurs in a tuple.
- `index()`: Searches the tuple for a specified value and returns the position of where it was found.

```python
fruits = ("apple", "banana", "cherry", "apple")

print(fruits.count("apple"))  # 2
print(fruits.index("banana")) # 1
```

### When to Use Tuples

Since tuples are immutable, they are often used for data that should not be changed, such as a collection of constants. They are also used as keys in dictionaries, as their immutability makes them hashable.

### Conclusion

Tuples are a useful data structure in Python for storing collections of items that you don't want to change. Their immutability provides a level of safety and can lead to performance optimizations. While lists are more flexible, tuples have their own important place in Python programming.
