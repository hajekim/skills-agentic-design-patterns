---
name: memory-management
description: This skill should be used when the user wants to "persist agent state", "remember past conversations", "short-term memory", "long-term memory", "session management", "vector database for agents", "agent memory", "context window management", "store user preferences", "recall previous interactions", "stateful agents", "agent with memory", "remember user preferences", "persistent agent context", "conversation history", "memory bank", "cross-session memory", "stateful conversation", or build agents that retain and utilize information across multiple turns or sessions. Also responds to Korean: "에이전트 상태 영속성", "이전 대화 기억", "단기 장기 기억 관리", "세션 관리", "사용자 선호도 저장", "상태 유지 에이전트", "메모리 관리", "에이전트 기억", "기억하는 에이전트", "메모리뱅크", "컨텍스트 유지", "세션 간 기억 유지", "대화 내용 기억", "기억 저장 에이전트". Also responds to Japanese: "エージェントの記憶", "過去の会話を記憶", "長期・短期メモリ管理", "セッション管理", "メモリ管理", "コンテキスト維持", "メモリバンク", "ユーザー設定を保存", "状態を保持するエージェント", "会話の内容を覚えてほしい", "セッション間の記憶保持" Also responds to Chinese: "智能体记忆", "记住过去的对话", "长短期记忆管理", "会话管理", "记忆管理", "保持上下文", "记忆库", "保存用户偏好", "有状态的智能体", "帮我记住对话内容", "跨会话记忆".. Apply this skill to design or implement the Memory Management agentic design pattern.
version: 1.0.0
---

# Memory Management Pattern

## Overview

The **Memory Management Pattern** enables agents to retain and utilize information from past interactions, observations, and learning experiences. Without memory, every agent turn is isolated — the agent has no awareness of prior context. With effective memory management, agents can maintain conversation continuity, personalize responses, track task progress, and improve over time.

**Core Principle:** Give agents the ability to remember — short-term for the current interaction, long-term for knowledge that persists across sessions.

## When This Skill Applies

Activate this pattern when:
- Agents must maintain context across multiple conversation turns
- User preferences or past behaviors should influence future responses
- Multi-step tasks require tracking progress and intermediate results
- Agents need to recall information from previous sessions
- Personalization and continuity are key user experience requirements
- RAG (Retrieval-Augmented Generation) is needed for knowledge-grounded responses

**Rule of thumb:** If an agent needs to "remember" anything beyond the current prompt — use Memory Management.

## Memory Types

### Short-Term Memory (Contextual Memory)
- Lives within the **context window** of the current LLM call
- Contains: recent messages, tool outputs, agent reflections, session state
- **Limited capacity**: context windows have token limits
- **Ephemeral**: lost when the session ends
- Management strategies: summarize older segments, prioritize key information, use long-context models

### Long-Term Memory (Persistent Memory)
- Stored **outside** the agent's immediate context in external systems
- Storage options: databases, knowledge graphs, vector databases
- **Semantic search**: vector embeddings enable similarity-based retrieval
- **Persistent**: survives session terminations, restarts, and time gaps
- Use cases: user preferences, learned knowledge, historical records

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the memory requirements:
1. What information must be retained within a single conversation? (Short-term)
2. What information must persist across sessions? (Long-term)
3. How will long-term memory be retrieved? (Semantic search, key-value, SQL)
4. What are the data retention and privacy requirements?

### PLAN
Design the memory architecture:
1. Choose short-term memory management strategy (context window, summarization, state)
2. Choose long-term memory storage (vector DB, relational DB, key-value store)
3. Design memory retrieval: when and how does the agent query long-term memory?
4. Define state schema: what data goes in session.state vs. long-term memory?
5. Plan memory lifecycle: creation, updates, expiration, deletion

### ACTION
Implement memory-enabled agents:
1. Configure appropriate SessionService for conversation history
2. Use session.state for temporary, session-scoped data
3. Integrate MemoryService for cross-session persistent knowledge
4. Implement retrieval logic: search long-term memory before generating responses
5. Update memory after significant interactions or new knowledge acquisition

## ADK Memory Architecture

Google ADK provides three core concepts for memory management:

| Concept | Scope | Purpose |
|---------|-------|---------|
| **Session** | Single conversation thread | Tracks all events, state in one chat |
| **State (session.state)** | Current active session | Temporary scratchpad for in-progress data |
| **Memory** | Cross-session repository | Persistent knowledge from past interactions |

