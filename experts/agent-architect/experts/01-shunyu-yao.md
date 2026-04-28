# Shunyu Yao - Architect of Modern Language Agents

---

## 1. Profile and Background

**Shunyu Yao** (in Chinese: 姚顺雨) is one of the most influential researchers in the field of AI agents based on large language models (LLMs). His work has laid the conceptual and technical foundations for what we now call "agentic AI" -- the ability of a language model to reason and act autonomously in the digital world.

### Academic background

Yao began his academic journey in China, where in 2014 he won a silver medal at the National Olympiad in Informatics (NOI). The following year, he ranked third in sciences in Anhui province in the university entrance examination, earning a place in the highly prestigious **Yao Class** at Tsinghua University -- an excellence program in computer science founded by Turing Award winner Andrew Chi-Chih Yao. He graduated from Tsinghua in 2019.

Subsequently, Yao pursued his PhD in Computer Science at **Princeton University**, under the supervision of Professor **Karthik Narasimhan** in the Princeton NLP Group. During his PhD, he received the **Harold W. Dodds Fellowship**, one of the most prestigious recognitions awarded by the university. He defended his doctoral thesis on May 2, 2024, with a work titled *"Language Agents: From Next-Token Prediction to Digital Automation"*.

Before his PhD, Yao accumulated research experiences at **Google**, **Microsoft**, and **MIT**, contributing to building an interdisciplinary vision spanning natural language processing, reinforcement learning, and cognitive sciences.

### Post-academic career

In August 2024, immediately after completing his PhD, Yao joined **OpenAI**, where he was rapidly placed on the core team and contributed to the development of the company's first agentic products launched in 2025: **Operator**, **Deep Research**, and the **Computer-Using Agent (CUA)**. In particular, he led the development of the CUA, a universal agent designed to interact with the digital world through graphical interfaces.

In 2025, Yao left OpenAI to join **Tencent** as **Chief AI Scientist**, reporting directly to President Martin Lau. This move marked one of the most significant talent acquisitions in the AI sector, with a contract reported by press sources at approximately 14 million dollars.

To date, his publications have accumulated over **15,000 citations**, an extraordinary number for such a young researcher (born 1997/1998).

---

## 2. Foundational Contribution: ReAct

### The original paper

**"ReAct: Synergizing Reasoning and Acting in Language Models"** was published in 2022 (arXiv: 2210.03629) and presented as **Oral** at **ICLR 2023**. The authors are Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao.

This paper probably represents the single most influential contribution in the field of language agents. It introduced a paradigm that has become the de facto standard for building agents based on LLMs.

### The problem ReAct solves

Before ReAct, the research world addressed reasoning and acting as separate problems:

- **Chain-of-Thought (CoT)**: models generated step-by-step reasoning chains to solve problems, but did so in a completely internal way, without interaction with the external world. This led to **hallucination** and **error propagation** problems -- the model could construct coherent reasoning but based on invented facts, with no verification mechanism.

- **Action-only approaches**: other approaches had models act directly (generating actions, calling APIs, navigating environments), but without an explicit reasoning process. This made the behavior opaque, difficult to interpret, and incapable of strategic planning.

ReAct resolves both limitations by unifying reasoning and action in a single interleaved flow.

### How ReAct works: the technical mechanism

The heart of ReAct is an iterative loop composed of three elements that alternate:

1. **Thought**: the model generates a reasoning trace in natural language. This thought can serve to:
   - Decompose the task into sub-goals
   - Track progress toward the goal
   - Formulate an action plan
   - Handle exceptions or unexpected results
   - Synthesize collected information

2. **Action**: based on the thought, the model generates a concrete action that interacts with an external environment. Actions can include:
   - Querying an API (e.g., Wikipedia search)
   - Executing a command
   - Navigating a web page
   - Manipulating an object in a simulated environment

3. **Observation**: the environment returns the result of the action. This observation is incorporated into the context and feeds the subsequent thought.

The loop repeats: **Think -> Act -> Observe -> Think -> Act -> Observe -> ...** until task completion.

### A concrete example of a ReAct trace

To illustrate how it works, consider a question answering task on HotpotQA:

**Question**: "Aside from the Apple Remote, what other device can control the program Apple Remote was originally designed to interact with?"

