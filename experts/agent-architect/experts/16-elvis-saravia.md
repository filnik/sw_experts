# Elvis Saravia and the Prompt Engineering Guide

## The Definitive Guide to Prompt Engineering for LLMs and AI Agents

---

## 1. Profile and Background

### Who is Elvis Saravia

Elvis Saravia is an artificial intelligence researcher, educator and entrepreneur who has built one of the most influential resources in the field of prompt engineering. With a Ph.D. in computer science, specializing in Natural Language Processing (NLP) and large language models, Saravia has accumulated leading experience working with some of the most advanced AI teams in the world.

His professional path includes a significant role at **Meta AI**, where he operated as Technical Product Marketing Manager, collaborating directly with excellence teams such as **FAIR** (Facebook AI Research), **PyTorch** and **Papers with Code**. In this position, he contributed to the co-creation of **Galactica**, a Large Language Model developed for scientific research. Before the Meta experience, Saravia held the role of Education Architect at **Elastic**, where he developed technical curriculum and training courses on technologies such as Elasticsearch, Kibana and Logstash.

### DAIR.AI: Democratizing AI

Saravia is co-founder of **DAIR.AI** (Democratizing AI Research, Education, and Technologies), an organization dedicated to democratizing research, education and technologies in the field of artificial intelligence. At DAIR.AI, he leads all AI research, education and engineering activities. The organization stands out for its mission to make advanced AI knowledge accessible to a global audience, through open-source resources, structured courses and educational materials.

### Open-Source Contributions

Saravia's impact in the open-source community is notable. His GitHub profile (omarsar) has over 4,100 followers and 149 repositories, with projects that have collected tens of thousands of stars:

- **Prompt-Engineering-Guide** -- 72,300+ stars, the main resource for prompt engineering
- **ML-YouTube-Courses** -- 17,100+ stars, a curation of ML courses from YouTube
- **ml-visuals** -- 17,000+ stars, reusable figures and templates for scientific writing
- **ML-Course-Notes** -- 6,400+ stars, structured notes from machine learning courses
- **Mathematics-for-ML** -- 5,900+ stars, resources for the mathematical foundations of ML
- **ML-Notebooks** -- 3,400+ stars, Jupyter notebooks for machine learning projects

In addition to GitHub projects, Saravia maintains the **NLP Newsletter**, a weekly publication that synthesizes the latest trends, papers, tools and best practices in the AI field. He also offers strategic consulting to important AI companies in areas spanning go-to-market strategy, product development, research and commercial expansion.

---

## 2. Prompt Engineering Guide: Resource Overview

### General Overview

The **Prompt Engineering Guide** is a complete and open-source educational resource that covers the "development and optimization of prompts to use language models efficiently" for a wide range of applications and research use cases. Published under MIT license and hosted on GitHub with the support of the DAIR.AI organization, the project has reached an extraordinary milestone: over **3 million students** by January 2024, with 193 contributors, 1,589 commits and 7,700 forks.

The guide is built with the Next.js framework and supports **13 languages**, making it accessible to a global community of researchers, developers and AI professionals.

### Structure and Organization

The Prompt Engineering Guide is organized into main sections that cover the entire spectrum of prompt engineering:

**Introduction and Fundamentals:**
- LLM settings (temperature, top_p, max tokens)
- Prompting basics and building blocks
- Design tips and practical examples

**Prompting Techniques (18 documented techniques):**
- Zero-shot Prompting
- Few-shot Prompting
- Chain-of-Thought (CoT)
- Meta Prompting
- Self-Consistency
- Generated Knowledge Prompting
- Prompt Chaining
- Tree of Thoughts (ToT)
- Retrieval Augmented Generation (RAG)
- Automatic Reasoning and Tool-use (ART)
- Automatic Prompt Engineer (APE)
- Active-Prompt
- Directional Stimulus Prompting
- Program-Aided Language Models (PAL)
- ReAct
- Reflexion
- Multimodal CoT
- Graph Prompting

**AI Agents:**
- Introduction to agents
- Agent components
- AI Workflow vs AI Agents
- Context Engineering for AI agents
- Function Calling
- Deep Agents

