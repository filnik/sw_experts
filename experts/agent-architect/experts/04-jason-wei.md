# Jason Wei: Chain-of-Thought Prompting and the Reasoning Revolution in Large Language Models

---

## 1. Profile and Background

### Who is Jason Wei

Jason Wei is one of the most influential researchers in modern artificial intelligence. His work has contributed decisively to transforming Large Language Models (LLMs) from simple text generators into systems capable of structured and complex reasoning. His research on Chain-of-Thought prompting, instruction tuning, and the emergent abilities of language models has redefined how the scientific community and industry conceive of interaction with language models.

### Academic Background

Jason Wei completed his studies in Computer Science at Dartmouth College, graduating in 2020. His training at Dartmouth -- an Ivy League institution with strong emphasis on the liberal arts -- exposed him to an interdisciplinary perspective that is reflected in his approach to research. During his undergraduate studies, Wei received the prestigious Goldwater Scholarship for his research in machine learning, a recognition awarded to outstanding students in STEM disciplines in the United States.

Before dedicating himself entirely to machine learning, Wei had initially considered a career in finance. The discovery of ML during a summer internship at Protago Labs, an artificial intelligence startup, radically changed his professional trajectory. Returning to Dartmouth, he began studying computer science and conducting research with Saeed Hassanpour, professor of biomedical data science at the Geisel School of Medicine, initially focusing on medical image analysis.

### Professional Career

#### Google Brain (2020-2023)

After graduating, Wei joined Google Brain as a Research Scientist, Google's artificial intelligence research lab. During this period, he produced some of the most cited and influential contributions in the recent history of Natural Language Processing. His three main research strands at Google Brain were:

1. **Chain-of-Thought Prompting** (2022): The technique that made step-by-step reasoning possible in LLMs, with the seminal paper that has accumulated over 5,000 citations in a few years.
2. **FLAN and Instruction Tuning** (2021-2022): The development of the FLAN methodology (Finetuned Language Net), which demonstrated how fine-tuning on natural language instructions drastically improves the zero-shot performance of models.
3. **Emergent Abilities** (2022): The identification and formalization of emergent abilities in LLMs, that is, abilities that appear only when the model reaches a sufficient scale.

When Google Brain was merged with DeepMind to form Google DeepMind, Wei briefly continued his work before changing organizations.

#### OpenAI (2023-2025)

In 2023, Wei moved to OpenAI as Member of Technical Staff, focusing on reasoning and agents. His most significant contribution at OpenAI was the co-development of the **o1** model, previewed in September 2024. The o1 model represents the natural evolution of his research on Chain-of-Thought: instead of relying exclusively on prompting to obtain step-by-step reasoning, o1 was trained via reinforcement learning to produce internal reasoning chains before generating the final answer. As Wei himself stated, the key progress was "not doing chain of thought purely via prompting, but training models to do better chain of thought using reinforcement learning."

#### Meta Superintelligence Labs (2025-present)

In July 2025, Wei left OpenAI to join **Meta Superintelligence Labs**, Meta's new research lab dedicated to the development of superintelligent systems. In this transition, he was accompanied by colleague Hyung Won Chung, signaling Meta's commitment to recruiting the best talents in the field of reasoning and model alignment.

---

## 2. Chain-of-Thought Prompting: Complete Explanation of the Technique

### The Foundational Paper

The paper "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (Wei et al., January 2022, arXiv:2201.11903) introduced an apparently simple technique with profound implications. The authors -- Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, and Denny Zhou -- demonstrated that adding intermediate reasoning steps in LLM prompts significantly improves their reasoning abilities.

### The Problem Solved

Before Chain-of-Thought prompting, LLMs were typically used in "direct input-output" mode: a question was provided and an immediate answer was requested. This approach worked reasonably well for simple tasks, but systematically failed on problems requiring multi-step reasoning, such as:

- **Arithmetic reasoning**: math problems requiring multiple sequential operations
- **Commonsense reasoning**: questions requiring inferences based on world knowledge
- **Symbolic reasoning**: manipulation of symbols according to logical rules

