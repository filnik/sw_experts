# Google Agent Development Kit (ADK) - Complete Guide

## 1. Overview

### What ADK Is

The **Agent Development Kit (ADK)** is an open-source framework developed by Google for the creation, evaluation, and deployment of sophisticated AI agents. Initially released in 2025 and continuously evolving (with ADK 2.0 currently in Alpha), ADK positions itself as Google's response to the growing demand for professional tools for building multi-agent systems.

ADK adopts a **code-first** approach: agent development follows the same practices as traditional software engineering, with modular, testable, and composable structures. This distinguishes it from more declarative or GUI-based approaches, making the framework familiar to developers coming from the classic software development world.

### Positioning in the Google AI Landscape

ADK fits into the Google AI ecosystem as the orchestration layer for intelligent agents. It is optimized for **Gemini** and the Google Cloud ecosystem (in particular Vertex AI Agent Engine), but is designed to be:

- **Model-agnostic**: supports Gemini, Claude, GPT, Mistral, and over 100 models via LiteLLM.
- **Deployment-agnostic**: local execution, Vertex AI Agent Engine, Cloud Run, Docker, or custom infrastructure.
- **Framework-agnostic**: compatible with tools and agents from LangChain, LlamaIndex, CrewAI, and LangGraph.

The framework is available in four languages: **Python**, **TypeScript**, **Go**, and **Java**, thus covering most enterprise development ecosystems.

---

## 2. Architecture

### Code-First Approach

ADK's architecture revolves around fundamental primitives that compose like building blocks. Each component has a well-defined responsibility and can be configured, extended, or replaced independently of the others.

### Fundamental Components

| Component | Description |
|------------|-------------|
| **Agent** | Fundamental unit of work, designed for specific tasks |
| **Tool** | Operational capabilities: APIs, search, code execution, external services |
| **Session** | Conversational context management (event history + state) |
| **Memory** | Long-term cross-session context, distinct from session state |
| **Callback** | Custom hooks executed at specific points of the lifecycle |
| **Artifact** | Management of versioned files and binary data (images, PDFs, audio) |
| **Model** | Underlying LLM that powers reasoning in LlmAgents |
| **Runner** | Execution engine that orchestrates interactions between agents and backends |

### Execution Flow

The **Runner** is the heart of the runtime: it receives user inputs, routes them to the appropriate agent, manages tool execution, maintains session state, and produces output events. The typical cycle is:

1. The user sends a message.
2. The Runner creates an `InvocationContext` with session, state, and history.
3. The root agent processes the input, consulting the LLM model.
4. The model may request the use of tools or delegation to sub-agents.
5. The results are aggregated and returned as events.
6. Session state is updated with the produced deltas.

### Developer Tooling

ADK provides integrated tools for local development:

- **CLI**: command-line interface to create, run, and test agents.
- **Development UI**: graphical interface for inspecting execution, decision flow, and state.
- **Evaluation system**: multi-turn datasets for evaluating both the quality of final responses and the step-by-step execution trajectory.
- **Native streaming**: bidirectional streaming (text and audio) via the Gemini Live API.

---

## 3. Agent Types

ADK defines three main categories of agents, each with distinct characteristics and use cases.

### LlmAgent

The `LlmAgent` is the most versatile and powerful agent in the framework. It uses a Large Language Model as its central engine to understand natural language, reason, plan, generate responses, and dynamically decide how to proceed.

```python
from google.adk.agents import LlmAgent

research_agent = LlmAgent(
    name="research_agent",
    model="gemini-2.0-flash",
    instruction="""You are an expert research assistant.
    Analyze the user's request and provide detailed information.
    Current topic: {topic}""",
    tools=[search_tool, web_browse_tool],
    output_key="research_results"
)
```

**Key characteristics of the LlmAgent:**

- **Non-deterministic**: responses vary based on context and the model's reasoning.
- **Tool support**: can invoke tools to extend its capabilities beyond conversation.
- **Instruction templates**: supports `{key}` placeholders that are automatically resolved from the session state.
- **output_key**: automatically saves the model's final response to the session state with the specified key.
- **Dynamic delegation**: can transfer control to other agents via `transfer_to_agent()`.

