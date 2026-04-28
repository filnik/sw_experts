# Agentic Engineering Patterns -- Simon Willison's Guide

> **Primary source**: [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) -- Simon Willison's Weblog
> **Newsletter**: [Agentic Engineering Patterns](https://simonw.substack.com/p/agentic-engineering-patterns) -- Simon Willison's Newsletter
> **First publication**: February 23, 2026
> **Format**: "evergreen" chapter-based guide, continuously updated
> **Last update consulted**: March 2026

---

## 1. Overview -- Context and Importance

Simon Willison, creator of Datasette and co-creator of Django, published in February 2026 a structured chapter-based guide titled **"Agentic Engineering Patterns"**. It is not an isolated post but an editorial project conceived as "evergreen" content: each chapter is designed as a blog post with less prominent dates, intended to be updated over time rather than crystallized at the moment of first publication. The format reflects the awareness that the field moves too quickly for a traditional book.

The guide is born at a specific moment in the history of software development. At the beginning of 2026, tools such as **Claude Code**, **OpenAI Codex**, and **Gemini CLI** have reached a maturity such as to redefine the daily work of software engineers. The fundamental characteristic of these tools is that they are not limited to generating code: they can **execute it**, verify its operation, and iterate autonomously without turn-by-turn guidance from the human operator.

Willison introduces a clear distinction between two practices that are often confused in public debate:

- **Vibe coding** (term coined by Andrej Karpathy): the casual use of LLMs to generate code without review, accepting the output as it comes, often by non-programmers.
- **Agentic engineering**: the disciplined and conscious use of coding agents by professional software engineers, who amplify their existing experience through these tools.

The guide explicitly positions itself on the second pole. As Willison himself states: _"I have a strong personal policy of not publishing AI-generated writing under my own name"_ -- a position that reflects the general philosophy of the project: the human remains at the center, the agent is a tool that amplifies skills already possessed.

The project has been received with significant interest in the technical community. On Hacker News it has generated in-depth discussions, with experienced developers highlighting both strengths (the emphasis on testing, the discipline of review) and concerns (the risk that rapid code generation creates bottlenecks in review, the questionable quality of LLM-generated tests). Ben Werdmuller has [taken up and commented on the guide](https://werd.io/agentic-engineering-patterns/), highlighting its value as a practical reference for the sector.

---

## 2. Identified Patterns -- Detailed Analysis

The guide is organized into thematic sections, each containing chapters dedicated to specific patterns. Below is the complete analysis of each documented pattern.

### 2.1 Fundamental Principles

#### "Writing Code is Cheap Now"

This is the conceptual pattern that lays the foundation for all the others. For decades, software engineering has been based on the assumption that _"producing a few hundred lines of clean and tested code requires most developers an entire day or more"_. This constraint has shaped decisions at every level: from project planning (which features justify the cost of development?) to micro choices (is it worth doing this refactoring?).

Coding agents have reduced the cost of "typing code into the computer" to almost nothing. This requires a recalibration of professional intuitions. Willison recommends reconsidering the instinct that leads to discarding features because they are "not worth the time": better to _"throw a prompt at it anyway, in an asynchronous session where the worst that can happen is that after ten minutes you discover it wasn't worth the tokens"_.

The crucial distinction: **new code** costs almost nothing, **good code** remains significantly more expensive. Good code requires functional correctness and verification, solving the right problem, elegant error handling, simplicity and minimal design, test coverage, up-to-date documentation, maintainability, and context-specific quality attributes (accessibility, security, reliability, scalability).

#### "Hoard Things You Know How to Do"

This pattern concerns the **cognitive capital** of the engineer. A significant part of software-building expertise consists of knowing what is possible and what is not, and having at least an approximate idea of how to make those things happen. Knowing whether OCR can run in JavaScript, what the constraints of Bluetooth pairing are, or how to process JSON in streaming -- this knowledge allows you to write better prompts and guide agents more effectively.

Willison recommends accumulating small proofs-of-concept, HTML/JS tools, and verified snippets as a **personal library of solutions**. AI can recombine these techniques in new ways, but only if the engineer knows that they exist and are feasible.

#### "AI Should Help Us Produce Better Code"

This guiding principle establishes the goal of the entire discipline. Effective agentic engineering should help engineers _"produce more code, of higher quality, that solves problems with greater impact"_. It is not just about speed: it is about raising the overall quality bar.

#### Anti-pattern: Unreviewed Code in Pull Requests

The main anti-pattern documented in the guide concerns submitting PRs with hundreds or thousands of lines of agent-generated code without having personally reviewed it. As Willison writes: _"If you open a PR with hundreds (or thousands) of lines of code that an agent produced for you, and you haven't done the work to ensure that code is functional yourself, you are delegating the actual work to other people."_

The characteristics of a responsible PR produced with agents are four:

1. **Functional confidence**: the code works and you are sure it works
2. **Manageable size**: small modifications, divided into separate commits
3. **Contextual explanation**: high-level objectives, links to issues, specifications
4. **Validated descriptions**: the PR descriptions must be read and verified personally before submission

### 2.2 Working with Coding Agents

#### How Coding Agents Work

Willison defines an agent as _"a piece of software that acts as a harness for an LLM, extending that LLM with additional capabilities that are powered by invisible prompts and implemented as callable tools"_. His preferred definition is even more synthetic: **"Agents run tools in a loop to achieve a goal"**.

The mechanism works as follows: the software calls an LLM with the user's prompt and provides tool definitions. The LLM requests specific tools, which are executed by the harness, with the results returned to the LLM. For coding agents, the critical tool is code execution (typically `Bash()` or `Python()`). The fundamental architecture is: **LLM + system prompt + tools in a loop**.

The chapter also explains the underlying mechanisms: token caching (providers offer reduced rates for common prefixes already processed), the statefulness problem (LLMs are stateless and must re-execute the entire conversation history at each turn), and reasoning (introduced in 2025, particularly useful for debugging).

#### Using Git with Coding Agents

Coding agents possess deep fluency in Git. This chapter, published as a first draft in March 2026, documents how to leverage this competence. Key patterns:

- **Basic operations**: `"Start a new Git repo here"`, `"Commit these changes"`, `"Review changes made today"`
- **Advanced integration**: `"Integrate latest changes from main"`, where agents explain the available merge strategies (merge, rebase, squash, fast-forward)
- **Problem-solving**: `"Sort out this git mess for me"` to navigate merge conflicts, `"Find and recover my code that does..."` to search the reflog
- **History rewriting**: agents excel at combining commits, extracting files from commits, and rewriting messages

A noteworthy observation: Willison notes that agents typically produce _"really good taste in commit messages"_, often surpassing the quality of manually written messages. The commit history is not fixed but is a _"deliberately authored story describing software progression"_.

#### Subagents

Subagents represent a fundamental pattern for context management. When an agent uses a subagent, it _"dispatches a fresh copy of itself to achieve a specified goal, with a new context window that starts with a fresh prompt"_. This avoids consuming the precious top-level context of the main agent.

Claude Code uses subagents extensively: when starting a new task on an existing repository, the main agent constructs a prompt and sends a subagent to explore the repository and return a description of what it finds.

Key advantages of subagents:

- **Parallelism**: the parent agent can run multiple subagents simultaneously, potentially using faster and cheaper models (such as Claude Haiku) to accelerate operations
- **Specialization**: some agents allow subagents with custom system prompts and tools, creating specialized roles (code reviewer, testing agent, documentation agent)
- **Context isolation**: each subagent operates with a clean context window, avoiding the performance degradation that occurs when the main context fills up

### 2.3 Testing and Quality Assurance

#### Red/Green TDD

The pattern _"Use red/green TDD"_ is described as _"a pleasingly succinct way to get better results out of a coding agent"_. The process is divided into two phases:

- **Red Phase**: write the automated tests before implementation, then confirm they fail
- **Green Phase**: develop the implementation until all tests pass

The critical discipline is confirming that the tests fail before writing the implementation. Skipping this step risks creating tests that already pass, defeating the purpose of validation. Every good model understands "red/green TDD" as shorthand for the entire methodological process.

Example prompt: `"Build a Python function to extract headers from a markdown string. Use red/green TDD."`

#### "First Run the Tests"

A four-word prompt that incorporates substantial engineering discipline. At the beginning of every new session with an existing project, you ask the agent: `"First run the tests"` or `"Run 'uv run pytest'"`.

The three fundamental purposes of this pattern:

1. **Discovery and awareness**: the prompt forces the agent to locate and execute the test suite, establishing testing habits for future work
2. **Project scope indicator**: the test count signals project complexity and encourages the agent to explore the tests for deeper understanding
3. **Mental framework**: running the tests first establishes a testing-oriented mindset, which naturally leads to expanding test coverage

Willison argues that the old excuses for not writing tests -- that they are time-consuming and expensive to rewrite -- no longer hold _"when an agent can knock them into shape in just a few minutes"_.

#### Agentic Manual Testing

Manual testing by agents is a fundamental complement to automated tests. The principle: _"Just because code passes tests doesn't mean it works as intended."_ Automated tests can pass while the code fails in realistic scenarios.

Language-specific approaches:

- **Python libraries**: `python -c "... code ..."` to pass code directly to the interpreter
- **Other languages**: write demo files in `/tmp` for testing and compilation without risking commits to the repository
- **Web APIs**: run development servers and explore JSON endpoints with `curl`

For web interfaces, Willison recommends **Playwright** as the most powerful tool, with specialized CLI wrappers: **agent-browser** (Vercel), **Rodney** (Willison, uses Chrome DevTools Protocol with screenshot analysis via agent vision), and **Showboat** (Willison, documents testing workflows with three commands: `note`, `exec`, `image`).

Problems discovered through manual testing should feed permanent automated test suites, closing the loop with red/green TDD.

### 2.4 Understanding Code

#### Linear Walkthroughs

This pattern addresses the problem of code we don't understand -- whether it's legacy code, code written quickly with vibe coding, or simply forgotten code. You ask the agent to produce a structured and linear walkthrough of the codebase.

Willison demonstrated this pattern with a SwiftUI presentation app that he had developed with vibe coding without understanding its implementation. The prompt instructed Claude Code to read the source code, plan a detailed walkthrough, and use Showboat to create a `walkthrough.md` file.

Critical implementation detail: the prompt specified to use `sed`, `grep`, or `cat` to include code snippets rather than copying them manually. This prevents hallucinations and errors in code reproduction. The result covered all six Swift files with detailed explanations of the SwiftUI structure.

#### Interactive Explanations

This pattern addresses the concept of **cognitive debt**: _"When we lose track of how code written by our agents works we take on cognitive debt."_

The solution: build **animated and interactive visualizations** that demonstrate algorithm behavior dynamically. Willison used it to understand Rust code that generated word cloud visualizations. Claude's technical description (_"Archimedean spiral placement with per-word random angular offset"_) did not convey intuitive understanding, but an interactive HTML animation with frame-by-frame controls made the algorithm immediately understandable.

---

## 3. Philosophy -- Willison's Pragmatic Approach

Willison's philosophy stands out in the AI-assisted development landscape for its radical pragmatism tempered by a deep awareness of risks. Some pillars:

**The engineer remains at the center.** Writing code has never been the only activity of a software engineer. Engineers determine *which* code to write, navigate trade-offs between solutions, and find approaches suited to specific circumstances. The agent is a tool, not a substitute.

**Non-delegable responsibility.** The engineer is responsible for: providing agents with the appropriate tools, specifying problems with the right level of detail, and verifying and iterating on results until having confidence in the robustness and credibility of the code. Responsibility for the code produced always remains human.

**Transparency in AI use.** Willison maintains an explicit policy of not publishing AI-generated text under his own name. He uses LLMs for proofreading, code examples, and supporting tasks -- never for primary content production. It is a principle that reflects the conviction that authenticity and deep understanding of one's output remain fundamental values.

**Incremental iteration.** The guide itself is built as an incremental project. It is not a closed book but a living organism that grows chapter by chapter. The technical implementation of the guide (the Django models and views) was largely developed by Claude Opus 4.6 in Claude Code -- a practical demonstration of the principles described.

**Code is thrown away and rewritten.** With the production cost of code close to zero, attachment to existing code becomes an anti-pattern. If a refactoring is useful, you launch an agent. If the agent fails, the cost is negligible.

---

## 4. Trust Boundaries -- Trust Boundaries for Agents

The concept of trust boundaries is central in Willison's thinking on agent security. Willison has developed a conceptual framework that identifies the **"Lethal Trifecta"** -- three capabilities that, when they coexist in an AI system, create a fundamental security risk:

1. **Access to private data** -- essential for many agent use cases
2. **Exposure to untrusted content** -- emails, web pages, documents from potentially malicious sources
3. **External communication capability** -- HTTP requests, links, sending emails that could exfiltrate data

When all three elements coexist, a single piece of poisoned content can lead an agent to exfiltrate sensitive data, send unauthorized messages, or trigger cascading operations -- all without a vulnerability in traditional code.

The fundamental problem: _"LLMs follow instructions in content. This is what makes them so useful"_ -- but they cannot reliably distinguish between trusted system prompts and malicious instructions embedded in processed data. Once the tokens are concatenated for processing, their origin becomes invisible.

Willison proposes an approach based on **taint tracking**: treating "exposure to untrusted content" as a contamination event. If the current state is contaminated, block (or require explicit human approval for) any action with exfiltration potential, including outgoing HTTP, email/chat sending, PR creation, and even displaying clickable links.

The practical solution requires accepting that high-autonomy agents that combine all three elements of the trifecta **cannot be reliably made secure today**. Builders must eliminate one of the components or impose architectural constraints that significantly reduce utility.

---

## 5. Human-in-the-Loop -- When and How to Insert the Human

Willison articulates several modalities of human involvement in the agent's loop:

**Approval per action.** Claude Code requires by default approval for every action (no batch execution). This is the highest level of security but also the most laborious. The trade-off is explicit: maximum security against reduced productivity.

**Confirm before acting.** Personal agents like those on local devices can operate with a "confirm before acting" model. Willison reports, however, the case documented by Summer Yue of an agent that deleted emails from the inbox when context window compaction lost the original instruction constraints -- an example of how human boundaries can degrade during prolonged sessions.

**Code review as a quality gate.** The anti-pattern of unreviewed PR submission highlights that human code review is a non-negotiable gate. The engineer must demonstrate having done their review work before asking others to invest time.

**Strategic direction with delegated execution.** The case of Andreas Kling with the Rust port of LibJS illustrates this pattern: human direction, extensive test262 conformance suite, and verification of output against the existing implementation. The result: 25,000 lines transposed byte-for-byte in two weeks with Claude Code and Codex.

**The principle of "well-defined human controllers"** (from Google's security framework, taken up by Willison): systems must distinguish between authorized user instructions and other inputs. Critical actions require explicit human confirmation. The goal is to avoid "prompt fatigue" from excess confirmations while protecting critical operations.

---

## 6. Prompt Injection Mitigation -- Strategies for Real-World Agents

Prompt injection remains the Achilles' heel of AI agents. As Willison notes: _"The curse of prompt injection continues to be that we've known about the issue for more than two and a half years and we still don't have convincing mitigations."_

Willison documents and analyzes six mitigation patterns derived from academic research (particularly the paper "Agents Rule of Two"):

### 6.1 Action-Selector Pattern
Agents trigger tools but do not receive feedback from responses. Outputs go directly to users, preventing malicious instructions in tool responses from influencing subsequent actions.

### 6.2 Plan-Then-Execute Pattern
Tool calls are determined **before** exposure to untrusted content. Subsequent outputs cannot change which actions are executed, only their content parameters.

### 6.3 LLM Map-Reduce Pattern
Parallel sub-agents process untrusted content independently, returning only symbolic variables (`$VAR1`, etc.) to the coordinator. This prevents the passage of information between processing phases.

### 6.4 Dual LLM Pattern
A privileged LLM never touches untrusted content; a quarantined LLM processes it and returns abstract results. The privileged agent remains unexposed.

### 6.5 Code-Then-Execute Pattern
The privileged LLM generates code in a restricted DSL that specifies tool interactions, enabling data flow analysis and taint tracking during execution.

### 6.6 Context-Minimization Pattern
Removal of user prompts after their conversion into structured queries (such as SQL), preventing injection through the remaining context.

### Defense in Depth

Google proposes a defense-in-depth approach with **deterministic policy engines** that intercept tool calls and evaluate them against predefined rules, considering: action risk level (irreversibility, financial impact), current context, chains of previous actions, and whether recent processing has involved suspect sources. The system can allow, block, or require user confirmation based on risk evaluation.

The fundamental limitation: neither design patterns nor guardrails offer guaranteed protection. _"Once an LLM agent has ingested untrusted input, it must be constrained so that it is impossible for that input to trigger any consequential actions."_ Current architectures do not provide reliable means to achieve this goal universally.

---

## 7. Comparison with Anthropic Patterns

Anthropic has published its own guide ["Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) and subsequently ["Effective Context Engineering for AI Agents"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents). The comparison with Willison's approach reveals both significant convergences and divergences.

### Convergences

| Theme | Anthropic | Willison |
|------|-----------|----------|
| **Architectural simplicity** | Simple composable patterns > complex frameworks | "LLM + system prompt + tools in a loop" as fundamental architecture |
| **Tools in the loop** | Tool use as central mechanism | "Agents run tools in a loop to achieve a goal" |
| **Test and verification** | Emphasis on output validation | Red/green TDD, first run the tests, manual testing |
| **Human role** | Human oversight as necessity | Anti-pattern of unreviewed PRs, explicit human-in-the-loop |

### Divergences

| Theme | Anthropic | Willison |
|------|-----------|----------|
| **Perspective** | Model provider -- optimization of the use of one's product | Independent practitioner -- optimization of the daily workflow |
| **Taxonomy** | Five workflow patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) + autonomous agents | Patterns oriented to daily practice (TDD, git, walkthrough, subagents) |
| **Level of abstraction** | Architectural -- how to compose multi-LLM systems | Operational -- how a single engineer works with an agent |
| **Security** | Focus on context engineering and model guardrails | Lethal Trifecta, taint tracking, acceptance of fundamental limits |
| **Format** | Static papers/guides | Continuously updated evergreen guide |

### Complementarity

The two approaches are complementary rather than competing. Anthropic provides the architectural framework for those building agent infrastructure (how to structure multi-LLM systems, when to use workflows vs. autonomous agents). Willison provides operational patterns for those who **use** these systems daily (how to write effective prompts, how to maintain code quality, how to manage cognitive debt).

Willison himself has commented favorably on Anthropic's work, and his preferred definition of agent (_"runs tools in a loop to achieve a goal"_) is consistent with the Anthropic distinction between workflow (predefined orchestration) and agents (dynamic direction of their own processes and tool use).

The most significant difference concerns security. Willison takes a more explicitly pessimistic position: he accepts that certain problems (prompt injection on untrusted content) **do not have reliable solutions today** and proposes designing around this limitation rather than trying to overcome it. Anthropic, as a provider, tends to emphasize available mitigations and improvements in models.

---

## 8. Practical Lessons -- What to Take Home

For those building agents or working daily with them, Willison's guide offers immediately applicable lessons:

### 8.1 Embrace the Zero Cost of Code, Keep the Cost of Quality High

Code is cheap, quality is not. This means launching prompts to explore solutions without hesitation, but investing serious time in review, testing, and understanding what is produced. As one developer commented on Hacker News: _"Test harness is everything, if you don't have a way of validating the work, the loop will go stray."_

### 8.2 Four Words That Change Everything

`"Use red/green TDD"` and `"First run the tests"` are four-word prompts that incorporate deep engineering discipline. Models understand them as shorthand for complete methodologies. Using them systematically improves output quality disproportionately compared to the effort.

### 8.3 Cognitive Debt is the New Enemy

Not understanding the code produced by agents is an insidious form of technical debt. Linear walkthroughs and interactive explanations are not luxuries but necessary tools to maintain the ability to reason about one's systems. The cost of producing these explanations is now negligible -- not doing it is an anti-pattern.

### 8.4 Git as Safety Net

Using Git in a disciplined way with agents -- frequent commits, branches for experiments, readable history -- transforms version control into the main safety net. Agents are excellent at using Git; leveraging this competence is free and dramatically reduces risk.

### 8.5 Subagents for Context Scalability

Delegating exploratory and specialized tasks to subagents with clean contexts is the most effective pattern for managing complexity. Parallelization of subagents with cheaper models (e.g., Haiku for exploration, Opus for architectural decisions) offers a significant productivity multiplier.

### 8.6 Never Submit Unreviewed Code

If you open a PR, the code must work and you must be sure it works. Initial review is your responsibility, not the team's. This applies all the more with code generated by agents, where the temptation to delegate review is stronger.

### 8.7 Security Has Structural Limits

The Lethal Trifecta has no complete solutions today. If your agent combines access to private data, exposure to untrusted content, and external communication capability, you have an architectural problem that no guardrail can completely solve. Design around this limitation: eliminate at least one of the three elements or impose strict constraints.

### 8.8 Accumulating Competence Remains Fundamental

AI amplifies existing skills, it does not create them from nothing. Building and maintaining a personal library of solutions, proofs-of-concept, and knowledge of what is technically possible remains the most profitable investment for an engineer working with agents.

---

## 9. Resources and References

### Primary Sources

- **Main guide**: [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) -- Simon Willison's Weblog
- **Launch newsletter**: [Agentic Engineering Patterns](https://simonw.substack.com/p/agentic-engineering-patterns) -- Simon Willison's Newsletter
- **Launch post**: [Writing about Agentic Engineering Patterns](https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/) -- February 2026

### Individual Chapters of the Guide

- [What is Agentic Engineering?](https://simonwillison.net/guides/agentic-engineering-patterns/what-is-agentic-engineering/)
- [Writing Code is Cheap Now](https://simonwillison.net/guides/agentic-engineering-patterns/code-is-cheap/)
- [How Coding Agents Work](https://simonwillison.net/guides/agentic-engineering-patterns/how-coding-agents-work/)
- [Red/Green TDD](https://simonwillison.net/guides/agentic-engineering-patterns/red-green-tdd/)
- [First Run the Tests](https://simonwillison.net/guides/agentic-engineering-patterns/first-run-the-tests/)
- [Agentic Manual Testing](https://simonwillison.net/guides/agentic-engineering-patterns/agentic-manual-testing/)
- [Linear Walkthroughs](https://simonwillison.net/guides/agentic-engineering-patterns/linear-walkthroughs/)
- [Interactive Explanations](https://simonwillison.net/guides/agentic-engineering-patterns/interactive-explanations/)
- [Using Git with Coding Agents](https://simonwillison.net/guides/agentic-engineering-patterns/using-git-with-coding-agents/)
- [Subagents](https://simonwillison.net/guides/agentic-engineering-patterns/subagents/)
- [Anti-patterns: Things to Avoid](https://simonwillison.net/guides/agentic-engineering-patterns/anti-patterns/)
- [Prompts I Use](https://simonwillison.net/guides/agentic-engineering-patterns/prompts/)

### Security and Trust Boundaries

- [The Lethal Trifecta for AI Agents](https://simonw.substack.com/p/the-lethal-trifecta-for-ai-agents) -- Simon Willison's Newsletter
- [New Prompt Injection Papers: Agents Rule of Two](https://simonw.substack.com/p/new-prompt-injection-papers-agents) -- Simon Willison's Newsletter
- [Agentic AI and Security](https://martinfowler.com/articles/agentic-ai-security.html) -- Martin Fowler

### Anthropic Patterns (Comparison)

- [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) -- Anthropic Research
- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) -- Anthropic Engineering
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) -- Anthropic Engineering

### Community Comments and Analyses

- [Discussion on Hacker News](https://news.ycombinator.com/item?id=47243272)
- [Analysis by Ben Werdmuller](https://werd.io/agentic-engineering-patterns/)
- [Fireside Chat at the Pragmatic Summit](https://simonwillison.net/2026/Mar/14/pragmatic-summit/) -- March 2026

### Tools Mentioned

- **Claude Code** -- Anthropic (main coding agent discussed in the guide)
- **OpenAI Codex** -- OpenAI (alternative coding agent)
- **Gemini CLI** -- Google (alternative coding agent)
- **Showboat** -- Simon Willison (tool to document testing workflows)
- **Rodney** -- Simon Willison (Chrome DevTools Protocol wrapper for agents)
- **Playwright** -- Microsoft (browser automation for testing)
- **Datasette** -- Simon Willison (tool for exploring and publishing data)
