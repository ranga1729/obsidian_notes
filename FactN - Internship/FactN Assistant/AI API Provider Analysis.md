## 🔵 PROVIDER: OPENAI

---
### API 1: OpenAI Realtime API

| Attribute            | Details                                                                     |
| -------------------- | --------------------------------------------------------------------------- |
| **Protocol**         | WebSocket (WSS) — persistent, bidirectional                                 |
| **Primary Use Case** | Real-time, low-latency voice agents, speech-to-speech AI, live conversation |
| **Status**           | Active / Generally Available                                                |
| **Official Docs**    | https://platform.openai.com/docs/guides/realtime                            |

#### ✅ Features & Capabilities

| Feature                            | Supported | Notes                                                      |
| ---------------------------------- | --------- | ---------------------------------------------------------- |
| **Text Input**                     | ✅         | Via WebSocket messages                                     |
| **Text Output**                    | ✅         | Text + delta streaming                                     |
| **Audio Input**                    | ✅         | Raw PCM16 audio, 24kHz                                     |
| **Audio Output**                   | ✅         | Natural speech synthesis, emotionally expressive           |
| **Image Input**                    | ✅         | Models can discuss images during live voice sessions       |
| **Video Input**                    | ❌         | Not natively supported                                     |
| **Voice Activity Detection (VAD)** | ✅         | Server-side + client-configurable; handles interruptions   |
| **Interruption Handling**          | ✅         | Model stops speaking when user interrupts                  |
| **Tool / Function Calling**        | ✅         | Sync & async; actions can run while conversation continues |
| **Streaming**                      | ✅         | Full bidirectional streaming over WebSocket                |
| **Noise Reduction**                | ✅         | Built-in audio noise filtering                             |
| **Multi-Language**                 | ✅         | Code-switching, mid-sentence language changes              |
| **SIP Telephony Integration**      | ✅         | Direct SIP integration for real phone calls                |
| **Remote MCP Server**              | ✅         | Connect to external context/tool servers                   |
| **Session Management**             | ✅         | Session-level state maintained over WebSocket connection   |
| **Structured Output**              | ⚠️        | Limited compared to Responses API                          |
| **Multi-Speaker Differentiation**  | ❌         | Cannot separately track multiple simultaneous speakers     |

#### 🤖 Supported Models

| Model ID                             | Notes                                                        |
| ------------------------------------ | ------------------------------------------------------------ |
| `gpt-4o-realtime-preview`            | High quality, multimodal, multilingual, feature-rich preview |
| `gpt-4o-realtime-preview-2024-12-17` | Date-stamped stable version                                  |
| `gpt-4o-mini-realtime-preview`       | Lightweight, cost-effective realtime model                   |
| `gpt-realtime`                       | Latest flagship speech-to-speech model (2025)                |
| `gpt-realtime-mini`                  | Cheaper, lower-latency variant                               |

#### ⚠️ Limitations
- No multi-speaker differentiation
- WebSocket-only — no REST fallback for streaming
- Context window constraints in long audio sessions
- Expensive: ~$32/1M audio input tokens, ~$64/1M audio output tokens
- Requires robust audio pipeline engineering on client side

---
### API 2: OpenAI Responses API

| Attribute | Details |
|---|---|
| **Protocol** | HTTPS REST (with SSE streaming) |
| **Primary Use Case** | Agentic AI, stateful multi-turn conversations, complex tool orchestration |
| **Status** | Active / Generally Available — **the new standard**, replacing Assistants API |
| **Official Docs** | https://platform.openai.com/docs/api-reference/responses |

#### ✅ Features & Capabilities

| Feature | Supported | Notes |
|---|---|---|
| **Text Input** | ✅ | Rich structured JSON prompts |
| **Text Output** | ✅ | Streaming (SSE) + non-streaming |
| **Audio Input** | ✅ | Model-dependent |
| **Audio Output** | ✅ | Model-dependent |
| **Image Input** | ✅ | Multimodal prompts with images |
| **Image Output** | ✅ | Native image generation via `gpt-image-1` |
| **Video Input** | ❌ | Not natively supported |
| **Stateful Conversations** | ✅ | Via `previous_response_id` parameter |
| **Tool / Function Calling** | ✅ | Web search, file search, code interpreter, computer use, custom functions |
| **Parallel Tool Use** | ✅ | Multiple tools in one call |
| **Remote MCP Integration** | ✅ | Connect to Stripe, Shopify, Twilio, HubSpot, Zapier, etc. |
| **Structured Output (JSON Schema)** | ✅ | Strict JSON responses |
| **Streaming** | ✅ | SSE token-by-token streaming |
| **Async / Background Mode** | ✅ | Long-running async tasks |
| **Encrypted Reasoning** | ✅ | Privacy-sensitive enterprise deployments |
| **Context Caching** | ✅ | For cost efficiency on repeated prompts |
| **Voice Activity Detection** | ❌ | Not applicable for REST API |

#### 🤖 Supported Models

