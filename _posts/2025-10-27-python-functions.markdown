---
layout: post
title: "A Quick Note on Python Functions"
date: 2025-10-27 02:00:00 +0545
categories: [Python, Basics]
tags: [python]
---

A function is a block of code which only runs when it is called. You can pass data, known as parameters, into a function. A function can return data as a result.

### Creating and Calling a Function

In Python, a function is defined using the `def` keyword.

```python
# Creating a function
def my_function():
    print("Hello from a function")

# Calling a function
my_function()
```

### Arguments (Parameters)

Information can be passed into functions as arguments. Arguments are specified after the function name, inside the parentheses.

```python
def greet(name):
    print("Hello, " + name)

greet("Shivraj")
```

### Arbitrary Arguments, `*args`

If you do not know how many arguments that will be passed into your function, add a `*` before the parameter name in the function definition.

```python
def my_function(*kids):
    print("The youngest child is " + kids[2])

my_function("Emil", "Tobias", "Linus")
```

### Keyword Arguments, `**kwargs`

If you do not know how many keyword arguments that will be passed into your function, add two asterisk: `**` before the parameter name in the function definition.

```python
def my_function(**kid):
    print("His last name is " + kid["lname"])

my_function(fname = "Tobias", lname = "Refsnes")
```

### Default Parameter Value

The following example shows how to use a default parameter value.

```python
def my_function(country = "Norway"):
    print("I am from " + country)

my_function("Sweden")
my_function() # This will use the default value
```

### Return Values

To let a function return a value, use the `return` statement.

```python
def my_function(x):
    return 5 * x

print(my_function(3)) # 15
```

### Conclusion

Functions are a fundamental building block of any Python program. They allow you to organize your code into reusable blocks, making your programs more modular, readable, and efficient. By mastering functions, you can write more complex and powerful Python applications.
