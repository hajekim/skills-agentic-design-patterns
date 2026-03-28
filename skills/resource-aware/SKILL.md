---
name: resource-aware
description: This skill should be used when the user wants to "optimize agent resource usage", "token budget management", "cost-aware agents", "efficient LLM calls", "agent cost optimization", "reduce API costs", "agent performance optimization", "latency optimization", "token counting", "model selection strategy", "compute budget for agents", "rate limit handling", "reduce token usage", "cost-efficient agent", "cheap LLM calls", "optimize LLM costs", "frugal agent", "budget-constrained agent", or build agents that efficiently manage computational resources, API costs, and latency constraints. Also responds to Korean: "에이전트 리소스 최적화", "토큰 예산 관리", "비용 인식 에이전트", "API 비용 절감", "지연 시간 최적화", "효율적인 LLM 호출", "비용 절약", "토큰 절약", "API 비용 줄이기", "토큰 아껴줘", "비용 최소화해줘", "저렴하게 에이전트 돌리기". Also responds to Japanese: "エージェントのリソース最適化", "トークン予算管理", "コスト最適化", "APIコスト削減", "トークンを節約したい", "費用を最小化したい", "効率的なLLM呼び出し", "遅延時間最適化", "安くエージェントを動かしたい", "モデル選択戦略" Also responds to Chinese: "智能体资源优化", "令牌预算管理", "成本优化", "降低API成本", "节省令牌", "最小化费用", "高效LLM调用", "降低延迟", "低成本运行智能体", "模型选择策略".. Apply this skill to design or implement the Resource-Aware Optimization agentic design pattern.
version: 1.0.0
---

# Resource-Aware Optimization Pattern

## Overview

The **Resource-Aware Optimization Pattern** equips agents with awareness of the computational resources they consume — tokens, API calls, latency, and cost — and strategies to optimize their use. Rather than calling the most capable model for every request or retrieving maximum context every time, resource-aware agents make intelligent trade-offs to achieve goals within budget constraints.

**Core Principle:** Capability without efficiency is waste — match resource consumption to task requirements.

## When This Skill Applies

Activate this pattern when:
- API costs need to stay within budgets (production deployments at scale)
- Latency requirements are strict (real-time user interactions)
- Token limits constrain context window usage
- Rate limits throttle request throughput
- Different sub-tasks have vastly different complexity requirements
- Long-running batch processes must manage resource consumption over time

**Rule of thumb:** If cost, latency, or quota is a constraint — build resource awareness in from the start, not as an afterthought.

## Resource Dimensions

| Resource | Constraint | Optimization Strategy |
|----------|------------|----------------------|
| **Tokens (input)** | Context window limit | Summarize, truncate, selective retrieval |
| **Tokens (output)** | Cost, latency | Precise prompting, structured output |
| **API calls** | Rate limits, cost | Caching, batching, model selection |
| **Latency** | User experience | Async, streaming, parallel calls |
| **Compute** | Server cost | Efficient inference, model cascade |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map resource constraints:
1. What are the token budget limits per session/task?
2. What is the cost budget? (Per query, per day, per user)
3. What are the latency requirements? (P50, P95, P99 targets)
4. What rate limits apply? (Requests per minute, tokens per minute)
5. Which sub-tasks are latency-sensitive vs. batch-OK?

### PLAN
Design resource optimization architecture:
1. Define model selection policy: capability vs. cost trade-off by task type
2. Design caching strategy: what to cache, TTL, cache invalidation
3. Plan context management: how to stay within token limits
4. Design batching: group independent requests to amortize overhead
5. Build monitoring: track resource usage against budgets in real-time

### ACTION
Implement resource-aware agent:
1. Add token counting before every LLM call
2. Implement model cascade: try cheaper model first, escalate if needed
3. Add response caching for repeated or similar queries
4. Implement context summarization when approaching token limits
5. Log resource usage metrics for monitoring and optimization

## Implementation: Token Budget Management

