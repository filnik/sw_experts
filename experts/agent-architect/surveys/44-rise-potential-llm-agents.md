# The Rise and Potential of Large Language Model Based Agents: A Survey

## In-Depth Analysis of the Reference Survey on LLM-Based Agents

---

## 1. Overview

**Full title:** "The Rise and Potential of Large Language Model Based Agents: A Survey"
**Authors:** Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, Boyang Hong, Ming Zhang, Junzhe Wang, Senjie Jin, Enyu Zhou, Rui Zheng, Xiaoran Fan, Xiao Wang, Limao Xiong, Yuhao Zhou, Weiran Wang, Changhao Jiang, Yicheng Zou, Xiangyang Liu, Zhangyue Yin, Shihan Dou, Rongxiang Weng, Wensen Cheng, Qi Zhang, Wenjuan Qin, Yongyan Zheng, Xipeng Qiu, Xuanjing Huang, Tao Gui
**Affiliation:** Fudan University (NLP Group)
**Publication:** arXiv:2309.07864, September 2023; subsequently published in Science China Information Sciences (2025)
**Size:** 86 pages, 12 figures, 400+ bibliographic references
**Repository:** https://github.com/WooooDyy/LLM-Agent-Paper-List (6200+ stars)

### Why This Survey is Foundational

This survey represents one of the most comprehensive and systematic contributions in the field of Large Language Model-based agents. With its 29 authors from the prestigious NLP group at Fudan University and its 86 pages of rigorous analysis, the paper stands out for three fundamental reasons.

First, it establishes a **unified conceptual framework** for understanding LLM agents, organizing them according to a tripartite taxonomy (Brain, Perception, Action) that has become a reference point in the research community. Before this survey, the literature on agents was fragmented across multiple definitions and approaches; Xi et al. provided a common language and an organizational structure that allowed researchers to place their contributions within a coherent framework.

Second, the survey traces the evolution of the concept of "agent" from its philosophical origins to contemporary LLM-based implementations, providing a **historical perspective** that connects classical symbolic artificial intelligence to modern neural architectures. The authors convincingly argue that Large Language Models represent the most promising foundations for building adaptive AI agents, positioning them as a potential path towards artificial general intelligence (AGI).

Third, the analysis goes beyond mere technical catalogation to explore **agent societies**, examining emergent behaviors, personalities and social phenomena that emerge from interaction among multiple agents, offering insights applicable to understanding human society itself. This interdisciplinary approach, which spans computer science to computational sociology, makes the survey a document of unique value.

The paper has accumulated over 600 citations and the associated GitHub repository has become a living resource, constantly updated by the community, that catalogues and organizes research in this rapidly expanding field.

---

## 2. Taxonomy of LLM Agents: The Tripartite Framework

The survey proposes a conceptual architecture for LLM-based agents that articulates around three fundamental components: **Brain**, **Perception** and **Action**. This taxonomy, inspired by the analogy with biological organisms, provides a modular framework that can be adapted and configured for different application scenarios.

### 2.1 Brain: The LLM as Cognitive Core

The Brain component represents the heart of the intelligent agent. The authors identify the brain as the most critical element because it not only stores knowledge and memories, but also performs indispensable functions such as information processing and decision-making. The Brain exhibits reasoning and planning capabilities and effectively handles tasks never seen before, manifesting the agent's intelligence.

The main subcategories of the Brain include:

**Natural Language Interaction:** The LLM at the base of the Brain handles natural language interactions with high generative quality and deep semantic understanding. This includes high-quality text generation, deep language understanding, multi-turn dialogue management, the ability to follow complex instructions and cross-linguistic fluency. Natural language interaction is the primary channel through which the agent communicates with the external environment and with users.

**Knowledge Management:** The Brain integrates and manages different types of knowledge. Linguistic knowledge comes from pre-training on vast textual corpora. Commonsense knowledge allows the agent to reason about everyday situations. Actionable knowledge enables the agent to know how to use tools, APIs and external resources. However, the survey also highlights critical issues related to knowledge, such as hallucinations (generation of untrue information) and difficulties in knowledge editing (selective updating of obsolete or erroneous information).

