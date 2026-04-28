# Pattern 38 - Model Context Protocol (MCP)

## 1. Overview

The **Model Context Protocol (MCP)** is an open protocol created by **Anthropic** and publicly announced in **November 2024**. MCP defines a standard for connecting Large Language Model (LLM) applications to external data sources, tools, and services in a uniform and interoperable way. The fundamental goal is to provide a single standardized interface through which any AI system can access the context it needs -- whether it's files on disk, remote APIs, databases, Git repositories, or any other resource.

MCP is inspired by the **Language Server Protocol (LSP)**, the standard that revolutionized how code editors support programming languages. Just as LSP standardized the integration of language servers in IDEs, MCP standardizes the integration of context and tools in the AI applications ecosystem.

The protocol is entirely based on **JSON-RPC 2.0** for message encoding, operates via stateful connections, and implements a sophisticated capabilities negotiation system between client and server. The official specification is defined first in **TypeScript** (as the source of truth schema) and then made available as **JSON Schema** to ensure cross-compatibility.

In **December 2025**, Anthropic donated MCP to the **Agentic AI Foundation (AAIF)**, a fund directed under the auspices of the **Linux Foundation**, marking the protocol's transition from a corporate project to an industry standard governed by the community. To date, the official SDKs in Python and TypeScript total approximately **97 million monthly downloads**, and the protocol is supported natively by Anthropic, OpenAI, Google, and Microsoft.

---

## 2. The Problem It Solves

### The N x M Problem

Before MCP, every AI application that needed access to external tools had to implement ad-hoc integrations. With **N** AI applications and **M** services to integrate, you potentially needed **N x M** custom integrations, each with its own protocol, data format, and connection logic.

Consider a concrete scenario: a team has an AI assistant in its IDE, an internal chatbot, and an AI automation system. These three systems must access GitHub, Slack, a PostgreSQL database, and a local file system. Without a standard, this requires **3 x 4 = 12** separate integrations, each to be developed, maintained, and updated independently.

### The MCP Solution

MCP reduces this complexity from **N x M** to **N + M**. Each AI application implements a single MCP client, and each service exposes a single MCP server. The MCP client speaks the same protocol with any server, and any MCP server is accessible from any compatible client.

This approach brings several advantages:

- **Reduction of integration complexity**: a single interface to implement for each side of the communication.
- **Reuse and composability**: an MCP server written for GitHub works with Claude, with ChatGPT, with Cursor, with any MCP client.
- **Standardization of security**: the protocol defines clear patterns for user consent, access control, and input validation.
- **Shared ecosystem**: the community can create and share MCP servers that work everywhere.
- **Independent evolution**: client and server can evolve independently, as long as they respect the protocol.

---

## 3. Architecture

### 3.1 Client-Host-Server Model

MCP follows a **client-host-server** architecture in which each host can run multiple client instances. The three main actors are:

**Host**: the application process that acts as a container and coordinator. The host creates and manages multiple client instances, controls connection permissions and lifecycle, applies security policies, and manages user authorization decisions. The host coordinates integration with the AI model and manages the aggregation of context from different clients. Examples of hosts are Claude Desktop, an IDE with AI support (like Cursor or VS Code with Copilot), or a custom application.

**Client**: each client is created by the host and maintains an isolated connection to a single server. There is a **1:1** relationship between client and server: each client establishes a stateful session with a specific server, manages protocol negotiation and capabilities exchange, routes messages bidirectionally, manages subscriptions and notifications, and maintains security boundaries between different servers.

**Server**: servers provide context and specialized functionality. They expose resources, tools, and prompts via MCP primitives, operate independently with focused responsibilities, can request sampling through client interfaces, and can be local processes or remote services. A fundamental aspect of the design is that **servers cannot see the entire conversation nor "spy" on other servers**: each server receives only the strictly necessary contextual information.

### 3.2 Transport Layer

MCP defines two standard transport mechanisms:

#### stdio (Standard Input/Output)

The **stdio** transport is the simplest and most direct. The client launches the MCP server as a subprocess: the server reads JSON-RPC messages from its `stdin` and writes responses to its `stdout`. Messages are delimited by newlines and must not contain embedded newlines. The server can use `stderr` for logging. This transport is ideal for local servers and command-line tools. Clients should support stdio whenever possible.

The lifecycle is managed directly: the client launches the subprocess, exchanges messages via stdin/stdout, and at shutdown closes stdin, waits for the server to terminate, and in case of failed termination first sends `SIGTERM` and then `SIGKILL`.

#### Streamable HTTP

**Streamable HTTP** is the modern transport for remote servers, introduced in the March 2025 specification to replace the previous HTTP+SSE transport. In this model the server operates as an independent process that handles multiple connections. The server exposes a single HTTP **MCP endpoint** (e.g., `https://example.com/mcp`) that supports both POST and GET.

Each JSON-RPC message from the client is a new HTTP POST to the MCP endpoint. The client must include an `Accept` header listing both `application/json` and `text/event-stream`. The server can respond with a single JSON object or open an SSE (Server-Sent Events) stream to send multiple server messages. The SSE stream can include JSON-RPC requests and notifications from the server before the final response.

The transport supports **session management** via a `MCP-Session-Id` header that the server can assign during initialization. It also supports **connection resumption** (resumability) via SSE event IDs and the `Last-Event-ID` header, allowing the client to reconnect and resume from where it left off in case of disconnection.

For security, servers must validate the `Origin` header on all incoming connections to prevent DNS rebinding attacks, and when running locally must bind only to localhost (127.0.0.1) instead of all network interfaces.

#### Custom Transport

It is also possible to implement custom transports, provided you preserve the JSON-RPC message format and the requirements of the MCP lifecycle.

### 3.3 Protocol Messages (JSON-RPC 2.0)

All MCP messages strictly follow the JSON-RPC 2.0 specification and must be encoded in UTF-8. The protocol defines three types of messages:

**Request**: sent from client to server or vice versa to initiate an operation. They contain an `id` field (string or integer, never null, never reused in the same session), a `method` field, and an optional `params` field:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": { "cursor": "optional-cursor-value" }
}
```

**Response**: sent in response to requests. They can be successful responses (with a `result` field) or error responses (with an `error` field containing `code` and `message`):

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": { "tools": [...] }
}
```

**Notification**: unidirectional messages sent without waiting for a response. They do not contain an `id` field:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/tools/list_changed"
}
```

---

## 4. Capabilities

Capabilities are the functional core of MCP. They are divided into **server features** (offered by the server to the client) and **client features** (offered by the client to the server).

### 4.1 Tools - Standardized Function Calling

**Tools** are executable functions that allow models to interact with external systems: query databases, call APIs, perform computations, write files. Each tool is uniquely identified by a name and includes metadata that describes its input schema.

Tools are designed to be **model-controlled**: the language model can discover and invoke tools automatically based on its understanding of the context. For security, however, there must always be a **human in the loop** with the ability to deny invocations.

The lifecycle of a tool consists of:

1. **Discovery** (`tools/list`): the client asks the server for the list of available tools. Each tool includes `name`, `description`, `inputSchema` (JSON Schema that defines the expected parameters), and optionally `outputSchema`, `annotations`, and icons.

2. **Invocation** (`tools/call`): the client invokes a tool by passing name and arguments. The server executes the operation and returns the result, which can contain text, images, audio, links to resources, or embedded resources.

3. **Update notifications** (`notifications/tools/list_changed`): the server notifies the client when the list of available tools changes.

Example of tool definition:

```json
{
  "name": "get_weather",
  "title": "Weather Information Provider",
  "description": "Get current weather information for a location",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name or zip code"
      }
    },
    "required": ["location"]
  }
}
```

Error handling in tools distinguishes two levels: **protocol errors** (JSON-RPC errors for unknown tools, malformed requests) and **execution errors** (reported in the result with `isError: true`, useful because the model can use them to self-correct and retry with different parameters).

Tools also support an optional **output schema** for structured validation of results, and **task support** for long-running asynchronous operations introduced with specification 2025-11-25.

### 4.2 Resources - Access to Data and Files

**Resources** provide a standardized way for servers to expose data to clients: files, database schemas, application-specific information. Each resource is uniquely identified by a **URI** (compliant with RFC 3986).

Resources are designed to be **application-controlled**: the host application decides how to incorporate the context based on its needs, for example via UI elements for explicit selection, search and filter, or automatic inclusion based on heuristics.

The main protocol methods for resources are:

- **`resources/list`**: discovers available resources (with pagination support).
- **`resources/read`**: retrieves the content of a specific resource via URI.
- **`resources/templates/list`**: lists templates of parameterized resources (based on URI Templates RFC 6570).
- **`resources/subscribe`**: subscribes to changes to a specific resource.
- **`notifications/resources/updated`**: notification of resource update.
- **`notifications/resources/list_changed`**: notification of change in the list.

Resources can contain text or binary data (encoded in base64) and support **annotations** to indicate the target audience (`user`, `assistant`, or both), priority (from 0.0 to 1.0), and the timestamp of last modification.

Common URI schemes include `https://` for web resources, `file://` for filesystem-like resources, `git://` for Git integrations, plus the ability to define custom schemes compliant with RFC 3986.

### 4.3 Prompts - Reusable Templates

**Prompts** are pre-defined message templates that guide interactions with language models. Unlike tools (model-controlled) and resources (application-controlled), prompts are **user-controlled**: they are exposed by servers to clients with the intent that the user explicitly select them, typically as slash commands or menu options in the interface.

The protocol methods for prompts are:

- **`prompts/list`**: lists available prompts (with pagination).
- **`prompts/get`**: retrieves a specific prompt with its compiled arguments.
- **`notifications/prompts/list_changed`**: notification of update in the list.

Each prompt can have **arguments** for personalization, with support for autocompletion via the completion API. The result of `prompts/get` is a sequence of messages (`PromptMessage`) with `user` or `assistant` roles, containing text, images, audio, or embedded resources.

Example of prompt definition:

```json
{
  "name": "code_review",
  "title": "Request Code Review",
  "description": "Asks the LLM to analyze code quality and suggest improvements",
  "arguments": [
    {
      "name": "code",
      "description": "The code to review",
      "required": true
    }
  ]
}
```

### 4.4 Sampling - LLM Completions

**Sampling** is a client-side feature that allows servers to request generations (completions) from the language model through the client, without the server needing direct access to API keys. This mechanism enables nested agentic behaviors: a server can request the model to reason about intermediate data during the execution of a complex operation.

The sampling flow follows a precise pattern:

1. The server sends a `sampling/createMessage` request to the client.
2. The client presents the request to the user for approval (**human in the loop**).
3. The user can review and modify the prompt before sending.
4. The client forwards the approved request to the LLM model.
5. The generated response is presented to the user for review.
6. The approved response is returned to the server.

The sampling request includes messages, an optional system prompt, a token limit, and **model preferences**. Preferences are expressed via:

- **Hints**: suggestions about specific models (e.g., `"claude-3-sonnet"`), treated as substrings for flexible matching.
- **Normalized abstract priorities** (0-1): `costPriority` (minimize costs), `speedPriority` (minimize latency), `intelligencePriority` (maximize capabilities).

Since November 2025, sampling also supports the use of **tools within sampling requests**, enabling complex agentic loops in which the model can call tools, receive results, and continue the conversation within a single sampling flow. The client must declare support via the `sampling.tools` capability.

### 4.5 Other Client Features

**Roots**: clients can expose filesystem "roots" to servers, defining the boundaries within which servers can operate. Roots provide information about directories and files that the server has access to, and support update notifications when the list changes.

**Elicitation**: servers can request additional information from the user through the client, a server-initiated communication mechanism to gather necessary input during the execution of operations.

---

## 5. Lifecycle

The lifecycle of an MCP connection consists of three strictly sequential phases:

### 5.1 Initialization

Initialization **must** be the first interaction between client and server. The client starts the phase by sending an `initialize` request containing:

- The supported **protocol version** (should be the most recent, e.g., `"2025-11-25"`).
- The **client capabilities** (roots, sampling, elicitation, tasks).
- The **client implementation information** (name, version, description, icons).

The server responds with its own capabilities and information. A **version negotiation** follows: if the server supports the requested version, it responds with the same; otherwise it proposes its most recent version. If the client does not support the version proposed by the server, it should disconnect.

After initialization, the client sends a `notifications/initialized` notification to signal that it is ready for normal operations. Before this notification the server should not send requests (except ping and logging).

### 5.2 Operation

During the operational phase, client and server exchange messages according to the negotiated capabilities. Both parties must respect the negotiated protocol version and use only the agreed capabilities. The typical flow includes:

- **Client requests**: actions initiated by the user or model (e.g., invoking tools, reading resources).
- **Server requests**: typically sampling or elicitation requests.
- **Notifications**: asynchronous updates on resources, changes in available lists.

The protocol also supports cross-cutting utilities: structured **logging**, **progress tracking** for long operations, **cancellation** of in-progress requests, and per-request configurable **timeouts** to prevent stuck connections.

### 5.3 Shutdown

Termination does not define specific messages but relies on the underlying transport mechanism. For **stdio**, the client closes the subprocess's stdin, waits for termination, and in case of no response proceeds with SIGTERM and then SIGKILL. For **HTTP**, shutdown occurs by terminating the associated HTTP connections; the client should send an HTTP DELETE to the MCP endpoint with the `MCP-Session-Id` header to explicitly terminate the session.

---

## 6. Ecosystem

### 6.1 Industry Adoption

The MCP ecosystem has reached significant size in less than two years since its introduction. According to public registries (including PulseMCP), as of October 2025, over **5,800 MCP servers** and **300+ MCP clients** were cataloged. The main AI vendors standardized around MCP during 2025:

- **Anthropic**: native support in Claude Desktop, Claude Code, and the Claude APIs.
- **OpenAI**: MCP integration in ChatGPT, in their Agents SDK, and in desktop applications.
- **Google**: support in Gemini, Android Studio, and Google Cloud through Vertex AI.
- **Microsoft**: support in GitHub Copilot, VS Code, and Azure AI.

### 6.2 Available MCP Servers

The community has developed MCP servers for practically every relevant service and platform:

- **Development**: GitHub, GitLab, Linear, Jira, local filesystem, Git.
- **Communication**: Slack, Discord, email.
- **Data and storage**: PostgreSQL, MySQL, MongoDB, Redis, Elasticsearch, S3.
- **Productivity**: Google Drive, Notion, Confluence, Google Calendar.
- **Web and API**: browser automation (Puppeteer, Playwright), web fetch, scraping.
- **Cloud and infrastructure**: AWS, GCP, Azure, Docker, Kubernetes.
- **Monitoring**: Sentry, Datadog, charts, and analytics.

### 6.3 SDKs and Tools

The official SDKs are available in **TypeScript** and **Python**, with community SDKs in Go, Rust, Java, C#, and other languages. Tools like the **MCP Inspector** allow you to test and debug MCP servers interactively, while the **MCP registry** (community-driven, introduced with specification 2025-11-25) facilitates the discovery and distribution of servers.

### 6.4 Governance

After the donation to the Linux Foundation through the AAIF, the protocol evolves via a community process of **Specification Enhancement Proposals (SEPs)**. The community gathers at dedicated events such as MCP Dev Summit, MCP Night, and MCP Dev Days. According to Gartner, by 2026, 75% of API gateway vendors and 50% of iPaaS vendors will have integrated MCP functionality.

---

## 7. Impact on Agents

### 7.1 Universal Tool Interoperability

MCP radically transforms how AI agents are built. Before MCP, every agentic framework (LangChain, CrewAI, AutoGen, etc.) implemented its own tool system, with its own conventions for defining functions, input schemas, and response formats. This created closed ecosystems: a tool written for LangChain did not work with CrewAI without modifications.

With MCP, a tool is a tool. An MCP server that exposes an interface to PostgreSQL works identically with any agentic framework that implements an MCP client. This frees developers from dependence on a single framework and creates an ecosystem of truly reusable tools.

### 7.2 Agent Composability

MCP enables unprecedented composability. An agent can connect simultaneously to dozens of MCP servers, each specialized in a different domain, and dynamically discover the available capabilities. There is no need to configure a priori which tools are available: the agent can query the connected servers and explore the functionality in real-time.

This pattern naturally supports multi-layered agentic architectures: an orchestrator agent can delegate tasks to sub-agents, each with its own MCP servers, creating complex but modular execution chains.

### 7.3 Security and Control

MCP's design places security at the center. The architecture ensures that:

- Servers cannot access the entire conversation history.
- Each server connection is isolated from the others.
- The user must explicitly approve access to data and operations.
- Tool annotations (such as `readOnlyHint`, `destructiveHint`, `idempotentHint`) provide metadata about tool behavior, allowing clients to implement differentiated approval policies.

These guarantees are essential in enterprise contexts where data is sensitive and operations can have irreversible consequences.

### 7.4 Sampling and Nested Agentic Behaviors

The sampling feature is particularly powerful for agents. An MCP server can request the model to reason about intermediate results, effectively creating **nested LLM calls** within the execution of a tool. This enables patterns such as:

- A server that analyzes the results of a database query and asks the model to generate a report.
- A server that performs a series of operations and asks the model to decide the next step.
- Multi-turn agentic loops with tool use within sampling requests.

All this happens in a controlled manner, with the user always in the loop, and without the server needing its own API keys to access the model.

---

## 8. Practical Implementation

### 8.1 Creating an MCP Server in TypeScript

To create an MCP server with the official TypeScript SDK:

**Project setup:**

```bash
mkdir mcp-weather-server && cd mcp-weather-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node
```

**TypeScript configuration** (`tsconfig.json`):

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "strict": true
  },
  "include": ["src/**/*"]
}
```

**Server implementation** (`src/index.ts`):

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// Create the MCP server
const server = new McpServer({
  name: "weather-server",
  version: "1.0.0",
  description: "Provides weather information"
});

// Register a tool
server.tool(
  "get_weather",
  "Get current weather for a location",
  {
    location: z.string().describe("City name or zip code")
  },
  async ({ location }) => {
    // Logic to get weather (simplified example)
    const weatherData = await fetchWeather(location);
    return {
      content: [
        {
          type: "text",
          text: `Weather in ${location}: ${weatherData.temperature}, ${weatherData.conditions}`
        }
      ]
    };
  }
);

// Register a resource
server.resource(
  "config",
  "weather://config",
  "Server configuration",
  async () => ({
    contents: [
      {
        uri: "weather://config",
        mimeType: "application/json",
        text: JSON.stringify({ units: "celsius", updateInterval: 300 })
      }
    ]
  })
);

// Register a prompt
server.prompt(
  "weather_report",
  "Generate a weather report for a region",
  { region: z.string().describe("Geographic region") },
  async ({ region }) => ({
    messages: [
      {
        role: "user",
        content: {
          type: "text",
          text: `Generate a detailed weather report for the ${region} region, including temperature trends, precipitation forecasts, and any weather advisories.`
        }
      }
    ]
  })
);

// Start with stdio transport
const transport = new StdioServerTransport();
await server.connect(transport);
```

**Important note**: for stdio-based servers, never use `console.log()` since it writes to stdout and corrupts JSON-RPC messages. Use `console.error()` or a logging library that writes to stderr.

### 8.2 Creating an MCP Server in Python

With the official Python SDK and the `FastMCP` class:

```python
from mcp.server.fastmcp import FastMCP

# Initialize the server
mcp = FastMCP("weather-server")

@mcp.tool()
async def get_weather(location: str) -> str:
    """Get current weather for a location.

    Args:
        location: City name or zip code
    """
    # Logic to get weather
    weather = await fetch_weather(location)
    return f"Weather in {location}: {weather['temperature']}C, {weather['conditions']}"

@mcp.resource("weather://config")
async def get_config() -> str:
    """Server configuration resource."""
    return '{"units": "celsius", "updateInterval": 300}'

@mcp.prompt()
async def weather_report(region: str) -> str:
    """Generate a weather report for a region."""
    return f"Generate a detailed weather report for {region}."

# Start the server
if __name__ == "__main__":
    mcp.run(transport="stdio")
```

`FastMCP` uses Python type hints and docstrings to automatically generate tool definitions, making development extremely fast.

### 8.3 Client Configuration

To connect an MCP server to Claude Desktop, you configure the `claude_desktop_config.json` file:

```json
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["/path/to/weather-server/dist/index.js"]
    }
  }
}
```

For a remote server with Streamable HTTP:

```json
{
  "mcpServers": {
    "remote-weather": {
      "url": "https://weather-server.example.com/mcp"
    }
  }
}
```

### 8.4 Testing and Debugging

The **MCP Inspector** is the main tool for testing MCP servers. It allows you to:

- Connect to a server and view the negotiated capabilities.
- List and invoke tools interactively.
- Browse resources and templates.
- Test prompts with different arguments.
- View the JSON-RPC messages exchanged.

You can also use the SDK CLI for quick testing:

```bash
npx @modelcontextprotocol/inspector node dist/index.js
```

---

## 9. Resources and References

### Official Specification and Documentation

- **MCP Specification 2025-11-25**: [modelcontextprotocol.io/specification/2025-11-25](https://modelcontextprotocol.io/specification/2025-11-25)
- **Specification GitHub repository**: [github.com/modelcontextprotocol/modelcontextprotocol](https://github.com/modelcontextprotocol/modelcontextprotocol)
- **TypeScript Schema**: [github.com/modelcontextprotocol/specification/blob/main/schema/2025-11-25/schema.ts](https://github.com/modelcontextprotocol/specification/blob/main/schema/2025-11-25/schema.ts)

### Official SDKs

- **TypeScript SDK**: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- **Python SDK**: [github.com/modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk)

### Guides and Tutorials

- **Build an MCP Server (official)**: [modelcontextprotocol.io/docs/develop/build-server](https://modelcontextprotocol.io/docs/develop/build-server)
- **Anthropic Announcement**: [anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)
- **MCP on Wikipedia**: [en.wikipedia.org/wiki/Model_Context_Protocol](https://en.wikipedia.org/wiki/Model_Context_Protocol)

### Ecosystem and Community

- **MCP Blog - First anniversary**: [blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary](http://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)
- **A Year of MCP (Pento)**: [pento.ai/blog/a-year-of-mcp-2025-review](https://www.pento.ai/blog/a-year-of-mcp-2025-review)
- **Why the MCP Won (The New Stack)**: [thenewstack.io/why-the-model-context-protocol-won](https://thenewstack.io/why-the-model-context-protocol-won/)
- **MCP Adoption Statistics**: [mcpmanager.ai/blog/mcp-adoption-statistics](https://mcpmanager.ai/blog/mcp-adoption-statistics/)
- **Enterprise MCP Guide**: [guptadeepak.com - Complete Guide to MCP](https://guptadeepak.com/the-complete-guide-to-model-context-protocol-mcp-enterprise-adoption-market-trends-and-implementation-strategies/)
- **2026: Enterprise-Ready MCP (CData)**: [cdata.com/blog/2026-year-enterprise-ready-mcp-adoption](https://www.cdata.com/blog/2026-year-enterprise-ready-mcp-adoption)

### Analysis and Roadmap

- **Official Roadmap**: [modelcontextprotocol.io/development/roadmap](https://modelcontextprotocol.io/development/roadmap)
- **MCP Gartner Insights**: [k2view.com/blog/mcp-gartner](https://www.k2view.com/blog/mcp-gartner/)
- **Thoughtworks - MCP Impact 2025**: [thoughtworks.com - MCP Impact](https://www.thoughtworks.com/en-us/insights/blog/generative-ai/model-context-protocol-mcp-impact-2025)
