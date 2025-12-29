---
layout: post
title: "NumPy in Practice: A Developer's Cheatsheet"
date: 2025-12-28 11:30:00 +0545
categories: [Python, Data Science]
tags: [numpy, python, numerical-computing, data-science, cheatsheet]
---

# NumPy in Practice: A Developer's Cheatsheet

## Introduction

If pandas is the tool for wrangling structured data, NumPy is the bedrock it's built on. NumPy (Numerical Python) is the fundamental package for numerical computation in Python. It provides a powerful N-dimensional array object, sophisticated broadcasting functions, and tools for integrating C/C++ and Fortran code.

This guide is a practical cheatsheet. We'll skip the deep computer science and focus on the essential commands and concepts you'll use daily to perform fast, vectorized computations.

## Installation

If you have pandas or other scientific computing libraries, you likely already have NumPy. If not, it's a simple install.

```bash
pip install numpy
```

By convention, NumPy is always imported with the alias `np`.

```python
import numpy as np
```

## The NumPy Array: `ndarray`

The core of NumPy is the `ndarray` object: a fast, flexible container for large datasets in Python. Arrays are typed and homogenous (all elements must be of the same type), which is why they are so much faster than standard Python lists.

### Creating Arrays

#### From a Python List
This is the most straightforward way to create an array.

```python
# 1-dimensional array
arr1d = np.array([1, 2, 3, 4, 5])
print(arr1d)
# Output: [1 2 3 4 5]

# 2-dimensional array (matrix)
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2d)
# Output:
# [[1 2 3]
#  [4 5 6]]
```

#### Intrinsic Array Creation
NumPy provides functions to create arrays from scratch.

```python
# Create an array of zeros (3 rows, 4 columns)
zeros = np.zeros((3, 4))
print(zeros)

# Create an array of ones with a specific data type
ones = np.ones((2, 5), dtype=np.float32)
print(ones)

# Create an array with a range of elements
# (from 0 up to, but not including, 10)
range_arr = np.arange(10)
print(range_arr)

# Create an array of 5 evenly spaced values from 0 to 1
linspace_arr = np.linspace(0, 1, 5)
print(linspace_arr)
```

#### Random Arrays
The `np.random` module is essential for creating arrays with random data.

```python
# 2x3 array of random values between 0 and 1
rand_arr = np.random.rand(2, 3)
print(rand_arr)

# 2x3 array of samples from a standard normal distribution (mean 0, variance 1)
randn_arr = np.random.randn(2, 3)
print(randn_arr)

# 10 random integers between 5 (inclusive) and 15 (exclusive)
randint_arr = np.random.randint(5, 15, 10)
print(randint_arr)
```

## Array Inspection: Attributes

Knowing the properties of your array is the first step to working with it.

```python
arr = np.random.randn(4, 5)

# Shape: The dimensions of the array (rows, columns)
print(f"Shape: {arr.shape}") # (4, 5)

# Dtype: The data type of the elements
print(f"Data Type: {arr.dtype}") # float64

# Ndim: The number of dimensions
print(f"Dimensions: {arr.ndim}") # 2

# Size: The total number of elements
print(f"Size: {arr.size}") # 20
```

## Indexing and Slicing

Slicing in NumPy is similar to Python lists but can be applied to multiple dimensions.

```python
arr = np.arange(10) # [0 1 2 3 4 5 6 7 8 9]

# Get the element at index 3
print(arr[3]) # 4

# Get a slice from index 2 to 5 (exclusive)
print(arr[2:5]) # [2 3 4]

# Slicing a 2D array
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Get the element at row 1, column 2
print(arr2d[1, 2]) # 6

# Get the first two rows
print(arr2d[:2])
# [[1 2 3]
#  [4 5 6]]

# Get the first two rows, and columns 1 and 2
print(arr2d[:2, 1:])
# [[2 3]
#  [5 6]]
```

### Conditional Logic and Filtering

NumPy's true power in data analysis shines when you start filtering, selecting, and transforming values based on conditions.

#### Comparison Operators
Element-wise comparisons return a boolean array.

```python
arr = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# Check which elements are greater than 5
bool_arr = arr > 5
print(bool_arr)
# [[False False False]
#  [False False  True]
#  [ True  True  True]]
```

#### Boolean Indexing
You can use these boolean arrays to select data from your array. This is incredibly powerful.

```python
# Select only the elements greater than 5
print(arr[bool_arr]) # or just arr[arr > 5]
# [6 7 8 9]

# You can also use it to modify values
# Set all values greater than 5 to 0
arr[arr > 5] = 0
print(arr)
# [[1 2 3]
#  [4 5 0]
#  [0 0 0]]
```

#### Logical Operators
To combine multiple boolean conditions, you must use the bitwise operators `&` (and), `|` (or), and `~` (not). Python's built-in `and`, `or`, and `not` do **not** work for element-wise array comparisons.

