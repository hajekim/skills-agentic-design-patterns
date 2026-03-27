---
name: goal-setting
description: This skill should be used when the user wants to "agent goal management", "define agent objectives", "goal decomposition", "goal tracking", "monitor agent progress", "OKR for agents", "success criteria for agents", "goal-oriented agents", "agent performance monitoring", "task completion tracking", "hierarchical goal structure", "agent with goals", "measure agent success", "agent KPIs", "set agent objectives", "track agent progress", "goal-driven agent", or build agents that manage and track their own goals and progress toward defined objectives. Also responds to Korean: "에이전트 목표 관리", "에이전트 목표 정의", "목표 분해 및 추적", "에이전트 진행 모니터링", "목표 지향 에이전트", "에이전트 성과 기준", "목표 설정", "목표 추적 에이전트", "목표 달성 확인해줘", "에이전트 KPI 설정", "성과 지표 설정", "목표 달성률 추적". Also responds to Japanese: "エージェントの目標管理", "目標の設定と追跡", "目標分解", "エージェントの進捗管理", "目標指向エージェント", "成果基準の設定", "KPI設定", "目標達成を確認したい", "目標追跡エージェント", "目標達成率の追跡" Also responds to Chinese: "智能体目标管理", "设定和追踪目标", "目标分解", "监控智能体进度", "目标导向智能体", "设定成功标准", "KPI设定", "帮我追踪目标达成", "目标追踪智能体", "目标达成率追踪".. Apply this skill to design or implement the Goal Setting & Monitoring agentic design pattern.
version: 1.0.0
---

# Goal Setting & Monitoring Pattern

## Overview

The **Goal Setting & Monitoring Pattern** equips agents with the ability to define, track, and adapt toward explicit objectives. Rather than passively executing instructions, goal-driven agents maintain awareness of what success looks like, measure their progress against defined criteria, and adjust their strategies when they detect goal drift or underperformance.

**Core Principle:** An agent without a goal is just a tool — an agent with a goal is a collaborator that knows when it has succeeded (or failed).

## When This Skill Applies

Activate this pattern when:
- Agent tasks span multiple steps and require progress tracking
- Success must be explicitly defined and measurable, not just qualitative
- Agents need to self-correct when drifting from objectives
- Long-running tasks require checkpointing and status reporting
- Hierarchical goals with sub-goals need coordination
- Agents must balance competing objectives with different priorities
- Stakeholders need visibility into agent progress and completion

**Rule of thumb:** Use Goal Setting when "did the agent succeed?" has a real answer that must be verified, not just assumed.

## Goal Hierarchy

### SMART Goals for Agents
Effective agent goals are:
- **Specific**: Clear definition of what must be accomplished
- **Measurable**: Observable criteria to determine completion
- **Achievable**: Within the agent's capability set and available tools
- **Relevant**: Aligned with the user's true intent, not just literal instruction
- **Time-bound**: Defined completion criteria or iteration limits

### Goal Decomposition Structure
```
High-Level Goal (Strategic)
├── Sub-Goal 1 (Tactical)
│   ├── Task 1.1 (Operational)
│   └── Task 1.2 (Operational)
├── Sub-Goal 2 (Tactical)
│   ├── Task 2.1 (Operational)
│   └── Task 2.2 (Operational)
└── Synthesis Goal (Integration)
    └── Validation Task
```

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Establish the goal framework:
1. What is the primary objective? (State in measurable terms)
2. What are the sub-goals that compose the primary goal?
3. How is progress measured at each level? (Metrics, completion criteria)
4. What constitutes success vs. partial success vs. failure?
5. What are the constraints? (Time, resources, iteration limits)

### PLAN
Design the goal management architecture:
1. Create a goal registry with hierarchical structure
2. Define completion criteria for each goal node
3. Design progress tracking: how and when is progress assessed?
4. Plan monitoring checkpoints: when does the agent evaluate goal status?
5. Define adaptation triggers: when should the agent change strategy?

### ACTION
Implement goal-driven agent behavior:
1. Initialize goal state at agent startup
2. Execute tasks while tracking progress toward each sub-goal
3. After each significant action, evaluate goal progress
4. Detect goal drift (moving away from objective) and trigger correction
5. Report completion status when all success criteria are met

## Implementation: Goal State Management

### Goal Registry with ADK Session State
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
from google.adk.sessions import InMemorySessionService
from google.adk.runners import Runner
from google.genai.types import Content, Part
import time

