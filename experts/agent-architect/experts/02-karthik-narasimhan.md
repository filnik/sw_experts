# Karthik Narasimhan: Architect of Modern Language Agents

## 1. Profile and Academic Background

### Training and Path

Karthik R. Narasimhan is currently **Associate Professor of Computer Science** at Princeton University, where he also holds the roles of **co-director of the Princeton NLP Group** and **Associate Director of Princeton Language and Intelligence (PLI)**. His academic and professional trajectory represents one of the most influential paths in contemporary research on agents based on language models.

Narasimhan obtained his **PhD at MIT in 2017**, under the supervision of Prof. Regina Barzilay, with a thesis titled *"Grounding Natural Language with Autonomous Interaction"*. The thesis presented innovative computational models that integrated reinforcement learning with natural language understanding to induce grounded representations of semantics, using unstructured feedback to enable representations optimized for the specific task. This work laid the theoretical foundations for all his subsequent research: the idea that language should not be treated as an isolated artifact, but as a tool that agents use to interact with and adapt to complex environments.

After his PhD, Narasimhan spent a year (2017-2018) as a **visiting research scientist at OpenAI**, where he co-authored the first GPT paper (*"Improving Language Understanding by Generative Pre-Training"*, Radford, Narasimhan et al., 2018). This work demonstrated how an autoregressive transformer-based approach could be used for language modeling and introduced the revolutionary idea of solving NLP tasks through token prediction, laying the foundations for the entire family of GPT models that has transformed the field of artificial intelligence.

Since 2018, he has served at Princeton, where he has built one of the most productive research groups in the world in the field of language agents. From 2023 to 2025 he simultaneously held the role of **Head of Research at Sierra**, the conversational AI startup founded by Bret Taylor and Clay Bavor, where he led the development of benchmarks and systems for AI agents oriented toward the real world.

### Recognitions

Narasimhan has been awarded numerous prizes and recognitions:
- **NSF CAREER Award** - the most prestigious recognition for young researchers from the National Science Foundation
- **Google Research Scholar Award** (2022)
- **Amazon Research Award** (2019)
- **Best Paper Awards/Nominations** at EMNLP (2015, 2016)
- Oral presentations at NeurIPS 2023 (Tree of Thoughts) and ICLR 2024 (SWE-bench)

### Research Vision

Narasimhan's research vision revolves around a unifying principle: **developing intelligent autonomous agents that interact with and adapt to complex real-world environments**. His contributions span from the creation of fundamental paradigms (ReAct, Tree of Thoughts), to theoretical frameworks (CoALA), rigorous benchmarks (SWE-bench, WebShop, TAU-bench), and complete agentic systems (SWE-agent). This combination of theory, evaluation infrastructure, and practical implementation makes him a unique figure in the AI agent research landscape.

---

## 2. CoALA: Cognitive Architectures for Language Agents

### Context and Motivation

The paper *"Cognitive Architectures for Language Agents"* (Sumers, Yao, Narasimhan, Griffiths, 2023), published in *Transactions on Machine Learning Research*, represents Narasimhan's most ambitious theoretical contribution. At a time when research on language agents was exploding with dozens of ad hoc approaches -- from AutoGPT to BabyAGI, from LangChain to various ChatGPT wrappers -- a systematic framework was missing to organize, classify, and guide the development of these systems. CoALA fills exactly this gap.

The central motivation of the paper is that, despite the remarkable empirical success of language agents, the research community lacked a systematic framework to: (a) organize existing agents into a coherent taxonomy, (b) identify common design principles, and (c) plan future directions toward more capable agents. CoALA draws on the tradition of cognitive sciences and symbolic AI to provide this framework.

### Architecture: The Three Pillars

CoALA organizes a language agent along three fundamental dimensions: **memory**, **action space**, and **decision process**.

#### 2.1 Modular Memory System

CoALA proposes a memory organization inspired by cognitive psychology, with a fundamental distinction between working memory and long-term memories.

**Working Memory:** Represents the short-term storage containing "active and immediately available information as symbolic variables for the current decision cycle". It acts as a central hub that synthesizes inputs for the LLM from prompt templates and variables, then parses outputs and converts them back into updatable components of the working memory. In implementation terms, the working memory corresponds to the agent's current context: active instructions, recent observations, intermediate results of reasoning.

**Episodic Memory:** Stores experiences from previous decision cycles, including training pairs, event histories, or game trajectories. These experiences can be retrieved during planning to support reasoning and can be written as a form of learning. For example, an agent that has successfully resolved a bug in a Python project can store the sequence of actions taken (file search, code reading, patch application, test execution) as a reusable episode in the future.

