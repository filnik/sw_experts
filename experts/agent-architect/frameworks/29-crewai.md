# CrewAI -- Framework for Orchestration of Collaborative AI Agents

## 1. Overview

### Positioning in the AI Landscape

CrewAI is an open-source framework (MIT license) written in Python, designed to orchestrate autonomous AI agents that collaborate on complex tasks. Unlike other frameworks that rely on LangChain or external libraries, CrewAI was **built entirely from scratch** -- a deliberate architectural choice that ensures a lean codebase, high performance, and full control over every internal component.

The framework positions itself as a **production-ready** solution for multi-agent systems, offering two complementary paradigms: **Crews** for autonomous collaboration between agents with specialized roles, and **Flows** for event-driven orchestration with granular control over state and execution routing.

- **GitHub Repository**: https://github.com/crewAIInc/crewAI
- **GitHub Stars**: over 47,000
- **Forks**: over 6,400
- **Community**: over 100,000 certified developers
- **Requirements**: Python >= 3.10, < 3.14
- **Official documentation**: https://docs.crewai.com

### Design Philosophy

The philosophy behind CrewAI is based on three pillars:

1. **Role-based collaboration**: each agent has a role, a goal, and a narrative background that drive its behavior and decisions, simulating a team of specialized professionals.
2. **Autonomy with control**: agents operate autonomously in decision-making, but the framework offers guardrails, callbacks, and validation systems to maintain control over output quality.
3. **Simplicity towards production**: from rapid prototyping to enterprise deployment, CrewAI maintains the same API and the same concepts, reducing the gap between experiment and production system.

### Community and Ecosystem

The CrewAI community is among the most active in the landscape of AI agent frameworks. DeepLearning.AI has produced courses dedicated to the design and deployment of multi-agent systems with CrewAI. The framework natively integrates with over 30 built-in tools, supports embeddings from 10+ different providers (OpenAI, Ollama, Azure, Google, Cohere, Bedrock, Hugging Face), and offers an enterprise platform (CrewAI AMP) for production deployment and monitoring.

---

## 2. Core Concepts

### 2.1 Agent

The Agent is the fundamental unit of CrewAI: an autonomous entity equipped with personality, skills, and tools. Each agent is defined by three essential attributes that determine its behavior:

| Attribute | Type | Description |
|-----------|------|-------------|
| `role` | `str` | Defines the agent's function and expertise within the team |
| `goal` | `str` | The individual goal that drives decisions |
| `backstory` | `str` | Narrative context that enriches personality and interactions |
| `llm` | `str/LLM` | Language model that powers the agent (default: GPT-4) |
| `tools` | `List[BaseTool]` | Tools available for task execution |
| `max_iterations` | `int` | Maximum attempts before providing the best response (default: 20) |
| `allow_delegation` | `bool` | Allows delegation of tasks to other agents |
| `verbose` | `bool` | Enables detailed logging for debugging |
| `memory` | `Memory` | Reference to shared or scoped memory |
| `reasoning` | `bool` | Enables reflection and planning before execution |
| `multimodal` | `bool` | Support for text and visual content |
| `code_execution_mode` | `str` | Code execution in `safe` (Docker) or `unsafe` mode |
| `respect_context_window` | `bool` | Automatically summarizes content that exceeds the token limit |
| `response_format` | `Type[BaseModel]` | Pydantic model for structured and validated outputs |

**YAML Configuration (recommended method)**:

```yaml
# config/agents.yaml
researcher:
  role: "{topic} Senior Data Researcher"
  goal: "Discover the most advanced developments in {topic}"
  backstory: >
    Expert researcher with years of experience in identifying
    emerging trends and synthesizing complex information.
  verbose: true
  allow_delegation: true
```

**Programmatic configuration**:

```python
from crewai import Agent
from crewai_tools import SerperDevTool

researcher = Agent(
    role="Senior AI Researcher",
    goal="Identify the latest innovations in AI",
    backstory="Expert with 15 years of research in machine learning",
    tools=[SerperDevTool()],
    llm="gpt-4o",
    verbose=True,
    allow_delegation=True,
    reasoning=True
)
```

**Direct interaction**: starting from the recent version, it is possible to invoke an agent individually without a Crew, using the `kickoff()` method:

```python
result = researcher.kickoff("What are the latest developments in AI?")
print(result.raw)
```

### 2.2 Task

A Task represents a specific assignment that an Agent must complete. It contains all execution details: description, responsible agent, required tools, and completion criteria.

| Attribute | Type | Description |
|-----------|------|-------------|
| `description` | `str` | Clear description of the task requirements |
| `expected_output` | `str` | Detailed completion criteria |
| `agent` | `BaseAgent` | Agent responsible for execution |
| `tools` | `List[BaseTool]` | Specific tools for this task |
| `context` | `List[Task]` | Dependencies on outputs of other tasks |
| `output_pydantic` | `Type[BaseModel]` | Pydantic model for structured output |
| `output_json` | `Type[BaseModel]` | JSON structure validation |
| `output_file` | `str` | File path for saving results |
| `async_execution` | `bool` | Non-blocking execution (default: False) |
| `guardrail` | `Callable/str` | Output validation function or prompt |
| `guardrails` | `List[Callable]` | List of sequential validations |
| `guardrail_max_retries` | `int` | Maximum attempts to pass the guardrails |
| `callback` | `Callable` | Post-completion action |
| `human_input` | `bool` | Requires human review |
| `markdown` | `bool` | Formats output as markdown |

**YAML Configuration**:

```yaml
# config/tasks.yaml
research_task:
  description: "In-depth research on: {topic}"
  expected_output: "10-point report with verified sources"
  agent: researcher
  markdown: true
  output_file: report.md
```

**Guardrail System**: CrewAI supports two types of validation:

```python
# Function-based guardrail
def validate_length(result: TaskOutput) -> Tuple[bool, Any]:
    if len(result.raw.split()) < 200:
        return (False, "Output too short, expand the analysis")
    return (True, result.raw)

# LLM-based guardrail
task = Task(
    description="Write a technical report",
    expected_output="Professional and detailed report",
    agent=writer,
    guardrail="The output must be under 500 words and in a professional tone",
    guardrail_max_retries=3
)

# Multiple guardrails in sequence
task = Task(
    ...,
    guardrails=[validate_length, validate_sources, format_output]
)
```

**Structured output with TaskOutput**: each task produces a `TaskOutput` object with access to `raw` (text), `pydantic` (structured model), `json_dict` (dictionary), `summary`, and metadata about the executing agent.

### 2.3 Crew

The Crew is the container that brings together agents and tasks, defining how they collaborate and in what order they operate.

| Attribute | Type | Description |
|-----------|------|-------------|
| `agents` | `List[Agent]` | List of agents in the team |
| `tasks` | `List[Task]` | List of assigned tasks |
| `process` | `Process` | Type of process: `sequential` or `hierarchical` |
| `memory` | `bool/Memory` | Enables the memory system |
| `manager_llm` | `str/LLM` | LLM for the manager (required in hierarchical) |
| `manager_agent` | `Agent` | Custom manager agent |
| `max_rpm` | `int` | Rate limiting for requests per minute |
| `verbose` | `bool` | Logging level |
| `planning` | `bool` | Enables pre-execution planning |
| `knowledge_sources` | `List` | Knowledge sources shared by the team |
| `embedder` | `dict` | Embedder configuration for memory |
| `output_log_file` | `str` | File for output log (.txt or .json) |

```python
from crewai import Crew, Process

crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research_task, writing_task, review_task],
    process=Process.sequential,
    memory=True,
    verbose=True,
    planning=True
)

result = crew.kickoff(inputs={"topic": "Generative Artificial Intelligence"})
print(result.raw)
print(result.token_usage)
```

