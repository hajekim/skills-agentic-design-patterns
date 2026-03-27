---
name: appendix-agentspace
description: This skill should be used when the user wants to build agents with "Google AgentSpace", "no-code agent builder", "agent designer Google", "AI Applications Google Cloud", "Google Cloud no-code agent", "enterprise agent platform", "no-code AI agent", "AgentSpace prompt gallery", "Google Agent Designer", "Knowledge Graph agent", "Google AI Applications", "Google no-code agent", "deploy agent without code", "Google enterprise AI platform", "AgentSpace tutorial", "Google AI agent builder", or deploy agents within an organization without writing code. Also responds to Korean: "AgentSpace 사용법", "노코드 에이전트 빌더", "에이전트 디자이너", "엔터프라이즈 에이전트 플랫폼", "코드 없이 에이전트 배포", "AI 애플리케이션 만들기", "AgentSpace 설정", "노코드 에이전트 만들기", "코딩 없이 에이전트 만들기", "Google 에이전트 플랫폼", "AgentSpace 써보고 싶어". Also responds to Japanese: "AgentSpaceの使い方", "ノーコードエージェント", "コードなしでエージェントを作りたい", "Google Cloudエージェント", "企業向けエージェントプラットフォーム", "AgentSpaceの設定", "ノーコードAIエージェント", "Googleエージェントビルダー", "プロンプトギャラリー", "AgentSpaceを使いたい" Also responds to Chinese: "AgentSpace使用指南", "无代码智能体构建器", "不写代码就能建智能体", "Google Cloud智能体", "企业级智能体平台", "AgentSpace配置", "无代码AI智能体", "Google智能体构建器", "提示词库", "想用AgentSpace".. Apply this skill to design or implement no-code agent deployments using Google AgentSpace's Agent Designer.
version: 1.0.0
---

# Appendix D - Google AgentSpace

## Overview

**Google AgentSpace** is a no-code enterprise agent platform that enables organizations to build and deploy AI agents without deep programming expertise. It provides a graphical user interface for agent construction, integrating Google Cloud's AI capabilities, enterprise datastores, and Knowledge Graph into a unified, configurable system.

AgentSpace abstracts the underlying technical complexity — autonomous reasoning, knowledge graph mapping, data source integration — into a visual interface where users define agent behavior through prompts, configure knowledge sources, and deploy agents accessible via web interface.

**Core Principle:** Not all agents need code. AgentSpace enables business teams to deploy context-aware AI agents by configuring prompts and data sources — democratizing agent development beyond engineering teams.

## When This Skill Applies

Activate this pattern when:
- Business teams (not just developers) need to deploy AI agents
- Agents need deep integration with enterprise Google Workspace data
- The use case is well-defined and can be specified through prompts
- Rapid prototyping is needed before committing to code-based agents
- Organization uses Google Cloud and wants native integration
- The agent needs Knowledge Graph enrichment for factual accuracy
- Analytics and monitoring of agent usage is required without custom instrumentation

**Rule of thumb:** If the agent's behavior can be described in a system prompt and the data sources are Google Cloud datastores, AgentSpace is faster than code. If you need custom logic, loops, or non-Google integrations, use ADK or LangGraph.

## AgentSpace Architecture

```
┌─────────────────────────────────────────────────────┐
│                 Google AgentSpace                    │
│                                                      │
│  ┌──────────────┐  ┌─────────────┐  ┌────────────┐ │
│  │ Prompt Gallery│  │  Agent      │  │ Analytics  │ │
│  │ (Google-made +│  │  Designer   │  │ Dashboard  │ │
│  │  Custom)      │  │  (No-code)  │  │            │ │
│  └──────┬───────┘  └──────┬──────┘  └────────────┘ │
│         │                 │                          │
│  ┌──────▼─────────────────▼──────────────────────┐  │
│  │              Agent Runtime                     │  │
│  │  (Gemini model + Prompt + Knowledge + Data)    │  │
│  └──────────────────────┬─────────────────────────┘  │
│                         │                            │
│  ┌──────────┐  ┌────────▼────────┐  ┌────────────┐  │
│  │ Google   │  │ Connected Data  │  │ Web UI     │  │
│  │ Knowledge│  │ Stores          │  │ (Chat)     │  │
│  │ Graph    │  │ (BigQuery, GCS) │  │            │  │
│  └──────────┘  └─────────────────┘  └────────────┘  │
└─────────────────────────────────────────────────────┘
```

## AgentSpace Key Components

