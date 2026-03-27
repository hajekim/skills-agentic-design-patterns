---
name: prompt-chaining
description: This skill should be used when the user wants to "chain prompts", "build a pipeline", "break down complex tasks into steps", "sequential LLM calls", "multi-step reasoning", "pipeline pattern", "sequential agent pipeline", "multi-step prompt pipeline", "LLM chain", "step-by-step agent", "prompt pipeline", "decompose task into prompts", "stage-by-stage processing", "chained LLM calls", or when a task is too complex for a single prompt and needs to be decomposed into sequential sub-tasks. Also responds to Korean: "프롬프트 체이닝", "체인 방식 에이전트", "단계별 LLM 호출", "복잡한 작업 분해", "순차적 프롬프트 파이프라인", "단계적 작업 처리", "단계별 처리", "순차 처리 에이전트", "파이프라인 만들어줘", "단계적으로 처리해줘", "연속 프롬프트", "체이닝 해줘". Also responds to Japanese: "プロンプトチェーニング", "ステップごとに処理", "順次LLM呼び出し", "パイプライン構築", "複雑なタスクを分解", "段階的エージェント", "連続プロンプト", "LLMパイプライン", "順番に処理してほしい", "チェーン方式エージェント" Also responds to Chinese: "提示词链", "顺序调用LLM", "构建流水线", "分解复杂任务", "逐步处理", "链式提示", "多步骤流水线", "按顺序处理任务", "提示词管道", "帮我一步一步处理".. Apply this skill to design, implement, or explain the Prompt Chaining agentic design pattern.
version: 1.0.0
---

# Prompt Chaining Pattern

## Overview

Prompt Chaining (also known as the **Pipeline Pattern**) is a foundational agentic design pattern that breaks complex tasks into a sequence of smaller, focused sub-tasks. Rather than overwhelming a single LLM call with a multifaceted problem, each sub-task is addressed by a specifically crafted prompt, and the output of one step feeds as input to the next.

**Core Principle:** Divide-and-conquer — decompose the complex into a logical chain of manageable steps.

## When This Skill Applies

Activate this pattern when:
- A task is too complex or multifaceted for a single prompt
- Multiple distinct processing stages are required
- Intermediate results need validation or transformation before the next step
- External tools or APIs must be called between reasoning steps
- You need to build agents capable of multi-step reasoning, planning, and decision-making
- The cognitive load on the model is causing instruction neglect, contextual drift, or hallucination

**Rule of thumb:** If a monolithic prompt struggles with multiple constraints and sequential reasoning steps, switch to prompt chaining.

## Context Engineering

**Context Engineering** is the overarching discipline that governs how AI agents are designed — it is the practice of constructing and delivering the right informational environment to the model at every step of a pipeline. It is not just prompt writing; it is the systematic engineering of everything the model sees.

