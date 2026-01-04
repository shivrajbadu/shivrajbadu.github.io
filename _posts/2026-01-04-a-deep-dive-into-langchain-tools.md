---
layout: post
title: "LangChain Series: A Deep Dive into Tools"
date: 2026-01-04 11:30:00 +0545
categories: [AI, Machine Learning, LangChain]
tags: [langchain, agents, tools, python]
---

# LangChain Series: A Deep Dive into Tools

## Introduction

In our previous posts, we introduced LangChain and built our first agent. We learned that agents are powerful because they can use **tools** to interact with the outside world. A well-built agent is only as good as the tools it has access to. Therefore, understanding how to create and configure tools is one of the most critical skills in the LangChain ecosystem.

This post will be a deep dive into creating LangChain tools. We'll focus on the simplest and most common method: the `@tool` decorator. We will explore how to go from a basic Python function to a fully configured tool that an agent can reliably use.

## What is a LangChain Tool?

At its core, a LangChain tool is just a Python function that has been made available to an LLM. The magic lies in how the function is described to the model. For an agent to use your function effectively, it needs to know three things:

1.  **Name:** A clear, descriptive name for the tool.
2.  **Description:** A detailed explanation of what the tool does, what it's good for, and when to use it. This is the most important part, as the LLM uses this description to make its decisions.
3.  **Arguments:** The inputs the function needs, including their names and data types.

LangChain takes this information and formats it into a schema (like a JSON Schema) that it presents to the LLM. When the LLM decides to use a tool, it formats its output to match this schema, which LangChain then parses to execute the correct function with the correct arguments.

## Creating Your First Tool with `@tool`

The easiest way to create a tool is to take a standard Python function and add the `@tool` decorator. LangChain will automatically **infer the schema** from the function's name, docstring, and type hints.

Let's start with a simple function:

```python
# This is just a regular Python function
def search_api(query: str) -> str:
    """Searches a private API for the given query and returns the results."""
    # In a real application, this would make an API call.
    return f"Results for '{query}' from the private API."
```

To turn this into a tool, we just add the decorator:

```python
from langchain.tools import tool

@tool
def search_api(query: str) -> str:
    """Searches a private API for the given query and returns the results."""
    # In a real application, this would make an API call.
    return f"Results for '{query}' from the private API."
```

By default, LangChain now understands this tool as:
-   **Name:** `search_api` (from the function name)
-   **Description:** "Searches a private API for the given query and returns the results." (from the docstring)
-   **Arguments:** A single required argument named `query` of type `string` (from the type hint).

## Customizing Tools with Decorator Arguments

The `@tool` decorator is highly configurable, allowing you to override the defaults and add more advanced behavior. Let's explore its key parameters.

### Customizing Name and `return_direct`

Sometimes, the function name isn't descriptive enough for the LLM, or you want the agent to immediately return the tool's output without further thought.

```python
@tool("private_api_search", return_direct=True)
def search_api(query: str) -> str:
    """Searches a private API for the given query and returns the results."""
    return f"Results for '{query}' from the private API."
```

In this example:
-   `"private_api_search"`: We've explicitly named the tool. The LLM will now see it as `private_api_search` instead of `search_api`.
-   `return_direct=True`: This is a powerful flag. It tells the agent that if it uses this tool, it should immediately stop its execution loop and return the tool's output directly to the user. This is perfect for tools like search, where the result is the final answer. It saves you an extra, often unnecessary, LLM call.

### Defining Complex Arguments with `args_schema`

Type hints work well for simple arguments, but what if you need more complex validation, like restricting a value to a specific set of choices? You can use a Pydantic model with the `args_schema` parameter.

```python
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(description="The search query.")
    state: str = Field(description="The state of the items to search for.", enum=["open", "closed"])

@tool(args_schema=SearchInput)
def search_issues(query: str, state: str) -> str:
    """Searches for issues with a specific state."""
    return f"Found issues for '{query}' with state '{state}'."
```
Now, the LLM knows that the `state` argument is not just any string; it must be either `"open"` or `"closed"`, leading to more reliable tool calls.

### Advanced Return Values with `response_format`

By default, a tool is expected to return a single string. However, sometimes you want to return both a human-readable summary for the LLM and a structured object (like a JSON blob) for further processing in your application. The `response_format` parameter enables this.

```python
from typing import Tuple, Dict, Any

@tool(response_format="content_and_artifact")
def search_api_with_artifact(query: str) -> Tuple[str, Dict[str, Any]]:
    """Searches the API and returns both a summary and a full JSON object."""
    full_results = {"count": 2, "items": [{"id": 1, "title": "First"}, {"id": 2, "title": "Second"}]}
    summary = "Found 2 items: First, Second"
    
    # The first element of the tuple is the 'content' (for the LLM)
    # The second is the 'artifact' (for your application)
    return summary, full_results
```
When an agent uses this tool, it will see the `summary` string as the tool's output in its reasoning loop. However, the full `full_results` dictionary is also preserved and can be accessed from the final result of the agent run, allowing you to use this rich, structured data in your application's frontend or backend.

## Conclusion

The `@tool` decorator is the gateway to empowering your LangChain agents. By moving beyond the basics and using its configuration options, you can create robust, reliable, and predictable tools. Remember, the quality of your tools—especially their descriptions—is the single most important factor in determining your agent's performance.

In future posts, we will explore more advanced tool-related topics, such as creating tools from existing classes (`BaseTool`) and bundling them into toolkits.

{% include inarticle-adsense.html %}
