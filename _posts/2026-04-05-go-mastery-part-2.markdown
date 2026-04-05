---
layout: post
title: "Go Mastery: A Comprehensive Guide - Part 2: Control Flow (If, Switch, Loops)"
date: 2026-04-05 21:00:00 +0545
categories: [Go, Backend]
tags: [go, golang, backend-development, control-flow]
---

# Go Mastery: A Comprehensive Guide - Part 2: Control Flow (If, Switch, Loops)

## Introduction

Control flow is how we dictate the execution path of our Go programs. Go keeps it simple, focusing on readability and maintainability.

## If/Else Statements

In Go, there are no parentheses around conditions, but curly braces are mandatory.

```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else {
    fmt.Println("x is 10 or less")
}
```

## Switch Statement

The `switch` statement in Go is powerful and flexible. It breaks automatically, so you don't need `break` at the end of each `case`.

```go
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X.")
case "linux":
    fmt.Println("Linux.")
default:
    fmt.Printf("%s.\n", os)
}
```

## For Loop

In Go, there is only one looping construct: `for`.

### Standard Loop

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

### While Loop (equivalent)

```go
for n < 10 {
    n++
}
```

### Infinite Loop

```go
for {
    // Infinite loop
}
```

## Conclusion

We have mastered the core control flow constructs of Go. These simple tools allow you to build complex logic with clarity. In the next part, we will look into data structures like arrays, slices, and maps.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Go Documentation: Control Flow](https://go.dev/doc/effective_go#control-structures)