### How CoT Works (Few-Shot)

Chain-of-Thought prompting in its original form (few-shot) works by providing the model with a few examples (exemplars) that include not only the question and the answer, but also the **intermediate reasoning steps**. The model, by imitation (in-context learning), learns to generate its own intermediate steps before arriving at the final answer.

**Classic example from the paper:**

```
Question: Roger has 5 tennis balls. He buys 2 more cans
of tennis balls. Each can has 3 tennis balls.
How many tennis balls does he have now?

Reasoning: Roger started with 5 balls. He bought
2 cans of 3 balls each, that is 2 * 3 = 6 balls.
So in total he has 5 + 6 = 11 balls.

Answer: 11
```

By providing 4-8 examples of this type as context, the model learns to decompose subsequent problems in the same way.

### Experimental Results

The results of the paper were extraordinary:

- **GSM8K** (math problems): PaLM 540B with 8 CoT exemplars achieved state-of-the-art, even surpassing GPT-3 fine-tuned with a dedicated verifier.
- The improvement was observed on **arithmetic reasoning**, **commonsense reasoning**, and **symbolic reasoning**.
- Tests were conducted on three families of large models, confirming the generalizability of the technique.

### Few-Shot CoT vs Zero-Shot CoT

The distinction between these two variants is fundamental:

**Few-Shot CoT** (Wei et al., 2022):
- Requires manual construction of examples with reasoning chains
- Offers greater control over the format and quality of reasoning
- More effective in specific and well-defined contexts
- Requires domain expertise to create good exemplars

**Zero-Shot CoT** (Kojima et al., 2022):
- Requires no examples, only the addition of a trigger phrase
- The most effective phrase is "Let's think step by step"
- Simpler to implement but less controllable
- Particularly useful when no reasoning examples are available in the target domain

---

## 3. Variants and Evolutions of Chain-of-Thought

### Self-Consistency (Wang, Wei et al., 2022)

The paper "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (arXiv:2203.11171), co-signed by Wei, introduced a complementary technique to CoT of considerable importance. The fundamental insight is that for a complex problem, **multiple valid reasoning paths** typically exist that lead to the same correct answer.

**How it works:**

1. Standard Chain-of-Thought prompting is applied
2. Instead of generating only one reasoning chain (greedy decoding), **multiple different chains** are sampled (typically 5-40) using high temperatures
3. The final answer is extracted from each chain
4. The answer that appears most frequently is selected (majority voting)

**Quantitative results:**
- GSM8K: **+17.9%** over standard CoT
- SVAMP: **+11.0%**
- AQuA: **+12.2%**
- StrategyQA: **+6.4%**
- ARC-challenge: **+3.9%**

Self-Consistency is particularly powerful because it requires no additional training, supplementary models, or human annotations. It is purely a decoding strategy that leverages the intrinsic diversity of the model's reasoning paths.

### Zero-Shot CoT: "Let's Think Step by Step" (Kojima et al., 2022)

The paper "Large Language Models are Zero-Shot Reasoners" (arXiv:2205.11916) by Takeshi Kojima and colleagues demonstrated a surprising result: simply adding the phrase **"Let's think step by step"** before the answer activates reasoning abilities in LLMs, without any demonstrative examples.

**Two-phase mechanism:**

1. **Phase 1 - Reasoning extraction**: "Let's think step by step" is added after the question, and the model generates a reasoning chain
2. **Phase 2 - Answer extraction**: A second prompt is used to extract the final answer from the generated reasoning chain

**Tested variants:**
The researchers tested different formulations of the trigger phrase:
- "Let's think step by step" (the most effective)
- "Let's solve this problem by splitting it into steps"
- "Let's think about this logically"
- "Take a deep breath and work through this step by step" (found effective on some Google models)

