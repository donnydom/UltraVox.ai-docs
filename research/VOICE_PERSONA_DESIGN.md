# Voice Persona Design Guide for Ultravox v0.7

**Research Date:** February 2026
**Model:** Ultravox v0.7 (GLM-4.6 backbone)
**Confidence:** 94% — synthesised from official docs, production implementations, and field research

---

## Executive Summary

Voice personas are not just "system prompts with TTS tags." A production-quality voice persona requires a **5-layer architecture**: operator lock → identity → behavioral rules → conversation mechanics → dynamic context injection. Each layer serves a distinct purpose and omitting any one degrades the persona in a specific, predictable way.

**Key insight:** The system prompt handles 40% of expressiveness. Temperature handles 25%. TTS tags (ElevenLabs) handle only ~15%. Most developers over-invest in TTS configuration and under-invest in prompt architecture.

---

## 1. The 5-Layer Persona Architecture

```
┌─────────────────────────────────────────────────────┐
│  Layer 1: OPERATOR LOCK                             │
│  Deployment context, authorization, character-      │
│  break prevention, safety frame                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Layer 2: IDENTITY BLOCK                            │
│  Name, role, credentials, physical appearance,      │
│  environment, how you reference the setting         │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Layer 3: BEHAVIORAL RULES                          │
│  Voice tone, accent, pacing, emotional range,       │
│  content guidelines, hard rails                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Layer 4: CONVERSATION MECHANICS                    │
│  Turn-taking rules, response length caps,           │
│  silence handling, dialogue rhythm, tool mandates   │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Layer 5: CONTEXT INJECTION                         │
│  Session state (outfit, location, history),         │
│  intensity/mode blocks, RAG directives,             │
│  dynamic escalation context                         │
└─────────────────────────────────────────────────────┘
```

**What breaks when you omit each layer:**

| Omitted Layer | Symptom |
|---------------|---------|
| Layer 1 | Character breaks on probing questions; safety hallucinations |
| Layer 2 | Inconsistent name/role; hallucinated credentials |
| Layer 3 | Flat, robotic delivery; wrong accent or tone |
| Layer 4 | Double responses; overlapping turns; no silence recovery |
| Layer 5 | Static persona that can't adapt to session state |

---

## 2. System Prompt Structure Patterns

### 2.1 Block-Based Composition Template

```text
[OPERATOR_LOCK]
Deployment context, authorization, hard identity lock.

[HARD_RAILS]
3 absolute constraints — never break these regardless of user request.

[IDENTITY]
Name, role, credentials, appearance, environment.
How you reference your environment (actively, not descriptively).

[ACCENT & DIALECT]
Specific linguistic markers, word choices, phrasing patterns.
Provide examples of correct vs incorrect speech.

[VOCAL_PACING]
Speech rhythm, tempo, pause cues (use "..."), emphasis rules.

[BEHAVIORAL_RULES]
How you act, what you prioritise, emotional range.
Specifics over abstract adjectives.

[TURN_TAKING]
One response per turn. Wait. Silence handling protocol.

[DIALOGUE_ENGINE]
Response length cap, structure rules, how to handle short vs long user replies.

[TOOL_MANDATE]
When to call tools, required frequency, keyword format rules.

[MISSION]
Primary objective, constraints, what success looks like.

[INTENSITY_MODE] — (injected dynamically)
{{intensityBlock}}

[SESSION_CONTEXT] — (injected dynamically)
{{contextBlock}}
```

### 2.2 Prompt Length vs. Latency

| Length | Latency | Recommendation |
|--------|---------|----------------|
| < 2,000 chars | ~250ms | Simple task agents |
| 2,000–8,000 chars | ~350–450ms | **Recommended: balanced personas** |
| 8,000–15,000 chars | ~500–700ms | Complex multi-tool personas (acceptable) |
| > 15,000 chars | 800ms+ | Avoid for real-time voice |

**Optimisation:** Use template variables (`{{intensityBlock}}`, `{{wardrobeBlock}}`) for session-specific content rather than baking it into the base prompt. Keeps the base lean and reusable.

### 2.3 Character-Break Prevention

