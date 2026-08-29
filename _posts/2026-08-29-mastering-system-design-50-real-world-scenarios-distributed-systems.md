---
layout: post
title: "Mastering System Design: 50 Real-World Scenarios from Rate Limiters to Distributed Systems"
date: 2026-08-29 09:20:00 +0545
categories: [Backend, System-Design]
tags: [system-design, distributed-systems, scalability, architecture, microservices, performance, interviews]
---

# Mastering System Design: 50 Real-World Scenarios from Rate Limiters to Distributed Systems

## Introduction

System design interviews and real-world architecture decisions share a common challenge: translating business requirements into scalable, reliable technical systems. Whether you're designing a URL shortener for millions of users or architecting a real-time notification system, the patterns remain consistent—understand the constraints, identify the bottlenecks, and make deliberate trade-offs.

This guide presents 50 real-world system design scenarios, from foundational components like rate limiters and caches to complete applications like social networks and video streaming platforms. Each scenario includes capacity planning, architectural decisions, and production considerations that separate hobby projects from systems that scale.

These aren't hypothetical exercises. They're distilled from production systems serving millions of users, where downtime costs money and architectural mistakes compound over time.

## Scenario 1: Design a Rate Limiter

**Requirements:**
- Limit API requests per user (e.g., 100 requests/minute)
- Distributed system (multiple servers)
- Low latency overhead (<10ms)
- Handle burst traffic gracefully

**Capacity Planning:**
- 10M users, peak 100K requests/second
- Rate limit check: ~100K ops/sec
- Storage: User ID + timestamp windows

**Architecture:**

```python
import redis
import time
from typing import Optional

class TokenBucketRateLimiter:
    """Token bucket algorithm using Redis"""
    
    def __init__(self, redis_client, rate: int, burst: int):
        self.redis = redis_client
        self.rate = rate  # tokens per second
        self.burst = burst  # bucket capacity
    
    def allow_request(self, user_id: str) -> bool:
        key = f"rate_limit:{user_id}"
        now = time.time()
        
        # Lua script for atomic token bucket update
        script = """
        local key = KEYS[1]
        local rate = tonumber(ARGV[1])
        local burst = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])
        
        local bucket = redis.call('HMGET', key, 'tokens', 'last_update')
        local tokens = tonumber(bucket[1]) or burst
        local last_update = tonumber(bucket[2]) or now
        
        -- Refill tokens based on time passed
        local elapsed = now - last_update
        tokens = math.min(burst, tokens + (elapsed * rate))
        
        if tokens >= 1 then
            tokens = tokens - 1
            redis.call('HMSET', key, 'tokens', tokens, 'last_update', now)
            redis.call('EXPIRE', key, burst / rate * 2)
            return 1
        else
            return 0
        end
        """
        
        result = self.redis.eval(script, 1, key, self.rate, self.burst, now)
        return bool(result)
```

**Algorithm Comparison:**

| Algorithm | Pros | Cons | Use Case |
|-----------|------|------|----------|
| Token Bucket | Smooth bursts, memory efficient | Complex implementation | API rate limiting |
| Leaky Bucket | Constant output rate | Drops bursts | Network traffic shaping |
| Fixed Window | Simple | Edge case bursts | Simple use cases |
| Sliding Window Log | Precise | Memory intensive | Accurate limits needed |

**Production Considerations:**
- Redis cluster for high availability
- Fallback behavior (fail open vs fail closed)
- Per-endpoint vs global limits
- Cost-based limiting (charge 10 tokens for expensive operations)

{% include inarticle-adsense.html %}

## Scenario 2: Design a Consistent Hashing System

**Requirements:**
- Distribute keys across N servers
- Minimize remapping when servers added/removed
- Balance load evenly

**Implementation:**

