# Andrej Karpathy: Software 3.0, Context Engineering and the New Era of AI Agents

## 1. Profile and Background

### Academic Background

Andrej Karpathy (born October 23, 1986 in Bratislava, Czechoslovakia) is one of the most influential artificial intelligence researchers of his generation. Having emigrated to Toronto at the age of 15, he built a top-level academic path: bachelor's degree in Computer Science and Physics at the University of Toronto (2009), master's at the University of British Columbia (2011) where he worked on physical simulation of figures, and finally a PhD at Stanford (2015) under the supervision of Fei-Fei Li, focusing on the intersection between Natural Language Processing and Computer Vision with deep learning models.

At Stanford, Karpathy created and taught as primary instructor the course CS 231n: Convolutional Neural Networks for Visual Recognition, which became one of the most attended courses at the university, growing from 150 students in 2015 to 750 in 2017. This course trained an entire generation of engineers and researchers in the field of deep learning and has remained a global reference point in AI education.

### OpenAI: Founding Member

Karpathy is a founding member of OpenAI, where he worked as research scientist from 2015 to 2017. During this period he contributed to the fundamental research that would later lead to the GPT family of models. He briefly returned to OpenAI in February 2023, only to leave again in February 2024. His experience inside OpenAI gave him a unique and privileged perspective on the evolution of Large Language Models, from the pure research phase to their transformation into consumer products with ChatGPT.

### Tesla: Director of AI and Autopilot

In June 2017 Karpathy became Director of Artificial Intelligence at Tesla, reporting directly to Elon Musk. For five years he led the team responsible for Autopilot and computer vision applied to autonomous driving. This experience was fundamental in shaping his thinking on "partial autonomy" systems -- a concept he would later transfer to his vision of LLM-based software. He left Tesla in July 2022 after a sabbatical period.

The Tesla experience taught him that the future of AI is not in total and immediate autonomy, but in a gradient of autonomy where human and machine collaborate with increasing levels of delegation -- a principle he applies today to his philosophy on agent engineering.

### Eureka Labs and AI-Native Education

In 2024 Karpathy founded Eureka Labs, an AI-native educational platform. The first course offered is LLM101n: Let's Build A Storyteller, which guides students in building an LLM from scratch using Python, C, and CUDA, culminating in a web application similar to ChatGPT to generate and illustrate short stories. The approach is revolutionary: the human teacher designs the course materials, but these are supported, expanded, and scaled by an AI teaching assistant optimized to guide students.

In parallel, his YouTube channel has become a global reference point for AI education: his video "How to Build GPT" has surpassed 4 million views, and "Deep Dive into LLMs like ChatGPT" offers an in-depth understanding of the foundations of language models. In 2020 he was named one of the MIT Technology Review Innovators Under 35.

---

## 2. Software 3.0: The Great Software Trilogy

### The Evolutionary Framework

Karpathy's most significant intellectual contribution to the applied AI debate is his taxonomy of software evolution into three distinct paradigms, organically presented at the YC AI Startup School in June 2025 with the talk "Software Is Changing (Again)". Karpathy himself commented: "Fair to say that software is changing quite fundamentally again. LLMs are a new kind of computer, and you program them in English."

### Software 1.0: Explicit Code

The first paradigm is the classic one of traditional programming. The programmer writes explicit instructions in languages like C++, Python, or Java. Every system behavior must be coded manually, every edge case anticipated, every rule specified. The "program" is a deterministic sequence of instructions that the computer faithfully executes. Software complexity grows linearly (or worse) with problem complexity.

Paradigmatic example: to build a sentiment analysis system, one would have to write hundreds of linguistic rules, dictionaries of positive and negative words, handle negations, sarcasm, context -- all manually. The code is fragile, difficult to maintain, and difficult to scale to new domains.

### Software 2.0: Neural Networks

The second paradigm, which Karpathy already defined in his celebrated post "Software 2.0" of 2017, shifts the focus from writing explicit rules to training neural networks on data. Instead of coding the logic, the programmer defines a network architecture, collects a training dataset, and optimizes the weights through backpropagation. The "program" is no longer human-readable code, but a matrix of numerical weights.

