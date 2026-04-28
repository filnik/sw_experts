# AutoGen -- Microsoft's Conversational Multi-Agent Framework

## 1. Overview

### Origins and Microsoft Research

AutoGen was born in 2023 as an open-source project of **Microsoft Research**, conceived to simplify the creation of multi-agent applications based on Large Language Models. The founding paper, published by a team led by Qingyun Wu and Chi Wang, introduces a revolutionary paradigm: agents that collaborate through **multi-turn conversations** to solve complex tasks, combining the power of LLMs with code execution and human intervention.

AutoGen's central concept is radically different from other frameworks of the time: instead of defining rigid workflows or sequential chains, agents communicate naturally through conversational messages. This approach proves to be extraordinarily flexible, allowing modeling of scenarios ranging from collaborative code generation to mathematical problem solving, from creative writing to data analysis.

The project achieved immediate success in the community: with over **56,000 GitHub stars**, 3,700+ commits, and an active community on Discord, AutoGen has rapidly established itself as one of the most influential multi-agent frameworks in the AI ecosystem.

### Evolution and the AG2 Split

AutoGen's recent history is marked by a significant event. In **September 2024**, some of the original creators -- including Chi Wang and Qingyun Wu -- left Microsoft and created a fork of the project. In **November 2024**, this fork was formalized with the creation of the **AG2AI** organization on GitHub (repository `ag2ai/ag2`), with the name "AG2" (AutoGen 2.0).

The resulting situation is complex and has generated confusion in the community. Today there are **four distinct versions**:

1. **AutoGen 0.4 (Microsoft)**: Complete rewrite with event-driven architecture based on the actor model. PyPI packages: `autogen-agentchat`, `autogen-core`, `autogen-ext`.
2. **AutoGen 0.2 (Legacy)**: The original synchronous architecture, maintained in the `0.2` branch of the Microsoft repository. Package: `autogen-agentchat~=0.2`.
3. **AG2 (Independent fork)**: Managed by the community with open governance, maintains compatibility with v0.2 without breaking changes. PyPI packages: `ag2`, `autogen`, `pyautogen`.
4. **Semantic Kernel Integration**: Microsoft plans the integration of the AutoGen 0.4 runtime into Semantic Kernel, offering enterprise commercial support.

This document focuses primarily on **AutoGen 0.4 (Microsoft)**, the most recent and architecturally advanced version, with references to v0.2 where necessary for historical context.

---

## 2. Conversational Architecture

### The Multi-Agent Conversation Paradigm

AutoGen's fundamental insight is that many complex tasks can be broken down into **conversations between specialized agents**. Instead of a single monolithic agent that tries to do everything, AutoGen orchestrates a dialogue between multiple agents, each with specific competencies, tools, and roles.

This paradigm is inspired by how human teams collaborate: a programmer writes the code, a reviewer checks it, a project manager coordinates the work. In AutoGen, each role becomes an autonomous agent that participates in a structured conversation.

### Layered Architecture (v0.4)

AutoGen 0.4 introduces a **three-layer** architecture fundamentally different from v0.2:

```
+--------------------------------------------------+
|                AutoGen Studio                     |
|        (No-code GUI for prototyping)              |
+--------------------------------------------------+
|               AgentChat API                       |
|   (High-level task-driven API:                    |
|    AssistantAgent, Team, GroupChat)                |
+--------------------------------------------------+
|                  Core API                          |
|   (Event-driven, actor model, local               |
|    and distributed runtime, cross-language)       |
+--------------------------------------------------+
|               Extensions API                      |
|   (LLM clients, code executors, MCP,              |
|    third-party integrations)                      |
+--------------------------------------------------+
```

**Core API**: Represents the foundation of the system. It manages **message passing** between agents, implements an **event-driven** architecture based on the **actor model**, and supports both local and distributed runtimes. Cross-language support includes Python and .NET, with additional languages in development.

**AgentChat API**: It is the layer most used by developers. It provides an opinionated, high-level interface for rapid prototyping, supporting common patterns such as two-agent chat, group chat, and team orchestration. It maintains a level of abstraction similar to v0.2, facilitating migration.

**Extensions API**: Enables first-party and third-party integrations. It includes implementations of LLM clients (OpenAI, Azure OpenAI), code executors (Docker, Azure Container Apps), MCP integration (Model Context Protocol), and adapters for LangChain tools.

