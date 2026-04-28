# GitHub Repositories with Agentic Patterns (Resources 40-43)

## In-Depth Analysis of Four Foundational Repositories for Designing Agentic AI Systems

---

## 1. Overview -- Why These Repositories Are Useful

The design of agentic AI systems has reached a level of maturity that requires a systematic and structured approach. It is no longer about assembling prompts and API calls in an ad hoc fashion, but about applying proven **architectural patterns** that solve recurring problems in building autonomous and semi-autonomous agents.

The four repositories analyzed in this document represent four complementary perspectives on the same domain:

- **promptadvisers/agentic-design-patterns-docs** offers a visual and textual taxonomy of 21 fundamental patterns, with Mermaid diagrams and detailed architectural discussions. It is the ideal entry point for those seeking a conceptual understanding of agentic building blocks.

- **nibzard/awesome-agentic-patterns** is the broadest and most curated catalogue available, with over 97 patterns documented across 8 categories, derived from real-world implementations in production. It is the reference resource for those seeking field-tested patterns rather than theoretical constructs.

- **arunpshankar/Agentic-Workflow-Patterns** translates patterns into executable Python code, integrated with the Google Cloud ecosystem (Vertex AI, Gemini). It is the repository for those who want to implement, not just understand.

- **muratcankoylan/Agent-Skills-for-Context-Engineering** introduces a dimension that is often overlooked: context management as an engineering discipline. With 13 reusable skills and the BDI (Belief-Desire-Intention) model, it raises the discussion to a higher level of architectural sophistication.

Together, these repositories cover the entire spectrum: from **conceptual understanding** to **systematic catalogation**, from **practical implementation** to **cognitive meta-architecture**.

### Why Patterns Matter

Agentic patterns are not an academic exercise. They solve concrete problems:

- **Reproducibility**: a documented pattern can be implemented by different teams with consistent results.
- **Communication**: they provide a shared vocabulary among architects, developers and stakeholders.
- **Composition**: well-defined patterns can be combined to build complex architectures.
- **Debugging**: when an agentic system fails, patterns help isolate the faulty component.
- **Scalability**: patterns such as parallelism, dynamic sharding and DAG orchestration are prerequisites for systems operating at scale.

---

## 2. promptadvisers/agentic-design-patterns-docs -- The 21 Agentic Patterns

**Repository**: https://github.com/promptadvisers/agentic-design-patterns-docs
**Stars**: ~281 | **Forks**: ~147 | **License**: MIT
**Format**: Mermaid Diagrams + Textual Discussions + ASCII Art

### Repository Structure

The repository organizes the documentation into three parallel formats:

1. **`/mermaid-diagrams/`** -- Visual flow diagrams in Mermaid syntax, renderable directly on GitHub.
2. **`/pattern-discussion/`** -- In-depth discussions for each pattern, with use cases, advantages/disadvantages, and implementation guidance.
3. **`/ascii-art/`** -- Textual visualizations for environments where Mermaid is not available.

### The 21 Patterns -- Complete List and Description

#### Category 1: Core Patterns (5 patterns)

**1. Prompt Chaining**
Decomposes a complex task into a sequence of steps, where the output of each becomes the input of the next. It is the most intuitive pattern: instead of asking an LLM to do everything at once, a pipeline of progressive transformations is built. Example: research -> analysis -> synthesis -> formatting.

**2. Routing**
Analyzes the incoming request and routes it to the most appropriate handler. It works as an intelligent dispatcher: given an input, the router semantically determines which sub-agent, tool or pipeline is best equipped to respond. Crucial for multi-domain systems where a single agent cannot excel at everything.

**3. Parallelization**
Executes independent tasks simultaneously to drastically reduce total processing time. When an agent must compare two products, it can research both in parallel. This pattern is fundamental for speed and efficiency, and composes well with the aggregation pattern.

**4. Reflection**
The simplest and most powerful agentic loop: generate output, evaluate it against defined criteria, accept or revise. The agent becomes its own reviewer. It typically produces acceptable output within 2-3 iterations, eliminating the need for manual review. Particularly effective for code generation, technical writing and structured data extraction.

**5. Tool Use**
Integrates external capabilities: APIs, databases, file systems, browsers, calculators. Transforms the agent from a purely linguistic system into an actor in the real world. The design of tool interfaces is critical: tools that are too generic are useless, those that are too specific limit flexibility.

