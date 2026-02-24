# Ultravox Voice AI API - Comprehensive Technical Research

**Research Date:** February 22, 2026
**Ultravox Version:** v0.7 (Default), v0.6 (Llama 3.3 70B Legacy)
**Repository:** https://github.com/fixie-ai/ultradox
**Documentation:** https://fixie-ai.github.io/ultradox/

---

## 1. ARCHITECTURE OVERVIEW

### 1.1 Core Philosophy
Ultravox is an **audio-native, no-ASR (Automatic Speech Recognition) platform**:
- Direct speech-to-understanding processing (no intermediate text transcription stage)
- Built on best-in-class open-weight models
- Preserves paralinguistic context (tone, inflection, emotion) that traditional ASR loses
- **Default Model (Dec 22, 2025):** GLM 4.6 (superior instruction following, tool calling, understanding)
- **Legacy Model:** `ultravox-v0.6` or `ultravox-v0.6-llama3.3-70b` (Llama 3.3 70B)

### 1.2 Key Differentiators
- **No ASR = Real Understanding:** Direct audio processing → faster responses + better context
- **Unlimited Scale:** No concurrency caps on paid plans
- **BYOT (Bring Your Own Telephony):** Total flexibility with any telephony provider
- **Enterprise Performance at Consumer Prices:** $0.05/min standard rate (rounded to nearest deciminute)
- **Billing Model:** Charged per 6 seconds (deciminute), minimum 6 seconds per call

### 1.3 Core Architecture Pattern
```
REST API (Create Call) → Call Configuration + joinUrl
                ↓
        Join via Multiple Methods:
        - Client SDK (WebRTC)
        - WebSocket (Server-to-Server)
        - Telephony Bridge (SIP/Twilio)
                ↓
        Real-time Bidirectional Communication
        (Audio + Data Messages)
```

---

## 2. CALL CONFIGURATION & API PARAMETERS

### 2.1 Creating Calls (REST API: POST /api/calls)

**Base URL:** `https://api.ultravox.ai/api/`

**Authentication:** `X-API-Key` header (41 char format: `XXXXXXXX.XXXXXXXXXXXXXXXXXXXXXXXX`)

#### Core Call Parameters:

```typescript
{
  // Agent Configuration
  "systemPrompt": "You are...",              // Full system prompt (no defaults appended)
  "model": "ultravox-v0.7",                  // Optional: defaults to v0.7 (GLM 4.6)
  "temperature": 0.4,                        // 0.0 (deterministic) to 2.0 (creative)

  // Voice Configuration
  "voice": "Jessica",                        // Built-in voice name
  // OR
  "externalVoice": {                         // Named TTS provider
    "elevenLabs": {
      "voiceId": "voice-id",
      "model": "eleven_turbo_v2_5",
      "speed": 1.0,                          // 0.5 to 2.0
      "stability": 0.8                       // 0.0 to 1.0
    }
  },

  // Conversation Setup
  "languageHint": "en",                      // Hint for speech understanding
  "recordingEnabled": true,                  // Enable audio recording + transcript
  "joinTimeout": 3600,                       // Seconds before timeout
  "maxDuration": 3600,                       // Max call length in seconds

  // Tool Configuration
  "selectedTools": [
    { "toolName": "hangUp" },
    { "toolName": "queryCorpus", "parameterOverrides": { "corpus_id": "..." } }
  ],

  // Inactivity Handling
  "inactivityMessages": [
    {
      "duration": "30s",
      "message": "Are you still there?",
      "endBehavior": "END_BEHAVIOR_HANG_UP_SOFT"  // or END_BEHAVIOR_HANG_UP_STRICT
    }
  ],

  // Call Modes (Medium)
  "medium": {
    "serverWebSocket": {
      "inputSampleRate": 48000,              // Required
      "outputSampleRate": 48000,             // Optional, defaults to input
      "clientBufferSizeMs": 60               // Smaller = faster interrupts (default: 60)
    }
  },
  // OR
  "medium": "webRTC",                        // Default for SDK-based clients

  // Initial Context
  "initialMessages": [                       // Set conversation history
    { "role": "MESSAGE_ROLE_USER", "text": "Hello" },
    { "role": "MESSAGE_ROLE_AGENT", "text": "Hi there!" }
  ],

  "initialState": {                          // Tool state persistence
    "userTier": "premium",
    "collectedData": { }
  }
}
```

