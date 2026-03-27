---
name: parallelization
description: This skill should be used when the user wants to "run tasks in parallel", "concurrent LLM calls", "fan-out and fan-in", "parallel agents", "simultaneous execution", "reduce latency with concurrency", "parallel sub-tasks", "parallel processing", "concurrent agent execution", "run agents simultaneously", "speed up with parallelism", "async agent execution", "batch LLM calls", "parallel workflow", "parallelize agent tasks", or needs to execute multiple independent tasks simultaneously. Also responds to Korean: "병렬 작업 실행", "동시 LLM 호출", "팬아웃 팬인 패턴", "병렬 에이전트", "동시 실행", "동시성으로 지연 시간 감소", "병렬로 실행", "동시에 여러 작업", "병렬 처리", "동시에 돌려줘", "빠르게 병렬로", "여러 작업 동시 실행해줘". Also responds to Japanese: "並列処理", "同時実行", "並列エージェント", "並行LLM呼び出し", "ファンアウト・ファンイン", "複数タスクを同時に実行したい", "処理速度を上げたい", "非同期エージェント", "バッチ処理", "並列で動かしたい" Also responds to Chinese: "并行处理", "并发执行", "并行智能体", "同时调用多个LLM", "扇出扇入模式", "多任务并行运行", "加快处理速度", "异步智能体", "批量LLM调用", "同时运行多个任务".. Apply this skill to design or implement the Parallelization agentic design pattern.
version: 1.0.0
---

# Parallelization Pattern

## Overview

The **Parallelization Pattern** enables multiple components — LLM calls, tool invocations, or entire sub-agents — to execute *simultaneously* rather than sequentially. By identifying parts of a workflow that are independent of each other, this pattern dramatically reduces total execution time and increases system throughput.

**Core Principle:** Identify independence — run what doesn't depend on each other at the same time, then synchronize results.

## When This Skill Applies

Activate this pattern when:
- Multiple sub-tasks can be performed independently (no data dependency between them)
- External services (APIs, databases) introduce latency that can be overlapped
- The same task must be performed on multiple data items independently
- Throughput and speed are critical system requirements
- A workflow has a natural fan-out (distribute) → fan-in (aggregate) structure

**Rule of thumb:** If Step B doesn't need Step A's output to begin, run them simultaneously.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Identify the parallelism opportunities:
1. Which sub-tasks have no data dependency on each other?
2. Which steps interact with external services (API calls, database queries, searches)?
3. Where does the workflow have a natural fan-out structure?
4. What synchronization point (fan-in) collects all parallel results?

### PLAN
Design the parallel architecture:
1. Map the dependency graph — which nodes can run concurrently?
2. Define the fan-out: distribute tasks to parallel workers/agents
3. Define the fan-in: collect and aggregate results from all parallel branches
4. Identify which subsequent steps are sequential (depend on aggregated results)
5. Plan error handling: what happens if one parallel branch fails?

### ACTION
Implement parallel execution:
1. Use async/await, threading, or framework-native parallel execution primitives
2. Launch all independent tasks concurrently
3. Use synchronization primitives (gather, join, await all) to collect results
4. Pass aggregated results to the sequential continuation step
5. Monitor and handle partial failures gracefully

## Core Parallel Patterns

### Fan-Out / Fan-In (Map-Reduce)
The fundamental parallelization structure:
```
                ┌─ Worker A ─┐
Input → Fan-Out ├─ Worker B ─┤ → Fan-In → Aggregate → Output
                └─ Worker C ─┘
```

**Sequential (slow):**
```
Search A → Summarize A → Search B → Summarize B → Synthesize
```

**Parallel (fast):**
```
Search A + Search B (simultaneously)
→ Summarize A + Summarize B (simultaneously)
→ Synthesize (sequential — depends on both summaries)
```

### Parallel Sectioning
Different agents handle different sections of the same document or task simultaneously:
```
Document → [Section 1 Agent || Section 2 Agent || Section 3 Agent] → Combine
```

### Parallel Voting / Consensus
Multiple agents independently solve the same problem; results are aggregated:
```
Query → [Agent 1 response || Agent 2 response || Agent 3 response]
      → Majority vote or best-of-N selection
```

### Parallel Tool Calls
Multiple external API/database calls issued simultaneously:
```
User request → [Weather API || Calendar API || News API] → Synthesize
```

## Practical Use Cases

### Research & Synthesis
```python
# Instead of sequential searches:
parallel_tasks = [
    search_web("topic A"),
    search_database("topic B"),
    search_documents("topic C")
]
results = await asyncio.gather(*parallel_tasks)
final_report = synthesize(results)
```

### Multi-Source Document Analysis
```
Report → Chunk into sections →
[Section 1 Analyzer || Section 2 Analyzer || ... || Section N Analyzer]
→ Aggregate findings → Final analysis
```

### Evaluation Pipeline
```
Model response →
[Accuracy Evaluator || Relevance Evaluator || Safety Evaluator || Tone Evaluator]
→ Combined evaluation score
```

### Multi-Language Processing
```
Content →
[English Translator || Spanish Translator || French Translator || German Translator]
→ Multi-language output
```

## Implementation with Google ADK

