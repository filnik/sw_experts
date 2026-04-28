# OpenAI Agents SDK

## 1. Overview

The **OpenAI Agents SDK** (PyPI package: `openai-agents`) is a lightweight, Python-first, minimalist framework for building multi-agent workflows. Released under the MIT license, the project has surpassed 20,000 stars on GitHub, has more than 230 active contributors, and over 1,200 commits on the main branch (v0.13.1, March 2026).

The design philosophy is summarized in three fundamental principles:

- **Few abstractions**: the framework introduces only the strictly necessary primitives -- Agent, Handoff, Guardrail, Tool -- avoiding proprietary orchestration layers.
- **Python-first**: orchestration is expressed with native language constructs (`async/await`, context managers, type annotations, Pydantic) rather than DSLs or YAML configurations.
- **Provider-agnostic**: while optimized for OpenAI APIs (Responses API and Chat Completions API), it supports over 100 alternative LLM models via extensions such as `AnyLLMModel` and LiteLLM.

The minimum requirement is Python 3.10+. Basic installation is done with:

```bash
pip install openai-agents
```

Optional extras are available for voice (`openai-agents[voice]`), Redis sessions (`openai-agents[redis]`) and LiteLLM (`openai-agents[litellm]`). Core dependencies include Pydantic for validation, the MCP Python SDK protocol for integration with MCP servers, and, optionally, SQLAlchemy for session persistence.

A minimal example illustrates the entire API surface needed to run an agent:

```python
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="You are a helpful assistant"
)

result = Runner.run_sync(agent, "Write a haiku about recursion in programming.")
print(result.final_output)
```

This is the core of the proposition: in less than ten lines of code you define an agent, run it, and obtain its final output. Everything else -- tools, handoffs, guardrails, tracing -- is composed incrementally around this base structure.

---

## 2. Internal Architecture

### 2.1 The Agent Loop

The heart of the SDK is an execution loop managed by the `Runner` class. At each iteration the loop performs the following steps:

1. **LLM model call**: the runner sends the agent's instructions, the conversation history, and the definition of available tools to the current model.
2. **Output evaluation**:
   - If the model produces a **final output** (text or structured output) without tool calls, the loop terminates and the result is returned.
   - If the model requests a **handoff**, the runner updates the current agent, modifies the input according to any configured filters, and resumes the loop from the beginning.
   - If the model emits **tool calls**, the runner executes the requested tools, adds the results to the history, and resumes the loop.
3. **Turn limit**: if the number of iterations exceeds `max_turns`, the `MaxTurnsExceeded` exception is raised (customizable with an `error_handler`).

The rule that determines whether an LLM output is "final" is simple: the model produces text (or structured output conforming to `output_type`) **and** does not emit any tool calls at the same time.

### 2.2 Tool Execution

When the model emits one or more tool calls, the runner resolves them in the following order:

1. Deserializes the JSON arguments of the tool call.
2. Invokes the corresponding Python function (or MCP server, or hosted tool).
3. Serializes the result and adds it to the conversation as `tool_output`.
4. Returns control to the loop for the next LLM iteration.

Tools can be synchronous or asynchronous. For asynchronous tools it is possible to configure a timeout with `@function_tool(timeout=2.0)`, with two configurable behaviors: `error_as_result` (default, returns an error message to the model) or `raise_exception` (raises `ToolTimeoutError`).

### 2.3 Handoff Mechanism

Handoffs are the native mechanism for delegation between agents. Internally, a handoff is represented as a **tool** from the LLM model's point of view. If an agent has a handoff to an agent called `Refund Agent`, the model will see a tool called `transfer_to_refund_agent`.

When the model invokes this tool:

1. The runner records the handoff event (emitting hooks and tracing spans).
2. Executes any `on_handoff` callback (useful for loading data or preparing context).
3. Applies any `input_filter` to modify the conversation history before passing it to the new agent.
4. Replaces the current agent with the destination one and resumes the loop.

This mechanism enables sophisticated triage architectures where a dispatcher agent routes requests to domain specialists.

---

## 3. Core API

### 3.1 Agent Class

The `Agent` class is the fundamental unit of the framework. It accepts the following configuration parameters:

| Parameter | Required | Description |
|-----------|:---:|-------------|
| `name` | Yes | Human-readable name of the agent |
| `instructions` | Yes | System prompt, string or dynamic function |
| `model` | No | LLM model (default configurable globally) |
| `tools` | No | List of tools available to the agent |
| `handoffs` | No | List of agents or `Handoff` objects for delegation |
| `output_type` | No | Structured output type (Pydantic model) |
| `model_settings` | No | Model parameters (temperature, top_p, tool_choice) |
| `hooks` | No | Callbacks for lifecycle events |
| `input_guardrails` | No | Validation guardrails on input |
| `output_guardrails` | No | Validation guardrails on output |
| `mcp_servers` | No | List of connected MCP servers |
| `mcp_config` | No | MCP configuration (strict schemas, error handling) |
| `tool_use_behavior` | No | Strategy for handling tool results |

**Dynamic instructions** allow generating the system prompt as a function of the execution context:

```python
def dynamic_instructions(
    context: RunContextWrapper[UserContext],
    agent: Agent[UserContext]
) -> str:
    return f"The user's name is {context.context.name}. Respond formally."

agent = Agent[UserContext](
    name="Personalized assistant",
    instructions=dynamic_instructions,
)
```

**Structured outputs** are defined via Pydantic models, ensuring that the agent's final output is always conformant to a precise schema:

```python
from pydantic import BaseModel

class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

agent = Agent(
    name="Calendar extractor",
    instructions="Extract events from the provided text",
    output_type=CalendarEvent,
)
```

The `tool_use_behavior` parameter controls how the runner handles tool results:

- `"run_llm_again"` (default): the model processes the tool results and generates a new output.
- `"stop_on_first_tool"`: the output of the first tool becomes the agent's final output.
- `StopAtTools(stop_at_tool_names=[...])`: the loop stops only for specific tools.
- Custom function (`ToolsToFinalOutputFunction`): programmed logic to determine when to stop.

### 3.2 Runner

The `Runner` class manages the execution of the agent loop. It exposes three main methods:

```python
# Async - returns RunResult
result = await Runner.run(agent, "User's question", max_turns=10)

# Sync - wrapper around run()
result = Runner.run_sync(agent, "User's question")

# Streaming - returns RunResultStreaming with real-time events
result = Runner.run_streamed(agent, "User's question")
async for event in result.stream_events():
    # Process token-by-token events
    pass
```

The input accepted by the runner can be a string (treated as a user message), a list of items in the OpenAI Responses API format, or a `RunState` object to resume interrupted executions.

The `RunConfig` allows granular configuration:

```python
from agents import Runner, RunConfig

result = await Runner.run(
    agent,
    "Question",
    max_turns=15,
    run_config=RunConfig(
        model="gpt-4o",
        tracing_disabled=False,
        trace_include_sensitive_data=False,
        workflow_name="customer-support-flow",
    )
)
```

For **multi-turn conversation management**, the framework offers four strategies:

| Strategy | Storage | Use case |
|-----------|---------|------------|
| `to_input_list()` | Application memory | Manual control of history |
| `session` | Local storage (SQLite/SQLAlchemy) | Persistent chats |
| `conversation_id` | OpenAI servers | Named conversations |
| `previous_response_id` | OpenAI servers | Lightweight chaining |

**Streaming** supports three categories of events:

- **`RawResponsesStreamEvent`**: token-by-token from the model, in Responses API format.
- **`RunItemStreamEvent`**: high-level semantic events (`message_output_created`, `handoff_requested`, `tool_called`, `tool_output`, etc.).
- **`AgentUpdatedStreamEvent`**: emitted when the current agent changes (typically during a handoff).

It is possible to cancel a stream with `result.cancel()` (immediate) or `result.cancel(mode="after_turn")` to complete the current turn before stopping.

### 3.3 Tool

The SDK supports five categories of tools:

**1. Function Tool** -- the `@function_tool` decorator automatically transforms a Python function into a tool, generating the JSON schema from type hints and docstring:

```python
from agents import function_tool

@function_tool
async def fetch_weather(location: str) -> str:
    """Retrieve the weather for a location.

    Args:
        location: The location to retrieve the weather for.
    """
    return f"Sunny in {location}"
```

The decorator supports name override (`name_override`), custom descriptions, timeouts, custom error handlers (`failure_error_function`), and deferred loading (`defer_loading=True`) to reduce token consumption.

