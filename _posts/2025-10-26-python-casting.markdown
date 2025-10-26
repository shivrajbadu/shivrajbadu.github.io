---
layout: post
title: "A Quick Note on Python Casting"
date: 2025-10-26 16:00:00 +0545
categories: [Python, Basics]
---

In Python, casting is the process of converting a variable from one data type to another. Python is a dynamically-typed language, but sometimes you need to explicitly convert a variable's type. This is where casting comes in.

### Casting to Integer (`int()`)

You can convert a float or a string to an integer using the `int()` function. When converting a float, the decimal part is truncated.

```python
x = int(3.14)    # x will be 3
y = int("10")    # y will be 10

print(x)
print(y)
```

### Casting to Float (`float()`)

You can convert an integer or a string to a float using the `float()` function.

```python
x = float(10)      # x will be 10.0
y = float("3.14")  # y will be 3.14

print(x)
print(y)
```

### Casting to String (`str()`)

You can convert almost any data type to a string using the `str()` function. This is useful when you want to concatenate a number with a string.

```python
x = str(10)
y = str(3.14)
z = str(True)

print("The value of x is " + x)
```

### Conclusion

Casting is a simple yet powerful feature in Python that allows you to control the data types of your variables. By using functions like `int()`, `float()`, and `str()`, you can ensure that your variables are in the correct format for the operations you want to perform.