**Scale dependence:** A crucial aspect is that Zero-Shot CoT works only with sufficiently large models. Smaller models (GPT-3 0.3B, 1.3B, 6.7B) show minimal or no improvements, while models such as InstructGPT (text-davinci-002) and PaLM 540B show drastic performance increases.

### Tree of Thoughts (Yao et al., 2023)

The paper "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (arXiv:2305.10601) by Shunyu Yao and colleagues, presented at NeurIPS 2023, represents a significant generalization of Chain-of-Thought. While CoT produces a single linear reasoning chain, Tree of Thoughts (ToT) maintains a **tree of thoughts** where each node represents a partial reasoning step.

**Conceptual architecture:**

```
                    [Problem]
                   /    |    \
              [Thought A] [Thought B] [Thought C]
              /    \         |           |
         [A1]    [A2]    [B1]        [C1]
          |       |        |           |
        [...]   [...]    [...]       [...]
```

**Key components:**

1. **Decomposition into thoughts**: The problem is decomposed into coherent text units (thoughts) that act as intermediate steps
2. **Generation**: For each node, the LLM generates multiple candidate thoughts
3. **Evaluation**: The LLM evaluates the promise of each thought (self-evaluation)
4. **Search**: Classical search algorithms (BFS, DFS) are applied to explore the tree with lookahead and backtracking

**Impressive results:**
- In Game of 24, GPT-4 with standard CoT solved only **4%** of problems, while with ToT it reached **74%** -- an improvement of over 18 times.

### Other Relevant Variants

**Automatic Chain-of-Thought (Auto-CoT):** Uses LLMs themselves to automatically generate reasoning exemplars, eliminating the need to create them manually. The key is diversity: using different questions and reasoning demonstrations to cover a broad spectrum of problems.

**Program-of-Thought (PoT):** Instead of generating reasoning in natural language, the model generates executable code that solves the problem step by step, delegating actual computation to an interpreter.

**Graph-of-Thought (GoT):** Extension of ToT that allows non-hierarchical connections between thoughts, modeling reasoning as a graph rather than a tree.

---

## 4. Impact on AI Agents

### Why CoT is Fundamental for Agent Reasoning

Chain-of-Thought prompting is not just a technique to improve chatbot responses: it has become an **architectural pillar** of modern agentic systems based on LLMs. The ability to reason step-by-step is the precondition for any autonomous agent that must plan, decide, and act in complex environments.

### ReAct: The Fusion of Reasoning and Acting

The ReAct framework (Yao et al., 2022, arXiv:2210.03629) represents perhaps the most direct and influential application of CoT in the context of AI agents. ReAct combines CoT reasoning with the ability to act, creating a synergistic cycle:

```
Thought: I need to find the population of Italy in 2023.
Action: Search[Italy population 2023]
Observation: Italy has approximately 58.9 million inhabitants in 2023.
Thought: Now I need to compare with the population of France.
Action: Search[France population 2023]
Observation: France has approximately 68.2 million inhabitants in 2023.
Thought: France has 68.2 - 58.9 = 9.3 million more inhabitants
than Italy.
Answer: France has approximately 9.3 million more inhabitants
than Italy.
```

This Thought-Action-Observation scheme is the basis of practically all modern agentic frameworks, from LangChain to AutoGPT, from CrewAI to Microsoft Semantic Kernel.

### Advantages of CoT for Agents

1. **Transparency and interpretability**: Reasoning chains make the agent's decision-making process inspectable and understandable to humans, fundamental for trust and debugging.

2. **Hallucination reduction**: Structured reasoning, especially when combined with access to external sources (as in ReAct), significantly reduces the generation of false information.

3. **Decomposition of complex tasks**: Agents must often solve multi-step problems requiring decomposition into sub-goals. CoT provides the cognitive mechanism for this decomposition.

4. **Adaptive planning**: Explicit reasoning allows the agent to evaluate intermediate results and adapt the action plan, rather than rigidly following a predefined sequence.

5. **Adaptive compute**: As Wei highlighted in the context of o1, Chain-of-Thought represents a form of "adaptive compute" -- the model can dedicate more computational resources (more reasoning tokens) to harder problems, scaling inference based on complexity.

