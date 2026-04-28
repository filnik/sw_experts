# Authoritative Resource Index: AI Agent Design & Orchestration

## PART 1: Experts and Thought Leaders (19 people)

### Foundational Academic Researchers

| # | Expert | Affiliation | Key Contribution | Primary Resource |
|---|---------|-------------|-------------------|---------------------|
| 1 | **Shunyu Yao** | Princeton / ex-OpenAI | **ReAct** paper (ICLR 2023) - reasoning+acting paradigm | [Paper](https://arxiv.org/abs/2210.03629), [PhD Dissertation](https://ysymyth.github.io/papers/Dissertation-finalized.pdf) |
| 2 | **Karthik Narasimhan** | Princeton | Co-author of ReAct, **SWE-bench**, **SWE-agent**, **CoALA** (Cognitive Architectures for Language Agents) | [Google Scholar](https://scholar.google.com/citations?user=euc0GX4AAAAJ) |
| 3 | **Noah Shinn** | Princeton | **Reflexion** (NeurIPS 2023) - verbal reinforcement learning for agents | [Paper](https://arxiv.org/abs/2303.11366), [GitHub](https://github.com/noahshinn/reflexion) |
| 4 | **Jason Wei** | OpenAI / ex-Google Brain | **Chain-of-Thought prompting** - foundational technique for agent reasoning (5000+ citations) | [Paper](https://arxiv.org/abs/2201.11903) |
| 5 | **Joon Sung Park** | Stanford | **Generative Agents** (UIST 2023, Best Paper) - 25 agents in social simulation with memory, reflection, planning | [Paper](https://arxiv.org/abs/2304.03442), [GitHub](https://github.com/joonspk-research/generative_agents) |

### Industry Leaders and Framework Builders

| # | Expert | Affiliation | Key Contribution | Primary Resource |
|---|---------|-------------|-------------------|---------------------|
| 6 | **Andrew Ng** | DeepLearning.AI | Defined the **4 agentic patterns** (Reflection, Tool Use, Planning, Multi-Agent) | [Course](https://www.deeplearning.ai/courses/agentic-ai/), [Blog](https://landing.ai/blog/andrew-ng-a-look-at-ai-agentic-workflows-and-their-potential-for-driving-ai-progress/) |
| 7 | **Harrison Chase** | LangChain (CEO) | Creator of **LangChain** and **LangGraph**, concepts of "context engineering" and "deep agents" | [Sequoia Podcast](https://sequoiacap.com/podcast/context-engineering-our-way-to-long-horizon-agents-langchains-harrison-chase/) |
| 8 | **Lilian Weng** | OpenAI | Seminal post **"LLM Powered Autonomous Agents"** - formula Agent = LLM + Memory + Planning + Tool Use | [Blog Post](https://lilianweng.github.io/posts/2023-06-23-agent/) |
| 9 | **Andrej Karpathy** | Independent / ex-OpenAI, Tesla | **"Software 3.0"** vision, concepts of "agentic engineering" and "context engineering" | [Blog](https://karpathy.bearblog.dev/year-in-review-2025/) |
| 10 | **Amanda Askell** | Anthropic | Author of **Claude's system prompt** and the Constitution - "agent character engineering" | [Case Study](https://promptengineering.org/claudes-system-prompt-a-prompt-engineering-case-study/) |
| 11 | **Erik Schluntz & Barry Zhang** | Anthropic | **"Building Effective Agents"** guide - taxonomy of composable patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) | [Guide](https://www.anthropic.com/research/building-effective-agents) |
| 12 | **Chi Wang** | Google DeepMind / ex-Microsoft | Creator of **AutoGen** - first framework for multi-agent conversations | [Paper](https://arxiv.org/abs/2308.08155) |
| 13 | **Joao Moura** | CrewAI (CEO) | Creator of **CrewAI** - role-based multi-agent orchestration (Crews + Flows) | [CrewAI](https://crewai.com/), [GitHub](https://github.com/crewaiinc/crewai) |
| 14 | **Jerry Liu** | LlamaIndex (CEO) | Concept of **Agentic RAG** - document-level agents orchestrated by a meta-agent | [DeepLearning.AI Course](https://www.deeplearning.ai/short-courses/building-agentic-rag-with-llamaindex/) |

### Educators and Community Builders

| # | Expert | Affiliation | Key Contribution | Primary Resource |
|---|---------|-------------|-------------------|---------------------|
| 15 | **Simon Willison** | Independent (co-creator of Django) | Coined "prompt injection", **"Agentic Engineering Patterns"** guide (2026) | [Guide](https://simonw.substack.com/p/agentic-engineering-patterns), [Blog](https://simonwillison.net/) |
| 16 | **Elvis Saravia** | DAIR.AI | **Prompt Engineering Guide** - the most comprehensive open-source resource for prompting techniques | [Guide](https://www.promptingguide.ai/), [GitHub](https://github.com/dair-ai/Prompt-Engineering-Guide) |
| 17 | **Chip Huyen** | Independent / ex-NVIDIA | **"AI Engineering"** book (O'Reilly 2025) - patterns for agents in production | [Book](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) |
| 18 | **Yohei Nakajima** | Untapped Capital | Creator of **BabyAGI** - first viral autonomous agent with task planning | [Blog](https://yoheinakajima.com/birth-of-babyagi/), [GitHub](https://github.com/yoheinakajima/babyagi) |
| 19 | **Swyx (Shawn Wang)** | Latent Space | Founder of the **Latent Space** podcast/newsletter, AI Engineer conference | [Latent Space](https://www.latent.space/) |

---

## PART 2: Official Guides from AI Companies (8 resources)

| # | Resource | Organization | What It Covers |
|---|---------|---------------|------------|
| 20 | [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) | Anthropic | Workflow vs agents taxonomy, 5 composable patterns, "simple over complex" philosophy |
| 21 | [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Anthropic | Context engineering as the successor to prompt engineering, "just in time" context loading |
| 22 | [Claude Cookbooks](https://github.com/anthropics/claude-cookbooks) | Anthropic | Practical recipes: tool use, agent skills, automatic compaction, multi-step workflows |
| 23 | [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) | OpenAI | Python-first multi-agent framework: handoffs, guardrails, MCP, tracing |
| 24 | [OpenAI Cookbook - Agents](https://cookbook.openai.com/topic/agents) | OpenAI | Practical notebooks: orchestration, routines/handoffs, Deep Research, context engineering |
| 25 | [GPT-4.1/GPT-5 Prompting Guides](https://cookbook.openai.com/examples/gpt4-1_prompting_guide) | OpenAI | Prompting specific to agentic systems, reminder types, structured tools |
| 26 | [Google Agent Development Kit (ADK)](https://google.github.io/adk-docs/) | Google | Code-first framework, hierarchical multi-agent, model-agnostic, graph-based workflows |
| 27 | [Multi-Agent Patterns in ADK](https://developers.googleblog.com/developers-guide-to-multi-agent-patterns-in-adk/) | Google | Specific patterns: sequential, parallel, hierarchical, dynamic routing |

---

## PART 3: Open-Source Frameworks (5 resources)

| # | Framework | Approach | GitHub | Docs |
|---|-----------|-----------|--------|------|
| 28 | **LangGraph** | Graph-based, stateful | [GitHub](https://github.com/langchain-ai/langgraph) | [Docs](https://docs.langchain.com/oss/python/langgraph/workflows-agents) |
| 29 | **CrewAI** | Role-based, Crews + Flows | [GitHub](https://github.com/crewAIInc/crewAI) | [Docs](https://docs.crewai.com) |
| 30 | **AutoGen** | Conversational multi-agent | [GitHub](https://github.com/microsoft/autogen) | [Docs](https://microsoft.github.io/autogen/stable/) |
| 31 | **LlamaIndex AgentWorkflow** | RAG-focused, document-centric | [GitHub](https://github.com/run-llama/llama_index) | [Docs](https://developers.llamaindex.ai/python/framework/use_cases/agents/) |
| 32 | **OpenAI Agents SDK** | Lightweight, Python-first | [GitHub](https://github.com/openai/openai-agents-python) | [Docs](https://openai.github.io/openai-agents-python/) |

---

## PART 4: Architecture and Design Patterns (7 resources)

| # | Resource | Author | URL |
|---|---------|--------|-----|
| 33 | Azure AI Agent Design Patterns | Microsoft | [Link](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns) |
| 34 | AWS Multi-Agent Orchestration | AWS | [Link](https://aws.amazon.com/solutions/guidance/multi-agent-orchestration-on-aws/) |
| 35 | Agent Design Patterns (blog) | Lance Martin (LangChain) | [Link](https://rlancemartin.github.io/2026/01/09/agent_design/) |
| 36 | Context Engineering for Agents (blog) | Lance Martin | [Link](https://rlancemartin.github.io/2025/06/23/context_engineering/) |
| 37 | Choosing the Right Multi-Agent Architecture | LangChain Blog | [Link](https://blog.langchain.com/choosing-the-right-multi-agent-architecture/) |
| 38 | Model Context Protocol (MCP) Spec | Anthropic / Linux Foundation | [Spec](https://modelcontextprotocol.io/specification/2025-11-25) |
| 39 | Agentic Engineering Patterns | Simon Willison | [Link](https://simonw.substack.com/p/agentic-engineering-patterns) |

---

## PART 5: GitHub Repositories with Patterns and Examples (4 resources)

| # | Repo | Description |
|---|------|-------------|
| 40 | [promptadvisers/agentic-design-patterns-docs](https://github.com/promptadvisers/agentic-design-patterns-docs) | 21 agentic patterns with diagrams and use cases |
| 41 | [nibzard/awesome-agentic-patterns](https://github.com/nibzard/awesome-agentic-patterns) | Curated catalog of agentic patterns |
| 42 | [arunpshankar/Agentic-Workflow-Patterns](https://github.com/arunpshankar/Agentic-Workflow-Patterns) | Python patterns with working code (from a Google engineer) |
| 43 | [muratcankoylan/Agent-Skills-for-Context-Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) | Agent Skills for context engineering and multi-agent |

---

## PART 6: Academic Surveys and Reports (4 resources)

| # | Resource | Description |
|---|---------|-------------|
| 44 | [The Rise and Potential of LLM-Based Agents (Survey, 86 pp.)](https://arxiv.org/abs/2309.07864) | Comprehensive survey + [paper list GitHub](https://github.com/WooooDyy/LLM-Agent-Paper-List) |
| 45 | [Emerging AI Agent Architectures Survey](https://arxiv.org/html/2404.11584v1) | Covers ReAct, Reflexion, and other architectural patterns |
| 46 | [LLM Agent: A Survey on Methodology, Applications and Challenges](https://arxiv.org/abs/2503.21460) | 2025 survey with methodology-centered taxonomy |
| 47 | [State of Agent Engineering 2025](https://www.langchain.com/state-of-agent-engineering) | LangChain report on 1,340 responses - data on how the industry builds agents |

---

## Quick Reference: Where to Start

**If you want to understand the theory:**
1. Lilian Weng - "LLM Powered Autonomous Agents" (#8)
2. Andrew Ng - 4 agentic patterns (#6)
3. Survey "Rise and Potential of LLM-Based Agents" (#44)

**If you want to build right away:**
1. Anthropic - "Building Effective Agents" (#20)
2. OpenAI Agents SDK (#23)
3. LangGraph docs (#28)

**If you want to optimize prompts:**
1. Elvis Saravia - Prompt Engineering Guide (#16)
2. OpenAI Prompting Guides for GPT-4.1/5 (#25)
3. Anthropic - Context Engineering (#21)

**If you want multi-agent:**
1. CrewAI (#29) for role-based approach
2. LangGraph (#28) for graph-based approach
3. AutoGen (#30) for conversational approach
