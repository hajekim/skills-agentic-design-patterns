---
name: prioritization
description: This skill should be used when the user wants to "agent task prioritization", "priority queue for agents", "dynamic task scheduling", "urgent task handling", "deadline-aware agents", "agent workload management", "task importance scoring", "weighted task execution", "interrupt-driven agents", "preemptive scheduling", "task queue management", "priority-based execution", "agent scheduler", "SLA-aware agent", "triage tasks for agents", or build agents that intelligently order and schedule their work based on priority, urgency, and importance. Also responds to Korean: "에이전트 작업 우선순위", "동적 작업 스케줄링", "긴급 작업 처리", "마감 기한 인식 에이전트", "에이전트 워크로드 관리", "중요도 기반 실행", "우선순위 설정", "긴급 작업 우선", "중요한 것 먼저 처리해줘", "작업 순서 정해줘", "스케줄러 에이전트 만들어줘". Also responds to Japanese: "タスクの優先順位付け", "動的タスクスケジューリング", "緊急タスク処理", "締め切り対応エージェント", "重要なものから先に処理してほしい", "タスクキュー管理", "スケジューラーエージェント", "作業の順番を決めたい", "優先度ベースの実行", "ワークロード管理" Also responds to Chinese: "任务优先级排序", "动态任务调度", "紧急任务处理", "截止日期感知智能体", "重要的先处理", "任务队列管理", "调度器智能体", "安排任务顺序", "基于优先级的执行", "工作负载管理".. Apply this skill to design or implement the Prioritization agentic design pattern.
version: 1.0.0
---

# Prioritization Pattern

## Overview

The **Prioritization Pattern** equips agents with the ability to intelligently order and schedule their work — managing multiple competing tasks by continuously evaluating their relative importance, urgency, and expected impact. Rather than executing tasks in arbitrary order, prioritizing agents allocate their limited attention and resources to what matters most.

**Core Principle:** Not all tasks are equal — do the right things in the right order, not just things in the order they arrived.

## When This Skill Applies

Activate this pattern when:
- Agents receive multiple concurrent requests that cannot all be handled immediately
- Some tasks are time-sensitive (deadlines, SLAs, real-time alerts) while others can wait
- Resources are limited and must be allocated to highest-value tasks first
- Different users or workloads have different priority levels
- Long-running tasks must be interruptible when higher-priority work arrives
- The agent must balance urgent-but-less-important vs. important-but-not-urgent tasks

**Rule of thumb:** If the agent has more to do than time to do it — it needs prioritization.

## Priority Frameworks

### Eisenhower Matrix (Urgency × Importance)
```
                │  IMPORTANT  │ NOT IMPORTANT │
────────────────┼─────────────┼───────────────┤
   URGENT       │ Do First    │ Delegate/Auto │
────────────────┼─────────────┼───────────────┤
   NOT URGENT   │ Schedule    │ Eliminate     │
────────────────┴─────────────┴───────────────┘
```

### Multi-Factor Priority Scoring
```
Priority = (Urgency × 0.4) + (Impact × 0.3) + (Effort × 0.2) + (Dependencies × 0.1)
```

### WSJF (Weighted Shortest Job First)
```
WSJF = (Business Value + Time Criticality + Risk Reduction) / Job Size
```
Best for software/product backlogs — prioritizes small, high-value tasks.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map prioritization requirements:
1. What task types does the agent handle? (Categorize by domain)
2. What dimensions determine priority? (Urgency, impact, effort, dependencies)
3. What are the SLAs per task type? (Response time requirements)
4. How dynamic is priority? (Static assignment vs. real-time re-evaluation)
5. Can tasks be preempted? (Can high-priority work interrupt in-progress tasks?)

### PLAN
Design the prioritization architecture:
1. Define the priority scoring function (weights for each dimension)
2. Design the task queue: FIFO, priority heap, or weighted fair queuing
3. Plan re-evaluation triggers: when do priorities get recalculated?
4. Define preemption policy: when and how to interrupt lower-priority tasks
5. Plan deadline-miss handling: what happens when SLAs are violated?

### ACTION
Implement priority-aware agent:
1. Build a priority queue with dynamic scoring
2. Assign initial priority scores when tasks arrive
3. Re-evaluate priorities periodically and on new task arrival
4. Execute from highest priority, preempting if needed
5. Track and report on priority-based metrics (SLA compliance, wait times)

## Implementation: Priority Queue Agent