### SequentialAgent

The `SequentialAgent` runs its sub-agents in the exact order in which they are specified in the list. The output of one agent can be passed as input to the next via the shared session state.

```python
from google.adk.agents import SequentialAgent, LlmAgent

pipeline = SequentialAgent(
    name="data_pipeline",
    sub_agents=[
        fetch_agent,      # Step 1: Retrieve data
        clean_agent,      # Step 2: Clean data
        analyze_agent,    # Step 3: Analyze data
        summarize_agent   # Step 4: Produce summary
    ]
)
```

**Typical use cases:**

- Multi-step data processing pipelines.
- Generator-critic flows where one agent produces content and the next validates it.
- Transformation chains where each step progressively enriches the result.

The communication mechanism between steps is based on shared state: each agent writes its results via `output_key` and the next reads them from the `session.state`.

### ParallelAgent

The `ParallelAgent` runs all of its sub-agents in parallel, concurrently. It acts as a manager assigning tasks to multiple employees simultaneously.

```python
from google.adk.agents import ParallelAgent, LlmAgent

parallel_research = ParallelAgent(
    name="multi_source_research",
    sub_agents=[
        web_search_agent,     # Web search
        academic_agent,       # Academic search
        news_agent            # News search
    ]
)
```

**Important characteristics:**

- All sub-agents share the same `session.state`.
- Each sub-agent receives a distinct branch of `InvocationContext`.
- Ideal for independent tasks that can be performed simultaneously.
- Results are aggregated in the shared state once all sub-agents have completed.

### LoopAgent

The `LoopAgent` runs its sub-agents sequentially, repeatedly in a loop. The loop terminates when the optional `max_iterations` is reached, or when a sub-agent returns an event with `escalate=True` in its Event Actions.

```python
from google.adk.agents import LoopAgent, LlmAgent

refinement_loop = LoopAgent(
    name="iterative_refinement",
    max_iterations=5,
    sub_agents=[
        draft_agent,    # Generates/improves the draft
        review_agent    # Evaluates and decides whether to continue
    ]
)
```

**Usage patterns:**

- **Iterative refinement**: one agent generates, another evaluates, the cycle continues until the desired quality is reached.
- **Polling and retry**: repetition of operations until success or timeout.
- **Convergence**: iteration until results stabilize.

### Custom Agent

By creating a subclass of `BaseAgent`, developers can implement unique operational logic, specific control flows, or specialized integrations not covered by standard types.

```python
from google.adk.agents import BaseAgent

class ValidationAgent(BaseAgent):
    async def _run_async_impl(self, context):
        # Custom validation logic
        data = context.state.get("input_data")
        if self.validate(data):
            context.state["validation_status"] = "passed"
        else:
            context.state["validation_status"] = "failed"
```

| Feature | LLM Agent | Workflow Agent | Custom Agent |
|----------------|-----------|----------------|--------------|
| **Engine** | Large Language Model | Predefined logic | Custom code |
| **Determinism** | Non-deterministic | Deterministic | Variable |
| **Use case** | Linguistic tasks, decisions | Flow orchestration | Specific requirements |
| **Flexibility** | High (LLM-driven) | Medium (structured) | Maximum |

---

## 4. Hierarchical Multi-Agent

### Tree Structure

ADK supports multi-agent systems through composable instances of `BaseAgent` organized hierarchically. Agents form tree structures via the `sub_agents` parameter during initialization. A fundamental constraint: **an agent instance can be added as a sub-agent only once**. Attempting to assign a second parent generates a `ValueError`.

```python
root_agent = LlmAgent(
    name="coordinator",
    model="gemini-2.0-flash",
    instruction="You are the coordinator. Delegate tasks to specialized agents.",
    sub_agents=[
        research_team,      # SequentialAgent with research sub-agents
        analysis_agent,     # LlmAgent for analysis
        report_agent        # LlmAgent for report generation
    ]
)
```

### Hierarchy Navigation

Navigation between agents uses:

- `parent_agent`: attribute to traverse to the parent.
- `find_agent(name)`: method to find a specific agent in the tree starting from the root.

### Communication Mechanisms

#### Shared Session State

The most common mechanism of communication between agents. One agent (or its tool/callback) writes a value to shared state, and a subsequent agent reads it:

```python
# Agent 1 writes
context.state['processed_data'] = processing_result

# Agent 2 reads
data = context.state.get('processed_data')
```

All agents share the same `session.state` through the `InvocationContext`, making communication transparent and direct.

#### LLM-Driven Delegation (Agent Transfer)

Agents can use the `transfer_to_agent(agent_name='target')` function to delegate control. The `AutoFlow`, used by default when sub-agents are present, intercepts this call, identifies the target agent via `root_agent.find_agent()`, and transfers the execution focus. To work effectively, every agent must have a **clear description** that guides the LLM model's decisions.

#### Explicit Invocation (AgentTool)

`AgentTool` encapsulates an agent instance as an invokable tool. It automatically generates a function declaration for the LLM model. When invoked, it runs the target agent, captures the final response, and propagates state and artifact changes to the parent's context.

```python
from google.adk.tools import AgentTool

specialist_tool = AgentTool(agent=specialist_agent)
coordinator = LlmAgent(
    name="coordinator",
    tools=[specialist_tool]  # The specialist agent is now a tool
)
```

### Common Multi-Agent Patterns

**Coordinator/Dispatcher**: a central `LlmAgent` receives requests and routes them to specialized sub-agents via transfer or tool invocation. It is the most flexible pattern, ideal when the type of request is not predictable in advance.

**Sequential Pipeline**: a `SequentialAgent` chains agents where the output of one feeds the input of the next through state. Perfect for deterministic multi-step processes.

**Parallel Fan-Out/Gather**: a `ParallelAgent` runs independent tasks in parallel; subsequent agents aggregate results from shared state keys.

**Hierarchical Decomposition**: multi-level trees that recursively decompose complex objectives into executable subtasks via delegation or tool invocation.

**Generator-Critic**: a `SequentialAgent` pairs agents where one generates content and another reviews/validates it via state communication. Essential pattern to ensure output quality.

**Iterative Refinement**: a `LoopAgent` repeatedly runs agents until conditions improve, with escalation signaling completion.

**Human-in-the-Loop**: agents that require human intervention at critical points, supporting policy-based approval workflows.

---

## 5. Tool System

### Overview

ADK's tool system is one of the framework's strengths. Tools are Python objects (or the equivalent in the chosen language) that extend agent capabilities beyond pure conversation, allowing interactions with APIs, databases, web services, and external systems.

### Tool Categories

#### Function Tools

The simplest tools to create: just define a Python function with type hints and docstring. ADK automatically generates the function declaration for the LLM model.

```python
def get_weather(city: str) -> dict:
    """Retrieves weather conditions for a specific city.

    Args:
        city: The name of the city for which to retrieve the weather.

    Returns:
        A dictionary with temperature and conditions.
    """
    # API call implementation
    return {"temperature": 22, "conditions": "sunny"}

agent = LlmAgent(
    name="weather_agent",
    model="gemini-2.0-flash",
    tools=[get_weather]
)
```

**Types of Function Tools:**

- **FunctionTool**: standard wrapping of a Python function.
- **LongRunningFunctionTool**: for asynchronous operations that take a long time, with support for intermediate notifications.
- **AgentTool**: encapsulates an entire agent as a reusable tool.

#### Built-in Tools

ADK provides pre-built tools for common operations:

- **Google Search**: integrated web search via Google APIs.
- **Code Execution**: execution of code in a sandboxed environment.
- **Vertex AI Extensions**: access to extensions of the Vertex AI platform.

#### Third-Party Tools

The framework supports integration with external ecosystems:

- **LangChain Tools**: tools from the LangChain ecosystem usable directly.
- **LlamaIndex Tools**: tools from the LlamaIndex ecosystem.
- **CrewAI Tools**: integration with CrewAI tools.

#### MCP Tools (Model Context Protocol)

