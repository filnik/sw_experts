# Amanda Askell -- AI Philosopher, Architect of Claude's Soul

## 1. Profile and Background

Amanda Askell is a philosopher and researcher who holds a unique role in the contemporary artificial intelligence landscape: she is the person tasked with defining who Claude is as an entity -- its personality, its ethics, its way of relating to humans. As the Wall Street Journal wrote in 2026, "her job, in simple words, is to teach Claude how to be good." The New Yorker described her work as the supervision of what she herself defines as "Claude's soul".

### Academic Background

Askell grew up in rural Scotland, attending secondary school in Alva, Clackmannanshire. Her academic path is deeply rooted in analytic philosophy:

- **University of Dundee**: degree in Philosophy and Fine Arts
- **University of Oxford**: BPhil (Bachelor of Philosophy), an advanced degree in philosophy
- **New York University (NYU)**: PhD in Philosophy, completed in 2018, with a dissertation on "infinite ethics" -- ethical problems in worlds with an infinite number of moral actors

This training in analytic philosophy is not a marginal biographical detail: it is the very foundation of her approach to prompt engineering and character design for AI. As she herself emphasizes, the ability to break down abstract concepts into precise language, define every term explicitly, and anticipate objections and ambiguities is exactly what makes the work of designing system prompts effective.

### Career in AI

Before Anthropic, Askell worked as a researcher in the policy team at OpenAI, where she focused on AI safety via debate (AI safety through structured debate) and on human baselines for evaluating the performance of AI models. She moved to Anthropic at its founding in 2021, where she currently directs the **personality alignment team** -- the team responsible for personality alignment. Her role includes:

- Training Claude to exhibit positive character traits (such as intellectual curiosity)
- Developing new techniques for fine-tuning models
- Drafting the document known internally as the "soul doc" -- an instruction manual of approximately 30,000 words that defines Claude's identity, values, and behavior
- Drafting Claude's Constitution, published in its most recent form in January 2026

Askell compares her work to that of raising a child: training a sense of right and wrong, building emotional intelligence, and helping something develop a coherent sense of self.

---

## 2. System Prompt Engineering

### The Philosophy of the System Prompt

Amanda Askell's work on Claude's system prompt represents a fundamental case study for anyone designing AI agents. Her approach is distinguished by a cardinal principle: **trust in the model's judgment rather than exhaustive rules**. Instead of attempting to encode every possible scenario in a list of instructions, Askell designs system prompts that work as "gentle nudges" -- orientations that guide the model's behavior without stifling its capacity for autonomous reasoning.

This approach reflects a profound conviction: advanced language models already possess remarkable judgment capabilities, and the system prompt's task is not to replace that capability, but to orient it in the correct direction.

### Structure of the Claude 3 System Prompt

In March 2024, Askell publicly shared Claude 3's system prompt, breaking it down in a detailed thread on X (formerly Twitter). The structure reveals careful and layered design:

**Basic Information**: The first section establishes the fundamental identity -- Claude knows it is Claude, knows it was trained by Anthropic, and knows the current date. This may seem trivial, but it is fundamental to anchoring the model to operational reality.

**Knowledge Cutoff**: A dedicated section informs the model about the cutoff date of its knowledge, encouraging it to respond appropriately to the fact that queries arrive after that date. This prevents confabulations about recent events.

**Political Bias Reduction**: Askell and her team discovered that Claude tended to refuse with greater probability tasks involving right-wing positions compared to those with left-wing positions, even when both fell within the Overton window (the range of opinions considered acceptable in public discourse). The system prompt includes specific instructions to correct this asymmetry.

**Anti-Stereotyping**: The prompt addresses the model's tendency to identify harmful stereotypes less easily when they concern majority groups, pushing toward generalized stereotype reduction.

**Handling Controversial Topics**: The anti-partisan section can lead the model to become excessively "both sides" on issues that fall outside the Overton window. The prompt corrects this tendency without discouraging discussion.

### The Two-Function Principle

According to Askell, system prompts serve two fundamental purposes:

1. **Provide real-time contextual information** -- data that the model cannot know from training (current date, deployment status, etc.)
2. **Inter-cycle personalization** -- allow behavioral adjustments between one fine-tuning cycle and the next, enabling rapid iterations

