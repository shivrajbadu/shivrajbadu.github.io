---
layout: post
title: "A Quick Note on Python Iterators"
date: 2025-10-27 06:00:00 +0545
categories: [Python, Basics]
---

An iterator is an object that contains a countable number of values. In Python, an iterator is an object that can be iterated upon, meaning that you can traverse through all the values. Technically, in Python, an iterator is an object which implements the iterator protocol, which consists of the methods `__iter__()` and `__next__()`.

### Iterables vs. Iterators

An **iterable** is any object that can be looped over, like a list, tuple, or string. You can get an iterator from an iterable by using the `iter()` function.

An **iterator** is an object that does the iterating. It produces the next value in the sequence when you call the `next()` function on it.

### The `iter()` and `next()` Functions

Let's see how `iter()` and `next()` work together.

```python
my_list = ["apple", "banana", "cherry"]
my_iterator = iter(my_list)

print(next(my_iterator))  # 'apple'
print(next(my_iterator))  # 'banana'
print(next(my_iterator))  # 'cherry'

# This would raise a StopIteration exception
# print(next(my_iterator))
```

### Looping Through an Iterator

The `for` loop in Python automatically creates an iterator from an iterable and calls `next()` on it until a `StopIteration` exception is raised.

```python
my_list = ["apple", "banana", "cherry"]

for fruit in my_list:
    print(fruit)
```

This is what happens behind the scenes in a `for` loop.

### Creating a Custom Iterator

To create your own iterator, you need to implement the `__iter__()` and `__next__()` methods in your class.

```python
class MyNumbers:
    def __iter__(self):
        self.a = 1
        return self

    def __next__(self):
        if self.a <= 5:
            x = self.a
            self.a += 1
            return x
        else:
            raise StopIteration

my_class = MyNumbers()
my_iter = iter(my_class)

for x in my_iter:
    print(x)
```

### Conclusion

Iterators are a fundamental concept in Python that provide a clean and efficient way to work with sequences of data. They are the underlying mechanism that powers `for` loops and other iterable operations. By understanding how iterators work, you can write more memory-efficient and Pythonic code, especially when dealing with large datasets.
