---
layout: post
title: "A Quick Note on Python For Loops"
date: 2025-10-27 01:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

A `for` loop is used for iterating over a sequence (that is either a list, a tuple, a dictionary, a set, or a string). This is less like the `for` keyword in other programming languages, and works more like an iterator method as found in other object-orientated programming languages.

### Looping Through a Sequence

You can loop through the items of any sequence.

```python
# Looping through a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Looping through a string
for char in "banana":
    print(char)
```

### The `range()` Function

To loop through a set of code a specified number of times, we can use the `range()` function.

```python
for i in range(5):  # Looping from 0 to 4
    print(i)
```

### The `break` Statement

With the `break` statement, we can stop the loop before it has looped through all the items.

```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
    if fruit == "banana":
        break
```

### The `continue` Statement

With the `continue` statement, we can stop the current iteration of the loop, and continue with the next.

```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    if fruit == "banana":
        continue
    print(fruit)
```

### The `else` Statement

With the `else` statement, we can run a block of code once when the loop is finished.

```python
for i in range(5):
    print(i)
else:
    print("Finally finished!")
```

### Conclusion

The `for` loop is a powerful and versatile tool for iterating over sequences in Python. It provides a clean and readable way to process lists, strings, and other iterable objects. By using `break`, `continue`, and `else`, you can control the flow of your loops with great precision. Mastering the `for` loop is a crucial step in becoming a proficient Python programmer.
