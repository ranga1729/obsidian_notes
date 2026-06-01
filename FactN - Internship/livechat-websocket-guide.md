# LiveChat API — WebSocket Integration Guide

> This guide covers everything you need to connect your application to the LiveChat API's real-time chat endpoint. Your project settings (system prompt, voice, tools) are configured separately via the dashboard — this guide focuses entirely on the `/v1/chat` WebSocket connection.

---

## Table of Contents

1. [How It Works](#1-how-it-works)
2. [Authentication](#2-authentication)
3. [Connecting](#3-connecting)
4. [Messages You Send](#4-messages-you-send)
5. [Messages You Receive](#5-messages-you-receive)
6. [Voice Turns](#6-voice-turns)
7. [Text Turns](#7-text-turns)
8. [Session Resumption](#8-session-resumption)
9. [Connection Close Codes](#9-connection-close-codes)
10. [Complete Examples](#10-complete-examples)
11. [Production Checklist](#11-production-checklist)

---

## 1. How It Works

When your application opens a WebSocket connection to `/v1/chat`, the platform:

1. Validates your API key or ephemeral token
2. Opens a Gemini Live session using your project's configuration (system prompt, voice, tools)
3. Bridges all messages between your client and Gemini in real time
4. Keeps the session alive on the server even if the client temporarily disconnects

Your client communicates with the platform over one WebSocket connection. You never talk to Gemini directly — the platform handles all of that complexity.

```
Your App                 LiveChat Platform              Gemini
   │                           │                          │
   │── WebSocket connect ──────►│                          │
   │                           │── Gemini session open ──►│
   │◄── session_ready ─────────│                          │
   │                           │                          │
   │── voice_start ────────────►│                          │
   │── [PCM audio chunks] ─────►│── audio stream ─────────►│
   │── voice_end ──────────────►│                          │
   │                           │◄── text + audio ─────────│
   │◄── assistant_text ─────────│                          │
   │◄── [PCM audio chunks] ─────│                          │
   │◄── turn_complete ──────────│                          │
```

---

## 2. Authentication

You need a credential to open a connection. There are two types.

### API Key (Development)

An API key looks like `pk_live_Xy7mQ3rN9vP2...`. You get it from the dashboard under **API Keys**. Pass it as a query parameter:

```
wss://your-host/v1/chat?api_key=pk_live_...
```

This is the quickest way to get started during development. For production browser applications, use ephemeral tokens instead (see below).

### Ephemeral Token (Production)

An ephemeral token is a short-lived, single-use credential. Your backend server mints a token using your secret key and passes it to the browser. The browser uses the token to open the WebSocket — the token is consumed on first use and cannot be replayed.

```
wss://your-host/v1/chat?token=<ephemeral_token>
```

**Why use ephemeral tokens in production?** An API key embedded in browser code can be extracted by anyone who views your page source. Ephemeral tokens expire in seconds and self-destruct on use, so there is nothing useful to steal.

**Token flow:**

```
Browser                Your Server              LiveChat Platform
   │                        │                         │
   │── "I need a token" ────►│                         │
   │                        │── POST /v1/tokens ──────►│
   │                        │◄── { token, expires_at } ─│
   │◄── token ──────────────│                         │
   │                        │                         │
   │── WebSocket ?token=... ──────────────────────────►│
   │◄── session_ready ────────────────────────────────│
```

Your server-side token minting request:

```bash
POST /v1/tokens
Authorization: Bearer sk_live_...   # your secret key, never in the browser
Content-Type: application/json

{ "ttl_seconds": 60 }
```

Response:

```json
{
  "ephemeral_token": "abc123xyz...",
  "expires_at": 1736942400,
  "ttl_seconds": 60,
  "project_id": "..."
}
```

Pass `ephemeral_token` to your browser and connect immediately — you have 60 seconds (or whatever TTL you set, up to 300).

---

## 3. Connecting

### Connection URL

```
wss://your-host/v1/chat?api_key=pk_live_...
wss://your-host/v1/chat?token=<ephemeral_token>
```

Optionally append `&session_id=<uuid>` to resume an existing session (see [Section 8](#8-session-resumption)).

### The First Thing You Receive

After connecting, wait for a `session_ready` message before sending anything. This confirms that the Gemini session is open and your project configuration has been loaded.

```json
{
  "type": "session_ready",
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "speaker_mode": false,
  "project_id": "...",
  "project_name": "My Assistant"
}
```

**Save the `session_id`.** You'll need it to resume the session if the connection drops.

### Minimal Connection Example

```javascript
const ws = new WebSocket('wss://your-host/v1/chat?api_key=pk_live_...');
ws.binaryType = 'arraybuffer'; // required for PCM audio

ws.onopen = () => {
  console.log('WebSocket connected, waiting for session...');
};

ws.onmessage = (evt) => {
  // Binary frames are PCM audio from Gemini
  if (evt.data instanceof ArrayBuffer) {
    playAudio(evt.data);
    return;
  }

  const msg = JSON.parse(evt.data);

  if (msg.type === 'session_ready') {
    console.log('Session open:', msg.session_id);
    // Enable your UI here — safe to start sending messages
  }
};

ws.onclose = (evt) => {
  console.log(`Closed with code ${evt.code}: ${evt.reason}`);
};
```

---

## 4. Messages You Send

All messages are JSON objects with a `type` field. The one exception is PCM audio, which is sent as raw binary frames.

### `text_input`

Send a text message and trigger a response from Gemini.

```json
{ "type": "text_input", "text": "What's the weather like in Paris?" }
```

Gemini will respond with one or more `assistant_text` messages followed by a `turn_complete`.

---

### `voice_start`

Signal that the user has started speaking. Must be sent before any audio data.

```json
{ "type": "voice_start" }
```

---

### Binary PCM audio

Raw microphone audio sent between `voice_start` and `voice_end`.

**Required format:** 16-bit signed integers, little-endian, 16 kHz sample rate, mono channel.

```javascript
// Send a chunk of PCM audio
ws.send(pcmBuffer); // ArrayBuffer containing 16-bit PCM samples at 16kHz
```

You can send audio in any chunk size. The platform streams it to Gemini as it arrives. Typical chunk sizes are 4096 samples (~256ms at 16kHz).

---

### `voice_end`

Signal that the user has stopped speaking.

```json
{ "type": "voice_end" }
```

The platform sends `activity_end` to Gemini, which then processes everything it received and begins generating its response.

---

### `set_speaker`

Toggle whether the server sends Gemini's audio output to your client. When `false`, you still receive text transcriptions — just no audio.

```json
{ "type": "set_speaker", "enabled": true }
```

Disable speaker mode if your user is in a noisy environment where audio output would be disruptive, or if your application is text-only.

---

### `interrupt`

Cancel the current active voice turn without ending the session. Useful if the user wants to rephrase or the application needs to cut Gemini off.

```json
{ "type": "interrupt" }
```

---

### `ping`

Keepalive message. The server responds with `pong`. Use this to verify the connection is still alive during long idle periods.

```json
{ "type": "ping" }
```

---

## 5. Messages You Receive

### `session_ready`

Confirms the Gemini session is open. Wait for this before sending anything.

```json
{
  "type": "session_ready",
  "session_id": "uuid",
  "speaker_mode": false,
  "project_id": "...",
  "project_name": "My Assistant"
}
```

---

### `assistant_text`

A chunk of text from Gemini's response. These arrive in real time as Gemini generates them — append each chunk to your UI as it arrives.

```json
{ "type": "assistant_text", "text": "The weather in Paris is currently " }
{ "type": "assistant_text", "text": "sunny with a high of 22°C." }
```

---

### `user_transcript`

Real-time transcription of what the user is saying (voice turns only). Arrives while the user is still speaking — useful for showing a live "you said..." indicator.

```json
{ "type": "user_transcript", "text": "What's the weather like in Paris?" }
```

---

### `turn_complete`

Gemini has finished responding for this turn. Use this to hide a loading indicator, re-enable the input, or do any post-turn housekeeping.

```json
{ "type": "turn_complete" }
```

---

### `tool_call`

A tool defined in your project was invoked and executed. This is informational — no action is required on your client.

```json
{
  "type": "tool_call",
  "tool": "get_weather",
  "args": { "city": "Paris" },
  "result": { "temperature": 22, "condition": "sunny" },
  "call_id": "fc_abc123"
}
```

---

### `speaker_mode_updated`

Confirms your `set_speaker` command was applied.

```json
{ "type": "speaker_mode_updated", "enabled": true }
```

---

### `error`

A non-fatal error occurred. The session remains open. Log it or show it to the user, but do not close the connection.

```json
{ "type": "error", "message": "Tool webhook returned HTTP 503" }
```

---

### `session_ended`

The Gemini session has been closed by the server (idle timeout, shutdown, or unrecoverable error). The WebSocket connection will close shortly after. If you want to continue, reconnect with the saved `session_id`.

```json
{ "type": "session_ended" }
```

---

### Binary PCM audio

Gemini's audio output. Only received when speaker mode is enabled (`set_speaker: true`).

**Format:** 16-bit signed integers, little-endian, 24 kHz sample rate, mono channel.

> Note the different sample rates: you **send** at 16 kHz, you **receive** at 24 kHz.

---

## 6. Voice Turns

A voice turn is a complete speak → respond cycle.

### The Full Sequence

```
Client                          Server
  │                               │
  │── voice_start ───────────────►│
  │── [binary PCM chunk] ─────────►│  } User speaks
  │── [binary PCM chunk] ─────────►│  } (send as many chunks as needed)
  │── [binary PCM chunk] ─────────►│  }
  │── voice_end ──────────────────►│
  │                               │
  │◄── user_transcript ────────────│  (what the user said)
  │◄── assistant_text ─────────────│  (Gemini's response text, streaming)
  │◄── [binary PCM] ──────────────│  (Gemini's audio, if speaker is on)
  │◄── turn_complete ──────────────│
```

### Capturing Microphone Audio

The Web Audio API gives you PCM samples. You need to resample to 16 kHz if the browser's default rate is different (typically 44.1 kHz or 48 kHz).

```javascript
async function startVoiceTurn(ws) {
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: {
      echoCancellation: true,
      noiseSuppression: true,
      sampleRate: 48000, // request 48kHz; we'll downsample to 16kHz
    }
  });

  const audioCtx = new AudioContext({ sampleRate: 48000 });
  const source = audioCtx.createMediaStreamSource(stream);
  const processor = audioCtx.createScriptProcessor(4096, 1, 1);

  source.connect(processor);
  processor.connect(audioCtx.destination);

  // Signal start
  ws.send(JSON.stringify({ type: 'voice_start' }));

  processor.onaudioprocess = (e) => {
    const float32 = e.inputBuffer.getChannelData(0);
    const pcm16 = downsampleAndConvert(float32, 48000, 16000);
    ws.send(pcm16.buffer);
  };

  // Return a stop function
  return () => {
    processor.disconnect();
    source.disconnect();
    stream.getTracks().forEach(t => t.stop());
    ws.send(JSON.stringify({ type: 'voice_end' }));
  };
}

function downsampleAndConvert(float32, fromRate, toRate) {
  const ratio = fromRate / toRate;
  const outLen = Math.floor(float32.length / ratio);
  const downsampled = new Float32Array(outLen);

  for (let i = 0; i < outLen; i++) {
    const pos = i * ratio;
    const idx = Math.floor(pos);
    const frac = pos - idx;
    downsampled[i] = float32[idx] + frac * ((float32[idx + 1] ?? 0) - float32[idx]);
  }

  // Convert float32 [-1,1] → int16 [-32768,32767]
  const int16 = new Int16Array(outLen);
  for (let i = 0; i < outLen; i++) {
    const s = Math.max(-1, Math.min(1, downsampled[i]));
    int16[i] = s < 0 ? s * 0x8000 : s * 0x7FFF;
  }

  return int16;
}
```

### Playing Gemini's Audio Response

When speaker mode is on, you receive binary PCM frames at 24 kHz. Use the Web Audio API to play them with gapless scheduling.

```javascript
let audioCtx = null;
let nextPlayAt = 0;

function playAudio(buffer) {
  if (!audioCtx) audioCtx = new AudioContext();
  if (audioCtx.state === 'suspended') audioCtx.resume();

  const int16 = new Int16Array(buffer);
  const float32 = new Float32Array(int16.length);
  for (let i = 0; i < int16.length; i++) {
    float32[i] = int16[i] / 32768.0;
  }

  const audioBuf = audioCtx.createBuffer(1, float32.length, 24000);
  audioBuf.copyToChannel(float32, 0);

  const source = audioCtx.createBufferSource();
  source.buffer = audioBuf;
  source.connect(audioCtx.destination);

  // Schedule gaplessly: each chunk starts where the last one ended
  const startAt = Math.max(audioCtx.currentTime + 0.02, nextPlayAt);
  source.start(startAt);
  nextPlayAt = startAt + audioBuf.duration;
}

// Reset the playback queue (e.g. when starting a new turn)
function resetAudio() {
  nextPlayAt = 0;
}
```

### Push-to-Talk vs Voice Activity Detection

The platform uses **manual VAD** by default: your client decides when the user starts and stops speaking, and sends `voice_start` / `voice_end` accordingly. This gives you the most control.

**Push-to-talk pattern** (simplest):

```javascript
holdButton.addEventListener('mousedown', () => stopVoice = startVoiceTurn(ws));
holdButton.addEventListener('mouseup', async () => (await stopVoice)());
```

**Silence detection pattern** (automatically ends the turn):

```javascript
let silenceTimer = null;

processor.onaudioprocess = (e) => {
  const rms = getRMS(e.inputBuffer.getChannelData(0));
  ws.send(pcmChunk);

  // End turn after 1.5s of silence
  clearTimeout(silenceTimer);
  if (rms < 0.01) {
    silenceTimer = setTimeout(() => stopVoice(), 1500);
  }
};
```

---

## 7. Text Turns

Text turns are simpler than voice turns. Send a `text_input` message and wait for `turn_complete`.

```javascript
function sendMessage(ws, text) {
  ws.send(JSON.stringify({ type: 'text_input', text }));
  // Gemini will respond with assistant_text chunks, then turn_complete
}
```

### Handling Streaming Text

`assistant_text` messages arrive as chunks. Append each one as it arrives:

```javascript
let currentResponse = '';

ws.onmessage = (evt) => {
  const msg = JSON.parse(evt.data);

  if (msg.type === 'assistant_text') {
    currentResponse += msg.text;
    responseEl.textContent = currentResponse; // update UI in real time
  }

  if (msg.type === 'turn_complete') {
    // Response is complete — add to history, re-enable input, etc.
    addToHistory('assistant', currentResponse);
    currentResponse = '';
    inputEl.disabled = false;
  }
};
```

---

## 8. Session Resumption

Sessions on the server persist until the idle TTL expires — even if your WebSocket connection drops. This means a user can refresh the page, lose Wi-Fi briefly, or close and reopen your app, and pick up the conversation exactly where they left off.

### How to Resume

1. When you receive `session_ready`, store the `session_id` somewhere durable (e.g. `sessionStorage`).
2. On reconnect, pass it in the URL: `?api_key=...&session_id=<stored_uuid>`
3. The server returns `session_ready` again with the same `session_id`, confirming the resume.

```javascript
// Store session_id on first connection
ws.onmessage = (evt) => {
  const msg = JSON.parse(evt.data);
  if (msg.type === 'session_ready') {
    sessionStorage.setItem('livechat_session_id', msg.session_id);
  }
};

// Resume on reconnect
function connect() {
  const sessionId = sessionStorage.getItem('livechat_session_id');
  const url = sessionId
    ? `wss://your-host/v1/chat?api_key=pk_live_...&session_id=${sessionId}`
    : `wss://your-host/v1/chat?api_key=pk_live_...`;

  return new WebSocket(url);
}
```

### Reconnection with Exponential Backoff

Don't hammer the server on disconnect. Use exponential backoff:

```javascript
class LiveChatConnection {
  constructor(getToken) {
    this.getToken = getToken; // async function that returns a fresh token/key
    this.ws = null;
    this.sessionId = sessionStorage.getItem('livechat_session_id') || null;
    this.retryDelay = 1000;
    this.maxDelay = 30000;
  }

  async connect() {
    const token = await this.getToken();
    const sessionParam = this.sessionId ? `&session_id=${this.sessionId}` : '';
    const url = `wss://your-host/v1/chat?token=${token}${sessionParam}`;

    this.ws = new WebSocket(url);
    this.ws.binaryType = 'arraybuffer';

    this.ws.onmessage = (evt) => this.handleMessage(evt);

    this.ws.onclose = (evt) => {
      if (evt.code === 4001 || evt.code === 4002) {
        // Auth or project error — don't retry
        this.onFatalError(evt.code, evt.reason);
        return;
      }
      if (evt.code === 4005) {
        // Ephemeral token expired — clear it, retry will get a fresh one
        this.sessionId = null;
      }
      // Retry with backoff for all other close codes
      setTimeout(() => this.connect(), this.retryDelay);
      this.retryDelay = Math.min(this.retryDelay * 2, this.maxDelay);
    };

    this.ws.onopen = () => {
      this.retryDelay = 1000; // reset backoff on successful connect
    };
  }

  handleMessage(evt) {
    if (evt.data instanceof ArrayBuffer) {
      playAudio(evt.data);
      return;
    }
    const msg = JSON.parse(evt.data);
    if (msg.type === 'session_ready') {
      this.sessionId = msg.session_id;
      sessionStorage.setItem('livechat_session_id', msg.session_id);
    }
    // ... handle other message types
  }
}
```

---

## 9. Connection Close Codes

When the WebSocket closes with a `4xxx` code, the `reason` field contains a human-readable explanation.

| Code | Reason | What to Do |
|------|--------|------------|
| `4001` | Invalid or missing API key / token | Check that the key or token is correct. If using ephemeral tokens, mint a fresh one. |
| `4002` | Project not found or inactive | The project has been deleted or deactivated in the dashboard. Contact your platform administrator. |
| `4003` | Rate limit exceeded | Too many connections per minute on this API key. Back off and retry after a few seconds. |
| `4004` | Max concurrent sessions reached | The project has hit its session cap. Retry when an existing session ends. |
| `4005` | Ephemeral token already redeemed or expired | Mint a new token from your backend and reconnect immediately. |
| `4006` | Origin not allowed | Your page's origin isn't in the project's CORS whitelist. Update the allowed origins in the dashboard. |
| `4007` | Account suspended | The account has a billing or quota issue. Contact the platform team. |
| `4008` | Daily token quota exceeded | Token usage has hit the daily plan limit. Quota resets at midnight UTC. |

### Codes that require immediate retry

`4005` — mint a new token and reconnect right away.

### Codes that require backoff

`4003`, `4004` — wait before retrying.

### Codes that require no retry

`4001`, `4002`, `4006`, `4007`, `4008` — these indicate a configuration or account issue that won't resolve itself on retry.

---

## 10. Complete Examples

### JavaScript — Text Chat

A minimal text-only chat client.

```javascript
async function createChatClient(apiKey) {
  return new Promise((resolve) => {
    const ws = new WebSocket(`wss://your-host/v1/chat?api_key=${apiKey}`);
    ws.binaryType = 'arraybuffer';

    let sessionId = null;
    let onMessage = null; // set by caller

    ws.onmessage = (evt) => {
      if (evt.data instanceof ArrayBuffer) return; // ignore audio

      const msg = JSON.parse(evt.data);

      if (msg.type === 'session_ready') {
        sessionId = msg.session_id;
        resolve({ send, getSessionId: () => sessionId, close: () => ws.close() });
      }

      if (onMessage) onMessage(msg);
    };

    function send(text, callback) {
      onMessage = callback;
      ws.send(JSON.stringify({ type: 'text_input', text }));
    }
  });
}

// Usage
const chat = await createChatClient('pk_live_...');

chat.send('Tell me a short joke', (msg) => {
  if (msg.type === 'assistant_text') process.stdout.write(msg.text);
  if (msg.type === 'turn_complete')  console.log('\n[done]');
});
```

---

### JavaScript — Push-to-Talk Voice + Text

```html
<!DOCTYPE html>
<html>
<body>
  <button id="talk">Hold to Talk</button>
  <input id="text" placeholder="Type a message..." />
  <button id="send">Send</button>
  <div id="chat"></div>

<script>
const ws = new WebSocket('wss://your-host/v1/chat?api_key=pk_live_...');
ws.binaryType = 'arraybuffer';

// ── Audio playback ─────────────────────────────────────────
let audioCtx = null;
let nextPlayAt = 0;

function playAudio(buffer) {
  if (!audioCtx) audioCtx = new AudioContext();
  const int16 = new Int16Array(buffer);
  const f32   = Float32Array.from(int16, v => v / 32768);
  const buf   = audioCtx.createBuffer(1, f32.length, 24000);
  buf.copyToChannel(f32, 0);
  const src   = audioCtx.createBufferSource();
  src.buffer  = buf;
  src.connect(audioCtx.destination);
  const at    = Math.max(audioCtx.currentTime + 0.02, nextPlayAt);
  src.start(at);
  nextPlayAt  = at + buf.duration;
}

// ── Message display ────────────────────────────────────────
function appendMessage(role, text) {
  const el = document.createElement('p');
  el.textContent = `${role}: ${text}`;
  document.getElementById('chat').appendChild(el);
}

let currentAssistantText = '';

// ── WebSocket messages ─────────────────────────────────────
ws.onmessage = (evt) => {
  if (evt.data instanceof ArrayBuffer) { playAudio(evt.data); return; }

  const msg = JSON.parse(evt.data);

  switch (msg.type) {
    case 'session_ready':
      sessionStorage.setItem('session_id', msg.session_id);
      appendMessage('system', 'Connected — start talking or type below');
      ws.send(JSON.stringify({ type: 'set_speaker', enabled: true }));
      break;

    case 'assistant_text':
      currentAssistantText += msg.text;
      break;

    case 'user_transcript':
      appendMessage('you (voice)', msg.text);
      break;

    case 'turn_complete':
      if (currentAssistantText) {
        appendMessage('assistant', currentAssistantText);
        currentAssistantText = '';
      }
      break;

    case 'error':
      console.error('Session error:', msg.message);
      break;
  }
};

// ── Text input ─────────────────────────────────────────────
document.getElementById('send').onclick = () => {
  const input = document.getElementById('text');
  if (!input.value.trim()) return;
  appendMessage('you', input.value);
  ws.send(JSON.stringify({ type: 'text_input', text: input.value }));
  input.value = '';
};

document.getElementById('text').addEventListener('keydown', (e) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    document.getElementById('send').click();
  }
});

// ── Push-to-talk ───────────────────────────────────────────
let mediaStream = null;
let scriptProcessor = null;
let audioContext = null;

async function startVoice() {
  audioContext  = new AudioContext({ sampleRate: 48000 });
  mediaStream   = await navigator.mediaDevices.getUserMedia({
    audio: { echoCancellation: true, noiseSuppression: true }
  });

  const source    = audioContext.createMediaStreamSource(mediaStream);
  scriptProcessor = audioContext.createScriptProcessor(4096, 1, 1);
  source.connect(scriptProcessor);
  scriptProcessor.connect(audioContext.destination);

  ws.send(JSON.stringify({ type: 'voice_start' }));

  scriptProcessor.onaudioprocess = (e) => {
    const f32     = e.inputBuffer.getChannelData(0);
    const ratio   = 48000 / 16000;
    const outLen  = Math.floor(f32.length / ratio);
    const down    = new Float32Array(outLen);
    for (let i = 0; i < outLen; i++) {
      const pos = i * ratio;
      const idx = Math.floor(pos);
      down[i]   = f32[idx] + (pos - idx) * ((f32[idx+1] ?? 0) - f32[idx]);
    }
    const int16 = new Int16Array(outLen);
    for (let i = 0; i < outLen; i++) {
      const s = Math.max(-1, Math.min(1, down[i]));
      int16[i] = s < 0 ? s * 0x8000 : s * 0x7FFF;
    }
    ws.send(int16.buffer);
  };
}

function stopVoice() {
  if (scriptProcessor) { scriptProcessor.disconnect(); scriptProcessor = null; }
  if (mediaStream)     { mediaStream.getTracks().forEach(t => t.stop()); mediaStream = null; }
  ws.send(JSON.stringify({ type: 'voice_end' }));
}

const talkBtn = document.getElementById('talk');
talkBtn.addEventListener('mousedown',  startVoice);
talkBtn.addEventListener('touchstart', startVoice, { passive: true });
talkBtn.addEventListener('mouseup',    stopVoice);
talkBtn.addEventListener('touchend',   stopVoice);
talkBtn.addEventListener('mouseleave', () => { if (mediaStream) stopVoice(); });
</script>
</body>
</html>
```

---

### Python — Text Chat (asyncio)

```python
import asyncio
import json
import websockets

async def chat():
    url = "wss://your-host/v1/chat?api_key=pk_live_..."

    async with websockets.connect(url) as ws:
        # Wait for session_ready
        while True:
            raw = await ws.recv()
            msg = json.loads(raw)
            if msg["type"] == "session_ready":
                print(f"Connected. Session: {msg['session_id']}")
                break

        # Send a message and collect the response
        await ws.send(json.dumps({
            "type": "text_input",
            "text": "What is the capital of France?"
        }))

        response = ""
        async for raw in ws:
            msg = json.loads(raw)

            if msg["type"] == "assistant_text":
                response += msg["text"]
                print(msg["text"], end="", flush=True)

            elif msg["type"] == "turn_complete":
                print()  # newline
                break

            elif msg["type"] == "error":
                print(f"\nError: {msg['message']}")
                break

asyncio.run(chat())
```

---

### React Hook

```typescript
import { useEffect, useRef, useState, useCallback } from 'react';

type MessageType =
  | { type: 'assistant_text'; text: string }
  | { type: 'user_transcript'; text: string }
  | { type: 'turn_complete' }
  | { type: 'error'; message: string }
  | { type: 'session_ready'; session_id: string; speaker_mode: boolean };

interface UseLiveChatOptions {
  getCredential: () => Promise<string>; // returns token or api_key
  credentialType: 'token' | 'api_key';
  onMessage: (msg: MessageType) => void;
  onAudio?: (buffer: ArrayBuffer) => void;
}

export function useLiveChat({
  getCredential,
  credentialType,
  onMessage,
  onAudio,
}: UseLiveChatOptions) {
  const wsRef = useRef<WebSocket | null>(null);
  const sessionIdRef = useRef<string | null>(null);
  const [connected, setConnected] = useState(false);

  const connect = useCallback(async () => {
    const credential = await getCredential();
    const param = credentialType === 'token' ? 'token' : 'api_key';
    const sessionParam = sessionIdRef.current
      ? `&session_id=${sessionIdRef.current}`
      : '';
    const url = `wss://your-host/v1/chat?${param}=${credential}${sessionParam}`;

    const ws = new WebSocket(url);
    ws.binaryType = 'arraybuffer';
    wsRef.current = ws;

    ws.onmessage = (evt) => {
      if (evt.data instanceof ArrayBuffer) {
        onAudio?.(evt.data);
        return;
      }
      const msg: MessageType = JSON.parse(evt.data);
      if (msg.type === 'session_ready') {
        sessionIdRef.current = msg.session_id;
        setConnected(true);
      }
      onMessage(msg);
    };

    ws.onclose = () => setConnected(false);
  }, [getCredential, credentialType, onMessage, onAudio]);

  const sendText = useCallback((text: string) => {
    wsRef.current?.send(JSON.stringify({ type: 'text_input', text }));
  }, []);

  const setSpeaker = useCallback((enabled: boolean) => {
    wsRef.current?.send(JSON.stringify({ type: 'set_speaker', enabled }));
  }, []);

  const disconnect = useCallback(() => {
    wsRef.current?.close();
  }, []);

  useEffect(() => {
    connect();
    return () => wsRef.current?.close();
  }, []);

  return { connected, sendText, setSpeaker, disconnect, reconnect: connect };
}
```

---

## 11. Production Checklist

Before going live, verify the following.

### Authentication
- [ ] Using ephemeral tokens instead of a publishable key in browser code
- [ ] Token minting happens on your backend server using a secret key (`sk_live_...`)
- [ ] Token TTL is set appropriately (60 seconds is enough for most page-load scenarios)
- [ ] Token rotation is implemented if your page load can exceed the TTL

### Connection Handling
- [ ] `ws.binaryType = 'arraybuffer'` is set before the connection opens
- [ ] No messages are sent before `session_ready` is received
- [ ] The `session_id` from `session_ready` is stored (e.g. `sessionStorage`) for reconnection
- [ ] Reconnection logic uses exponential backoff
- [ ] Close codes `4001`, `4002`, `4006` do **not** trigger retry loops

### Voice
- [ ] Microphone audio is downsampled to 16 kHz mono before sending
- [ ] `voice_start` is sent before the first binary frame
- [ ] `voice_end` is sent when recording stops
- [ ] Playback audio (received at 24 kHz) is scheduled using `AudioContext.createBufferSource` with gapless timing
- [ ] Echo cancellation is requested via `getUserMedia` constraints

### UX
- [ ] UI is disabled until `session_ready` is received
- [ ] Streaming `assistant_text` chunks are appended to the UI as they arrive
- [ ] A loading indicator is shown between sending a message and receiving `turn_complete`
- [ ] `error` messages are surfaced to the user (non-fatal — session stays open)
- [ ] `session_ended` triggers a graceful reconnect or end-of-session message

### CORS
- [ ] The domain(s) your WebSocket connects from are listed in the project's **Allowed Origins** in the dashboard (if you've set a whitelist)

---

## PCM Format Quick Reference

| | Sending (microphone) | Receiving (Gemini audio) |
|-|---------------------|--------------------------|
| **Sample rate** | 16,000 Hz | 24,000 Hz |
| **Bit depth** | 16-bit signed integer | 16-bit signed integer |
| **Channels** | Mono | Mono |
| **Byte order** | Little-endian | Little-endian |
| **Frame type** | Binary WebSocket frame | Binary WebSocket frame |