**Additional Sections:**
- Application guides (code generation, classification, reasoning)
- Prompt Hub (repository of curated prompts)
- Models (specific guides for ChatGPT, GPT-4, LLaMA, Mistral, Gemini, Phi-2, OLMo)
- Risks and misuse (adversarial prompting, bias, factuality)
- LLM research findings
- Resources (papers, tools, notebooks, datasets)

### Coverage and Audience

The guide addresses two main audiences with distinct needs:

- **Researchers**: focused on safety, capability enhancement and performance on complex tasks
- **Developers**: oriented towards practical implementation of techniques and integration into systems

The examples in the resource predominantly use `gpt-3.5-turbo` via the OpenAI Playground with default settings (temperature=1, top_p=1), although results may vary across comparable models.

---

## 3. Fundamental Prompting Techniques

### 3.1 Zero-shot Prompting

**Zero-shot prompting** represents the most elementary form of interaction with an LLM: an instruction is provided to the model without any demonstrative example. The model must rely exclusively on its pre-trained knowledge and understanding of the instruction to generate an appropriate response.

This technique works because large language models have been trained on enormous amounts of data and have developed the ability to follow instructions as a result of instruction tuning. The effectiveness of zero-shot prompting strongly depends on the clarity and specificity of the provided instruction, as well as on the complexity of the required task.

**Typical example:**
```
Classify the following text as positive, negative or neutral.
Text: "The new update significantly improved performance."
Sentiment:
```

Zero-shot is ideal for relatively simple tasks such as classification, translation, summarization and answering direct questions. However, for tasks that require complex reasoning or specific domain knowledge, its performance may be insufficient.

### 3.2 Few-shot Prompting

**Few-shot prompting** extends zero-shot by providing the model with a small number of demonstrative examples (typically 2-5) that illustrate the format and type of desired response. This technique leverages the in-context learning capabilities of LLMs, allowing the model to understand the expected pattern without needing fine-tuning.

The quality of the examples is crucial: they must be representative, diversified and consistent with the required task. The format of the examples implicitly establishes constraints on the type of output, the length of the response and the style of reasoning.

**Critical aspects of few-shot prompting:**
- The selection of examples significantly influences performance
- The order of examples can alter results
- Examples that are too similar to each other reduce generalizability
- The optimal number of examples varies based on task complexity

Few-shot prompting represents a balance between the simplicity of zero-shot and the complexity of more advanced approaches, proving particularly effective for medium-complexity tasks that benefit from contextual guidance.

### 3.3 Chain-of-Thought (CoT)

**Chain-of-Thought prompting**, introduced by Wei et al. (2022), represents a fundamental breakthrough in the field of prompt engineering. This technique allows LLMs to tackle complex reasoning tasks through a series of **intermediate reasoning steps**, making explicit the logical process that leads to the final response.

The intuition behind CoT is that, just as humans solve complex problems by decomposing them into more manageable subproblems, language models can also benefit from an explicit reasoning structure. CoT has been described as an "emergent capability that arises with sufficiently large language models", indicating that its effectiveness scales with model size.

**Zero-Shot CoT:** Kojima et al. (2022) discovered a surprisingly simple variant: by adding the phrase "Let's think step by step" at the end of a prompt, models significantly improve performance on reasoning tasks, without needing demonstrative examples. This suggests that LLMs already possess structured reasoning abilities that can be activated with the right linguistic stimulus.

**Automatic CoT (Auto-CoT):** Zhang et al. (2022) proposed an approach to automate the creation of CoT demonstrations. The process involves two phases: (1) clustering of questions into groups and (2) sampling of diversified representatives with automatic generation of reasoning chains via zero-shot CoT. Specific heuristics are applied: question length around 60 tokens and about 5 reasoning steps per chain. This approach eliminates the need to manually create demonstrative examples, maintaining quality through diversity.

### 3.4 Self-Consistency

**Self-Consistency**, developed as a natural extension of CoT, aims to replace the "naive greedy decoding used in chain-of-thought prompting" with a more robust approach based on the generation of multiple reasoning paths and the selection of the most frequent response.

The operational process unfolds in three phases:

1. **Prompting with CoT examples**: few-shot examples with reasoning chains are used
2. **Multiple sampling**: the model generates k independent reasoning paths for the same problem
3. **Aggregation by consensus**: the response that appears most frequently among the k paths is selected

