---
name: rag
description: This skill should be used when the user wants to "retrieval augmented generation", "RAG", "ground agent in documents", "knowledge base search", "vector search for agents", "semantic document retrieval", "augment LLM with external knowledge", "document QA", "knowledge grounding", "enterprise knowledge agent", "PDF question answering", "RAG pipeline", "chat with documents", "document chatbot", "grounded responses", "private data agent", "knowledge-grounded agent", "company knowledge base agent", or build agents that retrieve relevant context from external knowledge sources before generating responses. Also responds to Korean: "검색 증강 생성", "외부 문서 기반 답변", "지식 베이스 검색", "벡터 검색 에이전트", "기업 지식 에이전트", "PDF 질의응답", "RAG 만들어줘", "RAG 구성", "RAG 써보고 싶어", "RAG 설정", "문서 기반 챗봇 만들어줘", "내 문서로 답변하는 에이전트", "지식베이스 검색 에이전트". Also responds to Japanese: "検索拡張生成", "ドキュメントベース回答", "知識ベース検索", "ドキュメントチャットボット", "社内文書から回答", "RAGを作りたい", "ベクター検索エージェント", "PDFに質問したい", "自分の文書で答えるエージェント", "RAGパイプライン" Also responds to Chinese: "检索增强生成", "基于文档的回答", "知识库检索", "文档聊天机器人", "用公司内部文档回答", "想做RAG", "向量检索智能体", "对PDF提问", "用我的文档来回答", "RAG流水线".. Apply this skill to design or implement the Retrieval-Augmented Generation (RAG) agentic design pattern.
version: 1.0.0
---

# Retrieval-Augmented Generation (RAG) Pattern

## Overview

The **Retrieval-Augmented Generation (RAG) Pattern** grounds agent responses in external, up-to-date knowledge by retrieving relevant documents or data before generating a response. Rather than relying solely on knowledge baked into model weights (which has a training cutoff and may hallucinate), RAG agents dynamically fetch the most relevant context and use it to produce accurate, grounded responses.

**Core Principle:** Don't hallucinate what you can retrieve — anchor every response in verifiable, retrieved knowledge.

## When This Skill Applies

Activate this pattern when:
- The agent needs domain-specific knowledge not in the base LLM's training
- Responses must be grounded in authoritative documents (legal, medical, technical)
- Information changes frequently and training data is stale
- Users need citations and sources for claims made
- Private or proprietary knowledge must be accessed securely
- Reducing hallucination is a critical requirement

**Rule of thumb:** If the answer exists in a document and you need it to be accurate and verifiable — use RAG.

## RAG Architecture

### Standard RAG Pipeline
```
Query → [Embedding] → Vector Search → Retrieved Chunks
                                              ↓
                                   LLM + Retrieved Context
                                              ↓
                                     Grounded Response
```

### Advanced RAG Variants
| Variant | Description | Use Case |
|---------|-------------|----------|
| **Naive RAG** | Embed query → retrieve → generate | Simple Q&A |
| **Advanced RAG** | Query expansion, reranking, filtering | High-accuracy enterprise |
| **Modular RAG** | Pluggable retrieval strategies | Complex, multi-source |
| **Agentic RAG** | Agent decides when/what to retrieve | Dynamic reasoning |
| **Graph RAG** | Knowledge graph + vector retrieval | Complex entity relationships |

## DEFINE → PLAN → ACTION Workflow

### DEFINE
Map the knowledge retrieval requirements:
1. What knowledge sources need to be indexed? (PDFs, databases, APIs, web)
2. What query types will users ask? (factual, comparative, analytical)
3. What is the required accuracy/hallucination tolerance?
4. How frequently does the knowledge change? (indexing strategy)
5. What metadata is available for filtering? (date, source, category)

### PLAN
Design the RAG architecture:
1. Choose embedding model (text-embedding-004, embedding-001, etc.)
2. Select vector store (ChromaDB, Pinecone, Vertex AI Vector Search, FAISS)
3. Design chunking strategy: size, overlap, semantic vs. fixed
4. Plan retrieval: top-k, similarity threshold, MMR (Maximal Marginal Relevance)
5. Design the generation prompt: how to instruct LLM to use retrieved context

### ACTION
Implement the RAG pipeline:
1. Index documents: load → chunk → embed → store in vector DB
2. Implement retrieval: embed query → similarity search → rerank
3. Construct augmented prompt: retrieved context + user query
4. Generate grounded response
5. Add citations: link claims back to source documents