### The Evolution toward o1 and Native Reasoning

Wei's work on o1 at OpenAI represents the logical endpoint of the trajectory begun with CoT prompting. If in 2022 CoT was an "external" prompting technique -- the model was convinced to reason step-by-step through examples in the prompt -- with o1, reasoning has become an **intrinsic** capability of the model, trained via reinforcement learning. The model internally produces long reasoning chains before providing the answer, without the need for explicit prompting.

This evolution -- from prompting technique to native capability -- is perhaps the most significant contribution of Wei's career to AI research, and has paved the way for "reasoning-first" models such as o1, o3, Claude 3.5, and Gemini 2.0 Flash Thinking.

---

## 5. How to Use CoT in Practice

### Concrete Techniques and Prompt Templates

#### Template 1: Classic Few-Shot CoT

```
Solve the following problem showing all the steps
of your reasoning.

Example 1:
Question: A store sells t-shirts at 15 euros each.
If you buy 3 or more t-shirts, you get a 10% discount.
Maria buys 4 t-shirts. How much does she pay?

Reasoning:
- Base price for 4 t-shirts: 4 x 15 = 60 euros
- Maria buys 4 t-shirts (>= 3), so she gets the 10% discount
- Discount: 60 x 0.10 = 6 euros
- Final price: 60 - 6 = 54 euros

Answer: Maria pays 54 euros.

Example 2:
[... another example with explicit reasoning ...]

Now solve:
Question: [YOUR PROBLEM HERE]

Reasoning:
```

#### Template 2: Zero-Shot CoT with Structure

```
[QUESTION OR PROBLEM]

Solve this problem step by step.
First identify the key information, then develop
the reasoning, and finally provide the answer.
```

#### Template 3: CoT for Analysis and Decisions

```
Analyze the following situation and provide a recommendation.

Situation: [DESCRIPTION]

Proceed as follows:
1. Identify the key factors
2. Analyze the pros and cons of each option
3. Consider risks and uncertainties
4. Formulate your recommendation with justification

Analysis:
```

#### Template 4: CoT for Code Debugging

```
The following code produces an error. Analyze it
step by step.

```[language]
[code]
```

Follow these steps:
1. Read the code line by line and describe what it does
2. Identify where the logical flow breaks
3. Explain the specific cause of the error
4. Propose the fix and explain why it works

Step-by-step analysis:
```

#### Template 5: Self-Consistency in Practice

```python
# Pseudocode for Self-Consistency
import llm_api

def self_consistency_cot(question, n_samples=10):
    """
    Implements Self-Consistency with Chain-of-Thought.
    Generates n reasoning chains and selects the most
    frequent answer via majority voting.
    """
    answers = []
    for i in range(n_samples):
        prompt = f"""
        {question}

        Let's think step by step to solve
        this problem.
        """
        response = llm_api.generate(
            prompt=prompt,
            temperature=0.7  # high temperature for diversity
        )
        answer = extract_final_answer(response)
        answers.append(answer)

    # Majority voting
    from collections import Counter
    most_common = Counter(answers).most_common(1)[0][0]
    return most_common
```

### Best Practices for Chain-of-Thought

1. **Use sufficiently large models**: CoT is an emergent ability. With models below 10 billion parameters, CoT can worsen performance rather than improve it. Use frontier models (GPT-4, Claude Opus/Sonnet, Gemini Pro) to obtain the best results.

2. **Provide diversified examples in few-shot**: Do not use examples that are too similar to each other. The diversity of exemplars allows the model to generalize better to new problems.

3. **Specify the desired format**: If numbered steps, explicit calculations, or a specific format are needed, show it in the examples. The model will tend to replicate the structure of the exemplars.

4. **Combine CoT with Self-Consistency for critical tasks**: When accuracy is essential, generating multiple reasoning chains and using majority voting can significantly increase quality.

