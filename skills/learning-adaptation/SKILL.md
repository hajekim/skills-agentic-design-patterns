---
name: learning-adaptation
description: This skill should be used when the user wants to "agents that improve over time", "self-improving agents", "reinforcement learning", "few-shot learning", "online learning", "agent fine-tuning", "adapt from experience", "learn from feedback", "update agent behavior", "self-modifying agents", "RLHF", "DPO", "agent that learns", "adaptive agent", "continuous improvement agent", "agent from feedback", "self-tuning agent", "improve from examples", or build agents that evolve their capabilities based on new data, feedback, or interactions. Also responds to Korean: "시간에 따라 개선되는 에이전트", "자기 개선 에이전트", "강화 학습 에이전트", "경험에서 학습", "피드백 기반 적응", "에이전트 파인튜닝", "학습하는 에이전트", "적응형 에이전트", "에이전트 개선", "쓸수록 똑똑해지는 에이전트", "피드백 반영해줘", "스스로 개선하는 에이전트". Also responds to Japanese: "時間とともに改善するエージェント", "自己改善エージェント", "強化学習エージェント", "フィードバックから学習", "適応型エージェント", "使うほど賢くなるエージェント", "エージェントのファインチューニング", "継続的改善エージェント", "経験から学ぶ", "フィードバックを反映したい" Also responds to Chinese: "随时间改进的智能体", "自我改进智能体", "强化学习智能体", "从反馈中学习", "自适应智能体", "越用越智能的智能体", "智能体微调", "持续改进智能体", "从经验中学习", "反馈驱动适应".. Apply this skill to design or implement the Learning and Adaptation agentic design pattern.
version: 1.0.0
---

# Learning and Adaptation Pattern

## Overview

The **Learning and Adaptation Pattern** enables agents to improve their performance over time by learning from experience, feedback, and new data. Unlike static agents that follow fixed rules, adaptive agents evolve — refining their strategies, expanding their knowledge, and improving their decision-making based on what they observe in their environment.

**Core Principle:** Agents that learn from experience become better over time — evolving from rule-followers to genuine problem-solvers.

## When This Skill Applies

Activate this pattern when:
- Agent performance must improve autonomously without constant manual intervention
- The environment changes and the agent must adapt to new conditions
- Feedback from interactions should influence future behavior
- The agent needs to generalize from past experiences to novel situations
- Personalization requires learning individual user preferences over time
- The agent must distinguish between successful and unsuccessful strategies

**Rule of thumb:** Use Learning and Adaptation when the agent needs to become smarter from doing, not just from being told.

## Learning Mechanisms

### 1. Reinforcement Learning (RL)
Agents try actions and receive rewards for positive outcomes, penalties for negative ones:
- **Application**: Robotics, game-playing agents, optimization tasks
- **Key algorithms**: PPO (Proximal Policy Optimization), DPO (Direct Preference Optimization)
- **Strength**: Learns optimal behaviors in complex, sequential decision environments

### 2. Supervised Learning
Agents learn from labeled examples — input → desired output mappings:
- **Application**: Classification, prediction, decision support
- **Strength**: High accuracy on known task categories with labeled training data

### 3. Few-Shot / Zero-Shot Learning with LLMs
LLM-based agents adapt to new tasks with minimal examples or clear instructions:
- **Application**: Rapid adaptation to new domains or task formats
- **Strength**: No fine-tuning required — in-context learning from examples

### 4. Online Learning
Agents continuously update knowledge with new data as it arrives:
- **Application**: Real-time recommendation, fraud detection, stock trading
- **Strength**: Stays current with rapidly changing environments

### 5. Memory-Based Learning
Agents recall past experiences to adjust current actions in similar situations:
- **Application**: Conversational personalization, task automation
- **Strength**: Accumulates domain expertise through experience storage and retrieval

## PPO vs. DPO for LLM Alignment

