---
name: multi-agent-collaboration
description: This skill should be used when the user wants to "multiple agents working together", "agent teams", "specialist agents", "hierarchical agent systems", "agent orchestration", "multi-agent pipeline", "supervisor agent", "agent network", "collaborative AI", "expert agent teams", "researcher writer agent", "multi-agent system", "crew of agents", "agent delegation", "specialized agent roles", "build agent team", "parallel specialist agents", or build systems where multiple specialized agents cooperate to accomplish a shared objective. Also responds to Korean: "다중 에이전트 협업", "에이전트 팀 구성", "전문 에이전트 조합", "계층적 에이전트 시스템", "에이전트 오케스트레이션", "슈퍼바이저 에이전트 패턴", "멀티 에이전트", "에이전트 여러 개", "에이전트팀 구성", "에이전트 여러 개 협업시켜줘", "전문가 에이전트팀 만들어줘", "역할 분담 에이전트". Also responds to Japanese: "複数エージェントの協調", "エージェントチーム", "専門エージェント組み合わせ", "階層的エージェントシステム", "マルチエージェント", "エージェントを複数使いたい", "役割分担エージェント", "オーケストレーション", "スーパーバイザーエージェント", "専門家チームエージェント" Also responds to Chinese: "多智能体协作", "智能体团队", "专业智能体组合", "层次化智能体系统", "多智能体系统", "让多个智能体一起工作", "角色分工智能体", "编排调度", "监督智能体模式", "专家团队智能体".. Apply this skill to design or implement the Multi-Agent Collaboration agentic design pattern.
version: 1.0.0
---

# Multi-Agent Collaboration Pattern

## Overview

The **Multi-Agent Collaboration Pattern** structures a system as a cooperative ensemble of distinct, specialized agents rather than a single monolithic agent. Each agent has a defined role, specific goals, and potentially unique tool access or domain knowledge. The power of this pattern lies in the interaction and synergy between agents — the collective output surpasses what any individual agent could produce alone.

**Core Principle:** Divide by specialization, conquer through cooperation — assign each agent what it does best, then orchestrate their collaboration.

## When This Skill Applies

Activate this pattern when:
- The task spans multiple domains requiring different types of expertise
- Different phases of a workflow have fundamentally different requirements
- A single agent would be overloaded by the breadth of tools and responsibilities
- Quality improves through critic-reviewer feedback loops between agents
- Scalability demands distributing work across independent specialized units
- Tasks can be decomposed into sub-problems, each handled by a dedicated agent

**Rule of thumb:** Use Multi-Agent Collaboration when a task is complex enough that a human team of specialists would handle it better than a single generalist.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the collaboration requirements:
1. What are the distinct sub-domains or phases of the task?
2. What specialized role is needed for each sub-domain?
3. How will agents communicate and hand off work?
4. What is the overall orchestration model (sequential, parallel, hierarchical)?

### PLAN
Design the multi-agent architecture:
1. Define each agent with a clear role, goal, and tool set
2. Choose the communication structure (Sequential, Network, Supervisor, Hierarchical, Custom)
3. Design the handoff protocol: how does one agent's output become the next agent's input?
4. Plan for agent failure: what happens if one specialist agent fails?
5. Define the aggregation/synthesis step that produces the final unified output

### ACTION
Implement and orchestrate the agent team:
1. Create specialized agents with focused system prompts and appropriate tools
2. Wire agent interactions through the chosen orchestration framework
3. Test each agent in isolation before testing team collaboration
4. Monitor inter-agent communication for information loss or misinterpretation
5. Validate final output quality against the combined capabilities of the team

## Collaboration Structures

### 1. Sequential Handoffs (Pipeline)
One agent completes a task and passes output to the next:
```
Researcher → [Research Summary] → Writer → [Draft] → Editor → [Final Article]
```
Best for: content creation, data transformation pipelines.

### 2. Parallel Processing (Fan-Out + Fan-In)
Multiple agents work simultaneously on different parts:
```
Research Task → [Academic Agent || Web Agent || Database Agent] → Synthesizer
```
Best for: multi-source research, parallel data gathering.

### 3. Supervisor (Hierarchical Delegation)
A manager agent delegates to specialized workers:
```
                    Supervisor
                   /    |    \
          Researcher  Writer  Editor
```
Best for: complex workflows requiring dynamic task allocation.

### 4. Debate and Consensus
Multiple agents independently analyze the same problem; results are aggregated:
```
Query → [Agent A Analysis || Agent B Analysis || Agent C Analysis]
      → Consensus Agent → Final Verdict
```
Best for: high-stakes decisions, bias reduction, quality validation.

### 5. Critic-Reviewer
Agents create outputs; other agents critically assess them:
```
Generator → [Draft] → Critic [Compliance, Quality, Safety] → Reviser → Final Output
```
Best for: code generation, legal document review, policy compliance.