**Transferability and Generalization:** The Brain demonstrates remarkable transfer and generalization capabilities, including in-context learning (ICL, learning from context without parameter updating), zero-shot and few-shot adaptation to unseen tasks, and continual learning with mitigation of catastrophic forgetting. These properties are essential for agents that must operate in dynamic and unpredictable environments.

### 2.2 Perception: Multimodal Inputs

The Perception module represents the agent's ability to perceive and interpret information from the surrounding environment through multiple sensory modalities. The survey identifies the following input categories:

**Textual Input:** The primary and most natural modality for LLMs, including user prompts, documents, web pages, source code and any information encoded in textual format. It represents the fundamental perceptual channel.

**Visual Input:** The integration of visual encoders such as Vision Transformer (ViT), Q-Former and architectures like BLIP-2 allows agents to process images, screenshots, diagrams and videos. This capability is crucial for agents that must interact with graphical interfaces, analyze visual documents or navigate virtual and physical environments.

**Audio Input:** The processing of audio signals, including speech, ambient sounds and music, extends the agent's perceptual capabilities beyond the textual and visual domain. Models such as Whisper and audio-language architectures allow the integration of this modality.

**3D and Environmental Input:** For embodied agents, perception includes understanding of three-dimensional environments, gestural recognition, physiological data and spatial awareness. These inputs are particularly relevant for robotic and virtual reality applications.

The modularity of the perceptual system allows for selectively combining these modalities based on the specific requirements of the application, creating agents that can operate effectively in purely textual contexts as well as in complex multimodal environments.

### 2.3 Action: Output and Tool Use

The Action component defines the modalities through which the agent acts on the environment and produces output. The survey categorizes actions into different types:

**Textual Output:** The generation of responses in natural language remains the most common form of action. It includes the production of text, the formulation of clarifying questions, the generation of reports and communication with other agents or with the user.

**Tool Use and API Interaction:** One of the most significant capabilities of LLM agents is the use of external tools. This includes the invocation of APIs for information retrieval (search engines, databases), the execution of code in sandbox environments, interaction with web services, file manipulation and the management of computational resources. Frameworks such as ReAct, Toolformer and HuggingGPT have demonstrated how LLMs can learn to select and use tools autonomously to solve complex tasks.

**Embodied Actions:** For agents operating in physical or simulated environments, actions include robotic control, object manipulation, spatial navigation and interaction with virtual environments such as Minecraft. Projects such as Voyager, DEPS and Inner Monologue have demonstrated the feasibility of LLM agents controlling avatars or robots in complex environments.

**Action Sequence Planning:** The agent does not limit itself to executing single actions, but can generate multi-step plans that coordinate a sequence of actions toward achieving a complex goal. This capability represents the bridge between the Brain's reasoning and the concrete execution of actions.

---

## 3. Key Components of Agent Architecture

### 3.1 Knowledge

Knowledge management is a central aspect in the architecture of LLM agents. The survey distinguishes different types of knowledge:

**Linguistic Knowledge:** Acquired during pre-training on vast textual corpora, includes understanding of syntax, semantics and pragmatic structures of language. This basic knowledge allows the agent to communicate effectively and to understand complex instructions.

**Commonsense Knowledge:** Comprises implicit knowledge about the physical world, causal relationships, social norms and everyday expectations. LLMs acquire this knowledge implicitly during pre-training, but its reliability and completeness remain open questions.

**Domain Knowledge:** Specialized knowledge in specific sectors such as medicine, law, finance or software engineering. It can be integrated through fine-tuning on specialized datasets, Retrieval-Augmented Generation (RAG) or prompt engineering with specific context.

**Actionable Knowledge:** Perhaps the most distinctive type for agents, includes procedural knowledge on how to use tools, invoke APIs, perform operations and interact with external systems. This knowledge transforms the LLM from a passive text generator into an agent capable of acting in the world.