**Kickoff methods**: `kickoff()` and `kickoff_for_each()` for synchronous execution; `akickoff()` and `akickoff_for_each()` for native async (preferred for high concurrency); `kickoff_async()` and `kickoff_for_each_async()` for thread-based async.

**Planning**: when enabled, the Crew creates an execution plan before starting. The crew's data is sent to an AgentPlanner that distributes detailed plans to individual tasks, improving overall execution coherence.

---

## 3. Process Types

### 3.1 Sequential Process

The sequential process executes tasks **one after the other**, in linear order. The output of each task becomes available as context for subsequent ones. It is the default and the simplest to understand and debug.

```python
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential
)
```

**When to use it**: linear pipelines where each phase depends on the previous one -- for example research, then drafting, then review. It offers predictability and ease of debugging.

### 3.2 Hierarchical Process

The hierarchical process introduces a **manager agent** that coordinates the work, delegating tasks to the most appropriate agents and validating results before proceeding. It requires specifying `manager_llm` or a custom `manager_agent`.

```python
crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[research_task, analysis_task, writing_task],
    process=Process.hierarchical,
    manager_llm="gpt-4o"
)
```

**Manager behavior**: the manager analyzes available tasks, evaluates the agents' competencies, dynamically assigns work, reviews outputs, and may request rework. It simulates the structure of a team with a supervisor overseeing the flow.

**When to use it**: complex tasks with non-linear interdependencies, situations that require dynamic decision-making about work distribution, or scenarios where the quality of intermediate outputs must be validated before proceeding.

---

## 4. Flows -- Event-Driven Orchestration

Flows represent CrewAI's advanced orchestration paradigm, designed for **structured, event-driven, and production-ready** workflows. While Crews handle autonomous collaboration between agents, Flows provide precise control over execution flow, state, and conditional routing.

### Main Decorators

| Decorator | Function |
|------------|----------|
| `@start()` | Flow entry point. Supports unconditional starts and conditional gates |
| `@listen()` | Executes the method when a specified method completes. Receives the upstream output as argument |
| `@router()` | Enables conditional logic by returning route labels that trigger the corresponding listeners |
| `@persist` | Maintains flow state across restarts (SQLiteFlowPersistence by default) |
| `@human_feedback` | Pauses execution to collect human input, with optional LLM interpretation |

### State Management

CrewAI Flows supports two approaches to state management:

**Unstructured state** -- dictionary-based flexibility without a predefined schema:

```python
from crewai.flow.flow import Flow, listen, start

class ResearchFlow(Flow):
    @start()
    def collect_data(self):
        self.state["raw_data"] = "collected data..."
        return self.state["raw_data"]

    @listen(collect_data)
    def analyze(self, data):
        self.state["analysis"] = f"Analysis of: {data}"
        return self.state["analysis"]
```

**Structured state** -- based on Pydantic BaseModel for type-safety, validation, and IDE autocompletion:

```python
from pydantic import BaseModel

class ResearchState(BaseModel):
    topic: str = ""
    raw_data: str = ""
    analysis: str = ""
    score: float = 0.0

class ResearchFlow(Flow[ResearchState]):
    @start()
    def collect_data(self):
        self.state.raw_data = "collected data..."
        return self.state.raw_data
```

### Conditional Routing

The `@router()` decorator allows dynamic branching based on execution results:

```python
from crewai.flow.flow import Flow, start, listen, router

class QualityFlow(Flow):
    @start()
    def evaluate(self):
        score = compute_quality_score()
        return score

    @router(evaluate)
    def route_by_quality(self, score):
        if score > 0.8:
            return "publish"
        elif score > 0.5:
            return "revise"
        else:
            return "rewrite"

    @listen("publish")
    def publish_content(self):
        # Publish directly
        pass

    @listen("revise")
    def revise_content(self):
        # Send for review
        pass

    @listen("rewrite")
    def rewrite_content(self):
        # Complete rewrite
        pass
```

