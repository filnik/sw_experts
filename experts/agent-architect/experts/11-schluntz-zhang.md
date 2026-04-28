# Erik Schluntz & Barry Zhang -- "Building Effective Agents"

## Anthropic's reference guide for the design of agentic systems

---

## 1. Profile and Background

### Erik Schluntz

Erik Schluntz is a Member of Technical Staff at Anthropic, where he works on Large Language Models, tool use, computer use, and SWE-Bench. His professional path positions him as one of the most qualified figures at the intersection between robotics, automation, and applied artificial intelligence.

**Academic background.** Schluntz earned both a Bachelor's and a Master's in Electrical Engineering at Harvard University, both with honors (Cum Laude, 2015). During his studies he followed a strongly interdisciplinary path, ranging across artificial intelligence, computer science, electrical engineering, mechanical engineering, and economics.

**Professional path before Anthropic.** Schluntz's career is studded with experiences in technological frontier companies:

- **SpaceX (2015):** Worked in the Flight Software Team, testing new power systems for the Falcon 9 rockets. This experience gave him a deep understanding of mission-critical systems and the importance of rigorous testing.
- **Google[x] (2014):** Prototyped applications for low-power electronics in the Smart Contact Lens project, working at the intersection between advanced hardware and software.
- **Posmetrics (Founder):** Founded an iPad-based customer feedback platform, accepted into the Y Combinator Winter 2013 program and subsequently acquired. This entrepreneurial experience gave him sensitivity toward product-market fit and design simplicity.
- **Cobalt Robotics (Co-founder and CTO, 2016-2024):** This was the most formative experience before Anthropic. As co-founder and CTO, Schluntz built AI-powered security and inspection robots -- 1.5-meter-tall robots that patrolled offices and warehouses autonomously. The company reached Series C with investors such as Sequoia, with over 100 robots deployed globally. Years of work on autonomous navigation and safety guardrails in self-driving systems forged his philosophy: give the model maximum control possible, while keeping the scaffolding minimal.

**Key contributions to Anthropic.** Schluntz implemented the system that achieved state-of-the-art results on SWE-bench Verified (49%, beating the previous record of 45%) using a deliberately simple version of the SWE-Agent framework. The architecture consisted only of a prompt, a Bash tool, and an Edit tool -- demonstrating concretely that simplicity works. He also worked on Claude's function calling, tool use, and computer use APIs, contributing to defining how language models interact with external tools.

### Barry Zhang

Barry Zhang is Research Engineer and Member of Technical Staff in Anthropic's Applied AI team, where he focuses on developing agentic systems with enterprise companies and startups.

**Academic background.** Zhang earned degrees in Computer Science and Industrial Engineering at Northwestern University, a background that combines computational skills with an engineering approach to complex systems.

**Professional path before Anthropic:**

- **Splunk (Software Developer):** Worked as a software developer, specializing in resolving performance bottlenecks through the development of creative and lightweight testing software. This experience rooted in him the importance of simple and effective solutions.
- **Meta (Tech Lead, Monetization GenAI Team):** Led the GenAI monetization team, where he claimed the inaugural title of "AI Engineer". Working on Meta's knowledge graph, he developed expertise in large-scale data systems that proved valuable for the design of agentic architectures.

**Key contributions to Anthropic.** Zhang is co-author of two fundamental publications: "Building Effective Agents" (December 2024, with Erik Schluntz) and "Equipping agents for the real world with Agent Skills" (October 2025, with Keith Lazuka and Mahesh Murag). The latter introduces the Skills framework, a system for the dynamic loading of procedural knowledge that allows building a universal agent powered by a library of skills, instead of multiple specialized agents. Zhang presented this approach at the AI Engineering Code Summit 2025 with the talk "Don't Build Agents, Build Skills Instead".

### The Tandem

