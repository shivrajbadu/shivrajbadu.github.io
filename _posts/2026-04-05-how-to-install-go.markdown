---
layout: post
title: "How to Install Go (Golang) on Your System"
date: 2026-04-05 21:30:00 +0545
categories: [Go, Setup]
tags: [go, golang, installation, setup]
---

# How to Install Go (Golang) on Your System

Go is a powerful, efficient programming language. Here is a quick guide to getting it installed on your machine.

## 1. Download Go

Visit the [official Go download page](https://go.dev/dl/) and download the installer appropriate for your operating system (Windows, macOS, or Linux).

## 2. Installation

### For macOS

If you use Homebrew, installation is simple:

```bash
brew install go
```

Otherwise, download the `.pkg` file and run the installer.

### For Linux

1. Download the archive file (e.g., `go1.x.x.linux-amd64.tar.gz`).
2. Extract the archive:
   ```bash
   tar -C /usr/local -xzf go1.x.x.linux-amd64.tar.gz
   ```
3. Add Go to your `PATH` by adding the following line to your `~/.bashrc` or `~/.zshrc`:
   ```bash
   export PATH=$PATH:/usr/local/go/bin
   ```

### For Windows

Download the `.msi` file and run it. The installer will automatically add Go to your PATH.

## 3. Verify Installation

Open a terminal or command prompt and run:

```bash
go version
```

If successful, you will see the installed Go version.

## Conclusion

Now that you have Go installed, you are ready to start building!

{% include inarticle-adsense.html %}

## Suggested Reading

- [Official Go Documentation](https://go.dev/doc/)