**Knowledge Issues:** The survey highlights important critical issues. Hallucinations represent the most pervasive problem: the agent can generate plausible but false information with high confidence. Knowledge editing, that is, the ability to selectively update obsolete or erroneous information without compromising other knowledge, remains a significant technical challenge. Knowledge grounding, that is, anchoring responses to verifiable sources, is an active area of research.

### 3.2 Memory: Architectures for Recall and Learning

The memory system is one of the most articulated contributions of the survey. The authors identify a taxonomy of memories that reflects, in part, the distinctions of cognitive psychology:

**Short-Term Memory:** Corresponds to the LLM's context window and the immediate working buffer. It includes recent conversation information, current observations and the current state of the task. Short-term memory is intrinsically limited by the size of the underlying model's context window (e.g., 4K, 8K, 32K, 128K tokens). Techniques to extend this memory include attention scaling, context compression and incremental summarization.

**Long-Term Memory:** Represents the persistent knowledge that the agent accumulates over time and that survives beyond a single interaction session. The survey identifies different implementations: vector embeddings stored in vector databases (e.g., Pinecone, ChromaDB, FAISS), structured knowledge bases (knowledge graphs), summarized historical records and persistent user profiles. Retrieval from long-term memory typically occurs via similarity search on embeddings, with ranking and filtering mechanisms to select the most relevant memories for the current context.

**Episodic Memory:** Catalogs specific experiences of the agent, including past attempts, successes, failures and lessons learned from each episode. This type of memory is particularly relevant for the Reflexion framework by Shinn et al., where the agent generates verbal analyses of its own failures and uses these reflections as episodic memory to improve subsequent attempts. Episodic memory enables a form of experiential learning without updating the model's gradients.

**Memory Challenges:** With the accumulation of observations and action sequences, agents face an increasing mnemonic burden. The survey highlights the difficulty in extracting relevant memories from a continuously expanding archive, the risk of interference between old and new memories, and the need for active forgetting mechanisms to keep the mnemonic system efficient. The hierarchical organization of memories, with increasing levels of abstraction, is proposed as a possible architectural solution.

### 3.3 Reasoning: Chain-of-Thought, Tree-of-Thought and Self-Reflection

Reasoning represents the agent's ability to derive logical conclusions, solve complex problems and make informed decisions. The survey catalogs the main methodologies:

**Chain-of-Thought (CoT):** Introduced by Wei et al. (2022), the CoT paradigm has demonstrated that prompting LLMs with examples of step-by-step reasoning substantially improves performance on complex tasks. Instead of producing a direct answer, the agent generates an explicit chain of intermediate steps that make the reasoning process transparent. Variants include Zero-Shot CoT ("Let's think step by step") and Manual CoT with manually curated exemplars.

**Self-Consistency:** Proposed by Wang et al., this technique generates multiple chains of reasoning for the same problem and selects the answer that appears most frequently. The intuition is that different reasoning paths converging on the same answer provide greater confidence in the correctness of the solution.

**Tree-of-Thought (ToT):** Developed by Yao et al. (2023), ToT extends CoT by exploring multiple branches of reasoning in a tree structure. Instead of following a single linear path, the agent generates several possibilities at each step, evaluates them and can perform backtracking to explore alternatives. This approach is particularly effective for problems that require exploration (puzzles, strategic planning, creative problems) where the optimal path is not immediately evident.

**Self-Reflection and Reflexion:** Reflexion, proposed by Shinn et al., extends the reasoning paradigm by incorporating self-reflection. When an agent fails at a task, instead of immediately retrying, it generates a verbal analysis of what went wrong and how to improve. These reflections are stored in an episodic memory buffer. In subsequent attempts, the agent conditions its behavior on the accumulated reflections, enabling experiential learning without updating gradients. This mechanism simulates a form of artificial metacognition.

