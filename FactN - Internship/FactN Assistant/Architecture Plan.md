## High-level System Architecture: [link](https://drive.google.com/file/d/12gGpOL57WceT8FZB2QIIsGY96EFyIyNZ/view?usp=drive_link)

client will connect to the service via two interfaces,
1. Project Dashboard
2. A websocket connection

Clients will connect to a NGINX API gateway. 
This API gateway will utilize two main services,
1. REST API services(for dashboard and other management tasks)
2. WebSocket gateway service

#### WebSocket service
Users will connect to the Websocket gateway service via one of below methods,
```http
wss://chatn.com/ws?api_key=<api key>
```
OR
```http
wss://chatn.com/ws?token=<ephemeral token>
```

All the connection request come to the Websocket service will be,
1. Checked by the "Connection Manager"
	Validate the API key/ephemeral toke.
	Load the project config.
	Start the session life cycle.
If the connection manager validate the connection request, client's application will start a websocket connection with the Websocket service. 

Websocket service maintains a websocket protocol. 
This works as an abstraction layer to the LLM API provider. 

**Client --> Websocket service**
```json
// Text message
{ "type": "text_input", "text": "Hello" }

// Audio chunk - OpenAI Realtime API
{ "type": "audio_input", "audio": "<base64>", "sample_rate": 16000 }

// Return tool result
{ "type": "tool_result", "tool_call_id": "call_abc123", "result": { "temp": 22 } }

// Interrupt model speech
{ "type": "interrupt" }

// Keepalive
{ "type": "ping" }
```

**Gateway --> Client**
```json
// Session ready
{ "type": "session_started", "session_id": "uuid" }

// Streaming text
{ "type": "text_delta", "delta": "Hello" }

// Streaming audio
{ "type": "audio_delta", "audio": "<base64>", "sample_rate": 24000 }

// Tool invocation (client must execute and return result)
{ "type": "tool_call", "tool_call_id": "call_abc123", "name": "get_weather", "arguments": { "city": "London" } }

// Transcripts
{ "type": "transcript_delta", "role": "user", "text": "Hi" }
{ "type": "transcript_delta", "role": "assistant", "text": "Hello there!" }

// turn complete
{"type": "turn_complete"}

// Errors
{ "type": "error", "code": "rate_limit_exceeded", "message": "..." }
{ "type": "error", "code": "provider_error", "message": "...", "retryable": true }

// Pong
{ "type": "pong" }
```
WebSocket workflow: [link](https://drive.google.com/file/d/1LstDz0k3sCDdlmEgxuiU19Kpv4U9-pCf/view?usp=sharing)

## Database Design
Both REST API service and the Websocket Gateway service will use a shared DB infrastructure which is a combination of PostgreSQL and Redis(for caching/rate limiting/temporal data).
ERD: [link](https://drive.google.com/file/d/1AVqLdQmzS1NyIimpdYipvzXnIv-qsN3K/view?usp=sharing)

## Design patterns and detailed plan
For chat applications I'm considering 5 APIS for now,
- OpenAI Realtime API
- OpenAI Response API
- OpenAI chat completion API
- Gemini Live PAI
- Gemini generate Content API/with streaming

| API                        | Behavior                                                                                                                                                                     |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OpenAI Realtime API        | Opens a websocket to the OpenAI WSS.                                                                                                                                         |
| OpenAI Response API        | HTTP POST per turn. Streams SSE response. Maintains `previous_response_id` for state.                                                                                        |
| OpenAI chat completion API | HTTP POST per turn. Streams SSE response. No built-in mechanism for memory. Websocket gateway can utilize the Redis to maintain message history array. (keyed by session Id) |
| Gemini Live API            | Similar to Realtime API.                                                                                                                                                     |
| Gemini Generate content    | HTTP POST per turn. Stream SSE. No mechanism for memory. Utilize Redis for message history array.(keyed by session_id)                                                       |

The most fundamental difference between these APIs are the way their connection mode and the strategy of state conversation state management,

| Provider + API          | Connection Mode | Who holds state ?               |
| ----------------------- | --------------- | ------------------------------- |
| OpenAI Realtime API     | WS              | Provider(OpenAI)                |
| Gemini Live API         | WS              | Provider(OpenAI)                |
| OpenAI Chat completions | HTTP            | Redis                           |
| OpenAI Response API     | HTTP            | Provider - response id chaining |
| Gemini Generate Content | HTTP            | Redis                           |
![[Pasted image 20260427102134.png]]


#### Tech Stack

| Layer             | Tech                                                | Reason                                                                                 |
| ----------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Frontend          | NextJS with App router                              | SSR for dashboard, RSC and API routes for BFF pattern + Typescript + Tailwind + ShadCN |
| REST API Backend  | ASP>NET Core Web API                                | C# familiarity + performance of .NET APIs                                              |
| WebSocket Gateway | ASP.NET Core + SignalR(or raw websocket middleware) | Integrate SignalR for WebSocket management                                             |
| Primary DB        | PostgreSQL                                          |                                                                                        |
| Cache             | Redis                                               | Rate limiting, pub/sub for distributed WS nodes, ephemeral token store                 |
| Message broker    | RabbitMQ(optional)                                  | async tool execution, usage event processing                                           |
| API gateway       | nginx                                               | WebSocket proxying                                                                     |
