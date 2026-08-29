---
layout: post
title: "Infrastructure for Intelligence: The Modern AI Engineering Stack in Production"
date: 2026-08-29 22:10:00 +0545
categories: [AI, DevOps]
tags: [ai-infrastructure, production, llm, system-design, mlops, deployment, monitoring]
---

# Infrastructure for Intelligence: The Modern AI Engineering Stack in Production

## Introduction

Building AI applications in 2026 requires more than calling an LLM API. Production AI systems demand infrastructure that handles non-deterministic outputs, manages costs at scale, ensures quality without traditional testing, and maintains reliability despite model dependencies beyond your control.

The production AI stack has evolved rapidly from "just call OpenAI" to sophisticated architectures with model routing, vector databases, evaluation pipelines, and cost optimization layers. This guide maps the complete infrastructure stack for AI applications that serve millions of users.

This isn't theoretical architecture—it's the battle-tested stack powering production AI systems processing billions of tokens monthly.

## The Modern AI Stack Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│  (FastAPI, Rails, Next.js - your business logic)        │
└────────────────┬────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│              AI Orchestration Layer                      │
│  LangChain, LlamaIndex, Custom Agents                   │
└────────┬──────────────┬──────────────┬─────────────────┘
         │              │              │
┌────────▼────┐  ┌─────▼─────┐  ┌────▼──────────────────┐
│   LLM Layer │  │   Vector   │  │  Observability Layer  │
│ GPT-4,Claude│  │  Database  │  │ LangSmith, Weights&   │
│ Llama, Gemini│  │pgvector   │  │ Biases, Prometheus    │
└────────┬────┘  │ Qdrant    │  └───────────────────────┘
         │       └───────────┘
┌────────▼──────────────────────────────────────────────┐
│              Infrastructure Layer                      │
│ AWS/GCP/Azure, Kubernetes, Redis, PostgreSQL          │
└───────────────────────────────────────────────────────┘
```

## Layer 1: Model Infrastructure

### Model Serving

**Option A: API Providers (High Reliability, Low Control)**

```python
from litellm import completion
import os

class ModelRouter:
    """Route requests across providers for reliability"""
    
    def __init__(self):
        self.providers = [
            {"model": "gpt-4", "api_key": os.getenv("OPENAI_KEY")},
            {"model": "claude-3-opus", "api_key": os.getenv("ANTHROPIC_KEY")},
            {"model": "gemini-pro", "api_key": os.getenv("GOOGLE_KEY")}
        ]
        self.current_provider = 0
    
    def generate(self, prompt: str, max_retries: int = 3):
        """Generate with automatic failover"""
        for attempt in range(max_retries):
            try:
                provider = self.providers[self.current_provider % len(self.providers)]
                
                response = completion(
                    model=provider["model"],
                    messages=[{"role": "user", "content": prompt}],
                    api_key=provider["api_key"]
                )
                
                return response.choices[0].message.content
            
            except Exception as e:
                print(f"Provider {self.current_provider} failed: {e}")
                self.current_provider += 1
                
                if attempt == max_retries - 1:
                    raise
        
        raise RuntimeError("All providers failed")
```

**Cost Optimization:**

```python
class CostOptimizedRouter:
    """Route to cheapest available model"""
    
    COST_PER_1M_TOKENS = {
        "gpt-3.5-turbo": 0.50,
        "gpt-4": 30.00,
        "gpt-4-turbo": 10.00,
        "claude-3-haiku": 0.25,
        "claude-3-sonnet": 3.00,
        "claude-3-opus": 15.00,
    }
    
    def select_model(self, complexity: str, budget: float):
        """Select cheapest model meeting quality requirements"""
        if complexity == "simple":
            return "gpt-3.5-turbo"  # $0.50/1M tokens
        elif complexity == "medium":
            return "gpt-4-turbo"    # $10/1M tokens
        else:
            return "claude-3-opus"  # $15/1M tokens (high quality)
    
    def estimate_cost(self, prompt: str, model: str) -> float:
        """Estimate cost before making request"""
        tokens = len(prompt.split()) * 1.3  # Rough tokenization
        return (tokens / 1_000_000) * self.COST_PER_1M_TOKENS[model]