```
Thought 1: I need to search Apple Remote to find out what program
           it was originally designed to interact with.
Action 1:  Search[Apple Remote]
Observation 1: The Apple Remote was originally designed to control
               Apple's Front Row software...

Thought 2: Apple Remote was designed for Front Row. Now I need to search
           which other devices can control Front Row.
Action 2:  Search[Front Row software]
Observation 2: Front Row could be controlled via Apple Remote
               or the Mac keyboard via key combinations...

Thought 3: Front Row can be controlled by the Apple Remote or the
           keyboard. The answer is "keyboard".
Action 3:  Finish[keyboard]
```

The key is that each thought is **informed** by the previous observation, and each action is **motivated** by the current thought. It is not a predetermined linear chain, but an adaptive process.

### The reasoning-acting synergy: why it works

ReAct works because reasoning and action mutually reinforce each other:

- **Reasoning guides action**: thought traces allow the model to formulate plans, decide which information to seek, and handle unforeseen cases. Without explicit reasoning, an action-only agent would tend to perform random or repetitive actions.

- **Action grounds reasoning**: by interacting with external sources (databases, APIs, environments), the model obtains real information that anchors reasoning to reality. Without actions, a reasoning-only model is vulnerable to hallucinations because it reasons solely on the basis of its parametric knowledge.

### Experimental results

The paper demonstrates ReAct's effectiveness on four benchmarks covering both linguistic and decision-making tasks:

**Knowledge-intensive reasoning tasks:**

- **HotpotQA** (multi-hop question answering): ReAct outperforms pure chain-of-thought methods by interacting with the Wikipedia API, drastically reducing hallucinations. The model can verify its own reasoning by searching for real information.

- **FEVER** (fact verification): ReAct generates interpretable trajectories superior to baselines, allowing exact tracing of why the model classified a fact as true or false.

**Interactive decision-making tasks:**

- **ALFWorld** (simulated text environments): ReAct outperforms reinforcement learning-based methods by an impressive **34% absolute success rate**, using only one or two examples in the prompt.

- **WebShop** (simulated e-commerce): ReAct outperforms baselines by **10% absolute success rate**, again with very few context examples.

### Why ReAct is so important to industry

ReAct has transformed the way industry builds AI agents for several reasons:

1. **Interpretability**: every agent decision is accompanied by explicit reasoning in natural language. This makes the system debuggable, auditable, and understandable to humans.

2. **Generality**: the paradigm is applicable to any task requiring reasoning and interaction -- from information retrieval to web navigation, from code execution to file manipulation.

3. **Implementation simplicity**: ReAct does not require fine-tuning. It works with in-context prompting, making it immediately applicable with any sufficiently capable LLM.

4. **Composability**: the Thought-Action-Observation pattern is easily extensible with additional tools, persistent memories, or reflection mechanisms.

---

## 3. Other Contributions

Shunyu Yao's work goes well beyond ReAct. He has built a body of research covering the entire arc of the AI agents problem, from theoretical formulation to practical evaluation.

### 3.1 Tree of Thoughts (ToT) -- NeurIPS 2023, Oral

**Paper**: *"Tree of Thoughts: Deliberate Problem Solving with Large Language Models"* (arXiv: 2305.10601)

**Authors**: Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, Karthik Narasimhan

Tree of Thoughts generalizes Chain-of-Thought prompting by introducing a **tree exploration** of the reasoning space. Instead of following a single linear path of thought, ToT allows the model to:

- **Explore multiple reasoning paths** in parallel
- **Self-evaluate** the quality of each intermediate step
- Perform **lookahead** and **backtracking** when a path proves fruitless
- Use **search strategies** such as BFS (Breadth-First Search) and DFS (Depth-First Search) on the thought tree

The results are impressive: on the **Game of 24** task (finding a mathematical expression that yields 24 using four numbers), GPT-4 with Chain-of-Thought solved only **4%** of problems, while with Tree of Thoughts it reached **74%**. The paper was also tested on **Creative Writing** and **Mini Crosswords**.

The fundamental insight of ToT is that complex problem solving requires **deliberation** -- the ability to consider alternatives, evaluate them, and choose strategically -- rather than purely sequential generation.

### 3.2 WebShop -- NeurIPS 2022

**Paper**: *"WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents"*

**Authors**: Shunyu Yao, Howard Chen, John Yang, Karthik Narasimhan

WebShop is a simulated e-commerce environment containing **1.18 million real products** and **12,087 instructions** generated by human annotators. In this environment, an agent must navigate web pages, search for products, customize options, and complete purchases following natural language instructions.

