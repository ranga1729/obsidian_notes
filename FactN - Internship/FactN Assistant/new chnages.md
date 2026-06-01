## Complete SRS Change List — All Architectural Improvements (Excluding text_response_mode)

---

## Section 6.3 — Component Decomposition: Session Gateway Service

**Change 1: Remove "Delta Assembler" from Protocol Handler responsibilities**
The current SRS assigns delta assembly to the Protocol Handler (and indirectly to the adapter). This must be removed entirely. The Protocol Handler's responsibility list must be updated to: inbound message parsing and routing, tool interception, serializing LLMEvents to client-facing JSON, and forwarding events to the End Client. Nothing about text assembly or history management.

**Change 2: Add EventDispatcher as a named per-session component**
A new component called EventDispatcher must be introduced at the same architectural level as the Protocol Handler. Its sole responsibility is to read the single `IAsyncEnumerable<LLMEvent>` output stream produced by the adapter and fan it out to three independent bounded `Channel<LLMEvent>` instances — Channel A (Protocol Handler), Channel B (TextAccumulator), Channel C (UsageAccumulator). This exists because `IAsyncEnumerable` is a single-consumer stream by design. Without this component, having multiple independent consumers on the adapter output is architecturally impossible — one consumer would starve the other or both would race and drop events. The EventDispatcher is instantiated by the Connection Manager at session start.

**Change 3: Add TextAccumulator as a named per-session component**
A new component called TextAccumulator must be introduced. Its sole responsibility is to read LLMEvents from Channel B, accumulate text chunks into complete turns, and write full assembled conversation history to Redis asynchronously without blocking the client delivery path. It runs in the background parallel to the Protocol Handler. It is instantiated by the Connection Manager at session start with an injected `IHistoryFormatter` instance.

**Change 4: Add UsageAccumulator as a named per-session component**
A new component called UsageAccumulator must be introduced. Its sole responsibility is to read `UsageEvent`s from Channel C and atomically increment usage counters in Redis. It is instantiated by the Connection Manager at session start. Remove usage accumulation from the Protocol Handler's dual async loops entirely.

**Change 5: Clarify Protocol Handler's direct relationship with the adapter**
The SRS must explicitly state that the Protocol Handler holds a direct reference to `ILLMSessionAdapter` for the Client→Provider direction (outbound method calls: `SendTextInputAsync`, `SendAudioChunkAsync`, `SendToolResultAsync`, `SendInterruptAsync`). For the Provider→Client direction, the Protocol Handler reads from Channel A only — it does NOT read directly from the adapter's `IAsyncEnumerable`. The EventDispatcher is the sole consumer of the adapter's output stream.

---

## Section 7.5 — Active Session: Dual Async Loop

