---
layout: post
title: "A Quick Note on Python While Loops"
date: 2025-10-27 00:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

Loops are a fundamental concept in programming that allow you to execute a block of code repeatedly. In Python, the `while` loop is used to execute a set of statements as long as a condition is true.

### The `while` Loop

The `while` loop requires a condition to be set up before the loop. The loop will continue to run as long as the condition is true.

```python
i = 1
while i < 6:
    print(i)
    i += 1
```

### The `break` Statement

With the `break` statement, we can stop the loop even if the while condition is true.

```python
i = 1
while i < 6:
    print(i)
    if i == 3:
        break
    i += 1
```

### The `continue` Statement

With the `continue` statement, we can stop the current iteration and continue with the next.

```python
i = 0
while i < 6:
    i += 1
    if i == 3:
        continue
    print(i)
```

### The `else` Statement

With the `else` statement, we can run a block of code once when the condition no longer is true.

```python
i = 1
while i < 6:
    print(i)
    i += 1
else:
    print("i is no longer less than 6")
```

### Conclusion

The `while` loop is a powerful tool for creating loops that run as long as a certain condition is met. By using `break` and `continue`, you can control the flow of the loop with even more precision. The `else` statement provides a way to execute code after the loop has finished normally. Understanding how to use `while` loops effectively is a key skill for any Python programmer.
