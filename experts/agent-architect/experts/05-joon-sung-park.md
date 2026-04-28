# Joon Sung Park and the Generative Agents Project

## Interactive Simulacra of Human Behavior

---

## 1. Profile and Academic Background

### Training and Path

Joon Sung Park is one of the most influential researchers at the intersection of Human-Computer Interaction (HCI), Natural Language Processing (NLP), and generative artificial intelligence. His academic path winds through three institutions of excellence:

- **Swarthmore College** -- Bachelor of Arts in Computer Science, where he built the theoretical foundations in computer science and cognitive science.
- **University of Illinois at Urbana-Champaign (UIUC)** -- Master of Science in Computer Science, under the supervision of Karrie Karahalios, where he began exploring the social dimension of computational systems.
- **Stanford University** -- Ph.D. in Computer Science (completed in June 2025), co-supervised by Michael S. Bernstein and Percy Liang, within the Human-Computer Interaction Group and the Natural Language Processing Group.

At Stanford, Park is affiliated with the Stanford Artificial Intelligence Lab (SAIL) and the HAI (Human-Centered Artificial Intelligence) group, positioning himself at a strategic point between fundamental research on large language models and their concrete application in social and interactive contexts.

### Research Vision

Park's research vision revolves around a fundamental question: **how can we build computational agents that credibly simulate human behavior?** This question is not purely academic -- it has direct implications for the design of social systems, the simulation of public policies, the testing of digital platforms, and the very understanding of social cognition.

His approach is distinctive because it combines three rarely integrated competencies:

1. **HCI** -- deep understanding of how people interact with systems and with each other through technology.
2. **NLP/LLM** -- technical mastery of large language models as a computational substrate for behavior simulation.
3. **Social Computing** -- sensitivity to emergent dynamics in communities and social systems.

### Recognitions and Awards

Park's work has received significant recognition from the academic community:

- **Best Paper Award at UIST 2023** for "Generative Agents: Interactive Simulacra of Human Behavior"
- **Best Paper Award at CHI 2022** for "Jury Learning"
- **Best Paper Honorable Mention** at UIST 2025 and CHI 2021
- **Microsoft Research Ph.D. Fellowship** (2022)
- **Terry Winograd Fellowship** at Stanford
- **Siebel Scholar Award** (2019)
- **Stanford School of Engineering Fellowship** (2021)

His works have been covered by leading international media: The New York Times, The Guardian, The New Yorker, Forbes, WIRED, Nature, and Science. Currently his professional contact is associated with **Simile AI** (joon@simile.ai), suggesting an evolution from academic research toward industrial application of generative agents.

---

## 2. Generative Agents: The Social Simulation of Smallville

### The Project

The paper "Generative Agents: Interactive Simulacra of Human Behavior", published at UIST 2023 (arXiv:2304.03442), represents a turning point in research on autonomous agents. The authors -- Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, and Michael S. Bernstein -- introduced the concept of **generative agents**: computational agents powered by large language models capable of simulating credible human behavior in an interactive sandbox environment.

The central idea is radical in its simplicity: rather than explicitly programming behavioral rules, each agent is endowed with a **persistent memory** in natural language, **reflection** mechanisms to synthesize experiences into high-level insights, and a **planning** system to translate goals into concrete actions. The result is behavior that emerges organically from the interaction between agents, environment, and memory.

### Smallville: The Sandbox Environment

The simulation environment, called **Smallville**, is inspired by the game The Sims. It is a small virtual town implemented using the **Phaser** web development framework, in which 25 unique agents live their daily lives. Each agent possesses:

- **A defined identity** in natural language: name, occupation, relationships, personality, and goals, described in an initial "seed memory" paragraph.
- **Physical spaces** in which to operate: houses, offices, shops, a park, a bar, a library -- all mapped in a navigable 2D environment.
- **The ability to perceive** other agents and objects in the surrounding environment.

The technical backend consists of two main systems:

1. **Environment Server** -- a Django application that manages the visualization of the game world, the state of the environment, and agent navigation.
2. **Simulation Server** -- the agents' reasoning engine, which uses calls to large language models (initially GPT-3.5 and GPT-4 via OpenAI API) to generate perceptions, reflections, plans, and actions.