Schluntz and Zhang represent a complementary combination: Schluntz's experience in robotics and autonomous systems provides insights on model-environment interaction and on the design of tool-model interfaces, while Zhang's background in enterprise systems and industrial engineering brings architectural pragmatism and sensitivity toward production deployment needs. The "Building Effective Agents" guide is born precisely from this convergence, enriched by the experiences of building agents internally at Anthropic and by the lessons learned working with the company's clients.

---

## 2. Philosophy: "Simple over Complex"

The founding principle of Schluntz and Zhang's guide is condensed in a phrase that should be printed and posted in every AI development team:

> *"Success in the LLM space isn't about building the most sophisticated system. It's about building the right system for your needs."*

This is not an abstract maxim. It is the distillate of hundreds of hours spent building agents at Anthropic and observing how clients -- from enterprise to startup -- approach the problem. The conclusion is counterintuitive in an industry that celebrates complexity: **most applications do not need complex agentic systems**.

### The Recommended Progression

Schluntz and Zhang propose a rigorous progression:

1. **Start from optimized single LLM calls.** Before any multi-step architecture, one must exhaust the possibilities of a single call to the model, enriched with retrieval and in-context examples. Surprisingly, for many applications this is sufficient.

2. **Add complexity only when demonstrably necessary.** The key word is "demonstrably". Complexity is not added because "it might be needed" or because "the paper says to do it". It is added when the results of the simple solution are measurably insufficient.

3. **When scaling, use composable patterns.** The five patterns we will see are not monoliths to be implemented integrally. They are building blocks that combine and customize for the specific use case.

### Against Premature Frameworks

Another strong position of the guide concerns agentic frameworks. Schluntz and Zhang do not condemn them in absolute terms, but warn against adopting them prematurely:

> Most patterns can be implemented in a few lines of code. If you use a framework, you must understand the underlying code. Layers of abstraction can obscure prompts and responses, complicating debugging and tempting toward unnecessary complexity.

This position arises from direct observation: teams that start from abstract frameworks often lose contact with what is actually happening between the prompt and the model's response, making debugging a nightmare and optimization almost impossible.

### The Fundamental Building Block: the Augmented LLM

All patterns rest on a common foundation: the concept of **Augmented LLM**. A modern LLM does not operate in isolation but is enriched with three fundamental capabilities:

- **Retrieval:** The model can generate search queries to access external information.
- **Tools:** The model can select and invoke tools to interact with external systems.
- **Memory:** The model can determine what information to keep for future reference.

The concrete implementation requires adapting these capabilities to the specific use case, providing clear and well-documented interfaces. Schluntz and Zhang also recommend the Model Context Protocol (MCP) for integration with third-party tool ecosystems.

---

## 3. The 5 Composable Patterns

These five patterns represent the core of the guide. Each addresses a distinct architectural problem and can be combined with the others. Schluntz and Zhang present them in increasing order of complexity.

### 3.1 Prompt Chaining -- Sequential Pipelines with Gates

**Fundamental concept.** Prompt Chaining decomposes a task into a sequence of steps, where each LLM call processes the output of the previous one. It is the most intuitive pattern: a linear chain of transformations, where each link produces the input for the next.

**Operating mechanism.** The flow is linear: Input --> LLM Call 1 --> Gate --> LLM Call 2 --> Gate --> ... --> Output. The crucial element that distinguishes Prompt Chaining from a simple sequence of prompts is the concept of **gate**: programmatic checks inserted between one step and the next to verify that the process is still on the correct trajectory. A gate can be a syntactic validation, a length check, a format verification, or any other condition that must be satisfied before proceeding.

**When to use it:**
- The task is clearly and naturally decomposable into fixed and sequential sub-tasks.
- One prefers to sacrifice latency (more calls = more time) in exchange for greater accuracy, making each individual LLM call an easier and circumscribed task.
- Each step in the chain has a verifiable output before proceeding to the next.

**Concrete examples:**