**2. Hosted Tool** -- tools executed on OpenAI servers:
- `WebSearchTool`: web search
- `FileSearchTool`: queries Vector Stores with filters and ranking
- `CodeInterpreterTool`: code execution in sandbox
- `ImageGenerationTool`: image generation
- `HostedMCPTool`: tools from remote MCP servers
- `ToolSearchTool`: deferred tool loading

**3. Local Runtime Tool** -- executed in the local environment:
- `ComputerTool`: GUI/browser automation via the `Computer`/`AsyncComputer` interface
- `ApplyPatchTool`: application of diffs/patches
- `ShellTool`: execution of shell commands (hosted or with custom local executor)

**4. Agent as Tool** -- an agent exposed as a tool callable from another agent:

```python
spanish_agent = Agent(name="Spanish", instructions="Translate to Spanish")
french_agent = Agent(name="French", instructions="Translate to French")

orchestrator = Agent(
    name="Orchestrator",
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="Translates to Spanish",
        ),
        french_agent.as_tool(
            tool_name="translate_to_french",
            tool_description="Translates to French",
        ),
    ],
)
```

Tools derived from agents support structured input (Pydantic), approval (`needs_approval=True`), custom output extraction (`custom_output_extractor`), streaming of internal events (`on_stream`), and conditional enabling (`is_enabled`).

**5. Codex Tool** (experimental) -- execution of tasks in workspaces via the Codex CLI, with support for persistent sessions and sandbox configuration.

Tools can be grouped into namespaces via `tool_namespace()` for logical organization, particularly useful with `ToolSearchTool` for deferred loading:

```python
from agents import function_tool, tool_namespace, ToolSearchTool

@function_tool(defer_loading=True)
def get_customer_profile(customer_id: str) -> str:
    """Retrieve the customer's CRM profile."""
    return f"Profile for {customer_id}"

@function_tool(defer_loading=True)
def list_open_orders(customer_id: str) -> str:
    """List the customer's open orders."""
    return f"Orders for {customer_id}"

crm_tools = tool_namespace(
    name="crm",
    description="CRM tools for customer management.",
    tools=[get_customer_profile, list_open_orders],
)

agent = Agent(
    name="Operations assistant",
    tools=[*crm_tools, ToolSearchTool()],
)
```

### 3.4 Handoff

Handoffs allow explicit delegation between agents. They are defined in the `handoffs` list of the agent or via the `handoff()` function for advanced configuration:

```python
from agents import Agent, handoff
from pydantic import BaseModel

class EscalationData(BaseModel):
    reason: str

async def on_escalation(ctx, input_data: EscalationData):
    print(f"Escalation reason: {input_data.reason}")

billing_agent = Agent(name="Billing agent", instructions="Handle billing")
refund_agent = Agent(name="Refund agent", instructions="Handle refunds")

triage_agent = Agent(
    name="Triage agent",
    instructions="Route requests to the appropriate specialist.",
    handoffs=[
        billing_agent,
        handoff(
            agent=refund_agent,
            on_handoff=on_escalation,
            input_type=EscalationData,
            tool_description_override="Transfer to the refunds team",
        ),
    ],
)
```

The parameters of the `handoff()` function:

- **`agent`**: destination agent.
- **`tool_name_override`**: custom tool name (default: `transfer_to_<agent_name>`).
- **`tool_description_override`**: custom description.
- **`on_handoff`**: callback executed at the time of transfer.
- **`input_type`**: Pydantic schema for metadata generated by the model (reason, priority, etc.).
- **`input_filter`**: function to modify the history passed to the new agent.
- **`is_enabled`**: boolean or function for conditional enabling at runtime.
- **`nest_handoff_history`**: compresses the previous history into `<CONVERSATION HISTORY>` blocks.

The **input filters** (`input_filter`) receive a `HandoffInputData` object with access to `input_history`, `pre_handoff_items`, `new_items`, `input_items`, and `run_context`. The SDK provides predefined filters such as `handoff_filters.remove_all_tools`.

To optimize the behavior of handoffs in multi-agent scenarios, it is recommended to use the recommended prompt prefix:

```python
from agents.extensions.handoff_prompt import RECOMMENDED_PROMPT_PREFIX

billing_agent = Agent(
    name="Billing agent",
    instructions=f"{RECOMMENDED_PROMPT_PREFIX}\nGestisci domande sulla fatturazione."
)
```

---

## 4. Guardrails

Guardrails are configurable validation mechanisms that check inputs and outputs of agents. They are designed to use fast/economical models in order to identify problems before the main model consumes resources.

