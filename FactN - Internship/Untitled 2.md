Software Requirements Specification
Multi-Tenant AI API Gateway Platform - Technical Edition

________________

Table of Contents
1. System Overview
2. Functional & Non-Functional Requirements
3. Technology Stack
4. Dependencies
5. System Constraints
6. System Architecture
7. Session Gateway — Technical Specification
8. LLM Adapter Layer
9. RAG (Retrieval-Augmented Generation) Architecture
10. MCP (Model Context Protocol) Architecture
11. Authentication and Security Architecture
12. Data Architecture
13. Use Cases
14. External Interface Specifications
15. Protocol Reference
16. Cost Estimation
17. Pricing
18. Competitor Analysis

________________

1. System Overview

[UNCHANGED — 1.1 through 1.4 remain exactly as in the original SRS]

1.5 Terminology

| Term                  | Definition                                                                                                                                                                                                                                                                                                                        |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Platform              | The Multi-Tenant AI API Gateway system described in this document                                                                                                                                                                                                                                                                 |
| Platform Organization | The company that owns and operates the Platform                                                                                                                                                                                                                                                                                   |
| Platform User         | A developer or organization registered on the Platform to build AI-powered applications                                                                                                                                                                                                                                           |
| End Client            | The final human or system connecting through the Platform's Session Gateway on behalf of a Platform User's downstream application                                                                                                                                                                                                 |
| Project               | A logical, isolated unit created by a Platform User representing one application configuration                                                                                                                                                                                                                                    |
| Session               | A single active connection (WebSocket or SSE) between an End Client and the Session Gateway. A session is scoped to the client-side connection lifetime. It persists across provider-side reconnections and fallbacks.                                                                                                            |
| Provider              | An external LLM service provider (e.g., OpenAI, Google)                                                                                                                                                                                                                                                                           |
| Provider Credentials  | API keys held exclusively by the Platform Organization to authenticate with LLM providers                                                                                                                                                                                                                                         |
| API Type              | A specific API offered by a provider (e.g., Chat Completions, Realtime API, Gemini Live)                                                                                                                                                                                                                                          |
| Adapter               | A backend component that translates between the Platform's unified protocol and a specific Provider API. Pure stateless normalizer — no buffering, no history management.                                                                                                                                                         |
| Adapter Key           | A string in the provider catalog that maps to a concrete adapter implementation in code                                                                                                                                                                                                                                           |
| AdapterFeedLoop       | A thin, lightweight per-provider-connection background loop that reads from a specific adapter's IAsyncEnumerable LLMEvent output and writes each event into the EventDispatcher's internal input channel. It is the only component replaced during a provider fallback. One instance exists per active provider-side connection. |
|                       |                                                                                                                                                                                                                                                                                                                                   |
WSS | WebSocket Secure — encrypted, persistent, bidirectional WebSocket protocol
SSE | Server-Sent Events — HTTP-based unidirectional streaming protocol
VAD | Voice Activity Detection
PCM | Pulse-Code Modulation — raw uncompressed audio format
JWT | JSON Web Token
WS Bridge Adapter | An adapter that maintains a second WebSocket connection to a provider and proxies messages bidirectionally
HTTP Orchestrator Adapter | An adapter that makes HTTP REST/SSE requests to a provider per conversation turn
LLM Event | An internal normalized event produced by all adapters in a unified format. See Section 8.1 for the complete LLMEvent contract.
EventDispatcher | A per-client-session component that owns an internal bounded Channel LLMEvent as its input and fans every event out to three independent bounded output channels: Channel A (Protocol Handler), Channel B (ConversationAccumulator), Channel C (UsageAccumulator). Persists for the entire client session lifetime, surviving provider reconnections.
ConversationAccumulator | A per-client-session background component that reads LLMEvents from Channel B, accumulates text chunks and transcripts into complete turns, and writes fully assembled conversation history to Redis asynchronously. Persists for the entire client session lifetime. Replaces the former "delta assembler" responsibility.
UsageAccumulator | A per-client-session background component that reads UsageEvents from Channel C and atomically increments usage counters in Redis. Persists for the entire client session lifetime.
IHistoryFormatter | An interface injected into ConversationAccumulator at session start. Each adapter type provides its own implementation. Formats assembled turn data into the provider-specific Redis structure without ConversationAccumulator needing any provider knowledge.
IClientTransport | The interface abstracting the client-facing connection transport (WebSocket or SSE) from the rest of the gateway
TurnCompleteEvent | The single universal end-of-turn signal emitted by the adapter when the provider's end-of-stream marker arrives. Replaces both TextDoneEvent and AudioDoneEvent. Carries no payload.
ProviderDisconnectedEvent | An LLMEvent emitted by the adapter when the provider-side connection drops unexpectedly. Triggers the provider fallback mechanism. Distinct from session end.
UsageEvent | An LLMEvent emitted by the adapter when it parses usage metadata from the provider's native response stream. Carries input_tokens, output_tokens, audio_in_secs, audio_out_secs.
Tenant | A Platform User account
RAG | Retrieval-Augmented Generation
MCP | Model Context Protocol
MCP Server | A service registered by a Platform User that exposes tools the LLM can invoke via the MCP protocol
System Prompt | A project-level instruction injected into every LLM session at connect time
Subscription Plan | A tier-based plan that governs the feature set and resource limits available to a Platform User


________________

2. Functional & Non-Functional Requirements
[UNCHANGED]

3. Technology Stack
[UNCHANGED]

4. Dependencies
[UNCHANGED]

5. System Constraints
[UNCHANGED]

________________

6. System Architecture

6.1 High-Level Architecture
[UNCHANGED]

6.2 Component Decomposition - REST API Service
[UNCHANGED]

6.3 Component Decomposition — Session Gateway Service

The Session Gateway Service is composed of the following internal layers and per-session components:

**1. Connection Manager**
Handles authentication and session registry. On each inbound connection request (WebSocket upgrade or SSE GET), it validates the ephemeral token (atomic Redis GETDEL), enforces project session limits, loads the project configuration from Redis cache (DB fallback), retrieves credentials from Key Vault, resolves the full tool list (manual tools + RAG builtin + MCP-discovered tools), instantiates the correct ILLMSessionAdapter via the Adapter Factory, and instantiates the correct IClientTransport via the Transport Factory.

At session start, the Connection Manager also instantiates all three per-client-session background components — EventDispatcher, ConversationAccumulator (with the correct IHistoryFormatter injected based on adapter type), and UsageAccumulator — and creates the initial AdapterFeedLoop for the first adapter instance. It registers the session metadata hash in Redis (session_id, project_id, user_id, adapter_type, model_id, node_id, started_at, transport, system_prompt_id).