```
┌─────────────────────────────────────────────────────────────┐
│                     CONTEXT WINDOW                          │
│                                                             │
│  ┌──────────────┐  ┌───────────────┐  ┌────────────────┐  │
│  │ System Prompt│  │  RAG / Docs   │  │  Tool Outputs  │  │
│  │  (Role +     │  │  (Retrieved   │  │  (API results, │  │
│  │  Behavior)   │  │   Knowledge)  │  │  computations) │  │
│  └──────────────┘  └───────────────┘  └────────────────┘  │
│                                                             │
│  ┌──────────────┐  ┌───────────────┐                       │
│  │ State/History│  │   Structured  │                       │
│  │ (Prior chain │  │   Outputs     │                       │
│  │   outputs)   │  │  (JSON/schema)│                       │
│  └──────────────┘  └───────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

**Context Engineering = Prompt Engineering + RAG + State/History + Structured Outputs + Memory**

Each component contributes to output quality:

| Component | Role in the Chain |
|-----------|-------------------|
| **System Prompt** | Defines the agent's persona, constraints, and output format per step |
| **Retrieved Documents (RAG)** | Injects external knowledge relevant to the current step |
| **Tool Outputs** | Provides real-world data from API calls, computations, or database queries |
| **State / History** | Accumulates prior steps' outputs to maintain continuity across the chain |
| **Structured Outputs** | Enforces JSON/XML schemas so downstream steps receive parseable, validated data |
| **Memory** | Long-term facts persisted across conversations and sessions |

> **Key Insight:** The quality of the model's output is more determined by the quality of the context it receives than by the model's architecture. Improving context engineering yields larger gains than switching models.

## Agent Complexity Levels

Agentic Design Patterns map to a spectrum of agent complexity. Understanding where a pattern sits helps in selecting the right pattern for the task:

| Level | Name | Characteristics | Relevant Patterns |
|-------|------|-----------------|-------------------|
| **Level 0** | Core Reasoning Engine | Single-model calls, sequential decomposition, no external tools | Prompt Chaining, Reflection |
| **Level 1** | Connected Problem-Solver | Tool use, external APIs, RAG, memory integration | Tool Use, Memory Management, MCP |
| **Level 2** | Strategic Problem-Solver | Multi-step planning, adaptive decision-making, self-correction loops | Planning, Routing, Guardrails, Evaluation |
| **Level 3** | Collaborative Multi-Agent | Multiple specialized agents, inter-agent communication, A2A protocols | Multi-Agent Collaboration, A2A, Parallelization |

**Prompt Chaining** is a **Level 0** pattern — the foundational building block upon which all higher-level agentic capabilities are constructed. Every Level 1, 2, and 3 pattern internally relies on prompt chaining to orchestrate its component steps.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Identify the problem structure:
1. What is the final output goal?
2. What are the logical sub-problems that compose the task?
3. What data must flow between steps?
4. Where can structured output formats (JSON/XML) enforce data integrity?

### PLAN
Design the chain architecture:
1. Map each sub-problem to a focused prompt with a single responsibility
2. Specify input/output schema for each step (use JSON for machine-readable handoffs)
3. Identify steps that can run in parallel vs. those requiring sequential ordering
4. Define error handling and validation logic between steps
5. Assign distinct roles per step (e.g., "Market Analyst" → "Trade Analyst" → "Documentation Writer")

### ACTION
Implement and execute the chain:
1. Build each prompt as a focused, single-responsibility unit
2. Pass structured outputs between steps to minimize ambiguity
3. Insert validation/processing logic between LLM calls as needed
4. Execute the chain and monitor each step's output quality
5. Debug step-by-step when issues arise — the modular structure enables granular inspection

## Core Concepts

### Sequential Decomposition
Break complex tasks into a focused, sequential workflow:
```
Input → [Prompt 1: Summarize] → [Prompt 2: Extract Trends] → [Prompt 3: Draft Email] → Output
```

Each step has:
- A single, clear responsibility
- A well-defined input (from previous step or original input)
- A structured output (for next step or final delivery)

### Structured Output Between Steps
The reliability of a prompt chain depends on data integrity between steps. Always use structured formats:

```json
{
  "trends": [
    {
      "trend_name": "AI-Powered Personalization",
      "supporting_data": "73% of consumers prefer brands using personal information"
    }
  ]
}
```

Using JSON/XML minimizes interpretation errors and enables precise parsing.

## Practical Use Cases

### 1. Information Processing Workflows
```
Prompt 1: Extract text from URL/document
Prompt 2: Summarize the cleaned text
Prompt 3: Extract entities (names, dates, locations)
Prompt 4: Query knowledge base with entities
Prompt 5: Generate final report
```

### 2. Complex Query Answering
```
Prompt 1: Decompose query into sub-questions
Prompt 2: Research sub-question A
Prompt 3: Research sub-question B
Prompt 4: Synthesize findings into coherent answer
```

### 3. Data Extraction and Transformation
```
Prompt 1: Extract fields from unstructured document
Validation: Check completeness and format
Prompt 2 (conditional): Re-extract missing/malformed fields
Validation: Verify results
Output: Structured, validated data
```

### 4. Content Generation Workflows
```
Prompt 1: Generate topic ideas
Selection: User or automatic best-idea selection
Prompt 2: Create detailed outline
Prompt 3-N: Write each section (with prior sections as context)
Final Prompt: Review and refine for coherence, tone, grammar
```

### 5. Code Generation and Refinement
```
Prompt 1: Understand request → generate pseudocode/outline
Prompt 2: Write initial code draft
Prompt 3: Identify bugs or improvement areas (static analysis or LLM review)
Prompt 4: Rewrite/refine based on identified issues
Prompt 5: Add documentation and test cases
```

### 6. Conversational Agents with State
```
Prompt N: Process user utterance, extract intent and entities
Processing: Update conversation state
Prompt N+1: Generate response using accumulated state
Repeat: Each turn builds on the conversation history chain
```

### 7. Multimodal Multi-Step Reasoning
```
Prompt 1: Extract and comprehend text from image/document
Prompt 2: Link extracted content with corresponding labels
Prompt 3: Interpret gathered information to determine output
```

## Implementation with Google ADK / Gemini CLI

### Using Gemini CLI Sequential Execution
```python
from google import genai

