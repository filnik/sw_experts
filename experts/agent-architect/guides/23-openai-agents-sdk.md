# OpenAI Agents SDK - Complete Technical Guide

> **Official documentation:** [openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/)
> **GitHub Repository:** [github.com/openai/openai-agents-python](https://github.com/openai/openai-agents-python) (20.3k+ stars, 237+ contributors, MIT license)
> **Requirements:** Python 3.10+
> **Installation:** `pip install openai-agents`
> **Status:** Production framework, weekly updates, multi-provider support

---

## 1. Overview - What It Is, Philosophy, Python-First Approach

### Origins and Positioning

The OpenAI Agents SDK is OpenAI's official framework for building agentic applications in Python. It originated as the evolution of the experimental project **Swarm**, launched in 2024 as a proof-of-concept for multi-agent orchestration patterns. While Swarm was avowedly an educational experiment, the Agents SDK is a production-ready framework designed to be adopted in real applications.

The project has rapidly gained traction in the open-source community, exceeding 20,000 stars on GitHub with over 4,800 projects actively using it. The development pace is among the highest in the agentic framework landscape, with multiple releases each week.

### Design Philosophy

The SDK's philosophy revolves around two fundamental principles:

1. **Balance between functionality and simplicity**: "Enough features to be worth using, but few enough primitives to make it quick to learn." The SDK offers a limited number of primitives -- Agent, Handoff, Guardrail, Runner, Tool -- that can be composed to build arbitrarily complex systems without requiring weeks of study.

2. **Flexibility with accessibility**: The framework works effectively out-of-the-box with minimal configurations, but allows granular customization at every level when needed.

This approach deliberately contrasts with more complex frameworks like LangChain/LangGraph, where the learning curve is significantly steeper. The Agents SDK aims to take a developer from zero to a working agent in minutes, not days.

### Python-First Approach

A key architectural choice of the SDK is its **Python-first** approach: instead of introducing proprietary DSLs, YAML configuration formats, or declarative graphs, the framework leverages native Python features for orchestration.

This means that:
- Control flow is expressed with `if/else`, loops, and standard `asyncio`
- Tool definitions use Python type hints and native decorators
- Validation leverages Pydantic without additional wrappers
- Context is managed with idiomatic dataclasses and dependency injection
- Asynchronous execution relies on native `async/await`

The practical advantage is that any Python developer can read and understand the code of an agentic system built with the SDK without having to learn framework-specific abstractions.

---

## 2. Fundamental Concepts

The SDK is built on five primitives that, when combined, allow building any agentic architecture.

### 2.1 Agent - The Building Block

An **Agent** is an LLM equipped with instructions, tools, and the ability to delegate work to other agents. It represents the framework's fundamental unit.

#### Basic Configuration

```python
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="You are an expert Python programming assistant.",
    model="gpt-4.1",
)

result = Runner.run_sync(agent, "Explain decorators in Python.")
print(result.final_output)
```

#### Main Properties

| Property | Description |
|-----------|-------------|
| `name` | Human-readable identifier of the agent |
| `instructions` | System prompt (static string or dynamic callback) |
| `model` | LLM model to use (default: `gpt-4.1`) |
| `tools` | List of tools available to the agent |
| `handoffs` | List of agents to which it can delegate |
| `output_type` | Type of structured output (Pydantic model) |
| `model_settings` | Model parameters (temperature, top_p, tool_choice) |
| `input_guardrails` | Input validation guardrails |
| `output_guardrails` | Output validation guardrails |
| `hooks` | Callbacks for lifecycle events |
| `mcp_servers` | MCP servers from which to load tools |
| `mcp_config` | MCP behavior configuration |

#### Dynamic Instructions

Instructions can be generated dynamically based on the execution context:

```python
from dataclasses import dataclass
from agents import Agent, RunContextWrapper

@dataclass
class UserContext:
    name: str
    uid: str
    is_pro_user: bool

def dynamic_instructions(
    context: RunContextWrapper[UserContext],
    agent: Agent[UserContext]
) -> str:
    user = context.context
    tier = "Premium" if user.is_pro_user else "Standard"
    return f"The user's name is {user.name} (tier: {tier}). Respond appropriately."

agent = Agent[UserContext](
    name="Support Agent",
    instructions=dynamic_instructions,
)
```

#### Structured Output

Beyond free text, an agent can produce structured output via Pydantic models:

```python
from pydantic import BaseModel

class SentimentAnalysis(BaseModel):
    sentiment: str
    confidence: float
    keywords: list[str]

agent = Agent(
    name="Sentiment Analyzer",
    instructions="Analyze the sentiment of the provided text.",
    output_type=SentimentAnalysis,
)
```

When `output_type` is specified, the model uses structured outputs mode, ensuring that the response always conforms to the defined schema.

#### Cloning Agents

An agent can be duplicated with selective property overrides:

```python
italian_agent = base_agent.clone(
    name="Italian Assistant",
    instructions="Always respond in formal Italian.",
)
```

### 2.2 Handoffs - Delegation Between Agents

**Handoffs** are the mechanism that allows an agent to transfer control to another specialized agent. When an agent declares a handoff to another agent, this is represented as a tool that the model can call: a handoff to "Refund Agent" becomes an invokable tool `transfer_to_refund_agent`.

#### Basic Pattern

```python
from agents import Agent, handoff

billing_agent = Agent(
    name="Billing Agent",
    instructions="Handle billing questions.",
    handoff_description="Transfer here for billing matters."
)

refund_agent = Agent(
    name="Refund Agent",
    instructions="Handle refund requests.",
    handoff_description="Transfer here for refund requests."
)

triage_agent = Agent(
    name="Triage Agent",
    instructions="Route requests to the appropriate department.",
    handoffs=[billing_agent, refund_agent],
)
```

#### Handoff with Callback and Metadata

The `handoff()` method offers advanced configuration, including callbacks that are executed at the moment of transfer and schemas for metadata generated by the model:

```python
from pydantic import BaseModel
from agents import Agent, handoff, RunContextWrapper

class EscalationData(BaseModel):
    reason: str
    priority: str

async def on_escalation(ctx: RunContextWrapper[None], input_data: EscalationData):
    print(f"Escalation: {input_data.reason} (priority: {input_data.priority})")
    # Log, notify, update database...

escalation_agent = Agent(name="Escalation Agent")

handoff_obj = handoff(
    agent=escalation_agent,
    on_handoff=on_escalation,
    input_type=EscalationData,
    tool_name_override="escalate_to_supervisor",
    tool_description_override="Escalate to supervisor for complex issues",
)
```

The `input_type` parameter is used exclusively for metadata generated by the model (such as escalation reason or priority). It does not replace the main input of the destination agent.

#### Input Filters

Input filters modify the conversation history passed to the next agent, allowing control over how much history is propagated:

```python
from agents import Agent, handoff
from agents.extensions import handoff_filters

faq_agent = Agent(name="FAQ Agent")

handoff_obj = handoff(
    agent=faq_agent,
    input_filter=handoff_filters.remove_all_tools,
)
```

The filters receive a `HandoffInputData` object that contains `input_history`, `pre_handoff_items`, `new_items`, and `run_context`, offering complete control over the conversational history.

#### Conditional Handoffs

Handoffs can be enabled or disabled dynamically:

```python
def check_availability(ctx: RunContextWrapper, agent: Agent) -> bool:
    return ctx.context.business_hours  # enabled only during business hours

handoff_obj = handoff(agent=human_agent, is_enabled=check_availability)
```

#### Nested Handoffs (Beta)

The `RunConfig.nest_handoff_history` functionality compresses the previous history into a single summary message, wrapped in `<CONVERSATION HISTORY>` blocks. This reduces token consumption when the handoff chain is long.

### 2.3 Guardrails - Validation and Safety

**Guardrails** are validation mechanisms that verify agent inputs and outputs, implementing safety and compliance checks. They operate via a **tripwire** system: when a guardrail detects a violation, it triggers the tripwire that immediately interrupts execution.

#### Three Categories of Guardrails

1. **Input Guardrails**: Validate the user's input before agent execution
2. **Output Guardrails**: Verify the agent's output after completion
3. **Tool Guardrails**: Wrap function tools with pre/post-execution validation

#### Input Guardrails

Input guardrails receive the same input passed to the agent, execute a validation function that produces `GuardrailFunctionOutput`, and check if `tripwire_triggered` is `True`. In affirmative cases, an `InputGuardrailTripwireTriggered` exception is raised.

```python
from agents import Agent, Runner, input_guardrail, GuardrailFunctionOutput

@input_guardrail
async def check_no_homework(ctx, agent, input):
    # Use a lightweight model to classify
    result = await Runner.run(
        Agent(
            name="Homework Detector",
            instructions="Respond 'yes' if the input seems like school homework.",
            output_type=bool,
        ),
        input,
    )
    return GuardrailFunctionOutput(
        output_info={"is_homework": result.final_output},
        tripwire_triggered=result.final_output,
    )

agent = Agent(
    name="Math Tutor",
    instructions="Help with math concepts, but don't solve homework.",
    input_guardrails=[check_no_homework],
)
```

#### Execution Modes

- **Parallel** (default): The guardrail runs in parallel with the agent, optimizing latency. If the tripwire fires, the agent's execution is interrupted.
- **Blocking** (`run_in_parallel=False`): The guardrail completes before the agent starts, avoiding token consumption if validation fails.

#### Boundaries in the Workflow

Guardrails have specific boundaries in the context of handoff chains:
- **Input guardrails** execute only on the **first agent** of the chain
- **Output guardrails** execute only on the **last agent** that produces the final output
- **Tool guardrails** execute on every function tool invocation

### 2.4 Runner - The Execution Engine

The **Runner** is the engine that manages the agent loop: it invokes the model, handles tool calls, processes results, and iterates until task completion.

#### Three Execution Modes

```python
from agents import Agent, Runner

agent = Agent(name="Assistant", instructions="You are a helpful assistant.")

# 1. Asynchronous (recommended for production)
result = await Runner.run(agent, "Hello!")

# 2. Synchronous (convenient for scripts and tests)
result = Runner.run_sync(agent, "Hello!")

# 3. Streaming (for real-time interfaces)
result = Runner.run_streamed(agent, "Hello!")
async for event in result.stream_events():
    print(event)
```

#### The Execution Loop

The Runner implements a sequential loop with the following logic:

1. **Calls the LLM** with the current agent and input
2. **Processes the LLM output**:
   - **Final output**: The loop terminates, the result is returned
   - **Handoff**: Updates the active agent, re-runs the loop
   - **Tool calls**: Executes the tools, adds the results, re-runs the loop
3. **Verifies the turn limit**: If `max_turns` is exceeded, raises `MaxTurnsExceeded`

The rule for determining whether an output is "final" is that the model produces text in the requested format and there are no pending tool calls.

#### RunConfig

`RunConfig` allows customizing every aspect of execution:

```python
from agents import RunConfig

config = RunConfig(
    model="gpt-4.1",                    # Global model override
    max_turns=10,                         # Turn limit
    tracing_disabled=False,               # Enable tracing
    workflow_name="customer_support",     # Workflow name for tracing
    trace_include_sensitive_data=False,   # Exclude sensitive data
)

result = await Runner.run(agent, "User input", run_config=config)
```

#### Error Handling

The Runner supports custom error handlers to manage situations such as turn limit exceedance:

```python
from agents import Runner, RunErrorHandlerInput, RunErrorHandlerResult

def on_max_turns(data: RunErrorHandlerInput[None]) -> RunErrorHandlerResult:
    return RunErrorHandlerResult(
        final_output="I apologize, I was unable to complete the request in the allotted time.",
        include_in_history=False,
    )

result = Runner.run_sync(
    agent, prompt,
    max_turns=5,
    error_handlers={"max_turns": on_max_turns},
)
```

#### Conversation Management Strategies

| Strategy | Where state resides | Use case |
|-----------|----------------------|------------|
| `to_input_list()` | Application memory | Complete manual control |
| `session` | Storage + SDK | Persistent chat |
| `conversation_id` | OpenAI API | Server-side named conversation |
| `previous_response_id` | OpenAI API | Lightweight continuation |

---

## 3. Tool Integration

The SDK supports five distinct categories of tools, each with specific characteristics.

### 3.1 Function Tools

The most direct way to create tools is the `@function_tool` decorator:

```python
from agents import function_tool
from pydantic import Field

@function_tool
async def search_database(
    query: str = Field(description="Search query"),
    limit: int = Field(default=10, ge=1, le=100, description="Maximum number of results"),
) -> str:
    """Search the corporate database.

    Args:
        query: The search query to execute
        limit: Maximum number of results to return
    """
    # ... search logic
    return f"Found {limit} results for '{query}'"
```

The SDK automatically:
- Extracts the function signature via `inspect`
- Analyzes docstrings (Google, Sphinx, NumPy formats) with `griffe`
- Generates the JSON schema via Pydantic
- Creates argument descriptions from the docstring

The tools support timeouts, custom error handling, and multiple return types (text, images with `ToolOutputImage`, files with `ToolOutputFileContent`).

#### Deferred Loading with ToolSearch

For agents with many tools, deferred loading reduces token consumption:

```python
from agents import function_tool, Agent
from agents.tool import ToolSearchTool

@function_tool(defer_loading=True)
def get_customer_profile(customer_id: str) -> str:
    """Retrieves the customer profile from the CRM."""
    return f"Profile for {customer_id}"

agent = Agent(
    tools=[get_customer_profile, ToolSearchTool()],
)
```

The model loads only the necessary tools at runtime, significantly reducing token overhead.

### 3.2 Hosted Tools

Tools executed on OpenAI servers along with the model:

- **WebSearchTool**: Web search
- **FileSearchTool**: Information retrieval from OpenAI Vector Stores
- **CodeInterpreterTool**: Code execution in sandbox
- **HostedMCPTool**: Tools from remote MCP servers
- **ImageGenerationTool**: Image generation

```python
from agents import Agent
from agents.tool import WebSearchTool, CodeInterpreterTool

agent = Agent(
    name="Research Assistant",
    instructions="Use web search and code interpreter to respond.",
    tools=[WebSearchTool(), CodeInterpreterTool()],
)
```

### 3.3 Agents as Tools

An agent can be exposed as a tool of another agent, keeping control in the orchestrator agent:

```python
spanish_agent = Agent(
    name="Spanish Translator",
    instructions="Translate the provided text into Spanish.",
)

orchestrator = Agent(
    name="Orchestrator",
    instructions="Use translation tools when requested.",
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="Translates text into Spanish",
        ),
    ],
)
```

Unlike handoffs, where control passes to the destination agent, with `as_tool()` the orchestrator agent retains control and receives back the result of the sub-execution.

### 3.4 Custom Function Tools

For complex scenarios, it is possible to create `FunctionTool` objects manually:

```python
from agents import FunctionTool, RunContextWrapper

async def run_function(ctx: RunContextWrapper, args: str) -> str:
    parsed = json.loads(args)
    return await process_data(parsed)

tool = FunctionTool(
    name="process_data",
    description="Process structured data",
    params_json_schema={...},
    on_invoke_tool=run_function,
)
```

---

## 4. Tracing and Observability

The SDK's tracing system offers complete visibility into agent execution, fundamental for debugging, monitoring, and optimization.

### Architecture

Tracing is structured around two concepts:

- **Trace**: Represents a complete end-to-end operation, composed of multiple spans. Key properties: `workflow_name`, `trace_id`, optional `group_id` to link conversations.
- **Span**: A single discrete operation with start/end timestamps, containing `span_data` with operation-specific information.

### What Is Traced Automatically

The SDK automatically traces:
- Complete Runner calls (`run`, `run_sync`, `run_streamed`)
- Agent executions
- LLM generations (input/output)
- Function tool calls
- Guardrail evaluations
- Handoffs between agents
- Speech-to-text transcriptions
- Text-to-speech conversions

### Configuration

Tracing is **enabled by default**. To disable it:

```python
from agents import set_tracing_disabled, RunConfig

# Global
set_tracing_disabled(True)

# Environment variable
# OPENAI_AGENTS_DISABLE_TRACING=1

# Per single execution
config = RunConfig(tracing_disabled=True)
```

### Sensitive Data Handling

By default, sensitive information is captured in traces. To exclude it:

```python
config = RunConfig(trace_include_sensitive_data=False)
```

Or via environment variable `OPENAI_AGENTS_TRACE_INCLUDE_SENSITIVE_DATA=false`.

### Custom Traces

The SDK allows creating custom traces via context managers:

```python
from agents import trace

with trace("custom_workflow"):
    result1 = await Runner.run(agent1, input1)
    result2 = await Runner.run(agent2, input2)
```

### Processing Pipeline

The architecture follows the pipeline: `TraceProvider` -> `BatchTraceProcessor` -> `BackendSpanExporter`.

Two approaches for customization:
- **`add_trace_processor()`**: Adds supplementary processors while maintaining export to the OpenAI backend
- **`set_trace_processors()`**: Completely replaces the default processors

### Visualization and Dashboard

Traces are visible in the [OpenAI Traces dashboard](https://platform.openai.com/traces), which offers debugging functionality, span tree visualization, and production monitoring.

### Third-Party Integrations

Over 20 services offer native support for tracing processors, including:
- **Weights & Biases**
- **Arize-Phoenix**
- **MLflow**
- **Langfuse**
- **LangSmith**
- **Pydantic Logfire**

This means you can use your preferred observability infrastructure without modifying application code.

---

## 5. Multi-Agent Patterns

The SDK supports two fundamental approaches to orchestrating multiple agents.

### 5.1 LLM-Driven Orchestration

The model autonomously decides the workflow direction using instructions, tools, and handoffs. This approach leverages the model's intelligence for planning and reasoning.

```python
research_agent = Agent(
    name="Research Agent",
    instructions="You are a researcher. Use available tools to gather information.",
    tools=[WebSearchTool(), FileSearchTool()],
    handoffs=[writing_agent, analysis_agent],
)
```

It is the ideal approach when the workflow is not predictable in advance and the model must dynamically evaluate which path to follow.

### 5.2 Code-Based Orchestration

The workflow is controlled programmatically, making execution more deterministic and predictable in terms of speed, cost, and performance.

```python
import asyncio
from agents import Agent, Runner

# Classification with structured output
class Classification(BaseModel):
    category: str
    urgency: int

classifier = Agent(
    name="Classifier",
    output_type=Classification,
    instructions="Classify the user's request.",
)

# Deterministic routing
result = await Runner.run(classifier, user_input)
classification = result.final_output

if classification.category == "billing":
    final = await Runner.run(billing_agent, user_input)
elif classification.category == "technical":
    final = await Runner.run(tech_agent, user_input)
else:
    final = await Runner.run(general_agent, user_input)
```

### 5.3 Agents as Tools vs Handoffs

The choice between these two patterns depends on the use case:

| Pattern | Behavior | When to use it |
|---------|--------------|---------------|
| **Agents as Tools** | The manager agent retains control, invokes sub-agents as functions | When an agent must "own" the final response; for enforced shared guardrails |
| **Handoffs** | The specialist becomes the active agent after routing | When the specialist must respond directly; to keep prompts focused |

### 5.4 Parallel Execution

For independent tasks, `asyncio` allows native parallel execution:

```python
import asyncio

async def parallel_research(topic: str):
    web_task = Runner.run(web_researcher, f"Search the web: {topic}")
    paper_task = Runner.run(paper_researcher, f"Search papers: {topic}")

    web_result, paper_result = await asyncio.gather(web_task, paper_task)

    synthesis = await Runner.run(
        synthesizer,
        f"Web: {web_result.final_output}\nPapers: {paper_result.final_output}",
    )
    return synthesis.final_output
```

### 5.5 Feedback Loop with Evaluator Agent

An advanced pattern provides for an agent that evaluates the output of another agent and requests iterative revisions:

```python
class Evaluation(BaseModel):
    score: int
    feedback: str
    acceptable: bool

evaluator = Agent(
    name="Evaluator",
    output_type=Evaluation,
    instructions="Evaluate the quality of the text. Score 1-10, acceptable if >= 7.",
)

writer = Agent(name="Writer", instructions="Write high-quality content.")

# Refinement loop
draft = await Runner.run(writer, task)
for _ in range(3):  # Max 3 iterations
    evaluation = await Runner.run(evaluator, draft.final_output)
    if evaluation.final_output.acceptable:
        break
    draft = await Runner.run(
        writer,
        f"Rewrite based on this feedback: {evaluation.final_output.feedback}",
    )
```

---

## 6. MCP (Model Context Protocol)

Integration with the **Model Context Protocol** is one of the most powerful features of the SDK, allowing agents to connect to any MCP server to extend their capabilities.

### Supported Transports

The SDK supports four MCP transport modes:

#### 1. Hosted MCP Server Tools (`HostedMCPTool`)

Delegates the entire tool round-trip to OpenAI's Responses API. The model lists and invokes the remote tools without callbacks to the local process.

```python
from agents import Agent
from agents.tool import HostedMCPTool

agent = Agent(
    name="Git Expert",
    tools=[
        HostedMCPTool(
            tool_config={
                "type": "mcp",
                "server_label": "gitmcp",
                "server_url": "https://gitmcp.io/openai/codex",
                "require_approval": "never",
            }
        )
    ],
)
```

#### 2. Streamable HTTP (`MCPServerStreamableHttp`)

For locally-controlled or internal infrastructure servers. Minimizes latency through direct transport management.

```python
from agents.mcp import MCPServerStreamableHttp

async with MCPServerStreamableHttp(
    name="Local MCP Server",
    params={
        "url": "http://localhost:8000/mcp",
        "headers": {"Authorization": f"Bearer {token}"},
        "timeout": 10,
    },
    cache_tools_list=True,
    max_retry_attempts=3,
) as server:
    agent = Agent(
        name="Agent with MCP",
        mcp_servers=[server],
    )
    result = await Runner.run(agent, "Execute the task")
```

#### 3. Stdio (`MCPServerStdio`)

Launches a local subprocess communicating via stdin/stdout. Useful for rapid prototypes.

```python
from agents.mcp import MCPServerStdio

async with MCPServerStdio(
    name="Filesystem Server",
    params={
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", str(work_dir)],
    },
) as server:
    agent = Agent(
        name="File Agent",
        mcp_servers=[server],
    )
```

#### 4. SSE (Deprecated)

The Server-Sent Events transport is maintained only for compatibility with legacy servers. New integrations should prefer Streamable HTTP or Stdio.

### Approval Policy

MCP tools support approval policies for sensitive operations:

```python
from agents.mcp import MCPToolApprovalRequest, MCPToolApprovalFunctionResult

def approve_tool(request: MCPToolApprovalRequest) -> MCPToolApprovalFunctionResult:
    SAFE_TOOLS = {"read_file", "list_directory"}
    if request.data.name in SAFE_TOOLS:
        return {"approve": True}
    return {"approve": False, "reason": "Requires human approval"}

# Granular per-tool configuration
server = MCPServerStreamableHttp(
    params={"url": "http://localhost:8000/mcp"},
    require_approval={
        "always": {"tool_names": ["delete_file", "write_file"]},
        "never": {"tool_names": ["read_file", "list_directory"]},
    },
)
```

### Tool Filtering

It is possible to filter which tools to expose, both statically and dynamically:

```python
from agents.mcp import create_static_tool_filter

# Static filter
server = MCPServerStdio(
    tool_filter=create_static_tool_filter(
        allowed_tool_names=["read_file", "search"],
    ),
)

# Dynamic filter based on context
async def context_aware_filter(context, tool) -> bool:
    if context.agent.name == "Read-Only Agent" and tool.name.startswith("write"):
        return False
    return True
```

### MCPServerManager

To manage multiple MCP servers simultaneously:

```python
from agents.mcp import MCPServerManager

async with MCPServerManager(servers, drop_failed_servers=True) as manager:
    agent = Agent(
        name="Multi-MCP Agent",
        mcp_servers=manager.active_servers,
    )
    # manager.failed_servers contains servers that did not connect
```

### Caching and Performance

All MCP classes support `cache_tools_list=True` to reduce the latency of `list_tools()` calls. Use `invalidate_tools_cache()` to force a refresh if the definitions change.

---

## 7. Comparison with LangGraph and CrewAI

### Comparative Overview

| Feature | OpenAI Agents SDK | LangGraph | CrewAI |
|---------------|-------------------|-----------|--------|
| **Philosophy** | Minimalism, Python-first | Stateful graphs, total control | Role-based, agent collaboration |
| **Learning curve** | Low (hours) | High (1-2 weeks) | Medium (days) |
| **Orchestration model** | Handoff + tool | Directed graphs with states | Crew/Task/Process |
| **Vendor lock-in** | Low (supports 100+ LLMs) | None (model-agnostic) | Low (multi-provider) |
| **GitHub Stars** | 20.3k+ | 38k+ | 44.6k+ |
| **Native tracing** | Yes (OpenAI dashboard) | Yes (LangSmith) | Yes (CrewAI+) |
| **MCP Support** | Native, 4 transports | Via integrations | First-class |
| **Guardrails** | Built-in (tripwire) | Custom middleware | Custom |
| **Development pace** | Weekly releases | Active (stable 1.0) | Active |

### OpenAI Agents SDK - Strengths

- **Simplicity without compromise**: Less than 100 lines of code for a working multi-agent system with handoffs and guardrails
- **Integrated tracing**: Zero configuration to have complete observability in the OpenAI dashboard
- **Native handoffs**: Elegant and intuitive delegation mechanism between agents, first of its kind
- **Native MCP**: First-class support for the Model Context Protocol with four transports
- **Performance**: Minimal framework overhead thanks to few abstractions
- **Voice agents**: Native support for voice agents with `gpt-realtime-1.5`

### OpenAI Agents SDK - Limitations

- **Younger ecosystem** compared to LangGraph
- **Fewer pre-integrated third-party tools** compared to LangChain
- **State persistence**: Sessions available but less mature than LangGraph's checkpointing
- **Workflow complexity**: For very complex workflows with articulated conditional branching, LangGraph offers more declarative control

### When to Choose What

- **OpenAI Agents SDK**: Rapid prototypes, multi-agent systems with delegation, when you want a lightweight and Python-idiomatic framework. Excellent if you work primarily with OpenAI models and want fast time-to-market.
- **LangGraph**: Complex workflows with persistent state, need for granular control over flow, enterprise applications with checkpointing and replay requirements. To choose when the workflow has complex branching and an explicit graph model is needed.
- **CrewAI**: Team-based workflows where agents have defined roles, rapid prototypes for collaborative systems. Ideal for those who prefer to think in terms of "teams of specialists" rather than graphs or chains of handoffs.

### Migration Patterns

The most common path is: **prototype with CrewAI or OpenAI Agents SDK** -> **production with LangGraph** when the limits of flow control are reached. However, for many use cases the OpenAI Agents SDK is sufficient even in production, especially with the addition of Sessions and durable execution (integration with Temporal and Restate).

---

## 8. Practical Examples and Best Practices

### Complete Example: Customer Support System

```python
from dataclasses import dataclass
from pydantic import BaseModel
from agents import (
    Agent, Runner, RunContextWrapper, handoff,
    function_tool, input_guardrail, GuardrailFunctionOutput,
)

# --- Shared context ---
@dataclass
class SupportContext:
    customer_id: str
    customer_tier: str
    session_id: str

# --- Tools ---
@function_tool
async def lookup_order(order_id: str) -> str:
    """Look up the details of an order.

    Args:
        order_id: The ID of the order to look up
    """
    return f"Order {order_id}: Shipped on 03/15, expected delivery 03/20"

@function_tool
async def process_refund(order_id: str, amount: float) -> str:
    """Process a refund for an order.

    Args:
        order_id: The ID of the order to refund
        amount: The refund amount
    """
    return f"Refund of EUR {amount} processed for order {order_id}"

# --- Guardrails ---
@input_guardrail
async def block_abuse(ctx, agent, input):
    abuse_detector = Agent(
        name="Abuse Detector",
        instructions="Respond 'true' if the input contains offensive or abusive language.",
        output_type=bool,
    )
    result = await Runner.run(abuse_detector, input)
    return GuardrailFunctionOutput(
        output_info={"abuse_detected": result.final_output},
        tripwire_triggered=result.final_output,
    )

# --- Specialized agents ---
order_agent = Agent[SupportContext](
    name="Order Specialist",
    instructions="Handle order questions. Use lookup_order to find details.",
    tools=[lookup_order],
    handoff_description="For questions about orders and shipments",
)

refund_agent = Agent[SupportContext](
    name="Refund Specialist",
    instructions="Handle refund requests. Verify the order before proceeding.",
    tools=[lookup_order, process_refund],
    handoff_description="For refund requests",
)

# --- Triage agent ---
triage_agent = Agent[SupportContext](
    name="Triage Agent",
    instructions="""You are the first point of contact for customer support.
    Route requests to the appropriate specialist.
    For general questions, respond directly.""",
    handoffs=[order_agent, refund_agent],
    input_guardrails=[block_abuse],
)

# --- Execution ---
async def handle_request(customer_id: str, message: str):
    context = SupportContext(
        customer_id=customer_id,
        customer_tier="premium",
        session_id="sess_123",
    )

    try:
        result = await Runner.run(
            triage_agent,
            message,
            context=context,
            max_turns=10,
        )
        return result.final_output
    except Exception as e:
        return f"An error occurred: {e}"
```

### Best Practices

1. **Invest in prompt quality**: Clear and specific instructions are the most important factor for agent performance. Explicitly describe which tools are available and when to use them.

2. **Specialized agents**: Create agents that excel at a single task rather than "do-it-all" agents. A focused agent produces better results than a generic one.

3. **Monitor and iterate**: Use tracing to analyze failures. Every error is an opportunity to improve prompts, tools, or routing logic.

4. **Layered guardrails**: Apply input guardrails on the first agent, output guardrails on the last, and tool guardrails where necessary. Do not overload with redundant validations.

5. **Token management**: Use `input_filter` on handoffs to limit the history passed to subsequent agents. Consider `nest_handoff_history` for long chains.

6. **Structure outputs**: Use `output_type` with Pydantic models when possible. Structured output is more reliable and facilitates integration with downstream code.

7. **Graceful errors**: Always implement error handlers for `max_turns` and handle guardrail exceptions. A production-ready system must never crash without an informative message.

8. **MCP caching**: Enable `cache_tools_list=True` on MCP servers with stable definitions. It significantly reduces call latency.

9. **Separate LLM orchestration and code**: Use LLM-driven orchestration when the flow is unpredictable; code-based orchestration when determinism and predictability are needed.

10. **Lifecycle hooks**: Use `RunHooks` and `AgentHooks` for logging, metrics, and side-effects without polluting the agents' logic.

---

## 9. Resources and References

### Official Documentation
- **Docs**: [openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/)
- **GitHub**: [github.com/openai/openai-agents-python](https://github.com/openai/openai-agents-python)
- **API Reference**: [openai.github.io/openai-agents-python/ref/](https://openai.github.io/openai-agents-python/ref/)

### Specific Guides
- [Agents](https://openai.github.io/openai-agents-python/agents/) - Complete agent configuration
- [Running Agents](https://openai.github.io/openai-agents-python/running_agents/) - Runner and execution loop
- [Tools](https://openai.github.io/openai-agents-python/tools/) - All tool categories
- [Handoffs](https://openai.github.io/openai-agents-python/handoffs/) - Delegation between agents
- [Guardrails](https://openai.github.io/openai-agents-python/guardrails/) - Validation and safety
- [Tracing](https://openai.github.io/openai-agents-python/tracing/) - Observability and debugging
- [MCP](https://openai.github.io/openai-agents-python/mcp/) - Model Context Protocol
- [Multi-Agent](https://openai.github.io/openai-agents-python/multi_agent/) - Orchestration patterns
- [Models](https://openai.github.io/openai-agents-python/models/) - Model and provider configuration

### Comparisons and Analyses
- [LangGraph vs CrewAI vs OpenAI Agents SDK 2026](https://particula.tech/blog/langgraph-vs-crewai-vs-openai-agents-sdk-2026)
- [AI Agent Comparison - Langfuse](https://langfuse.com/blog/2025-03-19-ai-agent-comparison)
- [OpenAI Agents SDK vs LangGraph vs Autogen vs CrewAI - Composio](https://composio.dev/blog/openai-agents-sdk-vs-langgraph-vs-autogen-vs-crewai)

### Installation

```bash
# Base
pip install openai-agents

# With voice support
pip install 'openai-agents[voice]'

# With Redis sessions
pip install 'openai-agents[redis]'

# With LiteLLM support for non-OpenAI providers
pip install 'openai-agents[litellm]'
```

### Supported Models

The default model is `gpt-4.1`. The SDK supports two model interfaces:
- **OpenAIResponsesModel** (recommended): Uses the Responses API
- **OpenAIChatCompletionsModel**: Uses the Chat Completions API, compatible with 100+ providers

For non-OpenAI providers, the SDK offers integration via LiteLLM (beta) and support for OpenAI-compatible endpoints.