The Software 2.0 revolution was profound: image recognition, machine translation, and speech recognition systems went from systems based on artisanal rules to neural networks trained end-to-end with markedly superior performance. The programmer no longer writes the rules -- they "discover" them through optimization.

### Software 3.0: Natural Language as Code

The third paradigm, the current one, represents an even more radical conceptual leap. With the advent of Large Language Models, natural language becomes the programming interface. The programmer does not write code or train models: they formulate instructions in English (or any language) and the model executes them. As Karpathy illustrates with the sentiment analysis example, instead of writing hundreds of lines of code or collecting thousands of labeled examples, the "programmer" can simply tell the machine: "Analyze the sentiment of this text."

The barrier to entry collapses: anyone with an idea can create software without years of training in programming. Confirming this trend, Y Combinator reported that 25% of startups in the Winter 2025 batch had codebases generated 95% by AI.

Karpathy synthesized this evolution with a powerful statement: "It's the 1960s of operating systems again -- but this time, billions of people have access overnight." We are at the beginning of a new computing era, but with a crucial difference: access is democratic from day one.

---

## 3. Context Engineering: The New Fundamental Skill

### Beyond Prompt Engineering

In mid-2025, Karpathy took a clear position in the terminological debate that ran through the AI community. In a post on X he declared: "+1 for 'context engineering' over 'prompt engineering'. People associate prompts with short task descriptions you'd give an LLM in your day-to-day use. When in every industrial-strength LLM app, context engineering is the delicate art and science of filling the context window with just the right information for the next step."

This distinction is not just semantic -- it is conceptual. Prompt engineering evokes the image of a user formulating an intelligent question to a chatbot. Context engineering, instead, is an engineering discipline that concerns the design of the entire context that an LLM "sees" at the moment of inference.

### How Context Engineering Works

Context engineering operates on multiple simultaneous levels:

**Selection of relevant context:** In an industrial LLM application like Cursor, the system does not simply send the user's request to the model. It must select which code files to include, which documentation is relevant, which recent errors to provide, which conversation history to maintain. All of this must fit in the context window (the model's "RAM") without exceeding it.

**Multi-call orchestration:** Serious LLM applications do not work with a single call. Context engineering includes the design of pipelines where multiple LLM calls are concatenated, each with a different and optimized context. A first call might classify the type of request, a second retrieve information from the vector database, a third generate the final response.

**Cost/performance balancing:** Every token in the context window has a computational and economic cost. The context engineer must decide what to include and what to exclude, balancing response quality with budget and latency constraints. Too much context degrades performance (the model gets "distracted"), too little limits it (the model lacks sufficient information).

**Formatting and structuring:** It is not enough to select the right information -- it must be presented in the optimal format. The context engineer decides whether to use XML tags, markdown, JSON, or free text. Decides the order of information (the model is more attentive at the beginning and end of the window). Decides how to structure system instructions with respect to data.

### Context Engineering in Real Applications

Karpathy identified Cursor as the paradigmatic example of an application that excels at context engineering. Cursor is not just a "wrapper around an LLM" -- it is a sophisticated system that:

- Indexes the entire codebase for rapid retrieval of relevant files
- Maintains awareness of the current context (open file, cursor position, errors in the terminal)
- Autonomously manages the trade-off between models (fast for autocomplete, powerful for refactoring)
- Implements an "autonomy slider" that allows the user to calibrate the level of delegation

The success of Cursor demonstrates Karpathy's thesis: the value is not in the model itself (which is shared), but in the engineering of the context that surrounds the model. LLM applications do not win by luck or magical prompts -- they win by meticulously assembling the context around the model's queries.

---

## 4. Agentic Engineering: The Post-Vibe-Coding Paradigm

### The Origin of Vibe Coding

In February 2025, Karpathy coined the term "vibe coding" with a post that went viral: "There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists." He described an approach where you press "Accept All" always, you no longer read the diffs, and when you receive an error you copy and paste it into the chatbot without comments. "It's not really coding -- I just see stuff, say stuff, run stuff, and copy paste stuff, and it mostly works."