The **Model Context Protocol** is an open standard designed to standardize communication between LLMs and external applications, data sources, and tools. ADK provides the `McpToolset` class as the primary mechanism for integrating tools from an MCP server:

```python
from google.adk.tools.mcp import McpToolset

mcp_tools = McpToolset(
    server_url="http://localhost:3000",
    # Automatically handles interaction with the MCP server
)

agent = LlmAgent(
    name="mcp_agent",
    model="gemini-2.0-flash",
    tools=[mcp_tools]
)
```

When an `McpToolset` instance is included in the agent's tool list, it automatically handles communication with the specified MCP server, making integration transparent.

### Agent as Tool

A particularly powerful pattern in ADK is the use of agents as tools of other agents. Via `AgentTool`, a specialized agent is encapsulated and made invokable from the parent's LLM model, allowing arbitrarily complex compositions.

---

## 6. Model Agnostic

### Design Philosophy

Although ADK is optimized for Gemini and the Google ecosystem, it is designed from the foundations to be model-agnostic. This means developers can use the most appropriate model for each specific task, or replace models without modifying application logic.

### Integration Mechanisms

ADK supports models through two distinct mechanisms:

#### 1. Direct Integration (String/Registry)

For models tightly integrated with Google Cloud, such as Gemini models accessible via Google AI Studio or Vertex AI, access is through providing the model name or endpoint string:

```python
agent = LlmAgent(
    name="gemini_agent",
    model="gemini-2.0-flash",  # Direct string
    instruction="..."
)
```

#### 2. Model Connectors (Wrapper Classes)

For external models, instantiate a specific wrapper class and pass it as the `model` parameter:

```python
from google.adk.models.lite_llm import LiteLlm

# Use Anthropic's Claude
claude_agent = LlmAgent(
    name="claude_agent",
    model=LiteLlm(model="claude-sonnet-4-20250514"),
    instruction="..."
)

# Use OpenAI's GPT-4o
gpt_agent = LlmAgent(
    name="gpt_agent",
    model=LiteLlm(model="gpt-4o"),
    instruction="..."
)
```

### Available Connectors

| Connector | Description | Supported Models |
|------------|-------------|-------------------|
| **Direct Gemini** | Native Google integration | Gemini 2.0 Flash, Gemini 2.0 Pro, etc. |
| **LiteLLM** | Unified multi-provider interface | 100+ models (Claude, GPT, Mistral, Llama, etc.) |
| **Ollama** | Local models | Any model supported by Ollama |
| **vLLM** | High-performance serving | Open-source models |
| **LiteRT-LM** | Lightweight runtime | Models optimized for edge |
| **Apigee AI Gateway** | Enterprise gateway | Models managed via Apigee |

### Multi-Model in the Same Application

One of ADK's strengths is the ability to use different models for different agents in the same application:

```python
# Reasoning agent with a powerful model
reasoning_agent = LlmAgent(
    name="reasoner",
    model="gemini-2.0-pro",
    instruction="Analyze in depth..."
)

# Classification agent with a fast model
classifier_agent = LlmAgent(
    name="classifier",
    model=LiteLlm(model="gpt-4o-mini"),
    instruction="Classify quickly..."
)

# Synthesis agent with Claude
summarizer_agent = LlmAgent(
    name="summarizer",
    model=LiteLlm(model="claude-sonnet-4-20250514"),
    instruction="Synthesize the results..."
)

# Orchestration with SequentialAgent
pipeline = SequentialAgent(
    name="multi_model_pipeline",
    sub_agents=[classifier_agent, reasoning_agent, summarizer_agent]
)
```

This approach allows optimizing costs and performance, assigning more expensive/powerful models only where necessary.

---

## 7. State Management

### Session and State

State management in ADK is articulated on two complementary levels:

- **Session**: represents a single thread of conversation between user and agent. Contains the chronological sequence of messages and actions, plus temporary data relevant only to that conversation.
- **State** (`session.state`): key-value collection that serves as the agent's "scratchpad" for a specific conversational thread.

### State Prefixes and Scopes

ADK uses a prefix system to determine the scope and persistence of state values:

| Prefix | Scope | Persistence | Use case |
|----------|-------|-------------|------------|
| None | Current session only | Depends on the service | Conversation progress |
| `user:` | All sessions of the user | Persistent services | Preferences, profile |
| `app:` | All users/sessions of the app | Persistent services | Global settings, templates |
| `temp:` | Current invocation only | Never persisted | Intermediate calculations, temporary flags |

The `temp:` state is particularly useful for working variables that should not survive beyond the current invocation, avoiding pollution of permanent state.

### State Update Methods

ADK offers three recommended methods:

#### 1. output_key (Simplest)

Automatically saves the final text response of an `LlmAgent` into the state:

```python
greeting_agent = LlmAgent(
    name="Greeter",
    model="gemini-2.0-flash",
    instruction="Generate a friendly greeting.",
    output_key="last_greeting"  # The response is saved here
)
```

#### 2. EventActions.state_delta (Complex Updates)

To update multiple keys or non-string values:

```python
state_changes = {
    "task_status": "active",
    "user:login_count": 5,
    "temp:validation_needed": True
}
actions = EventActions(state_delta=state_changes)
event = Event(invocation_id="inv_123", author="system", actions=actions)
await session_service.append_event(session, event)
```

#### 3. CallbackContext/ToolContext (In Callbacks and Tools)

Direct modification of state inside callbacks or tool functions:

```python
def my_tool_function(context: ToolContext):
    count = context.state.get("user_action_count", 0)
    context.state["user_action_count"] = count + 1
    # Changes are automatically captured in state_delta
```

### Templates in Instructions

State can be injected directly into agent instructions via `{key}` placeholders:

```python
agent = LlmAgent(
    model="gemini-2.0-flash",
    instruction="Write a story about: {topic}"
    # If session.state['topic'] = "friendship",
    # the instruction becomes: "Write a story about: friendship"
)
```

### Artifact Management

Artifacts represent versioned binary data (files, images, audio, PDFs) identified by unique names within specific scopes. They are represented using the standard `google.genai.types.Part` object with an `inline_data` structure containing `data` (raw bytes) and `mime_type`.

**Automatic versioning**: every call to `save_artifact()` creates a new version (starting from 0). `load_artifact()` retrieves the latest version by default, or a specific one if indicated.

**Artifact Scoping:**

- **Session-scoped** (default): simple filename like `"report.pdf"`, accessible only in the current session.
- **User-scoped**: `"user:"` prefix like `"user:profile.png"`, accessible in all sessions of the user.

**Available services:**

- `InMemoryArtifactService`: fast but ephemeral, for local development.
- `GcsArtifactService`: persistent on Google Cloud Storage, for production.

### Memory (Long-Term Memory)

Distinct from session state, **Memory** is a searchable cross-session knowledge archive. It represents information that extends beyond the single conversation:

- **SessionService**: manages the lifecycle of conversational threads.
- **MemoryService**: manages the long-term knowledge archive, with ingestion and search capabilities.

---

## 8. Graph-Based Workflows

### Introduction (ADK 2.0)

ADK 2.0 introduces **graph-based workflows**, a significant evolution compared to purely prompt-based orchestration. Graph-based workflows allow defining the overall process explicitly in code, with each step defined as an execution node in a graph.

**Important note**: ADK 2.0 is currently in Alpha release and may cause breaking changes compared to previous versions. It is not recommended for production environments where backward compatibility is necessary.

### Graph Components

**Nodes**: individual units of execution that can be:
- AI agents (powered by LLM)
- Custom code functions
- Tools
- Nested workflows
- Human input

**Edges**: connections that define the execution flow between nodes, supporting sequential transitions, conditional routing, and branching logic.

### Defining a Workflow

The `Workflow` class assembles nodes using the `edges` parameter. Execution always starts from the keyword `START`:

```python
from google.adk.agents import Workflow

root_agent = Workflow(
    name="root_agent",
    edges=[
        ("START", city_generator_agent, lookup_time_function,
         city_report_agent, completed_message_function)
    ]
)
```

### Conditional Routing and Branching