**Semantic Memory:** Maintains world knowledge and the agent's knowledge about itself. In traditional approaches, semantic memory is limited to retrieval from fixed external databases (such as Wikipedia). In advanced agents, the agent writes auto-generated inferences into semantic memory, combining human knowledge with insights derived from the agent itself to enable "lifelong learning" efficient over the life cycle.

**Procedural Memory:** Comprises two forms: implicit knowledge in the LLM's weights and explicit knowledge in the agent's code. The agent code implements specific procedures for actions and decision processes, which designers must initialize to start the agent. This distinction is crucial: procedural memory is not just "what the model knows how to do", but also "how the system is structured to do it".

#### 2.2 Action Space

CoALA divides actions into two macro-categories: **internal** (memory-oriented) and **external** (environment-oriented).

**Internal Actions:**

- *Reasoning Actions:* Process the contents of working memory using the LLM to generate new information. They support both learning and decision-making through intermediate reasoning steps. This is the mechanism behind approaches such as Chain-of-Thought and ReAct.

- *Retrieval Actions:* Read information from long-term memories into working memory via rule-based, sparse, or dense retrieval methods. They enable adaptive and context-sensitive recall, allowing the agent to access relevant knowledge at the right moment.

- *Learning Actions:* Write to long-term memories through multiple mechanisms: episodic updates that store experiences, semantic updates that store reasoned inferences, parameter updates via fine-tuning, and code updates that modify prompt templates, grounding skills, or retrieval procedures.

**External Actions (Grounding Actions):** Execute in three types of environment:
- *Physical environments* via robotic systems that use vision-language models for perception
- *Dialogic interactions* with humans or other agents to receive instructions and learn
- *Digital environments* that include games, APIs, websites, and code execution environments

#### 2.3 Decision Process

CoALA structures the decision process in repeated cycles with two phases:

**Planning Phase:** Flexibly applies reasoning and retrieval to propose, evaluate, and select actions:
- *Proposal:* Generation of action candidates through reasoning (sampling one or more actions) or inclusion of all actions in simple domains
- *Evaluation:* Value assignment via heuristics, LLM perplexity, learned values, or LLM reasoning
- *Selection:* Choice via argmax, softmax, or alternatives such as majority voting

**Execution Phase:** Application of the procedures selected by the agent code, resulting in grounding or learning actions, observation of environmental feedback, and return to the cycle.

### Taxonomy of Existing Agents

One of the most valuable contributions of CoALA is the systematic classification of existing agents along the three dimensions of the framework:

| Agent | Memory | Grounding | Internal Actions | Decision |
|-------|--------|-----------|------------------|----------|
| **SayCan** | Procedural only | Physical skills (551) | None | Fixed-set evaluation |
| **ReAct** | Procedural only | Digital APIs | Reasoning | Single proposal step |
| **Voyager** | Procedural (skill library) | Digital (Minecraft) | Reasoning/Retrieval/Learning | Proposal with refinement |
| **Generative Agents** | Episodic/Semantic | Digital/Multi-agent | Reasoning/Retrieval/Learning | Plan-based execution |
| **Tree of Thoughts** | None | Digital (problem submission) | Reasoning only | Iterative proposal-evaluation-selection |

This taxonomy reveals that sophisticated agents combine multiple memory types with diversified internal action spaces, while simpler agents focus on reasoning or fixed evaluation strategies.

### Implications for Agent Design

CoALA outlines a path toward what the authors call **"general intelligence based on language"**, identifying concrete directions: deeper integration of episodic and semantic memory, the development of learning mechanisms that modify procedural memory autonomously, and the creation of more sophisticated decision processes that balance exploration and exploitation.

---

## 3. SWE-bench and SWE-agent: Revolutionizing Automated Software Engineering

### SWE-bench: The Benchmark

*"SWE-bench: Can Language Models Resolve Real-World GitHub Issues?"* (Jimenez, Yang, Wettig, Yao, Pei, Press, Narasimhan, 2024), published at **ICLR 2024 as an oral presentation**, represents one of the most influential and widely adopted benchmarks in the era of language models.

#### Motivation and Design

The fundamental motivation is incisive: **"Language models have surpassed our ability to evaluate them effectively."** Existing benchmarks such as HumanEval and MBPP evaluated code generation on isolated synthetic problems, completely disconnected from the complexity of real software engineering. SWE-bench radically changes the rules of the game.

The dataset comprises **2,294 software engineering problems** extracted from real GitHub issues and related pull requests, from **12 popular Python repositories** including Django, Flask, Scikit-learn, Matplotlib, Sympy, Requests, and others. Each problem presents a complete codebase accompanied by a description of the issue that the model must resolve by generating a patch.

