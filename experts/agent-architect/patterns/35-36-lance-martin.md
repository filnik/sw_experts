# Pattern 35-36: Agent Design Patterns and Context Engineering

## Source: Lance Martin (LangChain) - Blog Post Series 2025-2026

---

## 1. Profile: Lance Martin

Lance Martin is a software engineer at **LangChain**, the leading company in the field of frameworks for applications based on Large Language Models. With a doctorate (PhD) in applied machine learning from Stanford, Martin holds a leading technical role in defining architectures for AI agents within the LangChain/LangGraph ecosystem.

### Role and Contributions to LangChain

Martin is not simply a software engineer: he is one of the most authoritative voices in defining **architectural patterns** for agentic systems. His main contributions include:

- **RAG From Scratch**: a series of short videos (5-10 minutes each) and Jupyter notebooks that explain over a dozen academic papers on Retrieval-Augmented Generation. Martin personally implemented every technique with fully open source code, organized by pattern (Adaptive-RAG, Self-RAG, Corrective-RAG). This project has become a fundamental reference for anyone working with RAG systems.

- **Deep Agents (deepagents)**: Martin is among the contributors of the `langchain-ai/deepagents` project, an agent harness built with LangChain and LangGraph. This system is equipped with planning tools, filesystem backend, and the ability to spawn sub-agents, making it suitable for complex agentic tasks.

- **LangChain Blog**: he has published technical articles on the official LangChain blog, including "Using Skills with Deep Agents" (November 2025), where he explores the integration of Anthropic Skills in the deep agents framework.

- **Talk at Interrupt Conference 2025**: at LangChain's annual conference in San Francisco (May 2025), his session on how to build agentic applications opened the proceedings, addressing the critical theme of the transition from prototypes to production systems.

- **Latent Space Podcast**: in September 2025 he appeared as a guest on the podcast "Latent Space: The AI Engineer Podcast", in an episode dedicated entirely to context engineering for agents, where he delved into the distinctions between prompt engineering and context engineering and the challenges of context management in tool call flows.

Martin represents a figure linking academic research and practical implementation: his writings do not limit themselves to describing theoretical patterns but systematically tie them to concrete implementations in LangGraph, Claude Code, Manus, and other production systems.

---

## 2. Agent Design Patterns (January 2026)

The blog post "Effective Agent Design" published on January 9, 2026 represents one of the most complete and structured analyses of design patterns for AI agents ever published. The central thesis is radical in its simplicity: **effective agent design is fundamentally a problem of context management**.

### 2.1 The Fundamental Thesis: Context as a Finite Resource

Martin opens the post with a crucial empirical observation: as language models process longer contexts, their performance degrades. This phenomenon, which implicitly defines an "attention budget" per token, implies that context must be treated as a **finite resource with diminishing marginal returns**.

The operational definition he proposes is: context engineering consists of "filling the context window with exactly the right information for the next step". This formulation, originally attributed to Andrej Karpathy, becomes the organizing principle of the entire taxonomy.

### 2.2 The Seven Fundamental Patterns

Martin identifies seven distinct patterns, organized in a taxonomy that proceeds from the system level up to the level of continuous learning.

#### Pattern 1: Give Agents a Computer

The first pattern establishes that LLMs should direct actions through direct access to the operating system, including filesystem and shell. The key quote is: "It is more accurate to think of Claude Code as 'AI for your operating system'".

**Reference implementations:**
- **Claude Code** "lives on your computer" and interacts directly with the operating system
- **Manus** (acquired by Meta for over $2 billion) uses a complete virtual computer

The fundamental benefit is twofold: **context persistence** through the filesystem, and **extensibility** through the ability to execute utilities, CLIs, and custom scripts. This pattern transforms the agent from an isolated system to a system integrated with the computational environment.

#### Pattern 2: Multi-Layer Action Space

This pattern addresses a concrete problem: loading all tool definitions directly into context causes token overflow and model confusion. The solution consists of organizing actions **hierarchically**, using a few atomic tools (bash, file access) instead of dozens of specialized tools.

**Quantitative data reported by Martin:**
- Claude Code uses about 12 tools
- Manus uses fewer than 20 tools
- The GitHub MCP server exposes 35 tools with about 26,000 tokens of overhead

The operational mechanism is the **CodeAct** pattern: agents write and execute code rather than calling specialized tools, avoiding token consumption from intermediate results. This is a fundamental insight because it overturns the traditional approach of providing granular tools: fewer tools, more generic, lead to better performance.

#### Pattern 3: Progressive Disclosure