**ReAct (Reasoning + Acting):** Framework that alternates phases of reasoning and action, allowing the agent to think aloud while interacting with the environment. Each ReAct cycle includes a thought (reasoning about the next step), an action (concrete execution) and an observation (feedback from the environment), creating a structured interaction loop.

### 3.4 Planning: Task Decomposition and Feedback

Planning represents the agent's ability to formulate strategies to achieve complex goals:

**Task Decomposition:** The fundamental approach to planning consists of decomposing complex tasks into more manageable sub-tasks. The LLM analyzes the high-level objective and generates an ordered sequence of sub-objectives, each solvable in a relatively independent manner. Techniques such as Least-to-Most prompting and recursive decomposition allow tackling problems that would be intractable as a single step.

**Plan Formulation:** Plan formulation can follow different approaches. Top-down planning starts from the final goal and works backwards to identify the necessary steps. Bottom-up planning builds the plan starting from the elementary actions available. Hybrid planning combines both approaches.

**Plan Reflection and Refinement:** A crucial aspect is the agent's ability to reflect on its own plan and refine it iteratively. When the execution of a sub-task fails or produces unexpected results, the agent can re-evaluate the overall plan, adapt the strategy and generate alternative paths. This planning-execution-reflection-replanning cycle is fundamental for the agent's robustness in real environments.

**Feedback Integration:** The plan is continuously updated based on feedback received from the environment, tools and users. Feedback can be quantitative (success metrics, scores) or qualitative (user comments, error messages). Frameworks such as Inner Monologue explicitly integrate different sources of feedback into the planning cycle.

---

## 4. Applications of LLM Agents

The survey organizes applications in a hierarchical scheme that distinguishes between single-agent, multi-agent and human-agent cooperation scenarios.

### 4.1 Web Agents

Web agents represent one of the most mature and practical applications. These agents autonomously navigate the web, interact with user interfaces, fill in forms, perform searches and complete complex tasks that require traversal of multiple websites.

Projects such as WebGPT, WebAgent and Mind2Web have demonstrated that LLMs can interpret web pages (via HTML, screenshots or DOM), plan sequences of navigation actions (click, scroll, typing) and adapt to interfaces never seen before. The main challenge remains robustness: web pages change continuously, UI elements can be ambiguous and agents must handle error states, popups and authentication flows.

### 4.2 Coding Agents

Coding agents represent a rapidly evolving area of application. These agents can generate code from natural language specifications, perform automatic debugging, perform refactoring, write tests and manage entire software development workflows.

Systems like MetaGPT organize a team of agents with specialized roles (product manager, architect, programmer, tester) that collaborate to produce functional software. ChatDev simulates a virtual software development company with agents that communicate via structured dialogue. Devin and SWE-Agent have demonstrated that agents can solve real issues on GitHub repositories, navigating the file system, editing code and running tests.

Evaluation metrics include HumanEval, MBPP and SWE-bench, the latter particularly significant because it evaluates the ability of agents to solve real issues on open-source projects.

### 4.3 Scientific Agents

Scientific agents apply LLM capabilities to scientific discovery, experimental design and data analysis. The survey documents applications in different domains:

**Drug Discovery:** Agents that design molecules, predict pharmacological properties and optimize lead compounds by interacting with computational chemistry tools.

**Mathematical Reasoning:** Agents that solve complex mathematical problems, generate proofs and verify the correctness of formal proofs.

**Scientific Literature:** Agents that analyze scientific literature, identify research gaps and suggest experimental directions. Systems such as Galactica and ScholarGPT demonstrate these capabilities.

**Experimental Automation:** Agents that control laboratory instruments, plan experiments and analyze results, closing the loop between hypothesis and experimental verification.

### 4.4 Other Notable Applications

**Task-Oriented Agents:** Agents for home automation (household robotics), management of user interfaces (UI navigation) and execution of tasks in the real world.

**Lifecycle-Oriented Agents:** Agents that autonomously acquire new skills, continuously improve and learn curricula in open virtual environments. Voyager in Minecraft represents a paradigmatic example of an agent that explores, learns and accumulates skills autonomously.