#### Task Complexity

Unlike traditional benchmarks, SWE-bench problems frequently require:
- **Understanding and coordination of changes** across functions, classes, and multiple files
- **Interaction with execution environments** to verify code behavior
- **Processing of extremely long contexts** (entire codebases)
- **Complex reasoning** that goes well beyond traditional code generation
- **Domain knowledge** specific to the libraries involved

#### Initial Results and Evolution

The initial results were sobering: the best-performing model at the time of publication, Claude 2, achieved only a **resolution rate of 1.96%**. This figure highlighted the enormous gap between the capabilities demonstrated by LLMs on synthetic benchmarks and the reality of software engineering.

The subsequent evolution has been extraordinary. With the introduction of **SWE-bench Verified** (a curated version with human verification of task quality), performance has improved dramatically thanks to more powerful models and more sophisticated agentic scaffolds. As of today (March 2026), the best systems reach about 80% on SWE-bench Verified -- a leap of nearly two orders of magnitude compared to initial results. This progress is not attributable solely to the improvement of base models, but largely to the design of the agent-computer interface and agentic architectures, as demonstrated by SWE-agent itself.

SWE-bench has become the de facto benchmark for evaluating AI-assisted coding systems, adopted by OpenAI, Anthropic, Google, and practically every research lab working on agents for software engineering.

### SWE-agent: The Agent

*"SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering"* (Yang, Jimenez, Wettig, Lieret, Yao, Narasimhan, Press, 2024), published at **NeurIPS 2024**, introduces a fundamental concept: the **Agent-Computer Interface (ACI)**.

#### The ACI Concept

The central insight of SWE-agent is that language model agents represent a **new category of end users** with their own needs and capabilities, and would benefit from interfaces purpose-built for them. Just as the Human-Computer Interface (HCI) revolutionized the way humans interact with computers, the ACI proposes to do the same for agents.

The paper demonstrates that careful ACI design can **substantially improve agent performance without modifying the weights of the underlying language model**. This is a profound observation: it suggests that much of the agent potential remains unexplored due to suboptimal interfaces, not intrinsic model limitations.

#### ACI Design Principles

SWE-agent establishes two key principles for ACI design:

1. **Simplicity and understandability:** Actions must be simple and easy to understand. Commands with few options and concise documentation are easier for agents to use. This principle opposes the common approach of exposing agents to the entire complexity of Unix/Linux tools.

2. **Efficiency:** Important operations (file navigation, editing, execution) must be consolidated into the smallest possible number of actions. Each additional action increases the probability of error and token consumption in the context.

#### Interface Components

SWE-agent's customized ACI significantly improves an agent's ability to:
- **Create and modify code files** through structured editing commands that prevent common errors such as incorrect indentation or accidental overwriting
- **Navigate entire repositories** with optimized search and browsing commands that show relevant context without overloading the agent's context
- **Execute tests and other programs** with structured feedback that facilitates debugging

#### Performance

SWE-agent achieved state-of-the-art performance on both SWE-bench and HumanEvalFix with a pass@1 rate of **12.5%** and **87.7%** respectively, far surpassing the previous state-of-the-art obtained with non-interactive LLMs. The leap from Claude 2's 1.96% to SWE-agent's 12.5% demonstrates the massive impact of interface design.

---

## 4. Contribution to the ReAct Paradigm

### ReAct: The Synergy between Reasoning and Action

The paper *"ReAct: Synergizing Reasoning and Acting in Language Models"* (Yao, Zhao, Yu, Du, Shafran, Narasimhan, Cao, 2022), presented at **ICLR 2023**, is probably the most influential and widely cited contribution among Narasimhan's. ReAct defined a fundamental paradigm that pervades practically every modern agentic system.

#### The Problem

Before ReAct, two separate approaches to using LLMs existed:
- **Chain-of-Thought (CoT):** The model generates internal reasoning traces to improve performance on reasoning tasks, but does not interact with external sources and suffers from hallucinations and error propagation
- **Action-based:** The model generates actions to interact with external environments (APIs, databases), but without explicit reasoning, limiting planning capability and exception handling

#### The Innovation

ReAct proposes generating **task-specific reasoning traces and actions in an interleaved manner**, creating a powerful synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow interfacing with external sources (such as knowledge bases or environments) to gather additional information.

A typical ReAct cycle looks like this:
```
Thought: I need to find the founding year of Princeton University.
Action: search["Princeton University"]
Observation: Princeton University is a private research university in Princeton,
             New Jersey. Founded in 1746...
Thought: Princeton University was founded in 1746. Now I need to verify if...
Action: search["oldest universities United States"]
...
```