| Model ID | Notes |
|---|---|
| `gpt-4o` | High accuracy multimodal model |
| `gpt-4o-mini` | Cost-effective, fast |
| `gpt-4.1` | High-throughput tasks |
| `o1`, `o3`, `o3-mini`, `o4-mini` | Chain-of-thought reasoning models |
| `gpt-5`, `gpt-5-mini`, `gpt-5-nano` | Latest (2025), logic-heavy, complex planning |

#### ⚠️ Limitations
- No WebSocket/real-time audio streaming (use Realtime API for that)
- MCP integration requires correct server-side configuration
- Enterprise features (encryption, background mode) may require paid tiers
- No visual playground yet (code/SDK only)
- Assistants API (predecessor) deprecated August 2026 — migrate now

---
### API 3: OpenAI Chat Completions API

| Attribute | Details |
|---|---|
| **Protocol** | HTTPS REST (with SSE streaming option) |
| **Primary Use Case** | General-purpose text generation, chatbots, Q&A, summarization |
| **Status** | Active — but being superseded by Responses API for complex use cases |
| **Official Docs** | https://platform.openai.com/docs/api-reference/chat |

#### ✅ Features & Capabilities

| Feature | Supported | Notes |
|---|---|---|
| **Text Input** | ✅ | System/user/assistant message roles |
| **Text Output** | ✅ | Streaming + non-streaming |
| **Audio Input** | ✅ | With audio-capable models |
| **Audio Output** | ✅ | With audio-capable models |
| **Image Input** | ✅ | Multimodal prompts |
| **Image Output** | ❌ | Use DALL·E or image generation endpoint |
| **Tool / Function Calling** | ✅ | Synchronous function calling |
| **Parallel Tool Use** | ✅ | GPT-4 and above |
| **Streaming (SSE)** | ✅ | Token-by-token streaming |
| **Fine-Tuning Support** | ✅ | Domain-specific custom models |
| **Structured Output** | ✅ | JSON mode |
| **Stateful Memory** | ❌ | Stateless — developer must resend full history |
| **VAD** | ❌ | Not applicable |
| **Real-time Web Access** | ⚠️ | Only if tools/plugins enabled |

#### 🤖 Supported Models
All GPT-4o, GPT-4.1, GPT-4-turbo, o-series, and GPT-5 family models are supported.

#### ⚠️ Limitations
- Stateless — no built-in memory between calls
- No native persistent agents/threads (use Responses API instead)
- Context window is finite; very long conversations truncate early messages
- No audio pipeline (use Realtime API for voice)
- Being superseded by Responses API for advanced use cases

---
### API 4: OpenAI Assistants API *(Deprecating)*

| Attribute | Details |
|---|---|
| **Protocol** | HTTPS REST |
| **Status** | ⛔ **Deprecated August 2025 — Shutting down August 26, 2026** |
| **Replacement** | Responses API |

> **Do not use**.

---

## 🔴 PROVIDER: GOOGLE (Gemini)

---
### API 5: Google Gemini Live API

| Attribute | Details |
|---|---|
| **Protocol** | WebSocket (WSS) — stateful, persistent, bidirectional |
| **Primary Use Case** | Real-time voice/video AI agents, live conversations, interactive assistants |
| **Status** | Active / Generally Available (some models in Preview) |
| **Official Docs** | https://ai.google.dev/gemini-api/docs/live-api |

#### ✅ Features & Capabilities

| Feature | Supported | Notes |
|---|---|---|
| **Text Input** | ✅ | Via WebSocket JSON messages |
| **Text Output** | ✅ | Streamed text responses |
| **Audio Input** | ✅ | 16-bit PCM, 16kHz |
| **Audio Output** | ✅ | 24kHz, natural/expressive speech |
| **Image Input** | ✅ | JPEG frames, 1FPS from video/camera |
| **Video Input** | ✅ | Screen share + camera streaming |
| **Voice Activity Detection (VAD)** | ✅ | Server-side VAD + client-triggered |
| **Interruption Handling** | ✅ | Model stops when user interrupts |
| **Tool / Function Calling** | ✅ | Sync & async; Google Search grounding included |
| **Streaming** | ✅ | Full bidirectional over WebSocket |
| **Audio Transcription** | ✅ | Real-time transcripts of user & model audio |
| **Affective Dialog** | ✅ | Adapts vocal tone to user emotions/context |
| **Session Resumption** | ✅ | Reconnect after network drops; 24-hour retention |
| **Context Compression** | ✅ | Longer memory spans within a session |
| **Proactive Audio** | ✅ | Model decides when/whether to respond |
| **Ephemeral Tokens** | ✅ | Secure client-side auth (replaces static API keys) |
| **WebRTC Support** | ⚠️ | Via 3rd party (e.g., LiveKit) — not native |
| **Multi-Language** | ✅ | 90+ languages on latest models |
| **Text-Only Response Mode** | ⛔ | `response_modality="TEXT"` only mode **DEPRECATED** |

#### 🤖 Supported Models

