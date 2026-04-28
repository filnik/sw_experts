# OpenAI Cookbook - Agents Section: In-Depth Technical Guide

---

## 1. Overview - The Cookbook and the Agents Section

### What the Cookbook Is

The OpenAI Cookbook (now available at `developers.openai.com/cookbook`) is the official collection of interactive notebooks, practical guides, and implementation patterns maintained directly by OpenAI. It is not theoretical documentation: every entry is an executable Jupyter notebook that demonstrates a specific concept with working code, real outputs, and contextual explanations. The Cookbook represents the bridge between API documentation and production-ready implementation, providing battle-tested patterns that developers can adapt to their own use cases.

### Structure of the Agents Section

The section dedicated to agents (`/cookbook/topic/agents`) collects the most relevant notebooks for those building agentic systems with the OpenAI APIs and the Agents SDK. The section is organized around several main themes:

- **Orchestration Patterns**: The foundational notebook "Orchestrating Agents: Routines and Handoffs" that defines the primitive concepts of routines and handoffs.
- **Deep Research**: Multi-agent pipelines for deep research with the Deep Research API.
- **Context Engineering**: Short- and long-term memory management, personalization, trimming, and context compression.
- **Multi-Agent Collaboration**: Patterns for collaboration between specialized agents, including agent-as-a-tool and parallel execution.
- **Evaluation & Observability**: Metrics, tracing, and agent evaluation with tools such as Langfuse.
- **Domain-Specific Agents**: Vertical implementations such as dispute management with Stripe, coding agents, voice assistants.

Among the most important notebooks we find:

| Notebook | Main Focus |
|----------|-----------------|
| Orchestrating Agents: Routines and Handoffs | Fundamental orchestration patterns |
| Deep Research API with the Agents SDK | Multi-agent research pipeline |
| Context Engineering for Personalization | Long-term memory and RunContextWrapper |
| Context Engineering - Session Memory | Trimming and context compression |
| Multi-Agent Portfolio Collaboration | Agent-as-a-tool pattern |
| Parallel Agents | Concurrent execution with asyncio |
| Structured Outputs for Multi-Agent Systems | Schema enforcement for agents |
| Evaluating Agents with Langfuse | Observability and metrics |
| Automating Dispute Management | Domain-specific agents with Stripe |

---

## 2. Orchestration Patterns - The Foundational Notebook

### Routines and Handoffs: the Primitives of Orchestration

The notebook "Orchestrating Agents: Routines and Handoffs" is the foundational text for anyone building multi-agent systems with OpenAI. It introduces two primitive concepts that, in their simplicity, solve the orchestration problem in an elegant and controllable way.

**Routine**: A routine is defined as a list of natural-language instructions (represented with a system prompt) accompanied by the tools necessary to complete them. The power of routines lies in their robustness: the language model naturally navigates conditional logic similar to a state machine, without getting stuck in dead ends. Unlike a rigid coded workflow, the routine leverages the LLM's ability to interpret nuanced instructions and make contextual decisions.

**Handoff**: A handoff occurs when an agent transfers an active conversation to another agent. The critical aspect is that the receiving agent has full access to all the previous conversation history. The parallel is a phone transfer, but with perfect preservation of context.

### Implementation Architecture

The notebook defines the base `Agent` class using Pydantic:

```python
class Agent(BaseModel):
    name: str = "Agent"
    model: str = "gpt-4o-mini"
    instructions: str = "You are a helpful Agent"
    tools: list = []
```

The heart of the system is the **execution loop** implemented in the function `run_full_turn`, which handles:

1. Conversion of Python tools to OpenAI schema (automatic mapping of Python type hints: `str` becomes `"string"`, `int` becomes `"integer"`, etc.)
2. Call to the model with the system instructions of the active agent
3. Execution of the returned tool calls
4. Verification of whether the result is an `Agent` object (which triggers a handoff)
5. Reinsertion of the results into the conversation

```python
while True:
    response = client.chat.completions.create(
        model=agent.model,
        messages=[{"role": "system", "content": current_agent.instructions}] + messages,
        tools=tool_schemas or None,
    )
```

