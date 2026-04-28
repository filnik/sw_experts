# Harrison Chase -- LangChain, LangGraph and the Context Engineering Revolution

---

## 1. Profile and Background

**Harrison Chase** is the co-founder and CEO of **LangChain**, one of the most influential figures in the applied artificial intelligence ecosystem. His professional trajectory represents an exemplary case of how deep technical skills, combined with sharp entrepreneurial intuition, can catalyze an entire sector.

### Academic Background

Chase graduated from **Harvard University** in 2017 with a specialization in **Statistics and Computer Science**. His Harvard education provided him with solid foundations both in quantitative reasoning and in the design of complex software systems -- skills that would prove decisive in building LangChain.

### Professional Path Pre-LangChain

**Kensho Technologies (2017-2019)**: Right after Harvard, Chase joined Kensho, a fintech startup specialized in data analysis for the financial sector. Here he led the **entity linking** team, working on connecting disparate data points into coherent and actionable insights. This experience exposed him to the fundamental problems of information integration and structuring -- themes that would define his future work.

**Robust Intelligence (2019-2022)**: For over three years, Chase held the role of **Machine Learning Engineer** and later **Head of ML** at Robust Intelligence, a company focused on observability and integrity of machine learning models. Here he refined his skills in testing, validation, and monitoring of complex models in production. The experience with robustness and reliability of ML systems is reflected directly in LangChain's design philosophy, where observability and debugging are fundamental pillars.

### The Birth of the Entrepreneur

The idea for LangChain was born in the autumn of 2022, in the midst of the Large Language Models explosion. Chase recognized a critical gap: language models were extraordinarily powerful, but a coherent infrastructure for integrating them into real applications was missing. His philosophy is rooted in **collaboration, modularity, and developer-first principles** -- building tools that foster creativity without overwhelming users with unnecessary complexity.

---

## 2. LangChain: The Ecosystem

### The Origins (October 2022)

LangChain was launched in **October 2022** as an open source project, while Chase was still working at Robust Intelligence. The fundamental insight was simple but powerful: create a **modular framework** to connect Large Language Models with third-party tools, external memories, and data sources. At a time when most developers interacted with GPT-3 through simple API calls, LangChain proposed a higher abstraction: composable chains of operations, where each link could be an LLM, a retriever, a memory, or a tool.

### Explosive Growth (2023)

2023 marked LangChain's rapid rise:

- **April 2023**: LangChain incorporates as a company and raises over **$20 million** from Sequoia Capital at a valuation of at least **$200 million**, just one week after a seed round of **$10 million** led by Benchmark.
- **Q3 2023**: Introduction of the **LangChain Expression Language (LCEL)**, a declarative language for defining action chains in a more concise and readable way.
- **October 2023**: Release of **LangServe**, which allows chains to be deployed as scalable APIs, bridging the gap between prototyping and production.

The framework reaches impressive numbers: over **15 million monthly downloads**, more than **75,000 stars on GitHub**, and a community of over **3,000 contributors**.

### Platform Maturation (2024)

- **February 2024**: Release of **LangSmith**, a closed-source platform for observability and evaluation of LLM applications. LangSmith introduces unified tracing, evaluation, and monitoring, allowing teams to debug, test chains and agents, and track quality over time.
- **2024**: Announcement of a **Series A of $25 million** led by Sequoia Capital.
- **2024**: LangGraph becomes generally available, introducing a graph runtime with explicit state, routers, branches, loops, retries, and checkpointers for persistence.

### The Enterprise Era (2025-2026)

- **May 2025**: **LangGraph Platform** reaches General Availability, providing managed infrastructure for deploying stateful and long-running AI agents.
- **2025**: **Series B round of $100 million** led by IVP, consolidating LangChain's **unicorn** status.
- The framework reaches version **1.0**, with the introduction of node caching, deferred nodes, and hooks for complex state machine architectures.

### Ecosystem Components

The LangChain ecosystem today is articulated into five main components:

| Component | Function | Target |
|---|---|---|
| **LangChain** | Core framework for building modular LLM workflows (prompt, tool, memory, retriever) | Backend developers |
| **LangGraph** | Graph runtime for stateful agents with cycles, branches, and persistence | Agent architects |
| **LangSmith** | Tracing, evaluation, and monitoring platform (framework-agnostic) | ML/QA teams |
| **LangServe** | Deploy chains as production-ready APIs | DevOps/MLOps |
| **LangFlow** | Visual drag-and-drop interface for rapid prototyping | Non-coders, teams, workshops |

