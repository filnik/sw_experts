# Andrew Ng and the 4 Agentic Patterns

## Index

1. [Profile and Background](#1-profile-and-background)
2. [The 4 Agentic Patterns](#2-the-4-agentic-patterns)
3. [Agentic Workflows vs Non-Agentic](#3-agentic-workflows-vs-non-agentic)
4. [Practical Examples](#4-practical-examples)
5. [Ng's Vision of Agentic AI](#5-ngs-vision-of-agentic-ai)
6. [Resources and References](#6-resources-and-references)

---

## 1. Profile and Background

### Who is Andrew Ng

Andrew Ng is one of the most influential figures in the history of modern artificial intelligence. Born in London in 1976 and raised between Hong Kong and Singapore, he has built a career spanning academia, industry, and entrepreneurship, contributing decisively to the democratization of AI and its diffusion on a global scale. In 2023, he was included in the Time100 AI list of the most influential people in the field of artificial intelligence.

### Stanford University

Ng is Adjunct Professor at the Department of Computer Science at Stanford University, where for years he was director of the Stanford AI Lab (SAIL), one of the most prestigious AI research labs in the world, hosting over 20 active research groups. His CS229 - Machine Learning course has become legendary: it is the most popular course ever offered at Stanford, with over 1,000 students enrolled in some academic years. This course has trained an entire generation of AI professionals, many of whom today hold key roles in technology companies and research labs.

### Google Brain

In 2011, Ng founded and led the Google Brain Deep Learning Project together with Jeff Dean, Greg Corrado, and Rajat Monga. This project represented a fundamental turning point for Google, transforming the company into a modern AI powerhouse. The Google Brain team demonstrated that deep neural networks could be trained at massive scale using Google's distributed computing infrastructure, producing revolutionary results in image recognition and natural language processing.

### Baidu

After Google, Ng held the role of VP & Chief Scientist at Baidu, the Chinese tech giant, where he led a team of 1,300 people responsible for the entire AI strategy of the company. During this period, he supervised the development of cutting-edge technologies in deep learning, speech recognition, and autonomous driving.

### Coursera

In 2012, together with Stanford colleague Daphne Koller, Ng co-founded Coursera, the online learning platform that today serves over 150 million registered students worldwide. Coursera was born directly from the experience of the CS229 course: Ng intuited that quality education could and should be accessible to everyone, not just to students of elite universities. This vision radically changed the landscape of global education.

### DeepLearning.AI and AI Fund

In 2017 Ng founded DeepLearning.AI, an educational platform focused exclusively on artificial intelligence. Through courses on Coursera and on its own platform, over 8 million people worldwide have followed his training programs. DeepLearning.AI also publishes "The Batch", a weekly newsletter that has become a reference point for the global AI community.

In parallel, Ng founded AI Fund, a venture studio that finances and builds startups in the AI field, and Landing AI (today LandingAI), a company focused on computer vision for industrial applications. This ecosystem of organizations reflects his philosophy: it is not enough to do research, you also need to educate, build, and invest to bring AI from the laboratory to the real world.

---

## 2. The 4 Agentic Patterns

### Context and Origin

In March 2024, during the Sequoia Capital AI Ascent conference, Andrew Ng gave a presentation that redefined the way the AI community thinks about agentic systems. Instead of speaking of "agents" in vague and speculative terms, Ng identified four concrete and implementable patterns that define the agentic behaviors of Large Language Models. These patterns were subsequently elaborated in his newsletter The Batch and in the "Agentic AI" course on DeepLearning.AI.

Ng's central thesis is provocative but supported by data: **agentic workflows can generate more progress in AI than the next generation of foundational models**. In other words, the way we use models matters as much (or more) as the size and power of the models themselves.

---

### 2.1 Reflection

#### Definition

Reflection is the pattern through which an AI system critically evaluates its own output and refines it iteratively. It is the computational equivalent of the revision process that a human performs when re-reading and correcting their own work. Ng describes it as a mechanism in which the model "automatically critiques its own output and improves its response".

#### How It Works

The Reflection pattern is articulated in three fundamental phases:

1. **Initial generation**: the model produces an output in response to a prompt (for example, generates code, writes a text, or answers a question).
2. **Self-criticism**: the same model (or a second dedicated agent) analyzes the output produced, looking for errors, inefficiencies, style problems, or logical gaps. The typical prompt could be: *"Here is some code written for task X: [generated code]. Carefully check the code for correctness, style, and efficiency, and provide constructive criticism on how to improve it."*
3. **Iterative rewriting**: the model receives both the original output and the critical feedback, and produces an improved version. This cycle can be repeated multiple times, with each iteration progressively refining the quality.

#### Multi-Agent Approach to Reflection

A particularly effective implementation involves the creation of two distinct agents: one specialized in generating high-quality output and another specialized in providing constructive criticism. The resulting "discussion" between these two agents produces results superior to those obtainable with a single model. This approach is conceptually similar to the peer review process in academia or to code review in software development.

#### Reflection Enhanced with Tools

Reflection can be further strengthened by providing the model with tools to validate its own output. For example, in the case of code generation, the model can run unit tests to verify the correctness of the produced code, analyze the errors found, and use this information to guide the improvement process. This approach effectively combines the Reflection and Tool Use patterns.

#### Key Characteristics

Ng classifies Reflection as **"robust and consistently effective"**. Among the four patterns, it is the one with the most favorable cost-benefit ratio: it is relatively simple to implement but produces surprising improvements in performance. The reference scientific literature includes Self-Refine (Madaan et al., 2023), Reflexion (Shinn et al., 2023), and CRITIC (Gou et al., 2024).

---

### 2.2 Tool Use

#### Definition

The Tool Use pattern allows an LLM to interact with external tools, radically expanding its capabilities beyond pure text generation. Instead of being limited to producing responses based exclusively on its internal knowledge (which is static and limited to the training date), the agent can access real-time information, perform calculations, manipulate data, and take actions in the real world.

#### How It Works

The Tool Use mechanism is based on three components:

1. **Definition of available tools**: the agent receives a structured description (typically in JSON Schema format) of the tools at its disposal, including names, parameter descriptions, and expected output formats.
2. **Autonomous decision**: given a task, the agent decides which tool to use, with which parameters, and at which moment in the workflow. This decision is guided by the model's reasoning, not by hardcoded rules.
3. **Execution and integration**: the external system executes the tool call, returns the result to the agent, which integrates it into its context to continue reasoning or generate the final output.

#### Types of Tools

The tools that an agent can use cover a vast spectrum:

- **Web search**: access to up-to-date information via search engines
- **Code execution**: ability to write and execute code in secure sandboxes (Python, JavaScript, SQL, etc.)
- **External APIs**: interaction with third-party services (calendar, email, CRM, database, cloud services)
- **File system access**: reading and writing files, manipulating documents
- **Calculation tools**: calculators, symbolic algebra systems, simulation engines
- **Database querying**: SQL or NoSQL queries to retrieve structured data

#### Technical Mechanism: Function Calling

The most common technical implementation of Tool Use is "function calling", natively supported by models such as GPT-4, Claude, Gemini, and others. The model generates a structured function call (function name and parameters in JSON format), the runtime system executes the function and returns the result, and the model continues reasoning with this new information in the context.

#### Key Characteristics

Together with Reflection, Ng classifies Tool Use as **"consistently effective and fairly well defined"**. It is a mature pattern, well supported by existing frameworks, and represents the most immediate way to overcome the intrinsic limits of LLMs (static knowledge, inability to perform precise calculations, lack of access to the external world).

---

### 2.3 Planning

#### Definition

Planning is the pattern in which an LLM is used to autonomously decide which sequence of steps to execute to accomplish a complex task. Instead of receiving hardcoded step-by-step instructions, the agent analyzes the final goal, decomposes it into manageable sub-tasks, and orchestrates the execution of each step adapting dynamically to intermediate results.

#### How It Works

The Planning workflow is typically articulated as follows:

1. **Goal analysis**: the agent receives a high-level task (e.g., "Do in-depth research on the European electric vehicle market and produce a report").
2. **Decomposition**: the LLM generates a multi-step plan, identifying the necessary sub-tasks, their dependencies, and the optimal execution sequence.
3. **Iterative execution**: the agent executes each step of the plan, using Tool Use and Reflection where necessary. After each step, it evaluates the result and can adapt the remaining plan.
4. **Error recovery**: if a step fails or produces unsatisfactory results, the agent can reformulate the plan, attempt alternative approaches, or request additional information.

#### Technical Approaches to Planning

There are several formal approaches to Planning:

- **ReAct (Reason and Act)**: the model explicitly alternates phases of reasoning ("Thought") and actions ("Action"), keeping track of the decision process. This approach offers transparency and allows diagnosing where reasoning fails.
- **Chain-of-Thought Planning**: the model generates a structured reasoning chain before acting, exploring the implications of each step.
- **Tree-of-Thought**: variant that simultaneously explores multiple planning paths, evaluating them and selecting the most promising.
- **Plan-and-Execute**: architecture in which a "planner" agent generates the complete plan and an "executor" agent implements it, with feedback loop between the two.

#### The Planning Challenge

Ng is very clear in distinguishing Planning from the other patterns: he describes it as **"incredibly powerful, but less consistent"**. Planning works splendidly when it works, but it is the pattern most subject to failures. Typical problems include:

- **Wrong decomposition**: the model can break down the task into inadequate or incomplete sub-tasks
- **Decision deadlock**: the agent can get stuck in planning loops without ever executing
- **Error propagation**: an error in an initial step can invalidate the entire subsequent plan
- **Overplanning**: generating overly detailed plans that become rigid and fragile

The key, according to Ng, is that "the devil is in the details": Planning requires careful prompt engineering, good context management, and robust fallback mechanisms.

---

### 2.4 Multi-Agent Collaboration

#### Definition

Multi-Agent Collaboration is the pattern in which multiple specialized agents work together to solve complex problems. Each agent has a specific role, its own set of prompts, its own tools, and its own "memory", and the agents communicate with each other to coordinate work and converge toward a solution.

#### How It Works

The multi-agent paradigm is based on several principles:

1. **Role specialization**: each agent is configured with a system prompt that defines its role, skills, and expected behavior. For example, in a virtual software development team, one can have agents with the roles of Product Manager, Software Engineer, Designer, QA Tester.
2. **Communication protocols**: agents communicate through structured messages, following protocols that define who talks to whom, in what order, and how decisions are made.
3. **Pattern composition**: Multi-Agent Collaboration naturally integrates all the other patterns. Each individual agent can use Reflection to improve its output, Tool Use to access tools, and Planning to decompose its sub-task.
4. **Behavioral emergence**: collaboration between agents can produce emergent behaviors -- solutions and approaches that no single agent would have generated alone.

#### Multi-Agent Architectures

There are several architectures for organizing collaboration:

- **Sequential conversation**: agents pass work in sequence, like an assembly line (Agent A produces output -> Agent B refines it -> Agent C validates it)
- **Group conversation**: multiple agents participate in a shared discussion, each contributing from their own perspective
- **Hierarchy**: an "orchestrator" agent coordinates specialized agents, delegating sub-tasks and integrating results
- **Debate**: two or more agents discuss and debate an issue, converging toward a solution through dialectical confrontation

#### Examples of Frameworks and Projects

Ng cites several projects that implement the multi-agent pattern:

- **ChatDev**: an open source implementation that simulates an entire software company, with agents covering the roles of CEO, CTO, programmer, designer, and tester. The agents collaborate to develop software starting from a natural language description.
- **AutoGen** (Microsoft Research): pioneering framework for structured multi-agent conversations, with support for two-agent chat, group chat, sequential conversations, and nested chat patterns. In 2025, Microsoft integrated AutoGen with Semantic Kernel into the Microsoft Agent Framework.
- **CrewAI**: framework for orchestrating autonomous AI agents with defined roles. Incubated by AI Fund (the venture studio founded by Ng himself), CrewAI reached 3.2 million dollars in revenue and over 100,000 agent executions per day by 2025.

#### Key Characteristics

Ng describes Multi-Agent Collaboration as **"very powerful, but more emergent"** than the other patterns. The quality of the output is more difficult to predict, and the debugging of multi-agent systems is intrinsically complex. However, for high-complexity tasks (software development, scientific research, strategic analysis), this pattern offers capabilities that no single agent could match.

---

## 3. Agentic Workflows vs Non-Agentic

### The Writing Analogy

Ng uses a powerful analogy to illustrate the fundamental difference between agentic and non-agentic approaches. A non-agentic workflow is like asking a person to write an essay from start to finish **without ever using the backspace key**. They must produce the finished text on the first try, with no possibility of revision. This is exactly what we do when we use an LLM in "zero-shot" mode: a single prompt, a single response, no iteration.

An agentic workflow, on the contrary, mirrors how a human really works: produces a draft, re-reads it, identifies problems, conducts supplementary research, rewrites sections, asks for feedback, and iterates until reaching a satisfactory result. The agent has the possibility to "contemplate its own results and provide updates".

### The Difference in Numbers

The results of the HumanEval benchmark (a standardized test for code generation developed by OpenAI) concretely demonstrate this difference:

| Approach | Accuracy |
|----------|----------|
| GPT-3.5 (zero-shot, non-agentic) | 48.1% |
| GPT-4 (zero-shot, non-agentic) | 67.0% |
| GPT-3.5 with agentic workflow | **95.1%** |

This data is extraordinary: **a smaller and less powerful model (GPT-3.5), when placed in an agentic workflow, vastly outperforms the performance of a much more advanced model (GPT-4) used in direct mode**. The improvement obtained through the agentic workflow "outclasses" the improvement obtained by switching to a larger model.

### Architectural Implications

This discovery has profound implications for the design of AI systems:

- **The model is not everything**: investing exclusively in larger and more expensive models is not necessarily the optimal strategy. A well-designed agentic architecture can extract superior performance even from more modest models.
- **Latency vs quality**: agentic workflows require more time (minutes or hours vs seconds), but the quality of the result is drastically superior. Ng suggests recalibrating expectations: just as we would delegate a complex task to a collaborator giving them days to complete it, we should accept that an AI agent can take time to produce high-quality results.
- **Token generation speed**: token generation speed becomes critical in agentic workflows, since the model is invoked many times. A model that generates tokens faster enables faster iteration cycles.
- **Traceability**: agentic workflows produce valuable intermediate records (plans, reflections, tool results) that offer transparency and auditability. We can examine the agent's "reasoning", not just the final result.

### Structural Comparison

| Dimension | Non-Agentic | Agentic |
|-----------|-------------|---------|
| Prompt | Single | Multiple, iterative |
| Output | Direct, one-shot | Built incrementally |
| Revision | None | Self-criticism cycles |
| Tools | Not used | Integrated in the workflow |
| Planning | Absent | Dynamic decomposition |
| Adaptation | Rigid | Flexible, reactive |
| Typical quality | Good | Excellent |
| Time | Seconds | Minutes/Hours |
| Computational cost | Low | High |

---

## 4. Practical Examples

### 4.1 Reflection: Automated Code Review

**Scenario**: a development team wants to automate code review.

**Implementation**:
1. The developer submits the code to a "generator" agent that produces a first optimized version.
2. A "reviewer" agent analyzes the code looking for bugs, security vulnerabilities, style violations, and optimization opportunities.
3. The generator agent receives the feedback and rewrites the code, implementing the suggestions.
4. The cycle repeats 2-3 times, or until the reviewer no longer identifies significant problems.
5. Optionally, the reviewer agent can run unit tests and static analysis as additional tools to validate the code.

**Expected results**: Ng's team reported an accuracy improvement in database queries from 87% to 95% using this approach.

### 4.2 Tool Use: Financial Research Assistant

**Scenario**: a financial analyst needs an updated report on a company.

**Implementation**:
1. The agent receives the request: "Analyze Tesla's financial performance in the last quarter."
2. It uses a web search tool to find the most recent financial results.
3. Calls a financial data API to get precise numbers (revenue, margins, EPS).
4. Executes Python code to calculate derived metrics and generate charts.
5. Queries an internal database to compare with industry benchmarks.
6. Integrates all the information into a structured and coherent report.

**Key advantage**: the agent overcomes the limits of the model's static knowledge, accessing real-time data and producing precise and up-to-date analyses.

### 4.3 Planning: Autonomous Research Agent

**Scenario**: a researcher asks the agent to conduct a literature review on a specific topic.

**Implementation**:
1. The agent analyzes the request and generates a structured research plan:
   - Step 1: Define the key terms and search queries
   - Step 2: Search for articles on academic archives (arXiv, Google Scholar)
   - Step 3: Read and summarize the most relevant papers
   - Step 4: Identify common themes and divergences
   - Step 5: Draft a structured synthesis
   - Step 6: Review and refine the final report
2. At each step, the agent evaluates the results and can adapt the plan (e.g., adding search queries if initial results are insufficient).
3. If a step fails (e.g., a paper is not accessible), the agent searches for alternative sources.

**Note**: this is the capstone project of Ng's Agentic AI course on DeepLearning.AI, where students build an autonomous research agent.

### 4.4 Multi-Agent Collaboration: Virtual Software Development

**Scenario**: generate a complete web application starting from a natural language description.

**Implementation** (inspired by ChatDev):
1. **CEO Agent**: receives the customer request and defines high-level requirements.
2. **Product Manager Agent**: translates requirements into detailed functional specifications, defining user stories and priorities.
3. **Architect Agent**: designs the system architecture, chooses technologies, and defines the code structure.
4. **Developer Agent**: implements the code following the defined specifications and architecture. Uses Tool Use to execute the code and verify it compiles correctly.
5. **QA Tester Agent**: writes and runs tests, identifies bugs, and reports them to the Developer. Uses Reflection to evaluate test coverage.
6. **Designer Agent**: takes care of the user interface, proposing layouts and styles.

Each agent communicates with the others through structured messages, and the entire system produces functioning software through iterative cycles of development and review.

### 4.5 Combined Patterns: Intelligent Customer Support

**Scenario**: a next-generation customer support system.

**Implementation**:
1. **Planning**: the agent analyzes the customer's request and plans the resolution strategy.
2. **Tool Use**: queries the customer database, the ticketing system, the knowledge base, and the order history.
3. **Reflection**: after formulating a response, critically evaluates it for accuracy, completeness, and tone.
4. **Multi-Agent**: for complex cases, a first-level agent can escalate to a specialized agent (technical, billing, logistics), coordinating the resolution.

---

## 5. Ng's Vision of Agentic AI

### "The Future Is Agentic"

Andrew Ng's central thesis, articulated in the presentation at Sequoia Capital AI Ascent 2024 and reiterated in numerous subsequent communications, is that **agentic workflows represent the next big leap in AI**, potentially more significant than improvements in the foundational models themselves.

Ng argues that the AI community has invested enormous resources in making models bigger and more powerful, but has relatively neglected the way these models are used. The four agentic patterns demonstrate that intelligent architectures can extract radically superior performance from the same models, and that smaller and more efficient models can outperform much larger models when placed in well-designed agentic workflows.

### From Copilot to Autonomous Agent

Ng traces a clear evolutionary trajectory for AI systems:

1. **Current phase - Copilot**: AI models assist humans, generating suggestions that the user evaluates and adopts selectively. Control remains firmly in the user's hands.
2. **Emerging phase - Supervised agent**: AI systems perform complex tasks with periodic human supervision. The human defines goals and intervenes when necessary.
3. **Future phase - Autonomous agent**: AI systems complete tasks end-to-end independently, requiring human intervention only in exceptional cases.

Ng argues that as AI technology advances, agents will inevitably evolve from support tools to autonomous entities capable of completing tasks independently. This transition requires not only more capable models, but above all robust agentic architectures.

### Pattern Maturity

Ng proposes a maturity taxonomy for the four patterns:

- **Reflection and Tool Use**: mature patterns, consistently effective, well-defined and ready for production. Any team should start here.
- **Planning**: incredibly powerful but less reliable. It requires careful engineering and monitoring. It is the pattern where the fastest progress is being made.
- **Multi-Agent Collaboration**: the most emergent and least predictable pattern, but also the one with the greatest potential for high-complexity tasks. It is the area where research is most active.

### The Role of Speed

A technical aspect often underestimated and highlighted by Ng is the importance of token generation speed. In agentic workflows, the model is invoked tens or hundreds of times for a single task. If each invocation takes seconds, the entire workflow can take hours. Faster models (or more efficient inference) not only improve user experience but enable more sophisticated agentic patterns, with more reflection cycles, more planning steps, and more interactions between agents.

### Industry Impact

Industry forecasts confirm Ng's vision. According to Gartner (2024), by 2026, 75% of enterprises will operationalize AI architectures, with agentic AI as a fundamental component. IDC predicts that by 2027, 60% of global knowledge workers will interact daily with AI agents. Market data already shows explosive growth: in 2025, 72% of enterprise AI projects involve multi-agent architectures, compared to 23% in 2024.

### The Paradigm Shift in Training

Ng insists on a fundamental point: competence in designing agentic workflows is becoming one of the most sought-after skills in the AI job market. It is no longer enough to know how to do prompt engineering on a single model; you need to know how to decompose business problems into agentic workflows, choose the right patterns for each sub-task, design feedback and iteration mechanisms, and build robust systems that gracefully handle failures.

This is why DeepLearning.AI's "Agentic AI" course teaches how to build agents starting from fundamental principles in Python, before exploring existing frameworks. A deep understanding of the underlying mechanisms is essential to designing effective agentic systems.

### A Simple Recipe to Get Started

In his communications, Ng proposes a pragmatic approach for those who want to start with agentic workflows. He suggests:

1. Identify a repetitive and well-defined task in your workflow
2. First implement a non-agentic approach (zero-shot) as a baseline
3. Add Reflection: have the model evaluate its own output and iterate
4. Add Tool Use where necessary to overcome model limits
5. Consider Planning for tasks that require multiple steps
6. Explore Multi-Agent only when complexity justifies it

Each step should be measured with clear metrics, comparing output quality with the baseline and with the previous step.

---

## 6. Resources and References

### Course and Platform

- [Agentic AI with Andrew Ng - DeepLearning.AI](https://www.deeplearning.ai/courses/agentic-ai/) - Complete course on the 4 agentic patterns, 5 modules, capstone project included
- [DeepLearning.AI - Agentic Design Patterns](https://learn.deeplearning.ai/courses/agentic-ai/lesson/rm9bg7/agentic-design-patterns) - Video lesson on agentic patterns

### Articles and Newsletter

- [Andrew Ng - Agentic Design Patterns Part 2: Reflection (The Batch)](https://www.deeplearning.ai/the-batch/agentic-design-patterns-part-2-reflection/) - Deep dive on the Reflection pattern
- [Letters from Andrew Ng - The Batch](https://www.deeplearning.ai/the-batch/tag/letters/) - Ng's weekly letters on The Batch newsletter

### Presentations and Talks

- [What's Next for AI Agentic Workflows - Sequoia Capital AI Ascent 2024 (Transcript)](https://transcript.lol/read/youtube/@sequoiacapital/66043f7cd2f24aea73b215e4) - Transcript of the original presentation
- [Notes on Agentic Reasoning - Octet Consulting](https://octetdata.com/blog/notes-andrew-ng-agentic-reasoning-2024/) - Detailed notes on Ng's presentation at Sequoia AI Ascent
- [AI Agentic Workflows by Andrew Ng - Tzamtzis](https://tzamtzis.gr/2024/coding/ai-agentic-workflows-by-andrew-ng/) - Analysis of the presentation

### Social Media and Posts

- [Andrew Ng on X - Announcement of the 4 Patterns](https://x.com/AndrewYNg/status/1773393357022298617) - Original post describing the four patterns
- [Andrew Ng on X - Multi-Agent Collaboration](https://x.com/AndrewYNg/status/1780991671855161506) - Deep dive on the multi-agent pattern
- [Andrew Ng on X - Planning Pattern](https://x.com/AndrewYNg/status/1779606380665803144) - Deep dive on the planning pattern
- [Andrew Ng on X - AI Agentic Workflows](https://x.com/AndrewYNg/status/1770897666702233815) - Post on the central thesis of agentic workflows

### Profile and Biography

- [Andrew Ng - Official Site](https://www.andrewng.org/)
- [Andrew Ng - Wikipedia](https://en.wikipedia.org/wiki/Andrew_Ng)
- [Andrew Ng - Stanford HAI](https://hai.stanford.edu/people/andrew-ng)
- [Andrew Ng - LinkedIn](https://www.linkedin.com/in/andrewyng/)
- [Andrew Ng - Coursera Instructor](https://www.coursera.org/instructor/andrewng)

### Related Scientific Papers

- Madaan et al. (2023) - *Self-Refine: Iterative Refinement with Self-Feedback*
- Shinn et al. (2023) - *Reflexion: Language Agents with Verbal Reinforcement Learning*
- Gou et al. (2024) - *CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing*

### Cited Frameworks and Projects

- [ChatDev - GitHub](https://github.com/OpenBMB/ChatDev) - Multi-agent virtual software company
- [AutoGen - Microsoft Research](https://github.com/microsoft/autogen) - Framework for multi-agent conversations
- [CrewAI - GitHub](https://github.com/crewAIInc/crewAI) - Framework for agent orchestration (incubated by Andrew Ng's AI Fund)

### Analyses and Retrospectives

- [Agentic AI: One Year After Andrew Ng's Design Patterns - Hailey Quach](https://medium.com/@haileyq/agentic-ai-one-year-after-andrew-ngs-design-patterns-hype-or-reality-6fbd87dbe870) - Analysis one year after the original presentation
- [4 Agentic Design Patterns and 4 Key AI Trends 2024-2025 - ML Notes](https://mlnotes.substack.com/p/4-agentic-design-patterns-and-4-key) - Overview of patterns and trends
- [Andrew Ng Introduces Agentic AI Design Patterns for 2024 - BotNirvana](https://members.botnirvana.org/andrew-ng-introduces-agentic-ai-design-patterns-for-2024/) - Introduction to the patterns for 2024

---

*Document compiled in March 2026 based on the presentations, publications, and public communications of Andrew Ng on AI agentic workflows and patterns.*
