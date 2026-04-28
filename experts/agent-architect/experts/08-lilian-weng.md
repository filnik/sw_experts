# Lilian Weng -- "LLM Powered Autonomous Agents"

## The Founding Document of the Autonomous Agents Era

> *"Agent = LLM + Memory + Planning + Tool Use. This is probably just a start of a new era."*
> -- Lilian Weng, June 23, 2023

---

## 1. Profile and Background

### Who is Lilian Weng

Lilian Weng is one of the most influential figures in the field of applied artificial intelligence. A graduate of Peking University (bachelor's degree) and with a PhD from Indiana University Bloomington, she has built a career spanning academic research, robotics engineering, and AI systems safety at industrial scale.

Before OpenAI, Weng worked as a Research Scientist at Snapchat, where she focused on machine learning applications for social platforms. In 2017 she joined OpenAI, where she would spend seven years building some of the organization's fundamental pillars.

### The Career at OpenAI (2017-2024)

Weng's trajectory within OpenAI illustrates the very evolution of the organization:

**Phase 1 -- Robotics (2017-2020).** Weng started in the robotics team, contributing to the Dactyl project: a robotic hand trained to solve the Rubik's cube. The paper "Solving Rubik's Cube with a Robot Hand" (2019), of which she is co-author, demonstrated that neural networks trained entirely in simulation -- via reinforcement learning and a technique called Automatic Domain Randomization (ADR) -- could transfer complex skills to the physical world. This work required months of development to make the system robust enough to operate in the real world.

**Phase 2 -- Applied AI Research (2020-2023).** Weng founded and led the Applied AI Research team, which developed fundamental tools for the OpenAI ecosystem: the Fine-tuning API, the Embedding API, the Moderation endpoints, and the applied safety frameworks. These tools transformed OpenAI's models from research artifacts into products usable at scale.

**Phase 3 -- Safety Systems (2023-2024).** After the launch of GPT-4, Weng received the mandate to rethink OpenAI's safety systems vision and centralize the work into a single team. She built the Safety Systems team, which grew to over 80 scientists, engineers, project managers, and policy experts. Under her leadership, the team developed o1-preview, described as OpenAI's safest model up to that point, capable of resisting adversarial attacks while maintaining its utility. From August 2024 she held the role of Vice President of Research and Safety.

In November 2024 Weng left OpenAI after seven years, subsequently joining Fellows Fund as Distinguished Fellow. Her leadership was recognized in Business Insider's AI Power List 2024.

### The Lil'Log Blog

Since 2017, Weng has been publishing on the Lil'Log blog (lilianweng.github.io) in-depth study notes that cover the full spectrum of modern machine learning. These are not superficial posts: each article is a 20-30 minute read technical survey, with dozens of academic references, original diagrams, and explanations that start from mathematical foundations and reach practical implications.

Among the most relevant posts:

- **"A (Long) Peek into Reinforcement Learning"** (2018) -- fundamental overview of RL
- **"What are Diffusion Models?"** (2021) -- survey on diffusion models, became a standard reference
- **"Prompt Engineering"** (March 2023) -- technical guide to prompt engineering
- **"LLM Powered Autonomous Agents"** (June 2023) -- the post analyzed here
- **"Adversarial Attacks on LLMs"** (October 2023) -- adversarial attacks on language models
- **"Extrinsic Hallucinations in LLMs"** (July 2024) -- analysis of hallucinations in LLMs
- **"Reward Hacking in Reinforcement Learning"** (November 2024) -- the reward hacking phenomenon
- **"Why We Think"** (May 2025) -- on recent developments in the use of test-time compute

The quality and systematicity of these posts have made Lil'Log one of the most cited resources in the AI community, by both academic researchers and industry engineers.

---

## 2. The Agent Formula

### The Conceptual Architecture

The central contribution of the June 2023 post is the formalization of a conceptual architecture for LLM-based autonomous agents. Weng condenses the entire idea into an elegant formula:

```
Agent = LLM + Memory + Planning + Tool Use
```

In this framework, the Large Language Model functions as the **brain** of the agent -- the central controller that coordinates all operations. But the brain alone is not enough. A human is not just their brain: they have memory to remember past experiences, planning capability to break complex goals down into achievable sub-goals, and the ability to use tools (from the calculator to the search engine) to extend their native cognitive capabilities.

The analogy with human cognition is not coincidental. Weng explicitly draws parallels with neuroscience and cognitive psychology, particularly evident in the memory section. This interdisciplinary approach is one of the reasons why the post has had such a profound impact: it proposes not just an engineering architecture, but a **conceptual model** for thinking about what it means to build an agent.

### Why This Formula Matters

Before Weng's post, the AI agents field was fragmented. There were individual papers on chain-of-thought prompting, on tool use, on memory augmentation, on self-reflection. What was missing was a **unified vision** that connected all these components into a coherent architecture. Weng provided exactly this: a framework to organize thinking, categorize existing techniques, and identify areas where further research was needed.

The formula has become the common language of the field. When today we talk about "AI agents", we are implicitly referring to this four-component architecture.

---

## 3. Planning

Planning is the agent's ability to decompose complex tasks into manageable sub-goals and to refine its approach through self-reflection. Weng identifies two main dimensions.

### 3.1 Task Decomposition

Task decomposition transforms a broad and vague goal into a sequence of concrete and feasible steps. Weng analyzes three fundamental approaches:

**Chain of Thought (CoT) -- Wei et al., 2022.** CoT prompting instructs the model to "think step by step", making the reasoning process explicit, which would otherwise remain implicit in text generation. Instead of producing a direct answer, the model generates a chain of intermediate reasoning. This seemingly simple approach has a profound effect: it allows the model to allocate more compute to problems that require more reasoning steps, and makes the reasoning process inspectable and debuggable. In practice, it can be implemented with simple instructions like "Think step by step" or with few-shot examples that show explicit chains of reasoning.

**Tree of Thoughts (ToT) -- Yao et al., 2023.** ToT extends CoT by exploring **multiple reasoning paths** simultaneously at each step, creating a tree structure. Each node of the tree represents a partial "thought" -- an intermediate step of coherent reasoning. The tree is explored with classical search algorithms like BFS (Breadth-First Search) or DFS (Depth-First Search), and each state is evaluated through a classifier or majority voting. The fundamental advantage is the **backtracking** capability: if a reasoning path proves unfruitful, the agent can go back and try an alternative path, something impossible with the standard autoregressive generation of an LLM.

**LLM+P -- Liu et al., 2023.** This approach recognizes a fundamental limit: LLMs are good natural reasoners but poor formal planners. LLM+P externalizes planning to **external classical planners** using the Planning Domain Definition Language (PDDL). The flow is the following: (1) the LLM translates the problem from natural language into a PDDL specification, (2) a classical optimal planner generates the plan, (3) the LLM translates the PDDL plan back into natural language. This approach combines the linguistic flexibility of the LLM with the guaranteed correctness of formal planners.

### 3.2 Self-Reflection

Self-reflection allows the agent to iteratively improve by criticizing and refining its past actions. Weng examines several paradigms:

**ReAct -- Yao et al., 2023 (ICLR).** ReAct (Reasoning + Acting) integrates reasoning and action into a single cycle. The agent's action space is extended to include not only discrete task-specific actions, but also **natural language reasoning traces**. The cycle is: Thought --> Action --> Observation. The "Thought" is explicit reasoning about what to do and why; the "Action" is a concrete operation (e.g. web search, code execution); the "Observation" is the feedback from the environment. This cycle allows the agent to reason about what it is observing and adapt its behavior accordingly. ReAct has demonstrated that interleaving reasoning and action produces superior results compared to using reasoning or action alone.

**Reflexion -- Shinn & Labash, 2023.** Reflexion adds a **dynamic memory and self-reflection** mechanism using reinforcement learning signals. After the agent completes an attempt (potentially failed), a reflection component analyzes the trajectory and generates textual insights about what went wrong. These insights are stored and used in subsequent attempts. Reflexion also includes heuristic mechanisms to identify inefficient trajectories -- for example, detecting when the agent repeats the same identical action multiple times (signal of hallucination or loop). In case of inefficiency detection, the agent resets and starts over, but with the learned reflections at its disposal.

**Chain of Hindsight (CoH) -- Zhang et al., 2023.** CoH fine-tunes the model on sequences of past outputs ordered by quality. The model is trained on pairs (poor output, feedback, improved output), learning to progressively produce better solutions based on the history of feedback. The key idea is that the model can learn to improve itself by observing its own past improvement trajectory.

**Algorithm Distillation (AD) -- Laskin et al., 2023.** AD applies similar principles to reinforcement learning trajectories. The goal is to encapsulate an entire learning algorithm (not just a policy) into a policy conditioned on history. The model learns to improve its performance through successive episodes, effectively "distilling" the learning algorithm itself into the network's weights.

---

## 4. Memory

The memory section is where Weng's interdisciplinary approach manifests itself with the greatest clarity. Weng draws directly on cognitive neuroscience to build a memory framework for AI agents.

### 4.1 Taxonomy of Human Memory

Weng starts from the standard classification of human memory:

**Sensory Memory.** Holds raw sensory impressions (visual, auditory, tactile) for very brief periods -- fractions of a second to a few seconds. In the context of AI agents, it corresponds to raw input: the prompt text, the provided images, the data received from tools. It is the input buffer, the interface between the external world and the agent's cognitive system.

**Short-Term / Working Memory.** In human cognition, it holds about 7 elements (Miller's famous "7 +/- 2") for 20-30 seconds. For AI agents, this corresponds directly to **in-context learning**: the information contained in the LLM's context window. The context window is the agent's working memory -- everything that the model can "keep in mind" simultaneously during generation. It is powerful (the Transformer's attention operates over the entire window), but it is **limited and expensive**: every additional token in the context costs compute quadratically in the standard attention mechanism.

**Long-Term Memory.** In human cognition, it can store information for periods from days to decades, with essentially unlimited capacity. For AI agents, it is implemented through **external vector databases** (vector stores) with rapid retrieval mechanisms. The agent can "deposit" information in long-term memory and "retrieve" it when needed, without having to keep it in the context window.

### 4.2 Types of Long-Term Memory

Weng further distinguishes, following the neuroscience taxonomy:

- **Explicit / Declarative Memory**: facts and events that can be consciously recalled
  - *Episodic Memory*: specific events (e.g. "the last time I executed this task, approach X failed")
  - *Semantic Memory*: general knowledge (e.g. "Python is a programming language")
- **Implicit / Procedural Memory**: automated skills that do not require conscious recall (e.g. patterns learned during model pre-training)

### 4.3 Maximum Inner Product Search (MIPS)

The key mechanism for long-term memory is efficient retrieval. The agent encodes information as high-dimensional vectors (embeddings) and deposits them in a vector database. When it needs an information, it computes the similarity between a query vector and the stored vectors, retrieving the most similar ones.

Weng analyzes the main Approximate Nearest Neighbors search algorithms:

- **LSH (Locality-Sensitive Hashing)**: groups similar vectors into the same "buckets" using hash functions designed to preserve locality. Fast but with limited precision.
- **ANNOY (Approximate Nearest Neighbors Oh Yeah)**: builds random projection trees, recursively partitioning the space. Memory-efficient and fast for static datasets.
- **HNSW (Hierarchical Navigable Small World)**: builds a multi-level graph where each node is connected to the closest neighbors. Efficient navigation from top (long-range connections) to bottom (precise local connections). Excellent balance between speed and precision.
- **FAISS (Facebook AI Similarity Search)**: applies vector quantization with clustering. Designed for datasets of billions of vectors. Supports both CPU and GPU.
- **ScaNN (Scalable Nearest Neighbors)**: uses anisotropic vector quantization optimized specifically for inner product, achieving state-of-the-art performance on several benchmarks.

### 4.4 Architectural Implications

The distinction between short-term memory (in-context) and long-term memory (vector DB) has profound implications for the design of agents. In-context memory is "hot" -- the model has full attention on it -- but limited by the context window. Long-term memory is "cold" -- potentially unlimited, but retrieval is imperfect and loses the nuances that full attention would capture.

This trade-off underlies many of the practical challenges of modern agents and represents an active area of research.

---

## 5. Tool Use

Tool use is the third fundamental component: the agent's ability to extend its native capabilities by interacting with external systems. Weng notes that tool use is a distinguishing feature of human intelligence -- and analogously, it allows AI agents to overcome the intrinsic limits of the LLM (such as the inability to make precise calculations or access updated information).

### 5.1 MRKL -- Modular Reasoning, Knowledge and Language

The MRKL system (Karpas et al., 2022) proposes an architecture where the LLM functions as a **router** to specialized expert modules. These modules can be:

- **Neural**: deep learning models specialized for specific tasks (e.g. image classification, translation)
- **Symbolic**: deterministic tools such as calculators, currency converters, weather APIs, SQL databases

The LLM receives the user's request, determines which expert module is most appropriate, formulates the query in the appropriate format, invokes the module, and integrates the result into the response. The fundamental insight is that the LLM does not need to know how to do everything: it needs to know **when and how to delegate**.

### 5.2 Toolformer and TALM

Toolformer (Schick et al., 2023) and TALM (Tool Augmented Language Models) approach the problem from a different perspective: instead of having an external system that orchestrates tool calls, the model itself is **fine-tuned to learn how to use external APIs**. The Toolformer process is particularly elegant:

1. The training dataset is expanded with API call annotations. For every position in the text where an API call could be useful, an annotation with the call and the result is inserted.
2. The annotations are filtered: only those that actually **improve the prediction quality** (measured by the loss reduction) are kept.
3. The model is fine-tuned on this enriched dataset, learning autonomously when and how to invoke tools.

The result is a model that spontaneously inserts API calls in its output when it recognizes their utility -- for example, invoking a calculator for arithmetic operations or a search engine for factual questions.

### 5.3 HuggingGPT -- Multi-Model Orchestration

HuggingGPT (Shen et al., 2023) takes the tool use concept to a higher level. Instead of using simple tools (calculators, APIs), it uses **other specialized AI models** from the HuggingFace Model Hub as tools. The architecture comprises four phases:

1. **Task Planning**: the LLM (ChatGPT) analyzes the user's request and decomposes it into sub-tasks, determining the dependencies between them.
2. **Model Selection**: for each sub-task, the system selects the most appropriate model from the HuggingFace catalog, based on the model descriptions.
3. **Task Execution**: the specialized models are executed on their respective sub-tasks.
4. **Response Generation**: the LLM integrates all the results into a coherent response for the user.

This is an example of an "LLM as orchestrator" architecture, where the language model does not directly execute tasks but coordinates an ecosystem of specialized models.

### 5.4 ChatGPT Plugins and API-Bank

Weng also discusses ChatGPT Plugins as a practical implementation of the tool use paradigm at industrial scale, and the API-Bank benchmark that evaluates the ability of LLMs to use tools at three levels of increasing complexity:

- **Level 1**: accuracy in calling individual APIs
- **Level 2**: ability to retrieve the right API from a catalog
- **Level 3**: planning sequences of multi-step API calls

---

## 6. Case Studies and Proof-of-Concept

Weng does not limit herself to theory: she analyzes concrete implementations that demonstrate the potential (and limits) of the proposed architecture.

### 6.1 Generative Agents -- Social Simulation

The study by Park et al. (2023) created 25 virtual characters controlled by LLM agents in a sandbox environment similar to The Sims. Each agent combines:

- **Memory Stream**: a chronological record of all experiences
- **Retrieval Model**: retrieves relevant memories based on recency, importance, and relevance
- **Reflection Mechanism**: synthesizes memories into higher-level insights
- **Planning**: generates and updates daily plans

The results showed extraordinary emergent behaviors: spontaneous diffusion of information among agents, maintenance of coherent social relationships, and emergent social coordination (such as the autonomous organization of a Valentine's Day party).

### 6.2 ChemCrow -- Scientific Discovery

ChemCrow augments an LLM with 13 chemistry-specific tools, operating in ReAct format for chemical synthesis and drug discovery tasks. Human evaluations have demonstrated significantly superior performance compared to GPT-4 alone on chemical correctness.

### 6.3 AutoGPT and GPT-Engineer

Weng analyzes AutoGPT as a paradigmatic case study: an agent with about 4000 words of short-term memory, 20 explicit commands (search, file operations, code analysis), and strong dependence on JSON format for reliability. GPT-Engineer takes a different approach, iteratively clarifying requirements with the user before generating code, making architectural assumptions explicit (e.g. Model-View-Controller separation).

---

## 7. Challenges of Autonomous Agents

The challenges section is perhaps the most valuable of the post, because it identifies with lucidity the fundamental problems that, years later, remain largely unresolved.

### 7.1 Finite Context Length

The finite context window represents the agent's communicative bottleneck. Even with the expanded context windows of modern models (128K, 200K, 1M+ tokens), the problem persists in qualitative form: the LLM must operate with a limited "bandwidth" relative to the amount of historical information, instructions, and observations that can accumulate. Vector stores offer a partial solution for long-term memory, but lack the representational power of full attention. Retrieving fragments of information from a vector database is not the same as having that information in the context where the Transformer can reason on it with its attention mechanism.

### 7.2 Difficulty in Long-Term Planning

LLMs struggle to adapt plans when they encounter unforeseen errors. Long-term planning requires the ability to explore a vast solution space, maintain coherence between sub-goals, and recover from failures -- abilities at which humans excel thanks to decades of trial-and-error experience, but which LLMs, trained on next-token prediction, struggle to develop. Weng notes that this limitation is fundamental: it is not just a question of scale (larger models), but potentially a limitation of the training paradigm itself.

### 7.3 Reliability of the Natural Language Interface

The use of natural language as an interface between components introduces structural fragilities. Models produce formatting errors, occasionally refuse instructions, and generate unpredictable outputs. Weng observes that a significant portion of the code of agents like AutoGPT is dedicated to **parsing and validating output** -- an engineering overhead that reflects the intrinsic unreliability of natural language as a communication protocol between software components.

### 7.4 The Meta Challenge: Compounded Robustness

There is an implicit challenge that runs through all the others: in an agent system, errors **compound**. If each step has a 5% error probability, after 20 steps the probability of at least one error exceeds 64%. This compounded fragility is the reason why "pure" autonomous agents (without human intervention) remain problematic for complex long-term tasks.

---

## 8. Impact of the Post -- Why It is Considered the Foundational Document of the Field

### 8.1 The Historical Moment

Weng's post was published on June 23, 2023, at a crucial moment. GPT-4 had been released three months earlier (March 2023), demonstrating capabilities that made autonomous agents plausible. AutoGPT had captured the public's imagination (April 2023). BabyAGI was just born. The field was exploding with fragmented ideas, exciting but disorganized proof-of-concepts, and enormous confusion about what worked, what didn't, and why.

In this context, Weng's post provided exactly what was needed: a **unifying framework**. The formula "Agent = LLM + Memory + Planning + Tool Use" gave a common language to researchers, engineers, and product managers. The Planning, Memory, and Tool Use categories provided a taxonomy for organizing the existing and future literature.

### 8.2 The Lasting Legacy

The post has had an impact that goes well beyond the research community. As observed by several analyses, together with Andrej Karpathy's observations, Weng's work has brought the concept of "autonomous agent" to the center of the AI debate. Practically every paper, blog post, framework, or product that has dealt with AI agents after June 2023 has referenced -- explicitly or implicitly -- Weng's taxonomy.

The post has become:

- **A curriculum**: the standard reading path for those who want to understand AI agents
- **A language**: the four components have become the shared vocabulary of the field
- **A conceptual benchmark**: every new agent system is evaluated with respect to the completeness with which it implements the four components
- **A research roadmap**: the challenges identified by Weng have oriented the research agenda of subsequent years

### 8.3 What Makes It Unique

Several factors distinguish this post from traditional academic literature:

- **Interdisciplinary synthesis**: connects cognitive neuroscience, classical computer science (PDDL planners), reinforcement learning, and NLP into a coherent framework
- **Accessibility**: it is written in technical but understandable language, with clear diagrams and gradual explanations
- **Completeness**: covers the entire field, from theoretical foundations to practical implementations to open limits
- **Intellectual honesty**: does not hide the challenges and limits, dedicating a significant section to unresolved problems
- **Timing**: arrived at the exact moment the field needed it

---

## 9. Resources and References

### The Original Post

- Lilian Weng, "LLM Powered Autonomous Agents", Lil'Log (June 23, 2023): https://lilianweng.github.io/posts/2023-06-23-agent/

### Key Papers Cited in the Post

**Planning and Reasoning:**
- Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022)
- Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (2023)
- Liu et al., "LLM+P: Empowering Large Language Models with Optimal Planning Proficiency" (2023)
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models", ICLR (2023)
- Shinn & Labash, "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023)
- Zhang et al., "Chain of Hindsight Aligns Language Models with Feedback" (2023)
- Laskin et al., "Algorithm Distillation" (2023)

**Tool Use:**
- Karpas et al., "MRKL Systems: A Modular, Neuro-Symbolic Architecture" (2022)
- Schick et al., "Toolformer: Language Models Can Teach Themselves to Use Tools" (2023)
- Shen et al., "HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in Hugging Face" (2023)
- Li et al., "API-Bank: A Comprehensive Benchmark for Tool-Augmented LLMs" (2023)

**Case Studies:**
- Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023)
- Bran et al., "ChemCrow: Augmenting Large Language Models with Chemistry Tools" (2023)
- Boiko et al., "Autonomous chemical research with large language models" (2023)

### Other Relevant Lil'Log Posts

- "Prompt Engineering" (March 2023): https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/
- "Adversarial Attacks on LLMs" (October 2023): https://lilianweng.github.io/posts/2023-10-25-adv-attack-llm/
- "Extrinsic Hallucinations in LLMs" (July 2024): https://lilianweng.github.io/posts/2024-07-07-hallucination/
- "Why We Think" (May 2025): https://lilianweng.github.io/posts/2025-05-01-thinking/

### Profile and Biography

- Fellows Fund, "Lilian Weng, ex-VP of Research, Safety at OpenAI, as New Distinguished Fellow" (December 2024)
- TechCrunch, "OpenAI loses another lead safety researcher, Lilian Weng" (November 2024)
- Google Scholar -- Lilian Weng: https://scholar.google.com/citations?user=dCa-pW8AAAAJ

---

*This document is part of the "Experts" series of the Agent Architect project. Lilian Weng's post represents the conceptual starting point of the entire project: every architecture, framework, and implementation of AI agents can be understood through the lens of the four components -- LLM, Memory, Planning, Tool Use -- that Weng systematized in that fundamental post of June 2023.*