---

## 3. LangGraph: Agent Framework

### The Graph Philosophy

LangGraph represents the most significant conceptual evolution in the LangChain ecosystem. While LangChain's linear chains were adequate for simple pipelines (retrieval-augmented generation, summarization), **agentic systems** require something fundamentally different: **cycles, conditions, persistent state, and parallelism**.

LangGraph models agent workflows as **directed graphs** (potentially cyclic), using a paradigm inspired by **Google's Pregel** system for distributed message passing. Execution proceeds through **discrete super-steps**: nodes that can execute in parallel belong to the same super-step, while sequential ones belong to separate super-steps.

### The Three Architectural Pillars

#### State

State is the shared data structure that represents the current snapshot of the application. It is typically defined as a `TypedDict` or a **Pydantic** model and carries all relevant information from one node to another:

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    messages: Annotated[list, add_messages]
    plan: str
    current_step: int
    context: dict
    final_output: str
```

State is the "connective tissue" of the entire system: every node reads it, modifies it, and passes it to the next node. **Reducers** (like `add_messages`) define how multiple modifications to the same field are combined.

#### Nodes

Nodes are **Python functions** (or LangChain Runnable) that receive the current state as input and return an updated state. Each node encapsulates a logical unit of work:

```python
def planning_node(state: AgentState) -> dict:
    """Node that generates an execution plan."""
    plan = llm.invoke(
        f"Given the context: {state['context']}, "
        f"generate a structured plan to complete the task."
    )
    return {"plan": plan.content, "current_step": 0}

def execution_node(state: AgentState) -> dict:
    """Node that executes the current step of the plan."""
    result = execute_step(state["plan"], state["current_step"])
    return {
        "messages": [AIMessage(content=result)],
        "current_step": state["current_step"] + 1
    }

def review_node(state: AgentState) -> dict:
    """Node that evaluates the result and decides whether to continue."""
    evaluation = llm.invoke(
        f"Evaluate the following output: {state['messages'][-1].content}"
    )
    return {"context": {"evaluation": evaluation.content}}
```

#### Edges

Edges define the possible paths between nodes and the conditions under which they are traversed. There are three types:

- **Normal edges**: fixed and deterministic transitions between nodes.
- **Conditional edges**: functions that dynamically determine the next node based on the current state.
- **Entry/exit edges**: special connections to `START` and `END`.

```python
def should_continue(state: AgentState) -> str:
    """Conditional routing function."""
    if state["current_step"] >= len(state["plan"].split("\n")):
        return "review"
    if state["context"].get("evaluation") == "approved":
        return "end"
    return "execute"

# Graph construction
graph = StateGraph(AgentState)
graph.add_node("plan", planning_node)
graph.add_node("execute", execution_node)
graph.add_node("review", review_node)

graph.add_edge(START, "plan")
graph.add_edge("plan", "execute")
graph.add_conditional_edges("execute", should_continue, {
    "execute": "execute",
    "review": "review",
    "end": END
})
graph.add_conditional_edges("review", should_continue, {
    "execute": "execute",
    "end": END
})