### State Key Prefixes in ADK
```python
# Session-specific (default, no prefix)
state["task_status"] = "in_progress"

# User-specific (persists across all sessions for this user)
state["user:preferred_language"] = "English"
state["user:login_count"] = 5

# Application-wide (shared across all users)
state["app:model_version"] = "gemini-2.5-flash"

# Temporary (current turn only, not persisted)
state["temp:validation_needed"] = True
```

## Implementation with Google ADK

### Session Service Options
```python
# Option 1: In-Memory (development/testing only — not persistent)
from google.adk.sessions import InMemorySessionService
session_service = InMemorySessionService()

# Option 2: Database (production — SQLite, PostgreSQL, etc.)
from google.adk.sessions import DatabaseSessionService
session_service = DatabaseSessionService(db_url="sqlite:///./agent_data.db")

# Option 3: Vertex AI (scalable production on Google Cloud)
from google.adk.sessions import VertexAiSessionService
session_service = VertexAiSessionService(
    project="your-gcp-project-id",
    location="us-central1"
)
```

### Simple State Management with output_key
```python
from google.adk.agents import LlmAgent
from google.adk.sessions import InMemorySessionService, Session
from google.adk.runners import Runner
from google.genai.types import Content, Part

# Agent saves its response to session state automatically
greeting_agent = LlmAgent(
    name="Greeter",
    model="gemini-2.5-flash",
    instruction="Generate a short, friendly greeting.",
    output_key="last_greeting"  # Automatically saves response to state
)

app_name, user_id, session_id = "state_app", "user1", "session1"
session_service = InMemorySessionService()
runner = Runner(
    agent=greeting_agent,
    app_name=app_name,
    session_service=session_service
)

session = session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id
)

print(f"Initial state: {session.state}")

# Run the agent
user_message = Content(parts=[Part(text="Hello")])
for event in runner.run(user_id=user_id, session_id=session_id, new_message=user_message):
    if event.is_final_response():
        print("Agent responded.")

# Check updated state after run
updated_session = session_service.get_session(app_name, user_id, session_id)
print(f"State after agent run: {updated_session.state}")
# Output: {'last_greeting': 'Hello! How can I help you today?'}
```

### Tool-Based State Updates
```python
import time
from google.adk.tools.tool_context import ToolContext

def log_user_login(tool_context: ToolContext) -> dict:
    """Updates session state upon user login. Called automatically by agent."""
    state = tool_context.state

    # Update multiple state keys atomically
    login_count = state.get("user:login_count", 0) + 1
    state["user:login_count"] = login_count
    state["task_status"] = "active"
    state["user:last_login_ts"] = time.time()
    state["temp:validation_needed"] = True  # Temporary flag for this turn

    return {
        "status": "success",
        "message": f"User login tracked. Total logins: {login_count}."
    }
```

### Long-Term Memory with Vector Store
```python
from google.adk.agents import LlmAgent
from google.adk.memory import InMemoryMemoryService
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService

# Memory service stores knowledge across sessions
memory_service = InMemoryMemoryService()

# Agent with memory retrieval capability
agent = LlmAgent(
    name="MemoryAgent",
    model="gemini-2.5-flash",
    instruction="""You are a helpful assistant with persistent memory.
    Before answering, check if relevant information exists in your memory.
    After learning something new about the user, remember it for future sessions."""
)

runner = Runner(
    agent=agent,
    app_name="memory_app",
    session_service=InMemorySessionService(),
    memory_service=memory_service  # Attach memory service
)
```

### Conversation Memory with LangChain
```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

# In-memory session store
session_store: dict = {}

def get_session_history(session_id: str) -> ChatMessageHistory:
    if session_id not in session_store:
        session_store[session_id] = ChatMessageHistory()
    return session_store[session_id]

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
])

# RunnableWithMessageHistory replaces deprecated ConversationChain
conversation = RunnableWithMessageHistory(
    prompt | llm,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
)

config = {"configurable": {"session_id": "session1"}}

response1 = conversation.invoke({"input": "My name is Alice and I prefer Python."}, config=config)
response2 = conversation.invoke({"input": "What's my name and preferred language?"}, config=config)
# response2 correctly recalls: "Your name is Alice and you prefer Python."
```

