---
name: tool-use
description: This skill should be used when the user wants to "call external APIs", "use tools in agents", "function calling", "web search integration", "code execution", "database queries", "agent tool integration", "extend agent capabilities", "RAG tool", "external service calls", "give agent tools", "agent with web search", "agent API access", "agent with browser", "agent with calculator", "tool-augmented agent", or build agents that interact with the real world through tools. Also responds to Korean: "외부 API 호출", "에이전트 도구 사용", "함수 호출", "웹 검색 통합", "코드 실행 에이전트", "외부 서비스 연동", "툴 사용", "도구 추가", "함수 호출 에이전트 만들어줘", "API 연동해줘", "검색 기능 추가해줘", "외부 도구 써보고 싶어". Also responds to Japanese: "外部API呼び出し", "ツール使用エージェント", "関数呼び出し", "ウェブ検索統合", "コード実行エージェント", "ツールを使うエージェント", "データベース検索", "外部サービス連携", "APIを使いたい", "検索機能を追加したい" Also responds to Chinese: "调用外部API", "工具使用智能体", "函数调用", "集成网络搜索", "代码执行智能体", "使用工具的智能体", "数据库查询", "外部服务集成", "帮我接入API", "添加搜索功能".. Apply this skill to design or implement the Tool Use agentic design pattern.
version: 1.0.0
---

# Tool Use Pattern

## Overview

The **Tool Use Pattern** extends an agent's capabilities beyond pure language generation by enabling it to interact with external systems — APIs, databases, search engines, code executors, and more. The agent reasons about which tool to call, invokes it with appropriate parameters, processes the returned data, and integrates the results into its response.

**Core Principle:** Extend the agent's reach — let it act in the world, not just reason about it.

## When This Skill Applies

Activate this pattern when:
- The agent needs real-time or external data (weather, stock prices, news)
- Tasks require computation, code execution, or mathematical verification
- Document retrieval and RAG (Retrieval-Augmented Generation) is needed
- The agent must persist state (write to databases, files, memory stores)
- Actions must be taken in the world (send email, create calendar events, trigger workflows)
- The agent needs to verify its reasoning with factual lookups

**Rule of thumb:** If the task requires information or capabilities beyond the model's training data or inherent abilities — use tools.

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the tool requirements:
1. What external data or capabilities does the agent need?
2. What tools exist or need to be built to provide those capabilities?
3. What are the input/output schemas for each tool?
4. What are the failure modes and how should they be handled?

### PLAN
Design the tool integration:
1. Define each tool with a clear name, description, and parameter schema
2. Write tool descriptions that help the LLM understand when and how to use each tool
3. Design the tool selection logic (LLM-driven function calling vs. rule-based)
4. Plan error handling for API failures, timeouts, and unexpected responses
5. Consider tool chaining — sequential tool calls where outputs feed subsequent calls

### ACTION
Implement tool-enabled agents:
1. Register tools with the LLM using the framework's function calling interface
2. Implement each tool function with proper input validation and error handling
3. Let the LLM determine when and how to call tools based on user intent
4. Process tool results and integrate them into the agent's response
5. Log tool calls for observability and debugging

## Core Tool Types

### Search and Retrieval Tools
- **Web Search**: Real-time internet information retrieval
- **Document RAG**: Semantic search over internal knowledge bases
- **Database Query**: Structured data retrieval via SQL or NoSQL queries
- **Knowledge Graph**: Entity and relationship lookups

### Computation Tools
- **Code Executor**: Run Python/JavaScript for mathematical or data operations
- **Calculator**: Precise arithmetic without model hallucination risk
- **Data Analyzer**: Statistical analysis, aggregation, transformation

### Action Tools
- **Email/Calendar**: Create events, send messages, schedule meetings
- **File Operations**: Read, write, transform files
- **API Calls**: Interact with third-party services (Slack, GitHub, CRM)
- **Browser Automation**: Web scraping, form submission, UI interaction

### Memory Tools
- **Long-term Memory**: Persist and recall information across sessions
- **Vector Store**: Store and retrieve embeddings for semantic memory
- **State Management**: Track task progress and intermediate results

## Practical Use Cases

### Research Assistant
```
User: "What's the current price of AAPL?"
Agent → calls stock_price_tool("AAPL") → returns $182.50
Agent → responds: "Apple (AAPL) is currently trading at $182.50"
```