app = graph.compile(checkpointer=MemorySaver())
```

### Checkpointing: Persistence and Time Travel

The **checkpointing** system is one of LangGraph's most innovative contributions. A checkpoint is created at each super-step boundary, saving a complete snapshot of the graph's state.

**What checkpointing enables:**

- **Conversational memory**: state persists across multiple interactions organized into threads.
- **Human-in-the-loop**: humans can inspect, interrupt, and approve graph steps. The graph can be paused, allowing human review, and then resumed.
- **Time travel**: the ability to "travel through time" by re-executing previous executions for debugging or analysis. State forks can be created at arbitrary checkpoints to explore alternative trajectories.
- **Fault tolerance**: if one or more nodes fail in a given super-step, the graph can be restarted from the last successfully completed step.

**Supported persistence backends:**

| Backend | Use | Characteristics |
|---|---|---|
| `MemorySaver` | Development/test | In-memory, no real persistence |
| `SqliteSaver` | Local prototypes | Persistence on local file |
| `PostgresSaver` | Production | Scalable, reliable |
| `DynamoDBSaver` | AWS production | Metadata in DynamoDB, payload in S3 |
| `CouchbaseSaver` | Distributed production | Multi-datacenter |

The transition from prototype to production is as simple as replacing the checkpointer: switching from `MemorySaver` to `DynamoDBSaver` immediately provides persistent and scalable state management.

### Compilation and Immutability

Before execution, the graph undergoes a **compilation** process that validates the connections between nodes, identifies cycles, and optimizes execution paths. Once compiled, the graph becomes **immutable**, ensuring consistent behavior across all executions and preventing runtime modifications that could destabilize the workflow.

---

## 4. Context Engineering

### The Definition

**Context engineering** is the key concept that, according to Harrison Chase, defines LangChain's entire philosophy and represents the future of AI application development. The definition Chase prefers is:

> "Context engineering is building dynamic systems to provide the right information and tools, in the right format, so that the LLM can plausibly complete the task."

This definition contains five critical elements, each of which deserves an in-depth analysis.

### The Five Pillars of Context Engineering

**1. "Systems" -- Not single prompts, but architectures**

Context is not a static string written once. It is the result of a **complex system** that assembles information from multiple sources: developer instructions, user input, previous interactions, results of tool calls, data from external sources. This system must integrate all these sources into a coherent whole.

**2. "Dynamic" -- Runtime adaptivity**

The system is not static. Context assembly must **adapt** as new information flows from various sources during execution. An agent operating for hours must continuously reassess which context is relevant for the current step.

**3. "Right information" -- Information quality**

The "garbage in, garbage out" principle applies forcefully. Without relevant context, performance inevitably degrades. Most agent failures **do not depend on model limitations**, but on insufficient or inadequate context.

**4. "Right tools" -- Beyond information**

Beyond data, models require **appropriate tools** to retrieve information, perform actions, or interact with external systems. The selection and configuration of available tools is itself a form of context engineering.

**5. "Right format" -- Presentation matters**

The way of presenting is as significant as the content. Concise and structured communication outperforms verbose JSON blobs. The format must be optimized for the model's "comprehension."

### Context Engineering vs. Prompt Engineering

The relationship between the two concepts is one of **inclusion**: prompt engineering is a **subset** of context engineering. While prompt engineering focuses on word formulation -- how to phrase a request to obtain better responses -- context engineering embraces a much broader scope:

- **Tool integration** with digestible output formatting
- **Conversation summarization** for prolonged interactions
- **Retrieval of user preferences** from historical data
- **Dynamic retrieval** of information and insertion into context
- **Clear behavioral instructions** in system prompts
- **Memory management** at short and long term
- **Intelligent compaction** of context when the token budget runs out

Chase emphasizes that even having all the necessary context, the way you assemble it in the prompt **still matters enormously**. But in the hierarchy of priorities, providing complete and structured context is **much more important** than any magical formulation.

### Why Context Engineering is the Future

As LLM applications evolve from single prompts to complex agentic systems, context engineering becomes **the most important skill** an AI engineer can develop. Chase states it explicitly:

> "When agents fail, they fail because they don't have the right context. When they succeed, they succeed because they have the right context."

This principle radically shifts the focus of optimization: it is not the model that needs improvement (that improves on its own with each new release), but the **system that surrounds it**.

---

## 5. Deep Agents and Long-Horizon Tasks

### The Three Eras of AI Agents

Harrison Chase identifies three technical eras in the evolution of agents:

**Era 1 -- The Early LangChain (2022-2023)**: Raw text-in/text-out models without native tool calling or structured reasoning capabilities. Developers had to build everything manually: output parsing, error handling, retry loops.

**Era 2 -- Custom Cognitive Architectures (2023-2024)**: Foundation models begin to be trained with tool calling and planning capabilities. Custom cognitive architectures emerge (ReAct, Plan-and-Execute, Tree of Thoughts) that require sophisticated orchestration.

**Era 3 -- Long-Horizon Agents (Mid 2024 onwards)**: Agents operating over extended time horizons finally work reliably, thanks to better models and especially **more sophisticated harnesses**.

### What Are Deep Agents

**Deep Agents** represent the next evolution of LLM-driven autonomy. Chase describes them as "agents that can operate for minutes instead of seconds, tackle broader objectives, and go deeper into specific domains."

The key distinction with respect to traditional agents is the difference between **framework** and **harness**:

- **Framework**: non-opinionated abstractions around models, tools, and memory (LangGraph is a framework).
- **Harness (Deep Agents)**: "batteries included" approach with opinionated defaults, including planning tools, context compaction strategies, and file system integration.

### The Four Technical Pillars of Deep Agents

**1. Dynamic Planning Tools**

Instead of pre-scripted workflows, deep agents have **planning tools that the LLM can invoke dynamically**. The model decides when and how to plan without rigid process engineering. This approach is fundamentally different from defining a static workflow a priori.

**2. File System as Memory**

Chase declares himself "completely file system pilled". The file system represents a **natural and powerful** way to represent an agent's state. It allows:

- Offloading memory when the context window fills up
- Preserving documentation and intermediate results
- Managing long contexts without exhausting the token budget
- Applying summarization strategies: context is compacted, but the full history remains accessible on file for retrieval when needed

**3. Sub-Agent Architecture**

Specialized agents handle domain-specific or complex tasks with **isolated contexts**, preventing token bloat and "cognitive confusion" in the main agent. Each sub-agent operates in its reduced context, receives a specific task, and returns a result to the coordinator.

**4. Advanced Prompting**

System prompts function as **primary alignment mechanisms**. Chase notes that "Claude Code's system prompt is nearly 2,000 lines long", demonstrating how prompt engineering is elevated to a critical architectural component in deep agents.

### Killer Use Cases for Long-Horizon Agents

Chase identifies four domains where long-horizon agents produce "robust first drafts":

1. **Coding**: the most developed domain, where agents like Claude Code, Cursor, and Codex operate for minutes/hours on complex codebases.
2. **AI SRE (Site Reliability Engineering)**: incident investigation, log analysis, diagnosis of problems in production.
3. **Research and report generation**: synthesis of information from multiple sources, comparative analyses, due diligence.
4. **Advanced customer support**: as in the Klarna case, where agents handle complex interactions that require access to multiple systems.

### Trace as a Fundamental Artifact

One of Chase's most profound insights concerns the role of **traces** in the new paradigm:

In traditional software, **code is the source of truth**: behavior is deterministic and knowable before deployment. In agentic systems, behavior **emerges from the model and context** -- it is non-deterministic and knowable only through execution.

Traces document exactly what context reaches the LLM at each step, becoming essential for:
- **Debugging**: understanding why an agent made a wrong decision
- **Testing**: verifying that the context is correct in known scenarios
- **Collaboration**: "send us a trace" replaces "show us the code" as a support request

### Persistent Memory and Self-Improvement

Chase sees persistent memory as a **future differentiator and competitive moat**. The concept is that agents **review their own traces and update their own instructions** -- essentially learning from interactions.

Chase prefigures the concept of **"sleep time compute"**: agents that improve themselves overnight by analyzing the day's traces. However, he notes that the utility of memory varies by context: it is highly valuable for specific workflows (like his personal email agent), but less impactful for one-off interactions.

### Hybrid Async/Sync UIs

Long-running agents require **asynchronous management** (starting multiple agents in parallel) with the ability to switch to **synchronous mode** for feedback and course correction. The concept of **"Agent Inbox"** envisions ambient agents that operate in the background and can reach the user when needed, with sync modes for real-time interaction.

---

## 6. Practical Architecture

### Pattern 1: ReAct Agent with LangGraph

The most common pattern for an agent with tool calling:

```python
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import ToolNode
from langchain_openai import ChatOpenAI