### Dynamic Priority Task Queue
```python
import heapq
import time
import uuid
from dataclasses import dataclass, field
from typing import Optional, Callable
from google import genai

@dataclass
class PrioritizedTask:
    """A task with dynamic priority scoring."""
    task_id: str
    description: str
    task_type: str           # "urgent_alert", "user_request", "background", "maintenance"
    deadline: Optional[float]  # Unix timestamp, None if no deadline
    impact: float            # 0.0 to 1.0 (business impact)
    created_at: float = field(default_factory=time.time)
    context: dict = field(default_factory=dict)

    # These are computed, not passed in
    priority_score: float = 0.0
    status: str = "pending"  # pending/in_progress/completed/failed

    def compute_priority(self) -> float:
        """Compute dynamic priority score (higher = more urgent)."""
        now = time.time()

        # Time urgency component
        if self.deadline:
            time_to_deadline = self.deadline - now
            if time_to_deadline <= 0:
                urgency = 1.0  # Overdue — maximum urgency
            elif time_to_deadline < 60:   # Less than 1 minute
                urgency = 0.9
            elif time_to_deadline < 300:  # Less than 5 minutes
                urgency = 0.7
            elif time_to_deadline < 3600: # Less than 1 hour
                urgency = 0.4
            else:
                urgency = 0.1
        else:
            urgency = 0.2  # No deadline = low time urgency

        # Age component (older tasks get slight priority boost)
        age_hours = (now - self.created_at) / 3600
        age_bonus = min(0.2, age_hours * 0.02)  # Up to 0.2 bonus over 10 hours

        # Task type base priority
        type_priority = {
            "urgent_alert": 0.9,
            "user_request": 0.6,
            "background": 0.3,
            "maintenance": 0.1
        }.get(self.task_type, 0.4)

        # Combined score
        score = (urgency * 0.4) + (self.impact * 0.3) + (type_priority * 0.2) + (age_bonus * 0.1)
        self.priority_score = score
        return score

    def __lt__(self, other):
        """Higher priority score = lower heap value (max-heap via negation)."""
        return self.compute_priority() > other.compute_priority()


class PriorityTaskQueue:
    """Priority-ordered task queue with dynamic re-evaluation."""

    def __init__(self):
        self._heap: list = []
        self._tasks: dict = {}  # task_id → PrioritizedTask

    def add_task(self, task: PrioritizedTask):
        """Add a task to the priority queue."""
        task.compute_priority()
        heapq.heappush(self._heap, task)
        self._tasks[task.task_id] = task

    def get_next_task(self) -> Optional[PrioritizedTask]:
        """Get the highest-priority pending task."""
        # Re-evaluate all priorities before selecting
        for task in self._tasks.values():
            task.compute_priority()

        # Rebuild heap after re-evaluation
        self._heap = [t for t in self._tasks.values() if t.status == "pending"]
        heapq.heapify(self._heap)

        while self._heap:
            task = heapq.heappop(self._heap)
            if task.status == "pending":
                return task
        return None

    def get_queue_summary(self) -> dict:
        """Get a summary of the current task queue."""
        tasks = list(self._tasks.values())
        pending = [t for t in tasks if t.status == "pending"]
        in_progress = [t for t in tasks if t.status == "in_progress"]

        # Sort pending by priority
        pending.sort(key=lambda t: t.compute_priority(), reverse=True)

        return {
            "total_tasks": len(tasks),
            "pending": len(pending),
            "in_progress": len(in_progress),
            "completed": len([t for t in tasks if t.status == "completed"]),
            "next_3_tasks": [
                {
                    "id": t.task_id,
                    "description": t.description[:60],
                    "priority": f"{t.priority_score:.2f}",
                    "type": t.task_type
                }
                for t in pending[:3]
            ]
        }
```

### LLM-Based Priority Scoring
```python
def llm_prioritize_tasks(tasks: list, client, context: str = "") -> list:
    """Use LLM to intelligently score and rank tasks.

    Args:
        tasks: List of task description strings
        client: genai.Client instance to use for scoring
        context: Additional context about current state/constraints

    Returns:
        Tasks sorted by LLM-assigned priority (highest first)
    """
    task_list = "\n".join([f"{i+1}. {task}" for i, task in enumerate(tasks)])

    priority_prompt = f"""You are an expert task prioritizer. Rank these tasks by priority.

Current context: {context if context else "No specific context"}

Tasks to prioritize:
{task_list}

For each task, assign:
- Priority score: 1-10 (10 = most urgent/important)
- Rationale: Brief explanation

Output as a ranked list, highest priority first:
[Score] Task N: [task description] — [rationale]"""

    response = client.models.generate_content(model='gemini-2.5-flash', contents=priority_prompt)

    # Parse rankings (simplified — production should use structured output)
    import re
    ranked = []
    for line in response.text.split('\n'):
        match = re.match(r'\[(\d+(?:\.\d+)?)\]\s+Task\s+(\d+)', line)
        if match:
            score = float(match.group(1))
            task_idx = int(match.group(2)) - 1
            if 0 <= task_idx < len(tasks):
                ranked.append({
                    "task": tasks[task_idx],
                    "priority_score": score,
                    "rationale": line[match.end():].strip()
                })

    # Sort by score descending
    ranked.sort(key=lambda x: x["priority_score"], reverse=True)
    return ranked

# Usage
client = genai.Client()
tasks = [
    "Update user documentation for new API endpoints",
    "Fix critical bug causing login failures for 10% of users",
    "Review and merge developer PRs",
    "Respond to enterprise customer support ticket about data export",
    "Run weekly backup verification tests"
]

ranked_tasks = llm_prioritize_tasks(tasks, client, context="Production incident in progress")
for task in ranked_tasks:
    print(f"[{task['priority_score']:.0f}] {task['task']}")
```

