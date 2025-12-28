---
layout: post
title: "FastAPI: Handling Request Bodies and Data Validation"
date: 2025-12-28 12:00:00 +0545
categories: [FastAPI, Python]
tags: [fastapi, python, pydantic, validation, request-body]
---

# FastAPI: Handling Request Bodies and Data Validation

## Introduction

While path and query parameters are great for simple data, most `POST`, `PUT`, and `PATCH` operations require richer data structures sent in the request body. FastAPI uses Pydantic models to define, validate, and document these structures, making it incredibly easy to work with complex JSON data.

## Using Pydantic for Request Bodies

Pydantic is a data validation library that FastAPI is built upon. You define the "shape" of your data as a class that inherits from `BaseModel`.

Let's create a simple model for an item:

```python
from typing import Optional
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

app = FastAPI()

@app.post("/items/")
def create_item(item: Item):
    return item
```

When you send a `POST` request to `/items/` with a JSON body, FastAPI will:
1.  Parse the JSON.
2.  Validate that it matches the `Item` model (e.g., `name` is a string, `price` is a float).
3.  Convert the data into an `Item` object, available as the `item` argument.
4.  Automatically document the expected request body in the API docs.

## A More Complex Example: Articles, Authors, and Comments

Real-world applications often involve nested data structures and relationships. Let's model a scenario with articles, authors, and comments.

- An `Article` has a title, description, and active status.
- It has one `Author`, with a name and address.
- It has many `Comment`s, each with comment text and an active status.

We can define these relationships using nested Pydantic models.

### 1. Define the Models

Create a new file `schemas.py` to hold your Pydantic models:

```python
# schemas.py
from typing import List, Optional
from pydantic import BaseModel

class Author(BaseModel):
    name: str
    address: str

class Comment(BaseModel):
    comment: str
    active: bool

class Article(BaseModel):
    title: str
    description: str
    active: bool
    author: Author
    comments: List[Comment]
```

- `Author` and `Comment` are simple models.
- The `Article` model includes an `author` field of type `Author` and a `comments` field which is a `List` of `Comment` models.

### 2. Use the Models in Your API

Now, let's use these models in your main application file.

```python
# main.py
from typing import List
from fastapi import FastAPI
from schemas import Article, Author, Comment

app = FastAPI()

# In-memory "database" for demonstration
db_articles = []

@app.post("/articles/")
def create_article(article: Article):
    """
    Create a new article with its author and comments.
    """
    db_articles.append(article.dict())
    return article

@app.get("/articles/", response_model=List[Article])
def get_articles():
    """
    Retrieve all articles.
    """
    return db_articles
```

- The `POST /articles/` endpoint expects a JSON object matching the `Article` model. FastAPI handles all the validation of the nested structures.
- The `GET /articles/` endpoint returns a list of articles. The `response_model` argument ensures the output matches the `List[Article]` structure and documents it.

### Example `POST` Request Body

Here’s what a valid JSON body for a `POST` request to `/articles/` would look like:

```json
{
  "title": "Understanding FastAPI",
  "description": "A deep dive into the features of FastAPI.",
  "active": true,
  "author": {
    "name": "Shivraj Badu",
    "address": "Kathmandu, Nepal"
  },
  "comments": [
    {
      "comment": "This is a great article!",
      "active": true
    },
    {
      "comment": "Very informative, thank you.",
      "active": true
    }
  ]
}
```

## Conclusion

Pydantic models are the cornerstone of FastAPI's power and simplicity. They provide a clear, declarative way to define complex data structures, handle validation automatically, and generate excellent documentation. This allows you to focus on your application's logic instead of writing boilerplate data parsing and validation code.

Next, we'll explore how to structure larger applications to keep your code organized and maintainable.

## Suggested Reading

-   [FastAPI Documentation: Request Body](https://fastapi.tiangolo.com/tutorial/body/)
-   [FastAPI Documentation: Body - Nested Models](https://fastapi.tiangolo.com/tutorial/body-nested-models/)
-   [Pydantic Documentation](https://pydantic-docs.helpmanual.io/)

{% include inarticle-adsense.html %}
