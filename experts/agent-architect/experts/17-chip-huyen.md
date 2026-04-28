# Chip Huyen -- AI Engineering and AI Systems in Production

## 1. Profile and Background

### Academic and Professional Path

Chip Huyen is a writer and computer scientist of Vietnamese origin, who grew up in a small rice paddy village in rural Vietnam. Despite humble origins, she has built an extraordinary career that spans the academic world, the technology industry and scientific outreach.

She studied and graduated from **Stanford University**, where she did not limit herself to the role of student. In 2017 she created and taught the course **"TensorFlow for Deep Learning Research"** (CS 20), which became one of the most attended courses in Stanford's computer science department. In 2021 she created a second course, **"Machine Learning Systems Design"** (CS 329S), focused specifically on the design and deployment of ML systems in production. These courses laid the foundations for her first book and trained a generation of ML engineers.

### Industrial Experience

Chip Huyen's industrial career covers some of the most influential companies in the AI landscape:

- **NVIDIA**: She worked as a core developer of **NeMo**, NVIDIA's open-source framework for building, training and fine-tuning large-scale language models. This experience gave her a deep understanding of the infrastructure necessary to train and serve foundation models at industrial scale.
- **Netflix**: She worked on ML tools for personalization and recommendation systems, addressing problems of scale, latency and real-time inference in a consumer context.
- **Snorkel AI**: She contributed to the development of tools for the programmatic management of training data, a theme that permeates all her subsequent work on the importance of data quality.
- **AI Infra Startup**: She founded and subsequently sold a startup focused on infrastructure for AI systems, acquiring direct experience in the entrepreneurial challenges of the sector.

### Author and Educator

Chip Huyen is the author of works in two very different areas. In Vietnamese she has published four books that have sold over **100,000 copies**. In the technical area, she has published two fundamental texts:

1. **"Designing Machine Learning Systems"** (O'Reilly, 2022) -- bestseller #1 on Amazon in the AI category, translated into over 10 languages.
2. **"AI Engineering"** (O'Reilly, 2025) -- the most read book on the O'Reilly platform since its publication.

Her technical blog at **huyenchip.com** is a reference resource for the ML/AI community, with in-depth articles on MLOps, ML tools, generative AI platforms, and agents. Among the most influential posts are "Building A Generative AI Platform" (July 2024), "Agents" (January 2025), and "Common pitfalls when building generative AI applications" (January 2025).

---

## 2. AI Engineering (O'Reilly 2025) -- Book Overview

### Context and Central Thesis

**"AI Engineering: Building Applications with Foundation Models"** (534 pages, published December 4, 2024) represents the codification of a new discipline. Huyen's central thesis is that the **model-as-a-service** approach has transformed AI from an esoteric discipline reserved for a few specialists into a powerful development tool accessible to all developers.

The book is aimed at AI application developers of all levels of experience, not just ML researchers. This reflects Huyen's conviction that AI engineering is a discipline distinct from traditional ML engineering, with its own competencies, tools and architectural patterns.

### Structure and Chapters

The book is organized into 10 chapters that cover the entire life cycle of an AI application:

**Chapter 1 -- Introduction**: Explores the emergence of AI engineering as a discipline driven by foundation models. Analyzes successful application patterns in consumer and enterprise sectors (coding, writing, education, conversational chatbots) and the evolution of the technology stack.

**Chapter 2 -- Understanding Foundation Models**: Examines fundamental design decisions in foundation model development: training data curation, transformer architecture, scaling laws, post-training techniques (supervised finetuning and preference finetuning). Emphasizes the probabilistic nature of sampling and its implications for consistency and hallucinations.

**Chapter 3 -- Evaluation Methodology**: Tackles the challenges of evaluation for foundation models. Covers language modeling metrics (perplexity, cross-entropy), approaches to evaluating open-ended responses (functional correctness, similarity score, AI-as-a-judge) and comparative evaluation methods.

**Chapter 4 -- Evaluate AI Systems**: Focuses on the practical construction of evaluation pipelines. Discusses criteria for model selection, public benchmarks and their limits, and the creation of private leaderboards aligned with the specific needs of the application.

**Chapter 5 -- Prompt Engineering**: Covers the anatomy of prompts, in-context learning mechanisms and best practices for instruction clarity. Addresses prompt injection vulnerabilities and defensive strategies. Defines prompt engineering as "human-AI communication".

**Chapter 6 -- RAG and Agents**: Examines Retrieval-Augmented Generation and agentic systems. RAG combines retrieval (term-based and embedding-based) with generation to improve accuracy. Agents use AI planning with access to tools and memory systems for complex multi-step tasks.

**Chapter 7 -- Finetuning**: Analyzes when and how to fine-tune models. Covers PEFT (Parameter-Efficient Fine-Tuning) methods, LoRA (Low-Rank Adaptation), model merging techniques and memory/computation trade-offs. Clearly distinguishes RAG vs finetuning use cases, emphasizing that "fine-tuning is easy, but obtaining data for fine-tuning is difficult".

**Chapter 8 -- Dataset Engineering**: Tackles dataset creation as a critical challenge. Discusses quality, coverage and quantity as fundamental criteria. Explores the generation of synthetic data and the evaluation of data generated by AI.

**Chapter 9 -- Inference Optimization**: Details optimization techniques to reduce latency and costs. Covers efficiency metrics (time to first token, time per output token), model-level techniques (quantization, distillation, attention optimization) and service-level strategies (batching, parallelism, KV cache management, prompt caching).

**Chapter 10 -- AI Engineering Architecture and User Feedback**: Synthesizes previous concepts into a complete application architecture. Discusses monitoring/observability for AI-specific failure modes and the design of conversational feedback mechanisms for continuous improvement through the "data flywheel".

### Cross-Cutting Themes

Three themes run through the entire book: (a) evaluation is fundamental and must be present at every stage; (b) data quality determines system quality; (c) simplicity must precede complexity.

---

## 3. Agents in Production -- Patterns for Deploy, Monitoring, Testing

### Definition and Architecture of Agents

According to Huyen's definition, an agent is "anything that perceives its environment and acts on it". In the AI context, an agent is characterized by the environment in which it operates and the set of actions it can take. The AI model acts as the brain of the agent, leveraging its tools and feedback from the environment to plan how to achieve a goal.

An agent's architecture is composed of three elements:

1. **Plan Generator**: creates sequences of actions to achieve the goal.
2. **Plan Validator**: evaluates the feasibility of the plan before execution.
3. **Plan Executor**: implements validated plans.

This decoupled approach prevents the costly execution of flawed plans before validation confirms their feasibility.

### Tool Categories

Huyen classifies the tools available to agents into three categories:

**Knowledge Augmentation Tools**: text and image retrievers, SQL executors, web browsing (to prevent model staleness), APIs for internal and public information.

**Capability Extension Tools**: calculators (to overcome mathematical weaknesses), code interpreters, translators, timezone converters, multimodal transformations (text-to-image).

**Write Actions**: database modifications, email replies, financial transfers. These enable automation workflows but require rigorous safety safeguards.

Research demonstrates that agents augmented with tools significantly outperform base models. The **Chameleon** framework (GPT-4 with 13 tools) improved benchmarks by 11.37-17%.

### Control Flow Patterns

Agent control flows include:

- **Sequential**: A generates output, which becomes the input of B (tight dependencies).
- **Parallel**: simultaneous execution of independent tasks.
- **Conditional**: if/then routing based on intermediate outputs.
- **Loop**: repetition until a condition is satisfied.

The **ReAct** (Reasoning + Acting) pattern is particularly relevant: it interleaves reasoning, action and observation in a structured format until task completion. The model reasons about what to do, executes an action, observes the result, and repeats the cycle.

### Reflection and Error Correction

Reflection is a critical mechanism that identifies and corrects errors in multiple phases:

- **Post-query**: evaluation of the feasibility of the request.
- **Post-planning**: validation of the generated plan.
- **Post-action**: evaluation of each individual step.
- **Post-execution**: verification of goal achievement.

The **Reflexion** framework separates evaluation and self-reflection modules, allowing agents to learn from failures and generate iteratively improved plans. The trade-off is clear: reflection increases token costs and latency, but provides significant improvements in performance.

---

## 4. Evaluation and Testing -- How to Evaluate Agents

### The Centrality of Evaluation

For Huyen, "everything that follows in AI engineering -- prompting, memory, finetuning, inference -- depends on reliable evaluation". Evaluation is not a one-off activity but a continuous process that must permeate every stage of development and deployment.

### Foundation Model Metrics

The book covers different levels of metrics:

**Language Modeling Metrics**: perplexity and cross-entropy measure how well the model predicts the next token. They are useful for comparing models but do not capture the quality perceived by the user.

**Metrics for open-ended responses**:
- **Functional correctness**: does the model produce output that satisfies the functional requirements?
- **Similarity score**: how similar is the output to a reference response?
- **AI-as-a-judge**: an AI model evaluates the quality of the output of another model.

**Comparative metrics**: direct comparison between outputs of different models on the same inputs.

### AI-as-a-Judge: Power and Limits

The AI-as-a-judge approach is central to Huyen's framework. An LLM model is used to evaluate the outputs of another model, combining the scalability of automatic methods with the detailed and context-sensitive reasoning typical of expert evaluations.

However, Huyen warns against exclusive reliance on this approach. AI judges depend on the underlying model, on the quality of the prompt, and on the specificity of the use case. Limitations include systematic biases (preference for longer or verbose responses), inability to capture domain-specific nuances, and vulnerability to manipulation.

### Systematic Human Evaluation

The best teams conduct **daily human evaluation** on 30-1,000 examples to:

- Validate the accuracy of AI judges.
- Understand real user behaviors.
- Detect changes in patterns that algorithms might not capture.

Huyen emphasizes that "manual data inspection probably has the highest value-to-prestige ratio of any activity in machine learning". This practice, often underestimated because perceived as unsophisticated, is actually fundamental to building reliable systems.

### Public Benchmarks and Their Limits

Public benchmarks such as MMLU, BigBench, TruthfulQA, GSM8K, HellaSwag and ARC measure reasoning, factual accuracy, math, common sense and instruction-following. However, Huyen recommends the creation of **private leaderboards** aligned with the specific needs of the application, rather than blindly relying on public rankings that may not reflect real performance in your context.

### Specific Metrics for Agents

For agents, evaluation metrics include:

- **Valid plan generation rate**: how many generated plans are actually executable.
- **Plans needed for completion**: how many attempts are needed to successfully complete a task.
- **Tool call validity rate**: percentage of tool calls that are syntactically and semantically correct.
- **Number of steps vs baseline**: efficiency relative to a known optimal solution.
- **Average cost per task**: computational and monetary resources consumed.

---

## 5. MLOps for Agents -- Infrastructure, Pipelines, Observability

### The GenAI Platform Architecture

After studying how companies deploy generative AI applications, Huyen has identified common components that form a generative AI platform architecture. The architecture progresses from a simple query-to-model flow to a complex system with five main additions: context enhancement, guardrails, model router and gateway, caching, and complex logic management.

### Observability: The Metrics-Logs-Traces Trinomial

**Metrics**: Track both system and model performance.
- *System metrics*: throughput, memory, uptime, latency (TTFT -- Time to First Token, TBT -- Time Between Tokens, TPOT -- Time Per Output Token).
- *Model metrics*: accuracy, toxicity, hallucination rate, retrieval quality.
- *Cost metrics*: API calls, token volumes, compliance with rate limits.
- Granular breakdown by user, release and prompt version.

**Logs**: The recommendation is clear: "log everything". This includes configurations, queries, outputs, and state changes of every component, with identifying tags and IDs for traceability.

**Traces**: End-to-end execution paths of requests showing "the entire process from when a user submits a query to when the final response is returned, including the actions taken by the system and the documents retrieved". Traces are essential for debugging complex pipelines with multiple components.

### Orchestration

Orchestrators handle the definition and chaining of components:

- They define available models, databases and actions.
- They specify pipeline sequences and data flow between steps.
- They support branching, parallel processing and error handling.

Huyen recommends a pragmatic approach: "Although it is tempting to start immediately with an orchestration tool, start building your application without one". The orchestrator should be introduced when complexity actually requires it, not as an initial choice.

The evaluation criteria for orchestrators include: integration and extensibility for different components, support for complex pipelines, and performance/scalability without hidden latency.

### Model Gateway

The gateway provides a unified interface for multiple models with:

- Centralized access control and cost management.
- Load balancing and fallback policies.
- Integrated logging, analytics and caching.

Among the cited examples: Portkey, MLflow, Kong and Cloudflare AI Gateway.

### Feedback Pipelines and Data Flywheel

Chapter 10 of the book emphasizes the importance of the "data flywheel" -- a virtuous cycle where user feedback continuously improves the model. The design of conversational feedback mechanisms is fundamental to capture both explicit signals (rating, thumbs up/down) and implicit ones (behavior, engagement, abandonment).

---

## 6. Failure Modes -- What Can Go Wrong with Agents in Production

### The Compound Errors Problem

The most critical challenge for multi-step agents is that of **compound errors**. Huyen provides eloquent math: if model accuracy is 95% per single step, over 10 steps the overall accuracy drops to **60%**, and over 100 steps it plummets to **0.6%**. This means that complex agents with many steps are intrinsically fragile and require explicit mitigation strategies.

### Planning Failures

Failures in the planning phase include:

- **Selection of invalid tools**: the model chooses a tool that does not exist or is not available.
- **Valid tools with incorrect parameters**: the call is syntactically correct but semantically wrong.
- **Valid tools with wrong parameter values**: the parameters are correct but the values are wrong.
- **Goal failures**: the plan does not reach the correct destination or violates specified constraints.
- **Reflection errors**: the model falsely claims it has succeeded.

### Hallucinations in Agents

Since both the action sequence and the associated parameters are generated by AI models, they can be **hallucinated**. An agent tasked with "cleaning the logs" might decide that the most efficient method is `rm -rf ./logs`. If the model hallucinates the path as `rm -rf /`, the entire server is lost.

When agents encounter errors, they often cannot distinguish between "I failed the task" and "the task is impossible", and frequently hallucinate a success message to the user just to close the loop.

### Tool Failures

- **Incorrect tool outputs**: wrong SQL queries, incorrect image captions.
- **Translation errors**: discrepancy between planned actions and executable commands.
- **Gaps in tool inventory**: necessary tools not available in the agent's set.

### Efficiency Problems

- **Excessive steps**: the task is completed but with many more steps than necessary.
- **High API costs**: resource consumption disproportionate to baselines.
- **Costly single actions**: individual steps that require excessive time or money.

### The 80/20 Gap

A recurring pattern observed by Huyen is the **80/20 gap** in production. Initial progress is dramatically rapid: LinkedIn reached "80% of the desired experience in 1 month" but required "4 additional months to surpass 95%". An AI sales assistant startup found that "going from 0 to 80% took the same time as from 80 to 90%". Initial demos are deceptive and lead to severely underestimating the difficulty of reaching production.

### Dangers of Over-Engineering

Teams often prematurely adopt agentic frameworks, vector databases, fine-tuning or semantic caching when simpler approaches would work. Premature abstractions can hide critical implementation details and introduce bugs deriving from changes in frameworks that you don't control.

---

## 7. Production Patterns

### 7.1 Guardrails and Safety

Guardrails represent a fundamental component of production architecture and apply both in input and output.

**Input Guardrails** address two primary risks:
- **PII leakage**: automatic detection of sensitive data with masking or removal options. They prevent personal information from being sent to the model.
- **Jailbreaking**: intent classification, phrase filtering, and anomaly detection to prevent attempts to bypass model restrictions.

**Output Guardrails** evaluate quality and handle failures:
- Detection mechanisms for empty responses, malformed output, toxicity, hallucinations, and brand-risky statements.
- **Failure handling**: retry logic (sequential or parallel), escalation to human operators, and fallback policies.

Huyen recognizes a critical trade-off: "While recognizing the importance of guardrails, some teams have decided not to implement guardrails because they can significantly increase latency". The decision must balance security and performance.

Huyen's key analogy on safety: "Just as you wouldn't give an intern the authority to delete your production database, you shouldn't allow an unreliable AI to start bank transfers".

### 7.2 Cost Management

Cost management is a critical concern and Huyen dedicates significant attention to optimization strategies.

**Three-level caching**:

1. **Prompt Cache**: stores overlapping text segments (especially system prompts) for reuse. According to Google's implementation, "input tokens in cache have a 75% discount compared to regular input tokens".

2. **Exact Cache**: stores complete query-response pairs using in-memory storage or databases with LRU/LFU eviction policies. Requires heuristics to determine which queries are cacheable.

3. **Semantic Cache**: allows the reuse of similar (not identical) queries via embedding similarity. Eugene Yan calls it "perhaps the most underestimated component", although its implementation presents significant complexities.

**Inference Optimization** (Chapter 9):
- **Quantization**: reduction of the precision of model weights to reduce memory and computation.
- **Distillation**: training of smaller models that replicate the behavior of larger models.
- **Attention optimization**: techniques such as Flash Attention to reduce computational cost.
- **Batching**: grouping of requests to improve throughput.
- **Parallelism**: distribution of inference across multiple GPUs/nodes.
- **KV Cache management**: optimization of the key-value cache to reduce computational redundancy.

**Model Router**: A router component can direct simpler queries to cheaper and faster models, reserving more expensive models for complex queries. This includes an intent classifier that directs queries to the appropriate solution and a next-action predictor for complex workflows.

### 7.3 Latency Optimization

The key latency metrics identified by Huyen are:

- **TTFT (Time to First Token)**: time required to generate the first token of the response.
- **TBT (Time Between Tokens)**: interval between the generation of consecutive tokens.
- **TPOT (Time Per Output Token)**: average time to generate each output token.

Latency optimization strategies include all the caching techniques mentioned above, plus:

- **Prompt caching** to reuse processing of common system prompts.
- **Response streaming** to improve user-perceived latency.
- **Smaller and specialized models** for specific tasks.
- **Distillation** to obtain similar performance with lighter models.
- **Smart batching** that balances throughput and latency.

Huyen notes the fundamental trade-off between accuracy and latency: reflection and validation mechanisms improve quality but increase latency. The choice depends on the application context.

### 7.4 Human-in-the-Loop

Human integration is a fundamental pattern for reliable AI systems. Huyen identifies several patterns:

**Routing to human operators**:
- Transfer of ambiguous queries to humans for clarification when the model does not have sufficient confidence.
- Escalation of conversations after detection of negative sentiment or exceeding turn limits.
- Transfer of queries that require human judgment due to suspicious patterns.

**Approval gates**:
- Mandatory manual approval for high-risk write actions such as deletion or modification of data, financial operations, and external communications.
- Definition of automation levels by type of action: some actions completely automatic, others that require supervision, others that require explicit approval.

**Assisted planning**:
- For complex tasks, human experts can provide high-level plans that the agent then executes.
- This significantly reduces the risk of planning errors for critical operations.

**Continuous evaluation**:
- Daily inspection of output samples to monitor quality.
- Perceptions of what constitutes good and bad output change as developers interact with more data, making manual inspection an evolutionary process.

---

## 8. Resources and References

### Books by Chip Huyen

- **"AI Engineering: Building Applications with Foundation Models"** (O'Reilly, December 2024, 534 pages, ISBN 9781098166304) -- The reference text for building applications based on foundation models. The most read book on the O'Reilly platform since launch.
- **"Designing Machine Learning Systems: An Iterative Process for Production-Ready Applications"** (O'Reilly, 2022) -- Bestseller #1 Amazon in AI, translated into over 10 languages. Fundamental for understanding ML systems in production.

### Key Blog Posts and Articles (huyenchip.com)

- **"Agents"** (January 7, 2025) -- In-depth analysis of AI agents: definition, architecture, tools, planning, reflection, failure modes and evaluation.
- **"Common pitfalls when building generative AI applications"** (January 16, 2025) -- The six most common errors in building GenAI applications.
- **"Building A Generative AI Platform"** (July 25, 2024) -- Reference architecture for GenAI platforms in production: guardrails, caching, observability, orchestration.
- **"What I learned from looking at 900 most popular open source AI tools"** (March 14, 2024) -- Overview of the open-source AI tools ecosystem.
- **"Building LLM applications for production"** (April 11, 2023) -- Practical guide to bringing LLM applications to production.
- **"Real-time machine learning: challenges and solutions"** (January 2, 2022) -- Challenges and solutions for real-time ML.
- **"Data Distribution Shifts and Monitoring"** (February 7, 2022) -- Monitoring changes in data distribution.

### Stanford Courses

- **CS 20: TensorFlow for Deep Learning Research** (2017) -- Pioneering course on the practical use of TensorFlow.
- **CS 329S: Machine Learning Systems Design** (2021) -- Design of ML systems for production.

### GitHub Repositories

- **chiphuyen/aie-book** -- Resources and supporting materials for the AI Engineering book, including chapter summaries and additional materials.

### Frameworks and Reference Tools Cited

- **NeMo (NVIDIA)** -- Framework for building and training language models.
- **Chameleon** -- Multi-tool framework for agents (13 tools, benchmark improvement 11-17%).
- **Gorilla** -- API selection among 1,645 options.
- **SWE-agent** -- Agent for coding environments.
- **Reflexion** -- Framework for self-reflection and learning from failures.
- **Portkey, MLflow, Kong, Cloudflare AI Gateway** -- Solutions for model gateways in production.
- **FAISS, ScaNN, ANNOY** -- Algorithms for approximate nearest neighbor (ANN) search in the RAG context.

### Podcasts and Interviews

- **Lenny's Podcast** -- "AI Engineering 101 with Chip Huyen (NVIDIA, Stanford, Netflix)" -- In-depth interview on career, approach to AI engineering, and lessons learned.

### Benchmarks and Evaluation Metrics

- **MMLU** -- Massive Multitask Language Understanding.
- **BigBench** -- Beyond the Imitation Game Benchmark.
- **TruthfulQA** -- Evaluation of factual accuracy.
- **GSM8K** -- Benchmark for mathematical reasoning.
- **HellaSwag** -- Evaluation of common sense.
- **ARC** -- AI2 Reasoning Challenge.

---

*Document compiled based on the book "AI Engineering" (O'Reilly, 2025), the huyenchip.com blog, and Chip Huyen's public research. The positions expressed reflect the original work of the author and have been synthesized for the context of this project.*