It drives the session state machine and coordinates graceful teardown. All session state transitions are persisted atomically in Redis. It listens on the admin:commands Redis pub/sub channel throughout the session lifetime for admin force-close commands targeting this session, executing the teardown sequence within 5 seconds of receiving such a command. It owns the provider fallback sequence, triggered by a direct lifecycle signal from the AdapterFeedLoop when the adapter's output stream terminates unexpectedly — this signal is separate from the LLMEvent channel.

After both the client-side transport AND the provider-side adapter.ConnectAsync() are confirmed ready, the Connection Manager sends {"type":"session_started","session_id":"uuid"} to the End Client. This is a Connection Manager lifecycle action — not an adapter LLMEvent.

**2. Transport Handler (IClientTransport)**
Abstracts the client-facing connection transport from all upstream logic. Two implementations exist:
- WebSocketClientTransport — wraps the ASP.NET Core WebSocket; reads frames and writes events using the WebSocket framing protocol.
- SseClientTransport — wraps the long-lived SSE GET response for outbound events; routes inbound messages arriving via POST /sse/messages through Redis pub/sub to the owning session.

Everything above this layer interacts exclusively with IClientTransport and is transport-agnostic. Refer to Section 7.3 for Transport Capability Enforcement.

**3. Protocol Handler**
Handles all message-level routing logic. It holds a direct reference to ILLMSessionAdapter for outbound method calls (Client→Provider direction). For the Provider→Client direction, it reads from Channel A only — it does NOT read from the adapter's IAsyncEnumerable directly; the EventDispatcher is the sole consumer of the adapter's output stream.

Responsibilities:
- Reads inbound client messages via IClientTransport.ReadMessageAsync()
- Validates each message against the unified protocol schema
- Routes valid messages to the correct adapter method (SendTextInputAsync, SendAudioChunkAsync, SendToolResultAsync, SendInterruptAsync)
- When processing a text_input message: calls ConversationAccumulator.RecordUserTurn(text) first, then calls adapter.SendTextInputAsync(text). This is the only direct coupling between the Protocol Handler and ConversationAccumulator.
- Inspects LLMEvents received from Channel A and intercepts gateway-resolved tool calls (RAG search_internal_knowledge, MCP-proxied tools) before they reach the End Client
- Serializes LLMEvents to the client-facing JSON format and writes them via IClientTransport.SendEventAsync()
- Forwards all events to the End Client immediately — no text buffering, no assembly. The Protocol Handler is not responsible for conversation history.
- On receiving ProviderDisconnectedEvent from Channel A: sends {"type":"provider_reconnecting"} to the End Client and notifies the Connection Manager via a direct lifecycle signal to begin the fallback sequence.

Two concurrent async loops run in the active phase: a client-to-provider loop and a provider-to-client loop. See Section 7.5.

**4. EventDispatcher**
A per-client-session component instantiated once by the Connection Manager at session start. It persists for the entire lifetime of the client-side connection — it is NOT recreated during provider fallbacks.

The EventDispatcher owns an internal bounded Channel LLMEvent as its input. It runs a single background loop reading from this internal channel and writing every event to three independent bounded output channels:
- Channel A → Protocol Handler
- Channel B → ConversationAccumulator  
- Channel C → UsageAccumulator

The EventDispatcher never reads from any adapter's IAsyncEnumerable directly. It has no knowledge of which adapter is currently active. The AdapterFeedLoop bridges the gap between a specific adapter's output and the EventDispatcher's internal input channel.

**5. AdapterFeedLoop**
A thin, lightweight per-provider-connection background loop. Its only responsibility is to read from one adapter's IAsyncEnumerable LLMEvent and write each event into the EventDispatcher's internal input channel. It is the only component replaced when a provider fails and a new adapter is instantiated. On provider failure, the old AdapterFeedLoop exits when the adapter's IAsyncEnumerable completes. A new AdapterFeedLoop is created for the new adapter and begins writing into the same EventDispatcher's internal channel.

**6. ConversationAccumulator**
A per-client-session background component instantiated once by the Connection Manager at session start with an injected IHistoryFormatter. It persists for the entire lifetime of the client-side connection — it is NOT recreated during provider fallbacks.

Reads LLMEvents from Channel B and maintains a per-turn TurnState in memory. On TurnCompleteEvent, it assembles the full turn text and writes to both Redis history keys asynchronously (fire-and-forget, non-blocking). On ProviderDisconnectedEvent, it discards the current in-progress TurnState without writing to Redis — partial turns are never persisted, as they would corrupt history used for fallback context. All previously completed and flushed turns remain intact.

See Section 7.6 for full detail on TurnState, flush rules, IHistoryFormatter, and dual Redis key strategy.

**7. UsageAccumulator**
A per-client-session background component instantiated once by the Connection Manager at session start. It persists for the entire lifetime of the client-side connection — it is NOT recreated during provider fallbacks.

Reads UsageEvents from Channel C and atomically increments counters in Redis at session:{id}:usage using HINCRBY/HINCRBYFLOAT. On TurnCompleteEvent, increments turn_count. Because all writes are atomic and immediate, no in-memory state is held — usage data accumulated across all provider-side connections within the same client session accumulates continuously without interruption or reset.

**8. Adapter Factory**
Maps the adapter_key stored in the provider catalog to a concrete ILLMSessionAdapter implementation. Adding a new provider requires only a new adapter class and a database catalog insert — no changes to the session manager, protocol handler, or any existing adapter.

**Component Lifetime Summary**

| Component               | Scoped To                | Recreated on Provider Fallback?     |
| ----------------------- | ------------------------ | ----------------------------------- |
| Connection Manager      | Client session           | No                                  |
| IClientTransport        | Client session           | No                                  |
| Protocol Handler        | Client session           | No                                  |
| EventDispatcher         | Client session           | No                                  |
| ConversationAccumulator | Client session           | No                                  |
| UsageAccumulator        | Client session           | No                                  |
| AdapterFeedLoop         | Provider-side connection | Yes — new instance for each adapter |
| ILLMSessionAdapter      | Provider-side connection | Yes — new instance for each adapter |

6.4 Session State Machine
[UNCHANGED]

6.5 Design Patterns

| Pattern             | Applied To                                                   | Purpose                                                                                               |
| ------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| Adapter             | ILLMSessionAdapter interface                                 | Normalize all five provider APIs behind one interface                                                 |
| Factory             | LLMAdapterFactory, ClientTransportFactory                    | Select and instantiate the correct adapter and transport from project config and request type         |
| Template Method     | WebSocketBridgeAdapter, HttpOrchestratorAdapter base classes | Share common lifecycle logic; hook provider-specific steps                                            |
| State Machine       | LLMSession                                                   | Enforce valid session lifecycle transitions                                                           |
| Strategy            | Adapter registry (adapter_key → Type)                        | Swap provider implementations without changing calling code                                           |
| Observer / Pipeline | EventDispatcher + Channel A/B/C + AdapterFeedLoop            | Fan out LLMEvent stream from single adapter output to multiple independent consumers without coupling |
| Repository          | All database access                                          | Abstract data access behind interface; enforce tenant scope                                           |
| Interceptor         | Protocol Handler tool call routing                           | Intercept gateway-resolved tool calls before forwarding to End Client                                 |
| Strategy            | IHistoryFormatter per adapter type                           | Provider-agnostic history assembly in ConversationAccumulator via injected formatter                  |