This pattern prescribes showing only essential information initially and revealing details on demand. Concrete implementations include:

- **Indexing tool definitions** with on-demand retrieval
- **Help flags** (e.g., `--help`) instead of full documentation in context
- **MCP descriptions** stored in folders that agents access only when necessary
- **Anthropic Skills standard** that uses SKILL.md files with YAML frontmatter to define reusable procedures

This pattern recognizes that informational completeness is the enemy of effectiveness: an agent that knows everything simultaneously is less effective than one that knows where to find information when needed.

#### Pattern 4: Offload Context

The fourth pattern proposes writing old tool results and trajectories to the filesystem, summarizing only when diminishing returns justify it. Martin directly addresses the concern that summarization loses information: the solution is not to summarize aggressively, but to **externalize** context to persistent storage.

A particularly sophisticated application consists of preserving plans and progress files that agents read periodically to reinforce goals. This creates a **self-orientation** mechanism where long-term context survives the LLM's context window.

#### Pattern 5: Cache Context

Martin identifies the **cache hit rate** as "the most important metric for agents in production". This is an operational insight of enormous practical relevance. Prompt caching enables the reduction of costs and latency by reusing already computed prompt prefixes.

The key economic observation is counterintuitive: higher-capability models with caching can be **cheaper** than lower-cost models without caching. Without this technology, coding agents like Claude Code would be "prohibitive in terms of cost".

#### Pattern 6: Isolate Context

This pattern prescribes the delegation of tasks to sub-agents with separate context windows, tools, and instructions. Use cases include:

- **Parallelizable tasks**: code reviews that check different problems simultaneously
- **Map-reduce patterns**: lint rule updates, code migrations

Martin introduces the concept of the **"Ralph Wiggum Loop"**: long-running agents that loop repeatedly until plans meet completion criteria. The context lives in files, progress is communicated via git history. The structure provides for an initializer agent that prepares the environment and sub-agents that tackle individual tasks of the plan.

A concrete example is **Gas Town**, a multi-agent orchestrator that uses git-based work tracking and a "Mayor agent" to coordinate concurrent instances of Claude Code.

#### Pattern 7: Evolve Context

The final pattern concerns continuous learning through the updating of the agent's context (not the model's weights) based on experience. The methodologies include:

- **Task-specific prompts**: GEPA collects trajectories, evaluates them, reflects on failures, and proposes prompt variants
- **Memory learning**: distillation of sessions into journal entries, reflection between entries, updating documentation
- **Skill learning**: extraction of reusable procedures from trajectories and saving as new skills

Martin also cites emerging research directions such as **RLM (Recursive Language Model)**, which suggests that models can learn their own context management strategies, and **Sleep-Time Compute**, where agents reflect offline on context with potential for automatic memory consolidation.

### 2.3 Proposed Taxonomy

Martin's taxonomy organizes the seven patterns along three dimensions:

| Pattern | Level | Mechanism | Benefit |
|---------|---------|------------|-----------|
| Computer Access | System | Filesystem + Shell | Persistence + Extensibility |
| Multi-Layer Actions | Tool | Hierarchy (atomic tools) | Token efficiency |
| Progressive Disclosure | Information | On-demand retrieval | Context preservation |
| Offload Context | Storage | Writing to filesystem | Information preservation |
| Cache Context | Efficiency | Prompt caching | Cost reduction |
| Isolate Context | Parallelization | Sub-agents | Scalability |
| Evolve Context | Learning | Trajectory reflection | Adaptation over time |

### 2.4 Evolution of Patterns Over Time

Martin documents a significant evolution: the length of agent tasks doubles every 7 months according to METR evaluations. This data implies that the patterns are not static but must co-evolve with the growing capabilities of the models. The transition from Claude Code reaching $1 billion in run rate and the acquisition of Manus for over $2 billion attest that these patterns are not academic exercises but the foundations of commercial products of enormous value.

The article also identifies critical infrastructure gaps still unresolved: standards for agent observability, common debugging interfaces, patterns for human-in-the-loop monitoring, and frameworks for graceful degradation.

---

## 3. Context Engineering for Agents (June 2025)

The post "Context Engineering for Agents" of June 23, 2025 chronologically precedes the agent design post and constitutes its theoretical foundation. While the 2026 post focuses on design patterns, this one focuses specifically on **context management strategies**.

### 3.1 Definition of Context Engineering

Martin adopts and amplifies Andrej Karpathy's definition: context engineering is **"the delicate art and science of filling the context window with exactly the right information for the next step"**. The proposed analogy is with RAM management in an operating system: the context window of an LLM functions as working memory with intrinsic capacity limitations.

