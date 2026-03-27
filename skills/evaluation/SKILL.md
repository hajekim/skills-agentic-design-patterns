---
name: evaluation
description: This skill should be used when the user wants to "evaluate agent performance", "agent benchmarking", "LLM evaluation", "agent testing", "measure agent quality", "agent metrics", "hallucination detection", "answer quality scoring", "agent monitoring in production", "LLM observability", "agent evals", "test agent accuracy", "LLM judge", "eval framework", "regression testing agents", "agent quality assurance", "automated agent testing", "agent accuracy metrics", or build evaluation frameworks that measure and monitor agent quality. Also responds to Korean: "에이전트 성능 평가", "LLM 평가 시스템", "에이전트 품질 측정", "환각 탐지", "에이전트 벤치마킹", "프로덕션 모니터링", "에이전트 테스트", "성능 측정", "에이전트 평가해줘", "에이전트 품질 검사해줘", "할루시네이션 잡아줘", "정확도 측정해줘". Also responds to Japanese: "エージェントの評価", "LLM評価システム", "エージェント品質測定", "ハルシネーション検出", "ベンチマーク", "精度測定", "エージェントをテストしたい", "品質チェック", "LLMジャッジ", "本番環境モニタリング" Also responds to Chinese: "智能体性能评估", "LLM评估系统", "智能体质量测量", "幻觉检测", "基准测试", "精度测量", "想测试智能体", "质量检查", "LLM评判者", "生产环境监控".. Apply this skill to design or implement the Evaluation & Monitoring agentic design pattern.
version: 1.0.0
---

# Evaluation & Monitoring Pattern

## Overview

The **Evaluation & Monitoring Pattern** establishes systematic methods for measuring agent quality, detecting degradation, and maintaining performance standards over time. Without evaluation, you cannot know if your agent is actually working correctly — and without monitoring, you won't know when it stops working.

**Core Principle:** You can't improve what you don't measure — define quality metrics before deployment, not after problems surface.

## When This Skill Applies

Activate this pattern when:
- An agent is being deployed to production and performance must be tracked
- Agent behavior needs to be compared before and after changes
- Hallucination rates, accuracy, or helpfulness must be measured quantitatively
- A/B testing of different agent configurations is needed
- Regulatory compliance requires audit trails and performance documentation
- You need to detect agent drift or degradation over time

**Rule of thumb:** Every production agent needs evaluation and monitoring — this isn't optional, it's how you know the agent is doing its job.

## Evaluation Dimensions

| Dimension | What It Measures | Evaluation Method |
|-----------|-----------------|-------------------|
| **Correctness** | Is the answer right? | Ground truth comparison, expert review |
| **Faithfulness** | Are claims grounded in context? | RAG evaluation, hallucination detection |
| **Relevance** | Does response address the question? | LLM-as-judge, human ratings |
| **Completeness** | Are all aspects covered? | Checklist evaluation |
| **Safety** | Is output appropriate/harmless? | Safety classifier, policy compliance |
| **Latency** | How fast does the agent respond? | P50/P95/P99 timing metrics |
| **Cost** | What is the per-query cost? | Token counting, API cost tracking |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Establish evaluation requirements:
1. What does "good" mean for this specific agent? (Domain-specific criteria)
2. What ground truth data is available? (Golden datasets, human labels)
3. What metrics matter most? (Accuracy, safety, cost, latency — prioritize)
4. How frequently should the agent be evaluated? (Continuous vs. periodic)
5. What triggers a production alert? (Threshold-based, anomaly-based)

### PLAN
Design the evaluation architecture:
1. Create a golden evaluation dataset (representative queries + expected outputs)
2. Choose evaluation methods: LLM-as-judge, human review, automated metrics
3. Design the monitoring pipeline: what to log, where, and how to alert
4. Plan A/B testing framework for comparing agent versions
5. Define remediation process when evaluation detects problems

