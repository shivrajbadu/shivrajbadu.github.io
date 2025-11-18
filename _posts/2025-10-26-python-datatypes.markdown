---
layout: post
title: "A Quick Note on Python Data Types"
date: 2025-10-26 13:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

In Python, data types are the classification or categorization of data items. It represents the kind of value that tells what operations can be performed on a particular data. Since everything is an object in Python, data types are actually classes and variables are instance (object) of these classes.

### Common Data Types

Here are some of the most common data types in Python:

- **Text Type:** `str` (String)

  - Used for textual data. Strings are enclosed in single or double quotes.
  - Example: `name = "Shivraj"`

- **Numeric Types:** `int` (Integer), `float` (Floating-Point Number), `complex` (Complex Number)

  - `int`: Whole numbers, positive or negative, without decimals.
    - Example: `age = 25`
  - `float`: Numbers, positive or negative, containing one or more decimals.
    - Example: `price = 19.99`
  - `complex`: Numbers with a real and an imaginary part.
    - Example: `z = 1 + 2j`

- **Sequence Types:** `list`, `tuple`, `range`

  - `list`: A collection which is ordered and changeable. Allows duplicate members.
    - Example: `fruits = ["apple", "banana", "cherry"]`
  - `tuple`: A collection which is ordered and unchangeable. Allows duplicate members.
    - Example: `coordinates = (10, 20)`
  - `range`: Represents a sequence of numbers.
    - Example: `numbers = range(6)`

- **Mapping Type:** `dict` (Dictionary)

  - A collection of key-value pairs. It is unordered, changeable and does not allow duplicates.
  - Example: `person = {"name": "John", "age": 36}`

- **Set Types:** `set`, `frozenset`

  - `set`: A collection which is unordered and unindexed. No duplicate members.
    - Example: `unique_numbers = {1, 2, 3}`
  - `frozenset`: An immutable version of a set.
    - Example: `frozen = frozenset({1, 2, 3})`

- **Boolean Type:** `bool`
  - Represents one of two values: `True` or `False`.
  - Example: `is_student = True`

### Conclusion

Understanding data types is crucial for writing effective Python code. They are the building blocks of data manipulation and will be used in every program you write. Python's rich set of built-in data types makes it a versatile and powerful language.