**Response:**
```json
{
  "callId": "uuid",
  "joinUrl": "wss://api.ultravox.ai/realtime?call_id=...",
  "status": "created"
}
```

### 2.2 Agents (Templates for Reusable Calls)

**Endpoint:** `POST /api/agents`

```typescript
{
  "name": "Customer Support Agent",
  "callTemplate": {
    "systemPrompt": "You are {{agentName}}, a {{role}}...",
    "voice": "Jessica",
    "temperature": 0.4,
    "recordingEnabled": true,
    "firstSpeakerSettings": {
      "agent": {
        "text": "Hello! How can I help you today?"
      }
    },
    "selectedTools": [
      { "toolName": "knowledgebaseLookup" },
      { "toolName": "transferToHuman" }
    ]
  }
}
```

**Template Variables:** Use `{{variableName}}` syntax. Populated at call-time via `templateContext` param.

---

## 3. SYSTEM PROMPT & PERSONA CONFIGURATION

### 3.1 Prompt As-If-Text Approach

**Critical Rule:** Ultravox uses a **frozen LLM** (not fine-tuned for voice). Prompt it as you would a text model, but declare the voice interaction mode:

```text
You are [Name], a friendly AI [role].
You're interacting with the user over voice, so speak casually.
Keep your responses short and to the point, much like someone would in dialogue.
Since this is a voice conversation, do not use lists, bullets, emojis, or other
things that do not translate to voice. In addition, do not use stage directions
or otherwise engage in action-based roleplay (e.g., "(pauses)", "*laughs").
```

### 3.2 Key Prompting Patterns

#### Pattern: No Default Prompts
- Ultravox **does NOT append a default prompt** to your input
- You must provide complete prompts including all context/instructions
- You have full control—nothing hidden from view

#### Pattern: Jailbreak Minimization
```text
Your only job is to [primary job]. If someone asks you a question
that is not related to [primary job], politely decline and redirect
the conversation back to the task at hand.
```

#### Pattern: Voice-Friendly Numbers
```text
Output account numbers, codes, or phone numbers as individual digits,
separated by hyphens (e.g. 1234 → '1-2-3-4'). For decimals,
say 'point' and then each digit (e.g., 3.14 → 'three point one four').
```

#### Pattern: Natural Pauses
```text
You want to speak slowly and clearly, so you must inject pauses between sentences.
Do this by emitting "..." at the end of a sentence but before any final
punctuation (e.g., "Wow, that's really interesting… can you tell me a bit more
about that…?").
```

#### Pattern: Tool Usage Guidance
```text
You have access to an address book. If someone asks for information about
a particular person, you MUST use the lookUpAddressBook tool to find that
information before replying.
```

### 3.3 Temperature Control
- **Range:** 0.0 to 2.0 (unbounded but extreme values not recommended)
- **Semantics:** Controls creativity/determinism
  - 0.0–0.4: Focused, deterministic responses (good for task-oriented agents)
  - 0.5–1.0: Balanced, natural conversation
  - 1.0–2.0: Creative, varied responses

---

## 4. VOICE EXPRESSION & PROSODY

### 4.1 Built-in Ultravox Voices
- Ultravox provides high-quality pre-configured voices
- Accessible via `voice` parameter (e.g., "Jessica", "Mark")
- Browsable at https://app.ultravox.ai/voices
- **No explicit SSML or inline markup support documented** in base Ultravox

### 4.2 External TTS Providers (Advanced Expression Control)

Ultravox supports **Bring-Your-Own-TTS** for specialized voice expression needs.

#### 4.2.1 Named Providers (Optimized Integration)

**ElevenLabs (Most Common)**
```json
"externalVoice": {
  "elevenLabs": {
    "voiceId": "21m00Tcm4TlvDq8ikWAM",
    "model": "eleven_turbo_v2_5",
    "speed": 1.0                            // 0.5 to 2.0
  }
}
```
- **Timing Info:** Character-level timing ensures transcript sync
- **Special Note:** Known for occasional word slurring/hallucination (as of May 2025)
- **Workaround:** Avoid special characters like `*`; consider multilingual model for robustness
- **Streaming:** Parallel text-in, audio-out streaming
- **Expressive Tags (v3 Conversational):** `[excited]`, `[slow]`, `[whispers]`, `[laughs]`, `[sighs]`