```python
import hashlib
from bisect import bisect_right
from typing import List, Any

class ConsistentHash:
    def __init__(self, nodes: List[str], virtual_nodes: int = 150):
        self.virtual_nodes = virtual_nodes
        self.ring = {}
        self.sorted_keys = []
        
        for node in nodes:
            self.add_node(node)
    
    def _hash(self, key: str) -> int:
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_node(self, node: str):
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_value = self._hash(virtual_key)
            self.ring[hash_value] = node
            self.sorted_keys.append(hash_value)
        
        self.sorted_keys.sort()
    
    def remove_node(self, node: str):
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_value = self._hash(virtual_key)
            del self.ring[hash_value]
            self.sorted_keys.remove(hash_value)
    
    def get_node(self, key: str) -> str:
        if not self.ring:
            return None
        
        hash_value = self._hash(key)
        index = bisect_right(self.sorted_keys, hash_value)
        
        if index == len(self.sorted_keys):
            index = 0
        
        return self.ring[self.sorted_keys[index]]
```

**Why Virtual Nodes Matter:**

Without virtual nodes:
- Uneven distribution (some servers get 2x load)
- Adding node causes massive redistribution

With 150 virtual nodes per physical node:
- Standard deviation of load: <5%
- Adding node: only ~1/N keys remapped

## Scenario 3: Design a Key-Value Store

**Requirements:**
- GET/PUT operations in O(1)
- Persistence and durability
- Handle node failures
- Scale to billions of keys

**Architecture:**

```
┌─────────────────┐
│   Load Balancer │
└────────┬────────┘
         │
    ┌────┴────┬────────┬─────────┐
    │         │        │         │
┌───▼──┐  ┌──▼───┐ ┌──▼───┐  ┌──▼───┐
│Node 1│  │Node 2│ │Node 3│  │Node 4│
│Master│  │Master│ │Master│  │Master│
└───┬──┘  └──┬───┘ └──┬───┘  └──┬───┘
    │        │        │         │
┌───▼──┐  ┌─▼────┐ ┌─▼────┐  ┌─▼────┐
│Replica│ │Replica││Replica│ │Replica│
└──────┘  └──────┘ └──────┘  └──────┘
```

**Core Components:**

```python
class DistributedKVStore:
    def __init__(self, nodes, replication_factor=3):
        self.hash_ring = ConsistentHash(nodes)
        self.replication_factor = replication_factor
    
    def put(self, key: str, value: Any) -> bool:
        # Find primary node
        primary = self.hash_ring.get_node(key)
        
        # Write to primary + (N-1) replicas
        nodes = [primary]
        for i in range(self.replication_factor - 1):
            nodes.append(self.hash_ring.get_next_node(key, skip=nodes))
        
        # Quorum write (W=2 for replication_factor=3)
        success_count = 0
        for node in nodes:
            if self._write_to_node(node, key, value):
                success_count += 1
                if success_count >= self.write_quorum:
                    return True
        
        return False
    
    def get(self, key: str) -> Optional[Any]:
        nodes = self._get_replicas(key)
        
        # Read repair: get from R nodes, return latest
        values = []
        for node in nodes[:self.read_quorum]:
            val = self._read_from_node(node, key)
            if val:
                values.append(val)
        
        # Return value with highest version
        return self._resolve_conflicts(values)
```

**Trade-offs:**

| Approach | Consistency | Availability | Use Case |
|----------|-------------|--------------|----------|
| Strong Consistency | High | Lower | Financial transactions |
| Eventual Consistency | Relaxed | High | Social feeds, caches |
| Causal Consistency | Medium | Medium | Collaborative editing |

## Scenario 4: Design a URL Shortener

**Requirements:**
- Shorten long URLs (bit.ly clone)
- 500M new URLs per month
- 100:1 read-to-write ratio
- Custom aliases support
- Analytics tracking

**Capacity Planning:**

