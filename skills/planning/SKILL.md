---
name: planning
description: This skill should be used when the user wants to "plan complex tasks", "break down goals into steps", "autonomous task planning", "goal-oriented agents", "multi-step workflow automation", "generate execution plans", "strategic task decomposition", "adaptive planning", "deep research automation", "task planner agent", "plan and execute", "hierarchical planning", "agent task decomposition", "orchestration planning", "autonomous planner", or build agents that formulate and execute sequences of actions to achieve a goal. Also responds to Korean: "복잡한 작업 계획", "목표를 단계로 분해", "자율 작업 계획", "다단계 워크플로우 자동화", "실행 계획 생성", "딥 리서치 자동화", "플래닝 에이전트", "계획 세우는 에이전트", "플래닝 해줘", "자동으로 계획 세워줘", "작업 분해해줘", "리서치 자동화해줘". Also responds to Japanese: "複雑なタスクの計画", "目標をステップに分解", "自律的タスク計画", "マルチステップワークフロー", "プランニングエージェント", "自動で計画を立てて", "タスク分解", "ディープリサーチ自動化", "計画を立てるエージェント", "実行計画生成" Also responds to Chinese: "规划复杂任务", "将目标分解为步骤", "自主任务规划", "多步骤工作流自动化", "生成执行计划", "规划智能体", "自动制定计划", "任务分解", "深度研究自动化", "帮我做计划".. Apply this skill to design or implement the Planning agentic design pattern.
version: 1.0.0
---

# Planning Pattern

## Overview

The **Planning Pattern** enables an agent to move beyond simple reactive responses by formulating a structured sequence of actions to achieve a complex, high-level goal. Rather than reacting to each input in isolation, a planning agent first analyzes the goal, decomposes it into manageable sub-tasks, and then executes those steps — adapting its plan as new information emerges.

**Core Principle:** Transform a high-level objective into a structured, executable plan of actionable steps — then adapt the plan as reality unfolds.

## When This Skill Applies

Activate this pattern when:
- A user's request is too complex to handle with a single action or tool call
- The task requires multiple interdependent steps executed in a logical order
- The solution path is not known in advance and must be discovered dynamically
- Workflows involve orchestrating multiple tools, APIs, or sub-agents
- The agent must adapt its approach based on intermediate results
- Tasks require systematic information gathering, synthesis, and reporting

**Rule of thumb:** Use Planning when the "how" needs to be discovered, not just the "what." If the solution requires a sequence of interdependent operations to reach a synthesized outcome — plan first.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Understand the planning requirements:
1. What is the final goal state? What does success look like?
2. What is the current state, including available resources and constraints?
3. What sub-tasks are needed to move from initial state to goal state?
4. What tools, APIs, or agents are available to execute each sub-task?

### PLAN
Design the planning architecture:
1. Map the high-level goal into a directed sequence of executable sub-tasks
2. Identify dependencies between sub-tasks (which must complete before others begin)
3. Assign tools, agents, or capabilities to each sub-task
4. Define checkpoints where the plan can be reassessed based on intermediate results
5. Plan for contingencies: what if a step fails or returns unexpected results?

### ACTION
Implement and execute the plan:
1. Generate the initial plan (list of steps with assigned tools/agents)
2. Execute each step in order, collecting outputs
3. After each step, evaluate whether the plan needs adjustment
4. If obstacles arise (API failure, unexpected data), reformulate the relevant steps
5. Synthesize all outputs into the final deliverable

## Core Planning Patterns

### Linear Plan Execution
The simplest planning structure — a fixed sequence of steps:
```
Goal → Step 1 → Step 2 → Step 3 → ... → Step N → Final Output
```
Best for: well-understood workflows where the solution path is predictable.

### Adaptive Planning
The plan is updated dynamically based on intermediate results:
```
Goal → Plan → Execute Step 1 → Evaluate → [Adjust Plan if needed] → Execute Step 2 → ...
```
Best for: research tasks, novel problems where outcomes cannot be predicted.

### Hierarchical Planning
High-level plans are decomposed into sub-plans:
```
Goal → High-Level Plan [Sub-Goal 1, Sub-Goal 2, Sub-Goal 3]
     → Sub-Goal 1 Plan [Step 1.1, Step 1.2] → Execute
     → Sub-Goal 2 Plan [Step 2.1, Step 2.2] → Execute
     → Synthesize
```
Best for: very complex objectives requiring multi-domain expertise.