#### Category 2: Advanced Patterns (5 patterns)

**6. Planning**
Strategic decomposition of a goal into sub-goals, with creation of an action plan before execution. Unlike Prompt Chaining (fixed sequence), Planning dynamically generates the sequence of actions based on context. Includes variants such as plan-and-execute, adaptive re-planning and hierarchical planning.

**7. Multi-Agent Collaboration**
Coordinates multiple agents with distinct roles and competencies to solve problems that exceed the capabilities of a single agent. Includes communication patterns (broadcast, point-to-point, blackboard), negotiation protocols and conflict resolution strategies.

**8. Memory Management**
Structured storage and retrieval of context across sessions and interactions. Distinguishes between short-term memory (current conversation context), long-term memory (persistent knowledge) and episodic memory (past experiences). Crucial for agents that must maintain coherence over time.

**9. Learning and Adaptation**
Improvement of performance over time through feedback, examples and self-evaluation. Includes adaptive few-shot learning, fine-tuning based on interactions, and meta-learning to transfer competencies across domains.

**10. Model Context Protocol**
Standardized communication between agents and between agents and tools, defining how context is structured, transmitted and interpreted. MCP has become a de facto standard for interoperability in the agentic ecosystem.

#### Category 3: System Patterns (5 patterns)

**11. Goal Setting and Monitoring**
Structured tracking of goals with progress metrics, success/failure conditions and escalation triggers. Essential for agents with long-running tasks where drift from the goal is a concrete risk.

**12. Exception Handling and Recovery**
Error management with strategies for retry, fallback, graceful degradation and state recovery. A robust agent does not fail silently: it notifies, attempts alternatives and, if necessary, asks for help.

**13. Human-in-the-Loop**
Incorporation of human feedback at critical points in the workflow. It is not a fallback: it is an architectural choice that recognizes the limits of complete autonomy. Includes patterns for approval, review, escalation and co-creation.

**14. Knowledge Retrieval - RAG**
Access to external knowledge bases to enrich the agent's context with up-to-date, domain-specific or proprietary information. RAG (Retrieval-Augmented Generation) is the most widespread pattern, but also includes variants such as Graph RAG, Multi-hop RAG and RAG with re-ranking.

**15. Inter-Agent Communication**
Protocols for exchanging messages between agents: synchronous vs asynchronous, point-to-point vs publish-subscribe, with management of formats, serialization and message versioning.

#### Category 4: Optimization Patterns (4 patterns)

**16. Resource-Aware Optimization**
Efficient management of tokens, API calls, computation time and costs. Includes strategies such as result caching, dynamic model selection (powerful model for complex tasks, lightweight model for simple tasks) and request batching.

**17. Reasoning Techniques**
Structured approaches to thinking: Chain-of-Thought (CoT), Tree-of-Thought (ToT), Graph-of-Thought (GoT), and multi-step reasoning techniques. These patterns guide the model through explicit reasoning processes rather than letting it generate direct answers.

**18. Guardrails/Safety Patterns**
Constraints that ensure safe and controlled operations: input validation, output filtering, action limits, sandboxing and audit trails. They are not optional: they are architectural requirements for any production system.

**19. Evaluation and Monitoring**
Continuous tracking of performance through metrics, structured logs, distributed tracing and dashboards. Includes patterns such as LLM-as-Judge, automated evaluations and A/B testing for agents.

#### Category 5: Strategic Patterns (2 patterns)

**20. Prioritization**
Management of the relative importance of tasks when resources are limited. Includes scheduling algorithms, priority queue management and dynamic rebalancing based on urgency and impact.

**21. Exploration and Discovery**
Search for new solutions outside known paths. Includes guided exploration strategies, controlled serendipity and solution diversification. Essential for creative and research agents.

---

## 3. nibzard/awesome-agentic-patterns -- The Curated Catalogue

**Repository**: https://github.com/nibzard/awesome-agentic-patterns
**Stars**: ~2,800+ | **License**: Apache-2.0
**Website**: https://www.agentic-patterns.com/
**Documented Patterns**: 97+ across 8 categories

### Origin and Philosophy

This project was born from the experience documented in "What Sourcegraph Learned Building AI Coding Agents" (May 2025) and from the video diary "Raising an Agent". The philosophy is clear: **patterns tested in production, not theoretical constructs**. Every pattern included must be:

- **Repeatable**: adopted by multiple independent teams.
- **Agent-centric**: improves perception, reasoning or action of the agent.
- **Traceable**: documented in verifiable public sources.

### The 8 Categories and Key Patterns

#### 3.1 Orchestration & Control (~48 patterns)

The largest category, governing task decomposition, multi-agent coordination and execution planning.

**Notable Patterns:**
- **Plan-Then-Execute**: explicit separation between planning phase and execution phase.
- **Sub-Agent Spawning and Swarm Migrations**: dynamic creation of sub-agents and migration between swarm configurations.
- **Discrete Phase Separation**: division of the workflow into distinct phases with explicit transitions.
- **Planner-Worker Separation for Long-Running Agents**: dual architecture for tasks that require hours or days.
- **Oracle-and-Worker Multi-Model**: powerful model (oracle) for strategic decisions, lightweight model (worker) for execution.
- **Autonomous Workflow Agent Architecture**: architecture for fully autonomous agents.
- **Chain-of-Thought Monitoring & Interruption**: surveillance of reasoning with interruption capability.
- **Feature List as Immutable Contract**: the feature list as an immutable contract that guides execution.

#### 3.2 Tool Use & Environment (~24 patterns)

How agents interact with external systems: shell, APIs, sandboxed environments.

**Notable Patterns:**
- **CLI-Native Agent Orchestration**: agents that operate natively from the command line.
- **Code-Over-API Pattern**: code generation instead of API calls for greater flexibility.
- **Dynamic Code Injection**: injection of code at runtime to extend the agent's capabilities.
- **Virtual Machine Operator Agent**: agent that operates an entire VM as its environment.
- **Code Mode for Model Context Protocol**: code mode specific to the MCP protocol.
- **Parallel Tool Execution**: simultaneous execution of multiple tool calls.

#### 3.3 Context & Memory (~18 patterns)

Management of agent awareness and information retention.

**Notable Patterns:**
- **Prompt Caching via Exact Prefix Preservation**: prompt caching leveraging exact prefix preservation.
- **Semantic Context Filtering**: context filtering based on semantic relevance.
- **Self-Identity Accumulation**: progressive accumulation of the agent's identity through interactions.
- **Working Memory via TodoWrite**: working memory implemented through structured task lists.
- **Sliding-Window Curation**: context window curation with intelligent scrolling.
- **Context Window Anxiety Management**: proactive management of context window saturation.
- **Context-Minimization Pattern**: deliberate minimization of context to focus attention.

#### 3.4 Feedback Loops (~14 patterns)

How agents learn and improve from execution results.

**Notable Patterns:**
- **Graph of Thoughts**: reasoning structured as a graph rather than a linear chain.
- **Incident-to-Eval Synthesis**: transformation of incidents into evaluation tests.
- **Rich Feedback Loops**: multimodal and multi-dimensional feedback loops.
- **Self-Critique Evaluator Loop**: critical self-evaluation loop.
- **Iterative Multi-Agent Brainstorming**: iterative brainstorming among multiple agents.

#### 3.5 Reliability & Eval (~20 patterns)

Reliability assurance through testing, guardrails and observability.

**Notable Patterns:**
- **Schema Validation Retry**: automatic retry with schema validation.
- **Structured Output Specification**: formal specification of structured outputs.
- **Workflow Evals with Mocked Tools**: workflow evaluation with mocked tools.
- **Anti-Reward-Hacking Grader Design**: design of evaluators resistant to reward hacking.

#### 3.6 Security & Safety (~10 patterns)

Protection against abuse and uncontrolled behavior.

**Notable Patterns:**
- **Isolated VM per RL Rollout**: VM isolation for each reinforcement learning rollout.
- **PII Tokenization**: tokenization of personally identifiable information.
- **Hook-Based Safety Guard Rails**: safety guardrails based on hooks.
- **Zero-Trust Agent Mesh**: agent mesh with zero-trust architecture.

#### 3.7 Learning & Adaptation (~7 patterns)

Evolution and development of agent competencies.

**Notable Patterns:**
- **Agent Reinforcement Fine-Tuning (Agent RFT)**: agent fine-tuning via reinforcement learning.
- **Variance-Based RL Sample Selection**: variance-based RL sample selection.
- **Parallel Tool Call Learning**: parallel learning of tool calls.

