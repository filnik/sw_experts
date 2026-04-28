# Effective Context Engineering for AI Agents

## In-Depth Guide based on Anthropic's publication

**Primary source:** [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) -- Anthropic Engineering Blog
**Complementary source:** [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) -- Anthropic Engineering Blog
**Original publication date:** September 2025
**Last update of this guide:** March 2026

---

## 1. Overview: What Context Engineering Is

### From Prompt Formulation to Context Design

**Context engineering** represents a fundamental evolution in how developers interact with Large Language Models. While prompt engineering focuses on the linguistic formulation of instructions -- "how should I phrase this sentence?" -- context engineering addresses an architecturally deeper question: **"what configuration of context has the highest probability of generating the desired behavior from the model?"**

This distinction is not merely semantic. Prompt engineering treats the model as an interpreter of textual instructions; context engineering treats it as a cognitive system with real computational constraints. The context window is not a passive container but a **finite resource of attention** that requires careful and strategic curation.

### Why Context Engineering Surpasses Prompt Engineering

The fundamental reason context engineering emerged as a distinct discipline in 2025 lies in the growing complexity of AI agents. An agent that must navigate a codebase of millions of lines, interact with dozens of tools, and maintain coherence across hundreds of conversation turns cannot be effectively governed solely by the quality of a system prompt. Architectural strategies are needed to manage:

- **The complete information state**: system instructions, tool definitions, external data, message history, intermediate results.
- **Attention degradation**: the "context rot" phenomenon, where the model's accuracy progressively diminishes as tokens in the window increase.
- **Dynamic resource allocation**: deciding in real time which information deserves space in the context window and which can be offloaded, compressed, or eliminated.

As Anthropic observes, the transformer architecture creates pairwise relationships between every pair of tokens (n-squared complexity), which means that as context grows, every individual token must "compete" with all the others for the model's attention. Performance does not collapse suddenly at a fixed threshold but degrades gradually -- making the problem insidious because the loss of quality can pass unnoticed.

The guiding principle of context engineering is therefore: **maximize desired outcomes through the minimum set of high-signal informative tokens**. It is not about filling the window, but about curating it.

---

## 2. Fundamental Principles

### 2.1 Just-in-Time Context Loading

The principle of **just-in-time loading** represents perhaps the most significant contribution of Anthropic's guide. Instead of pre-processing and loading all relevant data into the context window before execution, the approach is to maintain **lightweight identifiers** (file paths, URLs, database queries) and dynamically load data only when actually needed during execution.

This approach mirrors human cognition: an engineer does not memorize the entire codebase before working on a bug. Rather, they keep a mental map of the project structure and consult specific files when needed. Similarly, a well-designed AI agent should:

1. **Maintain a lightweight index** of available resources rather than their full contents.
2. **Progressively explore** the context through search tools (grep, glob, selective file reading).
3. **Load only the relevant fragments** at the moment of decision.

**The canonical example** is Claude Code, which incorporates CLAUDE.md files into the initial context to provide high-level orientation, but uses grep and glob for the just-in-time retrieval of specific files during execution.

The **trade-off** is clear: runtime exploration is slower than pre-computed retrieval, but it avoids the problems of stale indexing and initial overload. For complex tasks with extensive codebases, the net benefit is significantly positive.

**Recommended hybrid approach:** Retrieve some data upfront for speed (for example, project structure, naming conventions, key configuration files) while enabling autonomous exploration for everything else.

### 2.2 Context Window Management

Context window management is the operational heart of context engineering. Anthropic identifies several critical aspects:

**Structured organization of the system prompt:**
- Use distinct, clearly delimited sections: background, instructions, tool guidance, expected output description.
- Structure with XML tags or Markdown headers to facilitate parsing by the model.
- Find the "right altitude" between overly prescriptive instructions (fragile hardcoded logic) and overly vague ones (which assume nonexistent shared context).

**Token budget management:**
The context window must be thought of as a **budget** to be strategically allocated across different categories:

| Category | Typical Allocation | Notes |
|---|---|---|
| System prompt | 5-15% | Stable and structural instructions |
| Tool definitions | 10-20% | Descriptions, parameters, examples |
| Retrieved context (RAG) | 15-30% | Relevant external data |
| Conversation history | 20-40% | Previous messages (subject to compaction) |
| Workspace | 15-25% | Space for the model's reasoning |

