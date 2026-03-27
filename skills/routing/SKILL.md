---
name: routing
description: This skill should be used when the user wants to "route requests", "classify input and delegate", "dynamic decision-making", "conditional agent flow", "dispatch to sub-agents", "intent classification", "triage incoming requests", "agent router", "request dispatcher", "multi-path agent", "semantic routing", "classifier agent", "input-based agent selection", "route to specialist", "smart request routing", or build a system that must choose between multiple specialized tools, agents, or workflows based on context. Also responds to Korean: "요청 라우팅", "입력 분류 후 위임", "동적 의사결정 에이전트", "조건부 에이전트 흐름", "인텐트 분류", "요청 트리아지", "라우팅 만들어줘", "분류 에이전트", "요청 분류", "요청 종류별로 처리해줘", "입력 분류해줘", "어떤 에이전트 쓸지 결정해줘". Also responds to Japanese: "リクエストルーティング", "入力を分類して振り分け", "エージェントルーター", "条件分岐エージェント", "インテント分類", "ルーティング作りたい", "分類エージェント", "動的意思決定", "専門エージェントに転送", "リクエストを振り分けたい" Also responds to Chinese: "请求路由", "输入分类后分发", "智能路由", "条件分支智能体", "意图分类", "请求分发器", "分类路由智能体", "动态决策", "转发给专业智能体", "帮我做请求路由".. Apply this skill to design or implement the Routing agentic design pattern.
version: 1.0.0
---

# Routing Pattern

## Overview

The **Routing Pattern** introduces conditional logic into an agent's operational framework, enabling dynamic decision-making about which specialized function, tool, or sub-agent should handle a given input. Rather than following a fixed linear execution path, a routing agent first analyzes the input to determine its intent or nature, then directs it to the most appropriate handler.

**Core Principle:** Analyze first, then dispatch — transform a static executor into a dynamic, context-aware system.

## When This Skill Applies

Activate this pattern when:
- An agent must decide between multiple distinct workflows, tools, or sub-agents
- Incoming requests vary significantly in type, intent, or required handling
- A system needs to triage or classify inputs before processing
- Different user intents require fundamentally different processing paths
- A static sequential flow cannot handle the variability of real-world inputs

**Rule of thumb:** Use Routing when an agent must intelligently choose the best possible action from a set of options based on input characteristics.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the decision space:
1. What are the distinct categories or intents of incoming requests?
2. What handler, tool, or sub-agent is optimal for each category?
3. What routing mechanism is most appropriate (LLM, embedding, rule-based)?
4. What is the fallback for unclear or uncategorized inputs?

### PLAN
Design the routing architecture:
1. Define the router component (LLM prompt, embedding comparator, or rule engine)
2. Define each destination handler with clear responsibility boundaries
3. Design the routing decision prompt or logic
4. Plan the fallback/clarification pathway for ambiguous inputs
5. Decide routing placement: at entry, mid-chain, or sub-routine selection

### ACTION
Implement the routing system:
1. Build the router that classifies input and outputs a destination identifier
2. Create specialized handlers for each route
3. Wire the router output to the appropriate handler selection logic
4. Test with representative inputs across all categories and edge cases
5. Implement fallback handling and monitoring of routing decisions

## Routing Mechanism Types

### 1. LLM-based Routing
The LLM itself analyzes the input and outputs a category identifier:
```
"Analyze the user query and output ONLY the category:
'Order Status', 'Product Info', 'Technical Support', or 'Other'."
```
- **Pros**: Handles nuanced, novel inputs; understands context and semantics
- **Cons**: Higher latency; less deterministic than rule-based approaches

### 2. Embedding-based Routing
Convert input to vector embedding; compare against embeddings representing each route:
- Route to the destination whose embedding is most semantically similar
- Ideal for semantic routing where meaning matters more than keywords
- **Pros**: Fast, scalable, language-agnostic similarity
- **Cons**: Requires embedding infrastructure; may miss domain-specific nuances

### 3. Rule-based Routing
Predefined if-else statements, switch cases, regex patterns, or keyword matching:
- **Pros**: Deterministic, fast, transparent, easy to audit
- **Cons**: Brittle for novel inputs; requires manual rule maintenance