### Iterative Research Planning (Deep Research)
The agent generates a plan, presents it for review, then executes iteratively:
```
Query → Research Plan (user-reviewable) → Execute Searches → Evaluate Gaps
      → Formulate Follow-up Searches → Synthesize → Final Report
```
Best for: complex research tasks requiring exhaustive information gathering.

## Practical Use Cases

### Research Report Generation
```
Goal: "Generate a report on European VC investment trends"
Plan:
  1. Search for total VC investment volume in Europe 2024-2025
  2. Identify top countries by investment volume
  3. Find data on year-over-year growth rates
  4. Identify countries with accelerating investment
  5. Analyze key sectors driving investment
  6. Synthesize findings into a structured report
```

### Employee Onboarding Automation
```
Goal: "Onboard new employee John Doe"
Plan:
  1. Create system accounts (email, Slack, GitHub)
  2. Assign required training modules
  3. Schedule orientation meetings
  4. Set up access permissions for relevant systems
  5. Send welcome email with resources
  6. Create 30/60/90 day check-in calendar events
```

### Competitive Analysis
```
Goal: "Analyze competitors in the CRM market"
Plan:
  1. Identify top 5 CRM competitors
  2. For each competitor: gather pricing, features, reviews
  3. Perform SWOT analysis for each
  4. Identify market positioning gaps
  5. Synthesize comparative report
```

## Implementation with Google ADK

### ADK Agent with Explicit Planning Instruction
```python
from google.adk.agents import LlmAgent, SequentialAgent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

def web_search(query: str) -> str:
    """Search the web for information on a topic."""
    return f"Search results for: {query}"

def analyze_data(data: str) -> str:
    """Analyze and extract key insights from data."""
    return f"Analysis of: {data[:100]}..."

def generate_report(content: str, title: str) -> str:
    """Generate a structured report from content."""
    return f"# {title}\n\n{content}"

# Planning agent - creates the execution plan
planner = LlmAgent(
    name="Planner",
    model="gemini-2.5-flash",
    instruction="""You are a strategic planner. When given a complex goal:
    1. Break it down into 3-7 concrete, executable steps
    2. For each step, specify: what to do, what tool to use, what output is expected
    3. Output the plan as a numbered list in JSON format:
       {"steps": [{"step": 1, "action": "...", "tool": "...", "output": "..."}]}

    Keep steps focused and achievable. Think about dependencies."""
)

# Executor agent - carries out the plan
executor = LlmAgent(
    name="Executor",
    model="gemini-2.5-flash",
    instruction="""You are a meticulous executor. Given a plan, execute each step
    in order using the available tools. After each step, briefly summarize what
    you found before proceeding to the next step. Adapt if a step produces
    unexpected results.""",
    tools=[
        FunctionTool(web_search),
        FunctionTool(analyze_data),
        FunctionTool(generate_report)
    ]
)

# Synthesizer - produces final output from executed steps
synthesizer = LlmAgent(
    name="Synthesizer",
    model="gemini-2.5-flash",
    instruction="""You are a synthesis expert. Take the outputs from all executed
    steps and combine them into a coherent, well-structured final report.
    Ensure all findings are integrated and the narrative flows logically."""
)

# Chain planner → executor → synthesizer
research_pipeline = SequentialAgent(
    name="ResearchPipeline",
    sub_agents=[planner, executor, synthesizer]
)

session_service = InMemorySessionService()
runner = Runner(agent=research_pipeline, app_name="planning_app", session_service=session_service)
session_service.create_session(app_name="planning_app", user_id="user1", session_id="session1")
message = Content(parts=[Part(text="Research and report on the top 3 emerging AI trends in 2025")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

### CrewAI Planning Implementation
```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_google_genai import ChatGoogleGenerativeAI

load_dotenv()
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

# Planner-writer agent that creates a plan then executes it
planner_writer_agent = Agent(
    role='Article Planner and Writer',
    goal='Plan and then write a concise, engaging summary on a specified topic.',
    backstory=(
        'You are an expert technical writer and content strategist. '
        'Your strength lies in creating a clear, actionable plan before writing, '
        'ensuring the final summary is both informative and easy to digest.'
    ),
    verbose=True,
    allow_delegation=False,
    llm=llm
)

topic = "The impact of AI agents on enterprise software development"
high_level_task = Task(
    description=(
        f"1. Create a bullet-point plan for a summary on the topic: '{topic}'.\n"
        f"2. Write the summary based on your plan, keeping it around 200 words."
    ),
    expected_output=(
        "A final report containing two distinct sections:\n\n"
        "### Plan\n"
        "- A bulleted list outlining the main points of the summary\n\n"
        "### Summary\n"
        "- A concise and well-structured summary of the topic."
    ),
    agent=planner_writer_agent,
)

