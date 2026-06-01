# Copilot Chat Conversation Export: Analysis of ChatN platform improvements

**User:** @ranga1729
**Thread URL:** https://github.com/copilot/c/fcf4b0da-579d-4f82-8013-1e68e0b1fa58

## @ranga1729

here is my proposed platform. read the whole document and get a good understanding. Architecture.darwio.xml are my architectural diagrams. ERD.drawio.xml is my ERD diagram for the database design. "Workflow Diagrams.drawio.xml" are the collection of flow charts explaining hwo the data and descitions are flowing in each scenario, to get an idea about how the system actually behave. also I have attached 2 github repos. these were the previous version of the same platform. but the issue with the previous version is it doesn't let users to choose between LLM providers, apis and models. but this new proposed system has more improvements than the previous version and should be more advanced, robust and relaible. First read those files and repos. then understand the new proposed system.

## Copilot

I now have a comprehensive understanding of both the previous version and the proposed new system. Here is a thorough analysis:

---

## Full Understanding — ChatN Platform

### What I've Read and Analyzed

- **SRS document** — Full platform specification
- **Architecture.drawio.xml** — 7-page architectural diagram set
- **ERD.drawio.xml** — Full database schema
- **Workflow Diagrams.drawio.xml** — 6 connection/message flow diagrams
- **`factn-assistant-service`** — Previous Python/FastAPI backend (v0.9.0)
- **`factn-assistant-tenant-dashboard`** — Previous Next.js 14 frontend

---

### Previous Version — What It Was

**Backend (`factn-assistant-service`)** — Python FastAPI, v0.9.0:
- **Single provider only**: Hardwired to **Google Gemini Live API** exclusively. The `gemini_runner.py` directly uses `genai.Client` — no abstraction layer.
- **MongoDB** as the primary database (not PostgreSQL as in the new spec)
- **Single project per tenant** with a fixed model (`gemini-2.5-flash-native-audio-preview`). The `gemini_model` field in `ProjectDoc` is a direct model string — no catalog/provider selection.
- Session state was stored **in-process memory** (`dict[str, SessionState]`) — not in Redis. Violated horizontal scaling.
- WebSocket gateway was a single `api/chat.py` route with no adapter abstraction.
- **No multi-provider routing** — you couldn't switch to OpenAI at all.
- Plan limits, webhook delivery, token quota — all present but provider-locked.

**Dashboard (`factn-assistant-tenant-dashboard`)** — Next.js 14:
- Pages: `/dashboard` (home), `/dashboard/keys`, `/dashboard/tools`, `/dashboard/prompt`, `/dashboard/voice-config`, `/dashboard/playground`, `/dashboard/stats`, `/dashboard/info`
- Single-project design — no project switcher, no provider/model selection UI
- No catalog browsing, no ephemeral token management UI
- No admin portal

---

### The New Proposed System — What's Different and Better

Here's a precise delta between old and new:

| Dimension | Previous (`factn-assistant-service`) | New (`ChatN` — SRS v1.0) |
|---|---|---|
| **Language/Runtime** | Python / FastAPI | **C# / ASP.NET Core 8** |
| **Database** | MongoDB | **PostgreSQL 15+** |
| **Session state storage** | In-process `dict` (NOT horizontally scalable) | **Redis** (fully distributed, cluster/sentinel) |
| **Provider support** | Google Gemini Live only (hardcoded) | OpenAI (Realtime, Chat Completions, Responses) + Google (Gemini Live, Generate Content) — **5 adapters** |
| **Provider abstraction** | None — direct `genai.Client` calls | `ILLMSessionAdapter` interface + Factory Method pattern |
| **LLM credentials** | `.env` file (`GEMINI_API_KEY`) | **Azure Key Vault** — never in env vars or DB |
| **Authentication** | JWT (`JWT_SECRET` in env) | RS256 JWT (private key in Key Vault) + HttpOnly refresh token cookie |
| **API Keys** | HMAC-SHA256 hash (implemented) | Same pattern, but enforced IDOR protection at repo layer, Redis fast-path lookup |
| **Ephemeral tokens** | Present but simple | Atomic Redis SETNX single-use enforcement |
| **Session isolation** | Single-node in-memory manager | Redis session registry, node_id-aware, admin pub/sub force-close |
| **Conversation history** | Not applicable (Gemini Live is stateful) | Redis for HTTP Orchestrator adapters; provider-side for WS Bridge adapters |
| **Protocol** | Custom but Gemini-specific event names | Fully standardized client-facing protocol (`text_input`, `audio_input`, `tool_result`, `interrupt`, `ping` → `text_delta`, `audio_delta`, `tool_call`, `transcript`, `turn_complete`, etc.) |
| **Admin portal** | None | Full admin: user mgmt, catalog CRUD, session force-close, audit log |
| **Provider catalog** | Hardcoded | `llm_providers` + `llm_provider_apis` + `llm_models` tables in PostgreSQL |
| **Usage tracking** | Basic (input/output tokens, turns, tool_calls) | Tokens + audio seconds, flushed to Redis every 30s, final persist at teardown |
| **Multi-project** | Single project per tenant | Multiple projects per tenant, each independently configured |
| **Cost estimation** | None | USD cost tracking per session/project/account |
| **Inactivity timeout** | 300s hardcoded `session_ttl_seconds` | Configurable, close code 4005 + `session_timeout` error event |
| **Horizontal scaling** | NGINX sticky session noted but sessions in-process | Fully stateless gateway nodes — all state in Redis |
| **Content privacy** | MongoDB stored session metadata (not raw messages) | PostgreSQL stores **zero** conversation content — absolute constraint C-04 |

---

### Key Architectural Patterns — New System

1. **Adapter Pattern** (`ILLMSessionAdapter`) — all 5 providers behind one interface
2. **Factory Method** (`LLMAdapterFactory`) — maps `adapter_key` string from DB catalog to concrete class. Zero-change extensibility (constraint C-06).
3. **Template Method** (`WebSocketBridgeAdapter`, `HttpOrchestratorAdapter` base classes) — common lifecycle logic, provider-specific hook methods (`BuildProviderUri`, `TranslateToProvider`, `TranslateFromProvider`, etc.)
4. **Session State Machine** — strict: `VALIDATING → INITIALIZING → ACTIVE ↔ RESPONDING → CLOSING → CLOSED` (and `REJECTED`, `FAILED` terminals)
5. **Observer/Pipeline** — `IAsyncEnumerable<LLMEvent>` channel decouples adapter event production from Protocol Handler serialization
6. **Repository Pattern** — all DB access abstracted; tenant predicate enforced at repo layer (IDOR prevention)
7. **CQRS/MediatR** — REST API service handler layer

---

### The Core Problem Being Solved

The previous version was a **proof-of-concept** locked to Gemini Live. The new system is a **production-grade multi-tenant gateway** where:
- A Platform User creates a project, picks any supported provider/API/model from a catalog
- Their downstream app connects via a **standardized WebSocket protocol** — the same protocol regardless of whether Gemini, OpenAI Realtime, Chat Completions, etc. is underneath
- Switching LLM providers = 2 clicks in the dashboard, zero code changes in the client app
- The platform organization owns all LLM credentials — tenants never touch them

---

I've now read everything thoroughly. What would you like to do next? For example:
- Design the new repositories for this platform
- Create a new GitHub repository/organization structure for ChatN
- Build out specific components (e.g., the adapter layer, the WebSocket gateway, the Next.js dashboard)
- Scaffold the ASP.NET Core solution structure
- Something else entirely

## @ranga1729

```
### How RAG works ?
Vector DB converts words into numbers. Ex: "Apple" : "137"
So the sentences will be a list of numbers(a vector). 
Similar concepts are placed close to each other in a mathematical "map".
The database doesn't look for exact word matches, it calculates the distance between vectors as well.

the ingestion workflow is, 
1. Chunking: The document gets broken into smaller pieces. 
2. Vectorization: Each chunk is send to an embedding model(like OpenAI's `text-embedding-3`) which returns a vector.
3. Storage: Store a record in the vector database that typically contains,
	- The vector - used for search itself
	- Metadata/Raw text - the actual text 
	The vector will be used to search, but the model read the actual data retrieved by the model.
Vector DB is queried in every single user interaction. 
Vectors are not injected into the LLM at the beginning like a permanent memory. Instead, they are queried in every single user interaction.
Ex:
	1. **User asks:** "What is the return policy?"
	2. **Query vectorization:** App must turn the query into a vector using the same model used for documents. 
	3. **Vector Search:** App asks the vector DB, "Find the 3 chunks of text whose vectors are closest this question"
	4. **The "Injection"(Prompt Augmentation):** App takes the original text + those 3 chunks and stuffs them into the prompt sent to the LLM. 
	**Example of what the LLM actually sees:** _"You are a helpful assistant. Use the following context to answer the question: `[Retrieved Text Chunk 1], Retrieved Text Chunk 2. Question: What is the return policy?"`

### Findings, 
Most of the vector databases support tenant isolation via different methods. 
1. Namespace isolation in pinecone
Pinecone uses namespaces for tenant isolation. You partition the vectors within a single index into different namespaces. when perform a query, you specify the namespace parameter. 
Benefit: Logically isolated. Highly efficient (since the search area is small). 
2. Azure AI Search
	Handles in two ways, 
	- Filter based: store `TenantId` or `UserId` as a metadata field alongside your vector embeddings. When the user sends a chat message, websocket gateway service appends a filter to the search query: `$filter=TenantId eq 'user_123'`. The search engine ignore all documents that don't match that ID before performing the vector similarity search. 
	- Index-per-Tenant: Create a separate physical index for every tenant. The highest level of isolation and allows for different schemas per tenant. high cost. 

Different providers, different APIs support RAG in different different ways. 

| API Name               | RAG Support Level | How it works ?                                                                                                                                                                        |
| ---------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OpenAI response API    | Native(Hosted)    | Has a built in `file_search`, `web_search` tools. If you upload files to an OpenAI-managed vector store, and then API handles retrieval automatically.                                |
| Google generateContent | Native/Hybrid     | Support `Grounding with Google search` or `Grounding with your data` (via Vertex AI Search). You can link it directly to a data store without manual chunking.                        |
| OpenAI Realtime API    | Via tooling       | No native RAG option. Must define a function call/tool (ex: `search_knowledge_base`). The model calls your function, and then your app query your vector DB and send the result back. |
| Gemini Live API        | Via tooling       | Similar to OpenAI Realtime API.                                                                                                                                                       |
| OpenAI chat completion | Manual Only       | Strictly stateless. Has to implement the entire RAG pipeline and inject the text into the prompt.                                                                                     |
### Recommended approach: 
Use Azure AI Search as the vector database. 
Let users upload documents for RAG.

**Vector DB setup:** Azure AI search with Metadata Filtering
To achieve tenant isolation in the vector DB, use Metadata filtering. This approach is more cost effective and easier to manage than creating separate indices. 

Every document/chunk in the vector DB index must have a field named `TenantId` or `UserId`(marked as `Filterable` and `Retrievable`). 
Backend must never perform a raw vector search, every search request sent to Azure must include an `OData` filter, enduring the model never sees data from other tenants. 
`$filter = TenantId eq 'tenant_abc_123'`

Since you we want to support multiple LLM APIs, we should define a Universal Search Tool that our backend exposes to every LLM. 

Workflow:
1. Define a function called `search_internal_knowledge(query: string)`
2. When the user asks a question, the LLM determines it needs more info. It sends a tool call request back to the backend. 
3. The backend retrieves the query, identifies the tenant Id from the users active session and queries Azure AI Search with the mandatory filter.
4. The backend sends the filtered text back to the LLM, which then answers the user. 

Match with the Current architecture: 
- during the adapter connect time, load the `"search_internal_knowledge"` tool to the tool list of the configuration. 
- Since the websocket gateway service already handle the "Provider <--> Client loop", it can intercept a tool call from the LLM. Perform the RAG search, and return the result to the model before the end client sees it.
- Use the `project_id` instead of `tenant_id` or `user_id`. Because in this application, tenants are not the smallest unit to be isolated, it's the project.

- Add a new endpoint to upload files to a project. 
- Add a new server-side event called the `"tool_call_intercept"`, which tells the gateway to perform a local lookup instead of forwarding to the project's web hook. 
****
```
````
# MCP Plan
### What is MCP ?
MCP is a protocol/standard designed to let AI models interact with external tools, data sources and services in a structured, consistent way. 
MCP defines,
- How tools/services expose capabilities
- How models request actions
- How data flows between them
These are heavily inspired by ideas from APISs, JOSN-RPC and function calling, but more standardized and model-friendly.

#### Key entities in a MCP workflow,
1. **Host**(or client)
	The application that runs the AI model. 
	Ex: chat app, IDE plugin, Browser assistant
	Responsibilities,
	- Sends user input to the model
	- Manages connections to MCP servers
	- Execute tool calls returned by the model
2. Model
	The AI itself.
	Responsibilities,
	- Understand user intent.
	- Decide when to call a tool
	- Generate structured request (not raw API calls)
3. MCP server
	This is where tools and data sources are exposed. 
	Ex: File system server, Database server, GitHub integration server
	Responsibilities,
	- Advertise available tools
	- Define schemas(input/output)
	- Execute requested actions
4. Tool
	There are the actual operations the model can invoke. 
	Ex: `read_file(path)`, `search(query)`, `execute_sql(query)`
	Each tool includes,
	- Name 
	- Description
	- Input schema
	- Output schema
5. Resources
	These represent data that tools can access.
	Ex: Files, Documents, APIs, Databases
6. Transport Layer
	Defines how communication happened between components. 
	Common methods,
	- WebSocket
	- HTTP
	- Local IPC (internally defined tools/functions)

#### MCP Workflow,
1. Host connects to MCP servers and the servers expose available tools(via schemas).
2. Model sees a user query matching to a tools defined by a MCP server. 
3. Model generates structured request.
```json
{
	"tool": "read_file",
	"arguments": {
		"path": "report.txt"
	}
}
```
4. Host sends a request to MCP server and then server executes the tool and returns result. 
5. Model receives tool output and continue reasoning. May call more tools or generate final answer.
6. Model return response to the user. 

#### How an platform discover tools defined in a MCP server ?
1. **Initialize**: Using the transport layer that server prefers, the platform opens a connection and send an initialization request. 
Ex:
```json
{
	"type":"initialize",
	"client": {
		"name": "your-platform",
		version: "1.0"
	}
}
```
2. **Check capabilities**: The MCP server replies with metadata about what it supports. 
Ex:
```json
{
	"capabilities": {
		"tools": true,
		"resources": true
	}
}
```
"I support tool discovery, go ahead and ask".
3. **Request tools**: Now your platform can explicitly ask for the tools list.
Ex:
```json
{
	"type": "tools/list"
}
```
4. Server returns tools definitions in a structured list like,
```json
{
  "tools": [
    {
      "name": "read_file",
      "description": "Read contents of a file",
      "inputSchema": {
        "type": "object",
        "properties": {
          "path": { "type": "string" }
        },
        "required": ["path"]
      }
    },
    {
      "name": "search",
      "description": "Search documents",
      "inputSchema": {
        "type": "object",
        "properties": {
          "query": { "type": "string" }
        }
      }
    }
  ]
}
```
#### What the platform should do ?
1. Store tool registry
	Maintain an internal structure like,
	`server_id -> tools[]`
2. Normalize Schemas
	Different slightly JSON schemas styles like adding custom fields. You need to standardize them internally. 
3. Make tools discoverable to models
	Convert tools into a format your model understands,
	- OpenAI --> function calling format
	- Google --> tool schema
	- Others --> adapters
4. Handle dynamic Updates
	Good MCP servers can change tools at runtime. 
	So,
	- Re-fetch tools periodically OR
	- Listen for updates(if supported)
#### Then how to call a tool in the MCP server ?
You don't call any function in an outside server. 
You send the tool invocation request over the same MCP connection, and the server executes it. 

MCP uses request-response pattern. 
so you send this message to the MCP server that owns the tool.
```json
{  
	"id": "req-123",  
	"type": "tools/call",  
	"tool": "read_file",  
	"arguments": {  
		"path": "/home/data/report.txt"  
	}  
}
```
Key things,
- `id` : Using this value you can match responses
- `tool`: exact name from discovery
- `arguments`: must match schema

MCP server will then execute the function, grab the result, and send it back to you. 
```json
{  
	"id": "req-123",  
	"result": {  
		"content": "File contents here..."  
	}  
}
```
OR if error,
```json
{
  "id": "req-123",
  "error": {
    "message": "File not found"
  }
}
```

#### Best practice:
Don't assume "All servers will behave nicely". 
You will face,
- missing schemas
- Invalid JSON
- Poor descriptions
- Conflicting tool names

So add a validation layer to,
- Validate schemas
- Reject broken tools
- Log issues

Server will crash, hang or be slow. So we need,
- Timeouts
- Retries
- Fallback handling

Try to build a MCP orchestrator like,
![[Pasted image 20260429192654.png]]

#### What should be improved in the tools table so it can work as a Tool Registry that can facilitate MCP tools.
Current table,
![[Pasted image 20260429193609.png]]
Missing columns,
- Where the tool actually lives(endpoint)
- How to call it(method, protocol)
- Output structure
- Versioning
- Auth details
- Execution constraints
- Status/health

New table,
```sql
tools (
  id UUID PK,
  project_id UUID FK,

  -- Identity
  name TEXT,
  display_name TEXT,
  description TEXT,

  -- Schema
  input_schema JSONB,     -- was parameters
  output_schema JSONB,    -- YOU NEED THIS

  -- Execution
  endpoint_url TEXT,      -- where to send the request
  http_method TEXT,       -- POST / GET (usually POST)
  protocol TEXT,          -- http / websocket / mcp

  -- Auth
  auth_type TEXT,         -- none / api_key / oauth
  auth_config JSONB,      -- encrypted or referenced

  -- Routing / Ownership
  user_id UUID,           -- who owns this tool
  app_id UUID,            -- which app/service handles it

  -- Versioning
  version TEXT,
  is_latest BOOLEAN,

  -- Status
  is_active BOOLEAN,
  is_deprecated BOOLEAN,
  health_status TEXT,     -- healthy / failing / unknown

  -- Execution controls
  timeout_ms INT,
  retry_count INT,

  -- Observability
  last_called_at TIMESTAMPTZ,
  last_success_at TIMESTAMPTZ,
  failure_count INT,

  created_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ
);
```

More improvements, 
Break the tools table into three tables,
1. `tools` : what the tool is.
2. `tool_endpoints` : execution layer
```sql
tool_endpoints (
  id UUID PK,
  tool_id UUID FK,

  endpoint_url TEXT,
  protocol TEXT,
  auth_config JSONB,

  is_active BOOLEAN,
  health_status TEXT
);
```
	Becuase one tool can have multiple endpoints.
	Use these additional endpoints in the fallback mechanism. 
3. `tool_calls` : Execution history
```sql
tool_calls (
  id UUID PK,
  tool_id UUID,
  request JSONB,
  response JSONB,

  status TEXT,
  latency_ms INT,
  error TEXT,

  created_at TIMESTAMPTZ
);
```
This architecture handles,
- Debugging
- Analytics
- Retires
````
In addition to the given SRS document, I'm planning to add RAG, MCP servers and System prompt versioning to the system. here is my study. I attached markdown content on how RAG and MCP is going to be in the system. and my requirement for the system prompt versioning is this, in the current system we are allowing one system prompt per project. instead, I thoug we can let users maintain several(maximum 2 for now) system prompts per project, but only one system prompt can be active at any given time(but may be in the future we will let the user to do A/B testing. half of the session will get a one system prompt and other get another prompt. then by looking at stats, they can decide what prompt to keep depend on how the users interacted with the new behavior. but this is a future plan so my current implementation must save space for future expansions without lots of changes). for now consider these changes. let me know if there are any mistakes in my above chnages. 

## Copilot

I've read both documents carefully. Here's a systematic review organized by feature area.

---

## RAG — Issues Found

### 1. Factual Error: Vectors Are Not Scalars
> *"Vector DB converts words into numbers. Ex: 'Apple': '137'"*

This is incorrect and will cause confusion later. A vector is a **high-dimensional array of floats**, not a single number.

`"Apple"` → `[0.23, -0.15, 0.89, 0.41, ...]` (typically 1536 dimensions with `text-embedding-3-small`)

The number `137` implies a scalar — that's how a traditional keyword index works, not a vector index. Distance calculations (`cosine similarity`, `dot product`) only make sense on arrays, not scalars.

---

### 2. Architectural Conflict: Internal vs. External Tool Calls
The current SRS (UC-12) defines tool calls as:
> *LLM invokes tool → Gateway forwards `tool_call` event to End Client → End Client executes it → returns `tool_result`*

Your RAG proposal introduces a **second class of tool** — one that is intercepted and resolved by the gateway itself, never reaching the End Client.

You've named the concept `"tool_call_intercept"` as a server-side event, but this is actually an **internal routing decision**, not a new WebSocket protocol event sent to the client. You need to define a clear taxonomy:

| Tool Type | Who Executes | Client Sees It? |
|---|---|---|
| **External tools** | End Client's application | Yes — full `tool_call` / `tool_result` cycle |
| **Gateway-intercepted tools** | WebSocket Gateway Service | No — resolved internally before client sees it |
| **MCP tools** (from your other doc) | MCP Server via gateway | No — also intercepted |

This taxonomy must be reflected in the project configuration (tools table needs a `routing` or `execution_mode` field — `"client"` vs `"gateway"`) and in the adapter layer's tool-call handling logic.

---

### 3. Constraint C-04 Risk: `tool_calls.request` and `tool_calls.response`
Your proposed `tool_calls` table:
```sql
tool_calls (
  request  JSONB,
  response JSONB,
  ...
)
```

The `request` payload for `search_internal_knowledge` will contain the user's query string — e.g., `{"query": "What is the return policy?"}`. That is **conversation content** derived directly from the user's message. Storing it in PostgreSQL violates:

> **C-04**: *"The Platform MUST NOT persist conversation content to the primary database at any point, under any circumstances."*

Fix: Store only **metadata** — tool name, latency, status, error code, token cost. Never store the arguments or response content. For debugging, route to a separate observability system (e.g., Application Insights, Datadog) that is outside the primary database scope.

---

### 4. Constraint C-02 Risk: `auth_config JSONB` in Tools Table
```sql
auth_config JSONB,  -- encrypted or referenced
```

If `auth_config` stores API keys for external tools or MCP servers (even encrypted), this potentially violates:

> **C-02**: *"LLM provider credentials MUST be stored exclusively in a cloud Secrets Manager. They MUST NOT appear in the database..."*

The SRS scopes C-02 specifically to LLM provider credentials, but the spirit of the constraint should apply to any third-party service credential. The cleaner design: `auth_config` stores only a **Key Vault secret reference name**, not the actual credential. The gateway resolves the real value at runtime from Key Vault.

---

### 5. Missing: Conditional Tool Injection
You state:
> *"During adapter connect time, load the `search_internal_knowledge` tool to the tool list"*

But this should only happen **if the project has at least one document uploaded and RAG is enabled**. Injecting a search tool into sessions for projects with no documents is wasteful and will cause the LLM to call a tool that returns empty results, which degrades response quality. 

The `ProjectConfig` needs a `rag_enabled: bool` flag, and the adapter's tool injection logic must check it.

---

### 6. Missing: Embedding Model Not Specified
You mention chunking and vectorization but never specify which embedding model is used. This matters because:
- The query vectorization at search time **must use the same model** as the ingestion-time vectorization. A mismatch produces incorrect similarity scores.
- The model should be stored on the project's RAG config (or at minimum be a platform-wide constant) so it can never drift between ingestion and retrieval.

Recommended: Store `embedding_model` in the project's RAG configuration. Default to `text-embedding-3-small`.

---

### 7. Missing: Document Lifecycle Management
You mention adding an upload endpoint but don't address:
- **Deletion**: When a project is deleted, all its vectors in Azure AI Search must be purged (filter by `project_id` and bulk-delete).
- **Document listing/deletion**: Users need to see what documents are uploaded and be able to remove individual ones.
- **File storage**: Where does the raw file live before chunking? (Azure Blob Storage is the natural pair to Azure AI Search.) The platform needs to store a reference to the blob, not the file content itself.
- **Supported file types**: PDF? DOCX? TXT? Markdown? This determines the parser you need before chunking.

---

### 8. Latency Warning for Voice Adapters
For WS Bridge adapters (OpenAI Realtime, Gemini Live), the RAG tool call adds a round-trip:

`LLM → tool_call event → Gateway intercepts → Azure AI Search query → result back to LLM → continues audio response`

This introduces **200–800ms of additional latency** in the middle of an audio turn. This is not a blocker, but it should be explicitly acknowledged in the SRS, and a `rag_timeout_ms` config per project is advisable so slow searches don't stall the voice session.

---

## MCP — Issues Found

### 9. Role Ambiguity: Who Is the MCP Host?
Your MCP document describes the "Host" generically. In your architecture, this must be explicit: **the WebSocket Gateway Service is the MCP Host**. It connects to the tenant's registered MCP servers, discovers tools, and proxies tool invocations — the End Client never communicates directly with MCP servers.

This should be stated clearly because it has security implications (the gateway is making outbound connections to user-specified URLs).

---

### 10. Critical Security Gap: SSRF Risk
MCP servers are defined by tenant-supplied URLs (`endpoint_url` in your tools table). The gateway will make outbound HTTP/WebSocket connections to these URLs. This is a **Server-Side Request Forgery (SSRF)** attack vector.

A malicious tenant could supply:
- `http://169.254.169.254/latest/meta-data/` (AWS metadata endpoint)
- `http://localhost:5432` (internal PostgreSQL)
- `http://10.0.0.1/admin` (internal network)

You need an explicit constraint added to the SRS:
- All MCP server URLs must be validated against an allowlist or blocklist of private IP ranges at registration time
- Only `https://` scheme should be permitted for production MCP servers
- Requests to RFC1918 address ranges must be rejected

---

### 11. MCP Connection Lifecycle Not Defined
When is the MCP connection opened? When is it closed? You don't address this.

Options and tradeoffs:

| Approach | Pro | Con |
|---|---|---|
| Per-session connection | Simple, isolated | Expensive — discovery latency on every session open |
| Per-project connection pool | Efficient, shared | Complex; stale tool lists if MCP server changes tools |
| At project config load | Fastest | Tools may be stale; needs refresh mechanism |

Recommended: **Per-project connection pool** with a **periodic re-discovery** (e.g., every 5 minutes, or on config change). The discovered tool list should be cached in Redis alongside the project config so gateway nodes share it.

---

### 12. `tool_calls.request`/`response` Still Violates C-04 for MCP
Same issue as point 3. MCP tool call arguments (e.g., a `search` tool called with the user's query) are conversation-derived content. The same C-04 constraint applies.

---

### 13. Conflicting Tool Names — Resolution Not Defined
Your best-practices section correctly notes:
> *"Conflicting tool names"*

But you don't define how to resolve them. If a tenant registers two MCP servers and both expose a tool called `search`, what happens? The LLM receives two tools with the same name, which breaks function calling on all providers.

Define a **namespacing strategy**: prefix tool names with the MCP server's registered ID, e.g., `mcp_server_a__search`, `mcp_server_b__search`. This must be applied consistently at discovery time and reversed when routing the actual tool invocation.

---

## System Prompt Versioning — Issues Found

### 14. Max=2 Should Be a Plan-Level Rule, Not a Schema Constraint
If you hardcode `max_prompts = 2` in the schema or application code, you'll need code changes to raise it later (or when doing A/B testing with >2 variants). Instead:

- The **schema** should support N prompts per project with no hard limit at the DB layer
- The **enforcement** of `max = 2` should come from a plan-level config (e.g., `plan_limits.max_system_prompts = 2` for free/starter, higher for pro/enterprise)
- The **application layer** (REST API) checks `count(prompts for project) < plan_limit` before allowing creation

This is already consistent with how the rest of your plan limits work (`max_concurrent_sessions`, etc.).

---

### 15. Missing: Database-Level Uniqueness for Active Prompt
The constraint "only one active prompt per project at a time" must be enforced at the **database level**, not just the application layer. Application-level enforcement alone can fail under race conditions (two concurrent API requests both activating different prompts).

```sql
-- PostgreSQL partial unique index
CREATE UNIQUE INDEX uq_one_active_prompt_per_project
  ON system_prompts (project_id)
  WHERE is_active = true;
```

This makes it physically impossible for two prompts to be active simultaneously for the same project, even under concurrent writes.

---

### 16. Missing: Session Record Must Store `system_prompt_id`
For the future A/B testing feature to work, you need to know **which prompt was active for each session**. Without this, you cannot correlate session stats (user satisfaction, turn count, errors) with prompt variants.

The `sessions` table (currently tracking tokens, duration, etc.) needs a `system_prompt_id UUID` column added now. Even if A/B testing is not implemented yet, the foreign key can be populated from day one at zero cost, and you'll have historical data ready when the feature ships.

---

### 17. Prompt Immutability for Result Integrity
If a user can **edit** an existing system prompt record in place, then any historical sessions that used `system_prompt_id = X` are now associated with a different prompt text than what was actually used. This breaks A/B testing analytics retroactively.

**System prompts should be immutable after creation.** Editing a prompt = creating a new version record. You can soft-archive old ones (`is_archived = true`). Add a `version_number INT` and optionally a `parent_prompt_id UUID` to form a version chain per project.

---

### 18. `traffic_weight` Column Should Be Added Now
Your A/B testing plan will need a `traffic_weight` column (e.g., `INT` 0–100, all active prompts for a project must sum to 100). Adding it now with a default value of `100` costs nothing and avoids a migration later. When A/B testing ships, the logic reads these weights — the column is already there.

---

## Summary Table

| # | Area | Severity | Issue |
|---|---|---|---|
| 1 | RAG | Medium | Vectors are arrays of floats, not scalars |
| 2 | RAG | High | No taxonomy of tool types (external vs gateway-intercepted vs MCP) |
| 3 | RAG | **Critical** | `tool_calls.request/response` JSONB violates C-04 |
| 4 | MCP | **Critical** | `auth_config` in DB potentially violates C-02 |
| 5 | RAG | Medium | `search_internal_knowledge` must be conditionally injected |
| 6 | RAG | Medium | Embedding model not specified; must match between ingestion and query |
| 7 | RAG | Medium | Document lifecycle (delete, list, blob storage) not addressed |
| 8 | RAG | Low | Latency impact on voice adapters not acknowledged |
| 9 | MCP | Medium | Gateway = MCP Host role not explicitly stated |
| 10 | MCP | **Critical** | SSRF risk from tenant-supplied MCP server URLs |
| 11 | MCP | High | MCP connection lifecycle/pooling not defined |
| 12 | MCP | **Critical** | Same C-04 violation as #3 for MCP tool calls |
| 13 | MCP | High | Conflicting tool names across MCP servers — no resolution strategy |
| 14 | Prompt | Medium | Max=2 hardcoded instead of plan-level config |
| 15 | Prompt | High | `is_active` uniqueness not DB-enforced (race condition risk) |
| 16 | Prompt | High | `system_prompt_id` not added to sessions table |
| 17 | Prompt | High | Prompts are mutable — breaks future A/B analytics |
| 18 | Prompt | Low | `traffic_weight` column absent — migration cost later |

The three **Critical** issues (#3, #4, #10, #12) should be resolved before any implementation starts, as they involve security constraints and data compliance that are load-bearing for the entire architecture.

## @ranga1729

Ok here. issue 1 - I acknoeledged the mistakes and thanks for correcting me. Issue 2 - here is the way I want to correct this. for the RAG, I was thinking to inspect the tool call request coming form the LLM provider to my backend. if the tool name is "search_internal_knowledge", i will not forward this to the user, instead I will intercept the tool call request, do the vector DB querying, and send the result back to the LLM. since this is the only tool I'm going to internally define, I think intercepting this particular tool is ok(but if you have a better solution, suggest me. but at the end what should be done is this. this "search_internal_knowledge" should be resolved internally without formwarding it to the user). and remove that "tool_call_intercept" event. instead I think it's better to have a server side event called "access_internal_knowledge", so if we maintain a log of server-side event, we can log that, and may be we can send it to the user so user know now their query is acessing the internal knowledge. Issue 3 - Yes I accept tha fact that we must remove this tool_call.request and tool_call.response columns from tables since they store user's messages and responses. and I need a better way to store tools in the backend. these are my concerns, since now I'm considering to allow users to add mcp servers, there should be a table to store them that linked with a project_id. also there should be a table which includes all the tools that linked with the project_id as well. but I heared that some mcp server have multiple endpoint urls for the same tool, so there should be a way to handle that as well. because we can use it as a fallback mechanism. if the first endpoint failed/timeout or error occured, we must try the second url. and if a particuar tool is coming from a mcp server, there should be a link between them a well. Issue 4 - yes. store the key vault secret reference name. Issue 5 - Completley agree with you. we must add a boolean called rag_enabled into the Projectconfig. If users has upload any document, that's the time we must enable it. so the adapter won't have to use that tool in unnecessary times. I need a proper mechanism to handle that in the architecture. also, since we allow users to uplaod documents, I was thinking to use Azure Blob storage. so we can use the Azure AI search(as the vector DB) with Blob service. may be We will need another table to track the links of those docuemnts that our user uploaded for a project. Issue 6 & Issue 7 - yes. I was added later. I need to mention it in the SRS. I guess the Azure AI search handle the entire pipeline of this vectorization including the embedding of those documents. and if a user delete the project, they must be deleted as well. from the block storage and the azure ai search as well. Supported file types will be PDF, DOCX, TEXT, Markdown. Issue 8 - Yes it should be acknowledged at the SRS. also if the user has enabled the RAG, we will communicate it in the front-end. and I think having that "rag_timeout_ms" at the project level is also a better idea. Issue 9 - acknoelwdged. Issue 10 - acknowledged. this must be mentioned in the SRS clearly. Issue 11 - I was thinking to give platform users a button so they can manually trigger the re-discovery of tools from connection mcp servers. in-addition, I think it's good to have a automatic re-discover mechanism at the prject level using the per-project connection pool option you mentioned. Issue 12 - yes. agree. must be removed. already mentioned above. Issue 13 - yes. this must be applied too. Issue 14 - yes. butI have added any table including the plan details(like free, pro, enterprises). and I'm agree on the fact that the prompr version count must be depend on the subscription plan. So we will need a new table called "subscription_plans". can you please suggest me a pricing strategy for this platforms ? Issue 15 - yes, since we allow multiple prompts, i think we might need to have a separate table for system_prompts. and that table has a FK relation to a project_config. and the table has a attribute to indicate which prompt must be active at the time like "is_active" or something. so when we imporing the project config, we can grab all the prompts including the is_active being true on the active prompt. Issue 16 - agree. Issue 17 - when I'm saying prompt_versioning, I mean I'm allowing the user to have multiple system prompts. not to track versions of the same prompt. since the users is allowed to have multiple prompts, we must keep track what is the active one. and my future plan was to let the users do A/B testing while keeping musltiple propmpts active. example: two system prompts are active. both of them are active for the A/B testing. half of the new webscoket connections will load the "prompt_a" and the other half will load "prompt_b". and we will let users keep it for a while. then we can show some matrices of hwo they performed. but this feature is not yet confirmed and have no idea what matrics we should show as the A/B testing reults. since that, keep this away from the current implementation. for now lets allow users to have x number of system propmpt(x must be depend on the payment plan) and one of them must be active at a time. platform users can change the active propmpt(all prompts must have a label to easily identify). then the next websocket open will load this new activated prompt. Issue 18 - no idea what's this "traffic_weight" means. but don't apply changes on that A/B testing feature. we will discuss it later. 

## Copilot

Good clarifications. Let me go through each point, flag any remaining problems with your responses, and fill in the gaps where you asked for guidance.

---

## Issue 2 — Tool Interception ✅ Solid, One Refinement

Your approach is correct. Inspect the tool name at the Protocol Handler layer — if it matches `search_internal_knowledge`, intercept it, resolve it internally, and return the result to the LLM without the End Client ever seeing it.

The `rag_search` event (renaming your `access_internal_knowledge`) is a good idea. One thing to clarify: this is a **Gateway → Client informational event**, exactly like `transcript`. It requires no response from the client.

Add it to the Gateway → Client message table:

| Type | Fields | Description |
|---|---|---|
| `rag_search` | `query: string`, `chunks_retrieved: int` | Sent when the gateway intercepts a `search_internal_knowledge` call and retrieves context. Informs the client that internal knowledge was accessed. |

Note: do **not** include the actual retrieved text in this event — that would be surfacing internal document content to the client unnecessarily. Just the query string and how many chunks were found is sufficient for UI purposes (e.g., showing a "🔍 Searching knowledge base..." indicator).

---

## Issue 3 — Tool Storage Schema

Your requirements: MCP servers table, tools table, multi-endpoint fallback, link between tools and their MCP server. Here is a clean schema that satisfies all of this plus the C-04 and C-02 constraints.

### `mcp_servers`
One row per MCP server registered to a project.

```sql
mcp_servers (
  id                  UUID PRIMARY KEY,
  project_id          UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- Identity
  name                TEXT NOT NULL,
  description         TEXT,

  -- Connection
  base_url            TEXT NOT NULL,           -- validated: must be HTTPS, non-RFC1918
  protocol            TEXT NOT NULL DEFAULT 'mcp',   -- 'mcp' | 'http'

  -- Auth (Key Vault reference only — never raw credentials)
  auth_type           TEXT NOT NULL DEFAULT 'none',  -- 'none' | 'api_key' | 'bearer'
  auth_secret_ref     TEXT,                    -- Key Vault secret name, e.g. "mcp-crm-apikey"

  -- Discovery state
  last_discovered_at  TIMESTAMPTZ,
  tool_count          INT DEFAULT 0,           -- denormalized count, updated after discovery
  connection_status   TEXT DEFAULT 'pending',  -- 'connected' | 'disconnected' | 'error'
  last_error          TEXT,

  -- Lifecycle
  is_active           BOOLEAN NOT NULL DEFAULT true,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### `tools`
One row per tool available to a project. Covers all three sources: manually defined (by the platform user), MCP-discovered, and the one platform-builtin (`search_internal_knowledge`).

```sql
tools (
  id                  UUID PRIMARY KEY,
  project_id          UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  mcp_server_id       UUID REFERENCES mcp_servers(id) ON DELETE CASCADE,
                      -- NULL for manual and builtin tools

  -- Source classification
  source              TEXT NOT NULL,           -- 'manual' | 'mcp' | 'builtin'

  -- Identity
  -- For MCP tools this MUST be namespaced: '{mcp_server_id}__{original_name}'
  -- to prevent collisions across MCP servers
  name                TEXT NOT NULL,
  display_name        TEXT,
  description         TEXT NOT NULL,

  -- Schema
  input_schema        JSONB NOT NULL,
  output_schema       JSONB,

  -- Execution controls
  timeout_ms          INT NOT NULL DEFAULT 30000,
  retry_count         INT NOT NULL DEFAULT 1,

  -- Lifecycle
  is_active           BOOLEAN NOT NULL DEFAULT true,
  is_deprecated       BOOLEAN NOT NULL DEFAULT false,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- A project cannot have two tools with the same name
  UNIQUE (project_id, name)
);
```

### `tool_endpoints`
One row per endpoint for a tool. Enables the multi-URL fallback you want.

```sql
tool_endpoints (
  id                  UUID PRIMARY KEY,
  tool_id             UUID NOT NULL REFERENCES tools(id) ON DELETE CASCADE,

  -- Ordering — priority 1 is tried first, 2 is first fallback, etc.
  priority            INT NOT NULL DEFAULT 1,

  endpoint_url        TEXT NOT NULL,           -- validated: HTTPS, non-RFC1918
  http_method         TEXT NOT NULL DEFAULT 'POST',

  -- Auth override at endpoint level (Key Vault reference only)
  -- If NULL, inherits from mcp_servers.auth_secret_ref
  auth_secret_ref     TEXT,

  -- Health
  is_active           BOOLEAN NOT NULL DEFAULT true,
  health_status       TEXT DEFAULT 'unknown',  -- 'healthy' | 'failing' | 'unknown'
  last_checked_at     TIMESTAMPTZ,

  UNIQUE (tool_id, priority)
);
```

### `tool_executions`
Observability only. **No request payload, no response payload** — C-04 compliant.

```sql
tool_executions (
  id                  UUID PRIMARY KEY,
  tool_id             UUID NOT NULL REFERENCES tools(id),
  endpoint_id         UUID REFERENCES tool_endpoints(id),

  -- session_id is stored for correlation but there is intentionally no FK
  -- to the sessions table — sessions are ephemeral and not conversation-content
  session_id          UUID NOT NULL,

  -- Outcome
  status              TEXT NOT NULL,           -- 'success' | 'error' | 'timeout'
  latency_ms          INT,
  error_code          TEXT,                    -- e.g. 'timeout', 'http_503', 'schema_invalid'
  attempted_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**What is intentionally absent**: `request JSONB`, `response JSONB`, `arguments`, any field that could contain user message content.

---

## Issue 5 — RAG Architecture

### `rag_enabled` Flag and When to Set It
Correct approach. Add `rag_enabled BOOLEAN DEFAULT false` to `project_configs`. The REST API automatically sets it to `true` when the first document is successfully indexed, and back to `false` if all documents are deleted. The adapter reads this flag at session init — if `false`, `search_internal_knowledge` is never injected into the tool list.

### Document Tracking Table

```sql
project_documents (
  id                  UUID PRIMARY KEY,
  project_id          UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- File identity
  file_name           TEXT NOT NULL,           -- original uploaded name
  file_type           TEXT NOT NULL,           -- 'pdf' | 'docx' | 'txt' | 'md'
  file_size_bytes     BIGINT NOT NULL,

  -- Azure Blob Storage reference (raw file)
  blob_container      TEXT NOT NULL,
  blob_name           TEXT NOT NULL,           -- unique path within container
  blob_url            TEXT NOT NULL,

  -- Azure AI Search indexing state
  index_status        TEXT NOT NULL DEFAULT 'pending',
                      -- 'pending' | 'indexing' | 'indexed' | 'failed'
  chunk_count         INT,                     -- populated after indexing
  embedding_model     TEXT NOT NULL DEFAULT 'text-embedding-3-small',
  index_error         TEXT,                    -- last error message if failed

  -- Lifecycle
  uploaded_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  indexed_at          TIMESTAMPTZ,
  deleted_at          TIMESTAMPTZ              -- soft delete; triggers async cleanup
);
```

### One Problem with Your Azure AI Search Assumption

You said:
> *"Azure AI Search handles the entire pipeline of vectorization including the embedding"*

This is **partially true but depends on your configuration**. Azure AI Search has a feature called **integrated vectorization** (using an indexer + skillset with an `AzureOpenAIEmbeddingSkill`). When configured correctly, yes — it pulls from Blob Storage, chunks, embeds, and indexes automatically.

However: this requires your Blob Storage account and your Azure OpenAI embedding deployment to be wired up to the AI Search service via an indexer. If you use the simpler "push API" approach (your backend does the chunking, calls the embedding model, then pushes vectors directly to the index), you control the entire pipeline explicitly but must manage it yourself.

**Recommended**: Use integrated vectorization for simplicity. Your backend then only needs to:
1. Upload the raw file to Blob Storage
2. Trigger the indexer run (or wait for the schedule)
3. Poll/webhook for indexing completion and update `index_status`

The `embedding_model` field on `project_documents` still matters — store the model name used so future queries always use the same model.

### Cascade Delete on Project Deletion
Add this to the SRS explicitly. When a project is deleted:
1. Soft-delete all `project_documents` rows (set `deleted_at`)
2. Async job: delete blobs from Azure Blob Storage
3. Async job: delete all vectors from Azure AI Search index filtered by `project_id`
4. These cleanup jobs must be logged and retried on failure — partial cleanup is a data leak

---

## Issue 11 — MCP Discovery

Your approach: manual re-discovery button + automatic per-project pool with periodic refresh. Good combination.

One thing to specify clearly:

- **Automatic interval**: every **10 minutes** per connected MCP server (configurable at platform level, not per-project — tenants shouldn't control how often you poll their servers)
- **Manual trigger**: `POST /api/v1/projects/{id}/mcp-servers/{serverId}/rediscover` — rate-limited to once per 30 seconds per server to prevent abuse
- **Discovered tool list storage**: cache in Redis under `project:{project_id}:mcp:{server_id}:tools` with a TTL matching the auto-refresh interval. Database (`tools` table) is the source of truth; Redis is the hot cache used at session init time
- **On discovery**: compare new tool list against DB. New tools → insert. Removed tools → mark `is_deprecated = true` (never hard-delete; active sessions may be using them). Changed schemas → update and bump `updated_at`

---

## Issue 14 — Subscription Plans Table + Pricing Strategy

### Schema

```sql
subscription_plans (
  id                                    UUID PRIMARY KEY,
  name                                  TEXT NOT NULL UNIQUE,  -- 'free' | 'starter' | 'pro' | 'enterprise'
  display_name                          TEXT NOT NULL,
  price_monthly_usd                     DECIMAL(10,2),         -- NULL for enterprise (custom)
  price_yearly_usd                      DECIMAL(10,2),

  -- Project limits
  max_projects                          INT NOT NULL,
  max_api_keys_per_project              INT NOT NULL,
  max_system_prompts_per_project        INT NOT NULL,

  -- Session limits
  max_concurrent_sessions_per_project   INT NOT NULL,
  max_ws_connections_per_second_per_key INT NOT NULL,
  session_inactivity_timeout_seconds    INT NOT NULL,

  -- Token/usage quotas (monthly, across all projects on the account)
  max_monthly_input_tokens              BIGINT,                -- NULL = unlimited
  max_monthly_output_tokens             BIGINT,
  max_monthly_audio_seconds             INT,

  -- RAG limits
  rag_enabled                           BOOLEAN NOT NULL DEFAULT false,
  max_documents_per_project             INT,
  max_document_size_mb                  INT,
  max_total_storage_gb                  DECIMAL(6,2),

  -- MCP limits
  mcp_enabled                           BOOLEAN NOT NULL DEFAULT false,
  max_mcp_servers_per_project           INT,

  -- Feature flags
  voice_enabled                         BOOLEAN NOT NULL DEFAULT false,
  ephemeral_tokens_enabled              BOOLEAN NOT NULL DEFAULT false,
  webhooks_enabled                      BOOLEAN NOT NULL DEFAULT false,
  admin_api_access                      BOOLEAN NOT NULL DEFAULT false,

  -- Metadata
  is_public                             BOOLEAN NOT NULL DEFAULT true,  -- false = enterprise/hidden
  is_active                             BOOLEAN NOT NULL DEFAULT true,
  created_at                            TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                            TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### Suggested Pricing Tiers

This is based on the platform's cost structure — you're paying for LLM API calls on behalf of users, so your margins come from the markup on token usage plus the flat subscription fee.

| | **Free** | **Starter** | **Pro** | **Enterprise** |
|---|---|---|---|---|
| **Price** | $0 | $29/mo | $99/mo | Custom |
| **Projects** | 1 | 3 | 15 | Unlimited |
| **Concurrent sessions/project** | 2 | 10 | 50 | Custom |
| **System prompts/project** | 1 | 2 | 5 | Custom |
| **Monthly input tokens** | 500K | 5M | 25M | Custom |
| **Monthly output tokens** | 200K | 2M | 10M | Custom |
| **Monthly audio seconds** | — | 1,800 (30 min) | 18,000 (5 hrs) | Custom |
| **RAG** | ❌ | ✅ | ✅ | ✅ |
| **Documents/project** | — | 5 | 50 | Custom |
| **Max document size** | — | 10 MB | 25 MB | Custom |
| **Total storage** | — | 0.5 GB | 5 GB | Custom |
| **MCP servers** | ❌ | ❌ | ✅ (3/project) | ✅ |
| **Voice providers** | ❌ | ✅ | ✅ | ✅ |
| **Webhooks** | ❌ | ✅ | ✅ | ✅ |
| **Ephemeral tokens** | ❌ | ✅ | ✅ | ✅ |
| **API key count/project** | 2 | 5 | 20 | Unlimited |
| **Support** | Community | Email | Priority email | Dedicated SLA |

**Rationale:**
- Free tier exists to drive signups and developer testing. No voice, no RAG = zero LLM audio cost risk for you.
- Starter gates voice and RAG behind payment since those drive real LLM costs on your side.
- MCP is a Pro+ feature because it requires outbound connections from your infrastructure and carries the SSRF risk surface you need to manage carefully.
- Enterprise is custom to allow volume discount negotiations.

---

## Issue 15 — System Prompts Table

Your design is correct. Here is the full table:

```sql
system_prompts (
  id                  UUID PRIMARY KEY,
  project_id          UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,

  -- Identity
  label               TEXT NOT NULL,           -- user-defined name, e.g. "Formal tone"
  content             TEXT NOT NULL,           -- the actual prompt text

  -- Immutability: content must not be editable after creation.
  -- To "edit", create a new row. Old rows are archived, not modified.
  is_active           BOOLEAN NOT NULL DEFAULT false,
  is_archived         BOOLEAN NOT NULL DEFAULT false,

  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Enforce: only one active prompt per project at the database level
  -- This index makes it physically impossible to have two is_active=true
  -- rows for the same project_id, even under concurrent writes
);

CREATE UNIQUE INDEX uq_one_active_prompt_per_project
  ON system_prompts (project_id)
  WHERE is_active = true;

-- Label must be unique within a project (helps users identify prompts)
CREATE UNIQUE INDEX uq_prompt_label_per_project
  ON system_prompts (project_id, label)
  WHERE is_archived = false;
```

**Two rules to enforce at the REST API layer**:
1. `PATCH .../activate` — sets the target prompt to `is_active = true` and the previously active one to `is_active = false` in a **single transaction**. The DB partial unique index is your safety net if the transaction logic ever has a bug.
2. Deletion is not allowed — only archiving (`is_archived = true`, `is_active = false`). You cannot archive the currently active prompt without first activating another one.

**One thing to note on immutability**: since `content` must not be editable, your REST API should only expose a `PATCH` for `label` (rename) and `is_active` (activate). There should be no endpoint to modify `content` — to change the prompt text, the user creates a new one.

---

## Issue 16 — `system_prompt_id` on Sessions

Add `system_prompt_id UUID REFERENCES system_prompts(id)` to the sessions table. Populate it at session initialization from the project config load. Even though A/B testing is not implemented yet, this column costs nothing and gives you full historical data from day one.

---

## Issue 18 — `traffic_weight` Explanation (No Action Required)

This was the mechanism that would make A/B testing work: each active prompt gets a weight (e.g., prompt A = 70, prompt B = 30), and the session manager uses those weights to probabilistically assign a prompt at connection time. Since A/B testing is deferred, skip the column entirely for now. You can add it in a migration when the feature is designed properly.

---

## Summary of What Needs to Be Added/Updated in the SRS

| Item | SRS Change |
|---|---|
| RAG vector accuracy | Clarify vectors are float arrays, not scalars |
| `rag_search` event | Add to Gateway → Client message table |
| Tool taxonomy | Document three tool classes: `client`, `gateway`, `builtin` |
| SSRF constraint | New constraint: MCP/tool endpoint URLs must be HTTPS and non-RFC1918 |
| Tool tables | Replace single `tools` table with `tools` + `tool_endpoints` + `tool_executions` |
| MCP tables | Add `mcp_servers` table |
| `auth_secret_ref` | All credential fields store Key Vault reference names only |
| `rag_enabled` flag | Add to ProjectConfig and project_configs table |
| `project_documents` | New table |
| Azure cascade delete | Blob + AI Search cleanup on project/document deletion |
| `rag_timeout_ms` | Add to project configuration |
| Voice RAG latency note | Acknowledge in SRS |
| MCP discovery | Define interval, manual trigger endpoint, Redis cache, deprecation vs deletion |
| `subscription_plans` | New table |
| `system_prompts` | New table, extracted from project_configs |
| `system_prompt_id` | Add to sessions table |
| Prompt immutability | No PATCH on content, only create/archive/activate |
| Plan-gated limits | `max_system_prompts_per_project` driven by subscription plan |

## @ranga1729

also another question. Users connect to my websocket gateway service via a websocket connection. I was thiking can I let users give options more than websockets, light weight solutions than websockets to connect to my server. and it will be good for my server as well to handle the heavy weight. what do you think ?

## Copilot

Good question. The answer is: **yes, but only for specific API types** — and your existing architecture already creates a natural boundary for where each transport makes sense.

---

## The Core Insight — Your Two Adapter Types Map to Two Transport Tiers

Look at what your adapters actually do on the provider side:

| Adapter | Provider-side protocol | What the gateway actually does |
|---|---|---|
| WS Bridge (Realtime, Gemini Live) | WSS (persistent, binary, bidirectional) | Relays a continuous audio stream in both directions simultaneously |
| HTTP Orchestrator (Chat Completions, Responses, Gemini Generate) | HTTPS/SSE (one request per turn) | Sends one HTTP request, streams the response back, waits for next input |

The WS Bridge adapters are inherently real-time and bidirectional — audio chunks flow in both directions continuously. WebSocket is the only sensible client-side transport here.

The HTTP Orchestrator adapters are turn-based — the client sends one message, the server streams one response, then it waits. This is a request/response pattern with streaming on the way back. **WebSocket is overkill here**, and SSE + HTTP POST handles it more naturally.

---

## The Two Transports

### Transport 1 — WebSocket (existing)
**Required for**: OpenAI Realtime, Gemini Live (any audio-capable WS Bridge adapter)

Keep exactly as designed. No changes needed.

```
wss://host/ws?project_id=xxx
Authorization: Bearer <api_key | ephemeral_token>
```

### Transport 2 — SSE + HTTP POST (new, lightweight)
**Available for**: OpenAI Chat Completions, OpenAI Responses API, Gemini Generate Content (HTTP Orchestrator adapters only)

Two separate HTTP connections:

```
# Open the event stream (long-lived GET, server pushes events down)
GET https://host/sse?project_id=xxx
Authorization: Bearer <api_key | ephemeral_token>

# Send messages (short-lived POST, one per turn)
POST https://host/sse/send?session_id=<session_id>
Authorization: Bearer <api_key | ephemeral_token>
Content-Type: application/json
{"type": "text_input", "text": "..."}
```

---

## Why SSE + HTTP POST Is Lighter

| Dimension | WebSocket | SSE + HTTP POST |
|---|---|---|
| Connection overhead | Single TCP upgrade handshake | Standard HTTP/1.1 or HTTP/2 |
| Proxy/firewall friendliness | Often blocked or requires special config | Works everywhere HTTP works |
| Browser support | Universal | Universal (EventSource API) |
| Server memory per connection | Higher — full duplex socket held open | Lower — SSE is essentially a streaming HTTP response |
| Sticky session requirement | Mandatory | Only for the SSE GET connection duration |
| Binary data | Native | Needs base64 (but HTTP Orchestrator adapters are text-only, so irrelevant) |
| Built-in reconnection | Manual (you implement it) | Automatic — browser EventSource reconnects natively |
| Load balancer complexity | nginx must be configured for WS upgrade + sticky | Standard HTTP — nginx handles it naturally |

For text-only sessions, **WebSocket's bidirectional full-duplex capability is wasted**. You're using a persistent tunnel to have a turn-based conversation. SSE is purpose-built for "server streams data to client."

---

## How the Session Flow Changes for SSE

### Connection and initialization
```
Client opens GET /sse?project_id=xxx
→ Gateway authenticates (same logic as WebSocket pre-upgrade)
→ Gateway initializes session, loads config, instantiates adapter
→ Server sends first SSE event:

data: {"type":"session_started","session_id":"abc-123"}
```

### Sending a message
```
Client POSTs to /sse/send?session_id=abc-123
Body: {"type":"text_input","text":"What is the capital of France?"}

→ Gateway routes to adapter
→ Adapter calls LLM provider (HTTP POST to OpenAI/Google)
→ Response streams back as SSE events on the GET connection:

data: {"type":"text_delta","delta":"The "}
data: {"type":"text_delta","delta":"capital "}
data: {"type":"text_delta","delta":"of France is Paris."}
data: {"type":"text_done","text":"The capital of France is Paris."}
data: {"type":"turn_complete"}
```

### Tool calls (client-executed tools)
```
→ Gateway sends tool call via SSE:
data: {"type":"tool_call","tool_call_id":"call_123","name":"get_weather","arguments":{"city":"London"}}

→ Client POSTs the result via /sse/send:
{"type":"tool_result","tool_call_id":"call_123","result":{"temp":18}}
```

### Ping/keepalive
SSE has a built-in keepalive mechanism — the server sends a `: heartbeat` comment line periodically. No need for a separate `ping`/`pong` message type.

### Disconnect
Client closes the GET connection. Gateway detects the dropped stream and runs the teardown sequence identically to WebSocket teardown.

---

## What Needs to Change in the Architecture

### 1. Capability flag on API types
Add `supports_sse_transport BOOLEAN` to the `llm_provider_apis` catalog table. HTTP Orchestrator adapters get `true`, WS Bridge adapters get `false`. When a client tries to connect via SSE to a project configured with a WS Bridge adapter, the gateway rejects with HTTP 400 and a clear error.

### 2. Protocol Handler splits into two implementations
The Protocol Handler layer (currently one implementation) needs to abstract over the transport:

```
ITransportHandler
├── WebSocketTransportHandler   (existing)
└── SseTransportHandler         (new)
```

Both implement the same interface: `ReadNextMessageAsync()` and `SendEventAsync(LLMEvent)`. Everything above this layer (session state machine, adapter calls, usage accumulation) stays identical.

### 3. New endpoints
```
GET  /sse              → open event stream (authentication + session init)
POST /sse/send         → send a client message to an active SSE session
```

The POST endpoint looks up the session in Redis by `session_id` and routes the message to the correct gateway node via Redis pub/sub (same mechanism as admin force-close). This means SSE doesn't strictly need sticky sessions at the load balancer — the GET connection does, but the POST can land on any node.

### 4. Close codes become SSE error events
WebSocket close codes (4001, 4002, etc.) don't exist in SSE. Replace them with a final error event before closing the stream:

```
data: {"type":"error","code":"session_timeout","message":"...","retryable":false}
data: {"type":"session_ended"}
[server closes the SSE stream]
```

### 5. Session registry in Redis
No structural change needed. SSE sessions are registered exactly the same way as WebSocket sessions — same fields, same TTL logic. The `adapter_type` field already distinguishes them at the adapter layer.

---

## What to Communicate to Platform Users

Platform users choosing a project configuration should see this clearly:

| API Type | Supports WebSocket | Supports SSE |
|---|---|---|
| OpenAI Realtime API | ✅ | ❌ |
| Gemini Live API | ✅ | ❌ |
| OpenAI Chat Completions | ✅ | ✅ |
| OpenAI Responses API | ✅ | ✅ |
| Gemini Generate Content | ✅ | ✅ |

WebSocket always works for everything. SSE is the lightweight option when your project does not need audio.

The integration guide page (UI-05 in the SRS) should show connection examples for both transports for applicable API types.

---

## What You Should NOT Add

- **HTTP long polling** — strictly inferior to SSE in every dimension. SSE exists precisely to replace it. No reason to support both.
- **WebTransport** — QUIC-based, not universally supported, adds significant infrastructure complexity. Worth revisiting in v2 if you have latency-sensitive use cases that justify it.
- **gRPC streaming** — efficient, but requires generated client stubs. Your platform users are web developers expecting a JSON API. The client-side complexity is not worth it at this stage.

---

## Summary

Add SSE + HTTP POST as a second supported client transport, available exclusively for HTTP Orchestrator adapter API types (text-only). WebSocket remains the only option for WS Bridge adapters (audio/voice). The adapter layer and session state machine are untouched — only the transport handler layer changes. This is a clean, well-scoped addition that meaningfully reduces server load for text-only sessions and makes your platform more accessible to clients in restricted network environments.

## @ranga1729

no no. you got it wrong. my platform work in the middle between my plaform users and the llm providers. yes, some llm apis are woring with websockets and some of them are using HTTP SSE with streaming. that's how my platform handles the communication with llm providers and their speicfic apis. that's not the direction I'm talking about. I'm talking about the how my users connect with my platform. currently, that's also always a websocket connection. that's why we call it the "Websocket gateway service". my platform users set their projects in my platform and connect to my service via a websocket. then my platform will load those project details to set a communication channel with their selected LLM provider. my platform working in the middle. like a translater. and I think we should keep the the communication between this platform and llm providers as llm providers want. like if gemini live api needs a websocket, this platform;s gemini live adapter will use a websocket, if open ai chatcompletion api uses http sse only, we will connect using that mehtod. but I was thinking why we always need websocket between the communication of my users and my platform ? aren't there any other solutions we can use ?

## Copilot

Your previous message is actually exactly what my previous answer was addressing — I was already talking about the **End Client → Platform** connection, not the Platform → LLM connection. It seems the response was unclear, so let me restate it directly and concisely.

---

## The Actual Situation

```
End Client  ──[???]──►  ChatN Platform  ──[whatever LLM needs]──►  LLM Provider
                 ↑
         This is what you're asking about.
         Currently always WebSocket.
```

The platform-to-LLM side stays exactly as designed — Gemini Live gets WSS, Chat Completions gets HTTPS/SSE, etc. You are right that you should let the LLM providers dictate that side. That is not changing.

The question is only about the left arrow.

---

## The Honest Answer

Whether WebSocket is replaceable on the client side depends entirely on **which adapter the project is configured with**. Your two adapter types create a hard split:

### WS Bridge adapter projects (Realtime, Gemini Live)
**WebSocket is non-negotiable.** The End Client is streaming raw PCM audio chunks continuously in both directions simultaneously. That requires a persistent, full-duplex, low-latency binary connection. There is no lighter alternative that handles this correctly. SSE is server-to-client only. HTTP/2 streams add complexity without solving the bidirectional audio problem. WebSocket is the right tool.

### HTTP Orchestrator adapter projects (Chat Completions, Responses, Gemini Generate)
**WebSocket is not required.** These are text-only, turn-based conversations:
- Client sends one message
- Server streams one response back
- Both sides wait
- Repeat

This is exactly what **SSE + HTTP POST** was designed for. It is genuinely lighter:

```
# Client opens one long-lived GET — server pushes events down this
GET /sse?project_id=xxx
Authorization: Bearer <token>

# Client sends each message as a normal POST
POST /sse/send?session_id=yyy
{"type":"text_input","text":"Hello"}
```

The server pushes `text_delta`, `text_done`, `turn_complete`, `tool_call`, `error`, etc. down the GET connection as SSE events. The client POSTs `tool_result`, `interrupt`, `ping` etc. as normal HTTP requests.

---

## Why SSE Specifically Is Better Than WebSocket Here

| | WebSocket | SSE + HTTP POST |
|---|---|---|
| Protocol overhead | Full duplex TCP tunnel maintained | Standard HTTP — no special handshake |
| Corporate proxy/firewall | Frequently blocked or requires special nginx config | Works everywhere HTTP works — no exceptions |
| Browser reconnection | You must implement it manually | Built into the browser's `EventSource` API natively |
| Server memory per connection | Higher — full duplex socket kept alive | Lower — a streaming HTTP response |
| Binary data support | Native | Not needed — HTTP Orchestrator adapters are text-only |
| Load balancer config | nginx needs WebSocket upgrade support + sticky sessions | Standard HTTP — nginx handles it without special config |

For a text-only turn-based session, maintaining a full-duplex WebSocket tunnel is genuinely wasteful. You have a persistent two-way pipe but you only ever use one direction at a time.

---

## What This Means for Your Architecture

The change is contained to one layer — the transport handler. Everything above it (session state machine, adapters, Redis, usage tracking) is completely untouched.

```
Current:
IProtocolHandler
└── WebSocketProtocolHandler (only option)

New:
ITransportHandler
├── WebSocketTransportHandler   (all adapter types)
└── SseTransportHandler         (HTTP Orchestrator adapters only)
```

Both implement the same interface upward. The session manager, adapter factory, and all adapters call the same methods regardless of which transport the client used.

Add one capability flag to the `llm_provider_apis` catalog table: `supports_sse_transport BOOLEAN`. WS Bridge adapters get `false`. HTTP Orchestrator adapters get `true`. If a client tries to open an SSE connection to a voice project, the gateway returns HTTP 400 immediately with a clear error before any session is created.

---

## What This Is NOT

This is not about supporting multiple transports simultaneously or letting clients pick arbitrarily. The transport is determined by the project's configured API type. A platform user configuring a Chat Completions project gets both options documented in their integration guide. A platform user configuring a Realtime API project only gets WebSocket documented — SSE is not presented as an option because it physically cannot support audio.

---

## Summary

My previous answer was already saying exactly this — sorry it was not clear. The bottom line:

- **Voice projects** (WS Bridge adapters): WebSocket only. No alternative exists that handles bidirectional audio.
- **Text projects** (HTTP Orchestrator adapters): SSE + HTTP POST is a legitimate, lighter, more firewall-friendly alternative to WebSocket. You should support both and let the End Client choose based on their environment.

## @ranga1729

Ok now I understand. as my understadning, based on the API users has selected, now users can decide if they use websockets or HTTP SSE. but in live api and realtime api, they must only connect via a websocket. this means, we can give users an option in some cases. and I hope this will reduce the unnecessary websocket connections that my platform has to handle. and ultimately reduce the operational cost as well. if this is possible what are the changes we must do ? and also, if we are going to do this, we may need to chnage the name "websocket gateway service" into something general. suggest a one. also I think we should do this, becuase I'm building this platform in a flexible, extendible way so later(like in a admin-dashboard) must be able to add new providers, new apis, new models easily without doing more code chnages.  

## Copilot

## New Service Name

**"Connection Gateway Service"** — protocol-neutral, accurately describes what it does (manages connections between End Clients and LLM providers), and doesn't imply a specific transport.

Other options if you want to consider:
- **"Session Gateway Service"** — emphasizes session lifecycle management
- **"Realtime Gateway Service"** — emphasizes the real-time nature but still protocol-neutral

I'd go with **Connection Gateway Service**.

---

## Full Change List

### 1. Catalog Table Change — `llm_provider_apis`

This is the most important change. The catalog drives everything. Add two columns:

```sql
ALTER TABLE llm_provider_apis
  ADD COLUMN supports_websocket     BOOLEAN NOT NULL DEFAULT true,
  ADD COLUMN supports_sse           BOOLEAN NOT NULL DEFAULT false;
```

Updated catalog data:

| Provider | API Type | supports_websocket | supports_sse |
|---|---|---|---|
| OpenAI | Realtime API | ✅ | ❌ |
| OpenAI | Chat Completions | ✅ | ✅ |
| OpenAI | Responses API | ✅ | ✅ |
| Google | Gemini Live | ✅ | ❌ |
| Google | Gemini Generate Content | ✅ | ✅ |

This is how a future admin adding a new provider/API correctly declares what transports it supports — **zero code changes**, just a database insert with the right flags.

---

### 2. Transport Abstraction Layer — Code Architecture

Currently your service has an implicit assumption that every client connection is a WebSocket. That assumption needs to be broken into an explicit interface.

```
Current (implicit):
ConnectionManager → WebSocket-specific code scattered throughout

New (explicit):
ConnectionManager → IClientTransport
                    ├── WebSocketClientTransport
                    └── SseClientTransport
```

`IClientTransport` is the only interface the rest of the service talks to. It exposes:

```csharp
interface IClientTransport
{
    // Read next message from the client
    Task<ClientMessage?> ReadMessageAsync(CancellationToken ct);
    
    // Send an event to the client
    Task SendEventAsync(LLMEvent evt, CancellationToken ct);
    
    // Send a raw typed event (error, session_started, etc.)
    Task SendControlEventAsync(string type, object payload, CancellationToken ct);
    
    // True when the client has disconnected
    bool IsConnected { get; }
    
    // Clean close
    Task CloseAsync(int closeCode, string reason, CancellationToken ct);
    
    // Which transport this is
    ClientTransportType TransportType { get; }  // WebSocket | SSE
}
```

The `ConnectionManager`, `ProtocolHandler`, session state machine, adapters, and usage accumulator **never reference WebSocket or SSE directly** — they only call `IClientTransport` methods. This is where the extensibility lives. If you ever add a third transport (gRPC streaming, WebTransport, etc.), only a new `IClientTransport` implementation is needed. Nothing else changes.

---

### 3. New Endpoints

```
# Existing WebSocket endpoint — unchanged
wss://host/ws?project_id=xxx
Authorization: Bearer <api_key | ephemeral_token>

# New SSE endpoints
GET  https://host/sse?project_id=xxx          → open event stream
Authorization: Bearer <api_key | ephemeral_token>

POST https://host/sse/messages?session_id=xxx  → send a client message
Authorization: Bearer <api_key | ephemeral_token>
Content-Type: application/json
```

### Connection validation at the gateway entry point:

```
Inbound SSE request arrives
        ↓
Authenticate (same logic as WebSocket)
        ↓
Load project config from Redis/DB
        ↓
Check llm_provider_apis.supports_sse for this project's api_type
        ↓
    supports_sse = false?
        → HTTP 400
          {"error": "sse_not_supported",
           "message": "This project uses an audio API type that requires WebSocket."}
        ↓
    supports_sse = true?
        → Continue session initialization
```

---

### 4. SSE Transport — How It Works

#### Opening the stream (`GET /sse`)
```
Client                          Gateway
  |                                |
  |── GET /sse?project_id=xxx ────►|
  |   Authorization: Bearer <key>  |
  |                                |── Authenticate
  |                                |── Load project config
  |                                |── Check supports_sse = true
  |                                |── Initialize session
  |                                |── Register in Redis
  |                                |
  |◄── HTTP 200 ───────────────────|
  |    Content-Type: text/event-stream
  |    Cache-Control: no-cache
  |    X-Session-Id: <session_id>   ← client reads this for subsequent POSTs
  |                                |
  |◄── data: {"type":"session_started","session_id":"xxx"} 
  |                                |
  |◄── : heartbeat ────────────────|  (every 15s, keeps connection alive)
```

#### Sending a message (`POST /sse/messages`)
```
Client                          Gateway
  |                                |
  |── POST /sse/messages?session_id=xxx
  |   {"type":"text_input","text":"Hello"}
  |                                |
  |                                |── Look up session in Redis
  |                                |── If session on this node: route directly
  |                                |── If session on another node: Redis pub/sub
  |                                |
  |◄── HTTP 202 Accepted ──────────|
  |   (response body is empty —    |
  |    actual response comes down  |
  |    the SSE GET connection)     |
```

#### Response streaming (down the GET connection)
```
data: {"type":"text_delta","delta":"Hello "}

data: {"type":"text_delta","delta":"there!"}

data: {"type":"text_done","text":"Hello there!"}

data: {"type":"turn_complete"}
```

#### Keepalive
SSE has a built-in mechanism — the server sends comment lines:
```
: heartbeat
```
Every 15 seconds. The browser's native `EventSource` handles reconnection automatically if the connection drops. No `ping`/`pong` message type needed for SSE (keep it in the protocol for WebSocket sessions only).

#### Session close
Since SSE has no close codes, the gateway sends a final event then closes the stream:
```
data: {"type":"error","code":"session_timeout","message":"No activity for 2 minutes","retryable":false}

data: {"type":"session_ended"}

[server closes HTTP response]
```

---

### 5. Close Codes → SSE Error Codes Mapping

WebSocket close codes don't exist in SSE. Add a mapping:

| WebSocket Close Code | SSE error event code |
|---|---|
| 4001 | `authentication_failed` |
| 4002 | `provider_unavailable` |
| 4003 | `session_limit_exceeded` |
| 4004 | `project_inactive` |
| 4005 | `session_timeout` |
| 4006 | `admin_force_close` |
| 4029 | `rate_limit_exceeded` |

All delivered as:
```json
{"type":"error","code":"<code>","message":"<human readable>","retryable":false}
```
followed immediately by:
```json
{"type":"session_ended"}
```

---

### 6. Session Registry — No Structural Change

Redis session registry entries stay identical. Add one field:

```
session:{session_id}
  session_id:      uuid
  project_id:      uuid
  user_id:         uuid
  adapter_type:    string
  model_id:        uuid
  node_id:         string
  started_at:      timestamp
  transport:       "websocket" | "sse"    ← NEW
```

The `transport` field lets admin endpoints and the session monitor UI show which transport each session is using. Useful for operational visibility and future analytics (e.g., "what % of our sessions are SSE vs WebSocket?").

---

### 7. Sessions Table — Add Transport Column

```sql
ALTER TABLE sessions
  ADD COLUMN transport TEXT NOT NULL DEFAULT 'websocket';
  -- 'websocket' | 'sse'
```

This feeds into usage analytics and the session monitor UI.

---

### 8. UI Changes (Dashboard)

**Project Configuration Wizard (UI-03)** — after the user selects an API type, show the supported transports:

```
Selected: OpenAI Chat Completions API
─────────────────────────────────────
Supported client transports:
  ✅ WebSocket    wss://host/ws
  ✅ SSE          https://host/sse
```

```
Selected: OpenAI Realtime API
─────────────────────────────────────
Supported client transports:
  ✅ WebSocket    wss://host/ws
  ❌ SSE          (Not available — audio API requires WebSocket)
```

**Integration Guide Page (UI-05)** — show code examples for both transports where applicable. Currently it only shows WebSocket examples. Add SSE section:

```
## Connecting via SSE (text-only API types)

### Open the event stream
const evtSource = new EventSource('/sse?project_id=xxx', {
  headers: { Authorization: 'Bearer <token>' }
});

const sessionId = ...; // read from session_started event

### Send a message
fetch(`/sse/messages?session_id=${sessionId}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', Authorization: 'Bearer <token>' },
  body: JSON.stringify({ type: 'text_input', text: 'Hello' })
});

### Receive events
evtSource.onmessage = (e) => {
  const event = JSON.parse(e.data);
  // handle text_delta, text_done, turn_complete, tool_call, error, etc.
};
```

**Session Monitor (UI-06)** — add a transport badge (`WS` / `SSE`) to the active sessions list and session detail view.

---

### 9. SRS Sections to Update

| Section | Change |
|---|---|
| Title / service name | "WebSocket Gateway Service" → "Connection Gateway Service" throughout |
| 5.1 High-Level Architecture | Update service name, mention dual transport |
| 5.3 Component Decomposition | Replace "Protocol Handler" with "Transport Handler (IClientTransport)" — explain WebSocket and SSE implementations |
| 6.1 Connection Endpoint | Add SSE endpoints alongside WebSocket endpoint |
| 6.2 Pre-Upgrade Authentication | Rename to "Pre-Connection Authentication" — cover both WebSocket upgrade and SSE GET request |
| 6.4 Active Session — Dual Async Loop | Clarify this applies to WebSocket; SSE uses POST handler + SSE push instead of two async loops |
| 6.8 WebSocket Close Codes | Rename to "Session Close Codes" — add SSE error event equivalents |
| 11.1 UI — Integration Page | Add SSE examples |
| 11.2 External LLM Provider Interfaces | Clarify this table describes Platform→Provider connections, not Client→Platform |
| 14. Appendix | Add "Supported Client Transports" column to provider support matrix |
| New constraint | C-11: SSE transport MUST only be offered for API types where `supports_sse = true` in the catalog. The gateway MUST reject SSE connection attempts for audio API types with HTTP 400. |

---

## Summary

```
Changes needed:
─────────────────────────────────────────────────────────
1. Service rename: "Connection Gateway Service"
2. DB: Add supports_websocket, supports_sse to llm_provider_apis
3. DB: Add transport column to sessions table
4. Redis: Add transport field to session registry
5. Code: Extract IClientTransport interface
6. Code: WebSocketClientTransport (refactor existing code into this)
7. Code: SseClientTransport (new implementation)
8. Endpoints: GET /sse and POST /sse/messages
9. UI: Transport badge in session monitor
10. UI: Transport capability shown in project config wizard
11. UI: SSE code examples in integration guide
12. SRS: Update service name, section 6 throughout, new constraint C-11
─────────────────────────────────────────────────────────

Zero changes to:
  Session state machine
  All five LLM adapters
  Adapter Factory
  Redis session registry structure (one new field only)
  Usage accumulation and flush logic
  Admin force-close via pub/sub
  Authentication logic
  PostgreSQL schema (one new column only)
```

The extensibility goal you mentioned is fully met: when an admin adds a new provider/API type via the admin dashboard, they set `supports_websocket` and `supports_sse` on the new catalog row. The gateway automatically enforces the correct transport rules for that API type with no code changes.

## @ranga1729

ok we must do this changes as well. now another question. I need you to evaluate the current authentication system between users and "session gateway service"(I like this name). Currrently I have a mechanism to create ephemeral tokens valid for 5 mins. I need to work it like this. users of my platform who need to establish a communicaton channels with my connection gateway service with a api via websocket or HTTP SSE, will send a HTTP POST request to "/auth/token" endpoint. request header will include "Authorization: Bearer <Long lived api key>" and the body will include details like the project id, scope(read-stream). then my server will validate the api key against some checks(like the validity, if the mentioned project has this api key, valid against the susbscripton plan constraints, ...etc ). if it's valid, then the server will generate a ephemeral token and the redis will store this token with a very short TTL. and the server will send this token back to the client "{"token": "xyz_123"}". now the clinet uses the ticket to open the real connection(websocket or HTTP SSE). then the Session gateway service will receive this request and extract the ticket from the query paramters. then the server look up the token in the redis, if it found, the connection should be established, and the token must be deleted immediately(since it's one time use only). if not found, reject the connection request with a 401. like this way, ephemeral token process is also automated and the securiy is hardened. is this ok ?

## Copilot

The overall flow is sound and the security thinking is correct. But there are several specific issues to address before this is production-ready.

---

## What Is Good

- Separating token issuance (REST API) from connection establishment (Session Gateway) is the right architecture
- Short TTL is correct
- One-time use enforcement is correct
- Automated flow removes the burden from platform users to implement their own token exchange

---

## Issues Found

### 1. Wrong Service Is Issuing the Token

You say the `/auth/token` endpoint issues the token. Which service hosts this endpoint matters architecturally.

Per your SRS section 5.1:
- **REST API Service** handles all management-plane operations — authentication, token issuance, project management
- **Session Gateway Service** handles all data-plane operations — connections, adapters, message routing

Token issuance is a management-plane operation. It must live in the **REST API Service**, not the Session Gateway Service. The Session Gateway Service only **validates** the token at connection time. This separation is already in your SRS — the endpoint should be:

```
POST /api/v1/projects/{project_id}/tokens
Authorization: Bearer <api_key>
```

Not a generic `/auth/token`. Tying it to the project in the URL makes it RESTful and enforces that the token is always scoped to a specific project.

---

### 2. Raw Token Must Never Be Stored in Redis

You said:
> *"Redis will store this token with a very short TTL"*

Per your own constraint C-03:
> *"Platform-issued API keys MUST be stored as one-way HMAC-SHA256 hashes. The raw key MUST be shown exactly once at creation time and never stored or returned thereafter."*

The same principle applies to ephemeral tokens. What Redis stores is the **HMAC-SHA256 hash of the token**, not the raw token. What you return to the client is the **raw token** (shown once only). At connection time, the gateway hashes the inbound token and looks up the hash in Redis.

```
Issuance:
  raw_token = cryptographically_random(256 bits)
  token_hash = HMAC_SHA256(raw_token, server_secret)
  Redis.SET(token_hash, session_metadata, TTL=60s)
  return raw_token to client  ← only time raw token exists outside client memory

Connection:
  inbound_token = from query param
  token_hash = HMAC_SHA256(inbound_token, server_secret)
  result = Redis.GETDEL(token_hash)  ← atomic get-and-delete
  if result == nil → reject 401
  if result exists → proceed
```

---

### 3. Use `GETDEL` Not `SETNX` for Atomicity

You said:
> *"The token must be deleted immediately (since it's one time use only)"*

Your SRS currently says SETNX. Neither is quite right. The correct atomic operation for this use case is Redis `GETDEL` (available since Redis 6.2, which is your minimum version):

```
GETDEL token_hash
```

This atomically gets the value AND deletes the key in a single operation. If two concurrent connection attempts come in with the same token:
- First request: `GETDEL` returns the value and deletes the key
- Second request: `GETDEL` returns nil — key is already gone — rejected

This is cleaner than SETNX because:
- SETNX marks as "used" but leaves the key in Redis until TTL expires (wasted memory)
- DELETE after GET has a race window between the GET and the DELETE
- GETDEL is atomic with no race window and immediately frees the Redis key

---

### 4. Token in Query Parameters Is a Security Tradeoff You Must Acknowledge

You said the client uses the token to open the connection via query parameters. This is the pragmatic choice for WebSocket (browsers cannot set custom headers on WebSocket connections) but it comes with a known risk:

Query parameters appear in:
- Nginx/load balancer access logs
- Server application logs
- Browser history
- CDN logs
- Referrer headers if the page navigates

For WebSocket connections this is an accepted industry tradeoff — there is no other option from a browser. But you must mitigate it:

1. **60-second TTL maximum** — even if the token leaks from a log, it expires before an attacker can use it
2. **Single-use** — a leaked token that was already used is worthless
3. **Nginx log masking** — configure your load balancer to redact the `token` query parameter from access logs

For SSE connections, the situation is different. SSE is a standard HTTP GET request, which means the browser **can** set an Authorization header via the `fetch` API or a custom EventSource implementation. For SSE, the token should go in the Authorization header, not the query string:

```
# WebSocket (browser limitation — query param is the only option)
wss://host/ws?project_id=xxx&token=<ephemeral_token>

# SSE (standard HTTP — use Authorization header)
GET /sse?project_id=xxx
Authorization: Bearer <ephemeral_token>
```

Add this distinction explicitly to your SRS.

---

### 5. Scope Field Needs to Be Defined Properly

You mentioned `scope: "read-stream"` in the request body. What does this mean in practice?

Right now you only have one scope. But if you leave scope as a free-text field, it becomes meaningless. Define it as an enum from day one:

| Scope | Meaning |
|---|---|
| `session:connect` | Allows opening one WebSocket or SSE connection to the specified project |

For now this is the only valid scope. The field exists so future scopes (e.g., `session:readonly` for monitoring, `admin:session` for force-close) can be added without changing the token structure. The token's scope must be validated at connection time by the gateway — a token issued with `session:connect` should not be accepted on any other endpoint.

The scope must be embedded in the Redis payload alongside the project_id, not just stored as a label:

```json
Redis value (stored against token_hash):
{
  "project_id": "uuid",
  "scope": "session:connect",
  "issued_to_ip": "203.0.113.42",
  "expires_at": "2026-04-30T12:01:00Z"
}
```

---

### 6. Audit Log Entry Is Missing

Your SRS (section 8.3) already specifies:
> *"Every issuance is audit-logged with: project_id, requesting_ip, created_at, expires_at, and whether the token was used or expired unused."*

Your proposed flow doesn't mention this. Two things needed:

**At issuance** (REST API Service): write an audit log entry immediately.

**At connection time** (Session Gateway): update the audit record — was the token used or did it expire unused? This requires a database record alongside the Redis entry, not just Redis alone. Redis handles fast lookup and single-use enforcement. PostgreSQL handles the audit trail.

```sql
ephemeral_tokens (
  id              UUID PRIMARY KEY,
  project_id      UUID NOT NULL REFERENCES projects(id),
  token_hash      TEXT NOT NULL UNIQUE,
  scope           TEXT NOT NULL DEFAULT 'session:connect',
  requesting_ip   INET NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at      TIMESTAMPTZ NOT NULL,
  used_at         TIMESTAMPTZ,          -- NULL until consumed
  outcome         TEXT                  -- NULL | 'used' | 'expired'
);
```

Records where `outcome IS NULL` and `expires_at < now()` are cleaned up by a background job after 24 hours (per your SRS).

---

### 7. Subscription Plan Validation Is Missing from Your Description

You said the server validates:
- Key validity
- Whether the project has this API key
- Subscription plan constraints

But what subscription plan constraints specifically apply to token issuance? You need to define these:

- **Token issuance rate**: how many ephemeral tokens can be issued per minute per project? (Prevents a compromised API key from flooding your Redis with token entries)
- **Concurrent session limit**: checked at token issuance time, not just at connection time. If a project is already at its concurrent session limit, reject the token issuance immediately rather than letting the client hold a token they can never use

---

## Corrected Full Flow

```
1. ISSUANCE (REST API Service)
───────────────────────────────
Client:
  POST /api/v1/projects/{project_id}/tokens
  Authorization: Bearer <long_lived_api_key>
  {"scope": "session:connect"}

REST API Service:
  a. Hash inbound API key → look up in Redis (fast path) → DB fallback
  b. Validate: key active, not expired, belongs to project_id in URL
  c. Validate: project is active
  d. Validate: subscription plan allows token issuance (rate check)
  e. Check concurrent session count vs plan limit → reject if at limit
  f. Generate raw_token = CSPRNG(256 bits)
  g. token_hash = HMAC_SHA256(raw_token, server_secret)
  h. Redis.SET(token_hash, {project_id, scope, issued_to_ip, expires_at}, TTL=60s)
  i. INSERT INTO ephemeral_tokens (audit record, outcome=NULL)
  j. Return {"token": "<raw_token>", "expires_in": 60}

2. CONNECTION (Session Gateway Service)
────────────────────────────────────────
WebSocket: wss://host/ws?project_id=xxx&token=<raw_token>
SSE:       GET /sse?project_id=xxx  +  Authorization: Bearer <raw_token>

Session Gateway:
  a. Extract token from query param (WS) or Authorization header (SSE)
  b. token_hash = HMAC_SHA256(inbound_token, server_secret)
  c. payload = Redis.GETDEL(token_hash)   ← atomic get-and-delete
  d. If nil → HTTP 401, no connection
  e. If expired_at < now() → HTTP 401 (should not happen if TTL set correctly, but defensive check)
  f. Validate scope == 'session:connect'
  g. Validate project_id in payload matches project_id in request
  h. UPDATE ephemeral_tokens SET used_at=now(), outcome='used'
  i. Continue with session initialization (load config, secrets, adapter, etc.)

3. BACKGROUND JOB
──────────────────
Every hour:
  UPDATE ephemeral_tokens
  SET outcome = 'expired'
  WHERE outcome IS NULL AND expires_at < now()
```

---

## Summary of Issues

| # | Issue | Severity |
|---|---|---|
| 1 | Token issuance must be in REST API Service, not gateway | High |
| 2 | Redis stores token hash, never raw token | Critical |
| 3 | Use `GETDEL` not SETNX or separate GET+DELETE | High |
| 4 | Query param risk acknowledged; SSE should use Authorization header | High |
| 5 | Scope must be an enum, embedded in Redis payload, validated at connection | Medium |
| 6 | Audit log record in PostgreSQL required alongside Redis entry | Medium |
| 7 | Plan-level rate limit and concurrent session check at issuance time | Medium |

The core idea is solid — it is the right pattern. The issues above are hardening details that make it production-safe.

## @ranga1729

```
1. ISSUANCE (REST API Service)
───────────────────────────────
Client:
  POST /api/v1/projects/{project_id}/tokens
  Authorization: Bearer <long_lived_api_key>
  {"scope": "session:connect"}

REST API Service:
  a. Hash inbound API key → look up in Redis (fast path) → DB fallback
  b. Validate: key active, not expired, belongs to project_id in URL
  c. Validate: project is active
  d. Validate: subscription plan allows token issuance (rate check)
  e. Check concurrent session count vs plan limit → reject if at limit
  f. Generate raw_token = CSPRNG(256 bits)
  g. token_hash = HMAC_SHA256(raw_token, server_secret)
  h. Redis.SET(token_hash, {project_id, scope, issued_to_ip, expires_at}, TTL=60s)
  i. INSERT INTO ephemeral_tokens (audit record, outcome=NULL)
  j. Return {"token": "<raw_token>", "expires_in": 60}

2. CONNECTION (Session Gateway Service)
────────────────────────────────────────
WebSocket: wss://host/ws?project_id=xxx&token=<raw_token>
SSE:       GET /sse?project_id=xxx  +  Authorization: Bearer <raw_token>

Session Gateway:
  a. Extract token from query param (WS) or Authorization header (SSE)
  b. token_hash = HMAC_SHA256(inbound_token, server_secret)
  c. payload = Redis.GETDEL(token_hash)   ← atomic get-and-delete
  d. If nil → HTTP 401, no connection
  e. If expired_at < now() → HTTP 401 (should not happen if TTL set correctly, but defensive check)
  f. Validate scope == 'session:connect'
  g. Validate project_id in payload matches project_id in request
  h. UPDATE ephemeral_tokens SET used_at=now(), outcome='used'
  i. Continue with session initialization (load config, secrets, adapter, etc.)

3. BACKGROUND JOB
──────────────────
Every hour:
  UPDATE ephemeral_tokens
  SET outcome = 'expired'
  WHERE outcome IS NULL AND expires_at < now()
```
your corrected flow is ok. I accept it. also another issue. some api are stateful like live api and open AI realtime api. but some needs conversatio_id chaining method to simulate the memory. and some apis need previous conversations to be sent with the new query. considering all of these facts, don't you think it's safe to keep the converdsation history in redis per session, so it will be easier us to maintain memory(like conversatio id chaining or sending the prev messages). but we won't save those message into the Database. what do you think ?

## Copilot

Your thinking is correct and this is actually already partially defined in your SRS. But there are important nuances depending on which adapter type you are dealing with, and one infrastructure concern you have not addressed yet. Let me break it down properly.

---

## What Your SRS Already Says

Your SRS already defines three different memory strategies across the five adapters:

| Adapter | Where Memory Lives | What Is Stored in Redis |
|---|---|---|
| OpenAI Realtime API | Provider holds all state | Nothing |
| Gemini Live API | Provider holds all state | Nothing |
| OpenAI Chat Completions | Redis per session | Full `messages[]` array |
| Gemini Generate Content | Redis per session | Full `contents[]` array |
| OpenAI Responses API | Provider holds state via chaining | Only `previous_response_id` |

So your intuition is already the implemented design for HTTP Orchestrator adapters. The question is whether this is safe and correct — and the answer is **yes, with specific conditions**.

---

## Why Redis Is the Right Place

### C-04 Compliance
Your constraint C-04 says:
> *"The Platform MUST NOT persist conversation content to the primary database at any point, under any circumstances."*

Redis is **not** your primary database. PostgreSQL is. Storing conversation history in Redis does not violate C-04, provided:
- The data has a TTL matching the session lifetime
- It is **explicitly deleted** at session teardown (not just left to expire)
- Redis is never used as a permanent store for this data

Your SRS constraint C-10 already captures this:
> *"Conversation history held in Redis for HTTP Orchestrator adapters MUST be explicitly deleted at session teardown and MUST NOT persist beyond the session lifetime."*

This is correct.

---

## The One Infrastructure Concern You Have Not Addressed

### Redis Persistence Configuration

Redis has two optional persistence mechanisms:
- **RDB** (Redis Database) — point-in-time snapshots written to disk periodically
- **AOF** (Append Only File) — logs every write operation to disk

If either of these is enabled on your Redis instance, conversation history that passes through Redis **is being written to disk**, even temporarily. This is a gray area for C-04 compliance — the data is not in PostgreSQL, but it is on disk somewhere in your infrastructure.

You have two options:

**Option A — Disable persistence on Redis entirely**
```
# redis.conf
save ""          # disable RDB snapshots
appendonly no    # disable AOF
```
This means Redis is purely in-memory. If the Redis node crashes, all session state (including conversation history, usage counters, session registry) is lost. Sessions in progress are dropped. This is acceptable because:
- Sessions are ephemeral by design
- Usage counters are flushed to Redis every 30 seconds and to PostgreSQL at teardown — max data loss is 30 seconds of counter increments
- The session registry is rebuilt as new sessions connect

**Option B — Enable persistence but use a dedicated Redis instance for conversation history**

Run two Redis deployments:
- **Redis-A** (persistence enabled): stores session registry, project config cache, token lookups, rate limit counters, usage counters — data you want to survive a restart
- **Redis-B** (no persistence): stores conversation history only — data that must never touch disk

This is operationally more complex but gives you the best of both: durable infrastructure data and guaranteed ephemeral conversation data.

**Recommendation**: Option A is simpler and acceptable for v1.0. Option B is the more robust long-term architecture. Add this as an explicit infrastructure constraint in the SRS.

---

## Per-Adapter Memory Strategy — Full Clarification

### WS Bridge Adapters (Realtime, Gemini Live)
```
Redis stores: nothing related to conversation
Provider stores: everything — turn history, audio context, tool call state

Why: The provider maintains a persistent WSS connection for the session lifetime.
The conversation context lives inside that connection on the provider side.
When the connection closes, the context is gone. Nothing to clean up in Redis.
```

### HTTP Orchestrator — Chat Completions and Gemini Generate
```
Redis stores: full messages[] or contents[] array, keyed by session_id
              TTL = session inactivity timeout (default 2 minutes, refreshed on activity)

On each user turn:
  1. Load full history from Redis
  2. Append new user message
  3. POST full history to provider
  4. Stream response back
  5. Append assistant response to history
  6. Save updated history back to Redis (refresh TTL)

On session teardown:
  DEL session:{session_id}:history  ← explicit delete, not waiting for TTL
```

The history grows with each turn. For long sessions this can become large. You should add a **max history size** safeguard — if the history exceeds a configurable token limit (e.g., 80% of the model's context window), implement a **sliding window**: drop the oldest turns (but never the system prompt, which is always first).

```
project_configs:
  max_history_tokens INT DEFAULT 8000   ← configurable per project
```

When history exceeds this limit, trim from the oldest user/assistant turn pairs, preserving the system prompt always.

### HTTP Orchestrator — Responses API
```
Redis stores: only previous_response_id (a short string, ~20 characters)
              keyed by session:{session_id}:response_id

On each user turn:
  1. Load previous_response_id from Redis (null on first turn)
  2. POST to /v1/responses with previous_response_id
  3. OpenAI reconstructs full conversation server-side
  4. Extract new response_id from response
  5. Save new response_id to Redis (overwrite previous)

On session teardown:
  DEL session:{session_id}:response_id
```

This is the most Redis-efficient adapter. Minimal memory footprint regardless of conversation length.

---

## Redis Key Structure — Recommended Convention

Define a consistent key naming convention now to avoid collisions as complexity grows:

```
# Session registry
session:{session_id}                    → session metadata hash

# Conversation history (HTTP Orchestrator adapters)
session:{session_id}:history            → JSON array of messages

# Response ID (Responses API adapter)
session:{session_id}:response_id        → string

# Usage counters (flushed every 30s)
session:{session_id}:usage              → hash of counter fields

# Project config cache
project:{project_id}:config             → JSON project configuration

# MCP tool discovery cache
project:{project_id}:mcp:{server_id}:tools  → JSON tool list

# API key fast lookup
apikey:{key_hash}                       → JSON key metadata

# Ephemeral token
token:{token_hash}                      → JSON token payload (GETDEL at use)

# Rate limiting
ratelimit:apikey:{key_hash}:conns       → sliding window counter
ratelimit:ip:{ip}:requests              → sliding window counter

# Admin pub/sub channel
admin:commands                          → pub/sub channel for force-close etc.
```

---

## One Additional Point — Session Recovery on Node Failure

Because conversation history lives in Redis:
- If the gateway node handling a session crashes mid-conversation, the session is dropped (WebSocket or SSE connection breaks)
- The client must reconnect, which opens a **new session**
- For WS Bridge adapters: provider-side context is also gone since the provider WSS connection was on the crashed node — the new session starts fresh
- For HTTP Orchestrator adapters: the history in Redis survives the node crash (it is in Redis, not in the node's memory) — **but the client has no way to reconnect to the same session**

This means session recovery is not something you can transparently support at v1.0. The correct behavior is:
- Node crash → client sees connection drop → client must open a new session
- New session starts with empty history for HTTP Orchestrator adapters

This is acceptable. Document it explicitly in the SRS as a known behavior. Do not attempt transparent session migration at this stage — it adds significant complexity for an edge case.

---

## Summary

| Question | Answer |
|---|---|
| Is Redis the right place for conversation history? | Yes — it is already your design for HTTP Orchestrator adapters |
| Does it violate C-04? | No — C-04 targets the primary database (PostgreSQL) |
| What is the one risk? | Redis persistence (RDB/AOF) could write conversation data to disk |
| How to mitigate? | Disable Redis persistence for v1.0, or use a dedicated no-persistence Redis instance for conversation data |
| WS Bridge adapters need Redis history? | No — provider holds all state |
| What is missing from current design? | Max history size + sliding window trimming for Chat Completions and Gemini Generate adapters |
| What about session recovery after node crash? | Not supported at v1.0 — document as known behavior |

## @ranga1729

for now I will use only redis instance for eveything to keep the cost at minimum level. if the conversation history get write to disk and its create a gray area against some constraints, just neglect it for now. and yes, we must keep the history size at a certain size so it won't grow larger endlessly. also we need to store minimum data, for these conversation in redis. also some llm apis stream data, i think using a proper mechanims, we must concatenate these chunks and create meaning full messages when storing them in the redis. like insterad of storing everything in reids, it will store the query aksed by the users and the final response given by the llm. for a one session there will be around 10(let's chnage this number if its now suitable) conversation rounds stored in, not in streamed chunks, but in a meaning way so llms can easily understand them when we used this history array to give it the context. also we need to keep the conversation_id for some llm apis. what do you think about this ? 

## Copilot

Good decisions. Let me work through the full design for this.

---

## What Gets Stored — The Core Principle

```
During streaming:    chunks accumulated in adapter memory only
After turn complete: one user message + one assembled assistant message written to Redis
In Redis:           only completed, meaningful turns — never raw deltas
```

The adapter already assembles the full response in memory to produce `TextDoneEvent`. That assembled text is exactly what gets written to Redis. The streaming chunks themselves are never written anywhere persistent.

---

## The History Array Structure

Keep it as simple as possible. The system prompt is **never** stored in the history array — it is loaded fresh from project config at each request and prepended at build time. This avoids duplicating it across every Redis entry.

### For Chat Completions adapter
```json
{
  "turns": [
    {"role": "user",      "content": "What is the capital of France?"},
    {"role": "assistant", "content": "The capital of France is Paris."},
    {"role": "user",      "content": "Tell me more about it."},
    {"role": "assistant", "content": "Paris is the capital and most populous city of France..."}
  ],
  "turn_count": 2
}
```

### For Gemini Generate adapter
```json
{
  "turns": [
    {"role": "user",  "parts": [{"text": "What is the capital of France?"}]},
    {"role": "model", "parts": [{"text": "The capital of France is Paris."}]}
  ],
  "turn_count": 2
}
```

Gemini uses `"model"` instead of `"assistant"` and wraps content in `parts[]`. The adapter handles this format difference — the storage structure follows the provider's native format so it can be sent directly without re-transformation.

### For Responses API adapter
```json
{
  "previous_response_id": "resp_abc123xyz"
}
```

That is the entire Redis payload for this adapter. No turn history needed — OpenAI holds it server-side.

---

## When to Write to Redis

```
User message arrives
        ↓
Adapter receives SendTextInputAsync(text)
        ↓
WRITE user turn to Redis history immediately
(before making the provider call — so if the node crashes
mid-response, we at least have the user's message recorded
for the turn_count limit tracking)
        ↓
Provider call made with full history
        ↓
Streaming begins — chunks accumulate in adapter memory only
        ↓
TextDoneEvent fires — full response assembled
        ↓
WRITE assistant turn to Redis history
Update turn_count
Refresh TTL on the history key
        ↓
Check if turn_count > max_turns → trim if needed
```

---

## The 10-Turn Limit and Trimming

A "turn" = one user message + one assistant response. 10 turns = 20 messages in the array.

10 is a reasonable starting point for most conversational use cases. Make it configurable at the project level so platform users can adjust it based on their use case:

```sql
-- add to project_configs table
history_turn_limit    INT NOT NULL DEFAULT 10,   -- max conversation rounds kept
```

### Trimming strategy

When `turn_count` exceeds `history_turn_limit`, drop the **oldest user+assistant pair**:

```
Before trim (11 turns, limit is 10):
[user_1, assistant_1, user_2, assistant_2, ... user_11, assistant_11]

After trim:
[user_2, assistant_2, user_3, assistant_3, ... user_11, assistant_11]

turn_count → 10
```

Always drop in pairs — never leave a dangling user message without its assistant response or vice versa. The system prompt is untouched since it is not in this array.

This trimming happens **after writing the new assistant turn**, before the next user message arrives. The adapter handles it synchronously — no background job needed.

---

## Assembly Mechanism — How Chunks Become Meaningful Messages

The adapter already has this partially. Make it explicit:

```
Adapter internal state during a turn:
  _currentAssemblyBuffer: StringBuilder   ← accumulates delta text in memory

On each TextDeltaEvent:
  _currentAssemblyBuffer.Append(delta)
  Forward TextDeltaEvent to client via IClientTransport
  (never write delta to Redis)

On TextDoneEvent:
  fullText = _currentAssemblyBuffer.ToString()
  _currentAssemblyBuffer.Clear()
  
  Append to Redis history:
    {"role": "assistant", "content": fullText}
  
  Emit TextDoneEvent to client
```

For tool calls that happen mid-response:

```
If a tool call interrupts the response:
  Partial text so far stays in _currentAssemblyBuffer
  Tool call handled (gateway-intercepted or forwarded to client)
  Tool result returned to provider
  Provider continues streaming
  Remaining text appended to _currentAssemblyBuffer
  When final TextDoneEvent arrives: full combined text written to Redis
```

The stored assistant message is always the **complete final response** for that turn, regardless of how many tool calls or interruptions happened mid-stream.

---

## Redis Key Structure for History

Following the key convention defined earlier:

```
# Chat Completions and Gemini Generate
session:{session_id}:history
  → JSON object with turns[] array and turn_count
  → TTL = session inactivity timeout (refreshed on every write)

# Responses API
session:{session_id}:response_id  
  → plain string, the previous_response_id value
  → TTL = session inactivity timeout

# WS Bridge adapters
(no history key created at all)
```

---

## How the Adapter Uses History on Each Turn

### Chat Completions — what gets sent to OpenAI on turn N

```python
# Built at request time, never stored in this form
request_messages = [
    {"role": "system", "content": project_config.system_prompt},  # from project config
    *redis_history["turns"],                                        # from Redis
    {"role": "user", "content": current_user_message}              # just received
]

# POST to /v1/chat/completions with request_messages
# After response: append user message + assembled response to Redis history
```

### Gemini Generate — same pattern, different format

```python
request_contents = [
    *redis_history["turns"],                    # from Redis (already in Gemini format)
    {"role": "user", "parts": [{"text": current_user_message}]}
]

# system_instruction sent as separate field, not in contents[]
request_body = {
    "system_instruction": {"parts": [{"text": project_config.system_prompt}]},
    "contents": request_contents
}
```

### Responses API — minimal

```python
previous_id = redis.get(f"session:{session_id}:response_id")  # None on first turn

request_body = {
    "model": model_id,
    "input": current_user_message,
    "previous_response_id": previous_id,  # None first turn, chained after that
    "instructions": project_config.system_prompt
}

# After response: save new response_id to Redis, overwrite previous
```

---

## Teardown

At session teardown, the teardown sequence already defined in your SRS handles cleanup:

```
DEL session:{session_id}:history        # Chat Completions, Gemini Generate
DEL session:{session_id}:response_id    # Responses API
# WS Bridge adapters: nothing to delete
```

Both deletes happen regardless of whether the key exists — `DEL` on a non-existent key is a no-op in Redis.

---

## Summary of Decisions

| Decision | Value |
|---|---|
| What is stored | Assembled complete turns only — never streaming chunks |
| When written | User message at turn start, assistant message after TextDoneEvent |
| Default turn limit | 10 rounds (configurable per project via `history_turn_limit`) |
| Trimming strategy | Drop oldest user+assistant pair when limit exceeded |
| System prompt location | Project config only — never duplicated into history array |
| Format | Provider-native format per adapter (OpenAI messages[] vs Gemini contents[]) |
| Responses API | Only `previous_response_id` stored — no message history |
| WS Bridge adapters | No Redis history at all — provider holds state |
| Cleanup | Explicit DEL at teardown — not relying on TTL expiry |
| TTL | Refreshed on every write — expires if session goes inactive |

## @ranga1729

this is my current database design in draw.io xlm format. now i need you to think about every change we talked about in this conversation, every update we agreed to do. now i need you to design the final database design according to those new changes in a draw.io xml format so i can see the final DB schema design. 

## Copilot



*System: User retried message*

## Copilot



*System: User retried message*

## @ranga1729

 now can you list down all the chnages we agreed to do in a summary. 

## Copilot



*System: User retried message*

## Copilot



*System: User retried message*

## @ranga1729

can you summarize what are the database chnages we should apply so far in this conversation ?

## Copilot



## @ranga1729

hello

## Copilot



## @ranga1729

ok. now I need you to create a markdown file mentioned all the changes we dicussed in this conversation. 

## Copilot

The markdown file has been created at `FactN-Assistant/factn-assistant-service/docs/schema-change-log.md`.

---

Here is the full summary of every database change we agreed on in this conversation:

---

## Database Changes Summary

### `ephemeral_tokens` — 3 changes
| Type | Field | Detail |
|---|---|---|
| **REMOVE** | `is_used BOOL` | Replaced by two more informative fields |
| **ADD** | `scope TEXT NOT NULL DEFAULT 'session:connect'` | Token permission scope, validated at connection time |
| **ADD** | `used_at TIMESTAMPTZ` | NULL until token is consumed at connection |
| **ADD** | `outcome TEXT` | NULL → `'used'` → `'expired'`, set by service or background job |

---

### `project_configs` — 1 change
| Type | Field | Detail |
|---|---|---|
| **ADD** | `history_turn_limit INT NOT NULL DEFAULT 10` | Max conversation rounds kept in Redis per session for HTTP Orchestrator adapters |

---

### `sessions` — 7 changes
| Type | Field | Detail |
|---|---|---|
| **ADD** | `status TEXT NOT NULL DEFAULT 'validating'` | Session state machine value |
| **ADD** | `node_id TEXT` | Gateway node identifier for Redis pub/sub routing |
| **ADD** | `close_code INT` | WebSocket close code at teardown |
| **ADD** | `error_code TEXT` | Machine-readable error if session failed |
| **ADD** | `error_message TEXT` | Human-readable error description |
| **ADD** | `ephemeral_token_id UUID FK ephemeral_tokens.id` | Token used to open session, NULL if API key was used directly |
| **ADD** | `connection_type TEXT NOT NULL DEFAULT 'websocket'` | Values: `'websocket'` or `'sse'` |

---

All other tables (`users`, `projects`, `api_keys`, `tools`, `llm_providers`, `llm_provider_apis`, `llm_models`, `audit_logs`) require no schema changes based on this session's decisions.

## @ranga1729

@Copilot Accepted Confirmation: Are you sure?

## Copilot

The markdown file has been committed to [`FactN-Assistant/factn-assistant-service/docs/schema-change-log.md`](https://github.com/FactN-Assistant/factn-assistant-service/blob/main/docs/schema-change-log.md).

---

Here is the complete database change summary:

---

## Database Changes — Full Summary

### `ephemeral_tokens`
| Action | Field | Detail |
|---|---|---|
| **REMOVE** | `is_used BOOL` | Replaced by `used_at` + `outcome` |
| **ADD** | `scope TEXT NOT NULL DEFAULT 'session:connect'` | Token permission scope, validated at connection |
| **ADD** | `used_at TIMESTAMPTZ` | NULL until token consumed, set by Session Gateway |
| **ADD** | `outcome TEXT` | NULL → `'used'` or `'expired'`, background job handles expired |

---

### `project_configs`
| Action | Field | Detail |
|---|---|---|
| **ADD** | `history_turn_limit INT NOT NULL DEFAULT 10` | Max conversation rounds kept in Redis per session. HTTP Orchestrator adapters only. Trimming drops oldest complete user+assistant pair when limit exceeded. |

---

### `sessions`
| Action | Field | Detail |
|---|---|---|
| **ADD** | `status TEXT NOT NULL DEFAULT 'validating'` | State machine: `validating` → `initializing` → `active` → `responding` → `closing` → `closed` / `rejected` / `failed` |
| **ADD** | `node_id TEXT` | Gateway node ID — required for Redis pub/sub admin force-close routing |
| **ADD** | `close_code INT` | WebSocket close code at teardown (1000, 4001–4006, 4029). NULL for SSE sessions |
| **ADD** | `error_code TEXT` | Machine-readable error (e.g. `provider_unavailable`, `session_timeout`). NULL for clean closes |
| **ADD** | `error_message TEXT` | Human-readable description. NULL for clean closes |
| **ADD** | `ephemeral_token_id UUID FK ephemeral_tokens.id` | Token used to open session. NULL if API key was used directly |
| **ADD** | `connection_type TEXT NOT NULL DEFAULT 'websocket'` | Values: `'websocket'` or `'sse'` |

---

### All Other Tables — No Changes Required
`users`, `projects`, `api_keys`, `tools`, `llm_providers`, `llm_provider_apis`, `llm_models`, `audit_logs`
