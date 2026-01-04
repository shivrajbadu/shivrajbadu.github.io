---
layout: post
title: "LangChain Series: Advanced Agent Control with Callbacks"
date: 2026-01-04 11:25:00 +0545
categories: [AI, Machine Learning, LangChain]
tags: [langchain, agents, deepagents, python, callbacks]
---

# LangChain Series: Advanced Agent Control with Callbacks

## Introduction

In our [previous post](/2026/01/04/building-a-basic-langchain-agent.html), we built our first agent and used the `verbose=True` flag to watch its reasoning process. While this is great for debugging, production applications require a more robust way to programmatically log, monitor, and interact with an agent's lifecycle.

This is where LangChain's **Callback system** comes in. Callbacks provide a powerful mechanism to "hook into" the various stages of an agent's or chain's execution. They are the key to building observable, controllable, and production-ready AI systems. This post will show you how to create and use a custom callback handler to gain fine-grained control over your agent.

## The Agent Lifecycle and Callbacks

When an agent runs, it doesn't just happen in one step. It's a sequence of discrete events:
-   The agent executor starts.
-   The LLM is called to decide on an action.
-   A tool is called.
-   The tool returns a result.
-   The LLM is called again to decide the next step.
-   The agent finishes.

The callback system allows you to register functions that will be executed automatically whenever one of these events occurs. This is incredibly useful for:
-   **Logging and Monitoring:** Send detailed logs about tool usage, LLM calls, and errors to platforms like LangSmith, Datadog, or a simple file.
-   **Streaming:** Stream the output of the LLM or the agent's thoughts back to a user interface in real-time.
-   **Cost and Token Tracking:** Calculate the cost of each LLM call and count token usage.
-   **Input/Output Validation:** Inspect and validate the inputs to tools or the outputs from the LLM.

## Implementing a Custom Callback Handler

The easiest way to use callbacks is to create a Python class that inherits from `BaseCallbackHandler` and implements the methods corresponding to the events you want to handle. The method names are standardized (e.g., `on_agent_start`, `on_tool_end`).

Let's create a simple handler that logs the key steps of our agent's journey to the console.

-   `on_agent_start`: Fires when the agent's `invoke` method is called.
-   `on_chat_model_start`: Fires just before the LLM is called. We can inspect the exact messages being sent.
-   `on_tool_start`: Fires before a tool is executed, letting us see the tool's name and input.
-   `on_tool_end`: Fires after a tool runs, letting us see its output.
-   `on_agent_finish`: Fires when the agent returns its final response.

## Code Example: A Logging Callback Handler

We'll use the same agent from our last post but attach our new custom callback handler to it.

```python
import os
from typing import Any, Dict, List
from uuid import UUID

from dotenv import load_dotenv
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain.callbacks.base import BaseCallbackHandler
from langchain.prompts import ChatPromptTemplate
from langchain import hub
from langchain.tools import tool
from langchain_openai import ChatOpenAI

# Load environment variables
load_dotenv()

# 1. Define our custom tool
@tool
def get_weather(city: str) -> str:
    """Returns the weather for a given city."""
    if "san francisco" in city.lower() or "sf" in city.lower():
        return "It is currently 65°F and foggy in San Francisco."
    return f"It's always sunny in {city}!"

# 2. Create our Custom Callback Handler
class MyCustomHandler(BaseCallbackHandler):
    def on_agent_start(self, serialized: Dict[str, Any], **kwargs: Any) -> Any:
        print("--- Agent Started ---")

    def on_chat_model_start(
        self,
        serialized: Dict[str, Any],
        messages: List[List[Any]],
        **kwargs: Any,
    ) -> Any:
        print("\n--- Calling LLM ---")
        # Print the user's message
        print(f"User message: {messages[0][0].content}")

    def on_tool_start(
        self,
        serialized: Dict[str, Any],
        input_str: str,
        **kwargs: Any,
    ) -> Any:
        print("\n--- Calling Tool ---")
        print(f"Tool: {serialized['name']}")
        print(f"Tool Input: {input_str}")

    def on_tool_end(self, output: str, **kwargs: Any) -> Any:
        print("\n--- Tool End ---")
        print(f"Tool Output: {output}")

    def on_agent_finish(self, finish: Any, **kwargs: Any) -> Any:
        print("\n--- Agent Finished ---")
        print(f"Final Output: {finish.return_values['output']}")
        print("--------------------")

# 3. Set up the Agent (same as before)
tools = [get_weather]
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
prompt = hub.pull("hwchase17/openai-functions-agent")
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

# 4. Run the Agent with the Callback Handler
print("Running agent with custom callback handler...")
response = agent_executor.invoke(
    {"input": "What is the weather like in San Francisco?"},
    {"callbacks": [MyCustomHandler()]}
)
```

When you run this code, instead of the default `verbose` output, you will see your custom log messages, giving you structured insight into the agent's execution flow.

## Other Important LangChain Concepts

You brought up a few other terms, and it's worth clarifying where they fit in.

-   **`init_chat_model`**: This isn't a specific LangChain function but a general term for the process of **initializing a chat model**. In our examples, `llm = ChatOpenAI(...)` is how we initialize the OpenAI chat model. You would do something similar for other providers, like `llm = ChatGoogleGenerativeAI(...)`.
-   **`add_texts`**: This is a common method found on **Vector Store** objects (like those from Chroma, FAISS, or Pinecone). Its job is to take a list of documents (texts), create vector embeddings for them, and add them to the vector database. It's a crucial function for the "indexing" stage of a RAG pipeline, where you are preparing your data to be searched.

## Conclusion

Callbacks are the bridge from simple LangChain prototypes to robust, production-grade applications. They provide the essential observability and control needed to log, monitor, and debug complex agentic systems. By mastering callback handlers, you gain a much deeper level of control over how your agents behave.

This concludes our initial series on LangChain. We've gone from the basic concepts to building chains and agents, and finally to controlling them. The LangChain framework is deep and constantly evolving, so we encourage you to continue exploring its powerful features.

{% include inarticle-adsense.html %}
