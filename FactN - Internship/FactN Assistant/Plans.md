Add RAG.
Add MCP servers. 
System Prompt versioning.
Let platform users decide if they want websocket or SSE ?
Fine tune the SRS to get a clear image on what's being built. 
consider pricing. 

Instead of keeping messages in the server and flush to Redis after 30 seconds, add them directly to Redis asynchronously. at the end of the websocket, accumulate the usage details into the main DB.

**

- Prompt versioning & A/B testing: Instead of hardcoding the system prompt which requires a new deployment to test, stakeholders/developers keep multiple versions of the system prompt and apply changes on the fly. No code change required. No new deployment required. 
    
- Fallback management (high availability with multi-provider integration): This platform offers users to select a primary provider and secondary provider. If the primary provider returns 500 errors, the platform’s internal translation layer can automatically re-route fallback to the secondary provider. Ensure 0-downtime. 
    
- Comprehensive cost analysis: Users can see current cost estimation and compare it with other LLM providers/APIs and models, letting them find the best matching solution with a minimum cost.
    

**

****
# RAG Plan
### How RAG works ?
Vector DB converts words into vectors An array is a high-dimensional array of floats. Ex: "Apple" : `[0.23, -0.15, 0.89, 0.41, ...]` So the sentences will be a list of numbers(a vector). 
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
- Since `"search_internal_knowledge"` function is declared in my backend, I will intercept the tool call once it reach the backend and will never forward it to the user. 
****
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
  description TEXT,

  -- Schema
  input_schema JSONB,     -- was parameters
  output_schema JSONB,    -- YOU NEED THIS

  -- Versioning
  version TEXT,

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

```sql
tool_endpoints (
  id UUID PK,
  tool_id UUID FK,

  -- Execution
  endpoint_url TEXT,      -- where to send the request
  http_method TEXT,       -- POST / GET (usually POST)
  protocol TEXT,          -- http / websocket / mcp

  -- Auth
  auth_type TEXT,         -- none / api_key / oauth
  auth_config JSONB,      -- encrypted or referenced

  -- Status
  is_active BOOLEAN,
  is_deprecated BOOLEAN,
  health_status TEXT,     -- healthy / failing / unknown
);
```
	Becuase one tool can have multiple endpoints.
	Use these additional endpoints in the fallback mechanism. 

****
```sql
CREATE TABLE mcp_servers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id),
  name TEXT NOT NULL,
  
  -- Transport Logic
  transport_type TEXT NOT NULL, -- 'stdio', 'sse', 'http'
  config JSONB NOT NULL,         -- Stores {url: '...'} or {command: 'npx', args: [...]}
  
  status TEXT DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

```sql
CREATE TABLE tools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id),
  
  -- The AI Contract
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  input_schema JSONB NOT NULL,
  output_schema JSONB,
  
  -- Routing Logic
  routing_strategy TEXT DEFAULT 'failover', -- 'failover', 'round_robin'
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(project_id, name)
);
```

```sql
CREATE TABLE tool_endpoints (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tool_id UUID REFERENCES tools(id) ON DELETE CASCADE,
  
  -- Origin
  endpoint_type TEXT NOT NULL, -- 'mcp_server' or 'direct_http'
  mcp_server_id UUID REFERENCES mcp_servers(id), -- Null if direct_http
  
  -- Execution Data (Used if direct_http)
  url TEXT,
  method TEXT DEFAULT 'POST',
  
  -- Reliability & Environment
  environment TEXT DEFAULT 'production', -- 'dev', 'staging', 'prod'
  priority INT DEFAULT 1,                -- For failover
  is_active BOOLEAN DEFAULT true,
  health_status TEXT DEFAULT 'healthy',
  
  -- Security
  auth_config_id UUID REFERENCES tool_secrets(id),
  
  last_called_at TIMESTAMPTZ
);
```

```sql
CREATE TABLE tool_secrets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id),
  name TEXT NOT NULL,
  secret_type TEXT, -- 'api_key', 'bearer_token', 'oauth2'
  encrypted_value TEXT NOT NULL, -- Encrypted string or reference to vault
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

- the websocket gateway is the MCP host. It connects to the tenant's registered MCP servers, discovers tools, and proxies tool invocations — the End Client never communicates directly with MCP servers.
****
### 10. Critical Security Gap: SSRF Risk

MCP servers are defined by tenant-supplied URLs (`endpoint_url` in your tools table). The gateway will make outbound HTTP/WebSocket connections to these URLs. This is a **Server-Side Request Forgery (SSRF)** attack vector.

A malicious tenant could supply:

- `http://169.254.169.254/latest/meta-data/` (AWS metadata endpoint)
- `http://localhost:5432` (internal PostgreSQL)
- `http://10.0.0.1/admin` (internal network)

You need an explicit constraint added to the SRS:

- All MCP server URLs must be validated against an allowlist or blocklist of private IP ranges at registration time
- Only `https://` scheme should be permitted for production MCP servers
- Requests to RFC1918 address ranges must be rejected

****
### 13. Conflicting Tool Names — Resolution Not Defined

Your best-practices section correctly notes:

> _"Conflicting tool names"_

But you don't define how to resolve them. If a tenant registers two MCP servers and both expose a tool called `search`, what happens? The LLM receives two tools with the same name, which breaks function calling on all providers.

Define a **namespacing strategy**: prefix tool names with the MCP server's registered ID, e.g., `mcp_server_a__search`, `mcp_server_b__search`. This must be applied consistently at discovery time and reversed when routing the actual tool invocation.

