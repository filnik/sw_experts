# LlamaIndex AgentWorkflow

## Index

1. [Overview](#1-overview)
2. [AgentWorkflow - The New Agent Framework](#2-agentworkflow---the-new-agent-framework)
3. [Workflows - Event-Driven Architecture](#3-workflows---event-driven-architecture)
4. [Agent Types](#4-agent-types)
5. [RAG Integration](#5-rag-integration)
6. [Multi-Agent Systems](#6-multi-agent-systems)
7. [Memory and State Management](#7-memory-and-state-management)
8. [Observability](#8-observability)
9. [Comparison with Other Frameworks](#9-comparison-with-other-frameworks)
10. [Resources and References](#10-resources-and-references)

---

## 1. Overview

LlamaIndex is an open-source framework born in 2022 under the name GPT Index, created by Jerry Liu. Its original mission was to bridge the gap between Large Language Models and the private data of organizations, offering tools to index, structure, and query documents through an LLM interface. Over time, LlamaIndex has evolved from a simple RAG (Retrieval-Augmented Generation) framework to a complete platform for building agentic applications.

### From RAG Framework to Agentic Platform

LlamaIndex's evolution reflects a broader trend in the AI ecosystem: the shift from linear retrieval pipelines to autonomous systems capable of multi-step reasoning. In 2025, LlamaIndex completed this transformation by repositioning itself as a "leading document agent and OCR platform", with the entire documentation reorganized around the AgentWorkflow concept.

### Modular Architecture

LlamaIndex's architecture is based on a strongly modular design:

- **LlamaIndex Core**: the fundamental package with the base abstractions (nodes, indexes, query engines, agents)
- **300+ Integration Packages**: available via LlamaHub, covering LLM providers (OpenAI, Anthropic, Mistral, HuggingFace), embedding models, vector stores (Pinecone, Weaviate, Chroma, Qdrant), and data connectors
- **LlamaParse**: enterprise platform for advanced parsing of documents (PDF, tables, images)
- **LlamaDeploy**: module for deployment and scaling of multi-agent systems in production

The import system uses a namespace that clearly distinguishes core modules from integrations: imports containing `core` refer to the base functionalities, while the others indicate specific integration packages.

### Design Philosophy

LlamaIndex embraces a "batteries included but composable" philosophy: a beginner can build a working RAG system in five lines of code, while an advanced developer can customize every single component of the pipeline. This duality also extends to the agentic system: it is possible to use preconfigured agents or build entirely custom workflows.

---

## 2. AgentWorkflow - The New Agent Framework

AgentWorkflow represents the heart of LlamaIndex's modern agentic system. Built on top of the Workflow abstractions (described in the next section), AgentWorkflow provides a high-level layer that drastically simplifies the creation, orchestration, and management of single-agent and multi-agent systems.

### Fundamental Concept

An agent in LlamaIndex is defined as an "automated reasoning and decision engine": a semi-autonomous software powered by an LLM that receives a task, autonomously decides which tools to use, executes them, evaluates the results, and iterates until the task is completed. Unlike a linear RAG pipeline, an agent can branch, loop, and adapt its strategy based on intermediate results.

### Quick Initialization

For a single agent with simple tools, AgentWorkflow offers an extremely concise factory method:

```python
from llama_index.core.agent.workflow import AgentWorkflow

async def search_web(query: str) -> str:
    """Use the web to answer questions."""
    client = AsyncTavilyClient()
    return str(await client.search(query))

workflow = AgentWorkflow.from_tools_or_functions(
    [search_web],
    llm=llm,
    system_prompt="You are a helpful assistant that answers using the web."
)

response = await workflow.run(user_msg="What is the capital of France?")
```

The `from_tools_or_functions` method accepts both `FunctionTool` objects and simple Python functions: the framework automatically analyzes the name, docstring, and type hints of the function to build the tool description.

### Internal Orchestration Flow

The AgentWorkflow execution cycle follows a well-defined sequence of internal steps:

1. **init_run**: initializes the Context and ChatMemory for the session
2. **setup_agent**: identifies the active agent, merges the system prompt with the chat history
3. **run_agent_step**: invokes the agent's `take_step` method, which receives the message history and available tools, then determines the next actions via `astream_chat_with_tools`
4. **parse_agent_output**: interprets the agent's output and identifies the tools to be called
5. **call_tool**: executes the requested tools concurrently
6. **aggregate_tool_results**: consolidates the results, handles any handoffs to other agents, and decides whether to return to step 3 or terminate

This cycle is completely transparent thanks to the event system, which allows monitoring each phase in real time.

### Event Streaming

During execution, AgentWorkflow emits five types of observable events:

| Event | Description |
|--------|-------------|
| `AgentInput` | Chat history and identity of the active agent |
| `AgentStream` | Intermediate streaming outputs from the LLM |
| `AgentOutput` | Complete result of a synchronous round |
| `ToolCall` | Tool invocation parameters |
| `ToolCallResult` | Result of tool execution |

These events enable both production monitoring and interactive patterns such as real-time streaming to the user.

### Human-in-the-Loop

AgentWorkflow natively supports human intervention in the agentic loop via two special events:

- **InputRequiredEvent**: emitted by the agent when it needs human approval or input
- **HumanResponseEvent**: the user's response that is reintegrated into the flow

This pattern is fundamental for critical workflows where certain decisions require human validation before proceeding.

---

## 3. Workflows - Event-Driven Architecture

The Workflows system constitutes the architectural foundation on which AgentWorkflow is built. Released as Workflows 1.0 in 2025 with the dedicated `llama-index-workflows` package, it represents a general-purpose orchestration framework that can operate independently of the rest of the LlamaIndex ecosystem.

### Fundamental Principles

A Workflow is defined as "an event-driven, step-based way to control the execution flow of an application". Unlike DAGs (Directed Acyclic Graphs) used by other frameworks, LlamaIndex Workflows natively support loops, conditional branches, and arbitrary control flows, all managed through a system of typed events.

### Step Functions

Steps are methods decorated with `@step` that receive events as input and emit new ones as output. The decorator automatically infers input/output types from type annotations:

```python
from llama_index.core.workflow import Workflow, step, StartEvent, StopEvent, Event

class JokeEvent(Event):
    joke: str

class JokeFlow(Workflow):
    llm = OpenAI(model="gpt-4o")

    @step
    async def generate_joke(self, ev: StartEvent) -> JokeEvent:
        topic = ev.topic
        response = await self.llm.acomplete(f"Write a joke about {topic}")
        return JokeEvent(joke=str(response))

    @step
    async def critique_joke(self, ev: JokeEvent) -> StopEvent:
        response = await self.llm.acomplete(
            f"Evaluate this joke: {ev.joke}"
        )
        return StopEvent(result=str(response))

# Execution
w = JokeFlow(timeout=60, verbose=True)
result = await w.run(topic="programmers")
```

### Event System

Events are Pydantic-based objects that carry data between steps. The framework provides three categories:

- **StartEvent**: workflow entry point, accepts arbitrary attributes passed to `run()`
- **StopEvent**: exit point that stops the workflow and returns the result
- **Custom Events**: user-defined classes that inherit from `Event` with typed fields

The power of the system lies in type hinting: the `@step` decorator automatically determines which step handles which event based on annotations. This allows validating the correctness of the workflow before execution.

### Loops and Branches

Workflows natively support complex flows via the union operator `|`:

```python
class LoopEvent(Event):
    loop_output: str

class ProcessingEvent(Event):
    intermediate_result: str

class ComplexWorkflow(Workflow):
    @step
    async def step_one(self, ev: StartEvent | LoopEvent) -> ProcessingEvent | LoopEvent:
        if need_retry():
            return LoopEvent(loop_output="Retrying the step.")
        return ProcessingEvent(intermediate_result="Step completed.")

    @step
    async def step_two(self, ev: ProcessingEvent) -> StopEvent:
        return StopEvent(result=f"Completed: {ev.intermediate_result}")
```

This pattern makes it possible to implement retry logic, refinement cycles, and conditional branches without resorting to explicit graphs.

### Context Management

The Context provides a shared state accessible to all steps of the workflow:

```python
from llama_index.core.workflow import Context

@step
async def query_step(self, ctx: Context, ev: StartEvent) -> StopEvent:
    # Write to the shared state
    await ctx.store.set("query", "What is the capital of France?")

    # Read from the shared state
    query = await ctx.store.get("query")

    return StopEvent(result=query)
```

With Workflows 1.0, the Context also supports strong state typing in both Python and TypeScript, and introduces the dynamic resource injection mechanism to inject dependencies (database clients, external services) at runtime.

### Workflow Visualization

LlamaIndex offers utilities to visualize workflows as diagrams:

```python
from llama_index.utils.workflow import draw_all_possible_flows

draw_all_possible_flows(my_workflow, "flow.html")
```

This generates an interactive HTML file showing all possible flows between steps, useful for debugging and system documentation.

### Async-First Design

Workflows adopt an async-first design based on Python asyncio. The `run()` method is asynchronous, allowing concurrent calls to LLMs and external tools without blocking execution. This architectural choice is particularly advantageous in scenarios with many parallel API calls.

---

## 4. Agent Types

LlamaIndex offers different types of agents, each optimized for specific scenarios. All inherit from the `BaseWorkflowAgent` base class and can be used either individually or within a multi-agent AgentWorkflow.

### FunctionAgent

The FunctionAgent is the primary agent type, designed for LLMs that natively support function/tool calling (OpenAI, Anthropic, Mistral, and others). It is the recommended option when working with providers that expose function calling APIs.

```python
from llama_index.core.agent.workflow import FunctionAgent
from llama_index.llms.openai import OpenAI

def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"The weather in {city} is sunny, 22 degrees."

agent = FunctionAgent(
    name="weather_agent",
    description="Specialized in providing weather information",
    system_prompt="You are a precise and concise weather assistant.",
    tools=[get_weather],
    llm=OpenAI(model="gpt-4o-mini"),
)

response = await agent.run(user_msg="What's the weather in Rome?")
```

The FunctionAgent implements three key methods in its internal architecture:

- **take_step**: receives the chat history and available tools, determines the next actions via `astream_chat_with_tools`, and produces execution streams
- **handle_tool_call_results**: stores the results of tool execution in the Context (tools are executed separately from the AgentWorkflow core)
- **finalize**: extracts the tool call stacks and preserves them as chat history in the ChatMemory

### ReActAgent

The ReActAgent uses the ReAct (Reasoning + Acting) prompting strategy, which explicitly alternates phases of reasoning ("Thought") and action ("Action"). This approach is compatible with any LLM, even those that do not natively support function calling.

```python
from llama_index.core.agent.workflow import ReActAgent, AgentWorkflow

def add(a: int, b: int) -> int:
    """Sum two integers."""
    return a + b

def multiply(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b

agent = ReActAgent(
    name="math_agent",
    description="Agent specialized in mathematical operations",
    system_prompt="You are a math assistant. Use the available tools.",
    tools=[add, multiply],
    llm=llm,
)

response = await agent.run(user_msg="What is 15 times 7?")
```

The advantage of the ReActAgent lies in the transparency of reasoning: each step produces an explicit "Thought" that explains the reason for the chosen action, making the process debuggable and interpretable. The disadvantage is greater token consumption compared to the FunctionAgent.

### CodeActAgent

The CodeActAgent represents an innovative approach where the agent generates and executes Python code to solve tasks. It requires a `code_execute_fn` function that handles the execution of the generated code:

```python
from llama_index.core.agent.workflow import CodeActAgent

def code_execute_fn(code: str) -> str:
    """Execute Python code and return the result."""
    # Sandboxed execution implementation
    local_vars = {}
    exec(code, {}, local_vars)
    return str(local_vars.get("result", "Execution completed"))

agent = CodeActAgent(
    name="code_agent",
    description="Agent that solves problems by writing Python code",
    tools=[code_execute_fn],
    llm=llm,
)
```

This type of agent is particularly effective for data analysis tasks, complex calculations, and data structure manipulation, where the flexibility of code surpasses that of predefined tools.

### StructuredPlanningAgent

The StructuredPlanningAgent, introduced in version 0.10.34 of LlamaIndex, implements a structured planning pattern. It functions as a wrapper around any other type of agent (ReAct, Function Calling, Chain-of-Abstraction), automatically decomposing a complex input into manageable sub-tasks.

The operational flow is:

1. Receives the main task from the user
2. Generates a structured plan with ordered sub-tasks
3. Executes each sub-task using the underlying agent
4. Aggregates the results into a coherent response

This approach is particularly useful for complex queries that require multiple steps of search, analysis, and synthesis. The planning agent can dynamically re-plan if a sub-task fails or produces unexpected results.

### Choosing the Agent Type

| Criterion | FunctionAgent | ReActAgent | CodeActAgent |
|----------|---------------|------------|--------------|
| LLM required | With function calling | Any | With code generation |
| Transparency | Medium | High | High |
| Token efficiency | High | Medium | Variable |
| Flexibility | Medium | Medium | High |
| Ideal use case | Standard API/tools | Generic LLMs | Data analysis |

---

## 5. RAG Integration

RAG integration represents one of the distinctive strengths of LlamaIndex compared to other agentic frameworks. The ability to transform query engines and indexes into tools usable by agents creates a natural bridge between retrieval and reasoning.

### QueryEngineTool

The fundamental pattern consists of wrapping a query engine as an agentic tool via `QueryEngineTool`:

```python
from llama_index.core import SimpleDirectoryReader, VectorStoreIndex
from llama_index.core.tools import QueryEngineTool

# Index construction
documents = SimpleDirectoryReader(input_files=["report_Q4.pdf"]).load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine(similarity_top_k=3)

# Wrapping as a tool
report_tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    name="report_q4",
    description="Useful for answering questions about Q4 financial results. "
                "Contains data on revenue, profits, margins, and projections."
)
```

The tool description is fundamental: the agent uses it to decide when and how to invoke the query engine. Precise and detailed descriptions significantly improve the quality of the agent's decisions.

### Document Agents

An advanced pattern involves the creation of agents specialized for individual documents or collections:

```python
from llama_index.core.agent.workflow import FunctionAgent, AgentWorkflow

# Tool creation for different documents
uber_tool = QueryEngineTool.from_defaults(
    query_engine=uber_index.as_query_engine(),
    name="uber_10k",
    description="Uber financial data from the 10-K annual report"
)

lyft_tool = QueryEngineTool.from_defaults(
    query_engine=lyft_index.as_query_engine(),
    name="lyft_10k",
    description="Lyft financial data from the 10-K annual report"
)

# Agent with access to multiple document sources
agent = FunctionAgent(
    tools=[uber_tool, lyft_tool],
    llm=OpenAI(model="gpt-4o"),
    system_prompt="You are a financial analyst. Compare data across different companies."
)
```

### Router Query Engine

For scenarios where a single document requires different query strategies, LlamaIndex offers the Router Query Engine which automatically selects the optimal approach:

- **Summary Index**: for summary-type questions about the entire document
- **Vector Index**: for retrieval of specific contexts
- **Keyword Index**: for keyword-based searches

The agent can use all of these as distinct tools, choosing the most appropriate retrieval strategy for each sub-query.

### Agentic RAG

Agentic RAG goes beyond the traditional "retrieve-then-generate" paradigm by introducing a complete agentic cycle:

1. The agent analyzes the user's query
2. Decides whether it needs retrieval or can respond directly
3. If necessary, selects the appropriate query engine(s)
4. Evaluates retrieval results
5. Decides whether further searches are needed or it can synthesize
6. Generates the final response combining the results

This approach is particularly powerful for complex questions that require information from multiple sources or multi-hop reasoning.

---

## 6. Multi-Agent Systems

LlamaIndex offers three main patterns for the implementation of multi-agent systems, each with different levels of abstraction and control.

### Pattern 1: AgentWorkflow with Handoff

The most direct pattern is AgentWorkflow with specialized agents and declarative handoffs:

```python
from llama_index.core.agent.workflow import AgentWorkflow, FunctionAgent

# Researcher agent
research_agent = FunctionAgent(
    name="researcher",
    description="Specialized in research and information gathering",
    system_prompt="You are an expert researcher. Gather detailed information.",
    tools=[search_web, record_notes],
    llm=llm,
    can_handoff_to=["writer"],
)

# Writer agent
writer_agent = FunctionAgent(
    name="writer",
    description="Specialized in writing structured reports",
    system_prompt="You are an expert writer. Write clear and complete reports.",
    tools=[write_report],
    llm=llm,
    can_handoff_to=["reviewer"],
)

# Reviewer agent
reviewer_agent = FunctionAgent(
    name="reviewer",
    description="Specialized in review and quality assurance",
    system_prompt="You are a critical reviewer. Evaluate the quality and completeness.",
    tools=[review_report],
    llm=llm,
    can_handoff_to=["researcher", "writer"],
)

# Multi-agent workflow creation
workflow = AgentWorkflow(
    agents=[research_agent, writer_agent, reviewer_agent],
    root_agent="researcher",
)

response = await workflow.run(
    user_msg="Write a report on the impact of AI in the healthcare sector"
)
```

The handoff mechanism works through an integrated tool called `handoff`. When an agent determines that another agent is more suitable for the next step, it invokes the handoff tool specifying the target agent. The framework uses the `DEFAULT_HANDOFF_OUTPUT_PROMPT` to brief the successor agent, ensuring a smooth continuation without restarting from the beginning.

The `can_handoff_to` properties define a directed collaboration graph: the researcher can pass to the writer, the writer to the reviewer, and the reviewer can return to both the researcher and the writer for improvement iterations.

### Pattern 2: Orchestrator Agent

In this pattern, specialized agents do not communicate directly with each other. Their `run` methods become tools available to a higher-level orchestrator agent:

```python
# The specialized agents become tools of the orchestrator
research_tool = AgentTool(agent=research_agent)
writing_tool = AgentTool(agent=writer_agent)

orchestrator = FunctionAgent(
    name="orchestrator",
    description="Coordinates work among specialized agents",
    tools=[research_tool, writing_tool],
    llm=llm,
)
```

Advantages of the orchestrator approach:
- Single point of control for decisions
- Better management of complex coordination logic
- Maintains streaming and state management
- More suitable when custom routing logic is needed

### Pattern 3: Custom Planner

The DIY approach offers maximum control. You create an LLM prompt that generates structured plans (XML/JSON), then parse and execute them imperatively:

```python
plan_prompt = """Given the following request, generate a structured plan in JSON:
{user_request}

Format:
[{"step": 1, "agent": "researcher", "task": "..."},
 {"step": 2, "agent": "writer", "task": "..."}]
"""
```

This pattern is recommended only when the first two cannot express the complexity of the required workflow.

### Recommended Progression

The official documentation suggests a natural progression: start with AgentWorkflow for prototyping, move to the Orchestrator when sequential control is needed, and adopt the Custom Planner only when previous patterns cannot model the requirements.

---

## 7. Memory and State Management

Memory and state management is a critical aspect for agents that must maintain coherence across multiple interactions and handoffs between different agents.

### Context and State

The AgentWorkflow Context provides a global state accessible to all agents and tools of the workflow:

```python
from llama_index.core.workflow import Context

async def record_notes(ctx: Context, notes: str) -> str:
    """Record research notes in the shared state."""
    current_state = await ctx.store.get("state")
    current_state["research_notes"].append(notes)
    await ctx.store.set("state", current_state)
    return f"Notes recorded: {notes}"

# Initialization with initial state
workflow = AgentWorkflow(
    agents=[research_agent, writer_agent],
    root_agent="researcher",
    initial_state={
        "research_notes": [],
        "num_fn_calls": 0,
        "draft_version": 0,
    },
    state_prompt="Current state: {state}. User message: {msg}",
)

# Execution with persistent context
ctx = Context(workflow)
response = await workflow.run(user_msg="Research on AI in medicine", ctx=ctx)

# Inspecting the state after execution
state = await ctx.store.get("state")
print(f"Notes collected: {state['research_notes']}")
print(f"Function calls: {state['num_fn_calls']}")
```

The `state_prompt` parameter is particularly important: it defines how the current state is injected into the agent's prompt, allowing it to make informed decisions based on the history of operations.

### ChatMemory

ChatMemory manages the persistence of conversation between agent and user across multiple sessions:

- Preserves the tool call stacks as part of the chat history
- Maintains the conversational context across handoffs between agents
- Supports recall of information from previous interactions

The `finalize` method of the FunctionAgent is responsible for extracting the tool call stacks and preserving them in the ChatMemory, ensuring that a successor agent has visibility into the actions performed by its predecessor.

### State Injection in Tools

A powerful feature is the ability of tools to access and modify the workflow state via the `ctx: Context` parameter:

```python
async def update_counter(ctx: Context, action: str) -> str:
    """Update the action counter."""
    cur_state = await ctx.store.get("state")
    cur_state["num_fn_calls"] += 1
    cur_state["last_action"] = action
    await ctx.store.set("state", cur_state)
    return f"Action '{action}' recorded. Total actions: {cur_state['num_fn_calls']}"
```

The framework automatically injects the Context as a parameter if the function signature includes it, without the need for additional configuration.

### Typed State (Workflows 1.0)

With Workflows 1.0, state supports strong typing in both Python and TypeScript, eliminating runtime errors due to missing keys or incorrect types:

```python
from dataclasses import dataclass

@dataclass
class WorkflowState:
    research_notes: list[str]
    num_fn_calls: int
    draft_version: int
    is_complete: bool = False
```

### Resource Injection

Workflows 1.0 introduces dynamic resource injection, allowing dependencies (database clients, external services, configurations) to be injected during the runtime of the workflow. This separates workflow logic from infrastructure dependencies, facilitating testing and deployment in different environments.

---

## 8. Observability

Observability is a fundamental pillar for agentic systems in production. LlamaIndex offers a rich ecosystem of integrations for monitoring, tracing, and debugging.

### Observability Architecture

Starting from version 0.10.20, LlamaIndex manages observability via the `instrumentation` module. Legacy implementations used the `CallbackManager` or `set_global_handler`, approaches still functional but deprecated in favor of the new system.

The event-driven architecture of Workflows naturally maps to the OpenTelemetry model of spans and traces: each step of the workflow emits and receives events that correspond directly to spans in the tracing system.

### Setup with OpenTelemetry

The OpenTelemetry integration traces all events produced by LlamaIndex components, including LLMs, agents, RAG pipelines, and tools:

```python
from llama_index.observability.otel import LlamaIndexOpenTelemetry
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

# Setup in 6 lines of code
span_exporter = OTLPSpanExporter("http://localhost:4318/v1/traces")
instrumentor = LlamaIndexOpenTelemetry(
    service_name_or_resource="my_agent_service",
    span_exporter=span_exporter,
)
```

### Integrated Observability Platforms

LlamaIndex natively supports numerous platforms:

**LlamaTrace / Arize Phoenix (Hosted)**:
```python
import os
os.environ["OTEL_EXPORTER_OTLP_HEADERS"] = f"api_key={PHOENIX_API_KEY}"
set_global_handler("arize_phoenix", endpoint="https://llamatrace.com/v1/traces")
```

**MLflow**:
```python
import mlflow
mlflow.llama_index.autolog()
```

**Langfuse** (with OpenInference):
```python
from openinference.instrumentation.llama_index import LlamaIndexInstrumentor
LlamaIndexInstrumentor().instrument()
```

**Other supported platforms**: SigNoz, Weights & Biases Weave, Literal AI, Comet Opik, OpenLLMetry, Agenta, DeepEval, Maxim AI.

### Captured Metrics

Observability integrations typically capture:

- **Execution time**: latency of each step, LLM call, and tool
- **Token usage**: input/output tokens for each LLM call
- **Input/Output**: data entering and exiting each component
- **Errors**: exceptions, timeouts, tool failures
- **Nested tracking**: nested operations with span hierarchy
- **Streaming data**: real-time flow monitoring
- **Cost analysis**: cost estimation based on token usage

### Custom Events for Workflows

For specific workflows, it is possible to define custom events that add domain metrics:

```python
from llama_index.core.instrumentation import dispatcher

# Custom event for document classification
dispatcher.event(event=ClassificationMetadata(
    duration=elapsed_time,
    metadata={
        "confidence": 0.95,
        "document_type": "financial_report",
        "pages_processed": 42,
    }
))
```

### Visualization with Jaeger

Jaeger UI visualizes the collected traces with detailed span hierarchies, temporal information, and metadata of custom events, allowing effective debugging of complex multi-step pipelines.

---

## 9. Comparison with Other Frameworks

### LlamaIndex AgentWorkflow vs LangGraph

**LangGraph** adopts a "graph-first" approach: you define an explicit state machine with nodes, edges, and conditional routing. This produces traceable and debuggable flows, ideal for complex business logic with sophisticated branching.

**LlamaIndex AgentWorkflow** uses an implicit event-driven approach: you declare agents with their capabilities and the framework manages orchestration. This reduces boilerplate but offers less granular control over transition flows.

| Aspect | LlamaIndex AgentWorkflow | LangGraph |
|---------|--------------------------|-----------|
| Paradigm | Event-driven, declarative | Graph-based, state machine |
| Learning curve | Low-medium | Medium-high |
| RAG integration | Native, excellent | Possible but manual |
| Flow control | Implicit via handoff | Explicit via graphs |
| Multi-agent | Built-in with handoff | Requires subgraph pattern |
| Debugging | Event streaming | Graph visualization |
| State persistence | Context + ChatMemory | Integrated checkpointer |

**When to choose LlamaIndex**: if the project is centered on data/documents, requires advanced RAG, or you prefer a declarative approach.
**When to choose LangGraph**: if you need fine control over flows, complex business logic branching, or structured approval cycles.

### LlamaIndex AgentWorkflow vs CrewAI

**CrewAI** adopts a "crew-of-peers" model with agents that have defined roles, goals, and operate as a structured team. The approach is role-based and oriented toward collaboration.

**LlamaIndex AgentWorkflow** typically uses a manager-worker pattern with a root agent that delegates to sub-agents, or agents that explicitly hand off control.

| Aspect | LlamaIndex AgentWorkflow | CrewAI |
|---------|--------------------------|--------|
| Model | Manager-worker / handoff | Crew-of-peers / role-based |
| Setup | Programmatic configuration | YAML + Python |
| RAG | Native and deep | Integrable via tools |
| Flexibility | High (custom Workflows) | Medium (opinionated) |
| Community tools | 300+ on LlamaHub | Growing ecosystem |
| Production readiness | High (LlamaDeploy) | Medium |

Important note: LlamaIndex and CrewAI can be effectively combined, using LlamaIndex-powered tools within CrewAI crews for more sophisticated research flows.

### LlamaIndex AgentWorkflow vs OpenAI Agents SDK

**OpenAI Agents SDK** is focused on the OpenAI ecosystem with a simplified model of agents, handoffs, and guardrails. It is optimized for rapid deployment with OpenAI models.

**LlamaIndex AgentWorkflow** is LLM-agnostic and integrates with 300+ providers, offering greater flexibility at the cost of slightly higher complexity.

### LlamaIndex AgentWorkflow vs AutoGen

**AutoGen** (Microsoft) excels in multi-agent conversational scenarios where agents talk to each other to reach consensus. It is particularly strong in collaborative tasks and debate patterns.

**LlamaIndex AgentWorkflow** is more oriented toward structured workflows with explicit handoffs and document integration, less so toward free conversations between agents.

### Comparative Synthesis

LlamaIndex AgentWorkflow occupies a unique niche in the ecosystem: it is the framework that best integrates advanced RAG capabilities with agentic orchestration. For projects where data is at the center (document analysis, knowledge management, compliance, research), LlamaIndex offers a significant competitive advantage. For purely logical workflows without a documental component, LangGraph or CrewAI may be more direct.

---

## 10. Resources and References

### Official Documentation
- [LlamaIndex OSS Documentation](https://developers.llamaindex.ai/) - Complete framework documentation
- [Agents Guide](https://developers.llamaindex.ai/python/framework/use_cases/agents/) - Agents guide
- [Building an Agent](https://developers.llamaindex.ai/python/framework/understanding/agent/) - Tutorial for building agents
- [Multi-Agent Patterns](https://developers.llamaindex.ai/python/framework/understanding/agent/multi_agent/) - Multi-agent patterns
- [Workflows Introduction](https://developers.llamaindex.ai/python/llamaagents/workflows/) - Introduction to Workflows
- [Observability Guide](https://developers.llamaindex.ai/python/framework/module_guides/observability/) - Observability guide

### Blog Posts and Announcements
- [Introducing AgentWorkflow](https://www.llamaindex.ai/blog/introducing-agentworkflow-a-powerful-system-for-building-ai-agent-systems) - Official announcement of AgentWorkflow
- [Workflows 1.0 Announcement](https://www.llamaindex.ai/blog/announcing-workflows-1-0-a-lightweight-framework-for-agentic-systems) - Release of Workflows 1.0
- [Observability in Agentic Document Workflows](https://www.llamaindex.ai/blog/observability-in-agentic-document-workflows) - Observability in agentic workflows
- [Agent Workflows Overview](https://www.llamaindex.ai/workflows) - Overview of agentic workflows

### Repository and Code
- [GitHub: run-llama/llama_index](https://github.com/run-llama/llama_index) - Main repository
- [LlamaHub](https://llamahub.ai/) - Marketplace of integrations and tools
- [Agents Observability Demo](https://github.com/run-llama/agents-observability-demo) - Observability demo for agents

### Tutorials and Courses
- [Hugging Face Agents Course - LlamaIndex Workflows](https://huggingface.co/learn/agents-course/en/unit2/llama-index/workflows) - Course on agents with LlamaIndex
- [ReAct Agent with Query Engine](https://developers.llamaindex.ai/python/examples/agent/react_agent_with_query_engine/) - Example of ReAct agent with RAG
- [Building RAG with Agents](https://developers.llamaindex.ai/python/cloud/llamacloud/examples/building-rag-with-agents/) - Building RAG with agents

### In-depth Articles
- [Diving into LlamaIndex AgentWorkflow](https://www.dataleadsfuture.com/diving-into-llamaindex-agentworkflow-a-nearly-perfect-multi-agent-orchestration-solution/) - In-depth analysis of AgentWorkflow
- [LlamaIndex vs CrewAI](https://www.zenml.io/blog/llamaindex-vs-crewai) - Comparison with CrewAI
- [Comparing AI Agent Frameworks](https://atla-ai.com/post/ai-agent-frameworks) - General comparison between agentic frameworks
- [Top AI Agent Frameworks 2025](https://www.getmaxim.ai/articles/top-5-ai-agent-frameworks-in-2025-a-practical-guide-for-ai-builders/) - Comparative framework guide

### Installation

```bash
# Core
pip install llama-index-core

# Workflows (standalone)
pip install llama-index-workflows

# Observability
pip install llama-index-observability-otel

# LLM Provider (OpenAI example)
pip install llama-index-llms-openai

# Embedding (OpenAI example)
pip install llama-index-embeddings-openai

# Vector Store (Chroma example)
pip install llama-index-vector-stores-chroma
```

### Key Versions
- **LlamaIndex Core**: the fundamental package
- **Workflows 1.0**: standalone orchestration framework (2025)
- **AgentWorkflow**: agentic layer over Workflows
- **LlamaDeploy**: deployment and scaling in production
- **LlamaParse**: enterprise document parsing

---

*Document updated to March 2026. LlamaIndex is a rapidly evolving project: always check the official documentation for the latest APIs and patterns.*