### Logical Operators

- **`or_()`**: the listener triggers when **any one** of the specified methods emits an output. Useful for parallel branches that converge.
- **`and_()`**: the listener triggers only when **all** of the specified methods have completed. Essential for synchronization points.

```python
from crewai.flow.flow import and_, or_

class ParallelFlow(Flow):
    @listen(or_(branch_a, branch_b))
    def on_any_complete(self, result):
        # Runs as soon as one of the two branches completes
        pass

    @listen(and_(research, analysis))
    def on_all_complete(self):
        # Runs only when both have completed
        pass
```

### Persistence and Visualization

- **Persistence**: the `@persist` decorator saves state to SQLite, allowing **resumption** of long workflows after interruptions.
- **Visualization**: `flow.plot("workflow.html")` generates interactive HTML diagrams showing dependencies and execution flow.
- **Asynchronous execution**: `await flow.kickoff_async()` for non-blocking workflows.
- **Streaming**: `stream=True` for real-time visibility into outputs.

### Integrating Crews into Flows

Real power emerges by combining Flows and Crews -- Flows orchestrate, Crews execute autonomously:

```python
class ContentPipeline(Flow):
    @start()
    def research_phase(self):
        research_crew = Crew(
            agents=[researcher],
            tasks=[research_task],
            process=Process.sequential
        )
        return research_crew.kickoff(inputs={"topic": self.state.topic})

    @listen(research_phase)
    def writing_phase(self, research_output):
        writing_crew = Crew(
            agents=[writer, editor],
            tasks=[writing_task, editing_task],
            process=Process.sequential
        )
        return writing_crew.kickoff(inputs={"research": research_output.raw})
```

---

## 5. Memory System

CrewAI implements a **unified memory system** that replaces the previous separation into short-term, long-term, entity, and external memory with a single intelligent `Memory` class.

### Memory Architecture

The system is based on three mechanisms:

1. **LLM analysis at save time**: when a memory is stored, an LLM infers scope, categories, and importance.
2. **Adaptive recall**: retrieval uses a composite scoring that balances semantic similarity, recency, and importance.
3. **Hierarchical organization by scope**: similar to a filesystem, memories are organized in a scope tree (`/project/alpha`, `/agent/researcher`).

### Composite Scoring Formula

```
score = semantic_weight * similarity + recency_weight * decay + importance_weight * importance
```

Where decay follows an exponential formula: `0.5^(age_days / half_life_days)`.

The weights are configurable to adapt to different scenarios: recency-oriented weights for fast sprints, importance-oriented weights for long-term architectural databases.

### Main APIs

```python
from crewai.memory import Memory

memory = Memory(
    llm="gpt-4o-mini",
    embedder={"provider": "openai", "config": {"model": "text-embedding-3-small"}},
    recency_weight=0.3,
    semantic_weight=0.5,
    importance_weight=0.2,
    recency_half_life_days=30,
    consolidation_threshold=0.85
)

# Store
memory.remember(
    "The API rate limit is 1000 requests per minute",
    scope="/project/config",
    source="documentation"
)

# Retrieve (shallow: ~200ms without LLM; deep: multi-step with LLM)
matches = memory.recall("What are the API limits?", limit=10, depth="deep")

# Extract atomic facts from raw text
facts = memory.extract_memories(raw_text)

# Explore the structure
tree = memory.tree()
info = memory.info(scope="/project")
records = memory.list_records(scope="/project/config")
```

### Four Usage Patterns

**Standalone** -- in scripts and notebooks without agents:
```python
memory = Memory()
memory.remember("Important fact to remember")
result = memory.recall("What do I remember?")
```

**With Crews** -- by passing `memory=True` or a configured instance; all agents share memory and the crew automatically extracts discrete facts from task outputs:
```python
crew = Crew(agents=[...], tasks=[...], memory=True)
```