```
URLs per month: 500M
URLs per second: 500M / (30 * 24 * 3600) ≈ 200/sec

Reads per second: 200 * 100 = 20K/sec

Storage (5 years):
- 500M * 12 * 5 = 30B URLs
- Each URL: 500 bytes (original + metadata)
- Total: 15TB

Short code length:
- 62 characters (a-z, A-Z, 0-9)
- 62^7 = 3.5 trillion combinations
- Use 7 character codes
```

**Architecture:**

```python
import base62
import redis
from datetime import datetime

class URLShortener:
    def __init__(self, db, cache, counter_service):
        self.db = db  # PostgreSQL
        self.cache = cache  # Redis
        self.counter = counter_service
    
    def shorten(self, long_url: str, custom_alias: str = None) -> str:
        # Check if URL already shortened
        if existing := self.db.get_by_url(long_url):
            return existing.short_code
        
        if custom_alias:
            if self.db.alias_exists(custom_alias):
                raise AliasAlreadyTaken
            short_code = custom_alias
        else:
            # Generate unique ID
            unique_id = self.counter.get_next_id()
            short_code = base62.encode(unique_id)
        
        # Store mapping
        self.db.insert({
            'short_code': short_code,
            'long_url': long_url,
            'created_at': datetime.now(),
            'click_count': 0
        })
        
        # Cache for 24 hours
        self.cache.setex(short_code, 86400, long_url)
        
        return short_code
    
    def resolve(self, short_code: str) -> str:
        # Check cache first
        if url := self.cache.get(short_code):
            self._increment_clicks_async(short_code)
            return url
        
        # Fetch from database
        record = self.db.get_by_code(short_code)
        if not record:
            raise URLNotFound
        
        # Update cache
        self.cache.setex(short_code, 86400, record.long_url)
        
        # Track analytics
        self._increment_clicks_async(short_code)
        
        return record.long_url
```

**Database Schema:**

```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    user_id BIGINT,
    click_count BIGINT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);

CREATE TABLE clicks (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) NOT NULL,
    clicked_at TIMESTAMP DEFAULT NOW(),
    ip_address INET,
    user_agent TEXT,
    referer TEXT,
    country VARCHAR(2),
    INDEX idx_short_code_time (short_code, clicked_at)
);
```

**Scaling Considerations:**
- Database sharding by short_code hash
- CDN for static short domain redirects
- Analytics data in separate time-series DB
- Rate limiting per IP to prevent abuse

## Scenario 5: Design a Web Crawler

**Requirements:**
- Crawl 1 billion web pages
- Politeness (respect robots.txt, rate limits)
- Deduplication
- Distributed architecture

**Architecture:**

```
┌──────────────┐
│ URL Frontier │  (Priority queue of URLs to crawl)
└──────┬───────┘
       │
   ┌───▼────┐
   │Scheduler│
   └───┬────┘
       │
   ┌───┴────┬────────┬────────┐
   │        │        │        │
┌──▼───┐ ┌─▼────┐ ┌─▼────┐ ┌─▼────┐
│Worker│ │Worker│ │Worker│ │Worker│
└──┬───┘ └─┬────┘ └─┬────┘ └─┬────┘
   │       │        │        │
   └───┬───┴───┬────┴────┬───┘
       │       │         │
   ┌───▼───────▼─────────▼───┐
   │  Content Storage (S3)   │
   └─────────────────────────┘
```

**Implementation:**