1. **Multilingual marketing:** Generate marketing text --> Gate (verify tone and length) --> Translate into target language. By separating generation and translation, each LLM call can focus on a single objective, improving overall quality.

2. **Structured writing:** Write an outline of the document --> Gate (validate structure and completeness) --> Write the complete document based on the outline. The intermediate gate allows correcting the outline before investing tokens in the full writing.

3. **Code generation with review:** Generate code --> Gate (run linter/static tests) --> Automatic review and correction. The programmatic gate (linter) provides objective feedback without requiring another LLM call.

**Key design principle.** Each step in the chain must be simple enough to be executed with high reliability by a single LLM call. If a step is too complex, it probably needs to be further decomposed. Gates must be as programmatic as possible (not LLM-based) to avoid introducing further uncertainty.

### 3.2 Routing -- Classification and Routing of Inputs

**Fundamental concept.** Routing classifies an incoming input and directs it to a specialized downstream task. It is an intelligent sorting pattern: instead of building a single monolithic prompt that handles all possible cases, a classifier is created that directs each input to the most appropriate path.

**Operating mechanism.** The flow has a fan shape: Input --> LLM Classifier --> { Route A (specialized LLM/prompt), Route B (specialized LLM/prompt), Route C (specialized LLM/prompt) }. The classifier analyzes the input and determines the category to which it belongs. Each downstream route has its own prompt, its own set of tools, and potentially its own model optimized for that specific type of request.

**When to use it:**
- The overall task comprises distinct categories that are better handled separately, with different prompts and logics.
- The classification can be performed with high accuracy (a router that errs frequently is worse than a non-routed system).
- The different categories benefit from independent optimizations (separation of concerns).

**Concrete examples:**

1. **Multi-tier customer service:** Customer queries are classified into categories (general questions, refund requests, technical support). Each category has a specialized prompt with specific instructions, access to different tools (the refund path accesses the payment system, the technical path accesses the technical knowledge base), and potentially different models.

2. **Cost-performance optimization:** Easy or common questions are routed to smaller and faster models (such as Claude Haiku), while complex or unusual questions are directed to more capable models (such as Claude Sonnet). This optimizes both the cost and the overall quality of the system.

3. **Document workflow:** Incoming documents are classified by type (invoice, contract, email, report) and each type is processed with a specialized pipeline that knows exactly which information to extract and in which format.

**Key design principle.** The quality of routing depends entirely on the quality of the classifier. Investing time in the classifier is fundamental: an incorrect routing not only produces poor results for that request, but can have cascade effects on the entire system. The main advantage of routing is the **separation of concerns** -- optimizing one path does not degrade the performance of the others.

### 3.3 Parallelization -- Sectioning and Voting

**Fundamental concept.** Parallelization executes multiple LLM calls simultaneously on the same task or on independent components of a task, then aggregating the results programmatically. It is the pattern that exploits concurrency to improve speed, reliability, or both.

**The two fundamental variants:**

#### Sectioning

The task is divided into independent sub-tasks that are executed in parallel. The final output is the aggregation of partial results.

**Operating mechanism.** Input --> { LLM Call A (subtask 1), LLM Call B (subtask 2), LLM Call C (subtask 3) } --> Aggregator --> Output.

**Concrete examples of Sectioning:**

1. **Safety guardrails in parallel:** One instance of the model processes the user's query normally, while another simultaneous instance analyzes the same query for policy violations or inappropriate content. The results are combined: if the guardrail detects a violation, the response is blocked or modified independently of what the main instance produced.

2. **Multi-aspect automatic evaluation:** To evaluate a complex output (for example an article or a piece of code), parallel evaluations are launched on different dimensions: one call evaluates technical correctness, another expository clarity, another completeness, another style compliance. Each evaluation is independent and benefits from having the entire model's attention capacity focused on a single aspect.

3. **Multi-field data extraction:** From a complex document, information of different categories is extracted simultaneously (personal data, financial information, legal clauses), each with a specialized prompt.

