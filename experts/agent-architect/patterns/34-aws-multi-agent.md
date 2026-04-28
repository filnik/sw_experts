# AWS Multi-Agent Orchestration

## Guide to Multi-Agent Orchestration on Amazon Web Services

---

## 1. Overview

### The AWS Guide for Multi-Agent Orchestration

The **Guidance for Multi-Agent Orchestration on AWS** represents an official reference architecture published by Amazon Web Services to address complex scenarios that require the coordinated collaboration of multiple specialized AI agents. This guide stems from the observation that real business problems can rarely be solved by a single monolithic agent: they require diverse competencies, access to heterogeneous systems, and the ability to break down articulated tasks into manageable sub-activities.

The architecture proposed by AWS is based on a central principle: a **Supervisor Agent** that acts as the entry point for user requests and intelligently orchestrates a set of specialized agents. Each agent possesses specific domain expertise, for example product recommendations, order tracking, technical support, or customer experience personalization. The supervisor analyzes the request, determines which agent or combination of agents is best suited to handle it, delegates the tasks, and aggregates the responses into a coherent response for the end user.

### The Amazon Bedrock Context

**Amazon Bedrock** is the AWS fully managed service that provides access to foundation models (FM) from various providers, including Anthropic (Claude), Meta (Llama), Amazon (Titan), Mistral, and others, through a single API. Bedrock is not limited to model inference: it integrates a complete ecosystem for building enterprise-level generative AI applications, which includes agents, knowledge bases, security guardrails, fine-tuning, and model evaluation.

**Amazon Bedrock Agents** is the specific component that allows you to create autonomous agents capable of planning, reasoning, and acting. Each agent can access external tools through action groups, query knowledge bases through knowledge bases, and follow instructions in natural language defined by the developer. The **multi-agent collaboration** feature, released in general availability in March 2025, extends this paradigm by allowing multiple Bedrock agents to collaborate natively within the platform.

More recently, AWS introduced **Amazon Bedrock AgentCore**, a service dedicated to deployment, management, and operation of AI agents at any scale, with native support for the **Agent-to-Agent (A2A)** protocol that enables interoperability between agents built with different frameworks.

---

## 2. Multi-Agent Architecture on AWS

### Fundamental Components

The multi-agent architecture on AWS revolves around four fundamental components that cooperate to manage the entire lifecycle of a user request:

**Supervisor Agent**: is the main agent that receives requests from the user and orchestrates the entire conversation. The supervisor maintains the session context, analyzes the user's intent, decides which sub-agents to involve, delegates tasks, and synthesizes partial responses into a coherent final response. The supervisor is provided with specific instructions describing the capabilities of each collaborator, allowing it to perform intelligent routing.

**Collaborator Agents**: are agents specialized in specific domains. Each collaborator has its own foundation model, its own instructions, its own action groups, and its own knowledge bases. Collaborators operate independently of each other, receiving tasks from the supervisor and returning structured results.

**Action Groups**: represent the interface between agents and the outside world. Each action group defines a set of actions that the agent can perform, typically implemented as AWS Lambda functions. The actions are described via OpenAPI specifications that allow the agent to understand what parameters are required and what to expect as a response.

**Knowledge Bases**: provide agents with access to enterprise information repositories through the Retrieval Augmented Generation (RAG) pattern. Knowledge bases index documents in vector stores such as Amazon OpenSearch Serverless or Amazon Aurora PostgreSQL, and allow agents to retrieve contextual information to enrich their responses.

### The Four Patterns of the AWS Guide

The official AWS guide proposes four distinct multi-agent orchestration patterns, each suited to different scenarios and requirements:

**Pattern 1 - Amazon Bedrock AgentCore**: uses the AgentCore service to route customer queries to expert agents specialized in order management, product recommendations, personalization, and troubleshooting. AgentCore natively manages deployment, monitoring, and communication between agents, supporting the A2A protocol for interoperability with agents built on different frameworks.

**Pattern 2 - Amazon Bedrock Multi-Agent Collaboration**: leverages Bedrock's native multi-agent collaboration feature, where a supervisor agent orchestrates specialized sub-agents through integrated collaboration features. This pattern handles task delegation and response aggregation with native monitoring and enterprise-grade reliability.

**Pattern 3 - Agent Squad (Multi-Agent as Microservices)**: AI agents operate as independent microservices with custom coordination logic through message queues. This approach offers complete control over agent behavior and integration with external systems and human agents. Agent Squad is an open source framework developed by AWS Labs that supports intelligent intent classification and routing to the appropriate agent.

**Pattern 4 - LangGraph on Amazon ECS**: a LangGraph-based supervisor agent running on Amazon ECS coordinates specialized sub-agents, facilitating task delegation, context sharing, and response synthesis across distributed agents. LangGraph allows modeling the orchestration flow as a directed graph, offering flexibility in defining execution paths.

### Typical Execution Flow

The flow of a request in an AWS multi-agent system follows a well-defined path:

1. The user submits a request through an entry channel (API Gateway, web application, chatbot)
2. The request reaches the Supervisor Agent that analyzes it using its foundation model
3. The supervisor identifies the intent and determines which collaborator or combination of collaborators to involve
4. The supervisor sends the sub-tasks to the relevant collaborators, potentially in parallel
5. Each collaborator processes its sub-task, invoking action groups and/or consulting knowledge bases
6. The collaborators return the results to the supervisor
7. The supervisor synthesizes the partial responses and generates the final response
8. If necessary, the supervisor reiterates the cycle by requesting further information from the user or delegating additional tasks

---

## 3. Amazon Bedrock Agents

### How Bedrock Agents Work

Amazon Bedrock Agents orchestrate interactions between foundation models, data sources, software applications, and conversations with the user. The internal architecture of a Bedrock agent is based on an iterative reasoning loop composed of four distinct phases:

**Pre-processing**: the agent analyzes the user input to contextualize the request, validate relevance, and prepare the prompt for the orchestration phase. In this phase, initial filters and guardrails are applied.

**Orchestration**: the heart of the agent's reasoning. The agent interprets the input with the foundation model and generates a **rationale** that makes the logic of the next step explicit. The agent predicts which action to invoke from an action group or which knowledge base to query. If the agent does not have sufficient information to invoke an action, it can query an associated knowledge base to retrieve additional context.

**Knowledge Base Response Generation**: when the agent queries a knowledge base, the retrieval results are processed to generate a contextualized response. The model synthesizes the retrieved information in the context of the current conversation.

**Post-processing**: the agent formats the final response, applies any transformations, and verifies that the response complies with the configured instructions and guardrails.

This cycle is **iterative**: after each observation derived from the invocation of an action or from the consultation of a knowledge base, the agent reevaluates the situation and determines whether further steps are necessary. The loop continues until the agent has gathered sufficient information to provide a complete response or needs further input from the user.

### Action Groups

The **Action Groups** represent the mechanism through which a Bedrock agent interacts with the outside world. An action group defines a set of actions that the agent can perform on behalf of the user. Each action is typically implemented as an **AWS Lambda** function that executes specific business logic.

The definition of an action group includes:

- **OpenAPI Schema**: describes the available actions, the required parameters, the data types, and the expected responses. The agent uses this specification to understand when and how to invoke each action.
- **Lambda Function**: contains the actual implementation of the action. When the agent decides to invoke an action, Bedrock sends the parameters extracted from the conversation to the configured Lambda function.
- **Natural language description**: a textual description that helps the agent understand the purpose and context of use of each action.

Action groups are fundamental when the agent needs deterministic, structured, or real-time data that cannot be reliably inferred from the foundation model alone. Typical examples include: querying a database for order status, making a reservation, verifying product availability, or sending a notification.

A single agent can be associated with **multiple action groups**, each focused on a specific functional domain. This modularity allows composing agents with different capabilities without coupling between the different action groups.

### Knowledge Bases

The **Knowledge Bases** of Amazon Bedrock provide agents with access to proprietary enterprise information repositories through the **Retrieval Augmented Generation (RAG)** pattern. This approach allows enriching the foundation model's responses with organization-specific data, reducing hallucinations and increasing factual precision.

The operation of a knowledge base involves:

1. **Ingestion**: source documents (PDFs, web pages, text files, structured data) are processed, split into chunks, and converted into vector embeddings
2. **Indexing**: the embeddings are stored in a supported vector store (Amazon OpenSearch Serverless, Amazon Aurora PostgreSQL, Pinecone, Redis Enterprise Cloud)
3. **Retrieval**: when the agent queries the knowledge base, the system converts the query into an embedding, searches the vector store for the most similar chunks, and returns the relevant results
4. **Generation**: the agent uses the retrieved chunks as additional context to generate responses grounded in enterprise data

Knowledge bases can be shared between multiple agents or dedicated to specific collaborators, allowing a separation of informational competencies consistent with the specialization of individual agents.

---

## 4. Orchestration Patterns

### Supervisor Agent

The **Supervisor Agent** pattern is the most widespread orchestration model and the one natively supported by Amazon Bedrock Multi-Agent Collaboration. In this pattern, an agent designated as supervisor acts as the single coordination point for the entire collaboration.

The supervisor agent is responsible for:

- **Intent analysis**: interprets the user request to determine which competencies are needed
- **Planning**: breaks complex tasks into sub-activities assignable to specific collaborators
- **Routing**: routes sub-activities to the appropriate collaborator agents
- **Aggregation**: collects collaborator responses and synthesizes them into a coherent response
- **Context management**: maintains conversation continuity through interactions with different collaborators

Amazon Bedrock offers two collaboration modes for the supervisor:

**Supervisor Mode**: the supervisor receives every request, fully analyzes it, and decides how to distribute it among collaborators. This mode is suitable for complex requests that require the involvement of multiple collaborators or that need sophisticated planning.

**Supervisor with Routing Mode**: an optimized mode where the supervisor routes simple requests directly to the appropriate collaborator, bypassing full orchestration. For complex requests, the system automatically falls back to the full supervisor mode. This mode reduces latency for requests that require a single collaborator.

### Peer-to-Peer Agents

The **peer-to-peer** pattern involves agents communicating directly with each other without a central coordinator. In this model, each agent is autonomous and can initiate interactions with other agents based on their needs.

On AWS, the peer-to-peer pattern can be implemented through several approaches:

- **Agent-to-Agent Protocol (A2A)**: supported by Amazon Bedrock AgentCore, the A2A protocol allows agents built with different frameworks (Strands Agents, OpenAI Agents SDK, LangGraph, Google ADK, Claude Agents SDK) to communicate directly. Each agent publishes an **Agent Card**, a JSON file that describes its identity, capabilities, endpoint, and authentication requirements. This dynamic discovery mechanism allows agents to query peers to determine their capabilities before delegating tasks.
- **Message queues (SQS/SNS)**: agents communicate asynchronously through Amazon SQS or Amazon SNS, decoupling producers from consumers and ensuring the resilience of communication.
- **Agent Squad Framework**: the AWS Labs open source framework implements a pattern where specialized agents operate as independent microservices with custom coordination logic through message queues.

The A2A protocol supports both synchronous and asynchronous interactions using **JSON-RPC 2.0** over HTTP/S or **Server-Sent Events (SSE)**. This enables scenarios where an analysis agent can request data from a data access agent, which in turn can consult a validation agent, all without a central coordinator.

### Hierarchical Orchestration

**Hierarchical** orchestration extends the supervisor pattern by creating a tree structure of agents on multiple levels. Amazon Bedrock Multi-Agent Collaboration natively supports hierarchies where a sub-agent can in turn be configured as a supervisor of other sub-agents, forming a tree of agents with a soft limit of **three hierarchical levels**.

This pattern is particularly suitable for complex enterprise scenarios where:

- Competencies are organized in domains and sub-domains (for example, a first-level supervisor for customer service that delegates to second-level supervisors for sales, technical support, and administration, each of which coordinates specialized operational agents)
- The volume and variety of requests require a hierarchical distribution of the workload
- Security and governance policies require differentiated levels of access and authorization

In practice, a typical hierarchy may take this structure:

```
Level 1 Supervisor (Entry Point)
    |
    |-- L2 Supervisor: Order Management
    |       |-- Agent: Shipment Tracking
    |       |-- Agent: Returns and Refunds
    |       |-- Agent: Order Modification
    |
    |-- L2 Supervisor: Product Support
    |       |-- Agent: Hardware Troubleshooting
    |       |-- Agent: Software Troubleshooting
    |       |-- Agent: FAQ and Documentation
    |
    |-- L2 Supervisor: Sales
            |-- Agent: Product Recommendations
            |-- Agent: Quote Configuration
            |-- Agent: Promotions and Discounts
```

Communication between hierarchical levels uses a standardized protocol and supports **payload referencing**, a mechanism that allows efficient sharing of large content blocks between agents using unique identifiers instead of repeatedly transmitting the same data.

---

## 5. AWS Services Involved

### AWS Lambda

**AWS Lambda** is the serverless computing service that serves as the backbone for the execution of agent business logic. Each action group of a Bedrock agent is typically supported by one or more Lambda functions that implement concrete actions: database queries, external API calls, data processing, sending notifications.

Lambda offers significant advantages in the multi-agent context: automatic scaling to handle peaks of concurrent requests from multiple agents, pay-per-use model that adapts to the actual volume of invocations, and no infrastructure management required. Each Lambda function operates in an isolated environment with its own IAM role, ensuring the separation of privileges between the actions of different agents.

### AWS Step Functions

**AWS Step Functions** acts as the central orchestrator in patterns that require complex multi-step workflows. Step Functions allows modeling the entire orchestration flow as a visual state machine, managing:

- **Parallel execution**: multiple agents can be invoked simultaneously on independent sub-tasks
- **Conditional logic**: the flow can branch based on the partial results of the agents
- **Retry management**: automatic retry policies with exponential backoff to handle transient failures
- **Timeout and compensation**: definition of time limits and compensation logic to ensure resilience
- **Human-in-the-loop**: integration of human approval points in the orchestration flow

Step Functions maintains the state of the entire execution, allowing each stateless agent to operate without worrying about the persistence of the global context. Native integration with Amazon Bedrock allows invoking agents directly as tasks within the state machine.

### Amazon SQS and Amazon SNS

**Amazon Simple Queue Service (SQS)** and **Amazon Simple Notification Service (SNS)** provide the messaging infrastructure for asynchronous communication between agents:

- **SQS** ensures reliable delivery of messages with at-least-once or exactly-once (FIFO) semantics, supporting dead-letter queues for handling unprocessed messages
- **SNS** enables the pub/sub pattern where an agent can notify events to multiple subscribers simultaneously

These services are particularly relevant in the Agent Squad pattern and in peer-to-peer patterns, where agents operate as decoupled microservices. A central router implemented as a Lambda function can intercept messages from the SQS queue and determine which specialized agent should handle the next task based on application metadata.

### Amazon DynamoDB

**Amazon DynamoDB** is the serverless NoSQL database used for state persistence and traceability in the multi-agent context:

- **Session state**: storage of conversation context between successive interactions
- **Execution history**: log of the entire execution chain for audit and debugging
- **Agent state**: current state of each agent and tasks being processed
- **Configuration store**: dynamic configuration parameters of agents

DynamoDB offers single-digit millisecond latency and automatic capacity scaling, essential characteristics for high-throughput multi-agent systems. The **DynamoDB Streams** feature also allows reacting in real-time to state changes, triggering Lambda functions for notifications or further processing.

---

## 6. State Management

### The Challenge of State in Multi-Agent Systems

State management in a multi-agent system represents one of the most significant architectural challenges. Unlike a monolithic application, where the state resides in a single context, a multi-agent system distributes processing among independent components that need to share contextual information.

### State Management Strategies on AWS

**Conversation State**: Amazon Bedrock Agents natively manages the conversation context through a session identifier. Each invocation within the same session maintains the history of previous messages, allowing the supervisor and collaborators to operate in the context of the complete conversation. The **payload referencing** feature optimizes communication between agents by allowing the sharing of large content blocks via unique references rather than repeatedly transmitting the data.

**Orchestration State**: AWS Step Functions maintains the state of the entire workflow, including the intermediate results of each agent, routing decisions, and execution metadata. The state is persistent and recoverable, allowing the resumption of interrupted workflows and post-hoc analysis of the execution flow.

**Domain State**: DynamoDB serves as a persistent store for business state: information about orders, customer preferences, support ticket status. Each agent accesses the domain state relevant to its area of competence, ensuring consistency and real-time updates.

**Shared State Between Agents**: for scenarios that require sharing state between agents in real-time, AWS offers several options:
- **Amazon ElastiCache (Redis)**: in-memory cache for low-latency shared state
- **DynamoDB with conditional writes**: atomic updates with consistency guarantee
- **Parameter Store / Secrets Manager**: for shared configurations and credentials

### Context Window and Limits

A critical aspect of state management is the limit of the foundation model's context window. In multi-agent systems, the accumulated context can grow rapidly. Mitigation strategies include:

- Periodic context summarization by the supervisor
- Selective transmission of context to collaborators (only information relevant to the sub-task)
- Storage of historical context in knowledge bases for on-demand retrieval
- Use of Bedrock's prompt management feature to optimize the use of the context window

---

## 7. Security and Governance

### IAM and the Principle of Least Privilege

Security in a multi-agent system on AWS is based on the **AWS Identity and Access Management (IAM)** service and the rigorous application of the principle of least privilege. Each component of the system operates with a dedicated IAM role that grants exclusively the permissions strictly necessary:

- **Bedrock agent role**: permissions to invoke the foundation model, access associated knowledge bases, and invoke Lambda functions of action groups
- **Lambda function role**: permissions to access the specific resources required (DynamoDB tables, external APIs, SQS queues)
- **Step Functions role**: permissions to invoke agents and services involved in the workflow

Each agent or family of agents should have a dedicated identity with a set of permissions strictly aligned to its responsibilities. Time-limited sessions reduce credential exposure, and workflows that require human participation should require multi-factor authentication (MFA) for elevated-privilege steps.

### Amazon Bedrock Guardrails

The **Amazon Bedrock Guardrails** provide a configurable level of protection that can be applied to each agent individually:

- **Content filtering**: blocking inappropriate, offensive, or dangerous content in agent responses
- **Denied topics**: definition of topics that agents should not address
- **Word filters**: filtering of specific words or phrases
- **PII detection**: automatic detection and masking of personally identifiable information (names, email addresses, credit card numbers)
- **Contextual grounding**: verification that responses are grounded in the data provided, reducing hallucinations

Each agent in the multi-agent hierarchy can have distinct guardrails, allowing differentiated security policies based on the role and access level of the agent.

### Validation Layer

A fundamental best practice involves implementing a **validation layer** between agents and external resources. Rather than allowing agents to invoke tools directly, requests should pass through a validation layer built with Amazon API Gateway and Lambda that:

- Verifies request authorization
- Ensures payloads meet security criteria
- Applies rate limiting to prevent abuse
- Checks business rules before forwarding the request
- Logs every interaction for audit and compliance

### Logging and Monitoring

Complete traceability of interactions is essential for governance and debugging:

- **Amazon CloudWatch Logs**: centralized collection of logs from all agents, Lambda functions, and Step Functions workflows
- **AWS CloudTrail**: logging of all API calls for security auditing
- **Amazon Bedrock Model Invocation Logging**: detailed log of every invocation to foundation models, including input, output, and usage metrics
- **AWS X-Ray**: distributed tracing to visualize the execution flow through the entire chain of agents and services
- **Amazon CloudWatch Metrics**: custom metrics to monitor latency, throughput, error rates, and usage patterns of individual agents

The combination of these services provides a complete view of the multi-agent system's behavior, allowing the identification of bottlenecks, anomalies, and optimization opportunities.

---

## 8. Enterprise Use Cases

### Intelligent Customer Service

The reference scenario of the AWS guide is a customer service system that orchestrates specialized agents for:
- **Order management**: shipment tracking, order modification, returns management with direct access to OMS (Order Management System) systems
- **Product recommendations**: personalized suggestions based on purchase history, preferences, and conversation context
- **Technical troubleshooting**: guided diagnosis of problems with access to technical knowledge bases and resolution procedures
- **Personalization**: experience adaptation based on customer profile, segmentation, and historical behavior

The supervisor agent analyzes each request and routes it to the appropriate agent, maintaining the conversational context across different interactions. Simple requests are handled directly by the single collaborator (routing mode), while complex ones involve multiple collaborators in sequence or in parallel.

### Financial Services

In the financial sector, a multi-agent system can orchestrate:
- A **risk analysis** agent that evaluates risk profiles by accessing market and historical data
- A **compliance** agent that verifies regulatory compliance of operations by consulting regulatory knowledge bases
- A **customer assistance** agent for mortgages that handles requests on existing mortgages, new applications, and general questions (as in the Online Mortgage Assistant example in the Bedrock documentation)
- A **fraud detection** agent that analyzes transactional patterns in real-time

### Healthcare and Life Sciences

Hierarchical multi-agent orchestration is particularly relevant in the pharmaceutical and healthcare sector:
- **Drug discovery** agents organized hierarchically for molecular analysis, efficacy prediction, and safety evaluation
- **Clinical triage** agents that combine symptom analysis, consultation of medical guidelines, and appointment scheduling
- **Document management** agents for review and automatic classification of scientific literature

### Supply Chain and Logistics

Multi-agent systems for supply chain management:
- **Demand forecasting** agent that analyzes historical trends and external factors
- **Inventory optimization** agent that manages stock levels and reorder points
- **Logistics routing** agent that optimizes routes and transportation assignments
- **Supplier management** agent that manages supplier relationships and performance

### IT Operations and DevOps

In the area of IT operations, multi-agent systems can coordinate:
- **Monitoring** agent that analyzes metrics and logs to identify anomalies
- **Incident response** agent that correlates alerts and proposes remediation actions
- **Change management** agent that evaluates the impact of infrastructure changes
- Coordination via A2A protocol between agents built with different frameworks, as demonstrated by AWS reference scenarios where a host agent based on Google ADK coordinates operational agents based on OpenAI Agents SDK

---

## 9. Resources and References

### Official AWS Documentation

- [Guidance for Multi-Agent Orchestration on AWS](https://aws.amazon.com/solutions/guidance/multi-agent-orchestration-on-aws/) - The complete reference guide with the four orchestration patterns
- [Use multi-agent collaboration with Amazon Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-multi-agent-collaboration.html) - Official documentation on multi-agent collaboration in Bedrock
- [How Amazon Bedrock Agents works](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-how.html) - Internal architecture of Bedrock agents
- [Use action groups to define actions for your agent](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-action-create.html) - Guide to creating action groups
- [How Amazon Bedrock Knowledge Bases work](https://docs.aws.amazon.com/bedrock/latest/userguide/kb-how-it-works.html) - Functioning of knowledge bases
- [Customize agent orchestration strategy](https://docs.aws.amazon.com/bedrock/latest/userguide/orch-strategy.html) - Customization of orchestration strategies
- [Create multi-agent collaboration](https://docs.aws.amazon.com/bedrock/latest/userguide/create-multi-agent-collaboration.html) - Practical guide to creating multi-agent collaborations

### AWS Blog Posts

- [Introducing multi-agent collaboration capability for Amazon Bedrock](https://aws.amazon.com/blogs/aws/introducing-multi-agent-collaboration-capability-for-amazon-bedrock/) - Announcement of the multi-agent collaboration feature
- [Unlocking complex problem-solving with multi-agent collaboration on Amazon Bedrock](https://aws.amazon.com/blogs/machine-learning/unlocking-complex-problem-solving-with-multi-agent-collaboration-on-amazon-bedrock/) - Technical deep dive on solving complex problems
- [Design multi-agent orchestration with reasoning using Amazon Bedrock and open source frameworks](https://aws.amazon.com/blogs/machine-learning/design-multi-agent-orchestration-with-reasoning-using-amazon-bedrock-and-open-source-frameworks/) - Orchestration with reasoning and open source frameworks
- [Build multi-agent systems with LangGraph and Amazon Bedrock](https://aws.amazon.com/blogs/machine-learning/build-multi-agent-systems-with-langgraph-and-amazon-bedrock/) - LangGraph integration with Bedrock
- [Introducing agent-to-agent protocol support in Amazon Bedrock AgentCore Runtime](https://aws.amazon.com/blogs/machine-learning/introducing-agent-to-agent-protocol-support-in-amazon-bedrock-agentcore-runtime/) - A2A protocol support
- [Advanced fine-tuning techniques for multi-agent orchestration](https://aws.amazon.com/blogs/machine-learning/advanced-fine-tuning-techniques-for-multi-agent-orchestration-patterns-from-amazon-at-scale/) - Advanced fine-tuning techniques for multi-agent orchestration
- [Using Strands Agents to create a multi-agent solution with Meta's Llama 4 and Amazon Bedrock](https://aws.amazon.com/blogs/machine-learning/using-strands-agents-to-create-a-multi-agent-solution-with-metas-llama-4-and-amazon-bedrock/) - Multi-agent solution with Strands Agents and Llama 4
- [Introducing Amazon Bedrock AgentCore](https://aws.amazon.com/blogs/aws/introducing-amazon-bedrock-agentcore-securely-deploy-and-operate-ai-agents-at-any-scale/) - Introduction to AgentCore

### Open Source Repositories and Frameworks

- [Agent Squad (AWS Labs)](https://github.com/awslabs/agent-squad) - Framework for managing multiple AI agents and complex conversations
- [Agentic Orchestration (AWS Samples)](https://github.com/aws-samples/agentic-orchestration) - Examples of agentic orchestration on AWS

### Community Articles

- [Multi-Agent Orchestration With AWS Step Functions (DZone)](https://dzone.com/articles/multi-agent-multi-function-orchestration-with-aws) - Multi-agent orchestration with Step Functions
- [Orchestrating Multi-Agent Systems with AWS Step Functions (XenonStack)](https://www.xenonstack.com/blog/multi-agent-systems-with-aws) - Guide to orchestration with Step Functions
- [Hierarchical Multi-Agent Systems with Amazon Bedrock (Medium)](https://jtanruan.medium.com/hierarchical-multi-agent-systems-with-amazon-bedrock-orchestrating-agents-for-drug-discovery-1c6b6aff9acd) - Hierarchical multi-agent systems for drug discovery

---

*Document updated March 2026. The information is based on official AWS documentation and features available at the time of writing. Amazon Bedrock and related services are continuously evolving; consult the official AWS documentation for the most recent updates.*