Self-Consistency particularly excels in:
- Arithmetic and mathematical reasoning
- Common sense based reasoning
- Complex multi-step problems

The fundamental intuition is that a complex problem can have multiple valid reasoning paths that converge on the same correct answer. If the majority of independent paths leads to the same conclusion, confidence in the correctness of that response significantly increases. Conversely, if the paths diverge, this signals an intrinsic uncertainty in the problem or in the model's limits.

### 3.5 Generated Knowledge Prompting

**Generated Knowledge Prompting**, introduced by Liu et al. (2022), operates in three distinct phases to improve the accuracy of responses on tasks that require world knowledge:

1. **Knowledge generation**: the model is asked to generate facts and information relevant to the question
2. **Knowledge integration**: the generated information is incorporated into a new prompt
3. **Final prediction**: the model responds using the knowledge context provided

This technique proves particularly powerful for common sense based reasoning tasks, where the model might otherwise rely on superficial associations rather than on a deep understanding of concepts. An important aspect highlighted by the research is that the generated knowledge can produce variable confidence levels depending on how it is formulated, highlighting important nuances in how the presentation of knowledge influences the model's reasoning.

### 3.6 Tree of Thoughts (ToT)

**Tree of Thoughts**, proposed by Yao et al. (2023), represents a significant evolution of Chain-of-Thought that introduces a tree exploration structure for solving complex problems that require strategic planning.

Unlike CoT, which follows a single linear reasoning path, ToT maintains a "tree of thoughts, where thoughts represent coherent linguistic sequences that serve as intermediate steps towards the resolution of a problem". The framework operates through four key mechanisms:

1. **Generation of multiple candidates**: at each step, several possible thoughts are generated
2. **Self-evaluation of progress**: the model evaluates the goodness of each intermediate path
3. **Search algorithms**: BFS (Breadth-First Search), DFS (Depth-First Search) or beam search are applied to systematically explore promising paths
4. **Lookahead and backtracking**: the system can look ahead and go back to find optimal solutions

**Main implementations:**

- **Yao et al. (2023)** use generic search algorithms (DFS/BFS/beam search) without specific adaptation to the problem
- **Long (2023)** employs a "ToT Controller" trained via reinforcement learning, allowing the system to learn and adapt from new datasets

**Simplified variant:** Hulbert (2023) created a version applicable in a single prompt, asking the LLM to "evaluate intermediate thoughts". An interesting technique consists of asking the model to imagine multiple experts collaboratively solving the problem step by step.

ToT substantially surpasses traditional prompting methods on complex reasoning benchmarks, particularly in mathematical problem-solving tasks such as the "Game of 24".

---

## 4. Advanced Techniques for Agents

### 4.1 ReAct Prompting

**ReAct** (Reasoning + Acting), introduced by Yao et al. (2022), represents the fundamental framework for building AI agents based on LLMs. ReAct allows language models to generate both "reasoning traces and task-specific actions in an alternating manner", combining internal reasoning with the retrieval of external information.

The framework operates through an iterative cycle of three components:

- **Thought**: the model generates reasoning to decompose the problem, extract information or guide the next strategy
- **Action**: the model interacts with external tools such as search engines, knowledge bases or calculators
- **Observation**: environmental feedback informs subsequent reasoning steps

This alternating pattern allows models to dynamically adjust their plans by incorporating real-world information, addressing the limitations of pure chain-of-thought which can hallucinate facts or propagate errors without the possibility of external verification.

**Key advantages of ReAct:**
- Improvement of factuality through access to external information
- Greater interpretability and reliability for the human user
- Hybrid approach that combines internal knowledge and external data
- Flexible applicability to both reasoning and decision-making tasks

Research results have shown that ReAct surpasses methods based only on action on knowledge-intensive tasks (HotPotQA, Fever) and decision-making environments (ALFWorld, WebShop). The combination ReAct + CoT + Self-Consistency has shown the overall strongest results. LangChain provides built-in ReAct functionality, with a `zero-shot-react-description` agent that automatically orchestrates thought-action-observation cycles without requiring few-shot examples.

