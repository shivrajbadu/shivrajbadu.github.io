---
layout: post
title: "A Quick Note on Python Numbers"
date: 2025-10-26 15:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

Numbers are a fundamental data type in any programming language, and Python is no exception. Python supports several types of numbers, but the most common are `int`, `float`, and `complex`.

### Integers (`int`)

Integers are whole numbers, both positive and negative, without any decimal points. In Python, integers can be of any length, limited only by the memory available.

```python
# Integer examples
x = 10
y = -3000
z = 12345678901234567890

print(type(x))  # <class 'int'>
```

### Floating-Point Numbers (`float`)

Floating-point numbers, or floats, are numbers that have a decimal point. They can also be scientific numbers with an "e" to indicate the power of 10.

```python
# Float examples
x = 10.5
y = -3.14
z = 35e3

print(type(x))  # <class 'float'>
```

### Complex Numbers (`complex`)

Complex numbers are written with a "j" as the imaginary part. They consist of a real part and an imaginary part.

```python
# Complex number examples
x = 3 + 5j
y = -5j
z = 3j

print(type(x))  # <class 'complex'>
```

You can access the real and imaginary parts of a complex number using the `real` and `imag` attributes:

```python
print(x.real)  # 3.0
print(x.imag)  # 5.0
```

### Conclusion

Python's number types provide a flexible and powerful way to work with numerical data. Whether you're doing simple arithmetic or complex scientific calculations, Python has the right tools for the job. Understanding the differences between `int`, `float`, and `complex` will help you write more accurate and efficient code.
