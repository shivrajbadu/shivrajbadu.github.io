---
layout: post
title: "Anatomy of an AI Agent: Building Intelligent Systems with Tools, Memory, and Reasoning"
date: 2026-08-29 08:30:00 +0545
categories: [AI, Backend]
tags: [ai-agents, llm, architecture, system-design, reasoning, memory, production]
---

# Anatomy of an AI Agent: Building Intelligent Systems with Tools, Memory, and Reasoning

## Introduction

An AI agent is more than a language model with an API wrapper. It's a system that perceives its environment, reasons about goals, uses tools to take actions, and learns from outcomes. The difference between a chatbot and an agent is the difference between answering questions and solving problems.

Building production agents means understanding their anatomy—the core components that transform stateless LLMs into autonomous systems capable of multi-step reasoning, persistent memory, and reliable tool usage. This isn't theoretical computer science; it's practical engineering for systems that work when users depend on them.

This guide dissects the agent architecture from first principles, building up from perception to action, with production-ready implementations that handle the edge cases where naive approaches fail.

## The Agent Loop: Perceive, Reason, Act

Every agent follows a fundamental loop:

```
┌─────────────┐
│   Perceive  │  (Understand current state)
└──────┬──────┘
       │
┌──────▼──────┐
│   Reason    │  (Decide what to do)
└──────┬──────┘
       │
┌──────▼──────┐
│     Act     │  (Use tools, generate output)
└──────┬──────┘
       │
┌──────▼──────┐
│   Observe   │  (See results, update state)
└──────┬──────┘
       │
       └─────── Loop back to Perceive
```

**Implementation:**

```python
from typing import Optional, List, Dict
from dataclasses import dataclass

@dataclass
class Observation:
    content: str
    metadata: Dict
    timestamp: float

@dataclass
class Action:
    tool: str
    input: Dict
    reasoning: str

class AgentCore:
    def __init__(self, llm, tools, memory):
        self.llm = llm
        self.tools = {tool.name: tool for tool in tools}
        self.memory = memory
        self.max_iterations = 10
    
    def run(self, task: str) -> str:
        # Initialize episode
        self.memory.start_episode(task)
        
        for iteration in range(self.max_iterations):
            # PERCEIVE: Gather context
            context = self._perceive(task, iteration)
            
            # REASON: Decide next action
            action = self._reason(context)
            
            if action.tool == "final_answer":
                return action.input["answer"]
            
            # ACT: Execute tool
            observation = self._act(action)
            
            # OBSERVE: Store result
            self.memory.add_observation(observation)
        
        return "Max iterations reached"
    
    def _perceive(self, task: str, iteration: int) -> Dict:
        """Gather relevant context for decision-making"""
        return {
            "task": task,
            "iteration": iteration,
            "memory": self.memory.recall(task, k=5),
            "available_tools": list(self.tools.keys())
        }
    
    def _reason(self, context: Dict) -> Action:
        """Decide what action to take based on context"""
        prompt = self._build_reasoning_prompt(context)
        response = self.llm.generate(prompt)
        return self._parse_action(response)
    
    def _act(self, action: Action) -> Observation:
        """Execute the chosen action"""
        tool = self.tools[action.tool]
        try:
            result = tool.execute(action.input)
            return Observation(
                content=result,
                metadata={"success": True, "tool": action.tool},
                timestamp=time.time()
            )
        except Exception as e:
            return Observation(
                content=f"Error: {str(e)}",
                metadata={"success": False, "tool": action.tool},
                timestamp=time.time()
            )
```

## Component 1: Perception

Perception transforms raw inputs into structured understanding.

### Input Processing

```python
class PerceptionModule:
    def __init__(self, multimodal_encoder):
        self.encoder = multimodal_encoder
    
    def process_input(self, input_data: Dict) -> Dict:
        """Process multimodal inputs into unified representation"""
        processed = {}
        
        # Text
        if "text" in input_data:
            processed["text"] = self._clean_text(input_data["text"])
        
        # Images
        if "images" in input_data:
            processed["image_descriptions"] = [
                self.encoder.describe_image(img)
                for img in input_data["images"]
            ]
        
        # Structured data
        if "data" in input_data:
            processed["data_summary"] = self._summarize_data(
                input_data["data"]
            )
        
        return processed
    
    def _clean_text(self, text: str) -> str:
        """Normalize and clean text input"""
        # Remove excessive whitespace
        text = " ".join(text.split())
        # Truncate if needed
        if len(text) > 10000:
            text = text[:10000] + "... [truncated]"
        return text
    
    def _summarize_data(self, data: Dict) -> str:
        """Convert structured data to natural language"""
        if isinstance(data, list):
            return f"List of {len(data)} items"
        elif isinstance(data, dict):
            keys = list(data.keys())[:5]
            return f"Object with keys: {', '.join(keys)}"
        return str(data)
```