5. **Avoid CoT for simple tasks**: For direct factual questions or trivial tasks, CoT adds latency and computational cost without significant benefits. Use it where multi-step reasoning is actually necessary.

6. **Iterate on exemplars**: The quality of few-shot exemplars greatly influences the result. Test different formulations and measure performance on a validation set.

7. **Monitor chain quality**: Not all reasoning generated by the model is correct. Reasoning chains can contain logical errors that lead to plausible but wrong answers. Implement verifications or use Self-Consistency as protection.

---

## 6. Scaling Laws and Emergent Abilities

### The Paper on Emergent Abilities

The paper "Emergent Abilities of Large Language Models" (Wei et al., June 2022, arXiv:2206.07682) is one of Jason Wei's most discussed and influential contributions. In this work, Wei and a broad team of co-authors (including Yi Tay, Rishi Bommasani, Colin Raffel, Barret Zoph, Percy Liang, Jeff Dean, and many others) identified and formalized a fascinating phenomenon: the **emergent abilities** of LLMs.

### Definition of Emergence

A capability is defined as **emergent** if:

1. It is not present (or is at a random level) in smaller models
2. It appears in larger models
3. Its appearance **cannot be predicted** simply by extrapolating the performance of smaller models

This is a crucial concept: it is not a gradual and predictable improvement, but a **qualitative leap** that manifests upon reaching a certain scale. The model suddenly transitions from random performance to high performance, without a gradual transition.

### Empirical Evidence

Wei has cataloged **137 emergent abilities** of LLMs in his blog and in the paper. These abilities cover a vast range of tasks:

- **Multi-step arithmetic reasoning**: The ability to solve math problems requiring multiple operations emerges only in models with hundreds of billions of parameters
- **Chain-of-Thought itself**: CoT prompting is itself an emergent ability -- it works only in sufficiently large models
- **Understanding complex instructions**: The ability to follow articulated and compositional instructions
- **Analogical reasoning**: The ability to identify and apply analogies between different domains

### The Connection with Scaling Laws

The work on emergent abilities fits within the broader context of **scaling laws** for LLMs, initially formalized by Kaplan et al. (2020) at OpenAI. Scaling laws describe predictable mathematical relationships between model size, amount of training data, compute used, and performance.

However, emergent abilities represent a **deviation** from standard scaling laws: while scaling laws predict gradual and smooth improvements, emergent abilities show sudden phase transitions. This has profound implications:

1. **Unpredictability**: We cannot know in advance which abilities will appear at the next scale level
2. **Motivation for scaling**: Emergent abilities provide a strong economic and scientific motivation to continue increasing the scale of models
3. **Unforeseen risks**: Just as desirable abilities emerge, undesirable or dangerous abilities could also emerge

### Debate and Critiques

The concept of emergence in LLMs has generated lively scientific debate. Some researchers have argued that the apparent emergence might be an artifact of the metrics used: if more granular or continuous metrics are used (rather than binary metrics such as "correct/incorrect"), the improvement appears more gradual. However, Wei's contribution remains fundamental in having drawn the community's attention to these phenomena and in having provided a conceptual framework for understanding them.

### FLAN and Instruction Tuning

The work on instruction tuning with FLAN (Finetuned Language Net) represents the other great research strand of Wei's work on scaling. The original paper "Finetuned Language Models Are Zero-Shot Learners" (Wei et al., 2021, arXiv:2109.01652) demonstrated that fine-tuning an LLM on a collection of tasks described via natural language instructions drastically improves zero-shot generalization on never-seen tasks.

**Key results of FLAN:**
- FLAN (137B parameters) outperformed GPT-3 (175B) in zero-shot mode on 20 of the 25 tasks evaluated
- Fine-tuning on over 60 NLP tasks with natural instruction templates was sufficient to obtain these improvements
- Subsequent scaling with **Flan-PaLM** (540B parameters, fine-tuned on 1,800 tasks) reached 75.2% on MMLU in 5-shot mode, the state-of-the-art at the time of publication

