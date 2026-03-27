---
name: appendix-agentic-frameworks
description: This skill should be used when the user wants to "choose agent framework", "compare LangChain vs LangGraph", "compare ADK vs LangGraph", "which framework to use for agents", "LangGraph tutorial", "CrewAI setup", "agentic framework comparison", "framework selection guide", "LangChain LCEL", "Google ADK tutorial", "AutoGen setup", "LlamaIndex agents", "Haystack agents", "LangGraph vs CrewAI", "build agent with ADK", "agent framework guide", "pick agent library", "LangChain vs ADK", or understand the trade-offs between the major agentic frameworks. Also responds to Korean: "에이전트 프레임워크 선택", "LangChain LangGraph 비교", "ADK 대 LangGraph", "에이전트 프레임워크 비교", "CrewAI 설정", "AutoGen 설정", "ADK 써보고 싶어", "LangGraph 시작", "LangChain 써보고 싶어", "CrewAI 써보고 싶어", "어떤 프레임워크 써야 해". Also responds to Japanese: "エージェントフレームワーク選択", "LangChain vs LangGraph比較", "ADK対LangGraph", "どのフレームワークを使うべきか", "LangGraphチュートリアル", "ADKを使いたい", "CrewAI設定", "LangChainを使いたい", "フレームワーク比較", "AutoGen設定" Also responds to Chinese: "智能体框架选择", "LangChain vs LangGraph对比", "ADK对比LangGraph", "该用哪个框架", "LangGraph教程", "想用ADK", "CrewAI配置", "想用LangChain", "框架对比", "AutoGen配置".. Apply this skill to select and correctly use the right agentic framework for a given use case.
version: 1.0.0
---

# Appendix C - Quick Overview of Agentic Frameworks

## Overview

The **Agentic Frameworks** landscape provides developers with structured tools for building, orchestrating, and deploying AI agents. Each framework operates at a different level of abstraction and optimizes for different design priorities — understanding their trade-offs is essential for building reliable, maintainable agent systems.

This appendix provides a comparative overview of the major frameworks: **LangChain**, **LangGraph**, **Google ADK**, **CrewAI**, **Microsoft AutoGen**, **LlamaIndex**, and **Haystack** — when to use each, their core abstractions, and key code patterns.

**Core Principle:** No framework is universally best — choose based on whether your workflow is linear (LangChain), cyclical (LangGraph), team-based (ADK/CrewAI), or data-intensive (LlamaIndex).

## Framework Comparison Matrix

| Framework | Core Abstraction | Workflow Type | State Management | Best For |
|-----------|-----------------|---------------|-----------------|----------|
| **LangChain** | Chain (LCEL) | Linear DAG | Stateless per run | Simple pipelines, RAG, extraction |
| **LangGraph** | Graph of nodes | Cyclical + loops | Explicit, persistent | Complex agents, tool loops, HITL |
| **Google ADK** | Agent team | Orchestrated multi-agent | Implicit (framework) | Production multi-agent systems |
| **CrewAI** | Agent + Task + Crew | Sequential/hierarchical | Role-based | Team simulation, research workflows |
| **AutoGen** | Conversational agents | Conversation-driven | Conversational | Dynamic multi-agent conversations |
| **LlamaIndex** | Data pipeline | RAG-focused | Index-based | Knowledge-intensive agents, RAG |
| **Haystack** | Pipeline nodes | Modular search | Search-optimized | Enterprise search, QA systems |

## When This Skill Applies

Activate this framework selection guide when:
- Starting a new agent project and choosing the technology stack
- Migrating from one framework to another
- Debugging framework-specific behavior or limitations
- Combining multiple frameworks in a hybrid architecture
- Evaluating build-vs-buy for agentic infrastructure

**Rule of thumb:**
- Linear workflow → **LangChain**
- Cyclical/looping agent → **LangGraph**
- Multi-agent team production → **Google ADK**
- Collaborative specialist team → **CrewAI**
- Data-intensive RAG → **LlamaIndex**

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Identify framework requirements:
1. Is the workflow linear (A→B→C) or cyclical (loop until done)?
2. Does the agent need to use tools and loop on results?
3. Are multiple specialized agents collaborating?
4. Is state persistence needed across steps or sessions?
5. Is the primary challenge data retrieval or agent orchestration?

### PLAN
Select and configure the framework:
1. Match workflow pattern to framework strength (see matrix above)
2. Define agent roles, tools, and communication protocols
3. Plan state schema — what data flows between nodes/agents?
4. Design error handling and retry logic within the framework
5. Plan evaluation and observability integration (LangSmith, ADK UI)

### ACTION
Implement with chosen framework:
1. Set up framework dependencies and configuration
2. Build core agent/graph/chain structure
3. Integrate tools and external services
4. Add state management and memory
5. Test with representative tasks and validate outputs

## Implementation: LangChain (Linear Pipelines)

### LCEL Chain Pattern
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser, JsonOutputParser
from pydantic import BaseModel