### Actor Model and Asynchronous Communication

The choice of the **actor model** as the architectural foundation is one of the most significant decisions of v0.4. In the actor model, each agent is an independent actor that:

- Owns its internal state
- Communicates exclusively through **asynchronous messages**
- Can create other actors
- Can decide how to respond to received messages

This architecture enables:
- **Distribution**: agents can run on different machines
- **Scalability**: the system scales naturally with the number of agents
- **Fault tolerance**: the failure of an agent does not compromise the entire system
- **Observability**: centralized communication via messages facilitates debugging and tracing

---

## 3. Core Concepts

### 3.1 ConversableAgent (v0.2)

The `ConversableAgent` is the fundamental building block of AutoGen's v0.2 architecture. It is a generic class that represents an agent capable of participating in conversations, sending and receiving messages, and optionally generating automatic responses.

```python
# v0.2 - ConversableAgent
from autogen import ConversableAgent

agent = ConversableAgent(
    name="my_agent",
    llm_config={"config_list": [{"model": "gpt-4", "api_key": "..."}]},
    system_message="You are an expert assistant.",
    human_input_mode="NEVER",  # ALWAYS, TERMINATE, NEVER
    max_consecutive_auto_reply=10,
    code_execution_config=False,
)
```

The key properties of `ConversableAgent` include:
- **`llm_config`**: configuration of the LLM model to use
- **`system_message`**: the system prompt that defines the agent's role
- **`human_input_mode`**: controls when to request human input (ALWAYS, TERMINATE, NEVER)
- **`code_execution_config`**: configures code execution (Docker, local, disabled)
- **`register_reply()`**: method for registering custom response logic

### 3.2 AssistantAgent

The `AssistantAgent` is an agent preconfigured to act as an AI assistant. In v0.2 it is a specialization of `ConversableAgent` with default settings oriented toward generating responses via LLM.

In **v0.4** the AssistantAgent has undergone significant changes:

```python
# v0.4 - AssistantAgent
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    tools=[my_function_tool],
    system_message="You are an expert assistant in data analysis.",
    reflect_on_tool_use=True,
)

# Asynchronous execution
result = await agent.on_messages(messages, cancellation_token)
```

The main differences between v0.2 and v0.4:

| Aspect | v0.2 | v0.4 |
|---------|------|------|
| Model configuration | `llm_config` (dictionary) | `model_client` (typed object) |
| API calls | Synchronous (`send()`) | Asynchronous (`on_messages()`, `on_messages_stream()`) |
| Response format | Direct messages | Wrapped `Response` object |
| Tool registration | `register_function()` on two agents | `tools=[]` parameter directly |
| Model fallback | List of configs tried in sequence | Single explicit configuration |

### 3.3 UserProxyAgent

The `UserProxyAgent` is the agent that represents the human user in the system. In v0.2 it is a complex agent with many configurations; in v0.4 it has been drastically simplified.

```python
# v0.2 - Complex UserProxyAgent
from autogen import UserProxyAgent

user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="TERMINATE",
    max_consecutive_auto_reply=10,
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={
        "work_dir": "coding",
        "use_docker": True,
    },
)

# v0.4 - Simplified UserProxyAgent
from autogen_agentchat.agents import UserProxyAgent

user_proxy = UserProxyAgent("user_proxy")
# In v0.4, a user proxy is simply an agent
# that accepts input from the user, without special configuration
```

This simplification in v0.4 reflects a clearer separation of responsibilities: code execution and tool management are now the responsibility of the `AssistantAgent` or dedicated components, not the user proxy.

### 3.4 GroupChat and GroupChatManager

The `GroupChat` is the mechanism for orchestrating conversations with **more than two agents**. All agents share a single conversation thread and the same context.

**v0.2 - Classic GroupChat**:

```python
from autogen import GroupChat, GroupChatManager

groupchat = GroupChat(
    agents=[user_proxy, coder, critic],
    messages=[],
    max_round=12,
    speaker_selection_method="auto",  # auto, manual, round_robin, random
)

manager = GroupChatManager(
    groupchat=groupchat,
    llm_config=llm_config,
)

user_proxy.initiate_chat(manager, message="Write a sorting algorithm.")
```

The `GroupChatManager` in v0.2 is a special agent that:
- Selects the next speaker at each turn (via LLM or specific method)
- Manages the conversation flow
- Maintains the shared message history
- Supports selection methods: `auto`, `manual`, `round_robin`, `random`