# State definition
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]

# Setup
tools = [search_tool, calculator_tool, file_tool]
model = ChatOpenAI(model="gpt-4").bind_tools(tools)

# Nodes
def call_model(state: AgentState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState):
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"
    return "end"

# Graph construction
graph = StateGraph(AgentState)
graph.add_node("agent", call_model)
graph.add_node("tools", ToolNode(tools))

graph.add_edge(START, "agent")
graph.add_conditional_edges("agent", should_continue, {
    "tools": "tools",
    "end": END
})
graph.add_edge("tools", "agent")

agent = graph.compile(checkpointer=MemorySaver())
```

### Pattern 2: Multi-Step Agent with Planning

For complex tasks requiring decomposition:

```python
class PlanExecuteState(TypedDict):
    messages: Annotated[list, add_messages]
    plan: list[str]
    current_step: int
    results: list[str]
    final_answer: str

def planner(state: PlanExecuteState):
    """Generate a structured plan."""
    plan = planning_llm.invoke(
        f"Decompose the following task into atomic steps: "
        f"{state['messages'][-1].content}"
    )
    steps = parse_plan(plan.content)
    return {"plan": steps, "current_step": 0, "results": []}

def executor(state: PlanExecuteState):
    """Execute the current step of the plan."""
    step = state["plan"][state["current_step"]]
    # The context includes results of previous steps
    context = "\n".join(state["results"])
    result = execution_llm.invoke(
        f"Previous context:\n{context}\n\n"
        f"Execute this step: {step}"
    )
    return {
        "results": [result.content],
        "current_step": state["current_step"] + 1
    }