def initialize_goals(
    primary_goal: str,
    sub_goals: list,
    tool_context: ToolContext
) -> dict:
    """Initialize the agent's goal hierarchy for this session.

    Args:
        primary_goal: The main objective to achieve
        sub_goals: List of sub-goal strings that compose the primary goal

    Returns:
        Initialized goal structure
    """
    goal_structure = {
        "primary": {
            "description": primary_goal,
            "status": "in_progress",
            "started_at": time.time(),
            "completed_at": None
        },
        "sub_goals": {
            f"sub_{i}": {
                "description": sg,
                "status": "pending",
                "progress": 0.0  # 0.0 to 1.0
            }
            for i, sg in enumerate(sub_goals)
        },
        "overall_progress": 0.0
    }

    tool_context.state["current_goals"] = goal_structure
    return {"status": "initialized", "goal_count": len(sub_goals) + 1}

def update_goal_progress(
    sub_goal_id: str,
    progress: float,
    status: str,
    tool_context: ToolContext
) -> dict:
    """Update the progress of a specific sub-goal.

    Args:
        sub_goal_id: ID of the sub-goal to update (e.g., 'sub_0', 'sub_1')
        progress: Progress value from 0.0 to 1.0
        status: Current status ('pending', 'in_progress', 'completed', 'failed')

    Returns:
        Updated goal status
    """
    goals = tool_context.state.get("current_goals", {})
    if sub_goal_id not in goals.get("sub_goals", {}):
        return {"error": f"Sub-goal {sub_goal_id} not found"}

    goals["sub_goals"][sub_goal_id]["progress"] = min(1.0, max(0.0, progress))
    goals["sub_goals"][sub_goal_id]["status"] = status

    # Calculate overall progress
    sub_progresses = [sg["progress"] for sg in goals["sub_goals"].values()]
    goals["overall_progress"] = sum(sub_progresses) / len(sub_progresses)

    # Check if all sub-goals complete → mark primary as complete
    all_complete = all(
        sg["status"] == "completed"
        for sg in goals["sub_goals"].values()
    )
    if all_complete:
        goals["primary"]["status"] = "completed"
        goals["primary"]["completed_at"] = time.time()

    tool_context.state["current_goals"] = goals
    return {
        "sub_goal_updated": sub_goal_id,
        "sub_goal_progress": progress,
        "overall_progress": goals["overall_progress"],
        "primary_status": goals["primary"]["status"]
    }

def get_goal_status(tool_context: ToolContext) -> dict:
    """Retrieve the current goal status and progress.

    Returns:
        Complete goal hierarchy with current status and progress
    """
    goals = tool_context.state.get("current_goals", {})
    if not goals:
        return {"message": "No goals initialized for this session"}

    return {
        "primary_goal": goals["primary"]["description"],
        "primary_status": goals["primary"]["status"],
        "overall_progress": f"{goals['overall_progress']:.1%}",
        "sub_goals": [
            {
                "id": k,
                "description": v["description"],
                "status": v["status"],
                "progress": f"{v['progress']:.1%}"
            }
            for k, v in goals.get("sub_goals", {}).items()
        ]
    }

