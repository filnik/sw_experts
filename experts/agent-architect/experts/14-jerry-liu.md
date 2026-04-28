# Jerry Liu and Agentic RAG with LlamaIndex

## 1. Profile and Background

### Who is Jerry Liu

Jerry Liu is the Co-Founder and CEO of LlamaIndex, one of the most influential platforms in the ecosystem of applications based on Large Language Models (LLMs). His academic and professional path reflects a training deeply rooted in artificial intelligence and software engineering.

**Academic background.** Jerry graduated from Princeton in 2017 with a B.S.E. in Computer Science and a certificate in finance. During his studies he received the Shapiro Prize for Academic Excellence in 2014 and the Outstanding Senior Thesis Award in 2017, attesting to consistent academic excellence.

**Pre-LlamaIndex career.** Liu's professional path winds through some of the most relevant technology companies:

- **Apple** (2014) -- Software Engineering Intern
- **Quora** (2015) -- Software Engineering Intern, then Machine Learning Engineer (2017-2018)
- **Two Sigma** (2016) -- Software Engineering Intern in the quantitative sector
- **Uber** (2018-2021) -- AI Resident, then Research Scientist, where he worked on large-scale ML systems
- **Robust Intelligence** (2021-2023) -- ML Engineering Manager, focused on model robustness and safety

This trajectory -- from quantitative finance to ML research, to model safety -- has provided Liu with a unique perspective on the concrete problems of integrating data with language models.

### The Birth of LlamaIndex

On November 8, 2022, Jerry Liu published his first commit and first tweet about what was initially called **GPT Index**. The problem he wanted to solve was concrete: he was trying to build a sales bot, but ran into the limitations of the GPT-3 context window. He didn't want to resort to fine-tuning the model; rather, he wanted to find a way to store data separately, ingest it and retrieve it efficiently.

The initial project launched the **GPT Tree Index**, a way to organize information in a tree structure. The community's initial interest quickly led to expansion towards List Index, Keyword Index and, crucially, support for embedding and vector store in December 2022.

In January 2023 the project reached an inflection point, with accelerated growth that led to the formation of the company. In June 2023, Jerry Liu (CEO) and Simon Suo (CTO) -- former Uber colleague -- announced the official founding of LlamaIndex with an $8.5 million seed round led by Greylock. The project was renamed from GPT Index to LlamaIndex, marking the transition from open-source project to enterprise platform.

Since then LlamaIndex has evolved dramatically: from a simple RAG framework to a complete platform for agentic document processing and multi-agent orchestration, with the entire documentation now built around the AgentWorkflow concept.

---

## 2. Agentic RAG: The Concept

### From the Limits of Traditional RAG to the Agentic Revolution

Traditional Retrieval-Augmented Generation (RAG) follows a linear and rigid flow: **query -> retrieve -> generate -> respond**. The backend computes the embedding of the query, queries a vector store to find similar content via top-k similarity, composes a prompt with the query and the retrieved text, calls an LLM to generate a response, and returns the result to the user.

The advantages of this architecture are evident: it is fast, economical and easy to implement. However, Jerry Liu has identified critical limitations that compromise its effectiveness in real-world scenarios:

- **Low retrieval accuracy**: simple vector similarity matching does not capture complex semantic nuances
- **Wasted context window**: retrieved chunks may contain irrelevant information that dilutes the useful context
- **Severe hallucination problems**: without verification mechanisms, the model can generate responses not supported by the data
- **Single retrieval strategy**: limited to top-k similarity matching without adapting to the type of query
- **Fixed knowledge base**: restricted to retrieving from a single index at a time
- **No query understanding**: unable to determine when to search by metadata, content or specific files

### The Definition of Agentic RAG

Agentic RAG, as conceptualized by Jerry Liu, introduces an **active reasoning capability** into the retrieval process. The central loop becomes:

**Plan -> Retrieve -> Act -> Reflect -> Answer**

This paradigm allows the system to:

1. **Decompose complex problems** into manageable sub-tasks
2. **Execute multi-round retrieval**, iterating based on intermediate results
3. **Self-correct**, evaluating the quality of retrieved information before proceeding
4. **Dynamically select** the most appropriate retrieval strategy for each sub-query

As Jerry Liu himself stated on his blog: "Naive RAG is dead, agentic retrieval is the future." This is not simply an incremental optimization of traditional RAG, but a fundamental paradigm shift in the approach to retrieval.

The key transition is the move from **rule-based static retrieval** to **adaptive and intelligent problem-solving**, where AI agents can iterate on previous processes to optimize results over time. Agentic RAG applications extract data from multiple external knowledge bases and allow the use of external tools, while standard RAG pipelines connect an LLM to a single external dataset.

---

## 3. Architecture

### LlamaIndex's Two-Layer Architecture

LlamaIndex's specific implementation for Agentic RAG is based on a two-layer architecture that Jerry Liu described in detail in the DeepLearning.AI course and in technical blog posts.

### Document Agents (Lower Layer)

The document corpus is divided into smaller and more manageable documents. For each document a **dedicated agent** is created with specific capabilities:

- **Search via embedding**: the agent can search for specific information within its document using vector indices
- **Summarization**: the agent can generate summaries of the document content
- **Specialized queries**: each agent operates under a system prompt that requires mandatory tool use: *"You must ALWAYS use at least one of the tools provided when answering a question"*

Each document agent is equipped with specific tools:

- **Vector Query Engine Tool**: for pointed semantic searches within the document
- **Summary Query Engine Tool**: for questions that require a comprehensive view of the document
- **Knowledge Graph Query Engine Tool**: for navigating structured relationships between entities

### Meta-Agent / Top-Level Agent (Upper Layer)

Above the document agents operates a **meta-agent** (or top-level agent) that acts as orchestrator:

1. **Tool Retrieval**: the meta-agent identifies which document agents are relevant for the current query
2. **Chain-of-Thought Reasoning**: uses structured reasoning to plan the response strategy
3. **Synthesis**: aggregates the responses of the various document agents into a coherent and complete response

### The Four-Level System for Agentic Retrieval

In the blog post "RAG is Dead, Long Live Agentic Retrieval", Liu outlined a four-level progressive system:

**Level 1 -- Auto-Routed Single Index.** An LLM-powered agent determines which retrieval modality best suits each query. Options include:

- Chunk-based retrieval for specific information
- File metadata retrieval for specific questions about the document
- Whole-file retrieval for questions on broad topics

**Level 2 -- Multi-Index Routing.** A composite retriever uses LLM-based classification to decide which sub-index (or sub-indices) are relevant for cross-database searches.

**Level 3 -- Adaptive Retrieval Strategy.** The system combines both previous levels, allowing agents to intelligently select both indices AND retrieval modalities simultaneously.

**Level 4 -- Full Agentic Retrieval.** The agent has full control over the retrieval strategy, including the ability to iterate, re-formulate queries and combine results from multiple sources.

### Query Planning

Query planning is the mechanism through which the agent analyzes the intent of the query before retrieval. This includes:

- **Intent analysis**: understanding what the user is really asking
- **Tool selection**: identifying which retrieval methods are available and appropriate
- **Multi-step reasoning**: each level of the architecture makes independent routing decisions
- **Knowledge base diversity management**: handling heterogeneous document formats through specialized indices

---

## 4. AgentWorkflow: The New Framework

### Evolution from Workflow to AgentWorkflow

LlamaIndex has introduced **AgentWorkflow**, a system that makes it simple to build and orchestrate AI agent systems. Built on top of the already popular Workflow abstractions, AgentWorkflow represents the natural evolution of the framework towards a fully agentic paradigm.

The system creates **multi-step, asynchronous and event-driven** workflows with precision and maximum control. AgentWorkflow handles coordination between agents, state maintenance and much more, while maintaining the flexibility and extensibility of the original Workflows.

### Event-Driven Architecture

LlamaIndex Workflows uses an event-driven, async-first engine that orchestrates multi-step AI processes, agents and document pipelines. The architecture is based on event-driven `@step` functions with the Context API for clean and maintainable Python code.

