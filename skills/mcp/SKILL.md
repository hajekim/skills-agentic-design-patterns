---
name: mcp-setup
description: This skill should be used when the user wants to "Model Context Protocol", "MCP server", "MCP client", "standardized tool integration", "JSON-RPC agent tools", "LLM tool protocol", "connect agents to external tools", "MCP primitives", "FastMCP", "ADK MCPToolset", "plug-and-play agent tools", "MCP integration", "tool server for agents", "MCP tools", "agent plugin system", "MCP setup", "model context protocol integration", or build agents that use the standardized MCP protocol for tool and resource integration. Also responds to Korean: "모델 컨텍스트 프로토콜", "MCP 서버", "MCP 클라이언트", "표준화된 도구 통합", "플러그 앤 플레이 에이전트 도구", "ADK MCP 툴셋", "MCP 구성", "MCP 설정", "MCP 써보고 싶어", "MCP 툴 연결", "MCP 서버 만들어줘", "도구 표준화해줘", "플러그인 에이전트". Also responds to Japanese: "モデルコンテキストプロトコル", "MCPサーバー", "MCPクライアント", "標準化ツール統合", "MCPの設定", "MCPを使いたい", "MCP構成", "エージェントのプラグインシステム", "ツールサーバー", "MCPツール連携" Also responds to Chinese: "模型上下文协议", "MCP服务器", "MCP客户端", "标准化工具集成", "MCP配置", "想用MCP", "MCP构建", "智能体插件系统", "工具服务器", "MCP工具集成".. Apply this skill to design or implement the Model Context Protocol (MCP) agentic design pattern.
version: 1.0.0
---

# Model Context Protocol (MCP) Pattern

## Overview

The **Model Context Protocol (MCP) Pattern** provides a standardized, open protocol for connecting LLM-based agents to external tools, data sources, and capabilities. MCP establishes a universal interface — like USB for AI tools — so any MCP-compatible agent can use any MCP-compatible tool server without custom integration code.

**Core Principle:** Standardize the contract between agents and tools — one protocol to connect everything.

## When This Skill Applies

Activate this pattern when:
- You need plug-and-play tool integration across multiple agent frameworks
- Tools should be reusable across different LLM providers and agent systems
- External services (databases, APIs, file systems) must be exposed as agent tools
- A team maintains shared tool servers used by multiple agents
- You want to decouple tool development from agent development
- Cross-organization tool sharing and interoperability is required

**Rule of thumb:** Use MCP when tool reusability and interoperability matter more than tight integration — build the tool once, use it everywhere.

## MCP Architecture

### Client-Server Model
```
┌─────────────────┐    JSON-RPC 2.0    ┌─────────────────┐
│   MCP Client    │ ◄─────────────────► │   MCP Server    │
│  (LLM Agent)   │    (stdio / HTTP)   │  (Tool Provider)│
└─────────────────┘                    └─────────────────┘
```

- **MCP Client**: The agent (LLM host) that consumes tools and resources
- **MCP Server**: Exposes tools, resources, and prompts via the MCP protocol
- **Transport**: Communication via stdio (local) or HTTP/SSE (remote)
- **Protocol**: JSON-RPC 2.0 for all message exchanges

### Four MCP Primitives

| Primitive | Description | Use Case |
|-----------|-------------|----------|
| **Tools** | Executable functions the LLM can call | Web search, code execution, API calls |
| **Resources** | Data the LLM can read (like files/DB) | Documents, database records, config files |
| **Prompts** | Reusable prompt templates with parameters | Standardized task instructions |
| **Sampling** | Server requests LLM completions | Server-side agentic reasoning |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the MCP integration requirements:
1. What tools/resources need to be exposed via MCP?
2. Will the server run locally (stdio) or remotely (HTTP/SSE)?
3. Which agent frameworks need to connect as MCP clients?
4. What authentication and security requirements apply?

### PLAN
Design the MCP architecture:
1. Define each tool with name, description, and input schema
2. Choose transport mechanism (stdio for local, HTTP for remote/shared)
3. Design resource URIs and access patterns for data exposure
4. Plan server lifecycle: startup, connection handling, shutdown
5. Design client integration in the target agent framework

### ACTION
Implement MCP server and client:
1. Create MCP server using `mcp` library with `FastMCP` decorator
2. Register tools with `@mcp.tool()` and resources with `@mcp.resource()`
3. Start server and test tool discovery with MCP Inspector
4. Connect agent as MCP client using framework integration (ADK `MCPToolset`)
5. Test end-to-end: agent calls tool → MCP routes → server executes → result returned

## Implementation: FastMCP Server