# Goal-driven research agent
goal_agent = LlmAgent(
    name="GoalDrivenResearcher",
    model="gemini-2.5-flash",
    instruction="""You are a goal-driven research agent that tracks your own progress.

    When given a research task:
    1. Use initialize_goals to set up your goal hierarchy
    2. Break the primary goal into 3-5 concrete sub-goals
    3. As you complete each sub-goal, call update_goal_progress
    4. Periodically check get_goal_status to ensure you're on track
    5. Only consider the task complete when all sub-goals reach 'completed' status

    Always be explicit about which goal you are currently pursuing.""",
    tools=[
        FunctionTool(initialize_goals),
        FunctionTool(update_goal_progress),
        FunctionTool(get_goal_status)
    ]
)
```

### Goal-Directed Agent with LangGraph
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List, Optional
from google import genai

class GoalState(TypedDict):
    primary_goal: str
    sub_goals: List[str]
    completed_sub_goals: List[str]
    current_sub_goal: Optional[str]
    goal_progress: float
    outputs: List[str]
    final_output: str
    iteration: int
    max_iterations: int

client = genai.Client()

def decompose_goals_node(state: GoalState) -> dict:
    """Decompose primary goal into concrete sub-goals."""
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=
        f"""Primary goal: {state['primary_goal']}

        Decompose this into 4-5 specific, measurable sub-goals.
        Each sub-goal should be independently verifiable.
        Output as a numbered list, one sub-goal per line."""
    )

    # Parse sub-goals from response
    lines = response.text.strip().split('\n')
    sub_goals = [
        line.lstrip('0123456789. ').strip()
        for line in lines
        if line.strip() and line[0].isdigit()
    ]

    return {
        "sub_goals": sub_goals,
        "current_sub_goal": sub_goals[0] if sub_goals else None,
        "goal_progress": 0.0,
        "iteration": 0
    }

def execute_sub_goal_node(state: GoalState) -> dict:
    """Execute the current sub-goal."""
    if not state.get("current_sub_goal"):
        return {}

    completed = state.get("completed_sub_goals", [])
    context = "\n".join([f"✓ {sg}" for sg in completed]) if completed else "None yet"

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""Primary goal: {state['primary_goal']}

        Already completed:
        {context}

        Now execute this sub-goal: {state['current_sub_goal']}

        Provide a thorough, concrete result for this sub-goal."""
    )

    outputs = state.get("outputs", []) + [
        f"[{state['current_sub_goal']}]\n{response.text}"
    ]
    completed_goals = completed + [state["current_sub_goal"]]
    progress = len(completed_goals) / len(state["sub_goals"])

    # Determine next sub-goal
    remaining = [sg for sg in state["sub_goals"] if sg not in completed_goals]
    next_goal = remaining[0] if remaining else None

    return {
        "outputs": outputs,
        "completed_sub_goals": completed_goals,
        "current_sub_goal": next_goal,
        "goal_progress": progress,
        "iteration": state.get("iteration", 0) + 1
    }

def monitor_progress_node(state: GoalState) -> dict:
    """Monitor overall progress and determine if goals are met."""
    completed = state.get("completed_sub_goals", [])
    total = len(state.get("sub_goals", []))

    if total == 0:
        return {"goal_progress": 0.0}

    progress = len(completed) / total
    return {"goal_progress": progress}

def synthesize_node(state: GoalState) -> dict:
    """Synthesize all sub-goal outputs into final deliverable."""
    all_outputs = "\n\n".join(state.get("outputs", []))

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""Primary goal: {state['primary_goal']}

        Results from all sub-goals:
        {all_outputs}

        Synthesize these results into a comprehensive final response that
        fully addresses the primary goal. Ensure the output is cohesive
        and complete."""
    )

    return {"final_output": response.text}

def should_continue_executing(state: GoalState) -> str:
    """Route based on whether more sub-goals remain."""
    if state.get("iteration", 0) >= state.get("max_iterations", 10):
        return "synthesize"  # Safety: avoid infinite loops
    if state.get("current_sub_goal"):
        return "execute"
    return "synthesize"

# Build goal-directed graph
graph = StateGraph(GoalState)
graph.add_node("decompose", decompose_goals_node)
graph.add_node("execute", execute_sub_goal_node)
graph.add_node("monitor", monitor_progress_node)
graph.add_node("synthesize", synthesize_node)

graph.set_entry_point("decompose")
graph.add_edge("decompose", "execute")
graph.add_edge("execute", "monitor")
graph.add_conditional_edges(
    "monitor",
    should_continue_executing,
    {"execute": "execute", "synthesize": "synthesize"}
)
graph.add_edge("synthesize", END)

goal_system = graph.compile()

result = goal_system.invoke({
    "primary_goal": "Analyze the competitive landscape for AI coding assistants in 2025",
    "sub_goals": [],
    "completed_sub_goals": [],
    "current_sub_goal": None,
    "goal_progress": 0.0,
    "outputs": [],
    "final_output": "",
    "iteration": 0,
    "max_iterations": 10
})

print(f"Progress: {result['goal_progress']:.1%}")
print(result["final_output"])
```

## Implementation: Goal Monitoring and Alerting