client = genai.Client()

def prompt_chain(input_text: str) -> str:
    # Step 1: Extract key information
    step1_prompt = f"Extract the technical specifications from:\n\n{input_text}"
    step1_result = client.models.generate_content(model='gemini-2.5-flash', contents=step1_prompt)
    specifications = step1_result.text

    # Step 2: Transform to structured JSON
    step2_prompt = f"""Transform the following specifications into a JSON object
    with 'cpu', 'memory', and 'storage' as keys:\n\n{specifications}"""
    step2_result = client.models.generate_content(model='gemini-2.5-flash', contents=step2_prompt)

    return step2_result.text

result = prompt_chain("The laptop features a 3.5 GHz octa-core processor, 16GB RAM, 1TB NVMe SSD.")
print(result)
```

### Using Google ADK Pipeline
```python
from google.adk.agents import Agent, SequentialAgent
from google.adk.tools import FunctionTool

# Define individual step agents
summarizer = Agent(
    name="summarizer",
    model="gemini-2.5-flash",
    instruction="You are a Market Analyst. Summarize the key findings from the provided text."
)

trend_extractor = Agent(
    name="trend_extractor",
    model="gemini-2.5-flash",
    instruction="You are a Trade Analyst. Extract the top 3 trends with supporting data as JSON."
)

report_writer = Agent(
    name="report_writer",
    model="gemini-2.5-flash",
    instruction="You are an Expert Documentation Writer. Draft a concise email from the trend analysis."
)

# Chain them sequentially
pipeline = SequentialAgent(
    name="analysis_pipeline",
    sub_agents=[summarizer, trend_extractor, report_writer]
)
```

### Using LangChain LCEL (LangChain Expression Language)
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

prompt_extract = ChatPromptTemplate.from_template(
    "Extract the technical specifications from:\n\n{text_input}"
)
prompt_transform = ChatPromptTemplate.from_template(
    "Transform to JSON with 'cpu', 'memory', 'storage' keys:\n\n{specifications}"
)

# Build the chain using LCEL pipe operator
extraction_chain = prompt_extract | llm | StrOutputParser()
full_chain = (
    {"specifications": extraction_chain}
    | prompt_transform
    | llm
    | StrOutputParser()
)

result = full_chain.invoke({"text_input": "The laptop features..."})
```

## Key Takeaways

- **Modularity**: Each step has a single, focused responsibility — easier to debug and optimize
- **Reliability**: Sequential decomposition significantly reduces instruction neglect and hallucination
- **Debuggability**: Granular control — inspect each step's output independently
- **Composability**: Combine with parallelization for independent sub-tasks; use chaining for dependent steps
- **Structured Handoffs**: Always use JSON/XML between steps to ensure data integrity
- **Framework Support**: LangChain/LangGraph, Google ADK, and Crew AI all provide native chain orchestration

## Anti-Patterns to Avoid

- **Over-chaining**: Don't create unnecessarily long chains for tasks a single prompt can handle reliably
- **Unstructured handoffs**: Passing raw, unvalidated text between steps causes error propagation
- **Ignoring parallelism**: Independent sub-tasks within a chain can run concurrently — don't serialize unnecessarily
- **Missing validation**: No error handling between steps means failures cascade silently

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Complex multi-step research pipeline | Prompt Chaining + **Planning** + **Tool Use** |
| Self-improving output quality | Prompt Chaining + **Reflection** |
| Context-aware sequential processing | Prompt Chaining + **RAG** + **Memory Management** |
| High-throughput parallel pipelines | Prompt Chaining + **Parallelization** |
| Safe production deployment | Prompt Chaining + **Guardrails** + **Exception Handling** |

## References

- LangChain LCEL Documentation: https://python.langchain.com/v0.2/docs/core_modules/expression_language/
- LangGraph Documentation: https://langchain-ai.github.io/langgraph/
- Google ADK Documentation: https://google.github.io/adk-docs/
- Prompt Engineering Guide - Chaining: https://www.promptingguide.ai/techniques/chaining
- Vertex AI Prompt Optimizer: https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-optimizer