| Model ID | Notes |
|---|---|
| `gemini-2.5-flash-live` | Current stable Live API model; 128K context, native audio |
| `gemini-2.5-flash-native-audio` | Mainline 2025 model; affective dialog, thinking, proactive audio |
| `gemini-live-2.5-flash-preview` | Preview; cutting-edge capabilities |
| `gemini-3.1-flash-live` | Latest (2026 preview); 90+ languages, highest audio quality |
| `gemini-2.0-flash-live-001` | ⚠️ Older, being deprecated — migrate away |

#### ⚠️ Limitations
- WebSocket-only — no REST polling for streaming
- `response_modality="TEXT"` only mode is **deprecated** (must support audio or both)
- Preview models have stricter rate limits and short deprecation notice windows
- No native WebRTC — must use third-party bridges (LiveKit, Daily.co)
- Billing/quota varies significantly between GA and Preview model tiers
- For production: must use ephemeral tokens (extra setup step)

---

### API 6: Google Gemini API (Generate Content / REST)
[generateContent](https://ai.google.dev/gemini-api/docs/text-generation) / [streamGenerateContent](https://ai.google.dev/gemini-api/docs/text-generation#streaming-responses)

| Attribute | Details |
|---|---|
| **Protocol** | HTTPS REST (with SSE streaming) |
| **Primary Use Case** | Text generation, chatbots, multimodal Q&A, summarization, code generation |
| **Status** | Active / Generally Available |
| **Official Docs** | https://ai.google.dev/gemini-api/docs |

#### ✅ Features & Capabilities

| Feature                           | Supprted | Notes                                                 |
| --------------------------------- | -------- | ----------------------------------------------------- |
| **Text Input**                    | ✅        | Rich structured JSON prompts                          |
| **Text Output**                   | ✅        | Streaming (SSE) + non-streaming                       |
| **Audio Input**                   | ✅        | Model-dependent                                       |
| **Audio Output**                  | ⚠️       | Only on Native Audio / Live models                    |
| **Image Input**                   | ✅        | Inline or file-referenced images                      |
| **Video Input**                   | ✅        | Video file references                                 |
| **Document Input**                | ✅        | PDF, text files via File API                          |
| **Image Output**                  | ✅        | Gemini 2.0+ (Imagen integration)                      |
| **Tool / Function Calling**       | ✅        | Custom tools, Google Search grounding, code execution |
| **Streaming (SSE)**               | ✅        | Token-by-token streaming                              |
| **System Instructions**           | ✅        | Persona, role, behavior steering                      |
| **JSON Mode / Structured Output** | ✅        | Response MIME type, schema enforcement                |
| **Context Caching**               | ✅        | Explicit and implicit caching                         |
| **Thinking Mode**                 | ✅        | `thinkingConfig: {thinkingLevel: "LOW/MEDIUM/HIGH"}`  |
| **Multi-Turn Chat**               | ✅        | Stateless (developer resends history)                 |
| **Safety Settings**               | ✅        | Per-category block thresholds                         |
| **Fine-Tuning**                   | ✅        | On select models via Vertex AI                        |
| **VAD**                           | ❌        | Not applicable for REST                               |

#### 🤖 Supported Models

| Model ID | Context Window | Output Tokens | Notes |
|---|---|---|---|
| `gemini-2.5-pro` | 1M tokens | 65,536 | Deep reasoning, complex tasks |
| `gemini-2.5-flash` | 1M tokens | 65,536 | Fast, cost-effective |
| `gemini-2.5-flash-lite` | 1M tokens | 32,768 | Lightest/fastest |
| `gemini-2.0-flash` | 1M tokens | 8,192 | Legacy, cost-efficient |
| `gemini-2.0-flash-lite` | 1M tokens | 8,192 | Lightweight legacy |
| `gemini-3.1-pro` *(Preview)* | 1M tokens | 65,536 | Advanced reasoning, agentic, coding |
| `gemini-3.1-flash` *(Preview)* | 1M tokens | 65,536 | Fast + capable, multimodal |
| `gemini-3.1-flash-lite` *(Preview)* | 1M tokens | 32,768 | Speed-optimized |

#### ⚠️ Limitations
- Stateless — no built-in conversation memory (developer manages history)
- Audio output only via native audio/Live API models
- Preview models subject to change and rate limits
- Vertex AI (Google Cloud) and AI Studio (`ai.google.dev`) have different authentication and quota systems

---

## 📊 Master Comparison Table

| API | Provider | Protocol | Text In | Text Out | Audio In | Audio Out | Video In | Image In | Image Out | VAD | Tool Calling | Stateful | Streaming | Real-time |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Realtime API** | OpenAI | WebSocket | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | Session | ✅ | ✅ |
| **Responses API** | OpenAI | HTTPS/SSE | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |
| **Chat Completions API** | OpenAI | HTTPS/SSE | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |
| **Gemini Live API** | Google | WebSocket | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | Session | ✅ | ✅ |
| **Gemini Generate Content API** | Google | HTTPS/SSE | ✅ | ✅ | ✅ | ⚠️ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ❌ |