### ADK Parallel Sub-Agents
```python
from google.adk.agents import LlmAgent, ParallelAgent
from google.adk.tools import FunctionTool

def search_web(query: str) -> str:
    """Search the web for information."""
    return f"Web results for: {query}"

def search_database(query: str) -> str:
    """Search internal database."""
    return f"Database results for: {query}"

def search_documents(query: str) -> str:
    """Search document store."""
    return f"Document results for: {query}"

# Parallel worker agents
web_agent = LlmAgent(
    name="WebSearcher",
    model="gemini-2.5-flash",
    tools=[FunctionTool(search_web)]
)

db_agent = LlmAgent(
    name="DatabaseSearcher",
    model="gemini-2.5-flash",
    tools=[FunctionTool(search_database)]
)

doc_agent = LlmAgent(
    name="DocumentSearcher",
    model="gemini-2.5-flash",
    tools=[FunctionTool(search_documents)]
)

# ParallelAgent runs all sub-agents simultaneously
parallel_searcher = ParallelAgent(
    name="ParallelResearcher",
    sub_agents=[web_agent, db_agent, doc_agent]
)

# Synthesizer processes all parallel results
synthesizer = LlmAgent(
    name="Synthesizer",
    model="gemini-2.5-flash",
    instruction="Synthesize the research findings from multiple sources into a coherent report."
)
```

### Python asyncio Implementation
```python
import asyncio
from google import genai

client = genai.Client()

async def analyze_section(section: str, section_id: int) -> dict:
    """Analyze a document section asynchronously."""
    prompt = f"Analyze the following section and extract key insights:\n\n{section}"
    response = await asyncio.get_event_loop().run_in_executor(
        None, lambda: client.models.generate_content(model='gemini-2.5-flash', contents=prompt)
    )
    return {"section_id": section_id, "analysis": response.text}

async def parallel_document_analysis(document: str) -> str:
    """Analyze document sections in parallel."""
    # Split document into independent sections
    sections = document.split("\n\n")

    # Launch all section analyses simultaneously
    tasks = [analyze_section(section, i) for i, section in enumerate(sections)]
    results = await asyncio.gather(*tasks)

    # Sequential synthesis step (depends on all parallel results)
    all_analyses = "\n".join([r["analysis"] for r in results])
    synthesis_prompt = f"Synthesize these section analyses:\n\n{all_analyses}"
    final_response = client.models.generate_content(model='gemini-2.5-flash', contents=synthesis_prompt)
    return final_response.text

# Run parallel analysis
result = asyncio.run(parallel_document_analysis(large_document))
```

### LangGraph Parallel Nodes
```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, List
import asyncio

class ResearchState(TypedDict):
    query: str
    web_results: str
    db_results: str
    final_report: str

# Define parallel worker functions
async def web_search_node(state: ResearchState) -> dict:
    result = await search_web(state["query"])
    return {"web_results": result}

async def db_search_node(state: ResearchState) -> dict:
    result = await search_database(state["query"])
    return {"db_results": result}

def synthesize_node(state: ResearchState) -> dict:
    # This runs AFTER both parallel nodes complete
    report = synthesize(state["web_results"], state["db_results"])
    return {"final_report": report}

# Build graph with parallel branches
graph = StateGraph(ResearchState)
graph.add_node("web_search", web_search_node)
graph.add_node("db_search", db_search_node)
graph.add_node("synthesize", synthesize_node)

# Fan-out: both nodes start simultaneously from START
graph.add_edge(START, "web_search")
graph.add_edge(START, "db_search")

# Fan-in: synthesize waits for both
graph.add_edge("web_search", "synthesize")
graph.add_edge("db_search", "synthesize")
graph.add_edge("synthesize", END)
```

## Performance Considerations

| Scenario | Sequential Time | Parallel Time | Speedup |
|----------|----------------|---------------|---------|
| 3 API calls (1s each) | 3s | ~1s | 3x |
| 5 document sections | 5s | ~1s | 5x |
| 4 translation tasks | 4s | ~1s | 4x |
| N independent tasks | N * t | ~t | Nx |

## Key Takeaways

- **Independence is key**: Only parallelize truly independent tasks — don't force concurrency on dependent steps
- **Fan-out + Fan-in**: The fundamental structure is distribute → execute concurrently → aggregate
- **Latency reduction**: Particularly effective for I/O-bound tasks (API calls, DB queries, web searches)
- **Framework support**: Google ADK's `ParallelAgent`, Python's `asyncio.gather()`, and LangGraph's parallel nodes
- **Sequential finalization**: The synthesis/aggregation step after parallel work is typically sequential
- **Error resilience**: Plan for partial failure — use `asyncio.gather(return_exceptions=True)` or similar

## Anti-Patterns to Avoid

- **False parallelism**: Parallelizing steps that have hidden dependencies causes race conditions
- **Ignoring rate limits**: Concurrent API calls may hit rate limits — implement throttling
- **No error handling**: A single failing parallel branch shouldn't crash the entire workflow
- **Over-parallelizing**: Very short tasks may not benefit from parallelization overhead

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| High-throughput multi-agent research | Parallelization + **Multi-Agent Collaboration** |
| Cost-optimized parallel execution | Parallelization + **Resource-Aware** + **Prioritization** |
| Parallel tool calls with error handling | Parallelization + **Exception Handling** + **Tool Use** |
| Fan-out planning with synthesis | Parallelization + **Planning** + **Reflection** |

## References

- Google ADK ParallelAgent: https://google.github.io/adk-docs/agents/workflow-agents/
- Python asyncio Documentation: https://docs.python.org/3/library/asyncio.html
- LangGraph Parallel Execution: https://langchain-ai.github.io/langgraph/
- Crew AI Parallel Tasks: https://docs.crewai.com/
