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

### Title Generation & Approval Process

**Before generating any blog content, ALWAYS follow this process:**

1. **Generate 5-6 Unique Title Options**
   - Titles must be plain English, clear, and descriptive
   - Each title must be unique compared to existing articles on the web
   - Avoid generic or overly common phrasing
   - Consider different angles: practical, conceptual, question-based, solution-oriented
   - Each title should be between 40-70 characters for optimal SEO

2. **Title Characteristics**
   - ✅ Specific and actionable (e.g., "Building Real-Time Notifications with Action Cable and Redis")
   - ✅ Unique perspective (e.g., "Why Rails Callbacks Are Not Always Evil")
   - ✅ Question format (e.g., "How Does Ollama Handle Context Windows Locally?")
   - ✅ Problem-solution format (e.g., "Solving N+1 Queries in Rails with Bullet Gem")
   - ❌ Avoid: Generic titles like "Introduction to Rails" or "Getting Started with Python"
   - ❌ Avoid: Clickbait or overly dramatic phrasing

3. **Present Options to Author**
   - Display all 5-6 title suggestions clearly
   - Wait for author (shivrajbadu) to select one OR provide a refined version
   - Do NOT proceed with content generation until title is approved

4. **After Title Selection**
   - Use the exact approved title in the front matter and main heading
   - Proceed with full blog content generation using all other rules

**Example Title Generation:**

When the prompt is: *"Create blog post about deploying Rails with Docker"*

**AI Response:**
> I'll create a blog post about deploying Rails with Docker. First, here are 6 unique title suggestions:
> 
> 1. **"Containerizing Rails: A Practical Docker Deployment Guide"**
> 2. **"From Development to Production: Rails + Docker + PostgreSQL"**
> 3. **"How to Deploy Rails 7 Applications Using Docker Compose"**
> 4. **"Rails in Containers: Avoiding Common Docker Pitfalls"**
> 5. **"Building a Production-Ready Rails Docker Environment"**
> 6. **"Why Docker Changed My Rails Deployment Workflow"**
> 
> Which title would you like to use, or would you prefer a modified version?

*[Wait for selection before generating content]*

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

### Reference & Attribution Standards

**Preferred Sources:**
- Official documentation (Ruby docs, Rails guides, AWS docs, language specifications)
- Wikipedia and public domain resources
- Open-source project repositories and their documentation
- Academic resources and educational materials (MIT OpenCourseWare, arXiv, etc.)
- Government and institutional publications

**Writing Philosophy:**
1. Research from multiple authoritative sources
2. Synthesize information to form unique perspectives
3. Express technical concepts in original language with personal insight
4. Verify technical accuracy through testing and official documentation
5. Provide readers with high-quality, authoritative resources for further learning

**Content Originality:**
- All content should be originally written, reflecting personal understanding and experience
- Technical concepts should be explained in your own words
- Code examples should be original implementations or clearly attributed to official documentation
- "Suggested Reading" should prioritize evergreen, authoritative sources over ephemeral content

**Reference Linking Policy:**
- Link only to freely accessible, publicly available resources
- Avoid linking to copyrighted commercial content, paywalled articles, or individual blog posts
- Prioritize primary sources (official docs) over secondary interpretations
- Ensure all linked resources are stable and unlikely to disappear

**Quality Sources for "Suggested Reading":**
- ✅ Official language/framework documentation
- ✅ Wikipedia technical articles
- ✅ Well-maintained open-source projects
- ✅ Academic papers and educational institutions
- ✅ Established technical standards (RFCs, W3C, IETF)

---

### Content Structure & Engagement Rules

**Introduction (Hook Readers Immediately):**
- Start with a problem, relatable scenario, or thought-provoking question
- State the practical value in the first 2-3 sentences
- Set clear expectations: what the reader will learn/build
- ❌ Avoid: "In this post, I will..." — Get straight to the value

