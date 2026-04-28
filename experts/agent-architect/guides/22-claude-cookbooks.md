# Anthropic Claude Cookbooks - Complete Guide

## 1. Overview

### What the Repository Is

The **Anthropic Cookbook** repository (https://github.com/anthropics/anthropic-cookbook) is the official collection of practical recipes, patterns, and reference implementations for building applications with Claude. With over 36,000 stars on GitHub, 500+ commits, and approximately 70 contributors, it represents the primary technical resource for developers working with Anthropic's APIs.

The cookbook is not theoretical documentation: every recipe is an executable Jupyter notebook (the repository is composed of approximately 96% `.ipynb` files) that contains working code, ready to be copied and adapted into one's own projects. The approach is pragmatic: starting from a concrete problem and arriving at an implemented, testable, and modifiable solution.

### Repository Structure

The repository is organized into thematic directories, each focused on a specific aspect of the Claude ecosystem:

```
anthropic-cookbook/
├── capabilities/             # Classification, RAG, Summarization
│   ├── classification/
│   ├── retrieval_augmented_generation/
│   └── summarization/
├── tool_use/                 # Tool calling, agent loops, compaction
│   ├── calculator_tool.ipynb
│   ├── customer_service_agent.ipynb
│   ├── tool_use_with_pydantic.ipynb
│   ├── tool_choice.ipynb
│   ├── parallel_tools.ipynb
│   ├── automatic-context-compaction.ipynb
│   ├── memory_cookbook.ipynb
│   ├── extracting_structured_json.ipynb
│   ├── tool_search_with_embeddings.ipynb
│   ├── vision_with_tools.ipynb
│   └── programmatic_tool_calling_ptc.ipynb
├── patterns/agents/          # Architectural patterns for agents
│   ├── basic_workflows.ipynb
│   ├── evaluator_optimizer.ipynb
│   ├── orchestrator_workers.ipynb
│   └── prompts/
├── skills/                   # Modular skills (Excel, PDF, Word, PPT)
│   ├── notebooks/
│   ├── custom_skills/
│   └── sample_data/
├── extended_thinking/        # Extended reasoning
├── multimodal/               # Vision, images, charts
├── third_party/              # Integrations (Pinecone, Voyage AI, Wikipedia)
├── misc/                     # Prompt caching, JSON mode, moderation
├── coding/                   # Code generation and analysis
├── finetuning/               # Model fine-tuning
├── observability/            # Monitoring and logging
└── tool_evaluation/          # Tool effectiveness evaluation
```

### How to Use It

**Prerequisites:**
- A Claude API key (free registration at anthropic.com)
- Python 3.10+ with Jupyter support
- The `anthropic` library installed (`pip install anthropic`)

**Recommended approach:**
1. Start with the [Claude API Fundamentals](https://github.com/anthropics/courses/tree/master/anthropic_api_fundamentals) course if you are new to the APIs
2. Identify the thematic directory pertinent to your use case
3. Open the corresponding notebook and run it cell by cell
4. Adapt the code to your context, replacing example data with real data

---

## 2. Tool Use Patterns

### Defining Tools

The heart of tool use in Claude is the definition schema. Each tool is described as a JSON dictionary with three fundamental fields: `name`, `description`, and `input_schema`. The input schema follows the JSON Schema format, with support for types, constraints, descriptions, and required fields.

```python
tools = [
    {
        "name": "get_customer_info",
        "description": "Retrieves customer information by ID. "
                       "Returns name, email, and phone.",
        "input_schema": {
            "type": "object",
            "properties": {
                "customer_id": {
                    "type": "string",
                    "description": "Unique identifier of the customer."
                }
            },
            "required": ["customer_id"]
        }
    },
    {
        "name": "get_order_details",
        "description": "Retrieves the details of a specific order.",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "Unique identifier of the order."
                }
            },
            "required": ["order_id"]
        }
    },
    {
        "name": "cancel_order",
        "description": "Cancels an order. Returns confirmation of the cancellation.",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "ID of the order to cancel."
                }
            },
            "required": ["order_id"]
        }
    }
]
```

### The Agent Loop

The fundamental pattern for Claude agents is the tool use cycle. The model receives a message, decides whether to invoke a tool, and the loop continues until the model terminates with `"end_turn"` instead of `"tool_use"`:

```python
def agent_loop(user_message):
    messages = [{"role": "user", "content": user_message}]

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        tools=tools,
        messages=messages
    )

    # Loop while Claude requests tools
    while response.stop_reason == "tool_use":
        tool_use = next(
            block for block in response.content
            if block.type == "tool_use"
        )
        tool_name = tool_use.name
        tool_input = tool_use.input

        # Client-side execution
        tool_result = process_tool_call(tool_name, tool_input)

        # Update message history
        messages = [
            {"role": "user", "content": user_message},
            {"role": "assistant", "content": response.content},
            {
                "role": "user",
                "content": [{
                    "type": "tool_result",
                    "tool_use_id": tool_use.id,
                    "content": str(tool_result)
                }]
            }
        ]

        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

    # Extract final text response
    final = next(
        (block.text for block in response.content if hasattr(block, "text")),
        None
    )
    return final
```

### Tool Selection Control with tool_choice

The `tool_choice` parameter allows control over selection behavior:

- **`auto`** (default): Claude autonomously decides whether and which tool to invoke. Ideal when you want the model to evaluate whether a tool is necessary based on context.

- **`tool`** with specific name: forces Claude to always use a particular tool. Useful for ensuring structured output, for example in sentiment analysis pipelines where every input must produce a JSON score.

- **`any`**: requires Claude to use at least one tool, but leaves the choice of which one. Suitable for scenarios like SMS chatbots where every interaction must pass through a communication tool.

```python
# Force the use of a specific tool
response = client.messages.create(
    model="claude-sonnet-4-6",
    tools=tools,
    tool_choice={"type": "tool", "name": "print_sentiment_scores"},
    messages=messages
)
```

### Validation with Pydantic

The cookbook shows how to use Pydantic models to validate tool inputs before execution, adding a critical layer of safety for production applications:

```python
from pydantic import BaseModel, EmailStr, Field

class Author(BaseModel):
    name: str
    email: EmailStr

class Note(BaseModel):
    note: str
    author: Author
    tags: list[str] | None = None
    priority: int = Field(ge=1, le=5, default=3)
    is_public: bool = False

def process_tool_call(tool_name, tool_input):
    if tool_name == "save_note":
        # Pydantic validation raises exceptions if data is not valid
        note = Note(
            note=tool_input["note"],
            author=Author(
                name=tool_input["author"]["name"],
                email=tool_input["author"]["email"]
            ),
            priority=tool_input.get("priority", 3),
            is_public=tool_input.get("is_public", False)
        )
        return save_note(note)
```

### Best Practices for Tool Use

1. **Detailed descriptions**: tool descriptions are the primary factor that drives Claude's selection. They must be precise, with examples of when to use and when not to use the tool.
2. **Rigorous schemas**: always define `required` fields, use `description` for every parameter, specify constraints (`minimum`, `maximum`, `enum`).
3. **Elaborate system prompts**: when using `tool_choice: auto`, a detailed system prompt helps Claude not be too aggressive in invoking tools.
4. **Client-side validation**: use Pydantic or schema validation to verify inputs before executing the tool.
5. **Graceful error handling**: return clear error messages as `tool_result` instead of crashing the loop.

---

## 3. Agent Skills

### The Modular Skill System

Skills are modular components that extend Claude's capabilities in a structured way. The cookbook documents four built-in skills and a framework for creating custom ones.

**Built-in Skills:**

| Skill | ID | Functionality |
|-------|-----|-------------|
| Excel | `xlsx` | Creates workbooks with multiple sheets, formulas, charts, pivot tables |
| PowerPoint | `pptx` | Generates presentations with layouts, charts, transitions |
| PDF | `pdf` | Creates formatted documents with tables and images |
| Word | `docx` | Generates documents with rich formatting |

### Progressive Disclosure Architecture

Skills load dynamically only when needed, optimizing token usage. This pattern is crucial because it avoids consuming context window with instructions for skills that will not be used:

```python
client = Anthropic(api_key="your-api-key")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": "Create an Excel file with the quarterly budget"
    }]
)
```

### Creating Custom Skills

Custom skills follow a standardized structure:

```
my_custom_skill/
├── SKILL.md           # Required: instructions for Claude
├── scripts/           # Optional: Python/JavaScript code
│   └── processor.py
└── resources/         # Optional: templates, reference data
    └── template.xlsx
```

The `SKILL.md` file is the heart of the skill: it contains the instructions Claude will follow to execute the task. It is a Markdown document that describes the capabilities, constraints, expected output format, and usage examples.

### File Generation Workflow

Skills produce files that are returned via the Files API:

```python
# 1. The skill generates the file internally
# 2. The response contains a file_id
# 3. Download the generated file
content = client.beta.files.download(file_id="file_abc123...")
with open("output.xlsx", "wb") as f:
    f.write(content.read())
```

The cookbook includes three progressive notebooks in the `skills/notebooks/` directory that guide from basic skill use, through financial applications, to creating custom skills.

---

## 4. Multi-Step Workflows

### Fundamental Patterns

The cookbook documents five architectural patterns for orchestrating complex workflows, based on Anthropic's "Building Effective Agents" research (Erik Schluntz and Barry Zhang). Each pattern is implemented as an executable notebook with reference code.

### Prompt Chaining (Sequential Concatenation)

Decomposes a task into sequential subtasks where each step's output feeds into the next:

```python
def chain(input: str, prompts: list[str]) -> str:
    """Concatenates sequential LLM calls, passing results between steps."""
    result = input
    for i, prompt in enumerate(prompts, 1):
        result = llm_call(f"{prompt}\nInput: {result}")
    return result
```

**Typical use case**: multi-phase data transformation (extracting numerical values, conversion to percentages, sorting, formatting as a markdown table).

### Parallelization

Distributes independent subtasks across multiple concurrent LLM calls:

```python
from concurrent.futures import ThreadPoolExecutor

def parallel(prompt: str, inputs: list[str], n_workers: int = 3) -> list[str]:
    """Processes multiple inputs concurrently with the same prompt."""
    with ThreadPoolExecutor(max_workers=n_workers) as executor:
        futures = [
            executor.submit(llm_call, f"{prompt}\nInput: {x}")
            for x in inputs
        ]
        return [f.result() for f in futures]
```

**Typical use case**: impact analysis on multiple stakeholders (customers, employees, investors, suppliers) executed simultaneously.

### Routing (Dynamic Routing)

Classifies the input and directs it toward specialized paths:

```python
def route(input: str, routes: dict[str, str]) -> str:
    """Routes input to a specialized prompt."""
    selector_prompt = f"""
    Analyze the input and select the most appropriate team from: {list(routes.keys())}

    <reasoning>Brief explanation of the routing.</reasoning>
    <selection>Name of the chosen team</selection>
    """
    route_response = llm_call(selector_prompt)
    route_key = extract_xml(route_response, "selection").strip().lower()
    selected_prompt = routes[route_key]
    return llm_call(f"{selected_prompt}\nInput: {input}")
```

**Typical use case**: customer service ticketing with routing to billing, technical, account, or product.

### Orchestrator-Workers

A central orchestrator analyzes the task and dynamically delegates to specialized workers. Unlike static parallelization, the orchestrator decides at runtime which subtasks to create:

```python
class FlexibleOrchestrator:
    def __init__(self, orchestrator_prompt, worker_prompt, model="claude-sonnet-4-6"):
        self.orchestrator_prompt = orchestrator_prompt
        self.worker_prompt = worker_prompt
        self.model = model

    def process(self, task: str, context: dict = None) -> dict:
        context = context or {}

        # Phase 1: the orchestrator analyzes and decomposes
        orchestrator_input = self._format_prompt(
            self.orchestrator_prompt, task=task, **context
        )
        orchestrator_response = llm_call(orchestrator_input, model=self.model)
        analysis = extract_xml(orchestrator_response, "analysis")
        tasks = parse_tasks(extract_xml(orchestrator_response, "tasks"))

        # Phase 2: each worker executes its subtask
        worker_results = []
        for task_info in tasks:
            worker_input = self._format_prompt(
                self.worker_prompt,
                original_task=task,
                task_type=task_info["type"],
                task_description=task_info["description"],
                **context
            )
            worker_response = llm_call(worker_input, model=self.model)
            worker_content = extract_xml(worker_response, "response")

            if not worker_content or not worker_content.strip():
                worker_content = f"[Error: Worker '{task_info['type']}' did not generate content]"

            worker_results.append({
                "type": task_info["type"],
                "description": task_info["description"],
                "result": worker_content
            })

        return {"analysis": analysis, "worker_results": worker_results}
```

The orchestrator uses a structured XML format to communicate the subtasks:

```xml
<analysis>
Explanation of the task decomposition.
</analysis>

<tasks>
  <task>
    <type>technical-specs</type>
    <description>Technical version with detailed specifications</description>
  </task>
  <task>
    <type>lifestyle-emotional</type>
    <description>Narrative version connecting to audience values</description>
  </task>
</tasks>
```

### Evaluator-Optimizer

An iterative cycle where one LLM generates a solution and another evaluates it, providing feedback for progressive improvement:

```python
def loop(task, evaluator_prompt, generator_prompt):
    memory = []
    chain_of_thought = []

    # Initial generation
    thoughts, result = generate(generator_prompt, task)
    memory.append(result)
    chain_of_thought.append({"thoughts": thoughts, "result": result})

    while True:
        evaluation, feedback = evaluate(evaluator_prompt, result, task)
        if evaluation == "PASS":
            return result, chain_of_thought

        # Context with previous attempts and feedback
        context = "\n".join([
            "Previous attempts:",
            *[f"- {m}" for m in memory],
            f"\nFeedback: {feedback}"
        ])

        thoughts, result = generate(generator_prompt, task, context)
        memory.append(result)
        chain_of_thought.append({"thoughts": thoughts, "result": result})
```

**Typical use case**: code generation and refinement. The cookbook demonstrates this pattern with the implementation of a `MinStack` with O(1) operations, where the evaluator identifies gaps in error handling, type hints, and documentation, and the generator produces iteratively improved versions.

### Pattern Comparison Table

| Pattern | Trade-off | When to Use |
|---------|------------|---------------|
| Prompt Chaining | Cost vs. Quality | Multi-phase transformations with sequential dependencies |
| Parallelization | Latency vs. Cost | Independent analyses on multiple inputs |
| Routing | Complexity vs. Precision | Context-dependent task management |
| Orchestrator-Workers | Flexibility vs. Overhead | Tasks requiring dynamic decomposition |
| Evaluator-Optimizer | Iterations vs. Quality | Progressive refinement with clear criteria |

---

## 5. Automatic Compaction

### The Context Window Problem

Long conversations and multi-step agentic workflows accumulate tokens rapidly. With a 200K token context window limit, an agent processing dozens of entities can run out of space before completing the work. Automatic compaction solves this problem by compressing the conversation history when token consumption exceeds a configurable threshold.

### How It Works

The compaction process follows five phases:

1. **Monitoring**: tracks tokens consumed at each turn
2. **Threshold detection**: when tokens exceed the configured threshold, compaction is triggered
3. **Summary generation**: a summary prompt is injected as a user turn, asking Claude to create a structured summary
4. **Tag wrapping**: Claude generates the summary wrapped in `<summary></summary>` tags
5. **Reset and resume**: the history is cleared, keeping only the summary, and the conversation continues with compressed context

### Configuration

```python
runner = client.beta.messages.tool_runner(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    tools=tools,
    messages=messages,
    compaction_control={
        "enabled": True,
        "context_token_threshold": 5000,
        # Optional: different model for summaries
        # "model": "claude-haiku-4-5",
        # Optional: custom prompt
        # "summary_prompt": "Custom summary prompt"
    }
)
```

### Measured Results

The cookbook demonstrates compaction with a customer service workflow that processes 5 support tickets. Each ticket goes through 7 steps (fetch, classify, research, prioritize, route, draft, complete). The measured results are significant:

**Without compaction:**
- Input tokens: 204,416
- Output tokens: 4,422
- Total: 208,838 tokens
- 37 turns

**With compaction (5,000 token threshold):**
- Input tokens: 82,171
- Output tokens: 4,275
- Total: 86,446 tokens
- 26 turns
- 2 compaction events

**Result: 58.6% reduction in tokens (122,392 tokens saved).**

### Example of Compacted Output

When compaction is triggered, the generated summary has this structure:

```
<summary>
## Ticket Processing Progress Status

### Overview
Sequential processing of 5 support tickets, completing all 7 steps for each.

### Completed Tickets (2 of 5)

**TICKET-1 (Chris Davis) - COMPLETED**
- Issue: Account locked, unlock link not working
- Category: account
- Priority: high
- Team: account-services
- Status: resolved

**TICKET-2 (Chris Williams) - COMPLETED**
- Issue: Unrecognized $49.99 charge
- Category: billing
- Priority: high
- Team: billing-team
- Status: resolved

### Current Status
**TICKET-3 (John Jones) - IN PROGRESS**
- Category: product
- Steps completed: 1 (fetch), 2 (classify)
- Steps remaining: 3-7

### Next Steps
1. Complete TICKET-3
2. Fetch and process TICKET-4
3. Fetch and process TICKET-5
</summary>
```

### Guidelines for Thresholds

| Threshold | Token Range | Use Case | Compaction Frequency |
|--------|-------------|------------|---------------------|
| Low | 5K-20K | Iterative processing with clear boundaries | High |
| Medium | 50K-100K | Multi-phase workflows | Balanced |
| High | 100K-150K | Tasks requiring extended historical context | Low |
| Default | 100K | Good general balance | Moderate |

### Manual Compaction for Chat Applications

For user conversations without tools, the cookbook shows the manual implementation:

```python
COMPACTION_THRESHOLD = 3000
SUMMARY_PROMPT = """Write a continuation summary that allows resuming the work.
Include: Task Overview, Current Status, Important Findings, Next Steps, Context to Preserve."""

messages = []

for user_input in user_inputs:
    messages.append({"role": "user", "content": user_input})
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=messages
    )
    messages.append({"role": "assistant", "content": response.content})

    total_tokens = response.usage.input_tokens + response.usage.output_tokens

    if total_tokens > COMPACTION_THRESHOLD:
        summary_response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            messages=messages + [{"role": "user", "content": SUMMARY_PROMPT}]
        )
        summary_text = summary_response.content[0].text
        messages = [{"role": "user", "content": summary_text}]
```

### When to Use (and Not Use) Compaction

**Ideal for:** sequential processing (ticket queues, batch operations), multi-phase workflows with natural checkpoints, extended analysis sessions, batch operations on independent entities.

**To avoid for:** short tasks completing within 50K-100K tokens, tasks requiring complete audit trails of every detail, server-side sampling loops (web search, server-side Extended Thinking) due to cache token accumulation.

---

## 6. Advanced Patterns

### 6.1 Structured Output and JSON Mode

Claude does not have a native "JSON Mode" with constrained sampling, but the cookbook documents three reliable techniques to obtain consistent JSON output.

**Technique 1 - Parsing with delimiters:**
```python
def extract_json(response):
    json_start = response.index("{")
    json_end = response.rfind("}")
    return json.loads(response[json_start : json_end + 1])
```

**Technique 2 - Prefilling the assistant response:**
```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Return a JSON with the athletes and their sports."},
        {"role": "assistant", "content": "Here is the requested JSON:\n{"}
    ]
)
# Reconstruction by adding the initial '{'
output = json.loads("{" + message[: message.rfind("}") + 1])
```

**Technique 3 - Wrapping with XML tags for complex outputs:**
```python
import re

def extract_between_tags(tag, string, strip=False):
    ext_list = re.findall(f"<{tag}>(.+?)</{tag}>", string, re.DOTALL)
    if strip:
        ext_list = [e.strip() for e in ext_list]
    return ext_list

# Parsing responses with multiple JSONs
data = json.loads(extract_between_tags("athlete_sports", message)[0])
```

### 6.2 Streaming Responses

The cookbook covers response streaming to reduce user-perceived latency. With streaming, tokens are sent as they are generated, allowing the response to be displayed progressively. This is particularly important for agents that produce long responses or perform extended reasoning.

The Python SDK supports streaming via the `stream()` method:

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

When combining streaming with tool use, the flow becomes more complex because both text blocks and tool_use blocks must be handled as they arrive. The recommended pattern is to accumulate the tool_use blocks, execute them, and then resume streaming with the results.

### 6.3 Error Recovery

The cookbook adopts several error recovery strategies in the agent loop:

**Tool level**: tool functions return descriptive error messages instead of exceptions. A `get_customer_info` tool that does not find the customer returns `"Customer not found"` as a string, allowing Claude to handle the error in the conversation.

**Validation level**: with Pydantic, validation errors are caught before tool execution, avoiding side effects from malformed inputs.

**Worker level (Orchestrator-Workers)**: if a worker fails to generate content, the pattern inserts an explicit error message:

```python
if not worker_content or not worker_content.strip():
    worker_content = f"[Error: Worker '{task_info['type']}' did not generate content]"
```

**Loop level (Evaluator-Optimizer)**: the pattern maintains a `memory` of all previous attempts, providing complete context to the generator to avoid repeating errors.

### 6.4 Parallel Tool Calls

Claude tends to make sequential tool calls by default. The cookbook introduces a `batch_tool` pattern that acts as a meta-tool to encourage parallel execution:

```python
batch_tool = {
    "name": "batch_tool",
    "description": "Invokes multiple tool calls simultaneously.",
    "input_schema": {
        "type": "object",
        "properties": {
            "invocations": {
                "type": "array",
                "description": "The tool calls to invoke.",
                "items": {
                    "type": "object",
                    "properties": {
                        "name": {
                            "type": "string",
                            "description": "Name of the tool to invoke"
                        },
                        "arguments": {
                            "type": "string",
                            "description": "Tool arguments in JSON format"
                        }
                    },
                    "required": ["name", "arguments"]
                }
            }
        },
        "required": ["invocations"]
    }
}
```

The batch tool processor iterates over the invocations and executes them all:

```python
def process_tool_with_maybe_batch(tool_name, tool_input):
    if tool_name == "batch_tool":
        results = []
        for invocation in tool_input["invocations"]:
            results.append(
                process_tool_call(
                    invocation["name"],
                    json.loads(invocation["arguments"])
                )
            )
        return "\n".join(results)
    else:
        return process_tool_call(tool_name, tool_input)
```

The mere presence of the `batch_tool` in the list of available tools encourages Claude to group parallel calls, reducing round-trips and overall latency.

### 6.5 Prompt Caching

Prompt caching reduces latency by more than 2x and costs by up to 90% for repetitive tasks, by storing and reusing context in prompts.

**Automatic caching (recommended):**
```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=300,
    cache_control={"type": "ephemeral"},
    messages=[{"role": "user", "content": prompt_with_large_context}]
)
```

**Explicit breakpoints (granular control):**
```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=300,
    system=[{
        "type": "text",
        "text": system_message,
        "cache_control": {"type": "ephemeral"}
    }],
    messages=[{
        "role": "user",
        "content": [{
            "type": "text",
            "text": user_query,
            "cache_control": {"type": "ephemeral"}
        }]
    }]
)
```

**Measured results from the cookbook (benchmark with text of ~187K tokens):**

| Scenario | Time | Input Tokens | Cache Write | Cache Read |
|----------|-------|-------------|-------------|------------|
| Without cache | 4.89s | 187,364 | - | - |
| First call (write) | 4.28s | 3 | 187,361 | - |
| Cache hit | 1.48s | 3 | - | 187,361 |

**Speedup: 3.3x faster** on cache hits. In multi-turn conversations, after the first turn approximately 100% of input tokens are read from cache.

**Technical details:** minimum cacheable length: 1,024 tokens (Sonnet), 4,096 tokens (Opus/Haiku 4.5). TTL: 5 minutes default (renewed at every hit), 1-hour option at twice the cost. Maximum 4 explicit breakpoints per request.

### 6.6 Extended Thinking

The `extended_thinking/` directory contains two notebooks documenting Claude's extended reasoning:

- **extended_thinking.ipynb**: fundamental patterns for enabling deep reasoning on complex problems, analysis, and logical deductions.
- **extended_thinking_with_tool_use.ipynb**: combination of extended reasoning with tool calling, allowing Claude to reason about when and how to use external tools before invoking them.

These patterns are particularly effective for code generation and analysis, mathematical reasoning, and agentic systems that benefit from an internal planning phase.

### 6.7 Tool Search with Embeddings

When the number of available tools is very large, passing all of them in the API request is inefficient and consumes context window. The cookbook proposes two approaches:

**Semantic search with embeddings**: embeddings are generated for tool descriptions, and given the user message, only the relevant tools are dynamically selected via cosine similarity. This allows scaling to hundreds of tools while keeping token consumption low.

**Alternative approaches**: the `tool_search_alternate_approaches.ipynb` notebook explores different strategies such as hierarchical tool categorization and keyword matching-based selection.

---

## 7. Practical Recipes for Agent Builders

### 7.1 Customer Service Agent

**File**: `tool_use/customer_service_agent.ipynb`

The most complete recipe for those building conversational agents. It implements a customer support agent with three tools (retrieve customer info, order details, cancel order) and a complete agent loop. The pattern shows how Claude autonomously decides which tools to invoke based on the user's request, handles multi-step (first retrieves info, then executes the action), and produces a natural response that integrates the tool results.

### 7.2 Programmatic Tool Calling (PTC)

**File**: `tool_use/programmatic_tool_calling_ptc.ipynb`

Extends the agent loop pattern with programmatic control logic. Instead of leaving Claude full control of the flow, the client code implements guardrails and decision logic that govern when and how tools are invoked. Fundamental for production applications where deterministic control is needed.

### 7.3 Memory Cookbook

**File**: `tool_use/memory_cookbook.ipynb` and `tool_use/memory_tool.py`

Implements persistent memory patterns for stateful agents. Includes a reusable memory tool that allows Claude to save and retrieve information between sessions, crucial for agents that must maintain context across previous interactions.

### 7.4 Vision with Tool Use

**File**: `tool_use/vision_with_tools.ipynb`

Combines Claude's vision capabilities with tool use, allowing image processing followed by execution of actions based on visual analysis. Useful for document processing workflows, data extraction from screenshots, and automated visual content analysis.

### 7.5 Structured Data Extraction

**File**: `tool_use/extracting_structured_json.ipynb`

Uses tools as a mechanism to enforce structured output. Instead of asking Claude to "return JSON", a tool is defined whose input schema corresponds to the desired data structure, and Claude is forced to use it with `tool_choice`. This guarantees consistent and validatable output.

### 7.6 RAG (Retrieval Augmented Generation)

**File**: `capabilities/retrieval_augmented_generation/`

Complete patterns for augmenting Claude with external knowledge. Includes integrations with Pinecone for vector databases, Wikipedia search, and web page reading. Fundamental for agents needing to access updated or domain-specific knowledge not present in Claude's training.

### 7.7 Sub-Agents

**File**: `multimodal/using_sub_agents.ipynb`

Demonstrates how to use lighter models (Haiku) as sub-agents of more powerful models (Opus), optimizing the cost-quality ratio. The main model delegates simple or pre-processing tasks to the sub-agent, reserving its own capacity for complex decisions.

### 7.8 Building Evaluations

**File**: `misc/building_evals.ipynb`

Framework to automate the evaluation of prompt and response quality. Essential for those building production agents who need objective metrics to iterate on prompts and measure regressions.

### 7.9 Moderation Filter

**File**: `misc/building_moderation_filter.ipynb`

Implements a content moderation filter using Claude as a classifier. Critical pattern for consumer-facing applications where inappropriate content must be filtered before being returned to the user.

---

## 8. Resources and References

### Repository and Code

- **Anthropic Cookbook**: https://github.com/anthropics/anthropic-cookbook
- **Anthropic Courses**: https://github.com/anthropics/courses
- **Claude API Fundamentals**: https://github.com/anthropics/courses/tree/master/anthropic_api_fundamentals
- **Anthropic on AWS**: https://github.com/aws-samples/anthropic-on-aws

### Official Documentation

- **Developer Documentation**: https://docs.anthropic.com
- **API Reference**: https://docs.anthropic.com/en/api
- **Support Docs**: https://support.anthropic.com

### Reference Research

- **Building Effective Agents** (Erik Schluntz, Barry Zhang) - The foundational paper on agentic patterns that inspired the `patterns/agents/` directory. Available at https://anthropic.com/research/building-effective-agents

### Community

- **Anthropic Discord**: https://www.anthropic.com/discord
- **GitHub Issues**: https://github.com/anthropics/anthropic-cookbook/issues - To report bugs, propose new recipes, or contribute

### Integrations Documented in the Cookbook

| Service | Type | Notebook |
|----------|------|----------|
| Pinecone | Vector Database | `third_party/Pinecone/rag_using_pinecone.ipynb` |
| Voyage AI | Embeddings | `third_party/VoyageAI/how_to_create_embeddings.md` |
| Wikipedia | Knowledge Base | `third_party/Wikipedia/wikipedia-search-cookbook.ipynb` |
| Stable Diffusion | Image Generation | `misc/illustrated_responses.ipynb` |

### Version Notes

The repository is actively maintained. Recipes are updated to support the latest versions of Claude models (Opus 4, Sonnet 4, Haiku 4.5) and the new API features such as prompt caching, extended thinking, and the skill system. Always check the date of the most recent commit of the specific notebook to make sure the code is up to date with the current version of the SDK.
