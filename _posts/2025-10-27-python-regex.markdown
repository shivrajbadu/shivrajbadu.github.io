---
layout: post
title: "A Quick Note on Python RegEx"
date: 2025-10-27 10:00:00 +0545
categories: [Python, Basics]
---

A Regular Expression, or RegEx, is a sequence of characters that forms a search pattern. RegEx can be used to check if a string contains the specified search pattern. Python has a built-in package called `re`, which can be used to work with Regular Expressions.

### The `re.search()` Function

The `re.search()` function searches the string for a match, and returns a match object if there is a match. If there is more than one match, only the first occurrence of the a match will be returned.

```python
import re

txt = "The rain in Spain"
x = re.search("^The.*Spain$", txt)

if x:
    print("YES! We have a match!")
else:
    print("No match")
```

### The `re.findall()` Function

The `re.findall()` function returns a list containing all matches.

```python
import re

txt = "The rain in Spain"
x = re.findall("ai", txt)
print(x)  # ['ai', 'ai']
```

### The `re.split()` Function

The `re.split()` function returns a list where the string has been split at each match.

```python
import re

txt = "The rain in Spain"
x = re.split("\s", txt)
print(x)  # ['The', 'rain', 'in', 'Spain']
```

### The `re.sub()` Function

The `re.sub()` function replaces the matches with the text of your choice.

```python
import re

txt = "The rain in Spain"
x = re.sub("\s", "-", txt)
print(x)  # The-rain-in-Spain
```

### Metacharacters and Special Sequences

RegEx uses metacharacters (e.g., `[]`, `.`, `^`, `$`, `*`, `+`) and special sequences (e.g., `\d`, `\s`, `\w`) to define search patterns. These are what make RegEx so powerful.

- `[]`: A set of characters
- `\d`: Returns a match where the string contains digits (numbers from 0-9)
- `*`: Zero or more occurrences

```python
import re

txt = "The rain in Spain falls mainly in the plain."

#Find all lower case characters alphabetically between "a" and "m":
x = re.findall("[a-m]", txt)
print(x)
```

### Conclusion

Regular Expressions are an incredibly powerful tool for text processing, and Python's `re` module provides a comprehensive set of functions for working with them. While the syntax can be a bit intimidating at first, mastering RegEx will open up a whole new world of possibilities for searching, validating, and manipulating strings.