### Prompt Gallery
Pre-assembled prompts from Google and your organization:
- `goog_analyze_data` — Analyze and visualize data
- `goog_book_time_off` — Book time off via HR systems
- `goog_chat_with_content` — Chat with document content
- `goog_deep_research` — Deep research on a topic
- `goog_draft_email` — Draft professional emails
- `goog_explain_technical_documentation` — Explain technical docs
- `goog_generate_code` — Generate code from requirements
- `goog_generate_image` — Generate images

You can also create custom prompts that appear in the gallery with your organization's branding.

### Agent Configuration
Each agent is configured with:
- **Name and Display Name**: How the agent appears to users
- **Description**: What triggers this agent capability
- **Prompt Type**: User query (conversational) or system instruction
- **Activation Behavior**: New session vs. persistent session
- **Knowledge Sources**: Connected datastores, Knowledge Graph

### Knowledge Graph Integration
- **Google Cloud Knowledge Graph**: Enriches search with external data context
- **Private Knowledge Graph**: Internal organizational data for context-aware answers (up to 24h to re-generate after enabling)

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map AgentSpace deployment requirements:
1. Who are the users? (Business teams, customers, internal staff)
2. What tasks should the agent perform? (Research, document Q&A, data analysis, code generation)
3. What data sources does the agent need? (Google Drive, BigQuery, internal docs)
4. Does the agent need Knowledge Graph enrichment for factual grounding?
5. What metrics matter? (Usage volume, satisfaction, task completion rate)

### PLAN
Design the AgentSpace configuration:
1. Select or create prompts from the Prompt Gallery (or create custom)
2. Choose activation behavior: new session per query or persistent conversation
3. Configure data store connections: which enterprise data sources to connect
4. Enable Knowledge Graph: Google Cloud KG for external facts, Private KG for internal data
5. Plan the web interface: how will users access the agent?

### ACTION
Deploy and monitor the agent:
1. Access Google AI Applications in Google Cloud Console
2. Create a new app in Agent Designer
3. Configure prompts from gallery or create custom prompts
4. Connect data sources (datastores) for grounding
5. Enable Knowledge Graph integration
6. Test via the Preview interface
7. Deploy web interface for user access
8. Monitor via Analytics dashboard

## Implementation: Agent Designer Configuration

### Custom Prompt Configuration
```yaml
# Example: Custom prompt configuration (defined via AgentSpace UI)
# This shows the structure of a custom prompt as configured in Agent Designer

prompt:
  name: "analyze_quarterly_report"
  display_name: "Analyze Quarterly Report"
  title: "Financial Report Analyst"
  description: "Analyzes quarterly financial reports and extracts key metrics"

  prompt_type: "user_query"

  user_query: |
    You are a senior financial analyst. When given a quarterly financial report:
    1. Extract key financial metrics (revenue, profit, YoY growth)
    2. Identify top 3 risks mentioned
    3. Summarize executive guidance for next quarter
    4. Rate overall business health as: Strong / Stable / Concerning

    Present findings in a structured format with clear section headers.

  activation_behavior: "new_session"
  enabled: true
```

### Programmatic AgentSpace Integration (Python SDK)
```python
from google.cloud import discoveryengine_v1alpha as discoveryengine
from google.api_core.client_options import ClientOptions

def create_agentspace_client(project_id: str, location: str = "global") -> discoveryengine.ConversationalSearchServiceClient:
    """Create an AgentSpace client for programmatic integration.

    Args:
        project_id: Google Cloud project ID
        location: AgentSpace deployment region

    Returns:
        Configured ConversationalSearchServiceClient
    """
    client_options = ClientOptions(
        api_endpoint=f"{location}-discoveryengine.googleapis.com"
    )
    return discoveryengine.ConversationalSearchServiceClient(
        client_options=client_options
    )

def query_agentspace_agent(
    project_id: str,
    engine_id: str,
    query: str,
    conversation_id: str = None,
    location: str = "global"
) -> dict:
    """Query an AgentSpace agent programmatically.

    Args:
        project_id: Google Cloud project ID
        engine_id: The AgentSpace app/engine ID
        query: User query to send to the agent
        conversation_id: Existing conversation ID for multi-turn (None for new)
        location: Deployment region

    Returns:
        Agent response with answer and sources
    """
    client = create_agentspace_client(project_id, location)

    # Build the serving config path
    serving_config = client.serving_config_path(
        project=project_id,
        location=location,
        data_store=engine_id,
        serving_config="default_config"
    )

    # Prepare the conversation request
    request = discoveryengine.ConverseConversationRequest(
        name=f"projects/{project_id}/locations/{location}/collections/default_collection/engines/{engine_id}/conversations/-",
        query=discoveryengine.TextInput(input=query),
        serving_config=serving_config,
        safe_search=True
    )

    response = client.converse_conversation(request)

    return {
        "answer": response.reply.summary.summary_text if response.reply.summary else response.reply.text,
        "sources": [
            {
                "title": ref.document_metadata.title,
                "uri": ref.document_metadata.uri
            }
            for ref in response.search_results[:3]
        ] if response.search_results else []
    }

# Usage
result = query_agentspace_agent(
    project_id="my-gcp-project",
    engine_id="my-agentspace-app",
    query="What were our Q3 revenue figures compared to Q2?"
)
print(f"Answer: {result['answer']}")
for source in result['sources']:
    print(f"Source: {source['title']} — {source['uri']}")
```

