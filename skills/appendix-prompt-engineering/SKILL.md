---
name: appendix-prompt-engineering
description: This skill should be used when the user wants to learn "prompt engineering", "few-shot prompting", "zero-shot prompting", "chain of thought prompting", "structured output prompting", "role prompting", "system prompt design", "prompt best practices", "CoT prompting", "Pydantic structured output", "prompt iteration", "prompt versioning", "write better prompts", "prompt design", "improve AI responses", "LLM prompt tips", "structured LLM output", "prompt optimization", or improve the quality and reliability of their LLM prompts for agentic systems. Also responds to Korean: "프롬프트 엔지니어링", "퓨샷 프롬프팅", "제로샷 프롬프팅", "생각의 사슬 프롬프팅", "구조화된 출력 프롬프팅", "시스템 프롬프트 설계", "프롬프트 작성법", "좋은 프롬프트", "프롬프트 최적화", "프롬프트 잘 쓰는 법", "AI에게 잘 물어보는 법", "퓨샷 예제 만들어줘". Also responds to Japanese: "プロンプトエンジニアリング", "フューショットプロンプティング", "ゼロショットプロンプティング", "思考の連鎖", "プロンプトの書き方", "上手なプロンプト", "AIへの上手な質問方法", "構造化出力", "プロンプト最適化", "システムプロンプト設計" Also responds to Chinese: "提示词工程", "小样本提示", "零样本提示", "思维链提示", "提示词写法", "好的提示词", "如何向AI提问", "结构化输出", "提示词优化", "系统提示设计".. Apply this skill to design or implement effective prompting strategies as a disciplined engineering practice.
version: 1.0.0
---

# Appendix A - Prompt Engineering

## Overview

**Prompt Engineering** is the disciplined practice of crafting, iterating, and optimizing the inputs given to language models to produce reliable, high-quality outputs. It is not a simple act of asking questions — it is a structured engineering discipline that transforms a general-purpose language model into a specialized, highly capable tool for specific tasks.

For agents, prompting is the foundational layer: every agentic pattern depends on well-crafted prompts. A powerful model with a poor prompt produces poor results. A well-engineered prompt transforms model outputs from probabilistic guesses into deterministic, structured, trustworthy cognitive operations.

**Core Principle:** Treat prompts as code — version them, test them, iterate them, and document what works and why.

## When This Skill Applies

Activate this pattern when:
- Building any LLM-powered agent that needs reliable, structured outputs
- Writing system prompts, user instructions, or tool descriptions for agents
- An agent produces inconsistent, hallucinated, or poorly formatted responses
- You need structured data (JSON, XML) from model outputs
- Implementing Chain-of-Thought, ReAct, or few-shot reasoning patterns
- Testing and comparing different prompt strategies
- Preparing prompts for production deployment

**Rule of thumb:** Every agent interaction is a prompt. Engineer them deliberately — don't leave agent behavior to chance.

## Prompting Technique Hierarchy

```
Zero-Shot: No examples — just instructions
    → Simple tasks with clear instructions

One-Shot: One example — demonstrate the pattern
    → When output format matters

Few-Shot: 2–5 examples — show the range
    → Complex tasks, classification, structured extraction

Chain-of-Thought: "Think step by step" + examples
    → Reasoning tasks, math, multi-step problems

ReAct: Thought → Action → Observation loop
    → Agents using tools in an iterative loop
```

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map prompting requirements:
1. What is the task? (Classification, generation, extraction, reasoning, tool use)
2. What output format is needed? (Free text, JSON, structured schema, code)
3. What domain knowledge must be conveyed? (Role, context, constraints)
4. How variable is the input? (Static template vs. dynamic slot-filling)
5. What constitutes a "good" output? (Define success criteria for evaluation)

### PLAN
Design the prompt architecture:
1. Choose technique: zero-shot, few-shot, CoT, or ReAct based on task complexity
2. Write a clear system prompt: role, task, constraints, output format
3. Draft few-shot examples that demonstrate edge cases, not just happy paths
4. Design structured output schema if needed (Pydantic model, JSON schema)
5. Plan evaluation: how will you measure if the prompt is working correctly?

### ACTION
Implement and iterate:
1. Write the initial prompt with clarity and explicit instructions
2. Test against representative inputs including edge cases
3. Identify failures: wrong format, hallucinations, missed instructions
4. Refine one element at a time — change one thing per iteration
5. Document what worked, what failed, and why — store in version control

## Implementation: Core Prompt Engineering Techniques