The event system supports:

- **AgentInput**: agent activation events, when an agent begins processing a request
- **AgentStream**: streaming token/delta output, for real-time feedback
- **ToolCallResult**: results of tool execution, for monitoring agent actions
- **InputRequiredEvent**: trigger for human interaction (human-in-the-loop)

### Supported Agent Types

AgentWorkflow supports multiple agent architectures:

**FunctionAgent** -- Designed for LLMs with native function calling capabilities (such as OpenAI, Anthropic). Leverages structured function calling APIs to invoke tools precisely and reliably.

**ReActAgent** -- Compatible with any LLM implementation, including open-source models. Implements the Reasoning + Acting (ReAct) pattern, where the agent alternates phases of explicit reasoning and action:

```
Thought: I need to look up information about document X
Action: search_tool("document X topic Y")
Observation: [search results]
Thought: I found the information, now I can respond
Answer: [synthesized response]
```

**Custom Agents** -- The framework allows extending the base class to create custom agents with specific logic.

### Multi-Agent Orchestration

Agents define handoff relationships via the `can_handoff_to` parameter, enabling controlled transitions between specialized agents:

```python
from llama_index.core.agent.workflow import AgentWorkflow, FunctionAgent

research_agent = FunctionAgent(
    name="ResearchAgent",
    description="Expert in information research",
    system_prompt="You are a specialized research agent...",
    tools=[search_tool, index_tool],
    can_handoff_to=["WriteAgent", "AnalysisAgent"]
)

write_agent = FunctionAgent(
    name="WriteAgent",
    description="Expert in writing reports",
    system_prompt="You are a writing agent...",
    tools=[write_tool, format_tool],
    can_handoff_to=["ResearchAgent"]
)

workflow = AgentWorkflow(
    agents=[research_agent, write_agent],
    root_agent="ResearchAgent"
)
```

The workflow designates a `root_agent` as the entry point and handles agent selection during execution. Agents can delegate tasks to other agents transparently.

### State Management with Context

The system provides a shared Context object accessible to all tools and functions:

```python
# Reading the current state
current_state = await ctx.get("state")

# Updating the state
current_state["research_complete"] = True
await ctx.set("state", current_state)
```

This allows agents to maintain shared information across multi-step processes, ensuring coherence and continuity in workflow execution.

### Initialization Pattern

For simple workflows with a single agent, the framework offers factory methods:

```python
workflow = AgentWorkflow.from_tools_or_functions(
    [search_web, query_database],
    llm=OpenAI(model="gpt-4"),
    system_prompt="You are a research assistant..."
)

response = await workflow.run(user_msg="Analyze document X")
```

The human-in-the-loop integration leverages event streaming for approval and input collection workflows, allowing users to intervene at critical points in the agent's decision-making process.

---

## 5. Advanced RAG Patterns

### 5.1 Router RAG

Router RAG is the fundamental pattern taught by Jerry Liu in Lesson 2 of the DeepLearning.AI course. In this pattern, a **router** selects between two or more query engines to execute a query on a single document.

**How it works:**

1. The user asks a question
2. The router analyzes the nature of the question (factual vs. summary)
3. Routes the query to the appropriate query engine:
   - **Vector Query Engine**: for specific factual questions that require retrieval based on semantic similarity
   - **Summary Query Engine**: for questions that require an overall or summary view of the content

**Conceptual implementation:**

```python
from llama_index.core.query_engine import RouterQueryEngine
from llama_index.core.selectors import LLMSingleSelector
from llama_index.core.tools import QueryEngineTool

# Definition of tools for the router
vector_tool = QueryEngineTool.from_defaults(
    query_engine=vector_query_engine,
    description="Useful for specific questions about facts and details"
)

summary_tool = QueryEngineTool.from_defaults(
    query_engine=summary_query_engine,
    description="Useful for summarizing the document content"
)

# Creation of the router
router_engine = RouterQueryEngine(
    selector=LLMSingleSelector.from_defaults(),
    query_engine_tools=[vector_tool, summary_tool]
)
```

