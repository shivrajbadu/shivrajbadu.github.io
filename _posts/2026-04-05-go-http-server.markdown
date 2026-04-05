---
layout: post
title: "Building a Simple HTTP Web Server in Go"
date: 2026-04-05 14:00:00 +0545
categories: [Go, Backend]
tags: [go, golang, backend-development, http, web-server]
---

# Building a Simple HTTP Web Server in Go

## Introduction

One of Go's most powerful capabilities for backend development is its standard library, which includes a robust, high-performance HTTP server out of the box. In this post, we will build a simple "Hello, World!" web server to see how easily you can get a functional web application running.

## The Web Server Implementation

Go makes setting up a server straightforward using the `net/http` package.

```go
package main

import (
	"fmt"
	"net/http"
)

// handler function to return "Hello, World!"
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "<h1>Hello, World!</h1>")
}

func main() {
	// Register the handler function for the root path
	http.HandleFunc("/", handler)

	// Start the server on port 8080
	fmt.Println("Server starting at http://localhost:8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Printf("Error starting server: %s\n", err)
	}
}
```

## Understanding the Code

1.  **Handler Function**: The `handler` function processes incoming requests. It takes an `http.ResponseWriter` to send data back to the client and an `*http.Request` which contains information about the incoming request.
2.  **Registration**: `http.HandleFunc("/", handler)` tells Go to use our `handler` function for all requests coming to the root (`/`) path.
3.  **ListenAndServe**: This function starts the server on the specified port (`:8080`) and begins listening for incoming HTTP requests.

## How to run it

1. Save the code in a file named `main.go`.
2. Run the following command in your terminal:
   ```bash
   go run main.go
   ```
3. Open your web browser and navigate to `http://localhost:8080`.

## Conclusion

Go’s standard library provides everything you need to start building web servers, from basic HTTP servers to complex APIs. This "Hello World" example is just the tip of the iceberg—you can expand this to handle routing, templates, middleware, and much more.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Go Documentation: net/http](https://pkg.go.dev/net/http)