### Zero-Shot and Role Prompting
```python
from google import genai

client = genai.Client()

# Role + instruction prompt (zero-shot)
def create_system_prompt(role: str, task: str, constraints: list, output_format: str) -> str:
    """Build a structured system prompt with role, task, constraints, and format."""
    constraints_text = "\n".join(f"- {c}" for c in constraints)
    return f"""You are {role}.

Your task: {task}

Constraints:
{constraints_text}

Output format: {output_format}"""

# Example: sentiment analysis agent
system_prompt = create_system_prompt(
    role="an expert sentiment analyst with deep knowledge of customer feedback",
    task="Classify customer feedback as POSITIVE, NEGATIVE, or NEUTRAL with a confidence score",
    constraints=[
        "Base your classification only on the text provided",
        "Never guess intent not supported by the text",
        "When uncertain, classify as NEUTRAL"
    ],
    output_format='JSON: {"sentiment": "POSITIVE|NEGATIVE|NEUTRAL", "confidence": 0.0-1.0, "reason": "brief explanation"}'
)
```

### Structured Output with Pydantic
```python
from google import genai
from pydantic import BaseModel
from typing import Literal
import json

client = genai.Client()

class SentimentResult(BaseModel):
    sentiment: Literal["POSITIVE", "NEGATIVE", "NEUTRAL"]
    confidence: float  # 0.0 to 1.0
    key_phrase: str    # The most indicative phrase
    reason: str        # Brief explanation

class ExtractedEntity(BaseModel):
    name: str
    entity_type: Literal["PERSON", "ORG", "LOCATION", "DATE", "PRODUCT"]
    confidence: float

def extract_structured_output(text: str, schema: type[BaseModel]) -> BaseModel:
    """Use Gemini with structured output schema enforcement.

    Args:
        text: Input text to analyze
        schema: Pydantic model defining the expected output structure

    Returns:
        Validated instance of the schema
    """
    prompt = f"""Analyze the following text and return a JSON response matching this schema:
{schema.model_json_schema()}

Text to analyze: {text}

Return ONLY valid JSON, no other text."""

    response = client.models.generate_content(model='gemini-2.5-flash', contents=prompt)

    # Parse and validate with Pydantic
    import re
    json_match = re.search(r'\{[\s\S]+\}', response.text)
    if json_match:
        data = json.loads(json_match.group())
        return schema(**data)
    raise ValueError(f"No valid JSON in response: {response.text}")

# Usage
result = extract_structured_output(
    "The new iPhone 16 launch was a massive success, exceeding all sales projections.",
    SentimentResult
)
print(f"Sentiment: {result.sentiment} ({result.confidence:.0%} confidence)")
print(f"Reason: {result.reason}")
```

### Few-Shot Prompting
```python
def build_few_shot_prompt(task_description: str, examples: list[dict], query: str) -> str:
    """Build a few-shot prompt from examples.

    Args:
        task_description: What the model should do
        examples: List of {"input": ..., "output": ...} dicts
        query: The actual input to process

    Returns:
        Complete few-shot prompt string
    """
    prompt = f"{task_description}\n\n"

    for i, example in enumerate(examples, 1):
        prompt += f"Example {i}:\n"
        prompt += f"Input: {example['input']}\n"
        prompt += f"Output: {example['output']}\n\n"

    prompt += f"Now complete this:\nInput: {query}\nOutput:"
    return prompt

# Example: Code review classification
examples = [
    {
        "input": "This function has O(n²) complexity in the inner loop",
        "output": '{"category": "performance", "severity": "medium", "actionable": true}'
    },
    {
        "input": "Variable name 'x' is not descriptive",
        "output": '{"category": "readability", "severity": "low", "actionable": true}'
    },
    {
        "input": "SQL query is vulnerable to injection via f-string formatting",
        "output": '{"category": "security", "severity": "critical", "actionable": true}'
    }
]

prompt = build_few_shot_prompt(
    task_description="Classify code review comments by category, severity, and whether they are actionable.",
    examples=examples,
    query="This module has no unit tests"
)
```

### Chain-of-Thought (CoT) Prompting
```python
def build_cot_prompt(problem: str, use_temperature_zero: bool = True) -> dict:
    """Build a Chain-of-Thought prompt for reasoning tasks.

    For tasks with a single correct answer (math, logic), use temperature=0.
    For creative reasoning, use higher temperature.
    """
    cot_prompt = f"""Solve the following problem step by step. Show all your reasoning before giving the final answer.

Problem: {problem}

Let me think through this step by step:
Step 1:"""

    return {
        "prompt": cot_prompt,
        "generation_config": {
            "temperature": 0.0 if use_temperature_zero else 0.7,
            "max_output_tokens": 1024
        }
    }

from google import genai
from google.genai import types

client = genai.Client()

# Math reasoning — always use temperature=0
cot_config = build_cot_prompt(
    "A store has 48 apples. It sells 60% on Monday and 25% of the remainder on Tuesday. How many remain?",
    use_temperature_zero=True
)

response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=cot_config["prompt"],
    config=types.GenerateContentConfig(**cot_config["generation_config"])
)
print(response.text)
```

