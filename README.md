# Agentic Design Patterns — Skills

A comprehensive skill library for building AI agents using proven agentic design patterns. Each skill follows the structured **DEFINE → PLAN → ACTION** workflow and is compatible with both **Gemini CLI / Antigravity** and **Claude Code**.

## Overview

This library contains **28 skills** covering the full spectrum of agentic design patterns — from foundational patterns (Prompt Chaining, Routing, Parallelization) to advanced patterns (A2A Communication, Resource-Aware Optimization, Exploration & Discovery) and appendix skills covering tools, frameworks, and reasoning engines.

All skills are implemented with:
- **Google ADK** (Agent Developer Kit) as the primary framework
- **LangChain / LangGraph** as secondary implementations
- **CrewAI** for multi-agent collaboration patterns
- **Gemini API** (`gemini-2.5-flash`) as the default LLM
- **Multilingual triggers** (English / Korean / Japanese / Chinese) for auto-activation across language preferences

## Platform Compatibility

This skill library works with both AI CLI platforms without any content modification.

| Feature | Gemini CLI | Antigravity | Claude Code |
|---------|------------|-------------|-------------|
| **Workspace skill path** | `.gemini/skills/<name>/` or `.agents/skills/<name>/` | `.agents/skills/<name>/` | `.claude/skills/<name>/` |
| **User skill path** | `~/.gemini/skills/<name>/` or `~/.agents/skills/<name>/` | `~/.agents/skills/<name>/` | `~/.claude/skills/<name>/` |
| **Auto-activation** | Semantic: model reads description and decides autonomously | Description keyword pattern matching | Semantic judgment (language-agnostic) |
| **Context loading** | Progressive disclosure — only name+description loaded until activated | Full SKILL.md loaded on match | Full SKILL.md loaded on activation |
| **Manual invocation** | `/skills link`, `gemini skills install` | `@skills/<name>/SKILL.md` reference | `/skill-name` slash command |
| **`name:` field** | Unique skill identifier | Trigger identifier | Registered as `/slash-command` name |
| **`description:` field** | When to activate (semantic match) | Trigger phrase list (keyword match) | Basis for semantic activation |
| **`version:` field** | Used | Used | Ignored silently (no error) |
| **Content compatibility** | ✅ Fully compatible | ✅ Fully compatible | ✅ Fully compatible |

> **Key insight**: Skill file content is identical across all platforms. The differences are only in **registration path**, **activation mechanism**, and **context loading strategy**.

> **Path note**: `.agents/skills/` is the cross-tool generic alias officially supported by Gemini CLI. Antigravity uses `.agents/skills/` as its standard path. Within the same scope, `.agents/skills/` takes precedence over `.gemini/skills/`.

## Quick Start

### Step 1 — Install Dependencies

```bash
# Install Python packages
pip install google-genai google-adk langchain langchain-google-genai
pip install langgraph crewai chromadb fastapi uvicorn
pip install langchain-chroma langchain-text-splitters

# Set your API key
export GOOGLE_API_KEY="your-api-key-here"
```

### Step 2 — Platform Setup

#### Gemini CLI

Gemini CLI manages skills with the `gemini skills` command and discovers them from two scopes:

**Option A — User-level install (available in all projects):**
```bash
# Link all 28 skills at once (creates symlinks in ~/.gemini/skills/)
gemini skills link /path/to/agentic-design-patterns-skills/skills

# Verify discovery
gemini skills list
```

**Option B — Workspace install (current project only):**
```bash
# Link to the project scope (.gemini/skills/ or .agents/skills/)
gemini skills link /path/to/agentic-design-patterns-skills/skills --scope workspace

# Or create symlinks manually
mkdir -p .gemini/skills
ln -s /path/to/agentic-design-patterns-skills/skills/* .gemini/skills/
```

**Option C — Install specific skills from Git:**
```bash
# Install individual skills by subdirectory
gemini skills install https://github.com/your-org/agentic-design-patterns-skills.git --path skills/prompt-chaining
gemini skills install https://github.com/your-org/agentic-design-patterns-skills.git --path skills/planning
```

**Option D — Install all skills from Git (workspace scope):**
```bash
gemini skills install https://github.com/your-org/agentic-design-patterns-skills.git --scope workspace
```