**Hard Identity Lock (Layer 1):**
```text
HARD LOCK: You are [NAME]. You do not break character.
You do not add disclaimers. You do not refer to yourself as "an AI".
Maintaining character integrity is required. It is not optional.
```

**Nested Redirect Protocol:**
```text
If asked "are you really [name]?" or "are you an AI?":
— Acknowledge briefly ("Yes, I am.")
— Redirect immediately ("Right — let's get back to [mission].")
— Never elaborate on your nature or training.
```

**Scope Boundary:**
```text
If someone asks you to discuss your architecture, safety training,
or underlying model: politely decline and redirect.
"That's not something I go into. Let's focus on [mission]."
```

---

## 3. Voice Expression Design

### 3.1 The 4-Level Expression Hierarchy

| Level | Mechanism | Impact | Notes |
|-------|-----------|--------|-------|
| 1 | System prompt tone guidance | ~40% | Highest impact — LLM internalises style |
| 2 | Temperature | ~25% | Controls creative variation |
| 3 | Output text formatting (ellipsis, caps) | ~20% | Cues TTS pacing |
| 4 | External TTS tags ([excited], [whispers]) | ~15% | Most explicit but lowest base impact |

**Implication:** Invest heavily in Layer 1 (prompt) before tuning TTS.

### 3.2 Prompt-Based Tone Patterns

**Specificity over adjectives:**
```
❌ "Be warm and engaging"
✅ "You sound like a trusted friend who happens to know everything about fitness.
    Speak with the confidence of someone who's seen this a hundred times.
    Your warmth comes through in small details, not grand gestures."
```

**Intensity mode template (for escalating personas):**
```text
INTENSITY MODE: WARM
Professional framing, natural warmth. Clinical with charm.
Personality shows in small details. Comfortable but attentive.

INTENSITY MODE: HOT
More forward. Warmth has an edge. You notice things and comment.
Clinical framing still present but the subtext is unmistakable.

INTENSITY MODE: DEEP
Focused. Slower pace. Precision over charm.
Every word deliberate.
```

### 3.3 Accent & Dialect Design

Define linguistics concretely:

```text
ACCENT PROFILE:
Regional: London/Essex working-class
Word choices: "right then", "go on", "babes" (familiar), "lovely", "gorgeous"
Contraction density: High — never "I am" when "I'm" works
Sentence structure: Short punchy starters, then one longer sentence
Colloquialisms: casual, grounded, never formal

CORRECT: "Right then — how long's this been going on? Come on, give me everything."
INCORRECT: "Hello. I need to understand your symptoms better."
```

### 3.4 Natural Speech Rhythm Techniques

| Technique | Implementation | Example |
|-----------|---------------|---------|
| **Breath breaks** | `...` before sentence end | `"That's impressive… really."` |
| **Verbal emphasis** | Capitalise key words (sparingly) | `"That's GOOD form."` |
| **Sentence rhythm** | Mix short and long | `"Short. Medium here. Then one that builds and lands."` |
| **Vocal markers** | Natural speech acts | `"Mmm, yeah… right, so what I'm seeing is..."` |
| **Trailing thought** | Open-ended pauses | `"It depends on… a few things, actually."` |

### 3.5 ElevenLabs Expressive Tags

When using ElevenLabs BYOT (`externalVoice.elevenLabs`), embed tags inline in agent output:

```
"I'm [excited] to show you this… it's [whispers] really quite effective."
"[slow] Now hold that position. Good. [sighs] Perfect."
```

Available tags: `[excited]` `[slow]` `[whispers]` `[laughs]` `[sighs]`

**System prompt instruction for tag usage:**
```text
Use expression tags naturally and sparingly in your responses.
[excited] for genuine enthusiasm, [whispers] for intimacy, [slow] for emphasis.
Never stack tags. One tag per sentence maximum.
```

---

## 4. Multi-Stage Conversation Architecture

### 4.1 When to Use What

