---
layout: post
title: "A Quick Note on Python's range()"
date: 2025-10-27 04:00:00 +0545
categories: [Python, Basics]
---

The `range()` function is a built-in function in Python that is used to generate a sequence of numbers. It is commonly used for looping a specific number of times in `for` loops.

### `range(stop)`

When you call `range()` with one argument, you get a sequence of numbers that starts at 0 and increments by 1, up to (but not including) the specified number.

```python
# Generate numbers from 0 to 4
for i in range(5):
    print(i)
```

### `range(start, stop)`

When you call `range()` with two arguments, you get a sequence of numbers that starts at the first argument, increments by 1, and goes up to (but not including) the second argument.

```python
# Generate numbers from 2 to 5
for i in range(2, 6):
    print(i)
```

### `range(start, stop, step)`

When you call `range()` with three arguments, the third argument is the step, which is the difference between each number in the sequence.

```python
# Generate even numbers from 2 to 10
for i in range(2, 11, 2):
    print(i)

# Generate numbers from 10 down to 1
for i in range(10, 0, -1):
    print(i)
```

### Conclusion

The `range()` function is a simple yet powerful tool for generating sequences of numbers in Python. It is memory efficient because it only stores the start, stop, and step values, and generates the numbers on the fly as you iterate over them. Whether you're looping a specific number of times or creating a custom sequence of numbers, `range()` is the perfect tool for the job.
