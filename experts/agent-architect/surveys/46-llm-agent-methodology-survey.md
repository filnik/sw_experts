# LLM Agent: A Survey on Methodology, Applications and Challenges

> **Paper:** Luo, J., Zhang, W., Yuan, Y., Zhao, Y. et al. (2025). *Large Language Model Agent: A Survey on Methodology, Applications and Challenges.* arXiv:2503.21460
> **Repository:** [Awesome-Agent-Papers](https://github.com/luo-junyu/Awesome-Agent-Papers)
> **Number of papers analyzed:** 329
> **Publication date:** March 2025

---

## 1. Overview

The survey "Large Language Model Agent: A Survey on Methodology, Applications and Challenges", published in March 2025 by a team of 26 researchers from international academic and industrial institutions, represents one of the most comprehensive and systematic contributions in the field of Large Language Model (LLM) based agents. With 329 papers analyzed and catalogued, the work stands out for adopting a **methodology-centric taxonomy**, surpassing previous classifications that focused mainly on architectures or applications.

The fundamental insight of the survey is that understanding LLM agents requires a three-dimensional framework that connects **architectural foundations**, **collaboration mechanisms** and **evolutionary paths**. This approach allows identifying fundamental connections between design principles and the emergent behaviors that characterize modern agentic systems.

The authors structure the LLM agent ecosystem along four interconnected dimensions:

- **Agent Methodology** -- which encompasses construction, collaboration and evolution of agents
- **Evaluation and Tools** -- with benchmarks, evaluation frameworks and development tools
- **Real-World Issues** -- which addresses security, privacy and social impact
- **Applications** -- with the application domains in which agents are deployed

The relevance of this survey in the 2025 landscape is particularly significant: the field of LLM agents is undergoing a critical transition from research prototypes to enterprise production systems, and the need for a unified taxonomy becomes essential not only for researchers, but also for industry professionals, policymakers and society as a whole.

---

## 2. Methodological Taxonomy

The taxonomy proposed by the survey organizes the methodologies for building and optimizing LLM agents along three main axes: prompt-based methods, fine-tuning-based methods and hybrid approaches. This categorization reflects the complete spectrum of available techniques, from zero-shot prompting to specialized training with reinforcement learning.

### 2.1 Prompt-based Methods

Prompt-based methods represent the **parameter-free** approach to building agents, where the agent's behavior is guided through the careful design of textual inputs without modifying the weights of the underlying model.

**Chain-of-Thought (CoT) and variants.** CoT prompting constitutes the basis of structured reasoning of agents. The survey identifies two main paradigms: **single-path chaining**, where reasoning proceeds linearly through sequential steps, and **multi-path tree expansion**, exemplified by Tree-of-Thoughts (ToT), which allows the parallel exploration of multiple chains of reasoning with possibility of backtracking. As the authors point out, "solving an entire problem can be challenging for LLM agents, but they can more easily handle sub-tasks".

**ReAct and reasoning-action paradigms.** The ReAct (Reasoning + Acting) framework integrates explicit thought with the execution of actions, creating a reflection cycle that improves the quality of decisions. This approach is fundamental for agents that must interact with external environments.

**Meta-Prompting and role-playing.** Advanced techniques such as Meta-Prompting allow a single model to assume multiple sub-roles (for example, plan-agent, tool-agent, reflect-agent in the AutoAct framework), creating a virtual multi-agent system within a single instance of the model.

**Retrieval-Augmented Generation (RAG).** The survey classifies RAG as a form of "knowledge retrieval as memory", where systems such as GraphRAG and Chain of Agents integrate external knowledge repositories with dynamic reasoning, effectively extending the agent's memory beyond the limits of the context window.

### 2.2 Fine-tuning-based Methods

Fine-tuning-based methods directly modify the model's parameters to optimize the agent's behavior on specific tasks. The survey identifies several fundamental sub-categories.

**Supervised Fine-Tuning (SFT) for agents.** Supervised training on datasets of agentic trajectories (sequences of observations, reasonings and actions) allows the model to internalize specific behavioral patterns. Frameworks such as DiverseEvol improve the quality of instruction tuning through the generation of diversified instructions.

**Parameter-Efficient Fine-Tuning (PEFT).** Techniques such as LoRA (Low-Rank Adaptation), QLoRA and DoRA offer computationally efficient alternatives to complete fine-tuning. LoRA adds small adapter layers that capture task-specific knowledge while keeping the base model frozen. The hybrid LoRA + Quantization approach increases training complexity but significantly reduces memory requirements at deployment.

**Reinforcement Learning (RL) for agents.** The survey devotes ample space to RL techniques for agent optimization. The Self-Rewarding paradigm allows the model to act as a judge of itself (LLM-as-Judge) during training, generating internal reward signals. RLCD (Reinforcement Learning from Contrastive Distillation) uses contrastive distillation to refine agent behavior without requiring explicit human feedback.

**Bootstrapping and self-improvement.** Approaches such as STaR (Self-Taught Reasoner) and V-STaR (Verified STaR) implement self-improvement cycles where the model generates its own training trajectories, verifies them and uses the correct ones for further fine-tuning. This creates a virtuous cycle of autonomous improvement.

### 2.3 Hybrid Methods

Hybrid approaches combine prompt engineering and fine-tuning to obtain the best of both paradigms, offering rapid iteration cycles and reduced computational costs while achieving significant performance improvements.

**Toolformer and tool integration.** Toolformer represents a paradigmatic example: the model is fine-tuned to learn when and how to invoke external APIs, but the specific tool usage behavior is guided by contextual prompting. This approach decouples the generic tool-use capability from the specific knowledge of each tool.

**CRITIC and tool-assisted self-correction.** The CRITIC framework combines interaction with external tools for verification with self-correction mechanisms based on prompts, creating an improvement loop that leverages both the model's internal capabilities and external resources.

**Knowledge-enhanced agents.** Systems such as KnowAgent and WKM (World Knowledge Model) synthesize external knowledge into the agent's decision-making process through a combination of fine-tuning on the structure of knowledge and prompting for its contextual application.

---

## 3. Agent Frameworks

### 3.1 Single-Agent Frameworks

Single-agent frameworks define the operational architecture of a single LLM agent. The survey identifies four fundamental components in building an agent.

**Profile.** The profile establishes the operational identity of the agent. The survey distinguishes between:
- **Manually curated static profiles**: fixed identities specified by humans, used in systems such as CAMEL, AutoGen, MetaGPT and ChatDev. These ensure "strict adherence to predefined behavioral guidelines" and standardized communication protocols.
- **Dynamically batch-generated profiles**: parameterized initialization that produces heterogeneous populations of agents. Systems such as Generative Agents and RecAgent create "controlled variations in personality traits, knowledge backgrounds or value systems".

**Memory.** The memory architecture is articulated on three levels:
- **Short-term memory**: transient contextual data (dialogues, immediate observations) that support current task execution. Frameworks such as ReAct and Graph of Thoughts operate primarily on this level. The main challenge is the limitation of the context window, which requires "active information compression".
- **Long-term memory**: the survey identifies three paradigms -- **skill libraries** (such as Voyager, which accumulates reusable skills), **experience repositories** (such as ExpeL and Reflexion, which transform past experiences into operational knowledge) and **tool synthesis frameworks** (such as TPTU and MemGPT, which implement memory management at hierarchical levels). This component transforms "ephemeral cognitive efforts into persistent operational assets".
- **Knowledge Retrieval as memory**: RAG, GraphRAG and Chain of Agents systems that integrate external knowledge repositories.

**Planning.** Two main approaches:
- **Task decomposition**: subdivision of complex problems into manageable sub-tasks, with single-path approaches (sequential chain) and multi-path (tree exploration with backtracking).
- **Feedback-driven iteration**: four sources of feedback are identified -- environmental, human, model introspection and multi-agent collaboration. Dynamic planning "generates only the next sub-task based on the current situation", adapting in real time.

**Action.** The execution of actions includes:
- **Tool use**: the decision whether to use a tool and the selection of the optimal tool, requiring "understanding of tools and the agent's current situation".
- **Physical interaction**: for embodied agents that require "robotic hardware, social knowledge and interaction with other LLM agents".

Reference single-agent frameworks include: **ReAct** (reasoning with reflection), **Voyager** (skill discovery in open environments), **Reflexion** (optimization through successive trials), **MemGPT** (hierarchical memory at levels), **CoALA** (cognitive architecture with modular memory and decision-making) and **CodeAct** (which uses executable Python code as a unified action space).

### 3.2 Multi-Agent Frameworks

The survey proposes a tripartite taxonomy for multi-agent architectures, based on the degree of centralization of control.

**Centralized control.** A dedicated coordination module orchestrates the activities of agents:
- **Explicit controllers**: systems such as Coscientist, LLM-Blender and MetaGPT implement a "hierarchical coordination mechanism where a central controller organizes agent activities". MetaGPT, in particular, integrates human workflows (such as software engineering practices) into the management of specialized technical roles.
- **Differentiation-based systems**: a single model assumes multiple sub-roles. AutoAct implements plan-agent, tool-agent and reflect-agent as distinct roles within the same instance, while Meta-Prompting orchestrates specialized roles through hierarchical prompts.

**Decentralized collaboration.** Agents interact directly without a central coordinator:
- **Review-based**: sequential modification among peers. MedAgents implements voting mechanisms, ReConcile iterative refinement, and METAL domain-specific revision.
- **Communication-based**: direct dialogues among agents. The MAD (Multi-Agent Debate) framework implements anti-degeneration protocols to prevent opinion collapse, MADR produces verifiable critiques, MDebate builds consensus through debate, and AutoGen implements a "group-chat framework that supports multi-agent participation in iterative debates".

**Hybrid architecture.** Combines centralized and decentralized elements:
- **Static systems**: predefined coordination patterns. CAMEL implements group role-play, AFlow uses a "three-level hierarchy composed of centralized strategic planning, decentralized tactical negotiation and market-driven operational resource allocation", and EoT (Exchange of Thought) formalizes communication patterns.
- **Dynamic systems**: neural topological optimizers. DiscoGraph implements position-aware collaboration, DyLAN (Dynamic LLM-Agent Network) an importance-aware topology, and MDAgents complexity-aware routing that dynamically adapts the collaborative structure to task difficulty.

Notable multi-agent frameworks also include: **AgentVerse** (inspired by group dynamics), **ChatDev** (conversational agents for software development) and **AutoAgents** (automatic agent generation for tasks).

### 3.3 Evaluation Frameworks

The survey devotes ample space to evaluation frameworks, recognizing that the lack of standardized metrics represents one of the most pressing challenges in the field.

**General evaluation frameworks:**
- **AgentBench**: eight interactive environments for multi-dimensional evaluation of agentic capabilities.
- **MMAU**: over 3,000 cross-domain tasks that map five fundamental competencies.
- **VisualAgentBench**: extends evaluation to multimodal interactions.
- **CRAB**: cross-platform evaluation based on graphical interfaces.

**Dynamic and self-evolutive evaluation:**
- **BENCHAGENTS**: automatically creates benchmarks through agents, eliminating human bias in test design.
- **Self-evolving benchmark**: implements six refactoring operations to keep benchmarks updated and resistant to contamination.

**Multi-agent evaluation:**
- **MLRB**: seven competition-level ML research tasks.
- **MultiAgentBench**: evaluates collaboration and competition among agents.
- Specific comparative frameworks for AutoGen and CrewAI.

---

## 4. Key Challenges

### 4.1 Hallucination in Agents

The problem of hallucination takes on a particularly critical dimension in agentic systems, where errors do not remain confined to single responses but propagate through chains of reasoning and action. The survey identifies the phenomenon of **error accumulation during chaining**: when an agent chains multiple reasoning steps or actions, a hallucination in an early step can propagate and amplify in subsequent steps, leading to catastrophically wrong results.

The problem is further aggravated in multi-agent systems, where a hallucination produced by one agent can be uncritically accepted by other agents in the system, creating an "echo chamber" effect that reinforces false information. Identified mitigation techniques include: self-reflection and self-correction (SELF-REFINE with iterative feedback), verification through external tools (CRITIC), and multi-agent consensus mechanisms where divergence among agent responses serves as a signal of potential hallucination.

### 4.2 Long-Horizon Planning

Long-horizon planning represents one of the most fundamental challenges for LLM agents. The survey distinguishes between **static planning** and **dynamic planning**, highlighting the limits of both approaches.

In static planning, "the agent is required to follow the predefined plan without any deviation", which creates significant **flexibility constraints** when environmental conditions change. Dynamic planning, on the other hand, requires "environmental feedback and adjustment", introducing computational complexity and the risk of decisional instability.

Specific challenges include: the decomposition of abstract goals into concrete and ordered sub-tasks, the maintenance of coherence among sub-tasks in long sequences, the balance between exploration of new strategies and exploitation of known strategies (exploration-exploitation tradeoff), and the management of temporal dependencies among actions that may have delayed effects.

### 4.3 Tool Use Reliability

Reliability in tool use is articulated by the survey in two critical dimensions: the **tool-use decision** and **tool selection**.

The use decision requires the agent to correctly evaluate when a problem requires an external tool versus when it can be solved with the model's capabilities alone. This evaluation requires "understanding of tools and the agent's current situation". Errors in this phase lead both to under-use (the model attempts to solve problems that require exact computation without using tools) and over-use (unnecessary invocations that increase latency and costs).

The selection of the correct tool, among a potentially vast inventory of available tools, requires understanding both the tool's capabilities and the context of the problem. The Seal-Tools benchmark, with 1,024 instances of nested tool-calls, highlights the complexity of scenarios where tools must be composed in chains.

### 4.4 Safety and Alignment

The survey systematically catalogs threats to the security of LLM agents in different categories:

**Adversarial attacks**: CheatAgent and GIGA demonstrate how malicious agents can manipulate multi-agent systems through strategic injections of false information.

**Jailbreaking**: RLTA (Reinforcement Learning for Token-level Attack), Atlas and RLbreaker represent increasingly sophisticated techniques to bypass model security protections. The agentic context amplifies the risk because a successful jailbreak can give access to external tools and resources.

**Backdoor attacks**: DemonAgent, BadAgent and DarkMind implement latent malicious behaviors that activate only in the presence of specific triggers, making detection extremely difficult.

**Privacy**: memorization vulnerabilities, intellectual property exploitation, and theft of models and prompts represent concrete risks, especially in enterprise contexts where agents interact with sensitive data.

**Ethics and bias**: the propagation of bias and misinformation through agentic chains, combined with accountability gaps when decisions emerge from complex multi-agent interactions, poses significant regulatory challenges.

---

## 5. Applications by Domain

The survey maps a vast and rapidly expanding application landscape, covering both traditional domains and emerging areas.

### 5.1 Scientific Discovery

- **Chemistry and materials science**: Coscientist represents an autonomous agent capable of designing and conducting chemical experiments. The integration of agents with molecular simulation tools opens new possibilities for materials discovery.
- **Biology and genomics**: specialized agents for the analysis of genomic sequences, the prediction of protein structures and the design of biological experiments.
- **Astronomy**: emerging applications in the analysis of astronomical data and observational planning.
- **Medicine**: MedAgentBench evaluates agents in an environment compliant with FHIR (Fast Healthcare Interoperability Resources), with tasks "designed by 300 clinicians". AI Hospital implements multi-agent clinical simulations.

### 5.2 Gaming and Simulation

- **Open environments**: Voyager in Minecraft demonstrates skill discovery and continuous learning in vast environments. Applications in GTA and multi-agent scenarios explore decision-making in complex contexts.
- **Social simulation**: Generative Agents creates simulations of human communities with agents endowed with personality, memory and social relationships, allowing the emergent study of social dynamics.

### 5.3 Social and Behavioral Sciences

RecAgent and similar systems model human behavior in social contexts, with applications in behavioral economics, political science and computational sociology. Multi-agent simulations allow testing social hypotheses in controlled environments.

### 5.4 Productivity and Automation

- **Code generation and software development**: ChatDev implements an entire software development cycle with specialized agents (CEO, CTO, programmer, tester).
- **Web automation**: Mind2Web covers 137 websites in 31 domains, evaluating the ability of agents to navigate and complete real tasks on the web.
- **Research and synthesis**: DeepResearch and DeepSearch represent agents capable of conducting autonomous bibliographic research and synthesizing results.

### 5.5 Specialized Domains

- **Autonomous driving**: LaMPilot integrates LLM agents into autonomous driving planning.
- **Data Science**: DSEval, DA-Code and DCA-Bench evaluate agents in data analysis and the construction of ML pipelines.
- **Machine Learning**: MLAgentBench and MLE-Bench test the ability of agents to design and conduct ML experiments.
- **Travel planning**: TravelPlanner evaluates the capacity for complex multi-constraint planning.
- **Cybersecurity**: AgentHarm models threats and evaluates the resilience of agentic systems.
- **Enterprise environment**: TheAgentCompany simulates complete corporate environments to evaluate agents in realistic work contexts. OSWorld represents the "first scalable ecosystem based on real computers, supporting 369 multi-application tasks".

---

## 6. Evaluation Metrics

The evaluation of LLM agents represents a significant methodological challenge, given the complex, multi-step and often non-deterministic nature of agentic behavior. The survey identifies several evaluation dimensions.

### 6.1 Task Completion Metrics

The most basic metric is the **success rate** in completing the assigned task. However, the survey emphasizes that this metric alone is insufficient: two agents with the same success rate can differ drastically in efficiency, process quality and robustness.

### 6.2 Process Metrics

- **Reasoning efficiency**: number of steps necessary to reach the solution.
- **Decomposition quality**: evaluation of the relevance and completeness of the generated sub-tasks.
- **Tool usage**: precision in tool selection and invocation, with the Seal-Tools benchmark providing 1,024 test instances for nested tool-calling evaluations.
- **Robustness**: consistency of performance across variations in the prompt or environment.

### 6.3 Multi-Agent Evaluation

Evaluation of multi-agent systems introduces additional dimensions: quality of inter-agent communication, coordination efficiency, ability to resolve conflicts and emergent synergy. MLRB proposes seven competition-level ML research tasks to evaluate these dimensions.

### 6.4 Domain-Specific Evaluation

- **CToolEval**: 398 Chinese APIs in 14 domains to evaluate tool use in specific linguistic contexts.
- **EgoLife**: 300-hour egocentric multimodal dataset capturing daily human activities, to evaluate embodied agents.
- **MedAgentBench**: FHIR-compliant environment designed with the contribution of 300 clinicians to evaluate medical agents.

### 6.5 Dynamic and Self-Evolutive Evaluation

An innovative contribution of the survey is the emphasis on **dynamic evaluation**. BENCHAGENTS eliminates human bias in benchmark design by automatically creating tests through agents. The self-evolving benchmark framework implements six refactoring operations to keep benchmarks updated, resistant to training data contamination and capable of capturing emergent capabilities.

This approach recognizes that static benchmarks quickly become obsolete in the field of LLM agents, where model capabilities advance at a sustained pace and benchmark contamination is a concrete risk.

---

## 7. 2025 Trends

The survey identifies six key areas that define the research and development trajectories for LLM agents in 2025 and beyond.

### 7.1 Scalability and Coordination

The management of complexity in multi-agent systems represents a growing challenge. As agentic systems scale from a few agents to dozens or hundreds, coordination mechanisms must evolve. Hybrid dynamic architectures (such as DyLAN and MDAgents) represent a promising direction, adapting the collaborative topology to the complexity of the task. The trend is towards **self-organizing** systems that emerge from simple local rules rather than being designed top-down.

### 7.2 Memory and Long-Term Adaptation

Memory constraints and long-term adaptation are identified as critical challenges. The transition from ephemeral session-bound memories to **persistent knowledge systems** that accumulate and refine experience across multiple interactions is a priority. MemGPT and similar systems with hierarchical memory represent the first steps, but the challenge of balancing retention and selective forgetting remains open.

### 7.3 Reliability and Scientific Rigor

The survey emphasizes the need to ensure "scientific rigor" in agent outputs, especially in critical domains such as scientific research, medicine and the legal sector. This requires not only the reduction of hallucinations, but also the ability to provide verifiable explanations, cite sources and quantify uncertainty in their conclusions.

### 7.4 Dynamic Multi-Turn and Multi-Agent Evaluation

Evaluation frameworks must evolve to capture complex collaborative scenarios, with multi-turn interactions among agents and environments that change over time. The trend is towards "living" benchmarks that continuously update and in-context evaluations that measure emergent behavior.

### 7.5 Regulation and Safe Deployment

The need for regulatory frameworks for the safe deployment of agents in critical domains is becoming increasingly urgent. The survey signals that "understanding the architectural foundations becomes essential not only for researchers but also for policymakers, industry professionals and society as a whole". Trends include the development of certification standards for agents, audit protocols for multi-agent systems and accountability frameworks for automated decisions.

### 7.6 From Static Architecture to Self-Optimization

The transition from static agentic architectures to **self-optimizing** systems represents perhaps the most transformative trend. Mechanisms of autonomous evolution (self-supervised learning, self-reflection, self-rewarding) and multi-agent co-evolution (cooperative and competitive) are converging towards agents capable of continuously improving their own capabilities without direct human intervention.

### 7.7 Enterprise Integration and Production

An emerging trend not explicitly addressed but clearly implied in the survey is the shift towards enterprise production agentic systems. Frameworks such as TheAgentCompany and OSWorld, which simulate realistic corporate environments, reflect the growing demand for agents capable of operating in complex work contexts with enterprise-level requirements for reliability, security and compliance.

---

## 8. Resources and References

### 8.1 Repositories and Collections

| Resource | Description |
|---------|-------------|
| [Awesome-Agent-Papers](https://github.com/luo-junyu/Awesome-Agent-Papers) | Collection curated by the authors of the survey, continuously updated |
| [arXiv:2503.21460](https://arxiv.org/abs/2503.21460) | Original paper of the survey |

### 8.2 Main Frameworks and Tools Cited

| Framework | Type | Focus |
|-----------|------|-------|
| MetaGPT | Centralized multi-agent | Software development with specialized roles |
| AutoGen | Conversational multi-agent | Group-chat and iterative debates |
| CAMEL | Role-playing multi-agent | Communication through role-play |
| ChatDev | Multi-agent for development | Complete software development cycle |
| AgentVerse | Dynamic multi-agent | Group dynamics |
| Voyager | Exploratory single-agent | Skill discovery in open environments |
| ReAct | Reasoning-action single-agent | Integration of thought and action |
| Reflexion | Reflective single-agent | Optimization through trials |
| MemGPT | Single-agent with memory | Hierarchical memory management |
| CoALA | Cognitive architecture | Modular memory and decision-making |
| CodeAct | Code-based action | Python code as action space |
| CRITIC | Self-correction | Tool-assisted verification |

### 8.3 Main Benchmarks

| Benchmark | Scope | Details |
|-----------|-------|----------|
| AgentBench | General | 8 interactive environments |
| Mind2Web | Web | 137 sites, 31 domains |
| MMAU | Cross-domain | 3,000+ tasks, 5 competencies |
| VisualAgentBench | Multimodal | Visual interactions |
| CRAB | Cross-platform | GUI-based evaluation |
| Seal-Tools | Tool use | 1,024 nested tool-call instances |
| CToolEval | Tool use (Chinese) | 398 APIs, 14 domains |
| MedAgentBench | Medicine | FHIR environment, 300 clinicians |
| OSWorld | Operating system | 369 multi-application tasks |
| EgoLife | Embodied | 300 hours of egocentric data |
| MLAgentBench | Machine Learning | ML experimentation |
| MLRB | ML research | 7 competitive tasks |
| MultiAgentBench | Multi-agent | Collaboration and competition |
| TheAgentCompany | Enterprise | Corporate simulation |

### 8.4 Foundational Papers Cited in the Survey

- **ReAct** -- Yao et al. (2023). Reasoning and acting with language models.
- **Tree of Thoughts** -- Yao et al. (2023). Deliberate exploration for problem solving.
- **Voyager** -- Wang et al. (2023). Continuous learning agent in Minecraft.
- **Generative Agents** -- Park et al. (2023). Simulacra of human behavior.
- **Reflexion** -- Shinn et al. (2023). Agents with verbal reflection.
- **MemGPT** -- Packer et al. (2023). Memory management for LLMs.
- **MetaGPT** -- Hong et al. (2023). Meta-programming for multi-agent collaboration.
- **AutoGen** -- Wu et al. (2023). Framework for multi-agent conversations.
- **CAMEL** -- Li et al. (2023). Communicative framework for agents.
- **ChatDev** -- Qian et al. (2023). Chat-powered software development.
- **Self-Rewarding LM** -- Yuan et al. (2024). Self-rewarding models.
- **STaR** -- Zelikman et al. (2022). Bootstrapping of reasoning.
- **CRITIC** -- Gou et al. (2024). Tool-interactive self-correction.
- **Toolformer** -- Schick et al. (2023). Learning to use APIs.

---

## Concluding Considerations

The survey by Luo et al. represents a fundamental contribution to the systematization of the LLM agent field in 2025. The three-dimensional methodological taxonomy (construction, collaboration, evolution) offers a conceptual framework that transcends previous categorizations, allowing understanding not only of how agents are built, but how they interact and how they improve over time.

Three main insights emerge from the overall analysis:

1. **Methodological convergence is underway**: the boundaries between prompt-based, fine-tuning and hybrid methods are blurring. The most effective systems combine all three categories, suggesting that the future of LLM agents lies in flexible architectures that dynamically select the optimal approach for each sub-task.

2. **Autonomous evolution is the key differentiator**: the ability of an agent to improve autonomously -- through self-reflection, self-rewarding and multi-agent co-evolution -- is emerging as the distinctive factor between research agentic systems and production-ready systems.

3. **Evaluation remains the bottleneck**: despite the proliferation of benchmarks, the community still lacks unified metrics that capture the complexity of agentic behavior in realistic scenarios. Self-evolutive benchmarks represent a promising direction but are still in an embryonic phase.

The coverage of 329 papers, combined with the systematic categorization of security, privacy and ethics challenges, makes this survey an essential reference point for anyone operating in the field of LLM-based agents, from academic researchers to engineering teams designing production systems.