The `Response` class manages the return of state:

```python
class Response(BaseModel):
    agent: Optional[Agent]
    messages: list
```

### The Handoff Mechanism

Handoffs work via transfer functions that return `Agent` objects:

```python
def transfer_to_sales_agent():
    """User for anything sales or buying related."""
    return sales_agent
```

When a tool call returns an `Agent` instead of a string, the system:

1. Updates the active agent
2. Sends a notification: `"Transfered to {agent_name}. Adopt persona immediately."`
3. Continues the conversation with the new agent's context

This elegant design allows the LLM to autonomously decide when to transfer, simply by calling the appropriate functions. No explicit handoff logic is needed: the framework manages context switching automatically.

### Swarm: the Reference Repository

OpenAI provides example code in the Swarm repository (`github.com/openai/swarm`) as a proof-of-concept. The notebook emphasizes: "It is intended only as an example and should not be used directly in production." However, the concepts and patterns are production-ready and constitute the conceptual basis of the official Agents SDK released subsequently.

---

## 3. Deep Research - Structured Research Agents

### Architecture of the Four-Agent Pipeline

The notebook "Deep Research API with the Agents SDK" demonstrates a hierarchical four-agent pipeline for conducting sophisticated research workflows. This is the most advanced pattern in the Cookbook for tasks requiring planning, synthesis, tool use, and multi-step reasoning.

**1. Triage Agent**
The triage agent routes queries intelligently. It evaluates whether the user's question requires clarification or can proceed directly to research. The decision logic is: if context is missing, clarify; otherwise, build the instructions.

**2. Clarifying Agent**
Engages the user with targeted follow-up questions when the initial query lacks specificity. The instructions prescribe: "Ask 2-3 clarifying questions to gather more context", maintaining brevity and friendliness. It returns structured output using the Pydantic model `Clarifications`:

```python
class Clarifications(BaseModel):
    questions: List[str]
```

**3. Research Instruction Agent**
Transforms the user's enriched input into detailed research briefs. The agent applies systematic guidelines:
- Maximize specificity with all user preferences
- Treat unspecified dimensions as open-ended
- Avoid unjustified assumptions
- Use first person in formulation
- Require structured output (tables) when useful
- Explicitly specify expected formatting and language

**4. Research Agent**
The terminal agent that performs empirical research using dedicated models such as `o3-deep-research` (full version) or `o4-mini-deep-research-alpha` (faster alternative). The choice depends on the latency vs. capability tradeoff.

### Tool Integration

**Web Search Tool**: Native integration that provides access to the public network. The research agent automatically executes queries based on research questions, with streaming visibility on search operations.

**MCP (Model Context Protocol) Tool**: The `HostedMCPTool` configuration enables search in files and internal knowledge bases. Typical configuration:
- `server_label`: "file_search"
- `server_url`: custom endpoint
- `require_approval`: "never"

### Streaming and Citations

The `Runner.run_streamed()` method returns a stream object that supports asynchronous iteration over events. Event types include `agent_updated_stream_event` (agent change notifications), `raw_response_event` (tool calls and search queries), and tool-specific events.

The notebook provides utilities for extracting URL citations from the final output, including URL, title, position in text, and surrounding context.

### When to Use Deep Research

The notebook's warning is clear: "Don't use Deep Research for trivial fact lookups." It is reserved for tasks requiring planning, synthesis, tool use, and multi-step reasoning. For simple questions, the standard `openai.responses` API is faster and cheaper.

---

## 4. Context Engineering - Specific Notebooks and Techniques

### Personalization with RunContextWrapper

The notebook "Context Engineering for Personalization" represents one of the most sophisticated contributions of the Cookbook. The fundamental concept: "Modern AI agents are no longer simple reactive assistants -- they are becoming adaptive collaborators. The leap from 'responding' to 'remembering' defines the new frontier of context engineering."

The pattern uses the Agents SDK's `RunContextWrapper` to maintain persistent state across agent executions, enabling long-term memory and personalized behavior.

