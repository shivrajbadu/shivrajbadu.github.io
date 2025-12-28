---
layout: post
title: "Handling Path and Query Parameters in FastAPI"
date: 2025-12-28 11:00:00 +0545
categories: [FastAPI, Python]
tags: [fastapi, python, path-parameters, query-parameters]
---

# Handling Path and Query Parameters in FastAPI

## Introduction

In our previous post, we created a basic FastAPI application. Now, let's learn how to handle dynamic inputs from the URL using path and query parameters. This is fundamental for building APIs that can respond to user-specific requests.

## Path Parameters

Path parameters are parts of the URL path that are variable. They are specified using curly braces `{}` and are passed as arguments to your path operation function.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

In this example, `item_id` is a path parameter. FastAPI uses the type hint (`int`) to validate and convert the incoming value. If you visit `http://127.0.0.1:8000/items/5`, the response will be `{"item_id": 5}`. If you provide a non-integer, FastAPI will return a clear validation error.

## Query Parameters

Query parameters are key-value pairs that appear at the end of a URL after a `?`. They are used for filtering, sorting, or pagination.

Any function parameter that is not part of the path is automatically interpreted as a query parameter.

```python
from fastapi import FastAPI

app = FastAPI()

# Example data
fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

If you go to `http://127.0.0.1:8000/items/?skip=0&limit=2`, you'll get the first two items. Since `skip` and `limit` have default values, you can also call `/items/` without any parameters.

## Optional Query Parameters

You can make query parameters optional by providing a default value of `None` or by using Python's `Optional` type.

```python
from typing import Optional
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: str, q: Optional[str] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

Here, `q` is an optional query parameter. A request to `/items/foo?q=searchquery` will include `q` in the response.

## Advanced Query Parameter Validation

For more complex validation rules (like length constraints or regular expressions), you can use the `Query` function.

```python
from typing import Optional
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
def read_items(
    q: Optional[str] = Query(
        None,
        min_length=3,
        max_length=50,
        regex="^fixedquery$"
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

This ensures that the optional parameter `q`, if provided, must have a length between 3 and 50 characters and match the regular expression.

## Conclusion

Path and query parameters are essential for creating dynamic and interactive APIs. FastAPI's use of type hints and the `Query` function makes handling them simple, robust, and self-documenting.

In our next post, we'll dive into handling request bodies and using Pydantic for complex data validation.

## Suggested Reading

-   [FastAPI Documentation: Path Parameters](https://fastapi.tiangolo.com/tutorial/path-params/)
-   [FastAPI Documentation: Query Parameters](https://fastapi.tiangolo.com/tutorial/query-params/)

{% include inarticle-adsense.html %}
