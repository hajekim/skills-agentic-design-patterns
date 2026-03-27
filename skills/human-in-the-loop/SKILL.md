---
name: human-in-the-loop
description: This skill should be used when the user wants to "human approval for agents", "agent oversight", "human-in-the-loop", "HITL", "human intervention", "agent confirmation", "interrupt agent for review", "human feedback during execution", "agent escalation", "require human approval", "agent checkpoints", "pause agent for human input", "human approval workflow", "supervised agent", "confirm before action", "manual intervention point", "require confirmation", "human review step", or build agents that involve humans at critical decision points or for oversight and approval. Also responds to Korean: "인간 승인 에이전트", "에이전트 감독", "휴먼 인 더 루프", "실행 중 인간 개입", "에이전트 체크포인트", "인간 피드백 루프", "사람 개입", "인간 검토", "사람이 확인하는 에이전트", "사람이 확인 후 실행해줘", "중요한 건 나한테 물어봐", "승인 후 진행해줘". Also responds to Japanese: "人間の承認フロー", "エージェントの監督", "ヒューマンインザループ", "実行前に確認してほしい", "人間のチェックポイント", "重要なことは私に聞いて", "承認後に実行", "人間のフィードバックループ", "監視付きエージェント", "確認してから進めてほしい" Also responds to Chinese: "人工审批流程", "智能体监督", "人在回路中", "执行前请确认", "人工检查节点", "重要的事先问我", "审批后执行", "人工反馈循环", "受监督的智能体", "确认后再继续".. Apply this skill to design or implement the Human-in-the-Loop (HITL) agentic design pattern.
version: 1.0.0
---

# Human-in-the-Loop (HITL) Pattern

## Overview

The **Human-in-the-Loop (HITL) Pattern** integrates human judgment, oversight, and approval into agent workflows at critical decision points. Rather than running fully autonomously, HITL agents pause at defined checkpoints — to confirm high-stakes actions, request clarification, seek approval, or collect feedback — before proceeding.

**Core Principle:** Autonomy is earned, not assumed — build human oversight into the design, not as an afterthought.

## When This Skill Applies

Activate this pattern when:
- Agent actions are irreversible (sending emails, deleting data, financial transactions)
- High-risk decisions require human judgment (medical, legal, financial advice)
- Ambiguous instructions need human clarification before proceeding
- Regulatory or compliance requirements mandate human approval
- Agent confidence is below acceptable threshold for autonomous action
- Actions affect third parties who have not consented to fully automated interaction
- The agent encounters situations outside its training distribution

**Rule of thumb:** If you'd want a human to review the action before it happens, build in a HITL checkpoint.

## HITL Spectrum

```
Fully Autonomous ←————————————————————→ Fully Manual
       |           |           |           |
   Auto-run    Approve     Review      Step-by-step
  (no review)  high-risk   all steps    approval
```

Choose the appropriate level based on:
- **Risk**: Higher risk → more human involvement
- **Trust**: Proven, well-tested agents can operate more autonomously
- **Context**: Production vs. development, internal vs. external-facing
- **Reversibility**: Irreversible actions need pre-approval

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the human oversight requirements:
1. Which actions are irreversible or high-risk?
2. Which decision points require human judgment (not just execution)?
3. What information does the human need to make an informed approval?
4. What are the response time requirements? (Synchronous vs. async approval)
5. What happens if the human is unavailable?

### PLAN
Design HITL checkpoints:
1. Identify specific nodes in the workflow where humans must engage
2. Design the approval request: clear context, proposed action, options (approve/reject/modify)
3. Choose synchronous (block until approved) vs. asynchronous (queue for review)
4. Define timeout behavior: auto-approve, auto-reject, or escalate to supervisor
5. Plan the feedback loop: how does human feedback improve future agent decisions?

### ACTION
Implement human checkpoints:
1. Insert approval gates at identified high-risk decision points
2. Build approval interface: clear, concise action summaries with consequences
3. Handle approval outcomes: proceed, abort, modify and retry
4. Log all human interventions for audit trails
5. Learn from rejections to improve future decisions

## Implementation: ADK Human-in-the-Loop

### Approval Gate Tool
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
import time

def request_human_approval(
    action_description: str,
    action_details: str,
    risk_level: str,
    tool_context: ToolContext
) -> dict:
    """Request human approval before executing a high-risk action.

    Args:
        action_description: Brief description of the proposed action
        action_details: Full details including parameters and expected impact
        risk_level: Risk level - 'low', 'medium', 'high', 'critical'

    Returns:
        Approval status with human response
    """
    # Store pending approval in state
    approval_request = {
        "id": f"approval_{int(time.time())}",
        "action": action_description,
        "details": action_details,
        "risk_level": risk_level,
        "requested_at": time.time(),
        "status": "pending"
    }
    tool_context.state["pending_approval"] = approval_request

    # In production: send to Slack, email, web UI, etc.
    # Here we simulate with console input
    print(f"\n{'='*60}")
    print(f"APPROVAL REQUIRED [{risk_level.upper()}]")
    print(f"{'='*60}")
    print(f"Action: {action_description}")
    print(f"Details: {action_details}")
    print(f"{'='*60}")
    human_response = input("Approve? [y/n/m=modify]: ").strip().lower()
    modification = None
    if human_response == 'm':
        modification = input("Provide modification instructions: ").strip()

    result = {
        "approved": human_response == 'y',
        "rejected": human_response == 'n',
        "modification_requested": human_response == 'm',
        "modification_instructions": modification,
        "reviewer": "human_operator",
        "reviewed_at": time.time()
    }

    # Update state with decision
    approval_request["status"] = "approved" if result["approved"] else "rejected"
    approval_request["result"] = result
    tool_context.state["pending_approval"] = approval_request

    return result

def execute_approved_action(
    action_type: str,
    parameters: dict,
    tool_context: ToolContext
) -> dict:
    """Execute an action only if it was previously approved.

    Args:
        action_type: Type of action to execute
        parameters: Action parameters

    Returns:
        Execution result or rejection
    """
    pending = tool_context.state.get("pending_approval", {})

    if pending.get("status") != "approved":
        return {
            "status": "blocked",
            "reason": "Action requires human approval before execution"
        }

    # Clear the approval after use (prevent replay)
    tool_context.state["pending_approval"] = {}

    # Execute the action
    if action_type == "send_email":
        return {"status": "sent", "to": parameters.get("to"), "subject": parameters.get("subject")}
    elif action_type == "delete_record":
        return {"status": "deleted", "record_id": parameters.get("record_id")}
    else:
        return {"status": "executed", "action": action_type, "parameters": parameters}

# HITL agent that requires approval for high-risk actions
hitl_agent = LlmAgent(
    name="ApprovalGatedAgent",
    model="gemini-2.5-flash",
    instruction="""You are an agent that requires human approval before executing
    any high-risk or irreversible actions.

    Before executing ANY action that:
    - Sends messages to external parties
    - Deletes or modifies data
    - Makes financial transactions
    - Accesses sensitive information

    You MUST call request_human_approval first. Only proceed with
    execute_approved_action if approval is granted.

    If approval is rejected, stop and explain to the user what happened.
    If modification is requested, adjust the action accordingly and seek approval again.""",
    tools=[
        FunctionTool(request_human_approval),
        FunctionTool(execute_approved_action)
    ]
)
```

### Async Approval with LangGraph
```python
from langgraph.graph import StateGraph, END, interrupt
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Optional, Literal

class HITLState(TypedDict):
    task: str
    proposed_action: Optional[str]
    proposed_details: Optional[str]
    approval_status: Optional[Literal["pending", "approved", "rejected", "modified"]]
    human_feedback: Optional[str]
    result: Optional[str]

def plan_action_node(state: HITLState) -> dict:
    """Agent plans what action to take."""
    from google import genai
    client = genai.Client()

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""Task: {state['task']}

        Plan the specific action to take. Describe:
        1. The exact action you propose
        2. The parameters/details of the action
        3. The expected impact

        Format as JSON: {{"action": "...", "details": "...", "impact": "..."}}"""
    )

    import json, re
    json_match = re.search(r'\{[\s\S]+\}', response.text)
    if json_match:
        try:
            plan = json.loads(json_match.group())
            return {
                "proposed_action": plan.get("action", ""),
                "proposed_details": plan.get("details", ""),
                "approval_status": "pending"
            }
        except json.JSONDecodeError:
            pass

    return {
        "proposed_action": response.text[:200],
        "proposed_details": response.text,
        "approval_status": "pending"
    }

def human_review_node(state: HITLState) -> dict:
    """INTERRUPT: Pause execution for human review."""
    # This interrupt() call pauses the graph and returns control to the caller
    # The caller can resume with human feedback via graph.update_state()
    human_input = interrupt({
        "proposed_action": state["proposed_action"],
        "proposed_details": state["proposed_details"],
        "message": "Review and approve the proposed action.",
        "options": ["approve", "reject", "modify"]
    })

    return {
        "approval_status": human_input.get("decision", "rejected"),
        "human_feedback": human_input.get("feedback", "")
    }

def execute_action_node(state: HITLState) -> dict:
    """Execute the approved action."""
    from google import genai
    client = genai.Client()

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"""Execute this approved action:
        Action: {state['proposed_action']}
        Details: {state['proposed_details']}

        Describe the successful execution result."""
    )
    return {"result": response.text}

def handle_rejection_node(state: HITLState) -> dict:
    """Handle human rejection of proposed action."""
    return {
        "result": f"Action rejected by human reviewer. "
                  f"Feedback: {state.get('human_feedback', 'No feedback provided')}. "
                  f"Task cancelled."
    }

def route_after_review(state: HITLState) -> str:
    status = state.get("approval_status", "rejected")
    if status == "approved":
        return "execute"
    elif status == "modify":
        return "plan"  # Re-plan with human feedback
    return "rejected"

# Build HITL graph with checkpointing (required for interrupt)
checkpointer = MemorySaver()
graph = StateGraph(HITLState)
graph.add_node("plan", plan_action_node)
graph.add_node("review", human_review_node)
graph.add_node("execute", execute_action_node)
graph.add_node("rejected", handle_rejection_node)

graph.set_entry_point("plan")
graph.add_edge("plan", "review")
graph.add_conditional_edges("review", route_after_review, {
    "execute": "execute",
    "plan": "plan",
    "rejected": "rejected"
})
graph.add_edge("execute", END)
graph.add_edge("rejected", END)

hitl_app = graph.compile(checkpointer=checkpointer, interrupt_before=["review"])

# Usage: Start the graph
config = {"configurable": {"thread_id": "task-001"}}
state = hitl_app.invoke(
    {"task": "Send quarterly report to all customers via email"},
    config=config
)

# At this point, graph is paused at "review" — human can inspect state["proposed_action"]
# Human decision:
hitl_app.update_state(config, {"approval_status": "approved", "human_feedback": ""})

# Resume execution
final_state = hitl_app.invoke(None, config=config)
print(final_state["result"])
```

## HITL Patterns

### Confidence-Gated Execution
```python
def confidence_gated_action(action: str, confidence: float, threshold: float = 0.8) -> dict:
    """Execute autonomously if confidence is high; require approval if low."""
    if confidence >= threshold:
        return {"mode": "autonomous", "action": action, "confidence": confidence}
    else:
        # Request human review
        print(f"Low confidence ({confidence:.1%}) — requesting human review")
        return request_human_approval(action, f"Confidence: {confidence:.1%}", "medium")
```

### Escalation Chain
```python
ESCALATION_CHAIN = ["agent_itself", "team_lead", "manager", "system_admin"]

def escalate_decision(issue: str, current_level: int = 0) -> dict:
    """Escalate decisions up the authority chain."""
    if current_level >= len(ESCALATION_CHAIN):
        return {"status": "unresolved", "issue": issue}

    reviewer = ESCALATION_CHAIN[current_level]
    # Notify reviewer (email, Slack, etc.)
    return {
        "escalated_to": reviewer,
        "issue": issue,
        "level": current_level
    }
```

## Key Takeaways

- **Risk-tiered oversight**: Match HITL intensity to action risk — not every action needs approval
- **Clear approval requests**: Humans need full context (action, details, impact) to make good decisions
- **Async over sync**: For non-urgent approvals, async workflows (queue to Slack/email) are more scalable
- **LangGraph interrupt()**: Built-in mechanism to pause graph execution for human review
- **Audit trails**: Log all human decisions (who approved what, when, with what feedback)
- **Timeout handling**: Define fallback behavior when human is unavailable — don't hang indefinitely

## Anti-Patterns to Avoid

- **Approval fatigue**: Too many approval requests trains humans to click "approve" without reading
- **Missing context**: Approval requests without sufficient detail lead to uninformed approvals
- **No feedback loop**: Human rejections should improve future agent decisions — capture why
- **Sync blocking production**: Blocking production pipelines waiting for human approval creates bottlenecks
- **Post-hoc review only**: Review after irreversible actions is notification, not oversight

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| High-stakes agent with safety gates | Human-in-the-Loop + **Guardrails** + **Evaluation** |
| Graceful failure escalation | Human-in-the-Loop + **Exception Handling** |
| Collaborative planning agent | Human-in-the-Loop + **Planning** + **Goal Setting** |
| Supervised multi-agent workflow | Human-in-the-Loop + **Multi-Agent Collaboration** |

## References

- LangGraph Human-in-the-Loop: https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/
- Google ADK Human-in-the-Loop: https://google.github.io/adk-docs/human-in-the-loop/
- AI Safety via HITL: https://hai.stanford.edu/news/human-loop-deep-learning
- LangGraph interrupt() API: https://langchain-ai.github.io/langgraph/reference/types/