def synthesizer(state: PlanExecuteState):
    """Synthesize results into a final answer."""
    all_results = "\n".join(state["results"])
    answer = synthesis_llm.invoke(
        f"Synthesize the following results:\n{all_results}"
    )
    return {"final_answer": answer.content}

def route_execution(state: PlanExecuteState):
    if state["current_step"] < len(state["plan"]):
        return "execute"
    return "synthesize"

graph = StateGraph(PlanExecuteState)
graph.add_node("plan", planner)
graph.add_node("execute", executor)
graph.add_node("synthesize", synthesizer)

graph.add_edge(START, "plan")
graph.add_edge("plan", "execute")
graph.add_conditional_edges("execute", route_execution, {
    "execute": "execute",
    "synthesize": "synthesize"
})
graph.add_edge("synthesize", END)
```

### Pattern 3: Multi-Agent Supervisor

To orchestrate specialized agents:

```python
class SupervisorState(TypedDict):
    messages: Annotated[list, add_messages]
    next_agent: str
    task_complete: bool

def supervisor(state: SupervisorState):
    """Decide which specialized agent to invoke."""
    decision = supervisor_llm.invoke(
        f"Given the messages: {state['messages']}, "
        f"which agent should handle the next step? "
        f"Options: researcher, coder, reviewer, FINISH"
    )
    return {"next_agent": decision.content.strip()}

def researcher(state: SupervisorState):
    """Sub-agent specialized in research."""
    result = research_agent.invoke(state["messages"])
    return {"messages": [AIMessage(content=result, name="researcher")]}

def coder(state: SupervisorState):
    """Sub-agent specialized in code writing."""
    result = coding_agent.invoke(state["messages"])
    return {"messages": [AIMessage(content=result, name="coder")]}

def reviewer(state: SupervisorState):
    """Sub-agent specialized in review."""
    result = review_agent.invoke(state["messages"])
    return {"messages": [AIMessage(content=result, name="reviewer")]}

def route_to_agent(state: SupervisorState):
    if state["next_agent"] == "FINISH":
        return "end"
    return state["next_agent"]

graph = StateGraph(SupervisorState)
graph.add_node("supervisor", supervisor)
graph.add_node("researcher", researcher)
graph.add_node("coder", coder)
graph.add_node("reviewer", reviewer)

graph.add_edge(START, "supervisor")
graph.add_conditional_edges("supervisor", route_to_agent, {
    "researcher": "researcher",
    "coder": "coder",
    "reviewer": "reviewer",
    "end": END
})
# Each agent returns to the supervisor
for agent_name in ["researcher", "coder", "reviewer"]:
    graph.add_edge(agent_name, "supervisor")
```

### Pattern 4: Human-in-the-Loop with Checkpointing

For workflows requiring human approval:

```python
from langgraph.checkpoint.memory import MemorySaver

def sensitive_action(state):
    """Action that requires human approval."""
    action = determine_action(state)
    return {"pending_action": action}

# Compile with interrupt
graph = StateGraph(WorkflowState)
# ... add nodes and edges ...
app = graph.compile(
    checkpointer=MemorySaver(),
    interrupt_before=["sensitive_action"]  # Pause before this node
)

# Execution
config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke(initial_state, config)

