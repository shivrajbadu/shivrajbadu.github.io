---
layout: post
title: "A Quick Note on Python Booleans"
date: 2025-10-26 18:00:00 +0545
categories: [Python, Basics]
---

In programming, you often need to know if an expression is `True` or `False`. In Python, the `bool` data type represents one of two values: `True` or `False`. These are used to control the flow of your program in conditional statements and loops.

### Boolean Values

There are only two boolean values: `True` and `False`. Note that they are case-sensitive.

```python
is_active = True
is_admin = False

print(type(is_active))  # <class 'bool'>
```

### Booleans in Expressions

Booleans are often the result of comparison operations.

```python
print(10 > 9)   # True
print(10 == 9)  # False
print(10 < 9)   # False
```

They are also used with logical operators (`and`, `or`, `not`) to create more complex conditions.

```python
x = 10
y = 5

print(x > 5 and y < 10)  # True
print(x > 10 or y < 10)  # True
print(not(x > 5 and y < 10)) # False
```

### The `bool()` Function

You can use the `bool()` function to evaluate any value and get `True` or `False` in return.

- Most values are evaluated to `True` if they have some sort of content.
- Any string is `True`, except empty strings.
- Any number is `True`, except `0`.
- Any list, tuple, set, and dictionary are `True`, except empty ones.

```python
print(bool("Hello"))  # True
print(bool(15))      # True
print(bool([]))        # False
print(bool(0))         # False
```

### Conclusion

Booleans are a fundamental concept in Python and programming in general. They are essential for controlling the logic and flow of your code. By understanding how `True` and `False` work, you can write more intelligent and responsive programs.