#### Voting

The same identical task is executed multiple times to obtain different outputs. The results are then aggregated (typically by majority or via a scoring model).

**Operating mechanism.** Input --> { LLM Call 1 (same prompt), LLM Call 2 (same prompt), LLM Call 3 (same prompt) } --> Aggregator (majority/scoring) --> Output.

**Concrete examples of Voting:**

1. **Multi-perspective code review:** The same code is analyzed by multiple LLM calls with slightly different prompts (or the same prompt that produces different outputs thanks to temperature). If multiple reviews identify the same vulnerability, confidence in the report is very high.

2. **Content moderation with threshold:** The same content is evaluated multiple times for appropriateness. A voting threshold (for example, 3 out of 5 evaluators consider it inappropriate) determines the action to take, drastically reducing both false positives and false negatives.

**Fundamental insight.** Schluntz and Zhang underline an important principle: LLMs perform better when each individual consideration receives the dedicated attention of a separate call. Concentrating too many evaluation criteria in a single prompt degrades the quality on each criterion. Parallelization solves this problem without penalizing latency.

### 3.4 Orchestrator-Workers -- Dynamic Delegation

**Fundamental concept.** A central LLM (the orchestrator) dynamically analyzes the task, decomposes it into sub-problems, delegates them to worker LLMs, and then synthesizes the results. Unlike Parallelization, where the sub-tasks are predefined in the code, here it is the orchestrator itself that decides which sub-tasks to create and how to distribute them.

**Operating mechanism.** Input --> Orchestrator LLM (analyzes, plans, decomposes) --> { Worker LLM A (dynamic subtask), Worker LLM B (dynamic subtask), Worker LLM C (dynamic subtask) } --> Orchestrator LLM (synthesizes) --> Output.

**Critical difference compared to Parallelization.** In Parallelization, sub-tasks are hardcoded: the developer knows a priori that task X is decomposed into sub-tasks A, B, and C, and writes code that launches them in parallel. In Orchestrator-Workers, sub-tasks emerge from the orchestrator's analysis. It is not known in advance how many there will be, what their content will be, or how they will be distributed. This makes the pattern more flexible but less predictable.

**When to use it:**
- The sub-tasks cannot be predicted in advance (fixed paths cannot be defined).
- The nature and number of sub-tasks change based on the specific input.
- Flexibility is essential to handle the variability of real-world inputs.

**Concrete examples:**

1. **Large-scale coding.** A coding product must make complex modifications to multiple files. The orchestrator analyzes the change request, identifies which files must be modified (which vary from request to request), generates specific instructions for each worker, and each worker modifies its file. The orchestrator then verifies the overall coherence of the modifications.

2. **Multi-source research and analysis.** A research task requires gathering and analyzing information from multiple sources. The orchestrator decides which sources to query (based on the specific query), sends workers to search in each source, and then synthesizes the gathered information into a coherent response, potentially launching further searches if it finds informational gaps.

3. **Complex documentary analysis.** A long and articulated document must be analyzed. The orchestrator identifies the relevant sections, assigns each worker the analysis of a specific section with specific criteria, and then composes an overall evaluation that takes into account the contributions of all the workers.

**Key design principle.** The orchestrator must have a complete view of the context to be able to effectively decompose the task. Workers must receive sufficiently detailed instructions to be able to operate autonomously. The orchestrator's synthesis phase is critical: it must not only aggregate the results, but also identify inconsistencies, gaps, or errors in the workers' contributions.

### 3.5 Evaluator-Optimizer -- Iterative Improvement Loop

**Fundamental concept.** One LLM generates responses (the generator/optimizer) while another provides evaluation and feedback (the evaluator) in an iterative loop. The process continues until the evaluator judges the result satisfactory or a limit of iterations is reached.