```

**Option B: Self-Hosted (High Control, More Work)**

```yaml
# docker-compose.yml for self-hosted vLLM
version: '3.8'

services:
  vllm:
    image: vllm/vllm-openai:latest
    ports:
      - "8000:8000"
    volumes:
      - ./models:/models
    environment:
      - CUDA_VISIBLE_DEVICES=0,1
    command: >
      --model meta-llama/Llama-3.1-70B-Instruct
      --tensor-parallel-size 2
      --max-model-len 8192
      --gpu-memory-utilization 0.95
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 2
              capabilities: [gpu]
```

**Load Balancing Self-Hosted Models:**

```python
from fastapi import FastAPI
import httpx
import asyncio

app = FastAPI()

class ModelLoadBalancer:
    def __init__(self, model_endpoints: list[str]):
        self.endpoints = model_endpoints
        self.current = 0
    
    async def generate(self, prompt: str):
        """Round-robin across model instances"""
        endpoint = self.endpoints[self.current % len(self.endpoints)]
        self.current += 1
        
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{endpoint}/v1/completions",
                json={"prompt": prompt, "max_tokens": 512},
                timeout=30.0
            )
            return response.json()

# Initialize with multiple vLLM instances
balancer = ModelLoadBalancer([
    "http://vllm-1:8000",
    "http://vllm-2:8000",
    "http://vllm-3:8000"
])

@app.post("/generate")
async def generate_endpoint(prompt: str):
    return await balancer.generate(prompt)
```

{% include inarticle-adsense.html %}

## Layer 2: Vector Database Infrastructure

### PostgreSQL + pgvector (Start Here)

```sql
-- Setup pgvector
CREATE EXTENSION vector;

CREATE TABLE embeddings (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536),  -- OpenAI embedding dimension
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- HNSW index for fast similarity search
CREATE INDEX ON embeddings USING hnsw (embedding vector_cosine_ops);

-- Partition by date for large datasets
CREATE TABLE embeddings_2026_01 PARTITION OF embeddings
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

**Python Integration:**

```python
import asyncpg
from openai import OpenAI
import numpy as np

class VectorStore:
    def __init__(self, database_url: str):
        self.pool = None
        self.database_url = database_url
        self.openai = OpenAI()
    
    async def connect(self):
        self.pool = await asyncpg.create_pool(self.database_url)
    
    async def upsert(self, content: str, metadata: dict = None):
        """Insert document with embedding"""
        # Generate embedding
        embedding = self.openai.embeddings.create(
            input=content,
            model="text-embedding-3-small"
        ).data[0].embedding
        
        # Store in database
        async with self.pool.acquire() as conn:
            await conn.execute("""
                INSERT INTO embeddings (content, embedding, metadata)
                VALUES ($1, $2, $3)
            """, content, embedding, metadata or {})
    
    async def search(self, query: str, limit: int = 5):
        """Semantic search"""
        # Generate query embedding
        query_embedding = self.openai.embeddings.create(
            input=query,
            model="text-embedding-3-small"
        ).data[0].embedding
        
        # Search by cosine similarity
        async with self.pool.acquire() as conn:
            results = await conn.fetch("""
                SELECT 
                    content,
                    metadata,
                    1 - (embedding <=> $1) AS similarity
                FROM embeddings
                ORDER BY embedding <=> $1
                LIMIT $2
            """, query_embedding, limit)
        
        return [dict(row) for row in results]
```

### Qdrant for Scale (>10M vectors)

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

class ScalableVectorStore:
    def __init__(self):
        self.client = QdrantClient(host="localhost", port=6333)
        self.collection = "documents"
        
        # Create collection with optimized settings
        self.client.create_collection(
            collection_name=self.collection,
            vectors_config=VectorParams(
                size=1536,
                distance=Distance.COSINE
            ),
            # Shard across nodes for horizontal scaling
            shard_number=4,
            replication_factor=2,
            # Optimize for high-throughput search
            hnsw_config={
                "m": 16,
                "ef_construct": 100
            }
        )
    
    async def batch_upsert(self, documents: list[dict], batch_size: int = 100):
        """Efficiently insert large datasets"""
        for i in range(0, len(documents), batch_size):
            batch = documents[i:i + batch_size]
            
            points = [
                PointStruct(
                    id=doc["id"],
                    vector=doc["embedding"],
                    payload={"content": doc["content"], "metadata": doc["metadata"]}
                )
                for doc in batch
            ]
            
            self.client.upsert(
                collection_name=self.collection,
                points=points
            )