Martin cites Cognition (the company behind Devin) which states that context engineering is "effectively the number one job of engineers building AI agents". This is not a marginal opinion but a position shared by the leading companies in the sector.

### 3.2 Three Types of Context

Before delving into the strategies, Martin classifies context into three categories:

1. **Instructions**: prompts, memories, few-shot examples, tool descriptions
2. **Knowledge**: facts and contextual information
3. **Tools**: feedback derived from tool execution

This tripartition is fundamental because each type of context requires different management strategies.

### 3.3 The Four Primary Strategies

#### Strategy 1: Write Context

Consists of saving information outside the context window for later retrieval.

**Scratchpads:**
Agents preserve planning and task-related information persistently. Implementation methods include tool calls, file writes, or runtime state objects. The example cited is Anthropic's multi-agent researcher, which saves plans in memory to prevent truncation beyond 200,000 tokens.

**Memories:**
The Reflexion paper introduced self-generated memories based on reflection between sessions. Generative Agents synthesize memories from accumulated feedback. Products that implement this strategy include ChatGPT, Cursor, and Windsurf. Martin identifies three types of memory:
- **Episodic**: concrete examples of past experiences
- **Procedural**: instructions on how to perform operations
- **Semantic**: declarative facts and knowledge

#### Strategy 2: Select Context

Consists of retrieving relevant information into the context window.

**Scratchpad Selection:**
Two complementary approaches: tool-based access via agent calls, and developer-controlled state exposure at every turn.

**Memory Selection Challenges:**
Approaches based on single files (CLAUDE.md, rules files) work for limited datasets. Larger collections require embeddings or knowledge graphs. Martin cites the risk highlighted by Simon Willison: ChatGPT injecting geographic location information from memories into contexts where they were not required.

**Tool Selection:**
Applying RAG to tool descriptions improves selection accuracy by a factor of three. This addresses the problem of overlapping descriptions that confuse the model.

**Knowledge Retrieval (RAG):**
Code agents represent RAG at production scale. Windsurf's approach combines: parsing of the Abstract Syntax Tree (AST), semantic chunking, embedding search, grep/file search, knowledge graph, and re-ranking.

#### Strategy 3: Compress Context

Consists of keeping only the essential tokens.

**Context Summarization:**
Claude Code implements "auto-compact" at 95% of context window usage. Strategies include recursive and hierarchical summarization. Applications include post-processing of tool outputs and reduction in agent-to-agent handoffs. Martin notes the challenge of capturing critical decisions: Cognition uses fine-tuned models specifically for summarization.

**Context Trimming:**
Two approaches: hard-coded heuristics (removing the oldest messages) and trained pruners like Provence for question-answering tasks.

#### Strategy 4: Isolate Context

Consists of splitting context across sub-systems.

**Multi-Agent Architecture:**
OpenAI Swarm implements "separation of concerns". Each agent has: isolated context window, specific tools, dedicated instructions. Anthropic's multi-agent researcher uses sub-agents that operate in parallel exploring different aspects. Martin notes the trade-off: up to **15 times more tokens** compared to a single chat agent.

**Environment-Based Isolation:**
HuggingFace's CodeAgent produces code executed in a sandbox. Return values are selectively passed to the LLM. The advantage is that token-heavy objects (images, audio) are kept as variables without occupying context.

**Runtime State Objects:**
Pydantic schemas isolate information in specific fields. Exposure to the LLM is selective while broader state is maintained.

### 3.4 Context Failure Modes

Martin takes up Drew Breunig's taxonomy of context failures:

- **Context Poisoning**: hallucinations that persist in context and propagate
- **Context Distraction**: excessive context that overwhelms reasoning
- **Context Confusion**: superfluous information that unduly influences responses
- **Context Clash**: pieces of context in conflict with each other

These failures are not bugs but **emergent properties** of inadequate context management, and the four strategies described above are precisely the countermeasures.

### 3.5 Relationship with Prompt Engineering

Martin explicitly distinguishes context engineering from prompt engineering. Prompt engineering optimizes the **clarity of instructions**: how to formulate a request so the model understands it best. Context engineering manages **what information appears in the window at each step**, emphasizing dynamic and trajectory-aware curation rather than static prompt design.

In practical terms: prompt engineering is relevant for the single turn, context engineering is relevant for the entire agent session. With agents handling hundreds of turns, the second becomes enormously more critical than the first.

---

## 4. Key Insights

From the cross-analysis of the two posts emerge fundamental lessons that deserve to be made explicit.

### 4.1 Context is the Bottleneck, Not the Model

