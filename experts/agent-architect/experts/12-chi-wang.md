# Chi Wang -- AutoGen and the Conversational Multi-Agent Revolution

## 1. Profile and Background

### Academic Background

Chi Wang holds a PhD in Computer Science earned at the **University of Illinois at Urbana-Champaign (UIUC)**, one of the most prestigious computer science programs in the world, and a bachelor's degree (BS) in Computer Science from **Tsinghua University** in Beijing, a university considered the Chinese equivalent of MIT. His doctoral dissertation focused on knowledge mining from text data and graphs, earning the prestigious **SIGKDD Data Science/Data Mining PhD Dissertation Award**, a recognition that signals an exceptional contribution in the field of data mining.

### Microsoft Research (2014--2024)

Chi Wang spent an entire decade at **Microsoft Research** as **Principal Researcher**, from 2014 to 2024. During this period he accumulated more than 15 years of overall experience in computer science research, working on a broad spectrum of problems: large language models and AI frameworks, automated machine learning, machine learning for systems, scalable solutions for data analytics and data science, and knowledge mining.

At Microsoft Research, Wang gave life to two open-source projects of great impact:

- **FLAML (Fast Lightweight AutoML)**: an open-source library for AutoML and hyperparameter tuning, widely adopted both within Microsoft and by the external community. FLAML stands out for computational efficiency in hyperparameter optimization and for its economical approach to model training.

- **AutoGen**: the conversational multi-agent framework that revolutionized the way of building LLM-based applications. AutoGen received the **Best Paper Award at the ICLR'24 LLM Agents Workshop**, was selected for **Open100**, and listed among the **5 favorite AI papers of 2023 by TheSequence**.

### EcoOptiGen: Cost-Effective Optimization for LLMs

Before AutoGen, Wang developed **EcoOptiGen**, a framework for the economic optimization of LLM inference hyperparameters. Published in 2023 at the Second International Conference on Automated Machine Learning, EcoOptiGen tackles a critical problem: maximizing the value of text generation given a limited inference budget.

The framework jointly optimizes parameters such as the number of responses, temperature, max tokens, probability mass, and the prompts themselves, using cost-based pruning techniques. Experiments with GPT-3.5 and GPT-4 on datasets such as APPS, HumanEval (code generation), MATH (math problem solving), and XSum (text summarization) verified its effectiveness. EcoOptiGen was implemented as part of the `autogen` package within the FLAML library, laying the conceptual foundations for the future multi-agent framework.

### Google DeepMind (2024--Present)

In 2024, Chi Wang left Microsoft Research to join **Google DeepMind** as **Senior Staff Research Scientist**. This transition coincides with the moment when AutoGen was reaching its peak popularity and the framework was forking into two distinct projects (AutoGen 0.4 and AG2). At DeepMind, Wang continues his research on LLMs and agentic systems. Among recent works, a paper on the optimization of LLM queries in relational workloads is scheduled for publication at **MLSys 2025**.

---

## 2. AutoGen: The First Conversational Multi-Agent Framework

### Context and Motivation

AutoGen is born at a precise moment in the history of AI: when Large Language Models had demonstrated extraordinary capabilities but practical applications require more than a single prompt. Building complex LLM-based systems poses significant challenges: orchestrating collaboration between different models, integrating external tools, incorporating human feedback, and managing articulated workflows.

The foundational paper -- **"AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation"** (arXiv:2308.08155, August 2023) -- presents AutoGen as an open-source framework that allows developers to build LLM applications via **multiple agents that converse with each other to complete tasks**. The paper, of 43 total pages (10 of main text, 3 of references, 30 of appendices), was written by 14 researchers with **Qingyun Wu** as first author and Chi Wang in a senior/leadership role.

### Design Philosophy

The fundamental insight of AutoGen is radical in its simplicity: **simplify and unify the complex workflows of LLM applications as multi-agent conversations**. This translates into a programming paradigm centered on inter-agent conversations, achieved through:

1. **Definition of conversable agents** with specific capabilities and roles
2. **Programming the interaction behavior** between agents through conversation-centered computation and control

This paradigm allows describing complex systems in intuitive terms: instead of managing chains of prompts, API calls, and conditional logics, the developer defines agents with personalities and skills, and puts them in conversation.