**Verify and manage skills in an interactive session:**
```
/skills list              → show all discovered skills and their status
/skills reload            → refresh after adding new skills
/skills disable planning  → temporarily disable a skill
/skills enable planning   → re-enable a disabled skill
```

#### Antigravity

```bash
# Clone the repository
git clone https://github.com/your-org/agentic-design-patterns-skills.git

# Option A — Workspace install (.agents/skills/ — cross-tool standard path)
mkdir -p .agents/skills
ln -s /path/to/agentic-design-patterns-skills/skills/* .agents/skills/

# Option B — User-level install (~/.agents/skills/)
mkdir -p ~/.agents/skills
ln -s /path/to/agentic-design-patterns-skills/skills/* ~/.agents/skills/

# Verify by typing a trigger phrase; the matching skill activates:
#   "Build a multi-step agent pipeline"   → Prompt Chaining
#   "에이전트 병렬 실행하고 싶어"          → Parallelization
```

#### Claude Code

Claude Code reads skills from `.claude/skills/`. Choose the installation scope that fits your workflow.

**Option A — Global install (available in all projects):**
```bash
# Symlink all 28 skills at once
ln -s "$(pwd)/skills/"* ~/.claude/skills/

# After this, type / in Claude Code to see all registered slash commands:
#   /prompt-chaining, /planning, /reflection, /tool-use ...
```

**Option B — Project-level install (current project only):**
```bash
# Run from the project root
mkdir -p .claude/skills
ln -s "$(pwd)/skills/"* .claude/skills/
```

**Option C — Selective install (specific skills only):**
```bash
# Register only the skills you need
ln -s "$(pwd)/skills/prompt-chaining" ~/.claude/skills/
ln -s "$(pwd)/skills/planning"        ~/.claude/skills/
ln -s "$(pwd)/skills/reflection"      ~/.claude/skills/
```

**Option D — No install, direct file reference:**
```
# Reference any skill inline without installation
@skills/prompt-chaining/SKILL.md Please design a pipeline using this skill.
```

**Optional: Claude Code-specific frontmatter fields**

The base format works as-is. To leverage additional Claude Code features, extend the frontmatter:

```yaml
---
name: planning
description: "복잡한 작업 계획", "plan complex tasks", ...
version: 1.0.0
# Claude Code-only fields below (ignored by Antigravity)
context: fork                              # Run in an isolated subagent
allowed-tools: Read, Grep, Bash(python *)  # Tools usable without a permission prompt
argument-hint: "[goal description]"        # Autocomplete hint: /planning [goal]
---
```

## Agent Complexity Levels

Every skill maps to one of four complexity levels. Use this framework to select the right patterns for your agent's scope:

| Level | Name | Characteristics | Key Skills |
|-------|------|-----------------|------------|
| **Level 0** | Core Reasoning Engine | Single-model calls, sequential decomposition, no external tools | Prompt Chaining, Reflection |
| **Level 1** | Connected Problem-Solver | Tool use, external APIs, RAG, memory integration | Tool Use, RAG, Memory Management, MCP |
| **Level 2** | Strategic Problem-Solver | Multi-step planning, adaptive decision-making, self-correction | Planning, Routing, Guardrails, Evaluation |
| **Level 3** | Collaborative Multi-Agent | Multiple specialized agents, inter-agent communication protocols | Multi-Agent Collaboration, A2A, Parallelization |

> **Start at Level 0, add complexity only when needed.** Most production agents operate at Level 1–2.

## Skill Directory

### Part One: Core Patterns (Chapters 1–7)

| Skill | Level | Description | When to Use |
|-------|-------|-------------|-------------|
| [Prompt Chaining](skills/prompt-chaining/SKILL.md) | 0 | Sequential decomposition of complex tasks into LLM call chains | Multi-step pipelines with structured output requirements |
| [Routing](skills/routing/SKILL.md) | 2 | Classify and direct requests to specialized handlers | When different inputs need different processing paths |
| [Parallelization](skills/parallelization/SKILL.md) | 3 | Fan-out concurrent execution with fan-in aggregation | Independent sub-tasks that can run simultaneously |
| [Reflection](skills/reflection/SKILL.md) | 0 | Generate → Critique → Refine iterative improvement loops | When output quality must be validated and improved |
| [Tool Use](skills/tool-use/SKILL.md) | 1 | Extend agents with external APIs, search, code execution | When agents need real-world information or actions |
| [Planning](skills/planning/SKILL.md) | 2 | Decompose high-level goals into executable step sequences | Complex tasks where solution path must be discovered |
| [Multi-Agent Collaboration](skills/multi-agent-collaboration/SKILL.md) | 3 | Specialist agent teams with structured collaboration | Tasks spanning multiple domains or requiring peer review |