#### 3.8 UX & Collaboration (~15 patterns)

Human-agent interaction and integration into workflows.

**Notable Patterns:**
- **Human-in-the-Loop Approval Framework**: structured framework for human approval.
- **Seamless Background-to-Foreground Handoff**: transparent transition from background to foreground execution.
- **Spectrum of Control / Blended Initiative**: spectrum of control with mixed human-agent initiative.

### Website Tools

The agentic-patterns.com site offers interactive tools: Pattern Explorer with filters, side-by-side comparison, decision guides, graph visualization and curated pattern bundles. It also includes an `llms.txt` file for machine-readable documentation optimized for AI assistants and RAG systems.

---

## 4. arunpshankar/Agentic-Workflow-Patterns -- Python Patterns with Google Approach

**Repository**: https://github.com/arunpshankar/Agentic-Workflow-Patterns
**License**: MIT | **Language**: Python 3.8+
**Stack**: Google Cloud Platform, Vertex AI, Gemini, SERP API
**Article**: "Designing Cognitive Architectures: Agentic Workflow Patterns from Scratch" (Google Cloud Community)

### Philosophy and Approach

This repository stands out for three characteristics:

1. **Executable code**: every pattern is implemented in working Python, not just documented.
2. **Google Cloud integration**: uses Vertex AI and Gemini models, demonstrating how patterns integrate with enterprise infrastructure.
3. **Modular architecture**: modular, scalable and reusable design with clean Python design patterns.

### Code Structure

```
./src/patterns/          -- Pattern implementations
./data/patterns/         -- Templates (system/user prompt, JSON schema, results)
./config/                -- Configuration
./img/                   -- Architectural diagrams
./mermaid/               -- Mermaid diagrams
```

### The 8 Implemented Patterns

#### 4.1 Reflection

**Architecture**: Actor-Critic framework with content generator (Actor) and reviewer (Critic) in an iterative loop.

**Flow**:
1. The Actor generates initial content.
2. The Critic evaluates the content against defined criteria.
3. If unsatisfactory, the Critic provides specific feedback.
4. The Actor revises the content incorporating the feedback.
5. The loop continues until convergence or iteration limit.

**Implementation**: `src/patterns/reflection/critic.py` contains the Critic agent that evaluates and provides structured feedback. The generator receives the feedback and produces improved versions iteratively.

#### 4.2 Web Access

**Architecture**: Orchestration of specialized agents for retrieving and processing web content.

**Specialized Agents**:
- **Search Agent**: performs web searches via SERP API.
- **Scraping Agent**: extracts content from the pages found.
- **Summarization Agent**: summarizes the extracted content.

**Flow**: The orchestrator coordinates the search -> scraping -> synthesis sequence, handling errors and timeouts.

#### 4.3 Semantic Routing

**Architecture**: Coordinator-Delegate with routing based on semantic intent.

**Flow**:
1. The Coordinator receives the user's query.
2. Analyzes the semantic intent of the request.
3. Routes to the appropriate specialized sub-agent.
4. The sub-agent processes the request and returns the result.

**Implemented Example**: Travel booking system with sub-agents for flights, hotels and car rentals. The Coordinator determines whether the user wants to book a flight, search for a hotel or rent a car and routes accordingly.

#### 4.4 Parallel Delegation

**Architecture**: Named Entity Recognition (NER) to identify distinct entities, followed by parallel delegation to specialized agents.

**Flow**:
1. The Coordinator receives a complex query with multiple entities.
2. The NER module identifies the distinct entities.
3. For each entity, a delegate agent is created.
4. The delegate agents process the entities in parallel.
5. The results are aggregated and returned.

**Use case**: "Compare the biographies of Einstein, Tesla and Curie" generates three parallel agents, each specialized in one biography.

#### 4.5 Dynamic Sharding

**Architecture**: Splitting large datasets into manageable shards for parallel processing.

**Flow**:
1. The Coordinator receives a large dataset (e.g., list of 100 celebrities).
2. Divides the list into shards of configurable size.
3. For each shard, dynamically creates a specialized Delegate agent.
4. The Delegate agents process the shards in parallel.
5. The results are collected and consolidated.

**Difference from Parallel Delegation**: Dynamic Sharding operates on homogeneous datasets (same operation on different partitions), while Parallel Delegation handles heterogeneous entities (potentially different operations on different entities).

