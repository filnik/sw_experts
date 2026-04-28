# Noah Shinn and the Reflexion Framework: Verbal Reinforcement Learning for Language Agents

---

## 1. Profile and Background

Noah Shinn is a programmer and researcher based in San Francisco, California, currently employed as a **Research Scientist at Sierra**, the conversational AI startup co-founded by Bret Taylor (former co-CEO of Salesforce) and Clay Bavor. Shinn was among the company's first employees, joining Sierra as a founding member of the research team.

### Academic Background

Shinn completed his studies at **Northeastern University**, at the Khoury College of Computer Sciences. During his university path, he ranged across different research areas:

- **Programming Language Group**: research on type prediction and code generation, working on type inference and automatic code generation problems
- **Computational Photochemistry Group**: research in excited-state molecular dynamics, an area very distant from AI but indicative of his scientific versatility
- **NU Aerospace Group**: member of the avionics team, demonstrating practical engineering skills

This heterogeneous training -- programming languages, computational chemistry, aerospace engineering -- forged a researcher capable of moving between different domains, a quality that emerges clearly in the versatility of the Reflexion framework.

### Main Publications

Noah Shinn's publications trace a coherent path toward understanding and improving LLM-based agents:

1. **Reflexion: Language Agents with Verbal Reinforcement Learning** (NeurIPS 2023) -- the seminal work that introduced the concept of verbal reinforcement for language agents. Co-authors: Federico Cassano, Edward Berman, Ashwin Gopinath (MIT), Karthik Narasimhan (Princeton), Shunyu Yao (Princeton).

2. **Can It Edit? Evaluating the Ability of Large Language Models to Follow Code Editing Instructions** (COLM 2024) -- a benchmark to evaluate the ability of LLMs to follow code editing instructions. Co-authors: Federico Cassano, Luisa Li, Akul Sethi, and others.

3. **tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains** (ICLR 2025) -- a benchmark to evaluate the interaction between tools, agents, and users in real-world domains. Co-authors: Shunyu Yao, Pedram Razavi, Karthik Narasimhan.

4. **Type Prediction With Program Decomposition** -- work on type prediction through program decomposition.

### Role at Sierra

At Sierra, Shinn continues his research on AI agents applied to the industrial context. His work on tau-bench, published on the company blog with the title "Sierra's tau-bench is shaping the development and evaluation of agents" (March 2025), demonstrates how his academic research translates into concrete tools for the development and evaluation of conversational agents in real-world contexts.

### Open Source Contributions

In addition to academic publications, Shinn is active in the development of open source tools, including utilities for web automation, git tools, terminal QA systems, CLI suggestion tools, and the **Leetcode Hard Gym**, a benchmark for evaluating the programming abilities of language models.

---

## 2. Reflexion: Verbal Reinforcement Learning

### The Fundamental Problem

Large Language Models (LLMs) are capable of generating text, code, and sophisticated reasoning, but they have a structural limit: **they do not learn from their own errors in real time**. When an LLM fails a task, it has no native mechanisms to reflect on the failure and improve in the next attempt. Traditional reinforcement learning requires costly model weight updates, specific training data, and significant computational resources.

### The Reflexion Proposal

Reflexion, presented at NeurIPS 2023, introduces a radically new paradigm: **verbal reinforcement learning**. Instead of updating the model weights (as in traditional RL), Reflexion keeps the model frozen and uses **textual feedback as a form of reinforcement**. The central idea is elegant: convert feedback signals (binary, scalar, or in natural language) into **linguistic reflections** that are stored in an episodic memory and provided as context in subsequent attempts.

### The "Semantic Gradient" Concept

The paper introduces the concept of **semantic gradient**: analogously to how traditional reinforcement learning uses numerical gradients to update the weights of a neural network, Reflexion uses textual reflections as "gradients" that modify the agent's behavior through context. The agent's policy is not parameterized by the model weights, but by the **combination of LLM parameters (fixed) and episodic memory (dynamic)**.

Formally, the policy becomes:

```
pi_theta(a_i | s_i) where theta = {M_a, mem}
```

Where `M_a` are the fixed parameters of the Actor model and `mem` is the content of the episodic memory, which evolves trial after trial. Policy optimization happens through changes to memory, not weights.

### Why It Works

The success of Reflexion rests on several insights:

1. **LLMs are already capable of self-criticism**: when an LLM is asked to analyze its own errors, it produces analyses that are often accurate and useful.
2. **Context is powerful**: providing an LLM with information about previous attempts and errors made significantly modifies its behavior.
3. **Reflection is richer than scalar reward**: a textual reflection like "I was wrong because I confused the sum with the product in step 3" contains much more actionable information than a simple numerical score of -1.
4. **No training data required**: Reflexion works out-of-the-box on any LLM without the need for datasets or fine-tuning procedures.

---

## 3. Technical Architecture

### The Three Fundamental Components

The Reflexion architecture is articulated in three distinct modules that collaborate in an iterative loop:

#### 3.1 Actor (M_a)

The **Actor** is the central component that generates text and actions based on environment observations. It works like a standard LLM agent -- it can use prompting strategies such as Chain-of-Thought (CoT) or ReAct -- but with a crucial addition: it also receives as input the content of the **episodic memory**, which contains reflections from previous trials.

The Actor operates similarly to a policy-based agent in RL: given a state `s_i`, it produces an action `a_i`. The difference is that its behavior is conditioned not only by the current state but also by past experiences encoded verbally.

In the context of code generation, for example, the Actor:
- Receives the problem specification
- Reads the reflections on previous failed attempts
- Generates a new solution that takes into account the identified errors

#### 3.2 Evaluator (M_e)

The **Evaluator** evaluates the quality of the trajectories (sequences of actions) produced by the Actor. Its task is to produce a reward signal `r_t = M_e(tau_t)` indicating the success or failure of the current attempt.

The Evaluator can take different forms depending on the domain:

- **Exact match grading**: for reasoning tasks (e.g., HotPotQA), it compares the generated answer with the correct one
- **Heuristic functions**: for decision-making tasks (e.g., AlfWorld), it detects problematic patterns such as repeating the same action more than 3 times or exceeding 30 total actions
- **Automatic tests**: for code generation, it executes unit tests (both provided and auto-generated) on the produced solution
- **LLM-based scoring**: it uses another LLM to semantically evaluate the quality of the output

The flexibility of the Evaluator is a strength: it does not require ground truth in all cases, allowing the application of Reflexion even in domains where evaluation is subjective.

#### 3.3 Self-Reflection (M_sr)

The **Self-Reflection** module is the innovative heart of the framework. It is an LLM (it can be the same as the Actor or a dedicated one) that receives as input:

- The reward signal from the current trial
- The complete trajectory of the attempt (short-term memory)
- Previous reflections (long-term memory)

And produces a **verbal reflection** `sr_t`: an analysis in natural language of what went wrong, why the reasoning failed, and what strategies to adopt in the next attempt. This reflection is then added to long-term memory.

A concrete example of a reflection generated for a coding task could be:

> "My previous implementation failed test case 3 because I incorrectly handled the edge case where the array is empty. The error was in the `merge_intervals` function where I did not check that `len(intervals) > 0` before accessing `intervals[0]`. In the next attempt, I must add an explicit check for empty arrays at the beginning of the function and return an empty list in that case."

### The Complete Loop

The Reflexion cycle unfolds as follows:

```
For each trial t = 1, 2, ..., T:
  1. The Actor generates a trajectory tau_t by interacting with the environment
     (conditioned by state + episodic memory)
  2. The Evaluator produces a reward r_t = M_e(tau_t)
  3. IF r_t indicates success -> TERMINATE
  4. OTHERWISE:
     a. The Self-Reflection module generates sr_t = M_sr(r_t, tau_t, mem)
     b. The reflection sr_t is added to memory: mem <- mem + [sr_t]
     c. Return to step 1
```

This creates a **meta-loop**: Act -> Evaluate -> Reflect -> Repeat.

### The Memory System

Reflexion implements a **dual memory** system:

- **Short-term memory**: contains the trajectory of the current attempt -- all observations, thoughts, and actions of the ongoing episode. It provides the detailed and granular context needed for the current step.

- **Long-term memory**: contains the reflections distilled from previous attempts, stored in a limited buffer. This buffer has a finite size (typically Omega = 1-3 experiences) to respect the limits of the LLM context window.

The **separation between the two memories** is crucial: short-term memory preserves the tactical details of the current episode, while long-term memory captures the strategic lessons learned from past failures. Ablation studies confirm that the memory component is fundamental to the framework's effectiveness.

### Domain Configurations

The framework adapts to different domains with specific configurations:

**Code generation (HumanEval, MBPP, LeetCode)**:
- The Actor generates code based on the specification and past reflections
- Chain-of-Thought prompting generates diversified unit tests (max 6 per problem)
- Syntactic validation through Abstract Syntax Tree (AST) construction
- Memory buffer: 1 experience (sufficient for coding tasks)

**Decision-making (AlfWorld)**:
- Heuristic trigger: same action-response pair repeated >3 times OR >30 total actions
- Alternative: LLM-based binary classification for autonomous detection
- Memory buffer: 3 recent experiences

**Reasoning (HotPotQA)**:
- 6 few-shot CoT examples; 2 ReAct and self-reflection examples
- Evaluation with exact match on the answer
- Memory buffer: 3 experiences

---

## 4. Results and Benchmarks

Reflexion has been evaluated on a wide range of benchmarks, demonstrating consistent improvements over baselines. Below are the detailed results.

### 4.1 HumanEval (Python Code Generation)

| Method | pass@1 |
|--------|--------|
| GPT-4 (baseline) | 80.1% |
| **Reflexion + GPT-4** | **91.0%** |

This result represented the new state-of-the-art at the time of publication, surpassing standard GPT-4 by **+11 percentage points**. The improvement is particularly significant because it was obtained without any modification to the model weights.

### 4.2 MBPP (Python Code Generation)

| Method | pass@1 |
|--------|--------|
| GPT-4 (baseline) | 80.1% |
| Reflexion + GPT-4 | 77.1% |

On MBPP, Reflexion does not improve over the baseline. This suggests that the framework's effectiveness depends on task complexity: on relatively simple problems, additional reflection may not provide significant benefits or may even introduce noise into the reasoning.

### 4.3 LeetCode Hard Gym (40 problems)

| Method | Accuracy |
|--------|----------|
| GPT-4 (baseline) | 7.5% |
| **Reflexion + GPT-4** | **15.0%** |

On extremely difficult programming problems, Reflexion doubles performance, although the absolute level remains low. This demonstrates that iterative reflection is particularly useful when the task requires exploration and progressive correction.

### 4.4 AlfWorld (Sequential Decision-Making)

| Method | Tasks completed (out of 134) |
|--------|------------------------------|
| ReAct (baseline) | ~86 (~64%) |
| **ReAct + Reflexion** | **130 (97%)** |

In AlfWorld, a textual environment for household decision-making, Reflexion leads to an improvement of **+22 percentage points** over 12 iterative trials. The learning curve shows a significant jump between the first and second trial, followed by gradual improvement in subsequent trials up to nearly perfect performance. This pattern recalls the learning curves of traditional RL, but obtained entirely through verbal reflection.

### 4.5 HotPotQA (Multi-hop Reasoning)

| Method | Accuracy (100 questions) |
|--------|--------------------------|
| Chain-of-Thought (baseline) | ~60% |
| **CoT + Reflexion** | **~80%** |
| ReAct (baseline) | ~39% |
| **ReAct + Reflexion** | **~51%** |

On HotPotQA, a question answering benchmark requiring multi-hop reasoning, Reflexion improves performance by **+20 percentage points** in the CoT variant. The improvement is obtained through the self-reflection loop that amplifies the binary success/failure signal.

### 4.6 Ablation Studies

Ablation studies reveal specific contributions of the components:

**Self-Reflection contribution (HotPotQA with CoT and ground truth context)**:
- Baseline CoT(GT): 60%
- CoT(GT) + episodic memory only (no reflection): 68%
- CoT(GT) + complete Reflexion: 76%
- **Net contribution of self-reflection: +8 percentage points** beyond the simple advantage of episodic memory

**Critical components for HumanEval Rust (50 most difficult problems)**:
- Without test generation: 52% (vs 60% baseline)
- Without self-reflection: 60% (no improvement)
- Complete Reflexion: 68%
- **Conclusion**: both components (test generation and self-reflection) are necessary; neither alone is sufficient

### 4.7 Limitations Emerging from Benchmarks

The paper identifies the framework's limitations with intellectual honesty:

- **WebShop**: experiments on WebShop (interactive shopping) failed. The domain requires high exploration, and Reflexion tends to get trapped in local minima.
- **Dependence on test quality**: in code generation, the quality of self-reflection depends on the quality of auto-generated tests. Fragile or incomplete tests produce false positives/negatives that degrade the learning trajectory.
- **Local minima**: being a natural-language-based optimization technique, Reflexion is vulnerable to local minima, especially in domains with high exploratory dimensionality.

---

## 5. Practical Applications

### 5.1 Implementing Reflexion in Your Own Agents

The practical implementation of Reflexion requires the definition of the three fundamental components adapted to your domain. Here is a structured approach.

#### Step 1: Define the Actor

The Actor is typically an LLM with a system prompt that includes:
- The task description
- Operational instructions
- Space for previous reflections (dynamically injected)

```python
def create_actor_prompt(task_description, reflections):
    reflection_text = "\n".join(reflections) if reflections else "No previous reflections."
    return f"""
    Task: {task_description}

    Reflections from previous attempts:
    {reflection_text}

    Based on the reflections above, generate an improved solution.
    """
```

#### Step 2: Define the Evaluator

The Evaluator must return a clear success/failure signal, ideally with diagnostic information:

- For **code generation**: run unit tests and collect output/errors
- For **reasoning**: compare with the expected answer or use a judge LLM
- For **decision-making**: verify the achievement of the goal within action limits

#### Step 3: Define the Self-Reflection Module

The reflection module receives the failed trajectory and produces a structured analysis. An effective prompt for self-reflection could be:

```python
def create_reflection_prompt(task, trajectory, error_info):
    return f"""
    You attempted the following task: {task}

    Your trajectory was:
    {trajectory}

    The result was a failure: {error_info}

    Analyze what went wrong:
    1. Identify the specific error
    2. Explain WHY you made this error
    3. Suggest a concrete strategy for the next attempt
    """
```

#### Step 4: Implement the Loop

```python
def reflexion_loop(task, max_trials=5):
    reflections = []
    for trial in range(max_trials):
        # Actor generates the solution
        solution = actor.generate(task, reflections)
        # Evaluator evaluates
        success, feedback = evaluator.evaluate(solution)
        if success:
            return solution
        # Self-Reflection generates the reflection
        reflection = self_reflect(task, solution, feedback)
        reflections.append(reflection)
        # Keep only the last N reflections
        if len(reflections) > 3:
            reflections = reflections[-3:]
    return None  # failure after max_trials
```

### 5.2 Integration with LangChain and LangGraph

LangChain's LangGraph framework offers native primitives for implementing reflection patterns. LangGraph models the agent's logic as a state graph, where the "nodes" are Python functions and the "edges" define the control flow. This naturally lends itself to implementing the Reflexion loop:

- **Actor node**: generates the output
- **Evaluator node**: evaluates the output
- **Conditional edge**: if successful, terminate; otherwise, continue
- **Reflection node**: generates the reflection
- **Return edge**: returns to the Actor node with the reflection added to the state

The `langchain-ai/langgraph-reflection` repository contains reference implementations, and the `examples/reflexion/` directory in the main LangGraph repository offers Jupyter notebooks with complete implementations.

### 5.3 Recommended Application Scenarios

Reflexion is particularly effective in scenarios where:

- **Feedback can be obtained automatically**: code generation (automated tests), tasks with verifiable answers, simulated environments
- **Tasks are complex but bounded**: problems that require multiple attempts but have clear success criteria
- **Exploration is not too broad**: domains where the number of reasonable strategies is limited
- **The cost of an attempt is low**: environments where retrying does not entail prohibitive costs

Less suitable scenarios include open-ended tasks without clear evaluation criteria, domains requiring massive exploration, and contexts where feedback is not available or not informative.

---

## 6. Comparison with Other Approaches

### 6.1 Reflexion vs ReAct

| Aspect | ReAct | Reflexion |
|--------|-------|-----------|
| **Mechanism** | Alternates reasoning and action in a single episode | Multi-episode loop with reflection between attempts |
| **Learning** | No learning between episodes | Learns from past failures via verbal reflection |
| **Memory** | Only the context of the current episode | Long-term episodic memory with reflections |
| **Strength** | Interpretability and step-by-step reasoning | Iterative self-improvement |
| **Weakness** | Does not improve between attempts | Requires more API calls (higher cost) |
| **Complementarity** | Can be the Actor in Reflexion | Can use ReAct as the Actor's strategy |

ReAct and Reflexion are not mutually exclusive alternatives: they are **complementary**. ReAct provides excellent structure for intra-episode reasoning (within a single attempt), while Reflexion adds inter-episode learning (between different attempts). The best results are often obtained by combining the two approaches: ReAct as the Actor inside the Reflexion loop.

On AlfWorld, ReAct alone converges to ~64%, but ReAct + Reflexion reaches 97%. On HotPotQA, ReAct alone obtains ~39%, while ReAct + Reflexion reaches ~51%. This demonstrates that inter-episode reflection adds significant value to ReAct's intra-episode reasoning.

### 6.2 Reflexion vs Traditional Fine-Tuning

| Aspect | Fine-Tuning | Reflexion |
|--------|-------------|-----------|
| **Model update** | Modifies LLM weights | Frozen model, only modifies memory |
| **Required data** | Training dataset (often hundreds/thousands of examples) | Zero additional data |
| **Required compute** | GPU/TPU for training | Only API inference cost |
| **Generalization** | Risk of overfitting | No overfitting risk (weights unchanged) |
| **Persistence** | Permanent (modified weights) | Temporary (in-context reflections) |
| **Interpretability** | Opaque (changes in weights) | Transparent (readable reflections) |
| **Setup** | Complex (training infrastructure) | Minimal (only LLM API) |

Traditional fine-tuning (for example with FireAct, which fine-tunes Llama2-7B with 500 GPT-4 trajectories on HotPotQA obtaining +77% improvement) offers advantages in terms of **inference efficiency**: fine-tuned models do not require few-shot examples in the context, reducing latency and cost per query. However, it requires significant investment in data and compute.

Reflexion excels as a **lightweight solution**: it works immediately on any LLM, does not require training infrastructure, and produces interpretable improvements. It is particularly advantageous when:
- Sufficient training data is not available
- You want to prototype quickly
- The interpretability of the improvement process is important
- You work with proprietary model APIs (where fine-tuning is not possible)

### 6.3 Reflexion vs Code Generation Approaches (CodeT, Self-Debugging, CodeRL)

In the specific domain of code generation, Reflexion stands out for a unique characteristic: it is the only approach that combines **auto-generated tests with iterative self-learning**. CodeT generates and selects among multiple solutions but does not learn from failures. Self-Debugging identifies errors but does not maintain a memory of lessons learned. CodeRL uses traditional RL for training, requiring weight updates.

Reflexion bridges the gap between **error identification and implementation improvement** through self-reflection, and qualifies as a genuine pass@1 method (it does not require validation with ground truth tests at runtime).

### 6.4 Positioning in the Agentic Landscape

In the broader landscape of design patterns for AI agents:

- **ReAct**: great for step-by-step problem-solving with tool calling, within a single conversation
- **Reflexion**: great for self-correction and self-improvement of a single solution (code, text, reasoning)
- **Auto-GPT / Plan-and-Execute**: great for long, open-ended tasks requiring planning, tool usage, and memory management
- **Multi-Agent**: great for tasks that benefit from different perspectives or role specialization

Reflexion occupies a specific but important niche: iterative improvement guided by reflection, applicable as a module within more complex agentic architectures.

---

## 7. Resources and References

### Papers and Publications

- **Original paper**: Shinn, N., Cassano, F., Berman, E., Gopinath, A., Narasimhan, K., & Yao, S. (2023). *Reflexion: Language Agents with Verbal Reinforcement Learning*. NeurIPS 2023. [arXiv:2303.11366](https://arxiv.org/abs/2303.11366)
- **OpenReview (peer review)**: [openreview.net/forum?id=vAElhFcKW6](https://openreview.net/forum?id=vAElhFcKW6)
- **NeurIPS Proceedings**: [papers.nips.cc/paper/2023/hash/1b44b878bb782e6954cd888628510e90](https://papers.nips.cc/paper_files/paper/2023/hash/1b44b878bb782e6954cd888628510e90-Abstract-Conference.html)
- **tau-bench**: Yao, S., Shinn, N., Razavi, P., & Narasimhan, K. (2025). *tau-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains*. ICLR 2025.
- **Can It Edit?**: Cassano, F., Li, L., Sethi, A., Shinn, N., et al. (2024). *Can It Edit? Evaluating the Ability of Large Language Models to Follow Code Editing Instructions*. COLM 2024.

### Repositories and Code

- **Official Reflexion repository**: [github.com/noahshinn/reflexion](https://github.com/noahshinn/reflexion) (MIT license)
- **Noah Shinn's GitHub**: [github.com/noahshinn](https://github.com/noahshinn)
- **LangGraph Reflexion examples**: [github.com/langchain-ai/langgraph/examples/reflexion](https://github.com/langchain-ai/langgraph/blob/main/examples/reflexion/reflexion.ipynb)
- **LangGraph Reflection library**: [github.com/langchain-ai/langgraph-reflection](https://github.com/langchain-ai/langgraph-reflection)

### Profiles and Personal Pages

- **Personal site**: [noahshinn.com](https://noahshinn.com/)
- **Google Scholar**: [scholar.google.com/citations?user=zTfIpA4AAAAJ](https://scholar.google.com/citations?user=zTfIpA4AAAAJ&hl=en)
- **Sierra**: [sierra.ai/author/noah-shinn](https://sierra.ai/author/noah-shinn)
- **DBLP**: [dblp.org/pid/342/9223.html](https://dblp.org/pid/342/9223.html)

### Guides and Tutorials

- **Prompt Engineering Guide -- Reflexion**: [promptingguide.ai/techniques/reflexion](https://www.promptingguide.ai/techniques/reflexion)
- **LangChain Blog -- Reflection Agents**: [blog.langchain.com/reflection-agents](https://blog.langchain.com/reflection-agents/)
- **Khoury College -- Article on NeurIPS**: [khoury.northeastern.edu -- Khoury undergrads take NeurIPS](https://www.khoury.northeastern.edu/satellite-drag-and-self-correcting-language-models-khoury-undergrads-take-neurips-conference/)

### Related Articles

- Yao, S., et al. (2023). *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023.
- Chen, A., et al. (2023). *FireAct: Toward Language Agent Fine-tuning*.
- Madaan, A., et al. (2023). *Self-Refine: Iterative Refinement with Self-Feedback*. NeurIPS 2023.

---

*Document compiled in March 2026. Reflexion remains one of the most influential frameworks in the field of language agents, with over 440 citations on Semantic Scholar and growing adoption in both academic and industrial settings.*