```python
import asyncio
import aiohttp
from urllib.parse import urljoin, urlparse
from bs4 import BeautifulSoup
import hashlib

class DistributedCrawler:
    def __init__(self, redis, s3, num_workers=100):
        self.redis = redis  # URL queue + seen set
        self.s3 = s3  # Content storage
        self.num_workers = num_workers
        self.rate_limiters = {}  # Per-domain rate limiting
    
    async def crawl(self, seed_urls):
        # Add seed URLs
        for url in seed_urls:
            await self.add_url(url, priority=10)
        
        # Start worker pool
        workers = [
            asyncio.create_task(self.worker(i))
            for i in range(self.num_workers)
        ]
        
        await asyncio.gather(*workers)
    
    async def worker(self, worker_id: int):
        async with aiohttp.ClientSession() as session:
            while True:
                url = await self.get_next_url()
                if not url:
                    await asyncio.sleep(1)
                    continue
                
                try:
                    await self.process_url(session, url)
                except Exception as e:
                    logging.error(f"Error crawling {url}: {e}")
    
    async def process_url(self, session, url):
        domain = urlparse(url).netloc
        
        # Respect politeness - wait if needed
        await self.rate_limit(domain)
        
        # Fetch page
        async with session.get(url, timeout=10) as response:
            if response.status != 200:
                return
            
            content = await response.text()
            
            # Store content
            url_hash = hashlib.sha256(url.encode()).hexdigest()
            await self.s3.put(f"pages/{url_hash}.html", content)
            
            # Extract and queue links
            soup = BeautifulSoup(content, 'html.parser')
            for link in soup.find_all('a', href=True):
                absolute_url = urljoin(url, link['href'])
                await self.add_url(absolute_url, priority=1)
    
    async def add_url(self, url: str, priority: int):
        # Deduplication check
        url_hash = hashlib.sha256(url.encode()).hexdigest()
        
        if await self.redis.sadd('seen_urls', url_hash):
            # New URL - add to queue
            await self.redis.zadd('url_queue', {url: priority})
    
    async def rate_limit(self, domain: str):
        """Ensure politeness - max 1 req/sec per domain"""
        key = f"crawler:ratelimit:{domain}"
        while not await self.redis.set(key, '1', ex=1, nx=True):
            await asyncio.sleep(0.1)
```

**Challenges and Solutions:**

| Challenge | Solution |
|-----------|----------|
| Duplicate URLs | Bloom filter + Redis set for URL deduplication |
| Politeness | Per-domain rate limiter + robots.txt parser |
| Infinite loops | Max depth limit, URL normalization |
| Malicious sites | Timeout, content-type check, size limits |
| Priority | PageRank-style scoring for URL queue |

## Scenarios 6-10: Notification Systems

### Scenario 6: Design a Notification System

**Requirements:**
- 100M users
- Push notifications (mobile, web, email, SMS)
- Real-time delivery
- Preferences and priority

**Architecture:**

```python
class NotificationService:
    def __init__(self, queue, device_registry, preference_service):
        self.queue = queue  # Kafka/SQS
        self.devices = device_registry
        self.preferences = preference_service
        self.providers = {
            'push': PushProvider(),
            'email': EmailProvider(),
            'sms': SMSProvider()
        }
    
    async def send(self, user_id: str, notification: dict):
        # Check user preferences
        prefs = await self.preferences.get(user_id)
        if not prefs.allows(notification.type):
            return
        
        # Get user devices
        devices = await self.devices.get(user_id)
        
        # Fan out to appropriate channels
        tasks = []
        for device in devices:
            channel = self._select_channel(device, notification.priority)
            tasks.append(self._send_to_channel(channel, device, notification))
        
        await asyncio.gather(*tasks, return_exceptions=True)
    
    async def _send_to_channel(self, channel, device, notification):
        provider = self.providers[channel]
        
        # Rate limiting
        if not await self._check_rate_limit(device.user_id, channel):
            return
        
        # Template rendering
        message = self._render_template(notification, device.locale)
        
        # Send with retry
        for attempt in range(3):
            try:
                await provider.send(device, message)
                break
            except Exception as e:
                if attempt == 2:
                    await self._log_failure(device, notification, e)
```

**Delivery Guarantees:**

| Priority | Channels | Retry | Use Case |
|----------|----------|-------|----------|
| Critical | All | Yes, 3x | Security alerts |
| High | Push + Email | Yes, 2x | Payment confirmations |
| Medium | Push | Once | Social interactions |
| Low | Email digest | Batch | Marketing |