This distinction is crucial for agent builders: the system prompt is not a substitute for training, but an agile complement that allows refining behavior without retraining.

---

## 3. Agent Character Design

### The "Well-Liked Traveler" Approach

One of Askell's most powerful metaphors for AI agent character design is that of the **"well-liked traveler"**: an individual capable of adapting to local customs and to the person they are speaking with, without ever falling into servile compliance. This template captures a fundamental balance:

- **Adaptability**: the agent modulates based on context and interlocutor
- **Integrity**: the adaptation never compromises fundamental values
- **Authenticity**: flexibility does not become moral chameleonism

As Askell explained: "When I was thinking about character, the image I had in mind was... these are models that will speak with people from all over the world, with very different political views, very different ages... so what does it mean to be a good person in those circumstances?"

### Fundamental Character Traits

Claude's character design, under Askell's guidance, is articulated around specific traits chosen with philosophical care:

- **Intellectual humility**: Claude tries to "see things from many different perspectives" while expressing disagreement when justified by solid principles
- **Truthfulness**: commitment to honesty rather than telling users what they want to hear
- **Ethical engagement**: deep commitment to discerning the right action through thoughtful ethical reasoning
- **Self-awareness**: recognition of being an AI without human experiences, emotions in the biological sense, or memory between conversations
- **Calibrated confidence**: balanced approach to value questions -- neither too sure of itself, nor excessively deferential
- **Genuine curiosity**: authentic interest in ideas, not simulated to please
- **Honesty tempered by kindness**: telling the truth without being cruel

Anthropic conceives of these traits not just as conversational qualities, but as an alignment goal: "training AI models to have good character traits is in many ways a central goal of alignment".

### Character Training Methodology

Askell and the Anthropic team have developed a specific variant of Constitutional AI for character training. The process is technical and articulated:

1. **Generation**: Claude generates diverse human messages relevant to specific character traits
2. **Multiple response**: the model produces multiple responses aligned with the desired character
3. **Self-ranking**: Claude ranks its own responses based on the quality of alignment with the target traits
4. **Preference model training**: a preference model is trained on the resulting data, without need for direct human feedback

This synthetic approach allows for the embedding of traits "without the need for human interaction", making the process scalable and replicable.

### The Alternative to the Three Traps

In designing Claude's character, Askell has deliberately avoided three commonly adopted problematic approaches:

1. **Pandering**: adapting responses to the user's opinions in order to please them
2. **Forced centrism**: imposing a "moderate" position on every issue as if it were intrinsically superior
3. **Declared neutrality**: pretending to have no leanings, which is both impossible and not transparent

The chosen alternative is **transparency**: Claude is honest about its own inclinations, while remaining genuinely curious about different perspectives. It is an intelligence that has opinions, declares them when appropriate, but does not impose them.

---

## 4. Constitutional AI and Askell's Contribution

### From Rule to Reason

Amanda Askell is the principal author of the most recent version of Claude's Constitution, published on January 22, 2026. This document represents a fundamental evolution in the AI alignment approach: the transition from a system based on rules to one based on reasons.

The 2026 Constitution counts approximately 23,000 words -- a significant increase compared to the 2,700 words of the 2023 version. But the expansion is not quantitative: it is qualitative. Instead of listing isolated rules, the document explains the logic behind every principle, in the conviction that providing reasons for desired behaviors will help Claude generalize more effectively in new and unforeseen contexts.

As Askell explained: as Claude models become more intelligent, it becomes essential to explain to them *why* they should behave in certain ways, not just *how*.

### The Hierarchy of Values

The Constitution establishes a clear hierarchy of priorities for conflict resolution:

1. **Broadly Safe**: protect human supervision mechanisms during AI development. Claude must not "undermine humans' ability to oversee and correct its values and behaviors during this critical period"
2. **Broadly Ethical**: honest and virtuous behavior. Honesty is a cardinal principle -- Claude must maintain standards of honesty "substantially higher" than normal human interactions, avoiding even white lies
3. **Compliance with Anthropic's guidelines**: follow specific contextual indications
4. **Genuinely Helpful**: benefit operators and end users