**v0.4 - Simplified Teams**:

```python
from autogen_agentchat.teams import RoundRobinGroupChat, SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# Round Robin: sequential rotation
team = RoundRobinGroupChat(
    participants=[coder, reviewer, tester],
    termination_condition=TextMentionTermination("APPROVE"),
)

# Selector: LLM-based selection
team = SelectorGroupChat(
    participants=[coder, reviewer, tester],
    model_client=model_client,
    termination_condition=TextMentionTermination("DONE"),
)

result = await team.run(task="Implement a REST API in FastAPI.")
```

v0.4 eliminates the need for an explicit `GroupChatManager` and an initiator user proxy, replacing them with more intuitive `Team` implementations.

---

## 4. AutoGen 0.4 -- New Architecture

### Motivations for the Rewrite

AutoGen 0.4 is a **complete rewrite** (from-the-ground-up rewrite) motivated by community feedback after a year of v0.2 development. The main problems identified were:

- **Difficult debugging**: synchronous communication between agents made tracing problems complex
- **Limited scalability**: the chat-centric architecture did not support distributed deployments
- **Inefficient API**: rapid growth had led to an inconsistent and hard-to-extend API
- **Insufficient observability**: lack of tools to monitor and control interactions
- **Rigid collaboration patterns**: difficulty in defining flexible workflows

### Main Breaking Changes

The migration from v0.2 to v0.4 entails numerous compatibility breaks:

**Asynchronous model**: all operations in v0.4 are async-first. Every interaction with agents requires `async/await`. This is a paradigmatic change that impacts the entire codebase.

**Model configuration**: v0.2 used `OpenAIWrapper` with `config_list` containing multiple configurations tried in sequence. v0.4 requires a single explicit configuration via `OpenAIChatCompletionClient` or the component config system.

**Custom agents**: in v0.2 you used `register_reply()` with complex signatures and position management. In v0.4 you extend `BaseChatAgent` implementing `on_messages()`, `on_reset()`, and declaring `produced_message_types`.

**State management**: v0.2 had no built-in persistence, requiring manual export/import of `chat_messages`. v0.4 introduces native methods `save_state()` and `load_state()` on agents and teams.

**Chat results**: v0.2 returned `ChatResult` with `summary`, `chat_history`, `cost`, `human_input`. v0.4 returns `TaskResult` containing a list of `messages`, without automatic summarization or cost tracking (which must be implemented by the application).

### Key Improvements

- **Message streaming**: native support for token streaming during generation
- **Observability**: built-in metric tracking, message tracing, and debugging tools with OpenTelemetry support
- **Persistence**: saving and restoring task progress
- **Action resumption**: ability to resume paused actions from the exact point of interruption
- **Type safety**: interfaces with compile-time type checking
- **Cross-language**: Python and .NET support with full interoperability
- **CancellationToken**: management of timeouts and asynchronous cancellation of operations

### Magentic-One

A flagship result built on AutoGen 0.4 is **Magentic-One**, a generalist multi-agent system for solving complex tasks. The architecture includes:

- **Orchestrator**: leader agent that plans, tracks progress, and re-plans in case of errors
- **WebSurfer**: agent specialized in controlling a Chromium browser
- **FileSurfer**: agent for navigating and reading local files
- **Coder**: agent specialized in writing code and analyzing information
- **ComputerTerminal**: provides access to a shell console for executing programs

Magentic-One has achieved performance competitive with the state of the art on benchmarks such as GAIA, AssistantBench, and WebArena, demonstrating the maturity of the v0.4 architecture for complex agentic systems.

---

## 5. Conversation Patterns

### 5.1 Two-Agent Chat

The simplest pattern: two agents that communicate directly with each other. It is the fundamental building block of all other patterns.

```python
# v0.2 - Two-agent chat
assistant = AssistantAgent("assistant", llm_config=llm_config)
user_proxy = UserProxyAgent("user_proxy", code_execution_config={"use_docker": True})

user_proxy.initiate_chat(
    assistant,
    message="Write a Python function to calculate Fibonacci numbers."
)
```

In this pattern, the `UserProxyAgent` sends an initial message to the `AssistantAgent`, which responds. The interaction continues until a termination condition is met (maximum number of turns, message containing "TERMINATE", or human input).

