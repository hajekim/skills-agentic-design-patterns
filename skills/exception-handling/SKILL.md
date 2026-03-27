---
name: exception-handling
description: This skill should be used when the user wants to "handle agent errors", "graceful degradation", "agent fault tolerance", "retry logic for agents", "error recovery strategies", "fallback mechanisms", "agent robustness", "exception management", "agent failure handling", "circuit breaker pattern", "tool error handling", "resilient agent", "agent recovery", "handle tool failures", "agent timeout handling", "robust agent design", "error-proof agent", or build agents that handle failures, retries, and unexpected conditions gracefully. Also responds to Korean: "에이전트 오류 처리", "우아한 성능 저하", "재시도 로직", "폴백 메커니즘", "에이전트 내결함성", "회로 차단기 패턴", "에러 처리", "예외 처리", "오류 복구", "에러 나도 계속 실행해줘", "실패 시 재시도해줘", "API 오류 대비해줘". Also responds to Japanese: "エージェントのエラー処理", "優雅なデグレード", "リトライロジック", "フォールバック機能", "エラーが出ても続けてほしい", "例外処理", "障害に強いエージェント", "API障害に対応", "サーキットブレーカー", "エラー回復" Also responds to Chinese: "智能体错误处理", "优雅降级", "重试逻辑", "回退机制", "出错了也要继续运行", "异常处理", "容错智能体", "应对API故障", "断路器模式", "错误恢复".. Apply this skill to design or implement the Exception Handling agentic design pattern.
version: 1.0.0
---

# Exception Handling Pattern

## Overview

The **Exception Handling Pattern** enables agents to detect, respond to, and recover from errors, failures, and unexpected conditions without crashing or producing incorrect outputs. Robust agents treat exceptions as information — they classify the failure, choose an appropriate recovery strategy, and continue operating or escalate gracefully.

**Core Principle:** Expect failure as a first-class event — design recovery paths before you need them.

## When This Skill Applies

Activate this pattern when:
- Agents call external APIs, databases, or services that can fail or time out
- Tool calls may return malformed, unexpected, or empty responses
- Agent reasoning can produce invalid actions (bad parameters, out-of-scope requests)
- Long-running tasks must survive transient failures without full restart
- Resource exhaustion (token limits, rate limits, quotas) is possible
- Partial failures in multi-step pipelines must not silently corrupt outputs

**Rule of thumb:** If an agent can fail (and it can), it needs exception handling — every external call, every tool invocation, every LLM response that requires a specific format.

## Exception Classification

### By Cause
| Exception Type | Examples | Recovery |
|----------------|----------|----------|
| **Transient** | Network timeout, rate limit | Retry with backoff |
| **Input Error** | Invalid parameters, bad format | Validate, re-prompt |
| **Tool Failure** | API down, auth expired | Fallback tool or skip |
| **Logic Error** | Contradictory goals, impossible task | Escalate or abort |
| **Resource Exhaustion** | Token limit, quota exceeded | Summarize, paginate, escalate |

### By Severity
- **Recoverable**: Agent can self-correct and continue
- **Degradable**: Agent continues with reduced capability
- **Fatal**: Agent must stop and notify the user/operator

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the failure landscape:
1. What external dependencies does the agent have? (APIs, tools, LLMs)
2. What can go wrong at each dependency? (Network, auth, rate limit, format)
3. What is the acceptable degraded behavior for each failure?
4. When should the agent retry vs. escalate vs. abort?

### PLAN
Design the exception handling architecture:
1. Define retry policies: max attempts, backoff strategy, jitter
2. Design fallback chains: primary → secondary → degraded → abort
3. Define error classification and routing logic
4. Plan state preservation: how to checkpoint before risky operations
5. Design escalation paths: when to involve humans or fail safely

### ACTION
Implement resilient agent behavior:
1. Wrap all external calls in try/except with specific exception types
2. Implement exponential backoff with jitter for transient failures
3. Add circuit breakers for persistently failing dependencies
4. Log all exceptions with context for debugging
5. Test failure modes explicitly — inject failures in tests

## Implementation: Retry with Exponential Backoff

