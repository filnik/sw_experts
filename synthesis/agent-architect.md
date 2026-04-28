# Agent Architect: Knowledge Base Synthesis

## Overview

This knowledge base covers **47 authoritative resources** on AI Agent Design & Orchestration, organized into 6 categories. It provides a comprehensive guide to designing, building, and orchestrating LLM-based agents.

---

## Competency Map

### 1. Theoretical Foundations

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 1 | ReAct (Shunyu Yao) | `experts/01-shunyu-yao.md` | Reasoning+acting paradigm, Thought-Action-Observation loop |
| 2 | CoALA (Karthik Narasimhan) | `experts/02-karthik-narasimhan.md` | Cognitive Architectures for Language Agents, SWE-bench |
| 3 | Reflexion (Noah Shinn) | `experts/03-noah-shinn.md` | Verbal reinforcement learning, self-correction |
| 4 | Chain-of-Thought (Jason Wei) | `experts/04-jason-wei.md` | CoT prompting, emergent reasoning, scaling laws |
| 5 | Generative Agents (Joon Sung Park) | `experts/05-joon-sung-park.md` | Social simulation, memory stream, tripartite retrieval |
| 8 | Agent Formula (Lilian Weng) | `experts/08-lilian-weng.md` | Agent = LLM + Memory + Planning + Tool Use |

### 2. Agentic Patterns and Design

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 6 | 4 Agentic Patterns (Andrew Ng) | `experts/06-andrew-ng.md` | Reflection, Tool Use, Planning, Multi-Agent |
| 11 | 5 Composable Patterns (Schluntz & Zhang) | `experts/11-schluntz-zhang.md` | Prompt Chaining, Routing, Parallelization, Orchestrator-Workers, Evaluator-Optimizer |
| 20 | Building Effective Agents | `guides/20-anthropic-building-effective-agents.md` | Workflows vs Agents, decision framework |
| 35-36 | Agent Design Patterns (Lance Martin) | `patterns/35-36-lance-martin.md` | 7 context patterns, Write/Select/Compress/Isolate |
| 37 | Multi-Agent Architecture | `patterns/37-langchain-multi-agent-architecture.md` | Network, Supervisor, Hierarchical, Custom |
| 39 | Agentic Engineering Patterns (Willison) | `patterns/39-willison-agentic-patterns.md` | Trust boundaries, Lethal Trifecta, HITL |
| 40-43 | GitHub Pattern Repos | `repos/40-43-github-pattern-repos.md` | 139+ cataloged patterns, unified taxonomy |

### 3. Context Engineering and Prompting

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 9 | Software 3.0 (Karpathy) | `experts/09-andrej-karpathy.md` | LLM as CPU, context engineering |
| 10 | System Prompt Design (Amanda Askell) | `experts/10-amanda-askell.md` | Agent character, Constitutional AI |
| 16 | Prompt Engineering Guide (Elvis Saravia) | `experts/16-elvis-saravia.md` | 18 techniques, 6-block template, anti-patterns |
| 21 | Context Engineering (Anthropic) | `guides/21-anthropic-context-engineering.md` | Just-in-time loading, budget allocation, compaction |
| 25 | Prompting Guides (OpenAI) | `guides/25-openai-prompting-guides.md` | Reminder types, agentic prompting, instruction hierarchy |

### 4. Frameworks and Implementation

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 7 | LangChain/LangGraph (Harrison Chase) | `experts/07-harrison-chase.md` | Graph-based agents, deep agents |
| 12 | AutoGen (Chi Wang) | `experts/12-chi-wang.md` | Conversational multi-agent |
| 13 | CrewAI (Joao Moura) | `experts/13-joao-moura.md` | Role-based orchestration, Crews + Flows |
| 14 | Agentic RAG (Jerry Liu) | `experts/14-jerry-liu.md` | Document agents, meta-agent |
| 28 | LangGraph | `frameworks/28-langgraph.md` | Nodes, edges, state, checkpointing |
| 29 | CrewAI | `frameworks/29-crewai.md` | Agent, Task, Crew, Flows, Memory |
| 30 | AutoGen | `frameworks/30-autogen.md` | ConversableAgent, GroupChat |
| 31 | LlamaIndex | `frameworks/31-llamaindex-agents.md` | AgentWorkflow, event-driven |
| 32 | OpenAI Agents SDK | `frameworks/32-openai-agents-sdk.md` | Handoffs, guardrails, tracing |