### Context Assembly

```python
class ContextAssembler:
    def __init__(self, memory, vector_db):
        self.memory = memory
        self.vector_db = vector_db
    
    def assemble_context(self, query: str, max_tokens: int = 4000) -> str:
        """Assemble relevant context within token budget"""
        components = []
        remaining_tokens = max_tokens
        
        # 1. Recent conversation history (highest priority)
        recent = self.memory.get_recent(k=5)
        recent_text = self._format_history(recent)
        recent_tokens = self._count_tokens(recent_text)
        
        if recent_tokens < remaining_tokens:
            components.append(recent_text)
            remaining_tokens -= recent_tokens
        
        # 2. Relevant long-term memory (semantic search)
        query_embedding = self._embed(query)
        relevant = self.vector_db.search(query_embedding, k=10)
        
        for memory in relevant:
            memory_text = memory["content"]
            memory_tokens = self._count_tokens(memory_text)
            
            if memory_tokens < remaining_tokens:
                components.append(memory_text)
                remaining_tokens -= memory_tokens
            
            if remaining_tokens < 500:  # Reserve space for response
                break
        
        # 3. System instructions (always included)
        components.insert(0, self.system_prompt)
        
        return "\n\n".join(components)
```

{% include inarticle-adsense.html %}

## Component 2: Memory Systems

Memory transforms agents from stateless responders to learning systems.

### Short-Term Memory (Working Memory)

```python
from collections import deque

class WorkingMemory:
    """Maintains recent context within a window"""
    
    def __init__(self, max_items: int = 20):
        self.buffer = deque(maxlen=max_items)
        self.max_tokens = 8000
    
    def add(self, item: Dict):
        """Add item to working memory"""
        self.buffer.append(item)
        
        # Trim if exceeds token limit
        while self._total_tokens() > self.max_tokens:
            self.buffer.popleft()
    
    def get_recent(self, k: int = 5) -> List[Dict]:
        """Get k most recent items"""
        return list(self.buffer)[-k:]
    
    def summarize_old(self):
        """Summarize and compress old memories"""
        if len(self.buffer) > 10:
            old_items = list(self.buffer)[:5]
            summary = self._create_summary(old_items)
            
            # Replace old items with summary
            for _ in range(5):
                self.buffer.popleft()
            
            self.buffer.appendleft({"role": "summary", "content": summary})
```

### Long-Term Memory (Episodic)

```python
class EpisodicMemory:
    """Stores and retrieves past experiences"""
    
    def __init__(self, vector_db, embedding_model):
        self.vector_db = vector_db
        self.embedding_model = embedding_model
    
    def store(self, experience: Dict):
        """Store an experience with semantic embedding"""
        # Extract key information
        text = self._format_experience(experience)
        embedding = self.embedding_model.encode(text)
        
        # Store with metadata
        self.vector_db.insert({
            "embedding": embedding,
            "text": text,
            "metadata": {
                "timestamp": experience["timestamp"],
                "success": experience.get("success", True),
                "task_type": experience.get("task_type"),
                "tools_used": experience.get("tools_used", [])
            }
        })
    
    def recall(self, query: str, filters: Dict = None, k: int = 5) -> List[Dict]:
        """Recall relevant past experiences"""
        query_embedding = self.embedding_model.encode(query)
        
        results = self.vector_db.search(
            query_embedding,
            filters=filters,
            k=k
        )
        
        return [
            {
                "content": r["text"],
                "relevance": r["score"],
                "metadata": r["metadata"]
            }
            for r in results
        ]
    
    def _format_experience(self, exp: Dict) -> str:
        """Convert experience to searchable text"""
        return f"""
Task: {exp['task']}
Actions: {', '.join(exp['actions'])}
Outcome: {exp['outcome']}
Success: {exp.get('success', 'unknown')}
        """.strip()
```

### Semantic Memory (Facts and Knowledge)

{% raw %}
```python
class SemanticMemory:
    """Stores factual knowledge and learned patterns"""
    
    def __init__(self, knowledge_graph_db):
        self.kg = knowledge_graph_db
    
    def learn_fact(self, subject: str, predicate: str, object: str):
        """Store a factual relationship"""
        self.kg.add_triple(subject, predicate, object)
    
    def query_knowledge(self, entity: str) -> List[Dict]:
        """Retrieve all known facts about an entity"""
        return self.kg.query(f"""
            SELECT ?predicate ?object
            WHERE {{
                <{entity}> ?predicate ?object
            }}
        """)
    
    def infer(self, query: str) -> List[str]:
        """Reason over knowledge graph"""
        # Use graph traversal or SPARQL for inference
        return self.kg.infer(query)
```
{% endraw %}