### Proximal Policy Optimization (PPO)
A reinforcement learning algorithm for training agents in continuous action spaces:
1. **Collect experiences**: Agent interacts with environment and records (state, action, reward) triples
2. **Evaluate policy updates**: Calculate how a potential update changes expected reward
3. **Clipping mechanism**: Prevents drastic policy updates — maintains "trust region" around current policy
4. **Iterate**: Gradually improve policy while avoiding catastrophic failures

### Direct Preference Optimization (DPO)
A simpler, more stable alternative specifically for aligning LLMs with human preferences:
1. **Collect preference data**: Humans compare responses (Response A is better than Response B)
2. **Direct optimization**: Skip the reward model — directly update LLM policy from preference data
3. **Mathematical link**: Uses relationship that connects preference data directly to optimal policy
4. **Result**: "Increase probability of preferred responses, decrease probability of disfavored ones"

| Aspect | PPO | DPO |
|--------|-----|-----|
| Reward model | Required (separate training) | Not needed |
| Complexity | High | Lower |
| Stability | Can be unstable | More robust |
| Use case | Continuous action spaces | LLM preference alignment |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Identify the learning requirements:
1. What performance dimension must improve? (Accuracy, relevance, user satisfaction)
2. What is the source of learning signal? (Feedback, rewards, new data)
3. What learning mechanism is most appropriate? (RL, few-shot, online, memory-based)
4. How will improvements be measured and validated?

### PLAN
Design the learning architecture:
1. Define the feedback collection mechanism (explicit ratings, implicit signals, outcomes)
2. Choose the learning algorithm or approach
3. Design how learned knowledge is stored (memory, updated prompts, fine-tuned weights)
4. Plan the learning cycle: collect → learn → update → evaluate
5. Define safety guardrails to prevent harmful adaptation

### ACTION
Implement the adaptive learning system:
1. Build the feedback collection pipeline
2. Implement the learning mechanism (few-shot prompt updates, fine-tuning, RAG updates)
3. Apply learned improvements to agent behavior
4. Monitor performance metrics before and after each learning cycle
5. Test for regression — ensure adaptation doesn't degrade other capabilities

## Practical Use Cases

### Personalized Assistant (Memory-Based Learning)
```
Session 1: User says "I prefer concise answers"
→ Store preference in user memory
Session 2: Agent retrieves preference → automatically gives concise responses
→ Agent has "learned" user preference without explicit retraining
```

### Knowledge Base Learning Agent (RAG-Based)
```
Agent encounters novel problem → fails to solve optimally
→ Human expert provides solution
→ Store (problem, solution) pair in vector knowledge base
Next time similar problem arises → retrieve successful pattern → apply it
→ Agent improves through accumulated experience
```

### Self-Improving Coding Agent (SICA Pattern)
```
Base agent code → Run on benchmark tests → Score performance
→ Agent analyzes own code → Identifies improvements
→ Modifies own source code (adds better tools, improves algorithms)
→ Run benchmarks again → Record improved performance
→ Select best version → Repeat improvement cycle
```