### Core Retry Decorator
```python
import time
import random
import functools
from typing import Type, Tuple, Callable, Any

def with_retry(
    max_attempts: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exceptions: Tuple[Type[Exception], ...] = (Exception,),
    jitter: bool = True
):
    """Decorator that retries a function on specified exceptions.

    Args:
        max_attempts: Maximum number of total attempts
        base_delay: Initial delay between retries in seconds
        max_delay: Maximum delay cap in seconds
        exceptions: Exception types that trigger retries
        jitter: Add randomness to avoid thundering herd
    """
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt == max_attempts - 1:
                        break  # Final attempt failed, raise

                    # Exponential backoff with optional jitter
                    delay = min(base_delay * (2 ** attempt), max_delay)
                    if jitter:
                        delay *= (0.5 + random.random())  # ±50% jitter

                    print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay:.1f}s...")
                    time.sleep(delay)

            raise last_exception
        return wrapper
    return decorator

# Usage
import requests

@with_retry(max_attempts=3, base_delay=2.0, exceptions=(requests.Timeout, requests.ConnectionError))
def fetch_external_data(url: str) -> dict:
    """Fetch data from external API with automatic retry."""
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

### Circuit Breaker Pattern
```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"    # Normal operation
    OPEN = "open"        # Blocking calls — dependency is down
    HALF_OPEN = "half_open"  # Testing if dependency recovered

class CircuitBreaker:
    """Prevents cascading failures by stopping calls to failing dependencies."""

    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: float = 60.0,
        success_threshold: int = 2
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.success_threshold = success_threshold

        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None

    def call(self, func: Callable, *args, **kwargs) -> Any:
        """Execute function through circuit breaker."""
        if self.state == CircuitState.OPEN:
            # Check if enough time passed to try again
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
                self.success_count = 0
            else:
                raise RuntimeError(
                    f"Circuit OPEN — dependency unavailable. "
                    f"Retry after {self.recovery_timeout}s"
                )

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        self.failure_count = 0
        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= self.success_threshold:
                self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN

# Usage
search_circuit = CircuitBreaker(failure_threshold=3, recovery_timeout=30.0)

def safe_web_search(query: str) -> str:
    """Web search protected by circuit breaker."""
    try:
        return search_circuit.call(actual_web_search, query)
    except RuntimeError as e:
        return f"Search unavailable: {e}. Using cached data instead."
```

## Implementation: ADK Agent with Exception Handling

### Resilient Tool Functions
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
import requests
import time

def search_with_fallback(query: str) -> dict:
    """Web search with automatic fallback to secondary provider.

    Args:
        query: Search query

    Returns:
        Search results or error with fallback indicator
    """
    providers = [
        ("primary", f"https://api.primary-search.com/search?q={query}"),
        ("secondary", f"https://api.secondary-search.com/q={query}")
    ]

    for provider_name, url in providers:
        try:
            response = requests.get(url, timeout=8)
            response.raise_for_status()
            return {
                "results": response.json(),
                "provider": provider_name,
                "status": "success"
            }
        except requests.Timeout:
            continue  # Try next provider
        except requests.HTTPError as e:
            if e.response.status_code == 429:  # Rate limited
                time.sleep(2)
                continue
            continue
        except Exception:
            continue

    return {
        "results": [],
        "provider": "none",
        "status": "all_providers_failed",
        "error": "All search providers unavailable — proceeding with knowledge only"
    }

def parse_structured_output(raw_text: str, schema: dict) -> dict:
    """Parse LLM output against expected schema with error recovery.

    Args:
        raw_text: Raw LLM response text
        schema: Expected output schema with required fields

    Returns:
        Parsed output or error with partial results
    """
    import json
    import re

    # Try direct JSON parse
    try:
        data = json.loads(raw_text)
        missing = [k for k in schema.get("required", []) if k not in data]
        if not missing:
            return {"data": data, "status": "success"}
        return {"data": data, "status": "partial", "missing_fields": missing}
    except json.JSONDecodeError:
        pass

    # Try extracting JSON from markdown code blocks
    json_match = re.search(r'```(?:json)?\s*([\s\S]+?)\s*```', raw_text)
    if json_match:
        try:
            data = json.loads(json_match.group(1))
            return {"data": data, "status": "extracted"}
        except json.JSONDecodeError:
            pass

    # Return structured error for agent to handle
    return {
        "data": None,
        "status": "parse_failed",
        "raw_text": raw_text[:500],
        "suggestion": "Ask the LLM to reformat as valid JSON"
    }

# Resilient agent
resilient_agent = LlmAgent(
    name="ResilientResearcher",
    model="gemini-2.5-flash",
    instruction="""You are a resilient research agent. When tools fail:
    1. Check the 'status' field in tool responses
    2. If status is 'all_providers_failed': proceed with your existing knowledge
    3. If status is 'partial': note which fields are missing in your response
    4. If status is 'parse_failed': reformat your output as valid JSON
    5. Always provide a useful response even when tools fail — degrade gracefully.""",
    tools=[
        FunctionTool(search_with_fallback),
        FunctionTool(parse_structured_output)
    ]
)
```