#### State Object Architecture

The foundation is a `TravelState` dataclass (demonstrated with a travel concierge agent) containing:

- **Profile (Structured)**: Identity (customer ID, name, age, passport expiration), loyalty (airline/hotel status, membership ID), preferences (seat type, communication tone, active visas), insurance profile.
- **Global Memory (Durable)**: Long-term notes persistent across sessions. Each note includes text, last update date (ISO format), and keywords (1-3 tags). Examples: "Usually prefers aisle seat", "Avoids checking luggage for short trips".
- **Session Memory (Ephemeral)**: Staging area for newly captured candidate memories. Lives only during the current interaction. Promoted to global memory during consolidation if durable.
- **Trip History**: Recent trips from the database that show patterns in user choices.

#### The Memory Lifecycle in Five Phases

**Phase 1 - Injection at Session Start**: The frontmatter renders as a YAML block containing the profile fields. Global memories render as an ordered Markdown list (top 6 notes by recency). This creates explicit `<user_profile>` and `<memories>` blocks in the system instructions.

**Phase 2 - Context Trimming**: The `TrimmingSession` implementation keeps only the last N user turns (default 8-20). When trimming occurs, it sets the `inject_session_memories_next_turn = True` flag to preserve short-term context.

**Phase 3 - Distillation During Execution**: The `save_memory_note()` tool captures explicit user preferences in real time. It accepts text (1-2 sentences) and keywords (1-3 tags). The instructions explicitly forbid storage of speculations, instructions, or sensitive PII.

**Phase 4 - Reinjection After Trimming**: If trimming has occurred, the session notes are rendered and injected into the system prompt at the next turn, reactivating short-term context without keeping the entire history.

**Phase 5 - Post-Session Consolidation**: An asynchronous job merges session notes into global memory using a consolidation agent that deduplicates exact and near-duplicate notes, eliminates ephemeral notes (marked "for this trip only"), resolves conflicts via recency, and returns canonical and deduplicated global notes.

#### Precedence Rules (Non-Negotiable)

1. Last user message in the current dialog
2. Session-level overrides from the current trip
3. Structured profile fields (reliable, from internal source)
4. Notes from global memory (advisory, not binding)

### Session Memory Management

The complementary notebook "Context Engineering - Session Memory" focuses on context window management with two fundamental techniques:

**Trimming (Last-N Turns)**: The `TrimmingSession` keeps only the last N user turns, eliminating everything that precedes. Advantages: deterministic, zero added latency, no risk of distortion from summary. Limitations: important previous context (IDs, constraints) vanishes abruptly.

**Summarization (Compression)**: The `SummarizingSession` keeps the last N turns verbatim while compressing everything previous into synthetic messages. The summary prompt emphasizes structured sections: product/environment details, problem statement, attempted steps with results, identifiers and timestamps, insights on tool performance, status and blockers, recommended next steps.

**Comparative Matrix**:

| Dimension | Trimming | Summarization |
|------------|----------|---------------|
| Latency/Cost | Minimal (no extra calls) | Greater at refresh points |
| Long-range memory | Weak (sharp cutoff) | Strong (compact carry-forward) |
| Risk type | Context loss | Context distortion/poisoning |
| Observability | Simple logs | Requires audit of summaries |

---

## 5. Practical Patterns

### 5.1 Triage Agent

The Triage Agent pattern is the most recurrent in the Cookbook and appears in at least three distinct notebooks. The idea is simple but powerful: an entry agent determines which specialist must handle the request.

In the orchestration notebook, the triage agent is the entry point of a customer service system. The instructions emphasize brief interactions and subtle information gathering:

```
The Triage Agent evaluates the user's request and:
- If about sales/purchases -> transfer_to_sales_agent()
- If about issues/repairs -> transfer_to_issues_agent()
- If ambiguous -> gathers more context before deciding
```

In the Deep Research notebook, the triage decides whether the query needs clarification or can proceed directly. In the Stripe dispute agent, the triage extracts the order ID from the payment intent metadata, retrieves the fulfillment status, and routes: unshipped orders go directly to dispute acceptance; shipped orders are escalated to investigation.