**Cartesia**
```json
"externalVoice": {
  "cartesia": {
    "voiceId": "af346552-54bf-4c2b-a4d4-9d2820f51b6c",
    "model": "sonic-2"
  }
}
```
- **Timing Info:** Word-level timing for transcript alignment
- **Streaming:** Bidirectional parallel streaming

**LMNT**
```json
"externalVoice": {
  "lmnt": {
    "voiceId": "lily",
    "model": "blizzard"                    // Experimental, higher quality
  }
}
```
- **Advantages:** Unlimited concurrency, no rate limits even on $10/month
- **Simplest SDK:** Only provider with a robust SDK
- **Models:** "aurora" (standard), "blizzard" (experimental, better quality)

#### 4.2.2 Generic TTS Integration (Maximum Flexibility)

For unsupported TTS providers, use generic HTTP POST with custom JSON:

```json
"externalVoice": {
  "generic": {
    "url": "https://api.deepgram.com/v1/speak?model=aura-2-asteria-en&...",
    "headers": {
      "Authorization": "Token YOUR_KEY",
      "Content-Type": "application/json"
    },
    "body": {
      "text": "{text}"                        // {text} is replaced by Ultravox
    },
    "responseSampleRate": 48000
  }
}
```

**Supported Generic Providers:**
- Deepgram (Aura-2)
- Google Cloud TTS (with `jsonAudioFieldPath`)
- OpenAI TTS (pcm format)
- Resemble
- Rime (with `spell()` for pronunciation)
- Sarvam (with `jsonAudioFieldPath`)
- Inworld (streaming JSONL with `responseMimeType`)
- Orpheus (self-hosted Llama 3 TTS)

**Tradeoffs with Generic:**
- No text-in streaming (Ultravox must buffer full text)
- Slightly higher response latency + possible audio discontinuities
- No timing info (Ultravox approximates transcript timing via WPM estimate)

### 4.3 Voice Expression Hierarchy

```
Level 1: System Prompt (tone, pacing, style guidance) — HIGHEST IMPACT
Level 2: Temperature (0.4 deterministic → 1.2+ creative)
Level 3: Output Text Formatting (ellipsis, punctuation cues)
Level 4: External TTS with inline tags ([excited], [slow], [whispers]) — MOST CONTROL
```

### 4.4 No Documented SSML Support
- Traditional SSML (`<prosody>`, `<emphasis>`, `<break>`) is **not documented** as a native feature
- Must be handled by external TTS provider (if supported)
- Best approach: **prompt the LLM** + **external TTS with advanced features**

---

## 5. TOOL CALLING & FUNCTION INVOCATION

### 5.1 Built-in Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `queryCorpus` | RAG (knowledge base lookup) | Requires `corpus_id` parameter |
| `leaveVoicemail` | Leave voicemail + end call | For outbound telephony |
| `hangUp` | End call gracefully | Optional farewell message |
| `playDtmfSounds` | Send DTMF tones (phone keypad) | For IVR/phone tree navigation |
| `coldTransfer` | Transfer to human operator | Telephony integration only |

### 5.2 Custom Tool Types

**HTTP Tools (Server-side execution):**
- Most common type
- Your server receives HTTP request from Ultravox
- Execute logic, return result
- Response headers can update call state

**Client Tools (Client-side execution):**
- Run in client application (SDK-based)
- Invoked when agent calls tool
- Can trigger UI changes, updates
- Useful for interactive experiences

### 5.3 Tool Definition & Invocation

```typescript
// Tool Definition
{
  "toolName": "get_weather",
  "description": "Retrieves current weather for a location",
  "http": {
    "url": "https://api.example.com/weather",
    "method": "POST"
  },
  "parameters": [
    {
      "name": "location",
      "type": "string",
      "location": "PARAMETER_LOCATION_BODY",
      "description": "City name",
      "required": true
    }
  ]
}
```