Conditional routing uses a router function that returns an `Event` with a `route` parameter:

```python
def router(node_input: str):
    return Event(route="RUN_TASK_C")

workflow = Workflow(
    name="conditional_workflow",
    edges=[
        ("START", task_A_node, router),
        (router, {
            "RUN_TASK_B": task_B_node,
            "RUN_TASK_C": task_C_node,
        }),
    ]
)
```

The router evaluates conditions and emits an event with a specific route value. The next entry in the edges maps these values to target nodes, allowing arbitrarily complex branching.

### Multi-Route Routing

A router function can return multiple routes, activating multiple paths:

```python
def message_classifier(node_input: str):
    routes = node_input.split(",")
    return Event(route=routes)

workflow = Workflow(
    name="multi_route_workflow",
    edges=[
        ("START", process_message, message_classifier),
        (message_classifier, {
            "BUG": response_bug_handler,
            "CUSTOMER_SUPPORT": response_support_handler,
            "LOGISTICS": response_logistics_handler,
        })
    ]
)
```

### Parallel Execution in Graphs

Multiple `START` entries enable the parallel start of tasks:

```python
workflow = Workflow(
    name="parallel_workflow",
    edges=[
        ("START", parallel_task_A),
        ("START", parallel_task_B),
    ]
)
```

Parallel outputs converge using a **JoinNode**, which waits for the completion of each parallel task and then passes the collection of outputs to the next node.

### Nested Workflows

`Workflow` objects can be embedded as nodes within parent workflows, enabling modular design and code reuse:

```python
sub_workflow = Workflow(
    name="data_processing",
    edges=[("START", fetch_node, transform_node, validate_node)]
)

main_workflow = Workflow(
    name="main",
    edges=[("START", input_node, sub_workflow, output_node)]
)
```

### Advantages of Graph Workflows

- **Explicit task mapping**: replaces long and fragile prompts with clear code structures.
- **Branching and state management**: native support for complex conditional logic.
- **Deterministic execution**: reliable combination of AI and code.
- **Improved debugging**: inspectable and traceable execution flows.
- **Modular composition**: reusable workflows as building blocks.

### Current Limitations

Graph-based workflows are not compatible with the Live Streaming functionality and with some third-party integrations. Being in Alpha, the API may undergo significant changes in subsequent versions.

---

## 9. Comparison with LangGraph, CrewAI, OpenAI Agents SDK

### Comparative Table

| Aspect | Google ADK | LangGraph | CrewAI | OpenAI Agents SDK |
|---------|-----------|-----------|--------|-------------------|
| **Orchestration model** | Hierarchical trees + graphs | Directed graphs with conditional edges | Role-based crews with processes | Explicit handoffs |
| **Approach** | Code-first, modular | Graph-first, explicit state | Role-based DSL | Minimalist, 4 primitives |
| **Model agnostic** | Yes (LiteLLM, 100+ models) | Yes (via LangChain) | Yes (via LiteLLM) | OpenAI only |
| **Learning curve** | Medium | Medium-high | Low | Low |
| **Production maturity** | Early (Alpha for v2) | High (LangSmith) | Medium | High |
| **Multi-agent** | Hierarchical, workflow, graphs | Graphs with subgraphs | Crews with roles and processes | Linear handoffs |
| **Streaming** | Bidirectional (text+audio) | Supported | Limited | Supported |
| **Deployment** | Vertex AI, Cloud Run, Docker | LangServe, custom | Custom | OpenAI platform |
| **Tool ecosystem** | Google, MCP, LangChain, LlamaIndex | LangChain native | LangChain, custom | Function calling |
| **Observability** | Built-in eval, debug UI | LangSmith (complete) | Limited | Integrated tracing |

### Google ADK vs LangGraph

**ADK** offers a richer orchestration model out-of-the-box with SequentialAgent, ParallelAgent, and LoopAgent as first-class primitives. Native integration with the Google ecosystem (Vertex AI, Cloud Storage, Google Search) makes it the natural choice for teams already invested in Google Cloud.