These percentages vary drastically based on the use case, but the principle remains: every category competes for the same finite budget and requires continuous optimization.

**Designing tools for efficiency:**
Tools must be designed as first-class citizens in context engineering:
- **Minimize functional overlap**: if a human cannot identify which tool to use in a situation, neither will the agent.
- **Return token-efficient information**: a tool that returns 10,000 tokens of raw JSON when 500 tokens of synthesis would suffice is wasting context budget.
- **Be self-contained**: extremely clear purpose, descriptive and unambiguous input parameters.

### 2.3 Information Density Optimization

Information density optimization is the principle that ties all the others together. Every token in the context window must **maximize the signal-to-noise ratio**. This implies:

**Curating few-shot examples:**
Anthropic recommends curating **diverse and canonical examples** rather than exhaustive lists of edge cases. Examples are "the images worth a thousand words" -- a single well-chosen example can convey more context than paragraphs of explicit instructions.

**Progressive disclosure through information architecture:**
Information should be organized so that agents can discover it incrementally:
- File sizes suggest complexity.
- Naming conventions indicate purpose.
- Timestamps approximate relevance.

This information architecture allows agents to self-manage their context windows, autonomously deciding what merits deeper investigation.

**Eliminating redundancy:**
A common mistake is keeping in context tool results already processed, intermediate data no longer needed, or previous versions of already-updated information. The discipline of actively removing superfluous context is just as important as that of adding useful context.

---

## 3. Practical Techniques

### 3.1 Compaction and Summarization

**Compaction** is the most critical technique for agents operating on long conversations. It consists of summarizing the conversation when it approaches the context window limits, reinitializing it with a compressed summary that preserves the essential information.

**Two-phase implementation strategy:**

**Phase 1 -- Maximize recall:**
In the first compaction pass, the goal is to capture all potentially relevant information. It is better to include something marginal than to risk losing a critical detail. This includes:
- Architectural decisions made during the conversation.
- Unresolved bugs and failed resolution attempts.
- Specific implementation details (variable names, file paths, configurations).
- Constraints and requirements that emerged during the interaction.

**Phase 2 -- Iterate for precision:**
Subsequent passes refine the summary, removing superfluous content. In this phase the following are removed:
- Raw tool results already integrated into the conclusions.
- Reasoning attempts that led to dead ends (preserving however the conclusion that that path is unfeasible).
- Verbose formatting and repetitions.

**The tool result clearing functionality** (available on the Claude Developer Platform) allows raw tool results to be eliminated once they have been processed and the relevant information has been extracted. This is particularly effective for tools that return large JSON payloads or extensive search results.

**Risks of compaction:**
Aggressive compaction can cause the loss of subtle but critical context. For example, an apparently marginal user comment about a style preference might be eliminated during compaction, causing inconsistency in subsequent responses. The calibration of the compression level requires iteration and testing.

### 3.2 Tool Result Processing

The processing of tool results is an often-overlooked area that has an enormous impact on context efficiency. The main strategies are:

**Pre-insertion filtering:**
Before inserting a tool result into the context window, apply filters that extract only the information relevant to the current query. A search tool that returns 50 results might require only the top 5 most relevant to enter the context.

**Structured formatting:**
Transform raw results into compact, structured formats. For example, a nested JSON of 2000 tokens could be represented as a 300-token table without significant information loss.

**Result expiration:**
Tool results have a "shelf life" -- their utility diminishes with temporal distance (in terms of conversation turns) from the moment they were generated. Implementing expiration mechanisms that remove or compact old results is fundamental.

**Deduplication:**
When an agent calls the same tool with similar parameters at different times, the results may overlap significantly. Intelligent deduplication systems prevent the context window from being filled with redundant information.

### 3.3 Memory Management in Long Conversations

Memory management in long conversations requires a layered approach that Anthropic articulates in three complementary mechanisms:

**Intra-context memory (Short-term):**
The information in the current context window. Subject to compaction and managed through the techniques already described.

**Persistent external memory (Long-term):**
The agent writes notes to storage external to the context window, retrieving them later. This enables **persistent memory with minimal overhead**. Applications include:
- Tracking progress on complex tasks.
- Maintaining critical dependencies between components.
- Creating to-do lists and incremental knowledge bases.
- Persisting state across context resets.

