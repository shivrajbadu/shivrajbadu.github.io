---
layout: post
title: "Building a REST API in Go: A Task Management CRUD Application"
date: 2026-04-05 10:00:00 +0545
categories: [Go, Backend]
tags: [go, golang, backend-development, rest-api, crud]
---

# Building a REST API in Go: A Task Management CRUD Application

## Introduction

Building a REST API is a classic rite of passage for any backend developer. In this post, we will build a simple Task Management API that supports Create, Read, Update, and Delete (CRUD) operations using Go's built-in `net/http` package and standard library.

## The Data Model

First, let's define our `Task` struct.

```go
type Task struct {
    ID    string `json:"id"`
    Title string `json:"title"`
    Done  bool   `json:"done"`
}

var tasks = []Task{
    {ID: "1", Title: "Learn Go", Done: false},
    {ID: "2", Title: "Build an API", Done: false},
}
```

## The Implementation

We will create handlers for each HTTP verb: `GET` (read), `POST` (create), `PUT` (update), and `DELETE`.

```go
package main

import (
    "encoding/json"
    "net/http"
)

// Simplified handler for reading tasks
func getTasks(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(tasks)
}

func main() {
    http.HandleFunc("/tasks", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet:
            getTasks(w, r)
        // Add cases for POST, PUT, DELETE here
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })

    http.ListenAndServe(":8080", nil)
}
```

## Why This Matters

By using standard library tools, you gain a deep understanding of how HTTP requests are handled in Go without the overhead of heavy frameworks. This is a foundational step toward building robust, scalable backend services.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Go Documentation: net/http](https://pkg.go.dev/net/http)
- [Go Documentation: encoding/json](https://pkg.go.dev/encoding/json)
