---
name: appendix-reasoning-engines
description: This skill should be used when the user wants to understand "reasoning models", "thinking tokens", "extended thinking", "LLM internal reasoning", "o1 style reasoning", "Gemini thinking model", "chain of thought internally", "reasoning vs standard models", "when to use reasoning models", "inference-time compute scaling", "deliberate reasoning LLMs", "when to use thinking model", "gemini-2.5-pro vs flash", "extended thinking agent", "deep reasoning agent", "inference time scaling", or choose between standard and reasoning-optimized LLMs for agentic tasks Also responds to Chinese: "推理模型", "思考令牌", "扩展思考", "LLM内部推理", "如何选择推理模型", "思考模型", "深度思考的模型", "解决复杂问题", "高精度模型选择", "推理时扩展".. Apply this skill to understand, select, and integrate reasoning engine capabilities into agent designs. Also responds to Korean: "추론 모델", "사고 토큰", "확장 사고", "LLM 내부 추론", "추론 vs 표준 모델", "추론 엔진 선택", "추론 모델 통합", "추론 모델 선택", "thinking 모델", "딥 리즈닝 모델", "깊게 생각하는 모델", "복잡한 문제 풀어줘", "정확도 높은 모델 선택해줘". Also responds to Japanese: "推論モデル", "思考トークン", "拡張思考", "LLM内部推論", "推論モデルの選び方", "thinkingモデル", "深く考えるモデル", "複雑な問題を解いてほしい", "精度の高いモデル選択", "推論時スケーリング".
version: 1.0.0
---

# Appendix F — Reasoning Engines: Under the Hood

## Overview

Modern LLMs can be divided into two broad categories based on how they generate output:

- **Standard Models**: Generate tokens directly from the prompt context. Fast and cost-efficient.
- **Reasoning Models**: Perform extended internal deliberation — generating hidden "thinking" tokens before producing the final answer. Slower and more expensive, but significantly better at complex multi-step problems.

Understanding which type to use, and how they work internally, is critical for designing effective agentic systems.

**Core Principle:** Match the reasoning depth to the task complexity — reasoning models excel where accuracy matters more than speed; standard models win on throughput and cost.

## When This Skill Applies

Activate this skill when:
- A task requires complex multi-step mathematical, logical, or strategic reasoning
- Accuracy is more important than latency (e.g., medical, legal, financial decisions)
- Standard models fail repeatedly on a problem that requires deliberate thinking
- You need to choose between `gemini-2.5-flash`, `gemini-2.5-flash` with high Thinking Budget, or `gemini-2.5-pro`
- Building agents that need to plan deeply before acting

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Assess the reasoning requirements:
1. How many reasoning steps does the task require?
2. Is accuracy or speed the primary constraint?
3. What is the token budget and cost tolerance?
4. Does the problem require backtracking or hypothesis testing?

### PLAN
Select the right reasoning configuration:
1. Map tasks to Standard vs. Reasoning model based on complexity
2. Design the "thinking budget" — how many thinking tokens to allow
3. Identify where to surface reasoning traces for debugging or auditing
4. Plan cost controls (thinking tokens cost more than output tokens)

### ACTION
Implement the reasoning-optimized agent:
1. Use thinking-enabled models for complex planning and reasoning nodes
2. Use standard models for simpler steps (retrieval, formatting, routing)
3. Expose thinking traces for transparency when required
4. Benchmark latency and accuracy trade-offs against your SLAs

## How Reasoning Models Work

### The Thinking Token Mechanism

```
Standard Model:
  Prompt → [Model processes in a single forward pass] → Output tokens

Reasoning Model:
  Prompt → [Model generates hidden thinking tokens] → [Model uses thinking as context] → Output tokens
              ↑
         Not visible in final output unless explicitly surfaced
         (these are the "scratchpad" tokens)
```

Thinking tokens allow the model to:
- Explore multiple solution paths before committing to one
- Identify and correct errors mid-reasoning
- Perform step-by-step mathematical or logical derivations
- Simulate hypotheticals: "If I take action A, then B will follow..."

### Inference-Time Compute Scaling

Unlike training-time scaling (more parameters, more data), reasoning models scale at **inference time**:

```
Standard model:   Fixed compute per token → fixed quality ceiling
Reasoning model:  More thinking tokens → higher quality (diminishing returns apply)
```