## Implementation: Basic RAG with LangChain + Chroma

### Document Indexing Pipeline
```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings, ChatGoogleGenerativeAI
from langchain_chroma import Chroma
from langchain_community.document_loaders import PyPDFLoader, DirectoryLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import PromptTemplate

# 1. Load documents
def load_and_index_documents(source_path: str, persist_dir: str) -> Chroma:
    """Load documents and create a searchable vector index."""

    # Load PDFs from directory
    loader = DirectoryLoader(source_path, glob="**/*.pdf", loader_cls=PyPDFLoader)
    documents = loader.load()
    print(f"Loaded {len(documents)} document pages")

    # Split into chunks (balance between context and precision)
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,      # Characters per chunk
        chunk_overlap=200,    # Overlap to preserve context across chunks
        separators=["\n\n", "\n", " ", ""]
    )
    chunks = splitter.split_documents(documents)
    print(f"Created {len(chunks)} chunks")

    # Create embeddings and index
    embeddings = GoogleGenerativeAIEmbeddings(model="models/text-embedding-004")
    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory=persist_dir
    )
    vectorstore.persist()
    print(f"Indexed {len(chunks)} chunks to {persist_dir}")
    return vectorstore

# 2. Create RAG chain
def create_rag_agent(persist_dir: str):
    """Create a RAG-powered Q&A agent from an existing index."""
    embeddings = GoogleGenerativeAIEmbeddings(model="models/text-embedding-004")
    vectorstore = Chroma(persist_directory=persist_dir, embedding_function=embeddings)

    # Configure retrieval
    retriever = vectorstore.as_retriever(
        search_type="mmr",  # Maximal Marginal Relevance for diversity
        search_kwargs={"k": 5, "fetch_k": 20}  # Retrieve 5 from 20 candidates
    )

    # Custom prompt that grounds response in context
    prompt_template = """You are a knowledgeable assistant. Use the following retrieved
context to answer the question accurately. If the answer is not in the context,
say "I don't have information about this in my knowledge base" — do not guess.

Always cite the specific source document and page number when making claims.

Retrieved Context:
{context}

Question: {question}

Answer (with citations):"""

    prompt = PromptTemplate(
        template=prompt_template,
        input_variables=["context", "question"]
    )

    llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

    def format_docs(docs):
        return "\n\n".join(doc.page_content for doc in docs)

    rag_chain = (
        {"context": retriever | format_docs, "question": RunnablePassthrough()}
        | prompt
        | llm
        | StrOutputParser()
    )

    return rag_chain

# Usage
vectorstore = load_and_index_documents("./documents", "./chroma_db")
rag_agent = create_rag_agent("./chroma_db")

answer = rag_agent.invoke("What are the key principles of agentic design patterns?")
print(answer)
```