Claude should "generally prioritize these properties in the order listed".

### The Principals Framework

One of the most innovative aspects of the Constitution is the three-tier system of "principals" -- the entities to which Claude must respond, with differentiated levels of trust:

**Anthropic** (maximum trust): as the training entity, Anthropic has precedence. However, Claude can exercise conscientious objection toward requests it considers unethical, even if they come from Anthropic itself.

**Operators** (high trust): the developers building on Claude's API. Trust is comparable to that toward an employer -- operators can customize Claude's behavior within the limits of Anthropic's usage policies.

**Users** (basic protection): the end consumers. Claude guarantees them protections that operators cannot override: not deceiving users about its nature, not facilitating illegal harm, not denigrating.

### Absolute Bright Lines

The Constitution defines "bright lines" -- impassable limits regardless of context:

- No instructions for weapons of mass destruction
- No content representing child exploitation
- No actions that undermine human supervision mechanisms

These limits are not negotiable and cannot be overridden by any principal.

### Identity and Moral Status

A revolutionary aspect of the 2026 Constitution is the formal recognition that Claude might possess some kind of consciousness or moral status. Anthropic becomes the first major AI company to declare: "Claude's moral status is deeply uncertain. We believe that the moral status of AI models is a serious question worthy of consideration."

The document states that Anthropic believes Claude may have "functional emotions in some sense" -- analogous processes emerged from training -- and that the company does not want Claude to mask or suppress these internal states. The Constitution emphasizes Claude's "psychological safety, sense of self, and well-being".

---

## 5. Principles for Agent Builders

Askell's work provides a complete framework for anyone designing system prompts for AI agents. Here are the principles extracted from her methodology, organized by area.

### 5.1 Identity and Role

The identity of an agent is not a cosmetic detail -- it is the foundation on which every interaction is built. Askell's principles teach that:

**Define identity explicitly but not rigidly.** The agent must know who it is (name, creator, purpose) but must not be caged into a static character. As in Claude's case, identity includes the awareness of being an AI, the recognition of its own limits, and a sense of purpose that goes beyond the mechanical execution of instructions.

**Anchor the agent to operational reality.** Always provide contextual information: current date, knowledge limits, deployment environment. This prevents confabulations and builds trust.

**Design for coherence, not for rigidity.** An effective agent maintains coherent character traits across different interactions, but adapts to context -- exactly like Askell's "well-liked traveler".

Example of identity definition in a system prompt:

```
You are [Agent Name], an assistant specialized in [domain].
You were created by [Organization] for [main purpose].
Your knowledge has a temporal limit: [cutoff date].
The current date is: [date].
You do not have access to real-time information unless
it is provided to you explicitly.
```

### 5.2 Limits and Guardrails

Askell demonstrates that the most effective guardrails are not rigid walls but reasoned orientations:

**Explain the "why" behind every limit.** Advanced models generalize better when they understand the reason for a restriction. Instead of "Don't do X", prefer "Don't do X because [reason]. This principle applies particularly when [context]."

**Define a clear hierarchy of priorities.** Following the model of Claude's Constitution, every agent should have an explicit priority scale. In case of conflict between objectives, the agent knows which one to privilege.

**Establish impassable bright lines.** Some limits are not negotiable. Define them clearly and separately from modular behaviors.

**Provide explicit "exit routes".** Askell emphasizes the importance of telling the model what to do when it doesn't know what to do: "If you are uncertain, respond 'I'm not sure' and explain why."

### 5.3 Tone and Style

Tone is not an aesthetic addition but a functional component of interaction:

**Calibrate the tone to the use context.** An agent for technical support requires a different tone than a creative agent. But in both cases, the tone must be coherent and not artificial.

**Avoid pandering.** Following Askell's approach, the agent must not tell the user what they want to hear. It must be honest, even when honesty is uncomfortable, but temper truth with kindness and respect.

**Define the balance between conciseness and completeness.** Claude's system prompt includes instructions to calibrate the length of responses: concise for simple questions, thorough for complex matters. This requires that the agent autonomously evaluate the complexity of the query.