### Base Architecture (v0.2)

The original architecture of AutoGen rests on three pillars:

**Customizable Agents**: each agent can be configured with an arbitrary combination of LLM models, human input, and tools. This allows creating specialized agents (a coder, a critic, a planner) that collaborate naturally.

**Native Conversability**: agents are designed to converse -- they can send, receive, and generate messages. Conversation is the primitive mechanism through which all work is performed.

**Flexible Interaction Patterns**: both natural language and code can be used to program flexible conversation patterns for different applications.

### Generic Infrastructure

AutoGen is designed as **generic infrastructure** for building diversified applications of varying complexity and LLM capability. The paper demonstrates the effectiveness of the framework in six distinct domains: math, coding, question answering, operations research, online decision-making, and entertainment. This breadth of application validates the generality of the conversational approach.

---

## 3. Conversable Agents: The Key Concept

### The ConversableAgent Class

The heart of the AutoGen architecture is the **`ConversableAgent`** class, a generic abstraction for agents capable of conversing through message exchange to complete a task collaboratively. Each `ConversableAgent` possesses three fundamental capabilities:

1. **Send messages** (`send`): the agent can initiate or continue a conversation by sending messages to other agents
2. **Receive messages** (`receive`): the agent processes received messages and determines the appropriate response
3. **Generate messages** (`generate_reply`): the agent uses its resources (LLM, code, human input) to produce responses

### Capability Composition

The power of the `ConversableAgent` lies in the flexible composition of its capabilities. A single agent can be configured with:

- **LLM models**: one or more models for text generation, with support for multiple configurations (fallback, cost-based selection)
- **Code execution**: ability to execute Python, shell, or other language code, with optional sandboxing via Docker
- **Human input**: integration of human feedback in the decision loop, with three modes (`ALWAYS`, `TERMINATE`, `NEVER`)
- **Tools and functions**: registration of custom functions that the agent can invoke

### Built-in Specialized Agents

AutoGen provides two main specializations of `ConversableAgent`:

**AssistantAgent**: configured by default as an LLM-powered AI assistant. It has `human_input_mode="NEVER"` and a system message that instructs it to solve problems, suggest Python code, and verify the results. It is the system's "thinker" agent.

**UserProxyAgent**: represents a proxy of the human user in the multi-agent system. It has `human_input_mode="ALWAYS"` (or `TERMINATE`) and can execute code. It is the "executor" agent that translates the assistant's proposals into concrete actions.

### Reply Mechanism

The `register_reply()` mechanism is the foundation of AutoGen's extensibility. It allows registering custom handlers that are invoked when an agent receives a message. This same mechanism is used internally by `GroupChatManager` to implement next speaker selection in group chats. The reply chain is processed in priority order, allowing sophisticated compositions of behaviors.

### The Communication Protocol

Communication between agents follows a structured protocol:

1. The sender agent calls `initiate_chat()` specifying the recipient and the initial message
2. The recipient receives the message and invokes its chain of `generate_reply()`
3. The response is sent to the sender, who in turn processes it
4. The cycle continues until a termination condition (maximum number of turns, special message such as "TERMINATE", or human intervention)
5. At the end, a `ChatResult` object is generated containing the history, a summary, and token usage statistics

---

## 4. Conversation Patterns

AutoGen defines four main conversation patterns that can be composed with each other as modular blocks.

### 4.1 Two-Agent Chat

The simplest pattern: two agents exchange messages alternately. One agent initiates the conversation with `initiate_chat()`, sending an initial message to the recipient.

```python
chat_result = student_agent.initiate_chat(
    teacher_agent,
    message="What is the triangle inequality?",
    summary_method="reflection_with_llm",
    max_turns=2,
)
```

The result (`ChatResult`) offers access to:
- `chat_result.summary`: summary generated via LLM
- `chat_result.chat_history`: complete conversation
- `chat_result.cost`: breakdown of token usage

Two summarization methods are available: `last_msg` (default, uses the last message) and `reflection_with_llm` (generates a summary via LLM).

### 4.2 Sequential Chat