### Advanced RAG with Query Expansion
```python
from langchain_google_genai import ChatGoogleGenerativeAI, GoogleGenerativeAIEmbeddings
from langchain_chroma import Chroma
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

embeddings = GoogleGenerativeAIEmbeddings(model="models/text-embedding-004")
vectorstore = Chroma(persist_directory="./chroma_db", embedding_function=embeddings)
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)

# Step 1: Query expansion — generate multiple search variations
query_expansion_prompt = ChatPromptTemplate.from_template("""
Generate 3 different search queries to retrieve relevant information for:
"{question}"

Output exactly 3 queries, one per line, no numbering or bullets."""
)

def expand_and_retrieve(question: str, k_per_query: int = 3) -> list:
    """Expand query into multiple variants and retrieve from each."""
    # Generate query variants
    chain = query_expansion_prompt | llm | StrOutputParser()
    expanded = chain.invoke({"question": question})
    queries = [question] + [q.strip() for q in expanded.strip().split('\n') if q.strip()]

    # Retrieve from all query variants, deduplicate
    all_docs = []
    seen_content = set()
    for query in queries[:4]:  # Original + 3 expansions
        docs = vectorstore.similarity_search(query, k=k_per_query)
        for doc in docs:
            content_hash = hash(doc.page_content[:100])
            if content_hash not in seen_content:
                seen_content.add(content_hash)
                all_docs.append(doc)

    return all_docs[:8]  # Return top 8 unique documents

# Step 2: Reranking — score retrieved docs for relevance
def rerank_documents(query: str, docs: list) -> list:
    """Score and rerank documents by relevance to query."""
    if not docs:
        return []

    scored_docs = []
    for doc in docs:
        score_response = llm.invoke(
            f"""Score the relevance of this document excerpt to the query on a scale of 0-10.
            Query: {query}
            Document: {doc.page_content[:300]}
            Output only a number 0-10."""
        )
        try:
            score = float(score_response.content.strip())
        except ValueError:
            score = 5.0
        scored_docs.append((score, doc))

    scored_docs.sort(key=lambda x: x[0], reverse=True)
    return [doc for _, doc in scored_docs[:5]]  # Return top 5

# Step 3: Generate with grounded context
def rag_with_expansion_and_reranking(question: str) -> dict:
    """Full advanced RAG pipeline."""
    # Retrieve
    docs = expand_and_retrieve(question)
    # Rerank
    reranked_docs = rerank_documents(question, docs)

    # Build context with metadata
    context = "\n\n".join([
        f"[Source: {doc.metadata.get('source', 'Unknown')}, "
        f"Page: {doc.metadata.get('page', '?')}]\n{doc.page_content}"
        for doc in reranked_docs
    ])

    # Generate grounded response
    generation_prompt = f"""Answer based on the retrieved context. Cite sources.

Context:
{context}

Question: {question}

Grounded Answer:"""

    response = llm.invoke(generation_prompt)

    return {
        "answer": response.content,
        "sources": [
            {"source": d.metadata.get("source"), "page": d.metadata.get("page")}
            for d in reranked_docs
        ],
        "docs_retrieved": len(docs),
        "docs_used": len(reranked_docs)
    }
```

## Implementation: Agentic RAG with Google ADK

### ADK Agent with Dynamic RAG
```python
from google.adk.agents import LlmAgent
from google.adk.tools import FunctionTool
from langchain_google_genai import GoogleGenerativeAIEmbeddings
from langchain_chroma import Chroma

# Initialize knowledge base
embeddings = GoogleGenerativeAIEmbeddings(model="models/text-embedding-004")
knowledge_base = Chroma(persist_directory="./knowledge_db", embedding_function=embeddings)

def search_knowledge_base(query: str, top_k: int = 5) -> dict:
    """Search the knowledge base for relevant information.

    Args:
        query: Search query to find relevant documents
        top_k: Number of documents to retrieve (1-10)

    Returns:
        Retrieved documents with content and source metadata
    """
    docs = knowledge_base.similarity_search(query, k=min(top_k, 10))

    if not docs:
        return {
            "found": False,
            "message": "No relevant documents found in knowledge base",
            "results": []
        }

    return {
        "found": True,
        "count": len(docs),
        "results": [
            {
                "content": doc.page_content,
                "source": doc.metadata.get("source", "Unknown"),
                "page": doc.metadata.get("page", None),
                "relevance_rank": i + 1
            }
            for i, doc in enumerate(docs)
        ]
    }

def add_to_knowledge_base(content: str, source: str, category: str = "general") -> dict:
    """Add new information to the knowledge base for future retrieval.

    Args:
        content: Text content to add to the knowledge base
        source: Source identifier (URL, filename, etc.)
        category: Category tag for the content

    Returns:
        Confirmation of successful indexing
    """
    from langchain.schema import Document
    doc = Document(
        page_content=content,
        metadata={"source": source, "category": category}
    )
    knowledge_base.add_documents([doc])
    return {
        "status": "indexed",
        "source": source,
        "content_length": len(content)
    }

# Agentic RAG agent
rag_agent = LlmAgent(
    name="KnowledgeGroundedAgent",
    model="gemini-2.5-flash",
    instruction="""You are a knowledge-grounded assistant. ALWAYS follow this process:

    1. RETRIEVE: Before answering any factual question, call search_knowledge_base
       with a precise query targeting the specific information needed
    2. GROUND: Base your answer primarily on the retrieved content
    3. CITE: Reference the source documents in your response
    4. ACKNOWLEDGE: If no relevant documents are found, clearly state that
       your answer is based on general knowledge, not retrieved documents
    5. LEARN: If you discover important new information during a conversation,
       use add_to_knowledge_base to persist it for future use

    Never fabricate information — if it's not in the knowledge base and you're
    uncertain, say so.""",
    tools=[
        FunctionTool(search_knowledge_base),
        FunctionTool(add_to_knowledge_base)
    ]
)
```