### AgentSpace + ADK Hybrid Pattern
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext

# Pattern: Use AgentSpace as a knowledge tool within an ADK agent
# AgentSpace handles enterprise knowledge retrieval
# ADK handles complex reasoning and tool orchestration

def query_enterprise_knowledge(
    question: str,
    tool_context: ToolContext
) -> dict:
    """Query the AgentSpace knowledge base for enterprise-grounded answers.

    Args:
        question: The question to answer using enterprise data

    Returns:
        Answer grounded in enterprise documents with source references
    """
    # In production: call AgentSpace API as shown above
    result = query_agentspace_agent(
        project_id="my-gcp-project",
        engine_id="enterprise-knowledge-base",
        query=question
    )
    return result

# ADK agent that uses AgentSpace as a knowledge tool
enterprise_agent = LlmAgent(
    name="EnterpriseAssistant",
    model="gemini-2.5-flash",
    instruction="""You are an enterprise AI assistant.
    For questions about company data, policies, or documents, use the query_enterprise_knowledge tool.
    For general reasoning and analysis, use your own capabilities.
    Always cite sources when drawing from enterprise knowledge.""",
    tools=[FunctionTool(query_enterprise_knowledge)]
)
```

## Key Takeaways

- **No-code democratization**: AgentSpace enables non-engineers to deploy AI agents — business teams can configure agents for their own workflows
- **Prompt Gallery as the API**: Google's pre-assembled prompts are your starting point — customize them before building from scratch
- **Knowledge Graph = factual grounding**: Enabling Knowledge Graph dramatically reduces hallucinations on factual questions by grounding responses in verified data
- **Datastore integration is the differentiator**: Connecting enterprise datastores transforms a generic chatbot into a company-specific knowledge agent
- **Analytics without code**: Built-in usage analytics lets you measure agent adoption and identify improvement areas without custom instrumentation
- **Hybrid with ADK**: For complex reasoning beyond what prompts can achieve, combine AgentSpace (knowledge retrieval) with ADK (orchestration)

## Anti-Patterns to Avoid

- **Over-relying on no-code for complex logic**: Conditional workflows, loops, and multi-step reasoning require ADK or LangGraph — AgentSpace is not a substitute
- **Skipping Knowledge Graph setup**: Without grounding, agents hallucinate facts — always enable appropriate Knowledge Graph for factual domains
- **Generic prompts**: Vague prompts like "be helpful" produce inconsistent results — write specific, task-focused prompts with output format instructions
- **No testing before deployment**: Always use the Preview interface extensively with edge cases before enabling for users
- **Ignoring Analytics**: AgentSpace provides usage data — review it regularly to detect misuse patterns, low satisfaction, or missing capabilities

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| No-code multi-agent orchestration | AgentSpace + **Multi-Agent Collaboration** + **Routing** |
| Enterprise knowledge + no-code agent | AgentSpace + **RAG** + **Memory Management** |
| AgentSpace + custom ADK agent | AgentSpace + **Agentic Frameworks** + **Tool Use** |
| Safe enterprise agent deployment | AgentSpace + **Guardrails** + **Human-in-the-Loop** |

## References

- Create a no-code agent with Agent Designer: https://cloud.google.com/agentspace/agentspace-enterprise/docs/agent-designer
- Google Cloud Skills Boost — Build a Gen AI Agent with AgentSpace: https://www.cloudskillsboost.google/
- Discovery Engine API (AgentSpace backend): https://cloud.google.com/generative-ai-app-builder/docs/
- Google Knowledge Graph: https://cloud.google.com/enterprise-knowledge-graph/docs/
