---
name: reasoning
description: This skill should be used when the user wants to "chain-of-thought prompting", "ReAct agent", "tree of thought", "step-by-step reasoning", "structured reasoning agents", "agent thinking", "scratchpad reasoning", "self-consistency", "reasoning traces", "deliberate thinking", "agent metacognition", "think before acting", "make agent think step by step", "chain of thought agent", "structured problem solving", "agent with reasoning traces", "explicit reasoning", or build agents that use explicit, structured reasoning techniques to improve accuracy and transparency. Also responds to Korean: "생각의 사슬 프롬프팅", "ReAct 에이전트", "사고 트리", "단계별 추론", "구조적 추론 에이전트", "행동 전 사고", "CoT 적용", "추론 에이전트 만들어줘", "단계적으로 생각하는 에이전트", "자기 일관성 검증", "스크래치패드 추론", "생각하면서 풀기", "추론 과정 보기". Also responds to Japanese: "思考の連鎖プロンプト", "ReActエージェント", "ステップバイステップ推論", "構造的推論エージェント", "行動前に考えるエージェント", "推論トレース", "段階的に考えてほしい", "思考過程を見たい", "自己整合性", "思考ツリー" Also responds to Chinese: "思维链提示", "ReAct智能体", "逐步推理", "结构化推理智能体", "行动前先思考", "推理轨迹", "一步一步思考", "查看思考过程", "自我一致性验证", "思维树".. Apply this skill to design or implement Reasoning Techniques as an agentic design pattern.
version: 1.0.0
---

# Reasoning Techniques Pattern

## Overview

The **Reasoning Techniques Pattern** equips agents with structured approaches to thinking through problems before acting. Rather than jumping directly from input to output, reasoning-enhanced agents explicitly decompose problems, explore possibilities, verify their work, and reflect on their reasoning process — dramatically improving accuracy on complex tasks.

**Core Principle:** Slow down to speed up accuracy — structured thinking catches errors that fast pattern-matching misses.

## When This Skill Applies

Activate this pattern when:
- Multi-step problems require logical deduction or causal reasoning
- Mathematical, coding, or analytical tasks require precise computation
- The agent must justify its conclusions (auditable decisions)
- Complex questions benefit from exploring multiple solution paths
- Self-consistency across multiple reasoning attempts is needed
- The agent must detect and correct its own errors before responding

**Rule of thumb:** If a human expert would "show their work" — the agent should too. Use reasoning techniques whenever accuracy matters more than speed.

## Key Reasoning Techniques

### 1. Chain-of-Thought (CoT)
Break reasoning into explicit intermediate steps:
```
Problem → Step 1 → Step 2 → ... → Step N → Answer
```

### 2. ReAct (Reason + Act)
Interleave reasoning with tool use:
```
Thought → Action (tool call) → Observation → Thought → Action → ... → Answer
```

### 3. Tree of Thought (ToT)
Explore multiple reasoning branches and select the best:
```
Problem → Branch A → Branch B → Branch C
                    ↓ (evaluate each)
              Select Best Path → Answer
```

### 4. Self-Consistency
Generate multiple independent solutions, pick the majority answer:
```
Problem → Solution 1 → Answer A
        → Solution 2 → Answer A (consensus)
        → Solution 3 → Answer B
```

### 5. Self-Critique (Reflexion)
Agent critiques its own output and corrects it:
```
Problem → Initial Answer → Critique → Revised Answer → Critique → Final Answer
```

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Choose the appropriate reasoning approach:
1. How complex is the problem? (Simple → CoT; Complex → ToT or ReAct)
2. Does it require external information? (Yes → ReAct)
3. How much accuracy is required? (High stakes → self-consistency or critique)
4. What's the latency budget? (Tight → CoT; Flexible → ToT or self-consistency)

### PLAN
Design the reasoning architecture:
1. Select reasoning technique(s) based on task type
2. Design the "thinking" prompt that encourages systematic reasoning
3. Define how reasoning traces are captured and used
4. Plan the critique/verification step if needed
5. Define convergence criteria for iterative techniques

### ACTION
Implement structured reasoning:
1. Craft prompts that elicit step-by-step thinking
2. Implement ReAct loop: thought → action → observation cycle
3. For ToT: generate branches, evaluate, select
4. For self-consistency: run N generations, aggregate
5. Add reasoning trace logging for transparency

## Implementation: Chain-of-Thought

### CoT Prompting
```python
from google import genai

client = genai.Client()

def chain_of_thought(problem: str) -> dict:
    """Solve a problem with explicit step-by-step reasoning."""

    cot_prompt = f"""Solve this problem step by step. Show your complete reasoning.

Problem: {problem}

Think through this carefully:
1. What type of problem is this?
2. What information do I have?
3. What approach should I use?
4. Work through each step:
   Step 1: ...
   Step 2: ...
   ...
5. Verify your answer makes sense.

Final Answer: [state the answer clearly]"""

    response = client.models.generate_content(model='gemini-2.5-flash', contents=cot_prompt)
    text = response.text

    # Extract final answer
    import re
    answer_match = re.search(r'Final Answer:\s*(.+?)(?:\n|$)', text, re.IGNORECASE)
    final_answer = answer_match.group(1).strip() if answer_match else text.split('\n')[-1]

    return {
        "problem": problem,
        "reasoning_trace": text,
        "final_answer": final_answer
    }

# Usage
result = chain_of_thought(
    "A train travels 120 km in 2 hours. How long will it take to travel 450 km at the same speed?"
)
print(f"Answer: {result['final_answer']}")
```