### Part Two: State Management (Chapters 8–11)

| Skill | Level | Description | When to Use |
|-------|-------|-------------|-------------|
| [Memory Management](skills/memory-management/SKILL.md) | 1 | Short-term context + long-term persistent knowledge | Agents that must remember across turns and sessions |
| [Learning & Adaptation](skills/learning-adaptation/SKILL.md) | 2 | Few-shot, online learning, and self-improving agents | Agents that must improve from experience and feedback |
| [MCP (Model Context Protocol)](skills/mcp/SKILL.md) | 1 | Standardized protocol for tool and resource integration | Reusable, interoperable tool servers for multiple agents |
| [Goal Setting](skills/goal-setting/SKILL.md) | 2 | Define, track, and adapt toward measurable objectives | Long-running agents with verifiable success criteria |

### Part Three: Reliability (Chapters 12–14)

| Skill | Level | Description | When to Use |
|-------|-------|-------------|-------------|
| [Exception Handling](skills/exception-handling/SKILL.md) | 1 | Retry, fallback, circuit breaker, graceful degradation | Any agent calling external APIs or services |
| [Human-in-the-Loop](skills/human-in-the-loop/SKILL.md) | 2 | Human approval gates for high-risk or irreversible actions | Agents executing consequential real-world actions |
| [RAG (Retrieval-Augmented Generation)](skills/rag/SKILL.md) | 1 | Ground responses in retrieved external knowledge | Domain-specific accuracy, citation requirements |

### Part Four: Advanced Patterns (Chapters 15–21)

| Skill | Level | Description | When to Use |
|-------|-------|-------------|-------------|
| [A2A Communication](skills/a2a/SKILL.md) | 3 | Standardized agent-to-agent discovery and task delegation | Cross-boundary agent interoperability and federation |
| [Resource-Aware Optimization](skills/resource-aware/SKILL.md) | 2 | Token budgets, model cascades, caching, cost control | Production agents with cost/latency constraints |
| [Reasoning Techniques](skills/reasoning/SKILL.md) | 0 | Chain-of-Thought, ReAct, Tree of Thought, self-consistency | Complex problems requiring explicit, traceable reasoning |
| [Guardrails & Safety](skills/guardrails/SKILL.md) | 2 | Input/output filtering, safety layers, constitutional AI | Any public-facing or high-stakes agent deployment |
| [Evaluation & Monitoring](skills/evaluation/SKILL.md) | 2 | Quality metrics, LLM-as-judge, production telemetry | All production agents — measure quality continuously |
| [Prioritization](skills/prioritization/SKILL.md) | 2 | Priority queues, dynamic task scheduling, deadline-awareness | Agents handling multiple concurrent tasks |
| [Exploration & Discovery](skills/exploration/SKILL.md) | 2 | ε-greedy, UCB, A/B testing for strategy discovery | Agents that must find better approaches over time |

### Appendix Skills (Supplementary)

| Skill | Description | When to Use |
|-------|-------------|-------------|
| [Prompt Engineering](skills/appendix-prompt-engineering/SKILL.md) | Zero/one/few-shot, CoT, structured output with Pydantic, prompt versioning | Any agent requiring reliable, structured LLM outputs |
| [GUI & Real-World Agents](skills/appendix-gui-agents/SKILL.md) | ACI pipeline, Computer Use, Browser Use, Project Astra, multimodal agents | Automating systems with no API via visual interfaces |
| [Agentic Frameworks](skills/appendix-agentic-frameworks/SKILL.md) | LangChain vs LangGraph vs ADK vs CrewAI vs AutoGen — when to use each | Selecting the right framework for a new agent project |
| [Google AgentSpace](skills/appendix-agentspace/SKILL.md) | No-code agent builder, Prompt Gallery, Knowledge Graph, Agent Designer | Enterprise teams deploying agents without writing code |
| [AI CLI Agents](skills/appendix-ai-cli/SKILL.md) | Claude Code, Gemini CLI, Aider, GitHub Copilot CLI, Terminal-Bench | AI-assisted software development from the terminal |
| [Coding Agent Teams](skills/appendix-coding-agents/SKILL.md) | Vibe Coding, Human-Agent Teams, Scaffolder/Test/Documenter/Optimizer agents | Organizing AI agents as specialists in a dev lifecycle |
| [Reasoning Engines ⭐ NEW](skills/appendix-reasoning-engines/SKILL.md) | Thinking tokens, standard vs. reasoning models, inference-time compute scaling, hybrid routing | Choosing between `gemini-2.5-flash` (high Thinking Budget) and `gemini-2.5-pro` for accuracy-critical tasks |