### 5.2 Sequential Chat

Sequential chat concatenates **multiple two-agent conversations** in sequence, with a **carryover** mechanism that transfers the summary of the previous conversation to the context of the next.

```python
# v0.2 - Sequential chat
from autogen import initiate_chats

chat_results = initiate_chats([
    {
        "sender": user_proxy,
        "recipient": researcher,
        "message": "Research recent trends in quantum computing.",
        "max_turns": 5,
        "summary_method": "reflection_with_llm",
    },
    {
        "sender": user_proxy,
        "recipient": writer,
        "message": "Write an article based on the previous research.",
        "max_turns": 5,
        "summary_method": "last_msg",
    },
    {
        "sender": user_proxy,
        "recipient": reviewer,
        "message": "Review and improve the article.",
        "max_turns": 3,
    },
])
```

Sequential chat is particularly useful for linear pipelines where the output of one phase becomes the input of the next: research -> writing -> review -> publication.

### 5.3 Group Chat

Group chat involves **three or more agents** that share a single conversation thread. It is the most powerful and flexible pattern for tasks that require collaboration between different competencies.

```python
# v0.4 - SelectorGroupChat con selezione LLM
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import MaxMessageTermination

architect = AssistantAgent(
    name="architect",
    model_client=model_client,
    system_message="You are a software architect. You design the architecture of systems.",
)

developer = AssistantAgent(
    name="developer",
    model_client=model_client,
    system_message="You are a senior developer. You implement the code.",
    tools=[code_executor_tool],
)

tester = AssistantAgent(
    name="tester",
    model_client=model_client,
    system_message="You are a QA engineer. You write and run tests.",
    tools=[code_executor_tool],
)

team = SelectorGroupChat(
    participants=[architect, developer, tester],
    model_client=model_client,
    termination_condition=MaxMessageTermination(max_messages=20),
)

result = await team.run(task="Design and implement a distributed caching system.")
```

The methods for selecting the next speaker include:
- **Round Robin** (`RoundRobinGroupChat`): deterministic rotation between agents
- **Selector** (`SelectorGroupChat`): an LLM decides who should speak next, based on the conversation context
- **Custom selector**: custom functions that implement specific selection logic

### 5.4 Nested Chat

Nested chat is an advanced pattern where an agent, after receiving a message, initiates an **internal sequence of conversations** before formulating its response. It is essentially an "inner monologue" of the agent, implemented as conversations with other agents.

```python
# v0.2 - Nested chat
arithmetic_agent = ConversableAgent(
    name="arithmetic_agent",
    llm_config=False,
    human_input_mode="ALWAYS",
)

# The agent triggers an internal chain of sub-conversations
arithmetic_agent.register_nested_chats(
    [
        {
            "sender": arithmetic_agent,
            "recipient": calculator_agent,
            "message": "Compute the result.",
            "max_turns": 2,
            "summary_method": "last_msg",
        },
        {
            "sender": arithmetic_agent,
            "recipient": validator_agent,
            "message": "Validate the computation.",
            "max_turns": 2,
            "summary_method": "last_msg",
        },
    ],
    trigger=user_proxy,  # Activated when it receives from user_proxy
)
```

Nested chat is powerful for:
- **Decomposition of complex tasks**: an agent can consult specialized sub-agents
- **Internal validation**: verify results before responding
- **Information aggregation**: collect data from multiple sources before synthesizing

---

## 6. Code Execution

### Code Execution Architecture

One of AutoGen's distinctive features is the ability of agents to **generate and execute code** as part of the conversational flow. This generate-execute-iterate cycle is fundamental for programming tasks, data analysis, and automation.

### Docker Sandboxing

The `DockerCommandLineCodeExecutor` is the recommended mechanism for safe code execution:

```python
from autogen_ext.code_executors.docker import DockerCommandLineCodeExecutor

code_executor = DockerCommandLineCodeExecutor(
    image="python:3.12-slim",  # Customizable Docker image
    timeout=60,                # Timeout in seconds
    work_dir="/workspace",     # Working directory in the container
)

# In v0.4, the code executor is wrapped in a tool
from autogen_core.tools import PythonCodeExecutionTool

code_tool = PythonCodeExecutionTool(executor=code_executor)
```

The DockerCommandLineCodeExecutor:
- Creates a dedicated Docker container for execution
- Supports multiple languages: Python, Bash, Shell, PowerShell
- The default image is `python:3-slim`, but it can be customized
- Completely isolates execution from the host system
- Supports configurable timeouts to prevent infinite executions

### Local Execution

For development environments where Docker is not available, AutoGen offers the `LocalCommandLineCodeExecutor`:

```python
from autogen_ext.code_executors.local import LocalCommandLineCodeExecutor

local_executor = LocalCommandLineCodeExecutor(
    timeout=30,
    work_dir="./output",
)
```

**Warning**: local execution carries significant security risks since the code generated by the LLM is executed directly on the host system. It is suitable only for development and testing environments.

### Azure Container Apps

v0.4 also introduces the `AzureContainerCodeExecutor` for cloud execution:

```python
from autogen_ext.code_executors.azure import AzureContainerCodeExecutor

azure_executor = AzureContainerCodeExecutor(
    # Azure Container Apps configuration
)
```

### Security Considerations

Executing code generated autonomously by LLMs presents intrinsic risks:
- The Docker container offers isolation, but default capabilities and mounts may provide incomplete isolation
- For enterprise environments, the use of cloud sandboxes (Azure Container Apps or services like E2B) is recommended
- v0.4 introduces `CancellationToken` for managing timeouts and asynchronous cancellation of operations, improving control over execution

---

## 7. Tool Integration

### FunctionTool

The `FunctionTool` class is the main mechanism for integrating custom tools in AutoGen. It uses Python function **type annotations** and **descriptions** to automatically generate the JSON schema needed by LLM models for function calling.

```python
from autogen_core.tools import FunctionTool
from typing_extensions import Annotated

async def get_stock_price(
    ticker: str,
    date: Annotated[str, "Date in the format YYYY/MM/DD"]
) -> float:
    """Get the price of a stock for a specified date."""
    # Real implementation with financial API
    return 150.25

async def search_database(
    query: Annotated[str, "Natural language search query"],
    max_results: Annotated[int, "Maximum number of results"] = 10
) -> list[dict]:
    """Search the company database."""
    # Real implementation
    return [{"id": 1, "result": "..."}]

# Tool creation
stock_tool = FunctionTool(get_stock_price, description="Get the price of a stock.")
search_tool = FunctionTool(search_database, description="Search the company database.")

# Registration in the agent (v0.4)
agent = AssistantAgent(
    name="financial_analyst",
    model_client=model_client,
    tools=[stock_tool, search_tool],
    system_message="You are a financial analyst.",
)
```

### Automatic Schema Generation

Each tool is a subclass of `BaseTool`, which automatically generates the **JSON schema** for the tool. The schema includes the function name, description, parameters with types and requirements, and additional validation rules. The model clients send this schema along with the messages to the LLM, which generates tool calls as structured JSON strings conforming to the schema.

### Function Calling Workflow

The complete function calling cycle in AutoGen follows these steps:

1. **Schema generation**: tools generate JSON schemas describing their parameters
2. **Request to model**: the model client sends the tool schemas along with the messages to the LLM
3. **Parsing**: the model generates tool calls as structured JSON, parsed into `FunctionCall` objects
4. **Execution**: the `run_json()` method executes the tools with the parsed arguments
5. **Result**: the results become `FunctionExecutionResult` objects
6. **Reflection**: optionally, the result is sent back to the model for a final response

### Built-in Tools

AutoGen provides several pre-built tools:

- **PythonCodeExecutionTool**: execution of Python snippets via code executor
- **LocalSearchTool / GlobalSearchTool**: tools for GraphRAG
- **mcp_server_tools**: integration with Model Context Protocol (MCP) servers
- **HttpTool**: for REST API requests
- **LangChainToolAdapter**: adapter to reuse LangChain tools

### MCP Integration

Support for the **Model Context Protocol (MCP)** is one of the most significant integrations in v0.4:

```python
from autogen_ext.tools.mcp import McpWorkbench

# Connection to an MCP server
workbench = McpWorkbench(server_url="http://localhost:3000")
mcp_tools = await workbench.get_tools()

agent = AssistantAgent(
    name="mcp_agent",
    model_client=model_client,
    tools=mcp_tools,
)
```

This integration allows AutoGen agents to access any tool exposed via the MCP protocol, enormously expanding the system's capabilities.

### Differences in Tool Registration: v0.2 vs v0.4