#### 4.6 Task Decomposition

**Architecture**: Explicit decomposition of a complex task into independent sub-tasks, each handled by a dedicated agent.

**Flow**:
1. The complex task is analyzed to identify sub-tasks.
2. The sub-tasks are explicitly defined in the code.
3. For each sub-task, a specialized agent is instantiated.
4. The sub-tasks are executed (sequentially or in parallel, depending on dependencies).
5. The results are aggregated.

#### 4.7 Dynamic Decomposition

**Architecture**: Unlike static Task Decomposition, here the LLM itself autonomously generates the sub-tasks.

**Flow**:
1. The LLM receives the complex task and generates a list of sub-tasks.
2. For each generated sub-task, an agent is created.
3. The agents process the sub-tasks.
4. The results are aggregated.

**Key difference**: in Task Decomposition the sub-tasks are defined by the programmer; in Dynamic Decomposition they are generated by the model, offering greater flexibility but less control.

#### 4.8 DAG Orchestration

**Architecture**: Management of complex workflows via directed acyclic graphs (DAGs) defined in YAML.

**Components**:
| Agent | Role | Input | Output |
|-------|------|-------|--------|
| CoordinatorAgent | Orchestration | DAG definition | Final result |
| CollectAgent | Document collection | Folder path | Document collection |
| PreprocessAgent | Normalization | Raw documents | Cleaned documents |
| ExtractAgent | Info extraction | Preprocessed documents | Structured info |
| SummarizeAgent | Synthesis | Preprocessed documents | Summaries |
| CompileAgent | Report compilation | Info + Summaries | Final report |

**YAML Configuration**: The DAG is defined in a YAML file that specifies nodes (tasks), dependencies (edges) and configuration parameters. This allows the workflow to be modified without touching the code.

**Execution**: The CoordinatorAgent loads the DAG definition, initializes the task states, enters an iterative loop that identifies tasks with satisfied dependencies, executes them (in parallel if possible) and updates the states until completion.

**Schema Validation**: Each agent validates inputs and outputs via JSON Schema, ensuring consistency in processing and error handling throughout the entire pipeline.

---

## 5. muratcankoylan/Agent-Skills-for-Context-Engineering -- Agent Skills and Context Engineering

**Repository**: https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering
**Stars**: 10,000+ | **License**: MIT
**Format**: Markdown Skills + Python Scripts + References
**Citations**: Peer-reviewed research (Peking University, 2026)

### The Context Engineering Paradigm

This repository introduces a fundamental paradigm shift: from **prompt engineering** to **context engineering**. The distinction is crucial:

- **Prompt Engineering**: focuses on formulating effective instructions.
- **Context Engineering**: addresses the holistic curation of ALL information that enters the model's limited attention budget: system prompt, tool definitions, retrieved documents, message history and tool outputs.

Context is not just text: it is attention. The effectiveness of context depends on attention mechanics rather than raw token capacity. Phenomena such as "lost-in-the-middle" (information in the middle of the context receives less attention) and attention scarcity as context length increases are real engineering problems that require architectural solutions.

### The 13 Skills -- Organization and Detail

Each skill follows a standard structure:
- `SKILL.md` -- mandatory instructions and metadata.
- `scripts/` -- executable demonstrations.
- `references/` -- additional documentation.

#### Foundational Skills

**context-fundamentals**: Fundamental concepts of context in agentic systems. Covers the structure of the context window, attention mechanisms, and how different types of information compete for the limited attention budget.

**context-degradation**: Recognition of context failure patterns. Identifies four critical patterns:
- *Lost-in-the-middle*: central information is ignored.
- *Context poisoning*: incorrect information that corrupts reasoning.
- *Context distraction*: irrelevant information that dilutes attention.
- *Context clash*: contradictory information that paralyzes decision-making.

**context-compression**: Strategies for compressing context in long-running sessions. Includes techniques for progressive summarization, selective elimination and semantic compaction.

#### Architectural Skills

**multi-agent-patterns**: Orchestrator, peer-to-peer and hierarchical architectures. Covers communication topologies between agents, coordination protocols and work distribution strategies.

**memory-systems**: Design of short-term, long-term and graph-based memories. Includes patterns for state persistence, contextual retrieval and management of memory decay.

