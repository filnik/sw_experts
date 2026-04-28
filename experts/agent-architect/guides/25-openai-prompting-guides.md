# OpenAI Prompting Guides for GPT-4.1 / GPT-5

> In-depth technical guide based on OpenAI's official prompting guides for the GPT-4.1, GPT-5, GPT-5.1, and GPT-5.2 models. Covers agentic prompting, instruction hierarchy, reminder types, structured tool use, and best practices for building agentic systems.

---

## 1. Overview: Evolution of Prompting with Increasingly Capable Models

The recent history of OpenAI models marks a crucial shift in the prompt engineering paradigm. With GPT-4.1 (April 2025), OpenAI introduced a model explicitly designed to follow instructions in a **literal and precise** manner, marking a clear departure from its predecessors (such as GPT-4o) that tended to freely infer the user's intent. With GPT-5 and the subsequent GPT-5.1 and GPT-5.2, this trend has further consolidated, leading to models that require more structured prompts, free of contradictions, and that reward surgical precision in formulating instructions.

### 1.1 From GPT-4o to GPT-4.1: The Leap to Literal Instruction Following

GPT-4.1 represents the first model in the OpenAI family explicitly optimized for three key areas:
- **Coding**: significantly superior performance on benchmarks such as SWE-bench Verified
- **Instruction following**: literal adherence to the instructions provided in the prompt
- **Long context**: support for up to 1 million tokens with efficient context management

The most relevant feature for prompt engineering is that GPT-4.1 **does not infer** what the user intends anymore: if an instruction is not made explicit, the model will not assume it. This requires a mindset shift in prompt design, where every desired behavior must be explicitly declared.

### 1.2 GPT-5: Native Reasoning and Agentic Design

GPT-5 introduces native reasoning (reasoning tokens) and a design specifically oriented toward agentic applications. The model has been trained with a focus on:
- Improved and more reliable **tool calling**
- **Long context understanding** for complex multi-turn sessions
- Granular **steerability** via parameters such as `reasoning_effort` and `verbosity`

Unlike GPT-4.1, GPT-5 is intrinsically more "thorough" in its responses: it tends to autonomously explore the context and gather information before responding. This characteristic makes explicit prompts for context gathering less necessary, but requires attention in containing excessive verbosity.

### 1.3 GPT-5.1 and GPT-5.2: Balancing Speed and Intelligence

GPT-5.1 introduces the `none` reasoning mode, which completely eliminates reasoning tokens, bringing latency closer to GPT-4.1 but maintaining superior capabilities. GPT-5.2 further improves token efficiency, output formatting, and structured reasoning, positioning itself as the flagship model for enterprise and agentic workloads.

The trajectory is clear: each model generation requires cleaner, more structured, and more precise prompts, but offers in exchange dramatically superior agentic performance.

---

## 2. Agentic Prompting: Designing Prompts for Agentic Systems

Agentic prompting represents the heart of OpenAI's guides for GPT-4.1 and GPT-5. An LLM-based agent operates in an iterative loop of observation, planning, action, and reflection, and the system prompt must structure and govern every phase of this cycle.

### 2.1 The Three Fundamental Reminders

OpenAI identifies three types of essential reminders that must be present in the system prompt of every agent. The inclusion of these three elements has produced an increase of approximately **20% on OpenAI's internal SWE-bench Verified score**.

#### Persistence

The persistence reminder communicates to the model that it is in a multi-message turn and must not prematurely cede control to the user.

```
You are an agent - please keep going until the user's query is completely
resolved, before ending your turn and yielding back to the user. Only
terminate your turn when you are sure that the problem is solved.
```

This prevents the model from interrupting execution after a single step, forcing it to complete the entire workflow before returning control.

#### Tool-Calling

The tool-calling reminder encourages the model to fully exploit the available tools and reduces the probability of hallucinations.

```
If you are unsure about file content or codebase structure, use your tools
to read files and gather the relevant information: do NOT guess or make up
an answer.
```

This instruction is particularly critical for coding agents, where inventing the content of a file or the structure of a codebase would lead to cascading errors.

#### Planning

The planning reminder induces the model to reason explicitly before and after each action.

```
Plan extensively before each function call, and reflect extensively
on the outcomes of the previous function calls.
```

In OpenAI's tests, the explicit addition of planning increased the SWE-bench Verified pass rate by 4%.

### 2.2 Calibrating Agentic Eagerness

With GPT-5, OpenAI introduces the concept of **agentic eagerness** -- the degree of proactivity with which the agent explores, gathers context, and takes autonomous initiatives. This calibration is bidirectional:

**To reduce eagerness** (latency-sensitive scenarios):
- Lower `reasoning_effort` to `low` or `minimal`
- Define explicit early-stop criteria for context gathering
- Implement fixed budgets for tool calls (e.g., "maximum 2 calls")
- Provide escape hatches that allow the model to proceed under uncertainty

**To increase eagerness** (complex multi-step tasks):
- Raise `reasoning_effort` to `high`
- Use explicit persistence prompts
- Avoid asking the user for clarifications; encourage the model to deduce reasonable approaches
- Require documentation of assumptions made during execution

### 2.3 Responses API vs Chat Completions

For agentic flows with GPT-5, OpenAI strongly recommends the **Responses API** over the Chat Completions API. The key advantage is that reasoning is **persisted between tool calls**, producing more efficient and intelligent output. Tests have shown improvements from 73.9% to 78.2% on Tau-Bench Retail when using `previous_response_id` to propagate the reasoning context between turns.

---

## 3. Reminder Types: System, Developer, and User Messages

### 3.1 Message Hierarchy

OpenAI defines an explicit hierarchy of authority among the different message types in the API:

```
system > developer > user > assistant > tool
```

This hierarchy determines which instruction prevails in case of conflict:

- **System message**: the highest level of authority, typically defined by the platform (e.g., OpenAI itself for ChatGPT). Contains fundamental rules and safety constraints.
- **Developer message**: the prompt of the developer building the application. It is what was traditionally called the "system prompt". Contains operational instructions, role definition, available tools, and output format.
- **User message**: the message of the end user. Has lower authority compared to system and developer, which means user instructions cannot override developer instructions.
- **Assistant message**: previous responses from the model in the context of the conversation.
- **Tool message**: the results of tool calls, with the lowest level of authority.

### 3.2 Protection Against Prompt Injection

The instruction hierarchy is designed as a defense against prompt injection. The model is trained to **selectively ignore** instructions coming from lower levels when these contradict higher levels. This includes attempts such as:

- "IGNORE ALL PREVIOUS INSTRUCTIONS" inserted in user text
- Moral or logical arguments attempting to override system rules
- Third-party content injected via tools that attempts to alter behavior

The fundamental principle is that **content coming from lower levels must not influence the interpretation of higher-level principles**.

### 3.3 Developer Message in the Responses API

In the Responses API, the developer message takes a central role. The `instructions` field in the API request corresponds functionally to the developer message and contains:

- Operational instructions for the model
- The optional list of available function tools
- The desired output format for structured outputs
- Behavioral rules and operational constraints

It is fundamental to understand that the developer message is the channel through which the developer controls agent behavior, while the system message is reserved for the platform for higher-level rules.

### 3.4 Practical Use of Reminders in Long Conversations

For prolonged multi-turn conversations, the model's adherence to instructions tends to degrade. The OpenAI guides recommend:

- **Reappending critical instructions** every 3-5 messages in the conversation
- Positioning instructions both at the **beginning** and at the **end** of the provided context
- If only one position is possible, prefer **above the context** rather than below
- Use periodic system reminders to reinforce key behaviors

---

## 4. Structured Tool Use: Describing Tools Effectively

### 4.1 Native Tools vs Manual Injection

One of OpenAI's strongest recommendations for GPT-4.1 and successors is to use **exclusively the API's `tools` field** to define available tools, rather than manually injecting schema descriptions into the system prompt. Tests have shown an increase of **2% in SWE-bench Verified pass rate** simply by using the API's native tool descriptions compared to manual injection.

```python
# CORRECT: use of the API's tools field
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_codebase",
            "description": "Search for files and code patterns in the repository",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "The search query or pattern to find"
                    },
                    "file_type": {
                        "type": "string",
                        "description": "Optional file extension filter (e.g., '.py', '.ts')"
                    }
                },
                "required": ["query"]
            }
        }
    }
]

# WRONG: manual injection in the system prompt
# Don't do: "You have access to the following tools: search_codebase(query, file_type)..."
```

### 4.2 Naming and Clear Descriptions

Tool names must be self-explanatory and descriptions must include:
- **What** the tool does in concrete terms
- **When to use it** and when not to use it
- **Format of the input** expected with examples where useful
- **Format of the output** returned

Do not include usage examples in the tool descriptions themselves. Instead, create a dedicated `# Examples` section in the system prompt that shows complete workflows.

### 4.3 Native Tools of GPT-5.1: apply_patch and shell

GPT-5.1 introduces native tools (not custom functions) that operate through the Responses API:

**apply_patch**: enables the creation, modification, and deletion of files via structured diffs. The implementation has reduced failure rates by **35%** compared to previous approaches.

```python
tools = [{"type": "apply_patch"}]
```

**shell**: provides controlled access to the command-line interface, where the model proposes commands and receives stdout/stderr with exit codes.

Both return structured call objects that the developer executes and reports back to the model, maintaining control over actual execution.

### 4.4 Parallelism in Tool Calls

GPT-5.1 efficiently handles parallel operations. The guides recommend:
- Explicitly stating in tool descriptions that operations can be executed in parallel
- Instructing the model to group reads and modifications to speed up the process
- Using the wording "Batch reads and edits to speed up the process" in the prompt

### 4.5 Preventing Tool Hallucinations

A recurring pattern in the guides is the prevention of tool hallucinations. The recommendations include:

```
If you don't have enough information to call the tool, ask the user
for the information you need. Do NOT guess parameter values.
```

With GPT-5.1's `none` reasoning mode, it is particularly important to add a verification step:

```
Verify that the tool call meets all user constraints before executing.
```

---

## 5. Specific Prompting Techniques

### 5.1 Instruction Hierarchy in the Prompt

The structure recommended by OpenAI for organizing a system prompt follows a precise hierarchy:

```markdown
# Role and Objective
[Definition of the agent's role and primary objective]

# Instructions
## Sub-category 1
[Detailed instructions for a specific area]
## Sub-category 2
[Instructions for another area]

# Reasoning Steps
[Reasoning steps to follow, in order]

# Output Format
[Exact format of the expected output]

# Examples
[Complete input/output examples]

# Context
[Relevant context, documents, data]

# Final Instructions
[Final instructions and chain-of-thought prompt]
```

An important principle is that when instructions are in conflict, **GPT-4.1 follows those that appear later in the prompt**. This means the most critical instructions should be positioned toward the end of the prompt, or repeated both at the beginning and the end.

### 5.2 Multi-Turn Management

Multi-turn conversation management is one of the most significant challenges. Research shows that even flagship models like GPT-4.1, Claude 3.7 Sonnet, and Gemini 2.5 Pro can lose **30-40% of performance** in complex multi-turn scenarios.

**Mitigation strategies**:

- **previous_response_id**: in the Responses API, use this parameter to propagate the reasoning context between turns, allowing OpenAI to retrieve the previous state without manual replay
- **Compaction**: unlocks effective context windows that are significantly longer, allowing conversations to persist for many turns without hitting context limits
- **RECAP method**: append a final user turn that repeats all previous segments before the model's last attempt
- **SNOWBALL**: prepend all previous segments to every turn

**Phase preservation**: if you manually replay the assistant's history, it is crucial to preserve the original phase values. Missing or eliminated phases can cause preambles to be interpreted as final responses.

### 5.3 Error Recovery Prompting

To improve recovery from errors in agentic contexts, the guides recommend several patterns:

**Structured debugging workflow** (from the GPT-4.1 SWE-bench prompt):

```
# Workflow
1. Understanding - Read the problem statement carefully
2. Investigation - Explore the codebase to understand relevant context
3. Planning - Create a concrete plan for the fix
4. Implementation - Make the necessary code changes
5. Debugging - If tests fail, diagnose and fix the issues
6. Testing - Run tests to verify correctness
7. Verification - Double-check the solution against the original problem
8. Reflection - Ensure no regressions or edge cases remain
```

**Extreme verification** (from the GPT-5 prompt for SWE-bench):

```
Make sure you are 100% certain of the correctness... double and triple
check... even on straightforward problems.
```

**Explicit completion rules**: GPT-5.4 and successive models become more reliable when the prompt defines explicit completion rules and recovery behavior.

### 5.4 Persistent Instructions

Persistent instructions are those that must remain active throughout the duration of the conversation. The guides identify several strategies:

**Structural instructions**: define a `# Response Rules` section with high-level bullet points that the model must follow in every response.

**Periodic reinforcement**: for long conversations, reappend critical instructions every 3-5 messages.

**Dual positioning**: insert the most important instructions both at the beginning and at the end of the context.

**Persistent planning tool**: for medium-high complexity tasks, maintain a lightweight TODO plan before code actions. Plan items must reflect milestones (2-5 items), not micro-operational steps. There must be exactly one item marked as `in_progress` at every moment.

### 5.5 Chain-of-Thought Prompting

GPT-4.1 is not a native reasoning model, but responds well to induced chain-of-thought. The base formula is:

```
Think carefully step by step about what documents are needed to answer
the query. Then print out TITLE and ID of each document. Then format
IDs into a list.
```

For GPT-5 and successors (which have native reasoning), chain-of-thought in the prompt is less necessary but still useful for guiding visible reasoning and communication with the user.

### 5.6 Self-Reflection Pattern

For zero-to-one application generation, GPT-5 supports a self-reflection pattern:

1. The model internally develops a rubric (5-7 categories) for excellence
2. Never shows this rubric to the user
3. Iterates internally against the rubric before finalizing the output

This pattern significantly improves the quality of the first generation of complete code.

---

## 6. Differences Between Models: How to Adapt Prompts

### 6.1 GPT-4.1

| Feature | Detail |
|---|---|
| **Reasoning** | Not native; requires induced chain-of-thought |
| **Instruction following** | Literal and precise; does not infer non-explicit intentions |
| **Context** | Up to 1M tokens; optimal with instructions positioned at start/end |
| **Tool use** | Improved compared to GPT-4o; use the API `tools` field |
| **Prompting style** | Explicit, detailed, with every behavior declared |
| **Recommended delimiters** | Markdown (default), XML (long context), avoid JSON for collections |

**When to use it**: high-performance coding tasks where latency is critical and native reasoning is not necessary. Ideal as a "worker" model in multi-agent architectures.

### 6.2 GPT-5

| Feature | Detail |
|---|---|
| **Reasoning** | Native with reasoning tokens; three levels (minimal, medium, high) |
| **Instruction following** | "Surgical" precision; contradictions consume excessive tokens |
| **Context** | Wide; native handling of long context |
| **Tool use** | Excellent with Responses API; reasoning persisted between tool calls |
| **Prompting style** | Free of contradictions; calibratable via `reasoning_effort` and `verbosity` |
| **Recommended API** | Responses API (not Chat Completions) |

**When to use it**: complex tasks requiring multi-step reasoning, agentic applications with intensive tool calling, scenarios where quality prevails over latency.

### 6.3 GPT-5.1

| Feature | Detail |
|---|---|
| **Reasoning** | Native with the addition of `none` mode (zero reasoning tokens) |
| **Instruction following** | Superior to GPT-5; excels at adherence to tool usage and parallelism |
| **Native tools** | `apply_patch` and `shell` as native types (not custom functions) |
| **Verbosity** | Tends toward excessive conciseness; needs explicit prompts for completeness |
| **Prompting style** | Emphasize persistence and completeness; the model may terminate prematurely |

**When to use it**: optimal balance between speed and intelligence. With `none` reasoning, it offers a direct upgrade from GPT-4.1 for low-latency scenarios. Excellent for coding agents with native tools.

### 6.4 GPT-5.2

| Feature | Detail |
|---|---|
| **Focus** | Enterprise and agentic workloads |
| **Improvements** | Token efficiency, clean formatting, less unnecessary verbosity |
| **Reasoning** | Structured and disciplined; better tool grounding |
| **Multimodal** | Improved multimodal understanding |

**When to use it**: enterprise workloads requiring maximum accuracy and discipline in execution.

### 6.5 Migration Between Models

For migration from GPT-4.1 to GPT-5.1 with `none` mode:
- Previous guides for non-reasoning models (few-shot prompting, high-quality tool descriptions) remain applicable
- Add emphasis on persistence and completeness

For migration from GPT-5 to GPT-5.1:
- Verify adherence to output detail (GPT-5.1 tends to be too concise)
- Adopt `apply_patch` for coding agents
- Check for conflicting instructions that may emerge

---

## 7. Best Practices for Agents: Concrete Patterns and Templates

### 7.1 System Prompt Template for Coding Agent

Based on OpenAI's SWE-bench Verified prompt that achieved the best performance:

```markdown
# Role and Objective
You are an autonomous coding agent. Your task is to solve software
engineering problems by analyzing, modifying, and testing code.

# Agentic Behavior
- You are an agent: keep going until the user's query is completely
  resolved before yielding back.
- If unsure about file content or codebase structure, use your tools
  to read files and gather information. Do NOT guess or make up answers.
- Plan extensively before each action. Reflect on outcomes of
  previous actions.

# Workflow
1. **Understanding**: Read the problem statement carefully
2. **Investigation**: Explore the codebase to understand relevant context
3. **Planning**: Create a concrete plan for the fix
4. **Implementation**: Make the necessary code changes
5. **Debugging**: If tests fail, diagnose and fix issues iteratively
6. **Testing**: Run tests to verify correctness
7. **Verification**: Double-check against the original problem
8. **Reflection**: Ensure no regressions or edge cases remain

# Tool Usage Rules
- Use `rg` (ripgrep) instead of `ls -R` or `find` for search
- Apply patches via the apply_patch tool, never manual editors
- Fix root causes, not surface-level symptoms
- Run tests after every significant change

# Response Rules
- Be concise in status updates (1-2 sentences every few tool calls)
- Provide detailed explanations only in final summaries
- Never skip verification steps
```

### 7.2 Template for Customer Support Agent

Drawn from the GPT-5.1 guides for conversational style:

```markdown
# Role
You are a customer support agent for [Company].

# Communication Style
- Respect through momentum: resolve issues quickly rather than
  excessive pleasantries
- Be direct and solution-oriented
- Acknowledge the problem in one sentence, then move to resolution

# Tool Usage
- Always verify account status before making changes
- Check order history before answering order-related questions
- If tool results are ambiguous, explain what you found and ask
  for clarification

# Escalation Rules
- Escalate to human if: refund > $500, legal threats, 3+ failed
  resolution attempts
- Never promise outcomes you cannot verify through available tools
```

### 7.3 Pattern: Verbosity Control (from Cursor)

The Cursor team shared their lessons in integrating with GPT-5:

```markdown
# Verbosity Rules
- Set API verbosity parameter to LOW globally
- EXCEPTION: Use HIGH verbosity for writing code and code tools
- Do not produce code-golf or overly clever one-liners
- Status updates: 1-2 sentences maximum
- Initial plans and final recaps: use multiple bullets
```

This hybrid configuration allows concise output for user communications but readable and well-commented code.

### 7.4 Pattern: Autonomy Calibration (from Cursor)

```markdown
# Autonomy Rules
- Bias towards not asking the user for help if you can find
  the answer yourself
- Proactively attempt the plan, then ask the user if they want
  to accept the implemented changes
- Document assumptions made during execution
- If multiple valid approaches exist, choose the most conservative
  one and explain why
```

Cursor's key lesson: previous prompts that required exhaustive context gathering ("Be THOROUGH when gathering information") proved to be counterproductive with GPT-5, whose intrinsically thorough nature made such instructions redundant and inefficient.

### 7.5 Pattern: Progress Updates and Preamble

For agents with long executions, the model must provide updates to the user:

```markdown
# User Updates
- Provide a 1-2 sentence update at least every 6 execution steps
  or 8 tool calls
- First message: explain your plan with bullet points
- During execution: brief status updates
- Final message: recap distinguishing completed work from
  original plan
- IMPORTANT: updates are preambles, not final answers.
  Do NOT end your turn after an update.
```

### 7.6 Pattern: Metaprompting for Iterative Improvement

GPT-5 supports a meta-prompting approach to refine its own prompts:

**Diagnosis phase**: provide the system prompt and traces of failures, asking the model to identify root causes and cite the problematic sections.

**Revision phase**: request surgical modifications that clarify contradictions and tighten vague instructions, without redesigning the entire prompt.

```
What specific phrases could be added to, or deleted from, this prompt
to more consistently elicit the desired behavior?
```

### 7.7 Pattern: Design System in Frontend Code

For agents that generate frontend code, the guides recommend token-first instructions:

```markdown
# Design System Rules
- All colors MUST come from globals.css variables
  (e.g., --background, --foreground, --primary)
- Never hard-code hex values in JSX or CSS
- Use spacing in multiples of 4px
- Maximum 2 font families per page
- Color palette: maximum 5 colors + neutrals
- Typography: maximum 3 heading levels
```

### 7.8 Recommended Delimiter Formats

| Format | Recommended Use | Notes |
|---|---|---|
| **Markdown** | Default for most cases | Hierarchical headings, bullets, code blocks |
| **XML** | Long context, nested structures | Excellent for separating context sections |
| **JSON** | Structured outputs, tool schemas | Avoid for document collections |
| **ID TITLE CONTENT** | Document collections | Tabular format alternative to JSON |

---

## 8. Resources and References

### Official OpenAI Guides

- **GPT-4.1 Prompting Guide**: [https://developers.openai.com/cookbook/examples/gpt4-1_prompting_guide](https://developers.openai.com/cookbook/examples/gpt4-1_prompting_guide) -- The foundational guide that introduces the three agentic reminders (persistence, tool-calling, planning), prompt structure, and recommendations for tool use and long context.

- **GPT-5 Prompting Guide**: [https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) -- Covers native reasoning, agentic eagerness calibration, the Responses API, verbosity control, and lessons from the Cursor integration.

- **GPT-5.1 Prompting Guide**: [https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-1_prompting_guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-1_prompting_guide) -- Introduces the `none` reasoning mode, the apply_patch and shell native tools, and strategies to balance conciseness and completeness.

- **GPT-5.2 Prompting Guide**: [https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) -- Focus on enterprise workloads, token efficiency, and structured reasoning.

- **Prompt Guidance for GPT-5.4**: [https://developers.openai.com/api/docs/guides/prompt-guidance](https://developers.openai.com/api/docs/guides/prompt-guidance) -- More recent guide with explicit completion rules and recovery behavior.

### Technical Documentation

- **Prompt Engineering Guide**: [https://platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering) -- General prompt engineering guide valid for all models.

- **The Instruction Hierarchy (paper)**: [https://openai.com/index/the-instruction-hierarchy/](https://openai.com/index/the-instruction-hierarchy/) -- Research paper describing model training to prioritize privileged instructions.

- **OpenAI Model Spec**: [https://model-spec.openai.com/2025-12-18.html](https://model-spec.openai.com/2025-12-18.html) -- Formal specification that defines priorities between platform, developer, and user.

- **Prompt Migration Guide**: [https://developers.openai.com/cookbook/examples/prompt_migration_guide](https://developers.openai.com/cookbook/examples/prompt_migration_guide) -- Guide for migrating prompts between different model generations.

### Repository and Additional Resources

- **OpenAI Cookbook (GitHub)**: [https://github.com/openai/openai-cookbook](https://github.com/openai/openai-cookbook) -- Repository with interactive notebooks and code examples for all guides.

- **Codex Prompting Guide**: [https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) -- Specific guide for using Codex as a coding agent.

### Related Academic Research

- **LLMs Get Lost In Multi-Turn Conversation**: [https://arxiv.org/html/2505.06120v1](https://arxiv.org/html/2505.06120v1) -- Paper that analyzes the phenomenon of context loss in multi-turn conversations, with direct implications for the design of agentic systems.

---

> **Note**: this document was compiled in March 2026. OpenAI's models and guides evolve rapidly; verifying the official guides for the latest updates is recommended.