#### Key Results

On question answering tasks (HotpotQA) and fact verification (Fever), ReAct overcomes the hallucination and error propagation problems typical of chain-of-thought reasoning by interacting with a simple Wikipedia API, generating task resolution trajectories similar to human ones. On two interactive decision-making benchmarks (ALFWorld and WebShop), ReAct outperforms imitation and reinforcement learning methods with an absolute success rate improvement of **34%** and **10%** respectively, while being fed with only one or two in-context examples.

#### Narasimhan's Role

Narasimhan's contribution to ReAct should be understood in the context of his supervisory and directorial work in the Princeton NLP Group. Shunyu Yao, first author of the paper, is one of his doctoral students. Narasimhan's vision -- agents using language to interact with complex environments -- provided the intellectual substrate on which ReAct was built. The continuity between Narasimhan's PhD thesis at MIT (grounding language through autonomous interaction) and ReAct is evident and direct.

#### Impact and Legacy

ReAct has had a transformative impact:
- It has become the dominant design pattern for LLM agents
- Frameworks such as LangChain, LlamaIndex, and CrewAI implement ReAct as a default architecture
- It has inspired variants such as Reflexion, LATS, and numerous other approaches
- The term "ReAct agent" itself has entered the standard vocabulary of AI engineering

---

## 5. Other Significant Contributions

### Tree of Thoughts (ToT)

*"Tree of Thoughts: Deliberate Problem Solving with Large Language Models"* (Yao, Yu, Zhao, Shafran, Griffiths, Cao, Narasimhan, 2023), presented as an **oral at NeurIPS 2023**, generalizes Chain-of-Thought into a tree structure. While CoT proceeds linearly (one thought after another), ToT allows the model to explore multiple parallel reasoning paths, evaluate them, and backtrack when necessary. This enables a form of *deliberate problem solving* inspired by tree search in classical AI (such as game trees), but applied to natural language reasoning.

### WebShop

WebShop (2022) introduced a benchmark for web agents that must navigate simulated but realistic e-commerce sites, searching for and purchasing products based on natural language instructions. It anticipated the line of web agents that today is one of the most active in research.

### TAU-bench

TAU-bench (*Tool-Agent-User benchmark*, 2024), developed during the Sierra period and accepted at **ICLR 2025**, introduces a dual-control evaluation environment for agents that interact with users and tools simultaneously. The benchmark requires that agents: interact seamlessly with humans and programmatic APIs over extended periods; accurately follow complex domain-specific policies; be consistent and reliable at scale. The results have shown that even the best available model (GPT-4o at the time of publication) achieved a success rate of less than 50%, highlighting how long the road to reliable AI agents in the real world still is.

---

## 6. Key Concepts for Agent Builders

### Lesson 1: The Interface is Fundamental (ACI)

SWE-agent demonstrates that the design of the interface between agent and environment can be more impactful than improving the underlying model. For those building agents:
- **Design simple and atomic tools** rather than exposing complex interfaces
- **Document tools concisely** thinking of the agent as a user
- **Consolidate frequent operations** into single efficient actions
- **Provide structured feedback** that minimizes ambiguity

### Lesson 2: Memory is Multidimensional (CoALA)

There is no single type of memory that is sufficient. CoALA teaches that a robust agent needs:
- **Working memory** for immediate context (context window management)
- **Episodic memory** to learn from past experiences (logs of previous executions)
- **Semantic memory** to access and accumulate knowledge (RAG, knowledge bases)
- **Procedural memory** both implicit (LLM capabilities) and explicit (code, prompt templates)

### Lesson 3: Reasoning and Action Must Be Synergistic (ReAct)

ReAct demonstrates that separating reasoning and action is suboptimal. For those designing agents:
- **Interleave thought and action** in every agent cycle
- **Make reasoning explicit** (scratchpad, chain of thought) to improve debugging and interpretability
- **Allow reasoning to inform actions** and observations to inform subsequent reasoning

### Lesson 4: Evaluate on Real Problems (SWE-bench)

SWE-bench has shown that synthetic benchmarks massively overestimate agent capabilities. For developers:
- **Test on real data** from the target domain, not on synthetic benchmarks
- **Measure end-to-end** (from problem understanding to verified solution)
- **Build sustainable benchmarks** that can grow with model improvement

### Lesson 5: Structured Exploration Beats Linear Generation (ToT)

Tree of Thoughts demonstrates that for complex problems, generating a single linear solution is insufficient:
- **Implement backtracking mechanisms** when a reasoning path fails
- **Evaluate intermediate solutions** rather than only the final result
- **Explore alternatives in parallel** when computational cost allows