### Basic MCP Server with Tools
```python
from mcp.server.fastmcp import FastMCP
import subprocess
import requests

# Create the MCP server
mcp = FastMCP("ResearchTools")

@mcp.tool()
def web_search(query: str) -> str:
    """Search the web for current information on any topic.

    Args:
        query: The search query to find information about

    Returns:
        Search results with relevant information
    """
    # In production, use a real search API (SerpAPI, Tavily, etc.)
    return f"Search results for '{query}': [results would appear here]"

@mcp.tool()
def execute_python(code: str) -> str:
    """Execute Python code safely and return the output.

    Args:
        code: Python code to execute

    Returns:
        Standard output or error message from execution
    """
    try:
        result = subprocess.run(
            ["python3", "-c", code],
            capture_output=True,
            text=True,
            timeout=30
        )
        return result.stdout if result.returncode == 0 else f"Error: {result.stderr}"
    except subprocess.TimeoutExpired:
        return "Error: Code execution timed out"

@mcp.tool()
def get_file_contents(filepath: str) -> str:
    """Read and return the contents of a file.

    Args:
        filepath: Path to the file to read

    Returns:
        File contents as text
    """
    try:
        with open(filepath, 'r') as f:
            return f.read()
    except FileNotFoundError:
        return f"Error: File not found: {filepath}"
    except PermissionError:
        return f"Error: Permission denied: {filepath}"

# Run the server (stdio transport by default)
if __name__ == "__main__":
    mcp.run()
```

### MCP Server with Resources
```python
from mcp.server.fastmcp import FastMCP
import json

mcp = FastMCP("KnowledgeBase")

# In-memory knowledge store (use a real database in production)
knowledge_store = {
    "python-best-practices": "Use type hints, write docstrings, follow PEP 8...",
    "agent-patterns": "Key patterns: ReAct, Chain-of-Thought, Plan-and-Execute..."
}

@mcp.resource("knowledge://{topic}")
def get_knowledge(topic: str) -> str:
    """Retrieve knowledge base entries by topic."""
    content = knowledge_store.get(topic, f"No knowledge found for topic: {topic}")
    return content

@mcp.tool()
def search_knowledge(query: str) -> dict:
    """Search the knowledge base for relevant information.

    Args:
        query: Search query to find relevant knowledge

    Returns:
        Dictionary with matching topics and content snippets
    """
    results = {}
    query_lower = query.lower()
    for topic, content in knowledge_store.items():
        if query_lower in topic or query_lower in content.lower():
            results[topic] = content[:200] + "..." if len(content) > 200 else content

    return {"matches": results, "total": len(results)}

@mcp.prompt()
def research_prompt(topic: str, depth: str = "comprehensive") -> str:
    """Generate a research prompt template.

    Args:
        topic: The topic to research
        depth: Research depth (quick/comprehensive/exhaustive)

    Returns:
        Formatted research prompt
    """
    depth_instructions = {
        "quick": "Provide a brief 3-5 sentence overview.",
        "comprehensive": "Provide a detailed analysis with examples and implications.",
        "exhaustive": "Provide an exhaustive analysis covering all aspects, controversies, and future directions."
    }
    instruction = depth_instructions.get(depth, depth_instructions["comprehensive"])

    return f"""Research the following topic: {topic}

{instruction}

Structure your response with:
1. Overview
2. Key concepts
3. Practical applications
4. Current limitations
5. Future directions"""

if __name__ == "__main__":
    mcp.run()
```

## Implementation: ADK MCPToolset Integration

### Connecting an ADK Agent to MCP Server
```python
import asyncio
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, StdioServerParameters
from google.adk.runners import InMemoryRunner

async def create_mcp_agent():
    """Create an ADK agent that uses tools from an MCP server."""

    # Connect to local MCP server via stdio
    mcp_toolset = MCPToolset(
        connection_params=StdioServerParameters(
            command="python3",
            args=["research_server.py"]  # Path to your MCP server
        )
    )

    # Create agent with MCP tools
    agent = Agent(
        name="MCPResearchAgent",
        model="gemini-2.5-flash",
        instruction="""You are a research assistant with access to web search,
        code execution, and a knowledge base through MCP tools.

        Use your tools to:
        - Search for current information (web_search)
        - Run computations (execute_python)
        - Access the knowledge base (search_knowledge)
        - Read files when needed (get_file_contents)

        Always cite the sources of your information.""",
        tools=[mcp_toolset]
    )

    return agent

async def main():
    agent = await create_mcp_agent()
    runner = InMemoryRunner(agent)

    result = await runner.run_async(
        "Research the latest advances in AI agents and calculate the year-over-year growth rate"
    )
    print(result)

asyncio.run(main())
```