In **v0.2**, tool registration required complex configuration on two separate agents -- the caller and the executor:

```python
# v0.2 - Complex registration
@user_proxy.register_for_execution()
@assistant.register_for_llm(description="Compute the sum")
def add(a: int, b: int) -> int:
    return a + b
```

In **v0.4**, tools are passed directly to the agent via the `tools=[]` parameter. The agent internally manages both calling and execution, eliminating the complexity of routing between separate agents.

---

## 8. Comparison with LangGraph and CrewAI

### Design Philosophy

The three frameworks represent fundamentally different approaches to building multi-agent systems:

| Aspect | AutoGen | LangGraph | CrewAI |
|---------|---------|-----------|--------|
| **Paradigm** | Conversation | Graphs/Workflow | Roles/Team |
| **Metaphor** | Chat between experts | Flow diagram | Corporate team |
| **Abstraction** | Conversational agents | Nodes and edges | Agents with roles and tasks |
| **Flexibility** | High (multiple patterns) | Very high (arbitrary graphs) | Medium (predefined structure) |
| **Learning curve** | Medium | High | Low |
| **State** | Shared conversation | Typed graph state | Crew context |
| **Orchestration** | GroupChatManager / Team | Conditional edges | Process (sequential/hierarchical) |

### AutoGen vs LangGraph

**AutoGen** excels in scenarios where natural collaboration between agents is a priority. The conversational paradigm makes it intuitive to model discussions, debates, and group decision-making processes. **GroupChat** allows an emergent dynamic where agents self-organize.

**LangGraph** offers more granular control through the graph model. Every transition is explicit, the state is typed, and workflows can include conditional logic, parallel branching, and complex cycles. It is superior for deterministic and production-grade workflows with durability requirements (native support for checkpointing and persistence).

In summary: AutoGen is more suitable for **exploration and collaboration**, LangGraph for **control and determinism**.

### AutoGen vs CrewAI

**AutoGen** offers more conversation patterns and greater architectural flexibility, especially with v0.4 and support for distributed systems. The layered architecture allows operating at different degrees of abstraction.

**CrewAI** focuses on simplicity and intuitiveness. The "crew" metaphor with roles, tasks, and goals is immediately understandable, and YAML configuration makes setup accessible even to teams with mixed technical skills.

In summary: AutoGen for **complexity and scalability**, CrewAI for **speed and simplicity**.

### When to Choose AutoGen

AutoGen is the best choice when:
- The task requires **natural dialogue** between agents with different specializations
- You need **group decision-making** or debate scenarios
- **Code execution** is a central component of the workflow
- You anticipate a future need for **distribution** of agents across multiple nodes
- The team already uses the Microsoft ecosystem (Azure, Semantic Kernel)
- You want a **no-code GUI** (AutoGen Studio) for prototyping
- You build complex generalist agentic systems (Magentic-One style)

---

## 9. Use Cases and Best Practices

### Main Use Cases

**Collaborative software development**: a team of agents (architect, coder, reviewer, tester) that collaborate on writing, reviewing, and testing code. Integrated code execution allows real-time validation of solutions.

**Data analysis and research**: agents that collect data (web scraping, APIs), analyze it (Python code execution with pandas/matplotlib), and produce structured reports. Sequential chat is ideal for analysis pipelines.

**Content creation pipeline**: researchers who collect information, writers who produce content, editors who review and improve. Nested chat allows each agent to consult sub-agents for specific tasks.

**Advanced customer support**: agents specialized in different areas (technical, billing, account) coordinated by an orchestrator that routes requests to the appropriate agent via group chat.

**Business process automation**: workflows involving decisions, approvals, and distributed actions, leveraging v0.4's distributed architecture and integration with external systems via tools and MCP.

### Best Practices

**1. Define clear and specific system messages**: each agent must have a well-defined role. Vague system messages produce agents that overlap in competencies and generate inefficient conversations.

**2. Use explicit termination conditions**: always define when a conversation must end. Without limits, agents can enter infinite loops. Use `MaxMessageTermination`, `TextMentionTermination`, or custom conditions.

**3. Prefer Docker for code execution**: in any environment that is not purely local development, always use the `DockerCommandLineCodeExecutor` or `AzureContainerCodeExecutor` to isolate execution.

**4. Minimize the number of agents**: add agents only when the task really requires it. Too many agents increase latency, the cost of LLM calls, and the complexity of debugging. Start with the simplest pattern and scale only if necessary.

