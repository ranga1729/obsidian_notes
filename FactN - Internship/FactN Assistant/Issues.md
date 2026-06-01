# Streaming or complete responses ? 
**Feature :** Let the user decide if they want streaming text responses or complete text responses

**Context :**
-  Adapter layer receives provider responses. then they will be normalized into `LLMEvents` objects. Then they will be emit to the protocol handler as they arrive.
- Protocol handler consumes `LLMEvents`. Serialize them to client-facing JSON standard. Sends to the client. 
- Redis history must store only fully assembled texts, never streaming chunks. 

**Issue :**
When user wants streaming responses,
```text
Provider send incremental responses --> Adapter layers normalizes them and send to the protocol handler --> protocol handler serialized LLMEvents to JSON protocol --> send to client.
```
To preserve history, we can run a asynchronous function to concatenate responses' text values and put it into Redis as a whole text along with the user's text query. 

In this scenario, we are prioritizing sending responses to the users and response concatenation is not affecting to the main job. 

When user user wants complete responses,
```text
Provider send incremental responses --> We must concatenate them in one place. --> Complete text will be send to the client & Redis.
```
In this scenario, we need to sequentially do the concatenation first and send the whole response to the client. 

**Solution :**
- We need to have a new property in the project config called "`text_response_mode`" and has two values, `streaming` or `buffered`
- Adapter layer must only focus on normalization only and emit `LLMEvents` as fast as they arrive. 

Text Accumulator,
```text
FOR EACH LLMEvent from eventStream from the adapter layer:

IF event is TextDeltaEvent:
    IF mode == "streaming":
      // Append to in redis buffer (fire & forget)
      Redis.APPEND(session:{sessionId}:text_buffer, event.delta)  // Async, non-blocking
    ELSE IF mode == "buffered":
      // Accumulate in local memory
      localStringBuilder.Append(event.delta)
  
  ELSE IF event is TextDoneEvent:
    IF mode == "streaming":
      // Redis already has accumulated deltas  
      Move them into the session history
      Redis.SET(session:{sessionId}:history_turn, event.text)
    ELSE IF mode == "buffered":
      // Write the locally accumulated text to Redis
      Redis.SET(session:{sessionId}:history_turn, localStringBuilder.ToString())
      //Also send the accumulated text to the protocol handler.
    
    // Reset for next turn
    localStringBuilder.Clear()
```

If the response mode is 

----
**Events the Adapter Emits (Only):**
```text
// Text streaming
TextDeltaEvent      { Text: string }
// (No TextDoneEvent — TurnCompleteEvent is the signal)

// Audio streaming  
AudioDeltaEvent     { Audio: string (base64), SampleRate: int }
// (No AudioDoneEvent — TurnCompleteEvent is the signal)

// Transcriptions (async, may arrive out of order)
TranscriptEvent     { Role: "user" | "assistant", Text: string }

// Tool interactions
ToolCallEvent            { ToolCallId: string, Name: string, Arguments: object }
ToolCycleCompleteEvent   { ToolCallId: string }  // adapter resumed after tool result

// Usage — RESTORED, was missing
UsageEvent          { InputTokens: int, OutputTokens: int, 
                      AudioInSecs: float, AudioOutSecs: float }

// Turn lifecycle — one signal for all content types
TurnCompleteEvent   { }

// Provider-initiated disconnect — replaces SessionEndedEvent
ProviderDisconnectedEvent { Reason: string }

// REMOVED: SessionStartedEvent  ← owned by Connection Manager
// REMOVED: SessionEndedEvent    ← owned by Connection Manager / State Machine
```

Different LLM providers end makers,

|**API Name**|**Protocol**|**Primary "End of Stream" Indicator**|**Secondary/Sub-Markers**|
|---|---|---|---|
|**Gemini Live API**|WebSocket|**`turn_complete: true`**|`generation_complete: true` (Model finished talking, but turn might stay open for audio).|
|**OpenAI Realtime API**|WebSocket|**`response.done`**|`response.text.done`, `response.audio.done`, `response.output_item.done`.|
|**Google `generateContent`**|HTTP (SSE)|**`finishReason: "STOP"`**|Found in the `candidates[0]` object of the final JSON chunk.|
|**OpenAI Chat Completion**|HTTP (SSE)|**`finish_reason: "stop"`**|The very last chunk is typically the string `data: [DONE]`.|
|**OpenAI Responses API**|HTTP (SSE)|**`response.completed`**|`response.output_item.done`, `response.content_part.done`.|

****
