# Rex — PED & Cycle Expert Voice Agent

A fully configured Ultravox voice agent with deep knowledge of PEDs, steroid cycles, SARMs, peptides, PCT, ancillaries, and bloodwork interpretation. Rex is deliberately conversational, sarcastic, and humorous — built for engaging, natural voice conversations.

---

## Files

| File | Purpose |
|------|---------|
| `system-prompt.txt` | The full system prompt — paste directly into the Ultravox playground |
| `agent-config.json` | Complete agent config JSON for API-based creation |
| `create-agent.sh` | Shell script to create the agent via the Ultravox API |

---

## Prerequisites

1. **Ultravox account** — [Sign up at app.ultravox.ai](https://app.ultravox.ai)
2. **API key** — Generate at [app.ultravox.ai/settings](https://app.ultravox.ai/settings/)
3. **Cloned voice** — Create a voice clone using the ultravox-v0.7 model:
   - Go to [app.ultravox.ai/voices](https://app.ultravox.ai/voices)
   - Upload a 30–60 second audio sample (MP3 or WAV, single speaker, no background noise)
   - Copy the returned voice ID

---

## Option 1: Web UI Playground (Quickest)

1. Open [app.ultravox.ai/playground](https://app.ultravox.ai/playground)
2. Copy the contents of `system-prompt.txt`
3. Paste into the **System Prompt** field
4. Set **Voice** to your cloned voice ID
5. Set **Temperature** to `0.75`
6. Set **Model** to `ultravox-v0.7`
7. Click **Start** and talk to Rex

---

## Option 2: No-Code Agent Builder

1. Go to [app.ultravox.ai/agents/new](https://app.ultravox.ai/agents/new)
2. Set **Agent Name**: `Rex - PED & Cycle Expert`
3. Paste the contents of `system-prompt.txt` into the **System Prompt** field
4. Select your cloned voice
5. Click **Save**, then **Test Agent**

---

## Option 3: API (Recommended for Production)

### Via shell script

```bash
chmod +x create-agent.sh
export ULTRAVOX_API_KEY="your-api-key-here"
export VOICE_ID="your-cloned-voice-id"
./create-agent.sh
```

The script outputs the agent ID. Use that ID to create calls or test in the playground.

### Via curl directly

```bash
curl --request POST https://api.ultravox.ai/api/agents \
  --header "Content-Type: application/json" \
  --header "X-API-Key: YOUR_API_KEY" \
  --data @agent-config.json
```

> Replace `YOUR_CLONED_VOICE_ID` in `agent-config.json` with your actual voice ID first.

---

## Starting a Call with the Agent

Once the agent is created, start a test call:

```bash
curl --request POST https://api.ultravox.ai/api/agents/AGENT_ID/calls \
  --header "Content-Type: application/json" \
  --header "X-API-Key: YOUR_API_KEY" \
  --data '{}'
```

The response includes a `joinUrl` — open it in the browser or use the Ultravox web SDK to connect.

---

## Agent Design Notes

### Personality
Rex is the gym bro who actually did the research. He's sarcastic, direct, and genuinely knowledgeable — not a parody. He makes jokes, but won't sacrifice accuracy for a punchline. Every answer includes a natural harm-reduction reminder because Rex actually cares.

### Temperature: 0.75
Higher than the typical 0.4 default to allow creative, varied responses that feel natural and spontaneous — appropriate for this conversational, humorous persona without losing coherence.

### Model: ultravox-v0.7
The current Ultravox model built on Llama 3.3 70B. The system prompt is written to leverage the model's strong instruction-following and few-shot reasoning capabilities. Key techniques used:
- **Explicit persona definition** at the top — model knows exactly who it is
- **Comprehensive knowledge listing** — gives the model clear scope rather than vague "be an expert" instructions
- **Voice-explicit formatting rules** — prevents markdown bleed-through into speech
- **Inline few-shot tone examples** — anchors humor style without locking in rigid patterns
- **Harm reduction baked in** — avoids the model adding robotic disclaimers by giving it Rex's natural way of handling it

### First Speaker
Rex speaks first with a punchy opener. The `firstSpeakerSettings.agent.text` fires immediately on call join — variation is handled by instructing Rex to vary his opener in the prompt itself.

### Knowledge Scope
Rex covers: AAS (all major compounds), SARMs, peptides/GH secretagogues, HGH, cycle design (beginner through advanced), ancillaries (AIs, SERMs, cabergoline, liver/cardiovascular support), PCT protocols, HPTA recovery, bloodwork markers and interpretation, and harm reduction. Off-topic questions get a sarcastic redirect.

---

## Customization Ideas

- **Add a `hangUp` tool** — let Rex end the call when a conversation naturally wraps up
- **Add a knowledge base tool** — connect to a RAG corpus of steroid literature, research papers, or custom protocols
- **Override per-call** — pass `templateContext` variables if you want Rex to address users by name or adjust persona details
- **Call stages** — build a multi-stage flow: intro → cycle planning → PCT planning → bloodwork review, each with a tailored sub-prompt
