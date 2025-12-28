---
layout: post
title: "Python DSA Series 02: Stacks and Queues"
date: 2025-12-25 14:00:00 +0545
categories: [Python, DSA]
tags: [python, dsa, stacks, queues, data-structures, algorithms]
---

# Python DSA Series 02: Stacks and Queues

In our previous post, we explored Python's `list`, a versatile and powerful data structure. We learned that its main weakness is the O(n) cost of inserting or deleting elements from the beginning. This is where abstract data types like Stacks and Queues come in. They are more restrictive, but these restrictions provide guaranteed efficiency for specific use cases.

## What are Abstract Data Types (ADTs)?

A Stack or a Queue isn't a concrete data structure like a list; it's an **Abstract Data Type (ADT)**. An ADT defines a set of operations (e.g., "add item," "remove item") and their behavior, without specifying *how* they are implemented. You can implement them using lists, linked lists, or other structures.

---

## Stacks: Last-In, First-Out (LIFO)

A stack follows the **LIFO** principle: the last element added is the first one to be removed.

*   **Analogy:** Think of a stack of plates. You add a new plate to the top, and you also take a plate from the top. You don't pull a plate from the bottom of the stack.

### Core Operations

1.  **Push:** Add an element to the top of the stack.
2.  **Pop:** Remove and return the element from the top of the stack.
3.  **Peek (or Top):** Return the top element without removing it.
4.  **isEmpty:** Check if the stack is empty.

### Python Implementation using `list`

A Python `list` is perfect for implementing a stack. We can use `append()` for our `push` operation and `pop()` for our `pop` operation. Both are O(1) on average.

```python
class Stack:
    def __init__(self):
        self.items = []

    def is_empty(self):
        return not self.items

    def push(self, item):
        self.items.append(item) # O(1)

    def pop(self):
        if not self.is_empty():
            return self.items.pop() # O(1)
        return None

    def peek(self):
        if not self.is_empty():
            return self.items[-1] # O(1)
        return None

    def size(self):
        return len(self.items)

# Usage
s = Stack()
s.push(10)
s.push(20)
s.push(30)

print(f"Top element is: {s.peek()}")   # Output: 30
print(f"Popped element: {s.pop()}") # Output: 30
print(f"New top element is: {s.peek()}") # Output: 20
```

### Common Use Case: Valid Parentheses

Stacks are ideal for problems involving matching pairs or reversing order. A classic interview question is to validate a string of parentheses.

```python
def is_valid_parentheses(s):
    stack = []
    mapping = {")": "(", "}": "{", "]": "["}

    for char in s:
        if char in mapping: # It's a closing bracket
            top_element = stack.pop() if stack else '#'
            if mapping[char] != top_element:
                return False
        else: # It's an opening bracket
            stack.append(char)

    return not stack # If stack is empty, all were matched

# Examples
print(is_valid_parentheses("()[]{}")) # Output: True
print(is_valid_parentheses("([)]"))   # Output: False
print(is_valid_parentheses("(]"))      # Output: False
```

---

## Queues: First-In, First-Out (FIFO)

A queue follows the **FIFO** principle: the first element added is the first one to be removed.

*   **Analogy:** Think of a line at a checkout counter. The first person to get in line is the first person to be served.

### Core Operations

1.  **Enqueue:** Add an element to the back (rear) of the queue.
2.  **Dequeue:** Remove and return the element from the front of the queue.
3.  **Peek (or Front):** Return the front element without removing it.
4.  **isEmpty:** Check if the queue is empty.

### Python Implementation using `collections.deque`

You *can* implement a queue with a Python `list`, but it's very inefficient. Enqueuing with `list.append()` is O(1), but dequeuing from the front with `list.pop(0)` is an **O(n)** operation because all other elements must be shifted.

A far better solution is to use Python's `collections.deque` (pronounced "deck," short for "double-ended queue"). It's specifically designed for fast appends and pops from both ends, with all operations being **O(1)**.

```python
from collections import deque

class Queue:
    def __init__(self):
        self.items = deque()

    def is_empty(self):
        return not self.items

    def enqueue(self, item):
        self.items.append(item) # O(1)

    def dequeue(self):
        if not self.is_empty():
            return self.items.popleft() # O(1)
        return None

    def peek(self):
        if not self.is_empty():
            return self.items[0] # O(1)
        return None

    def size(self):
        return len(self.items)

# Usage
q = Queue()
q.enqueue("Task 1")
q.enqueue("Task 2")
q.enqueue("Task 3")

print(f"Front element is: {q.peek()}")      # Output: Task 1
print(f"Dequeued element: {q.dequeue()}") # Output: Task 1
print(f"New front element is: {q.peek()}")  # Output: Task 2
```

### Common Use Case: Task Scheduling

Queues are perfect for managing tasks in the order they were received, like processing jobs in a background worker or implementing a breadth-first search (BFS) algorithm for trees and graphs.

## Conclusion

Stacks (LIFO) and Queues (FIFO) are fundamental ADTs that provide efficient, predictable performance for specific ordering needs.

-   Use a **Stack** when you need to process items in reverse order of their arrival (e.g., undo functionality, call stacks, parsing).
-   Use a **Queue** when you need to process items in the order they were received (e.g., task scheduling, messaging systems, BFS).

While a Python `list` can implement a stack perfectly, always prefer `collections.deque` for implementing a queue to avoid O(n) performance bottlenecks.

In our next post, we'll dive into **Linked Lists**, a data structure that offers a different set of trade-offs, particularly when it comes to insertions and deletions.

{% include inarticle-adsense.html %}

## Suggested Reading
- [collections.deque - Official Python Documentation](https://docs.python.org/3/library/collections.html#collections.deque)
- [Stacks in Python (Real Python)](https://realpython.com/how-to-implement-a-python-stack/)
- [Queues in Python (Real Python)](https://realpython.com/how-to-implement-a-python-queue/)