## Component 3: Reasoning

Reasoning is the agent's decision-making process.

### Chain-of-Thought Reasoning

```python
class ChainOfThought:
    def __init__(self, llm):
        self.llm = llm
    
    def reason(self, problem: str) -> Dict:
        """Break down problem into steps"""
        prompt = f"""
Solve this problem step by step:

Problem: {problem}

Think through this carefully:
1. What do I know?
2. What do I need to find out?
3. What steps are required?
4. What's the final answer?

Begin your step-by-step reasoning:
"""
        
        response = self.llm.generate(prompt, temperature=0.3)
        
        return {
            "reasoning": response,
            "answer": self._extract_final_answer(response)
        }
    
    def _extract_final_answer(self, reasoning: str) -> str:
        """Parse final answer from reasoning chain"""
        lines = reasoning.split('\n')
        for line in reversed(lines):
            if "answer" in line.lower():
                return line.split(':', 1)[-1].strip()
        return lines[-1].strip()
```

### Self-Reflection

```python
class ReflectionModule:
    def __init__(self, llm):
        self.llm = llm
    
    def reflect(self, task: str, attempt: str, result: str) -> Dict:
        """Critique own performance and suggest improvements"""
        prompt = f"""
Task: {task}
My attempt: {attempt}
Result: {result}

Reflect on this:
1. Did I solve the task correctly?
2. What worked well?
3. What could be improved?
4. Should I try a different approach?

Provide honest self-critique:
"""
        
        critique = self.llm.generate(prompt)
        
        return {
            "critique": critique,
            "should_retry": self._should_retry(critique),
            "suggested_improvements": self._extract_improvements(critique)
        }
```

### Planning

```python
class Planner:
    def __init__(self, llm):
        self.llm = llm
    
    def create_plan(self, goal: str, constraints: Dict) -> List[Dict]:
        """Generate step-by-step plan to achieve goal"""
        prompt = f"""
Goal: {goal}
Constraints: {json.dumps(constraints)}

Create a detailed plan:
- Break down into concrete steps
- Identify required tools
- Estimate time/cost for each step
- Consider failure modes

Return numbered steps:
"""
        
        response = self.llm.generate(prompt)
        steps = self._parse_plan(response)
        
        # Validate plan
        validated_steps = self._validate_plan(steps, constraints)
        
        return validated_steps
    
    def _validate_plan(self, steps: List[Dict], constraints: Dict) -> List[Dict]:
        """Ensure plan is feasible"""
        validated = []
        
        for step in steps:
            # Check if tools exist
            if step['tool'] not in self.available_tools:
                continue
            
            # Check constraints
            if self._meets_constraints(step, constraints):
                validated.append(step)
        
        return validated
```

## Component 4: Tool Usage

Tools extend agents beyond text generation.

### Tool Definition

```python
from pydantic import BaseModel, Field
from typing import Callable

class Tool(BaseModel):
    name: str
    description: str
    function: Callable
    parameters: Dict
    
    class Config:
        arbitrary_types_allowed = True
    
    def execute(self, **kwargs):
        """Execute tool with parameters"""
        try:
            # Validate parameters
            validated = self._validate_params(kwargs)
            
            # Execute with timeout
            result = self._execute_with_timeout(
                self.function,
                validated,
                timeout=30
            )
            
            return {"success": True, "result": result}
        
        except Exception as e:
            return {"success": False, "error": str(e)}
    
    def _validate_params(self, params: Dict) -> Dict:
        """Validate parameters match schema"""
        # Implement parameter validation
        return params

# Example tools
def search_web(query: str) -> str:
    """Search the web and return results"""
    # Implementation
    return results

def execute_python(code: str) -> str:
    """Execute Python code in sandbox"""
    # Sandboxed execution
    return output

def query_database(sql: str) -> List[Dict]:
    """Query database safely"""
    # Parameterized query
    return rows

tools = [
    Tool(
        name="web_search",
        description="Search the internet for information",
        function=search_web,
        parameters={"query": {"type": "string", "required": True}}
    ),
    Tool(
        name="python_execute",
        description="Execute Python code",
        function=execute_python,
        parameters={"code": {"type": "string", "required": True}}
    ),
    Tool(
        name="database_query",
        description="Query SQL database",
        function=query_database,
        parameters={"sql": {"type": "string", "required": True}}
    )
]
```