This pattern is the first step toward Agentic RAG because it introduces **decision-making** into the retrieval pipeline: the system no longer executes a fixed strategy, but actively chooses which approach to use.

### 5.2 Multi-Document RAG

Multi-Document RAG scales the architecture to multiple documents, creating an ecosystem of specialized agents. This is the pattern covered in Lesson 5 of the course.

**Architecture:**

1. **Indexing per document**: each document is indexed separately with its own vector store and summary index
2. **Document Agent per document**: each agent has access to both the vector query engine and the summary query engine of its document
3. **Top-Level Agent**: an orchestrator agent that handles routing to the appropriate document agents
4. **Tool Retrieval**: the meta-agent uses retrieval to identify which document agents are pertinent to the query

The central piece of parallel question answering on multiple documents is the **SubQuestionQueryEngine** with individual indices per document, where a query is first parsed into individual queries per index and responses are generated for each based on RAG.

### 5.3 Agentic RAG with Tool Use

Tool Calling (Lesson 3 of the course) extends the agent's capabilities by allowing the LLM not only to choose which function to execute, but also to **infer the arguments** to pass to the function. This goes beyond simple routing.

**Key features:**

- The LLM analyzes the query and determines which tool is most appropriate
- Automatically generates the necessary parameters for the tool
- Can combine results from multiple tools in a single response
- Supports custom user-defined tools

**Example tool definition:**

```python
from llama_index.core.tools import FunctionTool

def search_annual_reports(company: str, year: int, topic: str) -> str:
    """Search for information in a company's annual reports."""
    # Search logic
    ...

def compare_metrics(companies: list, metric: str) -> str:
    """Compare metrics across different companies."""
    # Comparison logic
    ...

search_tool = FunctionTool.from_defaults(fn=search_annual_reports)
compare_tool = FunctionTool.from_defaults(fn=compare_metrics)
```

The **Agent Reasoning Loop** (Lesson 4) takes this concept to the next level: the agent is capable of reasoning about tools across multiple steps, rather than in a single-shot execution. The agent can call a tool, observe the result, decide if further information is needed, call another tool, and only when it has gathered sufficient information provide the final response.

### 5.4 Sub-Question Decomposition

The Sub-Question Query Engine decomposes complex queries into sub-questions for each relevant data source, collects all intermediate responses and synthesizes a final response.

**LlamaIndex supports two approaches to decomposition:**

**Single-step decomposition**: transforms an initial query into a sub-question that can be answered more easily by the data. Useful when the original query is ambiguous or too broad.

**Multi-step decomposition**: divides an initial query into multiple sub-questions that can be solved independently. This approach is particularly powerful for comparative or multi-aspect questions.

**Practical example:**

Original query: *"Compare AI investment strategies of Google, Microsoft and Amazon in 2024"*

Decomposition into sub-questions:
1. "What is Google's AI investment strategy in 2024?"
2. "What is Microsoft's AI investment strategy in 2024?"
3. "What is Amazon's AI investment strategy in 2024?"

Each sub-question is routed to the appropriate document agent, and the responses are synthesized into a coherent comparative analysis.

```python
from llama_index.core.query_engine import SubQuestionQueryEngine
from llama_index.core.tools import QueryEngineTool

# Tool for each source
google_tool = QueryEngineTool.from_defaults(
    query_engine=google_engine,
    description="Annual reports and Google documentation"
)
microsoft_tool = QueryEngineTool.from_defaults(
    query_engine=microsoft_engine,
    description="Annual reports and Microsoft documentation"
)
amazon_tool = QueryEngineTool.from_defaults(
    query_engine=amazon_engine,
    description="Annual reports and Amazon documentation"
)

# Engine with automatic decomposition
engine = SubQuestionQueryEngine.from_defaults(
    query_engine_tools=[google_tool, microsoft_tool, amazon_tool]
)
```

---

## 6. Comparison: Traditional RAG vs Agentic RAG

The following table summarizes the architectural and functional differences between the two approaches:

| Dimension | Traditional RAG | Agentic RAG |
|---|---|---|
| **Flow** | Linear: query -> retrieve -> generate | Cyclic: plan -> retrieve -> act -> reflect -> answer |
| **Retrieval strategy** | Single (top-k similarity) | Multiple and adaptive (routing, decomposition, iteration) |
| **Data sources** | Single vector store | Multiple heterogeneous indices and knowledge bases |
| **Reasoning** | None (single-shot) | Multi-step with chain-of-thought |
| **Error handling** | No self-correction | Self-correction via reflection |
| **Query complexity** | Only simple and direct queries | Complex, comparative, multi-aspect queries |
| **Tool use** | Absent | Integrated (function calling, external APIs) |
| **Scalability** | Limited by the single index | Scalable via specialized document agents |
| **Computational cost** | Low (single LLM call) | Higher (multiple LLM calls for reasoning) |
| **Latency** | Low and predictable | Variable, depends on query complexity |
| **Accuracy** | Moderate, subject to retrieval noise | High, thanks to iterative retrieval and verification |
| **Customization** | Limited to vector store parameters | Highly customizable via custom tools and agents |

### When to use which approach

**Traditional RAG** remains appropriate for:
- Simple factual queries on a single document corpus
- Applications with stringent latency requirements
- Scenarios with limited computational budget
- Rapid prototypes and proofs-of-concept

**Agentic RAG** is necessary for:
- Complex queries that require multi-step reasoning
- Multi-document applications with heterogeneous sources
- Scenarios that require high accuracy and reliability
- Enterprise systems with the need to integrate with external tools
- Use cases where self-correction is fundamental

### Jerry Liu's Argument

Jerry Liu has clearly articulated his position: traditional RAG is a "hack", a temporary solution to a real problem. The natural evolution is towards agentic systems that can reason, plan and adapt. LlamaIndex itself has transformed from a RAG framework to a multi-agent platform, reflecting this vision. The documentation of the entire framework is now built around the AgentWorkflow concept, signaling that the future of retrieval is intrinsically agentic.

---

## 7. Practical Implementation

### Complete Pipeline: From Zero to Agentic RAG

Below is a complete practical implementation that illustrates the construction of a multi-document Agentic RAG system using LlamaIndex.

**Step 1: Setup and Indexing**

```python
from llama_index.core import (
    VectorStoreIndex,
    SummaryIndex,
    SimpleDirectoryReader,
    Settings
)
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

# Global configuration
Settings.llm = OpenAI(model="gpt-4")
Settings.embed_model = OpenAIEmbedding()

# Loading documents (one per file/source)
documents_map = {}
for doc_name in ["report_2023.pdf", "report_2024.pdf", "strategy.pdf"]:
    docs = SimpleDirectoryReader(
        input_files=[f"./data/{doc_name}"]
    ).load_data()
    documents_map[doc_name] = docs
```

**Step 2: Creating Document Agents**

```python
from llama_index.core.tools import QueryEngineTool
from llama_index.core.agent.workflow import FunctionAgent

document_agents = []

for doc_name, docs in documents_map.items():
    # Vector index for semantic search
    vector_index = VectorStoreIndex.from_documents(docs)
    vector_engine = vector_index.as_query_engine(similarity_top_k=3)

    # Summary index for summarization
    summary_index = SummaryIndex.from_documents(docs)
    summary_engine = summary_index.as_query_engine(
        response_mode="tree_summarize"
    )

    # Tools for the agent
    vector_tool = QueryEngineTool.from_defaults(
        query_engine=vector_engine,
        description=f"Search specific details in {doc_name}"
    )
    summary_tool = QueryEngineTool.from_defaults(
        query_engine=summary_engine,
        description=f"Summarize the content of {doc_name}"
    )

    # Creation of the document agent
    agent = FunctionAgent(
        name=f"Agent_{doc_name}",
        description=f"Expert on document {doc_name}",
        system_prompt=(
            f"You are a specialized agent for document {doc_name}. "
            "You must ALWAYS use at least one of the provided tools."
        ),
        tools=[vector_tool, summary_tool]
    )
    document_agents.append(agent)
```