### 4.2 Automatic Prompt Engineer (APE)

The **Automatic Prompt Engineer** (APE), developed by Zhou et al. (2022), represents a paradigm shift in the approach to prompt engineering: instead of relying exclusively on manual optimization, APE frames prompt optimization as a "black-box optimization problem that uses LLMs to generate and search among candidate solutions".

The framework operates in three sequential phases:

1. **Generation**: an LLM generates multiple candidate instructions based on output demonstrations
2. **Execution**: candidates are tested using a target model
3. **Selection**: the instruction with the best performance is identified through an evaluation score

A particularly significant result of APE is the discovery of a zero-shot CoT prompt superior to the manually designed one. The discovered prompt -- "Let's work this out in a step by step way to be sure we have the right answer" -- produced better results than the now classic "Let's think step by step" on the MultiArith and GSM8K benchmarks.

APE connects to a broader ecosystem of automatic prompt optimization approaches:
- **Prompt-OIRL**: uses offline inverse reinforcement learning for query-dependent prompts
- **OPRO**: employs LLMs themselves for prompt optimization
- **AutoPrompt**: gradient-guided automatic prompt creation
- **Prefix/Prompt Tuning**: approaches with trainable parameters for prompt learning

### 4.3 Active Prompting

**Active Prompting**, proposed by Diao et al. (2023), addresses a fundamental limitation of traditional Chain-of-Thought methods: the dependence on fixed sets of manually annotated examples that may not be optimal for different task-specific requirements.

The process unfolds in five phases:

1. **Initial query**: the LLM receives prompts with or without preliminary CoT examples
2. **Generation of responses**: the model produces k possible responses for the training questions
3. **Uncertainty measurement**: an uncertainty metric (based on disagreement) is calculated among the k responses
4. **Targeted human annotation**: questions with the highest uncertainty are selected for human annotation
5. **Inference**: the newly annotated examples guide predictions on test questions

The key innovation of Active Prompting lies in the dynamic identification of training instances that would most benefit from human annotation, creating task-specific examples that better match the particular problem domain. Instead of wasting human effort on examples where the model is already certain, focus is placed on those where human intervention has maximum impact.

### 4.4 Directional Stimulus Prompting

**Directional Stimulus Prompting**, proposed by Li et al. (2023), introduces an innovative approach to guide LLMs in generating desired outputs through a directional stimulus mechanism.

The technique employs a **trainable policy language model** that generates stimuli or hints to direct the output of the main LLM. A fundamental advantage is that "the policy model can be small and optimized to generate hints that guide a frozen black-box LLM". This means that there is no need to modify or fine-tune the primary language model: the guide model can be lightweight and built for a specific purpose.

The approach demonstrates "a growing use of reinforcement learning to optimize LLMs", indicating that RL optimization plays a key role in the process. Directional Stimulus Prompting differs from conventional methods by introducing a systematic mechanism to direct LLM behavior through the learned generation of stimuli, rather than relying exclusively on manual prompt engineering.

---

## 5. Prompts for Agents: Best Practices

### System Prompt: Architecture and Design

The system prompt represents the foundation of every AI agent. Its design must follow precise structural principles to guarantee coherent, reliable and predictable behaviors.

**Recommended structure of a system prompt for agents:**

```
[IDENTITY AND ROLE]
Clear definition of the role, competencies and limits of the agent.

[OBJECTIVES AND BEHAVIOR]
Specification of primary objectives and expected behavior.

[CONSTRAINTS AND GUARDRAILS]
Operational limits, things not to do, boundaries of competence.

[OUTPUT FORMAT]
Structure of responses, required formats, conventions.

[AVAILABLE TOOLS]
Description of tools and conditions of use.

[ERROR HANDLING]
Procedure for anomalous or out-of-domain situations.
```

**Fundamental principles:**

1. **Specificity without excessive rigidity**: the prompt must be specific regarding the desired behavior without stifling the model's ability to handle unexpected situations. An instruction such as "Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential" is more effective than a generic "You are a financial advisor"

2. **Complete and pertinent context**: provide all the information necessary for the task, including constraints, expected formats and domain context

3. **Concrete examples**: include demonstrative examples for ambiguous or complex tasks