### 4.1 Input Guardrails

Input guardrails perform validation on the user's first input. The process involves three phases:

1. The guardrail receives the same input passed to the agent.
2. The guardrail function produces a `GuardrailFunctionOutput` containing `output_info` and `tripwire_triggered`.
3. If `tripwire_triggered` is `True`, the `InputGuardrailTripwireTriggered` exception is raised.

Two execution modes exist:

- **Parallel execution** (default): the guardrail runs concurrently with the agent. It offers the best latency, but the agent might consume tokens before cancellation.
- **Blocking execution**: the guardrail completes **before** the agent starts. If the tripwire fires, the agent is never executed, saving tokens and preventing tool side effects.

```python
from pydantic import BaseModel
from agents import (
    Agent, GuardrailFunctionOutput, InputGuardrailTripwireTriggered,
    RunContextWrapper, Runner, input_guardrail,
)

class HomeworkCheck(BaseModel):
    is_homework: bool
    reasoning: str

guardrail_agent = Agent(
    name="Guardrail check",
    instructions="Verifica se l'utente chiede aiuto con compiti scolastici",
    output_type=HomeworkCheck,
)

@input_guardrail
async def homework_guardrail(ctx, agent, input):
    result = await Runner.run(guardrail_agent, input, context=ctx.context)
    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=result.final_output.is_homework,
    )

agent = Agent(
    name="Customer support",
    instructions="You are a customer service assistant",
    input_guardrails=[homework_guardrail],
)

# Utilizzo
try:
    await Runner.run(agent, "Risolvi 2x + 3 = 11")
except InputGuardrailTripwireTriggered:
    print("Guardrail attivato: richiesta compiti scolastici bloccata")
```

### 4.2 Output Guardrails

They work analogously but validate the agent's final output. They always run after the agent completes (they do not support `run_in_parallel`). They raise `OutputGuardrailTripwireTriggered` if the tripwire fires.

```python
@output_guardrail
async def pii_guardrail(ctx, agent, output):
    result = await Runner.run(pii_checker_agent, output.response, context=ctx.context)
    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=result.final_output.contains_pii,
    )
```

### 4.3 Tool Guardrails

Applicable only to `function_tool`, they validate inputs and outputs of each tool invocation:

```python
from agents import function_tool, tool_input_guardrail, tool_output_guardrail, ToolGuardrailFunctionOutput

@tool_input_guardrail
def block_secrets(data):
    args = json.loads(data.context.tool_arguments or "{}")
    if "sk-" in json.dumps(args):
        return ToolGuardrailFunctionOutput.reject_content(
            "Rimuovi i segreti prima di chiamare questo tool."
        )
    return ToolGuardrailFunctionOutput.allow()

@tool_output_guardrail
def redact_output(data):
    if "sk-" in str(data.output or ""):
        return ToolGuardrailFunctionOutput.reject_content(
            "L'output conteneva dati sensibili."
        )
    return ToolGuardrailFunctionOutput.allow()

@function_tool(
    tool_input_guardrails=[block_secrets],
    tool_output_guardrails=[redact_output],
)
def classify_text(text: str) -> str:
    """Classifica il testo per il routing interno."""
    return f"lunghezza:{len(text)}"
```

### 4.4 Execution Boundaries

It is fundamental to understand **where** each type of guardrail acts:

- **Input guardrail**: only on the **first agent** of the chain.
- **Output guardrail**: only on the **last agent** that produces final output.
- **Tool guardrail**: on **every invocation** of the function tool, regardless of the agent. This makes them ideal for workflows with handoffs and delegation.

---

## 5. Tracing

### 5.1 Built-in Tracing

The SDK includes an automatic tracing system based on two primitives: **Trace** (end-to-end operation of a workflow) and **Span** (single operation with start and end timestamps).

Tracing activates automatically and captures:

- Runner operations (`run()`, `run_sync()`, `run_streamed()`)
- Agent executions
- LLM generations
- Function tool invocations
- Guardrail checks
- Handoffs between agents
- Audio transcription and speech synthesis

Each trace includes `workflow_name`, `trace_id` (format `trace_<32_alphanumeric>`), an optional `group_id`, and arbitrary metadata. Spans contain timestamps, `trace_id`, parent-child relationships, and typed `span_data`.

### 5.2 Custom Traces

To group multiple executions under a single trace:

```python
from agents import Agent, Runner, trace

async def main():
    agent = Agent(name="Joke generator", instructions="Racconta barzellette.")

    with trace("Joke workflow"):
        first = await Runner.run(agent, "Raccontami una barzelletta")
        second = await Runner.run(
            agent,
            f"Valuta questa barzelletta: {first.final_output}"
        )
```

Traces use context managers (`with trace(...)`) and Python's `contextvar` for automatic concurrency management. For custom spans, the SDK offers the `custom_span()` function.

### 5.3 Sensitive Data

Two options control the capture of sensitive data:

- `RunConfig.trace_include_sensitive_data`: controls capture of LLM inputs/outputs and function call data (default: `True`).
- Environment variable: `OPENAI_AGENTS_TRACE_INCLUDE_SENSITIVE_DATA=0` to disable globally.

### 5.4 Custom Processors

The architecture uses a global `TraceProvider` with `BatchTraceProcessor` that sends spans to `BackendSpanExporter`:

- `add_trace_processor()`: adds processors alongside the OpenAI exporter.
- `set_trace_processors()`: completely replaces the default processors (disabling export to OpenAI unless explicitly included).

### 5.5 Disabling

Three methods to disable tracing:

```python
# 1. Variabile d'ambiente
# OPENAI_AGENTS_DISABLE_TRACING=1

# 2. Programmatico globale
from agents import set_tracing_disabled
set_tracing_disabled(True)

# 3. Per singola esecuzione
RunConfig(tracing_disabled=True)
```

Note: for organizations with Zero Data Retention (ZDR) policies, tracing is not available.

### 5.6 Integrations with Observability Tools

The SDK supports more than 25 third-party processors for observability, including:

- **Weights & Biases (Weave)**: ML experiment monitoring
- **Arize-Phoenix**: observability for LLMs in production
- **MLflow** (OSS and Databricks): experiment tracking
- **Braintrust**: evaluation and debugging
- **Pydantic Logfire**: structured logging
- **AgentOps**: agent-specific monitoring
- **LangSmith / Langfuse / Langtrace**: observability for LLM chains
- **PostHog**: product analytics
- **PromptLayer**: prompt management and versioning

Each integration provides a native processor that connects to the SDK's tracing architecture.

---

## 6. MCP Integration

The SDK natively integrates the **Model Context Protocol (MCP)**, an open protocol to standardize how applications provide context to LLM models. The integration supports three transports:

### 6.1 MCPServerStdio

Spawns a local process that communicates via stdin/stdout. Ideal for proofs of concept and command-line servers:

```python
from agents import Agent, Runner
from agents.mcp import MCPServerStdio

async with MCPServerStdio(
    name="Filesystem Server",
    params={
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    }
) as server:
    agent = Agent(name="Assistant", mcp_servers=[server])
    result = await Runner.run(agent, "Elenca i file nella directory")
```

### 6.2 MCPServerSse (deprecated)

HTTP transport with Server-Sent Events. Currently deprecated by the MCP project: it is recommended to use Streamable HTTP or stdio for new projects.

### 6.3 MCPServerStreamableHttp

HTTP connections for remote servers, optimal for low-latency infrastructure:

```python
from agents.mcp import MCPServerStreamableHttp

async with MCPServerStreamableHttp(
    name="Remote MCP Server",
    params={
        "url": "http://localhost:8000/mcp",
        "headers": {"Authorization": f"Bearer {token}"},
        "timeout": 10
    },
    cache_tools_list=True,
    max_retry_attempts=3,
    retry_backoff_seconds_base=2,
) as server:
    agent = Agent(name="Assistant", mcp_servers=[server])
```

### 6.4 Advanced Configuration

**Tool filter** -- static (allow/block list) or dynamic (callable function):

```python
async def context_aware_filter(context, tool) -> bool:
    if context.agent.name == "Reviewer" and tool.name.startswith("danger_"):
        return False
    return True
```

**Approval policy** -- to control the execution of potentially dangerous tools:

```python
# Per-tool mapping
require_approval={"delete_file": "always", "read_file": "never"}

# Raggruppata
require_approval={"always": {"tool_names": ["delete_file"]}, "never": {"tool_names": ["read_file"]}}
```

**Per-call metadata** (`tool_meta_resolver`) -- injects contextual metadata such as tenant ID into MCP requests.

**MCPServerManager** -- manages multiple MCP servers simultaneously with connection pooling, automatic retry, and failure handling:

```python
from agents.mcp import MCPServerManager

async with MCPServerManager(
    servers,
    drop_failed_servers=True,
    connect_timeout_seconds=10,
    connect_in_parallel=True
) as manager:
    agent = Agent(mcp_servers=manager.active_servers)
```

**Caching**: `cache_tools_list=True` prevents repeated calls to `list_tools()`, reducing latency. Use `invalidate_tools_cache()` to force a refresh.

---

## 7. Advanced Patterns

### 7.1 Triage + Handoff

The triage pattern is the most common: a dispatcher agent analyzes the request and delegates it to the appropriate specialist. Each specialist has focused instructions and tools specific to its domain.

```python
from agents import Agent, handoff
from agents.extensions.handoff_prompt import RECOMMENDED_PROMPT_PREFIX

billing_agent = Agent(
    name="Billing agent",
    instructions=f"{RECOMMENDED_PROMPT_PREFIX}\nGestisci domande su fatture e pagamenti.",
    tools=[get_invoice, update_payment_method],
)

refund_agent = Agent(
    name="Refund agent",
    instructions=f"{RECOMMENDED_PROMPT_PREFIX}\nGestisci richieste di rimborso.",
    tools=[process_refund, check_refund_status],
)

faq_agent = Agent(
    name="FAQ agent",
    instructions=f"{RECOMMENDED_PROMPT_PREFIX}\nRispondi a domande frequenti.",
    tools=[search_knowledge_base],
)

triage_agent = Agent(
    name="Triage agent",
    instructions="You are a routing agent. Analyze the customer's request and transfer it to the appropriate specialist.",
    handoffs=[billing_agent, refund_agent, faq_agent],
)
```

### 7.2 Deterministic Workflows

For workflows where execution order must be determined by the code (not by the model), code-based orchestration with structured outputs is used:

```python
from agents import Agent, Runner
from pydantic import BaseModel

class ResearchResult(BaseModel):
    summary: str
    key_points: list[str]

class Outline(BaseModel):
    sections: list[str]

# Step 1: Ricerca
research_agent = Agent(
    name="Researcher",
    instructions="Analyze the topic and produce a structured summary.",
    output_type=ResearchResult,
)

# Step 2: Outline
outline_agent = Agent(
    name="Outliner",
    instructions="Crea un outline basato sulla ricerca fornita.",
    output_type=Outline,
)

# Step 3: Scrittura
writer_agent = Agent(
    name="Writer",
    instructions="Write a complete article following the outline.",
)

async def deterministic_pipeline(topic: str):
    research = await Runner.run(research_agent, topic)
    outline = await Runner.run(
        outline_agent,
        f"Ricerca: {research.final_output.summary}\nPunti chiave: {research.final_output.key_points}"
    )
    article = await Runner.run(
        writer_agent,
        f"Outline: {outline.final_output.sections}\nRicerca: {research.final_output.summary}"
    )
    return article.final_output
```

For independent tasks, you can use `asyncio.gather` to parallelize:

```python
import asyncio

async def parallel_research(topics: list[str]):
    tasks = [Runner.run(research_agent, topic) for topic in topics]
    results = await asyncio.gather(*tasks)
    return [r.final_output for r in results]
```

### 7.3 Agents as Tools

When an agent must coordinate specialists without ceding control of the conversation, the "agents as tools" pattern is used. The orchestrator agent invokes the specialists as tools, receives their results, and composes the final response:

```python
researcher = Agent(name="Researcher", instructions="Ricerca approfondita.")
analyst = Agent(name="Analyst", instructions="Analisi quantitativa.")
writer = Agent(name="Writer", instructions="Scrittura persuasiva.")

manager = Agent(
    name="Project Manager",
    instructions="Coordinate the team to produce complete reports.",
    tools=[
        researcher.as_tool(
            tool_name="research",
            tool_description="Esegui ricerca su un argomento",
            needs_approval=False,
        ),
        analyst.as_tool(
            tool_name="analyze",
            tool_description="Analyze quantitative data",
            needs_approval=False,
        ),
        writer.as_tool(
            tool_name="write_section",
            tool_description="Write a section of the report",
            needs_approval=False,
        ),
    ],
)
```

The key difference compared to handoffs: with **agents as tools** the orchestrator agent maintains control and composes the final response; with **handoff** the specialist takes control and responds directly to the user.

### 7.4 Evaluation Loop