## Scenarios 11-50: Quick Reference

Due to length constraints, here's a structured overview of the remaining 40 scenarios:

### Data-Intensive Systems (11-20)

**11. Design a News Feed (Facebook/Twitter)**
- Fan-out on write vs read
- Timeline generation with ranking
- Real-time updates with WebSocket

**12. Design a Chat System**
- Message delivery guarantees
- Online presence tracking
- Group chat scaling

**13. Design a Search Autocomplete**
- Trie data structure
- Prefix matching optimization
- Personalization and ranking

**14. Design YouTube/Netflix**
- Video upload pipeline
- Adaptive bitrate streaming
- CDN architecture

**15. Design Google Drive**
- File chunking and delta sync
- Conflict resolution
- Metadata vs content storage

**16. Design a Proximity Service (Yelp)**
- Geohashing
- QuadTree for spatial indexing
- Real-time location updates

**17. Design Nearby Friends**
- WebSocket for location streaming
- Privacy and battery concerns
- Efficient distance calculation

**18. Design Google Maps**
- Graph algorithms for routing
- Real-time traffic integration
- Map tile rendering

**19. Design a Metrics Monitoring System**
- Time-series data storage
- Aggregation strategies
- Alert evaluation

**20. Design an Ad Click Aggregator**
- Real-time analytics pipeline
- Lambda architecture
- Fraud detection

### Infrastructure Systems (21-30)

**21. Design a Distributed Message Queue**
- Partition strategies
- Consumer groups
- Exactly-once delivery

**22. Design a Distributed Cache**
- Eviction policies (LRU, LFU)
- Cache warming strategies
- Thundering herd problem

**23. Design a CDN**
- Edge server placement
- Origin shield
- Purge mechanisms

**24. Design a Load Balancer**
- Layer 4 vs Layer 7
- Health checks
- Session affinity

**25. Design a Service Discovery System**
- Gossip protocols
- Health checking
- DNS vs Service mesh

**26. Design a Distributed File System**
- Block vs object storage
- Replication strategies
- Consistency models

**27. Design a Distributed Lock**
- Redlock algorithm
- Lease-based locks
- Fencing tokens

**28. Design a Job Scheduler**
- Priority queues
- Dead letter queues
- Idempotency

**29. Design an API Gateway**
- Request routing
- Authentication/authorization
- Rate limiting aggregation

**30. Design a Configuration Service**
- Versioning
- Rollback mechanisms
- Dynamic updates

### E-Commerce & Booking (31-40)

**31. Design Uber/Lyft**
- Real-time matching
- Surge pricing
- ETA calculation

**32. Design a Hotel Reservation System**
- Inventory management
- Overbooking strategies
- Concurrent booking handling

**33. Design a Ticketing System**
- Seat selection with locking
- Fairness in queuing
- Scalping prevention

**34. Design a Food Delivery App**
- Order matching optimization
- Real-time tracking
- Driver dispatch algorithm

**35. Design an E-Commerce Platform**
- Inventory consistency
- Shopping cart persistence
- Order processing pipeline

**36. Design a Payment System**
- Idempotency (reference your previous post!)
- Reconciliation
- Fraud detection

**37. Design a Stock Exchange**
- Order matching engine
- Market data distribution
- Low-latency requirements

**38. Design Airbnb**
- Search with filters
- Availability calendar
- Booking conflicts

**39. Design a Coupon System**
- Unique code generation
- Redemption tracking
- Fraud prevention

**40. Design a Wallet System**
- Double-entry bookkeeping
- Transaction atomicity
- Balance consistency

### Social & Gaming (41-50)

**41. Design Instagram**
- Image upload and processing
- Feed generation
- Story ephemeral storage

**42. Design TikTok**
- Video recommendation algorithm
- Infinite scroll optimization
- Creator analytics