**Operating mechanism.** Input --> Generator LLM --> Output v1 --> Evaluator LLM --> Feedback --> Generator LLM --> Output v2 --> Evaluator LLM --> Feedback --> ... --> Final output (approved by the Evaluator).

**Illuminating analogy.** Schluntz and Zhang compare this pattern to the iterative review process in writing: a writer produces a draft, an editor provides feedback, the writer revises, the editor reevaluates, and so on until the desired quality is reached.

**When to use it:**
- Clear and articulable evaluation criteria exist (the evaluator must know what to look for).
- Iterative improvement produces measurable value (not all tasks benefit from iteration).
- Human feedback demonstrably improves the LLM's responses (if a human would not be able to give useful feedback, probably neither could an LLM evaluator).
- The LLM is able to provide actually useful feedback on the type of output in question.

**Concrete examples:**

1. **Literary translation.** Translating literary texts requires capturing linguistic nuances, tone, registers, wordplay. The generator produces a translation, the evaluator analyzes it for fidelity to the original text, naturalness in the target language, and preservation of stylistic elements. The feedback guides the generator toward successive revisions that progressively capture more nuances.

2. **Complex multi-round research.** A research query requires multiple iterations of analysis. The generator performs searches and produces summaries, the evaluator determines whether the gathered information is sufficient or whether further searches are needed. The loop continues until the evaluator determines that the response is complete and well supported.

3. **Code generation with testing.** The generator produces code, the evaluator runs tests (programmatic, not LLM-based in this case) and provides the results of failed tests as feedback. The generator iterates based on the specific errors, converging toward a solution that passes all tests.

**Key design principle.** The evaluator must be able to provide specific and actionable feedback, not just a binary "good/bad" judgment. The more detailed and directional the feedback, the faster the generator will converge toward a quality result. It is also fundamental to establish a maximum iteration limit to avoid infinite loops in case of tasks for which the model cannot converge.

---

## 4. Workflows vs Agents -- The Fundamental Distinction

Schluntz and Zhang introduce an architectural distinction that permeates the entire guide and is essential for making correct design decisions.

### Workflow

A workflow is a system in which LLMs and tools are orchestrated through predefined code paths. The developer explicitly defines the sequence of operations, the decision points, and the conditional flows. The LLM operates within a fixed structure designed by the human engineer.

**Characteristics:**
- Predictable and testable execution paths.
- The developer maintains control of the flow.
- Easier to debug, monitor, and optimize.
- Ideal for well-defined tasks with limited variability.

The first four patterns (Prompt Chaining, Routing, Parallelization, Orchestrator-Workers) tend toward the "workflow" side of the spectrum, although Orchestrator-Workers approaches the boundary zone.

### Agent

An agent is a system in which the LLM dynamically directs its own processes and tool use, maintaining control over how to accomplish the task. The LLM autonomously decides which actions to take, in which order, and when to stop.

**Characteristics:**
- Flexibility in tackling open and unpredictable problems.
- The LLM operates over many turns, making decisions at each step.
- "Ground truth" is acquired from the environment at each step (tool results, code execution output, API responses).
- Stops at checkpoints or blocks, with completion criteria or iteration limits.
- May require human input when it encounters ambiguity or high-risk decisions.

**The implementation reality is surprisingly simple.** Schluntz and Zhang make a crucial observation that dismantles much of the hype around agents:

> *"Agents can handle sophisticated tasks, but their implementation is often straightforward. They are typically just LLMs using tools based on environmental feedback in a loop."*

An agent, in its essence, is an LLM that in a loop: (1) observes the state of the environment, (2) decides on an action, (3) executes the action through a tool, (4) observes the result, and (5) repeats until the task is complete or a limit is reached.

### The Trade-offs

Agents offer greater flexibility but entail:
- **Higher costs** (more LLM calls, more tokens consumed).
- **Potential compounding of errors** (each LLM decision introduces an error margin that accumulates).
- **Need for extensive testing** in sandboxed environments with appropriate guardrails.