A sequence of two-agent chats concatenated through a **carryover** mechanism. The summary of each chat becomes context for the next, allowing complex tasks to be decomposed into interdependent sub-activities.

```python
chat_results = number_agent.initiate_chats([
    {
        "recipient": adder_agent,
        "message": "14",
        "max_turns": 2,
        "summary_method": "last_msg",
    },
    {
        "recipient": multiplier_agent,
        "message": "These are my numbers",
        "max_turns": 2,
        "summary_method": "last_msg",
    },
])
```

The carryover accumulates: subsequent chats receive the summaries of **all** previous chats, not just the last one. The `initiate_chats()` method (plural) accepts a list of configurations and returns a list of `ChatResult`.

### 4.3 Group Chat

The most complex pattern: more than two agents contribute to a single conversation thread, orchestrated by a **`GroupChatManager`**. All agents share the same context and messages are broadcast to all participants.

```python
from autogen import GroupChat, GroupChatManager

group_chat = GroupChat(
    agents=[adder_agent, multiplier_agent, subtracter_agent,
            divider_agent, number_agent],
    messages=[],
    max_round=6,
)

group_chat_manager = GroupChatManager(
    groupchat=group_chat,
    llm_config={"config_list": [{"model": "gpt-4"}]},
)
```

The `GroupChatManager` supports four next speaker selection strategies:

1. **`round_robin`**: agents are selected in a predetermined order
2. **`random`**: random selection
3. **`manual`**: selection guided by the human user
4. **`auto`** (default): LLM-based selection that analyzes the context to choose the most appropriate agent

Advanced features include:

- **Send Introductions**: automatic broadcasting of agent names and descriptions before the chat begins
- **Constrained Speaker Selection**: definition of a graph of allowed transitions between agents, limiting which agents can speak after which
- **Custom Speaker Message Role**: customization of the role of selection messages (e.g. `"user"` instead of `"system"` for compatibility with APIs such as Mistral)

### 4.4 Nested Chat

The most powerful pattern: encapsulates complex workflows within a single agent, creating a unified interface that hides internal complexity. When an agent receives a message, it can activate a sequence of nested chats with other agents and use the result as its own response.

```python
nested_chats = [
    {
        "recipient": group_chat_manager_with_intros,
        "summary_method": "reflection_with_llm",
        "summary_prompt": "Summarize the sequence of operations...",
    },
    {
        "recipient": code_writer_agent,
        "message": "Write a Python script to verify...",
        "summary_method": "reflection_with_llm",
    },
    {
        "recipient": poetry_agent,
        "message": "Write a poem about this result.",
        "max_turns": 1,
        "summary_method": "last_msg",
    },
]

arithmetic_agent.register_nested_chats(
    nested_chats,
    trigger=lambda sender: sender not in [
        group_chat_manager_with_intros,
        code_writer_agent,
        poetry_agent
    ],
)
```

The procedural flow of nested chats is the following: message received, human-in-the-loop check, evaluation of trigger conditions, sequential execution of nested chats, generation of the final response from the summary of the last chat.

The main use cases of nested chats are: question-answering bots with unified interface, packaging of workflows for reuse in larger systems, and abstraction of tool use within the agent's interface.

### Pattern Composition

The true power of AutoGen emerges from the **composability** of these patterns: sequential chats can contain group chats; group chats can be nested within other agents; a `GroupChatManager` can act as recipient in a sequential pattern. This recursive composability allows building systems of arbitrary complexity while keeping each component understandable and testable.

---

## 5. AutoGen 0.4 / AG2: The Evolution of the Framework

### The Fork (November 2024)

In autumn 2024, the AutoGen project underwent a significant fork that created two parallel development paths. The original creators of AutoGen, including Chi Wang, left Microsoft in September 2024. In November, they created a new GitHub organization -- **AG2AI** -- and a new repository, inheriting the PyPI packages `autogen` and `pyautogen` and the Discord channel.

The stated motivation was to enable an **open governance** that would facilitate open collaboration and healthy growth driven by the community. AG2 has established a governance structure with clear community representation and a path toward greater community control over time. The project now counts **20 maintainers** from leading organizations including Google, IBM, Meta, and universities such as Penn State and University of Washington.

### Microsoft AutoGen 0.4