Martin's deepest insight is that the limit of current agents does not lie in the model's capabilities but in context management. More powerful models with poorly managed context perform worse than less powerful models with well-managed context. This overturns the dominant narrative that focuses on the model benchmark and brings attention to system engineering.

### 4.2 Fewer Tools, More Power

The discovery that successful agents like Claude Code (12 tools) and Manus (<20 tools) use significantly fewer tools than the GitHub MCP server (35 tools, 26K tokens) suggests a counterintuitive principle: tool proliferation is counterproductive. The CodeAct pattern -- writing and executing code rather than calling specialized tools -- is a manifestation of this principle.

### 4.3 Cache Hit Rate as a Primary Metric

The identification of the cache hit rate as the most important metric for agents in production is an extremely practical operational insight. It shifts attention from model optimization to caching infrastructure optimization, with direct implications on system architecture.

### 4.4 The Cost of Multi-Agent

The data that multi-agent architectures can consume up to 15 times more tokens compared to single agents is a wake-up call for those adopting these architectures without cost analysis. Decomposition into sub-agents has real benefits (context isolation, parallelism), but the economic trade-off must be explicitly evaluated.

### 4.5 Memory as a Triad

The classification of memory into episodic, procedural, and semantic provides an operational framework for the design of memory systems for agents. Each type requires different storage, retrieval, and use strategies, and confusing them leads to ineffective systems.

### 4.6 Context Evolution as a Frontier

The "Evolve Context" pattern represents the most advanced frontier: agents that improve not through fine-tuning of weights but through the evolution of their operational context. This approach is more accessible (does not require model training), more interpretable (changes to context are readable), and more reversible (one can always go back to previous versions of the context).

---

## 5. Practical Application with LangGraph

### 5.1 Implementing the "Write Context" Pattern with LangGraph State

LangGraph manages state through typed objects. To implement the Write Context pattern, you define a state that includes a scratchpad field:

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph

class AgentState(TypedDict):
    messages: list
    scratchpad: str  # Persistent context
    plan: str        # Current plan
    memories: list   # Long-term memories

def write_to_scratchpad(state: AgentState) -> AgentState:
    """Writes critical information to the scratchpad"""
    last_result = state["messages"][-1].content
    updated_pad = state["scratchpad"] + f"\n[Result]: {last_result}"
    return {"scratchpad": updated_pad}
```

### 5.2 Implementing Progressive Disclosure with Tool Routing

In LangGraph, progressive disclosure is implemented by creating a router node that dynamically selects which tools to expose:

```python
from langgraph.graph import StateGraph, END

def tool_router(state: AgentState) -> str:
    """Determines which group of tools to make available"""
    current_task = analyze_task(state["messages"][-1])
    if current_task == "code_review":
        return "code_tools"
    elif current_task == "documentation":
        return "doc_tools"
    else:
        return "general_tools"

graph = StateGraph(AgentState)
graph.add_node("router", tool_router)
graph.add_node("code_tools", code_tool_executor)
graph.add_node("doc_tools", doc_tool_executor)
graph.add_node("general_tools", general_tool_executor)
graph.add_conditional_edges("router", tool_router,
    {"code_tools": "code_tools",
     "doc_tools": "doc_tools",
     "general_tools": "general_tools"})
```

### 5.3 Implementing Isolate Context with Sub-Graphs

LangGraph natively supports sub-graphs for context isolation:

```python
# Sub-agent with isolated context
sub_agent = StateGraph(SubAgentState)
sub_agent.add_node("analyze", analyze_node)
sub_agent.add_node("report", report_node)
sub_agent.add_edge("analyze", "report")
sub_graph = sub_agent.compile()

# Main agent that delegates
def delegate_to_subagent(state: AgentState) -> AgentState:
    """Delegates a task to a sub-agent with isolated context"""
    sub_state = SubAgentState(
        task=state["current_subtask"],
        context=extract_relevant_context(state)
    )
    result = sub_graph.invoke(sub_state)
    return {"messages": [AIMessage(content=result["report"])]}
```

### 5.4 Implementing Context Compression

For the context compression pattern, you can insert a compaction node into the graph:

```python
def compress_context(state: AgentState) -> AgentState:
    """Compresses context when it exceeds a threshold"""
    messages = state["messages"]
    if count_tokens(messages) > THRESHOLD:
        # Save full context to file
        save_to_filesystem(messages, "context_backup.json")
        # Generate a summary
        summary = summarize_messages(messages)
        # Keep only the summary and the last N messages
        compressed = [SystemMessage(content=summary)] + messages[-5:]
        return {"messages": compressed}
    return state

