# Multi-Agent Patterns in Google ADK: Complete Guide

> In-depth analysis of the eight multi-agent patterns introduced by Google in its Agent Development Kit (ADK), with direct comparison to Anthropic's conceptual patterns.

**Main source:** [Developer's Guide to Multi-Agent Patterns in ADK - Google Developers Blog](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/)

---

## 1. Overview

### The context: from monolithic agent to multi-agent systems

The evolution of Large Language Model-based systems is going through a fundamental architectural shift. While the first generation of AI applications was based on a single monolithic agent -- a single LLM with a complex prompt and a heterogeneous set of tools -- the maturity of the field today requires a different approach. Google, with its **Agent Development Kit (ADK)**, proposes an open-source, code-first framework that fully embraces the multi-agent paradigm, offering explicit composition primitives to build modular, testable, and production-ready systems.

The blog post published by Google on the Developers Blog represents a systematic guide that catalogs **eight design patterns** fundamental for multi-agent systems. These patterns are not theoretical abstractions: each is directly implementable through ADK primitives (`SequentialAgent`, `ParallelAgent`, `LoopAgent`, `LlmAgent`, `AgentTool`) and supported by pseudocode and concrete examples.

The analogy proposed by Google is illuminating: building a multi-agent system is like designing a **microservices architecture** for AI. Each agent has a specific role (a Parser, a Critic, a Dispatcher), well-defined boundaries, and standardized communication interfaces. The result is a system that is intrinsically more modular, specialized, reusable, and maintainable than a monolith.

### ADK fundamental primitives

Before diving into the patterns, it is essential to understand the ADK building blocks:

| Primitive | Function |
|-----------|----------|
| `LlmAgent` | Individual agent powered by an LLM, with instructions, tools, and description |
| `SequentialAgent` | Orchestrator that runs sub-agents in linear order |
| `ParallelAgent` | Orchestrator that runs sub-agents in parallel |
| `LoopAgent` | Orchestrator that runs sub-agents in a loop with exit conditions |
| `AgentTool()` | Wrapper that transforms an agent into a tool invokable by another agent |
| `EventActions` | Signaling mechanism for early termination of loops |
| `AutoFlow` | Automatic routing system based on agent descriptions |

Communication between agents takes place primarily through **shared session state** (`session.state`), which serves as a shared whiteboard. Each agent can write results using `output_key` and read outputs from previous agents through template variables like `{key_name}`.

---

## 2. Sequential Pattern (Sequential Pipeline)

### Architecture

The Sequential pattern is the most intuitive: agents are executed one after the other in a predefined order, like an assembly line. Agent A completes its task and "passes the baton" directly to Agent B.

```
[Input] --> [Agent A: Parser] --> [Agent B: Extractor] --> [Agent C: Synthesizer] --> [Output]
```

In ADK, this pattern is natively implemented by the `SequentialAgent` primitive, which manages the sequential orchestration of sub-agents.

### Implementation in ADK

```python
from google.adk.agents import SequentialAgent, LlmAgent

# Definition of specialized agents
parser_agent = LlmAgent(
    name="pdf_parser",
    instructions="Extract raw text from the provided PDF document.",
    output_key="raw_text"
)

extractor_agent = LlmAgent(
    name="entity_extractor",
    instructions="From the text in {raw_text}, extract all key entities: "
                 "names, dates, amounts, regulatory references.",
    output_key="entities"
)

summarizer_agent = LlmAgent(
    name="summarizer",
    instructions="Using the text {raw_text} and the entities {entities}, "
                 "generate a structured summary of the document.",
    output_key="summary"
)

# Composition into the sequential pipeline
pipeline = SequentialAgent(
    name="document_processing_pipeline",
    sub_agents=[parser_agent, extractor_agent, summarizer_agent]
)
```

### State management

The fundamental mechanism is `output_key`: each agent writes its output to a specific key in the session state. Subsequent agents access this data through template variables. Google emphasizes the importance of using **descriptive keys**: "Session.state is your whiteboard. Use descriptive keys with `output_key` so that downstream agents know exactly what they are reading."

### When to use the Sequential pattern

- **Deterministic workflows** with a clear and immutable logical order
- **Data transformation pipelines** where each step depends on the output of the previous one
- **Processes with validation gates** between one step and the next
- **Simplified debugging**: linearity makes it immediate to identify where an error occurs

### Limitations

The main limitation is **latency**: the total time is the sum of the times of all the agents. There is no parallelism, and a bottleneck in one step blocks the entire pipeline.

---

## 3. Parallel Pattern (Fan-Out / Fan-In)

### Architecture

The Parallel pattern directly addresses the limitation of Sequential: when multiple tasks are independent of each other, they can be executed simultaneously. The architecture provides a **fan-out** phase (parallel distribution) followed by a **fan-in** phase (aggregation of results).

```
                    +--> [Security Auditor]  --+
                    |                           |
[Input] --> [Fan-Out] --> [Style Enforcer]  ---+--> [Synthesizer] --> [Output]
                    |                           |
                    +--> [Performance Analyst] -+
```

### Implementation in ADK

```python
from google.adk.agents import ParallelAgent, SequentialAgent, LlmAgent

# Parallel workers
security_agent = LlmAgent(
    name="security_auditor",
    instructions="Analyze the code for security vulnerabilities: "
                 "injection, XSS, weak authentication, secrets management.",
    output_key="security_report"
)

style_agent = LlmAgent(
    name="style_enforcer",
    instructions="Verify code conformance to style guidelines: "
                 "naming conventions, structure, documentation.",
    output_key="style_report"
)

performance_agent = LlmAgent(
    name="performance_analyst",
    instructions="Identify performance issues: algorithmic complexity, "
                 "memory leaks, N+1 queries, blocking operations.",
    output_key="performance_report"
)

# Parallel execution
parallel_review = ParallelAgent(
    name="parallel_code_review",
    sub_agents=[security_agent, style_agent, performance_agent]
)

# Synthesis agent
synthesizer = LlmAgent(
    name="review_synthesizer",
    instructions="Combine the review reports: "
                 "Security: {security_report}, "
                 "Style: {style_report}, "
                 "Performance: {performance_report}. "
                 "Generate a unified report with priorities.",
    output_key="final_review"
)

# Composition: parallel + synthesis
code_review_system = SequentialAgent(
    name="code_review_pipeline",
    sub_agents=[parallel_review, synthesizer]
)
```

### Race condition management

A critical aspect of the Parallel pattern is the **management of shared state**. ADK modifies the `InvocationContext.branch` for each child agent (e.g., `ParentBranch.ChildName`), but all sub-agents maintain access to shared session state. To avoid race conditions:

- Each worker **must write to unique and distinct `output_key`** keys
- Workers can **read** initial state but must not overwrite keys used by other workers
- The synthesizer agent operates **after** the completion of all workers

### When to use the Parallel pattern

- **Independent tasks** that have no mutual dependencies
- **Latency optimization**: total time is that of the slowest worker, not the sum
- **Multi-perspective analysis**: evaluating the same input from different angles (security, style, performance)
- **Data collection from multiple sources**: parallel API calls to different services

---

## 4. Hierarchical Pattern (Hierarchical Decomposition)

### Architecture

The Hierarchical pattern introduces a tree structure where higher-level agents decompose complex objectives into subtasks and delegate them to lower-level agents. It is the pattern closest to the concept of organizational "management": a supervisor does not perform the work directly, but coordinates specialists.

```
[Supervisor / Project Manager]
    |
    +-- [Research Lead]
    |       +-- [Web Researcher]
    |       +-- [Database Researcher]
    |
    +-- [Writing Lead]
    |       +-- [Draft Writer]
    |       +-- [Editor]
    |
    +-- [Quality Assurance]
```

### Implementation in ADK with AgentTool

The key mechanism is `AgentTool()`, which allows wrapping an entire sub-system of agents as a single tool invokable by the supervisor:

```python
from google.adk.agents import LlmAgent, SequentialAgent, AgentTool

# Research sub-system (composed of multiple agents)
web_researcher = LlmAgent(
    name="web_researcher",
    instructions="Search for information on the web related to the provided query.",
    output_key="web_findings"
)

db_researcher = LlmAgent(
    name="db_researcher",
    instructions="Query the internal database for relevant data.",
    output_key="db_findings"
)

research_team = SequentialAgent(
    name="research_team",
    sub_agents=[web_researcher, db_researcher]
)

# Writing sub-system
draft_writer = LlmAgent(
    name="draft_writer",
    instructions="Write a draft based on the provided research data.",
    output_key="draft"
)

editor = LlmAgent(
    name="editor",
    instructions="Review and improve the draft: clarity, coherence, tone.",
    output_key="edited_draft"
)

writing_team = SequentialAgent(
    name="writing_team",
    sub_agents=[draft_writer, editor]
)

# Supervisor that uses the teams as tools
supervisor = LlmAgent(
    name="project_manager",
    instructions="""You are a project manager for report creation.
    For each request:
    1. Use the research team to gather information
    2. Use the writing team to produce the document
    3. Evaluate the final result and request revisions if necessary.""",
    tools=[
        AgentTool(agent=research_team),
        AgentTool(agent=writing_team)
    ]
)
```

### Advantages of the hierarchy

- **Context window management**: each level operates with reduced and specific context, avoiding overload of a single monolithic agent
- **Failure isolation**: an error in a sub-system does not necessarily compromise the entire system
- **Composability**: sub-systems can be reused in different contexts
- **Scalability**: new hierarchical levels can be added without restructuring the entire system

### When to use the Hierarchical pattern

- **Complex problems** that require decomposition into sub-problems
- **Wide contexts** that exceed the effective context window of a single LLM
- **Functional teams** with distinct competencies (research, writing, QA)
- **Enterprise systems** where separation of concerns is fundamental

---

## 5. Dynamic Routing (Coordinator/Dispatcher)

### Architecture

Dynamic Routing is the pattern where a central agent, powered by an LLM, analyzes the user's intent and routes the request to the most appropriate specialist agent. Unlike the Sequential pattern (where the flow is fixed), here the path is determined **dynamically** by the content of the input.

```
                            +--> [Billing Specialist]
                            |
[Input] --> [Coordinator] --+--> [Tech Support Agent]
                            |
                            +--> [Account Manager]
```

### Implementation in ADK

ADK offers two mechanisms for dynamic routing:

#### Mechanism 1: AutoFlow with `sub_agents`

```python
from google.adk.agents import LlmAgent

billing_agent = LlmAgent(
    name="billing_specialist",
    description="Handles questions about billing, payments, "
                "refunds, and tariff plans.",
    instructions="You are a billing expert. Answer questions "
                 "about costs, invoices, payment methods, and refunds."
)

tech_support = LlmAgent(
    name="tech_support",
    description="Resolves technical issues: bugs, configurations, "
                "API integrations, and troubleshooting.",
    instructions="You are an expert technician. Guide the user through "
                 "the resolution of technical problems step by step."
)

account_manager = LlmAgent(
    name="account_manager",
    description="Manages account operations: profile editing, "
                "plan upgrades/downgrades, cancellation.",
    instructions="Manage requests related to the user's account."
)

# The coordinator analyzes intent and delegates
coordinator = LlmAgent(
    name="customer_service_coordinator",
    instructions="You are the customer service coordinator. "
                 "Analyze the user's request and transfer it "
                 "to the most appropriate specialist agent.",
    sub_agents=[billing_agent, tech_support, account_manager]
)
```

ADK's `AutoFlow` mechanism intercepts the `transfer_to_agent(agent_name='target')` call generated by the LLM, identifies the target agent through `root_agent.find_agent()`, and transfers the context.

#### Mechanism 2: AgentTool for explicit invocation

```python
coordinator = LlmAgent(
    name="coordinator",
    instructions="Analyze the request and invoke the appropriate specialist.",
    tools=[
        AgentTool(agent=billing_agent),
        AgentTool(agent=tech_support),
        AgentTool(agent=account_manager)
    ]
)
```

### The critical role of descriptions

Google emphasizes a crucial point: **agent descriptions function as API documentation for the LLM**. Routing quality depends directly on the precision and clarity with which sub-agent descriptions are written. Vague or overlapping descriptions lead to incorrect routing.

### When to use Dynamic Routing

- **Multi-domain systems** with specialists for different areas
- **Customer service** where requests cover heterogeneous categories
- **Adaptive systems** that must handle unpredictable input
- **Architectures where input classification is reliable** (if the classification is ambiguous, the routing will fail)

---

## 6. Loop Patterns: Generator-Critic and Iterative Refinement

ADK introduces two loop-based patterns that deserve joint treatment.

### 6a. Generator-Critic

The separation between generation and validation with conditional loop based on pass/fail criteria:

```python
from google.adk.agents import LoopAgent, SequentialAgent, LlmAgent

generator = LlmAgent(
    name="code_generator",
    instructions="Generate Python code based on the specification. "
                 "If you have received feedback, correct the indicated errors. "
                 "Current specification: {spec}. Previous feedback: {feedback}",
    output_key="draft"
)

critic = LlmAgent(
    name="code_critic",
    instructions="""Evaluate the code in {draft} against these criteria:
    1. Syntactic correctness
    2. Error handling
    3. Conformance to the specification

    If ALL criteria are met, respond ONLY with 'PASS'.
    Otherwise, provide specific feedback on the errors found.""",
    output_key="feedback"
)

review_cycle = SequentialAgent(
    name="review_cycle",
    sub_agents=[generator, critic]
)

generation_loop = LoopAgent(
    name="code_generation_loop",
    sub_agents=[review_cycle],
    condition_key="feedback",
    exit_condition="PASS",
    max_iterations=5
)
```

### 6b. Iterative Refinement

Unlike Generator-Critic (binary: pass/fail), Iterative Refinement aims at **progressive qualitative improvement**:

```python
generator = LlmAgent(
    name="content_generator",
    instructions="Generate or rewrite the content based on the brief "
                 "and on the optimization notes received.",
    output_key="draft"
)

critic = LlmAgent(
    name="quality_critic",
    instructions="Analyze {draft} and provide specific optimization "
                 "notes to improve clarity, depth, and impact. "
                 "If the content is excellent, signal completion.",
    output_key="optimization_notes"
)

refiner = LlmAgent(
    name="refiner",
    instructions="Rewrite {draft} applying the notes: {optimization_notes}. "
                 "Maintain the strengths and improve the indicated areas.",
    output_key="draft"  # Overwrites the original draft!
)

refinement_cycle = SequentialAgent(
    name="refinement_cycle",
    sub_agents=[generator, critic, refiner]
)

refinement_loop = LoopAgent(
    name="iterative_refinement",
    sub_agents=[refinement_cycle],
    max_iterations=3
)
```

A key aspect: the Refiner writes to **the same `output_key`** as the Generator (`"draft"`), overwriting the previous draft. This allows the next iteration to start from the improved version. Additionally, an agent can signal early completion by setting `escalate=True` in `EventActions`, exiting the loop before reaching `max_iterations`.

---

## 7. Combining Patterns

### Composite architectures for real-world scenarios

The previous patterns are building blocks. Real-world systems almost always require a **composition of multiple patterns**. Google provides a concrete example: a customer support system that combines:

1. **Coordinator** (Dynamic Routing) as an entry point to classify intent
2. **Parallel** (Fan-Out) to search simultaneously in documentation and knowledge base
3. **Generator-Critic Loop** to ensure tone consistency in the final response

```python
# Level 1: Parallel search
doc_searcher = LlmAgent(name="doc_searcher", ...)
kb_searcher = LlmAgent(name="kb_searcher", ...)

parallel_search = ParallelAgent(
    name="parallel_search",
    sub_agents=[doc_searcher, kb_searcher]
)

# Level 2: Generation with quality loop
response_generator = LlmAgent(name="response_generator", ...)
tone_critic = LlmAgent(name="tone_critic", ...)

quality_cycle = SequentialAgent(
    name="quality_cycle",
    sub_agents=[response_generator, tone_critic]
)

quality_loop = LoopAgent(
    name="quality_loop",
    sub_agents=[quality_cycle],
    condition_key="tone_check",
    exit_condition="APPROVED"
)

# Level 3: Complete pipeline for each specialist
billing_pipeline = SequentialAgent(
    name="billing_pipeline",
    sub_agents=[parallel_search, quality_loop]
)

tech_pipeline = SequentialAgent(
    name="tech_pipeline",
    sub_agents=[parallel_search, quality_loop]
)

# Level 4: Coordinator with routing
coordinator = LlmAgent(
    name="coordinator",
    instructions="Analyze the intent and route to the specialist.",
    sub_agents=[billing_pipeline, tech_pipeline]
)
```

### Guidelines for composition

Google suggests an incremental approach:

1. **Start simple**: begin with a sequential chain of 2-3 agents
2. **Add parallelism** where tasks are independent
3. **Introduce loops** where quality requires iteration
4. **Add routing** when the system must handle heterogeneous input
5. **Compose hierarchically** when complexity requires autonomous sub-systems

---

## 8. Comparison with Anthropic Patterns

Anthropic published its own taxonomy of agentic patterns in "[Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)", identifying five fundamental patterns. The comparison reveals significant convergences and some differences in perspective.

### Mapping table

| Anthropic Pattern | ADK Pattern (Google) | Convergence | Notes |
|---|---|---|---|
| **Prompt Chaining** | **Sequential Pipeline** | Strong | Same architecture. Anthropic emphasizes programmable validation gates between steps; ADK implements it with `SequentialAgent` + `output_key` |
| **Routing** | **Coordinator/Dispatcher** | Strong | Identical concept. ADK provides `AutoFlow` as a native mechanism; Anthropic describes it at the conceptual level |
| **Parallelization** (Sectioning) | **Parallel Fan-Out/Gather** | Strong | Same pattern. ADK offers `ParallelAgent` as a primitive; Anthropic also distinguishes the "Voting" sub-pattern (not present in ADK as a primitive) |
| **Orchestrator-Workers** | **Hierarchical Decomposition** | Moderate | Similar concept but different emphasis. Anthropic emphasizes **dynamism**: subtasks are not predefined. ADK focuses on the **structural hierarchy** with `AgentTool()` |
| **Evaluator-Optimizer** | **Generator-Critic** + **Iterative Refinement** | Moderate | ADK breaks down the Anthropic pattern into two distinct variants: a binary one (pass/fail) and a qualitative one (progressive improvement), offering more granularity |
| -- | **Human-in-the-Loop** | ADK only | Pattern not present in Anthropic's taxonomy as an explicit architectural pattern |
| **Parallelization** (Voting) | -- | Anthropic only | ADK does not have a primitive dedicated to voting, although it is implementable with `ParallelAgent` |
| -- | **Composite Patterns** | ADK only | ADK explicitly dedicates a pattern to composition; Anthropic treats it implicitly |

### Analysis of philosophical differences

**Anthropic approach: minimalism and transparency.** Anthropic adopts an avowedly minimalist philosophy: "Add complexity only when it demonstrably improves results." The patterns are described at a conceptual level with generic pseudocode, without binding to a specific framework. Anthropic even recommends starting from direct use of LLM APIs, noting that "many patterns can be implemented in just a few lines of code."

**Google ADK approach: explicit primitives and composability.** Google adopts a more structured approach, providing concrete orchestration primitives (`SequentialAgent`, `ParallelAgent`, `LoopAgent`) that encapsulate coordination logic. This reduces boilerplate code but introduces a dependency on the framework. The advantage is standardization: a `SequentialAgent` has predictable and testable behavior.

**On dynamism.** The most significant difference emerges in the Orchestrator-Workers pattern (Anthropic) vs Hierarchical Decomposition (ADK). Anthropic emphasizes that the distinguishing point of the orchestrator is the ability to **dynamically** determine the necessary subtasks, unlike parallelization where tasks are predefined. In ADK, the hierarchy is more structured: sub-systems are defined a priori and the supervisor invokes them as tools. Dynamism in ADK depends on the supervisor's LLM, but the structure of the sub-systems is fixed.

**On loops and refinement.** ADK offers greater granularity than Anthropic by explicitly distinguishing between Generator-Critic (binary validation with `condition_key`/`exit_condition`) and Iterative Refinement (qualitative improvement with `max_iterations` and `escalate`). Anthropic groups both under Evaluator-Optimizer.

### Which approach to choose?

- **Rapid prototyping**: Anthropic patterns, more abstract, allow quick implementations with any LLM SDK
- **Production-grade systems**: Google's ADK primitives offer robust infrastructure with state management, branching, and lifecycle management
- **Learning**: the two frameworks are complementary -- Anthropic explains the "why", ADK shows the "how"

---

## 9. Practical Implementation: Best Practices and Code Examples

### Complete pattern: Document Analysis System

An end-to-end example combining Sequential, Parallel, and Generator-Critic:

```python
from google.adk.agents import (
    LlmAgent, SequentialAgent, ParallelAgent, LoopAgent, AgentTool
)

# === PHASE 1: Ingestion and Parsing (Sequential) ===
ingestion_agent = LlmAgent(
    name="document_ingester",
    instructions="Receive the document and normalize the format. "
                 "Extract the clean text removing header/footer.",
    output_key="clean_text"
)

metadata_agent = LlmAgent(
    name="metadata_extractor",
    instructions="From the text {clean_text}, extract: title, author, date, "
                 "document type, language.",
    output_key="metadata"
)

ingestion_pipeline = SequentialAgent(
    name="ingestion",
    sub_agents=[ingestion_agent, metadata_agent]
)

# === PHASE 2: Multi-dimensional Analysis (Parallel) ===
sentiment_agent = LlmAgent(
    name="sentiment_analyzer",
    instructions="Analyze the sentiment of the text {clean_text}. "
                 "Provide score and justification.",
    output_key="sentiment_analysis"
)

entity_agent = LlmAgent(
    name="entity_extractor",
    instructions="Extract all named entities from the text {clean_text}: "
                 "people, organizations, places, dates, amounts.",
    output_key="entities"
)

topic_agent = LlmAgent(
    name="topic_classifier",
    instructions="Classify the main topics of the text {clean_text}. "
                 "Assign categories and sub-categories.",
    output_key="topics"
)

parallel_analysis = ParallelAgent(
    name="multi_analysis",
    sub_agents=[sentiment_agent, entity_agent, topic_agent]
)

# === PHASE 3: Synthesis with Quality Loop ===
synthesizer = LlmAgent(
    name="report_synthesizer",
    instructions="""Generate a structured report combining:
    - Metadata: {metadata}
    - Sentiment: {sentiment_analysis}
    - Entities: {entities}
    - Topics: {topics}

    The report must be professional, complete, and actionable.""",
    output_key="report_draft"
)

quality_reviewer = LlmAgent(
    name="quality_reviewer",
    instructions="""Evaluate the report in {report_draft}:
    1. Completeness: are all the analysis data included?
    2. Coherence: are the conclusions supported by the data?
    3. Format: is the report well structured?

    If all criteria are met: respond 'APPROVED'.
    Otherwise: provide specific corrections.""",
    output_key="review_result"
)

quality_cycle = SequentialAgent(
    name="quality_cycle",
    sub_agents=[synthesizer, quality_reviewer]
)

quality_loop = LoopAgent(
    name="quality_assurance",
    sub_agents=[quality_cycle],
    condition_key="review_result",
    exit_condition="APPROVED",
    max_iterations=3
)

# === COMPLETE PIPELINE ===
document_analyzer = SequentialAgent(
    name="document_analysis_system",
    sub_agents=[ingestion_pipeline, parallel_analysis, quality_loop]
)
```

### Best Practice Checklist

#### State Management

1. **Descriptive keys**: use names like `security_report` rather than `output_1`
2. **Isolation in parallel**: each parallel worker must write to unique keys
3. **Temporary state**: use the `temp:` prefix for data valid only within a single invocation
4. **Avoid side-effects**: agents must not modify keys that are not in their domain

#### Agent Design

1. **Precise instructions**: the `instructions` must be specific and unambiguous
2. **Descriptions as APIs**: the `description` of sub-agents is the documentation that the LLM uses for routing -- they must be clear and non-overlapping
3. **Single Responsibility**: each agent should have a single well-defined task
4. **Meaningful naming**: agent names must reflect their role

#### Error Handling

1. **Max iterations**: always set a limit in `LoopAgent` to avoid infinite loops
2. **Fallback**: provide fallback agents in routing for unclassifiable input
3. **Escalation**: implement `escalate=True` to allow early loop exits
4. **Logging**: trace transitions between agents for debugging

#### Test Strategies

1. **Per-agent test**: verify each agent in isolation
2. **Integration test**: verify the state flow between agents
3. **Routing test**: verify that the coordinator correctly routes each input category
4. **Convergence test**: verify that loops converge within the iteration limit

### Anti-patterns to avoid

- **God Agent**: a single agent with too many responsibilities and tools -- decompose it into sub-agents
- **Implicit state**: relying on undocumented conventions for state keys -- be explicit
- **Over-engineering**: using complex patterns where simple prompt chaining would suffice
- **Generic descriptions**: writing vague descriptions like "handles requests" that confuse routing
- **Loops without limits**: forgetting `max_iterations` risking infinite loops and uncontrolled costs

---

## 10. Resources and References

### Primary Google sources

- [Developer's Guide to Multi-Agent Patterns in ADK](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/) -- Main blog post analyzed in this guide
- [Multi-Agent Systems - ADK Documentation](https://google.github.io/adk-docs/agents/multi-agents/) -- Official ADK documentation on multi-agent systems
- [Agent Development Kit - Official Documentation](https://google.github.io/adk-docs/) -- Complete ADK documentation
- [Building Collaborative AI: Multi-Agent Systems with ADK](https://cloud.google.com/blog/topics/developers-practitioners/building-collaborative-ai-a-developers-guide-to-multi-agent-systems-with-adk) -- Google Cloud Blog
- [Build Multi-Agentic Systems Using Google ADK](https://cloud.google.com/blog/products/ai-machine-learning/build-multi-agentic-systems-using-google-adk) -- Google Cloud Blog
- [Build Multi-Agent Systems with ADK - Codelab](https://codelabs.developers.google.com/codelabs/production-ready-ai-with-gc/3-developing-agents/build-a-multi-agent-system-with-adk) -- Practical Google Codelabs tutorial
- [ADK Python - GitHub Repository](https://github.com/google/adk-python) -- Official source code

### Anthropic sources (for comparison)

- [Building Effective Agents - Anthropic Research](https://www.anthropic.com/research/building-effective-agents) -- Anthropic's guide to agentic patterns
- [How We Built Our Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system) -- Practical implementation of the orchestrator-workers pattern

### Third-party analyses

- [Google's Eight Essential Multi-Agent Design Patterns - InfoQ](https://www.infoq.com/news/2026/01/multi-agent-design-patterns/) -- Pattern analysis
- [Building Multi-Agent Systems with Google ADK - Dot Square Lab](https://dotsquarelab.com/resources/building-multi-agent-systems-with-google-adk-a-practical-developer-s-guide) -- Practical guide
- [Implementing Agentic Architectural Patterns with Google ADK - Medium](https://medium.com/@saeedhajebi/implementing-agentic-architectural-patterns-with-google-adk-75281096de32) -- Pattern implementation

---

*Guide written on March 26, 2026. Based on the analysis of the Google Developers blog post and the official ADK documentation.*