## Implementation: ReAct Agent

### ReAct Pattern (Reason + Act)
```python
from google import genai
from typing import Callable

client = genai.Client()

def react_agent(
    question: str,
    tools: dict[str, Callable],
    max_iterations: int = 10
) -> dict:
    """Implement ReAct: interleave thinking with tool use.

    Args:
        question: The question to answer
        tools: Dict of tool_name → callable function
        max_iterations: Maximum reasoning iterations

    Returns:
        Final answer with full reasoning trace
    """
    tool_descriptions = "\n".join([
        f"- {name}: {func.__doc__.split(chr(10))[0] if func.__doc__ else 'No description'}"
        for name, func in tools.items()
    ])

    system_prompt = f"""You are a reasoning agent. Solve problems using this format:

Thought: [Your reasoning about what to do next]
Action: tool_name[input]
Observation: [Tool output will be inserted here]
... (repeat Thought/Action/Observation as needed)
Thought: I now have enough information.
Final Answer: [Your answer]

Available tools:
{tool_descriptions}

Important: Always end with "Final Answer:" when you have the answer."""

    messages = [
        {"role": "user", "parts": [{"text": f"{system_prompt}\n\nQuestion: {question}"}]}
    ]
    full_trace = []

    for iteration in range(max_iterations):
        response = client.models.generate_content(model='gemini-2.5-flash', contents=messages)
        agent_output = response.text
        full_trace.append(f"[Iteration {iteration + 1}]\n{agent_output}")

        # Check for final answer
        if "Final Answer:" in agent_output:
            import re
            answer = re.search(r'Final Answer:\s*(.+)', agent_output, re.DOTALL)
            return {
                "question": question,
                "answer": answer.group(1).strip() if answer else agent_output,
                "reasoning_trace": "\n\n".join(full_trace),
                "iterations": iteration + 1
            }

        # Parse and execute tool action
        action_match = __import__('re').search(r'Action:\s*(\w+)\[([^\]]+)\]', agent_output)
        if action_match:
            tool_name = action_match.group(1)
            tool_input = action_match.group(2)

            if tool_name in tools:
                try:
                    tool_result = tools[tool_name](tool_input)
                except Exception as e:
                    tool_result = f"Error: {e}"
            else:
                tool_result = f"Tool '{tool_name}' not found. Available: {list(tools.keys())}"

            # Add agent output + observation to conversation
            messages.append({"role": "model", "parts": [{"text": agent_output}]})
            messages.append({
                "role": "user",
                "parts": [{"text": f"Observation: {tool_result}"}]
            })
        else:
            # No action parsed, add output and ask to continue
            messages.append({"role": "model", "parts": [{"text": agent_output}]})
            messages.append({"role": "user", "parts": [{"text": "Continue your reasoning."}]})

    return {
        "question": question,
        "answer": "Max iterations reached without final answer",
        "reasoning_trace": "\n\n".join(full_trace),
        "iterations": max_iterations
    }

# Tool functions
def search(query: str) -> str:
    """Search the web for current information."""
    return f"[Search results for '{query}']: Found relevant information..."

def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        result = eval(expression, {"__builtins__": {}}, {})
        return str(result)
    except Exception as e:
        return f"Calculation error: {e}"

# Usage
result = react_agent(
    question="If a product costs $149.99 and is on sale for 30% off, what is the final price including 8.5% tax?",
    tools={"search": search, "calculate": calculate}
)
print(f"Answer: {result['answer']}")
```

## Implementation: Tree of Thought