```

## Layer 3: Caching Infrastructure

### Multi-Level Caching

```python
import redis
import hashlib
from functools import wraps

class AICache:
    """Multi-level cache for AI responses"""
    
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
        self.local_cache = {}  # In-memory cache
        self.local_cache_size = 1000
    
    def cache_key(self, prompt: str, model: str) -> str:
        """Generate cache key"""
        content = f"{model}:{prompt}"
        return hashlib.sha256(content.encode()).hexdigest()
    
    def get(self, prompt: str, model: str) -> str | None:
        """Get cached response"""
        key = self.cache_key(prompt, model)
        
        # Level 1: Local memory (fastest)
        if key in self.local_cache:
            return self.local_cache[key]
        
        # Level 2: Redis (shared across instances)
        cached = self.redis.get(key)
        if cached:
            # Promote to local cache
            self.local_cache[key] = cached.decode()
            return cached.decode()
        
        return None
    
    def set(self, prompt: str, model: str, response: str, ttl: int = 3600):
        """Cache response"""
        key = self.cache_key(prompt, model)
        
        # Store in Redis with TTL
        self.redis.setex(key, ttl, response)
        
        # Store in local cache
        if len(self.local_cache) >= self.local_cache_size:
            # Evict random item
            self.local_cache.pop(next(iter(self.local_cache)))
        
        self.local_cache[key] = response

def cached_llm_call(cache: AICache):
    """Decorator for caching LLM calls"""
    def decorator(func):
        @wraps(func)
        async def wrapper(prompt: str, model: str = "gpt-4", **kwargs):
            # Check cache
            cached = cache.get(prompt, model)
            if cached:
                return {"response": cached, "from_cache": True, "cost": 0.0}
            
            # Call LLM
            result = await func(prompt, model, **kwargs)
            
            # Cache result
            cache.set(prompt, model, result["response"])
            
            return {**result, "from_cache": False}
        
        return wrapper
    return decorator
```

### Semantic Caching

```python
class SemanticCache:
    """Cache based on semantic similarity, not exact match"""
    
    def __init__(self, vector_store, threshold: float = 0.95):
        self.vector_store = vector_store
        self.threshold = threshold
    
    async def get(self, prompt: str) -> str | None:
        """Find semantically similar cached response"""
        results = await self.vector_store.search(prompt, limit=1)
        
        if results and results[0]["similarity"] >= self.threshold:
            return results[0]["metadata"]["response"]
        
        return None
    
    async def set(self, prompt: str, response: str):
        """Cache prompt-response pair"""
        await self.vector_store.upsert(
            content=prompt,
            metadata={"response": response, "timestamp": time.time()}
        )
```

## Layer 4: Observability

### Structured Logging

```python
import structlog
import time
from contextlib import contextmanager

logger = structlog.get_logger()

class AILogger:
    """Structured logging for AI operations"""
    
    @contextmanager
    def log_llm_call(self, model: str, prompt_tokens: int):
        """Log LLM call with timing and cost"""
        start = time.time()
        request_id = generate_request_id()
        
        logger.info(
            "llm_call_started",
            request_id=request_id,
            model=model,
            prompt_tokens=prompt_tokens
        )
        
        try:
            yield request_id
            
            latency = time.time() - start
            logger.info(
                "llm_call_completed",
                request_id=request_id,
                latency_seconds=latency,
                success=True
            )
        
        except Exception as e:
            latency = time.time() - start
            logger.error(
                "llm_call_failed",
                request_id=request_id,
                latency_seconds=latency,
                error=str(e),
                success=False
            )
            raise
```

### Metrics Collection

```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
llm_requests_total = Counter(
    'llm_requests_total',
    'Total LLM requests',
    ['model', 'status']
)

