---
layout: post
title: "A Quick Note on Python and JSON"
date: 2025-10-27 09:00:00 +0545
categories: [Python, Basics]
---

JSON (JavaScript Object Notation) is a lightweight data-interchange format that is easy for humans to read and write, and easy for machines to parse and generate. In Python, you can work with JSON data using the built-in `json` module.

### Parsing JSON: From JSON to Python

If you have a JSON string, you can parse it by using the `json.loads()` method. The result will be a Python dictionary.

```python
import json

# some JSON:
x = '{"name":"John", "age":30, "city":"New York"}'

# parse x:
y = json.loads(x)

# the result is a Python dictionary:
print(y["age"])  # 30
```

### Converting to JSON: From Python to JSON

If you have a Python object (like a dictionary), you can convert it into a JSON string by using the `json.dumps()` method.

```python
import json

# a Python object (dict):
person = {
    "name": "John",
    "age": 30,
    "city": "New York"
}

# convert into JSON:
y = json.dumps(person)

# the result is a JSON string:
print(y)
```

### Formatting the Output

The `json.dumps()` method has parameters to make the JSON string more readable.

- `indent`: Defines the number of spaces to use for indentation.
- `separators`: Defines the separators to use. The default is `(", ", ": ")`.
- `sort_keys`: Specifies if the result should be sorted or not.

```python
# Format the output with an indent of 4 and sorted keys
y = json.dumps(person, indent=4, sort_keys=True)
print(y)
```

### Conclusion

The `json` module in Python provides a simple and effective way to work with JSON data. Whether you're building a web application, working with APIs, or just need to store structured data, the `json` module is an essential tool for any Python developer. Its ability to seamlessly convert between Python objects and JSON strings makes it a pleasure to work with.
