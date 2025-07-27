---
layout: post
title: "Gemini CLI with MCP Integration: AI Powered Terminal Coding Assistant and AI Development Workflow"
date: 2025-07-27 8:20:00 +0545
categories: [Artificial Intelligence (AI), gemini-cli, MCP, mcp_server_git]
tags: [AI, gemini-cli, ai-coding-assistant, developer-tools, cli-tools, llm, claude-code-alternative, mcp, mcp_server_git]
---

# Gemini CLI: Google's AI Coding Assistant with MCP Integration

Google's **Gemini CLI** brings the power of Gemini 2.5 Pro directly into your terminal, offering code understanding, file manipulation, and intelligent automation. With its 1-million-token context window, it can handle large codebases and complex development tasks with ease.

## Key Features

- **Large Codebase Analysis**: Handle projects beyond typical token limits
- **Multimodal Capabilities**: Generate code from PDFs, sketches, and documents
- **Workflow Automation**: Streamline development tasks and troubleshooting
- **MCP Integration**: Connect to external tools like GitHub, Git, and databases

## Installation and Setup

### Install Gemini CLI
```bash
npm install -g @google/gemini-cli

OR

brew install gemini-cli

gemini
```

### Authentication Options

**Option 1: Google Account (Personal Use)**
```bash
gemini auth
```
Grants 60 model requests/minute and 1,000 model requests/day.

**Option 2: API Key (Production)**
1. Get API key from [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Set environment variable:
```bash
export GEMINI_API_KEY="your_api_key_here"
# Make persistent
echo 'export GEMINI_API_KEY="your_key_here"' >> ~/.bashrc
```

### Verify Installation
```bash
gemini --version
# 0.1.14
gemini --help
```

## Basic Usage Examples

### Code Generation
```bash
gemini "Create a React todo app with local storage"
gemini "Write a Python class for CSV file operations"
```

### Code Analysis
```bash
gemini "Review this code for performance issues: [paste code]"
gemini "Debug this function: [paste code and error]"
```

### File Operations
```bash
gemini "Create a Node.js project structure with package.json"
gemini "Generate README.md for this project"
```

## MCP Server Integration

### What is MCP?
Model Context Protocol (MCP) enables AI tools to connect to external data sources and tools. It's like a USB-C port for AI applications, providing standardized access to GitHub, Git, databases, and more.

### Install Git MCP Servers

**GitHub MCP Server:**
```bash
npm install -g @modelcontextprotocol/server-github
export GITHUB_PERSONAL_ACCESS_TOKEN="your_github_token"
```

**Local Git MCP Server:**
```bash
npm install -g @cyanheads/git-mcp-server
```

**MCP Inspector (for debugging):**
```bash
npm install -g @modelcontextprotocol/inspector
```

### MCP Configuration
Create `mcp-config.json`:
```json
{
  "servers": {
    "github": {
      "command": "node",
      "args": ["path/to/github-mcp-server"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token"
      }
    },
    "git": {
      "command": "git-mcp-server",
      "args": ["stdio"],
      "cwd": "/path/to/git/repository"
    }
  }
}
```

OR, In CLINE, Go to MCP sections,  install `Git Tools`.

### MCP + Gemini CLI Examples

**GitHub Operations:**
```bash
gemini "Show all open issues in this repository"
gemini "Create a GitHub issue for the API bug"
gemini "Review the latest 3 pull requests"
```

**Git Operations:**
```bash
gemini "Create feature branch 'user-auth' and switch to it"
gemini "Show diff for last 3 commits and explain changes"
gemini "Generate release notes since last tag"
```

**Repository Analysis:**
```bash
gemini "Analyze git history and find top contributors"
gemini "Review uncommitted changes before committing"
gemini "Audit repository for security issues"
```

## Best Practices

### Effective Prompting
- Be specific with requirements and context
- Include relevant code snippets and error messages
- Use follow-up questions to refine results

### MCP Security
- Store tokens in environment variables
- Use minimal necessary permissions
- Rotate access tokens regularly

### Performance Tips
- Start sessions in project root for full context
- Combine related questions in single prompts
- Use `/clear` to reset context when switching projects

## Troubleshooting

### Authentication Issues
```bash
# Reset authentication
gemini auth --reset

# Check environment variables
echo $GEMINI_API_KEY
```

### MCP Server Issues
```bash
# Debug MCP servers
mcp-inspector github-mcp-server

# Test connections
gemini "List available MCP resources and tools"
```

## Quick Reference

### Essential Commands
```bash
# Installation
npm install -g @google/gemini-cli
npm install -g @modelcontextprotocol/server-github
npm install -g @cyanheads/git-mcp-server

# Authentication
gemini auth
export GEMINI_API_KEY="your_key"

# Usage
gemini                           # Interactive session
gemini "your prompt"            # One-shot command
gemini --help                   # Help
```

### Key Prompts
- `"Analyze this codebase and suggest improvements"`
- `"Create tests for this function: [code]"`
- `"Show all GitHub issues and categorize by priority"`
- `"Review latest commits and suggest optimizations"`
- `"Generate project documentation"`

## Conclusion

Gemini CLI with MCP integration provides a powerful, context-aware development experience. The combination of Google's AI capabilities, extensive context window, and standardized protocol integration makes it an excellent choice for modern development workflows.

---

## Gemini-Cli in Action: A Visual Walkthrough

To demonstrate Gemini-Cli's capabilities in practice, here's a step-by-step visual guide showing the feature:

![GeminiCli]({{ "/assets/img/geminicli/gemini-cli-mcp-prompt.png" | relative_url }})
![GeminiCli]({{ "/assets/img/geminicli/git-added.png" | relative_url }})
![GeminiCli]({{ "/assets/img/geminicli/git-commit.png" | relative_url }})
![GeminiCli]({{ "/assets/img/geminicli/git-committed.png" | relative_url }})
![GeminiCli]({{ "/assets/img/geminicli/summarize-last-3-commits.png" | relative_url }})

{% include inarticle-adsense.html %}
