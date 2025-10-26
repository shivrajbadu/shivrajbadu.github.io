---
layout: post
title: "A Quick Note on Python Strings"
date: 2025-10-26 17:00:00 +0545
categories: [Python, Basics]
---

Strings are one of the most commonly used data types in Python. They are sequences of characters, enclosed in either single quotes (`'`) or double quotes (`"`).

### Creating Strings

Creating a string is as simple as assigning a value to a variable.

```python
# Single quotes
name = 'Shivraj'

# Double quotes
message = "Hello, World!"

# Multiline strings
long_string = """
This is a long string
that spans multiple lines.
"""
```

### String Slicing

You can access individual characters or a range of characters in a string using slicing.

```python
s = "Hello, World!"

# Get the character at position 1
print(s[1])  # 'e'

# Get the characters from position 2 to 5 (not included)
print(s[2:5])  # 'llo'
```

### String Methods

Python has a rich set of built-in methods that you can use on strings.

- `upper()`: Converts a string into upper case.
- `lower()`: Converts a string into lower case.
- `strip()`: Removes any whitespace from the beginning or the end.
- `replace()`: Replaces a string with another string.
- `split()`: Splits the string into substrings if it finds instances of the separator.

```python
s = "  Hello, World!  "

print(s.upper())         # '  HELLO, WORLD!  '
print(s.strip())         # 'Hello, World!'
print(s.replace('H', 'J')) # '  Jello, World!  '
```

### String Concatenation

You can combine two or more strings using the `+` operator.

```python
first_name = "Shivraj"
last_name = "Badu"
full_name = first_name + " " + last_name

print(full_name)  # 'Shivraj Badu'
```

### Conclusion

Strings are a fundamental part of Python, and understanding how to work with them is essential for any Python programmer. With a rich set of built-in methods and easy-to-use slicing and concatenation, Python makes string manipulation a breeze.