**Content Flow:**
- Build a narrative arc: Problem → Context → Solution → Application
- Use smooth transitions between sections (refer back, look forward)
- Break long technical sections with visual elements (lists, tables, diagrams)
- Each section should answer: "Why does this matter?"

**Visual & Practical Focus:**
- Prioritize diagrams, tables, code blocks over long paragraphs
- Use ASCII diagrams, Mermaid syntax, or describe visual architecture
- Include comparison tables (Before/After, Option A vs B)
- Add callouts for tips, warnings, gotchas using blockquotes

**Engagement Elements:**
- Real-world scenarios and use cases (not hypothetical examples)
- "Try it yourself" challenges or exercises
- Thought-provoking questions throughout (not just conclusion)
- Performance data, benchmarks, or concrete metrics where relevant

---

### Enhanced Code Standards

**Every Code Block Must Have:**
1. **Context Before** (2-3 lines explaining why this code, what problem it solves)
2. **The Code** (production-quality, not just demos)
3. **Explanation After** (what's happening, key lines highlighted)
4. **Output/Result** (show what the code produces when relevant)

**Code Quality Requirements:**
- Production-ready code with error handling
- Include comments for complex logic
- Show both ❌ problematic and ✅ correct approaches when teaching concepts
- Add "Common Pitfalls" or "Gotchas" subsections
- Security considerations for sensitive operations

**Example Structure:**
```markdown
### Implementing Background Jobs

We need async processing to avoid blocking user requests during email sends.

\`\`\`ruby
# ❌ Problematic: Blocks request
def create
  user = User.create(user_params)
  UserMailer.welcome_email(user).deliver_now  # Slow!
end

# ✅ Better: Background processing
def create
  user = User.create(user_params)
  UserMailer.welcome_email(user).deliver_later
end
\`\`\`

The `deliver_later` method queues the job in ActiveJob, returning control immediately.

**Output:**
- Request completes in ~50ms (vs 3000ms with deliver_now)
- Email sent asynchronously via Sidekiq

**Common Pitfall:** Forgetting to start Sidekiq in production causes emails to never send.
```

---

### Practical Value Requirements

**Every Post Must Include:**
- **Clear Takeaways:** End each major section with a key insight box
- **When to Use This:** Explicit guidance on applicability
- **Common Mistakes:** Dedicated section for gotchas and pitfalls
- **Next Steps:** What to explore after mastering this concept

**Actionable Elements:**
- Prerequisites clearly stated at the beginning
- Step-by-step implementation (numbered lists)
- Verification steps ("How to know it worked")
- Troubleshooting subsection for technical posts

**Real-World Impact:**
- Business value or practical benefits stated explicitly
- Performance implications (faster, cheaper, more reliable)
- Scalability considerations where relevant

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

### 📋 Category & Tag Taxonomy (SEO-Optimized)

**Purpose:** Maintain consistent, searchable categories and tags across all blog posts for better SEO and content discovery.

**Before selecting categories/tags:**
1. **Check the standardized taxonomy tables below first**
2. **Reuse existing terms** whenever applicable
3. **Add new terms only if justified** by unique content focus
4. **Never create synonyms** (e.g., don't use both `ci-cd` and `continuous-integration`)

---

#### **Standardized Categories**

Use **2-3 categories maximum** per post. Categories are broad topic areas.

| Category | When to Use | Related Topics |
|----------|-------------|----------------|
| **DevOps** | CI/CD, pipelines, infrastructure, deployment automation | CircleCI, Jenkins, GitHub Actions, Docker, Kubernetes |
| **Rails** | Ruby on Rails-specific content | Active Record, Action Cable, Hotwire, Turbo |
| **Ruby** | Pure Ruby language features | Blocks, lambdas, metaprogramming, gems |
| **Python** | Python language, libraries, frameworks | Django, FastAPI, data science, ML |
| **React** | React.js frontend development | Hooks, components, state management |
| **Next.js** | Next.js framework specifics | SSR, ISR, routing, API routes |
| **Go** | Golang backend development | Goroutines, channels, CLI tools |
| **AI** | Artificial Intelligence, machine learning, LLMs | GPT, BERT, transformers, embeddings |
| **Frontend** | General frontend development | CSS, Tailwind, UI/UX, JavaScript |
| **Backend** | General backend architecture | APIs, databases, authentication |
| **Machine Learning** | ML algorithms, models, training | Supervised/unsupervised learning, neural networks |
| **Deep Learning** | Neural networks, deep models | CNNs, RNNs, transformers |
| **Data Science** | Data analysis, visualization | Pandas, NumPy, Jupyter |
| **Docker** | Containerization, Docker-specific | Dockerfile, docker-compose, images |
| **Kubernetes** | Container orchestration | Pods, deployments, services |
| **AWS** | Amazon Web Services | EC2, S3, Lambda, RDS |
| **Security** | Application security, authentication | OAuth, JWT, encryption, vulnerabilities |
| **Performance** | Optimization, profiling, benchmarking | Query optimization, caching, CDN |
| **Testing** | Test automation, TDD, QA | RSpec, Jest, pytest, unit tests |
| **Database** | Database design and management | PostgreSQL, Redis, MySQL, MongoDB |
| **API** | API design and integration | REST, GraphQL, webhooks |

---

#### **Standardized Tags**

Use **4-8 tags maximum** per post. Tags are specific, searchable terms. **Always use kebab-case.**

**CI/CD & DevOps Tags:**
| Use This ✅ | Never Use ❌ | Context |
|-------------|-------------|---------|
| `ci-cd` | `cicd`, `continuous-integration`, `continuous-delivery` | General CI/CD concepts |
| `circleci` | `circle-ci`, `circle_ci` | CircleCI platform |
| `github-actions` | `github_actions`, `actions` | GitHub Actions |
| `jenkins` | `Jenkins` | Jenkins CI |
| `docker` | `Docker`, `containers` | Docker containerization |
| `docker-build` | `docker_build`, `build-optimization` | Docker build optimization |
| `kubernetes` | `k8s`, `K8s` | Kubernetes orchestration |
| `deployment` | `deploy`, `deploying` | Deployment processes |
| `pipelines` | `pipeline`, `build-pipeline` | CI/CD pipelines |
| `build-cache` | `caching`, `cache-optimization` | Build caching strategies |
| `performance-optimization` | `optimization`, `speed` | Performance improvements |

**Ruby & Rails Tags:**
| Use This ✅ | Never Use ❌ | Context |
|-------------|-------------|---------|
| `ruby-on-rails` | `rails`, `ror`, `ruby on rails` | Ruby on Rails framework |
| `ruby` | `Ruby` | Ruby language |
| `active-record` | `activerecord`, `active_record` | Rails ORM |
| `action-cable` | `actioncable`, `websockets` | Rails WebSocket framework |
| `rspec` | `RSpec`, `testing` | RSpec testing |
| `minitest` | `Minitest` | Minitest framework |
| `sidekiq` | `Sidekiq`, `background-jobs` | Background job processing |
| `hotwire` | `Hotwire`, `turbo` | Hotwire framework |

**Frontend Tags:**
| Use This ✅ | Never Use ❌ | Context |
|-------------|-------------|---------|
| `react` | `React`, `reactjs`, `react.js` | React library |
| `nextjs` | `next-js`, `next.js`, `Next.js` | Next.js framework |
| `jest` | `Jest`, `javascript-testing` | Jest testing |
| `tailwind-css` | `tailwind`, `tailwindcss` | Tailwind CSS |
| `typescript` | `TypeScript`, `ts` | TypeScript language |

**Python & AI Tags:**
| Use This ✅ | Never Use ❌ | Context |
|-------------|-------------|---------|
| `python` | `Python` | Python language |
| `pytest` | `Pytest` | Pytest framework |
| `unittest` | `unit-test`, `unit_test` | Python unittest |
| `fastapi` | `FastAPI`, `fast-api` | FastAPI framework |
| `machine-learning` | `ml`, `ML`, `machinelearning` | Machine learning |
| `deep-learning` | `dl`, `DL`, `deeplearning` | Deep learning |
| `neural-networks` | `neural_networks`, `nn` | Neural networks |
| `llm` | `LLM`, `large-language-models` | Large language models |
| `transformers` | `transformer`, `attention` | Transformer models |

**General Tags:**
| Use This ✅ | Never Use ❌ | Context |
|-------------|-------------|---------|
| `devops` | `DevOps`, `dev-ops` | DevOps practices |
| `api` | `API`, `apis` | API development |
| `graphql` | `GraphQL`, `graph-ql` | GraphQL |
| `rest-api` | `rest`, `REST`, `restful` | REST APIs |
| `postgresql` | `postgres`, `Postgres` | PostgreSQL |
| `redis` | `Redis` | Redis |
| `authentication` | `auth`, `Auth` | Authentication |
| `authorization` | `authz` | Authorization |
| `security` | `Security`, `sec` | Security topics |
| `aws` | `AWS`, `amazon` | Amazon Web Services |

---

#### **Tag Selection Process**

**When writing a blog post, follow this workflow:**

1. **Identify primary technology/platform** (e.g., CircleCI, Docker, Rails)
   - Add platform-specific tag: `circleci`, `docker`, `ruby-on-rails`

2. **Identify technical domain** (e.g., CI/CD, testing, performance)
   - Add domain tag: `ci-cd`, `testing`, `performance-optimization`

3. **Identify specific tools/concepts** (e.g., RSpec, caching, Docker builds)
   - Add specific tags: `rspec`, `build-cache`, `docker-build`

4. **Add related technologies** mentioned significantly in the post
   - Example: If post covers Docker + PostgreSQL + GitHub Actions
   - Tags: `docker`, `postgresql`, `github-actions`, `ci-cd`

**Example Applications:**

| Blog Topic | Categories | Tags |
|------------|-----------|------|
| Blacksmith for Docker builds in GitHub Actions | `[DevOps, CI/CD]` | `[blacksmith, github-actions, docker, ci-cd, docker-build, build-cache, performance-optimization]` |
| CircleCI for Rails with RSpec | `[DevOps, Rails]` | `[circleci, ci-cd, ruby-on-rails, rspec, testing, devops]` |
| Jenkins enterprise Rails setup | `[DevOps, Rails]` | `[jenkins, ci-cd, ruby-on-rails, docker, rspec, devops]` |
| Rails Action Cable with Redis | `[Rails, Backend]` | `[ruby-on-rails, action-cable, redis, websockets, real-time]` |
| Next.js ISR deployment | `[Next.js, Frontend]` | `[nextjs, react, isr, rendering, deployment, frontend]` |

---

#### **Adding New Terms**

If you need to add a **new category or tag** not in the taxonomy:

1. **Justify it:** Ensure it represents a genuinely new topic area
2. **Check for conflicts:** Make sure you're not creating a synonym
3. **Use kebab-case:** Always hyphenated lowercase
4. **Document it:** Add to the appropriate table in GEMINI.md
5. **Be specific enough:** `ml-ops` is better than `operations`
6. **Be general enough:** Don't create `rails-7-action-cable` when `action-cable` exists

**Bad additions (too specific/redundant):**
- ❌ `github-actions-docker` (use separate tags: `github-actions`, `docker`)
- ❌ `circle-ci-optimization` (use `circleci`, `performance-optimization`)
- ❌ `continuous-deployment` (use `ci-cd`, `deployment`)

**Good additions (justified new concepts):**
- ✅ `edge-computing` (new infrastructure paradigm)
- ✅ `vector-databases` (emerging database category)
- ✅ `webassembly` (new runtime technology)

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