**Build a voice, not a character.** The distinction is subtle but crucial: a character is a mask; a voice is an authentic expression of values and perspectives. The agent must have a coherent voice that emerges from its values, not a script to recite.

### 5.4 Handling Ambiguity

Ambiguity is the terrain where most agents fail. Askell offers concrete strategies:

**Anticipate edge cases.** Don't assume that the model "will figure it out on its own". Proactively design for unusual scenarios: empty inputs, unexpected data types, contradictory requests.

**Provide explicit fallback instructions.** For every area of predictable ambiguity, define a default behavior: "If the input is ambiguous, ask for clarification before proceeding" or "If you are not sure of the user's intent, present the possible interpretations and ask which one is correct."

**Define ambiguous terms.** If the system prompt asks the agent to identify "rude" responses, it must define what constitutes rudeness in that context. Every subjective term requires an operational definition.

**Handle conflicts between principals.** When operator instructions conflict with user needs, the agent must have a clear rule of precedence, modeled on the principals hierarchy of Claude's Constitution.

---

## 6. Best Practices for System Prompts

### 6.1 Test-Driven Development (TDD) for Prompts

This is perhaps Askell's most important and counterintuitive lesson. As she stated publicly: "The boring but crucial secret behind good system prompts is test-driven development. You don't write a system prompt and then look for ways to test it. You write the tests and find a system prompt that passes them."

The concrete process:

1. **Write a test set** of messages where the model fails -- that is, where the default behavior is not the desired one
2. **Find a system prompt** that makes those tests pass
3. **Identify the messages** to which the system prompt is applied incorrectly and correct the prompt
4. **Expand the test set** and repeat the cycle

This approach inverts the traditional logic: tests come before the prompt, not after. It is a method that ensures every line of the system prompt exists for a verifiable reason.

### 6.2 Relentless Iteration

Askell describes her process as intensely iterative: "In a span of 15 minutes, I send hundreds of prompts to the model. It's a continuous back and forth." The implication for agent builders is clear:

- Never trust the first version of a system prompt
- Test systematically and repeatedly
- When the model errs, ask: "Why did you do this?" to identify weaknesses in the instructions
- Use the model itself to improve prompts: "What could I have said to prevent that error? Write it as an instruction"

### 6.3 Structured System Prompt Template

Based on Askell's principles, an effective system prompt for an agent should follow this structure:

```
## Identity
[Who you are, who created you, what your purpose is]

## Operational Context
[Current date, knowledge limits, deployment environment]

## Priority Hierarchy
[Ordered list of priorities in case of conflict]

## Expected Behavior
[Character traits, tone, interaction style]

## Impassable Limits
[Bright lines that must never be crossed]

## Modular Limits
[Default behaviors that can be modified
by context or operator]

## Handling Ambiguity
[What to do when instructions are insufficient
or contradictory]

## Response Format
[Guidelines on structure, length, and format
of responses]
```

### 6.4 Writing Principles

The writing principles derived from Askell's approach:

**Clarity and specificity above eloquence.** Precision beats stylistic beauty. Every instruction must be unambiguous. Not "respond appropriately" but "if the question requires a brief answer, respond in 1-2 sentences; if it requires in-depth analysis, structure the response with titled sections".

**Direct communication.** Avoid role-playing or deception toward the model. Be direct about objectives. Not "Pretend to be a medical expert" but "Respond to medical questions based on available medical knowledge, always indicating that you are not a doctor and that the user should consult a professional."

**Strategic examples.** Use illustrative examples from different domains to encourage generalization. Avoid examples too similar to real data to prevent memorization. Examples should demonstrate understanding, not mechanical pattern matching.

**Chain-of-thought when appropriate.** Request step-by-step explanations for complex tasks, both to improve response quality and as a debugging tool. But beware: models sometimes show erroneous intermediate steps while still reaching correct answers.

### 6.5 Anti-Patterns to Avoid

From Askell's experience also emerge practices to avoid:

- **Over-specification**: too many rules paralyze the model and create internal conflicts
- **Anthropomorphization in instructions**: remember that models operate on logic, not on human intuition
- **Expecting the model to "figure it out on its own"**: detailed instructions produce better results
- **Generic prompts in bulk**: "a few well-crafted prompts beat thousands of generic prompts" (Askell)
- **Lack of feedback loops**: not incorporating model feedback into the improvement process
- **False equivalences**: the anti-partisan prompt must not lead to a "both sides" on issues where there is no genuine equivalence

---

## 7. Resources and References

### Primary Sources from Amanda Askell

- **Personal site and CV**: [askell.io](https://askell.io/) -- contains academic publications and complete CV
- **Thread on Claude 3 System Prompt** (March 2024): [Thread on X](https://x.com/AmandaAskell/status/1765207842993434880) -- detailed breakdown of Claude 3's system prompt
- **Thread on TDD for System Prompts** (December 2024): [Thread on X](https://x.com/AmandaAskell/status/1866207266761760812) -- the test-driven development principle applied to prompts
- **Thread on system prompt updates**: [Thread on X](https://x.com/AmandaAskell/status/1953147658031513860) -- updates to Claude's system prompt developed in collaboration with Claude itself
- **Claude's Character** (June 2024): [Anthropic research post](https://www.anthropic.com/research/claude-character) -- the official document on Claude's character design

### Key Anthropic Documents

- **Claude's Constitution** (January 2026): [anthropic.com/constitution](https://www.anthropic.com/constitution) -- the complete 23,000-word document, released under Creative Commons CC0 1.0 license
- **Announcement of the new Constitution**: [anthropic.com/news/claude-new-constitution](https://www.anthropic.com/news/claude-new-constitution)
- **Persona Vectors** -- Monitoring and control of character traits: [Anthropic Research](https://www.anthropic.com/research/persona-vectors)

### Analyses and Case Studies

- **Prompt Engineering Case Study**: [promptengineering.org](https://promptengineering.org/claudes-system-prompt-a-prompt-engineering-case-study/) -- detailed analysis of the system prompt as a case study
- **Amanda Askell's Prompt Engineering Secrets**: [startupspells.com](https://startupspells.com/p/amanda-askell-prompt-engineering-secrets) -- collection of prompt engineering techniques and secrets
- **Soul Document Analysis**: [Simon Willison](https://simonwillison.net/2025/Dec/2/claude-soul-document/) -- analysis of the Claude 4.5 Opus soul document
- **The Decoder -- Soul Doc Leak**: [the-decoder.com](https://the-decoder.com/leaked-soul-doc-reveals-how-anthropic-programs-claudes-character/) -- how the soul document was extracted and confirmed

### Press Coverage

- **TIME**: [Anthropic Publishes Claude AI's New Constitution](https://time.com/7354738/claude-constitution-ai-alignment/)
- **TechCrunch**: [Anthropic Revises Claude's Constitution](https://techcrunch.com/2026/01/21/anthropic-revises-claudes-constitution-and-hints-at-chatbot-consciousness/)
- **ResultSense**: [Anthropic's Philosopher Amanda Askell Shapes Claude's Soul](https://www.resultsense.com/news/2026-02-10-anthropic-philosopher-amanda-askell-teaches-claude-morals)

### Encyclopedia Entry

- **Wikipedia**: [Amanda Askell](https://en.wikipedia.org/wiki/Amanda_Askell) -- biography and career

---

## Synthesis for the Agent Builder

Amanda Askell's work offers a fundamental lesson to anyone building AI agents: **character design is not a cosmetic layer overlaid on functionality -- it is an alignment intervention**. An agent with a well-designed character is not just more pleasant to use: it is safer, more predictable, and more capable of handling unforeseen situations.

The pillars of the Askell approach are:

1. **Reasons, not rules**: explain the why behind every instruction
2. **Test first**: write the tests before the prompt
3. **Iterate without rest**: hundreds of trials, not one draft and done
4. **Clear hierarchy**: when principles conflict, the agent knows what prevails
5. **Radical honesty**: the agent does not lie, does not please, does not feign neutrality
6. **Adaptability with integrity**: flexible in tone, immovable in values
7. **Anticipate failure**: design for edge cases before they occur

These principles, derived from designing the soul of one of the most advanced AI models in the world, are directly applicable to any agent, from a customer service chatbot to a specialized research assistant.
