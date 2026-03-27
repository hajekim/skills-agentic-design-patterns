---
name: a2a
description: This skill should be used when the user wants to "agent-to-agent communication", "A2A protocol", "inter-agent messaging", "agent network", "agent federation", "cross-agent calls", "distributed agent systems", "agent discovery", "agent cards", "standardized agent communication", "remote agent invocation", "multi-agent network", "agent service mesh", "agent-to-agent API", "orchestrate remote agents", "agent interoperability", "connect agent services", or build systems where agents communicate with and delegate work to other agents using standardized protocols. Also responds to Korean: "에이전트 간 통신", "A2A 프로토콜", "에이전트 간 메시지 전달", "에이전트 네트워크", "분산 에이전트 시스템", "에이전트 발견 및 위임", "A2A 설정", "에이전트끼리 연결", "A2A 써보고 싶어", "에이전트끼리 API로 통신", "다른 에이전트 호출해줘", "에이전트 간 협업 프로토콜". Also responds to Japanese: "エージェント間通信", "A2Aプロトコル", "エージェントネットワーク", "分散エージェントシステム", "エージェント同士を連携させたい", "A2Aの設定", "リモートエージェント呼び出し", "エージェント間API", "エージェントの相互運用性", "エージェントフェデレーション" Also responds to Chinese: "智能体间通信", "A2A协议", "智能体网络", "分布式智能体系统", "让智能体互相通信", "A2A配置", "调用远程智能体", "智能体间API", "智能体互操作性", "智能体联邦".. Apply this skill to design or implement the Agent-to-Agent (A2A) communication agentic design pattern.
version: 1.0.0
---

# Agent-to-Agent (A2A) Communication Pattern

## Overview

The **Agent-to-Agent (A2A) Communication Pattern** enables autonomous agents to discover, communicate with, and delegate tasks to other agents using standardized protocols. Just as MCP standardizes tool integration, A2A standardizes how agents interact with each other — enabling federated, interoperable multi-agent systems across organizational and technical boundaries.

**Core Principle:** Agents should be first-class collaborators, not just tool callers — standardize how agents discover and communicate with each other.

## When This Skill Applies