### 5. Official Guides and Cookbooks

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 22 | Claude Cookbooks | `guides/22-claude-cookbooks.md` | Tool use, agent skills, compaction |
| 23 | OpenAI Agents SDK Guide | `guides/23-openai-agents-sdk.md` | Handoffs, guardrails, MCP |
| 24 | OpenAI Cookbook Agents | `guides/24-openai-cookbook-agents.md` | Deep Research, routines, handoffs |
| 26 | Google ADK | `guides/26-google-adk.md` | Hierarchical multi-agent, model-agnostic |
| 27 | Google Multi-Agent Patterns | `guides/27-google-multi-agent-patterns.md` | Sequential, Parallel, Hierarchical, Dynamic |

### 6. Cloud and Enterprise

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 33 | Azure Agent Patterns | `patterns/33-azure-agent-patterns.md` | Semantic Kernel, Agent Factory |
| 34 | AWS Multi-Agent | `patterns/34-aws-multi-agent.md` | Bedrock Agents, Step Functions |
| 38 | MCP Specification | `patterns/38-mcp-specification.md` | Model Context Protocol, tool standardization |

### 7. Agents in Production

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 15 | Security (Simon Willison) | `experts/15-simon-willison.md` | Prompt injection, Dual LLM, trust boundaries |
| 17 | AI Engineering (Chip Huyen) | `experts/17-chip-huyen.md` | Production patterns, failure modes, MLOps |
| 47 | State of Agent Engineering | `surveys/47-state-of-agent-engineering.md` | Real data: 57.3% in production, challenges, trends |

### 8. Vision and Community

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 18 | BabyAGI (Yohei Nakajima) | `experts/18-yohei-nakajima.md` | Task planning loop, first viral agent |
| 19 | Latent Space (Swyx) | `experts/19-swyx.md` | AI Engineer, IMPACT framework |

### 9. Academic Surveys

| # | Topic | File | Key Concept |
|---|-------|------|-----------------|
| 44 | Rise of LLM-Based Agents | `surveys/44-rise-potential-llm-agents.md` | Brain/Perception/Action, 400+ papers |
| 45 | Emerging Agent Architectures | `surveys/45-emerging-agent-architectures.md` | 9 architectural generations |
| 46 | LLM Agent Methodology | `surveys/46-llm-agent-methodology-survey.md` | Three-dimensional framework, 329 papers |

---

## Learning Paths

### Beginner: "What is an agent?"
1. `experts/08-lilian-weng.md` - The fundamental formula
2. `experts/06-andrew-ng.md` - The 4 patterns
3. `guides/20-anthropic-building-effective-agents.md` - Workflow vs Agent

### Intermediate: "How do I build an agent?"
1. `experts/11-schluntz-zhang.md` - The 5 building blocks
2. `guides/21-anthropic-context-engineering.md` - Context engineering
3. A framework of choice: `frameworks/28-langgraph.md` or `frameworks/29-crewai.md`

### Advanced: "How do I orchestrate multi-agent systems?"
1. `patterns/37-langchain-multi-agent-architecture.md` - Architectures
2. `patterns/38-mcp-specification.md` - MCP for standardized tools
3. `experts/17-chip-huyen.md` - Production and failure modes
4. `surveys/47-state-of-agent-engineering.md` - Real data from the field

### Specialist: "How do I optimize prompts and context?"
1. `experts/16-elvis-saravia.md` - Prompting techniques
2. `experts/10-amanda-askell.md` - System prompt design
3. `guides/25-openai-prompting-guides.md` - Agentic prompting
4. `patterns/35-36-lance-martin.md` - Advanced context engineering