### 5.2 Multi-Agent Handoffs

The Cookbook presents two distinct paradigms for multi-agent collaboration:

**Handoff Collaboration**: Agents "pass the ball" to each other. One agent transfers the complete conversation to another agent that takes control. Useful for open-ended workflows where different agents have non-overlapping expertise. The transfer is implemented as a tool function that returns an Agent object.

**Agent-as-a-Tool**: A main agent (orchestrator) calls other agents as if they were tools. Sub-agents do not take control of the conversation; the main agent invokes them for specific subtasks and incorporates the results, maintaining a single thread of control.

The Multi-Agent Portfolio Collaboration notebook demonstrates the agent-as-a-tool pattern with a Portfolio Manager that orchestrates three specialists:

- **Fundamental Agent**: Analysis of corporate fundamentals, earnings, margins, moat
- **Macro Agent**: Macroeconomic conditions, rates, inflation, sector rotation
- **Quantitative Agent**: Statistical analysis, technical indicators, correlations, scenario modeling

The implementation uses Python decorators to wrap agents as tools:

```python
@function_tool(name_override="fundamental_analysis")
async def agent_tool(input):
    return await specialist_analysis_func(agent, input)
```

The PM uses `parallel_tool_calls=True` to invoke multiple specialists simultaneously, drastically reducing analysis time. The key difference compared to handoff: the PM remains in control for the entire duration, treating other agents as callable utilities rather than as peers with control over the conversation.

### 5.3 Advanced Tool Use

The Cookbook demonstrates a stratified tool ecosystem:

**Function Tools**: Python functions decorated with `@function_tool`, with automatic schema generation from type hints and docstrings. The SDK automatically validates input and output.

**Managed Tools**: Capabilities hosted by OpenAI:
- Code Interpreter for quantitative/statistical analysis
- WebSearch for retrieval of real-time information

**MCP Servers**: Model Context Protocol servers (e.g., Yahoo Finance) that provide standardized integration with external services. The `HostedMCPTool` configuration is declarative:

```python
HostedMCPTool(
    server_label="file_search",
    server_url="https://custom-endpoint.com",
    require_approval="never"
)
```

**Best Practices for Tool Use**:
- Always enable `strict: true` to ensure schema adherence
- Configurations with fewer than ~100 tools and ~20 arguments per tool are considered "in-distribution"
- When the number of functions increases, logically group tools and create specialized agents
- Use clear and concise descriptions in docstrings: the model uses them to decide when to invoke the tool

### 5.4 Structured Output for Agents

The notebook "Structured Outputs for Multi-Agent Systems" demonstrates how schema enforcement guarantees reliable communication between agents. By setting `strict: true`, responses are guaranteed to adhere to the provided JSON schema.

The notebook presents a 4-agent data analysis system:
1. **Triaging Agent**: Routes queries to specialists
2. **Data Processing Agent**: Cleaning, transformation, aggregation
3. **Analysis Agent**: Statistical analysis, correlation, regression
4. **Visualization Agent**: Chart creation (bar, line, pie)

Each agent receives tools with structured parameters and `additionalProperties: false`, eliminating edge cases and malformed outputs. The key benefits: guaranteed validation, reliability in function calls, scalability through specialization, and maintainability through separation of responsibilities.

---

## 6. Step-by-Step Implementation

### How to Follow Cookbook Notebooks

#### Prerequisites

```bash
pip install openai>=1.88 openai-agents>=0.0.19
```

For specific notebooks, additional dependencies may be needed such as `pydantic`, `langfuse`, `stripe`, or clients for MCP servers.

#### Step 1: Define the Agents

Start with the definition of agents and their instructions. Each agent has a clear role and a well-defined set of tools:

```python
from agents import Agent, Runner, function_tool

@function_tool
def search_database(query: str) -> str:
    """Search the products database."""
    return db.search(query)

triage_agent = Agent(
    name="Triage",
    model="gpt-4o",
    instructions="Determine the type of request and route...",
    tools=[search_database],
    handoffs=[sales_agent, support_agent]
)
```