**Agents for Education and Health:** Applications in the instructor-executor paradigm, where the agent acts as a personalized tutor or clinical assistant, with the human providing supervision and guidance.

---

## 5. Multi-Agent Systems

The survey devotes a substantial section to multi-agent systems, recognizing that many complex problems benefit from the collaboration of multiple agents with diverse roles and competencies.

### 5.1 Cooperative Interaction

Cooperative systems represent the most common paradigm in LLM-based multi-agent systems:

**Ordered Collaboration (Pipelined):** Agents are organized in a production chain where each has a specific role and passes its output to the next. MetaGPT is the canonical example, with roles such as Product Manager, Architect, Programmer and Tester organized in a sequential workflow. CAMEL (Communicative Agents for "Mind" Exploration of Large Language Models) establishes a role-playing framework where two agents collaborate with complementary roles (e.g., instructor and assistant).

**Peer-Based Collaboration:** Agents with equal roles that contribute to the solution through voting, discussion or fusion of different perspectives. AgentVerse implements a framework where groups of agents collaborate dynamically, forming ad hoc teams for specific tasks.

**Division of Labor:** Systems that automatically decompose the task and assign sub-tasks to specialized agents. Orchestration can be centralized (a coordinator agent) or decentralized (emergent from interaction).

### 5.2 Competitive and Adversarial Interaction

Competitive interactions between agents serve purposes other than mere opposition:

**Debate Framework:** Two or more agents argue contrasting positions on an issue, forcing each to refine their arguments and confront counter-arguments. This mechanism improves the quality of reasoning and reduces hallucinations, since unfounded claims are challenged by other agents. Du et al. demonstrated that debate among LLM agents improves the correctness of answers on reasoning tasks.

**Self-Play:** An agent competes against versions of itself or against other agents in game, negotiation or strategy scenarios. This approach, inspired by AlphaGo's self-play, allows the continuous improvement of strategic capabilities.

**Uncertainty Calibration:** The opposition between agents can be used to calibrate confidence in answers: when multiple agents agree, confidence increases; when they diverge, it signals uncertainty or ambiguity.

### 5.3 Agent Societies

One of the most innovative aspects of the survey is the exploration of agent societies, where multiple LLM agents interact in simulated environments creating emergent social phenomena:

**Generative Agents:** The seminal work by Park et al. ("Generative Agents: Interactive Simulacra of Human Behavior") demonstrated that LLM agents equipped with memory, reflection and planning can simulate credible human behaviors in a virtual environment similar to a video game. The agents develop daily routines, form social relationships and react to events in an emergent manner.

**Emergent Properties:** The survey identifies key properties of agent societies: openness (dynamic populations), persistence (continuous simulation), situatedness (context-driven decisions), organization (emergent roles and hierarchies). These properties allow the study of phenomena such as the propagation of values, the formation of social structures and the emergence of collective intelligence.

**Implications for Social Sciences:** The authors argue that agent societies can serve as a computational laboratory for the social sciences, allowing the simulation and study of social dynamics (diffusion of opinions, formation of norms, economic behavior) in a controlled and reproducible manner.

---

## 6. Challenges and Future Directions

The survey identifies multiple open challenges and future research directions that define the agenda of the field:

### 6.1 Robustness and Security

**Adversarial Vulnerabilities:** LLM agents are susceptible to attacks such as prompt injection, jailbreaking and knowledge poisoning. Since agents interact with the external environment (web navigation, code execution, API invocation), the attack surface is significantly larger compared to traditional chatbots. A prompt injection inserted in a web page could hijack the agent's behavior.

**Multimodal Attack Surfaces:** The integration of visual, audio and textual perception creates new vulnerabilities. Adversarial images, manipulated audio and malicious textual content can be combined to compromise the system.

**Ethical and Social Risks:** The growing autonomy of agents raises profound ethical questions regarding responsibility, transparency and control. The authors emphasize the need for ethical guardrails and human intervention mechanisms.