## Skill File Structure

Each skill follows a consistent structure:

```
skills/<pattern-name>/
└── SKILL.md          # Complete skill definition
```

Every `SKILL.md` contains:

```markdown
---
name: <skill-name>
description: <EN + KR + JA + ZH trigger phrases for multilingual auto-activation>
version: 1.0.0
---

# Pattern Name

## Overview
## Context Engineering        ← foundational concept (Prompt Chaining skill)
## Agent Complexity Levels    ← 4-level framework reference (Prompt Chaining skill)
## When This Skill Applies
## DEFINE → PLAN → ACTION Workflow
## Core Concepts / Patterns
## Practical Use Cases
## Implementation (ADK + LangGraph/LangChain)
## Key Takeaways
## Anti-Patterns to Avoid
## Related Skills              ← skill combination guide (all skills)
## References
```

## Platform Usage

### Gemini CLI

Skills are discovered from `.gemini/skills/` or `.agents/skills/`. Gemini CLI reads only the `name` and `description` from each SKILL.md initially (**progressive disclosure**) — the full instructions are pulled in only when the model decides to activate the skill via the `activate_skill` tool.

**How auto-activation works:**
```
User request → Gemini scans name+description of all discovered skills
             → Model semantically matches request to relevant skills
             → activate_skill tool loads the full SKILL.md into context
             → Agent executes with full skill instructions
```

**English trigger examples:**
```
"Build an agent that searches the web and writes a report"
→ activates Prompt Chaining + Tool Use

"Create a multi-agent system for code review"
→ activates Multi-Agent Collaboration + Reflection

"How do I prevent my agent from hallucinating?"
→ activates RAG + Guardrails
```

**Korean trigger examples:**
```
"프롬프트 체이닝으로 리서치 파이프라인 만들어줘" → Prompt Chaining
"에이전트 팀 구성하는 방법 알려줘"              → Multi-Agent Collaboration
"병렬 작업 실행하는 에이전트 어떻게 만들어?"    → Parallelization
"에이전트 오류 처리 어떻게 해?"                → Exception Handling
"모델 컨텍스트 프로토콜 써보고 싶어"            → MCP
```

**Skill management commands:**
```bash
gemini skills list                    # List all discovered skills and status
gemini skills link /path/to/skills    # Link skills directory (user scope)
gemini skills install <url> --path skills/prompt-chaining  # Install from Git
gemini skills uninstall prompt-chaining                    # Uninstall a skill
```

**Manual reference (no install needed):**
```bash
# Explicitly load a specific skill inline
@skills/reflection/SKILL.md Please implement a self-evaluation loop using this skill.
```

---

### Antigravity

Skills auto-activate when the user's input matches trigger phrases in the `description` field via **keyword pattern matching**. The full SKILL.md is loaded when a match is found.

**Skill discovery paths:**
```
.agents/skills/<name>/SKILL.md      # Workspace-level (project)
~/.agents/skills/<name>/SKILL.md    # User-level (all projects)
```

**Auto-activation examples:**
```
"Build an agent that searches the web and writes a report"
→ description keyword match → activates Prompt Chaining + Tool Use

"에이전트 병렬 실행하고 싶어"
→ Korean keyword match → activates Parallelization
```

**Manual reference:**
```bash
@skills/reflection/SKILL.md Please implement a self-evaluation loop using this skill.
```

---

### Claude Code

After installation, skills are invocable as **slash commands** or auto-activated during conversation via **semantic judgment** — more flexible than keyword matching, activating on intent rather than exact phrases.

