---
layout: post
title: "Scaling Chunked Background Jobs: From Batch Size Tuning to Sidekiq Swarm"
date: 2025-12-24 11:20:00 +0545
categories: [Sidekiq]
tags: [sidekiq, sidekiq_swarm, batch_handling, performance]
---

# Why 100k Batch Job Got Stuck — and How Sidekiq Swarm Would Have Fixed It

When processing large datasets locally, there seems to have classic background processing problem:

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

### Redis RTT warnings, CPU saturation, and why batch size mattered

During further debugging, Sidekiq started emitting the warning: _“Your Redis network connection is performing extremely poorly… ideally RTT should be < 1000. If these values are close to 100,000, your Sidekiq process may be CPU-saturated.”_ At first glance, this message appears to implicate Redis or network latency, but in this case the root cause was **local CPU saturation**, not Redis itself. With a single Redis instance and `sidekiq.rb` concurrency set to 5, each batch job was pushing large argument payloads (2,000 records per chunk) into Redis while also maintaining idempotency and progress tracking keys (e.g., `batch_id → successful_entries, unsuccessful_entries, errors`).
In this case, each batch chunk (2,000 records) triggered `BulkProcessorWorker#update_progress_in_bulk`, which issued 8+ Redis commands per chunk—multiple `HINCRBY`s, `SADD`, repeated `RPUSH` calls per error and result, plus cardinality checks—resulting in hundreds of Redis operations per job when batch sizes were large.
The size and frequency of these Redis operations scaled directly with the batch size and argument volume. When batches contained 2,000 entries, Redis round-trip time (RTT) measurements ballooned—not because **Redis was slow, but because the Sidekiq process was CPU-bound** and could not service Redis IO promptly. In other words, Redis responses were waiting on a saturated Ruby VM. Reducing the batch size to 500 significantly lowered object allocation, serialization cost, and GC pressure per job, allowing the Sidekiq process to respond to Redis quickly again and eliminating the RTT warnings. This highlights an important nuance: **large job arguments and heavy Redis interaction can indirectly manifest as “network” warnings when the real bottleneck is CPU saturation in a single Sidekiq process**—a problem that Sidekiq Swarm mitigates by distributing work across multiple processes rather than overloading one.

### Why Sidekiq Swarm avoids this failure mode

Sidekiq Swarm addresses this class of problem by running multiple Sidekiq processes, each with low concurrency, instead of a single highly saturated process. Rather than one Ruby VM handling large batches, Redis serialization, GC, and job execution simultaneously, Swarm distributes these responsibilities across many isolated processes. Each process maintains a smaller heap, experiences shorter GC pauses, and keeps Redis RTTs low. This aligns with Sidekiq’s recommended multi-process architecture as described in the official documentation: https://github.com/sidekiq/sidekiq/wiki/Ent-Multi-Process.

## Final summary

The job got stuck not because of data size, but due to CPU saturation and excessive GC pressure in a single Ruby process or Sidekiq process, amplified by large batch sizes and inefficient Redis interactions. Reducing the batch size to 500 alleviated the immediate pressure by lowering allocation and Redis round-trip overhead, allowing the job to complete. However, this was a tactical fix, not a scalable solution. Adopting a Sidekiq Swarm model—combined with efficient Redis usage such as pipelining—shifts the system from a single overloaded worker to many small, isolated processes or workers, distributing CPU load, reducing GC pauses, stabilizing Redis RTTs, and removing the single-process bottleneck entirely.

---

{% include inarticle-adsense.html %}