________________

7. Session Gateway — Technical Specification

7.1 Connection Endpoints
[UNCHANGED]

7.2 Pre-Connection Authentication
[UNCHANGED]

7.3 Transport Capability Enforcement
[UNCHANGED]

7.4 Session Initialization Sequence

After successful pre-connection authentication:

1. Load the project configuration from Redis cache; fall back to PostgreSQL and populate cache on a miss.
2. Load the active system prompt for the project (the system_prompts row where is_active = true). Record its system_prompt_id for the session record.
3. Retrieve the LLM provider credential from Key Vault (with up to 5-minute in-process cache).
4. If rag_enabled = true on the project config: append the search_internal_knowledge builtin tool to the tool list.
5. Discover MCP tools from the Redis MCP tool cache for any active MCP servers registered to this project; append them (namespaced) to the tool list.
6. Invoke AdapterFactory.Create(config) to instantiate the correct ILLMSessionAdapter.
7. Call adapter.ConnectAsync(), injecting the active system prompt and the full tool list into the provider session.
8. Instantiate the EventDispatcher (with its internal input channel and three output channels).
9. Instantiate the ConversationAccumulator with the correct IHistoryFormatter for the adapter type.
10. Instantiate the UsageAccumulator.
11. Create and start the AdapterFeedLoop for the instantiated adapter.
12. Register the session in Redis: session_id, project_id, user_id, adapter_type, model_id, node_id, started_at, transport ("websocket" or "sse"), system_prompt_id.
13. Send {"type":"session_started","session_id":"uuid"} to the End Client. This is sent only after both the client-side transport and provider-side adapter.ConnectAsync() are confirmed ready.

Any failure in steps 2–7 results in {"type":"error","code":"provider_unavailable"} and close code 4002 (WebSocket) or equivalent SSE error event — the connection has already been established at this point but the session is torn down immediately.

Provider fallback for mid-session failures is handled separately — see Section 7.14.

7.5 Active Session — Parallel Processing Model

Once a session is in the ACTIVE state, four concurrent background operations run simultaneously and independently:

**Loop 1 — Client→Provider (Protocol Handler)**
Reads inbound messages via IClientTransport.ReadMessageAsync(), validates each message's JSON schema, and routes to the appropriate adapter method based on the type field:
- text_input → calls ConversationAccumulator.RecordUserTurn(text) first, then calls adapter.SendTextInputAsync(text)
- audio_input → calls adapter.SendAudioChunkAsync(pcmBase64, sampleRate) directly. ConversationAccumulator will capture the user query from TranscriptEvent{Role:"user"} when transcription is enabled.
- tool_result → calls adapter.SendToolResultAsync(toolCallId, result)
- interrupt → calls adapter.SendInterruptAsync()
- ping → responds with pong directly via IClientTransport, no adapter involvement

Invalid messages return {"type":"error","code":"invalid_message"} without closing the session. Messages of types unsupported by the configured API type return {"type":"error","code":"feature_not_supported"}.

**Loop 2 — EventDispatcher Loop**
The AdapterFeedLoop reads from the active adapter's IAsyncEnumerable LLMEvent and writes each event into the EventDispatcher's internal bounded input channel. The EventDispatcher reads from its internal channel and fans every event out to Channel A, Channel B, and Channel C in sequence. No business logic runs here — pure relay.

**Loop 3 — Provider→Client (Protocol Handler reading Channel A)**
The Protocol Handler reads LLMEvents from Channel A and for each event:
- TextDeltaEvent: immediately serialize as text_delta and forward to End Client
- AudioDeltaEvent: immediately serialize as audio_delta and forward to End Client. Audio is never buffered — always forwarded immediately regardless of any other processing.
- TranscriptEvent: immediately serialize as transcript and forward to End Client
- ToolCallEvent: route to tool interception logic (see below)
- TurnCompleteEvent: serialize as turn_complete and forward to End Client. Also serialize text_done with the full assembled text (see note below).
- UsageEvent: do NOT forward to End Client
- ProviderDisconnectedEvent: send {"type":"provider_reconnecting"} to End Client; notify Connection Manager via direct lifecycle signal

Tool interception logic (for ToolCallEvents):
- If tool name is search_internal_knowledge (builtin RAG): intercept, execute vector search against Azure AI Search with mandatory project_id filter, return result via adapter.SendToolResultAsync(), emit {"type":"tool_invocation","source":"builtin","tool_name":"search_internal_knowledge"} and {"type":"rag_search","query":"...","chunks_retrieved":N} to End Client. Raw tool_call never forwarded.
- If tool name is MCP-namespaced (mcp_{serverId}__{toolName}): intercept, execute via MCP orchestrator, return result via adapter.SendToolResultAsync(), emit {"type":"tool_invocation","source":"gateway","tool_name":"..."} to End Client. Raw tool_call never forwarded.
- All other tools (source='manual'): emit {"type":"tool_invocation","source":"client","tool_name":"..."} first, then forward full tool_call event to End Client. Await tool_result message from client (timeout: 30 seconds). Forward result to adapter via SendToolResultAsync(). Session closed on timeout.

Note on text_done: The Protocol Handler maintains a lightweight in-memory StringBuilder as it forwards TextDeltaEvents to the End Client. On TurnCompleteEvent, it uses this buffer to emit text_done with the assembled text to the End Client as a convenience marker, then clears the buffer. This is distinct from ConversationAccumulator's history assembly and must not be confused with it. The Protocol Handler's buffer exists solely to produce the client-facing text_done event.

**Loop 4 — ConversationAccumulator background loop (reading Channel B)**
See Section 7.6.

**Loop 5 — UsageAccumulator background loop (reading Channel C)**
Reads events from Channel C. On UsageEvent: atomically increments session:{id}:usage in Redis (HINCRBY input_tokens, HINCRBY output_tokens, HINCRBYFLOAT audio_in_secs, HINCRBYFLOAT audio_out_secs). On TurnCompleteEvent: HINCRBY turn_count 1. All other events are ignored.

All loops run concurrently and independently. No loop blocks another.

7.6 Conversation History Management

**Ownership**
Conversation history is owned exclusively by the ConversationAccumulator. The adapter layer has no history awareness. The Protocol Handler has no history awareness. All history writes happen asynchronously off the critical client-delivery path.

**ConversationAccumulator — TurnState**

The ConversationAccumulator maintains a per-turn in-memory TurnState object:

```
TurnState:
  UserText:             string?       — set by RecordUserTurn(text) for text_input turns,
                                        or by TranscriptEvent{Role:"user"} for audio turns.
                                        If transcription is disabled and turn is audio-only, remains null.
  AssistantTextParts:   List<string>  — appended on every TextDeltaEvent.
                                        Accumulates across tool cycles. Never reset on ToolCallEvent.
  AssistantTranscript:  string?       — set by TranscriptEvent{Role:"assistant"} when transcription enabled.
```

**What ConversationAccumulator does per LLMEvent received from Channel B:**

| Event | Action |
|---|---|
| TextDeltaEvent | Append Text to AssistantTextParts |
| TranscriptEvent {Role:"user"} | Set UserText (only if not already set by RecordUserTurn) |
| TranscriptEvent {Role:"assistant"} | Set AssistantTranscript |
| TurnCompleteEvent | Assemble and flush — see flush sequence below |
| ToolCallEvent | Ignore — TurnState is NOT reset. Tool cycles are mid-turn. |
| AudioDeltaEvent | Ignore entirely |
| UsageEvent | Ignore entirely |
| ProviderDisconnectedEvent | Discard current TurnState without flushing to Redis. Reset TurnState. All previously completed and flushed turns remain intact. |

**Answer resolution precedence (applied at flush time):**
- If AssistantTranscript is present → use it as the answer. Transcript is the authoritative text for audio turns.
- Otherwise → join all AssistantTextParts as the answer.

**On TurnCompleteEvent — flush sequence:**
1. Resolve the answer using the precedence rule above.
2. Call IHistoryFormatter.FormatTurn(UserText, resolvedAnswer) to obtain the provider-specific Redis payload.
3. Async fire-and-forget write to session:{id}:history in Redis — provider-specific format.
4. Async fire-and-forget write to session:{id}:history:unified in Redis — always {"query": UserText, "answer": resolvedAnswer} format.
5. Enforce history_turn_limit: if turn_count exceeds the project config limit, the IHistoryFormatter drops the oldest user+assistant pair (always in pairs — never a dangling message). The system prompt is unaffected.
6. Reset TurnState for the next turn.

**IHistoryFormatter injection**
ConversationAccumulator is provider-agnostic. It delegates all provider-specific formatting to an IHistoryFormatter instance injected by the Connection Manager at session construction time. Each adapter type has its own formatter implementation.

**Dual Redis key strategy**

Two Redis keys are maintained per session:

`session:{session_id}:history` — Provider-specific format. Written by ConversationAccumulator via IHistoryFormatter on every TurnCompleteEvent. Used by HTTP Orchestrator adapters to build the next request (loaded from Redis at the start of each turn). Used by the fallback mechanism when the fallback adapter is the same type as the primary. Explicitly deleted at session teardown.

`session:{session_id}:history:unified` — Always {"query": string?, "answer": string?} pairs, provider-agnostic. Written by ConversationAccumulator on every TurnCompleteEvent. Used for cross-provider fallback when the fallback adapter is a different type than the primary. This is the cross-provider Rosetta Stone — the only format both providers can reconstruct their native formats from. Explicitly deleted at session teardown.

**Provider-specific history formats:**

History structure (Chat Completions adapter — session:{id}:history):
```json
{
  "turns": [
    {"role": "user",      "content": "<assembled user message>"},
    {"role": "assistant", "content": "<assembled assistant response>"}
  ],
  "turn_count": 1
}
```

History structure (Gemini Generate adapter — session:{id}:history):
```json
{
  "turns": [
    {"role": "user",  "parts": [{"text": "<assembled user message>"}]},
    {"role": "model", "parts": [{"text": "<assembled assistant response>"}]}
  ],
  "turn_count": 1
}
```

History structure (Responses API adapter — session:{id}:history):
```json
{
  "turns": [
    {"role": "user",      "content": "<assembled user message>"},
    {"role": "assistant", "content": "<assembled assistant response>"}
  ],
  "turn_count": 1,
  "previous_response_id": "<response_id_from_openai>"
}
```
The Responses API adapter uses previous_response_id chaining for normal operation. The full content history is written in parallel as fallback-only — it is only read when the primary provider fails and a new session must be reconstructed with a different provider type. The session:{id}:response_id key continues to be maintained separately for normal operation.

**WS Bridge adapter history**
WS Bridge adapters (OpenAI Realtime, Gemini Live) now also write to both Redis history keys via the ConversationAccumulator. The provider still holds all conversational state for normal operation. The Redis history exists exclusively to support provider fallback recovery. During a normal session, this history is never read by the WS Bridge adapter.

**System prompt:** Never stored in any history key. Loaded fresh from project config at each request and injected as required by the provider's native format.

**Transcript timing:** TranscriptEvent may arrive asynchronously and out of order relative to TextDeltaEvents and AudioDeltaEvents. ConversationAccumulator tolerates this because TurnState is only flushed on TurnCompleteEvent — transcripts arriving at any point before TurnCompleteEvent are captured correctly.

7.7 Session Teardown

When a session ends — for any reason — the gateway executes the following teardown sequence in order:

1. Signal the active AdapterFeedLoop to stop reading from the adapter's IAsyncEnumerable. Wait for the AdapterFeedLoop to exit cleanly.
2. Signal the EventDispatcher to close its internal input channel. The EventDispatcher drains all remaining events through to Channel A, Channel B, and Channel C.
3. Wait for Channel B (ConversationAccumulator) and Channel C (UsageAccumulator) to be fully drained — meaning both have finished processing all remaining events and completed their final Redis writes. This guarantees the final turn's history and final UsageEvent are persisted before deletion.
4. Close the provider-side connection gracefully via adapter.DisconnectAsync().
5. Read session:{id}:usage from Redis. Flush the final accumulated usage counters to PostgreSQL (usage_records table). The Session Gateway Service never accumulates usage in process memory — all counters live in Redis.
6. Send the {"type":"session_ended"} event to the End Client.
7. Remove the session from the Redis session registry (DEL session:{session_id}).
8. Explicitly delete both conversation history keys from Redis:
   - DEL session:{session_id}:history
   - DEL session:{session_id}:history:unified
   - DEL session:{session_id}:response_id (where applicable for Responses API)
   - DEL session:{session_id}:usage

Additionally, Platform admins and Platform Users can view the active session list and forcibly end a session. Admin force-close is delivered to the owning gateway node via Redis pub/sub on the admin:commands channel. The node executes the teardown sequence above within 5 seconds of receiving the command, sending close code 4006 (WebSocket) or the equivalent SSE error event to the End Client before session_ended.

7.8 SSE-Specific Behavior
[UNCHANGED]

7.9 WebSocket Message Size Limits
[UNCHANGED]

7.10 Inactivity Timeout
[UNCHANGED]

7.11 Session Error Codes