The paradigmatic example cited by Anthropic is that of Claude playing Pokemon: the system maintains precise counts across thousands of steps, tracking objectives and strategic notes in external storage without the need for explicit prompting.

**Artifact-based memory (Episodic):**
In software development systems, Git history, progress files, and structured feature lists serve as **episodic memory**. Each new session reconstructs the context from these artifacts, allowing the agent to resume work from where it left off.

The **Memory Tool** (in public beta on the Claude platform) implements this pattern by allowing the construction of file-based knowledge bases and the persistence of project state across multiple sessions.

### 3.4 Context Window Budget Allocation

Context window budget allocation must be **dynamic and adaptive**, not static. Advanced strategies include:

**Allocation based on task phase:**
- **Exploration phase**: allocate more space to tool results and context discovery.
- **Planning phase**: allocate more space to reasoning and plan structuring.
- **Execution phase**: allocate more space to specific instructions and constraints.
- **Verification phase**: allocate more space to test results and comparison with requirements.

**Relevance-based prioritization:**
Not all information in context has the same importance. Implementing a scoring system that assigns different weights to different categories of information allows, during compaction, to preserve the highest-value information.

**Emergency reserve:**
Always keep a portion of the context window unallocated (10-15%) as an "emergency reserve" to handle unexpected situations, unexpectedly large tool results, or the need for extended reasoning on complex problems.

---

## 4. Patterns for Long-Running Agents

### 4.1 Maintaining Coherence on Long Tasks

The fundamental challenge of long-running agents is that they operate in **discrete sessions**, where every new session starts with an empty context. Anthropic has identified that filling hundreds of thousands of tokens of history into the window causes degradation of reasoning capability on what really matters.

**Two-agent architecture (Initializer + Worker):**

The approach proposed by Anthropic in their Agent SDK provides:

1. **Initializer Agent**: The first session uses specialized prompting to build the foundational environment:
   - `init.sh` script for the development infrastructure.
   - `claude-progress.txt` file to document activities.
   - Initial Git commit that establishes the baseline state.

2. **Worker Agent (Coding Agent)**: Subsequent sessions focus on incremental work while maintaining clean state. Each task is treated as an independent contribution, mergeable into production.

**Initial validation pattern:**
Each session starts with prescribed steps:
- Verification of the working directory (`pwd`).
- Reading Git logs and progress files to reconstruct context.
- Validation of application functionality through automated tests before implementing new features.

This prevents agents from inheriting broken code and wasting tokens debugging inherited problems.

**Structured closing pattern:**
Each session concludes with:
- Committing changes with descriptive messages.
- Updating progress documentation.
- Marking completed features in the structured list.

### 4.2 Episodic Memory Management

Episodic memory for AI agents is implemented through **external artifacts** that persist between sessions:

**Git as episodic memory:**
Git history becomes the chronological record of changes. Descriptive commit messages serve as "memories" that the agent can consult to understand what was done, when, and why. In case of regressions, the agent can use `git revert` to return to working states of the code.

**Progress files as semantic memory:**
Files like `claude-progress.txt` provide readable summaries of the current situation, completed objectives, and next steps. These files are optimized to be read quickly at the start of each session, providing orientation without requiring reading the entire history.

**Feature lists as ground truth:**
Structured JSON-formatted lists establish the ground truth on the project's scope. The agent receives explicit instructions not to modify this file except to update completion status, preventing goal drift.

**Context reconstruction protocol:**
At the start of each session, the agent:
1. Reads the feature list to understand the overall scope.
2. Reads the progress file to understand the current state.
3. Consults recent Git logs for implementation details.
4. Runs tests to validate the state of the code.

Only after this reconstruction phase does the actual work begin.

### 4.3 Checkpoint and Recovery

Checkpoint and recovery patterns are essential for the resilience of long-running agents:

**Git-based checkpoints:**
Every significant unit of work is committed with a message that captures not only the "what" but also the "why" of the change. This enables:
- **Granular rollback**: returning to any previous point in the work.
- **Speculative branching**: exploring alternative solutions without risking work already done.
- **Complete audit trail**: understanding the agent's decision logic in retrospect.

**Recovery from corrupted states:**
When an agent inherits a broken state (failing tests, missing dependencies), the protocol provides:
1. Identification of the broken state through automated tests.
2. Consultation of the Git log to find the last working state.
3. Revert or cherry-pick to restore a valid state.
4. Only after restoration, proceed with new work.