The term hit the mark: it was picked up by the New York Times, Ars Technica, The Guardian, and the entire tech community. It captured perfectly the spirit of a moment in which LLMs were becoming powerful enough to generate functional applications starting from natural language descriptions.

But Karpathy always specified that vibe coding was suitable for "throwaway weekend projects". AI-generated code can contain hidden bugs, inefficient logic, poor architectural design, and become difficult to maintain, audit, and scale without supervision.

### The Transition to Agentic Engineering

In February 2026, Karpathy introduced a new term: "agentic engineering". The distinction is fundamental. If vibe coding is a casual and experimental approach, agentic engineering is a rigorous professional discipline where AI agents plan, use tools, and perform real work under the supervision of the human engineer.

As Karpathy explained: 99% of the time you are not writing code directly, but commanding agents and acting as supervisor. It is an activity that involves art, science, and professional skills -- not a "vibes only" approach.

### The December 2024 Inflection Point

Karpathy described a precise moment of change: "In December, something flipped where I went from 80-20 of writing code myself versus delegating to agents to like 20-80." Around December 2024-2025, agents crossed a "coherence threshold" -- they became capable of completing entire tasks, not just generating code fragments.

This transition changed the fundamental verb of the developer's work. As Karpathy said: "Code's not even the right word anymore. But I have to express my will to my agents for 16 hours a day. Manifest." The new verb is not "to program" but "to manifest" -- to express one's intention and delegate its execution.

### Claude Code as Paradigm

In his 2025 Year in Review, Karpathy identified Claude Code as the first convincing demonstration of a functional LLM agent. The characteristics that make it paradigmatic:

- **Iterative tool use:** the agent does not generate a single response but executes cycles of reasoning, command execution, output reading, and iteration
- **Local environment:** runs on the user's machine with access to their private environment, data, and context
- **Resident spirit:** the AI becomes a "spirit/ghost that lives on your computer" rather than an external service

This represents an architectural shift: from AI as a centralized cloud service, to AI as a local agent with access to the user's real context.

### The Problem of "Jaggedness" (Jagged Intelligence)

One of Karpathy's most important conceptual contributions concerns the radically uneven nature of LLM capabilities. In his analysis he described LLMs as "genius polymath and confused grade schooler, seconds away from getting tricked" -- simultaneously a polymath genius and a confused elementary school student, seconds away from being deceived.

This "jaggedness" is a structural trait, not a bug. LLMs are "ghosts", not "animals": they are optimized for text imitation and for specific reward domains, not for survival. Consequently, they can solve complex distributed systems problems but fail on trivial tasks. Karpathy described this experience: "I simultaneously feel like I'm talking to an extremely brilliant PhD student... and a 10-year-old."

For the agentic engineer, this awareness is crucial: one must intercept the failure moments before they propagate into hours of debugging.

---

## 5. The LLM as "New CPU": The Operating System Analogy

### The LLM OS Architecture

One of Karpathy's most powerful and influential metaphors is his description of LLMs as a new type of operating system. In a celebrated post on X he wrote: "With many pieces dropping recently, a more complete picture is emerging of LLMs not as a chatbot, but the kernel process of a new Operating System." He then specified the "specs" of this new OS:

- **LLM (e.g. GPT-4 Turbo):** equivalent of the processor (CPU) -- the central unit that executes all commands. Like a multi-core processor, it can handle batches of requests in parallel
- **Context window:** equivalent of RAM -- the working memory containing the current instructions and the data necessary for the operation
- **Tools and plugins:** equivalents of peripherals and drivers -- access to the Internet, calculator, code interpreter, file system
- **Embedding store:** equivalent of the file system -- long-term storage based on vector representations

### The Prompt as Program

In Software 3.0, natural language prompts become programs. An LLM can be thought of as a general-purpose CPU that executes "programs" expressed in natural language. It has a working memory (the context window), can use tools and retrieve information as instructed -- analogous to how an OS manages memory and peripherals.

This analogy has profound implications for how we think about programming:

- **Compilation:** the prompt is like source code that the model "compiles" into actions
- **Debugging:** errors are not bugs in the code but ambiguities or gaps in the context
- **Optimization:** optimizing an LLM system means optimizing the context, not the code
- **Testing:** testing an LLM system requires evaluation frameworks, not traditional unit tests

### The Platform Ecosystem

Karpathy observed that the industry is taking on a structure similar to that of traditional operating systems: just as Windows, macOS, and Linux defined the PC era, GPT, Claude, Gemini, and Llama/Mistral are defining the LLM era. Each "OS" has its application ecosystem, its strengths, and its community.

Karpathy's prediction is that open source will succeed like Linux: it will handle 80% of the simpler use cases, while frontier (proprietary) models will tackle the most difficult problems. This creates a stratified market where value is distributed across the infrastructure layer (the models), the platform layer (the "OS" that orchestrates the models), and the application layer (vertical solutions).

### LLM as Economic Utility

Karpathy also presents a fundamental economic analogy. Training an LLM requires a massive capital investment (capex) comparable to building an electrical grid or a semiconductor manufacturing plant. Use then occurs through APIs with usage-based pricing, similar to an electric bill with cost per token. This economic structure suggests that LLMs will become a fundamental utility of the digital economy, like electricity or the Internet.

---

## 6. Practical Advice for Those Building with LLMs

### The Autonomy Slider

From his experience at Tesla with Autopilot, Karpathy derived the concept of "autonomy slider" -- one of the most practical design principles for LLM applications. The idea is simple but powerful: every AI application should offer the user gradual control over the system's level of autonomy.

At one extreme of the slider, the AI offers minimal suggestions (e.g. code autocomplete). At the other extreme, the AI operates in full autonomy (e.g. generates an entire application). The user chooses the appropriate level based on the criticality of the task, their trust in the system, and the need for control.

Cursor implements this principle: you can use Tab autocomplete (low autonomy), Edit mode (medium autonomy), or Agent mode (high autonomy). Each level is appropriate for different situations.

### Practical Guide to Agentic Engineering

From the analysis of his daily workflow, Karpathy distilled concrete recommendations:

1. **Identify a manual task with verifiable output and delegate it entirely.** Don't start with ambiguous tasks -- choose something where you can measure success objectively.

2. **Maximize token throughput rather than serial task completion.** Launch multiple agents in parallel on independent tasks. Don't wait for one agent to finish before starting another.

3. **Remove yourself from the loop once delegation has begun.** The most common error is to micromanage the agent. Once delegated, let it work and review the final result.

4. **Build APIs before UIs; document for machines, not just for humans.** In a world where agents consume software, machine-readable interfaces (APIs, markdown, curl commands) become more important than graphical interfaces.

5. **Accept jaggedness and design for it.** Agents will fail in surprising ways. Build systems that intercept failures, allow rollback, and keep the human in the loop for critical decisions.

### Building Software for Agents

A particularly forward-looking recommendation by Karpathy concerns the design of software that agents can use. As LLMs and AI agents become increasingly integrated into workflows, we must start building software that agents can consume, not just humans. This means:

- LLM-readable markdown documentation, not just PDFs formatted for humans
- Well-structured and self-documenting APIs
- Structured outputs (JSON, YAML) rather than text formatted for human reading
- Accessible CLI commands that agents can invoke

### The Importance of Private Data and Feedback Loops

In his Year in Review, Karpathy emphasizes that vertical LLM applications do not compete with foundation models but specialize them. The competitive advantage lies in private data, sensors, and domain-specific feedback loops. A college-level model can be transformed into a domain expert through context -- and this is where context engineering becomes strategic.

The applications that will win are those that:
- Have access to proprietary data that no foundation model possesses
- Build feedback loops that continuously improve the quality of context
- Specialize the generic capabilities of the model into vertical competencies through context engineering

### Nano Banana and the LLM GUI

A less discussed but potentially revolutionary insight by Karpathy concerns the user interface of LLM applications. He proposed the concept of "LLM GUI" inspired by his Nano Banana project: the joint generation of text and images as a fundamental layer of the user interface. Recognizing that humans prefer visual and spatial consumption over pure text, this evolution is analogous to the emergence of GUIs in traditional computing -- a transition from text terminals to graphical interfaces that democratized access to computers.

