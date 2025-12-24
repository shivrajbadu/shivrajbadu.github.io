---
layout: post
title: "Agile, Scrum, and Similar Practices"
date: 2025-11-20 08:40:00 +0545
categories: [Agile, Scrum]
tags: [agile, scrum]
---

# 📘 Notes: Agile, Scrum, and Similar Practices

## 1. What is Agile?

**Agile** is a mindset and a set of **values/principles** for building software (or any product) in an iterative way.

### Core Ideas

- Deliver work in **small increments**
- **Respond to change** instead of rigid long-term plans
- **Continuous feedback** from users/stakeholders
- **Collaboration** over heavy documentation
- Frequent release of **working product**

### Agile Values (from the Agile Manifesto)

- **Individuals & interactions** over processes & tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

---

## 2. What is Scrum?

**Scrum** is a framework **within Agile**, and the most widely used one.

> **Scrum is Agile, but Agile is not always Scrum.**

### Scrum Focuses On

- Fixed-length iterations called **Sprints** (1–4 weeks)
- A small cross-functional development team
- Frequent inspection, adaptation, and transparency

### Main Scrum Roles

- **Product Owner (PO)** – Manages product backlog & priorities
- **Scrum Master (SM)** – Removes blockers and guides the Scrum process
- **Developers/Team Members** – Build, test, and deliver product increments

### Scrum Events

- **Sprint Planning**
- **Daily Standup (Daily Scrum)**
- **Sprint Review**
- **Sprint Retrospective**
- **Backlog Refinement** (optional but commonly used)

---

## 3. How Agile and Scrum Relate

- **Agile = philosophy / mindset**
- **Scrum = one method to practice Agile**

Think of Agile as a religion and Scrum as one specific path within it.

---

## 4. Other Agile Frameworks / Methodologies Similar to Scrum

### a) Kanban

- Visual board with workflow columns
- Continuous delivery
- Work-in-progress (WIP) limits  
  **Best for:** Support, DevOps, continuous flow teams

### b) XP (Extreme Programming)

Engineering-focused Agile approach:

- TDD (Test-Driven Development)
- Pair programming
- Continuous integration
- Refactoring  
  **Best for:** High-quality code and frequent releases

### c) Lean Software Development

Principles:

- Remove waste
- Deliver fast
- Optimize flow
- Continuous learning

### d) SAFe (Scaled Agile Framework)

- Framework to scale Agile across large enterprises
- Uses Agile Release Trains, Program Increments, etc.

### e) LeSS (Large-Scale Scrum)

- Expands Scrum to multiple teams
- Lightweight and simple

### f) DSDM (Dynamic Systems Development Method)

- Timeboxing
- MoSCoW prioritization (Must/Should/Could/Won’t)

---

### Jr. ML/AI engineers asked me this question if Agile or Scrum applicable for them.

Many junior ML and AI developers wonder whether Agile or Scrum actually applies to their work, since model development often feels more like research than traditional software engineering. The truth is that Agile absolutely fits ML workflows because it encourages iteration, experimentation, and continuous feedback—core parts of building models. Scrum can also be applied, but usually with more flexibility, using things like research spikes, experiment logs, and hybrid Scrum-ban flows to adapt to the uncertainty of ML tasks. In short, Agile gives ML teams the mindset they need, while Scrum provides structure they can tailor to their unique exploratory work.

#### 🧠 Why Agile is relevant for ML/AI teams

Agile is a mindset, and it definitely fits ML workflows:
ML projects evolve through experiments, which Agile supports.
Requirements often change after insights — Agile embraces this.
Iterative cycles help teams deliver incremental value, not wait months.
Stakeholder feedback is critical for models — Agile enables that.

#### 📘 How Scrum fits ML/AI — but with tweaks

Scrum can work well, but ML is different from normal software engineering. Why?

Because ML tasks often look like:

- Try 5 different model approaches — whichever works best
- Tune hyperparameters until performance improves
- Collect more/better data
- Run experiment → evaluate → adjust

These don’t always fit neatly into:

- User stories
- Fixed Sprint commitments
- Definition of done (DoD)

But teams adapt Scrum by:

- Making research spikes (time-boxed learning tasks)
- Using experiment logs as deliverables
- Explicit DoD like: "Model accuracy > baseline by X%"
- Accepting uncertainty as part of the Sprint

#### 🔧 Alternative Agile frameworks sometimes better for ML

Some ML teams prefer:

##### Kanban

- Because ML work flows continuously
- Good for variable-length tasks (training, experiments)
- No pressure of Sprint commitments

##### Hybrid (Scrum-ban)

- Scrum for planning/reviews
- Kanban for execution

### 5. Summary Table

| Concept         | Type               | Description                                   |
| --------------- | ------------------ | --------------------------------------------- |
| **Agile**       | Mindset            | Values + principles for iterative development |
| **Scrum**       | Framework          | Role + event-based structured Agile method    |
| **Kanban**      | Framework          | Visual flow, continuous delivery              |
| **XP**          | Methodology        | Strong engineering practices                  |
| **Lean**        | Methodology        | Waste reduction, speed, flow                  |
| **SAFe / LeSS** | Scaling frameworks | Agile for multiple large teams                |

---

{% include inarticle-adsense.html %}
