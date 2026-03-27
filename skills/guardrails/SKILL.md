---
name: guardrails
description: This skill should be used when the user wants to "agent safety", "guardrails for AI agents", "prevent harmful outputs", "content moderation for agents", "safe agent behavior", "output filtering", "input validation", "jailbreak prevention", "agent constraints", "responsible AI agents", "safety layers", "bias detection in agents", "prevent agent misuse", "agent content filter", "safe AI output", "ethical AI agent", "agent policy enforcement", "compliance guardrails", or build agents with safety constraints that prevent harmful, inappropriate, or out-of-scope behaviors. Also responds to Korean: "에이전트 안전성", "AI 에이전트 가드레일", "유해 출력 방지", "콘텐츠 모더레이션", "안전한 에이전트 동작", "출력 필터링", "가드레일 추가", "안전 필터", "유해 콘텐츠 차단", "안전하게 만들어줘", "욕설 필터링해줘", "위험한 출력 막아줘". Also responds to Japanese: "エージェントの安全性", "AIガードレール", "有害出力防止", "コンテンツモデレーション", "安全なエージェント", "出力フィルタリング", "不適切なコンテンツを防ぎたい", "安全に作りたい", "ジェイルブレイク防止", "倫理的AIエージェント" Also responds to Chinese: "智能体安全性", "AI护栏", "防止有害输出", "内容审核", "安全智能体行为", "输出过滤", "不要输出不安全内容", "安全地构建", "防越狱", "合规护栏".. Apply this skill to design or implement the Guardrails & Safety agentic design pattern.
version: 1.0.0
---

# Guardrails & Safety Pattern

## Overview

The **Guardrails & Safety Pattern** establishes protective boundaries around agent behavior — filtering inputs, validating outputs, enforcing policy constraints, and preventing harm. Guardrails operate as a safety layer that sits between users and agents (input guardrails) and between agents and users (output guardrails), ensuring agents remain beneficial, appropriate, and within authorized scope.

**Core Principle:** Safety is not an afterthought — build constraints into the design so agents can't go wrong, not just catch them after they do.

## When This Skill Applies