---

## 5. When to Use What -- Practical Decision Framework

Schluntz and Zhang provide a decision framework that helps choose the appropriate level of complexity for each situation.

### Level 0: Optimized Single LLM Call

**When:** The application needs single-turn responses, where quality can be improved with prompt engineering, retrieval (RAG), and in-context examples.

**Signals that it is sufficient:** Most queries are handled adequately; errors are manageable with more precise prompts; latency is a priority.

### Level 1: Workflow with Composable Patterns

**When to move here:** The single call does not produce sufficiently good results on a significant portion of cases. The task has an internal structure that can be exploited.

**Choose the pattern based on the task structure:**

| Situation | Recommended Pattern |
|---|---|
| Task decomposable into fixed sequential steps | **Prompt Chaining** |
| Input with distinct categories needing different treatments | **Routing** |
| Independent sub-tasks executable simultaneously | **Parallelization (Sectioning)** |
| Need for high confidence through multiple perspectives | **Parallelization (Voting)** |
| Sub-tasks not predictable a priori, dependent on input | **Orchestrator-Workers** |
| Quality improvable through iterative review | **Evaluator-Optimizer** |

### Level 2: Autonomous Agent

**When to move here:**
- The required steps cannot be predicted in advance.
- Fixed paths cannot be defined in code.
- The task requires the LLM to operate over many turns making decisions at each step.
- Trust exists in the model's decision-making capability for the domain in question.
- The cost and latency trade-offs are accepted.

### The Guiding Principle

At each level, measure performance and iterate. Do not move up a level by default or out of fashion. Added complexity must be justified by a measurable and significant improvement compared to the previous level.

---

## 6. Implementation Patterns -- Concrete Code and Architecture

### The Agent-Computer Interface (ACI)

One of the most original and practical contributions of the guide concerns the concept of **Agent-Computer Interface (ACI)**. Schluntz and Zhang argue that the design of tool-model interfaces deserves the same attention that has historically been dedicated to the Human-Computer Interface (HCI).

**ACI design principles:**

1. **Intrinsic clarity.** The use of the tool must be obvious from its description and parameters. A good tool definition includes usage examples, edge cases, input format requirements, and clear boundaries with respect to other tools.

2. **Deliberate naming.** The names of parameters and their descriptions must be sufficiently explanatory to make use obvious without ambiguity.

3. **Iterative testing.** Run many example inputs to identify the errors the model makes, and iterate on the tool definition. This process is analogous to usability testing in human interfaces.

4. **Poka-yoke (error prevention).** Restructure tool arguments to make errors structurally impossible or difficult to commit.

**Case study: SWE-bench.** Schluntz's implementation for SWE-bench spent more time optimizing tools than general prompts. A concrete example: requiring absolute file paths instead of relative paths eliminated the model's confusion regarding directory context. This single change produced a measurable improvement in performance.

### Architecture of a Minimal Agent

The architecture used for SWE-bench is paradigmatic of the Schluntz-Zhang approach:

```
Components:
  - A detailed but not excessively long system prompt
  - A Bash tool (command execution with instructions on escaping,
    network restrictions, background processes in the description itself)
  - An Edit tool (viewing, creating, and modifying files
    via string replacement, with exact match of old_str to
    prevent errors)

Main loop:
  1. The model receives the context (problem, current state)
  2. The model decides the action (bash command, file edit, or "I'm done")
  3. The action is executed and the result is returned to the model
  4. Repeat until the model declares completion or the
     limit of 200k tokens is reached

Principle: no forced discrete steps. The model transitions freely
between exploration, coding, and testing.
```

The simplicity of this architecture -- which achieved state-of-the-art results -- is the most convincing proof of the "simple over complex" philosophy.

### Principles for Tool Format

Schluntz and Zhang identify specific principles for designing tool formats:

1. **Provide sufficient tokens for "reasoning".** The model must have space to "think" before committing to an action. Formats that require immediate decisions without elaboration space degrade quality.

2. **Maintain natural formats.** Formats must be close to text the model has seen during training (internet text). Artificial or highly structured formats can confuse the model.

3. **Eliminate cognitive overhead.** Do not require the model to perform operations that are difficult for it: counting lines, handling string escaping, calculating offsets. If the tool can handle these details internally, it must do so.

### Three Fundamental Architectural Principles

The entire guide converges on three principles that must guide every implementation decision:

1. **Simplicity.** Keep the agent design as simple as possible. Each additional component is a point of potential failure and an obstacle to debugging.

2. **Transparency.** Show planning steps explicitly. The agent must not be a black box: its reasoning and decisions must be inspectable.

3. **Documentation and rigorous testing of tools.** The ACI must be curated with the same attention with which one curates the user interface of a consumer product.

### Validated Practical Applications

The guide identifies two domains where agents have demonstrated concrete value:

**Customer Support** is a natural fit because: conversations flow naturally while accessing external information; tools can extract customer data, order history, knowledge base; programmatic actions are possible (refunds, ticket updates); success is clearly measurable. Anthropic's confidence in this domain is reflected in their usage-based pricing model (one pays only for successful resolutions).

**Coding** has notable potential because: solutions are verifiable through automated tests; agents can iterate using test feedback; the problem space is well defined; quality is objectively measurable. The results on SWE-bench Verified confirm this potential, although human review remains crucial for alignment with the broader system.

---

## 7. Resources and References

### Primary Publications

- **"Building Effective Agents"** (December 2024) -- Erik Schluntz, Barry Zhang. The complete guide to composable patterns for agentic systems.
  Source: https://www.anthropic.com/research/building-effective-agents

- **"SWE-bench Sonnet"** -- Details on Schluntz's state-of-the-art implementation for SWE-bench.
  Source: https://www.anthropic.com/engineering/swe-bench-sonnet

- **"Equipping agents for the real world with Agent Skills"** (October 2025) -- Barry Zhang, Keith Lazuka, Mahesh Murag. The Skills framework for universal agents.

### Implementation Resources

- **Anthropic Cookbook -- Agents Patterns:** Official repository with notebooks and recipes for Prompt Chaining, Routing, Parallelization, Orchestrator-Workers, Evaluator-Optimizer.
  Source: https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents

- **Claude Agent SDK:** Official SDK for building agents with Claude.
  Source: https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk

- **Model Context Protocol (MCP):** Protocol for integration with third-party tool ecosystems.

### Talks and Podcasts

- **"The new Claude 3.5 Sonnet, Computer Use, and Building SOTA Agents"** -- Erik Schluntz on Latent Space Podcast. In-depth discussion on computer use, tool design, and the minimal approach to agents.
  Source: https://www.latent.space/p/claude-sonnet

- **"Don't Build Agents, Build Skills Instead"** -- Barry Zhang and Mahesh Murag at the AI Engineering Code Summit 2025. Presentation of the Skills paradigm.

### Community Analyses and Commentary

- Simon Willison -- Detailed analysis of the guide: https://simonwillison.net/2024/Dec/20/building-effective-agents/
- Cloudflare Agents -- Implementation of Anthropic patterns: https://github.com/cloudflare/agents

### Author Profiles

- **Erik Schluntz:** Personal site (https://erikschluntz.com/), former Co-founder and CTO of Cobalt Robotics, Harvard University (EE, BS+MS Cum Laude).
- **Barry Zhang:** Research Engineer Anthropic, former Tech Lead Meta (Monetization GenAI), Northwestern University (CS + Industrial Engineering).

---

*This document is part of the series of profiles on agentic architecture experts. Schluntz and Zhang's "Building Effective Agents" guide is considered the fundamental industry reference for the design of agentic systems, and its influence extends well beyond the Anthropic ecosystem.*