This line of research has shown that not only the scale of the model, but also the **scale and diversity of instruction tuning data** are critical factors for LLM performance.

---

## 7. Resources and References

### Foundational Papers by Jason Wei

| Paper | Year | Citations | Link |
|-------|------|-----------|------|
| Chain-of-Thought Prompting Elicits Reasoning in Large Language Models | 2022 | 5,000+ | [arXiv:2201.11903](https://arxiv.org/abs/2201.11903) |
| Emergent Abilities of Large Language Models | 2022 | 3,000+ | [arXiv:2206.07682](https://arxiv.org/abs/2206.07682) |
| Self-Consistency Improves Chain of Thought Reasoning in Language Models | 2022 | 2,000+ | [arXiv:2203.11171](https://arxiv.org/abs/2203.11171) |
| Finetuned Language Models Are Zero-Shot Learners (FLAN) | 2021 | 4,000+ | [arXiv:2109.01652](https://arxiv.org/abs/2109.01652) |
| Scaling Instruction-Finetuned Language Models (Flan-PaLM) | 2022 | 2,000+ | [arXiv:2210.11416](https://arxiv.org/abs/2210.11416) |

### Essential Related Papers

| Paper | Authors | Year | Link |
|-------|---------|------|------|
| Large Language Models are Zero-Shot Reasoners | Kojima et al. | 2022 | [arXiv:2205.11916](https://arxiv.org/abs/2205.11916) |
| Tree of Thoughts: Deliberate Problem Solving with LLMs | Yao et al. | 2023 | [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) |
| ReAct: Synergizing Reasoning and Acting in Language Models | Yao et al. | 2022 | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) |

### Online Resources

- **Jason Wei's personal site**: [jasonwei.net](https://www.jasonwei.net)
- **Google Scholar**: [Jason Wei's profile](https://scholar.google.com/citations?user=wA5TK_0AAAAJ)
- **Blog on Emergent Abilities**: [137 emergent abilities of large language models](https://www.jasonwei.net/blog/emergence)
- **Prompt Engineering Guide - CoT**: [promptingguide.ai/techniques/cot](https://www.promptingguide.ai/techniques/cot)

### Practical Guides

- **Chain of Thought Prompting Guide** (PromptHub): [prompthub.us/blog/chain-of-thought-prompting-guide](https://www.prompthub.us/blog/chain-of-thought-prompting-guide)
- **CoT Prompting** (Learn Prompting): [learnprompting.org/docs/intermediate/chain_of_thought](https://learnprompting.org/docs/intermediate/chain_of_thought)
- **Zero-Shot CoT** (Learn Prompting): [learnprompting.org/docs/intermediate/zero_shot_cot](https://learnprompting.org/docs/intermediate/zero_shot_cot)
- **Tree of Thoughts Guide** (Prompt Engineering Guide): [promptingguide.ai/techniques/tot](https://www.promptingguide.ai/techniques/tot)

### Chronological Timeline of CoT Innovations

```
September 2021  FLAN - Instruction Tuning (Wei et al.)
January 2022    Chain-of-Thought Prompting (Wei et al.)
March 2022      Self-Consistency (Wang, Wei et al.)
May 2022        Zero-Shot CoT "Let's think step by step" (Kojima et al.)
June 2022       Emergent Abilities (Wei et al.)
October 2022    Flan-PaLM / Scaling Instruction Tuning (Wei et al.)
October 2022    ReAct: Reasoning + Acting (Yao et al.)
May 2023        Tree of Thoughts (Yao et al.)
September 2024  OpenAI o1 - CoT via Reinforcement Learning (Wei et al.)
```

---

*Document compiled with references to peer-reviewed papers and primary sources. Jason Wei represents one of the central figures in the transition of LLMs from text generation tools to systems capable of structured reasoning. His trajectory -- from Google Brain to OpenAI to Meta Superintelligence Labs -- reflects the growing importance of reasoning as a frontier of artificial intelligence research.*
