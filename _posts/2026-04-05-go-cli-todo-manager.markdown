---
layout: post
title: "Building a CLI Tool in Go: A Simple Todo List Manager"
date: 2026-04-05 13:00:00 +0545
categories: [Go, Backend]
tags: [go, golang, backend-development, cli, tools]
---

# Building a CLI Tool in Go: A Simple Todo List Manager

## Introduction

Command-line tools are a staple in a developer's workflow. Go is exceptionally well-suited for building fast, portable, and reliable CLI tools. In this post, we'll build a basic Todo List Manager that allows you to add and list tasks directly from your terminal.

## The CLI Tool Implementation

We will use the `os` package to handle command-line arguments and a simple text file for storage.

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Usage: todo [add|list] [task]")
        return
    }

    command := os.Args[1]

    switch command {
    case "add":
        if len(os.Args) < 3 {
            fmt.Println("Please provide a task description.")
            return
        }
        addTask(os.Args[2])
    case "list":
        listTasks()
    default:
        fmt.Println("Unknown command:", command)
    }
}

func addTask(task string) {
    f, _ := os.OpenFile("tasks.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
    defer f.Close()
    f.WriteString(task + "\n")
    fmt.Println("Task added.")
}

func listTasks() {
    data, _ := os.ReadFile("tasks.txt")
    fmt.Print(string(data))
}
```

## How to use it

After compiling your code with `go build -o todo main.go`, you can use it in your terminal:

```bash
./todo add "Learn Go CLI"
./todo list
```

## Why Go for CLI?

- **Single Executable**: Go compiles your code into a single binary, making it easy to distribute without dependencies.
- **Fast Startup**: Go programs start almost instantaneously, which is critical for a smooth CLI experience.
- **Rich Standard Library**: Packages like `os`, `flag`, and `fmt` provide everything needed to interact with the system, parse flags, and output text.

## Conclusion

Building your own CLI tools in Go is a rewarding experience. It not only helps you automate your daily tasks but also deepens your understanding of system-level programming in Go.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Go Documentation: os package](https://pkg.go.dev/os)
- [Go Documentation: flag package](https://pkg.go.dev/flag)
