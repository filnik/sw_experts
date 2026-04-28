# LangGraph -- Orchestration Framework for Stateful Agents

## 1. Overview

### What is LangGraph

LangGraph is a low-level orchestration framework for the construction, management, and deployment of stateful, long-running AI agents. Developed by LangChain Inc., the project has reached significant adoption in the open-source community with over 27,000 GitHub stars, nearly 5,000 forks, 288 contributors, and documented usage in more than 37,000 projects. Leading companies such as Klarna, Uber, Replit, Elastic, and J.P. Morgan use it in production.

Unlike high-level frameworks that hide complexity behind rigid abstractions, LangGraph offers developers full control over agent orchestration logic, while maintaining powerful primitives for state management, persistence, streaming, and human interaction. Its architecture is inspired by distributed processing systems such as **Pregel** (Google's graph computation model) and **Apache Beam**, while the public interface draws inspiration from **NetworkX**, the well-known Python library for graph analysis.

### Relationship with LangChain

LangGraph is an independent project that can be used in a completely standalone way, without any dependency on LangChain. However, it integrates fluidly with the LangChain ecosystem when needed:

- **LangChain**: provides composable components (models, tools, retrievers, prompt templates) that can be used as nodes within a LangGraph graph.
- **LangSmith**: offers observability, evaluation, and debugging tools that integrate natively with LangGraph for execution tracing, capturing state transitions, and monitoring runtime metrics.
- **LangGraph Platform** (now LangSmith Deployment): scalable deployment infrastructure specifically designed for stateful and long-running workflows.

Installation is via pip:

```bash
pip install -U langgraph
```

---

## 2. Graph-Based Architecture

### The computational model

LangGraph organizes workflows as **directed graphs** in which nodes execute specific actions and edges define the flow of operations. This model allows workflows to dynamically adapt at runtime based on conditional logic embedded in the edges, supporting branching, cycles, parallelism, and conditional decision-making.

The underlying algorithm uses **message passing** to define a general program. Execution occurs in discrete **super-steps**: parallel nodes are executed within a single step, while sequential nodes traverse different steps. Each node transitions between an `inactive` state and an `active` state based on incoming messages. The graph halts when all nodes are inactive and there are no pending messages.

### Nodes

Nodes are Python functions that encode agent logic. They receive the current state as input, execute computations or side-effects, and return an updated state. A node can contain an LLM, a tool call, or simple procedural logic.

```python
from langgraph.graph import StateGraph, START, END

def call_model(state):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

def check_output(state):
    last_message = state["messages"][-1]
    if "error" in last_message.content:
        return {"messages": [HumanMessage(content="Try again")]}
    return state

builder = StateGraph(MessagesState)
builder.add_node("model", call_model)
builder.add_node("check", check_output)
```

Each node accepts up to three parameters:

1. **`state`**: the current state of the graph
2. **`config`**: a `RunnableConfig` object with runtime configuration
3. **`runtime`**: runtime context and dependencies

Behind the scenes, functions are converted into `RunnableLambda`, which add support for batch and asynchronous execution.

**Special nodes**:
- **`START`**: virtual node that represents the entry point of user input
- **`END`**: terminal node that marks workflow completion

### Edges

Edges determine the execution flow between nodes. LangGraph supports several types of edges:

**Normal edges** -- fixed transitions between nodes:

```python
builder.add_edge("model", "check")
builder.add_edge(START, "model")
```

**Conditional edges** -- dynamic routing based on state:

```python
def route_output(state):
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"
    return END

builder.add_conditional_edges("model", route_output)
```

The routing function returns the name of the next node (or can use an optional mapping dictionary). This enables creation of highly dynamic workflows where the execution path depends entirely on the current state.

**Conditional entry points** -- routing right from the start:

```python
builder.add_conditional_edges(START, classify_input)
```

### The Command primitive

The `Command` is a construct that combines state updates and control flow in a single return value:

```python
from langgraph.types import Command
from typing import Literal

def my_node(state) -> Command[Literal["next_node"]]:
    return Command(
        update={"key": "value"},
        goto="next_node"
    )
```

The parameters of `Command` include `update` (state modifications), `goto` (routing), `graph` (navigation between subgraphs), and `resume` (continuation after interrupt).

### Send objects (Map-Reduce pattern)

For dynamic edges whose number is not known a priori, LangGraph offers `Send` objects, fundamental for implementing map-reduce style patterns:

```python
from langgraph.types import Send

def continue_to_analysis(state):
    return [Send("analyze", {"document": doc}) for doc in state["documents"]]

builder.add_conditional_edges("splitter", continue_to_analysis)
```

### Compilation

A graph **must be compiled** before use. Compilation performs structural validation and enables runtime configuration:

```python
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["human_review"]
)
```

The `.compile()` method accepts checkpointer, breakpoints, and other runtime configurations. The **recursion limit** (default: 1000 super-steps) can be set to prevent infinite loops:

```python
result = graph.invoke(inputs, config={"recursion_limit": 25})
```

---

## 3. State Management

### State schema

The state is the shared data structure that represents the current snapshot of the application. It serves as the "working memory" of the graph, accessible for reading and writing by all nodes. LangGraph supports three modes for defining the schema:

**TypedDict** (recommended for basic usage):

```python
from typing import TypedDict, Annotated
from operator import add
from langgraph.graph import MessagesState

class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    current_plan: str
    iteration_count: int
    final_answer: str
```

**Dataclass** (when default values are needed):

```python
from dataclasses import dataclass, field

@dataclass
class AgentState:
    messages: Annotated[list, add_messages] = field(default_factory=list)
    plan: str = ""
```

**Pydantic BaseModel** (for recursive validation, but less performant):

```python
from pydantic import BaseModel

class AgentState(BaseModel):
    messages: list = []
    context: str = ""
```

### Channels and Annotation

State schemas are defined using `Annotation.Root`, which creates a structured representation of the graph's channels. Each key in the annotation represents a **channel** from which nodes can read and to which they can write. A node becomes active when it receives a new message (state update) on any of its incoming edges.

### Reducers

Reducers define how to apply node outputs to the current state. They represent one of the most powerful concepts of LangGraph:

- **Without a reducer**: the update overwrites the previous value
- **With reducer `operator.add`**: new values are appended to the existing list
- **With `add_messages`**: intelligent message handling with support for IDs, updates, and deduplication

```python
from typing import Annotated
from operator import add

class State(TypedDict):
    # Overwrite: each update replaces the value
    current_step: str

    # Accumulation: new results are added to previous ones
    results: Annotated[list, add]

    # Intelligent message handling
    messages: Annotated[list, add_messages]
```

The reducer receives two arguments: `current` (existing state) and `update` (new values from the node), returning the unified result. This allows, for example, merging a new response into the conversation history, or aggregating output from multiple agent nodes.

### MessagesState

`MessagesState` is a pre-built state class with a `messages` key that uses `add_messages` as its reducer. It is commonly extended with additional fields:

```python
from langgraph.graph import MessagesState

class MyState(MessagesState):
    current_tool: str
    iteration: int
```

### Memory Store

In addition to checkpointing (per-thread state), LangGraph offers a `Store` interface to share information **across different threads** through namespaces:

```python
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()
namespace = ("user_123", "preferences")
store.put(namespace, "food", {"preference": "pizza"})

# Retrieval
memories = store.search(namespace)
```

The Store also supports **semantic search** with embeddings:

```python
from langchain.embeddings import init_embeddings

store = InMemoryStore(
    index={
        "embed": init_embeddings("openai:text-embedding-3-small"),
        "dims": 1536,
        "fields": ["$"]
    }
)

results = store.search(namespace, query="What do you like to eat?", limit=3)
```

---

## 4. Checkpointing and Persistence

### Fundamental concepts

Checkpointing is LangGraph's central mechanism for state persistence. When a graph is compiled with a checkpointer, a **state snapshot** is saved at each super-step of execution, organized into **threads**.

A **thread** is a unique identifier assigned to a sequence of checkpoints. It accumulates state from multiple executions within the same conversation or workflow:

```python
config = {"configurable": {"thread_id": "conversation_42"}}
result = graph.invoke({"messages": [msg]}, config=config)
```

### StateSnapshot

Each checkpoint is represented as a `StateSnapshot` with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `values` | dict | Channel values at the checkpoint |
| `next` | tuple | Next nodes to execute (empty if complete) |
| `config` | dict | thread_id, checkpoint_ns, checkpoint_id |
| `metadata` | dict | Source, writes, step counter |
| `created_at` | string | ISO 8601 timestamp |
| `parent_config` | dict/None | Config of the previous checkpoint |
| `tasks` | tuple | Tasks scheduled for execution |

### Checkpointer Implementations

**InMemorySaver** -- development and testing:

```python
from langgraph.checkpoint.memory import InMemorySaver
checkpointer = InMemorySaver()
graph = builder.compile(checkpointer=checkpointer)
```

Fast but volatile: data is lost when the process restarts.

**SqliteSaver** -- prototyping and local workflows:

```python
from langgraph.checkpoint.sqlite import SqliteSaver
checkpointer = SqliteSaver.from_conn_string("checkpoints.db")
```

Available in synchronous (`SqliteSaver`) and asynchronous (`AsyncSqliteSaver`) versions.

**PostgresSaver** -- production:

```python
from langgraph.checkpoint.postgres import PostgresSaver
checkpointer = PostgresSaver.from_conn_string("postgresql://user:pass@host/db")
checkpointer.setup()
```

Used internally by LangSmith Deployment. Also available in async version.

**CosmosDBSaver** -- Azure environments:

For Azure deployments with persistence on Cosmos DB, with sync/async support.

### Time Travel and Replay

Time travel allows you to **re-execute previous runs** starting from a specific checkpoint, enabling debugging and exploration of alternative trajectories:

```python
# Get state history
history = list(graph.get_state_history(config))

# Find a specific checkpoint
before_tool = next(s for s in history if s.next == ("tool_node",))

# Filter by step
step_3 = next(s for s in history if s.metadata["step"] == 3)

# Find forks (manual updates)
forks = [s for s in history if s.metadata["source"] == "update"]

# Find interruptions
interrupted = next(
    s for s in history
    if s.tasks and any(t.interrupts for t in s.tasks)
)
```

The **replay** re-executes steps from a previous checkpoint: nodes before the checkpoint are skipped (results are already saved), while those after are re-executed, including LLM calls, API requests, and interrupts.

### Updating state

`update_state()` creates new checkpoints with modified values without altering the originals. Values pass through reducer functions:

```python
graph.update_state(
    config,
    {"messages": [HumanMessage(content="Correct the response")]},
    as_node="human_reviewer"
)
```

The optional `as_node` parameter controls which node will be executed next.

### Fault tolerance

If one or more nodes fail in a given super-step, the graph can be restarted from the last successfully completed step. **Pending writes** preserve the outputs of completed nodes, avoiding the re-execution of tasks already finished upon restoration.

### Serialization and Encryption

The default serializer is `JsonPlusSerializer`, which handles LangChain types, datetime, enum, and more using msgpack/JSON. For unsupported types (such as Pandas DataFrame), it is possible to enable the pickle fallback:

```python
from langgraph.checkpoint.serde.jsonplus import JsonPlusSerializer

checkpointer = InMemorySaver(
    serde=JsonPlusSerializer(pickle_fallback=True)
)
```

For encryption of persisted state:

```python
from langgraph.checkpoint.serde.encrypted import EncryptedSerializer

serde = EncryptedSerializer.from_pycryptodome_aes()
checkpointer = PostgresSaver.from_conn_string(conn_string, serde=serde)
```

The encryption reads the key from the `LANGGRAPH_AES_KEY` environment variable.

---

## 5. Fundamental Patterns

### 5.1 ReAct Agent

The ReAct (Reasoning + Acting) pattern is the most common. The agent reasons about the task, decides which tool to use, observes the result, and iterates until completion:

```python
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.prebuilt import ToolNode

tools = [search_tool, calculator_tool]
tool_node = ToolNode(tools)

def call_model(state: MessagesState):
    response = model.bind_tools(tools).invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: MessagesState):
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"
    return END

builder = StateGraph(MessagesState)
builder.add_node("agent", call_model)
builder.add_node("tools", tool_node)
builder.add_edge(START, "agent")
builder.add_conditional_edges("agent", should_continue)
builder.add_edge("tools", "agent")

graph = builder.compile()
```

The `agent -> tools -> agent` cycle continues until the model decides to respond directly without invoking any tool.

### 5.2 Plan-and-Execute

In this pattern, a planner agent creates a structured plan, and then an executor agent executes each step of the plan. The plan can be updated based on intermediate results:

```python
class PlanExecuteState(TypedDict):
    messages: Annotated[list, add_messages]
    plan: list[str]
    current_step: int
    results: Annotated[list, add]
    final_response: str

def planner(state):
    plan = llm.invoke(f"Create a plan for: {state['messages'][-1].content}")
    return {"plan": parse_plan(plan), "current_step": 0}

def executor(state):
    step = state["plan"][state["current_step"]]
    result = execute_step(step)
    return {
        "results": [result],
        "current_step": state["current_step"] + 1
    }

def should_continue(state):
    if state["current_step"] < len(state["plan"]):
        return "executor"
    return "synthesizer"
```

This pattern is particularly useful for complex tasks that require decomposition, such as in-depth research or report generation.

### 5.3 Multi-Agent Supervisor

The supervisor pattern implements a hierarchical multi-agent system where specialized agents operate under the coordination of a central supervisor agent:

```python
from langgraph_supervisor import create_supervisor

# Specialized agents
research_agent = create_react_agent(
    model, tools=[web_search], name="researcher"
)
writer_agent = create_react_agent(
    model, tools=[write_doc], name="writer"
)
reviewer_agent = create_react_agent(
    model, tools=[check_grammar], name="reviewer"
)

# Supervisor creation
workflow = create_supervisor(
    [research_agent, writer_agent, reviewer_agent],
    model=model,
    prompt="You are a project manager. Delegate tasks to the appropriate agents.",
    output_mode="last_message"
)

app = workflow.compile()
```

The supervisor analyzes incoming requests and routes them to the specialized agents using a tool-based handoff mechanism. It supports two output modes: `last_message` (only the final response) and `full_history` (full conversation).

### 5.4 Hierarchical Teams

Hierarchical teams extend the supervisor pattern across multiple levels. A high-level supervisor delegates to sub-teams, each with its own supervisor and specialized agents:

```python
# Research team
research_team = create_supervisor(
    [web_researcher, academic_researcher],
    model=model,
    prompt="Coordinate research between the web and academic sources."
)

# Writing team
writing_team = create_supervisor(
    [drafter, editor],
    model=model,
    prompt="Coordinate the drafting and review of the content."
)

# High-level supervisor
top_supervisor = create_supervisor(
    [research_team, writing_team],
    model=model,
    prompt="Manage the project by delegating research and writing to the teams."
)
```

This approach scales naturally for complex organizations, where each sub-team operates semi-autonomously with its own coordination logic.

### 5.5 Map-Reduce

The map-reduce pattern is fundamental for processing data in parallel. It uses `Send` objects to dynamically create parallel nodes and then aggregate the results:

```python
from langgraph.types import Send

class MapReduceState(TypedDict):
    documents: list[str]
    summaries: Annotated[list, add]
    final_summary: str

def map_documents(state):
    """Send each document to a parallel summarization node."""
    return [
        Send("summarize", {"document": doc})
        for doc in state["documents"]
    ]

def summarize(state):
    """Summarize a single document."""
    summary = llm.invoke(f"Summarize: {state['document']}")
    return {"summaries": [summary.content]}

def reduce_summaries(state):
    """Aggregate all summaries into a final one."""
    combined = "\n".join(state["summaries"])
    final = llm.invoke(f"Final synthesis from: {combined}")
    return {"final_summary": final.content}

builder = StateGraph(MapReduceState)
builder.add_node("summarize", summarize)
builder.add_node("reduce", reduce_summaries)
builder.add_conditional_edges(START, map_documents)
builder.add_edge("summarize", "reduce")
builder.add_edge("reduce", END)
```

This pattern is ideal for analyzing multiple documents, parallel content generation, and any scenario where multiple independent tasks must be executed and then aggregated.

---

## 6. Human-in-the-Loop

### Interruption and approval

LangGraph natively supports human-in-the-loop workflows, allowing execution to be suspended, state to be inspected, and resumption after human intervention. The runtime pauses execution, saves state, and waits for human input without blocking threads.

**Interrupt before** -- pause before the execution of a node:

```python
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_before=["execute_action"]
)
```

**Interrupt after** -- pause after the execution of a node:

```python
graph = builder.compile(
    checkpointer=checkpointer,
    interrupt_after=["generate_plan"]
)
```

### Typical Human-in-the-Loop flow

1. The graph executes up to the interruption point
2. State is automatically saved via the checkpointer
3. State can be inspected by the user (`graph.get_state(config)`)
4. The user approves, modifies, or rejects
5. Execution resumes from the exact point where it was suspended

```python
# Initial execution -- stops before "execute_action"
result = graph.invoke(input_data, config)

# State inspection
state = graph.get_state(config)
print(f"Next node: {state.next}")
print(f"Proposed action: {state.values['proposed_action']}")

# Modify state if necessary
graph.update_state(config, {"proposed_action": "modified_action"})

# Resume execution
final_result = graph.invoke(None, config)
```

### In-flight state modification

Updating state during an interruption creates a **new checkpoint** (fork) without altering the original state. Values pass through reducers, allowing both overwrite and accumulation:

```python
# Add an approval message
graph.update_state(
    config,
    {"messages": [HumanMessage(content="Approved. Proceed.")]},
    as_node="human_reviewer"
)
```

This mechanism is crucial for production scenarios where agent actions must be validated before execution: financial transaction approval, email review before sending, database query validation, and confirmation of irreversible actions.

---

## 7. Streaming

### Streaming categories

LangGraph supports three main categories of streaming data:

1. **Workflow progress**: state updates after the execution of each node
2. **LLM tokens**: language model tokens generated in real time
3. **Custom updates**: user-defined signals (e.g. "Processed 10/100 records")

### Streaming modes

Graphs expose the `stream` (synchronous) and `astream` (asynchronous) methods with five combinable modes:

| Mode | Description | Use case |
|----------|-------------|------------|
| `values` | Full state after each step | Complete state evolution |
| `updates` | Only state increments | When only changes matter |
| `messages` | LLM tokens + metadata | Chat applications with verbatim output |
| `custom` | User-defined custom data | Progress updates, metrics |
| `events` | Detailed execution events | Advanced debugging and observability |

### Practical examples

**Streaming state updates**:

```python
for chunk in graph.stream(
    {"messages": [HumanMessage(content="Analyze the data")]},
    config,
    stream_mode="updates"
):
    print(f"Node: {list(chunk.keys())[0]}")
    print(f"Output: {chunk}")
```

**Real-time LLM token streaming**:

```python
async for event in graph.astream_events(
    {"messages": [HumanMessage(content="Write an article")]},
    config,
    version="v2"
):
    if event["event"] == "on_chat_model_stream":
        token = event["data"]["chunk"].content
        print(token, end="", flush=True)
```

**Combined streaming** (multiple modes simultaneously):

```python
async for mode, chunk in graph.astream(
    inputs, config,
    stream_mode=["updates", "messages"]
):
    if mode == "updates":
        handle_state_update(chunk)
    elif mode == "messages":
        handle_token(chunk)
```

Best practices suggest always using async methods (`ainvoke`, `astream`) in I/O-bound applications to prevent thread blocking and enable real-time event propagation.

---

## 8. LangGraph Platform (LangSmith Deployment)

### Overview

LangGraph Platform (renamed LangSmith Deployment in October 2025) is a runtime for the deployment and management of stateful and long-running agentic workflows in production. It offers over **30 API endpoints** for personalized user experiences, horizontal scaling for bursty traffic, and a native persistence layer.

### Deployment options

| Option | Description | Plan |
|---------|-------------|-------|
| **Cloud (SaaS)** | Fully managed, deployment from LangSmith | Plus, Enterprise |
| **Hybrid** | SaaS control plane + self-hosted data plane | Enterprise |
| **Self-Hosted** | Entire platform in your own infrastructure | Enterprise |
| **Developer** | Free tier, up to 100k nodes/month | Free |

### Key features

- **One-click deployment**: from graph definition to production deployment
- **Automatic horizontal scaling**: native handling of bursty traffic typical of agentic workflows
- **Built-in persistence**: conversational memory and workflow state managed automatically
- **LangGraph Studio**: real-time visualization, detailed inspection of trajectories, support for branching, checkpointing, and memory modules for rewind and rerun of failures
- **Agent Registry**: centralized agent catalog with assistant versioning
- **Remote Graphs**: distributed multi-agent architectures where graphs can invoke other remotely deployed graphs
- **Monitoring**: management console with real-time metrics (errors, performance, costs), integrable with LangSmith for in-depth tracing and observability

### Deployment configuration

Deployment is configured via a `langgraph.json` file at the project root:

```json
{
    "graphs": {
        "my_agent": "./agent.py:graph"
    },
    "store": {
        "index": {
            "embed": "openai:text-embeddings-3-small",
            "dims": 1536,
            "fields": ["$"]
        }
    }
}
```

When using LangSmith's Agent Server, checkpointing is handled automatically without manual configuration, and encryption is enabled automatically when the `LANGGRAPH_AES_KEY` variable is present in the environment.

---

## 9. Comparison with Other Frameworks

### LangGraph vs CrewAI

| Aspect | LangGraph | CrewAI |
|---------|-----------|--------|
| **Paradigm** | Graph-based workflow | Role-based (team of agents) |
| **Abstraction level** | Low -- granular control | High -- organizational metaphor |
| **Learning curve** | Steep (requires thinking in graphs) | Accessible (intuitive concepts) |
| **State management** | Explicit with reducers and channels | Implicit, managed by the framework |
| **Flexibility** | Maximum -- any topology | Limited to predefined patterns |
| **Production** | Mature, used in enterprise | Growing, less battle-tested |
| **Ideal use case** | Complex workflows with state | Sequential processes with clear roles |

### LangGraph vs AutoGen (Microsoft)

| Aspect | LangGraph | AutoGen |
|---------|-----------|---------|
| **Paradigm** | Graph-based workflow | Conversational agents |
| **Communication** | Message passing on graphs | Natural conversation between agents |
| **State** | Explicit, structured, persistent | Implicit in conversation |
| **Human-in-the-loop** | Native with interrupt/resume | Supported via conversation |
| **Deployment** | Dedicated LangGraph Platform | Requires custom infrastructure |
| **Ideal use case** | Complex orchestration | Brainstorming, conversational tasks |

### LangGraph vs direct approach (without framework)

| Aspect | LangGraph | Custom code |
|---------|-----------|---------------|
| **State** | Managed with checkpointing | Manual implementation |
| **Persistence** | Built-in | To be built from scratch |
| **Fault tolerance** | Automatic | Developer's responsibility |
| **Streaming** | Native multi-mode | Custom implementation |
| **Human-in-the-loop** | Dedicated primitives | Application logic |
| **Overhead** | Learning curve | No additional dependencies |

### When to choose LangGraph

LangGraph is the ideal choice when:

- You need **granular control** over agent execution flow
- The workflow requires **complex state** with persistence and fault tolerance
- You implement **multi-agent patterns** with non-trivial topologies
- The system must support **human-in-the-loop** in production
- You require **advanced streaming** with multiple modes
- The application needs **time travel and debugging** of state
- You plan to deploy in **enterprise production** with monitoring

It is not the optimal choice for:
- Rapid prototypes where a simple LangChain chain is sufficient
- Teams without experience in graph architectures
- Simple applications with linear flow and no persistent state

---

## 10. Resources and References

### Official documentation

- **GitHub Repository**: https://github.com/langchain-ai/langgraph
- **Documentation**: https://docs.langchain.com/oss/python/langgraph/overview
- **API Reference**: https://reference.langchain.com/python/langgraph
- **Graph API**: https://docs.langchain.com/oss/python/langgraph/graph-api
- **Persistence**: https://docs.langchain.com/oss/python/langgraph/persistence
- **Streaming**: https://docs.langchain.com/oss/python/langgraph/streaming
- **Workflows and Agents**: https://docs.langchain.com/oss/python/langgraph/workflows-agents
- **LangGraph Platform**: https://docs.langchain.com/langgraph-platform
- **Community Forum**: https://forum.langchain.com

### In-depth articles and guides

- [LangGraph Platform GA -- Official Blog](https://blog.langchain.com/langgraph-platform-ga/)
- [Mastering LangGraph State Management in 2025](https://sparkco.ai/blog/mastering-langgraph-state-management-in-2025)
- [LangGraph AI Framework 2025: Complete Architecture Guide](https://latenode.com/blog/ai-frameworks-technical-infrastructure/langgraph-multi-agent-orchestration/langgraph-ai-framework-2025-complete-architecture-guide-multi-agent-orchestration-analysis)
- [LangGraph -- Architecture and Design (Medium)](https://medium.com/@shuv.sdr/langgraph-architecture-and-design-280c365aaf2c)
- [Building Multi-Agent Systems with LangGraph Supervisor](https://dev.to/sreeni5018/building-multi-agent-systems-with-langgraph-supervisor-138i)
- [LangGraph Streaming 101: 5 Modes](https://dev.to/sreeni5018/langgraph-streaming-101-5-modes-to-build-responsive-ai-applications-4p3f)
- [Human-in-the-Loop Plan-and-Execute Agents](https://www.marktechpost.com/2026/02/16/how-to-build-human-in-the-loop-plan-and-execute-ai-agents-with-explicit-user-approval-using-langgraph-and-streamlit/)
- [LangGraph Agents in Production: Architecture, Costs and Real-World Outcomes](https://www.alphabold.com/langgraph-agents-in-production/)

### Framework comparisons

- [CrewAI vs LangGraph vs AutoGen (DataCamp)](https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen)
- [A Detailed Comparison of Top 6 AI Agent Frameworks in 2026](https://www.turing.com/resources/ai-agent-frameworks)
- [LangGraph vs AutoGen vs CrewAI: Complete Comparison 2025](https://latenode.com/blog/platform-comparisons-alternatives/automation-platform-comparisons/langgraph-vs-autogen-vs-crewai-complete-ai-agent-framework-comparison-architecture-analysis-2025)

### Related libraries and packages

- `langgraph` -- core framework
- `langgraph-checkpoint` -- base interface for checkpointers (includes InMemorySaver)
- `langgraph-checkpoint-sqlite` -- SQLite checkpointer
- `langgraph-checkpoint-postgres` -- PostgreSQL checkpointer
- `langgraph-checkpoint-cosmosdb` -- Azure Cosmos DB checkpointer
- `langgraph-supervisor` -- library for multi-agent supervisor pattern
- `langgraph-cli` -- CLI for deployment and management

### Project metrics (March 2026)

- GitHub Stars: 27,600+
- Forks: 4,700+
- Contributors: 288
- Releases: 487+
- License: MIT
- Projects using it: 37,200+