crew = Crew(
    agents=[planner_writer_agent],
    tasks=[high_level_task],
    process=Process.sequential,
)

result = crew.kickoff()
print(result)
```

### LangGraph Adaptive Planning
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List
from google import genai

class PlanningState(TypedDict):
    goal: str
    plan: List[str]
    completed_steps: List[str]
    current_step_index: int
    final_report: str

client = genai.Client()

def create_plan_node(state: PlanningState) -> dict:
    """Generate an execution plan for the goal."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"Break down this goal into 4-6 concrete steps:\n{state['goal']}\n\n"
        "Output as a numbered list, one step per line."
    )
    steps = [line.strip() for line in response.text.split('\n') if line.strip() and line[0].isdigit()]
    return {"plan": steps, "current_step_index": 0, "completed_steps": []}

def execute_step_node(state: PlanningState) -> dict:
    """Execute the current step in the plan."""
    current_step = state["plan"][state["current_step_index"]]
    context = "\n".join(state["completed_steps"]) if state["completed_steps"] else "No previous steps"

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"Goal: {state['goal']}\n\n"
        f"Previous steps completed:\n{context}\n\n"
        f"Now execute this step: {current_step}\n\n"
        "Provide a detailed result of executing this step."
    )

    completed = state["completed_steps"] + [f"Step {state['current_step_index']+1}: {response.text[:500]}"]
    return {
        "completed_steps": completed,
        "current_step_index": state["current_step_index"] + 1
    }

def synthesize_node(state: PlanningState) -> dict:
    """Synthesize all completed steps into a final report."""
    all_steps = "\n\n".join(state["completed_steps"])
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"Goal: {state['goal']}\n\nCompleted steps:\n{all_steps}\n\n"
        "Synthesize all findings into a coherent final report."
    )
    return {"final_report": response.text}

def should_continue(state: PlanningState) -> str:
    if state["current_step_index"] >= len(state["plan"]):
        return "synthesize"
    return "execute"

graph = StateGraph(PlanningState)
graph.add_node("plan", create_plan_node)
graph.add_node("execute", execute_step_node)
graph.add_node("synthesize", synthesize_node)
graph.set_entry_point("plan")
graph.add_edge("plan", "execute")
graph.add_conditional_edges("execute", should_continue, {
    "execute": "execute",
    "synthesize": "synthesize"
})
graph.add_edge("synthesize", END)

planner = graph.compile()
result = planner.invoke({
    "goal": "Research the economic impact of AI agents on software development costs",
    "plan": [],
    "completed_steps": [],
    "current_step_index": 0,
    "final_report": ""
})
print(result["final_report"])
```

## Key Takeaways

- **Goal decomposition**: Planning breaks high-level objectives into concrete, executable steps
- **Adaptability**: The plan is a starting point, not a rigid script — good planners adapt to new information
- **Flexibility vs. predictability**: Use planning when the "how" must be discovered; use fixed workflows when the path is known
- **LLM as planner**: LLMs excel at generating plausible plans from task descriptions
- **Google DeepResearch**: An exemplary planning agent — creates iterative research plans that adapt based on search findings
- **Sequential finalization**: Synthesis/aggregation of plan outputs is typically sequential

## Anti-Patterns to Avoid

- **Over-planning**: Don't generate 20-step plans for tasks a 3-step workflow can handle
- **Rigid adherence**: A plan that ignores contradictory evidence leads to wrong conclusions
- **No contingencies**: Plans should handle step failures gracefully, not crash the entire workflow
- **Disconnected steps**: Each step must build on previous results — avoid steps that ignore prior context
- **Planning without execution**: A plan is only useful when followed by action — avoid plans that stop at planning

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Autonomous research agent | Planning + **Tool Use** + **Reflection** |
| Multi-agent task coordination | Planning + **Multi-Agent Collaboration** + **Parallelization** |
| Goal-driven adaptive planning | Planning + **Goal Setting** + **Reasoning** |
| Resilient long-running tasks | Planning + **Exception Handling** + **Human-in-the-Loop** |

## References

- Google DeepResearch (Gemini Feature): https://gemini.google.com
- OpenAI Deep Research API: https://openai.com/index/introducing-deep-research/
- Perplexity Deep Research: https://www.perplexity.ai/hub/blog/introducing-perplexity-deep-research
- CrewAI Documentation: https://docs.crewai.com/
- Google ADK Documentation: https://google.github.io/adk-docs/