An advanced pattern in which one agent generates content and another evaluates it in an iterative cycle:

```python
async def evaluation_loop(task: str, max_iterations: int = 3):
    generator = Agent(name="Generator", instructions="Genera contenuto di alta qualita.")
    evaluator = Agent(
        name="Evaluator",
        instructions="Valuta il contenuto. Rispondi APPROVED se soddisfa gli standard.",
    )

    content = await Runner.run(generator, task)
    for i in range(max_iterations):
        evaluation = await Runner.run(
            evaluator, f"Valuta:\n{content.final_output}"
        )
        if "APPROVED" in evaluation.final_output:
            return content.final_output
        content = await Runner.run(
            generator,
            f"Migliora basandoti su questo feedback:\n{evaluation.final_output}"
        )
    return content.final_output
```

---

## 8. Lifecycle Hooks

The SDK provides two classes of hooks for observing and reacting to lifecycle events:

### 8.1 RunHooks

They observe the entire `Runner.run()` invocation, including handoffs between agents. They are passed as the `hooks` parameter to the `run()` method:

```python
from agents import RunHooks, Runner

class MonitoringHooks(RunHooks):
    async def on_agent_start(self, context, agent):
        print(f"[START] Agent: {agent.name}")

    async def on_agent_end(self, context, agent, output):
        print(f"[END] Agent: {agent.name}, usage: {context.usage}")

    async def on_llm_start(self, context, agent):
        print(f"[LLM_START] {agent.name} is calling the model")

    async def on_llm_end(self, context, agent, response):
        print(f"[LLM_END] {agent.name}: {len(response.output)} output items")

    async def on_tool_start(self, context, agent, tool):
        print(f"[TOOL_START] {agent.name} -> {tool.name}")

    async def on_tool_end(self, context, agent, tool, result):
        print(f"[TOOL_END] {tool.name} completato")

    async def on_handoff(self, context, from_agent, to_agent):
        print(f"[HANDOFF] {from_agent.name} -> {to_agent.name}")

result = await Runner.run(agent, "Domanda", hooks=MonitoringHooks())
```

### 8.2 AgentHooks

Attached to a specific agent instance via the `hooks` parameter of the `Agent`. They receive callbacks only for events related to that agent:

```python
from agents import AgentHooks, Agent

class BillingHooks(AgentHooks):
    async def on_agent_start(self, context, agent):
        print(f"Billing agent avviato")

    async def on_tool_start(self, context, agent, tool):
        # Log specifico per il billing
        log_billing_tool_usage(tool.name)

billing_agent = Agent(
    name="Billing",
    instructions="Gestisci fatturazione",
    hooks=BillingHooks(),
)
```

### 8.3 Available Hooks

| Hook | Context | When |
|------|----------|--------|
| `on_agent_start` | `AgentHookContext` | Agent begins its execution |
| `on_agent_end` | `AgentHookContext` | Agent produces final output |
| `on_llm_start` | `RunContextWrapper` | Before each model call |
| `on_llm_end` | `RunContextWrapper` | After each model response |
| `on_tool_start` | `RunContextWrapper` | Before invocation of a local tool |
| `on_tool_end` | `RunContextWrapper` | After tool completion |
| `on_handoff` | `RunContextWrapper` | When control passes to another agent |

`RunHooks` are ideal for cross-cutting observability (logging, metrics, audit). `AgentHooks` are ideal for agent-specific logic (resource initialization, cleanup, domain metrics).

---

## 9. Deploy and Production

### 9.1 API Configuration

```python
from agents import set_default_openai_key, set_default_openai_client, set_default_openai_api

# API key da variabile d'ambiente (default: OPENAI_API_KEY)
set_default_openai_key("sk-...")

# Client custom (base_url per proxy, Azure, ecc.)
from openai import AsyncOpenAI
set_default_openai_client(AsyncOpenAI(base_url="https://proxy.example.com", api_key="..."))

# Switch da Responses API a Chat Completions API
set_default_openai_api("chat_completions")
```

### 9.2 Logging and Debug

```python
from agents import enable_verbose_stdout_logging
enable_verbose_stdout_logging()
```

The loggers are `openai.agents` and `openai.agents.tracing`. The environment variables `OPENAI_AGENTS_DONT_LOG_MODEL_DATA=1` and `OPENAI_AGENTS_DONT_LOG_TOOL_DATA=1` (default: active) protect sensitive data in logs.