**Slash command invocation:**
```
/prompt-chaining    → loads and runs the Prompt Chaining skill
/planning           → loads and runs the Planning skill
/multi-agent-collaboration → loads and runs the Multi-Agent Collaboration skill
```

**Auto-activation examples:**
```
"Build me an agent that searches the web and writes a report"
→ Claude selects prompt-chaining + tool-use based on description semantics

"I need agents to work in parallel on different research topics"
→ Claude activates parallelization automatically

"에이전트 병렬 실행하고 싶어"
→ Korean intent matched semantically — activates parallelization
```

**Direct file reference (no installation needed):**
```
@skills/guardrails/SKILL.md Design a safe agent using this skill.
```

**Browse all registered skills:**
```
# Type / in Claude Code to see the autocomplete list of all 28 registered slash commands
/p → /planning, /parallelization, /prompt-chaining, /prioritization ...
```

## Multilingual Trigger System

Every skill contains trigger phrases in **four languages** — English, Korean, Japanese, and Simplified Chinese. The AI platform reads these phrases from the `description:` field in each `SKILL.md` and activates the matching skill automatically.

### How Triggers Are Structured

Each `SKILL.md` description field follows this format:

```
"EN trigger 1", "EN trigger 2", ..., or [prose]. Also responds to Korean: "KR1", "KR2", .... Also responds to Japanese: "JA1", "JA2", .... Also responds to Chinese: "ZH1", "ZH2", .... Apply this skill to ...
```

**Example — Planning skill:**
```yaml
description: >
  "plan complex tasks", "task decomposition", "step-by-step agent plan", ...
  Also responds to Korean: "복잡한 작업 계획", "단계별 실행 계획", "목표 분해해줘", ...
  Also responds to Japanese: "複雑なタスクの計画", "ステップバイステップの計画", "目標を分解して", ...
  Also responds to Chinese: "复杂任务规划", "分步执行计划", "帮我分解目标", ...
  Apply this skill to design or implement the Planning agentic design pattern.
```

### Trigger Coverage

| Language | Triggers | Style |
|----------|--------:|-------|
| English | 474 | Technical terms + natural requests |
| Korean | 337 | 기술 용어 + 구어체 ("~해줘", "~어떻게 해?") |
| Japanese | 284 | 技術用語 + 口語体 ("~したい", "~教えて") |
| Chinese | 281 | 技术术语 + 口语化 ("帮我~", "怎么~") |
| **Total** | **1,376** | Across 28 skills |

### English Trigger Examples

```
"Build a multi-step agent pipeline"          → Prompt Chaining
"Create a multi-agent system for code review" → Multi-Agent Collaboration
"How do I prevent my agent from hallucinating?" → RAG + Guardrails
"retry logic for API failures"               → Exception Handling
"set up MCP server"                          → MCP
"semantic search over documents"             → RAG
"choose agent framework"                     → Agentic Frameworks
"thinking model for complex reasoning"       → Reasoning Engines
```

### Korean Trigger Examples (한국어 트리거)

한국어는 기술 용어뿐만 아니라 실제로 사람들이 사용하는 구어체 표현으로도 활성화됩니다.

```
"프롬프트 체이닝으로 파이프라인 만들어줘"   → Prompt Chaining
"에이전트 팀 구성하는 방법 알려줘"          → Multi-Agent Collaboration
"병렬 작업 실행하는 에이전트 어떻게 만들어?" → Parallelization
"MCP 구성을 해줘"                           → MCP
"모델 컨텍스트 프로토콜 써보고 싶어"         → MCP
"메모리뱅크 만들어줘"                       → Memory Management
"컨텍스트 유지할 수 있는 에이전트"           → Memory Management
"에이전트 오류 처리 어떻게 해?"             → Exception Handling
"RAG 파이프라인 구성해줘"                   → RAG
"추론 모델 언제 써야 해?"                   → Reasoning Engines
"어떤 프레임워크 써야 해"                   → Agentic Frameworks
```

### Japanese Trigger Examples (日本語トリガー)

```
"マルチエージェントを構築したい"             → Multi-Agent Collaboration
"並列処理するエージェントを作って"           → Parallelization
"MCPサーバーを設定したい"                   → MCP
"エージェントにメモリを持たせたい"           → Memory Management
"APIのエラーハンドリングを実装して"          → Exception Handling
"RAGパイプラインを作りたい"                 → RAG
"推論モデルの使い方を教えて"                → Reasoning Engines
"どのフレームワークを使うべきか"             → Agentic Frameworks
"複雑な問題を深く考えるモデルを使いたい"     → Reasoning Engines
```