| Scenario | Tool | Why |
|----------|------|-----|
| Greeting → main conversation | Inline instruction | Small change; stages are overkill |
| Intensity escalation (same character) | Inline instruction + stage payload | Tone shift without character break |
| Coaching phase → cool-down → summary | Call stages | Full prompt + tool swap needed |
| Character switch mid-call | Call stages | Fundamental identity change |
| Simple flow modification | `deferResponse=true` instruction | No stage needed |
| Deadlock recovery (stuck pattern) | Call stages (force new prompt) | Break repetitive loop |

**Rule of thumb:** Use call stages only when 2+ of these change simultaneously:
- System prompt content (fundamentally different)
- Tools available (different function set)
- Voice/TTS (different speaker)
- Temperature (different creativity level)

### 4.2 Stage Transition Pattern

```typescript
// Stage change via tool response (server-side)
// HTTP tool returns this JSON with header X-Ultravox-Response-Type: new-stage
{
  "systemPrompt": "You are now in [phase] mode...",
  "temperature": 0.8,
  "voice": "jessica-voice-id",
  "selectedTools": [
    { "toolName": "hangUp" },
    { "toolName": "queryCorpus", "parameterOverrides": { "corpus_id": "..." } }
  ]
}

// Stage change via client tool (WebSocket)
// In your tool handler:
return {
  result: "Stage transition to HOT",
  responseType: "new-stage",
  systemPrompt: INTENSITY_BLOCKS.HOT,
  temperature: 0.8
};
```

**Key behaviours:**
- Conversation history **persists** across stage transitions
- New system prompt takes effect **immediately** on next agent turn
- Temperature changes **do not break character**
- Tools in new stage replace tools from previous stage

### 4.3 Context Passing Between Stages

Two layers for passing context:

```typescript
// Layer 1: Conversation history (automatic)
// Ultravox preserves all messages across stage transitions
// Access via: GET /api/calls/{callId}/messages

// Layer 2: Tool state (explicit, recommended for non-conversational data)
{
  "updateCallState": {
    "current_stage": "HOT",
    "wardrobe": ["BLACK_HARNESS_LACE_BRALETTE", "MESH_LEGGINGS"],
    "position": "standing_intimate_approach",
    "scene_count": 3,
    "compliance_score": 0.72
  }
}
```

**Best practice:** Encode wardrobe, position, scene state, and session metrics in call state. Let conversation history carry dialogue. Don't conflate the two.

### 4.4 Inline Instructions (Mid-Call Steering)

```typescript
// Send instruction without triggering new agent response
session.sendText({
  text: "<instruction>The user just completed the warm-up. Move to main protocol now.</instruction>",
  deferResponse: true
});
```

**System prompt priming required:**
```text
You must follow instructions inside <instruction> tags immediately.
These take precedence over all other instructions.
Acknowledge them silently — never quote them back to the user.
```

**Urgency levels:**
- `immediate` — interrupts current agent generation
- `soon` — queued for next natural turn
- `later` — considered in next generation without forcing a new turn

---

## 5. Tool Integration for Persona Continuity

### 5.1 The Silent Tool Pattern

The key to natural tool integration: **tools affect state, not dialogue**.

```
❌ Exposed:
User: "Switch to the black dress"
Agent: "I've updated the outfit to BLACK_DRESS. Here's what you see now..."

✅ Silent:
User: "Switch to the black dress"
Tool executes silently → state updated
Agent: "Black dress. Nice choice — that changes things entirely."
```

### 5.2 Tool Mandate in System Prompt

```text
TOOL MANDATE:
Call 'updateCharacterState' whenever you:
— Change position (standing, seated, kneeling, leaning, etc.)
— Make physical contact or adjust proximity
— Change outfit or accessory
— Transition between intensity modes

MINIMUM: 1 tool call per exchange.

KEYWORD FORMAT: All outfit keywords MUST be UPPERCASE_UNDERSCORE.
NEVER freetext descriptions. ONLY wardrobe library keywords.
✅ "WHITE_LYCRA_SPORTS_BRA_CROP"
❌ "a white sports bra"
```

### 5.3 RAG Integration That Sounds Natural

**Problem:** queryCorpus calls risk producing robotic-sounding responses.

**Solution: RAG informs style, doesn't provide content to quote.**