### 6. Expert Teams
Domain specialists collaborate on complex multi-faceted problems:
```
Complex Query → Requirements Analyst → Code Generator → Tester → Documentation Writer
```
Best for: software development, research papers, business strategy.

## Communication Models

| Model | Description | Best For |
|-------|-------------|----------|
| Single Agent | One agent with all tools | Simple, focused tasks |
| Network | Peer-to-peer agent communication | Resilient, decentralized tasks |
| Supervisor | Central coordinator + workers | Structured delegation |
| Supervisor as Tool | Coordinator provides services | Resource sharing |
| Hierarchical | Multiple levels of supervisors | Very complex, large-scale tasks |
| Custom | Hybrid or novel topology | Domain-specific optimization |

## Practical Use Cases

### Complex Research and Analysis
```
Research Goal →
  Agent 1 (Academic DB Search) → Papers and citations
  Agent 2 (Web Search) → Current news and trends
  Agent 3 (Data Analyst) → Statistical analysis
  Agent 4 (Synthesizer) → Integrated research report
```

### Software Development Team
```
Feature Request →
  Agent 1 (Requirements Analyst) → Specifications
  Agent 2 (Code Generator) → Initial implementation
  Agent 3 (Tester) → Test cases and bug reports
  Agent 4 (Refactorer) → Optimized code
  Agent 5 (Documentation Writer) → API docs
```

### Marketing Campaign Creation
```
Campaign Brief →
  Agent 1 (Market Researcher) → Audience insights
  Agent 2 (Copywriter) → Ad copy and messaging
  Agent 3 (Visual Concept Agent) → Design briefs
  Agent 4 (Social Media Scheduler) → Publication plan
```

### Financial Analysis
```
Analysis Request →
  Agent 1 (Data Fetcher) → Stock prices, economic indicators
  Agent 2 (News Analyst) → Sentiment from financial news
  Agent 3 (Technical Analyst) → Chart patterns and signals
  Agent 4 (Report Writer) → Investment recommendations
```

## Implementation with CrewAI

### Researcher + Writer Collaboration
```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_google_genai import ChatGoogleGenerativeAI

load_dotenv()
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

# Agent 1: Research Specialist
researcher = Agent(
    role='Senior Research Analyst',
    goal='Find and summarize the latest trends in AI.',
    backstory="You are an experienced research analyst with a knack for "
              "identifying key trends and synthesizing information.",
    verbose=True,
    allow_delegation=False,
    llm=llm
)

# Agent 2: Content Creation Specialist
writer = Agent(
    role='Technical Content Writer',
    goal='Write a clear and engaging blog post based on research findings.',
    backstory="You are a skilled writer who can translate complex technical "
              "topics into accessible content for a general audience.",
    verbose=True,
    allow_delegation=False,
    llm=llm
)

# Task 1: Research (assigned to researcher agent)
research_task = Task(
    description="Research the top 3 emerging trends in Artificial Intelligence "
                "in 2024-2025. Focus on practical applications and potential impact.",
    expected_output="A detailed summary of the top 3 AI trends, including key points and sources.",
    agent=researcher,
)

# Task 2: Writing (depends on research task output)
writing_task = Task(
    description="Write a 500-word blog post based on the research findings. "
                "The post should be engaging and easy for a general audience to understand.",
    expected_output="A complete 500-word blog post about the latest AI trends.",
    agent=writer,
    context=[research_task],  # Writer receives researcher's output
)

# Crew orchestrates sequential collaboration
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential,
    verbose=True
)

result = crew.kickoff()
print(result)
```

### Google ADK Supervisor Pattern
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

# Specialized sub-agents with domain-specific tools
def search_academic(query: str) -> str:
    """Search academic databases for research papers."""
    return f"Academic results for: {query}"

def search_industry(query: str) -> str:
    """Search industry news and market reports."""
    return f"Industry results for: {query}"

def analyze_data(data: str) -> str:
    """Perform statistical analysis on data."""
    return f"Analysis: {data[:100]}"

academic_agent = LlmAgent(
    name="AcademicResearcher",
    model="gemini-2.5-flash",
    description="Specializes in academic research, papers, and scientific literature.",
    instruction="Search academic sources for peer-reviewed research. Cite sources.",
    tools=[FunctionTool(search_academic)]
)

industry_agent = LlmAgent(
    name="IndustryAnalyst",
    model="gemini-2.5-flash",
    description="Specializes in industry trends, market data, and business applications.",
    instruction="Search industry sources for market trends and practical applications.",
    tools=[FunctionTool(search_industry)]
)