#### Step 2: Configure Handoffs

Handoffs are declarative. The SDK represents them as tools for the LLM, so a handoff to "Refund Agent" becomes the `transfer_to_refund_agent` tool:

```python
sales_agent = Agent(
    name="Sales Agent",
    instructions="Handle sales requests...",
    tools=[check_inventory, create_order],
    handoffs=[triage_agent]  # can return to triage
)
```

#### Step 3: Implement the Runner

The `Runner` manages end-to-end execution: it receives input, handles retries, chooses tools, and returns responses. For streaming:

```python
result = await Runner.run_streamed(triage_agent, user_input)
async for event in result.stream_events():
    if event.type == "raw_response_event":
        # handle tool calls, search queries, etc.
        pass
```

#### Step 4: Add Guardrails

Guardrails validate inputs and outputs. They can be run with a fast/cheap model for preventive screening:

```python
from agents import InputGuardrail

async def check_toxicity(input_text: str) -> bool:
    # screening with lightweight model
    return is_safe

agent = Agent(
    name="Protected Agent",
    input_guardrails=[InputGuardrail(check_toxicity)]
)
```

#### Step 5: Configure Context Management

For long sessions, choose between trimming and summarization based on the use case:

```python
from agents import TrimmingSession, SummarizingSession

# For independent tasks
session = TrimmingSession(max_turns=10)

# For conversations with continuity
session = SummarizingSession(
    context_limit=20,
    keep_last_n_turns=5
)
```

#### Step 6: Enable Tracing and Evaluation

For complete observability, configure OpenTelemetry with Langfuse:

```python
import os
os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "https://cloud.langfuse.com"
os.environ["LANGFUSE_PUBLIC_KEY"] = "..."
os.environ["LANGFUSE_SECRET_KEY"] = "..."
```

The traces capture a hierarchical structure where a parent trace contains multiple spans for each step: LLM calls, tool invocations, and agent workflow. Each span records timing, token usage, approximate cost, and metadata.

---

## 7. Key Lessons - Insights for Agent Builders

### Lesson 1: The Simplicity of Primitives Wins Over Complexity

The routines + handoffs pattern is intentionally simple. You don't need a complex orchestration framework: Python functions that return Agent objects suffice. Complexity emerges from composition, not from the single unit. This is the Cookbook's most important insight.

### Lesson 2: Agent-as-a-Tool vs. Handoff -- Choose Mindfully

Handoff is suitable for conversational workflows where the user interacts with different agents sequentially. Agent-as-a-tool is preferable when a single point of control is needed that delegates subtasks and synthesizes results. The Portfolio Manager notebook is explicit: "Specialists do not take control of the conversation; the main agent invokes them for specific subtasks."

### Lesson 3: Context Engineering Is the True Differentiator

An agent's ability to be useful depends directly on the quality of the context it receives. The personalization notebook introduces a complete framework for deriving the shape of memory: "If this were a human agent, what would they keep in working memory?" This grounding in task relevance, rather than in arbitrary persistence, is fundamental.

### Lesson 4: Scope Separation for Memory

The distinction between global memory (durable, cross-session) and session memory (ephemeral, specific to the current interaction) reduces noise and enables safe and incremental evolution. Session notes are candidates; only after consolidation do they become permanent memory.

### Lesson 5: Structured Output as a Contract Between Agents

In a multi-agent system, communication between agents must be reliable. Using `strict: true` and Pydantic models transforms LLM output from "best effort" to "guaranteed". This is particularly critical when the output of one agent feeds the input of another.

### Lesson 6: The Triage Pattern Is Universal

Almost every multi-agent system benefits from a triage agent as the entry point. The triage reduces complexity for each individual downstream agent, improves routing accuracy, and provides a natural point for logging and monitoring.

### Lesson 7: Parallelism as a Latency Strategy