## Implementation: Few-Shot Adaptive Learning
```python
from google import genai
from typing import List, Dict

client = genai.Client()

class AdaptiveFewShotAgent:
    """Agent that learns from successful examples through few-shot prompting."""

    def __init__(self):
        self.successful_examples: List[Dict] = []
        self.failed_examples: List[Dict] = []

    def respond(self, user_query: str) -> str:
        """Generate response using accumulated examples as few-shot context."""
        # Build few-shot context from successful examples
        few_shot_context = ""
        if self.successful_examples:
            few_shot_context = "Here are examples of successful interactions:\n\n"
            for ex in self.successful_examples[-5:]:  # Last 5 successes
                few_shot_context += f"Query: {ex['query']}\nResponse: {ex['response']}\n\n"

        prompt = f"{few_shot_context}Now respond to: {user_query}"
        response = client.models.generate_content(model='gemini-2.5-flash', contents=prompt)
        return response.text

    def record_feedback(self, query: str, response: str, success: bool, rating: float):
        """Record feedback to improve future responses."""
        example = {"query": query, "response": response, "rating": rating}
        if success and rating >= 4.0:
            self.successful_examples.append(example)
            # Keep only top N examples by rating
            self.successful_examples.sort(key=lambda x: x["rating"], reverse=True)
            self.successful_examples = self.successful_examples[:20]
        elif not success:
            self.failed_examples.append(example)

    def adapt_instruction(self) -> str:
        """Generate improved instruction based on learned patterns."""
        if not self.successful_examples:
            return "Respond helpfully to user queries."

        examples_text = "\n".join([f"- {ex['query']}: {ex['response'][:100]}"
                                   for ex in self.successful_examples[:5]])
        adaptation_prompt = f"""Based on these successful interactions:
{examples_text}

What patterns make these responses successful? Write a concise instruction
that captures these patterns for future responses."""

        adaptation = client.models.generate_content(model='gemini-2.5-flash', contents=adaptation_prompt)
        return adaptation.text

# Usage
agent = AdaptiveFewShotAgent()

# Initial responses
response = agent.respond("What is machine learning?")
agent.record_feedback("What is machine learning?", response, success=True, rating=4.5)

response2 = agent.respond("Explain neural networks")
agent.record_feedback("Explain neural networks", response2, success=True, rating=5.0)

# Adapt based on learned patterns
new_instruction = agent.adapt_instruction()
print(f"Adapted instruction: {new_instruction}")
```

## Implementation: Online Learning with Feedback Loop
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from typing import List
import json

# Feedback store
feedback_store: List[dict] = []

def record_interaction_feedback(
    query: str,
    response: str,
    user_rating: int,
    improvement_suggestion: str = ""
) -> dict:
    """Records user feedback on agent responses for learning.

    Args:
        query: The original user query
        response: The agent's response
        user_rating: Rating from 1-5 (5 = excellent)
        improvement_suggestion: Optional suggestion for improvement

    Returns:
        Confirmation of recorded feedback
    """
    feedback_entry = {
        "query": query,
        "response": response,
        "rating": user_rating,
        "suggestion": improvement_suggestion,
        "timestamp": __import__("time").time()
    }
    feedback_store.append(feedback_entry)

    return {
        "status": "recorded",
        "total_feedback_entries": len(feedback_store),
        "average_rating": sum(f["rating"] for f in feedback_store) / len(feedback_store)
    }

def get_learning_summary() -> dict:
    """Retrieves a summary of learning from feedback.

    Returns:
        Summary of feedback patterns and improvement areas
    """
    if not feedback_store:
        return {"message": "No feedback collected yet"}

    low_rated = [f for f in feedback_store if f["rating"] <= 2]
    high_rated = [f for f in feedback_store if f["rating"] >= 4]

    return {
        "total_interactions": len(feedback_store),
        "average_rating": sum(f["rating"] for f in feedback_store) / len(feedback_store),
        "areas_needing_improvement": [f["query"] for f in low_rated[-5:]],
        "successful_patterns": [f["query"] for f in high_rated[-5:]],
        "suggestions": [f["suggestion"] for f in low_rated if f["suggestion"]]
    }

# Adaptive agent with feedback tools
adaptive_agent = LlmAgent(
    name="AdaptiveAssistant",
    model="gemini-2.5-flash",
    instruction="""You are an adaptive assistant that learns from user feedback.

    When responding to queries:
    1. Provide your best response
    2. If the user rates your response, use record_interaction_feedback to log it
    3. Periodically check get_learning_summary to understand patterns
    4. Adapt your response style based on feedback patterns

    Your goal is to continuously improve based on what users find most helpful.""",
    tools=[
        FunctionTool(record_interaction_feedback),
        FunctionTool(get_learning_summary)
    ]
)
```

## Implementation: SICA-Inspired Self-Improvement Loop
```python
from google import genai
import json

client = genai.Client()