| Code            | Error Message            | Meaning                                         |
| --------------- | ------------------------ | ----------------------------------------------- |
| 4000            | —                        | Normal closure                                  |
| 4001 / HTTP 401 | authentication_failed    | Invalid, expired, or already-used token         |
| 4002            | provider_unavailable     | LLM provider connection failed at session init  |
| 4003            | session_limit_exceeded   | Session limit exceeded for this project         |
| 4004            | project_inactive         | Project is deactivated                          |
| 4005 / HTTP 408 | session_timeout          | Session inactivity timeout                      |
| 4006            | admin_force_close        | Admin force-close                               |
| 4007            | provider_fallback_failed | Both primary and fallback providers unavailable |
| 4029 / HTTP 429 | rate_limit_exceeded      | Connection rate limit exceeded                  |

7.12 Session Limits
[UNCHANGED]

7.13 Node Failure Behavior

If a gateway node crashes mid-session:
- The End Client's connection drops.
- The client must open a new session (new ephemeral token required).
- Conversation history written to session:{id}:history and session:{id}:history:unified survives in Redis for all adapter types, including WS Bridge adapters, because ConversationAccumulator now writes history for all adapter types.
- However, the new session uses a new session_id. The old history keys are bound to the old session_id and are not automatically loaded into the new session. History-based recovery for node failures is a planned feature (v1.1). For v1.0, the client reconnects with a fresh session.
- Node failure recovery (client reconnects to same session_id within a 30-second window) is distinct from provider fallback (provider fails mid-session, client connection stays open). See Section 7.14 for provider fallback.

7.14 Provider Fallback Mechanism

Provider fallback handles the case where the LLM provider's connection drops unexpectedly during an active session. Unlike node failure, the End Client's connection stays open throughout the entire fallback sequence. The client session ID remains unchanged.

**Trigger**
The AdapterFeedLoop detects that the adapter's IAsyncEnumerable LLMEvent has terminated unexpectedly (not as part of a clean session shutdown). The AdapterFeedLoop writes a ProviderDisconnectedEvent{Reason: string} into the EventDispatcher's internal input channel and then exits. The EventDispatcher fans this event to Channel A, Channel B, and Channel C.

**Immediate responses per component:**

| Component | Response to ProviderDisconnectedEvent |
|---|---|
| Protocol Handler (Channel A) | Sends {"type":"provider_reconnecting"} to End Client via open IClientTransport. Client connection is NOT closed. |
| ConversationAccumulator (Channel B) | Discards current in-progress TurnState without flushing to Redis. Partial turns are never written — they would corrupt fallback context. All previously completed turns remain intact in Redis. |
| UsageAccumulator (Channel C) | Does nothing. All previous UsageEvents were already written atomically to Redis and are fully preserved. |

**Connection Manager fallback sequence (triggered by a direct lifecycle signal from the AdapterFeedLoop, separate from the LLMEvent channel):**

1. Check secondary_project_config for this project. If present: use the secondary config for the new adapter. If not: attempt reconnect with the same primary config.
2. Read session:{id}:history and session:{id}:history:unified from Redis.
3. Determine which history format to use for context injection:
   - Same adapter type as original: use session:{id}:history (provider-specific format) directly
   - Different adapter type: use session:{id}:history:unified; the new adapter's IHistoryFormatter converts the {query, answer} pairs into the new provider's required format
4. Invoke AdapterFactory.Create(newConfig) to instantiate the new ILLMSessionAdapter.
5. Call newAdapter.ConnectAsync() injecting the system prompt, tool list, and the resolved conversation history as context.
6. Create a new AdapterFeedLoop for the new adapter. This is the only new component created during fallback. EventDispatcher, ConversationAccumulator, UsageAccumulator, Protocol Handler, and IClientTransport all continue running — untouched.
7. The new AdapterFeedLoop begins writing into the same EventDispatcher's internal input channel.
8. Once the new provider session is confirmed ready: Connection Manager sends {"type":"provider_reconnected"} to the End Client via the open IClientTransport. The session resumes seamlessly.

**Fallback failure:** If the fallback adapter also fails to connect: Connection Manager sends {"type":"error","code":"provider_fallback_failed","message":"All providers unavailable","retryable":false} followed by {"type":"session_ended"} and proceeds to normal session teardown.

**Compatible fallback pairs (enforced at configuration time, not runtime):**
- Audio-to-audio pairs: OpenAI Realtime API ↔ Google Gemini Live API
- Text-to-text pairs: OpenAI Chat Completions ↔ Google Gemini Generate Content ↔ OpenAI Responses API
- Cross-category fallbacks (audio primary → text fallback) are not permitted. Validated at secondary_project_config save time.

**secondary_project_config data model:**
A separate configuration record associated with a project. It inherits the system prompt, tools, voice settings, RAG settings, and MCP settings from the primary project_config. It differs only in the LLM provider, API type, model, and provider-specific parameters. The foreign key relationship is: secondary_project_configs.project_config_id → project_configs.id. The compatible fallback pairs are validated at save time. The UI exposes this as an optional "Fallback Provider" step in the project configuration wizard (Pro plan and above).

________________

8. LLM Adapter Layer

8.1 The ILLMSessionAdapter Interface

All five adapters implement the same ILLMSessionAdapter interface. The adapter's sole responsibility is to be a pure, stateless converter between the Platform's unified LLMEvent format and a specific provider's native protocol. It does not buffer, does not assemble text, does not write to Redis, does not track usage, and has no knowledge of conversation history, ConversationAccumulator, UsageAccumulator, or EventDispatcher.

Outbound methods (Protocol Handler → Provider):
- ConnectAsync() — establishes the provider-side connection; injects system prompt, tool list, and optionally pre-loaded conversation history for context reconstruction
- SendTextInputAsync(text) — sends a user text turn to the provider
- SendAudioChunkAsync(pcmBase64, sampleRate) — sends a raw PCM audio chunk
- SendToolResultAsync(toolCallId, result) — returns a function call result to the provider
- SendInterruptAsync() — interrupts the current provider response
- DisconnectAsync() — closes the provider-side connection gracefully

Inbound output (Provider → Platform):
- IAsyncEnumerable LLMEvent — output channel producing normalized LLMEvents immediately as they arrive from the provider. No buffering. No assembly. Each event is emitted the moment it is received and normalized.

**Complete LLMEvent Contract:**

| Event | Payload | Notes |
|---|---|---|
| TextDeltaEvent | Text: string | Emitted immediately per chunk. No waiting for turn completion. |
| AudioDeltaEvent | Audio: string (base64), SampleRate: int | Emitted immediately per chunk. |
| TranscriptEvent | Role: "user" or "assistant", Text: string | May arrive asynchronously and out of order relative to other events. |
| ToolCallEvent | ToolCallId: string, Name: string, Arguments: object | Emitted when provider requests a tool call. |
| UsageEvent | InputTokens: int, OutputTokens: int, AudioInSecs: float, AudioOutSecs: float | Emitted once per turn, parsed from provider response metadata (e.g., final SSE chunk, response.done event, usageMetadata field). Emitted immediately when the metadata arrives — not tied to TurnCompleteEvent timing. |
| TurnCompleteEvent | (no payload) | Pure signal. Emitted when the provider's end-of-stream marker arrives. No full text payload expected or required. Replaces both TextDoneEvent and AudioDoneEvent. |
| ProviderDisconnectedEvent | Reason: string | Emitted when the provider-side connection drops unexpectedly. Not a normal turn event — triggers fallback. |