### Lesson 6: Reliability Benchmarking (TAU-bench)

TAU-bench reminds us that average performance is not enough:
- **Measure agent consistency** over repeated executions
- **Test with dynamic users** who change requirements and present edge cases
- **Evaluate adherence to policy** as a critical requirement for production deployment

---

## 7. Impact and Resources

### Quantitative Impact

Narasimhan's contributions have had measurable and massive impact on the community:
- **ReAct** is among the most cited papers of recent years in the AI field, with thousands of citations
- **SWE-bench** is adopted as a standard benchmark by OpenAI, Anthropic, Google, Meta, and most research labs
- **SWE-agent** is one of the most active open-source projects in the field of coding agents
- **Tree of Thoughts** has catalyzed an entire line of research on deliberate reasoning in LLMs
- **CoALA** provides the vocabulary and conceptual framework used by the community to discuss agentic architectures

### Main Resources

**Papers:**
- ReAct: [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) - ICLR 2023
- Tree of Thoughts: [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) - NeurIPS 2023 (Oral)
- CoALA: [arXiv:2309.02427](https://arxiv.org/abs/2309.02427) - TMLR 2024
- SWE-bench: [arXiv:2310.06770](https://arxiv.org/abs/2310.06770) - ICLR 2024 (Oral)
- SWE-agent: [arXiv:2405.15793](https://arxiv.org/abs/2405.15793) - NeurIPS 2024

**Code and Benchmarks:**
- SWE-bench: [github.com/SWE-bench/SWE-bench](https://github.com/SWE-bench/SWE-bench)
- SWE-agent: [github.com/SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent)
- Tree of Thoughts: [github.com/princeton-nlp/tree-of-thought-llm](https://github.com/princeton-nlp/tree-of-thought-llm)
- SWE-bench Leaderboard: [swebench.com](https://www.swebench.com/)
- TAU-bench: [github.com/sierra-research/tau-bench](https://github.com/sierra-research/tau-bench)
- Awesome Language Agents (CoALA): [github.com/ysymyth/awesome-language-agents](https://github.com/ysymyth/awesome-language-agents)

**Profiles:**
- Homepage: [karthikncode.github.io](https://karthikncode.github.io/)
- Princeton CS: [cs.princeton.edu/people/profile/karthikn](https://www.cs.princeton.edu/people/profile/karthikn)
- Google Scholar: [scholar.google.com/citations?user=euc0GX4AAAAJ](https://scholar.google.com/citations?user=euc0GX4AAAAJ)
- Sierra: [sierra.ai/author/karthik-narasimhan](https://sierra.ai/author/karthik-narasimhan)

### Conceptual Map of Contributions

```
Karthik Narasimhan - Main Contributions
|
+-- REASONING PARADIGMS
|   +-- ReAct (2022) --> Reasoning-action synergy
|   +-- Tree of Thoughts (2023) --> Deliberate exploration
|
+-- THEORETICAL FRAMEWORKS
|   +-- CoALA (2023) --> Cognitive architecture for agents
|       +-- Memory: Working / Episodic / Semantic / Procedural
|       +-- Actions: Internal (reasoning, retrieval, learning) / External (grounding)
|       +-- Decision: Propose -> Evaluate -> Select -> Execute
|
+-- BENCHMARKS AND EVALUATION
|   +-- SWE-bench (2024) --> Real SW engineering benchmark
|   +-- WebShop (2022) --> Web agents benchmark
|   +-- TAU-bench (2024) --> Tool-agent-user interaction benchmark
|
+-- AGENTIC SYSTEMS
|   +-- SWE-agent (2024) --> Agent Computer Interface (ACI)
|
+-- FOUNDATIONS
    +-- GPT (2018) --> Co-author of the first GPT paper at OpenAI
    +-- PhD MIT (2017) --> Language grounding through interaction
```

### Conclusion

Karthik Narasimhan occupies a unique position in contemporary artificial intelligence research. His trajectory -- from language grounding at MIT, to co-development of GPT at OpenAI, to the creation of fundamental paradigms such as ReAct, theoretical frameworks such as CoALA, transformative benchmarks such as SWE-bench, and practical systems such as SWE-agent -- traces a coherent and cumulative narrative arc. Each contribution builds on previous ones, progressively expanding the vision of autonomous agents that use language as a tool to interact with the real world. For anyone building agentic systems today, Narasimhan's work and that of his group are not simply recommended reading: they are essential readings that define the vocabulary, architectures, and evaluation criteria of the entire field.