### 6.2 Evaluation and Benchmark

**Inadequacy of Current Benchmarks:** The survey highlights the lack of multidimensional evaluation frameworks for LLM agents. Existing benchmarks tend to evaluate individual capabilities (reasoning, tool use, planning) without capturing the complexity of end-to-end agent behavior.

**Utility Metrics:** Beyond correctness, it is necessary to evaluate the efficiency, robustness, security and usability of agents. Metrics for sociability (ability to collaborate with other agents and with humans) and for alignment with human values are still in an embryonic phase.

### 6.3 Scalability

**Computational Complexity:** The scalability of multi-agent systems remains a significant challenge. As the number of agents increases, the computational cost of interactions grows rapidly, and the dilution of information can degrade performance.

**Bias Amplification:** In large-scale agent societies, the biases present in individual LLMs can be amplified by network dynamics, creating computational echo chambers and reinforcing prejudices.

**Coordination Overhead:** Coordination between multiple agents introduces communication complexity that can outweigh the benefits of collaboration if not managed effectively.

### 6.4 Simulation-to-Reality Transfer

**Perceptual Gap:** There is a significant divide between the simulated environments where agents are developed and tested and the real world. Perception in real environments is noisy, ambiguous and constantly evolving.

**Action Generalization:** Actions that work perfectly in simulation can fail in the physical world due to hardware variability, environmental conditions and unmodeled constraints.

### 6.5 Identified Future Directions

1. **Rigorous Evaluation Frameworks:** Development of multidimensional benchmark suites that capture the complexity of agent behavior in realistic scenarios.

2. **Scalability Solutions:** Efficient algorithms for inter-agent communication, hierarchical architectures and optimized coordination protocols.

3. **Value Alignment:** Deeper integration of human values, ethical guardrails and security mechanisms into agent design.

4. **Embodied Integration:** Bridging the gap between virtual agents and deployment in the physical world, with focus on robust perception and reliable actions.

5. **Agent-as-a-Service:** Cloud ecosystem models where specialized agents can be composed and orchestrated as services, analogously to microservices in software engineering.

6. **Computational Social Sciences:** Use of large-scale agent societies for social, economic and political simulations that generate insights applicable to understanding and improving human society.

---

## 7. The GitHub Paper List: Navigating 400+ References