### Token Counter and Budget Tracker
```python
from google import genai
from google.genai import types
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class ResourceBudget:
    """Tracks resource consumption against defined budgets."""
    max_input_tokens: int = 100_000
    max_output_tokens: int = 50_000
    max_api_calls: int = 100
    max_cost_usd: float = 1.0

    used_input_tokens: int = 0
    used_output_tokens: int = 0
    used_api_calls: int = 0
    estimated_cost_usd: float = 0.0

    # Pricing (approximate, check current pricing)
    input_cost_per_1k: float = 0.00015   # Gemini Flash input
    output_cost_per_1k: float = 0.0006   # Gemini Flash output

    def record_usage(self, input_tokens: int, output_tokens: int):
        """Record token usage and update cost estimate."""
        self.used_input_tokens += input_tokens
        self.used_output_tokens += output_tokens
        self.used_api_calls += 1
        self.estimated_cost_usd += (
            input_tokens / 1000 * self.input_cost_per_1k +
            output_tokens / 1000 * self.output_cost_per_1k
        )

    @property
    def input_budget_remaining(self) -> int:
        return self.max_input_tokens - self.used_input_tokens

    @property
    def cost_budget_remaining(self) -> float:
        return self.max_cost_usd - self.estimated_cost_usd

    @property
    def is_within_budget(self) -> bool:
        return (
            self.used_input_tokens < self.max_input_tokens and
            self.used_api_calls < self.max_api_calls and
            self.estimated_cost_usd < self.max_cost_usd
        )

    def summary(self) -> dict:
        return {
            "tokens_used": f"{self.used_input_tokens:,} in / {self.used_output_tokens:,} out",
            "api_calls": f"{self.used_api_calls}/{self.max_api_calls}",
            "estimated_cost": f"${self.estimated_cost_usd:.4f} / ${self.max_cost_usd:.2f}",
            "within_budget": self.is_within_budget
        }

class ResourceAwareAgent:
    """Agent that tracks and respects resource budgets."""

    def __init__(self, budget: ResourceBudget):
        self.budget = budget
        self.client = genai.Client()

    def generate(self, prompt: str, max_output_tokens: int = 1000) -> Optional[str]:
        """Generate response only if budget allows."""
        if not self.budget.is_within_budget:
            return f"Budget exceeded: {self.budget.summary()}"

        response = self.client.models.generate_content(
            model='gemini-2.5-flash',
            contents=prompt,
            config=types.GenerateContentConfig(max_output_tokens=max_output_tokens)
        )

        # Record actual usage
        if hasattr(response, 'usage_metadata'):
            self.budget.record_usage(
                response.usage_metadata.prompt_token_count,
                response.usage_metadata.candidates_token_count
            )

        return response.text
```

### Model Cascade (Cost Optimization)
```python
from google import genai
from typing import Callable

client = genai.Client()

class ModelCascade:
    """Try cheaper models first, escalate to more capable ones only when needed."""

    def __init__(self):
        self.models = [
            ("gemini-2.5-flash", "fast and cheap"),
            ("gemini-2.5-flash", "reasoning tasks (high Thinking Budget)"),
            ("gemini-1.5-pro", "complex, long-context"),
        ]

    def generate_with_cascade(
        self,
        prompt: str,
        quality_checker: Callable[[str], bool],
        max_escalations: int = 2
    ) -> dict:
        """Try models from cheapest to most capable until quality threshold met.

        Args:
            prompt: The prompt to generate a response for
            quality_checker: Function that returns True if response is acceptable
            max_escalations: Maximum number of model upgrades to try

        Returns:
            Response with model used and escalation count
        """
        escalations = 0

        for model_name, model_description in self.models[:max_escalations + 1]:
            response = client.models.generate_content(model=model_name, contents=prompt)

            if quality_checker(response.text):
                return {
                    "response": response.text,
                    "model_used": model_name,
                    "escalations": escalations,
                    "status": "success"
                }

            escalations += 1
            if escalations <= max_escalations:
                print(f"Quality check failed for {model_name}, escalating to next model...")

        # Return best attempt if quality never satisfied
        return {
            "response": response.text,
            "model_used": model_name,
            "escalations": escalations,
            "status": "best_effort"
        }

def simple_quality_check(response: str, min_length: int = 100) -> bool:
    """Basic quality check: response must be sufficiently long."""
    return len(response) >= min_length and not response.startswith("I cannot")

# Usage
cascade = ModelCascade()
result = cascade.generate_with_cascade(
    "Explain the implications of quantum computing for cryptography",
    quality_checker=lambda r: simple_quality_check(r, min_length=200)
)
print(f"Used: {result['model_used']} (escalations: {result['escalations']})")
```