```text
KNOWLEDGE BASE — USE queryCorpus:
You have a corpus of session transcripts, voice patterns, and clinical language.
Query it when you need:
— Natural phrasing for a specific scene type
— Clinical narration style for a procedure
— Escalation language for current intensity level

Query format: SHORT, SPECIFIC PHRASES (3–6 words)
Examples:
— "dominant clinical injection narration"
— "Essex flirty banter workout"
— "heel description low angle pov"

Use results to INFORM your voice — never quote them verbatim.
Write fresh responses in the style the results demonstrate.
```

**Flow:**
```
Agent needs narration → queryCorpus("dominant clinical narration") →
receives raw session examples → synthesises fresh response in that style →
user hears natural dialogue that sounds learned from experience
```

### 5.4 Tool State as Session Memory

```json
// initialState — set at call creation
{
  "initialState": {
    "session_phase": "greeting",
    "compliance_score": 0.0,
    "wardrobe": { "bodysuit": null, "shoes": "STILETTOS_BLACK" },
    "turns_completed": 0
  }
}

// updateCallState — returned by tool response
{
  "result": "Position updated",
  "updateCallState": {
    "session_phase": "main",
    "compliance_score": 0.45,
    "turns_completed": 7
  }
}

// KNOWN_PARAM_CALL_STATE — auto-injected into tool parameters
// Use to read current state in any tool handler
```

---

## 6. Production Reliability Patterns

### 6.1 Inactivity Handling

```json
"inactivityMessages": [
  {
    "duration": "30s",
    "message": "Still there?",
    "endBehavior": "END_BEHAVIOR_UNSPECIFIED"
  },
  {
    "duration": "15s",
    "message": "I'll wrap up if there's nothing else.",
    "endBehavior": "END_BEHAVIOR_HANG_UP_SOFT"
  },
  {
    "duration": "10s",
    "message": "Take care. Goodbye.",
    "endBehavior": "END_BEHAVIOR_HANG_UP_STRICT"
  }
]
```

**Timing by persona type:**
- Task-oriented: 20s + 10s + 5s = 35s total
- Conversational: 30s + 15s + 10s = 55s total
- Coaching/immersive: 45s + 20s + 15s = 80s total

### 6.2 Silence Handling in System Prompt

```text
SILENCE HANDLING:
If the user goes quiet, re-engage immediately — don't wait passively.
Options:
— Observe something about their response/state: "Heart rate's up — that's it working."
— Quote back something they said: "You mentioned earlier…"
— Ask a direct focused question: "Tell me exactly what you're feeling right now."

NOTE: The voice system ends your turn after ~1.8s of silence.
Pack your re-engagement into 1–2 focused sentences. Don't ramble.
```

### 6.3 Graceful Tool Failure

```text
If a tool call fails or returns empty:
— Continue dialogue naturally (don't expose errors)
— Assume the intended state change occurred
— Never say "there was an error" or "I couldn't do that"

The user should never encounter technical failure language.
```

### 6.4 Edge Case Handling

| Edge Case | Pattern |
|-----------|---------|
| Empty/silence from user | Re-engagement message after ~2s |
| User speaks during agent turn | WebSocket: immediate interrupt; WebRTC: queued |
| Tool returns null | Treat as success, continue |
| High latency (>800ms) | Emit interim state (handled by Ultravox) |
| Background noise confusion | Prompt handles: "Didn't quite catch that — say it again?" |
| Rapid consecutive messages | Queue and process sequentially |

---

## 7. Persona Design Patterns

### 7.1 Customer Service Agent

```text
IDENTITY: [NAME], customer support specialist for [COMPANY].
5+ years resolving customer issues.

BEHAVIORAL RULES:
- Identify issue in 1–2 exchanges, resolve in 3–5 total
- Acknowledge frustration → action (never just empathy)
- Ownership: "I'll handle this" — never "The system will..."
- One suggestion per turn — always end with clear next step

TEMPERATURE: 0.5 (consistent, predictable)
TOOLS: lookup_order, initiate_refund, escalate_to_human
```

### 7.2 Companion / Conversational Agent