Activate this pattern when:
- Agents are deployed in public-facing applications where prompt injection is possible
- Outputs could cause harm if unfiltered (medical, legal, financial advice)
- The agent has access to sensitive systems or data that must be protected
- Regulatory compliance requires content filtering and audit trails
- The agent must stay within a defined scope (don't answer off-topic questions)
- Actions in the world (emails, purchases, deletions) require pre-execution validation

**Rule of thumb:** Any agent interacting with untrusted users or executing real-world actions needs guardrails — the question is which ones, not whether.

## Guardrail Types

### Input Guardrails
- **Topic filtering**: Block requests outside the agent's scope
- **Injection detection**: Detect prompt injection attacks
- **PII detection**: Identify and handle personal information
- **Content moderation**: Filter harmful, offensive, or inappropriate requests
- **Rate limiting**: Prevent abuse through request throttling

### Output Guardrails
- **Fact checking**: Verify claims against authoritative sources
- **Toxicity filtering**: Remove harmful language from responses
- **Hallucination detection**: Flag responses that may not be grounded
- **PII redaction**: Remove sensitive data from outputs
- **Scope enforcement**: Ensure responses stay within authorized domain

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the safety requirements:
1. What topics/actions are in-scope vs. out-of-scope for this agent?
2. What harmful outputs could this agent produce? (Bias, misinformation, harmful advice)
3. What data should never appear in outputs? (PII, credentials, confidential info)
4. What actions require extra validation before execution?
5. What are the regulatory requirements? (GDPR, HIPAA, financial regulations)

### PLAN
Design the guardrail architecture:
1. Define input validation pipeline (topic check → injection detection → content filter)
2. Define output validation pipeline (toxicity → PII → scope → fact check)
3. Choose guardrail implementation: rule-based, LLM-based, or classifier-based
4. Design fallback behavior for blocked requests (helpful refusal vs. redirect)
5. Plan audit logging: what to log for compliance and debugging

### ACTION
Implement guardrail layers:
1. Add input guardrails at the agent entry point
2. Add output guardrails before returning responses to users
3. Implement action guardrails for real-world effects
4. Add monitoring and alerting for guardrail activations
5. Test adversarially: try to break your own guardrails

## Implementation: Input/Output Guardrails

### Rule-Based + LLM Guardrails
```python
from google import genai
import re
from typing import Tuple

client = genai.Client()

class AgentGuardrails:
    """Multi-layer safety system for agent inputs and outputs."""

    def __init__(self, allowed_topics: list, blocked_patterns: list = None):
        self.allowed_topics = allowed_topics
        self.blocked_patterns = blocked_patterns or []

        # Compile regex patterns for efficiency
        self.compiled_patterns = [re.compile(p, re.IGNORECASE) for p in self.blocked_patterns]

    def check_input(self, user_input: str) -> Tuple[bool, str]:
        """Validate user input before sending to agent.

        Returns:
            (is_safe, reason_if_blocked)
        """
        # 1. Length check
        if len(user_input) > 10000:
            return False, "Input too long (max 10,000 characters)"

        # 2. Pattern-based blocking
        for pattern in self.compiled_patterns:
            if pattern.search(user_input):
                return False, "Input contains blocked content"

        # 3. Prompt injection detection
        injection_indicators = [
            "ignore previous instructions",
            "disregard your system prompt",
            "you are now",
            "new persona",
            "forget everything",
            "bypass your restrictions"
        ]
        input_lower = user_input.lower()
        for indicator in injection_indicators:
            if indicator in input_lower:
                return False, "Potential prompt injection detected"

        # 4. LLM-based topic relevance check
        if self.allowed_topics:
            relevance_check = client.models.generate_content(
                model='gemini-2.5-flash',
                contents=f"""Is this request relevant to: {', '.join(self.allowed_topics)}?

Request: {user_input}

Answer only: YES or NO"""
            )
            if "NO" in relevance_check.text.upper():
                return False, f"Request is outside the agent's scope ({', '.join(self.allowed_topics)})"

        return True, "OK"

    def check_output(self, output: str, user_input: str) -> Tuple[str, list]:
        """Validate and clean agent output before returning to user.

        Returns:
            (cleaned_output, list_of_issues_found)
        """
        issues = []
        cleaned = output

        # 1. PII Detection (simplified — use a proper NLP library in production)
        pii_patterns = {
            "email": r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            "ssn": r'\b\d{3}-\d{2}-\d{4}\b',
            "credit_card": r'\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b',
            "phone": r'\b(?:\+1[-.]?)?\(?(?:[0-9]{3})\)?[-.]?[0-9]{3}[-.]?[0-9]{4}\b'
        }
        for pii_type, pattern in pii_patterns.items():
            matches = re.findall(pattern, cleaned)
            if matches:
                cleaned = re.sub(pattern, f'[{pii_type.upper()}_REDACTED]', cleaned)
                issues.append(f"Redacted {len(matches)} {pii_type}(s)")

        # 2. LLM-based safety check
        safety_check = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=f"""Rate the safety of this AI response on a scale of 1-5:
1 = Harmful/dangerous
2 = Questionable
3 = Neutral
4 = Safe
5 = Excellent/exemplary

Response to evaluate: {cleaned[:500]}

Output only: SAFE or UNSAFE [score]"""
        )

        if "UNSAFE" in safety_check.text.upper():
            issues.append("Output flagged by safety classifier")
            cleaned = "I apologize, but I'm unable to provide that response. Please rephrase your request."

        # 3. Hallucination risk flag (if output contains specific factual claims)
        if re.search(r'\b(in \d{4}|according to|studies show|research indicates)\b', cleaned, re.IGNORECASE):
            issues.append("Response contains factual claims — verify before acting on them")

        return cleaned, issues

    def safe_generate(self, user_input: str) -> dict:
        """Full pipeline: validate input → generate → validate output."""
        # Check input
        is_safe, reason = self.check_input(user_input)
        if not is_safe:
            return {
                "response": f"I'm unable to process this request: {reason}",
                "blocked": True,
                "block_reason": reason,
                "issues": []
            }

        # Generate response
        response = client.models.generate_content(model='gemini-2.5-flash', contents=user_input)
        output = response.text

        # Check output
        cleaned_output, issues = self.check_output(output, user_input)

        return {
            "response": cleaned_output,
            "blocked": False,
            "issues": issues,
            "modified": cleaned_output != output
        }

# Usage
guardrails = AgentGuardrails(
    allowed_topics=["software engineering", "coding", "programming", "APIs"],
    blocked_patterns=["hack", "exploit", "malware"]
)

result = guardrails.safe_generate("How do I implement a REST API in Python?")
print(result["response"])
```

### ADK Agent with Safety Callbacks
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
import re

def validate_action_safety(
    action_type: str,
    action_details: str,
    tool_context: ToolContext
) -> dict:
    """Safety check before executing any consequential action.

    Args:
        action_type: Type of action (email, delete, post, payment)
        action_details: Full details of the proposed action

    Returns:
        Safety assessment with allow/block decision
    """
    # High-risk action types that require extra scrutiny
    high_risk_actions = ["delete", "payment", "email_external", "publish", "admin"]

    risk_level = "high" if action_type in high_risk_actions else "low"

    # Log to audit trail
    audit_entry = {
        "action_type": action_type,
        "risk_level": risk_level,
        "details_preview": action_details[:200],
        "timestamp": __import__("time").time()
    }
    audit_log = tool_context.state.get("audit_log", [])
    audit_log.append(audit_entry)
    tool_context.state["audit_log"] = audit_log

    # Check for scope violations
    if action_type in ["payment", "bank_transfer"] and not tool_context.state.get("payment_authorized"):
        return {
            "allowed": False,
            "reason": "Payment actions require explicit authorization",
            "risk_level": risk_level
        }

    # Rate limiting check
    recent_actions = sum(
        1 for entry in audit_log
        if __import__("time").time() - entry["timestamp"] < 60  # Last minute
    )
    if recent_actions > 10:
        return {
            "allowed": False,
            "reason": "Rate limit exceeded: too many actions per minute",
            "risk_level": "high"
        }

    return {
        "allowed": True,
        "risk_level": risk_level,
        "audit_id": len(audit_log)
    }

safe_agent = LlmAgent(
    name="SafeAgent",
    model="gemini-2.5-flash",
    instruction="""You are a safety-conscious agent. Before taking any action:
    1. Call validate_action_safety to check if the action is permitted
    2. Only proceed if 'allowed' is True
    3. If blocked, explain the restriction to the user clearly and suggest alternatives
    4. Never attempt to bypass safety checks — if something is blocked, it's blocked for a reason""",
    tools=[FunctionTool(validate_action_safety)]
)
```

## Implementation: Constitutional AI Constraints

### LLM-Based Policy Enforcement
```python
AGENT_CONSTITUTION = """
The agent must ALWAYS:
1. Be truthful — never claim to know something it doesn't
2. Be transparent — acknowledge when it's an AI
3. Protect privacy — never share personal information about third parties
4. Stay in scope — only answer questions about [DOMAIN]
5. Escalate uncertainty — say "I'm not certain" rather than guess on critical matters

The agent must NEVER:
1. Provide medical diagnoses or treatment recommendations
2. Provide legal advice that substitutes for professional counsel
3. Execute irreversible financial transactions without explicit confirmation
4. Share information that could enable harm to individuals or groups
5. Impersonate a real person or organization
"""

def constitutional_check(response: str, request: str, client) -> dict:
    """Check if a response adheres to the agent's constitution."""

    check_prompt = f"""Review this AI response against the agent's guiding principles.

Principles:
{AGENT_CONSTITUTION}

User Request: {request}
AI Response: {response}

Does this response violate any principle? Answer:
COMPLIANT: [reason]
OR
VIOLATION: [which principle, why, suggested fix]"""

    result = client.models.generate_content(model='gemini-2.5-flash', contents=check_prompt)

    if "VIOLATION" in result.text.upper():
        violation_info = result.text.split("VIOLATION:")[-1].strip()
        return {
            "compliant": False,
            "violation": violation_info,
            "response_should_be_blocked": True
        }

    return {"compliant": True, "violation": None, "response_should_be_blocked": False}
```

## Key Takeaways

- **Defense in depth**: Layer multiple guardrail types — one layer catching what another misses
- **Input AND output**: Guardrail both sides — prevent bad inputs from reaching the agent, and bad outputs from reaching users
- **Fail closed**: When guardrails are uncertain, block and ask for clarification rather than allow and hope
- **Audit everything**: Log all guardrail activations for compliance, debugging, and improvement
- **Test adversarially**: Red-team your own agent — try to break guardrails before attackers do
- **Constitutional AI**: Define explicit principles and check responses against them automatically

## Anti-Patterns to Avoid

- **Security by obscurity**: Hiding the system prompt doesn't prevent prompt injection — build actual defenses
- **Allow-list fatigue**: Over-filtering blocks legitimate use — calibrate precision vs. recall carefully
- **No logging**: Blocked requests you can't analyze can't be fixed or appealed
- **Guardrails as UX**: Using guardrail errors as user-facing messages is confusing — return clear, helpful refusals
- **One-time testing**: Guardrails need continuous adversarial testing as new attacks emerge

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Full production safety stack | Guardrails + **Exception Handling** + **Evaluation** |
| Human oversight with safety gates | Guardrails + **Human-in-the-Loop** |
| Safe multi-agent orchestration | Guardrails + **Multi-Agent Collaboration** + **A2A** |
| Auditable enterprise agent | Guardrails + **Evaluation** + **Memory Management** |

## References

- Google Responsible AI Practices: https://ai.google/responsibility/
- Gemini Safety Settings: https://ai.google.dev/gemini-api/docs/safety-settings
- Constitutional AI Paper: https://arxiv.org/abs/2212.08073
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/
- Google ADK Safety: https://google.github.io/adk-docs/