Microsoft continued the development of the framework under the name **AutoGen 0.4**, released in January 2025, representing a **complete redesign** of the original architecture.

**Actor Model**: AutoGen 0.4 embraces the **actor model** to support distributed, highly scalable, and event-driven agentic systems. Agents are implemented as actors that exchange messages asynchronously through a runtime.

**Three-Layer Architecture**:

1. **Core**: the fundamental building blocks for an event-driven agentic system. Implements the actor model, provides asynchronous message exchange between agents and event-driven agents that perform computations in response to messages. Offers tools to observe and control agent behavior, allows the execution of agents on different processes and in different languages, and enables both static and dynamic multi-agent patterns.

2. **AgentChat**: a high-level task-driven API built on top of Core, perfect for rapid prototyping. Maintains the functionalities of the original AutoGen while adding support for streaming, serialization, state and memory management for agents, and full-time support for a better development experience.

3. **Extensions**: implementations of Core interfaces and integrations with third-party software, including Azure code executor, OpenAI model clients, and others.

### AG2: The Community-Driven Fork

AG2 maintains the familiar architecture of AutoGen 0.2, offering **stability and backward compatibility** under community governance. The main difference compared to Microsoft's AutoGen is that AG2 is community-driven rather than driven by a single large company.

**AG2 Beta** represents a ground-up redesign with a streaming and event-driven architecture where each conversation runs on a **MemoryStream**, a pub/sub event bus that isolates state and enables real-time streaming.

### Impact of the Fork

This fork created temporary confusion in the community, with developers uncertain about which version to adopt. In practice:

- Those who want **stability and continuity** with existing 0.2 code should look at **AG2**
- Those who want a **modern, distributed, and scalable architecture** for enterprise production should evaluate **AutoGen 0.4**
- Those seeking **open governance and community contribution** will find a more welcoming environment in **AG2**

---

## 6. Comparison with Other Frameworks

### AutoGen vs LangGraph

**Architectural approach**: LangGraph uses graph-based orchestration where workflows are represented as nodes and edges, enabling modular and conditional execution. AutoGen focuses on conversational architecture where agents interact through natural language, adapting their roles based on context.

**State management**: LangGraph offers production-grade state management with durability, checkpointing, and replay. AutoGen manages state through the conversation history shared between agents.

**Flexibility**: LangGraph excels in complex decision pipelines with conditional logic, branching, and dynamic adaptation. AutoGen excels in scenarios where natural language interaction and rapid prototyping are priorities.

**Learning curve**: LangGraph requires familiarity with LangChain and graph concepts. AutoGen has a gentler curve thanks to the intuitive conversational abstraction.

**Ideal use case**: choose LangGraph for stateful production workflows with fine-grained control. Choose AutoGen for conversational multi-agent systems with diverse interaction patterns, especially group decision-making or debate scenarios.

### AutoGen vs CrewAI

**Paradigm**: CrewAI follows a role-based model where agents behave like employees with specific responsibilities. AutoGen follows a conversational paradigm where natural communication drives collaboration.

**Simplicity**: CrewAI is the simplest to configure -- define roles, objectives, and tasks and the framework manages the orchestration. AutoGen requires more configuration but offers greater flexibility in interaction patterns.

**Scalability**: AutoGen, especially in version 0.4, is designed for distributed and scalable systems. CrewAI is more oriented to teams of agents of moderate size.

**Human-in-the-loop**: AutoGen has native and granular support for human intervention (three distinct modes). CrewAI supports human input but with less granularity.

**Ideal use case**: choose CrewAI to model teams of agents with clear roles and obtain results quickly, especially for business workflow automation. Choose AutoGen for complex conversational systems with diverse interaction patterns and need for deep customization.

### Summary Table

| Dimension | AutoGen | LangGraph | CrewAI |
|---|---|---|---|
| Paradigm | Conversational | Graph | Role-based |
| Strength | Interaction flexibility | State control | Simplicity |
| Scalability | High (0.4) | Medium-High | Medium |
| Human-in-loop | Native, granular | Configurable | Supported |
| Learning curve | Medium | High | Low |
| Production | Maturing (0.4) | Production-ready | Rapid setup |

---

## 7. Practical Use Cases