**Step 3: Meta-Agent and Workflow**

```python
from llama_index.core.agent.workflow import AgentWorkflow

# The meta-agent can delegate to all document agents
meta_agent = FunctionAgent(
    name="MetaAgent",
    description="Orchestrator that coordinates research across documents",
    system_prompt=(
        "You are an orchestrator agent. Analyze the user's query, "
        "determine which documents are relevant, and delegate to the "
        "appropriate document agents. Synthesize the responses coherently."
    ),
    tools=[],
    can_handoff_to=[a.name for a in document_agents]
)

# Complete workflow configuration
all_agents = [meta_agent] + document_agents
workflow = AgentWorkflow(
    agents=all_agents,
    root_agent="MetaAgent"
)
```

**Step 4: Execution with Event Streaming**

```python
import asyncio

async def run_query(query: str):
    handler = workflow.run(user_msg=query)

    # Event streaming for monitoring
    async for event in handler.stream_events():
        if hasattr(event, 'agent_name'):
            print(f"[{event.agent_name}] Processing...")
        if hasattr(event, 'tool_name'):
            print(f"  Tool called: {event.tool_name}")

    response = await handler
    return response

# Execution
result = asyncio.run(run_query(
    "Compare the financial performance of 2023 with 2024 "
    "and identify how it aligns with the corporate strategy"
))
print(result)
```

**Step 5: Implementation with Sub-Question Decomposition**

```python
from llama_index.core.query_engine import SubQuestionQueryEngine

# Alternative approach with automatic decomposition
doc_tools = []
for doc_name, docs in documents_map.items():
    index = VectorStoreIndex.from_documents(docs)
    engine = index.as_query_engine(similarity_top_k=5)
    tool = QueryEngineTool.from_defaults(
        query_engine=engine,
        description=f"Information about {doc_name}"
    )
    doc_tools.append(tool)

# The SubQuestionQueryEngine decomposes automatically
sub_question_engine = SubQuestionQueryEngine.from_defaults(
    query_engine_tools=doc_tools,
    llm=OpenAI(model="gpt-4")
)

# The complex query is decomposed automatically
response = sub_question_engine.query(
    "How did revenues change between 2023 and 2024?"
)
```

### Architectural Considerations for Production

When implementing an Agentic RAG system in production, it is essential to consider:

**Latency management.** Multiple LLM calls increase latency. Use aggressive caching for intermediate results, parallelize calls to document agents when possible, and implement timeouts to avoid infinite loops in agent reasoning.

**Costs.** Multi-step reasoning involves multiple LLM calls. Implement a per-query budget (maximum number of steps), use cheaper models for routing and reserve powerful models for the final synthesis.

**Observability.** AgentWorkflow's event streaming is fundamental for debugging. Log every routing decision, every tool call and every reasoning step. Implement metrics on retrieval quality and response satisfaction.

**Index scalability.** For hundreds or thousands of documents, the meta-agent's tool retrieval becomes critical. Use metadata-based pre-filtering strategies and clustering of agents by thematic domain.

---

## 8. Resources and References

### Course and Educational Material