### 9.3 Non-OpenAI Models

To use models from alternative providers while maintaining tracing toward OpenAI:

```python
from agents import set_tracing_export_api_key
from agents.extensions.models.any_llm_model import AnyLLMModel

set_tracing_export_api_key("sk-openai-key-for-tracing")

model = AnyLLMModel(
    model="anthropic/claude-3-opus",
    api_key="your-anthropic-key",
)

agent = Agent(name="Assistant", model=model, instructions="...")
```

### 9.4 Durable Execution

For long-running workflows that must survive crashes and restarts, the SDK integrates with durable execution frameworks:

- **Temporal**: orchestration of distributed workflows with automatic retry and state persistence.
- **Restate**: framework for resilient applications with automatic checkpoints.
- **DBOS**: database-based durable execution for Python workflows.

These integrations allow suspending and resuming agent runs in case of infrastructure failures.

### 9.5 Error Handling in Production

The runner supports `error_handlers` to handle error scenarios in a controlled manner:

```python
from agents import Runner, RunErrorHandlerResult

def handle_max_turns(context, error):
    return RunErrorHandlerResult(
        final_output="Mi scuso, non sono riuscito a completare la richiesta nei tempi previsti.",
        include_in_history=True,
    )

result = await Runner.run(
    agent,
    "Richiesta complessa",
    max_turns=10,
    error_handlers={"max_turns": handle_max_turns},
)
```

### 9.6 Sessions and Persistence

For applications in production, sessions automatically manage conversation history:

- **SQLite / SQLAlchemy**: local persistence for single-server applications.
- **Redis** (`openai-agents[redis]`): distributed sessions for multi-server architectures.
- **Encrypted sessions**: variants with encryption for sensitive data.

The `session_settings` configuration in `RunConfig` controls history retrieval limits, and `session_input_callback` allows custom logic for merging history with new inputs.

### 9.7 Human-in-the-Loop

For production scenarios where human approval is needed, the SDK supports interruptions and resumption:

```python
result = await Runner.run(agent, "Esegui operazione critica")

if result.interruptions:
    # Presenta all'utente le azioni pendenti
    state = result.to_state()
    # L'utente approva o rifiuta
    state.approve(item_id)
    # Riprendi l'esecuzione
    result = await Runner.run(agent, state)
```

---

## 10. Resources and References

### Official Repository and Documentation

- **GitHub Repository**: [github.com/openai/openai-agents-python](https://github.com/openai/openai-agents-python)
- **Documentation**: [openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/)
- **PyPI**: [pypi.org/project/openai-agents](https://pypi.org/project/openai-agents/)
- **License**: MIT

### Documentation Sections

| Section | URL |
|---------|-----|
| Quickstart | `openai.github.io/openai-agents-python/quickstart/` |
| Agents | `openai.github.io/openai-agents-python/agents/` |
| Running Agents | `openai.github.io/openai-agents-python/running_agents/` |
| Tools | `openai.github.io/openai-agents-python/tools/` |
| Handoffs | `openai.github.io/openai-agents-python/handoffs/` |
| Guardrails | `openai.github.io/openai-agents-python/guardrails/` |
| Tracing | `openai.github.io/openai-agents-python/tracing/` |
| MCP Integration | `openai.github.io/openai-agents-python/mcp/` |
| Multi-Agent | `openai.github.io/openai-agents-python/multi_agent/` |
| Streaming | `openai.github.io/openai-agents-python/streaming/` |
| Configuration | `openai.github.io/openai-agents-python/config/` |
| Sessions | `openai.github.io/openai-agents-python/sessions/` |

### Technical Specifications

- **Python**: >= 3.10
- **Current version**: v0.13.1 (March 2026)
- **Total releases**: 76
- **Core dependencies**: Pydantic, OpenAI Python SDK, MCP Python SDK
- **Optional dependencies**: SQLAlchemy, Redis, LiteLLM, websockets
- **Supported APIs**: OpenAI Responses API (default), Chat Completions API
- **Provider models**: Native OpenAI + 100+ LLMs via extensions

### External Integrations for Tracing

Weights & Biases (Weave), Arize-Phoenix, MLflow, Braintrust, Pydantic Logfire, AgentOps, LangSmith, Langfuse, Langtrace, PostHog, PromptLayer, and others.

### Integrations for Durable Execution

Temporal, Restate, DBOS.