# LangChain excels at linear, predictable pipelines
# Use LCEL pipe syntax: prompt | model | parser

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

# Simple extraction chain
class ArticleSummary(BaseModel):
    title: str
    key_points: list[str]
    sentiment: str

extraction_prompt = ChatPromptTemplate.from_template("""
Extract structured information from this article.

Article: {article}

Return JSON with: title, key_points (list), sentiment (positive/negative/neutral)
""")

# LCEL chain: prompt | model | parser
extraction_chain = extraction_prompt | llm | JsonOutputParser()

# Multi-step pipeline: extract → summarize → translate
summarize_prompt = ChatPromptTemplate.from_template(
    "Summarize this in 2 sentences: {text}"
)
translate_prompt = ChatPromptTemplate.from_template(
    "Translate to {language}: {text}"
)

# Compose into a pipeline
summary_chain = (
    {"text": extraction_chain | (lambda x: str(x["key_points"]))}
    | summarize_prompt
    | llm
    | StrOutputParser()
)

# Usage
result = summary_chain.invoke({
    "article": "AI agents are transforming software development...",
    "language": "Spanish"
})
```

## Implementation: LangGraph (Cyclical Agents)

### StateGraph with Tool Loop
```python
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import ToolNode
from langchain_core.messages import HumanMessage, AIMessage
from typing import TypedDict, Annotated
import operator

# LangGraph excels at cyclical agents that loop until done
# Core: define State, Nodes (functions), and Edges (conditions)

class AgentState(TypedDict):
    messages: Annotated[list, operator.add]
    iteration_count: int

def should_continue(state: AgentState) -> str:
    """Decide whether to call tools or end."""
    last_message = state["messages"][-1]

    # If the agent called tools, continue the loop
    if hasattr(last_message, 'tool_calls') and last_message.tool_calls:
        return "tools"

    # Safety: max iterations
    if state["iteration_count"] >= 10:
        return "end"

    return "end"

def call_agent(state: AgentState, llm_with_tools) -> AgentState:
    """Node: call LLM with current message history."""
    response = llm_with_tools.invoke(state["messages"])
    return {
        "messages": [response],
        "iteration_count": state["iteration_count"] + 1
    }

# Build the graph
def build_react_agent(tools: list):
    from langchain_google_genai import ChatGoogleGenerativeAI
    from langgraph.prebuilt import ToolNode

    llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")
    llm_with_tools = llm.bind_tools(tools)

    builder = StateGraph(AgentState)

    # Add nodes
    builder.add_node("agent", lambda s: call_agent(s, llm_with_tools))
    builder.add_node("tools", ToolNode(tools))

    # Add edges
    builder.add_edge(START, "agent")
    builder.add_conditional_edges("agent", should_continue, {"tools": "tools", "end": END})
    builder.add_edge("tools", "agent")  # After tools, go back to agent

    return builder.compile()

# Usage
from langchain_community.tools import DuckDuckGoSearchRun
agent = build_react_agent([DuckDuckGoSearchRun()])
result = agent.invoke({
    "messages": [HumanMessage(content="What is the current price of gold?")],
    "iteration_count": 0
})
```

## Implementation: Google ADK (Multi-Agent Production)

### ADK Agent Team
```python
from google.adk.agents import LlmAgent, SequentialAgent, ParallelAgent
from google.adk.tools import FunctionTool, google_search
from google.adk.tools.tool_context import ToolContext

# Google ADK excels at structured multi-agent production systems
# Core: pre-built SequentialAgent and ParallelAgent orchestrators

def search_web(query: str, tool_context: ToolContext) -> dict:
    """Search the web for current information."""
    # In production, use google_search tool or custom search API
    return {"results": f"Search results for: {query}"}

def analyze_data(data: str, tool_context: ToolContext) -> dict:
    """Analyze provided data and return insights."""
    return {"analysis": f"Analysis of: {data}"}

# Individual specialist agents
researcher = LlmAgent(
    name="Researcher",
    model="gemini-2.5-flash",
    instruction="Research the given topic thoroughly using web search. Return structured findings.",
    tools=[FunctionTool(search_web)]
)

analyst = LlmAgent(
    name="Analyst",
    model="gemini-2.5-flash",
    instruction="Analyze the research findings and provide data-driven insights.",
    tools=[FunctionTool(analyze_data)]
)

writer = LlmAgent(
    name="Writer",
    model="gemini-2.5-flash",
    instruction="Write a clear, concise report based on the analysis. Use professional language."
)

# Orchestrate: Research → Analyze → Write (sequential)
report_pipeline = SequentialAgent(
    name="ReportPipeline",
    sub_agents=[researcher, analyst, writer]
)

# Or run research tasks in parallel
parallel_research = ParallelAgent(
    name="ParallelResearch",
    sub_agents=[researcher, researcher]  # Two researchers, different topics
)
```

## Implementation: CrewAI (Collaborative Teams)

### Multi-Agent Crew
```python
from crewai import Agent, Task, Crew, Process
from langchain_google_genai import ChatGoogleGenerativeAI