**With Agents** -- agents receive scoped views for private context:
```python
researcher = Agent(..., memory=memory.scope("/agent/researcher"))
```

**With Flows** -- built-in methods `self.remember()`, `self.recall()`, `self.extract_memories()`:
```python
class MyFlow(Flow):
    @start()
    def step_one(self):
        self.remember("Important intermediate result")
        context = self.recall("What do we know so far?")
```

### Memory Slices

`MemorySlice` provides views over **multiple, disjoint scopes**, unlike single scopes. Typically used in read-only mode:

```python
agent_view = memory.slice(
    scopes=["/agent/researcher", "/company/knowledge"],
    read_only=True,
)
```

### Advanced Features

- **Non-blocking save**: `remember_many()` dispatches to a background thread; `recall()` automatically waits via read barrier.
- **Consolidation**: when saving, similar existing records trigger LLM decisions (keep, update, delete, insert).
- **Source tracking and privacy**: tags for origin; private memories visible only when the source matches.
- **Intra-batch dedup**: `remember_many()` discards near-duplicates within batches via vector similarity before storage.
- **Graceful degradation**: LLM analysis failures do not block -- memories are saved with default scope `/`, empty categories, and importance `0.5`.

### Storage

- **Default**: LanceDB stored in `./.crewai/memory`
- **Privacy**: content is sent to the configured LLM for analysis. For sensitive data, use local models (Ollama).
- **Embedder**: 10+ providers supported including OpenAI, Ollama, Azure, Google, Cohere, Bedrock, Hugging Face.

---

## 6. Knowledge -- Integrated RAG

CrewAI's Knowledge system functions as a **reference library** that agents can consult during task execution. It implements an integrated RAG (Retrieval-Augmented Generation) with provider-neutral abstraction.

### Supported Knowledge Sources

| Source | Class | Format |
|----------|--------|---------|
| Raw text | `StringKnowledgeSource` | In-memory strings |
| Text files | `TextFileKnowledgeSource` | .txt |
| PDF | `PDFKnowledgeSource` | .pdf |
| CSV | `CSVKnowledgeSource` | .csv |
| Excel | `ExcelKnowledgeSource` | .xlsx |
| JSON | `JSONKnowledgeSource` | .json |
| Web content | `CrewDoclingSource` | URL |

Files should be placed in a `knowledge` directory at the project root, referenced with relative paths.

### Chunking and Configuration

Sources are automatically processed into chunks. Configurable parameters include:

- **`chunk_size`**: target chunk size in characters
- **`chunk_overlap`**: overlap between chunks to preserve context
- **`results_limit`**: number of relevant documents returned (default: 3)
- **`score_threshold`**: minimum relevance score (default: 0.35)

### Embedding

By default, CrewAI uses **OpenAI embeddings (text-embedding-3-small)** for knowledge storage, even when other LLM providers are used. Alternatives include:

- **Voyage AI** (recommended for Claude users)
- **Ollama** (local providers)
- **Google Generative AI**
- **Azure OpenAI**

### Knowledge Levels

**Agent-level Knowledge**: specific to individual agents, saved in collections named by role, initialized during `crew.kickoff()`:

```python
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

knowledge_source = StringKnowledgeSource(
    content="Internal guidelines for technical documentation...",
    metadata={"domain": "tech-writing"}
)

writer = Agent(
    role="Technical Writer",
    goal="Produce high-quality documentation",
    backstory="Technical writing expert",
    knowledge_sources=[knowledge_source]
)
```

**Crew-level Knowledge**: shared among all agents, saved in the "crew" collection:

```python
crew = Crew(
    agents=[researcher, writer],
    tasks=[...],
    knowledge_sources=[pdf_source, csv_source]
)
```

### Advanced Features