**tool-design**: Effective construction of tools for use by agents. Covers interface design, tool documentation (which the agent reads to decide which tool to use), and error handling strategies specific to agentic tools.

**filesystem-context**: Dynamic context discovery and offloading of outputs to the file system. Patterns for agents that must operate on large codebases, where relevant context must be discovered dynamically rather than provided statically.

**hosted-agents**: Background coding agents with sandboxed VMs and multiplayer support. Covers the architecture of agents operating in isolated environments with collaboration capabilities.

#### Operational Skills

**context-optimization**: Compaction, masking and caching strategies. Includes concrete techniques to optimize the use of the context window: compaction of past messages, masking of non-relevant information and caching of intermediate results.

**evaluation**: Framework for evaluating agentic systems. Covers quantitative and qualitative metrics, standardized benchmarks and end-to-end evaluation methodologies.

**advanced-evaluation**: LLM-as-a-Judge techniques with bias mitigation. Delves into the use of language models as evaluators, addressing known biases (position bias, verbosity bias, self-enhancement bias) and mitigation strategies.

**project-development**: End-to-end methodology for the development of LLM projects. Complete guide from concept to production, with quality checkpoints and decision gates.

#### Cognitive Architecture

**bdi-mental-states**: Formal Belief-Desire-Intention modeling for deliberative reasoning. This is the most sophisticated pattern: it transforms external RDF context into the agent's mental states (beliefs, desires, intentions) using formal BDI ontologies. It allows for explainable and deliberative reasoning, where the agent does not merely react to stimuli but reasons about its own beliefs, formulates desires and generates action intentions.

### Design Principles

**Progressive Disclosure**: Skills load incrementally. Initially, agents load only names and descriptions; the full content activates when relevant. This optimizes the use of the attention budget.

**Platform Agnosticism**: The principles transfer between Claude Code, Cursor and any agent platform that supports skills.

**Conceptual + Practical**: Each skill combines theoretical foundations with executable Python pseudocode.

### Notable Implementations

The repository includes complete implementations that demonstrate the principles in action:

- **digital-brain-skill**: Personal operating system with 6 modules and 4 automation scripts.
- **llm-as-judge-skills**: TypeScript evaluation tool with 19 passing tests.
- **book-sft-pipeline**: Training pipeline for stylistic writing (70% human score at $2 cost).
- **x-to-book-system**: Multi-agent monitoring and synthesis system.

---

## 6. Synthesis of Common Patterns

Analyzing the four repositories transversally, recurring patterns emerge that represent the consensual core of the agentic discipline.

### Universal Patterns (present in all 4 repositories)

| Pattern | promptadvisers | nibzard | arunpshankar | muratcankoylan |
|---------|---------------|---------|--------------|----------------|
| **Reflection/Self-Critique** | Pattern #4 | Feedback Loops | Actor-Critic | context-degradation |
| **Tool Use** | Pattern #5 | Tool Use & Env | Web Access | tool-design |
| **Multi-Agent** | Pattern #7 | Orchestration | Parallel Delegation | multi-agent-patterns |
| **Memory/Context** | Pattern #8 | Context & Memory | State across agents | memory-systems, context-* |
| **Human-in-the-Loop** | Pattern #13 | UX & Collab | -- | -- |
| **Planning/Decomposition** | Pattern #6 | Orchestration | Task Decomposition | project-development |

### Convergent Patterns

1. **Iterative Reflection**: All repositories recognize that single-shot generation is insufficient. The generate-evaluate-revise loop is the most fundamental pattern.

2. **Multi-Agent Orchestration**: Decomposition into specialized agents coordinated by an orchestrator is the dominant architectural pattern.

3. **Context Management as a Discipline**: Not only does muratcankoylan make this explicit, but all repositories implicitly address the problem of what to put in the context window and how to structure it.

4. **Parallelism as a Requirement**: Parallel execution is not an optimization: it is an architectural requirement for systems operating at scale.

5. **Security and Guardrails**: All repositories devote significant attention to security, recognizing that an agent without constraints is a risk, not an innovation.

---

## 7. Unified Taxonomy of Agentic Patterns

Based on the cross-repository analysis, we propose a unified taxonomy that organizes patterns into 6 architectural levels, from the lowest (infrastructure) to the highest (strategy).

### Level 1: Computational Foundations

Patterns that govern basic interaction with the language model.