### Chinese Trigger Examples (中文触发词)

```
"帮我构建多智能体系统"                      → Multi-Agent Collaboration
"怎么让智能体并行处理任务"                  → Parallelization
"配置MCP服务器"                            → MCP
"帮我记住对话内容"                          → Memory Management
"智能体API调用失败怎么处理"                 → Exception Handling
"搭建RAG知识库问答系统"                    → RAG
"该用哪个框架构建智能体"                    → Agentic Frameworks
"推理模型和普通模型有什么区别"              → Reasoning Engines
"复杂任务规划和分解"                       → Planning
```

### Platform Behavior by Language

| Platform | Language Handling |
|----------|------------------|
| **Claude Code** | Semantic — understands intent in any language; all 4 languages work equally |
| **Gemini CLI** | Semantic — autonomously matches description to user request across languages |
| **Antigravity** | Keyword matching — trigger phrase must appear as a substring in the user's message |

> **Antigravity note**: Because Antigravity uses substring matching, conversational phrases matter. `"MCP 구성을 해줘"` works because `"MCP 구성"` is included as a trigger. Add custom triggers to the `description:` field if your team uses different terminology.

---

## Skill Combination Guide

Each skill's `## Related Skills` section shows concrete combination scenarios. Common production stacks:

### Autonomous Research Agent
```
Planning → Tool Use → Reflection → Evaluation
```
_Plan a research strategy → execute with web/db tools → self-critique → measure quality_

### Production-Ready Safe Agent
```
Guardrails → Exception Handling → Evaluation → Human-in-the-Loop
```
_Filter inputs → handle failures gracefully → monitor quality → escalate edge cases_

### Enterprise Knowledge Agent
```
RAG → Memory Management → Tool Use → Resource-Aware
```
_Retrieve relevant docs → maintain session context → call external APIs → control costs_

### Scalable Multi-Agent System
```
Multi-Agent Collaboration → A2A → Parallelization → Prioritization
```
_Specialist teams → federated communication → concurrent execution → priority scheduling_

### Accuracy-Critical Deep Reasoning
```
Reasoning Engines → Planning → Reflection → Evaluation
```
_Use thinking model → deliberate planning → self-critique → automated quality assessment_

## Pattern Selection Guide

```
Is the task complex and multi-step?
├── Yes, path is known → Prompt Chaining
└── Yes, path must be discovered → Planning

Does the task need external information?
├── Yes + Real-time → Tool Use
├── Yes + Documents/KB → RAG
└── No → Proceed without retrieval

Are there multiple types of input/task?
└── Yes → Routing

Can sub-tasks run in parallel?
└── Yes → Parallelization

Does output quality need verification?
└── Yes → Reflection + Evaluation

Does the task span multiple domains?
└── Yes → Multi-Agent Collaboration

Must agents communicate across systems?
└── Yes → A2A + MCP

Must the agent remember across sessions?
└── Yes → Memory Management

Must the agent improve over time?
└── Yes → Learning & Adaptation

Are actions irreversible or high-risk?
└── Yes → Human-in-the-Loop + Exception Handling

Is the agent public-facing?
└── Yes → Guardrails + Evaluation

Does the task require deep reasoning?
└── Yes, accuracy > speed → Reasoning Engines (thinking model)
```

## Model Reference

| Use Case | Recommended Model | Thinking Budget |
|----------|-------------------|----------------|
| General agent tasks, routing, formatting | `gemini-2.5-flash` | Dynamic (leave unset) |
| Complex reasoning, math, code debugging | `gemini-2.5-flash` | Set high |
| Research-grade analysis, strategic planning | `gemini-2.5-pro` | Set high |

All code examples in this library use `gemini-2.5-flash` as the default. Thinking Budget is a parameter available on Flash and Pro models — not a separate model variant. See [Reasoning Engines](skills/appendix-reasoning-engines/SKILL.md) for a full selection guide.

## Framework Requirements