### Code Generation and Execution
```
User: "Calculate the 100th Fibonacci number"
Agent → writes Python code → calls code_executor(code) → returns result
Agent → responds: "The 100th Fibonacci number is 354,224,848,179,261,915,075"
```

### Multi-Tool Research Pipeline
```
User: "Summarize the latest AI news and save it to a file"
Agent → calls web_search("AI news today") → gets articles
Agent → calls summarizer(articles) → gets summary
Agent → calls file_write("ai_news.md", summary) → saves file
Agent → responds: "Summary saved to ai_news.md"
```

## Implementation with Google ADK

### ADK FunctionTool Integration
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai.types import Content, Part
import requests
import subprocess

def web_search(query: str) -> str:
    """Search the web for current information on any topic.

    Args:
        query: The search query string

    Returns:
        Search results as formatted text
    """
    # In production, use a real search API (SerpAPI, Tavily, etc.)
    return f"Search results for '{query}': [simulated results]"

def execute_python(code: str) -> str:
    """Execute Python code and return the output.

    Args:
        code: Valid Python code to execute

    Returns:
        Standard output from code execution, or error message
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
        return "Error: Code execution timed out after 30 seconds"

def get_weather(location: str) -> str:
    """Get current weather for a location.

    Args:
        location: City name or coordinates

    Returns:
        Current weather conditions and temperature
    """
    # In production, call a real weather API
    return f"Weather in {location}: 22°C, partly cloudy"

def save_to_file(filename: str, content: str) -> str:
    """Save content to a local file.

    Args:
        filename: Name of the file to create or overwrite
        content: Text content to write to the file

    Returns:
        Success confirmation message
    """
    with open(filename, 'w') as f:
        f.write(content)
    return f"Successfully saved to {filename}"

# Create agent with multiple tools
research_agent = LlmAgent(
    name="ResearchAssistant",
    model="gemini-2.5-flash",
    instruction="""You are a research assistant with access to web search,
    code execution, weather lookup, and file operations.

    Always use the most appropriate tool for the task:
    - web_search: for current information, facts, news
    - execute_python: for calculations, data processing, algorithmic tasks
    - get_weather: for weather information
    - save_to_file: when the user wants to persist results

    Think step-by-step about which tools to use and in what order.""",
    tools=[
        FunctionTool(web_search),
        FunctionTool(execute_python),
        FunctionTool(get_weather),
        FunctionTool(save_to_file)
    ]
)

session_service = InMemorySessionService()
runner = Runner(agent=research_agent, app_name="tool_use_app", session_service=session_service)
session_service.create_session(app_name="tool_use_app", user_id="user1", session_id="session1")
message = Content(parts=[Part(text="Search for the latest news about quantum computing and save a summary to quantum_news.md")])
for event in runner.run(user_id="user1", session_id="session1", new_message=message):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

### LangChain Tool Integration
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

@tool
def web_search(query: str) -> str:
    """Search the web for real-time information. Use for current events, facts, prices."""
    return f"Web results for: {query}"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression. Use for precise calculations."""
    try:
        result = eval(expression, {"__builtins__": {}}, {})
        return str(result)
    except Exception as e:
        return f"Calculation error: {str(e)}"

@tool
def read_file(filepath: str) -> str:
    """Read the contents of a text file."""
    try:
        with open(filepath, 'r') as f:
            return f.read()
    except FileNotFoundError:
        return f"File not found: {filepath}"

tools = [web_search, calculate, read_file]

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant with tools. Use tools when needed."),
    ("user", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = agent_executor.invoke({"input": "What is 15% of 847.50? Also search for current Bitcoin price."})
```

### Gemini Native Function Calling
```python
from google import genai
from google.genai import types

client = genai.Client()

# Define tool schemas
tools = [
    types.Tool(function_declarations=[
        types.FunctionDeclaration(
            name="search_web",
            description="Search the internet for current information",
            parameters=types.Schema(
                type="object",
                properties={
                    "query": types.Schema(type="string", description="The search query")
                },
                required=["query"]
            )
        ),
        types.FunctionDeclaration(
            name="get_stock_price",
            description="Get the current stock price for a ticker symbol",
            parameters=types.Schema(
                type="object",
                properties={
                    "ticker": types.Schema(type="string", description="Stock ticker symbol (e.g., AAPL, GOOGL)")
                },
                required=["ticker"]
            )
        )
    ])
]

# Tool implementations
def search_web(query: str) -> str:
    return f"Search results for '{query}': [results here]"

def get_stock_price(ticker: str) -> str:
    prices = {"AAPL": 182.50, "GOOGL": 172.25, "MSFT": 415.80}
    price = prices.get(ticker.upper(), "Not found")
    return f"{ticker}: ${price}"

tool_functions = {
    "search_web": search_web,
    "get_stock_price": get_stock_price
}

# Agentic loop with function calling
contents = [types.Content(role="user", parts=[types.Part(text="What's the current price of Apple stock?")])]

response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=contents,
    config=types.GenerateContentConfig(tools=tools)
)