### Priority-Aware ADK Agent
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
import time

# Global task queue
task_queue = PriorityTaskQueue()

def add_task_to_queue(
    description: str,
    task_type: str,
    impact: float,
    deadline_minutes: Optional[float],
    tool_context: ToolContext
) -> dict:
    """Add a new task to the priority queue.

    Args:
        description: What needs to be done
        task_type: Category (urgent_alert/user_request/background/maintenance)
        impact: Business impact from 0.0 to 1.0
        deadline_minutes: Minutes until deadline, or None for no deadline

    Returns:
        Task ID and initial priority score
    """
    deadline = time.time() + (deadline_minutes * 60) if deadline_minutes else None

    task = PrioritizedTask(
        task_id=str(uuid.uuid4())[:8],
        description=description,
        task_type=task_type,
        deadline=deadline,
        impact=impact
    )
    task_queue.add_task(task)

    return {
        "task_id": task.task_id,
        "priority_score": task.priority_score,
        "queue_position": len([t for t in task_queue._tasks.values() if t.status == "pending"])
    }

def get_next_priority_task(tool_context: ToolContext) -> dict:
    """Get the highest-priority task to work on next.

    Returns:
        Next task details or empty queue indicator
    """
    task = task_queue.get_next_task()
    if not task:
        return {"status": "queue_empty", "message": "No pending tasks"}

    task.status = "in_progress"
    return {
        "task_id": task.task_id,
        "description": task.description,
        "task_type": task.task_type,
        "priority_score": f"{task.priority_score:.2f}",
        "deadline": task.deadline,
        "context": task.context
    }

def complete_task(task_id: str, result: str, tool_context: ToolContext) -> dict:
    """Mark a task as completed.

    Args:
        task_id: ID of the completed task
        result: Summary of what was accomplished

    Returns:
        Completion confirmation and next queue summary
    """
    if task_id in task_queue._tasks:
        task_queue._tasks[task_id].status = "completed"

    summary = task_queue.get_queue_summary()
    return {
        "completed_task_id": task_id,
        "result": result,
        "queue_summary": summary
    }

priority_agent = LlmAgent(
    name="PrioritizationAgent",
    model="gemini-2.5-flash",
    instruction="""You are a priority-driven agent that manages a task queue intelligently.

    Your workflow:
    1. When new tasks arrive, use add_task_to_queue with appropriate priority attributes
    2. Always call get_next_priority_task to determine what to work on next
    3. Work on the highest-priority task first
    4. When done with a task, call complete_task with the result
    5. Never work on a lower-priority task while higher-priority tasks are pending

    When assessing task impact: 0.9+ = critical, 0.7 = high, 0.5 = medium, 0.3 = low
    When assessing urgency: use deadline_minutes to reflect time sensitivity""",
    tools=[
        FunctionTool(add_task_to_queue),
        FunctionTool(get_next_priority_task),
        FunctionTool(complete_task)
    ]
)
```

## Key Takeaways

- **Priority ≠ arrival order**: FIFO queues waste high-priority work on low-value tasks
- **Dynamic re-evaluation**: Priorities change as deadlines approach — re-score continuously, not just at arrival
- **Multiple dimensions**: Urgency + importance + effort + dependencies = better decisions than single-dimension priority
- **Age bonus prevents starvation**: Low-priority tasks that age in the queue should eventually get attention
- **LLM for complex prioritization**: Use LLM judgment when domain-specific context makes rule-based scoring insufficient
- **Preemption threshold**: Define when it's worth interrupting in-progress work for higher-priority tasks

## Anti-Patterns to Avoid

- **Priority inversion**: Low-priority tasks blocking high-priority ones through shared resources
- **Starvation**: Always-high-priority work that prevents low-priority tasks from ever completing
- **Static priorities**: Priorities set at creation time and never updated ignore deadline drift
- **Too many priority levels**: 100 priority levels → essentially FIFO with extra complexity; 3-5 levels are sufficient
- **Ignoring effort**: Prioritizing by importance alone ignores that small-effort tasks might have better ROI

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Intelligent task scheduling | Prioritization + **Goal Setting** + **Planning** |
| Budget-conscious execution order | Prioritization + **Resource-Aware** |
| Deadline-aware multi-agent system | Prioritization + **Multi-Agent Collaboration** + **Parallelization** |
| Adaptive priority with human oversight | Prioritization + **Human-in-the-Loop** |

## References

- Eisenhower Matrix: https://en.wikipedia.org/wiki/Time_management#The_Eisenhower_Method
- WSJF (Weighted Shortest Job First): https://scaledagileframework.com/wsjf/
- Priority Queue in Python (heapq): https://docs.python.org/3/library/heapq.html
- Google ADK Task Management: https://google.github.io/adk-docs/
