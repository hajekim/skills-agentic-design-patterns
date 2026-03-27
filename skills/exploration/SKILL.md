---
name: exploration
description: This skill should be used when the user wants to "agent exploration", "exploitation vs exploration", "bandit algorithms for agents", "curiosity-driven agents", "agent discovery", "novel solution finding", "try new approaches", "epsilon-greedy agents", "UCB agents", "agent experimentation", "adaptive sampling", "explore unknown domains", "A/B testing agents", "agent experimentation framework", "optimize agent strategy", "explore solution space", "multi-armed bandit agent", or build agents that balance trying known-good approaches with discovering new, potentially better ones. Also responds to Korean: "에이전트 탐색과 활용", "밴딧 알고리즘 에이전트", "호기심 기반 에이전트", "새로운 해결책 발견", "엡실론 탐욕 정책", "적응형 샘플링", "탐색 전략", "최적 전략 찾기", "탐색 에이전트", "새로운 방법 찾아줘", "더 나은 전략 탐색해줘", "실험적으로 시도해줘". Also responds to Japanese: "探索と活用", "バンディットアルゴリズム", "好奇心駆動エージェント", "新しい解決策を見つけたい", "ε-greedy方策", "エージェントの実験", "最適戦略を探索したい", "新しいアプローチを試したい", "UCBエージェント", "適応的サンプリング" Also responds to Chinese: "探索与利用", "多臂老虎机算法", "好奇心驱动智能体", "寻找新解决方案", "ε-贪心策略", "智能体实验", "探索最优策略", "尝试新方法", "UCB智能体", "自适应采样".. Apply this skill to design or implement the Exploration & Discovery agentic design pattern.
version: 1.0.0
---

# Exploration & Discovery Pattern

## Overview

The **Exploration & Discovery Pattern** enables agents to balance exploiting known-good strategies with exploring new, potentially better approaches. Agents that only exploit what worked before become stagnant — they miss improvements, can't adapt to changing environments, and plateau at local optima. Discovery-driven agents systematically explore their action space to find better solutions over time.

**Core Principle:** The best known solution is rarely the optimal solution — reserve capacity to discover what you don't yet know.

## When This Skill Applies

Activate this pattern when:
- The agent faces multiple possible approaches with uncertain outcomes
- Known strategies may not be optimal — better methods may exist undiscovered
- The environment changes and previously good strategies may degrade
- A/B testing of agent strategies is needed to find better approaches
- Creative or novel solutions are more valuable than reliable familiar ones
- The agent must find solutions in a large, partially-unknown solution space

**Rule of thumb:** If the agent always does the same thing, use exploration. If it never commits to what works, use exploitation. The goal is the right balance.

## Exploration-Exploitation Trade-off

### The Core Dilemma
```
Exploit: Use the strategy that has worked best so far
    → Safe, predictable, but misses better options

Explore: Try new/less-tested strategies
    → Risky, uncertain, but may find better solutions

Balance: ε-greedy, UCB, Thompson Sampling
    → Systematic, principled trade-off
```

### Key Algorithms

| Algorithm | Exploration Strategy | Best For |
|-----------|---------------------|----------|
| **ε-greedy** | Random with prob ε | Simple, easy to implement |
| **UCB (Upper Confidence Bound)** | Explore high-uncertainty options | When you can count trials |
| **Thompson Sampling** | Sample from belief distribution | Probabilistic rewards |
| **Boltzmann Exploration** | Temperature-based softmax | Graduated exploration |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the exploration requirements:
1. What is the action/strategy space to explore? (List of options, approaches, tools)
2. What is the reward/quality signal? (User satisfaction, task completion, accuracy)
3. What is the acceptable exploration cost? (How much worse can we be while exploring?)
4. How large is the exploration budget? (Time, API calls, user interactions)
5. When should exploration stop? (Convergence criteria, budget exhaustion)

### PLAN
Design the exploration strategy:
1. Choose exploration algorithm based on context (ε-greedy for simplicity, UCB for principled)
2. Define the reward function: how is each action's success measured?
3. Design the exploration schedule: how does ε decay as confidence grows?
4. Plan for non-stationarity: how does the agent detect when the environment changes?
5. Define diversity mechanisms: ensure exploration covers the space, not just nearby alternatives

