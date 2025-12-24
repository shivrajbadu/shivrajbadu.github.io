---
layout: post
title: "A Quick Note on Python If...Else"
date: 2025-10-26 23:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

Conditional statements are a fundamental part of programming, allowing you to execute different blocks of code based on whether a certain condition is true or false. In Python, this is achieved using `if`, `elif`, and `else` statements.

### The `if` Statement

The `if` statement is used to execute a block of code only if a specified condition is true.

```python
x = 10
y = 5

if x > y:
    print("x is greater than y")
```

### The `elif` Statement

The `elif` keyword is Python's way of saying "if the previous conditions were not true, then try this condition".

```python
x = 10
y = 10

if x > y:
    print("x is greater than y")
elif x == y:
    print("x and y are equal")
```

### The `else` Statement

The `else` keyword catches anything which isn't caught by the preceding conditions.

```python
x = 5
y = 10

if x > y:
    print("x is greater than y")
elif x == y:
    print("x and y are equal")
else:
    print("y is greater than x")
```

### Short Hand If

If you have only one statement to execute, you can put it on the same line as the `if` statement.

```python
if x > y: print("x is greater than y")
```

### Short Hand If...Else

This is a more compact way to write an `if...else` statement, also known as the ternary operator.

```python
x = 10
y = 5

print("x is greater") if x > y else print("y is greater or equal")
```

### Conclusion

Conditional statements are essential for controlling the flow of your Python programs. By using `if`, `elif`, and `else`, you can create flexible and intelligent applications that respond to different conditions. The shorthand versions can also help you write more concise code for simple conditions.

---

{% include inarticle-adsense.html %}