class SelfImprovingAgent:
    """Agent that iteratively improves its own instructions based on performance."""

    def __init__(self, base_instruction: str):
        self.current_instruction = base_instruction
        self.performance_history = []

    def run_with_benchmark(self, benchmark_tasks: list) -> float:
        """Run agent on benchmark tasks and return performance score."""
        scores = []
        for task in benchmark_tasks:
            response = client.models.generate_content(
                model='gemini-2.5-flash',
                contents=f"System: {self.current_instruction}\n\nTask: {task['query']}"
            )
            # Evaluate response against expected output
            eval_prompt = f"""Rate this response (0.0-1.0):
Task: {task['query']}
Expected: {task['expected']}
Response: {response.text}
Output only a number between 0.0 and 1.0."""
            score = float(client.models.generate_content(model='gemini-2.5-flash', contents=eval_prompt).text.strip())
            scores.append(score)

        avg_score = sum(scores) / len(scores)
        self.performance_history.append({
            "instruction": self.current_instruction[:100],
            "score": avg_score
        })
        return avg_score

    def self_improve(self, benchmark_tasks: list) -> str:
        """Analyze performance and propose improved instructions."""
        current_score = self.run_with_benchmark(benchmark_tasks)

        history_text = json.dumps(self.performance_history[-3:], indent=2)
        improvement_prompt = f"""Current agent instruction:
{self.current_instruction}

Recent performance history:
{history_text}

Current score: {current_score:.2f}

Analyze what's working and what's not. Propose an improved instruction that
addresses weaknesses while preserving strengths. Output ONLY the new instruction."""

        new_instruction = client.models.generate_content(model='gemini-2.5-flash', contents=improvement_prompt).text
        new_score = self.run_with_benchmark(benchmark_tasks)

        if new_score > current_score:
            self.current_instruction = new_instruction
            return f"Improved! Score: {current_score:.2f} → {new_score:.2f}"
        else:
            return f"No improvement. Keeping current instruction. Score: {current_score:.2f}"

# Usage
agent = SelfImprovingAgent(
    base_instruction="You are a helpful assistant. Answer questions concisely."
)

benchmarks = [
    {"query": "Explain recursion in one sentence", "expected": "Function calling itself"},
    {"query": "What is O(n log n)?", "expected": "Linearithmic time complexity"},
]

for iteration in range(3):
    result = agent.self_improve(benchmarks)
    print(f"Iteration {iteration + 1}: {result}")
```

## Key Takeaways

- **Multiple learning mechanisms**: RL, supervised, few-shot, online, and memory-based learning each suit different scenarios
- **PPO for stability**: Uses clipping to prevent drastic policy updates that could cause catastrophic forgetting
- **DPO for simplicity**: Directly aligns LLMs with human preferences without a separate reward model
- **SICA pattern**: Self-improving agents can modify their own code/instructions based on benchmark performance
- **Feedback loops are essential**: Learning requires a feedback signal — explicit ratings, rewards, or outcome measurements
- **Memory as learning**: Storing and retrieving successful interactions is a practical form of agent learning

## Anti-Patterns to Avoid

- **Learning without validation**: Adapting blindly without measuring improvement can degrade performance
- **Catastrophic forgetting**: Over-adapting to recent data erases previously learned capabilities
- **Reward hacking**: Agents find shortcuts to maximize rewards without achieving the true objective
- **Unbounded adaptation**: Agents must have constraints on what they can modify about themselves
- **No rollback mechanism**: Always maintain a checkpoint of the previous best version before adaptation

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Continuously improving agent | Learning & Adaptation + **Reflection** + **Evaluation** |
| Adaptive exploration strategy | Learning & Adaptation + **Exploration** + **Goal Setting** |
| Fine-tuning from interaction data | Learning & Adaptation + **Memory Management** + **Evaluation** |
| Self-optimizing multi-agent team | Learning & Adaptation + **Multi-Agent Collaboration** |

## References

- SICA (Self-Improving Coding Agent): https://github.com/MaximeRobeyns/self_improving_coding_agent/
- PPO Algorithm: https://arxiv.org/abs/1707.06347
- DPO Paper: https://arxiv.org/abs/2305.18290
- LangGraph Memory for Adaptation: https://langchain-ai.github.io/langgraph/concepts/memory/
- Google ADK: https://google.github.io/adk-docs/
