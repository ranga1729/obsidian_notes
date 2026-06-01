Findings: 
We need to focus on mainly 3 entities.
1. AI API provider
2. API
3. Model

Ex: 
Provider: Google
API: Live API
Model: gemini-live-preview

Based on the selection combination, these are the considerations get differ, 
1. Modalities(Input/output)
	text in, text out
	audio in, audio out
2. Interaction mode/Transport layer
	real-time(WebSocket/WebRTC)
	streaming(SSE/chunked HTTP)
3. Intelligence features
	function calling/tools
	structured output(JSON mode)
	memory(stateful vs stateless)
	multimodal reasoning
4. Voice-specific
	VAD(voice activity detection)
	speech-to-speech(no STT/TTS split)
	interrupt handling
5. Constraints
	Context window
	rate limits
	pricing model
	model lifecycle
6. Memory
	Stateless/stateful connections
	Conversation Id chaining


Look at the [[AI API Provider Analysis]]
Summary table of the API provider analysis is, 

| API                             | Provider | Protocol  | Text In | Text Out | Audio In | Audio Out | Video In | Image In | Image Out | VAD | Tool Calling | Stateful | Streaming | Real-time |
| ------------------------------- | -------- | --------- | ------- | -------- | -------- | --------- | -------- | -------- | --------- | --- | ------------ | -------- | --------- | --------- |
| **Realtime API**                | OpenAI   | WebSocket | ✅       | ✅        | ✅        | ✅         | ❌        | ✅        | ❌         | ✅   | ✅            | Session  | ✅         | ✅         |
| **Responses API**               | OpenAI   | HTTPS/SSE | ✅       | ✅        | ✅        | ✅         | ❌        | ✅        | ✅         | ❌   | ✅            | ✅        | ✅         | ❌         |
| **Chat Completions API**        | OpenAI   | HTTPS/SSE | ✅       | ✅        | ✅        | ✅         | ❌        | ✅        | ❌         | ❌   | ✅            | ❌        | ✅         | ❌         |
| **Gemini Live API**             | Google   | WebSocket | ✅       | ✅        | ✅        | ✅         | ✅        | ✅        | ❌         | ✅   | ✅            | Session  | ✅         | ✅         |
| **Gemini Generate Content API** | Google   | HTTPS/SSE | ✅       | ✅        | ✅        | ⚠️        | ✅        | ✅        | ✅         | ❌   | ✅            | ❌        | ✅         | ❌         |

Our WebSocket protocol must have standards to handle,
- Text input
- Text output
- Audio input
- Audio output
- Transcription of audio input
- Transcription of audio output
- Interruption handling
- VAD control
- Tool calling
- Memory management(session or id chaining)