### 4. ML Model-based Routing
A fine-tuned **discriminative classifier** (e.g., BERT, DistilBERT, or a small classification head on an embedding model) trained on labeled routing examples. Unlike LLM-based routing, inference does not require a generative LLM call — the routing decision is encoded in the model's weights.

**How it works:**
```
Input text
   ↓
Embedding model (e.g., text-embedding-004)
   ↓
Classification head (softmax over N route categories)
   ↓
Route label: "billing" | "support" | "general" | ...
```

**Training data generation workflow:**
1. Define N route categories with descriptions
2. Use an LLM to generate synthetic labeled examples (`gpt-4o`, `gemini-2.5-flash`, etc.)
3. Fine-tune a small discriminative model on the synthetic + real data
4. Deploy the classifier as a lightweight microservice

```python
from sklearn.linear_model import LogisticRegression
from google import genai
from google.genai.types import EmbedContentConfig

# Step 1: Generate embeddings for training examples
client = genai.Client()

training_texts = [
    "I need to cancel my subscription",      # → billing
    "How do I reset my password?",           # → support
    "What is your refund policy?",           # → billing
    "My app keeps crashing on startup",      # → support
    "Tell me about your enterprise plans",   # → sales
]
training_labels = ["billing", "support", "billing", "support", "sales"]

result = client.models.embed_content(
    model="text-embedding-004",
    contents=training_texts,
    config=EmbedContentConfig(task_type="SEMANTIC_SIMILARITY")
)
embeddings = [e.values for e in result.embeddings]

# Step 2: Train a lightweight classifier on embeddings
classifier = LogisticRegression(max_iter=1000)
classifier.fit(embeddings, training_labels)

# Step 3: Route at inference time (no LLM call needed)
def ml_router(user_input: str) -> str:
    result = client.models.embed_content(
        model="text-embedding-004",
        contents=user_input,
        config=EmbedContentConfig(task_type="SEMANTIC_SIMILARITY")
    )
    embedding = result.embeddings[0].values
    return classifier.predict([embedding])[0]

route = ml_router("I was overcharged on my last invoice")
# → "billing"
```

**Comparison with other routing mechanisms:**

| Mechanism | Latency | Cost | Flexibility | Training Data Needed |
|-----------|---------|------|-------------|----------------------|
| Rule-based | Very low | Free | Low | No |
| ML Classifier | Low | Low | Medium | Yes (labels) |
| Embedding similarity | Low | Low | Medium | No |
| LLM-based | Medium | High | High | No |

- **Pros**: Fast inference (milliseconds), low cost, high accuracy for known categories, scales with volume
- **Cons**: Requires labeled training data (LLMs can generate it synthetically); less flexible when new categories emerge

## Practical Use Cases

### Customer Service Triage
```
Input: User query
Router analyzes intent →
  "check order status" → Order Database Sub-Agent
  "product information" → Catalog Search Sub-Agent
  "technical support" → Troubleshooting Chain → Human Escalation
  "unclear" → Clarification Sub-Agent
```

### Document Processing Pipeline
```
Incoming document →
Router classifies by type/format →
  Email → Sales Lead Ingestion Workflow
  Support Ticket → Priority Assessment Chain
  Invoice → JSON Extraction Pipeline
  Legal Document → Compliance Review Agent
```

### Multi-Tool AI Coding Assistant
```
Code snippet + user request →
Router identifies: language + intent →
  Debug request → Debugger Tool
  Explanation request → Documentation Agent
  Translation request → Code Translator
  Optimization request → Performance Analyzer
```

### Research Multi-Agent System
```
Research task →
Router assigns to most suitable agent →
  Web Search Agent (current events)
  Document Retrieval Agent (internal knowledge)
  Data Analysis Agent (quantitative questions)
  Synthesis Agent (once sub-tasks complete)
```

## Implementation with Google ADK