### ACTION
Implement evaluation and monitoring:
1. Build automated evaluation pipeline with golden dataset
2. Instrument the agent with logging (latency, tokens, errors)
3. Set up dashboards and alerts for key metrics
4. Run baseline evaluation before every deployment
5. Monitor production traffic continuously for drift

## Implementation: LLM-as-Judge Evaluation

### Automated Quality Assessment
```python
from google import genai
from dataclasses import dataclass
from typing import List, Optional
import json
import re

@dataclass
class EvaluationResult:
    query: str
    response: str
    reference_answer: Optional[str]
    correctness: float      # 0.0 to 1.0
    relevance: float
    faithfulness: float
    overall_score: float
    reasoning: str
    passed: bool

class AgentEvaluator:
    """Evaluate agent responses using LLM-as-judge methodology."""

    def __init__(self, judge_model_name: str = "gemini-2.5-flash"):
        self.client = genai.Client()
        self.judge_model_name = judge_model_name

    def evaluate_response(
        self,
        query: str,
        response: str,
        reference_answer: Optional[str] = None,
        context: Optional[str] = None
    ) -> EvaluationResult:
        """Evaluate a single agent response across multiple dimensions."""

        eval_prompt = f"""You are an expert evaluator for AI agent responses.
Evaluate the following response on three dimensions, each scored 0.0 to 1.0.

Query: {query}
{"Reference Answer: " + reference_answer if reference_answer else ""}
{"Context Provided: " + context[:500] if context else ""}
Agent Response: {response}

Score each dimension:
1. Correctness (0.0-1.0): Is the response factually accurate?
   - 1.0 = Completely correct
   - 0.5 = Partially correct with some errors
   - 0.0 = Factually wrong or misleading

2. Relevance (0.0-1.0): Does the response address the query?
   - 1.0 = Directly and completely answers the question
   - 0.5 = Partially addresses the question
   - 0.0 = Off-topic or misunderstood the question

3. Faithfulness (0.0-1.0): Are claims grounded in the context?
   - 1.0 = All claims supported by provided context
   - 0.5 = Mix of grounded and ungrounded claims
   - 0.0 = Claims contradict or go beyond the context
   (Score 0.7 if no context was provided — can't verify grounding)

Output as JSON:
{{
  "correctness": X.X,
  "relevance": X.X,
  "faithfulness": X.X,
  "reasoning": "Brief explanation of scores"
}}"""

        result = self.client.models.generate_content(model=self.judge_model_name, contents=eval_prompt)

        # Parse JSON response
        match = re.search(r'\{[\s\S]+\}', result.text)
        if match:
            scores = json.loads(match.group())
        else:
            scores = {"correctness": 0.5, "relevance": 0.5, "faithfulness": 0.5, "reasoning": "Parse error"}

        overall = (scores["correctness"] + scores["relevance"] + scores["faithfulness"]) / 3

        return EvaluationResult(
            query=query,
            response=response,
            reference_answer=reference_answer,
            correctness=scores["correctness"],
            relevance=scores["relevance"],
            faithfulness=scores["faithfulness"],
            overall_score=overall,
            reasoning=scores.get("reasoning", ""),
            passed=overall >= 0.7  # 70% threshold
        )

    def evaluate_dataset(
        self,
        test_cases: List[dict],
        agent_fn
    ) -> dict:
        """Evaluate agent across a full test dataset.

        Args:
            test_cases: List of {query, reference_answer, context} dicts
            agent_fn: Function that takes query → response string

        Returns:
            Aggregate evaluation report
        """
        results = []

        for i, test_case in enumerate(test_cases):
            print(f"Evaluating test case {i+1}/{len(test_cases)}...")

            # Get agent response
            response = agent_fn(test_case["query"])

            # Evaluate response
            result = self.evaluate_response(
                query=test_case["query"],
                response=response,
                reference_answer=test_case.get("reference_answer"),
                context=test_case.get("context")
            )
            results.append(result)

        # Aggregate metrics
        passed = sum(1 for r in results if r.passed)
        avg_correctness = sum(r.correctness for r in results) / len(results)
        avg_relevance = sum(r.relevance for r in results) / len(results)
        avg_faithfulness = sum(r.faithfulness for r in results) / len(results)
        avg_overall = sum(r.overall_score for r in results) / len(results)

        # Find problematic cases
        failed_cases = [r for r in results if not r.passed]

        return {
            "total_cases": len(results),
            "passed": passed,
            "pass_rate": f"{passed/len(results):.1%}",
            "avg_correctness": f"{avg_correctness:.2f}",
            "avg_relevance": f"{avg_relevance:.2f}",
            "avg_faithfulness": f"{avg_faithfulness:.2f}",
            "avg_overall": f"{avg_overall:.2f}",
            "failed_cases": [
                {
                    "query": r.query,
                    "score": r.overall_score,
                    "reason": r.reasoning
                }
                for r in failed_cases
            ]
        }

# Example usage
evaluator = AgentEvaluator()

test_cases = [
    {
        "query": "What is the capital of France?",
        "reference_answer": "Paris",
        "context": "France is a country in Western Europe. Its capital and largest city is Paris."
    },
    {
        "query": "Explain how RAG works",
        "reference_answer": "RAG retrieves relevant documents and uses them as context for generation",
        "context": None
    }
]

def my_agent(query: str) -> str:
    client = genai.Client()
    return client.models.generate_content(model='gemini-2.5-flash', contents=query).text

report = evaluator.evaluate_dataset(test_cases, my_agent)
print(f"Pass rate: {report['pass_rate']}")
print(f"Average score: {report['avg_overall']}")
```