### Tool Selection

```python
class ToolSelector:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools
    
    def select_tool(self, task: str, context: Dict) -> Optional[Tool]:
        """Choose the most appropriate tool for the task"""
        tool_descriptions = "\n".join([
            f"- {t.name}: {t.description}"
            for t in self.tools
        ])
        
        prompt = f"""
Available tools:
{tool_descriptions}

Task: {task}
Context: {json.dumps(context)}

Which tool should be used? Respond with just the tool name.
If no tool is needed, respond with "none".
"""
        
        response = self.llm.generate(prompt, temperature=0)
        tool_name = response.strip().lower()
        
        for tool in self.tools:
            if tool.name == tool_name:
                return tool
        
        return None
```

### Safe Execution

```python
import subprocess
import signal
from contextlib import contextmanager

class SafeExecutor:
    """Execute tools with safety constraints"""
    
    def __init__(self, max_execution_time: int = 30):
        self.max_execution_time = max_execution_time
    
    @contextmanager
    def timeout(self, seconds: int):
        """Context manager for execution timeout"""
        def timeout_handler(signum, frame):
            raise TimeoutError("Execution exceeded time limit")
        
        original_handler = signal.signal(signal.SIGALRM, timeout_handler)
        signal.alarm(seconds)
        
        try:
            yield
        finally:
            signal.alarm(0)
            signal.signal(signal.SIGALRM, original_handler)
    
    def execute_safe(self, tool: Tool, params: Dict):
        """Execute tool with safety checks"""
        # Validate inputs
        if not self._validate_input_safety(params):
            raise SecurityError("Unsafe input detected")
        
        # Execute with timeout
        try:
            with self.timeout(self.max_execution_time):
                result = tool.execute(**params)
            
            # Validate outputs
            if not self._validate_output_safety(result):
                raise SecurityError("Unsafe output detected")
            
            return result
        
        except TimeoutError:
            return {"error": "Execution timeout"}
        except Exception as e:
            return {"error": str(e)}
    
    def _validate_input_safety(self, params: Dict) -> bool:
        """Check inputs for security issues"""
        # Check for code injection attempts
        # Validate parameter types
        # Check for path traversal
        return True
    
    def _validate_output_safety(self, result: Dict) -> bool:
        """Check outputs for sensitive data"""
        # PII detection
        # Secret detection
        return True
```

## Component 5: Action Execution

### Structured Action

```python
from enum import Enum

class ActionType(Enum):
    TOOL_CALL = "tool_call"
    INTERNAL_REASONING = "internal_reasoning"
    FINAL_ANSWER = "final_answer"
    CLARIFICATION = "clarification"

@dataclass
class StructuredAction:
    type: ActionType
    content: Dict
    reasoning: str
    confidence: float
    
    def execute(self, executor):
        """Execute action through appropriate handler"""
        if self.type == ActionType.TOOL_CALL:
            return executor.execute_tool(self.content)
        elif self.type == ActionType.FINAL_ANSWER:
            return self.content["answer"]
        elif self.type == ActionType.CLARIFICATION:
            return executor.request_clarification(self.content)
        else:
            return executor.continue_reasoning()
```

### Error Handling and Recovery

```python
class ErrorRecovery:
    def __init__(self, max_retries: int = 3):
        self.max_retries = max_retries
    
    def execute_with_recovery(self, action: Action) -> Observation:
        """Execute action with automatic error recovery"""
        for attempt in range(self.max_retries):
            try:
                result = action.execute()
                return Observation(success=True, content=result)
            
            except ToolNotFoundError as e:
                # Try alternative tool
                alternative = self._find_alternative_tool(action.tool)
                if alternative:
                    action.tool = alternative
                    continue
                else:
                    return Observation(
                        success=False,
                        content=f"No tool available: {e}"
                    )
            
            except TimeoutError:
                # Reduce scope and retry
                action.input = self._reduce_scope(action.input)
                continue
            
            except ParameterError as e:
                # Fix parameters and retry
                action.input = self._fix_parameters(action.input, e)
                continue
            
            except Exception as e:
                if attempt == self.max_retries - 1:
                    return Observation(
                        success=False,
                        content=f"Failed after {self.max_retries} attempts: {e}"
                    )
        
        return Observation(success=False, content="Max retries exceeded")
```

## Putting It Together: Production Agent

