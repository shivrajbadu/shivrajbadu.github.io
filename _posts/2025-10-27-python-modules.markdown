---
layout: post
title: "A Quick Note on Python Modules"
date: 2025-10-27 07:00:00 +0545
categories: [Python, Basics]
---

In Python, a module is a file containing Python definitions and statements. The file name is the module name with the suffix `.py` appended. Modules allow you to organize your code into logical, reusable units.

### Creating a Module

To create a module, you just save the code you want in a file with a `.py` extension. For example, let's create a file named `my_module.py`:

```python
# my_module.py

def greeting(name):
    print("Hello, " + name)

person = {
    "name": "Shivraj",
    "age": 30
}
```

### Using a Module

You can use the `import` statement to use the functions and variables defined in another module.

```python
# main.py
import my_module

my_module.greeting("World")
print(my_module.person["age"])
```

### Renaming a Module

You can create an alias when you import a module, by using the `as` keyword.

```python
# main.py
import my_module as mm

mm.greeting("World")
```

### Importing From a Module

You can choose to import only parts from a module, by using the `from` keyword.

```python
# main.py
from my_module import person

print(person["name"])
```

You can also import all names from a module using `*`, but this is generally discouraged as it can make your code less readable.

```python
# main.py
from my_module import *

greeting("Everyone")
```

### Conclusion

Modules are a fundamental part of Python programming that help you keep your code organized, reusable, and maintainable. By breaking your code into smaller, logical modules, you can build more complex and scalable applications. Python's extensive standard library is a great example of the power of modules, providing a vast collection of pre-built functionality that you can use in your own programs.