**Removed from the previous LLMEvent contract:**
- TextDoneEvent — removed. Providers do not reliably send full assembled text at stream end. TurnCompleteEvent is the single end-of-turn signal.
- AudioDoneEvent — removed. TurnCompleteEvent covers it.
- SessionStartedEvent — removed. Owned by Connection Manager, not the adapter.
- SessionEndedEvent — removed. Owned by Connection Manager, not the adapter.
- ToolResultProcessedEvent — removed.

8.2 Adapter Types

**WS Bridge Adapters:** Maintain a persistent WebSocket connection to the LLM provider for the lifetime of the provider-side connection. Conversation state is held by the provider. The gateway translates the Platform's unified protocol to the provider's native event format in both directions. Used for: OpenAI Realtime API, Google Gemini Live API.

WS Bridge adapters now participate in conversation history via the ConversationAccumulator. The ConversationAccumulator writes to both session:{id}:history and session:{id}:history:unified on every TurnCompleteEvent for WS Bridge sessions. During normal operation, this history is not read by the WS Bridge adapter (the provider holds state). The Redis history exists exclusively for provider fallback recovery — allowing a new provider session to be reconstructed from real conversation content if the provider connection drops unexpectedly.

**HTTP Orchestrator Adapters:** Communicate with the LLM provider via HTTPS REST requests, one request per user turn. On each turn: load history from session:{id}:history in Redis, build the full request (including history in provider-native format), send HTTP POST, stream the SSE response, normalize events, emit LLMEvents immediately per chunk, emit TurnCompleteEvent when the provider's end marker arrives. History is written to Redis by ConversationAccumulator after TurnCompleteEvent — the adapter itself never writes history. Used for: OpenAI Chat Completions, Google Gemini Generate Content, OpenAI Responses API.

8.3 Adapter Implementations

**OpenAI Chat Completions Adapter (HTTP Orchestrator):**
Communicates via POST /v1/chat/completions with streaming enabled. Reads conversation history from session:{id}:history (messages[] format) at the start of each turn and includes it in the request. System prompt injected as {"role":"system"} message at request build time (not stored in Redis history). Normalizes SSE chunks and emits TextDeltaEvent per chunk immediately. Emits UsageEvent when the final SSE chunk containing usage metadata arrives. Emits TurnCompleteEvent when the provider's [DONE] marker arrives. Does not buffer or assemble text — ConversationAccumulator owns assembly.

**OpenAI Responses API Adapter (HTTP Orchestrator — Responses):**
Communicates via POST /v1/responses. Uses previous_response_id from session:{id}:response_id for stateful conversation chaining during normal operation. Reads full content history from session:{id}:history only when reconnecting after a provider fallback (to reconstruct context for a new session). Supports built-in tool types if configured. Emits TextDeltaEvent per chunk immediately. Emits UsageEvent from provider response metadata. Emits TurnCompleteEvent when the provider's end marker arrives.

**OpenAI Realtime API Adapter (WS Bridge):**
Opens and maintains a WSS connection to the OpenAI Realtime API. Translates bidirectionally between the Platform's unified protocol and OpenAI's Realtime event format. Handles VAD events, audio deltas, interruptions, and function calls natively. Emits AudioDeltaEvent per audio chunk immediately. Emits TranscriptEvent for both user and assistant roles when transcription is enabled. Emits UsageEvent from the response.done event metadata. Emits TurnCompleteEvent when the provider's response end marker arrives. Does not buffer — all events emitted immediately upon receipt.

At ConnectAsync() time, if conversation history is provided (fallback scenario): injects the history into the Realtime session configuration as initial conversation context in OpenAI's required format.

**Google Gemini Generate Content Adapter (HTTP Orchestrator):**
Communicates via POST to the Gemini streamGenerateContent endpoint. Reads conversation history from session:{id}:history (contents[] format) at the start of each turn. System instruction sent as a separate systemInstruction field (not in contents[] history). Normalizes Gemini streaming response chunks and emits TextDeltaEvent per chunk immediately. Emits UsageEvent from the usageMetadata field in the final response chunk. Emits TurnCompleteEvent when the provider's stream completes.

**Google Gemini Live API Adapter (WS Bridge):**
Opens and maintains a WSS connection to the Gemini Live API using the BidiGenerateContent protocol. Uses org-level Google API credentials from Key Vault. Optionally acquires a Google-issued ephemeral token for the provider-side connection. Handles audio, VAD, transcription, and tool events. Emits AudioDeltaEvent per audio chunk immediately. Emits TranscriptEvent for both roles when transcription is enabled. Emits UsageEvent from provider metadata events. Emits TurnCompleteEvent when the provider's turn end marker arrives. Does not buffer.

At ConnectAsync() time, if conversation history is provided (fallback scenario): injects the history into the Gemini Live session configuration as initial conversation context in Gemini's required format.

8.4 System Prompt and Tool Injection
[UNCHANGED]

8.5 Usage Event Emission

Usage metadata (token counts, audio seconds) is embedded in the LLM provider's own response stream at the adapter level — not computed by the gateway. The location varies by provider:

| Provider | Usage Metadata Location |
|---|---|
| OpenAI Chat Completions | Final SSE chunk with a usage object (prompt_tokens, completion_tokens) |
| OpenAI Responses API | Final SSE event with usage metadata |
| OpenAI Realtime API | response.done WebSocket event with usage metadata |
| Gemini Generate Content | usageMetadata field in the final streaming chunk |
| Gemini Live API | Dedicated usage metadata event |

Each adapter parses usage metadata from wherever the provider embeds it and emits a standalone UsageEvent{InputTokens, OutputTokens, AudioInSecs, AudioOutSecs} immediately when it arrives. The adapter never buffers waiting to collect usage — it emits UsageEvent the moment it parses the metadata, consistent with its single-responsibility as a pure normalizer.

The UsageAccumulator receives UsageEvent via Channel C and atomically increments the session:{id}:usage hash in Redis using HINCRBY/HINCRBYFLOAT. This happens entirely in the background, off the client delivery path.

At session teardown, the Connection Manager reads the final session:{id}:usage hash from Redis and writes one usage_records row to PostgreSQL. The Session Gateway Service never accumulates usage in process memory — it is fully stateless with respect to usage tracking.

At the end of every conversation turn, a TurnCompleteEvent must be emitted by the adapter. The UsageEvent may arrive before or alongside TurnCompleteEvent depending on the provider — the UsageAccumulator handles both orderings correctly.

________________

9. RAG Architecture
[UNCHANGED]

10. MCP Architecture
[UNCHANGED]