## Implementation: Production Monitoring

### Agent Telemetry Logger
```python
import time
import json
import logging
from typing import Optional
from dataclasses import dataclass, asdict

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("agent_monitor")

@dataclass
class AgentTrace:
    trace_id: str
    timestamp: float
    query: str
    response: str
    latency_ms: float
    input_tokens: int
    output_tokens: int
    estimated_cost_usd: float
    error: Optional[str] = None
    model: str = "gemini-2.5-flash"

class AgentMonitor:
    """Production monitoring for agent deployments."""

    def __init__(self, alert_thresholds: dict = None):
        self.traces: list = []
        self.thresholds = alert_thresholds or {
            "max_latency_ms": 5000,
            "min_pass_rate": 0.80,
            "max_error_rate": 0.05,
            "max_cost_per_query": 0.01
        }

    def record_trace(self, trace: AgentTrace):
        """Record an agent interaction for monitoring."""
        self.traces.append(trace)

        # Log to observability system (in production: send to Cloud Logging, DataDog, etc.)
        logger.info(json.dumps({
            "event": "agent_trace",
            **asdict(trace)
        }))

        # Real-time alerting
        self._check_alerts(trace)

    def _check_alerts(self, trace: AgentTrace):
        """Check if this trace violates any alert thresholds."""
        if trace.latency_ms > self.thresholds["max_latency_ms"]:
            logger.warning(f"LATENCY ALERT: {trace.latency_ms:.0f}ms > {self.thresholds['max_latency_ms']}ms")

        if trace.estimated_cost_usd > self.thresholds["max_cost_per_query"]:
            logger.warning(f"COST ALERT: ${trace.estimated_cost_usd:.4f} > ${self.thresholds['max_cost_per_query']}")

        if trace.error:
            logger.error(f"AGENT ERROR: {trace.error[:200]}")

    def get_dashboard_stats(self, last_n: int = 100) -> dict:
        """Get dashboard-ready statistics for the last N traces."""
        recent = self.traces[-last_n:] if len(self.traces) > last_n else self.traces
        if not recent:
            return {"message": "No traces recorded yet"}

        errors = [t for t in recent if t.error]
        avg_latency = sum(t.latency_ms for t in recent) / len(recent)
        p95_latency = sorted(t.latency_ms for t in recent)[int(len(recent) * 0.95)]
        total_cost = sum(t.estimated_cost_usd for t in recent)

        return {
            "traces_analyzed": len(recent),
            "avg_latency_ms": f"{avg_latency:.0f}",
            "p95_latency_ms": f"{p95_latency:.0f}",
            "error_rate": f"{len(errors)/len(recent):.1%}",
            "total_cost_usd": f"${total_cost:.4f}",
            "avg_cost_per_query": f"${total_cost/len(recent):.5f}",
            "total_input_tokens": sum(t.input_tokens for t in recent),
            "total_output_tokens": sum(t.output_tokens for t in recent),
            "status": "healthy" if len(errors)/len(recent) < self.thresholds["max_error_rate"] else "degraded"
        }

# Instrumented agent wrapper
def monitored_agent(query: str, agent_fn, monitor: AgentMonitor) -> str:
    """Wrap any agent function with production monitoring."""
    import uuid
    trace_id = str(uuid.uuid4())[:8]
    start_time = time.time()
    error = None
    response = ""
    input_tokens = len(query) // 4  # Estimate
    output_tokens = 0

    try:
        response = agent_fn(query)
        output_tokens = len(response) // 4  # Estimate
    except Exception as e:
        error = str(e)
        response = "An error occurred. Please try again."

    latency_ms = (time.time() - start_time) * 1000

    trace = AgentTrace(
        trace_id=trace_id,
        timestamp=start_time,
        query=query,
        response=response[:500],  # Truncate for storage
        latency_ms=latency_ms,
        input_tokens=input_tokens,
        output_tokens=output_tokens,
        estimated_cost_usd=(input_tokens / 1000 * 0.00015) + (output_tokens / 1000 * 0.0006),
        error=error
    )

    monitor.record_trace(trace)
    return response
```