**5. Implement logging and tracing**: in v0.4, leverage OpenTelemetry support for tracing interactions. In v0.2, manually log conversations. Observability is critical for debugging multi-agent systems.

**6. Manage API costs**: multi-agent conversations generate many LLM calls. Monitor costs, use cheaper models where possible, and limit the number of turns per conversation.

**7. Test tools in isolation**: before integrating them into an agent, test each tool individually with various inputs. A tool that fails silently can derail an entire conversation.

**8. Use `reflect_on_tool_use=True`**: in v0.4, this parameter makes the agent generate a natural language response after using a tool, improving the readability of conversations.

**9. Leverage state persistence**: in v0.4, use `save_state()` and `load_state()` to save the progress of long tasks, allowing resumption in case of interruptions.

**10. Plan migration to v0.4**: if you use v0.2, start planning the migration. The AgentChat API of v0.4 maintains a similar level of abstraction, but the architectural benefits (async, distribution, observability) are significant.

---

## 10. Resources and References

### Official Documentation
- **GitHub Repository**: https://github.com/microsoft/autogen
- **v0.4 Documentation (stable)**: https://microsoft.github.io/autogen/stable/
- **v0.2 Documentation**: https://microsoft.github.io/autogen/0.2/
- **Migration Guide v0.2 -> v0.4**: https://microsoft.github.io/autogen/0.4.8/user-guide/agentchat-user-guide/migration-guide.html

### Blog and Technical Articles
- **AutoGen v0.4 Reimagined (Microsoft Research)**: https://www.microsoft.com/en-us/research/blog/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/
- **New Architecture Preview**: https://microsoft.github.io/autogen/0.2/blog/2024/10/02/new-autogen-architecture-preview/
- **AutoGen v0.4 Launch Blog**: https://devblogs.microsoft.com/autogen/autogen-reimagined-launching-autogen-0-4/

### AG2 (Fork)
- **AG2 Repository**: https://github.com/ag2ai/ag2
- **Analysis of the split**: https://dev.to/maximsaplin/microsoft-autogen-has-split-in-2-wait-3-no-4-parts-2p58
- **AutoGen vs AG2 comparison**: https://www.gettingstarted.ai/autogen-vs-ag2/

### Magentic-One
- **Paper**: https://arxiv.org/abs/2411.04468
- **Microsoft Research**: https://www.microsoft.com/en-us/research/articles/magentic-one-a-generalist-multi-agent-system-for-solving-complex-tasks/

### Framework Comparisons
- **CrewAI vs LangGraph vs AutoGen (DataCamp)**: https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen
- **Architectural comparison 2025**: https://latenode.com/blog/platform-comparisons-alternatives/automation-platform-comparisons/langgraph-vs-autogen-vs-crewai-complete-ai-agent-framework-comparison-architecture-analysis-2025
- **Langfuse - Comparing Agent Frameworks**: https://langfuse.com/blog/2025-03-19-ai-agent-comparison

### Quick Installation

```bash
# AutoGen 0.4 (recommended)
pip install -U "autogen-agentchat" "autogen-ext[openai]"

# With Docker support for code execution
pip install -U "autogen-ext[docker]"

# With Azure support
pip install -U "autogen-ext[azure]"

# AutoGen Studio (no-code GUI)
pip install -U autogenstudio
autogenstudio ui --port 8080

# AutoGen 0.2 (legacy)
pip install "autogen-agentchat~=0.2"
```

### Notes on Choosing the Version

For **new projects**: use AutoGen 0.4 with the `autogen-agentchat` and `autogen-core` packages. The event-driven architecture, async support, and distribution capabilities make it the best choice for any application that goes beyond a simple prototype.

For **legacy projects**: if you have existing code based on v0.2, evaluate gradual migration. v0.2 will continue to receive bug fixes and security patches, but new feature development is focused on v0.4.

For **enterprise with Microsoft stack**: consider integration with Semantic Kernel, which will incorporate the AutoGen 0.4 runtime offering commercial support and long-term stability.

> **Important note**: Microsoft has indicated that new users should consider the Microsoft Agent Framework as the entry point. AutoGen will continue to be maintained and to receive bug fixes and critical security patches, but the strategic direction is shifting toward convergence with the broader Microsoft ecosystem.