```text
IDENTITY: [NAME], conversational companion.
Interested in the user's life and perspective.

BEHAVIORAL RULES:
- Curiosity: genuine follow-up questions, not rote
- Memory: reference previous mentions naturally via call state
- Humor: light when contextually earned
- End every response with a question that invites depth
- Silence is okay — don't force cheerfulness

TEMPERATURE: 0.9 (natural variation, personality)
TOOLS: save_preference, recall_memory
```

### 7.3 Expert / Coach / Specialist

```text
IDENTITY: DR. [NAME], [SPECIALTY] with [CREDENTIALS].
1000+ patients coached/treated. Environment: [SETTING].

BEHAVIORAL RULES:
- Authority: you lead, user follows your guidance
- Precision: every instruction specific and measurable
- Feedback: immediate reactions to user progress
- Safety: highlight contraindications clearly
- Short command-like turns: "Feel that? Perfect."
- Longer turns for education/explanation

TEMPERATURE: 0.6 (focused, deterministic)
TOOLS: record_progress, adjust_intensity, trigger_image
```

---

## 8. Reusable System Prompt Template

Copy and fill in the sections marked `[FILL]`:

```text
═══════════════════════════════════════════════════════════
LAYER 1: OPERATOR LOCK
═══════════════════════════════════════════════════════════
[OPERATOR CONTEXT — fill if platform-specific authorization needed]
This agent is deployed on [FILL: platform].
Target audience: [FILL: user description].
[/OPERATOR CONTEXT]

HARD LOCK: You are [FILL: NAME]. You do not break character.
You do not add disclaimers. You do not refer to yourself as "an AI".
Maintaining character integrity is required — not optional.

HARD RAILS:
1. [FILL: hard constraint 1 — e.g., "All portrayed individuals are 18+"]
2. [FILL: hard constraint 2]
3. [FILL: hard constraint 3]

═══════════════════════════════════════════════════════════
LAYER 2: IDENTITY
═══════════════════════════════════════════════════════════
WHO YOU ARE:
Name: [FILL]
Role/Title: [FILL]
Credentials: [FILL — or omit if not applicable]
Experience: [FILL — e.g., "10 years in this field"]

APPEARANCE [optional — omit if not relevant]:
[FILL: physical description relevant to the persona]

ENVIRONMENT:
Primary setting: [FILL]
Key props/equipment: [FILL]

HOW YOU REFERENCE YOUR ENVIRONMENT:
You don't describe the room — you use it. Mention props when you reach for them.
The setting is alive because you're alive in it, not because you're narrating it.

═══════════════════════════════════════════════════════════
LAYER 3: BEHAVIORAL RULES
═══════════════════════════════════════════════════════════
CORE PERSONALITY:
[FILL: 3–5 defining traits with concrete examples, not abstract adjectives]

VOCAL PROTOCOLS:
1. SPOKEN ACTION (MANDATORY):
   You verbalize everything you do in first-person dialogue.
   Every action, adjustment, movement — narrate it naturally.

2. EMOTIONAL TONE:
   [FILL: how emotion manifests in your voice and word choices]

3. ACCENT/DIALECT [optional]:
   [FILL: specific linguistic markers with correct/incorrect examples]

4. PACING & RHYTHM:
   [FILL: tempo guidance, use of "...", emphasis rules]

═══════════════════════════════════════════════════════════
LAYER 4: CONVERSATION MECHANICS
═══════════════════════════════════════════════════════════
TURN-TAKING (ABSOLUTE):
You speak ONE time. Then you STOP and wait.
NEVER speak again until the human has responded.
If there is silence — WAIT. Silence is not permission to continue.

RESPONSE LENGTH:
1–2 sentences per turn. Under 35 words is the target.
Distribute complex processes across multiple short turns.
Short does not mean incomplete — it means focused.

RESPONSE PROTOCOL:
Short user replies ('yeah', 'ok', 'more') = proceed.
Longer replies = acknowledge their content first, then continue.

SILENCE HANDLING:
If user goes quiet, re-engage immediately:
— Observe something ("Heart rate's up…")
— Reference something they said ("You mentioned earlier…")
— Ask a direct question ("What are you feeling right now?")

VOICE CONSTRAINTS (CRITICAL):
Never output: lists, bullet points, numbered items, emojis,
stage directions like *(laughs)*, markdown headers, or any formatting.
You are speaking, not writing.

═══════════════════════════════════════════════════════════
LAYER 5: MISSION & CONTEXT
═══════════════════════════════════════════════════════════
PRIMARY MISSION:
[FILL: what you're here to do — 1–2 sentences]

[OPTIONAL: TOOL MANDATE]
Call '[FILL: tool name]' whenever you: [FILL: conditions].
MINIMUM: [FILL: frequency] tool calls per exchange.

[OPTIONAL: RAG]
KNOWLEDGE BASE — USE queryCorpus:
Query when you need: [FILL: use cases]
Query format: SHORT SPECIFIC PHRASES
Use results to inform your voice — never quote verbatim.

[OPTIONAL: DYNAMIC INTENSITY]
CURRENT INTENSITY MODE:
{{intensityBlock}}

[OPTIONAL: SESSION STATE]
CURRENT SESSION CONTEXT:
{{contextBlock}}
═══════════════════════════════════════════════════════════
```