```python
class ProductionAgent:
    def __init__(self, config: Dict):
        # Core components
        self.llm = self._init_llm(config)
        self.tools = self._init_tools(config)
        self.memory = self._init_memory(config)
        
        # Specialized modules
        self.perception = PerceptionModule(config["encoder"])
        self.planner = Planner(self.llm)
        self.reflector = ReflectionModule(self.llm)
        self.executor = SafeExecutor(config["timeout"])
        self.error_recovery = ErrorRecovery(config["max_retries"])
        
        # Guardrails
        self.input_filter = InputFilter()
        self.output_filter = OutputFilter()
        
        # Observability
        self.logger = StructuredLogger(config["log_config"])
        self.metrics = MetricsCollector()
    
    async def process(self, user_input: str) -> str:
        """Main entry point for agent processing"""
        request_id = uuid.uuid4()
        
        try:
            # Log request
            self.logger.info("Agent request started", {
                "request_id": request_id,
                "input_length": len(user_input)
            })
            
            # Filter input
            if not self.input_filter.is_safe(user_input):
                return "I cannot process that request"
            
            # Process
            with self.metrics.timer("agent_process"):
                # Perceive
                context = self.perception.process_input({"text": user_input})
                
                # Plan
                plan = self.planner.create_plan(
                    goal=user_input,
                    constraints={"max_cost": 0.50, "max_time": 60}
                )
                
                # Execute plan
                result = await self._execute_plan(plan, context)
                
                # Reflect
                critique = self.reflector.reflect(
                    task=user_input,
                    attempt=str(result),
                    result=result.get("outcome")
                )
                
                # Improve if needed
                if critique["should_retry"] and result.get("success") is False:
                    result = await self._retry_with_improvements(
                        plan,
                        critique["suggested_improvements"]
                    )
            
            # Filter output
            final_output = self.output_filter.clean(result["answer"])
            
            # Log success
            self.logger.info("Agent request completed", {
                "request_id": request_id,
                "success": True,
                "tokens_used": result.get("tokens"),
                "cost": result.get("cost")
            })
            
            return final_output
        
        except Exception as e:
            self.logger.error("Agent request failed", {
                "request_id": request_id,
                "error": str(e)
            })
            return "I encountered an error processing your request"
    
    async def _execute_plan(self, plan: List[Dict], context: Dict) -> Dict:
        """Execute planned steps"""
        results = []
        
        for step in plan:
            action = Action(
                tool=step["tool"],
                input=step["parameters"],
                reasoning=step["reasoning"]
            )
            
            observation = self.error_recovery.execute_with_recovery(action)
            results.append(observation)
            
            # Update context with observation
            self.memory.add_observation(observation)
        
        # Synthesize final answer
        answer = self._synthesize_results(results)
        
        return {
            "success": all(r.success for r in results),
            "answer": answer,
            "steps": results
        }
```

## Production Checklist

**Before deploying agents:**

✅ **Safety**
- Input validation and sanitization
- Output filtering (PII, harmful content)
- Tool whitelisting
- Execution timeouts

✅ **Reliability**
- Retry logic with exponential backoff
- Fallback models
- Circuit breakers
- Graceful degradation

✅ **Observability**
- Structured logging
- Distributed tracing
- Cost tracking per request
- Tool usage metrics

✅ **Performance**
- Response streaming
- Parallel tool execution
- Caching (embeddings, tool results)
- Context window management

✅ **Cost Management**
- Budget limits per user
- Cheaper models for simple tasks
- Aggressive caching
- Monitor and alert on anomalies

## Conclusion

Building production AI agents requires more than prompt engineering. It demands a systematic architecture: perception modules that process inputs, memory systems that enable learning, reasoning engines that make decisions, and execution frameworks that safely use tools.

The agents that succeed in production aren't the ones with the latest model or most sophisticated prompts—they're the ones with robust error handling, proper guardrails, comprehensive observability, and clear separation of concerns.

Start simple: build the agent loop, add memory, integrate tools incrementally, and layer in safety guardrails. The complexity emerges from composition, not from over-engineering individual components.

{% include inarticle-adsense.html %}

## Suggested Reading

- [The AI Agent Framework](https://lilianweng.github.io/posts/2023-06-23-agent/) - Comprehensive overview by Lilian Weng
- [ReAct Paper: Reasoning and Acting](https://arxiv.org/abs/2210.03629) - Original agent pattern research
- [LangChain Agent Documentation](https://python.langchain.com/docs/modules/agents/) - Practical implementations
- [Anthropic: Building Effective Agents](https://www.anthropic.com/index/building-effective-agents) - Production best practices
- [Memory in AI Systems](https://arxiv.org/abs/2304.03442) - Memory architecture patterns
- [Tool Use in Language Models](https://arxiv.org/abs/2302.04761) - Academic foundations