### The Lives of the Agents

The Smallville agents are not simple chatbots: they **live**. They wake up in the morning, prepare breakfast, go to work, interact with colleagues, have lunch, return home, and have hobbies and relationships. Some examples from the agent identities:

- **Klaus Mueller** -- research student working on gentrification, often busy in the local library.
- **Isabella Rodriguez** -- runs a bar and organizes events for the community.
- **Sam Moore** -- interested in local politics, considering running for mayor.
- **Maria Lopez** -- artist working in her studio.
- **Tom Moreno** -- shop owner, a reference point for the neighborhood.

Each agent has a network of pre-existing relationships (family, friends, colleagues) and the ability to form new ones through interaction.

### User Interaction

A notable aspect of the project is the possibility for a **human user to interact with the agents**. The user can observe the simulation in real time, enter natural language input to influence the narrative (for example, suggesting to an agent that they could organize a party), and observe how these inputs propagate through the agents' social network.

---

## 3. Memory Architecture

### The Memory Stream

The heart of the Generative Agents architecture is the **memory stream**: a complete and chronological log of all the agent's experiences, stored in natural language. Each entry in the memory stream is a **memory object** containing:

- **Textual description** of the event or observation (e.g., "Klaus Mueller is working on his thesis in the library").
- **Creation timestamp** -- when the observation was recorded.
- **Last access timestamp** -- when the memory was last retrieved.
- **Importance score** -- a numerical value assigned by the LLM.

The memory stream accumulates three fundamental types of objects:

1. **Observations** -- direct perceptions of the environment and other agents' actions. As an agent moves through the environment, it observes its surroundings and these perceptions are recorded as observations (e.g., "Isabella Rodriguez is talking with Tom Moreno at the bar", "The whiteboard in the lab shows the study results").
2. **Reflections** -- high-level syntheses derived from the aggregation of observations and other previous reflections.
3. **Plans** -- intentions and action programs for the near and far future.

### The Retrieval Mechanism

The fundamental problem of memory is **retrieval**: with hundreds or thousands of memory objects accumulated, how does the agent decide which memories are relevant to the current situation? The system uses a scoring function that combines three dimensions:

**Score = alpha_recency * recency + alpha_importance * importance + alpha_relevance * relevance**

Where all alpha values are set to 1 in the original implementation, and each component is normalized in the range [0, 1] via min-max scaling.

#### Recency

Recency models the **temporal decay** of memory: memories accessed more recently receive a higher score. The mechanism uses an **exponential decay** function based on the time elapsed since the last access. This reflects a well-known principle in cognitive psychology: recent experiences are more easily accessible in human memory.

The decay formula is: **recency = decay_rate ^ (hours since last access)**

Where the decay rate is a parameter that controls the speed of decay. A value of 0.99, for example, produces a slow and gradual decay, keeping memories relatively fresh for longer periods.

#### Importance

Importance distinguishes between **trivial and significant memories**. The score is assigned directly by the LLM on a scale from 1 to 10, where:

- **1** = ordinary and routine events (e.g., "Maria Lopez is brushing her teeth", "Klaus Mueller is having breakfast")
- **5** = moderately significant events (e.g., "Sam Moore had an interesting discussion about politics with Tom")
- **10** = events of great emotional or practical impact (e.g., "Isabella Rodriguez discovered that her best friend is moving", "Klaus received an important job offer")

This classification occurs at the time of memory creation, via a prompt to the LLM that presents the observation and asks to evaluate its relevance to the agent's life.

#### Relevance

Relevance measures the **semantic similarity** between a memory and the current context. The calculation occurs through:

1. Generation of a **vector embedding** for each memory object.
2. Generation of an embedding for the **current query** (the agent's situation or current thought).
3. Calculation of the **cosine similarity** between the two vectors.

This ensures that memories semantically related to the present situation are retrieved even if they are not recent or particularly important in absolute terms.

#### The Elegance of the Tripartite System

The combination of these three dimensions produces a sophisticated and credible retrieval behavior:

- If an agent is talking with Klaus about gentrification research, the system retrieves **relevant** memories (previous conversations about the research), **recent** memories (interactions with Klaus today), and **important** memories (key moments of the research).
- A very old but extremely important memory (score 10) can still emerge if sufficiently relevant to the current context.
- Recent trivial memories can be retrieved if directly pertinent to the situation.

---

## 4. Reflection and Planning

### The Reflection Mechanism

Reflection is the mechanism that distinguishes Generative Agents from a simple chatbot with memory. Without reflection, agents would operate exclusively on raw observations, unable to synthesize experiences into **high-level understandings**.

#### Reflection Trigger

Reflection does not happen continuously, but is **activated by a trigger**: when the cumulative sum of importance scores of recent events exceeds a **predefined threshold**. In the original implementation, this happens approximately **two or three times a day** (in simulated time), corresponding to natural moments of pause and introspection.

#### The Reflection Process

The process is articulated in three phases:

1. **Extraction of salient questions**: the system retrieves the 100 most recent memories and presents them to the LLM with the prompt: "Given the statements above, what are the 3 most salient high-level questions we can answer?". This generates questions such as: "What are Klaus's priorities in his research?" or "How is the relationship between Isabella and Tom evolving?".

2. **Evidence gathering**: for each generated question, the system performs a targeted retrieval in the memory stream, retrieving the memories most relevant to that specific question.

3. **Insight generation**: the LLM synthesizes the retrieved memories into **high-level statements** (reflections), complete with citations to the source memories. For example: "Klaus Mueller is deeply dedicated to his research on gentrification (based on memories 1, 2, 8, 15)".

#### Tree Structure of Reflections

Reflections are not based only on direct observations: they can also be based on **other previous reflections**. This creates a hierarchical tree structure where:

- **Leaves** = direct observations (perceptions of the environment)
- **Intermediate nodes** = first-level reflections (syntheses of observations)
- **Higher nodes** = higher-level reflections (meta-reflections, syntheses of reflections)

This architecture allows agents to develop a progressively more abstract and sophisticated understanding of themselves, of others, and of the world. An agent might, through successive levels of reflection, go from specific observations ("I talked to Sam about politics three times this week") to general insights ("Sam is a person with strong political convictions who is considering an active role in the community") to meta-reflections ("People in my neighborhood are increasingly involved in local political life").

### The Planning System

Planning translates reflections, memories, and environmental context into **concrete actions**. The approach is **top-down and recursive**, inspired by the way humans plan their days.

#### Recursive Decomposition

The planning process is articulated in three levels of granularity:

1. **High-level daily plan**: at the beginning of each simulated day, the agent generates an overall plan with 5-8 main points, based on its identity description, the summary of the previous day, and the accumulated reflections. Example: "Klaus wakes up and has breakfast (7:00), goes to the library to work on his thesis (8:30), has lunch with his advisor (12:00), continues research in the afternoon (14:00), attends the department seminar (16:00), has dinner at home (19:00), reads a book before sleeping (21:00)".

2. **Hourly decomposition**: each point of the high-level plan is expanded into hourly blocks with greater detail.

3. **Fine-grained actions**: hourly blocks are further decomposed into actions of **5-15 minutes**, the level of granularity at which the agent actually operates in the environment. Example: "8:30 -- Arrives at the library. 8:35 -- Looks for his usual spot. 8:40 -- Opens the laptop and resumes the review of chapter 3. 8:55 -- Consults a new paper on gentrification downloaded yesterday."

#### Reactivity and Re-planning

Plans are not rigid. Agents **continuously perceive the environment** and can decide to deviate from the plan in response to unforeseen events. The mechanism works as follows:

1. The agent perceives a new event (e.g., "Isabella Rodriguez approaches and greets Klaus").
2. The system presents the LLM with the agent's current plan and the new observation.
3. The LLM decides whether the agent should **continue with the plan** or **react to the event**.
4. If it reacts, a new sequence of fine-grained actions is generated that replaces the current plan.

This dialectic between planning and reaction is fundamental to the credibility of behavior: agents are neither completely deterministic nor completely reactive, but balance intention and adaptation, exactly like human beings.

---

## 5. Emergent Behaviors

### The Phenomenon of Emergence

One of the most significant aspects of the Generative Agents project is the demonstration that **complex social behaviors can emerge spontaneously** from the interaction between individual agents, without being explicitly programmed. These emergent behaviors represent the most convincing validation of the proposed architecture.

### The Valentine's Day Party

The most celebrated and iconic example is the **Valentine's Day party**. The initial setup was minimal: a user simply suggested to Isabella Rodriguez that she could organize a Valentine's Day party at her bar. From this single input, an entire cascade of autonomous social interactions developed:

1. **Information diffusion**: Isabella began mentioning the party in her daily conversations with other agents. The information propagated through the agents' social network over the course of two simulated days, going from 4% to **48% of agents informed**.

2. **Couple formation**: some agents, having learned of the party, autonomously decided to **invite someone as a companion**. This required a chain of complex reasoning: evaluating their own relationships, identifying a potential partner, finding the right moment and way to ask.

3. **Temporal coordination**: invited agents had to **plan their day** to be present at the party at the correct time, modifying their plans coherently.

4. **Realistic participation**: of the 12 invited agents, only **5 actually showed up**. Three cited **schedule conflicts** as the reason for absence, four expressed interest but did not plan to attend. This pattern is extraordinarily realistic: in real life, not all invitees to a party show up, and the reasons vary.

### Diffusion of Information about the Mayoral Candidacy

Another notable example concerns Sam Moore's candidacy for mayor. Starting from initial information known only to Sam himself (4% of agents), over the course of the simulation, **32% of agents** became aware of it through natural conversations. The information propagated through chains of conversations: Sam talked about it with Tom at the shop, Tom mentioned it to Isabella at the bar, Isabella discussed it with her customers, and so on. This diffusion faithfully mirrors the patterns of information propagation observed in real social networks.

### Formation and Evolution of Relationships

Agents have demonstrated the ability to **form new relationships** and to **evolve existing ones** in a credible way:

- **Sam and Latoya**: initially strangers, after a chance meeting in which Latoya mentioned her photography project, Sam memorized this information. In a subsequent meeting, Sam **spontaneously asked Latoya for updates on her project**, demonstrating relational continuity and genuine interest -- a behavior that requires memory, retrieval, and conversational planning.

- **Increasing relational density**: analyzing the network of relationships between agents from the beginning to the end of the simulation, the researchers observed a **significant increase in the density of connections**, with new relationships formed through emergent interactions.

### Group Coordination

Beyond the Valentine's Day party, agents have shown other forms of coordination:

- **Sharing of informational resources**: agents with common interests exchanged useful information during casual conversations.
- **Emotional support**: agents who observed another agent in difficulty offered words of comfort or assistance, without this behavior being programmed.
- **Shared routines**: some agents developed emergent social routines, such as meeting regularly at the same place at the same time.

---

## 6. Lessons for Agent Builders

### Transferable Architectural Patterns

The Generative Agents project offers a treasure trove of **architectural patterns** applicable to any autonomous agent system. Here are the most important lessons for those building agent-based systems.

### 6.1 The Memory Stream Pattern

**Principle**: every agent must maintain a persistent and chronological log of its experiences in natural language.

**Practical application**:
- Do not limit yourself to storing recent interactions in a fixed context window.
- Implement a **persistent database** of all the agent's experiences.
- Each entry must contain metadata (timestamp, importance, vector embedding).
- The format must be natural language, not rigid data structures -- this allows flexibility in semantic retrieval.

**Common error**: many agent systems implement memory as a simple FIFO (first-in, first-out) buffer that discards older memories. Park's architecture demonstrates that **long-term persistence** is fundamental for credible behavior.

### 6.2 The Tripartite Retrieval Pattern

**Principle**: memory retrieval must balance recency, importance, and semantic relevance.

**Practical application**:
- Implement a **composite scoring system** that combines:
  - Exponential temporal decay for recency
  - Importance classification (automatic via LLM or rule-based)
  - Cosine similarity on embeddings for relevance
- Normalize each component in [0, 1] before combining them.
- Make the weights (alpha) **configurable** and possibly adaptive to context.

**Key insight**: tripartite retrieval outperforms both simple temporal retrieval (which retrieves only recent memories) and pure semantic retrieval (which ignores the temporal factor). The combination produces more human and contextually appropriate behavior.

### 6.3 The Periodic Reflection Pattern

**Principle**: agents must periodically synthesize their experiences into high-level insights.

**Practical application**:
- Implement a **threshold-based trigger**: accumulate importance scores and activate reflection when the threshold is exceeded.
- The reflection process should: (a) generate salient questions from recent memories, (b) gather evidence, (c) synthesize insights.
- Reflections must be **stored in the memory stream** along with observations, with references to source memories.
- Allow **reflections on reflections** to build a hierarchy of growing abstraction.

**When to apply it**: this pattern is particularly useful for agents operating over **long time spans** (hours, days, weeks) where the accumulation of raw experiences without synthesis would lead to information overload.

### 6.4 The Recursive Planning Pattern

**Principle**: plans must be generated top-down, from general to specific, with re-planning capability in response to unforeseen events.

**Practical application**:
- Generate first a **high-level plan** (main objectives).
- Recursively decompose into **sub-plans** with increasing granularity.
- Implement a **reaction mechanism**: the agent perceives the environment and decides whether to follow the plan or react.
- Re-planning does not require redoing the entire plan: it is enough to replace the current segment with new actions.

**Critical trade-off**: too detailed planning wastes compute and risks being invalidated by unforeseen events; too generic planning does not provide sufficient guidance for action. Park's recursive approach elegantly balances this trade-off.

### 6.5 The Perception-Action Pattern

**Principle**: the fundamental cycle of the agent is environmental perception, memory retrieval, reasoning, action.

**Practical application**:
- At each step, the agent must: (1) perceive the current state of the environment, (2) retrieve relevant memories, (3) decide whether to follow the plan or react, (4) execute an action, (5) memorize the experience.
- Perception must be **filtered and selective** -- agents cannot process everything that happens in the environment, exactly like humans.

### 6.6 Considerations on Costs and Scalability

A fundamental pragmatic aspect: the original simulation of 25 agents for 2 simulated days required **thousands of dollars** in API costs and **several days of real time**. This implies:

- **LLM call optimization**: not every decision requires complete reasoning. Implement decision hierarchies where routine actions are handled by cheaper models or simple rules, reserving powerful models for complex decisions.
- **Intelligent caching**: frequently accessed memories and stable reflections can be cached to reduce API calls.
- **Batch processing**: where possible, aggregate requests to reduce overhead.
- **Local models**: consider the use of local open-source models for routine operations, reserving cloud models for the most complex tasks.

---

## 7. Impact, Evolution, and Resources

### Academic and Industrial Impact

The "Generative Agents" paper has had a profound and rapid impact on the research community and industry:

- **Best Paper Award at UIST 2023**, the most prestigious recognition in the main conference for software and user interface technology.
- It catalyzed an entire **new area of research** on autonomous agents based on LLMs, inspiring dozens of derivative projects.
- It provided a **shared vocabulary** (memory stream, reflection, retrieval) that has become standard in the agent builders community.
- It demonstrated the feasibility of **computational social simulation** as a tool for the social sciences and policy design.

### Research Evolution: From 25 to 1,000 Agents

Park's subsequent work has significantly expanded the scope of the research. In the paper "Generative Agent Simulations of 1,000 People" (arXiv:2411.10109, November 2024), Park and colleagues:

- Interviewed **1,052 real people** for two hours each.
- Created generative agents based on these qualitative interviews.
- Demonstrated that the agents replicate the responses to the **General Social Survey** with an accuracy of **85%** relative to the participants' own ability to replicate their responses two weeks later.
- Showed that the agents reduce **accuracy bias** between racial and ideological groups compared to agents endowed only with demographic information.

This work paves the way for **population-scale simulations** to test public policies, predict social reactions to interventions, and understand collective dynamics.

### Trajectory: From Social Simulacra to Generative Agents

The Generative Agents project was not born in a vacuum. Park's previous work, **"Social Simulacra: Creating Populated Prototypes for Social Computing Systems"** (UIST 2022), had already explored the use of LLMs to generate realistic social interactions in prototypes of digital platforms. Social Simulacra took as input the description of a community design (goals, rules, member personas) and produced simulated behaviors, including posts, responses, and antisocial behaviors. Generative Agents represent the natural evolution of this approach, moving from textual interactions to **embodied agents** in a spatial world with memory and cognition.

### Teaching

Park has also brought his research into teaching, instructing the course **CS 222: AI Agents and Simulations** at Stanford (Fall 2024), training the next generation of researchers and engineers on the techniques and principles of generative agents.

### Technical Resources

**Papers and Publications**:
- Original paper: [arXiv:2304.03442](https://arxiv.org/abs/2304.03442)
- Published version: [ACM Digital Library, UIST '23](https://dl.acm.org/doi/10.1145/3586183.3606763)
- Paper on 1000 agents: [arXiv:2411.10109](https://arxiv.org/abs/2411.10109)
- Social Simulacra: [arXiv:2208.04024](https://arxiv.org/abs/2208.04024)

**Code and Implementation**:
- GitHub repository: [joonspk-research/generative_agents](https://github.com/joonspk-research/generative_agents)
- License: Apache 2.0
- Requirements: Python 3.9.12, OpenAI API, Django
- Includes pre-configured simulations (3 agents and 25 agents)
- Editable maps with Tiled editor

**Researcher Profile**:
- Personal site: [joonsungpark.com](https://www.joonsungpark.com/)
- Google Scholar: [Joon Sung Park](https://scholar.google.com/citations?user=Y4Oc0cMAAAAJ)
- Academic CV: [joonsungpark.com/cv](https://www.joonsungpark.com/cv)

### Evaluation and Ablation Study

The system evaluation was conducted through **human evaluators** who judged the credibility of agent behavior on five dimensions:

1. **Self-knowledge** -- how much the agent demonstrates knowledge of itself
2. **Memory retrieval** -- how appropriate the retrieved memories are
3. **Plan generation** -- how realistic and coherent the plans are
4. **Reaction** -- how credible the reactions to unforeseen events are
5. **Reflection** -- how correctly reflections synthesize experiences

The **ablation study** confirmed that the complete architecture (observation + planning + reflection) produces the most credible behavior. Key results:

- **Without reflection**: agents lose the ability to synthesize experiences and make coherent decisions in the long term. Reflection proved essential for proper cognitive functioning.
- **Without planning**: agents become purely reactive, losing the ability to pursue long-term goals.
- **Observation only**: even without planning and reflection, the system produces "decent" results, thanks to the power of the memory stream and retrieval. But the overall quality is significantly inferior to the complete architecture.

### Ethical Considerations and Limitations

Park and the co-authors also addressed the ethical implications of their work:

- **Risk of parasocial relationship formation**: users who interact at length with credible agents could develop inappropriate emotional bonds.
- **Risk of surveillance and manipulation**: the ability to simulate the behavior of real individuals (as demonstrated in the paper on 1000 people) raises questions about privacy and consent.
- **Computational limits**: high costs currently limit the accessibility of the technology.
- **Model bias**: agents inherit the biases of the underlying language model, which can distort simulated behaviors.

---

## Synthesis: The Fundamental Contribution

Joon Sung Park's work on Generative Agents has established an **architectural paradigm** for autonomous agents based on LLMs that continues to profoundly influence the field. The three pillars -- **persistent memory with intelligent retrieval**, **periodic reflection for high-level synthesis**, and **recursive planning with reactivity** -- represent a reference framework for anyone building agent systems.

The demonstration that complex social behaviors can emerge from agents endowed with these cognitive abilities, without explicit programming of the behaviors themselves, is one of the most powerful insights of contemporary research on artificial intelligence. It is not just a technical result: it is a profound reflection on how individual cognition -- memory, reflection, planning -- can generate social complexity when autonomous agents interact in a shared environment.

For agent builders, the message is clear: **memory is not optional, it is a prerequisite**. Reflection is not a luxury, it is a necessity. And planning must be flexible, hierarchical, and context-sensitive. These principles, emerging from the simulation of a small virtual town, are universally applicable to any agent system aspiring to credible and autonomous behavior.