# Execution stops before sensitive_action
# The human can inspect the state and approve or modify:
state = app.get_state(config)
# After approval, resume:
result = app.invoke(None, config)  # Resumes from checkpoint
```

### Production Considerations

For production deployment, Chase and the LangChain team recommend:

1. **Replace `MemorySaver` with a persistent backend** (Postgres, DynamoDB) to ensure state durability.
2. **Implement compaction strategies** for long-running agents: summarize messages while keeping the full history on the file system.
3. **Use LangSmith** for end-to-end tracing: every trace becomes a debugging and testing artifact.
4. **Define timeouts and limits** to prevent infinite executions.
5. **Structure the system prompt** as an architectural document: instructions must be exhaustive, not minimal.

---

## 7. Resources and References

### Primary Sources

- **Sequoia Capital Podcast** -- "Context Engineering Our Way to Long-Horizon Agents": [sequoiacap.com/podcast/context-engineering-our-way-to-long-horizon-agents-langchains-harrison-chase](https://sequoiacap.com/podcast/context-engineering-our-way-to-long-horizon-agents-langchains-harrison-chase/)
- **LangChain Blog** -- "The Rise of Context Engineering": [blog.langchain.com/the-rise-of-context-engineering](https://blog.langchain.com/the-rise-of-context-engineering/)
- **Harrison Chase on X/Twitter**: [@hwchase17](https://x.com/hwchase17)
- **Harrison Chase on LinkedIn**: [linkedin.com/in/harrison-chase-961287118](https://www.linkedin.com/in/harrison-chase-961287118/)

### Technical Documentation

- **LangGraph Docs**: [docs.langchain.com/oss/python/langgraph](https://docs.langchain.com/oss/python/langgraph/overview)
- **LangChain Overview**: [python.langchain.com](https://python.langchain.com/docs/langserve)
- **LangSmith**: [langchain.com](https://www.langchain.com)

### Articles and Analysis

- "Harrison Chase on Deep Agents: The Next Evolution in Autonomous AI" -- [OpenDataScience](https://opendatascience.com/harrison-chase-on-deep-agents-the-next-evolution-in-autonomous-ai/)
- "LangChain's CEO argues that better models alone won't get your AI agent to production" -- [VentureBeat](https://venturebeat.com/orchestration/langchains-ceo-argues-that-better-models-alone-wont-get-your-ai-agent-to)
- "LangGraph Architecture and Design" -- [Medium](https://medium.com/@shuv.sdr/langgraph-architecture-and-design-280c365aaf2c)
- "LangGraph Multi-Agent Orchestration: Complete Framework Guide" -- [Latenode](https://latenode.com/blog/ai-frameworks-technical-infrastructure/langgraph-multi-agent-orchestration/langgraph-multi-agent-orchestration-complete-framework-guide-architecture-analysis-2025)
- "Persistence in LangGraph: Building AI Agents with Memory and Fault Tolerance" -- [Medium](https://medium.com/@iambeingferoz/persistence-in-langgraph-building-ai-agents-with-memory-fault-tolerance-and-human-in-the-loop-d07977980931)
- "Founder Story: Harrison Chase of LangChain" -- [Frederick AI](https://www.frederick.ai/blog/harrison-chase-langchain)
- "LangChain vs LangGraph vs LangSmith: Understanding the Ecosystem" -- [DEV Community](https://dev.to/rajkundalia/langchain-vs-langgraph-vs-langsmith-understanding-the-ecosystem-3m5o)
- "Context Engineering Guide" -- [Prompting Guide](https://www.promptingguide.ai/guides/context-engineering-guide)
- "The New Skill in AI is Not Prompting, It's Context Engineering" -- [Philipp Schmid](https://www.philschmid.de/context-engineering)

### Company Profile

- **LangChain on Crunchbase**: [crunchbase.com/person/harrison-chase](https://www.crunchbase.com/person/harrison-chase)
- **LangChain on Wikipedia**: [en.wikipedia.org/wiki/LangChain](https://en.wikipedia.org/wiki/LangChain)
- **BigDataWire People to Watch 2024**: [bigdatawire.com/people-to-watch-2024-harrison-chase](https://www.bigdatawire.com/people-to-watch-2024-harrison-chase/)

---

> *"Everything's context engineering."* -- Harrison Chase

> *"When agents fail, they fail because they don't have the right context; when they succeed, they succeed because they have the right context."* -- Harrison Chase, Sequoia Capital Podcast, 2025