11. Authentication and Security Architecture
[UNCHANGED]

________________

12. Data Architecture

12.1 Primary Database (PostgreSQL)

[UNCHANGED except the following addition:]

**secondary_project_configs table:**
Stores the fallback provider configuration for a project. Fields: id, project_config_id (FK → project_configs.id), provider_id (FK → providers.id), llm_provider_api_id (FK → llm_provider_apis.id), model_id (FK → models.id), and any provider-specific parameter overrides. All other configuration (system prompt, tools, voice settings, RAG settings, MCP settings) is inherited from the associated project_config and is not duplicated. A project_config may have at most one secondary_project_config. Compatible fallback pairs are validated at save time: audio API types may only fall back to other audio API types; text API types may only fall back to other text API types.

12.2 Redis (Cache and Session Store)

Redis serves multiple distinct roles. A single Redis instance is used for v1.0. Redis persistence (RDB/AOF) may be enabled for durability of infrastructure data; conversation history keys are always explicitly deleted at session teardown and carry a TTL as a safety net.

Key naming conventions:

| Key Pattern                                | Content                                                                                                                                                                                                                                                                                   | TTL                                                  |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| session:{session_id}                       | Session metadata hash                                                                                                                                                                                                                                                                     | Session lifetime                                     |
| session:{session_id}:history               | JSON turn history array in provider-specific format. Written by ConversationAccumulator via IHistoryFormatter for ALL adapter types including WS Bridge. Used by HTTP Orchestrator adapters for next-turn request building and by the fallback mechanism for same-type provider recovery. | Session inactivity timeout (refreshed on each write) |
| session:{session_id}:history:unified       | JSON array of {query: string?, answer: string?} pairs. Written by ConversationAccumulator for ALL adapter types. Provider-agnostic. Used for cross-provider fallback recovery.                                                                                                            | Session inactivity timeout (refreshed on each write) |
| session:{session_id}:response_id           | previous_response_id string (OpenAI Responses API only). Used for normal operation chaining.                                                                                                                                                                                              | Session inactivity timeout                           |
| session:{session_id}:usage                 | Usage counters hash (input_tokens, output_tokens, audio_in_secs, audio_out_secs, turn_count). Written by UsageAccumulator. Read by Connection Manager at teardown for PostgreSQL flush.                                                                                                   | Session lifetime                                     |
| project:{project_id}:config                | JSON project configuration                                                                                                                                                                                                                                                                | Until invalidated on config update                   |
| project:{project_id}:mcp:{server_id}:tools | JSON MCP tool list                                                                                                                                                                                                                                                                        | 10 minutes (auto-refresh)                            |
| apikey:{key_hash}                          | JSON API key metadata                                                                                                                                                                                                                                                                     | Until revoked                                        |
| token:{token_hash}                         | JSON ephemeral token payload                                                                                                                                                                                                                                                              | 60s (GETDEL at use)                                  |
| ratelimit:apikey:{key_hash}:conns          | Sliding window counter                                                                                                                                                                                                                                                                    | Rolling window                                       |
| ratelimit:ip:{ip}:requests                 | Sliding window counter                                                                                                                                                                                                                                                                    | Rolling window                                       |
| admin:commands                             | Pub/sub channel                                                                                                                                                                                                                                                                           | N/A                                                  |

________________

13. Use Cases
[UNCHANGED]

14. External Interface Specifications
[UNCHANGED]

________________

15. Protocol Reference

15.1 Client → Gateway Messages
[UNCHANGED]

15.2 Gateway → Client Messages

| Type                  | Fields                                                        | Description                                                                                                                                                                                      |
| --------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| session_started       | session_id: string                                            | Sent once after both client-side transport and provider-side connection are confirmed ready. Sent by Connection Manager — not an adapter event.                                                  |
| text_delta            | delta: string                                                 | Incremental text token from the LLM. Forwarded immediately as received.                                                                                                                          |
| text_done             | text: string                                                  | Full assembled text response for this turn. Assembled by the Protocol Handler's lightweight in-memory buffer from forwarded text_delta events. Sent at TurnCompleteEvent time.                   |
| audio_delta           | audio: string (base64), sample_rate: int                      | Incremental audio chunk from the LLM. Always forwarded immediately — never buffered.                                                                                                             |
| audio_done            | —                                                             | Audio response for this turn is complete. Sent at TurnCompleteEvent time for audio turns.                                                                                                        |
| tool_call             | tool_call_id: string, name: string, arguments: object         | LLM requesting a client-executed function call (source='manual' tools only). Gateway-intercepted tools (RAG, MCP) are never forwarded as tool_call.                                              |
| tool_invocation       | source: "builtin" \| "gateway" \| "client", tool_name: string | Informational event sent for ALL tool invocations regardless of type. Sent before the tool is executed. Allows End Client to display appropriate UI indicators (searching, processing, waiting). |
| transcript            | role: "user" \| "assistant", text: string                     | Audio transcript when transcription is enabled.                                                                                                                                                  |
| rag_search            | query: string, chunks_retrieved: int                          | Informational: gateway intercepted a search_internal_knowledge call and retrieved context. The retrieved text is NOT included.                                                                   |
| turn_complete         | —                                                             | Indicates the completion of one full conversation turn including all tool cycles within the turn.                                                                                                |
| provider_reconnecting | —                                                             | Provider-side connection lost. Fallback in progress. Client connection remains open.                                                                                                             |
| provider_reconnected  | —                                                             | Fallback provider connected. Session resumed.                                                                                                                                                    |
| error                 | code: string, message: string, retryable: bool                | Error event.                                                                                                                                                                                     |
| session_ended         | —                                                             | Sent as the final message before the server closes the connection (both WS and SSE).                                                                                                             |
| pong                  | —                                                             | Keepalive response to ping.                                                                                                                                                                      |

**Removed from the previous protocol:**
- No protocol-level change to how the client sends messages.
- text_done semantics updated: assembled by Protocol Handler's passthrough buffer, not by the adapter or ConversationAccumulator. Client behavior is unchanged.
- audio_done semantics unchanged: still sent at turn completion for audio turns.
- tool_invocation is a new event type added for all tool types.
- provider_reconnecting and provider_reconnected are new event types added for the provider fallback mechanism.

15.3 SSE-Specific Protocol Notes
[UNCHANGED]

________________

16. Cost Estimation
[UNCHANGED]

17. Pricing
[UNCHANGED]

18. Competitor Analysis
[UNCHANGED]

________________

References
- See all architectural diagrams: Architecture.drawio.xml
- See the database schema design: ERD.drawio.xml
- See the database Schema query: schema.sql
- See the extended requirement list with acceptance criterias: Requirements.docs
- See workflow diagrams: Workflow Diagrams.drawio.xml

________________

Change Log (vs. previous SRS version)