### Exception Handling in LangGraph
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Optional
from google import genai
import time

class ResilientState(TypedDict):
    task: str
    result: Optional[str]
    errors: list
    retry_count: int
    max_retries: int
    status: str  # running/completed/failed/degraded

client = genai.Client()

def execute_with_retry_node(state: ResilientState) -> dict:
    """Execute task with automatic retry on failure."""
    try:
        response = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=f"Complete this task: {state['task']}"
        )
        return {
            "result": response.text,
            "status": "completed",
            "retry_count": state.get("retry_count", 0)
        }
    except Exception as e:
        retry_count = state.get("retry_count", 0) + 1
        errors = state.get("errors", []) + [str(e)]

        if retry_count >= state.get("max_retries", 3):
            return {
                "errors": errors,
                "retry_count": retry_count,
                "status": "failed"
            }

        time.sleep(2 ** retry_count)  # Exponential backoff
        return {
            "errors": errors,
            "retry_count": retry_count,
            "status": "running"
        }

def fallback_node(state: ResilientState) -> dict:
    """Provide degraded response when main execution fails."""
    errors = "\n".join(state.get("errors", []))
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""The following errors occurred while completing the task:
        {errors}

        Task: {state['task']}

        Provide the best possible partial response despite these failures.
        Be transparent about what you could and could not accomplish."""
    )
    return {
        "result": response.text,
        "status": "degraded"
    }

def route_after_execution(state: ResilientState) -> str:
    if state["status"] == "completed":
        return END
    elif state["status"] == "failed":
        return "fallback"
    return "execute"  # Retry

graph = StateGraph(ResilientState)
graph.add_node("execute", execute_with_retry_node)
graph.add_node("fallback", fallback_node)
graph.set_entry_point("execute")
graph.add_conditional_edges("execute", route_after_execution, {
    END: END,
    "fallback": "fallback",
    "execute": "execute"
})
graph.add_edge("fallback", END)

resilient_pipeline = graph.compile()
```

## Key Takeaways

- **Classify exceptions**: Transient vs. logic vs. resource failures require different recovery strategies
- **Retry with backoff**: Exponential backoff + jitter prevents overwhelming recovering services
- **Circuit breakers**: Stop calling failing dependencies early rather than wasting retries
- **Fallback chains**: Design primary → secondary → degraded → abort paths for every critical dependency
- **Degrade gracefully**: A partial response is almost always better than an exception
- **Log everything**: Include error type, context, attempt number, and recovery action in all logs

## Anti-Patterns to Avoid

- **Silent failures**: Catching all exceptions and returning empty results hides bugs
- **Infinite retries**: Always set a maximum retry count — retrying forever degrades everything
- **No jitter**: Synchronized retries (all agents retry at the same time) cause retry storms
- **Over-broad catches**: `except Exception` swallows bugs — catch specific exception types
- **Retry on non-transient errors**: Don't retry authentication failures, bad input, or logic errors

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Production-ready safe agent | Exception Handling + **Guardrails** + **Evaluation** |
| Resilient tool-using agent | Exception Handling + **Tool Use** + **Resource-Aware** |
| Human-escalation on failure | Exception Handling + **Human-in-the-Loop** |
| Self-healing long-running pipeline | Exception Handling + **Planning** + **Reflection** |

## References

- Google ADK Error Handling: https://google.github.io/adk-docs/
- Python Tenacity Library (retry): https://tenacity.readthedocs.io/
- Circuit Breaker Pattern: https://martinfowler.com/bliki/CircuitBreaker.html
- LangGraph Error Handling: https://langchain-ai.github.io/langgraph/how-tos/