### Continuous Goal Monitoring
```python
import time
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class Goal:
    description: str
    success_criteria: str
    priority: int  # 1 (highest) to 5 (lowest)
    status: str = "pending"  # pending/in_progress/completed/failed
    progress: float = 0.0
    started_at: Optional[float] = None
    completed_at: Optional[float] = None

@dataclass
class GoalMonitor:
    """Monitors agent goals and triggers interventions when drift detected."""
    goals: List[Goal] = field(default_factory=list)
    drift_threshold: float = 0.1  # Alert if progress drops by this much
    _last_progress_snapshot: dict = field(default_factory=dict)

    def add_goal(self, description: str, criteria: str, priority: int = 3) -> int:
        """Add a new goal and return its index."""
        goal = Goal(description=description, success_criteria=criteria, priority=priority)
        self.goals.append(goal)
        return len(self.goals) - 1

    def update_progress(self, goal_index: int, progress: float) -> dict:
        """Update goal progress and check for drift."""
        if goal_index >= len(self.goals):
            return {"error": "Goal index out of range"}

        goal = self.goals[goal_index]
        old_progress = goal.progress
        goal.progress = progress

        if goal.status == "pending" and progress > 0:
            goal.status = "in_progress"
            goal.started_at = time.time()

        if progress >= 1.0:
            goal.status = "completed"
            goal.completed_at = time.time()

        # Detect drift (progress regression)
        drift_detected = (old_progress - progress) > self.drift_threshold

        return {
            "goal": goal.description,
            "progress": f"{progress:.1%}",
            "status": goal.status,
            "drift_detected": drift_detected,
            "alert": f"DRIFT ALERT: Progress dropped {old_progress - progress:.1%}" if drift_detected else None
        }

    def get_summary(self) -> dict:
        """Get overall goal achievement summary."""
        total = len(self.goals)
        completed = sum(1 for g in self.goals if g.status == "completed")
        failed = sum(1 for g in self.goals if g.status == "failed")
        avg_progress = sum(g.progress for g in self.goals) / total if total > 0 else 0

        critical_goals = [g for g in self.goals if g.priority == 1 and g.status != "completed"]

        return {
            "total_goals": total,
            "completed": completed,
            "failed": failed,
            "avg_progress": f"{avg_progress:.1%}",
            "completion_rate": f"{completed/total:.1%}" if total > 0 else "0%",
            "blocked_critical_goals": [g.description for g in critical_goals]
        }

# Usage
monitor = GoalMonitor()
monitor.add_goal("Gather market data", "At least 5 data sources analyzed", priority=1)
monitor.add_goal("Analyze competitors", "Top 3 competitors profiled", priority=2)
monitor.add_goal("Write report", "1500+ word structured report", priority=3)

# Simulate progress updates
monitor.update_progress(0, 0.5)
monitor.update_progress(0, 1.0)
monitor.update_progress(1, 0.33)

print(monitor.get_summary())
```

## Goal Patterns

### Competing Goals Resolution
```python
def resolve_competing_goals(goals: list, client) -> str:
    """Resolve conflicts between competing goals using LLM judgment."""
    goal_list = "\n".join([f"{i+1}. (Priority {g['priority']}) {g['description']}"
                           for i, g in enumerate(goals)])

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""You have these competing goals to balance:
{goal_list}

Determine the optimal execution order considering:
- Priority levels (lower number = higher priority)
- Dependencies between goals
- Resource conflicts
- Time constraints

Output a recommended execution plan with rationale."""
    )
    return response.text
```

## Key Takeaways

- **Goals need criteria**: A goal without measurable success criteria cannot be evaluated — always define what "done" means
- **Hierarchy matters**: Break complex goals into sub-goals; completion of sub-goals drives primary goal progress
- **Monitor continuously**: Don't just set goals and forget — check progress after each significant action
- **Detect drift early**: If progress is regressing, trigger intervention before the agent goes too far off track
- **Iteration limits**: Always set maximum iteration counts as a safety valve against infinite goal pursuit
- **Priority resolution**: When goals compete, explicit priority levels enable principled trade-off decisions

## Anti-Patterns to Avoid

- **Vague goals**: "Do a good job" is not a goal — "Produce a 1000-word analysis covering X, Y, Z" is a goal
- **No completion criteria**: Goals without exit conditions cause agents to loop indefinitely
- **Ignoring sub-goal failures**: A failed sub-goal should trigger replanning, not silent continuation
- **Goal obsession**: Agents that rigidly pursue goals despite clear impossibility waste resources — know when to escalate or abort
- **Unmonitored long-running tasks**: Tasks running for hours without progress checks risk silent failure

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Autonomous goal-driven agent | Goal Setting + **Planning** + **Tool Use** |
| Adaptive goal management | Goal Setting + **Learning & Adaptation** + **Evaluation** |
| Priority-based goal execution | Goal Setting + **Prioritization** + **Resource-Aware** |
| Long-term stateful agent | Goal Setting + **Memory Management** + **Reflection** |

## References

- Goal-Oriented Agent Design: https://google.github.io/adk-docs/agents/
- LangGraph Workflow Design: https://langchain-ai.github.io/langgraph/concepts/
- OKR Framework for AI: https://ai.google.dev/gemini-api/docs/
- AutoGPT Goal Framework: https://github.com/Significant-Gravitas/AutoGPT