This means you can tune the "thinking budget" to balance cost and quality:
```python
# Low budget: faster, cheaper, less accurate on hard problems
# High budget: slower, more expensive, more accurate

thinking_budget = {
    "simple_query": 128,    # tokens
    "medium_task": 1024,
    "complex_analysis": 8192,
    "research_grade": 32768
}
```

## Model Selection Guide

| Task Type | Recommended Model | Reasoning |
|-----------|-------------------|-----------|
| Simple Q&A, summarization | `gemini-2.5-flash` | Fast, cheap, sufficient |
| Code generation (moderate) | `gemini-2.5-flash` | Good at structured output |
| Complex math / proofs | `gemini-2.5-flash` (high Thinking Budget) | Extended reasoning needed |
| Multi-hop logical reasoning | `gemini-2.5-flash` (high Thinking Budget) | Hypothesis tracking needed |
| Strategic planning agent | `gemini-2.5-pro` | Maximum capability |
| Real-time routing / classification | `gemini-2.5-flash` | Latency-critical path |
| Code debugging (complex) | `gemini-2.5-flash` (high Thinking Budget) | Root cause analysis depth |
| Legal / medical document analysis | `gemini-2.5-pro` | Accuracy-critical |

## Implementation: Standard vs. Thinking Model

### Standard Model (Fast Path)
```python
from google import genai

client = genai.Client()

def fast_agent(prompt: str) -> str:
    """Standard model for latency-sensitive tasks."""
    response = client.models.generate_content(model='gemini-2.5-flash', contents=prompt)
    return response.text
```

### Thinking Model (Deep Reasoning Path)
```python
from google import genai
from google.genai import types

client = genai.Client()

def reasoning_agent(problem: str, thinking_budget: int = 4096) -> dict:
    """Reasoning model with configurable thinking budget."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',  # use high Thinking Budget for extended reasoning
        contents=problem,
        config=types.GenerateContentConfig(
            # thinking_config is model-specific; check current API docs
            max_output_tokens=8192,
            temperature=1.0  # Reasoning models use temperature=1 by default
        )
    )

    return {
        "answer": response.text,
        # Thinking traces may be accessible via response.candidates[0].content
        # when think_aloud / thought output is enabled in the API
    }

# Usage: complex multi-step reasoning
result = reasoning_agent(
    "A company has 3 products with margins of 40%, 25%, and 60%. "
    "If they sell 1000, 2000, and 500 units respectively at prices "
    "$50, $30, $120, what is the weighted average margin and "
    "total profit? Which product should they prioritize to maximize profit?"
)
print(result["answer"])
```

### Hybrid Agent: Route by Complexity
```python
from google import genai
from enum import Enum

client = genai.Client()

class TaskComplexity(Enum):
    SIMPLE = "simple"
    MEDIUM = "medium"
    COMPLEX = "complex"

class HybridReasoningAgent:
    """Uses standard or reasoning model based on task complexity."""

    def __init__(self):
        self.client = genai.Client()

    def classify_complexity(self, task: str) -> TaskComplexity:
        """Classify task complexity using a lightweight model."""
        classification_prompt = f"""Classify the complexity of this task.
Output ONLY one word: 'simple', 'medium', or 'complex'.

Simple: factual lookup, summarization, translation, formatting
Medium: code generation, analysis with 2-3 steps, structured extraction
Complex: mathematical proof, multi-hop reasoning, strategic planning, debugging deep issues

Task: {task}"""

        response = self.client.models.generate_content(model='gemini-2.5-flash', contents=classification_prompt)
        label = response.text.strip().lower()

        if "complex" in label:
            return TaskComplexity.COMPLEX
        elif "medium" in label:
            return TaskComplexity.MEDIUM
        return TaskComplexity.SIMPLE

    def solve(self, task: str) -> dict:
        """Route to appropriate model based on complexity."""
        complexity = self.classify_complexity(task)

        if complexity == TaskComplexity.SIMPLE:
            model_name = "gemini-2.5-flash"
        elif complexity == TaskComplexity.MEDIUM:
            model_name = "gemini-2.5-flash"  # use high Thinking Budget for reasoning tasks
        else:
            model_name = "gemini-2.5-pro"

        response = self.client.models.generate_content(model=model_name, contents=task)

        return {
            "answer": response.text,
            "model_used": model_name,
            "complexity": complexity.value
        }

# Usage
agent = HybridReasoningAgent()

# Simple task → gemini-2.5-flash
result = agent.solve("Translate 'hello world' to French")
print(f"[{result['model_used']}] {result['answer']}")

# Complex task → gemini-2.5-pro
result = agent.solve(
    "Design a distributed caching strategy for a system serving 10M requests/day "
    "with 99.99% uptime requirement and <50ms p99 latency. Consider cache invalidation, "
    "consistency guarantees, and failure modes."
)
print(f"[{result['model_used']}] {result['answer']}")
```