### ToT for Complex Problems
```python
from google import genai
from typing import List

client = genai.Client()

def tree_of_thought(problem: str, num_branches: int = 3, depth: int = 2) -> dict:
    """Explore multiple solution paths and select the best.

    Args:
        problem: The problem to solve
        num_branches: Number of solution approaches to explore
        depth: How deep to pursue each branch

    Returns:
        Best solution with evaluation rationale
    """

    # Step 1: Generate multiple solution approaches
    branch_prompt = f"""For this problem, generate {num_branches} distinct solution approaches.
Each approach should use a different strategy or perspective.

Problem: {problem}

Output {num_branches} approaches, each starting with "Approach X:" where X is the number."""

    branches_response = client.models.generate_content(model='gemini-2.5-flash', contents=branch_prompt)
    branches_text = branches_response.text

    # Parse branches
    import re
    approaches = re.findall(r'Approach \d+:(.+?)(?=Approach \d+:|$)', branches_text, re.DOTALL)
    approaches = [a.strip() for a in approaches if a.strip()][:num_branches]

    # Step 2: Develop each approach
    developed_solutions = []
    for i, approach in enumerate(approaches):
        develop_prompt = f"""Develop this solution approach fully for the problem.

Problem: {problem}
Approach: {approach}

Work through the complete solution step by step, then state the final answer."""

        response = client.models.generate_content(model='gemini-2.5-flash', contents=develop_prompt)
        developed_solutions.append({
            "approach": approach,
            "solution": response.text,
            "branch_id": i + 1
        })

    # Step 3: Evaluate all branches and select best
    solutions_text = "\n\n---\n\n".join([
        f"Branch {s['branch_id']}:\n{s['solution']}"
        for s in developed_solutions
    ])

    eval_prompt = f"""Evaluate these solution branches for the problem and select the best.

Problem: {problem}

Solutions:
{solutions_text}

Evaluation criteria:
1. Correctness: Is the reasoning logically sound?
2. Completeness: Does it fully address the problem?
3. Clarity: Is it easy to follow?

Output:
- Best branch: [number]
- Why it's best: [explanation]
- Final Answer: [the answer from the best branch]"""

    eval_response = client.models.generate_content(model='gemini-2.5-flash', contents=eval_prompt)

    # Extract best branch
    best_match = re.search(r'Best branch:\s*(\d+)', eval_response.text)
    best_branch_id = int(best_match.group(1)) - 1 if best_match else 0

    return {
        "problem": problem,
        "branches_explored": len(developed_solutions),
        "all_solutions": developed_solutions,
        "evaluation": eval_response.text,
        "best_solution": developed_solutions[best_branch_id] if developed_solutions else None
    }
```

## Implementation: Self-Consistency

### Majority Vote for High-Accuracy Answers
```python
from google import genai
from google.genai import types

client = genai.Client()

def self_consistent_answer(problem: str, num_samples: int = 5, temperature: float = 0.7) -> dict:
    """Generate multiple solutions and use majority voting for accuracy.

    Args:
        problem: The problem to solve
        num_samples: Number of independent solution attempts
        temperature: Sampling temperature (higher = more diverse)

    Returns:
        Most consistent answer with confidence score
    """
    from collections import Counter

    answers = []
    traces = []

    for i in range(num_samples):
        prompt = f"""Solve this problem step by step. Be precise and show your work.

Problem: {problem}

Work through this systematically, then give your Final Answer: [answer]"""

        response = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=prompt,
            config=types.GenerateContentConfig(temperature=temperature)
        )
        text = response.text
        traces.append(text)

        # Extract answer
        import re
        match = re.search(r'Final Answer:\s*(.+?)(?:\n|$)', text, re.IGNORECASE)
        if match:
            answers.append(match.group(1).strip().lower())
        else:
            # Use last line as answer
            answers.append(text.strip().split('\n')[-1].lower())

    # Majority vote
    vote_counts = Counter(answers)
    most_common_answer, vote_count = vote_counts.most_common(1)[0]
    confidence = vote_count / num_samples

    return {
        "problem": problem,
        "answer": most_common_answer,
        "confidence": f"{confidence:.1%}",
        "vote_distribution": dict(vote_counts),
        "samples_generated": num_samples,
        "reasoning_traces": traces
    }
```

## Key Takeaways

- **CoT for linear problems**: Chain-of-thought adds minimal overhead but catches logical errors
- **ReAct for information-needing tasks**: Interleaving reasoning with tool calls grounds answers in facts
- **ToT for design/strategy**: Exploring multiple solution branches finds better answers for open-ended problems
- **Self-consistency for high stakes**: Majority voting across N samples dramatically improves accuracy at the cost of N× API calls
- **Reasoning traces are valuable**: Captured thinking helps debug errors and build user trust
- **Temperature matters**: Use higher temperature (0.7-1.0) for ToT/self-consistency to get diverse paths

## Anti-Patterns to Avoid

- **Reasoning theater**: Prompting for "step by step" without actually requiring steps — the model skips them
- **Unchecked ToT scaling**: Exploring too many branches is exponentially expensive — limit depth and breadth
- **Self-consistency for open-ended tasks**: Voting only works for tasks with definite correct answers
- **Ignoring reasoning traces**: Generated reasoning that no one reads provides no debugging value
- **Fixed reasoning for all tasks**: Simple factual queries don't need multi-step reasoning — match technique to complexity

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Deliberate planning with reasoning traces | Reasoning + **Planning** + **Reflection** |
| Tool-using ReAct agent | Reasoning + **Tool Use** + **Prompt Chaining** |
| Complex problem decomposition | Reasoning + **Goal Setting** + **Exploration** |
| Reasoning engine selection guide | Reasoning + **appendix-reasoning-engines** |

## References

- Chain-of-Thought Paper: https://arxiv.org/abs/2201.11903
- ReAct Paper: https://arxiv.org/abs/2210.03629
- Tree of Thought Paper: https://arxiv.org/abs/2305.10601
- Self-Consistency Paper: https://arxiv.org/abs/2203.11171
- Gemini Thinking Models: https://ai.google.dev/gemini-api/docs/thinking