### Vertex AI Vector Search (Production RAG)
```python
from google import genai
from google.genai.types import EmbedContentConfig
from google.cloud import aiplatform
from google.adk.tools import FunctionTool
import numpy as np

# Initialize Vertex AI
aiplatform.init(project="your-gcp-project", location="us-central1")

def vertex_vector_search(query: str, index_endpoint_id: str, top_k: int = 5) -> dict:
    """Search Vertex AI Vector Search index for similar documents.

    Args:
        query: Text query to search for
        index_endpoint_id: Vertex AI Vector Search endpoint ID
        top_k: Number of nearest neighbors to retrieve

    Returns:
        Nearest neighbor results with distances and metadata
    """
    client = genai.Client()

    # Embed the query
    result = client.models.embed_content(
        model="text-embedding-004",
        contents=query,
        config=EmbedContentConfig(task_type="RETRIEVAL_QUERY")
    )
    query_embedding = result.embeddings[0].values

    # Search the vector index
    endpoint = aiplatform.MatchingEngineIndexEndpoint(index_endpoint_id)
    results = endpoint.find_neighbors(
        deployed_index_id="your_deployed_index",
        queries=[query_embedding],
        num_neighbors=top_k
    )

    return {
        "results": [
            {"id": neighbor.id, "distance": neighbor.distance}
            for neighbor in results[0]
        ]
    }
```

## RAG Quality Metrics

### Evaluation Framework
```python
from google import genai

client = genai.Client()

def evaluate_rag_response(question: str, retrieved_docs: list, answer: str) -> dict:
    """Evaluate RAG response quality across key dimensions."""

    context = "\n".join([doc.page_content for doc in retrieved_docs])

    eval_prompt = f"""Evaluate this RAG response on 3 dimensions (score 1-5):

Question: {question}
Retrieved Context: {context[:1000]}
Generated Answer: {answer}

Score and explain:
1. Faithfulness: Is the answer supported by the context? (1=hallucinated, 5=fully grounded)
2. Relevance: Does the answer address the question? (1=off-topic, 5=precisely answers)
3. Completeness: Does the answer use all relevant context? (1=missed key info, 5=comprehensive)

Output JSON: {{"faithfulness": X, "relevance": X, "completeness": X, "explanation": "..."}}"""

    response = client.models.generate_content(model="gemini-2.5-flash", contents=eval_prompt)
    import json, re
    match = re.search(r'\{[\s\S]+\}', response.text)
    if match:
        return json.loads(match.group())
    return {"error": "Could not parse evaluation"}
```

## Key Takeaways

- **Grounding beats hallucination**: Retrieved facts are more reliable than model-generated "facts"
- **Chunking strategy matters**: 500-1000 char chunks with 20% overlap balances precision and context
- **MMR for diversity**: Maximal Marginal Relevance retrieves relevant but non-redundant chunks
- **Query expansion**: Generating multiple query variants catches more relevant documents
- **Cite sources**: Always link responses to source documents for verifiability
- **Agentic RAG**: Agents that decide when to retrieve (vs. answer from memory) are more efficient

## Anti-Patterns to Avoid

- **Stuffing everything**: Too much retrieved context degrades generation quality — be selective
- **No reranking**: Retrieved docs need relevance scoring, not just similarity distance
- **Static indexes**: Knowledge bases need regular updates to stay current
- **Ignoring retrieval failures**: When no relevant docs found, be explicit — don't hallucinate
- **Missing metadata**: Without source/date metadata, citations are impossible and trust is undermined

## Related Skills

Combine this skill with others for common scenarios:

| Scenario | Skills to Combine |
|----------|-------------------|
| Agentic RAG with tool augmentation | RAG + **Tool Use** + **Planning** |
| Knowledge-grounded agent with persistence | RAG + **Memory Management** |
| Evaluated RAG pipeline | RAG + **Evaluation** + **Reflection** |
| Enterprise knowledge agent | RAG + **MCP** + **Multi-Agent Collaboration** |

## References

- LangChain RAG Tutorial: https://python.langchain.com/docs/tutorials/rag/
- Google Vertex AI Vector Search: https://cloud.google.com/vertex-ai/docs/vector-search/
- ChromaDB Documentation: https://www.trychroma.com/
- RAG Survey Paper: https://arxiv.org/abs/2312.10997
- Google ADK RAG Integration: https://google.github.io/adk-docs/tools/