**LangGraph** excels in graph flexibility with conditional edges, advanced checkpointing, and deep integration with LangSmith for observability. It is more mature in production and has a larger community. Explicit state management via `StateGraph` offers granular control that ADK is only achieving with the graph workflows of version 2.0.

**When to choose ADK**: Google Cloud teams, need for bidirectional streaming, preference for hierarchical agent composition.
**When to choose LangGraph**: need for advanced observability (LangSmith), complex graph workflows in production, large community and resources.

### Google ADK vs CrewAI

**CrewAI** adopts a role-based paradigm where agents have roles, objectives, and backstories, making the definition of agent teams intuitive and quick (~20 lines to start). The learning curve is the lowest among all frameworks.

**ADK** offers greater architectural flexibility with its various agent types and composition patterns. The ability to use agents as tools and the dynamic delegation system are more sophisticated than CrewAI's process model.

**When to choose ADK**: complex enterprise applications, need for deterministic workflows, Google Cloud integration.
**When to choose CrewAI**: rapid prototyping, multi-agent teams with well-defined roles, minimal learning curve.

### Google ADK vs OpenAI Agents SDK

**OpenAI Agents SDK** is the most minimalist: four primitives (agents, handoffs, guardrails, tracing) and a focus on simplicity and development speed. The integrated tracing and guardrails are unique strengths.

**ADK** offers a more complete ecosystem with artifact management, cross-session memory, workflow agents, and multi-model support. Deployment flexibility is significantly higher.

**When to choose ADK**: need for model-agnostic, flexible deployment, artifact management, complex workflows.
**When to choose OpenAI SDK**: already in the OpenAI stack, need for integrated guardrails, preference for minimal API.

---

## 10. Resources and References

### Official Documentation

- **ADK Documentation**: https://google.github.io/adk-docs/
- **Technical Overview**: https://google.github.io/adk-docs/get-started/about/
- **Multi-Agent Systems**: https://google.github.io/adk-docs/agents/multi-agents/
- **Agent Types**: https://google.github.io/adk-docs/agents/
- **Graph Workflows**: https://google.github.io/adk-docs/workflows/
- **Graph Routes**: https://google.github.io/adk-docs/workflows/graph-routes/
- **State Management**: https://google.github.io/adk-docs/sessions/state/
- **Artifacts**: https://google.github.io/adk-docs/artifacts/
- **AI Models**: https://google.github.io/adk-docs/agents/models/

### GitHub Repositories

- **ADK Python**: https://github.com/google/adk-python
- **ADK Go**: https://github.com/google/adk-go
- **ADK Docs**: https://github.com/google/adk-docs

### Google Cloud

- **Vertex AI Agent Builder**: https://docs.cloud.google.com/agent-builder/agent-development-kit/overview
- **Multi-Agentic Systems Blog**: https://cloud.google.com/blog/products/ai-machine-learning/build-multi-agentic-systems-using-google-adk

### Blogs and Tutorials

- **ADK TypeScript Announcement**: https://developers.googleblog.com/introducing-agent-development-kit-for-typescript-build-ai-agents-with-the-power-of-a-code-first-approach/
- **Multi-Agent Patterns in ADK**: https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/
- **ADK Introduction Blog**: https://developers.googleblog.com/en/agent-development-kit-easy-to-build-multi-agent-applications/
- **LiteLLM + ADK Tutorial**: https://docs.litellm.ai/docs/tutorials/google_adk
- **Google ADK Java Codelab**: https://codelabs.developers.google.com/adk-java-getting-started

### Comparisons and Analyses

- **ADK vs LangGraph (ZenML)**: https://www.zenml.io/blog/google-adk-vs-langgraph
- **AI Agent Frameworks Comparison (Langfuse)**: https://langfuse.com/blog/2025-03-19-ai-agent-comparison
- **Multi-Agent Framework Comparison**: https://newsletter.victordibia.com/p/autogen-vs-crewai-vs-langgraph-vs
- **Best AI Agent Frameworks 2025 (LangWatch)**: https://langwatch.ai/blog/best-ai-agent-frameworks-in-2025-comparing-langgraph-dspy-crewai-agno-and-more