### ACTION
Implement exploration-aware agent:
1. Maintain action history with reward estimates
2. Apply chosen exploration algorithm to select actions
3. Execute and observe outcomes
4. Update reward estimates based on observations
5. Decay exploration rate as confidence grows (or reset if environment changes)

## Implementation: ε-Greedy Strategy Selection

### Multi-Armed Bandit for Agent Strategies
```python
import random
import math
from dataclasses import dataclass, field
from typing import List, Callable, Optional
from google import genai

@dataclass
class Strategy:
    """A candidate strategy with tracked performance."""
    name: str
    description: str
    prompt_template: str
    trials: int = 0
    total_reward: float = 0.0
    rewards: list = field(default_factory=list)

    @property
    def avg_reward(self) -> float:
        return self.total_reward / self.trials if self.trials > 0 else 0.0

    @property
    def ucb_score(self, total_trials: int, confidence: float = 2.0) -> float:
        """Upper Confidence Bound score for exploration."""
        if self.trials == 0:
            return float('inf')  # Unexplored = maximum priority
        exploitation = self.avg_reward
        exploration = confidence * math.sqrt(math.log(total_trials) / self.trials)
        return exploitation + exploration

class ExplorationAgent:
    """Agent that explores different prompting/reasoning strategies."""

    def __init__(
        self,
        strategies: List[Strategy],
        epsilon: float = 0.2,     # 20% exploration rate initially
        epsilon_decay: float = 0.995,  # Decay toward exploitation
        min_epsilon: float = 0.05     # Always keep 5% exploration
    ):
        self.strategies = strategies
        self.epsilon = epsilon
        self.epsilon_decay = epsilon_decay
        self.min_epsilon = min_epsilon
        self.total_trials = 0
        self.client = genai.Client()

    def select_strategy(self, method: str = "epsilon_greedy") -> Strategy:
        """Select a strategy using the specified exploration method."""
        self.total_trials += 1

        if method == "epsilon_greedy":
            if random.random() < self.epsilon:
                # Explore: pick random strategy
                strategy = random.choice(self.strategies)
                print(f"[EXPLORE] Trying: {strategy.name}")
            else:
                # Exploit: pick best known strategy
                strategy = max(self.strategies, key=lambda s: s.avg_reward)
                print(f"[EXPLOIT] Using best: {strategy.name} (avg reward: {strategy.avg_reward:.2f})")

            # Decay epsilon
            self.epsilon = max(self.min_epsilon, self.epsilon * self.epsilon_decay)

        elif method == "ucb":
            # Upper Confidence Bound: balance exploration and exploitation
            strategy = max(
                self.strategies,
                key=lambda s: s.ucb_score(self.total_trials)
            )
            print(f"[UCB] Selected: {strategy.name}")

        elif method == "random":
            strategy = random.choice(self.strategies)

        else:
            strategy = max(self.strategies, key=lambda s: s.avg_reward)

        return strategy

    def execute_strategy(self, task: str, strategy: Strategy) -> str:
        """Execute a specific strategy on a task."""
        prompt = strategy.prompt_template.format(task=task)
        response = self.client.models.generate_content(model='gemini-2.5-flash', contents=prompt)
        return response.text

    def update_reward(self, strategy: Strategy, reward: float):
        """Update strategy reward estimates after observing outcome."""
        strategy.trials += 1
        strategy.total_reward += reward
        strategy.rewards.append(reward)

    def run_with_exploration(self, task: str, reward_fn: Callable[[str], float]) -> dict:
        """Execute task with automatic strategy exploration and learning.

        Args:
            task: The task to complete
            reward_fn: Function that scores a response (returns 0.0 to 1.0)

        Returns:
            Response with strategy used and reward
        """
        strategy = self.select_strategy(method="ucb")
        response = self.execute_strategy(task, strategy)
        reward = reward_fn(response)
        self.update_reward(strategy, reward)

        return {
            "response": response,
            "strategy_used": strategy.name,
            "reward": reward,
            "epsilon": self.epsilon,
            "strategy_stats": {
                s.name: {"trials": s.trials, "avg_reward": f"{s.avg_reward:.2f}"}
                for s in self.strategies
            }
        }

    def get_learning_summary(self) -> dict:
        """Summarize what the agent has learned about strategies."""
        ranked = sorted(self.strategies, key=lambda s: s.avg_reward, reverse=True)
        return {
            "total_trials": self.total_trials,
            "current_epsilon": f"{self.epsilon:.2%}",
            "strategy_ranking": [
                {
                    "rank": i + 1,
                    "strategy": s.name,
                    "avg_reward": f"{s.avg_reward:.2f}",
                    "trials": s.trials
                }
                for i, s in enumerate(ranked)
            ],
            "best_strategy": ranked[0].name if ranked else None
        }

# Define competing strategies
strategies = [
    Strategy(
        name="Chain-of-Thought",
        description="Step-by-step reasoning",
        prompt_template="Solve step by step:\n\n{task}\n\nStep 1:"
    ),
    Strategy(
        name="Direct-Answer",
        description="Concise direct response",
        prompt_template="Answer concisely:\n\n{task}"
    ),
    Strategy(
        name="Socratic",
        description="Question-based exploration",
        prompt_template="To answer '{task}', first clarify: What are the key unknowns? Then solve."
    ),
    Strategy(
        name="Analogical",
        description="Solve via analogy",
        prompt_template="Solve by analogy:\n\n{task}\n\nThis is similar to..."
    )
]

agent = ExplorationAgent(strategies, epsilon=0.3)

# Reward function (in production: user ratings, task completion, etc.)
def simple_reward(response: str) -> float:
    """Simple length-based reward (replace with domain-specific metric)."""
    if len(response) < 50:
        return 0.2  # Too short
    elif len(response) > 2000:
        return 0.6  # Too long
    else:
        return 0.8  # Good length
```

