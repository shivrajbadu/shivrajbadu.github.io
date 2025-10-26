---
layout: post
title: "A Quick Note on Python Operators"
date: 2025-10-26 14:00:00 +0545
categories: [Python, Basics]
---

Operators are special symbols in Python that carry out arithmetic or logical computation. The value that the operator operates on is called the operand. In this note, we'll cover the different types of operators available in Python.

### Arithmetic Operators

These are used to perform mathematical operations like addition, subtraction, multiplication, etc.

- `+` (Addition), `-` (Subtraction), `*` (Multiplication), `/` (Division)
- `%` (Modulus), `**` (Exponentiation), `//` (Floor division)

```python
x = 10
y = 3
print(x + y)  # 13
print(x % y)  # 1
```

### Assignment Operators

Used to assign values to variables.

- `=` (Assignment), `+=` (Add and assign), `-=` (Subtract and assign), etc.

```python
a = 5
a += 3  # a is now 8
```

### Comparison (Relational) Operators

Used to compare two values.

- `==` (Equal), `!=` (Not equal), `>` (Greater than), `<` (Less than), `>=` (Greater than or equal to), `<=` (Less than or equal to)

```python
x = 5
y = 10
print(x == y)  # False
```

### Logical Operators

Used to combine conditional statements.

- `and` (Returns True if both statements are true)
- `or` (Returns True if one of the statements is true)
- `not` (Reverse the result, returns False if the result is true)

```python
x = True
y = False
print(x and y)  # False
```

### Identity Operators

Used to compare the objects, not if they are equal, but if they are actually the same object, with the same memory location.

- `is` (Returns True if both variables are the same object)
- `is not` (Returns True if both variables are not the same object)

```python
x = ["apple", "banana"]
y = x
print(x is y)  # True
```

### Membership Operators

Used to test if a sequence is presented in an object.

- `in` (Returns True if a sequence with the specified value is present in the object)
- `not in` (Returns True if a sequence with the specified value is not present in the object)

```python
fruits = ["apple", "banana"]
print("apple" in fruits)  # True
```

### Bitwise Operators

Used to compare (binary) numbers.

- `&` (AND), `|` (OR), `^` (XOR), `~` (NOT), `<<` (Zero fill left shift), `>>` (Signed right shift)

```python
print(6 & 3)  # 2
```

### Conclusion

Operators are the backbone of any programming language, and Python provides a rich set of them. Understanding how to use them effectively will allow you to write more concise and efficient code.