WebShop was one of the first benchmarks to test the ability of language agents to interact with realistic web interfaces, and represented a fundamental precursor for subsequent work on web agents such as Operator and CUA.

### 3.3 Reflexion -- NeurIPS 2023

**Paper**: *"Reflexion: Language Agents with Verbal Reinforcement Learning"*

**Authors**: Noah Shinn, Federico Cassano, Beck Labash, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao

Reflexion introduces a **verbal reflection learning** mechanism: after failing a task, the agent generates a textual reflection on what went wrong and stores it in an **episodic memory**. In subsequent attempts, the agent uses these reflections to avoid previous errors.

The most notable result: Reflexion reaches **91% pass@1** on the **HumanEval** coding benchmark, surpassing the previous state-of-the-art of GPT-4, which stopped at 80%.

Reflexion is conceptually complementary to ReAct: while ReAct provides the reasoning-action loop within a single episode, Reflexion adds a learning loop **between episodes**, allowing the agent to improve over time.

### 3.4 CoALA: Cognitive Architectures for Language Agents -- TMLR 2024

**Paper**: *"Cognitive Architectures for Language Agents"*

**Authors**: Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, Thomas L. Griffiths

CoALA is perhaps Yao's most ambitious contribution from a theoretical standpoint. It is a **unifying conceptual framework** that organizes and systematizes the entire field of language agents, drawing on the tradition of cognitive sciences and symbolic artificial intelligence.

The CoALA architecture describes a language agent through three fundamental components:

1. **Modular memory**:
   - **Working Memory**: a data structure that persists between LLM calls, acting as the central hub connecting all agent components. At each call, a subset of the working memory is synthesized as input to the LLM, and the output is reprocessed and stored back.
   - **Long-term Memory**: divided into episodic memory (past experiences), semantic memory (factual knowledge), and procedural memory (skills and procedures).

2. **Structured action space**: actions that can be directed toward internal memory (reading/writing information), toward external environments (calling APIs, navigating the web), or toward the LLM itself (generating reasoning, planning).

3. **Generalized decision process**: a loop that selects actions based on the current state of working memory, inspired by cognitive decision-making models.

CoALA contextualizes language agents within the broader history of AI and traces a path toward general intelligence based on language.

### 3.5 SWE-bench and SWE-agent -- ICLR 2024 (Oral) and NeurIPS 2024

**SWE-bench** (*"Can Language Models Resolve Real-World GitHub Issues?"*) is a benchmark that tests the ability of models to solve real issues from GitHub repositories, with **2,294 tasks** extracted from 12 popular Python repositories. It has become the reference benchmark for evaluating software engineering agents.

**SWE-agent** (*"Agent-Computer Interfaces Enable Automated Software Engineering"*) introduces the fundamental concept of **Agent-Computer Interface (ACI)** -- the idea that language agents represent a new category of users that benefit from purpose-designed interfaces. SWE-agent's customized ACI drastically improves the agent's ability to create and modify code files, navigate repositories, and run tests, achieving **12.5% pass@1** on SWE-bench (state-of-the-art at the time) and **87.7%** on HumanEvalFix.

### 3.6 tau-bench -- ICLR 2025

**Paper**: *"tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains"*

**Authors**: Shunyu Yao, Noah Shinn, Pedram Razavi, Karthik Narasimhan

tau-bench addresses a critical gap in existing benchmarks: the lack of evaluation of agent-human user interaction in realistic contexts. The benchmark simulates dynamic conversations in two domains (retail and airlines) where a customer service agent must interact with simulated users, use domain-specific APIs, and follow company policies.

The paper introduces the **pass^k** metric to evaluate the reliability of agent behavior over multiple attempts. The results are sobering: even the best function-calling agents (GPT-4o) succeed in less than **50%** of tasks, with terribly low consistency (pass^8 < 25% in retail).

---

## 4. Key Concepts for Agent Builders

Anyone building AI agents must deeply understand the following concepts derived from Yao's work:

### 4.1 The Thought-Action-Observation loop as a fundamental primitive

Any agent that interacts with the external world should implement some variant of the ReAct cycle. This does not necessarily mean using the original ReAct prompting -- many modern frameworks use native LLM function calling -- but the **conceptual structure** remains identical:

```
while not task_completed:
    thought = llm.reason(context)          # Reasoning
    action = llm.select_action(thought)     # Decision
    observation = environment.execute(action)  # Execution
    context.update(thought, action, observation)  # Update
```

This pattern is the foundation upon which frameworks like **LangChain**, **LlamaIndex**, **CrewAI**, **AutoGen**, and many others are built.

### 4.2 The importance of Agent-Computer Interfaces (ACI)

From the work on SWE-agent emerges a crucial principle: **the interface you give the agent matters as much as the model itself**. An agent should not simply be "attached" to generic tools; interfaces must be designed specifically for non-human users:

- Simplified, atomic commands rather than complex ones
- Structured and predictable outputs
- Explicit feedback on errors and state
- Sufficient but not excessive context

### 4.3 Memory as an architectural component

From CoALA and Reflexion emerges the idea that a serious agent must have a **structured memory system**:

- **Working memory** for the context of the current conversation/task
- **Long-term memory** to accumulate knowledge and experiences
- **Episodic memory** to remember past errors and reflections (Reflexion pattern)

### 4.4 Deliberate exploration vs. sequential generation

Tree of Thoughts teaches that for complex problems, linear generation (even with CoT) is not enough. An agent builder must consider:

- When it is appropriate to explore multiple paths
- How to implement self-evaluation of intermediate steps
- When to allow backtracking
- The trade-off between computational cost and solution quality

### 4.5 Rigorous agent evaluation

From SWE-bench and tau-bench emerge fundamental lessons on evaluation:

- Static benchmarks do not capture the complexity of real use
- **Consistency** (pass^k) is as important as average performance
- Interaction with real users introduces complexity not present in isolated benchmarks
- Current agents are still far from the reliability needed for production deployment

### 4.6 Recurring architectural patterns

From the body of Yao's work, patterns emerge that an agent builder should know:

| Pattern | Source | When to use |
|---------|--------|-------------|
| ReAct loop | ReAct | Tasks requiring reasoning + tool interaction |
| Tree search | ToT | Problems with a wide solution space requiring exploration |
| Verbal reflection | Reflexion | Repeated tasks where the agent can learn from its own errors |
| Modular memory | CoALA | Agents that need to maintain complex state over time |
| Custom ACI | SWE-agent | When designing tools/interfaces for agents |

---

## 5. Impact on the Field

### Academic impact

Shunyu Yao's work has contributed decisively to establishing **language agents** as an autonomous and recognized field of research. Before ReAct (2022), the idea of using LLMs as controllers of autonomous agents was embryonic. After ReAct, Tree of Thoughts, and CoALA, a dedicated research community has formed, with specific workshops at major conferences (NeurIPS, ICLR, ICML) and a steady stream of publications.

The citations speak clearly: with over 15,000 total citations, Yao is one of the most cited researchers of his generation in the NLP/AI field. The ReAct paper alone has accumulated thousands of citations in less than three years.

### Industrial impact

The influence on industry is even more profound and pervasive:

**Frameworks and libraries**: practically every framework for building AI agents implements the ReAct pattern as a primary or default modality. LangChain offers a ready-to-use `ReActAgent`. LlamaIndex has a native implementation of the ReAct loop. BeeAI, CrewAI, AutoGen -- all adopt variants of the paradigm. The Thought-Action-Observation cycle has become so fundamental that many developers use it without even knowing it derives from a specific paper.

**Commercial products**: Yao's direct work at OpenAI led to the creation of:
- **Operator**: an agent capable of completing tasks on websites autonomously
- **Deep Research**: an agent that navigates the internet to conduct multi-step research and generate complete reports
- **Computer-Using Agent (CUA)**: a universal agent that interacts with graphical interfaces

These products represent the commercial materialization of Yao's research principles: agents that reason, act, observe, and iterate.

**Evaluation standards**: SWE-bench has become the standard benchmark for evaluating software engineering agents. Every new system -- from Cognition's Devin to Anthropic's Claude -- is measured primarily on SWE-bench. This has created common ground for comparing and improving different systems.

### The conceptual legacy

Perhaps Yao's most lasting impact is in the **conceptual vocabulary** he introduced. Terms and concepts such as:

- **Language agent**: an autonomous agent whose "brain" is a language model
- **Reasoning + Acting**: the integration of thought and action as a fundamental paradigm
- **Agent-Computer Interface**: the idea that agents need dedicated interfaces
- **Verbal reinforcement learning**: learning through textual reflection