## Implementation: Curiosity-Driven Discovery

### Novelty-Based Exploration
```python
import numpy as np
from typing import List

class CuriosityDrivenAgent:
    """Agent that prioritizes exploring novel states/approaches."""

    def __init__(self, exploration_bonus: float = 0.3):
        self.exploration_bonus = exploration_bonus
        self.visited_states: List[str] = []
        self.state_embeddings: List[list] = []
        self.client = genai.Client()

    def get_novelty_score(self, candidate: str) -> float:
        """Score how novel a candidate approach is relative to visited states."""
        if not self.visited_states:
            return 1.0  # First state is maximally novel

        # Simple text-overlap novelty (use embeddings in production)
        candidate_words = set(candidate.lower().split())
        max_overlap = max(
            len(candidate_words & set(visited.lower().split())) / len(candidate_words | set(visited.lower().split()))
            for visited in self.visited_states
        )

        return 1.0 - max_overlap  # Higher = more novel

    def explore_solution_space(self, problem: str, num_candidates: int = 5) -> List[dict]:
        """Generate diverse candidate approaches and rank by novelty + quality.

        Args:
            problem: The problem to solve
            num_candidates: Number of diverse approaches to generate

        Returns:
            Ranked list of approaches with novelty scores
        """
        # Generate diverse approaches
        diversity_prompt = f"""Generate {num_candidates} distinctly different approaches to solve this problem.
Each approach should use a different strategy, framework, or perspective.

Problem: {problem}

Output {num_candidates} approaches, clearly labeled as Approach 1:, Approach 2:, etc."""

        response = self.client.models.generate_content(model='gemini-2.5-flash', contents=diversity_prompt)

        # Parse approaches
        import re
        approaches = re.findall(r'Approach \d+:\s*(.+?)(?=Approach \d+:|$)', response.text, re.DOTALL)
        approaches = [a.strip() for a in approaches if a.strip()]

        # Score by novelty
        scored_approaches = []
        for approach in approaches:
            novelty = self.get_novelty_score(approach)
            scored_approaches.append({
                "approach": approach,
                "novelty_score": novelty,
                "combined_score": novelty + self.exploration_bonus
            })

        # Sort by novelty (prefer unexplored territory)
        scored_approaches.sort(key=lambda x: x["combined_score"], reverse=True)

        # Track that we've explored these approaches
        for sa in scored_approaches:
            self.visited_states.append(sa["approach"])

        return scored_approaches

# Usage
curiosity_agent = CuriosityDrivenAgent()
approaches = curiosity_agent.explore_solution_space(
    "Design a system for real-time fraud detection in financial transactions",
    num_candidates=4
)

for i, approach in enumerate(approaches):
    print(f"[Novelty: {approach['novelty_score']:.2f}] Approach {i+1}: {approach['approach'][:100]}")
```

