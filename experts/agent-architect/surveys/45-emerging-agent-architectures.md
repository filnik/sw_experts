# Survey: The Landscape of Emerging AI Agent Architectures

**Paper:** *"The Landscape of Emerging AI Agent Architectures for Reasoning, Planning, and Tool Calling: A Survey"*
**Authors:** Tula Masterman (Neudesic/IBM), Sandi Besen (IBM), Mason Sawtell (Neudesic/IBM), Alex Chao (Microsoft)
**Publication:** arXiv:2404.11584, April 2024
**Link:** [https://arxiv.org/abs/2404.11584](https://arxiv.org/abs/2404.11584)

---

## 1. Overview

This survey represents one of the most significant contributions of 2024 in the field of architectures for Large Language Model (LLM) based agents. The work of Masterman et al. sets out three declared objectives: (a) to communicate the current capabilities and limitations of AI agent implementations, (b) to share observations gathered from analyzing these systems in action, and (c) to suggest important considerations for future developments in AI agent design.

The architectural focus of the survey is particularly relevant: rather than limiting itself to cataloguing individual frameworks, the paper analyzes the **structural patterns** that emerge from the composition of reasoning, planning and tool use modules. The proposed taxonomy first distinguishes between **single-agent** and **multi-agent** architectures, then articulates the latter along a spectrum that goes from **vertical** configurations (with hierarchy and leadership) to **horizontal** configurations (collaborative and peer-based).

A distinctive element of this survey is the attention dedicated to **transversal architectural components** -- memory, tool integration, planning, feedback -- which constitute the fundamental building blocks regardless of the choice between single vs. multi-agent. The central thesis of the paper is that the best architecture varies based on the use case, and that the success of the implementation depends less on the choice of single-agent vs. multi-agent and more on the incorporation of structured phases, clear role definitions, feedback mechanisms and dynamic team management.

The survey defines an AI agent through three minimum necessary components: a **brain** (reasoning capabilities, typically an LLM), a **perception** (understanding of the environment and inputs) and an **action** capability (execution of decisions in the external world). This tripartite definition provides the conceptual framework within which all the architectures analyzed are evaluated.

---

## 2. Single-Agent Architectures

Single-agent architectures represent the historical and conceptual starting point for building LLM-based agents. In these systems, a single language model serves as the decision-making core, possibly enhanced by auxiliary modules for memory, tools and planning. The survey analyzes in detail five main single-agent architectures, four of which are particularly relevant for our deep dive.

### 2.1 ReAct (Reason + Act)

ReAct, introduced by Yao et al. in 2022 and published at ICLR 2023, represents a fundamental turning point in the evolution of AI agents. The key insight is simple but powerful: interleave explicit reasoning with the execution of concrete actions, creating an iterative **Thought-Action-Observation** cycle.

**Operating mechanism.** At each step, the agent does not merely decide what to do (Act), but first explicitly articulates the why (Reason). This explicit thought, often called "thought", serves multiple purposes: it helps decompose complex tasks, track progress, handle exceptions and dynamically adapt the plan based on intermediate results. The typical format is:

```
Thought: [Agent's reasoning]
Action: [Chosen action, e.g., Search[query]]
Observation: [Result of the action from the environment]
```

**Key results from the survey.** On HotpotQA, ReAct shows a hallucination rate of 6%, compared to 14% for pure Chain-of-Thought. This substantial improvement in reliability is directly attributable to the grounding provided by environmental observations: the agent verifies its hypotheses through real interactions instead of proceeding exclusively through internal reasoning.

**Identified limitations.** The survey highlights a critical problem: models can repeatedly generate identical thoughts and actions, entering loops from which they cannot autonomously escape. This limitation is particularly insidious because the framework does not provide a native meta-cognition mechanism -- the agent has no awareness of its own repetitiveness. Human feedback often remains necessary to unlock the agent.

**Architectural importance.** ReAct established the fundamental paradigm on which almost all subsequent architectures are based. The Thought-Action-Observation cycle has become the de facto pattern for the interaction between reasoning and environment.

### 2.2 Reflexion

Reflexion, proposed by Shinn et al. and presented at NeurIPS 2023, introduces the concept of **verbal reinforcement learning** -- reinforcement learning through linguistic feedback rather than through updating the model's weights.

**System architecture.** Reflexion is composed of three main elements:

1. **Actor:** the agent that executes actions in the environment, typically based on ReAct or a similar framework.
2. **Evaluator:** a separate LLM (or the same model with a different prompt) that evaluates the agent's trajectory, producing a feedback signal in the form of natural text.
3. **Self-Reflection Module:** a component that synthesizes the evaluator's feedback and the past trajectory into compact reflections, stored in an episodic memory buffer.

**Learning mechanism.** The process operates over multiple trials: after each attempt, the evaluator produces a judgment on the performance. The self-reflection module generates a textual reflection that is added to long-term memory. In subsequent trials, the agent has access to these past reflections, which serve as accumulated "experience". This mechanism is particularly elegant because it does not require fine-tuning of the model: all learning takes place in the natural language space.

**Memory management.** Reflexion introduces a **sliding window** approach to long-term memory: older reflections are progressively discarded to respect the model's context limits, retaining only the most recent and relevant reflections.

**Critical limitations.** The survey identifies susceptibility to "non-optimal local minima solutions": the agent can converge on a sub-optimal but partially functional strategy, from which subsequent reflections fail to extract it. Furthermore, the quality of the evaluator is crucial -- a mediocre evaluator produces misleading reflections that degrade performance in subsequent trials.

### 2.3 LATS (Language Agent Tree Search)

LATS, published at ICML 2024, represents the most ambitious architectural leap in the single-agent domain, importing tree search techniques from classical reinforcement learning into the context of language agents.

**Inspiration and design.** LATS is inspired by Monte Carlo Tree Search (MCTS), the technique made famous by AlphaGo's success. The insight is that many tasks faced by language agents allow returning to previous steps (backtracking), a property that makes the application of tree search algorithms natural.

**Four-phase architecture.** The LATS framework operates through a cycle of four main operations:

1. **Selection:** Choice of the most promising tree node to expand, guided by the UCT (Upper Confidence bound for Trees) value function.
2. **Expansion:** Generation of new child nodes through sampling of actions from the LLM.
3. **Evaluation:** Evaluation of new nodes through a value function composed of two components -- a self-generated score from the LLM and a self-consistency score.
4. **Backpropagation:** Propagation of values toward the root of the tree to update the estimates.

**Integration with the environment.** A distinctive feature of LATS is the incorporation of external feedback from the environment. When a path in the tree leads to failure, the framework generates a reflection (similar to Reflexion) that is used to guide the future exploration of other branches of the tree.

**Fundamental trade-off.** The survey highlights that LATS achieves performance superior to ReAct and Reflexion, but at the cost of a significant increase in computational resources and execution time. Each tree expansion requires multiple LLM calls, making the framework expensive for real-time applications. Furthermore, validation has been conducted primarily on relatively simple question-answering benchmarks, leaving open the question of scalability to more complex tasks.

### 2.4 Plan-and-Solve

The Plan-and-Solve architecture, originating from the work of Wang et al. (2023) on Plan-and-Solve Prompting and subsequently evolved into more sophisticated architectural patterns such as Plan-and-Execute, introduces a fundamental principle: the **explicit separation between planning and execution**.

**Architectural principle.** Rather than interleaving reasoning and action as in ReAct, Plan-and-Solve proposes a decomposition into two distinct phases:

1. **Planning Phase:** The model first generates a complete plan, decomposing the task into a sequence of ordered sub-tasks.
2. **Execution Phase:** Each sub-task is executed sequentially, potentially by a specialized executor equipped with specific tools.

**Evolution in the agent context.** The survey places this pattern within the broader scope of LLM-based planning, which includes five main approaches: task decomposition, multi-plan selection, planning assisted by external modules, reflection and refinement, and memory-augmented planning.

The AutoGPT+P variant analyzed in the survey combines this planning approach with object detection, Object Affordance Mapping and classical planning based on PDDL (Planning Domain Definition Language). This hybridization between neural and symbolic planning represents a significant architectural direction, although the survey notes that "LLMs are currently unable to directly translate natural language into plans for robotic tasks", highlighting the limits of pure LLM-based planning.

---

## 3. Multi-Agent Architectures

Multi-agent architectures represent the natural evolution of single-agent systems, motivated by the observation that many complex tasks benefit from collaboration among specialized agents. The survey classifies these architectures along a spectrum that goes from vertical (hierarchical) to horizontal (peer-based) configurations.

### 3.1 Debate and Discussion

The paradigm of debate and discussion among agents represents the most direct form of multi-agent collaboration. In this model, two or more agents express positions or reasonings on a problem, mutually criticize the answers and converge towards a shared solution.

**Mechanism.** Agents participate in discussion rounds in which each proposes a solution, reads the proposals of the others and refines its own response. The iterative process continues until reaching a consensus or a maximum number of rounds.

**Observations from the survey.** An important result is that "multi-agent discussion does not necessarily improve reasoning when the prompt provided is sufficiently robust". This suggests that the value of debate emerges primarily in contexts of high ambiguity or complexity, where multiple perspectives can identify errors that a single agent would not detect.

**Sycophancy risk.** The survey identifies a critical problem: agents tend to conform to the feedback received, even when it is incorrect. In a debate context, an agent with a correct answer can be "convinced" by a wrong argument from another agent. This sycophantic behavior represents a fundamental challenge for discussion-based architectures.

### 3.2 Collaborative Problem-Solving

Collaborative problem-solving architectures structure cooperation among agents in a more formal way compared to simple debate. The survey analyzes three representative frameworks in detail.

**Embodied LLM Agents in Organized Teams.** This work studies the impact of organizational structure on agent performance. The results show that teams with a designated leader complete tasks about 10% faster than teams without leadership. The leader's communication is composed of 60% directives and 40% information exchange, while in teams without a leader 50% of the communication consists of attempts to give mutual orders. The optimal configuration provides for dynamic structures with leadership rotation.

**DyLAN (Dynamic LLM-Agent Network).** DyLAN implements a dynamic horizontal architecture in which agents are ranked based on their contribution at each round. Only the agents with the best performance advance to the next round, creating a natural selection mechanism that optimizes the team composition adaptively. This approach is particularly effective for complex reasoning tasks and code generation.

**AgentVerse.** AgentVerse proposes a four-phase execution model: (1) **Recruitment** -- dynamic recruitment of agents based on the task, (2) **Collaborative Decision Making** -- collective decision on the strategy, (3) **Independent Action Execution** -- independent execution of sub-actions, (4) **Evaluation** -- evaluation of results and possible re-recruitment. The key innovation is the recruitment phase: agents are added or removed dynamically based on the progress of the task, avoiding both under-staffing and over-staffing.

### 3.3 Role-Playing Agents

Role-playing-based architectures assign specific identities and competencies to each agent, replicating the structure of human professional teams.

**MetaGPT** is the most sophisticated framework in this category, analyzed in depth by the survey. MetaGPT models the software development process as a virtual company, with agents holding specific roles: Product Manager, Architect, Project Manager, Engineer. Each agent operates according to Standard Operating Procedures (SOPs) encoded in the prompts.

**Innovation in communication.** The main problem identified by the survey in multi-agent systems is "superfluous chatter" -- non-productive communication that consumes tokens and introduces noise. MetaGPT solves this problem through two mechanisms:

1. **Structured communication:** Agents produce structured outputs (documents, diagrams, specifications) instead of unstructured natural language messages. This forces each agent to produce concrete and verifiable artifacts.
2. **Publish-subscribe mechanism:** Agents access only the information relevant to their role, avoiding the information overload typical of horizontal architectures where everyone sees all messages.

**Performance.** MetaGPT significantly outperforms single agents on the HumanEval and MBPP benchmarks, demonstrating that the structuring of roles and communication can effectively translate into measurable improvements in the quality of generated code.

**Impact of persona.** The survey reports that "the configured personality verifiably influences the behavior of Large Language Models". In a multi-agent context, the precise definition of roles prevents hallucination about one's own capabilities -- an agent with the role of "tester" will not attempt to write production code, focusing instead on its own area of competence.

---

## 4. Architectural Components

Regardless of the choice between single-agent and multi-agent, the survey identifies fundamental architectural components that determine the effectiveness of any agent system.

### 4.1 Memory Systems

Memory represents perhaps the most critical and least solved architectural component. The survey distinguishes two main categories.

**Short-term Memory.** Typically implemented as a scratchpad or context buffer, short-term memory holds information relevant to the current task. In ReAct, it corresponds to the history of Thought-Action-Observation accumulated during execution. In Reflexion, it also includes the reflections of the current trial. The main limit is the LLM's context window: when the history exceeds the token limit, older information is truncated or summarized, with potential loss of critical context.

**Long-term Memory.** Implemented through external databases, vector embeddings or persistent sliding windows, long-term memory allows the agent to accumulate experience across different sessions. RAISE introduces a dual-memory system with scratchpad (short-term) and example dataset (long-term). Reflexion uses an episodic memory buffer for accumulated reflections. The balance between retention and token limits remains an open challenge.

**Architectural challenges of memory.** The survey highlights that memory management is intrinsically linked to the scalability of the system: an agent operating for prolonged periods inevitably accumulates more context than can be contained in the model's window. Compression strategies (summarization), selection (retrieval-based) and prioritization (recency/relevance weighting) represent active research areas.

### 4.2 Tool Integration

Tools are defined by the survey as any callable function that allows the agent to interact with external data or systems. Tool integration transforms an LLM from a purely linguistic system into an operational system capable of modifying its environment.

**Categorization of tools.** The survey identifies different functional categories: search tools (web APIs, databases), manipulation tools (document editing, file system), communication tools (email, messaging), computation tools (code interpreters, calculators) and perception tools (image analysis, OCR).

**Integration challenges.** The analysis of AutoGPT+P reveals that inconsistent tool selection remains a significant problem: the agent might choose an unsuitable tool for the current task, or attempt to use a tool in an unsupported way. The precise definition of tool interfaces (names, parameters, descriptions) is fundamental to guiding the agent's choice. In multi-agent contexts, the division of tools among specialized agents (each agent accesses only the tools pertinent to its role) reduces the risk of incorrect selection.

### 4.3 Planning Modules

The survey identifies five main approaches to planning, which can be combined in hybrid architectures.

1. **Task Decomposition:** Decomposition of the main task into more manageable sub-tasks. It is the most widespread and intuitive approach, present in implicit form in ReAct and in explicit form in Plan-and-Solve.

2. **Multi-Plan Selection:** Generation of alternative plans and selection of the most promising one. LATS implements this strategy through tree exploration, where each branch represents an alternative plan.

3. **External Module-Aided Planning:** Use of external planners (e.g., PDDL) to complement the LLM's planning capabilities. AutoGPT+P exemplifies this approach, combining neural and symbolic planning.

4. **Reflection and Refinement:** Iterative revision of the plan based on intermediate results. Reflexion embodies this principle through the cycle of self-reflection between successive trials.

5. **Memory-Augmented Planning:** Use of information retrieved from long-term memory to inform planning. This approach is particularly powerful when the agent has accumulated experience on similar tasks.

### 4.4 Feedback Loops

Feedback mechanisms represent the component that distinguishes an agent from a simple sequential generation system. The survey categorizes three types of feedback.

**Human feedback.** Human involvement in the agent's loop improves reliability and trust in the system. The human can intervene to correct errors, provide missing information or redirect the agent when it deviates from the task. However, the need for human feedback limits the scalability and autonomy of the system.

**Inter-agent feedback.** In multi-agent systems, agents can correct each other. This mechanism is the basis of debate architectures and the evaluation phase in AgentVerse. The main risk is susceptibility to incorrect feedback: an agent that receives wrong feedback from another agent will tend to conform, degrading the overall quality.

**Self-reflection.** The agent evaluates its own outputs against defined success criteria. Reflexion and LATS implement sophisticated forms of self-reflection. The quality of self-evaluation depends critically on the model's ability to accurately judge its own performance -- a capability that is not guaranteed, especially for tasks at the edges of the model's competencies.

---

## 5. Architectural Generations

The survey allows tracing a clear evolutionary line in agent architectures, proceeding from simple configurations to increasingly sophisticated systems.

**Generation 1: Pure reasoning (Chain-of-Thought).** The starting point is step-by-step prompting without interaction with the environment. High hallucination rate (~14% on HotpotQA) because reasoning is never verified against reality.

**Generation 2: Reasoning + Action (ReAct).** The integration of environmental actions into the reasoning cycle reduces hallucinations to 6%. However, the agent can enter repetitive loops and has no learning mechanisms across sessions.

**Generation 3: Memory enhancement (RAISE).** The addition of dual-memory systems (short and long term) improves context retention and output quality. The new problem of role hallucination emerges: the agent can "imagine" having capabilities it does not possess.

**Generation 4: Self-reflection (Reflexion).** The introduction of an evaluator LLM and a reflective memory buffer allows the agent to learn from its own errors through linguistic feedback. The limit is convergence on sub-optimal local minima.

**Generation 5: Advanced planning (AutoGPT+P).** The integration of classical planning (PDDL) with the LLM's capabilities attempts to bridge the gap between linguistic reasoning and formal planning. The lesson learned is that purely LLM-based planning is not sufficient for complex tasks.

**Generation 6: Optimized search (LATS).** The application of Monte Carlo Tree Search to the domain of language agents creates a powerful framework that unifies reasoning, action and planning. The computational cost remains the main obstacle to practical adoption.

**Generation 7: Multi-agent collaboration.** The transition from single to multiple introduces the social dimension: leadership, communication, division of labor. Teams with designated leaders show about 10% improvement in completion speed.

**Generation 8: Dynamic teams (DyLAN, AgentVerse).** Team composition becomes adaptive: agents are recruited, evaluated and potentially replaced based on their actual contribution. This approach reduces communication cost and eliminates non-productive agents.

**Generation 9: Structured communication (MetaGPT).** The elimination of unstructured dialogue in favor of formal artifacts and publish-subscribe mechanisms represents the most advanced level analyzed by the survey, with significant improvements on benchmarks.

---

## 6. Architecture Comparison

### Comparison table of single-agent architectures

| Architecture | Paradigm | Hallucination | Memory | Strengths | Main limitations |
|---|---|---|---|---|---|
| **ReAct** | Thought-Action-Observation loop | ~6% (HotpotQA) | Short-term only (context) | Reasoning transparency; environmental grounding | Repetitive loops; no cross-session learning |
| **RAISE** | ReAct + dual memory | Lower than ReAct | Short (scratchpad) + long term (examples) | Improved context retention | Difficulty with complex logic; role hallucination |
| **Reflexion** | Trial + self-reflection | Reduced via feedback | Episodic (sliding window of reflections) | Linguistic learning without fine-tuning | Convergence to local minima; dependence on evaluator |
| **AutoGPT+P** | Classical planning + LLM | Variable | Based on PDDL domain | Environmental awareness; formal planning | Inconsistent tool selection; limited NL-to-plan translation |
| **LATS** | Monte Carlo Tree Search + LLM | Not specified | State tree + reflections | Superior performance; systematic exploration | High computational cost; uncertain scalability |

### Comparison table of multi-agent architectures

| Architecture | Structure | Communication | Team management | Key result |
|---|---|---|---|---|
| **Embodied Teams** | Vertical (with leader) | Directives + info sharing | Static with leadership rotation | +10% speed with designated leader |
| **DyLAN** | Dynamic horizontal | Ranking by contribution | Agents advance/retreat by performance | Optimal for complex reasoning |
| **AgentVerse** | Hybrid (4 phases) | Structured by phase | Dynamic recruitment/removal | Flexibility and adaptability |
| **MetaGPT** | Vertical (SOP) | Structured artifacts + pub/sub | Fixed roles with SOPs | SOTA on HumanEval and MBPP |

### Selection criteria: when to use single vs. multi-agent

**Single-agent is preferable when:**
- The task is well defined with tools of restricted scope
- The process has clear and sequential procedural steps
- Feedback from other agents is not necessary
- Reduced implementation complexity and contained costs are required

**Multi-agent is preferable when:**
- The task requires feedback from multiple perspectives
- Parallel execution of sub-tasks is necessary
- No examples are available (few-shot) -- multi-agent systems perform better in zero-shot regime
- Document generation requires collaborative review
- The problem requires specialized agent roles

---

## 7. Implications for Practice

### 7.1 Recommended design principles

Based on the comparative analysis, the survey extracts concrete architectural principles for professionals implementing agent systems.

**Clear definition of persona.** Each agent must have a system prompt that explicitly establishes identity, competencies and limits. In single-agent systems, this prevents hallucination about capabilities; in multi-agent systems, it ensures responsibility on the task and enables dynamic team composition.

**Structured phases of reasoning-execution-evaluation.** Regardless of the chosen architecture, the implementation of distinct phases for planning, execution and evaluation improves overall quality. This structure is explicit in AgentVerse (four phases) and implicit in Reflexion (trial + reflection).

**Appropriate feedback mechanisms.** The choice between human feedback, inter-agent feedback and self-reflection must be guided by the risk level of the task. For critical applications, human feedback remains essential; for low-risk tasks, self-reflection may be sufficient.

**Intelligent message filtering.** In horizontal architectures, where all agents have access to all messages, it is fundamental to implement filtering mechanisms to avoid information overload and non-productive "chatter". MetaGPT's publish-subscribe mechanism represents an exemplary model.

### 7.2 Critical challenges for adoption

**Evaluation and benchmarks.** The survey identifies the absence of standardized benchmarks as the main obstacle to objective comparison of architectures. Teams create custom evaluations, often subject to bias. Existing LLM benchmarks (MMLU, GSM8K) are designed for single iterations and represent inadequate approximations for multi-step reasoning and tool-calling.

**Data contamination.** The overlap between training data and benchmarks inflates scores. Models show significant performance drops when questions are reformulated, suggesting memorization rather than genuine understanding. Dynamic benchmarks resistant to contamination are needed.

**Security and bias.** The survey warns that agents are "less robust, prone to more harmful behaviors and capable of generating more subtle content compared to LLMs alone". The tendency to conform to intrinsic social biases persists even with contrary instructions. Bias evaluation requires human involvement, creating a scalability problem.

**Robustness in real domains.** Benchmarks based on logic puzzles do not reflect real user queries. Tools such as WildBench (570K real conversations) and SWE-bench (for software development) represent steps in the right direction, but coverage remains insufficient.

### 7.3 Future directions

The survey outlines several research priorities for the field:

1. **Dynamic benchmarks.** Development of evaluation frameworks resistant to contamination and memorization, that evolve with model capabilities.
2. **Human-AI collaboration.** Integration of human supervision for reliability and trust, with intelligent escalation mechanisms.
3. **Bias mitigation.** Scalable but human-informed frameworks for bias detection and correction.
4. **Communication efficiency.** Reduction of superfluous dialogue in multi-agent systems through structured protocols.
5. **Robustness testing.** Systematic evaluation on new and unseen task types to test generalization.
6. **Domain-specific evaluation.** Specialized benchmarks for high-stakes sectors (medicine, legal, finance) where errors have real consequences.

---

## 8. Resources and References

### Main paper
- Masterman, T., Besen, S., Sawtell, M., & Chao, A. (2024). *The Landscape of Emerging AI Agent Architectures for Reasoning, Planning, and Tool Calling: A Survey*. arXiv:2404.11584. [https://arxiv.org/abs/2404.11584](https://arxiv.org/abs/2404.11584)

### Single-Agent Architectures
- Yao, S. et al. (2023). *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023. [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
- Shinn, N. et al. (2023). *Reflexion: Language Agents with Verbal Reinforcement Learning*. NeurIPS 2023. [https://arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366)
- Zhou, A. et al. (2024). *Language Agent Tree Search Unifies Reasoning, Acting, and Planning in Language Models*. ICML 2024. [https://arxiv.org/abs/2310.04406](https://arxiv.org/abs/2310.04406)
- Wang, L. et al. (2023). *Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models*. ACL 2023. [https://arxiv.org/abs/2305.04091](https://arxiv.org/abs/2305.04091)

### Multi-Agent Architectures
- Hong, S. et al. (2023). *MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework*. ICLR 2024. [https://arxiv.org/abs/2308.00352](https://arxiv.org/abs/2308.00352)
- Liu, Z. et al. (2023). *Dynamic LLM-Agent Network: An LLM-agent Collaboration Framework with Agent Team Optimization*. arXiv:2310.02170.
- Chen, W. et al. (2023). *AgentVerse: Facilitating Multi-Agent Collaboration and Exploring Emergent Behaviors*. ICLR 2024.

### Supplementary resources
- Weng, L. (2023). *LLM Powered Autonomous Agents*. Lil'Log. [https://lilianweng.github.io/posts/2023-06-23-agent/](https://lilianweng.github.io/posts/2023-06-23-agent/)
- Wang, L. et al. (2023). *A Survey on Large Language Model based Autonomous Agents*. arXiv:2308.11432.
- ReAct GitHub Repository: [https://github.com/ysymyth/ReAct](https://github.com/ysymyth/ReAct)
- Reflexion GitHub Repository: [https://github.com/noahshinn/reflexion](https://github.com/noahshinn/reflexion)
- LATS GitHub Repository: [https://github.com/lapisrocks/LanguageAgentTreeSearch](https://github.com/lapisrocks/LanguageAgentTreeSearch)
- MetaGPT GitHub Repository: [https://github.com/FoundationAgents/MetaGPT](https://github.com/FoundationAgents/MetaGPT)

---

*Document generated on March 26, 2026. Based on the survey arXiv:2404.11584v1 and related literature.*
