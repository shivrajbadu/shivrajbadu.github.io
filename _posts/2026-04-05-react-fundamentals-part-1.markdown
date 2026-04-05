---
layout: post
title: "React Fundamentals: A Modern Guide - Part 1: Getting Started"
date: 2026-04-05 21:05:00 +0545
categories: [React, Frontend]
tags: [react, javascript, frontend-development]
---

# React Fundamentals: A Modern Guide - Part 1: Getting Started

## Introduction

React has transformed how we build web interfaces. By focusing on components, a declarative syntax, and a robust ecosystem, it allows developers to create complex, interactive user interfaces with predictability and ease. Whether you are coming from a traditional backend background or are new to modern JavaScript frameworks, understanding React's core philosophy is essential for building scalable applications.

In this series, we will explore the fundamentals of React, mirroring the core concepts from the official documentation, and providing clear, actionable examples to get you up and running.

## What is React?

At its heart, React is a JavaScript library for building user interfaces. Instead of manually manipulating the DOM to update what the user sees, React allows you to define how your UI should look based on the current state of your application. When that state changes, React efficiently updates the DOM to reflect those changes.

### The Component-Based Architecture

The building block of a React application is the **Component**. Components are independent, reusable pieces of UI that hold their own logic and styling. Think of a webpage as a composition of smaller, modular components like a `Navbar`, `Sidebar`, `Button`, and `Footer`.

## Setting Up Your Environment

Before diving into code, ensure you have [Node.js](https://nodejs.org/) installed. You can quickly scaffold a new project using Vite, which is the current industry standard for fast development.

```bash
# Create a new React project using Vite
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

Once running, you will have a local development server typically available at `http://localhost:5173`.

## JSX: JavaScript XML

React uses **JSX**, a syntax extension that allows you to write HTML-like structures directly within your JavaScript code. This makes the code more intuitive, as the structure of your UI is visually coupled with its logic.

```jsx
// A simple React component using JSX
function Welcome() {
  const name = "Developer";
  
  return (
    <div className="welcome-container">
      <h1>Hello, {name}!</h1>
      <p>Welcome to our React tutorial series.</p>
    </div>
  );
}
```

Key things to note:
*   We use `className` instead of `class`, as `class` is a reserved keyword in JavaScript.
*   You can embed any JavaScript expression within curly braces `{}`.

## Components and Props

Components are either **Function Components** (the modern standard) or **Class Components** (older, but still widely used). A component can receive data via **props** (short for properties), which are read-only arguments passed from parent to child.

```jsx
// Greeting component receiving props
function Greeting(props) {
  return <h2>Hello, {props.name}!</h2>;
}

// App component using Greeting
function App() {
  return (
    <div>
      <Greeting name="Alice" />
      <Greeting name="Bob" />
    </div>
  );
}
```

Props allow you to make your components reusable, as demonstrated above where the same `Greeting` component renders different content based on the `name` prop.

## Conclusion

In this first part, we have set the stage by understanding what React is, how it utilizes a component-based architecture, and the basics of JSX and props. These building blocks are the foundation upon which more advanced features like State and Hooks are built.

In the next installment, we will dive deeper into **State Management** and how to make your components truly interactive.

{% include inarticle-adsense.html %}

## Suggested Reading

*   [Official React Documentation](https://react.dev/)
*   [Vite Documentation](https://vite.dev/)
*   [MDN Web Docs: JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
