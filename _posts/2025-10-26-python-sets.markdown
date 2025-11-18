---
layout: post
title: "A Quick Note on Python Sets"
date: 2025-10-26 21:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

A set is a collection of items which is both **unordered** and **unindexed**. In Python, sets are written with curly brackets.

### Creating Sets

Creating a set is as simple as placing different comma-separated values between curly brackets. Sets do not allow duplicate items.

```python
# A set of strings
fruits = {"apple", "banana", "cherry"}

# A set with duplicate items (duplicates will be removed)
numbers = {1, 2, 3, 2, 1}
print(numbers)  # {1, 2, 3}
```

### Accessing Items

Since sets are unordered, you cannot access items by referring to an index. However, you can loop through the set items using a `for` loop, or ask if a specified value is present in a set, by using the `in` keyword.

```python
fruits = {"apple", "banana", "cherry"}

for fruit in fruits:
    print(fruit)

print("banana" in fruits)  # True
```

### Set Methods

Python has a large set of built-in methods that you can use on sets.

- `add()`: Adds an element to the set.
- `update()`: Add multiple items to a set.
- `remove()`: Removes the specified element.
- `union()`: Returns a new set containing all items from both sets.
- `intersection()`: Returns a new set, that only contains the items that are present in both sets.

```python
fruits = {"apple", "banana", "cherry"}

fruits.add("orange")
print(fruits)  # {'apple', 'orange', 'banana', 'cherry'}

fruits.update(["grape", "mango"])
print(fruits)  # {'mango', 'apple', 'orange', 'grape', 'banana', 'cherry'}

fruits.remove("banana")
print(fruits)  # {'mango', 'apple', 'orange', 'grape', 'cherry'}

set1 = {1, 2, 3}
set2 = {3, 4, 5}

print(set1.union(set2))        # {1, 2, 3, 4, 5}
print(set1.intersection(set2)) # {3}
```

### When to Use Sets

Sets are useful when you need to store a collection of unique items and you don't care about the order. They are also very efficient for membership testing (checking if an item is in the set) and for performing mathematical set operations like union, intersection, and difference.

### Conclusion

Sets are a powerful and efficient data structure in Python for managing collections of unique items. Their mathematical properties make them ideal for tasks involving membership testing and set logic. If you have a use case that requires uniqueness and you don't need to maintain order, a set is the perfect tool for the job.