# CrewAI excels at team metaphor: agents have roles, backstories, and goals
# Core: Agent + Task + Crew + Process

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

# Define agents with personas
researcher = Agent(
    role="Senior Research Analyst",
    goal="Uncover cutting-edge developments and trends in {topic}",
    backstory="""You are an experienced researcher with 10 years in tech journalism.
    You excel at finding authoritative sources and synthesizing complex information.""",
    verbose=True,
    allow_delegation=False,
    llm=llm
)

writer = Agent(
    role="Tech Content Strategist",
    goal="Write compelling articles about {topic} for a technical audience",
    backstory="""You are a skilled writer who translates complex technical topics
    into engaging, accessible content. You know what developers want to read.""",
    verbose=True,
    allow_delegation=True,
    llm=llm
)

# Define tasks
research_task = Task(
    description="Research the latest developments in {topic}. Focus on the past 6 months.",
    expected_output="A structured report with 5-7 key findings, each with supporting evidence.",
    agent=researcher
)

writing_task = Task(
    description="Write a 500-word article based on the research findings.",
    expected_output="A well-structured article with title, introduction, 3 sections, and conclusion.",
    agent=writer
)

# Assemble and run the crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential,  # or Process.hierarchical for manager-led
    verbose=True
)

result = crew.kickoff(inputs={"topic": "AI agent frameworks"})
```

## Framework Decision Guide

```
Start here:
│
├── Is your workflow linear (A→B→C, no loops)?
│   └── YES → Use LangChain (LCEL)
│
├── Does your agent need to loop (tool use, retry, plan-execute)?
│   └── YES → Use LangGraph
│
├── Are multiple specialized agents collaborating on production workloads?
│   └── YES → Use Google ADK (if Google Cloud) or CrewAI (if team simulation)
│
├── Is the primary challenge data retrieval from large document sets?
│   └── YES → Use LlamaIndex
│
├── Do you need production-grade enterprise search?
│   └── YES → Use Haystack
│
└── Do you want agents to interact via conversation (AutoGen style)?
    └── YES → Use Microsoft AutoGen
```

## Other Frameworks Summary

**Microsoft AutoGen**: Conversation-driven multi-agent framework. Agents solve tasks through back-and-forth dialogue. Flexible but less predictable execution paths. Best for: research tasks, code generation with critic loops.

**LlamaIndex**: Data framework connecting LLMs to external/private data sources. Exceptional RAG pipelines, data indexing, and query engines. Best for: knowledge-intensive agents requiring deep data retrieval.

**Haystack**: Production-ready search systems powered by LLMs. Modular, interoperable pipeline nodes for document retrieval, QA, and summarization. Best for: enterprise-grade search at scale.

## Key Takeaways

- **LangChain for linear**: If you can draw it as A→B→C without loops, LangChain LCEL is the cleanest solution
- **LangGraph for loops**: Any agent that uses tools, retries, or loops until satisfied needs LangGraph's cyclical graph
- **ADK for production multi-agent**: Google ADK's pre-built SequentialAgent/ParallelAgent eliminates boilerplate orchestration
- **CrewAI for team metaphor**: When thinking in terms of roles, responsibilities, and team collaboration
- **Don't mix frameworks unnecessarily**: Each framework adds its own state and session management — minimize boundary crossings
- **Framework ≠ quality**: The best agent with a poor prompt still produces poor results — framework choice is secondary to prompt engineering

## Anti-Patterns to Avoid

- **Choosing by popularity**: Pick the framework that matches your workflow pattern, not the one with the most GitHub stars
- **Over-engineering with LangGraph**: For simple sequential tasks, LangGraph adds unnecessary complexity — LangChain is sufficient
- **Under-engineering with LangChain**: Agents that need loops cannot be built cleanly with LangChain DAGs — accept the upgrade to LangGraph
- **Framework lock-in**: Build business logic independent of framework primitives where possible — frameworks evolve fast

## Related Skills

This skill provides the **framework selection layer** for all other patterns. Use it in combination with:

| Scenario | Skills to Combine |
|----------|-------------------|
| Choose framework for prompt chaining | Agentic Frameworks + **Prompt Chaining** |
| Choose framework for multi-agent systems | Agentic Frameworks + **Multi-Agent Collaboration** + **A2A** |
| Choose framework for production pipelines | Agentic Frameworks + **Evaluation** + **Exception Handling** |
| Framework comparison for RAG agents | Agentic Frameworks + **RAG** + **Memory Management** |

## References

- LangChain LCEL: https://python.langchain.com/docs/concepts/lcel/
- LangGraph: https://langchain-ai.github.io/langgraph/
- Google ADK: https://google.github.io/adk-docs/
- CrewAI: https://docs.crewai.com/
- Microsoft AutoGen: https://microsoft.github.io/autogen/
- LlamaIndex: https://docs.llamaindex.ai/
- Haystack: https://haystack.deepset.ai/
