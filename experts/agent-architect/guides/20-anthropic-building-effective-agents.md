# Building Effective Agents - The Definitive Guide by Anthropic

> **Primary source:** [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) - Anthropic Engineering Blog
> **Authors:** Erik Schluntz and Barry Zhang (Anthropic)
> **Publication date:** December 19, 2024
> **Status:** Foundational reference guide for the architecture of agentic systems

---

## 1. Overview - Context, Authors, and Relevance

### Why This Guide Matters

In December 2024, at the height of the "AI agents" explosion, Anthropic published what quickly became the reference guide for anyone building LLM-based systems. While the market was flooded with complex frameworks and over-engineered architectures, Erik Schluntz and Barry Zhang -- two engineers with direct experience in building agents like Claude -- took a counterintuitive and radical stance: **simplicity wins**.

The central thesis of the guide can be summed up in a single sentence:

> *"Success in the LLM space isn't about building the most sophisticated system -- it's about building the right system for your needs."*

This document is not an academic exercise. It comes from Anthropic's concrete experience in building real products -- from automatic GitHub issue resolution (SWE-bench) to computer use automation. The authors worked directly on these systems and the guide reflects lessons learned "in the trenches".

### Who the Authors Are

**Erik Schluntz** is a senior engineer at Anthropic specialized in agentic systems and tool use. He has worked on the practical implementation of agents that solve real code problems, contributing to the SWE-bench benchmarks where Claude demonstrates the ability to tackle GitHub issues starting solely from the pull request description.

**Barry Zhang** works on Anthropic's Applied AI team, where he focuses on bringing Claude's agentic capabilities into production contexts. His experience covers both architecture design and the optimization of model-tool interactions.

### The Industry Context

At the time of publication, the industry was experiencing a significant "agent hype" phase. Every startup was promising autonomous agents, frameworks like LangChain and CrewAI offered increasingly complex abstractions, and many development teams were finding themselves building unnecessarily elaborate systems. Anthropic's guide arrived as an antidote: you don't need an agent for everything, and when you do need one, the simplest version is almost always the right one.

