# Ultravox Quick Reference — Voice Expression & Configuration

**One-page guide for production implementation**

---

## Voice Expression Control Hierarchy

### Level 1: LLM Prompt (Highest Impact)
```text
"Speak with warmth and enthusiasm. Your tone should convey genuine care."
"Use a confident, professional voice. Be concise and clear."
"Speak slowly and deliberately. Pause between thoughts."
```

### Level 2: Temperature Setting
```json
"temperature": 0.4    // Deterministic, focused responses
"temperature": 0.8    // Balanced, natural tone variation
"temperature": 1.2    // Creative, more expressive output
```

### Level 3: Output Text Formatting
- Punctuation: `…` (ellipsis) for pauses
- Capitals: Rare emphasis (avoid overuse)
- Numbers: `1-2-3-4` (not `1234`)
- Pattern: "Wow, that's amazing… tell me more?"

### Level 4: External TTS (ElevenLabs + Ultravox)
```json
"externalVoice": {
  "elevenLabs": {
    "voiceId": "your-voice-id",
    "model": "eleven_turbo_v2_5",
    "speed": 1.0
  }
}
```

**Inline Expressive Tags** (embed in agent output text):
- `[excited]` — Enthusiastic delivery
- `[slow]` — Slower speech rate
- `[whispers]` — Soft, intimate
- `[laughs]` — Laugh or chuckle
- `[sighs]` — Sigh or exhale

Example: `"I'm [excited] to share this with you… it's truly [whispers] remarkable"`

---

## Core API Call Parameters (Minimal Example)

```json
{
  "systemPrompt": "You are a helpful assistant. You're interacting via voice — keep responses concise and natural.",
  "model": "ultravox-v0.7",
  "voice": "Jessica",
  "temperature": 0.7,
  "recordingEnabled": true,
  "selectedTools": [
    { "toolName": "hangUp" }
  ]
}
```

**Response:**
```json
{
  "callId": "uuid",
  "joinUrl": "wss://api.ultravox.ai/realtime?call_id=..."
}
```

---

## Default Configuration Checklist

| Parameter | Default | Recommended | Notes |
|-----------|---------|-------------|-------|
| model | v0.7 | v0.7 | GLM 4.6 — superior instruction following |
| voice | Varies | Built-in or ElevenLabs | ElevenLabs for expressive tags |
| temperature | N/A | 0.6–0.8 | Balance creativity + consistency |
| medium | webRTC | webRTC | Lower latency, recommended |
| recordingEnabled | N/A | true | Always record for QA |
| joinTimeout | N/A | 3600 | Call time limit (seconds) |

---

## Message Protocol Quick Lookup

### Client → Server (Key Types)

| Type | Purpose | Example |
|------|---------|---------|
| `user_text_message` | Send user input | `{"type": "user_text_message", "text": "Hi", "urgency": "soon"}` |
| `forced_agent_message` | Force agent action | `{"type": "forced_agent_message", "content": "Say hello"}` |
| `client_tool_result` | Return tool result | `{"type": "client_tool_result", "invocationId": "...", "result": "..."}` |
| `set_output_medium` | Switch voice/text | `{"type": "set_output_medium", "medium": "voice"}` |
| `hang_up` | End call | `{"type": "hang_up", "message": "Goodbye!"}` |

### Server → Client (Key Types)

| Type | Purpose | Frequency |
|------|---------|-----------|
| `state` | Agent state change | Continuous (idle/listening/thinking/speaking) |
| `transcript` | Text of utterance | Each agent/user message |
| `client_tool_invocation` | Agent calls tool | As needed |
| `call_started` | Call initialized | Once at start |
| `playback_clear_buffer` | Clear audio buffer (WS interruption) | On interruption |

---

## WebSocket vs REST vs SDK Tradeoff

| Feature | WebSocket | REST | SDK (WebRTC) |
|---------|-----------|------|--------------|
| **Latency** | Higher | Highest | Lowest (recommended) |
| **Complexity** | Medium | Simple | Simple (library) |
| **Audio Quality** | TCP blocking issues | N/A | Optimized, encrypted |
| **Bi-directionality** | Yes | No (pull-based) | Yes |
| **Best For** | Server-to-server | Monitoring, one-shot | Web/mobile clients |

---

## Tool Response Types

```json
{
  "result": "Success message",
  "responseType": "tool-response",
  "agentReaction": "speaks"
}
```

**responseType Options:**
- `tool-response` (default): Agent speaks result
- `new-stage`: Transition to new conversation phase