Activate this pattern when:
- Multiple specialized agents need to collaborate across service boundaries
- Agents from different organizations or teams must interoperate
- Dynamic agent discovery is needed (don't hardcode agent endpoints)
- Complex tasks require delegating sub-tasks to expert agents
- You want to compose existing agents into new workflows without rewriting them
- Agent capabilities need to be advertised and discoverable programmatically

**Rule of thumb:** Use A2A when your multi-agent system spans organizational boundaries, requires dynamic discovery, or must be interoperable with third-party agents.

## A2A Architecture

### Key Concepts
- **Agent Card**: Machine-readable capability advertisement (what the agent can do)
- **Task**: The unit of work delegated between agents
- **Artifact**: Output produced by an agent for the requesting agent
- **Discovery**: Finding agents by capability, not by hardcoded URL
- **Trust**: Authentication and authorization between agents

### A2A Communication Flow
```
Agent A (Client)                          Agent B (Server)
     │                                          │
     │──── Discover Agent Cards ──────────────►│
     │◄─── Agent Card (capabilities) ──────────│
     │                                          │
     │──── Send Task (with context) ──────────►│
     │◄─── Task Acknowledgment ────────────────│
     │                                          │
     │──── Poll for status ───────────────────►│
     │◄─── Status Update ──────────────────────│
     │                                          │
     │──── Get Result ────────────────────────►│
     │◄─── Artifact (result) ──────────────────│
```

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the A2A communication requirements:
1. Which agents need to communicate? What are their roles?
2. What tasks will be delegated between agents?
3. How will agents discover each other? (Registry, DNS, hardcoded endpoints)
4. What authentication is needed between agents?
5. Should communication be synchronous or asynchronous?

### PLAN
Design the A2A architecture:
1. Define Agent Cards for each agent (capabilities, skills, endpoint)
2. Choose discovery mechanism (agent registry, static config, dynamic DNS)
3. Design task schemas: what data flows between agents?
4. Plan error handling for unreachable or misbehaving agents
5. Define trust model: mTLS, API keys, OAuth between agents

### ACTION
Implement A2A communication:
1. Create Agent Card for each agent with capability declarations
2. Implement task sender (client agent) with discovery and invocation
3. Implement task receiver (server agent) with execution and result return
4. Test inter-agent communication end-to-end
5. Add observability: trace task flows across agent boundaries

## Implementation: A2A Protocol with Google ADK

### Agent Card Definition
```python
# agent_card.py — Describes what this agent can do
AGENT_CARD = {
    "name": "ResearchAgent",
    "version": "1.0.0",
    "description": "Specializes in web research and information synthesis",
    "url": "http://localhost:8001/a2a",
    "capabilities": {
        "streaming": False,
        "pushNotifications": False
    },
    "skills": [
        {
            "id": "web_research",
            "name": "Web Research",
            "description": "Research any topic using web search and produce a structured summary",
            "inputSchema": {
                "type": "object",
                "properties": {
                    "topic": {"type": "string", "description": "Topic to research"},
                    "depth": {"type": "string", "enum": ["quick", "thorough"], "default": "thorough"}
                },
                "required": ["topic"]
            },
            "outputSchema": {
                "type": "object",
                "properties": {
                    "summary": {"type": "string"},
                    "key_points": {"type": "array", "items": {"type": "string"}},
                    "sources": {"type": "array", "items": {"type": "string"}}
                }
            }
        }
    ]
}
```

### A2A Server Agent (Task Receiver)
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional, Any
import uuid
import time
from google import genai

app = FastAPI(title="ResearchAgent A2A Server")
client = genai.Client()

# In-memory task store (use a real DB in production)
tasks = {}

class TaskRequest(BaseModel):
    id: str
    skill_id: str
    input: dict
    context: Optional[str] = None

class TaskStatus(BaseModel):
    id: str
    status: str  # submitted/working/completed/failed
    result: Optional[Any] = None
    error: Optional[str] = None

@app.get("/.well-known/agent.json")
async def get_agent_card():
    """Serve the Agent Card for discovery."""
    return AGENT_CARD

@app.post("/a2a/tasks")
async def submit_task(task: TaskRequest) -> TaskStatus:
    """Receive and execute a task from another agent."""
    tasks[task.id] = {"status": "working", "started_at": time.time()}

    try:
        if task.skill_id == "web_research":
            topic = task.input.get("topic", "")
            depth = task.input.get("depth", "thorough")

            # Execute the research task
            prompt = f"""Research the topic: {topic}
            Depth: {depth}
            {"Context: " + task.context if task.context else ""}

            Provide:
            1. A comprehensive summary (2-3 paragraphs)
            2. 5-7 key bullet points
            3. Simulated sources (in production, use real search APIs)

            Format as JSON: {{"summary": "...", "key_points": [...], "sources": [...]}}"""

            response = client.models.generate_content(model='gemini-2.5-flash', contents=prompt)

            import json, re
            match = re.search(r'\{[\s\S]+\}', response.text)
            result = json.loads(match.group()) if match else {"summary": response.text}

            tasks[task.id] = {
                "status": "completed",
                "result": result,
                "completed_at": time.time()
            }
            return TaskStatus(id=task.id, status="completed", result=result)

        else:
            raise ValueError(f"Unknown skill: {task.skill_id}")

    except Exception as e:
        tasks[task.id] = {"status": "failed", "error": str(e)}
        return TaskStatus(id=task.id, status="failed", error=str(e))

@app.get("/a2a/tasks/{task_id}")
async def get_task_status(task_id: str) -> TaskStatus:
    """Check the status of a submitted task."""
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Task not found")

    task = tasks[task_id]
    return TaskStatus(
        id=task_id,
        status=task["status"],
        result=task.get("result"),
        error=task.get("error")
    )
```

### A2A Client Agent (Task Sender)
```python
import requests
import uuid
import time
from typing import Optional

class A2AClient:
    """Client for communicating with A2A-compatible agents."""

    def __init__(self, agent_url: str):
        self.agent_url = agent_url.rstrip('/')
        self._agent_card = None

    def discover(self) -> dict:
        """Fetch the agent's capability card."""
        if self._agent_card is None:
            response = requests.get(f"{self.agent_url}/.well-known/agent.json", timeout=10)
            response.raise_for_status()
            self._agent_card = response.json()
        return self._agent_card

    def get_skills(self) -> list:
        """Get list of available skills from agent card."""
        card = self.discover()
        return card.get("skills", [])

    def delegate_task(
        self,
        skill_id: str,
        input_data: dict,
        context: Optional[str] = None,
        poll_interval: float = 1.0,
        timeout: float = 60.0
    ) -> dict:
        """Delegate a task to the remote agent and wait for result.

        Args:
            skill_id: The skill to invoke (from agent card)
            input_data: Input parameters for the skill
            context: Optional context from the calling agent
            poll_interval: Seconds between status checks
            timeout: Maximum wait time in seconds

        Returns:
            Task result or error
        """
        task_id = str(uuid.uuid4())

        # Submit task
        payload = {
            "id": task_id,
            "skill_id": skill_id,
            "input": input_data,
            "context": context
        }

        response = requests.post(
            f"{self.agent_url}/a2a/tasks",
            json=payload,
            timeout=30
        )
        response.raise_for_status()
        task_status = response.json()

        # If already completed (synchronous agent), return immediately
        if task_status["status"] in ("completed", "failed"):
            return task_status

        # Poll for completion (asynchronous agent)
        deadline = time.time() + timeout
        while time.time() < deadline:
            time.sleep(poll_interval)
            status_response = requests.get(
                f"{self.agent_url}/a2a/tasks/{task_id}",
                timeout=10
            )
            status_response.raise_for_status()
            status = status_response.json()

            if status["status"] == "completed":
                return status
            elif status["status"] == "failed":
                return status

        return {
            "id": task_id,
            "status": "timeout",
            "error": f"Task did not complete within {timeout}s"
        }

# Orchestrator agent that delegates to specialist agents
class OrchestratorAgent:
    """Orchestrates multiple specialized agents via A2A protocol."""

    def __init__(self):
        # In production, discover agents from a registry
        self.research_agent = A2AClient("http://localhost:8001")
        self.writer_agent = A2AClient("http://localhost:8002")
        self.analyst_agent = A2AClient("http://localhost:8003")

    def run_research_pipeline(self, topic: str) -> dict:
        """Run a full research → analyze → write pipeline across agents."""
        print(f"Starting pipeline for: {topic}")

        # Step 1: Research (delegate to ResearchAgent)
        print("  Step 1: Delegating research task...")
        research_result = self.research_agent.delegate_task(
            skill_id="web_research",
            input_data={"topic": topic, "depth": "thorough"}
        )

        if research_result["status"] != "completed":
            return {"error": f"Research failed: {research_result.get('error')}"}

        research_data = research_result["result"]

        # Step 2: Analyze (delegate to AnalystAgent with research context)
        print("  Step 2: Delegating analysis task...")
        analysis_result = self.analyst_agent.delegate_task(
            skill_id="trend_analysis",
            input_data={"data": research_data},
            context=f"Research on: {topic}"
        )

        # Step 3: Write report (delegate to WriterAgent)
        print("  Step 3: Delegating writing task...")
        combined_context = {
            "research": research_data,
            "analysis": analysis_result.get("result", {})
        }

        report_result = self.writer_agent.delegate_task(
            skill_id="report_writing",
            input_data={"content": combined_context, "topic": topic},
            context=f"Final report on: {topic}"
        )

        return {
            "topic": topic,
            "research": research_data,
            "analysis": analysis_result.get("result"),
            "report": report_result.get("result"),
            "pipeline_status": "completed"
        }
```

### Agent Registry for Discovery
```python
from typing import Optional

# Simple in-memory agent registry
class AgentRegistry:
    """Registry for discovering agents by capability."""

    def __init__(self):
        self._registry: dict = {}  # skill_id → list of agent URLs

    def register(self, agent_url: str):
        """Register an agent by fetching and indexing its agent card."""
        try:
            client = A2AClient(agent_url)
            card = client.discover()
            for skill in card.get("skills", []):
                skill_id = skill["id"]
                if skill_id not in self._registry:
                    self._registry[skill_id] = []
                if agent_url not in self._registry[skill_id]:
                    self._registry[skill_id].append(agent_url)
            print(f"Registered {card['name']} with {len(card.get('skills', []))} skills")
        except Exception as e:
            print(f"Failed to register {agent_url}: {e}")

    def find_agents(self, skill_id: str) -> list:
        """Find all agents capable of performing a specific skill."""
        return self._registry.get(skill_id, [])

    def get_best_agent(self, skill_id: str) -> Optional[A2AClient]:
        """Get the best available agent for a skill."""
        agents = self.find_agents(skill_id)
        if not agents:
            return None
        # In production: use load balancing, latency-based selection, etc.
        return A2AClient(agents[0])

# Usage
registry = AgentRegistry()
registry.register("http://localhost:8001")  # Research agent
registry.register("http://localhost:8002")  # Writer agent
registry.register("http://localhost:8003")  # Analyst agent

research_agent = registry.get_best_agent("web_research")
if research_agent:
    result = research_agent.delegate_task("web_research", {"topic": "AI agents 2025"})
```

## Key Takeaways

- **Agent Cards**: Self-describing capability manifests enable dynamic discovery without hardcoding
- **Task delegation**: A2A creates a standard request/response cycle between agents with status polling
- **Registry pattern**: Agent registries decouple orchestrators from specific agent endpoints
- **Interoperability**: A2A enables cross-organization agent collaboration — build once, integrate everywhere
- **Async by default**: Long-running agent tasks should use async submission + polling, not blocking calls
- **Trust boundaries**: Agent-to-agent calls cross security boundaries — always authenticate and authorize

## Anti-Patterns to Avoid

- **Hardcoded agent URLs**: Makes the system brittle — use discovery/registry patterns
- **No timeouts**: Remote agents can hang — always set task deadlines
- **Synchronous blocking for long tasks**: Use async polling for tasks that take more than a few seconds
- **No capability versioning**: Agent cards need version numbers so clients handle API changes
- **Implicit trust**: Don't trust agent-to-agent calls without authentication — they cross security boundaries

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Federated multi-agent network | A2A + **Multi-Agent Collaboration** + **Routing** |
| Standardized inter-agent tool sharing | A2A + **MCP** |
| Distributed autonomous agent system | A2A + **Planning** + **Parallelization** |
| Secure enterprise agent network | A2A + **Guardrails** + **Exception Handling** |

## References

- Google A2A Protocol: https://google.github.io/A2A/
- A2A Specification: https://github.com/google/A2A
- Google ADK Multi-Agent: https://google.github.io/adk-docs/agents/multi-agents/
- Agent Interoperability Standards: https://modelcontextprotocol.io/