**Server-side Tool Response:**
```json
{
  "result": "It's 72°F and sunny in Seattle",
  "responseType": "tool-response",
  "agentReaction": "speaks",
  "X-Ultravox-Update-Call-State": {
    "lastWeatherLocation": "Seattle"
  }
}
```

**Client Tool Registration (JavaScript SDK):**
```typescript
session.registerToolImplementation('get_weather', (parameters) => {
  const location = parameters.location;
  return `It's 72°F and sunny in ${location}`;
});
```

### 5.4 Tool State Persistence

```typescript
// Set state during tool response
{
  "result": "Created profile",
  "updateCallState": {
    "userProfileId": "12345",
    "collectionStage": "completed"
  }
}
```

**Access state in subsequent tool calls:**
- Automatic parameter: `KNOWN_PARAM_CALL_STATE` injects current state
- Use in prompt: "Here's the user profile we just created: {{callState.userProfileId}}"

---

## 6. CLIENT ↔ SERVER MESSAGE PROTOCOL

### 6.1 Connection Methods

#### WebRTC (SDK-based clients, Recommended)
- Secure, encrypted, optimized for browser/mobile
- Bidirectional audio + data channel
- Lower latency than WebSocket
- Used by Ultravox Client SDKs

#### WebSocket (Server-to-Server)
- Direct TCP-based connection
- Less optimal for audio (TCP blocking/ordering constraints)
- Suitable for backend integrations, telephony gateways
- **Warning:** Not recommended for direct client use

#### Telephony/SIP
- External telephony provider (Twilio, Voximplant, custom SIP)
- Ultravox bridges calls via SIP or carrier integration

### 6.2 Data Message Protocol

All messages are JSON with `type` field + specific fields per type.

#### CLIENT-TO-SERVER MESSAGES

**UserTextMessage** (Send user text input)
```json
{
  "type": "user_text_message",
  "text": "Your message here",
  "urgency": "soon"
}
```
- `immediate`: Interrupts agent, starts generation immediately
- `soon`: Doesn't interrupt, triggers generation at next opportunity
- `later`: Considered during next generation, doesn't force new generation

**ForcedAgentMessage** (Server instructs agent to say/do something)
```json
{
  "type": "forced_agent_message",
  "content": "Text for agent to speak",
  "toolCalls": [
    {
      "id": "unique-invocation-id",
      "name": "tool_name",
      "arguments": { "param1": "value1" }
    }
  ],
  "uninterruptible": false,
  "urgency": "soon"
}
```

**ClientToolResult** (Client responds to tool invocation)
```json
{
  "type": "client_tool_result",
  "invocationId": "matching-invocation-id",
  "result": "Tool execution result",
  "responseType": "tool-response",
  "agentReaction": "speaks",
  "errorType": null,
  "errorMessage": null,
  "updateCallState": { }
}
```

**SetOutputMedium** (Control agent output)
```json
{
  "type": "set_output_medium",
  "medium": "voice"
}
```

**HangUp** (End call)
```json
{
  "type": "hang_up",
  "message": "Goodbye!"
}
```

**Ping** (Measure latency)
```json
{
  "type": "ping",
  "timestamp": 1234567890.123
}
```

#### SERVER-TO-CLIENT MESSAGES

**Transcript** (Speech-to-text updates)
```json
{
  "type": "transcript",
  "role": "agent",
  "medium": "voice",
  "text": "Full transcript so far",
  "delta": null,
  "final": false,
  "ordinal": 1
}
```

**State** (Server state notifications)
```json
{
  "type": "state",
  "state": "listening"
}
```
States: `"idle"` | `"listening"` | `"thinking"` | `"speaking"`

**ClientToolInvocation** (Agent invokes client tool)
```json
{
  "type": "client_tool_invocation",
  "toolName": "get_weather",
  "invocationId": "unique-invocation-id",
  "parameters": {
    "location": "Seattle"
  }
}
```

**PlaybackClearBuffer** (WebSocket only — clear buffered audio on interruption)
```json
{
  "type": "playback_clear_buffer"
}
```

**CallStarted** (Call initialization)
```json
{
  "type": "call_started",
  "callId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Pong** (Response to ping)
```json
{
  "type": "pong",
  "timestamp": 1234567890.123
}
```

### 6.3 REST API Message Injection

**Endpoint:** `POST /api/calls/{call_id}/send_data_message`

Supported message types: `ForcedAgentMessage`, `HangUp`, `UserTextMessage`

**Response:** 204 No Content on success

---

## 7. WEBSOCKET SETUP & JOINURL

### 7.1 Creating a WebSocket-based Call

```json
{
  "systemPrompt": "You are a helpful assistant...",
  "model": "ultravox-v0.7",
  "voice": "Mark",
  "medium": {
    "serverWebSocket": {
      "inputSampleRate": 48000,
      "outputSampleRate": 48000,
      "clientBufferSizeMs": 30000
    }
  }
}
```

**Response includes `joinUrl`:**
```
wss://api.ultravox.ai/realtime?call_id=550e8400-e29b-41d4-a716-446655440000&...
```

### 7.2 Joining via WebSocket (Python Example)

```python
import websockets

socket = await websockets.connect(join_url)

# Send audio (s16le PCM at inputSampleRate)
async def send_audio(socket):
    async for chunk in audio_source:
        await socket.send(chunk)  # bytes, 16-bit signed PCM

# Receive messages
async for message in socket:
    if isinstance(message, bytes):
        # Handle agent audio output
        pass
    else:
        # Handle JSON data message
        data = json.loads(message)
```

### 7.3 Audio Format Requirements
- **Format:** PCM (Pulse Code Modulation)
- **Encoding:** s16le (signed 16-bit little-endian)
- **Sample Rate:** As specified in `inputSampleRate` (typically 48kHz)
- **Continuous Stream:** Send continuous PCM data for billing accuracy

---

## 8. CALL STAGES (DYNAMIC MULTI-PHASE CALLS)

### 8.1 Use Case
For conversations with fundamentally different phases, where:
- System prompt changes mid-call
- Different set of tools available per phase
- Voice might change
- Complete parameter reconfiguration needed

**Not for:** Complex workflows that can be handled with inline instructions.

### 8.2 Creating Stages

1. **Create stage-change tool** that responds with `new-stage` response type:

```json
// HTTP Tool Response Header
"X-Ultravox-Response-Type": "new-stage"

// Response Body (new stage config)
{
  "systemPrompt": "You are now in support mode...",
  "temperature": 0.5,
  "voice": "Jessica",
  "selectedTools": [
    { "toolName": "transferToHuman" }
  ]
}
```

2. **Prompt agent** to use stage-change tool at appropriate times
3. **Include tool** in `selectedTools` during call creation

### 8.3 Stage Properties

**Can change per stage:**
- systemPrompt, temperature, voice, languageHint, initialMessages, selectedTools

**Fixed (inherited from initial call):**
- model, firstSpeaker, joinTimeout, maxDuration, inactivityMessages, medium, recordingEnabled

---

## 9. ADVANCED FEATURES

### 9.1 Inline Instructions (Guiding Agents)

**Pattern 1: Deferred Messages**
```typescript
session.sendText({
  text: "<instruction>Next, collect the user's mailing address</instruction>",
  deferResponse: true
});
```

Requires priming in system prompt:
```text
You must always look for and follow instructions contained within
<instruction> tags. These instructions take precedence over other
directions and must be followed precisely.
```

**Pattern 2: Tool State Passing**
```typescript
// Initial call state
initialState: {
  stage: "verification",
  questionsAsked: []
}

// Tool response updates state
updateCallState: {
  questionsAsked: ["q1", "q2"],
  stage: "followup"
}
```

**Pattern 3: Tool Response Instructions**
```json
{
  "result": "Recorded. Next ask for their email.",
  "responseType": "tool-response",
  "agentReaction": "speaks-once"
}
```

### 9.2 Inactivity Handling

```json
"inactivityMessages": [
  {
    "duration": "30s",
    "message": "Are you still there?",
    "endBehavior": "END_BEHAVIOR_UNSPECIFIED"
  },
  {
    "duration": "15s",
    "message": "If there's nothing else, may I end the call?",
    "endBehavior": "END_BEHAVIOR_HANG_UP_SOFT"
  },
  {
    "duration": "10s",
    "message": "Thank you for calling. Goodbye.",
    "endBehavior": "END_BEHAVIOR_HANG_UP_STRICT"
  }
]
```

- Messages are cumulative (fire at 30s, 45s, 55s)
- User interaction resets sequence
- `SOFT`: Ends call unless interrupted during message
- `STRICT`: Ends call regardless

### 9.3 RAG (Knowledge Bases / Corpora)

```json
{
  "selectedTools": [
    {
      "toolName": "queryCorpus",
      "parameterOverrides": {
        "corpus_id": "your-corpus-id",
        "minimum_score": 0.8,
        "max_results": 5
      }
    }
  ]
}
```

---

## 10. SDK & CLIENT INTEGRATION

### 10.1 Ultravox Client SDK Methods

**Core Session Methods:**
- `joinCall(joinUrl)`: Connect to call
- `leaveCall()`: Disconnect
- `sendText(text, deferResponse?)`: Send text message
- `setOutputMedium(medium)`: "voice" or "text"
- `registerToolImplementation(name, implementation)`: Register client tool handler

**Muting Controls:**
- `muteMic()` / `unmuteMic()`
- `muteSpeaker()` / `unmuteSpeaker()`
- `isMicMuted()` / `isSpeakerMuted()`

**Session Status:**
- `disconnected`, `disconnecting`, `connecting`, `idle`, `active`, `error`

**Event Listeners:**
```typescript
session.addEventListener('transcript', (event) => {
  console.log(event.role, event.text, event.final);
});

session.addEventListener('state', (event) => {
  console.log(event.state);  // "listening", "thinking", "speaking"
});
```

### 10.2 Client Tool Implementation

```typescript
session.registerToolImplementation('update_profile', (parameters) => {
  return {
    result: "Profile updated successfully",
    responseType: "tool-response",
    agentReaction: "speaks"
  };
});
```

---

## 11. COMPARISON: ULTRAVOX vs ELEVENLABS CONVAI

| Aspect | Ultravox | ElevenLabs ConvAI |
|--------|----------|-------------------|
| **Core Model** | Audio-native (no ASR) | Speech-to-text + LLM + TTS |
| **Latency** | Lower (direct audio processing) | Higher (ASR pipeline) |
| **Context Preservation** | Full paralinguistic (tone, emotion) | Limited (ASR bottleneck) |
| **Native Prosody Control** | Prompt-based only | Expressive tags ([excited], [slow], etc.) |
| **Temperature Control** | Yes (0.0–2.0) | Not exposed |
| **Voice Customization** | Built-ins + BYOT | Voice cloning, built-ins |
| **Tool Calling** | Native, robust | Supported |
| **Call Concurrency** | Unlimited (paid) | Variable by plan |
| **Pricing** | $0.05/min | Variable |

**Recommended combination:** Ultravox orchestration + ElevenLabs BYOT = audio-native intelligence + expressive TTS tags.

---

## 12. KNOWN ISSUES & EDGE CASES

### ElevenLabs Slurring (May 2025 onward)
- Known quality degradation issue
- Workaround: Avoid `*` and special characters in prompt output
- Alternative: Use multilingual model for robustness

### Transcript Timing Misalignment
- If user audio not continuous (gaps), billing duration > recording length
- Ensure continuous PCM stream for accurate timing
- Generic TTS has no timing info (Ultravox approximates via WPM)

### Model Defaults
- Dec 22, 2025: Default switched from Llama 3.3 70B to GLM 4.6
- Older agents/calls default to v0.7 unless explicitly set
- Legacy: Use `ultravox-v0.6` for Llama retention

### PlaybackClearBuffer (WebSocket)
- Server sends this when audio needs to be dropped (interruption handling)
- Client should clear unplayed buffered audio
- Enables smaller `clientBufferSizeMs` with reliability

---

## 13. PRODUCTION IMPLEMENTATION CHECKLIST

### Pre-Launch
- [ ] Define system prompt (no defaults appended — be explicit)
- [ ] Choose base model: v0.7 (GLM 4.6) or v0.6 (Llama) for compatibility
- [ ] Select voice: built-in or external TTS provider
- [ ] Configure tools: identify all needed function calls
- [ ] Set temperature for desired creativity level
- [ ] Design call stages if multi-phase conversation needed
- [ ] Plan tool state management strategy

### Voice Expression
- [ ] If expression critical: Integrate external TTS (ElevenLabs recommended for tags)
- [ ] Test prompt-based tone guidance (e.g., "speak warmly")
- [ ] Verify temperature affects response variety appropriately
- [ ] For emotion: Leverage LLM + expressive TTS combination

### Reliability
- [ ] Implement inactivity messages if calls are long-running
- [ ] Handle `PlaybackClearBuffer` for interruptions (WebSocket only)
- [ ] Ensure continuous PCM stream to avoid billing anomalies
- [ ] Set join timeout and max duration limits
- [ ] Test tool failure handling with `errorType` responses

### Monitoring
- [ ] Log all calls via `recordingEnabled: true`
- [ ] Retrieve transcripts and analyse agent behaviour regularly
- [ ] Monitor `state` messages to verify agent progression
- [ ] Track tool invocation success rates
- [ ] Use webhooks for real-time event notifications

---

## 14. QUICK START CODE SNIPPETS

### Create & Join WebRTC Call (JavaScript)

```javascript
// 1. Create call via REST API
const createResponse = await fetch('https://api.ultravox.ai/api/calls', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': 'your-api-key'
  },
  body: JSON.stringify({
    systemPrompt: "You are a helpful assistant.",
    model: "ultravox-v0.7",
    voice: "Jessica",
    temperature: 0.7,
    recordingEnabled: true,
    selectedTools: [{ toolName: "hangUp" }]
  })
});

const { joinUrl } = await createResponse.json();

// 2. Import & use Ultravox SDK
import { Ultravox } from "@fixie-ai/ultravox-sdk";
const session = new Ultravox.UltravoxSession();

// 3. Register client tools if needed
session.registerToolImplementation('my_tool', (params) => {
  return `Processed: ${JSON.stringify(params)}`;
});

// 4. Join call
session.joinCall(joinUrl);

// 5. Listen to events
session.addEventListener('transcript', (event) => {
  console.log(`${event.role}: ${event.text}`);
});

session.addEventListener('state', (event) => {
  console.log(`Agent state: ${event.state}`);
});

// 6. Leave when done
await session.leaveCall();
```

### Create WebSocket Call (Python)

```python
import json, asyncio, websockets, requests

# 1. Create call
response = requests.post(
    'https://api.ultravox.ai/api/calls',
    headers={'X-API-Key': 'your-api-key'},
    json={
        'systemPrompt': 'You are a helpful assistant.',
        'model': 'ultravox-v0.7',
        'voice': 'Mark',
        'medium': {
            'serverWebSocket': {
                'inputSampleRate': 48000,
                'outputSampleRate': 48000
            }
        }
    }
)

join_url = response.json()['joinUrl']

# 2. Connect via WebSocket
async def run_call():
    async with websockets.connect(join_url) as socket:
        async def send_audio():
            async for chunk in get_audio_stream():
                await socket.send(chunk)

        async def receive_messages():
            async for message in socket:
                if isinstance(message, bytes):
                    play_audio(message)
                else:
                    data = json.loads(message)
                    print(f"Message: {data}")

        await asyncio.gather(send_audio(), receive_messages())

asyncio.run(run_call())
```

---

## SUMMARY

Ultravox provides a robust, audio-native voice AI platform with:

1. **Direct speech understanding** without ASR (preserves paralinguistic context)
2. **Flexible configuration** via REST API + SDKs
3. **Rich expression control** via prompt engineering + external TTS
4. **Tool ecosystem** for functional integration
5. **Multi-stage conversations** for complex workflows
6. **Enterprise pricing** ($0.05/min) with unlimited concurrency
7. **WebRTC + WebSocket + Telephony** integration options

**Key docs in this repo:**
- `gettingstarted/how-ultravox-works.mdx` — Architecture overview
- `gettingstarted/prompting.mdx` — Prompt best practices
- `apps/websockets.mdx` — WebSocket protocol
- `apps/datamessages.mdx` — Complete message format spec
- `voices/bring-your-own.mdx` — External TTS setup
- `agents/call-stages.mdx` — Multi-stage conversations
- `agents/guiding-agents.mdx` — Inline instruction patterns