- **Query Rewriting**: the system automatically transforms task prompts into optimized search queries using the agent's LLM.
- **Knowledge Events**: the system emits events such as `KnowledgeRetrievalStartedEvent` and `KnowledgeRetrievalCompletedEvent` for monitoring and debugging.
- **Custom Sources**: extend `BaseKnowledgeSource` to create sources from APIs or specialized data.
- **Reset**: `crew.reset_memories(command_type='knowledge')` to clear the knowledge base.

### Persistent Storage

The system uses ChromaDB (default) or Qdrant as vector store, with platform-specific storage paths:

- **macOS**: `~/Library/Application Support/CrewAI/{project}/knowledge/`
- **Linux**: `~/.local/share/CrewAI/{project}/knowledge/`
- **Windows**: `C:\Users\{username}\AppData\Local\CrewAI\{project}\knowledge\`

---

## 7. Tools

### Built-in Tools

CrewAI offers more than 30 pre-configured tools, installable with `pip install 'crewai[tools]'`:

**Data processing**: CSVSearchTool, JSONSearchTool, PDFSearchTool, XMLSearchTool, DOCXSearchTool, TXTSearchTool, MDXSearchTool.

**Web operations**: FirecrawlSearchTool, WebsiteSearchTool, ScrapeWebsiteTool, BrowserbaseLoadTool, ApifyActorsTool.

**Code and documentation**: CodeDocsSearchTool, GithubSearchTool, CodeInterpreterTool.

**Specialized**: EXASearchTool, SerperDevTool, YoutubeChannelSearchTool, YoutubeVideoSearchTool, PGSearchTool, DALL-E Tool, Vision Tool, ComposioTool, LlamaIndexTool, RagTool.

### Creating Custom Tools

**Approach 1 -- Subclassing BaseTool**:

```python
from crewai.tools import BaseTool
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(description="The search query")

class CustomSearchTool(BaseTool):
    name: str = "Database Search"
    description: str = "Search the company's internal database"
    args_schema: type[BaseModel] = SearchInput

    def _run(self, query: str) -> str:
        # Custom search logic
        results = internal_db.search(query)
        return str(results)
```

**Approach 2 -- @tool decorator**:

```python
from crewai.tools import tool

@tool("Sentiment Analyzer")
def sentiment_analyzer(text: str) -> str:
    """Analyze the sentiment of a text and return positive, negative, or neutral."""
    score = analyze(text)
    return f"Sentiment: {'positive' if score > 0 else 'negative'} (score: {score})"
```

### Asynchronous Tools

For I/O-intensive operations, asynchronous tools can be implemented:

```python
@tool("Async Fetcher")
async def async_fetcher(url: str) -> str:
    """Retrieve content from a URL asynchronously."""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
```

### Custom Caching

Result caching is configurable to control when results are reused:

```python
class MyTool(BaseTool):
    name: str = "Cached Tool"
    description: str = "Tool with selective caching"
    cache_function: Callable = lambda self, args, result: len(result) > 100
```

### LangChain Integration

CrewAI natively supports LangChain tools, allowing you to leverage the entire ecosystem of already available tools, including tools for databases, external APIs, file systems, and much more.

---

## 8. Training and Testing

### Training Crews

The `crewai train` command allows you to **train the crew** through multiple iterations, gathering human feedback to improve performance over time:

```bash
# Training with 10 iterations
crewai train -n 10

# Training with custom output file
crewai train -n 5 -f training_data.pkl
```

During training, the system:
1. Runs the crew for the specified number of iterations
2. Collects feedback on each iteration's output
3. Uses the feedback to refine agent behavior
4. Saves the results for future use

### Testing Crews

The `crewai test` command systematically evaluates crew performance:

```bash
# Test with 3 iterations using gpt-4o-mini
crewai test -n 3 -m gpt-4o-mini