**Multi-agent architecture for resilience:**
The sub-agent approach provides specialized agents with isolated context windows:
- Each sub-agent explores extensively (tens of thousands of tokens) but returns condensed summaries (1,000-2,000 tokens).
- The lead agent coordinates high-level planning, receiving only the syntheses.
- The failure of a sub-agent does not corrupt the context of others.

This pattern has demonstrated substantial improvements compared to single-agent systems for complex research tasks, precisely because the separation of contexts prevents the cross-contamination of errors.

---

## 5. Common Errors

### 5.1 Context Stuffing

**Context stuffing** -- filling the context window with as much information as possible in the hope that the model will "find what it needs" -- is the most widespread anti-pattern. The negative effects are multiple:

- **Performance degradation**: the model loses focus on critical information, "drowned" in a sea of irrelevant data.
- **Increased costs**: more input tokens means higher API costs without proportional benefit.
- **Slower responses**: increased latency for processing useless tokens.
- **"Needle in a haystack" problem**: important information buried in large quantities of text is missed even by powerful models.

The classic temptation is to insert an exhaustive list of edge cases into the system prompt, trying to articulate every possible rule. This approach is counterproductive: it transforms the prompt into a sprawling document full of if/then conditionals that the model struggles to navigate coherently.

### 5.2 Information Overload in Tools

A bloated tool set, with functional overlaps and unclear decisional boundaries, is one of the most frequent causes of agent failure. If a human reading the tool descriptions cannot unambiguously determine which to use in a given situation, the agent certainly will not.

The problem also manifests in the results: tools that return enormous and unfiltered payloads force the agent to spend precious context budget parsing information of which only a fraction is relevant.

### 5.3 Stale Context

**Stale context** -- obsolete information remaining in the context window -- is a particularly insidious problem because it causes **confident errors**: the model produces wrong answers with high certainty because it relies on data that was correct in the past but no longer is.

Typical manifestations include:
- Project specifications superseded by more recent decisions.
- Tool results that reflect a previous state of the system.
- Instructions that have been implicitly modified by new user requests.

### 5.4 Context Poisoning

**Context poisoning** is the most dangerous phenomenon: when a hallucination or error enters the context and is repeatedly referenced, compounding the error over time. Since objectives and information in the context are consulted at every decision point, the false information self-reinforces. The agent can spend dozens of turns chasing impossible goals, unable to recover because the poisoned context continues to validate the error.

Mitigation requires:
- **Periodic validation** of information in context against external sources.
- **"Fresh start" mechanisms** that allow the context to be reconstructed from scratch when poisoning is suspected.
- **Separation of sources of truth** (immutable feature list) from derived information (notes, summaries).

### 5.5 Premature or Excessive Compaction

The opposite extreme of context stuffing is overly aggressive compaction, which eliminates subtle but critical details. A user preference incidentally mentioned, a technical constraint discovered during a failed exploration, a specific naming pattern -- all of these may seem irrelevant to the compaction algorithm but turn out to be essential steps later.

The solution is a **conservative and incremental** approach to compaction, with validation of summaries before they replace the original context.

---

## 6. Practical Implementation

### 6.1 Starting Philosophy

Anthropic's guide is explicit about the starting point: **"Do the simplest thing that works."** This principle breaks down into:

1. **Start with minimal prompts** using the best available models.
2. **Add instructions incrementally** based on the failure modes observed during testing.
3. **Avoid preemptive over-engineering** -- many anticipated problems never materialize.
4. **Iterate based on real data** rather than on theoretical hypotheses.

As model capabilities improve, agents operate with growing autonomy and require less prescriptive engineering. Investment must focus on infrastructure (memory management, tool orchestration) rather than on the micro-management of instructions.

### 6.2 Implementation Checklist

**For the System Prompt:**
- [ ] Organized in distinct sections with clear headers.
- [ ] Balances specificity and flexibility (the "right altitude").
- [ ] Includes diverse and canonical few-shot examples (not exhaustive edge cases).
- [ ] Tested against specific failure modes and iterated accordingly.

**For Tools:**
- [ ] Each tool has a clear and unambiguous purpose.
- [ ] No significant functional overlap between tools.
- [ ] Results are token-efficient and filtered.
- [ ] Input parameters are descriptive and self-explanatory.