These concepts have entered the everyday vocabulary of the AI community and have shaped the way we think about the design of agentic systems.

### The overall vision: from the doctoral thesis

Yao's dissertation, titled *"Language Agents: From Next-Token Prediction to Digital Automation"*, offers a coherent vision of the entire research trajectory. The central thesis is that language models, originally designed to predict the next token in a sequence, can be transformed into agents capable of complex digital automation through appropriate architectures that integrate reasoning, action, memory, and learning.

The dissertation traces an arc from WebShop (grounded web interaction) to ReAct (reasoning-action integration) to Tree of Thoughts (deliberate reasoning) to CoALA (unifying cognitive architecture), showing how each contribution builds on the previous toward an increasingly complete vision of what it means to build a capable language agent.

---

## 6. Resources and References

### Main papers (in chronological order)

| Year | Paper | Venue | Link |
|------|-------|-------|------|
| 2022 | WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents | NeurIPS 2022 | [GitHub](https://github.com/princeton-nlp/WebShop) |
| 2023 | ReAct: Synergizing Reasoning and Acting in Language Models | ICLR 2023 (Oral) | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) |
| 2023 | Reflexion: Language Agents with Verbal Reinforcement Learning | NeurIPS 2023 | [arXiv:2303.11366](https://arxiv.org/abs/2303.11366) |
| 2023 | Tree of Thoughts: Deliberate Problem Solving with Large Language Models | NeurIPS 2023 (Oral) | [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) |
| 2024 | Cognitive Architectures for Language Agents (CoALA) | TMLR 2024 | [arXiv:2309.02427](https://arxiv.org/abs/2309.02427) |
| 2024 | SWE-bench: Can Language Models Resolve Real-World GitHub Issues? | ICLR 2024 (Oral) | [swebench.com](https://www.swebench.com/) |
| 2024 | SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering | NeurIPS 2024 | [arXiv:2405.15793](https://arxiv.org/abs/2405.15793) |
| 2025 | tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains | ICLR 2025 | [arXiv:2406.12045](https://arxiv.org/abs/2406.12045) |

### PhD Dissertation

- **Title**: *Language Agents: From Next-Token Prediction to Digital Automation*
- **University**: Princeton University, Department of Computer Science
- **Advisor**: Karthik Narasimhan
- **Year**: 2024
- **PDF**: [ysymyth.github.io/papers/Dissertation-finalized.pdf](https://ysymyth.github.io/papers/Dissertation-finalized.pdf)

### Profiles and personal pages

- **Homepage**: [ysymyth.github.io](https://ysymyth.github.io/)
- **Google Scholar**: [scholar.google.co.uk/citations?user=qJBXk9cAAAAJ](https://scholar.google.co.uk/citations?user=qJBXk9cAAAAJ&hl=en)
- **Twitter/X**: [@ShunyuYao12](https://x.com/shunyuyao12)
- **GitHub (awesome-language-agents)**: [github.com/ysymyth/awesome-language-agents](https://github.com/ysymyth/awesome-language-agents)

### Code repositories

- **ReAct**: Integrated in LangChain, LlamaIndex, and other frameworks
- **Tree of Thoughts**: [github.com/princeton-nlp/tree-of-thought-llm](https://github.com/princeton-nlp/tree-of-thought-llm)
- **WebShop**: [github.com/princeton-nlp/WebShop](https://github.com/princeton-nlp/WebShop)
- **SWE-agent**: [github.com/SWE-agent](https://github.com/SWE-agent/mini-swe-agent)
- **tau-bench**: [github.com/sierra-research/tau-bench](https://github.com/sierra-research/tau-bench)

### Interviews and talks

- **Latent Space Podcast**: *"Language Agents: From Reasoning to Acting"* -- [latent.space/p/shunyu](https://www.latent.space/p/shunyu)
- **Thesis Defense Talk** (May 2024): covering WebShop, ReAct, ToT, CoALA, and SWE-bench/agent

### Practical guides and implementations

- **Prompt Engineering Guide -- ReAct**: [promptingguide.ai/techniques/react](https://www.promptingguide.ai/techniques/react)
- **IBM -- What is a ReAct Agent?**: [ibm.com/think/topics/react-agent](https://www.ibm.com/think/topics/react-agent)

---

*Document compiled as a reference resource for the agent-architect project. Last update: March 2026.*