The GitHub repository [WooooDyy/LLM-Agent-Paper-List](https://github.com/WooooDyy/LLM-Agent-Paper-List), with over 6200 stars, represents a living resource that continuously extends and updates the survey.

### Repository Structure

The repository is organized following the survey's taxonomy faithfully, facilitating thematic navigation:

**Section 1 - Construction of LLM-based Agents:**
- **Brain:**
  - Natural Language Interaction (High-quality generation, Deep understanding)
  - Knowledge (Pretrain model knowledge, Linguistic knowledge, Commonsense knowledge, Actionable knowledge, Knowledge issues)
  - Memory (Memory capability, Memory retrieval)
  - Reasoning & Planning (Reasoning, Planning with formulation, Planning with reflection)
  - Transferability & Generalization (Unseen task generalization, In-context learning, Continual learning)
- **Perception:**
  - Visual (Image/Video processing)
  - Audio (Speech/Sound processing)
- **Action:**
  - Tool Use (API invocation, Code execution)
  - Embodied Action (Robotic control, Virtual environments)

**Section 2 - Applications:**
- Single-Agent (Task-oriented, Innovation-oriented, Lifecycle-oriented)
- Multi-Agent (Cooperative, Adversarial)
- Human-Agent Cooperation (Instructor-Executor, Equal Partnership)

**Section 3 - Agent Society:**
- Behavior and Personality
- Environment (Text-based, Virtual Sandbox, Physical)
- Society Simulation
- Emergent Properties

### How to Navigate Effectively

For those approaching the literature on LLM agents, the following navigation path is suggested:

**For beginners:** Start from the Brain > Natural Language Interaction section to understand the foundations, then explore Memory and Reasoning to understand how agents extend the basic capabilities of LLMs. Read seminal papers such as ReAct, Chain-of-Thought and Reflexion.

**For researchers on specific applications:** Navigate directly to the Applications section and select the domain of interest (web, coding, science). Each sub-section lists the most relevant papers with direct links.

**For those working on multi-agent:** Explore the Multi-Agent section with focus on Cooperative Interaction. Key papers: CAMEL, MetaGPT, AgentVerse, ChatDev.

**For those exploring social aspects:** The Agent Society section is particularly innovative. Start from the Generative Agents paper by Park et al. and continue with social simulations.

The repository is regularly updated by the community, so it is advisable to "Watch" it to receive notifications of new papers added. Pull Requests from the community help keep the list updated with the most recent publications.

---

## 8. Resources and References

### Main Paper
- **Xi, Z., Chen, W., Guo, X., et al.** (2023). "The Rise and Potential of Large Language Model Based Agents: A Survey." arXiv:2309.07864. Subsequently published in *Science China Information Sciences* (2025). DOI: 10.1007/s11432-024-4222-0
  - arXiv: https://arxiv.org/abs/2309.07864
  - PDF: https://arxiv.org/pdf/2309.07864

### Official Repository
- **LLM-Agent-Paper-List:** https://github.com/WooooDyy/LLM-Agent-Paper-List

### Foundational Papers Cited in the Survey

**Reasoning and Planning:**
- Wei, J., et al. (2022). "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models." NeurIPS 2022.
- Yao, S., et al. (2023). "Tree of Thoughts: Deliberate Problem Solving with Large Language Models." NeurIPS 2023.
- Wang, X., et al. (2023). "Self-Consistency Improves Chain of Thought Reasoning in Language Models." ICLR 2023.
- Shinn, N., et al. (2023). "Reflexion: Language Agents with Verbal Reinforcement Learning." NeurIPS 2023.
- Yao, S., et al. (2023). "ReAct: Synergizing Reasoning and Acting in Language Models." ICLR 2023.

**Tool Use and Action:**
- Schick, T., et al. (2023). "Toolformer: Language Models Can Teach Themselves to Use Tools." NeurIPS 2023.
- Shen, Y., et al. (2023). "HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in Hugging Face." NeurIPS 2023.

**Application Agents:**
- Wang, G., et al. (2023). "Voyager: An Open-Ended Embodied Agent with Large Language Models." arXiv.
- Hong, S., et al. (2023). "MetaGPT: Meta Programming for Multi-Agent Collaborative Framework." ICLR 2024.
- Qian, C., et al. (2023). "ChatDev: Communicative Agents for Software Development." ACL 2024.
- Li, G., et al. (2023). "CAMEL: Communicative Agents for 'Mind' Exploration of Large Scale Language Model Society." NeurIPS 2023.

**Agent Societies:**
- Park, J.S., et al. (2023). "Generative Agents: Interactive Simulacra of Human Behavior." UIST 2023.
- Du, Y., et al. (2023). "Improving Factuality and Reasoning in Language Models through Multiagent Debate." ICML 2024.

### Related Surveys
- Wang, L., et al. (2023). "A Survey on Large Language Model based Autonomous Agents." arXiv:2308.11432.
- Guo, T., et al. (2024). "Large Language Model based Multi-Agents: A Survey of Progress and Challenges." IJCAI 2024.

### Additional Resources
- **Semantic Scholar:** https://www.semanticscholar.org/paper/0c72450890a54b68d63baa99376131fda8f06cf9
- **Hugging Face Papers:** https://huggingface.co/papers/2309.07864
- **AlphaXiv Discussion:** https://www.alphaxiv.org/resources/2309.07864v2
- **Springer (published version):** https://link.springer.com/article/10.1007/s11432-024-4222-0

---

*Document prepared as an in-depth analysis of the survey "The Rise and Potential of Large Language Model Based Agents" by Xi et al. (2023). Last revision: March 2026.*