**Change 6: Rename and expand to "Parallel Processing Model"**
The section must be renamed from "Dual Async Loop" to reflect that there are now four parallel operations running during an active session:
- Loop 1: Client→Provider (Protocol Handler reads from IClientTransport, calls adapter methods directly)
- Loop 2: EventDispatcher loop (reads adapter's IAsyncEnumerable, writes to all three channels)
- Loop 3: TextAccumulator background loop (reads Channel B, assembles history, writes to Redis asynchronously)
- Loop 4: UsageAccumulator background loop (reads Channel C, increments Redis counters)

All four run concurrently and independently. No loop blocks another.

**Change 7: Add RecordUserTurn call in Client→Provider loop**
When the Protocol Handler processes a `text_input` message from the End Client in Loop 1, before calling `adapter.SendTextInputAsync(text)`, it must call `TextAccumulator.RecordUserTurn(text)`. This is the only direct coupling between the Protocol Handler and the TextAccumulator. It captures the user's text query for history purposes since the text_input message never passes through the EventDispatcher — it flows directly from the Protocol Handler to the adapter.

**Change 8: Remove usage accumulation from dual async loops**
The current SRS states "Both loops accumulate usage counters in Redis." This must be removed. Usage accumulation is now exclusively owned by the UsageAccumulator via Channel C.

---

## Section 7.6 — Conversation History Management

**Change 9: Remove delta assembly from adapter; TextAccumulator owns all assembly**
The current SRS states: *"streaming text deltas are accumulated in the adapter's in-memory StringBuilder during a turn."* This must be completely removed. The adapter never buffers. The adapter emits `TextDeltaEvent` immediately per chunk and emits `TurnCompleteEvent` when the provider's end-of-stream marker arrives. TextAccumulator receives these events via Channel B and performs all assembly independently off the critical path.

**Change 10: Replace TextDoneEvent/AudioDoneEvent with TurnCompleteEvent as the history write trigger**
The current SRS uses `TextDoneEvent` arrival as the trigger for writing assembled history to Redis. This must be replaced. `TurnCompleteEvent` is the single trigger for flushing the accumulated turn to Redis. The previous architecture relied on providers sending full assembled text inside a `TextDoneEvent` — this is a false assumption. No LLM provider guarantees sending concatenated text at the end of a stream. They only send an end-of-stream marker signal. TextAccumulator waits for `TurnCompleteEvent`, then assembles from its accumulated `TextDeltaEvent` parts.

**Change 11: Introduce dual Redis key strategy per session**
History must now be stored in two parallel Redis keys:
- `session:{id}:history` — provider-specific format (Chat Completions `messages[]`, Gemini `contents[]`, etc.) for reconstructing correct provider API requests. This is the key used by HTTP Orchestrator adapters when building the next request.
- `session:{id}:history:unified` — always `{ "query": string?, "answer": string? }` pairs, provider-agnostic. This is the cross-provider Rosetta Stone used for provider fallback when the fallback provider is a different type than the primary. Both keys are written by TextAccumulator on every `TurnCompleteEvent`. Both keys are explicitly deleted at session teardown.

**Change 12: WS Bridge adapters must now write conversation history**
The current SRS states: *"For WS Bridge adapters, no history array is created — the provider holds all state."* This must be reversed. WS Bridge adapters (OpenAI Realtime, Gemini Live) must now also write to both Redis history keys via the TextAccumulator. During normal operation, this history is unused (the provider holds state). However, for provider fallback to work, history must exist in Redis so a new provider session can be reconstructed from real conversation content if the provider connection drops unexpectedly. TextAccumulator writes this history for all adapter types uniformly.

**Change 13: Responses API must write full content history alongside response_id**
The current SRS stores only `previous_response_id` for the Responses API adapter. This is insufficient for fallback. The Responses API `IHistoryFormatter` must write full `{ role: "user", content: "..." } / { role: "assistant", content: "..." }` turn pairs to `session:{id}:history` in addition to maintaining `session:{id}:response_id`. During normal operation, the adapter uses `previous_response_id` chaining. The full content history in `session:{id}:history` is fallback-only — it is only read when the primary provider fails and a new session must be reconstructed. This is a parallel async write with negligible overhead.

**Change 14: Introduce IHistoryFormatter injection pattern**
TextAccumulator must remain provider-agnostic. It must not contain any provider-specific formatting logic. A new `IHistoryFormatter` interface must be defined. Each adapter type provides its own formatter implementation. The Connection Manager injects the correct `IHistoryFormatter` into TextAccumulator at session construction time based on the resolved adapter type. TextAccumulator calls `formatter.FormatTurn(userText, resolvedAnswer)` and receives the provider-specific JSON structure to write to `session:{id}:history`. TextAccumulator itself never knows about provider formats.

**Change 15: TextAccumulator per-turn state object and flush rules**
The SRS must define the TextAccumulator's internal per-turn state object:
- `UserText: string?` — populated by `RecordUserTurn(text)` for text inputs, or by `TranscriptEvent { Role: "user" }` for audio inputs
- `AssistantTextParts: List<string>` — appended on every `TextDeltaEvent`, accumulates across tool cycles
- `AssistantTranscript: string?` — populated by `TranscriptEvent { Role: "assistant" }` when transcription is enabled

Answer resolution precedence at flush time: `AssistantTranscript` wins over joined `AssistantTextParts` if present, because transcript is the authoritative text for audio turns. The TurnState is NOT reset on `ToolCallEvent` — tool cycles are mid-turn. TurnState is only reset after `TurnCompleteEvent` flush completes.

**Change 16: Add Redis key entries for new history keys**
Section 12.2 Redis key naming must add:
- `session:{id}:history:unified` ��� unified `{query, answer}` format, TTL refreshed on each write, explicit DEL at teardown
- Update the existing `session:{id}:history` description to clarify it is now written by TextAccumulator (not the adapter) for all adapter types including WS Bridge

---

## Section 7.7 — Session Teardown

**Change 17: Update teardown step 1 — flush from UsageAccumulator's Redis counters**
The current SRS step 1 says "Flush the final accumulated usage counters to PostgreSQL." This must be updated to clarify: the Connection Manager reads from `session:{id}:usage` in Redis (populated by UsageAccumulator throughout the session) and writes the final record to PostgreSQL. The Session Gateway Service itself never accumulates usage in process memory — UsageAccumulator writes everything to Redis atomically as it arrives.

**Change 18: Add explicit drain of EventDispatcher channels before teardown**
Teardown must include a step where the Connection Manager signals the EventDispatcher to stop reading from the adapter's output stream, and waits for Channel B and Channel C to be fully drained before proceeding. This guarantees TextAccumulator writes the final turn's history and UsageAccumulator writes the final `UsageEvent` before their Redis keys are deleted. Without this drain step, the last turn's history and usage data could be lost on teardown.

**Change 19: Delete both history keys at teardown**
Step 5 of the teardown sequence must be updated to explicitly delete both `session:{id}:history` and `session:{id}:history:unified` in addition to `session:{id}:response_id` where applicable.

---

## Section 7.13 — Node Failure Behavior

**Change 20: Implement proper provider fallback mechanism**
The current SRS footnotes [h], [i], [j] acknowledge the need for fallback but provide no architectural design. This section must be fully rewritten with the following mechanism:

When `ProviderDisconnectedEvent` arrives from the adapter during an active session:
1. Protocol Handler sends `{ "type": "provider_reconnecting" }` to the End Client. The client-side connection stays open throughout.
2. TextAccumulator performs a best-effort flush of any in-progress TurnState to Redis before resetting.
3. Connection Manager receives a direct lifecycle signal from the adapter (separate from the LLMEvent channel — this is a state machine trigger) and begins the fallback sequence.
4. Connection Manager reads `session:{id}:history` and `session:{id}:history:unified` from Redis.
5. If a `secondary_project_config` exists: Connection Manager instantiates a new adapter using the secondary config. If not: Connection Manager attempts to reconnect with the same adapter type.
6. If the fallback is the same adapter type: `session:{id}:history` (provider-specific format) is used directly to reconstruct context.
7. If the fallback is a different adapter type: `session:{id}:history:unified` is used, and the new adapter's `IHistoryFormatter` converts the `{ query, answer }` pairs into the new provider's required format.
8. New EventDispatcher, TextAccumulator, and UsageAccumulator are instantiated for the new adapter session. Usage counters in `session:{id}:usage` continue accumulating without reset.
9. Once the new provider session is ready: Connection Manager sends `{ "type": "provider_reconnected" }` to the End Client and the session resumes.

This is why the dual-format history strategy is architecturally essential. Without `session:{id}:history:unified`, cross-provider fallback is impossible because provider-specific formats are mutually incompatible.

**Change 21: Add `secondary_project_config` to the data model**
A new `secondary_project_config` table must be added to the PostgreSQL schema. It mirrors `project_configs` but only differs in the LLM provider and API type fields. System prompt, tools, voice settings, RAG settings, and MCP settings are inherited from the primary `project_config`. The `secondary_project_config` has a foreign key relationship back to `project_configs`. The compatible fallback provider/API combinations must be validated at configuration time (e.g., Gemini Live and OpenAI Realtime are compatible audio-to-audio pairs; Chat Completions and Gemini Generate Content are compatible text-to-text pairs).

---

## Section 8.1 — ILLMSessionAdapter Interface

**Change 22: Remove all assembly and history responsibilities from the interface contract**
The interface definition and its description must be updated to explicitly state: the adapter's sole responsibility is to be a pure, stateless converter between the platform's unified LLMEvent format and a specific provider's native protocol. It does not buffer, does not assemble text, does not write to Redis, does not track usage, and does not know about conversation history. Any documentation implying otherwise must be removed.

**Change 23: Remove `TextDoneEvent` and `AudioDoneEvent` from the adapter output**
These two events must be removed from the adapter's LLMEvent output contract. They were designed around the false assumption that providers send full assembled text at stream end. `TurnCompleteEvent` replaces both as the single, universal end-of-turn signal. It carries no payload — it is a pure signal. The adapter emits it when the provider's end-of-stream marker arrives, regardless of content type (text, audio, or mixed).

**Change 24: Add `ProviderDisconnectedEvent` to the adapter's LLMEvent contract**
A new `ProviderDisconnectedEvent { Reason: string }` must be added to the adapter's output. The adapter emits this when the provider-side connection drops unexpectedly (not as part of a normal session end). This is distinct from `TurnCompleteEvent` — it signals an error condition that triggers the fallback/reconnect flow. The EventDispatcher routes it to all three channels as normal.

**Change 25: Remove `SessionStartedEvent` and `SessionEndedEvent` from the adapter's LLMEvent contract**
If these exist in any form in the adapter's output, they must be removed. Session lifecycle events are owned exclusively by the Connection Manager. `session_started` is sent by the Connection Manager only after both the client-side transport and the provider-side `ConnectAsync()` are confirmed ready. `session_ended` is sent by the Connection Manager as the final step of the teardown sequence. The adapter has no authority over session lifecycle.

---

## Section 8.2 — Adapter Types

**Change 26: Update WS Bridge adapter description to include history writing**
The description currently states WS Bridge adapters have no Redis history. This must be updated to state: WS Bridge adapters now participate in history writing via the TextAccumulator (which writes to both `session:{id}:history` and `session:{id}:history:unified`). The provider still holds all conversational state for normal operation. The Redis history exists exclusively to support provider fallback recovery.

---

## Section 8.3 — Adapter Implementations

**Change 27: Remove delta assembly code from all HTTP Orchestrator adapter descriptions**
All five adapter descriptions that mention "assembles full response text in memory" or "emits TextDoneEvent with assembled text on completion" must be updated. The adapter emits `TextDeltaEvent` per chunk immediately and `TurnCompleteEvent` when the provider's end marker arrives. No StringBuilder, no assembly, no waiting.

---

## Section 8.5 — Usage Event Emission

**Change 28: Replace TextDoneEvent/AudioDoneEvent usage payload with standalone UsageEvent**
The current SRS states: *"Adapters emit usage data within TextDoneEvent and AudioDoneEvent payloads."* This is architecturally wrong for two reasons: (1) `TextDoneEvent` and `AudioDoneEvent` are being removed, and (2) usage metadata arrives from providers as separate metadata in provider-native response events — typically in the last chunk of the stream for HTTP APIs, or in a dedicated response metadata event for WS Bridge APIs. The adapter must parse this metadata from wherever the provider embeds it and emit a standalone `UsageEvent { InputTokens: int, OutputTokens: int, AudioInSecs: float, AudioOutSecs: float }` immediately when it arrives. The UsageAccumulator receives this via Channel C and increments Redis counters atomically using HINCRBY/HINCRBYFLOAT on `session:{id}:usage`.

---

## Section 15.2 — Gateway → Client Messages (Protocol Reference)

**Change 29: Update the protocol table to reflect the correct event semantics**
- `text_done` — currently described as "Full assembled text response for this turn." This must be updated: in the new architecture without `text_response_mode`, `text_done` is still sent to the client at `TurnCompleteEvent` time, but the full text it carries is assembled by the Protocol Handler's own lightweight accumulation of deltas it forwarded. The client still receives a final `text_done` with the full text as a convenience marker. However, this assembly in the Protocol Handler is a simple passthrough accumulation only for the purpose of sending the final `text_done` event — it is separate from TextAccumulator's Redis write path and must not be confused with it.
- `audio_done` — keep as-is, triggered by `TurnCompleteEvent` on audio turns.
- Remove any reference to `TextDoneEvent` or `AudioDoneEvent` as internal events in the protocol documentation. These are internal adapter events that no longer exist.
- Add `tool_invocation` event to the table: `{ "type": "tool_invocation", "source": "builtin" | "gateway" | "client", "tool_name": string }` — sent for all tool calls regardless of type so the End Client can show appropriate UI indicators.

---

## New Section Required: Provider Fallback

**Change 30: Add a dedicated "Provider Fallback" section to the SRS**
This section does not exist in the current SRS. It must be added covering:
- The trigger: `ProviderDisconnectedEvent` from the adapter during an active session
- Client behavior during reconnect: `provider_reconnecting` event, connection stays open
- The reconnect sequence as described in Change 20
- The dual-history strategy and why it enables cross-provider fallback
- Compatible fallback pairs (audio-to-audio, text-to-text enforcement at config time)
- The `secondary_project_config` configuration and its UI in the project configuration wizard
- Failure case: if fallback also fails, send `{ "type": "error", "code": "provider_unavailable" }` and proceed to normal session teardown
- The 30-second reconnect window referenced in footnote [i] must be clarified: this applies to node failure (client must reconnect with new token), not provider fallback (client stays connected during provider fallback — no new token needed)

---

## Summary Table

| Change # | SRS Section | Nature of Change                                                    |
| -------- | ----------- | ------------------------------------------------------------------- |
| 1        | 6.3         | Remove delta assembler from Protocol Handler                        |
| 2        | 6.3         | Add EventDispatcher component                                       |
| 3        | 6.3         | Add TextAccumulator component                                       |
| 4        | 6.3         | Add UsageAccumulator component                                      |
| 5        | 6.3         | Clarify Protocol Handler ↔ Adapter direct method call relationship  |
| 6        | 7.5         | Rename to Parallel Processing Model; describe 4 parallel loops      |
| 7        | 7.5         | Add RecordUserTurn call in Client→Provider loop                     |
| 8        | 7.5         | Remove usage accumulation from dual async loops                     |
| 9        | 7.6         | Remove delta assembly from adapter                                  |
| 10       | 7.6         | Replace TextDoneEvent write trigger with TurnCompleteEvent          |
| 11       | 7.6         | Introduce dual Redis key strategy                                   |
| 12       | 7.6         | WS Bridge adapters now write history                                |
| 13       | 7.6         | Responses API writes full content history alongside response_id     |
| 14       | 7.6         | Introduce IHistoryFormatter injection pattern                       |
| 15       | 7.6         | Define TextAccumulator per-turn state and flush rules               |
| 16       | 12.2        | Add new Redis key entries                                           |
| 17       | 7.7         | Update teardown step 1 for UsageAccumulator Redis path              |
| 18       | 7.7         | Add channel drain step before teardown                              |
| 19       | 7.7         | Delete both history keys at teardown                                |
| 20       | 7.13        | Full provider fallback mechanism                                    |
| 21       | 12.1        | Add secondary_project_config to data model                          |
| 22       | 8.1         | Remove assembly/history from adapter interface contract             |
| 23       | 8.1         | Remove TextDoneEvent and AudioDoneEvent                             |
| 24       | 8.1         | Add ProviderDisconnectedEvent                                       |
| 25       | 8.1         | Remove SessionStartedEvent and SessionEndedEvent from adapter       |
| 26       | 8.2         | Update WS Bridge adapter description for history                    |
| 27       | 8.3         | Remove delta assembly from all adapter implementation descriptions  |
| 28       | 8.5         | Replace usage payload with standalone UsageEvent + UsageAccumulator |
| 29       | 15.2        | Update protocol table; add tool_invocation event                    |
| 30       | New         | Add Provider Fallback section                                       |