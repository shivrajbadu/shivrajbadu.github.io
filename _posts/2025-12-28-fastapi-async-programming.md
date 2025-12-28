---
layout: post
title: "Async Programming in FastAPI: `async def` vs `def`"
date: 2025-12-28 13:00:00 +0545
categories: [FastAPI, Python]
tags: [fastapi, python, async, asynchronous, performance]
---

# Async Programming in FastAPI: `async def` vs `def`

## Introduction

One of FastAPI's core strengths is its native support for asynchronous programming using Python's `async` and `await` keywords. Understanding when and how to use `async` functions is key to unlocking the high performance that makes FastAPI so popular. This post clarifies the difference between `async def` and normal `def` endpoints.

## What is Async?

Asynchronous code allows a program to pause a task that is waiting for something (like a network request to complete) and work on another task in the meantime. This is especially useful for web servers, which spend a lot of time waiting for I/O operations (e.g., reading from a database, calling another API, or reading a file).

## How FastAPI Handles Endpoints

FastAPI is smart about how it runs your endpoint functions:

-   **`async def` functions:** If you declare your path operation function with `async def`, FastAPI will run it directly on the main event loop. This means it can handle many concurrent requests efficiently, as long as you don't use any "blocking" I/O calls.

-   **`def` functions:** If you use a normal `def` function, FastAPI will run it in an external thread pool and `await` it. This prevents a long-running, blocking function from freezing the main event loop.

## When to Use `async def`

Use `async def` for I/O-bound tasks. This is when your code has to wait for an external resource. To get the benefits, you must use an `async` compatible library for the I/O operation.

**Use `async def` when you need to `await`:**
-   Database queries (with an async driver like `asyncpg` or `motor`).
-   Requests to external APIs (with a library like `httpx`).
-   Reading/writing to files or the network (with `aiofiles`).

### Example: Calling an External API

Here, we use `httpx` to make an asynchronous call to an external API.

```python
import asyncio
import httpx
from fastapi import FastAPI

app = FastAPI()

@app.get("/weather/{city}")
async def get_weather(city: str):
    # This part doesn't block the server while waiting for the response
    async with httpx.AsyncClient() as client:
        # Simulate a slow API call
        await asyncio.sleep(1)
        # In a real app, you'd call an external service
        # response = await client.get(f"https://api.weather.com/{city}")
        # return response.json()
        return {"city": city, "temperature": "25°C"}
```

While this endpoint is waiting for `asyncio.sleep()` or the `httpx` request, the server can process other requests.

## When to Use `def`

Use a normal `def` for CPU-bound tasks. These are operations that perform heavy computations. Running them in a separate thread prevents them from blocking the event loop, which would make your entire application unresponsive.

**Use `def` for:**
-   Processing a large file in memory.
-   Performing complex mathematical or data science calculations.
-   Any operation that uses a library that makes blocking I/O calls (like the standard `requests` library).

### Example: A CPU-Bound Task

This endpoint simulates a CPU-intensive calculation.

```python
import time
from fastapi import FastAPI

app = FastAPI()

def complex_calculation(data: dict):
    # Simulate a CPU-intensive task
    time.sleep(2)
    return {"result": sum(data.values())}

@app.post("/calculate/")
def calculate(data: dict):
    # FastAPI runs this in a thread pool, so it doesn't block
    result = complex_calculation(data)
    return result
```

Even though `time.sleep()` is blocking, FastAPI is smart enough to run this function in a separate thread, so other requests can still be handled concurrently.

## Conclusion

FastAPI gives you the flexibility to choose the right tool for the job.
- Use **`async def`** for I/O-bound operations with `await` to achieve high concurrency.
- Use **`def`** for CPU-bound or blocking I/O operations to keep the main event loop free.

By understanding this distinction, you can build highly efficient and scalable APIs that make the most of modern hardware and asynchronous capabilities.

## Suggested Reading

-   [FastAPI Documentation: Async](https://fastapi.tiangolo.com/async/)
-   [A-sync Python in 7 minutes](https://www.youtube.com/watch?v=E7Yn5gH5Y44)
-   [The `httpx` library](https://www.python-httpx.org/)

{% include inarticle-adsense.html %}