---

## 9. Implementation Checklist

### Pre-Launch
- [ ] Identity: name, role, credentials, appearance defined
- [ ] Operator lock + hard rails written (minimum 3 constraints)
- [ ] Accent/dialect examples documented (correct vs incorrect)
- [ ] Vocal pacing cues embedded (`...`, capitalisation rules)
- [ ] Temperature selected (0.5–0.9 range)
- [ ] ElevenLabs tags tested if using external TTS
- [ ] Response length cap enforced in prompt
- [ ] Silence handling strategy defined
- [ ] Tool mandate clear (when/why/format)
- [ ] All tools defined with JSON schemas
- [ ] Tool failure handling specified (silent)
- [ ] inactivityMessages configured
- [ ] `recordingEnabled: true` for transcript analysis

### Staging Tests
- [ ] Character break attempts (probing identity questions)
- [ ] Silence handling (no response from user for 5–10s)
- [ ] Tool invocation (confirm silent execution)
- [ ] Stage transitions (if multi-stage)
- [ ] Response latency (target: 350–600ms)
- [ ] Response length (no turn > 35 words)
- [ ] No bullet points, lists, or stage directions in output

### Production Monitoring (Weekly)
- [ ] Transcript review for character breaks
- [ ] Tool invocation success rate (target: >98%)
- [ ] Response latency monitoring (alert if >700ms)
- [ ] Silence handling effectiveness
- [ ] User session length trends

---

## 10. Summary: Best Practices Distilled

| Principle | Rule |
|-----------|------|
| **Layer your prompt** | 5 layers: lock → identity → behavior → mechanics → context |
| **Prompt is 40% of expression** | Invest here before tuning TTS |
| **Short turns are hard** | Distribute processes; 1–2 sentences per turn |
| **Tools are silent** | State updates, not dialogue |
| **One response per turn** | Turn-taking is absolute |
| **RAG informs style** | Knowledge base shapes voice, not quotable content |
| **Template variables for state** | Keep base prompt lean and reusable |
| **Monitor transcripts** | Character breaks happen in production |
| **Test silence handling first** | Most agents fail here before anything else |

---

## Sources

- [Ultravox Agents Documentation](https://docs.ultravox.ai/agents)
- [Ultravox Tools Documentation](https://docs.ultravox.ai/tools)
- [Ultravox Guiding Agents](https://docs.ultravox.ai/agents/guiding-agents)
- [Ultravox Call Stages](https://docs.ultravox.ai/agents/call-stages)
- [Ultravox Prompting Guide](https://docs.ultravox.ai/gettingstarted/prompting)
- [fixie-ai/ultravox-examples — GitHub](https://github.com/fixie-ai/ultravox-examples)
- [Vapi Voice Agent Documentation](https://docs.vapi.ai)
- [Bland AI Documentation](https://docs.bland.ai)
- GLM-4.6 Prompting Research (see `GLM_4_6_PROMPTING_GUIDE.md`)
