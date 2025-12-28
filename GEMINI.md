# 🧠 GEMINI.md

## 1. CLI Custom Instructions

### Git Automation Workflow

When I use any of these trigger phrases, automatically execute the complete git workflow:

**Trigger Phrases:**

- "Create Blog post for my website shivrajbadu.com.np"
- "commit and push"
- "save changes"
- "deploy"
- "update repo"
- "save work"
- "push to github"

**Automated Steps:**

1. Run `git status` to check current state
2. Execute `git add .` to stage all changes
3. Generate meaningful commit message based on file changes
4. Run `git commit -m "[generated message]"`
5. Execute `git push` to current branch

> ⚠️ **Note:** Do **not** actually push to GitHub — I will review and push manually.

---

### Commit Message Convention

Follow **Conventional Commit** format:

- `feat:` — New features
- `fix:` — Bug fixes
- `docs:` — Documentation or blog post changes
- `style:` — Formatting or structure only
- `refactor:` — Code restructuring
- `chore:` — Maintenance or configuration
- `update:` — General updates

**Examples:**

- Blog post → `docs: add [topic] blog post`
- Code → `feat: implement [feature]`
- Config → `chore: update [config]`
- Mixed changes → `update: [brief summary]`

---

## 2. ✍️ Blog Writing Rule Set

### Purpose

This section defines the **structure, tone, and format** for all blog posts written for  
[`shivrajbadu.com.np`](https://shivrajbadu.com.np).  
When I say “Create blog post…” or similar, follow these rules exactly — no need to ask for formatting details again.

---

### Blog Post Format

Each post **must begin** with a Jekyll front matter block:

```yaml
---
layout: post
title: "Full Title of the Post"
date: YYYY-MM-DD HH:MM:SS +0545
categories: [Category1, Category2]
tags: [tag-one, tag-two, tag-three]
---
```

Then continue in Markdown format:

# [Title of the Blog]

## Introduction

(A concise intro that sets up the theme or question.)

## Main Sections

(Divide ideas or technical concepts using `##` headings.)

## Code / Technical Section

(Include properly fenced code blocks and explanations.)

## Conclusion

(Summarize learnings, provide final insight or reflection.)

## Suggested Reading

(A short Markdown list of related articles, books, or docs.)

Code / Technical Section Rules

When writing about code, use fenced Markdown code blocks with language tags:

## Code Example: Generating Embeddings with Ollama

````bash
ollama run qwen2.5-coder "Write a Ruby method to calculate factorial."

Explanation

Use commented code where appropriate

Provide short contextual commentary under the block

Highlight important commands or lines using bold or inline backticks

Include prerequisites, installation steps, or environment notes as needed

LastLine must include this call {% include inarticle-adsense.html %}

Supported syntax highlighting languages include `ruby`, `js`, `bash`, `python`, `yaml`, `html`, `json`, `sql`, and more.

---

### Tone & Style Guide

- **Voice:** Thoughtful, calm, technically competent, and reflective.
- **Tone:** Professional but conversational — like teaching a peer.
- **Perspective:** Blend technical precision with personal insight where relevant.
- **Goal:** Educate clearly; inspire curiosity or reflection.
- **Structure:**
  - Use `##` for main sections, `###` for sub-sections
  - Keep paragraphs short and readable
  - Alternate between explanation, code, and reflection for rhythm
- **Markdown Practices:**
  - Use fenced code blocks for all code
  - Inline code → `like_this`
  - **Bold** important keywords
  - *Italicize* conceptual or reflective thoughts
- **Length:** Aim for 1200–2500 words
- **Ending:** Always close with a short reflective conclusion and “Suggested Reading” section

---

### Formatting Conventions

| Element | Rule |
|----------|------|
| Layout | `layout: post` |
| Date | Use Nepal Time (+0545) |
| File Extension | `.md` |
| Category | Should be broad (e.g., Philosophy, Rails, AI, DevOps) |
| Tags | Use kebab-case (e.g., `ruby-on-rails`, `ollama`, `ai-integration`) |
| Headings | Use `##` for main sections |
| Code Blocks | Fenced with language tags (```ruby, ```bash, etc.) |
| Inline Code | Enclosed with backticks `like_this` |
| Images | Use `![alt text](image_url)` — optional |
| Links | `[link text](https://example.com)` |
| End of Post | Must include **Suggested Reading** |

---

### Example Blog Prompt Template

When I say:

> “Create blog post titled *Deploying Rails 6 with Redis and Action Cable on Heroku*”

The AI should automatically generate:

1. Correct YAML front matter (with today’s timestamp +0545)
2. Markdown body with:
   - Introduction
   - Main sections with headings
   - One or more technical/code sections
   - Clear explanations and best practices
   - Conclusion + Suggested Reading
3. Proper Markdown formatting
4. No extra commentary — just final `.md` content ready for `_posts/`

---

### Common Blog Themes

**Technical**
- Ruby on Rails guides
- QuickBooks, Redis, AWS, Heroku integrations
- AI & LLM workflows (Ollama, OpenAI, LangChain)
- Python scripting, data workflows
- DevOps automation and git workflows

**Philosophical / Reflective**
- Human–AI interaction
- Consciousness, cognition, meaning in tech
- Cross-cultural perspectives on science and ethics
- Technology and human creativity

**Hybrid Topics**
- Coding as philosophy
- Ethics in automation
- Developer mindfulness and focus
- The art of writing maintainable code

---

### Example Triggers

- “Create blog post titled *Building a GraphQL API in Rails 8*”
- “Write blog on *Integrating Ollama with Python for Local LLM Inference*”
- “Generate blog post for *The Illusion of Time and Consciousness*”
- “Add new markdown post for *Rails + Redis Action Cable Setup Guide*”

---

### Quick Style Summary

| Aspect | Rule |
|--------|------|
| Tone | Clear, analytical, calm |
| Voice | Reflective yet technical |
| Reading Flow | Smooth transitions between explanation and reflection |
| Post Type | Supports both technical + philosophical |
| Minimum Length | 1200 words |
| Ending | Reflective summary + Suggested Reading |
| Output Format | Markdown (.md) |
| Code Formatting | Fenced blocks with language tags |
| Push Behavior | Do not auto-push to GitHub |

---

## 3. 🧩 Optional Future Extensions

You can later add:
- **Snippet Style & Visuals:** how to embed screenshots or diagrams
- **Featured Image Rules:** thumbnail paths or YAML keys (`image:` field)
- **AI Persona Voice:** to preserve consistent tone and rhythm in writing

---


### Style
- Tone: calm, thoughtful, technically clear
- Use **bold** for key terms and `inline code` where needed
- Use proper Markdown formatting for headings, lists, and code
- Length: ~1200+ words preferred
- Always end with **“Suggested Reading”**

---

## Example Triggers

- “Create blog post titled *Using Ruby to Automate Vendor Imports with AWS S3*”
- “Write blog on *Action Cable and Redis in Rails 6.1*”
- “New markdown blog for *Qwen Code with Ollama*”

AI should:
1. Generate correct front matter (Nepal time +0545)
2. Write structured Markdown
3. Include properly fenced code blocks
4. Be ready for publishing at `shivrajbadu.com.np`

````