**For the Context Window:**
- [ ] Compaction implemented with recall-then-precision strategy.
- [ ] Tool results with expiration mechanism.
- [ ] Budget dynamically allocated by task phase.
- [ ] Emergency reserve of 10-15% maintained.

**For Long-Running Agents:**
- [ ] Git-based checkpoints with descriptive commit messages.
- [ ] Progress files updated at the end of each session.
- [ ] Immutable feature list as ground truth.
- [ ] Initial validation protocol at every new session.
- [ ] Recovery strategy from corrupted states defined and tested.

### 6.3 Strategy Selection Matrix

Anthropic provides a decisional guide for choosing between the three main strategies:

| Strategy | Ideal Use Case | Strengths | Limitations |
|---|---|---|---|
| **Compaction** | Extended conversations with intense bidirectional flow | Preserves conversational flow; simple implementation | Risk of subtle context loss |
| **Note-Taking** | Iterative development with clear milestones | Excellent for progress tracking; low overhead | Requires discipline in note structuring |
| **Multi-Agent** | Complex research with parallel exploration | Context isolation; scalability | Coordination overhead; architectural complexity |

The choice is not exclusive: in practice, the most effective systems combine all three strategies in different measures depending on the task phase and the nature of the work.

### 6.4 Evaluation Metrics

To measure the effectiveness of context engineering, monitor:

- **Token efficiency**: ratio between useful tokens and total tokens in the context window.
- **Task completion rate**: percentage of tasks completed successfully without human intervention.
- **Recovery rate**: the agent's ability to recover from errors without manual resets.
- **Context staleness index**: frequency with which obsolete information causes errors.
- **Compaction fidelity**: percentage of critical information preserved after compaction.

---

## 7. Resources and References

### Primary Anthropic Sources

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) -- Anthropic's foundational guide on context engineering.
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) -- Complementary guide on patterns for long-running agents.
- [Memory Tool Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) -- Official documentation of Claude's Memory Tool.
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering) -- Hub for all of Anthropic's technical publications.

### Community Analyses and Insights

- [Context Engineering: AI Agent Optimization Guide](https://howaiworks.ai/blog/anthropic-context-engineering-for-agents) -- HowAIWorks.ai, guide to agent optimization.
- [Context Is the New Prompt](https://medium.com/data-science-collective/context-is-the-new-prompt-why-context-engineering-is-shaping-the-future-of-ai-46eb062ed270) -- Data Science Collective, analysis of the paradigm shift.
- [Context Engineering Explained](https://pub.towardsai.net/context-engineering-explained-the-anthropic-guide-thats-changing-how-developers-work-with-ai-40fae176a18d) -- Towards AI, in-depth explanation.
- [Why AI Teams Are Moving From Prompt Engineering to Context Engineering](https://neo4j.com/blog/agentic-ai/context-engineering-vs-prompt-engineering/) -- Neo4j, enterprise perspective.
- [The New Skill in AI is Not Prompting, It's Context Engineering](https://www.philschmid.de/context-engineering) -- Philipp Schmid, practical perspective.
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) -- Lessons from the Manus project.

### Academic Research

- [Context Engineering for AI Agents in Open-Source Software](https://arxiv.org/html/2510.21413v1) -- ArXiv, analysis in the open-source context.
- [Active Context Compression: Autonomous Memory Management in LLM Agents](https://arxiv.org/html/2601.07190) -- ArXiv, active context compression.
- [Codified Context: Infrastructure for AI Agents in a Complex Codebase](https://arxiv.org/html/2602.20478v1) -- ArXiv, infrastructure for codified context.

### Tools and Frameworks

- [Context Engineering Guide](https://www.promptingguide.ai/guides/context-engineering-guide) -- Prompting Guide, practical guide.
- [Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) -- GitHub, collection of skills for agents.
- [Context Engineering Repository](https://github.com/bonigarcia/context-engineering) -- GitHub, research project on context engineering.
- [LangChain Context Engineering](https://blog.langchain.com/context-engineering-for-agents/) -- LangChain, framework integration.

---

*This guide is based on Anthropic's original publication "Effective Context Engineering for AI Agents" (September 2025) and integrated with community analyses, academic research, and patterns that have emerged from practice. Context engineering is a rapidly evolving discipline: the fundamental principles will remain valid, but the specific techniques will evolve with the capabilities of the models.*