---

## 7. Resources and References

### Karpathy's Writings and Blog

- **2025 LLM Year in Review** -- His most recent annual post, with complete analysis of the state of LLMs: key themes include RLVR, jagged intelligence, vibe coding, Claude Code, and the LLM GUI
  - https://karpathy.bearblog.dev/year-in-review-2025/
- **Personal site** -- Central hub with links to all his works and projects
  - https://karpathy.ai/
- **Original post on Software 2.0** (2017) -- The founding essay that introduced the concept of Software 2.0
  - https://karpathy.medium.com/software-2-0-a64152b37c35

### Talks and Presentations

- **"Software Is Changing (Again)"** -- Talk at the YC AI Startup School (June 2025), where he presented the complete Software 3.0 framework
  - https://www.ycombinator.com/library/MW-andrej-karpathy-software-is-changing-again
- **Latent Space Podcast: Software 3.0** -- In-depth conversation on the Software 3.0 vision
  - https://www.latent.space/p/s3

### Essential YouTube Videos

- **"How to Build GPT"** -- Educational video with over 4 million views, step-by-step construction of a GPT model
- **"Deep Dive into LLMs like ChatGPT"** -- In-depth analysis of LLM foundations
- **"Let's build GPT: from scratch, in code, spelled out"** -- Complete implementation of GPT with detailed explanations
- **YouTube Channel:** https://www.youtube.com/@AndrejKarpathy

### Fundamental X (Twitter) Posts

- **Definition of Context Engineering:** "+1 for 'context engineering' over 'prompt engineering'..." (June 2025)
  - https://x.com/karpathy/status/1937902205765607626
- **Definition of Vibe Coding:** "There's a new kind of coding I call 'vibe coding'..." (February 2025)
  - https://x.com/karpathy/status/1886192184808149383
- **LLM OS Vision:** "With many pieces dropping recently, a more complete picture is emerging of LLMs not as a chatbot, but the kernel process of a new Operating System" (October 2023)
  - https://x.com/karpathy/status/1707437820045062561
- **LLM OS Specs:** "LLM OS. Bear with me I'm still cooking..." (November 2023)
  - https://x.com/karpathy/status/1723140519554105733

### Projects and Initiatives

- **Eureka Labs** -- AI-native educational platform founded in 2024
  - https://eurekalabs.ai/
- **LLM101n** -- Course to build an LLM storyteller from scratch
- **Personal vibe-coded projects:** menugen, llm-council, reader3, HN time capsule
- **Nano Banana** -- Experiment on joint text+image generation for LLM GUI

### Third-Party Analyses and Insights

- **NextBigFuture: Software 3.0 By Karpathy** -- Detailed analysis of the YC presentation
  - https://www.nextbigfuture.com/2025/06/software-3-0-by-karpathy.html
- **Analytics Vidhya: Andrej Karpathy on the Rise of Software 3.0** -- Technical breakdown of the talk
  - https://www.analyticsvidhya.com/blog/2025/06/andrej-karpathy-on-the-rise-of-software-3-0/
- **Sequoia Capital: Software Eating Software Eating Software** -- Analysis of economic impact
  - https://inferencebysequoia.substack.com/p/andrej-karpathys-software-30-software
- **The AI Corner: Andrej Karpathy AI Workflow Shift** -- In-depth on agentic engineering
  - https://www.the-ai-corner.com/p/andrej-karpathy-ai-workflow-shift-agentic-era-2026
- **Context Engineering Guide** -- Complete guide inspired by Karpathy's framework
  - https://www.promptingguide.ai/guides/context-engineering-guide
- **LangChain: Context Engineering for Agents** -- Practical implementation of Karpathy's concepts
  - https://blog.langchain.com/context-engineering-for-agents/

---

*Document compiled in March 2026. Andrej Karpathy continues to be one of the most influential voices in defining the future of software engineering in the era of artificial intelligence. His trajectory -- from academic research at Stanford, to industrial leadership at OpenAI and Tesla, to education and public reflection -- gives him a unique perspective that combines scientific rigor, practical experience, and communicative ability.*
