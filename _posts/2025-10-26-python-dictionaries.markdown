---
layout: post
title: "A Quick Note on Python Dictionaries"
date: 2025-10-26 22:00:00 +0545
categories: [Python, Basics]
---

Dictionaries are used to store data values in **key:value** pairs. A dictionary is a collection which is ordered (starting from Python 3.7), changeable, and does not allow duplicate keys.

### Creating Dictionaries

Dictionaries are written with curly brackets, and have keys and values.

```python
# A dictionary of a person's information
person = {
    "first_name": "Shivraj",
    "last_name": "Badu",
    "age": 30
}
```

### Accessing Items

You can access the items of a dictionary by referring to its key name, inside square brackets.

```python
print(person["first_name"])  # 'Shivraj'
```

There is also a method called `get()` that will give you the same result:

```python
print(person.get("age"))  # 30
```

The difference is that `get()` will return `None` if the key does not exist, while using square brackets will raise a `KeyError`.

### Dictionary Methods

Python has a set of built-in methods that you can use on dictionaries.

- `keys()`: Returns a view object that displays a list of all the keys in the dictionary.
- `values()`: Returns a view object that displays a list of all the values in the dictionary.
- `items()`: Returns a view object that displays a list of a given dictionary’s key-value tuple pair.

```python
print(person.keys())    # dict_keys(['first_name', 'last_name', 'age'])
print(person.values())  # dict_values(['Shivraj', 'Badu', 30])
print(person.items())   # dict_items([('first_name', 'Shivraj'), ('last_name', 'Badu'), ('age', 30)])
```

### Adding and Updating Items

You can add new items or change the value of existing items using the assignment operator.

```python
# Add a new item
person["city"] = "New York"

# Update an existing item
person["age"] = 31

print(person)
```

### Conclusion

Dictionaries are a powerful and flexible data structure in Python. Their ability to store data in key-value pairs makes them ideal for a wide range of applications, from simple data storage to complex data mapping. If you need to store data that has a clear relationship between a key and a value, a dictionary is the perfect tool for the job.
