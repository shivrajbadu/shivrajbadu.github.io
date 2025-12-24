---
layout: post
title: "Scale chunking with Sidekiq swarm"
date: 2025-12-24 11:20:00 +0545
categories: [Sidekiq]
tags: [sidekiq, sidekiq_swarm, batch_handling, performance]
---

# Why My 100k Batch Job Got Stuck — and How Sidekiq Swarm Would Have Fixed It

When processing large datasets locally, I ran into a classic background processing problem:

> **A job that worked at smaller batch sizes but completely stalled at larger ones.**

This post explains **why that happened**, **why reducing the batch size fixed it**, and **how Sidekiq Swarm would have solved the problem more cleanly**.

---

## The scenario

- Total records: **~100,000**
- Processing strategy: chunking
- Initial chunk size: **2,000**
- Environment: **localhost**
- Result:
  - CPU spiked to ~100%
  - Process appeared “stuck”
- After change:
  - Chunk size reduced to **500**
  - CPU stabilized
  - Job completed successfully

At first glance, this looks counter-intuitive.

Same code.  
Same data.  
Smaller batch → magically works.

Let’s break down why.

---

## What actually went wrong

### 1. Chunking does _not_ equal throttling

Chunking only controls **how much data you load per iteration**.  
It does _not_ control:

- How fast the CPU is consumed
- How many Ruby threads are runnable
- How much memory is allocated at once

With a batch size of 2,000:

- Each iteration:
  - Loads 2,000 records
  - Allocates large Ruby objects
  - Executes business logic
  - Possibly performs validations, callbacks, DB writes, and GC

This creates **bursty CPU pressure**.

---

### 2. Ruby + CPU saturation = starvation

Ruby (MRI) has:

- A **Global VM Lock (GVL)**
- Cooperative thread scheduling
- Stop-the-world garbage collection

When CPU is fully saturated:

- GC runs more frequently
- Threads wait longer to acquire the GVL
- Scheduling latency increases
- IO callbacks get delayed

The process _looks stuck_, even though it is technically still running.

This is **CPU starvation**, not a deadlock.

---

### 3. Large batches amplify GC pressure

A batch size of 2,000 means:

- More objects allocated at once
- More short-lived objects
- Larger memory churn
- More frequent GC cycles

As GC frequency increases:

- CPU usage spikes
- Effective throughput drops
- Wall-clock time increases

This explains why **reducing the batch size actually improved performance**.

---

## Why batch size 500 worked

Reducing the batch size didn’t make the job “lighter” — it made it **smoother**.

With 500 records per chunk:

- Allocation pressure is spread out
- GC runs less aggressively
- CPU stays below saturation
- Ruby scheduler has breathing room
- The OS can context-switch normally

This is a classic **latency vs throughput tradeoff**.

You traded:

- 🔻 Peak CPU usage  
  for
- 🔺 Stable forward progress

---

## The real limitation: single-process processing

Even with chunking, the workload was still:

> **One Ruby process processing 100k rows sequentially**

That means:

- One GC heap
- One GVL
- One CPU bottleneck
- One point of failure

This is where Sidekiq Swarm fundamentally changes the equation.

---

## How Sidekiq Swarm would have solved this

### Parallelism with isolation

Sidekiq Swarm doesn’t make batches bigger or smaller.

It changes **where the pressure is applied**.

Instead of:

One process
→ 100k rows
→ internal chunking
→ CPU saturation

You get:

Many small workers
→ small jobs
→ distributed CPU load

---

## What the job would look like with Sidekiq Swarm

Instead of one giant job:

1. Split 100k rows into jobs of 500 (or even 200)
2. Enqueue ~200 jobs into Redis
3. Let the swarm process them

Mental model:

100,000 rows
= 200 jobs × 500 rows

20 Sidekiq workers
= ~10 jobs per worker

Each worker:

- Uses a small amount of CPU
- Has its own GC lifecycle
- Is fully disposable
- Can be restarted safely

---

## Why this prevents jobs from “getting stuck”

### 1. CPU load is distributed

Instead of one process hitting 100% CPU:

- Multiple workers run at moderate utilization
- OS scheduling remains healthy
- No single process starves itself

---

### 2. Garbage collection pressure is localized

Each worker has:

- A smaller heap
- Fewer live objects
- Shorter GC pauses

This dramatically improves tail latency.

---

### 3. Failure is no longer catastrophic

Without Swarm:

- Job stalls → everything stalls

With Swarm:

- One worker stalls → job retries
- Other workers continue processing
- System keeps moving forward

---

### 4. Throughput scales horizontally

To speed things up:

- Add more workers
- Not bigger batch sizes
- Not more threads per process

This is the core architectural win.

---

## Limitations and tradeoffs of Sidekiq Swarm

Sidekiq Swarm is powerful, but it is **not free or universally optimal**.

Understanding its limitations is important before adopting it.

---

### 1. Higher operational complexity

Swarm shifts complexity from code to infrastructure.

You now need to manage:

- Many Sidekiq processes or containers
- Scaling rules
- Deployment orchestration
- Monitoring and alerting

For small systems, this may be unnecessary overhead.

---

### 2. Increased Redis load

Swarm relies heavily on Redis:

- More workers polling queues
- More job enqueue/dequeue operations
- More retries and acknowledgements

Redis must be:

- Properly sized
- Monitored for latency
- Highly available

Redis becomes a critical dependency.

---

### 3. Requires strong job idempotency

With many workers:

- Jobs can be retried
- Jobs can run more than once
- Workers can crash mid-execution

All jobs **must be idempotent**.

If your jobs assume “exactly once” execution, Swarm will expose bugs fast.

---

### 4. Not ideal for very long-running jobs

Sidekiq Swarm favors:

- Short
- Predictable
- Restartable jobs

Very long-running jobs:

- Hold worker capacity hostage
- Delay retries on failure
- Complicate deploys and shutdowns

Such jobs should often be broken into smaller steps anyway.

---

### 5. More processes ≠ less resource usage

While load is distributed:

- Total CPU usage may increase
- Total memory usage may be higher
- Context switching overhead exists

Swarm improves **system behavior**, not raw efficiency.

---

### 6. Harder local development

Running a real swarm locally can be awkward:

- Many processes
- Multiple queues
- Redis dependency
- Noisy logs

Local setups often simulate Swarm only partially.

---

## Why reducing batch size felt like a fix (but isn’t a solution)

Lowering batch size was a **symptom-level fix**.

It helped because it avoided pathological CPU and GC behavior.

But it did not:

- Improve fault tolerance
- Improve scalability
- Improve deploy safety
- Remove the single-process bottleneck

Sidekiq Swarm addresses these **system-level concerns**.

---

## Key takeaway

> **If CPU saturation causes jobs to stall, the problem is rarely batch size alone — it’s lack of parallelism and isolation.**

Chunking is a local optimization.  
Sidekiq Swarm is an architectural solution.

---

## Final summary

The job got stuck not because of data size, but because a single Ruby process was overloaded with CPU and GC pressure. Reducing the batch size lowered peak resource usage and allowed progress, but didn’t fundamentally solve the scalability issue. With Sidekiq Swarm, the workload would be split into many small, isolated jobs processed by multiple workers, distributing CPU load, reducing GC pressure, and eliminating the single-process bottleneck entirely.

---

{% include inarticle-adsense.html %}