```python
arr = np.arange(10) # [0 1 2 3 4 5 6 7 8 9]

# Find elements that are greater than 3 AND less than 8
condition = (arr > 3) & (arr < 8)
print(arr[condition])
# [4 5 6 7]

# Find elements that are less than 2 OR greater than 8
condition = (arr < 2) | (arr > 8)
print(arr[condition])
# [0 1 9]

# Find elements that are NOT greater than 5
condition = ~(arr > 5)
print(arr[condition])
# [0 1 2 3 4 5]
```

### Conditional Logic with `np.where()`
`np.where()` is NumPy's vectorized version of an `if/else` statement. It's a powerful tool for creating new arrays based on conditions. The syntax is `np.where(condition, value_if_true, value_if_false)`.

```python
arr = np.arange(10)

# If the element is even, multiply by 2; otherwise, leave it as is
result = np.where(arr % 2 == 0, arr * 2, arr)
print(result)
# [ 0  1  4  3  8  5 12  7 16  9]

# You can create more complex classifications
# If value < 5, label 'Low'; otherwise, label 'High'
result = np.where(arr < 5, 'Low', 'High')
print(result)
# ['Low' 'Low' 'Low' 'Low' 'Low' 'High' 'High' 'High' 'High' 'High']
```

## Vectorization and Broadcasting

This is NumPy's killer feature. Instead of writing slow Python loops, you can perform operations on entire arrays at once.

### Vectorized Operations
Arithmetic operations are applied element-wise.

```python
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])
arr3 = np.array([2, 10])

# Element-wise addition
print(arr1 + arr2)
# [[ 6  8]
#  [10 12]]

# Element-wise multiplication
print(arr1 * arr2)
# [[ 5 12]
#  [21 32]]

# Power (element-wise)
print(arr1 ** 2)
# [[ 1  4]
#  [ 9 16]]

# Floor Division and Modulo
print(arr2 // arr1)
# [[5 3]
#  [2 2]]
print(arr2 % arr1)
# [[0 0]
#  [1 0]]

# Scalar operations
print(arr1 * 10)
# [[10 20]
#  [30 40]]
```

### Broadcasting
Broadcasting describes how NumPy treats arrays with different shapes during arithmetic operations. The smaller array is "broadcast" across the larger array so that they have compatible shapes.

```python
# Example 1: Adding a 1D array to a 2D array
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
vector = np.array([10, 20, 30])

# The vector is broadcast across each row of the matrix
result = matrix + vector
print(result)
# [[11 22 33]
#  [14 25 36]
#  [17 28 39]]
```

## Universal Functions (ufuncs)

NumPy provides fast, element-wise functions called ufuncs.

```python
arr = np.arange(1, 6) # [1 2 3 4 5]

# Square root of each element
print(np.sqrt(arr))

# Exponential (e^) of each element
print(np.exp(arr))

# Sine of each element
print(np.sin(arr))
```

## Aggregate Functions

Functions that perform an operation on a set of values and produce a single result.

```python
arr = np.random.randn(5, 3)

# Sum of all elements
print(arr.sum())

# Mean of all elements
print(arr.mean())

# You can also compute along a specific axis
# axis=0 computes along columns, axis=1 computes along rows
print(f"Mean of each column: {arr.mean(axis=0)}")
print(f"Sum of each row: {arr.sum(axis=1)}")
```

## Array Manipulation

### Reshaping
Changes the shape of an array without changing its data.

```python
arr = np.arange(12) # A 1D array of 12 elements

# Reshape it to a 3x4 matrix
reshaped_arr = arr.reshape((3, 4))
print(reshaped_arr)

# Use -1 to have NumPy infer one of the dimensions
inferred_shape = arr.reshape((2, -1)) # Becomes 2x6
print(inferred_shape.shape)
```

### Concatenating and Stacking
Combining multiple arrays into one.

```python
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])

# Concatenate along rows (axis=0)
cat_rows = np.concatenate([arr1, arr2], axis=0)
print(cat_rows)

# Concatenate along columns (axis=1)
cat_cols = np.concatenate([arr1, arr2], axis=1)
print(cat_cols)

# vstack and hstack are convenient helpers
print(np.vstack((arr1, arr2))) # Same as concatenate with axis=0
print(np.hstack((arr1, arr2))) # Same as concatenate with axis=1
```

## Conclusion

NumPy is the engine of the scientific Python ecosystem. Its speed, efficiency, and power come from its `ndarray` object and the principles of vectorization and broadcasting. By avoiding explicit loops in Python and leveraging NumPy's optimized, C-based functions, you can achieve performance that is orders of magnitude faster. This cheatsheet covers the fundamentals, but mastering them will unlock new capabilities in your data analysis and scientific computing projects.

{% include inarticle-adsense.html %}

## Suggested Reading

- **Official NumPy Documentation:** The definitive guide for all NumPy functions and concepts. [numpy.org/doc/stable/](https://numpy.org/doc/stable/)
- **"Python for Data Analysis" by Wes McKinney:** Provides a great introduction to NumPy in the context of data analysis.
- **NumPy: the absolute basics for beginners:** A user-friendly tutorial from the official docs. [numpy.org/doc/stable/user/absolute_beginners.html](https://numpy.org/doc/stable/user/absolute_beginners.html)