As highlighted by [independent analyses of the guide](https://www.agentsdecoded.com/p/an-analysis-of-anthropics-guide-to), the document's main merit lies precisely in addressing the primary risk in agent development: teams building unnecessarily complex systems to look innovative rather than to satisfy functional requirements.

---

## 2. Workflows vs. Agents - The Fundamental Distinction

### Anthropic's Definitions

Anthropic introduces an architectural distinction that underpins the entire document:

**Workflow:** A system in which LLMs and tools are orchestrated through predefined code paths. The flow is determined by the developer in advance. The LLM executes specific tasks within a rigid and predictable structure.

**Agent:** A system in which the LLM dynamically directs its own processes and tool usage, maintaining control over how it completes tasks. The LLM autonomously decides which steps to take, which tools to use, and when to stop.

The key difference lies in the **locus of control**: in workflows, control resides in the code written by the developer; in agents, control resides in the model.

### Practical Implications

This distinction is not purely theoretical. It has direct consequences on design:

| Aspect | Workflow | Agent |
|---------|----------|-------|
| Flow control | Predefined code | LLM decides dynamically |
| Predictability | High | Low |
| Latency | More controllable | Potentially high |
| Cost | Predictable | Variable (can explode) |
| Debugging | Relatively simple | Complex |
| Use cases | Structured, repeatable tasks | Open-ended, unpredictable problems |

### Critiques of the Distinction

It's worth noting that this separation isn't universally accepted. As highlighted in the [Agents Decoded analysis](https://www.agentsdecoded.com/p/an-analysis-of-anthropics-guide-to), the definitions present some inconsistencies. For example, a routing workflow that produces results identical to an LLM-with-tool system shouldn't be considered "second class" merely because of where the decision occurs. The main critique is that the definitions should focus on *what* agents are (goal-oriented design components) rather than on *how* they are implemented (LLM dependency). This technology-based distinction may not age well.

Despite these critiques, the distinction remains very useful as a mental framework for deciding **how much** control to delegate to the model and **how much** to keep in deterministic code.

---

## 3. The 5 Building Blocks - The Heart of the Guide

### The Foundation: The Augmented LLM

Before describing the five patterns, Anthropic establishes the foundational building block: the **Augmented LLM**. This is an LLM enhanced with three capabilities:

1. **Retrieval** - Access to external knowledge bases (RAG, web search, databases)
2. **Tools** - Capability to perform actions in the real world (APIs, file system, browser)
3. **Memory** - Capability to retain and recall information across sessions

Modern models can actively use these capabilities: they generate their own search queries, select appropriate tools, and determine which information to retain. One approach to implement these augmentations is Anthropic's **Model Context Protocol (MCP)**, which standardizes integration with third-party tools through a single client implementation.

All five subsequent patterns are built on this foundation. It's not about orchestrating "naked" LLMs, but LLMs already equipped with these augmented capabilities.

---

### 3.1 Prompt Chaining - The Sequential Chain

#### Concept

Prompt Chaining decomposes a task into a sequence of steps, where each LLM call processes the output of the previous call. Between one step and the next, it's possible to insert **programmatic gates** -- quality checkpoints that verify the output is adequate before proceeding.

#### Conceptual Diagram

```
[Input] --> [LLM Call 1] --> [Gate/Check] --> [LLM Call 2] --> [Gate/Check] --> [Output]
                                  |                                  |
                                  v                                  v
                             [Fallback/                          [Fallback/
                              Retry]                              Retry]
```

The flow is linear and predetermined. Each block knows its task, and gates serve as intermediate validators.

#### When to Use It

- The task is **decomposable into fixed, sequential subtasks**
- You prefer to **trade latency for accuracy**, making each individual call a simpler task
- You need **quality checkpoints** between phases
- The process is **repeatable and predictable**

#### Practical Examples

**Marketing content generation:** A first LLM generates the copy in the original language, a gate verifies that it complies with brand guidelines, a second LLM translates into the target language, and a final gate verifies the correctness of the translation.

**Generation of structured documents:** A first LLM creates an outline of the document, a gate validates the structure and completeness, then a second LLM generates the full content based on the approved outline.

**Blogging pipeline:** As described in [the AIMultiple analysis](https://aimultiple.com/building-ai-agents), a first LLM generates the structured outline, a second evaluates it against predefined criteria, and a third handles the final publication.

#### Strengths and Limitations

**Strengths:** Each step is simpler and more accurate; gates guarantee quality; the flow is easily debuggable; latency is predictable (sum of the individual steps).

**Limitations:** The structure is rigid; it doesn't handle tasks well that require iteration or dynamic adaptation; if the task doesn't decompose cleanly into sequential phases, forcing it into this pattern creates problems.

---

### 3.2 Routing - Classification and Dispatch

#### Concept

Routing classifies the input and directs it toward specialized downstream tasks. The idea is **triage**: a first LLM (or even a simpler classifier) determines the nature of the request and routes it to the most appropriate handler. This allows each category to be optimized separately.

#### Conceptual Diagram

```
                         +--> [Specialized Handler A] --> [Output A]
                         |
[Input] --> [Classifier] +--> [Specialized Handler B] --> [Output B]
                         |
                         +--> [Specialized Handler C] --> [Output C]
```

The classifier operates as a dispatcher: it analyzes the input, determines the category, and routes it to the handler with the most suitable prompt and skills.

#### When to Use It

- Inputs of **different nature** require specialized handling
- You need **per-category optimization** (different prompts, different models, different tools)
- The cost/complexity of each category is significantly **different**
- You want to implement **load balancing** across systems

#### The Customer Service Example

This is the canonical example of the guide. A customer support system receives requests of very different nature:

- **General questions** --> Handler with FAQ knowledge base, fast model (e.g., Claude Haiku 4.5)
- **Refund requests** --> Handler with access to the order system, refund policies, ability to execute transactions
- **Technical support** --> Handler with technical documentation, system logs, diagnostic capabilities

Each handler has its own optimized prompt, its own tools, and potentially its own model. Routing avoids overloading a single prompt with all these responsibilities.

#### Routing for Cost/Speed Optimization

A particularly elegant use of routing is **dynamic model selection**. Simple and common queries are routed to faster and cheaper models (like Claude Haiku), while complex or unusual queries are sent to more capable models (like Claude Sonnet). This simultaneously optimizes cost, latency, and quality.

#### Strengths and Limitations

**Strengths:** Separation of responsibilities; each handler can be optimized independently; eases testing; scales well with the addition of new categories.

**Limitations:** Requires a reliable classifier; ambiguous inputs may be misclassified; routing quality is a single point of failure.

---

### 3.3 Parallelization - Simultaneous Execution

#### Concept

Parallelization allows multiple LLMs to work **simultaneously** on the same task, with results aggregated programmatically. Anthropic identifies two fundamental variations:

**Sectioning:** The task is split into **independent** subtasks that are executed in parallel. Each LLM works on a different portion of the problem.

**Voting:** The **same task** is executed **multiple times** (potentially with different prompts) to obtain different outputs. The results are then aggregated to obtain greater confidence or coverage.

#### Conceptual Diagram - Sectioning

```
                 +--> [LLM - Subtask A] --+
                 |                          |
[Input] --------+--> [LLM - Subtask B] --+--> [Aggregator] --> [Output]
                 |                          |
                 +--> [LLM - Subtask C] --+
```

#### Conceptual Diagram - Voting

```
                 +--> [LLM - Prompt 1] --+
                 |                        |
[Input] --------+--> [LLM - Prompt 2] --+--> [Aggregator/Voting] --> [Output]
                 |                        |
                 +--> [LLM - Prompt 3] --+
```

#### When to Use It

- The subtasks are **genuinely independent** (sectioning)
- You need **multiple perspectives** to increase confidence (voting)
- **Speed** is critical and the task is parallelizable
- Complex problems that benefit from **focused reasoning** on separate dimensions

#### Practical Examples

**Safety guardrails (Sectioning):** One LLM processes the user's query normally, while in parallel another dedicated LLM performs content screening. Both operate simultaneously without waiting for each other. If the screener identifies problematic content, the processor's output is blocked.

**Automated evaluation (Sectioning):** Multiple LLMs simultaneously evaluate different aspects of a system's performance: one analyzes accuracy, another completeness, a third style, a fourth conformance to guidelines.

**Code vulnerability review (Voting):** Multiple different prompts analyze the same code looking for vulnerabilities. The aggregation of multiple perspectives catches problems that a single prompt might miss.

**Content appropriateness evaluation (Voting):** Multiple prompts evaluate whether content is appropriate. If the percentage of "flags" exceeds a predefined threshold, the content is blocked.

#### Strengths and Limitations

**Strengths:** Drastic latency reduction (in sectioning); greater reliability and coverage (in voting); easy to implement; each parallel flow is independent and debuggable.

**Limitations:** Cost multiplied by the number of parallel calls; in sectioning, only works if subtasks are genuinely independent; in voting, the majority is not always right.

---

### 3.4 Orchestrator-Workers - The Most Powerful Pattern

#### Concept

This is the pattern Anthropic considers the most powerful among workflows. A central **orchestrator LLM** analyzes the task, dynamically decomposes it into subtasks, delegates them to specialized **worker LLMs**, and then synthesizes the results into a coherent output.

The crucial difference compared to Parallelization: in parallelization the subtasks are predefined by code; in Orchestrator-Workers, **subtasks are determined dynamically by the orchestrator** based on the specific input. The orchestrator "decides" what is needed, how many workers to activate, and how to integrate the results.

#### Conceptual Diagram

```
                                    +--> [Worker A] --+
                                    |                  |
[Input] --> [Orchestrator LLM] -----+--> [Worker B] --+--> [Orchestrator] --> [Output]
                |                   |                  |         ^
                |                   +--> [Worker C] --+          |
                |                                                |
                +-------- analyzes, decomposes, synthesizes ----+
```

The orchestrator has a global view of the problem. It determines which workers are needed, provides each with the necessary context, gathers the results, and integrates them.

#### When to Use It

- The number and nature of subtasks **are not predictable in advance**
- You need **flexibility** in adapting to very different inputs
- The task requires **coordination** among multiple components
- The decomposition of the problem itself requires **intelligence**

#### Practical Examples

**Complex modifications to a codebase:** This is the most illuminating example in the guide. When an agent must modify a codebase, the number of files involved and the nature of the modifications depend entirely on the specific task. The orchestrator analyzes the request, identifies the relevant files, determines the necessary modifications for each, delegates workers for each file, and then verifies overall consistency.

**CEO Agent (from external analyses):** As described in [the AIMultiple analyses](https://aimultiple.com/building-ai-agents), a CEO orchestrator receives a brief, decides which departments to involve (Marketing, Operations, Finance), refines the inputs for each department, invokes the workers as tools, and compiles a complete plan integrating all the outputs.

**Multi-source synthesis:** The orchestrator determines which sources to consult, delegates research and extraction to specialized workers, and then synthesizes the information into a coherent report.

#### Why It's the Most Powerful Pattern

The power of Orchestrator-Workers lies in its **adaptability**. It is the pattern that comes closest to the behavior of a true agent, while still maintaining the structure of a workflow. The orchestrator is not an autonomous agent -- the general flow (analyze, decompose, delegate, synthesize) is predefined -- but the decomposition is dynamic. This offers an excellent compromise between flexibility and controllability.

#### Strengths and Limitations

**Strengths:** High flexibility; handles complex and unpredictable tasks well; each worker can be specialized; the orchestrator maintains the overall view.

**Limitations:** High computational cost (the orchestrator itself requires a powerful LLM); the quality of decomposition depends entirely on the orchestrator; debugging is more complex because flows are dynamic; potentially high latency.

---

### 3.5 Evaluator-Optimizer - The Refinement Loop

#### Concept

In the Evaluator-Optimizer pattern, a **generator** LLM produces a response, and another **evaluator** LLM analyzes it and provides feedback. If the response does not meet the quality criteria, the feedback is sent back to the generator for an improvement iteration. The cycle continues until the evaluator approves the output.

It is essentially an **iterative feedback cycle** that emulates the human process of review and refinement.

#### Conceptual Diagram

```
[Input] --> [Generator LLM] --> [Response] --> [Evaluator LLM]
                 ^                                    |
                 |                              [Approved?]
                 |                              /          \
                 |                            Yes           No
                 |                            |              |
                 |                       [Output]      [Feedback]
                 |                                          |
                 +------------------------------------------+
```

The loop continues until the evaluator approves, or until a maximum number of iterations is reached (to avoid infinite loops).

#### When to Use It

- There are **clear and articulable evaluation criteria**
- **Iterative refinement** produces demonstrable improvements
- **Output quality** is critical and justifies the cost of multiple iterations
- The task has quality standards that can be verified programmatically or by an LLM

#### Practical Examples

**Literary translation:** The generator produces the translation, the evaluator verifies that it captures linguistic nuances, emotional tone, cultural references. Feedback guides successive iterations that progressively refine the translation. This is a case where the first version is almost always "good enough" but the value lies in the excellence achieved through iterations.

**Complex multi-round research:** The generator collects and synthesizes information. The evaluator determines if coverage is sufficient, if there are gaps, if additional sources are needed. The loop continues until research is deemed complete. In each iteration, the generator knows exactly what is missing thanks to the evaluator's feedback.

**Code generation with tests:** The generator writes the code, the evaluator (which could even be the execution of the tests themselves) verifies correctness. If the tests fail, the feedback includes the specific error messages, guiding the generator toward correction.

#### Evaluation Criteria

The strength of this pattern depends entirely on the quality of the evaluation criteria. Anthropic suggests that criteria must be:

- **Specific and measurable**: not "write well" but "the text must not exceed 200 words and must contain at least 3 citations"
- **Articulable in natural language**: the LLM evaluator must be able to understand and apply them
- **Discriminating**: they must clearly distinguish between acceptable and unacceptable output
- **Actionable**: the feedback must indicate *what* to improve, not just *that* something is wrong

#### Strengths and Limitations

**Strengths:** Produces output of significantly higher quality; the feedback cycle is transparent and inspectable; emulates a natural human review process.

**Limitations:** Cost multiplied by the number of iterations; risk of infinite loops if criteria are poorly defined; can converge toward "average good" outputs rather than excellent ones if the evaluator is not discriminating enough; high latency.

---

## 4. When to Use a True Agent - The Decision Framework

### Anthropic's Golden Rule

> *"Start by using LLM APIs directly: many patterns are implementable in just a few lines of code."*

Anthropic's decision framework is clear and intentionally conservative:

### Level 0: Single LLM Call (No Pattern)

**Always start here.** A single LLM call with a good prompt, possibly with retrieval, solves the vast majority of use cases. Do not deploy an agent for something that a well-written prompt can do.

### Level 1: Simple Workflows

If a single call isn't enough, first try the simplest workflows: prompt chaining or routing. These add minimal structure with maximum benefit.

### Level 2: Complex Workflows

If more sophisticated patterns are needed (parallelization, orchestrator-workers, evaluator-optimizer), implement them only when you have **measurable evidence** that they add value. Do not assume: test and compare.

### Level 3: Autonomous Agent

Use a true agent **only when**:

- The problem is **open-ended and unpredictable** -- the necessary steps cannot be coded in advance
- You need **real autonomy** -- the agent must autonomously decide what to do based on environmental feedback
- The domain is **trusted and circumscribed** -- the agent operates in an environment with adequate guardrails
- The **value justifies the costs** -- true agents are significantly more expensive and slower

### Signals That an Agent Is Appropriate

- Resolution of real GitHub issues (as demonstrated in SWE-bench)
- Computer automation for multi-step tasks
- Complex customer support requiring real actions (refunds, order updates)
- Coding agent where solutions are verifiable through tests

### Signals That an Agent Is NOT Appropriate

- The task is structured and repeatable
- The steps are predictable in advance
- A deterministic pipeline would work just as well
- The cost of compounding errors is too high
- There is no reliable mechanism for verifying outputs

---

## 5. Agents in Production - Operational Best Practices

### The Three Cardinal Principles

Anthropic identifies three non-negotiable principles for agents in production:

1. **Simplicity** -- Minimal agent design; complexity only where demonstrably necessary
2. **Transparency** -- Explicit visibility into the agent's planning and decision phases
3. **Documentation** -- Comprehensive documentation and thorough testing of tools

### Tool Design: The Agent-Computer Interface

This is one of the most valuable contributions of the guide. Anthropic explicitly states:

> *"The investment in creating good agent-computer interfaces is on par with the effort needed for human-computer interfaces."*

Tool design is not an implementation detail: it is a design discipline in its own right. Anthropic, in its work on SWE-bench, spent **more time optimizing tools than the overall prompt**.

As elaborated in Anthropic's complementary guide "[Writing Tools for Agents](https://www.anthropic.com/engineering/writing-tools-for-agents)", the key principles are:

#### Naming and Organization

- **Coherent namespaces**: group related tools under consistent prefixes (`asana_search`, `asana_create` or `asana_projects_search`, `asana_users_search`)
- **Clear and distinct purpose**: each tool must have a unique purpose -- if a human engineer cannot immediately tell which tool to use in a given situation, an AI agent cannot do better
- **Avoid overly broad tool sets**: bloated tool sets create ambiguity and distract the agent from efficient strategies

#### Parameters and Response Format

- **Unambiguous parameter names**: use `user_id` instead of `user` to reduce hallucinations
- **Consolidate operations**: instead of separate `list_users`, `list_events`, `create_event`, implement `schedule_event` that handles multiple steps internally
- **Flexible response format**: offer a `response_format` parameter with "concise" and "detailed" options
- **High-signal data**: avoid low-level technical identifiers (UUIDs, MIME types); expose semantically meaningful fields

#### Tool Documentation

This is identified as **one of the most effective methods to improve tools**:

- Explain context that an expert might take for granted
- Avoid ambiguity through rigorous data models
- Function as if you were explaining the tool to a new team member
- Even small refinements to tool descriptions can produce dramatic improvements

Anthropic reports a concrete example: when they improved the description of the web search tool to clarify how the query parameter should work, Claude stopped unnecessarily adding "2025" to search terms.

### Error Handling

Error handling for agents requires a radically different approach from traditional software:

- **Actionable errors**: error responses must clearly communicate specific and correctable problems, not opaque codes or tracebacks
- **Constructive feedback**: instead of "Error: Invalid parameter", guide the agent toward the correct usage pattern by suggesting specific filtering strategies or properly formatted inputs
- **Behavioral steering**: truncation messages can encourage more efficient tool-use patterns, pushing the agent to perform multiple targeted searches rather than a single broad query

### Human-in-the-Loop

Human integration remains essential, especially in production:

- **Approval checkpoints**: the agent can pause to request human feedback at critical points or when it encounters blockers
- **Ground truth from the environment**: at each step, the agent must obtain real feedback from the environment to evaluate its progress
- **Human review**: automated tests verify functionality, but human review remains crucial to ensure that solutions are aligned with broader system requirements
- **Escalation**: clearly define when the agent should ask for help rather than proceed autonomously

### Production Deployment Checklist

Based on Anthropic's recommendations, an evaluation framework should consider:

1. **Task complexity**: does the task justify an agent or would a simpler workflow suffice?
2. **Value justification**: is the additional cost justified by the value produced?
3. **Capability validation**: is the agent capable of executing the required subtasks reliably?
4. **Cost of errors**: how serious is an agent error? What are the consequences?
5. **Sandboxing**: does the agent operate in an isolated environment where potential damages are contained?
6. **Monitoring**: do metrics exist for monitoring agent performance in real time?
7. **Scalability**: is the design stateless, with session history and long-term memory in external systems?

---

## 6. Anti-Patterns - What NOT to Do

### Anti-Pattern 1: Premature Complexity

This is the main anti-pattern identified by Anthropic and the central message of the guide:

> *"Only increase complexity when it demonstrably improves outcomes."*

Many teams make the mistake of starting immediately with complex agentic architectures when a single LLM call with a good prompt would have solved the problem. The impulse to build sophisticated systems is strong, but almost always counterproductive.

**Signals of premature complexity:**
- You are using an agentic framework before having tried a single optimized prompt
- You have more than 3 levels of nesting in your LLM chains
- You cannot explain why each component of the system is necessary
- You don't have metrics demonstrating the added value of complexity

### Anti-Pattern 2: Over-reliance on Frameworks

Anthropic is explicit on this point:

> *"Frameworks often create additional layers of abstraction that obscure prompts and responses, making debugging more difficult. They tend to add unnecessary complexity."*

The specific risk: frameworks like LangChain or CrewAI may seem productive at first, but their abstractions often hide unexpected behaviors. When something doesn't work, debugging through layers of abstraction is significantly harder than debugging direct code.

Anthropic's recommendation: if you use a framework, **understand the underlying code**. Do not assume that abstractions work the way you expect. Incorrect assumptions about framework abstractions are among the most common errors.

It must be said, as highlighted by [critiques of the guide](https://www.agentsdecoded.com/p/an-analysis-of-anthropics-guide-to), that this position is perhaps too drastic. Frameworks embody the collective wisdom of a community, accelerate development, and prevent reinventing the wheel. A balanced approach combining both frameworks and direct APIs is probably more practical than dismissing frameworks entirely.

### Anti-Pattern 3: Bloated Tool Sets

A tool set that is too broad or has overlapping functionality is one of the most common failure modes. If the agent must choose among 50 tools with similar purposes, the probability of error increases dramatically. Each tool must have:
- A clear and distinct purpose
- No significant overlap with other tools
- Documentation that makes it unambiguous when to use it

### Anti-Pattern 4: Mechanical API Wrapping

Don't mechanically expose all the endpoints of an API as tools. Interfaces for agents are fundamentally different from traditional software interfaces. Intentional design is needed that considers how the agent perceives and uses the tools.

### Anti-Pattern 5: Autonomy Without Guardrails

Agents with too much autonomy and few guardrails are a recipe for disaster. Costs can explode (each iteration costs), errors compound (an early error infects all subsequent decisions), and behavior becomes unpredictable. Every agent in production needs:
- Budget limits (maximum number of iterations/tokens)
- Execution sandbox
- Real-time monitoring
- Human escalation mechanisms

### Anti-Pattern 6: Ignoring Tool Testing

Anthropic reports that in their work on SWE-bench, they spent more time optimizing tools than the overall prompt. A specific example: switching from relative paths to **absolute paths** in the file system tools eliminated an entire class of errors. Tools must be tested with diverse inputs, edge cases, and unexpected formats.

### Anti-Pattern 7: Inadequate Response Formats

The choice of format (XML, JSON, Markdown) affects performance because LLMs are trained on next-token prediction and tend to perform better with formats that match their training data. There is no universal solution: you must evaluate with your own specific use cases, selecting formats that:
- Provide sufficient tokens for the model's reasoning
- Use formats that occur naturally in internet text
- Eliminate formatting overhead (precise line counts, string escaping)

---

## 7. Real-World Applications: The Validated Cases

### Coding Agents

The use case where agents have demonstrated the most convincing value. Anthropic cites SWE-bench, where Claude-based agents resolve real GitHub issues starting solely from the pull request description. Coding agents are particularly effective because:
- Solutions are **verifiable through tests**
- Agents can **iterate using feedback** from failed tests
- The problem space is **structured** (language, file system, syntax)
- Quality is **objectively measurable**

### Customer Support

Customer support is described as a "natural fit" for agents because it combines conversation with action. Success factors include: natural conversational flow, access to external tools, ability to perform programmatic actions (refunds, updates), and measurable resolutions. Significantly, Anthropic notes that some companies use usage-based pricing (charging only for successful resolutions) as an indicator of confidence in the system's effectiveness.

---

## 8. Resources and References

### Primary Anthropic Sources

- [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) - The original guide (December 2024)
- [Writing Tools for Agents](https://www.anthropic.com/engineering/writing-tools-for-agents) - Complementary guide on tool design
- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Context engineering for agents
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) - Managing long-running agents
- [Anthropic Cookbook - Agent Patterns](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents) - Practical implementations

### Analyses and Commentary

- [An Analysis of Anthropic's Guide to Building Effective Agents](https://www.agentsdecoded.com/p/an-analysis-of-anthropics-guide-to) - Detailed critical analysis
- [Building AI Agents with Anthropic's 6 Composable Patterns](https://aimultiple.com/building-ai-agents) - Practical guide with implementations
- [Learning from Anthropic about Building Effective Agents](https://maa1.medium.com/learning-from-anthropic-about-building-effective-agents-2a7469941428) - Medium analysis
- [Cloudflare Agents - Anthropic Patterns](https://github.com/cloudflare/agents/blob/main/guides/anthropic-patterns/README.md) - Cloudflare implementation of the patterns

### Related Concepts

- **Model Context Protocol (MCP)**: Anthropic's standard for integrating tools with LLMs
- **SWE-bench**: Benchmark for the automatic resolution of real software issues
- **Computer Use**: The ability of LLMs to directly control graphical interfaces

---

## Summary: The Fundamental Takeaways

1. **Start simple, add complexity only with evidence.** The majority of problems are solved with a single well-designed LLM call.

2. **The 5 building blocks are composable.** They are not mutually exclusive alternatives but bricks that combine. A real system can use routing at the first level, prompt chaining in one branch, and parallelization in another.

3. **Tool design is design, not wrapping.** Investing in tool design produces returns superior to any architectural optimization.

4. **Transparency above all.** An agent whose reasoning is visible and inspectable is infinitely more useful than a "magical" agent that produces opaque results.

5. **Guardrails and human-in-the-loop are not optional.** Every agent in production needs limits, monitoring, and the possibility of human escalation.

6. **Measure, don't assume.** Every increment of complexity must be justified by metrics. If you cannot demonstrate that the agent is better than the workflow, use the workflow.

This Anthropic guide remains the most pragmatic and operational reference document available for those building agentic systems. Its strength does not lie in theoretical innovation, but in the engineering discipline it promotes: build the minimum necessary, measure, and iterate.
