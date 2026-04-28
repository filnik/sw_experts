# Decision Matrix: AI Agent Design

This matrix helps in making architectural decisions for the design of AI agents.

---

## 1. Workflow vs Autonomous Agent

> Source: `guides/20-anthropic-building-effective-agents.md`, `experts/11-schluntz-zhang.md`

| Criterion | Workflow (deterministic) | Agent (autonomous) |
|----------|--------------------------|-------------------|
| **Control** | High, predictable | Low, flexible |
| **Latency** | Low, fixed | Variable, potentially high |
| **Cost** | Predictable | Variable |
| **Task complexity** | Well-defined | Open, ambiguous |
| **Debugging** | Easy | Hard |
| **Reliability** | High | Medium |

**Rule**: Start with a workflow. Move to an agent only when the task requires dynamic decisions you cannot anticipate.

---

## 2. Choosing the Single-Agent Pattern

> Source: `experts/01-shunyu-yao.md`, `experts/03-noah-shinn.md`, `surveys/45-emerging-agent-architectures.md`

| Pattern | When to Use It | Complexity | File |
|---------|--------------|-------------|------|
| **Prompt Chaining** | Sequential tasks, quality gates | Low | `experts/11-schluntz-zhang.md` |
| **Routing** | Heterogeneous inputs, classification | Low | `experts/11-schluntz-zhang.md` |
| **ReAct** | Tool use + reasoning | Medium | `experts/01-shunyu-yao.md` |
| **Reflexion** | Iterative tasks, self-correction | Medium | `experts/03-noah-shinn.md` |
| **Plan-and-Execute** | Complex tasks, decomposition | High | `experts/06-andrew-ng.md` |
| **Evaluator-Optimizer** | Output that needs refinement | Medium | `experts/11-schluntz-zhang.md` |
| **LATS** | Search in complex spaces | High | `surveys/45-emerging-agent-architectures.md` |

**Decision Tree**:
1. Is the task simple and sequential? → **Prompt Chaining**
2. Are external tools needed? → **ReAct**
3. Is self-correction needed? → **Reflexion**
4. Is decomposition needed? → **Plan-and-Execute**
5. Is iterative quality needed? → **Evaluator-Optimizer**

---

## 3. Choosing the Multi-Agent Architecture

> Source: `patterns/37-langchain-multi-agent-architecture.md`, `guides/27-google-multi-agent-patterns.md`

| Architecture | When to Use It | Communication | Complexity |
|-------------|--------------|---------------|-------------|
| **Supervisor** | Centralized coordination, clear task | Hub-and-spoke | Medium |
| **Hierarchical** | Team organization, very complex task | Tree | High |
| **Network/Swarm** | Peer routing, dynamic handoffs | Peer-to-peer | Medium |
| **Sequential Pipeline** | Ordered phases, ETL-like | Linear | Low |
| **Parallel** | Independent sub-tasks | Fan-out/fan-in | Medium |

**Decision Tree**:
1. Are sub-tasks independent? → **Parallel**
2. Is there a natural order? → **Sequential**
3. Is a coordinator needed? → **Supervisor**
4. Are there sub-teams? → **Hierarchical**
5. Do agents need to hand off work? → **Network/Swarm**

---

## 4. Choosing the Framework

> Source: `frameworks/28-32`, `surveys/47-state-of-agent-engineering.md`

| Framework | Paradigm | Learning Curve | Flexibility | Production | File |
|-----------|-----------|-------------|--------------|------------|------|
| **LangGraph** | Graph-based, stateful | High | Maximum | Excellent | `frameworks/28-langgraph.md` |
| **CrewAI** | Role-based, Crews+Flows | Low | Medium | Good | `frameworks/29-crewai.md` |
| **AutoGen** | Conversational | Medium | High | Medium | `frameworks/30-autogen.md` |
| **LlamaIndex** | RAG-first, event-driven | Medium | Medium | Good | `frameworks/31-llamaindex-agents.md` |
| **OpenAI SDK** | Lightweight, Python-first | Low | Low | Good | `frameworks/32-openai-agents-sdk.md` |
| **Google ADK** | Hierarchical multi-agent | Medium | High | Good | `guides/26-google-adk.md` |
| **Custom** | Any | N/A | Maximum | Variable | `guides/20-anthropic-building-effective-agents.md` |

**Decision Tree**:
1. Do you need maximum control and flexibility? → **LangGraph**
2. Want to start fast with roles and teams? → **CrewAI**
3. Is the focus RAG and documents? → **LlamaIndex**
4. Want minimalism and using OpenAI? → **OpenAI SDK**
5. Need model-agnostic enterprise? → **Google ADK**
6. Want a conversational approach? → **AutoGen**
7. No framework fits? → **Custom** (follow Anthropic guide)

---

## 5. Memory Strategy

> Source: `experts/05-joon-sung-park.md`, `experts/08-lilian-weng.md`, `frameworks/29-crewai.md`

| Type | Implementation | When to Use It |
|------|----------------|---------------|
| **In-context (short-term)** | Messages in the context window | Short conversations |
| **Summarization** | Periodic compaction | Long conversations |
| **Vector store (long-term)** | Embedding + similarity search | Persistent knowledge |
| **Episodic** | Structured logs of experiences | Learning from errors |
| **Entity memory** | Graph of entities and relationships | Tracking people/concepts |
| **Tripartite retrieval** | Recency + Importance + Relevance | Complex simulations |

**Decision Tree**:
1. Conversation < 10 turns? → **In-context**
2. Long conversation? → **Summarization** + **Vector store**
3. Does the agent need to learn? → **Episodic** (Reflexion pattern)
4. Need to track entities? → **Entity memory**
5. Social simulation? → **Tripartite retrieval** (Generative Agents)

---

## 6. Context Engineering vs Prompt Engineering

> Source: `guides/21-anthropic-context-engineering.md`, `patterns/35-36-lance-martin.md`, `experts/09-andrej-karpathy.md`

| Aspect | Prompt Engineering | Context Engineering |
|---------|-------------------|---------------------|
| **Scope** | Single prompt | Entire lifecycle |
| **Focus** | Static instructions | Dynamic context |
| **Complexity** | Template + few-shot | Write/Select/Compress/Isolate |
| **Who does it** | Anyone | Agent architect |
| **When** | Simple tasks | Agents in production |

**The 4 context engineering strategies** (Lance Martin):
1. **Write** - Scratchpad, notes, files
2. **Select** - Just-in-time context retrieval
3. **Compress** - Summarization, compaction
4. **Isolate** - Sub-agent with dedicated context

---

## 7. Agent Security

> Source: `experts/15-simon-willison.md`, `patterns/39-willison-agentic-patterns.md`

| Risk | Mitigation | Pattern |
|---------|-------------|---------|
| **Prompt injection** | Input sanitization, Dual LLM | `experts/15-simon-willison.md` |
| **Data exfiltration** | Trust boundaries, permission scoping | `patterns/39-willison-agentic-patterns.md` |
| **Hallucination** | Grounding, citations, eval | `experts/17-chip-huyen.md` |
| **Cascading failures** | Circuit breakers, timeouts | `patterns/33-azure-agent-patterns.md` |
| **Cost explosion** | Token budgets, max iterations | `experts/17-chip-huyen.md` |

**The Lethal Trifecta** (Willison): Never combine private data + untrusted content + external communication in a single agent.

---

## 8. Agents in Production: Checklist

> Source: `experts/17-chip-huyen.md`, `surveys/47-state-of-agent-engineering.md`

| Area | Question | Reference File |
|------|---------|---------------------|
| **Architecture** | Workflow or agent? | `guides/20-anthropic-building-effective-agents.md` |
| **Pattern** | Which pattern for the task? | This matrix, section 2 |
| **Framework** | Custom or framework? | This matrix, section 4 |
| **Memory** | What kind of memory is needed? | This matrix, section 5 |
| **Tools** | How to standardize tools? | `patterns/38-mcp-specification.md` |
| **Security** | Trust boundaries defined? | `experts/15-simon-willison.md` |
| **Eval** | How do I evaluate the agent? | `surveys/46-llm-agent-methodology-survey.md` |
| **Observability** | How do I monitor? | `experts/17-chip-huyen.md` |
| **HITL** | Where is the human needed in the loop? | `patterns/39-willison-agentic-patterns.md` |
| **Costs** | Budget for tokens/iterations? | `surveys/47-state-of-agent-engineering.md` |

---

## 9. Mapping Across Taxonomies

| Anthropic (Schluntz & Zhang) | Google ADK | Andrew Ng | LangChain |
|-----------------------------|------------|-----------|-----------|
| Prompt Chaining | SequentialAgent | - | Sequential |
| Routing | Dynamic Routing / AutoFlow | - | Router |
| Parallelization | ParallelAgent | - | Parallel |
| Orchestrator-Workers | Hierarchical / AgentTool | Multi-Agent | Supervisor |
| Evaluator-Optimizer | LoopAgent | Reflection | Feedback Loop |
| - | - | Tool Use | Tool Node |
| - | - | Planning | Plan-and-Execute |