- **Prompt Chaining**: sequencing of model calls.
- **Reasoning Techniques**: CoT, ToT, GoT to guide reasoning.
- **Context Engineering**: holistic curation of the context window.
- **Context Compression**: compaction for long-running sessions.
- **Resource-Aware Optimization**: management of tokens, costs, latency.

### Level 2: Single Agent Capabilities

Patterns that define what a single agent knows how to do.

- **Tool Use**: integration of external capabilities.
- **Reflection**: self-evaluation and iterative improvement.
- **Memory Management**: persistence and retrieval of context.
- **Planning**: strategic decomposition of goals.
- **Learning and Adaptation**: improvement over time.
- **BDI Mental States**: formal deliberative reasoning.

### Level 3: Multi-Agent Coordination

Patterns governing interaction between agents.

- **Multi-Agent Collaboration**: coordination of agents with distinct roles.
- **Routing/Semantic Routing**: intelligent routing of requests.
- **Parallel Delegation**: concurrent delegation to sub-agents.
- **Inter-Agent Communication**: message exchange protocols.
- **Sub-Agent Spawning**: dynamic creation of sub-agents.
- **DAG Orchestration**: workflows structured as acyclic graphs.

### Level 4: Workflow Management

Patterns that structure the overall flow of work.

- **Dynamic Sharding**: data partitioning for parallel processing.
- **Task Decomposition / Dynamic Decomposition**: static and dynamic decomposition.
- **Discrete Phase Separation**: distinct phases with explicit transitions.
- **Planner-Worker Separation**: dual architecture for long-running tasks.
- **Goal Setting and Monitoring**: tracking goals and progress.

### Level 5: Reliability and Security

Patterns ensuring robustness and security in production.

- **Exception Handling and Recovery**: error management and recovery.
- **Guardrails/Safety Patterns**: operational constraints and action limits.
- **Schema Validation Retry**: formal validation with retry.
- **Evaluation and Monitoring**: metrics, logs, tracing.
- **Zero-Trust Agent Mesh**: zero-trust security between agents.
- **PII Tokenization**: protection of personal data.

### Level 6: Human-Agent Interaction

Patterns governing collaboration with human users.

- **Human-in-the-Loop**: human feedback at critical points.
- **Spectrum of Control**: mixed human-agent initiative.
- **Background-to-Foreground Handoff**: transparent transitions.
- **Prioritization**: priority management in collaboration.
- **Exploration and Discovery**: guided exploration with supervision.

### Interactions Between Levels

The levels are not isolated. The most important interactions:

- **Level 1 -> 2**: the quality of context engineering determines the effectiveness of all higher-level patterns.
- **Level 2 -> 3**: the reflection capability of a single agent improves the quality of multi-agent coordination.
- **Level 3 -> 4**: multi-agent orchestration enables complex workflows.
- **Level 5 transversal**: reliability and security must permeate all levels.
- **Level 6 -> all**: human feedback can intervene at any level.

---

## 8. How to Use These Repositories -- Practical Guide

### For Beginners

1. **Start with promptadvisers**: read the discussions of the 21 patterns to build the conceptual vocabulary. The Mermaid diagrams make the concepts immediately visualizable.

2. **Deepen with nibzard**: use the Pattern Explorer on agentic-patterns.com to explore the 8 categories. Start with Orchestration & Control and Context & Memory, which are the foundations.

3. **Implement with arunpshankar**: choose one of the 8 Python patterns and get it running locally. Start with the Reflection pattern (the simplest) and progress towards DAG Orchestration (the most complex).

4. **Integrate with muratcankoylan**: add context engineering skills to your workflow. Start with `context-fundamentals` and `context-degradation`.

### For Architects of Agentic Systems

1. **Define requirements**: which patterns are needed for your use case? Use the unified taxonomy (Section 7) to identify the levels involved.

2. **Select patterns**: for each level, choose the appropriate patterns. Use the nibzard catalogue to compare alternatives.

3. **Validate with code**: implement a proof-of-concept using arunpshankar's templates. Adapt to your stack (not necessarily Google Cloud).

4. **Optimize context**: apply muratcankoylan's skills to ensure that context is managed as an engineering resource, not as an afterthought.

### For Production Teams

1. **Security first**: start with security patterns (Level 5). Zero-Trust Agent Mesh, PII Tokenization and Guardrails are not optional.