| #   | Section | Change Summary                                                                                                                                                                                                                                                                                                                                                    |
| --- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | 1.5     | Added new terminology: AdapterFeedLoop, EventDispatcher, ConversationAccumulator, UsageAccumulator, IHistoryFormatter, TurnCompleteEvent, ProviderDisconnectedEvent, UsageEvent. Updated Session definition to clarify client-session scope.                                                                                                                      |
| 2   | 6.3     | Major rewrite. Added EventDispatcher, AdapterFeedLoop, ConversationAccumulator, UsageAccumulator as named components. Added component lifetime table. Removed delta assembler and chat history management from Protocol Handler. Clarified Protocol Handler direct adapter reference for outbound calls.                                                          |
| 3   | 6.5     | Updated Observer/Pipeline pattern description. Added IHistoryFormatter strategy pattern.                                                                                                                                                                                                                                                                          |
| 4   | 7.4     | Added steps 8-11 for instantiating EventDispatcher, ConversationAccumulator, UsageAccumulator, and AdapterFeedLoop. Clarified session_started timing.                                                                                                                                                                                                             |
| 5   | 7.5     | Renamed from "Dual Async Loop" to "Parallel Processing Model". Described all parallel loops. Added RecordUserTurn call. Added audio unconditional passthrough rule. Added Protocol Handler text_done assembly note. Removed usage accumulation from Protocol Handler.                                                                                             |
| 6   | 7.6     | Major rewrite. ConversationAccumulator owns all history. TurnState defined. Dual Redis key strategy. IHistoryFormatter pattern. WS Bridge adapters now write history. Responses API writes full content history alongside response_id. TurnCompleteEvent replaces TextDoneEvent as flush trigger. ProviderDisconnectedEvent causes TurnState discard (not flush). |
| 7   | 7.7     | Added channel drain sequence before teardown. Updated usage flush to read from Redis (not process memory). Added deletion of both history keys and usage key.                                                                                                                                                                                                     |
| 8   | 7.11    | Added error code 4007 provider_fallback_failed.                                                                                                                                                                                                                                                                                                                   |
| 9   | 7.13    | Updated node failure description to reflect WS Bridge history now exists in Redis. Noted history-based node recovery as v1.1 planned feature.                                                                                                                                                                                                                     |
| 10  | 7.14    | New section. Full provider fallback mechanism. AdapterFeedLoop replacement. Persistent per-client-session components. ProviderDisconnectedEvent handling. Compatible fallback pairs. secondary_project_config.                                                                                                                                                    |
| 11  | 8.1     | Removed TextDoneEvent, AudioDoneEvent, SessionStartedEvent, SessionEndedEvent, ToolResultProcessedEvent. Added UsageEvent, ProviderDisconnectedEvent, TurnCompleteEvent. Stated adapter single-responsibility constraint explicitly.                                                                                                                              |
| 12  | 8.2     | Updated WS Bridge adapter description to include ConversationAccumulator history writing for fallback.                                                                                                                                                                                                                                                            |
| 13  | 8.3     | Removed delta assembly from all adapter descriptions. Updated all five adapters to reflect immediate LLMEvent emission and UsageEvent emission.                                                                                                                                                                                                                   |
| 14  | 8.5     | Complete rewrite. Standalone UsageEvent per provider. UsageAccumulator ownership. Removed incorrect TextDoneEvent/AudioDoneEvent usage payload design.                                                                                                                                                                                                            |
| 15  | 12.1    | Added secondary_project_configs table description.                                                                                                                                                                                                                                                                                                                |
| 16  | 12.2    | Added session:{id}:history:unified key. Updated session:{id}:history description to reflect ConversationAccumulator ownership for all adapter types. Added session:{id}:usage description.                                                                                                                                                                        |
| 17  | 15.2    | Updated text_done semantics. Added tool_invocation event. Added provider_reconnecting and provider_reconnected events. Noted removed events.                                                                                                                                                                                                                      |

| Type                  | Fields                                                        | Description                                                                                                                                                                                                                           |
| --------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| session_started       | session_id: string                                            | Sent once after both client-side transport and provider-side connection are confirmed ready. Sent by Connection Manager — not an adapter event.                                                                                       |
| text_delta            | delta: string                                                 | Incremental text token from the LLM. Forwarded immediately by Protocol Handler (Channel A) regardless of text_response_mode setting.                                                                                                  |
| text_done             | text: string                                                  | Full assembled text response for this turn. In COMPLETE mode: buffered in Protocol Handler's TurnTextBuffer and sent at TurnCompleteEvent time. In STREAMING mode: derived from TextAccumulator's assembled text at turn flush.       |
| audio_delta           | audio: string (base64), sample_rate: int                      | Incremental audio chunk from the LLM. ALWAYS forwarded immediately by Protocol Handler (Channel A) — unconditional passthrough, never buffered regardless of text_response_mode setting.                                              |
| audio_done            | —                                                             | Audio response for this turn is complete. Sent at TurnCompleteEvent time for audio turns.                                                                                                                                             |
| tool_call             | tool_call_id: string, name: string, arguments: object         | LLM requesting a client-executed function call (source='manual' tools only). Gateway-intercepted tools (RAG, MCP) are never forwarded as tool_call.                                                                                   |
| tool_invocation       | source: "builtin" \| "gateway" \| "client", tool_name: string | Informational event sent for ALL tool invocations regardless of type. Sent BEFORE the tool is executed (by Protocol Handler or interceptor). Allows End Client to display appropriate UI indicators (searching, processing, waiting). |
| transcript            | role: "user" \| "assistant", text: string                     | Audio transcript when transcription is enabled. Forwarded immediately by Protocol Handler (Channel A) as received from adapter.                                                                                                       |
| rag_search            | query: string, chunks_retrieved: int                          | Informational: gateway intercepted a search_internal_knowledge call and retrieved context from Azure AI Search. The retrieved text is NOT included.                                                                                   |
| turn_complete         | —                                                             | Indicates the completion of one full conversation turn including all tool cycles within the turn. All buffered content (TextDeltaEvent accumulation in COMPLETE mode, any partial state) is flushed to client before this event.      |
| provider_reconnecting | —                                                             | Provider-side connection lost unexpectedly. AdapterFeedLoop detected failure and emitted ProviderDisconnectedEvent. Fallback in progress. Client connection remains open. End Client should display reconnecting indicator.           |
| provider_reconnected  | —                                                             | Fallback provider connected and initialized. Session resumed with history preloaded from Redis. End Client can dismiss reconnecting indicator.                                                                                        |
| error                 | code: string, message: string, retryable: bool                | Error event describing a failure condition (authentication, session limit, timeout, adapter failure, etc.). May precede session_ended if fatal.                                                                                       |
| session_ended         | —                                                             | Sent as the final message before the server closes the connection (both WS and SSE). Indicates graceful session teardown or forced close.                                                                                             |
| pong                  | —                                                             | Keepalive response to ping (WebSocket primarily; SSE uses heartbeat comment lines).                                                                                                                                                   |