---
layout: post
title: "JSON Data Processor: Reading and Writing API Data in Go"
date: 2026-04-05 11:00:00 +0545
categories: [Go, Backend]
tags: [go, golang, backend-development, json]
---

# JSON Data Processor: Reading and Writing API Data in Go

## Introduction

Working with JSON is a daily task in backend development. Whether you are consuming external APIs or building your own, Go's `encoding/json` package provides a robust and performant way to marshal (write) and unmarshal (read) JSON data.

## 1. Unmarshalling JSON (Reading)

To read JSON data, we define a struct with `json` tags that map the JSON keys to struct fields.

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func main() {
    jsonData := []byte(`{"id": 1, "name": "Alice", "email": "alice@example.com"}`)

    var user User
    err := json.Unmarshal(jsonData, &user)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Printf("User: %+v\n", user)
}
```

## 2. Marshalling JSON (Writing)

To turn a Go struct back into JSON, we use `json.Marshal` or `json.MarshalIndent`.

```go
func main() {
    user := User{ID: 2, Name: "Bob", Email: "bob@example.com"}

    jsonData, err := json.MarshalIndent(user, "", "  ")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Println(string(jsonData))
}
```

## Conclusion

Go’s `encoding/json` makes data interchange simple and type-safe. By properly tagging your structs, you ensure your API responses and requests are consistently formatted, which is essential for any modern web application.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Go Documentation: encoding/json](https://pkg.go.dev/encoding/json)