### ADK Agent with Thinking Model
```python
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

# Use a thinking-capable model for the planning node
planning_agent = LlmAgent(
    name="DeepPlanner",
    model="gemini-2.5-flash",  # use high Thinking Budget for extended reasoning
    instruction="""You are a strategic planning agent. When given a complex goal:
    1. Think through all possible approaches and their trade-offs
    2. Identify dependencies and critical path
    3. Anticipate failure modes and design mitigations
    4. Produce a detailed, executable plan

    Take as much time as needed to reason carefully before answering."""
)

# Use a standard model for execution (speed matters here)
executor_agent = LlmAgent(
    name="Executor",
    model="gemini-2.5-flash",  # Fast execution following the plan
    instruction="Execute the provided plan step by step. Report completion after each step."
)

session_service = InMemorySessionService()
runner = Runner(
    agent=planning_agent,
    app_name="deep_planning_app",
    session_service=session_service
)
session_service.create_session(
    app_name="deep_planning_app",
    user_id="user1",
    session_id="session1"
)

message = Content(parts=[Part(text="Design a migration plan for moving a monolithic e-commerce app to microservices with zero downtime")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

## Cost and Latency Trade-offs

| Model | Latency | Cost | Best For |
|-------|---------|------|----------|
| `gemini-2.5-flash` | ~1-3s | Low | Routing, formatting, simple Q&A |
| `gemini-2.5-flash` (high Thinking Budget) | ~5-30s | Medium | Math, code debugging, analysis |
| `gemini-2.5-pro` | ~10-60s | High | Research-grade reasoning, strategic planning |

**Cost optimization patterns:**
1. **Cascade**: Try fast model first → escalate to thinking model only on failure
2. **Routing**: Classify complexity before choosing model (adds ~100ms but saves on expensive calls)
3. **Budget cap**: Set `max_output_tokens` for thinking to control cost
4. **Cache**: Cache reasoning results for identical or near-identical inputs

## Agentic Design Pattern Integration

Reasoning models enhance these patterns specifically:

| Pattern | How Reasoning Models Help |
|---------|--------------------------|
| **Planning** | Generate higher-quality multi-step plans with fewer logical errors |
| **Reflection** | Deeper self-critique — identifies subtle issues standard models miss |
| **Reasoning** (CoT/ReAct/ToT) | Internal thinking augments external structured reasoning techniques |
| **Routing** | More accurate intent classification for complex or ambiguous inputs |
| **Evaluation** | LLM-as-judge is more reliable when judge uses a reasoning model |

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Deep reasoning for planning agents | Reasoning Engines + **Planning** + **Reflection** |
| Cost-optimized reasoning cascade | Reasoning Engines + **Resource-Aware** + **Routing** |
| Transparent reasoning traces | Reasoning Engines + **Evaluation** + **Human-in-the-Loop** |
| Research-grade problem solving | Reasoning Engines + **Reasoning** + **Tool Use** |

## Key Takeaways

- **Two tiers**: Standard models for speed-critical tasks; reasoning models for accuracy-critical tasks
- **Thinking tokens** are the mechanism: hidden scratchpad tokens enable hypothesis exploration and error correction
- **Inference-time scaling**: Reasoning quality can be tuned by controlling the thinking token budget
- **Hybrid architecture**: Use different model tiers for different nodes in an agent pipeline (plan deep, execute fast)
- **Cost awareness**: Thinking tokens are billed like output tokens — always cap the budget in production

## Anti-Patterns to Avoid

- **Over-using reasoning models**: Not every task needs extended thinking — simple tasks waste cost and latency
- **Under-using reasoning models**: Using standard models for complex proofs or strategic planning leads to unreliable outputs
- **Fixed model assignment**: Hardcoding one model for all nodes ignores the natural cost/accuracy trade-off
- **No thinking budget**: Uncapped thinking tokens in production can cause unexpected cost spikes

## References

- Gemini Thinking Documentation: https://ai.google.dev/gemini-api/docs/thinking
- Google DeepMind Reasoning Research: https://deepmind.google/research/
- Scaling LLM Test-Time Compute (OpenAI o1 paper): https://arxiv.org/abs/2408.03314
- Anthropic Extended Thinking: https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking
