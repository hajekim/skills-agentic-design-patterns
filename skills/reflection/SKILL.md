---
name: reflection
description: This skill should be used when the user wants to "self-evaluate output", "iterative refinement", "LLM as critic", "self-correction loop", "quality improvement loop", "review and revise", "agent self-assessment", "critique and improve", "self-improving output", "feedback loop agent", "generate then critique", "refine agent output", "improve response quality", "agent quality loop", or build a system where an agent evaluates and improves its own outputs. Also responds to Korean: "자기 평가 루프", "반복적 출력 개선", "LLM 비평가 패턴", "자기 교정 루프", "출력 검토 및 수정", "비판 및 개선", "자기 검토 에이전트", "피드백 루프", "자기 수정", "스스로 검토해줘", "결과 개선해줘", "다시 검토하고 고쳐줘", "출력 퀄리티 높여줘". Also responds to Japanese: "自己評価ループ", "反復的改善", "LLM批評家パターン", "自己修正ループ", "出力品質向上", "批判と改善", "やり直しエージェント", "出力をレビューして改善", "自己検証エージェント", "品質改善ループ" Also responds to Chinese: "自我评估循环", "迭代改进", "LLM批评家模式", "自我修正循环", "输出质量改进", "批评与改进", "重新检查并修改", "提高输出质量", "自我验证智能体", "生成后批评".. Apply this skill to design or implement the Reflection agentic design pattern.
version: 1.0.0
---

# Reflection Pattern

## Overview

The **Reflection Pattern** enables an agent to critically evaluate its own outputs and iteratively improve them through self-assessment loops. Rather than accepting the first response as final, the agent acts as its own critic — identifying flaws, gaps, and inaccuracies — then generates a refined output based on that critique.

**Core Principle:** Generate → Critique → Refine — treat the first output as a draft, not a final answer.

## When This Skill Applies

Activate this pattern when:
- Output quality must meet a high standard before delivery
- The task involves complex reasoning where first attempts are often suboptimal
- You need an agent to self-correct factual errors or logical inconsistencies
- A separate "critic" role would improve the overall quality of the response
- Tasks require iterative refinement (code writing, essay drafting, plan generation)
- The cost of external human review is high and automated quality checks are preferable

**Rule of thumb:** If the task is complex enough that a human expert would draft, review, and revise — apply the Reflection pattern.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Identify the reflection requirements:
1. What is the quality bar for the final output?
2. What specific dimensions should be critiqued (accuracy, completeness, clarity, safety)?
3. How many refinement iterations are appropriate before accepting output?
4. What constitutes a "good enough" output to exit the loop?

### PLAN
Design the reflection architecture:
1. Define the **Generator**: initial response producer with a focused prompt
2. Define the **Critic**: evaluator that identifies specific weaknesses with explanations
3. Define the **Refiner**: takes original + critique and produces improved output
4. Set convergence criteria: max iterations, quality threshold, or "no more critique" signal
5. Plan logging of each iteration for debugging and audit

### ACTION
Implement the reflection loop:
1. Generate initial output with the Generator prompt
2. Apply the Critic prompt to evaluate the output on defined dimensions
3. If critique finds issues, pass original + critique to the Refiner
4. Repeat until convergence criteria met
5. Return final refined output

## Core Reflection Patterns

### Self-Critique Loop
The agent generates, then switches role to critic:
```
Generate Response → Critique Response → Refine Response → [Repeat or Exit]
```

### Dual-Agent Reflection
Two separate agents — Generator and Critic — with different system prompts:
```
Generator Agent → Draft → Critic Agent → Critique → Generator Agent → Refined Draft
```

### Multi-Dimensional Evaluation
Critic evaluates across multiple quality dimensions simultaneously:
```
Draft → [Accuracy Check || Completeness Check || Clarity Check] → Aggregated Critique → Refined Draft
```

### Stepwise Reflection
Apply reflection at each step of a pipeline, not just the final output:
```
Step 1 Draft → Reflect → Step 1 Final → Step 2 Draft → Reflect → Step 2 Final → ...
```

## Practical Use Cases

### Code Review and Improvement
```
Write initial code → Critic reviews for bugs/style → Rewrite based on critique
→ Critic re-evaluates → Accept when no critical issues remain
```

### Essay and Report Writing
```
Draft essay → Critic checks structure, evidence, clarity → Revise
→ Final proofread pass → Deliver
```

### Reasoning Verification
```
Solve problem → Verify solution step-by-step → Correct any logical errors
→ Re-verify → Output verified answer
```

### Safety Evaluation
```
Generate response → Safety critic checks for harmful content → Rewrite if flagged
→ Re-check → Output safe response
```

## Implementation with Google ADK

### ADK Iterative Reflection Loop
```python
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

# Generator agent - produces initial output
generator = LlmAgent(
    name="Generator",
    model="gemini-2.5-flash",
    instruction="""You are a skilled writer. Generate a high-quality response
    to the user's request. Focus on accuracy, clarity, and completeness."""
)

# Critic agent - evaluates and provides specific critique
critic = LlmAgent(
    name="Critic",
    model="gemini-2.5-flash",
    instruction="""You are a rigorous critic. Evaluate the given response on:
    1. Factual accuracy - are all claims correct?
    2. Completeness - are important points missing?
    3. Clarity - is the explanation clear and well-structured?
    4. Logical consistency - does the reasoning hold?

    Output a structured critique. If no issues found, output: APPROVED"""
)

# Refiner agent - applies critique to improve output
refiner = LlmAgent(
    name="Refiner",
    model="gemini-2.5-flash",
    instruction="""You are a skilled editor. Given the original response and
    critic's feedback, produce an improved version that addresses all issues
    raised in the critique."""
)

def run_agent(agent: LlmAgent, prompt: str, session_id: str) -> str:
    """Run an ADK agent with a single message and return the response text."""
    session_service = InMemorySessionService()
    runner = Runner(agent=agent, app_name="reflection_app", session_service=session_service)
    session_service.create_session(app_name="reflection_app", user_id="user1", session_id=session_id)
    message = Content(parts=[Part(text=prompt)])
    for event in runner.run(user_id="user1", session_id=session_id, new_message=message):
        if event.is_final_response():
            return event.content.parts[0].text
    return ""

def reflection_loop(initial_prompt: str, max_iterations: int = 3) -> str:
    """Run the reflection loop until approved or max iterations reached."""
    current_output = run_agent(generator, initial_prompt, "gen_session_0")

    for iteration in range(max_iterations):
        # Evaluate current output
        critique = run_agent(critic, f"Evaluate this response:\n\n{current_output}", f"critic_session_{iteration}")

        if "APPROVED" in critique:
            print(f"Approved after {iteration + 1} iteration(s)")
            return current_output

        # Refine based on critique
        current_output = run_agent(
            refiner,
            f"Original response:\n{current_output}\n\nCritique:\n{critique}\n\nImprove the response.",
            f"refiner_session_{iteration}"
        )

    return current_output
```

### Python asyncio Self-Reflection
```python
from google import genai

client = genai.Client()

def reflect_and_improve(task: str, max_iterations: int = 3) -> str:
    """Self-reflection loop using a single model."""

    # Generate initial response
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=f"Complete the following task:\n\n{task}"
    )
    current_output = response.text

    for i in range(max_iterations):
        # Self-critique
        critique_prompt = f"""Review the following response to the task: '{task}'

Response:
{current_output}

Identify specific issues with accuracy, completeness, or clarity.
If the response is satisfactory, output ONLY: APPROVED
Otherwise, list the specific issues to fix."""

        critique = client.models.generate_content(model='gemini-2.5-flash', contents=critique_prompt)

        if "APPROVED" in critique.text:
            return current_output

        # Refine based on critique
        refine_prompt = f"""Task: {task}

Current response:
{current_output}

Issues identified:
{critique.text}

Rewrite the response to fix all identified issues."""

        refined = client.models.generate_content(model='gemini-2.5-flash', contents=refine_prompt)
        current_output = refined.text

    return current_output

# Usage
result = reflect_and_improve("Explain how neural networks learn")
print(result)
```

### LangGraph Reflection Graph
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict
from google import genai

class ReflectionState(TypedDict):
    task: str
    current_output: str
    critique: str
    iteration: int
    max_iterations: int
    approved: bool

client = genai.Client()

def generate_node(state: ReflectionState) -> dict:
    if state["iteration"] == 0:
        response = client.models.generate_content(model='gemini-2.5-flash', contents=f"Complete: {state['task']}")
    else:
        response = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=(
                f"Task: {state['task']}\n\nPrevious response:\n{state['current_output']}\n\n"
                f"Issues:\n{state['critique']}\n\nImprove the response."
            )
        )
    return {"current_output": response.text, "iteration": state["iteration"] + 1}

def critique_node(state: ReflectionState) -> dict:
    critique = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=(
            f"Evaluate this response to '{state['task']}':\n\n{state['current_output']}\n\n"
            "List issues or output APPROVED if satisfactory."
        )
    )
    approved = "APPROVED" in critique.text
    return {"critique": critique.text, "approved": approved}

def should_continue(state: ReflectionState) -> str:
    if state["approved"] or state["iteration"] >= state["max_iterations"]:
        return "end"
    return "generate"

# Build the graph
graph = StateGraph(ReflectionState)
graph.add_node("generate", generate_node)
graph.add_node("critique", critique_node)
graph.set_entry_point("generate")
graph.add_edge("generate", "critique")
graph.add_conditional_edges("critique", should_continue, {"generate": "generate", "end": END})

reflection_graph = graph.compile()

result = reflection_graph.invoke({
    "task": "Write a concise explanation of quantum entanglement",
    "current_output": "",
    "critique": "",
    "iteration": 0,
    "max_iterations": 3,
    "approved": False
})
print(result["current_output"])
```

## Performance Considerations

| Iterations | Quality Improvement | Additional Latency |
|------------|--------------------|--------------------|
| 1 (no reflection) | Baseline | 0x |
| 2 (one reflection) | Significant improvement | ~2x |
| 3 (two reflections) | Near-optimal for most tasks | ~3x |
| 4+ | Diminishing returns | ~4x+ |

## Key Takeaways

- **Generate → Critique → Refine**: The fundamental loop structure
- **Separate roles**: Distinct Generator and Critic personas yield better critique quality
- **Convergence criteria**: Always define when to stop — max iterations OR quality threshold
- **Structured critique**: Specific, actionable feedback produces better refinements than vague critique
- **Diminishing returns**: Most quality gains occur in the first 1-2 reflection cycles
- **Cost trade-off**: Each iteration adds LLM call cost — balance quality vs. latency/cost

## Anti-Patterns to Avoid

- **Infinite loops**: Always set a maximum iteration limit
- **Vague critique**: "Make it better" produces worse refinements than specific issue identification
- **Self-approval bias**: A single model critiquing its own output may be overly lenient — use a separate critic model/prompt
- **Over-reflection**: Applying reflection to simple tasks wastes resources with no quality benefit
- **Ignoring convergence**: Re-running reflection after APPROVED signals wastes compute

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Automated quality improvement loop | Reflection + **Evaluation** + **Prompt Chaining** |
| Self-correcting reasoning agent | Reflection + **Reasoning** + **Planning** |
| Safe iterative output generation | Reflection + **Guardrails** + **Human-in-the-Loop** |
| Learning from past performance | Reflection + **Learning & Adaptation** + **Evaluation** |

## References

- Google ADK Agent Documentation: https://google.github.io/adk-docs/agents/
- LangGraph Reflection Pattern: https://langchain-ai.github.io/langgraph/
- Reflexion Paper: https://arxiv.org/abs/2303.11366
- Self-Refine Paper: https://arxiv.org/abs/2303.17651