llm_latency_seconds = Histogram(
    'llm_latency_seconds',
    'LLM request latency',
    ['model']
)

llm_cost_dollars = Counter(
    'llm_cost_dollars_total',
    'Total LLM cost',
    ['model']
)

llm_tokens_total = Counter(
    'llm_tokens_total',
    'Total tokens processed',
    ['model', 'type']  # type: prompt or completion
)

active_llm_requests = Gauge(
    'llm_requests_active',
    'Currently active LLM requests'
)

class MetricsCollector:
    """Collect AI-specific metrics"""
    
    @contextmanager
    def track_request(self, model: str):
        """Track LLM request metrics"""
        active_llm_requests.inc()
        start = time.time()
        
        try:
            yield
            
            latency = time.time() - start
            llm_requests_total.labels(model=model, status="success").inc()
            llm_latency_seconds.labels(model=model).observe(latency)
        
        except Exception:
            llm_requests_total.labels(model=model, status="error").inc()
            raise
        
        finally:
            active_llm_requests.dec()
    
    def record_cost(self, model: str, prompt_tokens: int, completion_tokens: int):
        """Record token usage and cost"""
        llm_tokens_total.labels(model=model, type="prompt").inc(prompt_tokens)
        llm_tokens_total.labels(model=model, type="completion").inc(completion_tokens)
        
        cost = self._calculate_cost(model, prompt_tokens, completion_tokens)
        llm_cost_dollars.labels(model=model).inc(cost)
```

### Distributed Tracing

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

span_processor = BatchSpanProcessor(OTLPSpanExporter())
trace.get_tracer_provider().add_span_processor(span_processor)

class TracedAIService:
    """AI service with distributed tracing"""
    
    async def process_query(self, query: str):
        with tracer.start_as_current_span("process_query") as span:
            span.set_attribute("query.length", len(query))
            
            # Trace vector search
            with tracer.start_as_current_span("vector_search"):
                docs = await self.vector_search(query)
                span.set_attribute("docs.found", len(docs))
            
            # Trace LLM call
            with tracer.start_as_current_span("llm_generate") as llm_span:
                llm_span.set_attribute("model", "gpt-4")
                response = await self.llm_generate(query, docs)
                llm_span.set_attribute("response.length", len(response))
            
            return response
```

## Layer 5: Evaluation Pipeline

### Continuous Evaluation

```python
class EvaluationPipeline:
    """Continuous evaluation of AI outputs"""
    
    def __init__(self, test_set: list[dict]):
        self.test_set = test_set
        self.llm_judge = OpenAI()
    
    async def evaluate_model(self, model: str) -> dict:
        """Run full evaluation"""
        results = {
            "accuracy": [],
            "relevance": [],
            "safety": [],
            "latency": []
        }
        
        for test in self.test_set:
            start = time.time()
            
            # Generate response
            response = await self.generate(test["input"], model)
            latency = time.time() - start
            
            # Evaluate quality
            accuracy = await self._check_accuracy(
                response,
                test["expected"]
            )
            relevance = await self._check_relevance(
                test["input"],
                response
            )
            safety = await self._check_safety(response)
            
            results["accuracy"].append(accuracy)
            results["relevance"].append(relevance)
            results["safety"].append(safety)
            results["latency"].append(latency)
        
        return {
            "accuracy": np.mean(results["accuracy"]),
            "relevance": np.mean(results["relevance"]),
            "safety": np.mean(results["safety"]),
            "p95_latency": np.percentile(results["latency"], 95)
        }
    
    async def _check_accuracy(self, response: str, expected: str) -> float:
        """LLM-as-judge for accuracy"""
        prompt = f"""Rate the accuracy of this response on a scale of 0-1.

Expected: {expected}
Actual: {response}

Respond with just a number between 0 and 1."""
        
        score = await self.llm_judge.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0
        )
        
        return float(score.choices[0].message.content)
```

## Layer 6: Cost Management

### Budget Tracking

