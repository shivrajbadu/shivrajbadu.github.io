---
layout: post
title: "React Fundamentals: A Modern Guide - Part 2: State and Hooks"
date: 2026-04-05 21:00:00 +0545
categories: [React, Frontend]
tags: [react, javascript, frontend-development, hooks]
---

# React Fundamentals: A Modern Guide - Part 2: State and Hooks

## Introduction

In the first part of this series, we covered the basics of React, components, JSX, and props. We established that components are the building blocks of any React application, but to make those components truly useful, they need to be interactive. This is where **State** comes in.

## What is State?

State is an object that represents the part of a component that can change over time. When state changes, React automatically re-renders the component to reflect the new state in the user interface.

## The `useState` Hook

In modern React, we manage state using the `useState` hook. Hooks are special functions that allow you to "hook into" React features like state and lifecycle from function components.

```jsx
import React, { useState } from "react";

function Counter() {
  // Declare a state variable named 'count' initialized to 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

Key takeaways:

- `useState` returns two values: the current state (`count`) and a function to update it (`setCount`).
- When `setCount` is called, React schedules a re-render of the component with the new value.

## The `useEffect` Hook

Often, components need to perform "side effects"—data fetching, manual DOM manipulations, or setting up subscriptions. The `useEffect` hook allows you to perform these side effects in function components.

```jsx
import React, { useState, useEffect } from "react";

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Perform side effect: fetching data
    fetch("https://api.example.com/data")
      .then((response) => response.json())
      .then((json) => setData(json));
  }, []); // Empty dependency array means this runs only once on mount

  return <div>{data ? JSON.stringify(data) : "Loading..."}</div>;
}
```

The dependency array (the second argument to `useEffect`) determines when the effect runs:

- `[]`: Runs only once when the component mounts.
- `[prop, state]`: Runs whenever `prop` or `state` changes.
- No array: Runs after every render.

## Conclusion

By mastering `useState` and `useEffect`, you have the power to create dynamic, interactive, and functional React applications. These two hooks alone handle the vast majority of use cases in modern React development.

In the next installment, we will explore advanced state management techniques, including Context API and managing complex state logic.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Official React Documentation: Hooks](https://react.dev/reference/react)
- [React Hooks FAQ](https://react.dev/warnings/react-hooks-warning)