graph.add_node("compress", compress_context)
```

### 5.5 Implementing Evolve Context with LangGraph Memory

LangGraph offers an integrated memory system that supports the Evolve Context pattern:

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store.memory import InMemoryStore

# Store for long-term memories
memory_store = InMemoryStore()

def reflect_on_session(state: AgentState) -> AgentState:
    """Reflects on the current session and updates memories"""
    trajectory = state["messages"]
    # Analyze successes and failures
    reflection = generate_reflection(trajectory)
    # Save as memory for future sessions
    memory_store.put(
        namespace=("agent", "reflections"),
        key=generate_key(),
        value={"reflection": reflection, "session_id": state["session_id"]}
    )
    return state
```

### 5.6 Ralph Wiggum Loop Pattern in LangGraph

The most sophisticated pattern described by Martin, the Ralph Wiggum Loop, is implemented with a conditional cycle:

```python
def check_completion(state: AgentState) -> str:
    """Checks if the plan has been completed"""
    plan = read_from_filesystem("plan.md")
    progress = read_from_filesystem("progress.md")
    if all_tasks_complete(plan, progress):
        return "complete"
    return "continue"

graph.add_conditional_edges(
    "execute_task",
    check_completion,
    {"complete": END, "continue": "plan_next_task"}
)
```

---

## 6. Resources and References

### 6.1 Lance Martin's Main Blog Posts

- **"Effective Agent Design"** (January 9, 2026): https://rlancemartin.github.io/2026/01/09/agent_design/ -- The complete post on the seven design patterns for agents, with taxonomy, examples, and analysis of production implementations.

- **"Context Engineering for Agents"** (June 23, 2025): https://rlancemartin.github.io/2025/06/23/context_engineering/ -- The foundational post on the four context engineering strategies (Write, Select, Compress, Isolate) with references to production systems.

- **"Using Skills with Deep Agents"** (November 2025): published on the official LangChain blog (https://blog.langchain.com/author/lance/), explores the integration of Anthropic Skills in deep agent frameworks.

### 6.2 Open Source Projects

- **RAG From Scratch**: GitHub repository with notebooks and videos that implement over a dozen papers on RAG. Repository: https://github.com/rlancemartin
- **Deep Agents**: agent harness built with LangChain and LangGraph. Repository: https://github.com/langchain-ai/deepagents
- **LangGraph**: framework for agent orchestration. Documentation: https://docs.langchain.com/oss/python/langgraph/

### 6.3 Talks and Podcasts

- **LangChain Interrupt 2025** (May 2025, San Francisco): inaugural session on the transition from prototypes to agentic applications in production.
- **Latent Space Podcast** (September 2025): episode "Context Engineering for Agents - Lance Martin, LangChain", available on Apple Podcasts and Spotify.
- **Neo4j GenAI Talk**: presentation on GenAI solutions with LangChain covering LLMs, agents, evaluations, and more (https://neo4j.com/videos/genai-solutions-with-langchain-lance-martin-on-llms-agents-evals-and-more/).

### 6.4 Referenced Papers and Concepts

- **Reflexion**: paper on self-generated memories based on reflection between sessions
- **Generative Agents** (Stanford): simulation of agents with memory synthesis
- **GEPA**: system for trajectory collection, evaluation, and proposal of prompt variants
- **CodeAct**: pattern where agents write and execute code rather than calling specialized tools
- **RLM (Recursive Language Model)**: research on models that learn their own context management strategies
- **Sleep-Time Compute**: agents that reflect offline on context for memory consolidation
- **Provence**: trained pruner for context trimming in question-answering tasks

### 6.5 Tools and Frameworks Mentioned

- **Model Context Protocol (MCP)**: standard for providing tools to agents
- **Claude Code**: Anthropic's commercial agent for coding
- **Manus**: agent based on virtual computer (acquired by Meta)
- **Cursor Agent**: dynamic context discovery with MCP management
- **Gas Town**: multi-agent orchestrator with git-based tracking
- **Anthropic Skills Standard** (agentskills.io): framework for reusable procedures
- **Letta**: platform for continuous learning with token-space updates
- **OpenAI Swarm**: multi-agent framework with separation of concerns
- **Windsurf**: editor with multi-technique retrieval system (AST, embedding, knowledge graph)

### 6.6 Profiles and Contacts

- **Personal blog**: https://rlancemartin.github.io/
- **GitHub**: https://github.com/rlancemartin
- **Twitter/X**: https://x.com/RLanceMartin
- **LangChain Blog**: https://blog.langchain.com/author/lance/