### Prompt Versioning and Documentation
```python
from dataclasses import dataclass
from datetime import datetime
from typing import Optional

@dataclass
class PromptVersion:
    """Track prompt iterations with metadata."""
    version: str
    system_prompt: str
    created_at: str
    author: str
    description: str
    test_results: Optional[dict] = None
    notes: str = ""

# Maintain a versioned prompt library
PROMPT_LIBRARY = {
    "sentiment_classifier": [
        PromptVersion(
            version="1.0",
            system_prompt="Classify sentiment as POSITIVE, NEGATIVE, or NEUTRAL.",
            created_at="2025-01-01",
            author="team",
            description="Initial simple classifier",
            test_results={"accuracy": 0.72, "test_set_size": 100},
            notes="Poor on sarcasm"
        ),
        PromptVersion(
            version="1.1",
            system_prompt="""You are an expert sentiment analyst.
Classify sentiment as POSITIVE, NEGATIVE, or NEUTRAL.
Pay special attention to sarcasm, irony, and mixed sentiments.
When in doubt, classify as NEUTRAL.""",
            created_at="2025-01-15",
            author="team",
            description="Added sarcasm handling",
            test_results={"accuracy": 0.89, "test_set_size": 100},
            notes="Improved sarcasm detection significantly"
        )
    ]
}

def get_latest_prompt(name: str) -> PromptVersion:
    """Retrieve the latest version of a named prompt."""
    versions = PROMPT_LIBRARY.get(name, [])
    if not versions:
        raise KeyError(f"Prompt '{name}' not found in library")
    return versions[-1]  # Latest is last

latest = get_latest_prompt("sentiment_classifier")
print(f"Using {latest.version}: accuracy={latest.test_results['accuracy']:.0%}")
```

## Key Takeaways

- **Clarity over cleverness**: Explicit, unambiguous instructions outperform clever tricks — the model cannot read intent
- **Temperature=0 for determinism**: Use temperature=0 for tasks with single correct answers (math, JSON extraction, classification)
- **Structured output is non-negotiable**: For agents, always request structured output (JSON + Pydantic) — free text breaks automation
- **Few-shot examples are powerful**: 2-5 well-chosen examples outperform long explanations
- **CoT for reasoning**: Add "think step by step" to unlock model reasoning on complex problems
- **Version your prompts**: Treat prompts as code — document changes, test results, and rationale
- **Test adversarially**: Include edge cases, ambiguous inputs, and adversarial examples in your test set

## Anti-Patterns to Avoid

- **Ambiguous instructions**: "Be helpful" is not a prompt — specify exactly what "helpful" means in your context
- **Overloading the prompt**: One prompt doing 10 things produces mediocre results for all — decompose into a chain
- **No output format specification**: Unstructured output breaks downstream processing — always define the format
- **High temperature for factual tasks**: Temperature > 0.3 for factual/extraction tasks introduces unnecessary variation
- **Testing only happy paths**: Prompts that work on clean inputs often fail on real-world noisy data
- **Single-model self-evaluation**: Don't use the same model to judge its own outputs — use a separate judge model

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Structured multi-step prompting | Prompt Engineering + **Prompt Chaining** |
| Reasoning-enhanced prompts | Prompt Engineering + **Reasoning** + **Reflection** |
| Evaluated prompt optimization | Prompt Engineering + **Evaluation** |
| Agent system prompt design | Prompt Engineering + **Guardrails** + **Goal Setting** |

## References

- Prompt Engineering (Kaggle Whitepaper): https://www.kaggle.com/whitepaper-prompt-engineering
- Chain-of-Thought Prompting: https://arxiv.org/abs/2201.11903
- Self-Consistency for CoT: https://arxiv.org/pdf/2203.11171
- ReAct: https://arxiv.org/abs/2210.03629
- Tree of Thoughts: https://arxiv.org/pdf/2305.10601
- DSPy (Programming Foundation Models): https://github.com/stanfordnlp/dspy
- Gemini Structured Output: https://ai.google.dev/gemini-api/docs/structured-output