### Multi-Server MCP Integration
```python
import asyncio
from google.adk.agents import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, StdioServerParameters

async def create_multi_mcp_agent():
    """Agent connected to multiple specialized MCP servers."""

    # Research tools server
    research_toolset = MCPToolset(
        connection_params=StdioServerParameters(
            command="python3",
            args=["research_server.py"]
        )
    )

    # Database tools server
    db_toolset = MCPToolset(
        connection_params=StdioServerParameters(
            command="python3",
            args=["database_server.py"]
        )
    )

    # File system tools server
    fs_toolset = MCPToolset(
        connection_params=StdioServerParameters(
            command="python3",
            args=["filesystem_server.py"]
        )
    )

    # Agent with access to all three tool servers
    agent = Agent(
        name="EnterpriseAgent",
        model="gemini-2.5-flash",
        instruction="""You are an enterprise AI assistant with access to:
        - Research tools: web search, code execution, knowledge base
        - Database tools: query, update, and analyze data
        - File system tools: read, write, and organize files

        Use the most appropriate tool for each task.""",
        tools=[research_toolset, db_toolset, fs_toolset]
    )

    return agent
```

## Implementation: HTTP/SSE Transport (Remote MCP)

### Remote MCP Server
```python
from mcp.server.fastmcp import FastMCP
from mcp.server.sse import SseServerTransport
from starlette.applications import Starlette
from starlette.routing import Route, Mount
import uvicorn

mcp = FastMCP("RemoteResearchTools")

@mcp.tool()
def analyze_sentiment(text: str) -> dict:
    """Analyze the sentiment of provided text.

    Args:
        text: Text to analyze for sentiment

    Returns:
        Sentiment analysis result with score and label
    """
    # Simplified sentiment — use a real model in production
    positive_words = ["good", "great", "excellent", "amazing", "positive"]
    negative_words = ["bad", "poor", "terrible", "awful", "negative"]

    text_lower = text.lower()
    pos_count = sum(1 for w in positive_words if w in text_lower)
    neg_count = sum(1 for w in negative_words if w in text_lower)

    if pos_count > neg_count:
        return {"label": "POSITIVE", "score": pos_count / (pos_count + neg_count + 1)}
    elif neg_count > pos_count:
        return {"label": "NEGATIVE", "score": neg_count / (pos_count + neg_count + 1)}
    else:
        return {"label": "NEUTRAL", "score": 0.5}

# Create SSE transport for HTTP
sse = SseServerTransport("/messages")

async def handle_sse(request):
    async with sse.connect_sse(request.scope, request.receive, request._send) as streams:
        await mcp._mcp_server.run(
            streams[0], streams[1], mcp._mcp_server.create_initialization_options()
        )

app = Starlette(routes=[
    Route("/sse", endpoint=handle_sse),
    Mount("/messages", app=sse.handle_post_message)
])

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Connecting to Remote MCP via HTTP
```python
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, SseServerParams

# Connect to remote MCP server
remote_toolset = MCPToolset(
    connection_params=SseServerParams(
        url="http://your-mcp-server.example.com/sse"
    )
)
```

## MCP vs. Direct Tool Integration

| Aspect | Direct Tools | MCP Tools |
|--------|-------------|-----------|
| **Reusability** | Single agent/framework | Any MCP-compatible agent |
| **Maintenance** | Update per agent | Update server once |
| **Discovery** | Static configuration | Dynamic tool discovery |
| **Transport** | In-process | Stdio / HTTP / SSE |
| **Standardization** | Framework-specific | Open standard |
| **Complexity** | Lower setup | Higher setup, lower long-term cost |

## Key Takeaways

- **Universal protocol**: MCP-compatible tools work with any MCP-compatible agent framework
- **Four primitives**: Tools (executable), Resources (readable), Prompts (templates), Sampling (LLM requests)
- **FastMCP simplicity**: `@mcp.tool()` and `@mcp.resource()` decorators make server creation straightforward
- **Transport flexibility**: stdio for local dev/security, HTTP/SSE for remote/shared services
- **ADK integration**: `MCPToolset` with `StdioServerParameters` or `SseServerParams` connects ADK agents to any MCP server
- **Tool discovery**: Agents dynamically discover available tools from MCP servers at connection time

## Anti-Patterns to Avoid

- **Overengineering simple tools**: Use direct `FunctionTool` for tools only one agent uses — MCP overhead isn't worth it
- **No authentication**: Remote MCP servers need auth — never expose tool servers without security controls
- **Single monolithic server**: Separate concerns into focused servers (research tools, DB tools, file tools)
- **Ignoring errors**: Always handle MCP connection failures and tool execution errors gracefully
- **Skipping tool descriptions**: Poor descriptions lead to incorrect tool selection — write them for the LLM

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Standardized multi-agent tool access | MCP + **A2A** + **Multi-Agent Collaboration** |
| Enterprise tool integration hub | MCP + **Tool Use** + **Routing** |
| Knowledge + tool standardization | MCP + **RAG** + **Memory Management** |
| Secure tool access with guardrails | MCP + **Guardrails** + **Exception Handling** |

## References

- MCP Official Documentation: https://modelcontextprotocol.io/docs/
- MCP Python SDK: https://github.com/modelcontextprotocol/python-sdk
- FastMCP Framework: https://github.com/jlowin/fastmcp
- Google ADK MCP Integration: https://google.github.io/adk-docs/tools/mcp-tools/
- MCP Specification: https://spec.modelcontextprotocol.io/