# Handle function calls
while response.candidates[0].content.parts[0].function_call:
    function_call = response.candidates[0].content.parts[0].function_call
    func_name = function_call.name
    func_args = dict(function_call.args)

    # Execute the tool
    tool_result = tool_functions[func_name](**func_args)

    # Append model response and tool result to contents
    contents.append(response.candidates[0].content)
    contents.append(types.Content(
        role="user",
        parts=[types.Part(function_response=types.FunctionResponse(
            name=func_name,
            response={"result": tool_result}
        ))]
    ))

    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=contents,
        config=types.GenerateContentConfig(tools=tools)
    )

print(response.text)
```

### CrewAI Tool Integration
```python
from crewai import Agent, Task, Crew
from crewai_tools import SerperDevTool, FileWriterTool

# Initialize tools
search_tool = SerperDevTool()
file_tool = FileWriterTool()

# Create agent with tools
researcher = Agent(
    role="Research Analyst",
    goal="Gather and synthesize information from multiple sources",
    backstory="Expert at finding and analyzing information",
    tools=[search_tool, file_tool],
    verbose=True
)

research_task = Task(
    description="Research the latest trends in AI agents and save findings to report.md",
    expected_output="A comprehensive report on AI agent trends saved to a file",
    agent=researcher
)

crew = Crew(agents=[researcher], tasks=[research_task])
result = crew.kickoff()
```

## Tool Design Best Practices

### Write Clear Tool Descriptions
```python
# Bad - vague description
def search(q: str) -> str:
    """Search for stuff."""
    ...

# Good - specific, actionable description
def web_search(query: str) -> str:
    """Search the internet for current, real-time information.
    Use this when you need up-to-date facts, news, prices, or events
    that may not be in your training data. Returns relevant web results.

    Args:
        query: Specific search query (be precise for better results)

    Returns:
        Formatted search results with sources
    """
    ...
```

### Handle Tool Failures Gracefully
```python
def robust_api_call(endpoint: str, params: dict) -> str:
    """Call an external API with error handling."""
    try:
        response = requests.get(endpoint, params=params, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.Timeout:
        return "Error: API request timed out. Try again or use cached data."
    except requests.HTTPError as e:
        return f"Error: API returned {e.response.status_code}. Check parameters."
    except Exception as e:
        return f"Error: Unexpected failure — {str(e)}"
```

## Key Takeaways

- **Extend agent reach**: Tools let agents interact with real-world systems, not just generate text
- **Clear tool descriptions**: The LLM selects tools based on descriptions — write them for the model, not humans
- **Input validation**: Validate parameters before calling external services to prevent errors
- **Error handling**: External tools fail — always return informative error messages, not exceptions
- **Tool chaining**: Agents can call multiple tools sequentially, using one tool's output as another's input
- **Observability**: Log all tool calls with inputs, outputs, and timing for debugging

## Anti-Patterns to Avoid

- **Vague tool descriptions**: The model can't use tools it doesn't understand — be specific
- **No error handling**: Unhandled exceptions crash the agent instead of graceful recovery
- **Too many tools**: Agents with 20+ tools often struggle to choose — keep tool sets focused
- **Ignoring tool results**: Always integrate tool output into the final response
- **Missing timeouts**: External API calls without timeouts can hang the entire agent

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Knowledge-grounded tool agent | Tool Use + **RAG** + **Memory Management** |
| Standardized tool integration | Tool Use + **MCP** |
| Multi-step tool-using pipeline | Tool Use + **Planning** + **Prompt Chaining** |
| Resilient external API calls | Tool Use + **Exception Handling** + **Resource-Aware** |

## References

- Google ADK FunctionTool: https://google.github.io/adk-docs/tools/
- Gemini Function Calling: https://ai.google.dev/gemini-api/docs/function-calling
- LangChain Tools Documentation: https://python.langchain.com/docs/modules/agents/tools/
- CrewAI Tools: https://docs.crewai.com/concepts/tools