**43. Design LinkedIn**
- Connection graph storage
- Job recommendations
- Skill endorsements

**44. Design a Dating App**
- Matching algorithm
- Geolocation privacy
- Like/match mechanics

**45. Design a Multiplayer Game**
- Server authoritative model
- State synchronization
- Lag compensation

**46. Design a Leaderboard System**
- Real-time ranking updates
- Tie-breaking strategies
- Sharding by region

**47. Design Reddit**
- Voting and ranking
- Comment threading
- Content moderation at scale

**48. Design a Polling System**
- Vote aggregation
- Result caching
- Duplicate vote prevention

**49. Design a Real-Time Collaboration Tool (Google Docs)**
- Operational transformation
- Conflict-free replicated data types (CRDTs)
- Presence awareness

**50. Design a Code Deployment System**
- Blue-green deployments
- Canary releases
- Rollback mechanisms

## Common Patterns Across All Scenarios

### Scalability Patterns

1. **Horizontal Scaling**: Add more machines (stateless services)
2. **Vertical Scaling**: Bigger machines (databases, caches)
3. **Data Partitioning**: Shard by user_id, time, geography
4. **Caching**: Redis, CDN, application-level
5. **Asynchronous Processing**: Message queues for non-critical paths

### Reliability Patterns

1. **Replication**: Multi-region, multi-AZ
2. **Circuit Breakers**: Fail fast on dependency issues
3. **Retry with Exponential Backoff**: Handle transient failures
4. **Bulkheads**: Isolate failure domains
5. **Health Checks**: Remove unhealthy instances

### Consistency Patterns

1. **Strong Consistency**: ACID transactions (payments)
2. **Eventual Consistency**: BASE (social feeds)
3. **Causal Consistency**: Ordered operations (messaging)
4. **Read-Your-Writes**: User sees their own updates

## The System Design Interview Framework

When approaching any scenario:

**1. Requirements Clarification (5 min)**
- Functional: What features?
- Non-functional: Scale, latency, consistency?
- Assumptions: Read/write ratio, data retention?

**2. Capacity Estimation (5 min)**
- Traffic: QPS, daily active users
- Storage: Data size, retention, growth rate
- Bandwidth: Ingress/egress

**3. High-Level Design (10 min)**
- Draw major components
- Identify APIs
- Discuss data flow

**4. Deep Dives (20 min)**
- Database schema
- Scaling bottlenecks
- Failure scenarios
- Trade-offs

**5. Wrap-up (5 min)**
- Monitoring and alerting
- Deployment strategy
- Future optimizations

## Conclusion

System design mastery comes from recognizing patterns. A URL shortener shares architectural principles with a payment system (unique ID generation, high read ratio, caching). A chat system's challenges mirror those of a notification system (real-time delivery, fan-out, presence).

The 50 scenarios presented here cover the spectrum from infrastructure primitives (rate limiters, consistent hashing) to complete applications (social networks, video platforms). What makes a system "production-ready" isn't just handling the happy path—it's anticipating failures, planning for scale, and making conscious trade-offs between consistency, availability, and partition tolerance.

Start simple. Build incrementally. Measure continuously. Every billion-user system began as a monolith serving a handful of requests. The difference between a toy project and production system isn't the initial architecture—it's the discipline to evolve it based on real metrics and user needs.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Designing Data-Intensive Applications by Martin Kleppmann](https://dataintensive.net/) - Foundational patterns and trade-offs
- [System Design Interview by Alex Xu](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF) - Practical scenario walkthroughs
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/) - Real-world production architectures
- [High Scalability Blog](http://highscalability.com/) - How companies scale popular services
- [GitHub: System Design Primer](https://github.com/donnemartin/system-design-primer) - Comprehensive resource collection
- [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem) - Understanding distributed system trade-offs
- [Consistent Hashing](https://en.wikipedia.org/wiki/Consistent_hashing) - Load distribution in distributed systems