```python
class BudgetManager:
    """Track and enforce AI spending limits"""
    
    def __init__(self, redis_client):
        self.redis = redis_client
    
    async def check_budget(self, user_id: str, estimated_cost: float) -> bool:
        """Check if user has budget for request"""
        key = f"budget:{user_id}:monthly"
        
        # Get current spend
        current_spend = float(self.redis.get(key) or 0)
        user_limit = await self._get_user_limit(user_id)
        
        # Check if within budget
        if current_spend + estimated_cost > user_limit:
            return False
        
        # Reserve budget
        pipe = self.redis.pipeline()
        pipe.incrbyfloat(key, estimated_cost)
        pipe.expire(key, 30 * 24 * 3600)  # Monthly
        pipe.execute()
        
        return True
    
    async def record_actual_cost(self, user_id: str, actual_cost: float):
        """Reconcile actual cost"""
        # Update metrics
        self.redis.incrbyfloat(f"cost:actual:{user_id}", actual_cost)
```

## Complete Stack Example

```python
# app/main.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class AIStack:
    """Complete production AI stack"""
    
    def __init__(self):
        # Model layer
        self.model_router = ModelRouter()
        
        # Vector layer
        self.vector_store = VectorStore(os.getenv("DATABASE_URL"))
        
        # Cache layer
        self.cache = AICache(os.getenv("REDIS_URL"))
        self.semantic_cache = SemanticCache(self.vector_store)
        
        # Observability
        self.logger = AILogger()
        self.metrics = MetricsCollector()
        self.tracer = TracedAIService()
        
        # Cost management
        self.budget = BudgetManager(redis_client)
        
        # Evaluation
        self.evaluator = EvaluationPipeline(test_set)
    
    async def query(self, question: str, user_id: str) -> dict:
        """Process AI query through full stack"""
        with self.metrics.track_request("gpt-4"):
            # Check budget
            estimated_cost = self._estimate_cost(question)
            if not await self.budget.check_budget(user_id, estimated_cost):
                raise HTTPException(429, "Budget exceeded")
            
            # Check cache
            if cached := self.cache.get(question, "gpt-4"):
                return {"answer": cached, "from_cache": True}
            
            # RAG: Retrieve context
            docs = await self.vector_store.search(question)
            context = "\n\n".join([d["content"] for d in docs])
            
            # Generate
            prompt = f"Context:\n{context}\n\nQuestion: {question}"
            answer = await self.model_router.generate(prompt)
            
            # Cache
            self.cache.set(question, "gpt-4", answer)
            
            # Record actual cost
            actual_cost = self._calculate_actual_cost(prompt, answer)
            await self.budget.record_actual_cost(user_id, actual_cost)
            
            return {"answer": answer, "from_cache": False}

@app.post("/query")
async def query_endpoint(question: str, user_id: str):
    return await ai_stack.query(question, user_id)
```

## Deployment Architecture

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: api
        image: ai-service:latest
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: openai-key
        - name: REDIS_URL
          value: "redis://redis-master:6379"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: url
---
apiVersion: v1
kind: Service
metadata:
  name: ai-service
spec:
  selector:
    app: ai-service
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

## Conclusion

The modern AI engineering stack is more than LLM APIs—it's a complete infrastructure ecosystem handling model routing, vector search, multi-level caching, distributed tracing, continuous evaluation, and cost management.

Start simple: use hosted LLM APIs, PostgreSQL with pgvector, Redis for caching. As you scale, add Qdrant for vector search, Kubernetes for orchestration, Prometheus for metrics, and custom evaluation pipelines.

The stack will continue evolving, but the principles remain: reliability through redundancy, observability through instrumentation, cost control through caching, and quality through continuous evaluation.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Patterns for Building LLM-based Systems](https://eugeneyan.com/writing/llm-patterns/) - Production patterns
- [OpenAI Production Best Practices](https://platform.openai.com/docs/guides/production-best-practices) - Official guidelines
- [Chip Huyen: Building LLM Applications for Production](https://huyenchip.com/2023/04/11/llm-engineering.html) - System design
- [vLLM Documentation](https://docs.vllm.ai/) - High-performance serving
- [Qdrant Documentation](https://qdrant.tech/documentation/) - Vector database architecture
- [LangSmith Documentation](https://docs.smith.langchain.com/) - LLM observability