4. **Positive instructions**: prefer positive formulations ("respond concisely") to negative ones ("don't write long responses")

### Tool Description

The quality of tool descriptions is critical for the correct functioning of an agent. Each tool must include:

- **Clear and descriptive name**: the name must immediately communicate the function
- **Detailed description**: explanation of what the tool does, when to use it and when not to use it
- **Well-documented parameters**: type, format, constraints and default values for each parameter
- **Usage examples**: at least one concrete example of correct invocation
- **Error handling**: description of possible errors and expected behavior

**Structured example:**
```json
{
  "name": "search_database",
  "description": "Searches the database for products by name, category or ID.
    Use when the user asks for information about specific products.
    DO NOT use for generic questions about the catalog.",
  "parameters": {
    "query": "search string (min 3 characters)",
    "category": "optional, filters by category",
    "limit": "maximum number of results (default: 10, max: 50)"
  }
}
```

### Error Handling

A robust agent must handle errors gracefully, without losing the conversation context or generating unpredictable behaviors.

**Error handling strategies in prompts:**

1. **Explicit fallbacks**: define alternative behaviors when a tool fails
2. **Retry with backoff**: specify retry policies for transient errors
3. **Escalation**: define when and how to escalate to a human operator
4. **Transparency**: instruct the agent to clearly communicate to the user what went wrong
5. **State preservation**: maintain the conversation context even in case of error

**Prompt Scaffolding**: the practice of wrapping user inputs in structured and protected templates that limit the model's ability to behave inappropriately. You don't simply ask the model to respond, but tell it how to think, respond and refuse inappropriate requests, inserting the user input within rules, constraints and security logic.

### Context Engineering for Agents

One of the most recent and important areas documented in the guide is **Context Engineering**, which goes beyond simple prompt engineering to consider the entire context that is provided to an AI agent. This includes:

- **Short-term memory**: management of the context window of the current conversation
- **Long-term memory**: retrieval of information from previous conversations or documents
- **Environmental context**: information about the system, current state and available capabilities
- **Task context**: specific details about the task to be performed

---

## 6. Anti-Patterns and Common Mistakes

### Anti-Patterns in Prompt Engineering

Consolidated experience in the prompt engineering community has identified numerous recurring anti-patterns that compromise the effectiveness of interactions with LLMs.

**1. Vagueness and ambiguity**

The most widespread error is the formulation of vague prompts. Asking for "a summary" without specifying length, format or focus leaves too much room for the model's interpretation, producing inconsistent and hardly reproducible results. Each prompt should specify the desired output format, the approximate length, the target audience and the level of detail.

**2. Single prompt overload**

Inserting multiple unrelated tasks into a single prompt generates confusion in the model and degraded results. It is preferable to decompose complex tasks into a chain of simpler and more focused prompts (prompt chaining), where the output of one becomes the input of the next.

**3. Negative instructions**

Telling the model what NOT to do is generally less effective than specifying what it MUST do. "Don't use technical language" is less effective than "Use language accessible to a non-technical audience, explaining specialized terms when necessary".

**4. Inappropriate forcing on tools**

Instructing an agent with "you must ALWAYS use a tool" can cause hallucinations: the model could invent calls to non-existent tools or force the use of a tool in inappropriate contexts to comply with the instruction. Instructions on tool use must be conditional and contextual.

**5. Optimization for the demo, not for real use**

A critical error at the organizational level: building prompts that work perfectly with clean and predictable inputs, but fail with real user data. Prompts must be tested with noisy, ambiguous, incomplete and potentially malformed inputs.

**6. Ignoring model specifics**

Using the same identical prompt on different models (GPT-4, Claude, Gemini, Llama) without adaptations is an anti-pattern that wastes the specific capabilities of each model. Each family of models responds differently to certain prompt structures.

**7. Absence of evaluation systems**

You cannot optimize what you don't measure. The absence of quantitative metrics to evaluate prompt quality leads to optimization based on subjective impressions rather than concrete data. It is essential to implement automated evaluation systems that measure the quality of outputs on representative test datasets.

### Specific Errors in Agent Design

**8. Failure to define operational boundaries**

An agent without clear boundaries tends to interpret its mandate too broadly, attempting to respond to questions outside its domain of competence with potentially dangerous results.

