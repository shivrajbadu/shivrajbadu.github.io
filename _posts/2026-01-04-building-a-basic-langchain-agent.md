---
layout: post
title: "LangChain Series: Building Your First Agent"
date: 2026-01-04 11:20:00 +0545
categories: [AI, Machine Learning, LangChain]
tags: [langchain, agents, ai-development, python]
---

# LangChain Series: Building Your First Agent

## Introduction

In our [previous post](/2026/01/04/introduction-to-langchain.html), we introduced the fundamentals of LangChain and built a simple "chain." Chains are powerful, but they follow a predetermined path. What if you need the LLM to make decisions, choose actions, and interact with the outside world? For that, you need an **Agent**.

An agent uses an LLM not just to generate text, but as a reasoning engine. It can analyze a request, decide which tools to use (if any), execute them, observe the results, and repeat this process until it has a final answer. This post will guide you through building your first, simple agent that can use a custom tool to answer questions.

## What is a LangChain Agent?

Think of an agent as a loop that follows the **ReAct (Reason + Act)** framework:
1.  **Reason:** Based on the user's input, the agent's LLM "thinks" about what to do next. Does it have enough information to answer? Does it need to use a tool?
2.  **Act:** If it decides to use a tool, it specifies the tool and the input for it.
3.  **Observe:** The agent executes the tool and gets a result (an "observation").
4.  **Repeat:** The agent takes this observation and feeds it back into its reasoning process, repeating the loop until it's ready to give the user a final answer.

To build an agent, you need three key components:
-   **LLM:** The "brain" of the agent that makes all the decisions.
-   **Tools:** Functions the agent can call to get information or perform actions. These can be anything from a Google search API to a database query function or a simple calculator.
-   **Prompt:** A specialized system prompt that tells the LLM how to behave. It explains the reasoning process, what tools are available, and how to format its responses so the framework can parse them.

## Building a Basic Agent: A Code Walkthrough

Let's build an agent that has access to a custom tool: a simple function to get the weather.

### Step 1: Installation

If you followed the last post, you should have the necessary libraries. If not, install them now. We'll also add `langchainhub` to pull a pre-built prompt.

```bash
pip install langchain langchain-openai python-dotenv langchainhub
```

Don't forget to set up your `.env` file with your `OPENAI_API_KEY`.

### Step 2: Define a Tool

A tool is just a Python function with a clear docstring and type hints. LangChain uses the `@tool` decorator to easily convert a function into a format the agent can use. The docstring is crucial—the LLM reads it to understand what the tool does and when to use it.

```python
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """Returns the weather for a given city."""
    # This is a dummy function for demonstration
    if "san francisco" in city.lower() or "sf" in city.lower():
        return "It is currently 65°F and foggy in San Francisco."
    return f"It's always sunny in {city}!"

# The agent will have access to this tool
tools = [get_weather]
```

### Step 3: Create the Agent

Creating an agent involves a few components: a prompt, the LLM, and the tools. We'll use a pre-built prompt from LangChain Hub that is specifically designed for this type of agent.

```python
from langchain_openai import ChatOpenAI
from langchain import hub
from langchain.agents import create_openai_functions_agent, AgentExecutor

# 1. Initialize the LLM
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# 2. Get the prompt template
# This prompt is designed to work with OpenAI's functions calling feature
prompt = hub.pull("hwchase17/openai-functions-agent")

# 3. Create the agent
# This combines the LLM, prompt, and tools into a runnable agent
agent = create_openai_functions_agent(llm, tools, prompt)

# 4. Create the Agent Executor
# This is the runtime for the agent. It's what actually invokes the agent and executes tools.
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```
The `verbose=True` argument is highly recommended when you're starting out. It prints the agent's entire reasoning process to the console, so you can see exactly what it's thinking.

### Step 4: Run the Agent

Now, let's run our agent and ask it a question that requires using our custom tool.

```python
# Run the agent with a user query
response = agent_executor.invoke(
    {"input": "What is the weather like in San Francisco?"}
)

print("\nFinal Answer:")
print(response["output"])
```

When you run this, the `verbose=True` output will show you something like this:

```
> Entering new AgentExecutor chain...

Invoking: `get_weather` with `{'city': 'San Francisco'}`
It is currently 65°F and foggy in San Francisco.
The weather in San Francisco is currently 65°F and foggy.

> Finished chain.

Final Answer:
The weather in San Francisco is currently 65°F and foggy.
```

This output clearly shows the agent's thought process: it correctly identified that it needed to call the `get_weather` tool with the city "San Francisco," got the result, and then formulated a final answer.

## Conclusion

Agents are a fundamental leap beyond simple chains. They provide LLMs with the ability to reason, strategize, and interact with their environment through tools. This simple example is just the beginning. By creating more sophisticated tools and prompts, you can build powerful agents capable of tackling complex, multi-step problems.

In our next post, we will explore how to gain more fine-grained control and observability over your agents using **Callbacks**, a crucial feature for building production-ready applications.

{% include inarticle-adsense.html %}