### ADK Auto-Flow Routing (Coordinator + Sub-Agents)
```python
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part
from google.adk.tools import FunctionTool

# Define specialized handler tools
def booking_handler(request: str) -> str:
    """Handles flight and hotel booking requests."""
    return f"Booking processed: {request}"

def info_handler(request: str) -> str:
    """Handles general information requests."""
    return f"Information retrieved for: {request}"

# Create specialized sub-agents
booking_agent = LlmAgent(
    name="Booker",
    model="gemini-2.5-flash",
    description="Handles all flight and hotel booking requests.",
    tools=[FunctionTool(booking_handler)]
)

info_agent = LlmAgent(
    name="Info",
    model="gemini-2.5-flash",
    description="Handles general information questions.",
    tools=[FunctionTool(info_handler)]
)

# Coordinator routes via ADK Auto-Flow (LLM-driven delegation)
coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.5-flash",
    instruction="""You are the main coordinator. Analyze incoming requests
    and delegate to the appropriate specialist agent. Do not answer directly.
    - Booking requests (flights, hotels) → delegate to 'Booker' agent
    - General information questions → delegate to 'Info' agent""",
    description="Routes user requests to the correct specialist agent.",
    sub_agents=[booking_agent, info_agent]
)

session_service = InMemorySessionService()
runner = Runner(agent=coordinator, app_name="routing_app", session_service=session_service)
session_service.create_session(app_name="routing_app", user_id="user1", session_id="session1")
message = Content(parts=[Part(text="Book me a flight to Seoul")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

### LangChain with RunnableBranch
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough, RunnableBranch

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

# Router chain - classifies the intent
router_prompt = ChatPromptTemplate.from_messages([
    ("system", """Analyze the user request. Output ONLY one word:
     'booker' for booking requests, 'info' for information requests,
     'unclear' for ambiguous requests."""),
    ("user", "{request}")
])
router_chain = router_prompt | llm | StrOutputParser()

# Specialized handlers
def booking_handler(x): return f"Booking handled: {x['request']['request']}"
def info_handler(x): return f"Info provided for: {x['request']['request']}"
def unclear_handler(x): return f"Please clarify: {x['request']['request']}"

# Route based on classification
delegation_branch = RunnableBranch(
    (lambda x: x['decision'].strip() == 'booker', RunnablePassthrough.assign(
        output=lambda x: booking_handler(x))),
    (lambda x: x['decision'].strip() == 'info', RunnablePassthrough.assign(
        output=lambda x: info_handler(x))),
    RunnablePassthrough.assign(output=lambda x: unclear_handler(x))
)

# Complete router agent
coordinator_agent = (
    {"decision": router_chain, "request": RunnablePassthrough()}
    | delegation_branch
    | (lambda x: x['output'])
)

# Usage
result = coordinator_agent.invoke({"request": "Book me a flight to London"})
```

## Key Takeaways

- **Dynamic flow**: Routing moves agents beyond static, predetermined execution paths
- **Intent-first design**: Always analyze input intent before selecting the handler
- **Multiple mechanisms**: LLM-based (flexible), embedding-based (semantic), rule-based (fast), or ML classifier (specialized)
- **Placement flexibility**: Apply routing at entry point, mid-chain, or within sub-routines
- **ADK Auto-Flow**: Google ADK's coordinator+sub_agents pattern implements LLM-driven routing automatically
- **LangGraph**: State-based graph structure is ideal for complex multi-step routing scenarios

## Anti-Patterns to Avoid

- **Monolithic routing**: A single router handling too many categories degrades accuracy — use hierarchical routing
- **Missing fallback**: Always implement a fallback for inputs that don't match any category
- **Over-engineering**: Simple keyword matching may outperform LLM routing for structured, predictable inputs
- **No monitoring**: Track routing decisions to detect misclassification patterns and improve the router

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Intelligent multi-agent dispatch | Routing + **Multi-Agent Collaboration** |
| Dynamic tool selection | Routing + **Tool Use** + **Planning** |
| Adaptive workflow orchestration | Routing + **Prompt Chaining** + **Planning** |
| Fault-tolerant routing | Routing + **Exception Handling** + **Human-in-the-Loop** |

## References

- LangGraph Documentation: https://www.langchain.com/
- Google Agent Developer Kit Documentation: https://google.github.io/adk-docs/
- LangChain RunnableBranch: https://python.langchain.com/docs/expression_language/