**9. Reasoning chains that are too long without checkpoints**

In complex agentic systems, extremely long reasoning chains without intermediate verification points can accumulate errors that propagate and amplify at every step. It is essential to insert validation checkpoints at regular intervals.

**10. Failure to manage limited context**

Not considering the limits of the context window leads to prompts that lose critical information when the conversation lengthens, causing inconsistencies and forgetting on the part of the agent.

**11. Underestimating prompt injection**

Not implementing defenses against prompt injection -- where the user's input attempts to override system instructions -- exposes the agent to manipulation. Prompt scaffolding, input validation and clear separation between system instructions and user input are essential measures.

---

## 7. Resources and References

### Main Resources of Elvis Saravia and DAIR.AI

- **Prompt Engineering Guide**: [https://www.promptingguide.ai/](https://www.promptingguide.ai/) -- The complete guide with 18 documented techniques, guides for specific models, and educational resources
- **GitHub Repository**: [https://github.com/dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) -- 72,300+ stars, MIT license, 193 contributors
- **Elvis Saravia GitHub Profile**: [https://github.com/omarsar](https://github.com/omarsar) -- 149 repositories, 4,100+ followers
- **DAIR.AI Academy**: Self-paced courses on prompt engineering, RAG and AI agents
- **Substack**: [https://elvissaravia.substack.com/](https://elvissaravia.substack.com/) -- Articles and lectures on prompt engineering
- **Maven Courses**: [https://maven.com/dair-ai/prompt-engineering-llms](https://maven.com/dair-ai/prompt-engineering-llms) -- Advanced Prompt Engineering for LLMs

### Reference Papers Cited in the Guide

- **Chain-of-Thought**: Wei et al. (2022) -- "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"
- **Zero-Shot CoT**: Kojima et al. (2022) -- "Large Language Models are Zero-Shot Reasoners"
- **Auto-CoT**: Zhang et al. (2022) -- Automation of CoT demonstration creation
- **Self-Consistency**: Wang et al. (2022) -- Consensus-based decoding for CoT
- **Generated Knowledge**: Liu et al. (2022) -- Knowledge generation for reasoning
- **Tree of Thoughts**: Yao et al. (2023) -- Tree exploration for problem solving
- **ReAct**: Yao et al. (2022) -- "Synergizing Reasoning and Acting in Language Models"
- **APE**: Zhou et al. (2022) -- "Large Language Models Are Human-Level Prompt Engineers"
- **Active-Prompt**: Diao et al. (2023) -- Adaptive selection of examples
- **Directional Stimulus**: Li et al. (2023) -- Directional guidance via policy models (arxiv.org/abs/2302.11520)
- **Reflexion**: Shinn et al. (2023) -- Self-reflection for iterative improvement

### Complementary Resources

- **Elvis Saravia's NLP Newsletter**: Weekly updates on AI trends, papers and tools
- **ML-YouTube-Courses**: [https://github.com/dair-ai/ML-YouTube-Courses](https://github.com/dair-ai/ML-YouTube-Courses) -- Curation of ML courses
- **ml-visuals**: [https://github.com/dair-ai/ml-visuals](https://github.com/dair-ai/ml-visuals) -- Visual resources for scientific communication
- **DAIR.AI Community**: Active on Discord, Twitter/X, YouTube and newsletter

### Sources Used for This Document

- [Prompt Engineering Guide - DAIR.AI](https://www.promptingguide.ai/)
- [GitHub - Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)
- [Elvis Saravia GitHub Profile](https://github.com/omarsar)
- [Elvis Saravia Substack](https://elvissaravia.substack.com/p/the-prompt-engineering-guide)
- [Advanced Prompt Engineering for LLMs - Maven](https://maven.com/dair-ai/prompt-engineering-llms)
- [Elvis Saravia LinkedIn](https://bz.linkedin.com/in/omarsar)
- [DAIR.AI Academy - Introduction to Prompt Engineering](https://dair-ai.thinkific.com/courses/introduction-prompt-engineering)

---

*Document compiled based on the public resources of Elvis Saravia, the Prompt Engineering Guide (promptingguide.ai) and the relevant scientific literature. Last updated: March 2026.*
