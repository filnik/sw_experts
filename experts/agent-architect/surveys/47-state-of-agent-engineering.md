# State of Agent Engineering 2025 -- LangChain Report

> In-depth analysis of the "State of Agent Engineering 2025" report published by LangChain,
> based on 1,340 responses collected between November 18 and December 2, 2025.

---

## 1. Overview

The **"State of Agent Engineering 2025"** report represents the second annual edition of LangChain's survey on the AI agents ecosystem. The survey was open to the public for two weeks, from November 18 to December 2, 2025, collecting **1,340 responses** from professionals around the world involved in the design, development and deployment of agentic systems.

### Methodology

Data collection took place through a public questionnaire distributed through LangChain's official channels (blog, social media, Discord community and newsletter). Respondents include a heterogeneous mix of roles: software engineers, product managers, technical leaders, data scientists and C-suite executives. The questionnaire covered topics such as: agentic architectures adopted, LLM models used, development frameworks, evaluation strategies, barriers to deployment in production and implemented use cases.

Compared to the previous edition (2024, which had received about 1,300 responses with 51% of agents in production), the 2025 report shows concrete growth in adoption: **57.3%** of respondents declare they have agents operational in production, with an additional **30.4%** in active development with concrete deployment plans. This means that nearly 90% of the professionals involved are actively working on agentic systems, signaling that the field has exited the exploratory phase to enter the operational one.

A relevant data point concerns the company composition: large enterprises (over 10,000 employees) lead the adoption with **67%** of agents in production, against **50%** of organizations with fewer than 100 employees. This gap suggests that enterprises have the resources, infrastructure and governance necessary to take agentic systems beyond the prototype.

---

## 2. Who Is Building Agents

### Builder Profile

The sample in the report reflects an ecosystem dominated by the technology sector but with growing industrial diversification:

| Sector                      | Percentage |
|-----------------------------|------------|
| Technology                  | 63%        |
| Financial Services          | 10%        |
| Healthcare                  | 6%         |
| Education                   | 4%         |
| Consumer Goods              | 3%         |
| Manufacturing               | 3%         |
| Other sectors               | 11%        |

The concentration in the tech sector is not surprising: it is the first to experiment with new technologies. But the significant presence of financial services (10%) and healthcare (6%) indicates that regulated sectors are concretely investing in agents, despite the additional regulatory constraints they must face.

### Company Size

The distribution by company size reveals a fragmented landscape:

| Size (employees)         | Percentage |
|--------------------------|------------|
| < 100                    | 49%        |
| 100 - 500                | 18%        |
| 500 - 2,000              | 15%        |
| 2,000 - 10,000           | 9%         |
| 10,000+                  | 9%         |

Almost half of the respondents come from small organizations (under 100 employees), which includes startups, AI consulting agencies and independent teams. This data is consistent with the typical distribution of open-source communities and suggests that agentic innovation is mainly emerging from the bottom up, in startups and agile teams, before propagating towards enterprises.

### Roles Involved

Agent engineering is not solely a developer activity. The report highlights the participation of product managers, business leaders and executives, confirming that building agents requires interdisciplinary competencies: product thinking for the definition of prompts and flows, engineering for infrastructure and tools, and data science for evaluation and optimization.

---

## 3. Most Used Architectures

### The Predominance of Graph Patterns

The dominant architecture in 2025 is the one based on **stateful graphs**, where each node represents a step of the agent (a model call, the use of a tool, a decision) and the edges define the conditional transitions between steps. This model, primarily promoted by **LangGraph**, has supplanted the linear chains that characterized the first wave of LangChain applications.

The main architectural patterns include:

- **Single-agent with tool use**: a single LLM that reasons and invokes tools in a ReAct-style loop. It remains the most widespread pattern for well-defined tasks.
- **Multi-agent with orchestration**: multiple specialized agents coordinated by a central orchestrator. Used for complex workflows where different domains require separate competencies.
- **Human-in-the-loop**: architectures that provide for interruption points where a human operator approves, modifies or redirects the agent's action. Particularly widespread in enterprise and regulated contexts.
- **RAG-augmented agents**: agents that combine retrieval from knowledge bases with reasoning and action capabilities. The majority of organizations (57%) do not do fine-tuning but rely on base models combined with prompt engineering and RAG.

### Primary Use Cases by Architecture

Use cases are distributed unevenly among architectures:

| Use Case                             | Percentage |
|--------------------------------------|------------|
| Customer Service                     | 26.5%      |
| Research & Data Analysis             | 24.4%      |
| Internal Workflow Automation         | 18%        |
| Coding Assistants (daily use)        | High       |

An interesting insight: in large enterprises (10,000+ employees), **internal automation** and productivity are the number one priority (26.8%), surpassing customer service. Enterprises prefer to validate agentic systems on internal processes before exposing them to customers, a de-risking strategy that reduces the impact of any failures.

### From Chain to Graph: The Architectural Evolution

The transition from linear chains to stateful graphs represents the most significant architectural change of 2025. Chains, by their sequential nature, do not adequately support reasoning cycles, conditional branching and persistent state management. Graphs solve these limits by offering:

- **Persistent state** across executions
- **Conditional branching** based on model output
- **Cycles** for iterative reasoning (retry, reflection)
- **Checkpoints** for error recovery and human-in-the-loop

---

## 4. Frameworks Used

### LangGraph: The De Facto Standard for Production

LangGraph has emerged as the reference framework for building agents in production. The LangChain team itself has explicitly communicated: **"Use LangGraph for agents, not LangChain"**, recognizing that the chain architecture of the original LangChain is not adequate for complex agentic systems.

LangGraph reached **General Availability in May 2025** and released the **stable 1.0 version in October 2025** -- the first stable major release in this segment. According to available data, LangGraph is used in production by over **400 companies**, including realities such as LinkedIn and Uber.

The key features of LangGraph include:
- Graph architecture with nodes, edges and shared state
- Built-in memory to maintain context across sessions
- Native support for human-in-the-loop
- Integration with the **Model Context Protocol (MCP)** for standardized connection to tools
- Durable execution for workflow resilience

### CrewAI: The Role-Based Alternative

CrewAI positions itself as an alternative with a different paradigm: agents organized into "crews" with specific roles that collaborate on shared tasks. The framework has raised **18 million dollars in Series A** and declares revenues of **3.2 million dollars** (data from July 2025), with 60% of Fortune 500 companies as users.

The advantage of CrewAI compared to LangGraph is the speed of prototyping: teams report being able to put agents into production in **2 weeks** with CrewAI versus **2 months** with LangGraph, at the cost of less architectural flexibility.

### AutoGen (AG2) and the Microsoft World

AutoGen, now renamed AG2, represents Microsoft's approach to multi-turn agents. The framework is oriented towards conversation between agents and integrates natively with the Azure ecosystem. It is particularly adopted in Microsoft-first enterprise contexts.

### Custom Solutions

A significant portion of teams choose to build custom frameworks, often combining components from different ecosystems. This trend is particularly strong among organizations with specific requirements for performance, security or integration with legacy systems.

---

## 5. Preferred LLM Models

### OpenAI Dominates, But Diversification Is the Norm

The most significant data point regarding models is: **over two thirds of organizations use OpenAI's GPT models**, but simultaneously **over 75% use multiple models** in production or development. The multi-model strategy has become the norm, not the exception.

The reasons for this diversification are multiple:

- **Routing by complexity**: simple tasks are handled by cheaper and faster models, complex tasks by frontier models.
- **Cost optimization**: not all tasks require GPT-4o; lighter models significantly reduce costs at high volumes.
- **Reduction of vendor lock-in**: depending on a single provider exposes one to risks of pricing, availability and changes in APIs.
- **Regulatory requirements**: some sectors require that data not leave certain jurisdictions, making necessary on-premise models or specific providers.

### The Map of Models

Models with significant adoption include:

- **OpenAI GPT-4o / GPT-4 Turbo**: market leader for versatility and quality. They dominate in complex reasoning tasks and customer service.
- **Anthropic Claude (3.5 Sonnet, Opus, Haiku)**: strong adoption as an alternative to GPT, particularly appreciated for context length, coding capabilities and adherence to instructions.
- **Google Gemini (1.5 Pro, Flash)**: significant growth thanks to integration in the Google Cloud ecosystem and the extended context window.
- **Open-Source Models**: **33%** of organizations invest in infrastructure for in-house deployment of open-source models. The main motivations are cost optimization at high volume, data residency requirements and regulatory constraints.

### Fine-Tuning: The Majority Doesn't Do It

A surprising data point: **57%** of organizations **do not perform fine-tuning** of models, preferring an approach based on prompt engineering and RAG. This reflects both the increasing quality of base models and the cost (computational and organizational) of fine-tuning. Organizations that do fine-tuning are typically those with very high volumes or extremely specific domain requirements.

---

## 6. Main Challenges

### Quality As Production Killer

The number one challenge for bringing agents to production is **quality**, cited by **32%** of respondents as the main barrier. This data represents a change compared to 2024, where costs were a greater concern. The reduction in model prices during 2025 has shifted attention from economic sustainability to functional reliability.

The sub-dimensions of quality include:

- **Hallucinations**: agents generate plausible but factually incorrect responses, a problem amplified when the agent acts on real systems.
- **Output consistency**: the same query can produce significantly different results between successive executions, making it difficult to ensure a predictable service level.
- **Context engineering**: managing long contexts, maintaining coherence across multiple turns and deciding which information to include in the prompt requires sophisticated engineering that many teams underestimate.

### Latency

**20%** of respondents cite latency as a critical barrier. Agents, by their nature, require multiple calls to models and tools, accumulating latency at each step. In interactive contexts (chatbots, coding assistants), response times exceeding a few seconds drastically degrade the user experience.

### Security

Security emerges as a growing concern, particularly in enterprises: **24.9%** of organizations with over 2,000 employees cite it as the main barrier. The dimensions of security include:

- **Prompt injection**: attacks that manipulate the agent's behavior through malicious inputs
- **Data leakage**: risk that the agent exposes sensitive data present in the context
- **Tool abuse**: a compromised agent that invokes tools with side effects (writing to databases, sending emails, transactions)
- **Compliance**: respect for specific regulations (GDPR, HIPAA, SOC2) when the agent processes regulated data

### The Continuously Evolving Stack

A less quantifiable but repeatedly cited challenge in open responses is the **instability of the technology stack**. New frameworks, APIs and orchestration layers emerge faster than organizations can standardize and validate. Some regulated enterprises report having to **rebuild their agent stack every three months**, an unsustainable rate of change for organizations with structured approval processes.

---

## 7. Tool Use and Integrations

### Tool Calling As Fundamental Capability

Accurate tool invocation is cited by engineering leaders as one of the main challenges, highlighting how a large part of enterprise AI is still focused on the reliability of tool calling rather than deep reasoning.

Tool integration patterns include:

- **Native function calling**: use of providers' function calling APIs (OpenAI, Anthropic, Google) for structured tool invocation with JSON schema.
- **Model Context Protocol (MCP)**: the emerging protocol for standardizing the connection between agents and tools, promoted by Anthropic and adopted by LangGraph. MCP is becoming the de facto standard for tool interoperability.
- **Custom tools**: many organizations build custom wrappers around their internal services, exposing them as tools invokable by the agent.

### Most Common Tool Categories

From the analysis of the report and the broader context, the following categories of most integrated tools emerge:

1. **Web search and retrieval**: access to external information in real time
2. **Database and SQL**: text-to-SQL for querying structured databases
3. **Code execution**: sandbox for executing code generated by the agent
4. **External APIs**: integration with third-party services (CRM, ticketing, ERP)
5. **File system and document processing**: reading, writing and transforming documents
6. **Vector store and knowledge base**: semantic retrieval from proprietary knowledge bases

### The Reliability Challenge

Tool calling remains a critical point because it introduces side effects in the real world. A reasoning error in a chatbot produces a wrong response; a tool calling error can delete a record from the database, send a wrong email or execute an unauthorized transaction. This is why the human-in-the-loop pattern is particularly widespread in contexts where tools have irreversible effects.

---

## 8. Evaluation and Testing

### Observability Surpasses Evaluation

Perhaps the most revealing data of the report: **89%** of organizations have implemented some form of observability for their agents, but only **52.4%** conduct structured offline evaluations. This gap (89% vs 52%) indicates that teams recognize the importance of seeing what their agents do, but have not yet systematized the process of measuring quality.

### Observability in Detail

- **89%** has implemented observability in some form
- **62%** has detailed tracing that allows inspection of single steps and tool calls
- Among those with agents in production: **94%** has observability, **71.5%** has complete tracing

These numbers confirm that observability is considered a non-negotiable requirement for production. Without visibility into how the agent reasons and acts, teams cannot debug failures, optimize performance or build trust with stakeholders.

### Offline and Online Evaluation

| Type of Evaluation   | General Adoption  | Among those with agents in production |
|----------------------|-------------------|----------------------------------------|
| Offline (test set)   | 52.4%             | Higher                                 |
| Online (production)  | 37.3%             | 44.8%                                  |
| Both                 | ~24%              | -                                      |

Offline evaluation on test sets is the most widespread method, but less than half of teams practice it systematically. Online evaluation (monitoring of performance on real traffic) has even lower adoption but is growing, particularly among those who already have agents in production (44.8% vs 37.3% general).

### Evaluation Methods

| Method               | Adoption |
|----------------------|----------|
| Human review         | 59.8%    |
| LLM-as-judge         | 53.3%    |
| Traditional ML metrics (ROUGE, BLEU) | Limited |

**Human review** remains the most widespread method (59.8%), reflecting the difficulty of fully automating the evaluation of non-deterministic systems. The **LLM-as-judge** approach (using an LLM to evaluate the output of another LLM) is adopted by 53.3%, a significant data point indicating growing trust in this technique despite its known limitations (bias, inconsistency). Traditional ML metrics such as ROUGE and BLEU have limited adoption, confirming their inadequacy for evaluating conversational and agentic outputs.

---

## 9. Trends

### Agent Engineering As a New Discipline

Perhaps LangChain's most important conceptual contribution in 2025 is the definition of **agent engineering** as a distinct discipline. In the blog post "Agent Engineering: A New Discipline", the team defines agent engineering as:

> "The iterative process of refining non-deterministic LLM systems into reliable production experiences."

This framing implies a continuous cycle: **build -> test -> ship -> observe -> refine -> repeat**. Unlike traditional software development, where one tests exhaustively before release, in agent engineering **the release is the primary mechanism of learning**: "Shipping is how you learn, not what you do after learning."

The discipline requires three interdisciplinary competencies:

1. **Product Thinking**: defining detailed prompts (often hundreds or thousands of lines), understanding the core job of the agent and defining evaluations that validate the expected behavior.
2. **Engineering**: building tools, developing interfaces with streaming and interruption handling, creating resilient runtimes with durable execution.
3. **Data Science**: establishing measurement systems, conducting A/B testing, analyzing usage patterns and error sources specific to agentic systems.

### Agents in the Daily Life of Builders

The report reveals that builders of agents are also intensive users of agents. The three most cited categories in daily use are:

1. **Coding Assistants**: Claude Code, Cursor, GitHub Copilot, Amazon Q and Windsurf have become daily tools for code generation, debugging, test creation and navigation of complex codebases.
2. **Research Agents**: ChatGPT, Claude, Gemini and Perplexity for exploration, synthesis and cross-source analysis.
3. **Custom Agents**: agents built on LangChain and LangGraph for QA testing, knowledge search, text-to-SQL, demand planning and support.

### From PoC to Production

The macro trend of 2025 is the definitive transition from proof-of-concept to production. Organizations no longer ask **whether** to build agents, but **how** to deploy them reliably, efficiently and scalably. This shift manifests itself in:

- Growing investments in observability and tracing
- Adoption of more robust architectures (stateful graphs vs linear chains)
- Focus on security and governance, especially in enterprises
- Multi-model strategies to optimize cost, latency and quality

### Framework Market Consolidation

The framework landscape is consolidating around a few dominant players: LangGraph for architectural flexibility, CrewAI for prototyping speed, and the native solutions of cloud providers (Azure AI Agent Service, AWS Bedrock Agents) for enterprise integration. Smaller or too specific frameworks are losing traction in favor of these consolidated platforms.

### MCP As Emerging Standard

The **Model Context Protocol (MCP)**, introduced by Anthropic, is emerging as a standard for connecting agents and external tools. LangGraph has natively integrated MCP support, and its adoption is rapidly growing. MCP promises to solve the problem of integration fragmentation, offering a common protocol for exposing tools, resources and context to agents.

---

## 10. Lessons for Agent Builders

### Operational Insights from the Report

Based on the data from the report and the analysis of the blog post "Agent Engineering: A New Discipline", the following concrete lessons emerge for those building agents:

**1. Invest first in observability, then in evals**

94% of those with agents in production have observability. It is not optional. Implementing detailed tracing (every step, every tool call, every decision) is the prerequisite for everything else: debugging, optimization, building trust. Only after establishing complete visibility does it make sense to build structured evaluation pipelines.

**2. Accept non-determinism**

Agents are not deterministic software. "Every input is an edge case" because natural language accepts infinite variations. Traditional debugging fails when the logic resides inside the models. Accepting this reality and designing for it (with fallbacks, guardrails and monitoring) is more productive than trying to eliminate it.

**3. Adopt a multi-model strategy**

With 75% of organizations already using multiple models, lock-in on a single provider is an unnecessary risk. Design model-agnostic architectures that allow intelligent routing among different models based on cost, latency and task complexity.

**4. Start from the inside, scale outward**

Successful enterprises start with agents for internal automation before exposing them to customers. This approach allows iteration in a controlled environment, building organizational competencies and establishing governance processes before errors have external impact.

**5. Prompt engineering is product work**

Writing prompts for agents is not a secondary technical task. Prompts for production agents are often hundreds or thousands of lines long and require the same care as product definition. Companies like Clay, Vanta and LinkedIn treat prompt engineering as product work, not engineering work.

**6. Don't underestimate the cost of tool calling**

Reliable tool invocation is harder than it seems. Every tool introduces potential side effects, and a single invocation error can have real consequences (corrupted data, sent emails, executed transactions). Implementing human-in-the-loop for high-risk actions is not a weakness, it is an architectural necessity.

**7. Release early, observe, iterate**

The agent engineering paradigm requires a mindset shift: release is not the end of the development process, but the beginning of the learning process. Successful organizations release improvements in days, not quarters, based on data collected in production.

**8. Plan for the unstable stack**

The technology stack of agents evolves faster than any other area of software engineering. Designing modular architectures with clear interfaces between components reduces the cost of replacement when (not if) a component must be updated.

**9. Evaluation is a continuous investment**

Combining offline evaluation (on test sets) with online evaluation (on real traffic) is the best practice, but only ~24% of organizations do it. Investing in both dimensions is a measurable competitive advantage.

**10. Security is not an afterthought**

With 24.9% of enterprises citing security as the main barrier, it is clear that agent security requires dedicated attention from the earliest design phases: tool sandboxing, input validation, complete audit trails and specific tests for prompt injection.

---

## 11. Resources and References

### Reports and Primary Sources

- [State of Agent Engineering 2025 -- Complete Report](https://www.langchain.com/state-of-agent-engineering) -- The original LangChain report with all the data and visualizations.
- [State of AI Agents 2024 -- Previous Report](https://www.langchain.com/stateofaiagents) -- The 2024 edition for year-over-year comparison.
- [Agent Engineering: A New Discipline -- LangChain Blog](https://blog.langchain.com/agent-engineering-a-new-discipline/) -- The foundational blog post that defines agent engineering as a discipline.
- [LangChain Series B -- $125M Fundraise](https://blog.langchain.com/series-b/) -- The context of LangChain's investment in the agent engineering platform.

### Analyses and Commentary

- [LangChain LinkedIn -- Report Announcement](https://www.linkedin.com/posts/langchain_state-of-agent-engineering-2025-activity-7406757229313417216-WdjT) -- Official LangChain post with key findings.
- [The AI Agent Framework Landscape in 2025 -- Medium](https://medium.com/@hieutrantrung.it/the-ai-agent-framework-landscape-in-2025-what-changed-and-what-matters-3cd9b07ef2c3) -- Analysis of the framework landscape in the context of the report.
- [Top AI Agent Frameworks in 2025 -- Medium](https://medium.com/@iamanraghuvanshi/agentic-ai-3-top-ai-agent-frameworks-in-2025-langchain-autogen-crewai-beyond-2fc3388e7dec) -- Comparison between frameworks with reference to report data.
- [200+ AI Agent Statistics for 2025 -- PragmaticCoders](https://www.pragmaticcoders.com/resources/ai-agent-statistics) -- Collection of statistics on the AI agent market.
- [AI Agents in Production 2025 -- Cleanlab](https://cleanlab.ai/ai-agents-in-production-2025/) -- Best practices for production deployment.

### Frameworks and Tools Cited

- [LangGraph -- Agent Orchestration Framework](https://www.langchain.com/langgraph) -- The framework recommended by LangChain for building agents in production.
- [LangChain Platform](https://www.langchain.com) -- The complete platform for observability, evaluation and deployment.
- [CrewAI](https://www.crewai.com) -- The role-based framework for rapid prototyping of multi-agent agents.
- [AutoGen (AG2) -- Microsoft](https://microsoft.github.io/autogen/) -- The Microsoft framework for conversational agents.

---

> **Note**: This document was compiled by aggregating data from the official LangChain report,
> the blog post "Agent Engineering: A New Discipline", and third-party analyses.
> Quantitative data is extracted directly from the original report based on 1,340 responses.
> Last updated: March 2026.