# Test with specific model
crewai test -n 5 -m gpt-4o
```

Testing runs the crew multiple times and produces comparative metrics to evaluate consistency, output quality, and token usage.

### Replay

The `crewai replay` command allows you to **resume execution** from a specific task, maintaining the context of previous tasks:

```bash
# Resume from the specified task
crewai replay -t task_id_123
```

This is particularly useful for debugging: if a task fails halfway through the pipeline, you can fix it and restart from that point without re-executing everything from the beginning.

### Interactive Chat

Starting from version 0.98.0, the `crewai chat` command launches an **interactive session** with the crew, allowing you to test and iterate in real time:

```bash
crewai chat
```

It requires configuration of the `chat_llm` property in the `crew.py` file.

---

## 9. Production Deployment

### CrewAI AMP (Agent Management Platform)

CrewAI AMP is the enterprise platform for deploying and managing crews in production. It offers a **Crew Control Plane** with:

- **Tracing and Observability**: complete monitoring of agent executions, visibility into every step and decision.
- **Unified management**: centralized dashboard for all deployed crews.
- **Security**: RBAC (Role-Based Access Control), team and organization management.
- **Analytics**: performance metrics, token usage, and costs.
- **24/7 Support**: dedicated enterprise assistance.

### CLI Commands for Deployment

```bash
# Authentication
crewai login

# Organization management
crewai org list
crewai org switch <org_name>

# Deploy
crewai deploy create    # Create the deployment
crewai deploy push      # Push the code
crewai deploy status    # Check the status
crewai deploy logs      # View the logs
crewai deploy list      # List active deployments
crewai deploy remove    # Remove a deployment