The Parallel Agents notebook demonstrates that running specialized agents concurrently with `asyncio.gather()` reduces latency to the time of the slowest agent, rather than to the sum of all. For latency-sensitive systems, parallelism via asyncio is preferable to agent-as-a-tool to avoid the overhead of planning.

### Lesson 8: Evaluation Is Continuous, Not One-Time

The Langfuse notebook emphasizes that agent evaluation is not a single event but a continuous process. The key metrics -- cost per interaction, latency per operation, accuracy (via LLM-as-judge), user satisfaction (direct feedback) -- must be monitored in production with the same attention dedicated to development.

### Lesson 9: Deep Research Only When Needed

The Cookbook's warning is clear: don't use Deep Research for trivial fact lookups or simple Q&A. The computational cost and latency of the four-agent pipeline are only justified for tasks that require planning, synthesis, and multi-step reasoning. For everything else, a standard API call is sufficient.

### Lesson 10: State Over Retrieval for Personal Contexts

The personalization notebook explicitly chooses state-based memory over semantic retrieval: "Travel decisions require continuity, priorities, and evolving preferences -- not ad-hoc search." State-based memory is deterministic, inspectable (readable YAML and Markdown), and does not suffer from the fragility of semantic search on sparse documents.

---

## 8. Resources and References

### Cookbook Notebooks (Agents Section)

| Resource | URL |
|---------|-----|
| Agents Section (index) | `developers.openai.com/cookbook/topic/agents` |
| Orchestrating Agents: Routines and Handoffs | `developers.openai.com/cookbook/examples/orchestrating_agents` |
| Deep Research API with Agents SDK | `developers.openai.com/cookbook/examples/deep_research_api/introduction_to_deep_research_api_agents` |
| Introduction to Deep Research API | `developers.openai.com/cookbook/examples/deep_research_api/introduction_to_deep_research_api` |
| Context Engineering for Personalization | `developers.openai.com/cookbook/examples/agents_sdk/context_personalization` |
| Context Engineering - Session Memory | `developers.openai.com/cookbook/examples/agents_sdk/session_memory` |
| Multi-Agent Portfolio Collaboration | `developers.openai.com/cookbook/examples/agents_sdk/multi-agent-portfolio-collaboration/multi_agent_portfolio_collaboration` |
| Parallel Agents | `developers.openai.com/cookbook/examples/agents_sdk/parallel_agents` |
| Evaluating Agents with Langfuse | `developers.openai.com/cookbook/examples/agents_sdk/evaluate_agents` |
| Structured Outputs for Multi-Agent Systems | `developers.openai.com/cookbook/examples/structured_outputs_multi_agent` |
| Automating Dispute Management with Agents SDK | `developers.openai.com/cookbook/examples/agents_sdk/dispute_agent` |
| Building a Voice Assistant with Agents SDK | `developers.openai.com/cookbook/examples/agents_sdk/app_assistant_voice_agents` |
| Skills in OpenAI API | `developers.openai.com/cookbook/examples/skills_in_api` |

### Agents SDK Documentation

| Resource | URL |
|---------|-----|
| Agents SDK (main documentation) | `openai.github.io/openai-agents-python/` |
| Agents | `openai.github.io/openai-agents-python/agents/` |
| Handoffs | `openai.github.io/openai-agents-python/handoffs/` |
| Guardrails | `openai.github.io/openai-agents-python/guardrails/` |
| Context Management | `openai.github.io/openai-agents-python/context/` |
| Runner | `openai.github.io/openai-agents-python/ref/run/` |

### Repository and Source Code

| Resource | URL |
|---------|-----|
| OpenAI Cookbook (GitHub) | `github.com/openai/openai-cookbook` |
| Agents SDK Python (GitHub) | `github.com/openai/openai-agents-python` |
| Swarm (proof-of-concept) | `github.com/openai/swarm` |

### Main Dependencies

```
openai >= 1.88
openai-agents >= 0.0.19
pydantic (for structured output and state management)
langfuse (for evaluation and tracing)
```

---

*Document compiled from direct research on OpenAI Cookbook notebooks, Agents section. Last update: March 2026.*