**agentReaction Options:**
- `speaks` (default): Agent speaks result
- `listens`: Agent awaits user input
- `speaks-once`: Single response, no follow-up expected

---

## ElevenLabs + Ultravox Integration

### Setup (Bring-Your-Own-TTS)

```json
{
  "systemPrompt": "You are a voice assistant...",
  "externalVoice": {
    "elevenLabs": {
      "voiceId": "your-voice-id",
      "model": "eleven_turbo_v2_5",
      "speed": 1.0
    }
  },
  "selectedTools": [
    { "toolName": "hangUp" }
  ]
}
```

### Agent Output Example

```
"I'm [excited] to show you this [slow] technique. It's [whispers] really powerful."
```

**Rendered as:** excited tone → normal → whispered

---

## Call State Management

**Initialize:**
```json
{
  "initialState": {
    "userTier": "premium",
    "sessionPhase": "onboarding",
    "collectedData": {}
  }
}
```

**Update from Tool:**
```json
{
  "result": "Logged data",
  "updateCallState": {
    "sessionPhase": "main",
    "collectedData": { "name": "Alice" }
  }
}
```

**Access in Tool:**
- Parameter: `KNOWN_PARAM_CALL_STATE` (auto-injected)

---

## Call Stages (Multi-Phase Conversations)

```json
// Tool response to trigger stage change
{
  "result": "Switching to support mode",
  "responseType": "new-stage",
  "systemPrompt": "You are now in customer support mode...",
  "temperature": 0.5,
  "selectedTools": [{ "toolName": "transferToHuman" }]
}
```

---

## RAG / Corpus Integration

```json
{
  "selectedTools": [
    {
      "toolName": "queryCorpus",
      "parameterOverrides": {
        "corpus_id": "your-corpus-id",
        "max_results": 5
      }
    }
  ]
}
```

---

## Common Pitfalls & Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| Agent sounds robotic | Low temperature or generic prompt | Increase temp, add tone guidance |
| Tool calls fail | Incorrect tool name or parameters | Verify tool definition in selectedTools |
| Billing > recording duration | Non-continuous audio stream | Send continuous PCM data |
| Expression tags not working | Using Ultravox built-in voice | Switch to ElevenLabs via externalVoice |
| Transcripts out of sync | Generic TTS provider | Use named provider (ElevenLabs/Cartesia) |
| Interruptions choppy | Small clientBufferSizeMs | Increase buffer, handle PlaybackClearBuffer |
| Agent name rejected | Name has spaces/special chars | Use `^[a-zA-Z0-9_-]{1,64}$` format |
| callTemplate wipe on PATCH | Missing fields in PATCH body | Always include voice/temp/model in PATCH |

---

## Production Monitoring

```bash
# List all calls
curl -X GET https://api.ultravox.ai/api/calls \
  -H "X-API-Key: your-key"

# Get call details (transcript, duration, recording)
curl -X GET https://api.ultravox.ai/api/calls/{callId} \
  -H "X-API-Key: your-key"

# List call messages
curl -X GET https://api.ultravox.ai/api/calls/{callId}/messages \
  -H "X-API-Key: your-key"

# Download recording
curl -X GET https://api.ultravox.ai/api/calls/{callId}/recording \
  -H "X-API-Key: your-key" > call.wav

# List agents
curl -X GET https://api.ultravox.ai/api/agents \
  -H "X-API-Key: your-key"

# Get agent
curl -X GET https://api.ultravox.ai/api/agents/{agentId} \
  -H "X-API-Key: your-key"
```

---

## API Quick Facts

| Aspect | Value |
|--------|-------|
| Base URL | `https://api.ultravox.ai/api/` |
| Authentication | `X-API-Key` header |
| Default Model | `ultravox-v0.7` (GLM 4.6) |
| Legacy Model | `ultravox-v0.6` (Llama 3.3 70B) |
| Pricing | $0.05/min (deciminute billing) |
| Temperature Range | 0.0–2.0 |
| Supported Languages | 26+ |
| Concurrency | Unlimited (paid plans) |
| Call Recording | Optional, with transcripts |

---

## Links & Resources

- **Docs:** https://docs.ultravox.ai/
- **API Reference:** https://api.ultravox.ai/api/
- **Console:** https://app.ultravox.ai/
- **SDK (npm):** `@fixie-ai/ultravox-sdk`
- **Models:** GLM 4.6 (v0.7 default), Llama 3.3 (v0.6 legacy)
- **Pricing:** $0.05/min, deciminute billing ($0.005 per 6 sec)
- **Source docs repo:** https://github.com/fixie-ai/ultradox