### 7.1 Coding and Code Generation

AutoGen excels in collaborative multi-agent coding. The typical pattern involves:

- An **AssistantAgent** that generates code based on specifications
- A **UserProxyAgent** with code execution enabled that runs the proposed code
- An automatic feedback loop where execution errors are reported back to the assistant for correction

This approach enables **autonomous self-debugging**: agents can write, execute, and self-correct code iteratively. In enterprise settings, this has been applied to automatic debugging of live applications, such as the use of kubectl to diagnose and resolve misconfigured Kubernetes deployments.

### 7.2 Research and Scientific Analysis

For research, AutoGen allows building end-to-end pipelines where teams of agents synthesize information from disparate sources (databases, APIs, documents) into a coherent narrative. The typical pattern uses group chats with specialized agents:

- A **Research Planner** that decomposes the research question
- One or more **Data Gatherers** that collect information
- An **Analyst** that interprets the data
- A **Writer** that synthesizes the results

This approach is particularly powerful for market research, literature review, and competitive analysis.

### 7.3 Data Analysis

AutoGen lends itself naturally to data analysis through the combination of agents that reason about data and agents that execute code. The typical workflow involves:

1. The user describes the desired analysis in natural language
2. A planner agent translates the request into analytical steps
3. A coder agent generates the necessary Python/pandas/matplotlib code
4. An executor agent runs the code and reports the results
5. A review agent evaluates the results and suggests further investigations

### 7.4 Enterprise Applications

The enterprise adoption of AutoGen is growing significantly. **Novo Nordisk**, the pharmaceutical giant, is actively using AutoGen to develop a production-ready multi-agent framework that helps its community derive insights from technical data. This signals the transition of AutoGen from academic experiment to real business tool.

The enterprise sectors showing interest include: pharmaceutical, biotech, finance, fintech, consulting, supply chain, healthcare, e-commerce, and manufacturing. Typical applications include supply chain optimization, financial reconciliation, multi-source market research, report generation, and automation of multi-step processes that previously required days or weeks for entire teams.

### 7.5 AutoGen Studio

Microsoft has released **AutoGen Studio**, a low-code interface for building multi-agent workflows. Studio provides a visual interface for defining agents, configuring their parameters, and orchestrating conversations, lowering the entry barrier for teams with mixed technical skills. This tool represents the bridge between the researchers who conceived AutoGen and the professionals who apply it in business contexts.

---

## 8. Resources and References

### Foundational Papers

- **AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation** -- Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W. White, Doug Burger, Chi Wang. arXiv:2308.08155, August 2023.

- **Cost-Effective Hyperparameter Optimization for Large Language Model Generation Inference** -- Chi Wang, Xueqing Liu, Ahmed Hassan Awadallah. Proceedings of the Second International Conference on Automated Machine Learning, 2023. arXiv:2303.04673.

### Repositories and Documentation

- Microsoft AutoGen Repository (0.4): https://github.com/microsoft/autogen
- AG2 Repository: https://github.com/ag2ai/ag2
- AutoGen 0.2 Documentation: https://microsoft.github.io/autogen/0.2/
- AutoGen 0.4 Documentation: https://microsoft.github.io/autogen/stable/
- Chi Wang's GitHub: https://github.com/sonichi
- Chi Wang's Google Scholar: https://scholar.google.com/citations?user=IiSNwnAAAAAJ

### Comparisons and Analyses

- DataCamp: CrewAI vs LangGraph vs AutoGen -- https://www.datacamp.com/tutorial/crewai-vs-langgraph-vs-autogen
- IBM: What is AutoGen? -- https://www.ibm.com/think/topics/autogen
- Latenode: LangGraph vs AutoGen vs CrewAI Architecture Analysis 2025 -- https://latenode.com/blog/platform-comparisons-alternatives/automation-platform-comparisons/langgraph-vs-autogen-vs-crewai-complete-ai-agent-framework-comparison-architecture-analysis-2025

### Awards and Recognitions

- Best Paper Award, ICLR 2024 LLM Agents Workshop
- Open100 Selection
- TheSequence's Pick of 5 Favorite AI Papers 2023
- SIGKDD Data Science/Data Mining PhD Dissertation Award (Chi Wang)
