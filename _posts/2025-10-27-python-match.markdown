---
layout: post
title: "A Quick Note on Python's Match Statement"
date: 2025-10-27 03:00:00 +0545
categories: [Python, Basics]
---

Introduced in Python 3.10, the `match` statement is a powerful tool for structural pattern matching. It allows you to compare a value against a series of patterns and execute a block of code when a match is found. It's similar to a `switch` statement in other languages, but much more powerful.

### Basic `match` Statement

Here's a basic example of a `match` statement:

```python
def http_status(status):
    match status:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return "Unknown status"

print(http_status(200))  # OK
print(http_status(404))  # Not Found
```

### Matching Multiple Values

You can match multiple values in a single `case` using the `|` (or) operator.

```python
def http_status_category(status):
    match status:
        case 200 | 201 | 202:
            return "Success"
        case 400 | 401 | 404:
            return "Client Error"
        case 500 | 501 | 503:
            return "Server Error"
        case _:
            return "Unknown"

print(http_status_category(201))  # Success
print(http_status_category(404))  # Client Error
```

### The Wildcard `_`

The underscore `_` is a wildcard that will match anything. It's used as a default case if no other case matches.

```python
def process_value(value):
    match value:
        case 1:
            print("It's one!")
        case "hello":
            print("It's a greeting!")
        case _:
            print("It's something else.")

process_value(1)
process_value("hello")
process_value([1, 2, 3])
```

### `match` with `if` Guards

You can add an `if` condition to a `case` statement to create a "guard". The case will only match if the pattern matches and the guard condition is true.

```python
def process_point(point):
    match point:
        case (x, y) if x == y:
            print(f"The point ({x}, {y}) is on the diagonal.")
        case (x, y):
            print(f"The point is at ({x}, {y}).")

process_point((5, 5))
process_point((5, 10))
```

### Conclusion

The `match` statement is a powerful and expressive feature in Python that simplifies complex conditional logic. It allows you to write cleaner and more readable code by matching against a variety of patterns. If you're using Python 3.10 or later, the `match` statement is a great tool to have in your arsenal.