### Response Caching
```python
import hashlib
import json
import time
from typing import Optional

class ResponseCache:
    """Cache LLM responses to avoid redundant API calls."""

    def __init__(self, ttl_seconds: int = 3600, max_size: int = 1000):
        self._cache: dict = {}
        self.ttl = ttl_seconds
        self.max_size = max_size
        self.hits = 0
        self.misses = 0

    def _make_key(self, prompt: str, model: str, params: dict) -> str:
        """Create a deterministic cache key."""
        content = json.dumps({"prompt": prompt, "model": model, "params": params}, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()

    def get(self, prompt: str, model: str, params: dict = {}) -> Optional[str]:
        """Retrieve cached response if available and not expired."""
        key = self._make_key(prompt, model, params)
        if key in self._cache:
            entry = self._cache[key]
            if time.time() - entry["timestamp"] < self.ttl:
                self.hits += 1
                return entry["response"]
            else:
                del self._cache[key]  # Expired
        self.misses += 1
        return None

    def set(self, prompt: str, model: str, response: str, params: dict = {}):
        """Store a response in the cache."""
        if len(self._cache) >= self.max_size:
            # Evict oldest entry (simple LRU approximation)
            oldest_key = min(self._cache, key=lambda k: self._cache[k]["timestamp"])
            del self._cache[oldest_key]

        key = self._make_key(prompt, model, params)
        self._cache[key] = {
            "response": response,
            "timestamp": time.time()
        }

    @property
    def hit_rate(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0.0

    def stats(self) -> dict:
        return {
            "size": len(self._cache),
            "hits": self.hits,
            "misses": self.misses,
            "hit_rate": f"{self.hit_rate:.1%}"
        }

# Cached agent
cache = ResponseCache(ttl_seconds=1800)

def cached_generate(prompt: str, model_name: str = "gemini-2.5-flash") -> str:
    """Generate with caching to avoid repeated identical calls."""
    cached = cache.get(prompt, model_name)
    if cached:
        return cached

    response = client.models.generate_content(model=model_name, contents=prompt)
    cache.set(prompt, model_name, response.text)
    return response.text
```

### Context Window Management
```python
def manage_context_window(
    messages: list,
    max_tokens: int,
    client,
    reserve_for_response: int = 1000
) -> list:
    """Trim message history to stay within token budget.

    Strategy: Keep system message + recent messages + summarize middle.
    """
    # Estimate tokens (rough: 4 chars ≈ 1 token)
    def estimate_tokens(text: str) -> int:
        return len(text) // 4

    effective_limit = max_tokens - reserve_for_response
    total_tokens = sum(estimate_tokens(m.get("content", "")) for m in messages)

    if total_tokens <= effective_limit:
        return messages  # Already within limits

    # Strategy: Keep first (system), last 5 messages, summarize the rest
    system_msgs = [m for m in messages if m.get("role") == "system"]
    recent_msgs = messages[-5:]
    middle_msgs = messages[len(system_msgs):-5]

    if middle_msgs:
        # Summarize middle portion
        middle_text = "\n".join([f"{m['role']}: {m['content']}" for m in middle_msgs])
        summary_response = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=f"Summarize this conversation segment concisely:\n{middle_text}"
        )
        summary_msg = {
            "role": "system",
            "content": f"[Earlier conversation summary]: {summary_response.text}"
        }
        return system_msgs + [summary_msg] + recent_msgs

    # If no middle, just keep system + recent
    return system_msgs + recent_msgs
```

## Key Takeaways

- **Budget-first design**: Define token, cost, and rate budgets before building — retrofit is painful
- **Model cascade**: Use cheap models for simple tasks, escalate only when quality requires it
- **Cache aggressively**: Many agent queries are repeated or semantically equivalent — cache them
- **Context pruning**: Summarize or truncate old context rather than exceeding token limits
- **Measure everything**: You can't optimize what you don't measure — log all resource usage
- **Async for throughput**: Parallelize independent calls to maximize throughput within rate limits

## Anti-Patterns to Avoid

- **Max model always**: Using the most capable model for every task is expensive and often unnecessary
- **Unbounded context**: Growing message history without pruning hits token limits and increases cost
- **No caching**: Repeated queries to the same model waste money and latency
- **Synchronous rate limiting**: Sleeping between calls is wasteful — use async queues
- **Ignoring usage metadata**: LLM APIs return token counts — always read and track them

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Cost-optimized production agent | Resource-Aware + **Exception Handling** + **Evaluation** |
| Budget-aware parallel execution | Resource-Aware + **Parallelization** + **Prioritization** |
| Efficient long-running pipeline | Resource-Aware + **Planning** + **Goal Setting** |
| Token-optimized RAG agent | Resource-Aware + **RAG** + **Memory Management** |

## References

- Gemini API Pricing: https://ai.google.dev/pricing
- Google ADK Resource Management: https://google.github.io/adk-docs/
- Token Counting API: https://ai.google.dev/api/tokens
- LangChain Caching: https://python.langchain.com/docs/concepts/caching/
- Rate Limiting Best Practices: https://cloud.google.com/apis/design/design_patterns