### A/B Testing Framework for Agent Strategies
```python
import random
from collections import defaultdict
from typing import Optional

class AgentABTest:
    """Run controlled experiments across agent variants."""

    def __init__(self, variants: dict, traffic_split: Optional[dict] = None):
        """
        Args:
            variants: Dict of variant_name → agent_function
            traffic_split: Dict of variant_name → traffic fraction (must sum to 1.0)
        """
        self.variants = variants
        self.traffic_split = traffic_split or {k: 1/len(variants) for k in variants}
        self.results = defaultdict(list)

    def route_request(self, request_id: str) -> str:
        """Deterministically route a request to a variant."""
        # Use hash for consistent routing (same request always goes to same variant)
        hash_val = hash(request_id) % 100
        cumulative = 0
        for variant_name, fraction in self.traffic_split.items():
            cumulative += fraction * 100
            if hash_val < cumulative:
                return variant_name
        return list(self.variants.keys())[-1]

    def run_experiment(self, request_id: str, task: str, reward_fn) -> dict:
        """Route request to variant and record outcome."""
        variant_name = self.route_request(request_id)
        agent_fn = self.variants[variant_name]

        response = agent_fn(task)
        reward = reward_fn(response)

        self.results[variant_name].append(reward)

        return {
            "variant": variant_name,
            "response": response,
            "reward": reward
        }

    def get_experiment_results(self) -> dict:
        """Get statistical summary of A/B test results."""
        summary = {}
        for variant, rewards in self.results.items():
            if rewards:
                avg = sum(rewards) / len(rewards)
                summary[variant] = {
                    "trials": len(rewards),
                    "avg_reward": f"{avg:.3f}",
                    "total_reward": f"{sum(rewards):.3f}"
                }

        # Identify winner
        if summary:
            winner = max(summary.items(), key=lambda x: float(x[1]["avg_reward"]))[0]
            summary["winner"] = winner

        return summary
```

## Key Takeaways

- **ε-greedy simplicity**: Start with 20-30% exploration, decay toward 5% as you gain confidence
- **UCB principled exploration**: Automatically balances exploration and exploitation using uncertainty
- **Curiosity bonus**: Explicitly reward visiting novel states — prevents getting stuck in local optima
- **Strategy history**: Track all approaches tried and their outcomes — this is your exploration database
- **A/B testing**: Formal controlled experiments give statistical confidence in strategy comparisons
- **Reset on drift**: If the environment changes and rewards drop, increase epsilon to re-explore

## Anti-Patterns to Avoid

- **Pure exploitation**: Only doing what worked before misses improvements and fails to adapt
- **Pure exploration**: Random action forever achieves nothing — you must commit to what works
- **No reward signal**: You cannot learn from exploration without measuring outcomes
- **Confounding variables**: A/B tests must isolate the variable being tested — change one thing at a time
- **Too-fast epsilon decay**: Decaying exploration too quickly locks in local optima permanently

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Adaptive learning agent | Exploration + **Learning & Adaptation** + **Evaluation** |
| Creative problem-solving agent | Exploration + **Reasoning** + **Reflection** |
| Goal-driven discovery | Exploration + **Goal Setting** + **Planning** |
| Resource-aware exploration | Exploration + **Resource-Aware** + **Prioritization** |

## References

- Multi-Armed Bandit Algorithms: https://en.wikipedia.org/wiki/Multi-armed_bandit
- UCB Algorithm Paper: https://homes.di.unimi.it/~cesabian/Pubblicazioni/ml-02.pdf
- Thompson Sampling: https://arxiv.org/abs/1209.3352
- Exploration in RL: https://spinningup.openai.com/en/latest/spinningup/rl_intro2.html
- Google Vertex AI Experiments: https://cloud.google.com/vertex-ai/docs/experiments/