- **Building Agentic RAG with LlamaIndex** -- Course on DeepLearning.AI taught by Jerry Liu, 6 lessons with 4 code examples: [deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex](https://www.deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex/)

### Official LlamaIndex Documentation

- **Agents -- Use Cases**: [developers.llamaindex.ai/python/framework/use_cases/agents](https://developers.llamaindex.ai/python/framework/use_cases/agents/)
- **Multi-Agent Patterns**: [developers.llamaindex.ai/python/framework/understanding/agent/multi_agent](https://developers.llamaindex.ai/python/framework/understanding/agent/multi_agent/)
- **Sub Question Query Engine**: [developers.llamaindex.ai/python/examples/query_engine/sub_question_query_engine](https://developers.llamaindex.ai/python/examples/query_engine/sub_question_query_engine/)
- **Multi-Document Agents (V1)**: [docs.llamaindex.ai/en/stable/examples/agent/multi_document_agents-v1](https://docs.llamaindex.ai/en/stable/examples/agent/multi_document_agents-v1/)
- **ReAct Agent with Query Engine Tools**: [developers.llamaindex.ai/python/examples/agent/react_agent_with_query_engine](https://developers.llamaindex.ai/python/examples/agent/react_agent_with_query_engine/)

### LlamaIndex Technical Blog Posts

- **Agentic RAG with LlamaIndex**: [llamaindex.ai/blog/agentic-rag-with-llamaindex-2721b8a49ff6](https://www.llamaindex.ai/blog/agentic-rag-with-llamaindex-2721b8a49ff6)
- **RAG is Dead, Long Live Agentic Retrieval**: [llamaindex.ai/blog/rag-is-dead-long-live-agentic-retrieval](https://www.llamaindex.ai/blog/rag-is-dead-long-live-agentic-retrieval)
- **Introducing AgentWorkflow**: [llamaindex.ai/blog/introducing-agentworkflow-a-powerful-system-for-building-ai-agent-systems](https://www.llamaindex.ai/blog/introducing-agentworkflow-a-powerful-system-for-building-ai-agent-systems)
- **Announcing Workflows 1.0**: [llamaindex.ai/blog/announcing-workflows-1-0-a-lightweight-framework-for-agentic-systems](https://www.llamaindex.ai/blog/announcing-workflows-1-0-a-lightweight-framework-for-agentic-systems)
- **LlamaIndex is More Than a RAG Framework**: [llamaindex.ai/blog/llamaindex-is-more-than-a-rag-framework](https://www.llamaindex.ai/blog/llamaindex-is-more-than-a-rag-framework)
- **LlamaIndex Turns 1**: [llamaindex.ai/blog/llamaindex-turns-1-f69dcdd45fe3](https://www.llamaindex.ai/blog/llamaindex-turns-1-f69dcdd45fe3)

### Jerry Liu Profile

- **LinkedIn**: [linkedin.com/in/jerry-liu-64390071](https://www.linkedin.com/in/jerry-liu-64390071/)
- **Medium**: [medium.com/@jerryjliu98](https://medium.com/@jerryjliu98)
- **X/Twitter**: [@jerryjliu0](https://x.com/jerryjliu0)
- **Crunchbase**: [crunchbase.com/person/jerry-liu-acfb](https://www.crunchbase.com/person/jerry-liu-acfb)

### Third-Party Articles and Analyses

- **Agentic RAG vs Traditional RAG** (NVIDIA Technical Blog): [developer.nvidia.com/blog/traditional-rag-vs-agentic-rag](https://developer.nvidia.com/blog/traditional-rag-vs-agentic-rag-why-ai-agents-need-dynamic-knowledge-to-get-smarter/)
- **RAG, AI Agents, and Agentic RAG: Comparative Analysis** (DigitalOcean): [digitalocean.com/community/conceptual-articles/rag-ai-agents-agentic-rag-comparative-analysis](https://www.digitalocean.com/community/conceptual-articles/rag-ai-agents-agentic-rag-comparative-analysis)
- **What is Agentic RAG?** (IBM): [ibm.com/think/topics/agentic-rag](https://www.ibm.com/think/topics/agentic-rag)
- **Diving into LlamaIndex AgentWorkflow**: [dataleadsfuture.com/diving-into-llamaindex-agentworkflow](https://www.dataleadsfuture.com/diving-into-llamaindex-agentworkflow-a-nearly-perfect-multi-agent-orchestration-solution/)
- **Deep Dive into LlamaIndex Workflow: Event-Driven LLM Architecture** (Towards Data Science): [towardsdatascience.com/deep-dive-into-llamaindex-workflow](https://towardsdatascience.com/deep-dive-into-llamaindex-workflow-event-driven-llm-architecture-8011f41f851a/)
- **RAG Is A Hack -- with Jerry Liu** (Latent Space Podcast): [latent.space/p/llamaindex](https://www.latent.space/p/llamaindex)
