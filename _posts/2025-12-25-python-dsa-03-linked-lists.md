---
layout: post
title: "Python DSA Series 03: Understanding Linked Lists"
date: 2025-12-25 14:05:00 +0545
categories: [Python, DSA]
tags: [python, dsa, linked-lists, data-structures, algorithms]
---

# Python DSA Series 03: Understanding Linked Lists

In our previous posts, we saw that Python `list`s are fast for appends and access but slow for insertions and deletions at the beginning (O(n)). Stacks and Queues are abstract types that enforce rules to guarantee O(1) performance for specific use cases.

Now, we explore a fundamental data structure that tackles the insertion/deletion problem from a different angle: the **Linked List**.

## What is a Linked List?

Unlike a Python `list` (dynamic array), which stores elements in a contiguous block of memory, a linked list stores elements in individual objects called **nodes**. Each node contains two pieces of information:
1.  The **data** (the value it holds).
2.  A **pointer** (or reference) to the next node in the sequence.

The linked list itself is defined by a single pointer to its first node, called the **head**. The last node in the list points to `None`, indicating the end of the list.

**Visual Representation:**

`[Head] -> [Node(Data|Next)] -> [Node(Data|Next)] -> [Node(Data|Next)] -> None`

This structure is the key to its performance. To insert or delete a node, you don't need to shift other elements; you just need to change a few pointers.

## Implementing a Linked List in Python

Python doesn't have a built-in linked list, so we build it from scratch. This is a very common task in technical interviews.

### The `Node` Class

First, we need a simple class to represent a node.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None # Pointer to the next node
```

### The `SinglyLinkedList` Class

Now, we create the main class that manages the nodes.

```python
class SinglyLinkedList:
    def __init__(self):
        self.head = None # The list is initially empty

    # Method to print the list's contents
    def print_list(self):
        current_node = self.head
        while current_node:
            print(current_node.data, end=" -> ")
            current_node = current_node.next
        print("None")

    # Add a new node to the beginning of the list (O(1))
    def prepend(self, data):
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node

    # Add a new node to the end of the list (O(n))
    def append(self, data):
        new_node = Node(data)
        if not self.head: # If the list is empty
            self.head = new_node
            return
        
        last_node = self.head
        while last_node.next: # Traverse to the last node
            last_node = last_node.next
        last_node.next = new_node

    # Delete a node with a given key (O(n))
    def delete_node(self, key):
        current_node = self.head

        # If the head node itself holds the key
        if current_node and current_node.data == key:
            self.head = current_node.next
            current_node = None
            return

        # Search for the key to be deleted
        prev = None
        while current_node and current_node.data != key:
            prev = current_node
            current_node = current_node.next

        # If the key was not present in the list
        if not current_node:
            return

        # Unlink the node from the list
        prev.next = current_node.next
        current_node = None
```

### Example Usage

```python
llist = SinglyLinkedList()
llist.append("A")
llist.append("B")
llist.prepend("Start")
llist.append("C")

llist.print_list() # Output: Start -> A -> B -> C -> None

llist.delete_node("B")
llist.print_list() # Output: Start -> A -> C -> None
```

## Time Complexity: Linked List vs. Python List

Here's how linked lists stack up against Python's dynamic arrays (`list`).

| Operation | Python `list` | Singly Linked List | Why the Difference? |
| :--- | :--- | :--- | :--- |
| **Access (by index)** | **O(1)** | **O(n)** | Python lists have contiguous memory, allowing instant index calculation. Linked lists must be traversed from the head. |
| **Search (by value)** | O(n) | O(n) | Both may need to scan the entire structure. |
| **Insertion (at beginning)** | O(n) | **O(1)** | A Python list must shift all elements. A linked list only needs to update the `head` pointer. |
| **Deletion (at beginning)** | O(n) | **O(1)** | Same reason as insertion. |
| **Insertion (at end)** | **O(1)** | O(n) | A Python list has amortized O(1) appends. A singly linked list must traverse to the end to append. (This can be O(1) if we also store a `tail` pointer). |
| **Deletion (at end)** | O(1) | O(n) | A Python list can pop in O(1). A singly linked list must traverse to the second-to-last node. |

## Types of Linked Lists

1.  **Singly Linked List:** The one we just built. Nodes have a single pointer to the next node. Traversal is one-way.
2.  **Doubly Linked List:** Each node has *two* pointers: one to the `next` node and one to the `previous` node. This allows for backward traversal and makes deleting a specific node O(1) *if you already have a reference to it*. The trade-off is higher memory usage.
3.  **Circular Linked List:** The `next` pointer of the last node points back to the `head` instead of `None`. This can be useful for applications where you need to cycle through a list of items continuously (e.g., a slideshow).

## Conclusion

Linked lists are a foundational concept in computer science, and building one is a rite of passage in coding interviews.

**Key Trade-offs:**
-   **Choose a Linked List** when you need fast insertions and deletions at the beginning of your sequence and you don't need fast random access to elements by index.
-   **Stick with a Python `list`** for almost everything else. Its O(1) access and append operations, combined with better cache performance (due to contiguous memory), make it the superior general-purpose choice.

Understanding linked lists is less about using them in day-to-day Python and more about understanding the fundamental trade-offs between different data storage patterns.

Next up, we'll explore **Hash Tables** (the powerhouse behind Python's dictionaries), a data structure that provides average O(1) performance for lookups, insertions, *and* deletions.

{% include inarticle-adsense.html %}

## Suggested Reading
- [Linked Lists in Python (Real Python)](https://realpython.com/linked-lists-python/)
- [Data Structures & Algorithms in Python](https://www.amazon.com/Structures-Algorithms-Python-Michael-Goodrich/dp/1118290275) - A comprehensive textbook.