# Tracing
crewai traces enable
crewai traces status
```

### Enterprise Integrations

The enterprise platform supports native integrations with:

- **Communication**: Gmail, Slack, Microsoft Teams, Outlook
- **CRM and Sales**: Salesforce, HubSpot
- **Storage**: OneDrive, Google Drive
- **Cloud**: Amazon Bedrock Agents
- **Automatic triggers**: webhooks, scheduling, external events

### Production Requirements

For production-grade environments, recommendations include:

| Aspect | Recommendation |
|---------|-----------------|
| **OS** | Ubuntu 22.04 LTS |
| **Python** | 3.10-3.13 with `uv` package manager |
| **RAM** | 2-4 GB per agent instance; 16-32 GB for 5-10 concurrent agents |
| **Network** | Sub-50ms latency, minimum 100 Mbps throughput |
| **Storage** | PostgreSQL for structured data, MongoDB for unstructured; 100 GB+ |
| **Persistent memory** | Redis or PostgreSQL for state across executions |
| **Monitoring** | Prometheus/Grafana for metrics |
| **Orchestration** | Docker/Kubernetes for containerization |

---

## 10. Best Practices and Patterns

### Agent Design

1. **Specific and unambiguous roles**: each agent must have a clearly defined role. Avoid "do-it-all" agents in favor of focused specialists.
2. **Rich and contextual backstories**: the backstory is not decorative -- it actively guides the agent's behavior. Include experience, skills, and approach to work.
3. **Appropriate LLMs for tasks**: use powerful models (GPT-4o, Claude) for complex tasks and economical models (GPT-4o-mini) for tool calling and simple tasks.
4. **Limit iterations**: set reasonable `max_iterations` to avoid infinite loops and excessive costs.

### Task Design

5. **Precise descriptions**: describe exactly what must be done, in what format, and with what constraints.
6. **Explicit expected outputs**: clearly specify the format, length, and expected content.
7. **Use context for dependencies**: connect tasks via `context` to share output explicitly.
8. **Layered guardrails**: combine functional validation (programmatic checks) with LLM validation (semantic checks).

### Crew Architecture

9. **Start with Sequential**: the sequential process is easier to debug and understand. Move to hierarchical only when necessary.
10. **Enable memory for continuity**: use `memory=True` for tasks that benefit from accumulated context across executions.
11. **Planning for complex tasks**: enable `planning=True` for crews with many interconnected tasks.
12. **Rate limiting**: set `max_rpm` to avoid exceeding API limits.

### Production Patterns

13. **Flows for orchestration**: use Flows for execution flow control and Crews for autonomous collaboration. Don't try to force complex orchestration logic into Crews.
14. **Structured state**: prefer Pydantic BaseModel for Flow state in production -- it offers type-safety, validation, and automatic documentation.
15. **Persistence for long workflows**: use `@persist` for workflows that may last hours or be interrupted.
16. **Monitoring with events**: leverage CrewAI's event system for logging, metrics, and alerting.
17. **Knowledge for domain expertise**: load domain-specific documentation into knowledge sources rather than relying solely on the LLM's base knowledge.
18. **Iterative testing**: use `crewai test` and `crewai train` regularly to measure and improve performance.
19. **Local models for sensitive data**: when data is confidential, configure Ollama or other local providers for embedding and memory analysis.
20. **Aggressive caching**: enable tool caching to reduce redundant API calls and improve response times.

### Anti-Patterns to Avoid

- **Too many agents**: managing more than 5 agents with interdependencies becomes complex. Prefer smaller crews orchestrated with Flows.
- **Overlapping roles**: agents with similar roles create confusion in delegation. Each role must be distinct.
- **Unstructured outputs**: in production, always use `output_pydantic` or `output_json` to ensure parsable and validatable outputs.
- **Ignoring costs**: monitor `token_usage` and optimize -- each additional iteration has a real cost.

---

## 11. Resources and References

### Official Documentation

- **Docs**: https://docs.crewai.com
- **GitHub**: https://github.com/crewAIInc/crewAI
- **Website**: https://crewai.com

### Training

- **DeepLearning.AI**: [Design, Develop, and Deploy Multi-Agent Systems with CrewAI](https://www.deeplearning.ai/courses/design-develop-and-deploy-multi-agent-systems-with-crewai/) -- complete course on the design and deployment of multi-agent systems.

### Articles and Tutorials

- [CrewAI Framework 2025: Complete Review](https://latenode.com/blog/ai-frameworks-technical-infrastructure/crewai-framework/crewai-framework-2025-complete-review-of-the-open-source-multi-agent-ai-platform) -- in-depth review of the platform.
- [CrewAI Flows: Event-Driven Agent Orchestration Tutorial](https://markaicode.com/crewai-flows-event-driven-agent-orchestration/) -- tutorial on Flows.
- [Building Multi-Agent Systems With CrewAI](https://www.firecrawl.dev/blog/crewai-multi-agent-systems-tutorial) -- complete tutorial on building multi-agent systems.
- [CrewAI Guide: Build Multi-Agent AI Teams](https://mem0.ai/blog/crewai-guide-multi-agent-ai-teams) -- practical guide.
- [CrewAI - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-frameworks/crewai.html) -- AWS guide for agentic frameworks.

### Specific Concepts in the Documentation

- [Agents](https://docs.crewai.com/en/concepts/agents) -- complete documentation on agents.
- [Tasks](https://docs.crewai.com/en/concepts/tasks) -- complete documentation on tasks.
- [Crews](https://docs.crewai.com/en/concepts/crews) -- complete documentation on crews.
- [Flows](https://docs.crewai.com/en/concepts/flows) -- complete documentation on flows.
- [Memory](https://docs.crewai.com/en/concepts/memory) -- complete documentation on the memory system.
- [Knowledge](https://docs.crewai.com/en/concepts/knowledge) -- complete documentation on the knowledge system.
- [Tools](https://docs.crewai.com/en/concepts/tools) -- complete documentation on tools.
- [CLI](https://docs.crewai.com/en/concepts/cli) -- complete reference for the CLI.

---

*Document generated on March 26, 2026. Information is based on CrewAI's official documentation and publicly available sources at the time of writing.*