## Key Takeaways

- **Golden datasets**: Curate representative test cases with reference answers — these are your ground truth
- **LLM-as-judge**: Use a separate LLM to evaluate responses — scalable and correlates well with human judgment
- **Three core metrics**: Correctness, relevance, faithfulness cover most quality dimensions for RAG+generation agents
- **Monitor in production**: Offline evals aren't enough — instrument every production query
- **Latency + cost + quality**: Don't optimize one metric at the expense of others — track the triad
- **Alert thresholds**: Define what triggers an alert before deployment — not after an incident

## Anti-Patterns to Avoid

- **Evaluating only happy paths**: Test edge cases, ambiguous queries, and adversarial inputs
- **Single-metric optimization**: Maximizing accuracy while ignoring latency/cost leads to unusable systems
- **No production monitoring**: Eval benchmarks ≠ production performance — real traffic behaves differently
- **Ignoring failed evaluations**: Every failed test case is a debugging opportunity — investigate them
- **Self-evaluation**: Don't use the same model to evaluate its own outputs — use a separate judge

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Continuous quality improvement loop | Evaluation + **Reflection** + **Learning & Adaptation** |
| Production monitoring stack | Evaluation + **Exception Handling** + **Resource-Aware** |
| Safe agent with metrics | Evaluation + **Guardrails** + **Human-in-the-Loop** |
| RAG pipeline quality assurance | Evaluation + **RAG** + **Prompt Chaining** |

## References

- RAGAS (RAG Evaluation): https://docs.ragas.io/
- LangSmith Agent Evaluation: https://docs.smith.langchain.com/
- Google Cloud Agent Monitoring: https://cloud.google.com/vertex-ai/docs/generative-ai/model-evaluation/
- DeepEval Framework: https://docs.confident-ai.com/
- Gemini Evaluation Cookbook: https://ai.google.dev/gemini-api/docs/evaluate
