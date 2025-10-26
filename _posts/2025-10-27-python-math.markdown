---
layout: post
title: "A Quick Note on Python Math"
date: 2025-10-27 08:00:00 +0545
categories: [Python, Basics]
---

Python has a set of built-in mathematical functions, and also a `math` module that provides more advanced mathematical functions. In this note, we'll explore both.

### Built-in Math Functions

Python has a few built-in functions that are useful for mathematical operations.

- `min()` and `max()`: Find the lowest or highest value in an iterable.
- `abs()`: Returns the absolute (positive) value of a number.
- `pow(x, y)`: Returns the value of x to the power of y.

```python
print(min(5, 10, 25))  # 5
print(max(5, 10, 25))  # 25
print(abs(-7.25))    # 7.25
print(pow(4, 3))     # 64
```

### The `math` Module

For more advanced mathematical functions, you need to import the `math` module.

```python
import math
```

### Common `math` Module Functions

Here are some of the most commonly used functions in the `math` module:

- `math.sqrt()`: Returns the square root of a number.
- `math.ceil()`: Rounds a number upwards to its nearest integer.
- `math.floor()`: Rounds a number downwards to its nearest integer.

```python
print(math.sqrt(64))   # 8.0
print(math.ceil(1.4))  # 2
print(math.floor(1.4)) # 1
```

### `math` Module Constants

The `math` module also provides some useful constants.

- `math.pi`: The mathematical constant Pi (3.1415...).
- `math.e`: The mathematical constant e (2.7182...).

```python
print(math.pi)
print(math.e)
```

### Conclusion

Python provides a solid foundation for mathematical operations with its built-in functions and the `math` module. Whether you're performing simple calculations or more complex scientific computations, Python has the tools you need. For even more advanced mathematical and scientific operations, you can explore libraries like NumPy and SciPy.