2. **Observability**: implement Evaluation and Monitoring from the beginning. Debugging agentic systems without observability is impossible.

3. **Human-in-the-Loop as default**: start with strong human supervision and reduce it gradually as trust in the system grows.

4. **Contribute**: all four repositories accept contributions. If you implement an undocumented pattern, share it with the community.

### Recommended Pattern Combinations

**Research Agent**: Routing + Web Access + Reflection + RAG
- The router determines the type of search, web access retrieves information, reflection evaluates quality, RAG enriches with internal knowledge.

**Coding Agent**: Planning + Tool Use + Reflection + Human-in-the-Loop
- Planning decomposes the task, tools (file system, terminal) execute, reflection verifies the code, the human approves critical changes.

**Data Analysis Pipeline**: Dynamic Sharding + Parallel Delegation + DAG Orchestration + Evaluation
- Sharding partitions the data, parallel delegation processes, the DAG coordinates the flow, evaluation verifies the results.

**Enterprise Multi-Agent System**: Semantic Routing + Multi-Agent Collaboration + Memory Management + Zero-Trust + Monitoring
- Routing dispatches requests, agents collaborate, memory maintains context, security protects, monitoring observes.

---

## 9. Resources and References

### Primary Repositories

| # | Repository | URL | Focus |
|---|-----------|-----|-------|
| 40 | promptadvisers/agentic-design-patterns-docs | https://github.com/promptadvisers/agentic-design-patterns-docs | 21 patterns with diagrams |
| 41 | nibzard/awesome-agentic-patterns | https://github.com/nibzard/awesome-agentic-patterns | Catalogue of 97+ patterns |
| 42 | arunpshankar/Agentic-Workflow-Patterns | https://github.com/arunpshankar/Agentic-Workflow-Patterns | 8 Python/GCP patterns |
| 43 | muratcankoylan/Agent-Skills-for-Context-Engineering | https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering | 13 context engineering skills |

### Related Resources

- **Interactive Site**: https://www.agentic-patterns.com/ -- Pattern Explorer, side-by-side comparison, decision guides
- **Medium Article**: "Designing Cognitive Architectures: Agentic Workflow Patterns from Scratch" by Arun Shankar (Google Cloud Community)
- **Book**: "Agentic Design Patterns: A Hands-On Guide to Building Intelligent Systems" (Springer, October 2025) -- implementations with LangChain/LangGraph, CrewAI and Google ADK
- **SitePoint Guide**: "Agentic Design Patterns: The 2026 Guide to Building Autonomous Systems" -- https://www.sitepoint.com/the-definitive-guide-to-agentic-design-patterns-in-2026/
- **Architect's Guide**: "Architect's Guide to Agentic Design Patterns" by Sunil Rao (Medium, Data Science Collective)

### Complementary Repositories

- **zeljkoavramovic/agentic-design-patterns**: 29 agentic patterns (extension of the 21)
- **ColtMercer/agentic-design-patterns**: Summaries and cheat sheets of Antonio Gulli's 21 patterns
- **Mathews-Tom/Agentic-Design-Patterns**: Alternative implementation of agentic patterns
- **practice.learnagenticpatterns.com**: Practical exercises for building AI agents

### Related Academic Publications

- "Applying Cognitive Design Patterns to General LLM Agents" (arXiv:2505.07087)
- "Agentic Artificial Intelligence: Architectures, Taxonomies, and Evaluation of Large Language Model Agents" (arXiv:2601.12560)
- "Agentic AI: A Comprehensive Survey of Architectures, Applications, and Future Directions" (arXiv:2510.25445)
- "Agentic AI Frameworks: Architectures, Protocols, and Design Challenges" (arXiv:2508.10146)
- AWS Prescriptive Guidance: "From Event-Driven to Cognition-Augmented Systems"

### Notes on the Taxonomy

The unified taxonomy proposed in Section 7 is an attempt at synthesis based on the analysis of the four repositories. It is not a standard: it is a working framework that can be adapted, extended and contextualized. The repositories themselves use different taxonomies (5 categories for promptadvisers, 8 for nibzard, flat for arunpshankar, 4 groups for muratcankoylan), reflecting different perspectives on the same domain.

---

*Document generated on March 26, 2026. Repository data (stars, forks, number of patterns) is subject to update.*