### Vector-Based Long-Term Memory
```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings, ChatGoogleGenerativeAI
from langchain_chroma import Chroma

# Initialize embedding model
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")

# Vector store for semantic memory retrieval
vectorstore = Chroma(embedding_function=embeddings, persist_directory="./agent_memory")
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# Store memories as documents
from langchain_core.documents import Document
vectorstore.add_documents([
    Document(
        page_content="User mentioned their budget is $50,000 for Q3.",
        metadata={"input": "What's my budget for this project?"}
    )
])

# Retrieve semantically relevant memories
relevant_docs = retriever.invoke("project finances")
for doc in relevant_docs:
    print(doc.page_content)
# Returns: context about the $50,000 budget
```

## Memory Management Strategies

### Context Window Optimization
```python
# Strategy 1: Sliding window — keep only recent N messages
recent_messages = conversation_history[-10:]  # Last 10 messages

# Strategy 2: Summarization — compress older history
def summarize_history(messages: list, model) -> str:
    history_text = "\n".join([f"{m['role']}: {m['content']}" for m in messages])
    summary = model.generate_content(f"Summarize this conversation:\n{history_text}")
    return summary.text

# Strategy 3: Key information extraction — extract only critical facts
def extract_key_facts(messages: list, model) -> str:
    facts = model.generate_content(
        "Extract only the key facts, preferences, and decisions from this conversation:\n"
        + "\n".join([m['content'] for m in messages])
    )
    return facts.text
```

### Hybrid Memory Architecture
```python
class HybridMemoryAgent:
    """Combines short-term context with long-term vector memory."""

    def __init__(self, model, vectorstore):
        self.model = model
        self.vectorstore = vectorstore
        self.session_history = []

    def respond(self, user_input: str) -> str:
        # 1. Retrieve relevant long-term memories
        long_term_context = self.vectorstore.similarity_search(user_input, k=3)
        long_term_text = "\n".join([doc.page_content for doc in long_term_context])

        # 2. Build prompt with both short-term and long-term memory
        recent_context = self.session_history[-5:]  # Short-term: last 5 turns
        prompt = f"""Long-term memory:\n{long_term_text}

Recent conversation:
{chr(10).join([f'{m["role"]}: {m["content"]}' for m in recent_context])}

User: {user_input}"""

        # 3. Generate response
        response = self.model.generate_content(prompt)

        # 4. Update memories
        self.session_history.append({"role": "user", "content": user_input})
        self.session_history.append({"role": "assistant", "content": response.text})

        # 5. Store significant exchanges in long-term memory
        if len(user_input) > 50:  # Store substantial interactions
            self.vectorstore.add_texts([f"User asked: {user_input}\nAgent said: {response.text}"])

        return response.text
```

## Key Takeaways

- **Two memory types**: Short-term (context window, session state) and long-term (external storage, vector DB)
- **ADK triad**: Session (conversation thread), State (session scratchpad), Memory (cross-session repository)
- **State prefixes**: `user:` (cross-session), `app:` (all users), `temp:` (current turn), no prefix (session-only)
- **SessionService choices**: InMemorySessionService (testing), DatabaseSessionService (production), VertexAiSessionService (GCP scale)
- **Semantic retrieval**: Vector embeddings enable finding relevant memories by meaning, not just keywords
- **Hybrid memory**: Combine short-term session context with long-term vector retrieval for optimal performance

## Anti-Patterns to Avoid

- **Context window overflow**: Filling the context with all history degrades performance and increases cost
- **No long-term storage**: Agents that can't persist knowledge must relearn everything each session
- **Memory without retrieval**: Storing memories is useless without effective retrieval logic
- **Storing everything**: Indiscriminate memory storage creates noise that degrades retrieval quality
- **Privacy violations**: Storing sensitive user data without appropriate access controls and retention policies

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Persistent knowledge agent | Memory Management + **RAG** + **Tool Use** |
| Long-running stateful workflows | Memory Management + **Goal Setting** + **Planning** |
| Personalized user agent | Memory Management + **Prompt Chaining** + **Learning & Adaptation** |
| Session-aware multi-agent system | Memory Management + **Multi-Agent Collaboration** |

## References

- Google ADK Session & Memory Documentation: https://google.github.io/adk-docs/sessions/
- LangChain Memory: https://python.langchain.com/docs/modules/memory/
- ChromaDB Vector Store: https://www.trychroma.com/
- Vertex AI Agent Engine Memory: https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine
