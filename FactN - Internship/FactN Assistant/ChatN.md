# ChatN Platform — Session Memory

> **Purpose**: This document captures all architectural decisions, code implementations, dependency registrations, and troubleshooting fixes from the development session. Use this as context when starting a new conversation about the ChatN platform.

---

## 1. Project Structure (Monorepo)

The project is organized as a single solution with multiple projects:

```
ChatN/
├── ChatN.sln
├── src/
│   ├── ChatN.Core/               # Shared entities, enums, interfaces, events
│   ├── ChatN.Infrastructure/     # DbContext, Migrations, Redis, KeyVault, Repositories
│   ├── ChatN.Application/        # CQRS Handlers, Validators (RestApi only)
│   ├── ChatN.RestApi/            # Host #1 - REST Controllers (Port: 7055/5116)
│   ├── ChatN.Gateway/            # Host #2 - WebSocket/SSE (Port: 5002)
│   └── ChatN.ApiGateway/         # Host #3 - YARP Reverse Proxy (Port: 5000)
├── tests/
└── docker-compose.yml
```

**Key Decision**: Single monorepo with shared `Core` and `Infrastructure` projects eliminates cross-repo DLL hell. Migrations are run **only** from RestApi; Gateway consumes the same `AppDbContext`.

---

## 2. Infrastructure Setup

### 2.1 PostgreSQL (Shared DbContext)
- `AppDbContext` in `Infrastructure` project
- Migrations assembly: `Infrastructure`
- Connection string from Key Vault: `SecretNames.DBConnection`

### 2.2 Redis (Shared Cache and Pub/Sub)
- Upstash Redis with TLS endpoint (`rediss://...`)
- Registered in **both** RestApi and Gateway:
  ```csharp
  builder.Services.AddStackExchangeRedisCache(options =>
  {
      options.Configuration = builder.Configuration[SecretNames.RedisConnection];
      options.InstanceName = "ChatN";
  });
  builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
      ConnectionMultiplexer.Connect(builder.Configuration[SecretNames.RedisConnection]));
  ```
- Key patterns:
  - `project:{projectId}:config` — Project configuration cache
  - `apikey:{keyHash}` — API key cache
  - `token:{tokenHash}` — Ephemeral token (GETDEL on use)
  - `session:{sessionId}:*` — Session state and history
  - `ratelimit:ip:{ip}` — Rate limiting sorted set

### 2.3 Azure Key Vault
- Accessed via `DefaultAzureCredential`
- Secrets retrieved by name using `SecretNames` constants
- Registered as `ISingleton` for Gateway:
  ```csharp
  builder.Services.AddSingleton<ISecretProvider, AzureKeyVaultSecretProvider>();
  ```

### 2.4 API Gateway (YARP)
- Routes `/api/**` → RestApi (Port 7055)
- Routes `/ws/**` and `/sse/**` → Gateway (Port 5002)
- YARP cluster configuration in `appsettings.json`

---

## 3. Session Gateway Architecture

### 3.1 Core Components (Phase 4)

| Component | File | Responsibility |
|-----------|------|----------------|
| `LLMEvent` | `Core/Events/` | Base record for all events; concrete types: `TextDeltaEvent`, `AudioDeltaEvent`, `TranscriptEvent`, `ToolCallEvent`, `UsageEvent`, `TurnCompleteEvent`, `ProviderDisconnectedEvent` |
| `IClientTransport` | `Gateway/Transport/` | Abstraction over WebSocket/SSE transports |
| `WebSocketClientTransport` | `Gateway/Transport/` | Wraps ASP.NET Core WebSocket; enforces 1MB text / 256KB audio limits |
| `SseClientTransport` | `Gateway/Transport/` | SSE with heartbeat (15s); inbound via Redis pub/sub `session:{id}:inbox` |
| `EventDispatcher` | `Gateway/Events/` | Bounded channels (input:256, A:128, B:128, C:64); fans events to three consumers |
| `AdapterFeedLoop` | `Gateway/Events/` | Reads `IAsyncEnumerable<LLMEvent>` → writes to `EventDispatcher.InputWriter`; replaced during provider fallback |
| `ConversationAccumulator` | `Gateway/Accumulators/` | Maintains `TurnState`; flushes to Redis on `TurnCompleteEvent`; discards partial turns on `ProviderDisconnectedEvent` |
| `UsageAccumulator` | `Gateway/Accumulators/` | Increments Redis hash `session:{id}:usage` on `UsageEvent` and `TurnCompleteEvent` |
| `ProtocolHandler` | `Gateway/Protocol/` | Two loops: Client→Provider and Provider→Client; tool interception (RAG, MCP, manual) |
| `ConnectionManager` | `Gateway/ConnectionManager/` | Full session lifecycle: auth, init, state machine, fallback, teardown |

### 3.2 LLM Adapters (Phase 5)

**Base Classes**:
- `HttpOrchestratorAdapterBase` — Template method for HTTP/SSE adapters
- `WebSocketBridgeAdapterBase` — Template method for WebSocket adapters

**Concrete Adapters**:

| Adapter | Type | Endpoint | Key Features |
|---------|------|----------|--------------|
| `OpenAIChatCompletionsAdapter` | HTTP | `POST /v1/chat/completions` | SSE with `[DONE]`; history from `session:{id}:history` |
| `OpenAIResponsesAdapter` | HTTP | `POST /v1/responses` | Uses `previous_response_id` chaining; event-based SSE |
| `GeminiGenerateContentAdapter` | HTTP | `POST /v1/models/{model}:streamGenerateContent` | Newline-delimited JSON; `systemInstruction` field |
| `OpenAIRealtimeAdapter` | WS | `wss://api.openai.com/v1/realtime` | `session.update`, `response.*` events; watchdog timer |
| `GeminiLiveAdapter` | WS | `wss://generativelanguage.googleapis.com/ws/...` | `setup` message, `serverContent`; `goAway` handling |

**AdapterFactory**:
- Maps `adapter_key` → concrete type via `Dictionary<string, Type>`
- Registered as `Singleton`

### 3.3 History Formatters

| Formatter | Adapter Key | Format |
|-----------|-------------|--------|
| `ChatCompletionsHistoryFormatter` | `openai-chat-completions`, `openai-realtime-api` | `{"turns": [{"role":"user","content":""}, ...], "turn_count": N}` |
| `GeminiGenerateHistoryFormatter` | `gemini-generate-content`, `gemini-live-api` | `{"turns": [{"role":"user","parts":[{"text":""}]}, ...], "turn_count": N}` |
| `ResponsesApiHistoryFormatter` | `openai-responses-api` | Adds `previous_response_id` field |

---

## 4. ConnectionManager — Key Implementation Details

### 4.1 Constructor (Dependencies)
```csharp
public ConnectionManager(
    IConnectionMultiplexer redis,
    IServiceProvider serviceProvider,  // For scoped repository resolution
    ILogger<ConnectionManager> logger,
    ISecretProvider secretProvider,
    AdapterFactory adapterFactory,
    IHistoryFormatterFactory historyFormatterFactory,
    IClientTransportFactory transportFactory,
    IHttpContextAccessor httpContextAccessor)
```

### 4.2 Session Initialization Sequence
1. Load project config from Redis → fallback to DB via `_projectRepository.GetProjectWithConfigAsync()`
2. Load active system prompt via `_promptRepository.GetActivePromptAsync()`
3. Retrieve LLM credential from Key Vault: `llm-credential-{ProviderApiId}` or `llm-credential-{AdapterKey}`
4. Resolve tool list (manual + RAG builtin + MCP from Redis cache)
5. Create adapter via `AdapterFactory.Create()`
6. Instantiate `EventDispatcher`, `ConversationAccumulator`, `UsageAccumulator`
7. Create and start `AdapterFeedLoop`
8. Create `ProtocolHandler`
9. Create session record in PostgreSQL
10. Register session in Redis hash
11. Send `session_started` to client

### 4.3 Teardown Sequence (Strict Order)
1. Stop `AdapterFeedLoop`
2. Close `EventDispatcher` input channel and wait for drain
3. Wait for accumulators to drain
4. Disconnect adapter
5. Flush usage to PostgreSQL: `_sessionRepository.UpdateSessionUsageAsync()`
6. Send `session_ended` to client
7. Delete Redis keys: `session:{id}`, `session:{id}:history`, `session:{id}:history:unified`, `session:{id}:usage`, `session:{id}:response_id`
8. Dispose components

### 4.4 Provider Fallback
- Triggered by `ProviderDisconnectedEvent` from `AdapterFeedLoop`
- Loads secondary config (or retries primary)
- Reads history from `session:{id}:history:unified`
- Creates new adapter and `AdapterFeedLoop`
- Sends `provider_reconnected` to client

### 4.5 Admin Commands
- Subscribes to Redis pub/sub channel `admin:commands`
- On `force_close` command → executes teardown within 5 seconds

---

## 5. HttpOrchestratorAdapterBase — Fixes Applied

### 5.1 Missing Fields Added
```csharp
protected string _systemPrompt = string.Empty;
protected IReadOnlyList<ToolDefinition> _tools = Array.Empty<ToolDefinition>();
protected ConversationHistory? _history;
```

### 5.2 SendRequestAsync — Made `protected virtual`
```csharp
protected virtual async Task SendRequestAsync(string? userText, IReadOnlyList<ToolDefinition>? tools, CancellationToken ct)
```

### 5.3 IDisposable Implementation
- `_streamCts` disposed in `Dispose()` and `DisconnectAsync()`

### 5.4 ParseSseStreamAsync — Cancellation Handling
- `OperationCanceledException` caught separately (no error log)

---

## 6. Redis Caching Implementation (RestApi)

### 6.1 Project Config Cache
- Key pattern: `project:{projectId}:config`
- TTL: 10 minutes
- Cache-aside pattern in `GetProjectByIdHandler`

### 6.2 Cache Invalidation
- On `UpdateProject`, `DeleteProject`, `UpdateProjectConfig` → remove cache key

### 6.3 API Key Cache
- Key pattern: `apikey:{keyHash}`
- Populated on first successful validation; invalidated on revoke

---

## 7. Redis Rate Limiting (RestApi)

### 7.1 Implementation
- **Sliding window** using Redis sorted sets
- Lua script for atomic operations
- **Fallback**: Allow on Redis errors (fail-open)

### 7.2 Limits
| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Authenticated users | 300 | 1 minute |
| Unauthenticated (login/register) | 10 | 1 minute |

### 7.3 Middleware
- Checks IP from `HttpContext.Connection.RemoteIpAddress`
- Returns 429 with `Retry-After: 60` header on breach
- Skips `/health` endpoints

---

## 8. Program.cs — Gateway Service Registrations

```csharp
// Infrastructure
builder.Services.AddDbContext<AppDbContext>(options => ...);
builder.Services.AddSingleton<IConnectionMultiplexer>(...);
builder.Services.AddSingleton<ISecretProvider, AzureKeyVaultSecretProvider>();

// Transport
builder.Services.AddSingleton<IClientTransportFactory, ClientTransportFactory>();

// Gateway components
builder.Services.AddSingleton<IHistoryFormatterFactory, HistoryFormatterFactory>();
builder.Services.AddScoped<IToolExecutor, StubToolExecutor>();

// HTTP Client for adapters
builder.Services.AddHttpClient("LLMProvider")
    .ConfigureHttpClient(client => client.Timeout = TimeSpan.FromSeconds(30));

// Adapter Factory
builder.Services.AddSingleton<AdapterFactory>();

// HttpContextAccessor
builder.Services.AddHttpContextAccessor();

// Repositories (scoped)
builder.Services.AddScoped<IProjectRepository, ProjectRepository>();
builder.Services.AddScoped<IProviderRepository, ProviderRepository>();
builder.Services.AddScoped<IPromptRepository, PromptRepository>();
builder.Services.AddScoped<IToolRepository, ToolRepository>();
builder.Services.AddScoped<ISessionRepository, SessionRepository>();

// ConnectionManager (singleton)
builder.Services.AddSingleton<ConnectionManager>();

// Controllers
builder.Services.AddControllers();
```

---

## 9. Key Vault Secret Naming

### 9.1 Recommendation
Use **adapter_key** based names for simplicity:
- `llm-credential-openai-chat-completions`
- `llm-credential-openai-responses-api`
- `llm-credential-openai-realtime-api`
- `llm-credential-gemini-generate-content`
- `llm-credential-gemini-live-api`

### 9.2 Code Change Required
In `InitializeSessionAsync`:
```csharp
// Use AdapterKey instead of ProviderApiId for human-readable secret names
var credential = await _secretProvider.GetSecretAsync($"llm-credential-{providerConfig.AdapterKey}");
```

### 9.3 Alternative (Current)
Using `ProviderApiId` GUID — requires querying the database to know secret names.

---

## 10. Port Configuration

| Service | HTTP Port | HTTPS Port |
|---------|-----------|------------|
| RestApi | 5116 | 7055 |
| Gateway | 5002 | 5002 |
| ApiGateway | — | 5000 |

**ApiGateway Cluster Configuration**:
```json
"Clusters": {
  "rest-cluster": { "Destinations": { "rest": { "Address": "https://localhost:7055/" } } },
  "gateway-cluster": { "Destinations": { "gateway": { "Address": "https://localhost:5002/" } } }
}
```

---

## 11. Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `FileNotFoundException: Microsoft.EntityFrameworkCore.Relational` | Version mismatch; Gateway had direct EF Core package | Remove direct EF references; rely on transitive from Infrastructure |
| `Cannot consume scoped service from singleton` | `ConnectionManager` singleton depends on scoped repositories | Use `IServiceProvider.CreateScope()` inside methods |
| `The name '_systemPrompt' does not exist` | Missing fields in base class | Added `_systemPrompt`, `_tools`, `_history` to `HttpOrchestratorAdapterBase` |
| `CS0115: no suitable method found to override` | `SendRequestAsync` was `private` | Changed to `protected virtual` |
| `ProviderConfig` vs `ProjectProviderConfig` type mismatch | `SessionContext.ProviderConfig` expected entity, not DTO | Changed to use `ProviderConfig` DTO consistently |

---

## 12. Next Steps

1. **Implement remaining concrete adapters** — fill `BuildRequest`, `ParseChunk`, `ParseUsage` for all 5 adapters
2. **Implement RAG** — replace `StubToolExecutor` with real Azure AI Search integration
3. **Implement MCP** — replace `StubToolExecutor` with real MCP orchestrator
4. **Implement Session Events** — log fallback events to `session_events` table
5. **Add admin endpoints** — session monitoring, usage reporting, force-close
6. **Frontend** — build React dashboard with shadcn/ui

---

## 13. Files Referenced in This Session

| File | Purpose |
|------|---------|
| `ChatN-SRS(Tech).md` | Full Software Requirements Specification |
| `Development Plan.md` | Phased development roadmap |
| `chatn_schema.sql` | PostgreSQL schema definition |
| `Program.cs` (RestApi) | DI container, middleware, authentication |
| `Program.cs` (Gateway) | DI container, WebSocket/SSE endpoints |
| `ConnectionManager.cs` | Full session lifecycle |
| `HttpOrchestratorAdapterBase.cs` | Base class for HTTP adapters |
| `WebSocketBridgeAdapterBase.cs` | Base class for WebSocket adapters |
| `OpenAIChatCompletionsAdapter.cs` | OpenAI Chat Completions implementation |
| `OpenAIResponsesAdapter.cs` | OpenAI Responses API implementation |
| `GeminiGenerateContentAdapter.cs` | Google Gemini Generate Content implementation |
| `OpenAIRealtimeAdapter.cs` | OpenAI Realtime API implementation |
| `GeminiLiveAdapter.cs` | Google Gemini Live API implementation |
| `EventDispatcher.cs` | Fan-out event routing |
| `ConversationAccumulator.cs` | History persistence |
| `UsageAccumulator.cs` | Usage counter tracking |
| `ProtocolHandler.cs` | Bidirectional message routing |
| `IProjectConfigCache.cs` | Redis caching for project configs |
| `RedisSlidingWindowRateLimiter.cs` | Redis-based rate limiting |

---

## 14. Key Links

- [SRS Document](https://drive.google.com/file/d/1AhFxv88xfjXd1LmBxUrTrbsrFvyBf39P/view)
- [Development Plan](./Development Plan.md) (attached)
- [Database Schema](https://dbdiagram.io/d/ChatN-69f3c678c6a36f9c1bd287f9)
- [OpenAI Realtime API](https://platform.openai.com/docs/api-reference/realtime)
- [Gemini Live API](https://ai.google.dev/api/live)