```python
# Core dependencies
google-genai>=1.0.0        # Gemini API SDK (from google import genai)
google-adk>=1.0.0          # Google Agent Developer Kit
langchain>=0.2.0           # LangChain framework
langchain-google-genai     # LangChain + Gemini integration
langgraph>=0.1.0           # LangGraph for stateful workflows
crewai>=0.1.0              # Multi-agent framework

# Optional by pattern
chromadb                   # Vector storage (RAG, Memory)
langchain-chroma           # LangChain Chroma integration (replaces langchain_community.vectorstores)
langchain-text-splitters   # Text splitting utilities (replaces langchain.text_splitter)
fastapi                    # MCP/A2A server implementation
uvicorn                    # ASGI server for MCP/A2A
mcp                        # Model Context Protocol Python SDK
scikit-learn               # ML-based routing classifier
google-cloud-aiplatform    # Vertex AI Vector Search (MatchingEngine) — RAG production only
```

## ADK API Reference

All ADK code examples in this library use the correct session-based API:

```python
from google.adk.agents import LlmAgent          # NOT Agent
from google.adk.runners import Runner            # NOT InMemoryRunner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part

# Standard pattern for running an ADK agent
agent = LlmAgent(name="MyAgent", model="gemini-2.5-flash", instruction="...")

session_service = InMemorySessionService()
runner = Runner(agent=agent, app_name="my_app", session_service=session_service)
session_service.create_session(app_name="my_app", user_id="user1", session_id="session1")

message = Content(parts=[Part(text="Your prompt here")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

> **Note**: `InMemoryRunner.run(text)` does **not** exist in the ADK. Always use the `Runner` + `InMemorySessionService` pattern shown above.

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-27 | Initial release — 28 skills across 21 chapters + 7 appendices |
| 1.1 | 2026-03-27 | **Priority 1**: Fixed ADK API bugs (`LlmAgent`, `Runner` + session-based pattern) |
| 1.2 | 2026-03-27 | **Priority 2**: Added Context Engineering, ML Model-Based Routing, Agent Complexity Levels |
| 1.3 | 2026-03-27 | **Priority 3**: Added Korean trigger phrases to all 28 skills |
| 1.4 | 2026-03-27 | **Priority 4**: Upgraded to `gemini-2.5-flash`, added Related Skills, added Appendix F (Reasoning Engines) |
| 1.5 | 2026-03-27 | **Claude Code compatibility**: Platform Compatibility table, branched Quick Start and Platform Usage, Claude Code install options A–D |
| 1.6 | 2026-03-27 | **Language standardization**: README rewritten in English; Korean retained only in trigger phrase examples |
| 1.7 | 2026-03-27 | **Gemini CLI / Antigravity guide expanded**: Verified paths (`.gemini/skills/`, `.agents/skills/`), skill management commands, install options A–D, split into separate sections |
| 1.8 | 2026-03-27 | **Library migration**: `google-generativeai` → `google-genai` (23 files), `vertexai` → `client.models.embed_content` (2 files), deprecated LangChain → `langchain-chroma` / `langchain-text-splitters` / `RunnableWithMessageHistory` (2 files) |
| 1.9 | 2026-03-27 | **Multilingual triggers**: Added Japanese (281) and Chinese (281) triggers to all 28 skills; total 1,376 triggers across EN/KR/JA/ZH; added Multilingual Trigger System section to README |
| 1.10 | 2026-03-27 | **Extension format**: Created Gemini CLI Extension (`gemini-extension.json`); added `github-release-extension/` and `github-release-skills/` as separate release folders |
| 1.11 | 2026-03-27 | **MCP skill rename**: Renamed `mcp` frontmatter `name` to `mcp-setup` to avoid collision with Gemini CLI built-in `/mcp` command |
| 1.12 | 2026-03-28 | **Cleanup**: Deleted legacy `github-release/` folder (superseded by `github-release-skills/`) |

## Source

Based on **"Agentic Design Patterns"** by Antonio Gulli (424 pages, 21 chapters + 6 appendices).

Covers all major categories:
- **Core Patterns**: Foundational building blocks for all agents
- **State Management**: Persistence, learning, goal tracking
- **Reliability**: Error handling, human oversight, knowledge grounding
- **Advanced Patterns**: Federation, optimization, reasoning, safety, evaluation
- **Appendix Skills**: Prompt engineering, GUI agents, framework selection, AgentSpace, CLI tools, coding teams, reasoning engines


## License

MIT License — free to use, modify, and distribute.