data_agent = LlmAgent(
    name="DataAnalyst",
    model="gemini-2.5-flash",
    description="Specializes in quantitative analysis and statistical interpretation.",
    instruction="Analyze numerical data, identify patterns, and provide statistical insights.",
    tools=[FunctionTool(analyze_data)]
)

# Supervisor coordinates the specialist team (ADK auto-routes based on descriptions)
supervisor = LlmAgent(
    name="ResearchSupervisor",
    model="gemini-2.5-flash",
    instruction="""You are the lead research coordinator. For each research request:
    1. Determine which specialists are needed
    2. Delegate to appropriate sub-agents based on their expertise
    3. Collect and synthesize their findings
    4. Produce a comprehensive, integrated report

    Delegate academic questions to AcademicResearcher,
    business/market questions to IndustryAnalyst,
    and quantitative analysis to DataAnalyst.""",
    sub_agents=[academic_agent, industry_agent, data_agent]
)

session_service = InMemorySessionService()
runner = Runner(agent=supervisor, app_name="multi_agent_app", session_service=session_service)
session_service.create_session(app_name="multi_agent_app", user_id="user1", session_id="session1")
message = Content(parts=[Part(text="Analyze the current state and future prospects of AI agents in enterprise software")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

### LangGraph Multi-Agent Network
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List, Annotated
import operator
from google import genai

class CollaborationState(TypedDict):
    task: str
    research_output: str
    analysis_output: str
    final_report: str

client = genai.Client()

def research_agent(state: CollaborationState) -> dict:
    """Agent 1: Gathers information."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"You are a research specialist. Research this topic thoroughly:\n{state['task']}\n\n"
        "Gather key facts, statistics, and insights."
    )
    return {"research_output": response.text}

def analysis_agent(state: CollaborationState) -> dict:
    """Agent 2: Analyzes the research."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"You are an analysis specialist. Analyze these research findings:\n{state['research_output']}\n\n"
        "Identify patterns, implications, and key conclusions."
    )
    return {"analysis_output": response.text}

def synthesis_agent(state: CollaborationState) -> dict:
    """Agent 3: Synthesizes everything into final output."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"Task: {state['task']}\n\n"
        f"Research: {state['research_output']}\n\n"
        f"Analysis: {state['analysis_output']}\n\n"
        "Synthesize into a comprehensive final report."
    )
    return {"final_report": response.text}

# Build sequential multi-agent graph
graph = StateGraph(CollaborationState)
graph.add_node("researcher", research_agent)
graph.add_node("analyst", analysis_agent)
graph.add_node("synthesizer", synthesis_agent)
graph.set_entry_point("researcher")
graph.add_edge("researcher", "analyst")
graph.add_edge("analyst", "synthesizer")
graph.add_edge("synthesizer", END)

multi_agent_system = graph.compile()

result = multi_agent_system.invoke({
    "task": "Analyze the impact of generative AI on software engineering productivity",
    "research_output": "",
    "analysis_output": "",
    "final_report": ""
})
print(result["final_report"])
```

## Key Takeaways

- **Specialization is power**: Each agent with a focused role outperforms a single generalist on complex multi-domain tasks
- **Communication structure matters**: Choose between Sequential, Network, Supervisor, Hierarchical, or Custom based on task requirements
- **Modularity and scalability**: Adding a new specialist agent is less disruptive than expanding a monolithic agent's capabilities
- **Robustness**: A failed sub-agent doesn't crash the system — other agents continue functioning
- **Synergistic output**: The combined output quality exceeds what any single agent produces alone
- **CrewAI and ADK**: Both frameworks natively support multi-agent collaboration with task context sharing

## Anti-Patterns to Avoid

- **Agent proliferation**: Creating too many agents for simple tasks adds overhead without benefit
- **Unclear role boundaries**: Overlapping agent responsibilities cause redundant work and conflicting outputs
- **Poor handoffs**: Information lost between agents leads to context drift and quality degradation
- **No orchestration**: Agents without a coordinator can produce incoherent, contradictory outputs
- **Ignoring failure modes**: A single agent failing should not silently corrupt the entire pipeline

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Cross-system agent federation | Multi-Agent Collaboration + **A2A** |
| Parallel specialist agent teams | Multi-Agent Collaboration + **Parallelization** |
| Orchestrated research pipeline | Multi-Agent Collaboration + **Planning** + **Tool Use** |
| Production multi-agent system | Multi-Agent Collaboration + **Guardrails** + **Evaluation** + **Exception Handling** |

## References

- Google ADK Multi-Agent Documentation: https://google.github.io/adk-docs/agents/multi-agents/
- CrewAI Multi-Agent Framework: https://docs.crewai.com/concepts/crews
- LangGraph Multi-Agent Workflows: https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi-agent-collaboration/
- AutoGen Multi-Agent Framework: https://microsoft.github.io/autogen/
