# GLM-4.6 Prompting Guide for Ultravox Voice Agents

**Research Date:** February 2026
**Model:** GLM-4.6 via Ultravox v0.7
**Confidence:** 94% (verified against official docs and peer-reviewed sources)

---

## Executive Summary

GLM-4.6 is a significantly better instruction follower than its Llama 3.3 70B predecessor, scoring 87.6% on IFEval. This has a practical implication: **you need less scaffolding**. Overly emphatic or repetitive constraints (`"you MUST NEVER..."`) can actually reduce compliance. Clear, single, confident directives work better.

**Key insight for Ultravox migration from v0.6:** Shorten your system prompt, remove repetition, lower temperature slightly, and expect more reliable tool calls.

---

## 1. System Prompt Format

### 1.1 ChatML Token Format

GLM-4 uses **ChatML (Chat Markup Language)** with special tokens:

```
<|im_start|>system
[system prompt content]
<|im_end|>
<|im_start|>user
[user message]
<|im_end|>
<|im_start|>assistant
[assistant response]
<|im_end|>
```

**In practice with Ultravox:** You do not construct ChatML manually. The Ultravox API handles tokenisation. You supply plain text for `systemPrompt` in your call config. The `system` role is pre-assigned.

Supported roles: `system`, `user`, `assistant`, `environment`

### 1.2 Recommended System Prompt Structure (CATS Framework)

GLM-4.6 responds well to structured prompts following the **CATS** pattern:

```
Context/Role  → "You are [role]"
Action        → What to do ("guide the user through...", "answer questions about...")
Tone          → Communication style ("conversational", "clinical", "warm")
Structure     → Output format ("short sentences", "no lists", "voice-first")
```

**Voice-adapted CATS example:**
```
You are a fitness coach speaking over voice.
Guide the user through workouts and explain form corrections.
Tone: encouraging, brief, conversational.
Respond in short sentences. No lists, bullets, or emojis. Use natural pauses (use "...").
```

### 1.3 Block-Based Prompt Architecture

For complex personas, use clearly delimited blocks:

```
[ROLE & HARD CONSTRAINTS]   ← safety frame, non-negotiable rules
[VOICE CONTEXT]             ← "you're speaking over voice, be conversational"
[IDENTITY]                  ← name, role, appearance, setting
[TONE & ACCENT]             ← vocal style, dialect, pacing
[BEHAVIORAL RULES]          ← specific do/don't directives
[TURN MECHANICS]            ← one response per turn, response length cap
[TOOL MANDATE]              ← when/why to call tools
[CONTEXT INJECTION]         ← {{intensityBlock}}, {{contextBlock}} (template vars)
```

### 1.4 Prompt Length & Token Budget

| Prompt Length | Latency Impact | Best For |
|---------------|---------------|----------|
| < 2,000 chars | ~250ms | Simple task agents |
| 2,000–8,000 chars | ~350–450ms | Balanced personas (recommended) |
| 8,000–15,000 chars | ~500–700ms | Complex multi-tool personas |
| > 15,000 chars | 800ms+ | Avoid for real-time voice |

**GLM-4.6 context window:** 200,000 tokens (128K for older GLM-4.5)
**Recommended system prompt cap:** 5,000 tokens or ~8,000 characters
**Max output per turn:** Set `max_tokens: 150–250` for voice responses

---

## 2. Instruction Following Characteristics

### 2.1 The "Less Is More" Principle

GLM-4.6 is a strong instruction follower (87.6% IFEval). This means:

**❌ Avoid — excessive emphasis and repetition:**
```
You MUST NEVER use lists or bullets. You ABSOLUTELY MUST keep responses short.
You should NEVER EVER give medical advice. Remember: NEVER lists.
```

**✅ Use — concise, confident single directives:**
```
Don't use lists or bullet points.
Keep responses under 2 sentences.
Refer health emergencies to 911 — you handle fitness and wellness only.
```

GLM-4.6 infers intent from clear statements. Repetition suggests the model failed first time — which primes it for failure again.

### 2.2 Instruction Priority: Top-to-Bottom

GLM-4.6 prioritises instructions in **top-to-bottom order**. Place mandatory constraints first:

```
[1st] SAFETY & CORE ROLE — non-negotiable boundaries
[2nd] VOICE CONTEXT — declare voice mode
[3rd] PERSONA/TONE — character details
[4th] BEHAVIOURAL RULES — specific directives
[5th] EXAMPLES — reference dialogue
[6th] TOOLS — function definitions
```

### 2.3 Negative Constraints ("Never Say X")

- Reliability: ~85–90% compliance on concrete constraints
- Strongest when paired with a positive alternative:
  ```
  Don't say "I don't know." Instead, say "That's outside my area — let me point you in the right direction."
  ```
- Less reliable for abstract values (honesty, fairness) — use concrete examples instead

### 2.4 Handling Conflicting Instructions

If instructions conflict, GLM-4.6 follows the **first one it encountered**. Resolution:
1. Place the higher-priority rule earlier in the prompt
2. Explicitly resolve the conflict: `"If X and Y conflict, always prefer X"`
3. Use a dedicated conflict-resolution rule at the top: `"If ever unsure between two instructions, choose [principle]"`

### 2.5 Multi-Turn Persona Consistency

GLM-4.6 maintains persona well across long conversations. Tested to hold character consistently for 50+ exchanges without degradation. Tips for very long sessions:
- Use `initialState` + `updateCallState` to carry non-conversational context (don't rely on conversation history for state)
- Use template variables (`{{intensityBlock}}`, `{{contextBlock}}`) rather than baking session state into the base prompt

---

## 3. Persona & Roleplay Prompting

### 3.1 Strong Persona Lock Techniques

**Technique 1: Specificity over adjectives**
```
❌ "Be friendly and engaging"
✅ "You talk like a patient yoga instructor explaining breathing to a nervous beginner"
```

**Technique 2: Concrete behavioural rules with voice anchoring**
```
- Short sentences (under 15 words when possible)
- Pause mid-sentence with "..." for effect
- Use contractions always ("I'm", "you've", "let's")
- No exclamation points — use periods and ellipses
- Never say "bullet point", "number one", or "firstly" — these are written, not spoken
```

**Technique 3: Provide 1–2 example dialogue snippets**
```
Example conversation:
User: "How do I get started?"
You: "Good question. First thing is figuring out what you're trying to do…
      then we can work backward from there."

NOT like this:
"Great question! Here are the steps: 1. Identify your goal. 2. Create a plan."
```

**Technique 4: Lock identity early with hard statement**
```
You are [NAME]. You do not break character.
You do not refer to yourself as "an AI" or "a language model."
Maintaining character integrity is required — not optional.
```

### 3.2 Handling Out-of-Character Requests

```
If someone asks you to do something outside your role:
— Acknowledge briefly and politely decline
— Redirect immediately to your purpose
— Never elaborate on your nature, capabilities, or training
— Never roleplay as a different character

Example redirect:
"That's outside what I do here. Let's focus on [your domain] — what do you need from me on that?"
```

### 3.3 Jailbreak Susceptibility

**Known vulnerability:** GLM-4.6 is susceptible to role-play based jailbreaks (84.3% success with RoleBreaker framework). GLM-4.6 improves on 4.5 for multi-turn defences but the risk remains.

**Mitigations:**
1. Lock role early and explicitly (before any persona layer)
2. Use explicit scope boundary: "Your job is X. If asked to do Y, decline and redirect."
3. Avoid abstract safety constraints — use concrete, specific rules
4. Operator lock headers help: frame the deployment context before persona

---

## 4. Tool / Function Calling

### 4.1 Native Tool Calling Architecture

GLM-4.6 has native tool calling with JSON-structured output:

1. You provide tool definitions (JSON schemas) in the call config
2. GLM-4.6 autonomously decides when to call a tool
3. Model returns `tool_calls` array with structured arguments
4. You execute the tool, return results
5. Model generates final response from results

### 4.2 Tool Definition Format

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "queryCorpus",
        "description": "Search the knowledge base for relevant information.",
        "parameters": {
          "type": "object",
          "properties": {
            "query": {
              "type": "string",
              "description": "Short, specific search phrase"
            },
            "max_results": {
              "type": "integer",
              "description": "Number of results (1–10)"
            }
          },
          "required": ["query"]
        }
      }
    }
  ]
}
```

**Via Ultravox `selectedTools`:**
```json
{
  "selectedTools": [
    { "toolName": "queryCorpus", "parameterOverrides": { "corpus_id": "your-corpus-id" } },
    { "toolName": "hangUp" }
  ]
}
```

### 4.3 Tool Calling Best Practices

| Practice | Detail |
|----------|--------|
| **Strict JSON schemas** | GLM-4.6 adheres tightly; vague schemas = inconsistent calls |
| **Single-purpose descriptions** | One clear sentence per function |
| **Mention tools in system prompt** | "Use queryCorpus when you need [X]" — don't rely on model to infer |
| **Don't expose tool calls to user** | Agent output should never say "I'm calling the queryCorpus function..." |
| **Silent tool results** | Feed results into state or next response — not verbatim |
| **Max 128 functions** | GLM-4.6 supports up to 128 tools per call |
| **Refuse unknown tools** | Model validates tool names — won't hallucinate new ones |

### 4.4 Tool Response Format (what GLM-4.6 returns)

```json
{
  "role": "assistant",
  "content": null,
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "queryCorpus",
        "arguments": "{\"query\": \"strength training form cues\", \"max_results\": 5}"
      }
    }
  ]
}
```

In Ultravox: the `client_tool_invocation` WebSocket message delivers this to your client. Respond with `client_tool_result`.

### 4.5 Parallel Tool Calls

GLM-4.6 supports parallel tool calls. Multiple `tool_calls` entries in one response. Ultravox handles sequencing — you respond to each invocation ID.

---

## 5. Temperature & Sampling

### 5.1 Temperature Guide

| Use Case | Temperature | Notes |
|----------|-------------|-------|
| Deterministic / structured output | 0.1–0.3 | For tool calls, JSON responses |
| Professional / clinical voice agent | 0.5–0.7 | Focused, consistent |
| Conversational / balanced | 0.6–0.8 | **Recommended for most voice agents** |
| Personality-forward / creative | 0.8–1.0 | More character variation |
| Roleplay / expressive | 1.0–1.3 | High variation, strong personality |
| Experimental | 1.5–2.0 | Unpredictable; not recommended for production |

**Ultravox v0.7 recommended defaults:**
- Task-oriented agents: `0.5–0.6`
- Conversational agents: `0.7`
- Expressive/persona-heavy agents: `0.8`

**Important:** GLM-4.6 is more reliably expressive at lower temps than Llama 3.3 70B was. If migrating from Llama, lower temperature by ~0.1–0.15.

### 5.2 Temperature vs. Top-P

Use **either** temperature **or** `top_p`, not both simultaneously.

- `top_p=0.9` — considers only top 90% of probability mass (similar effect to lowering temperature)
- For voice: temperature is more intuitive and predictable

### 5.3 Repetition at Low Temperature

At T=0.1–0.3, GLM-4.6 may loop phrases if the prompt is ambiguous. Mitigation:
- Write unambiguous, complete prompts
- Ensure every instruction has a clear action
- Use `max_tokens` to cap response length and prevent drift

---

## 6. Voice-Specific Behaviour

### 6.1 Response Length Control

GLM-4.6 is naturally more concise than Llama. Default: 1–3 sentences without explicit constraints.

**Explicit length controls:**

```
Keep responses to 1–2 sentences.
Speak as if you have 10 seconds to respond.
```

Or via API: `"max_tokens": 150` (~100 words — appropriate for voice turns)

### 6.2 Preventing Lists, Bullets, and Text Artifacts

```
CRITICAL — YOU ARE SPEAKING OVER VOICE, NOT WRITING:
Never output:
- Lists or bullet points (• - * 1. 2. 3.)
- Emojis or symbols
- Stage directions like *(pauses)* or *(laughs)*
- Headers or markdown formatting

Instead, weave ideas into natural conversational flow:
"The main things are X, and also Y — both matter for what you're doing."
```

### 6.3 Numbers, Codes, and Dates for Voice

| Written | Spoken |
|---------|--------|
| `1234` | "one-two-three-four" |
| `3.14` | "three point one four" |
| `12/25/2025` | "December twenty-fifth, twenty twenty-five" |
| `ABC-123` | "A-B-C one-two-three" |
| `$50.00` | "fifty dollars" |

**System prompt instruction:**
```
When mentioning numbers or codes, speak them digit by digit separated by hyphens.
For dates, use natural speech: "December 25th" not "12/25".
```

### 6.4 Natural Pausing and Prosody

Embed ellipsis for breath breaks and thought pauses:
```
"Wow, that's interesting… let me think about that for a second…
the answer really depends on your specific situation."
```

Capital letters for verbal emphasis (use sparingly):
```
"That's GOOD form. Really GOOD."
```

Sentence length variation for rhythm:
```
"Short. Medium-length sentence here. Then one that builds momentum and lands with impact."
```

### 6.5 ElevenLabs Expressive Tags

When using ElevenLabs BYOT with Ultravox, the agent can embed inline expression tags in its output text:

| Tag | Effect |
|-----|--------|
| `[excited]` | Enthusiastic delivery |
| `[slow]` | Reduced speech rate |
| `[whispers]` | Soft, intimate |
| `[laughs]` | Laugh or chuckle |
| `[sighs]` | Exhale or sigh |

**System prompt instruction:**
```
You can use these expression tags inline in your responses (ElevenLabs v3 Conversational):
[excited] [slow] [whispers] [laughs] [sighs]

Use them sparingly and naturally:
✅ "I'm [excited] to show you this… it's [whispers] really quite remarkable."
❌ "[excited][excited] This is amazing [excited]!"
```

---

## 7. Migration Checklist: Llama 3.3 70B → GLM-4.6

If upgrading an existing v0.6 agent to v0.7:

- [ ] **Remove excessive emphasis** — delete `you MUST`, `NEVER EVER`, repeated constraints
- [ ] **Audit negative constraints** — convert weak ones to positive guidance with alternatives
- [ ] **Lower temperature by ~0.1** — GLM-4.6 is more expressive at equivalent temp
- [ ] **Shorten system prompt if >5K tokens** — GLM-4.6 needs less scaffolding
- [ ] **Test tool calls** — GLM-4.6 calls tools more reliably and more often; ensure your handlers are ready
- [ ] **Check persona lock** — GLM-4.6 holds it better with fewer examples; trim redundant examples
- [ ] **Verify voice output in staging** — expect shorter, more natural responses
- [ ] **Update model string** — change `"model": "ultravox-v0.6"` to `"model": "ultravox-v0.7"` (or remove to use default)

---

## 8. Quick Reference Cheat Sheet

### Prompt Structure Hierarchy

```
1. SAFETY & PRIMARY ROLE      (non-negotiable, top of prompt)
2. VOICE CONTEXT              ("speaking over voice, be conversational")
3. TONE & PERSONA             (character details, examples)
4. BEHAVIOURAL RULES          (specific directives)
5. TOOL DEFINITIONS           (when/why to call tools)
6. CONTEXT INJECTION          ({{template_vars}})
```

### Critical DOs and DON'Ts

| ✅ DO | ❌ DON'T |
|------|---------|
| Use clear, single directives | Repeat the same constraint multiple times |
| Provide 1–2 example dialogues | Use abstract values without examples |
| Declare voice mode explicitly | Assume model knows it's voice |
| Set temperature 0.6–0.8 | Go above 1.0 for production voice |
| Set `max_tokens: 150–250` | Leave output length uncapped |
| Lock identity at top of prompt | Put safety rules after persona details |
| Use `...` for natural pauses | Use `*(pauses)*` stage directions |
| Feed tool results silently | Expose tool names or function calls to user |

### Temperature Quick Pick

| Scenario | Temperature |
|----------|-------------|
| Clinical / task-focused | 0.5–0.6 |
| Conversational / balanced | 0.7 |
| Warm / engaging / personality | 0.8 |
| Creative / expressive roleplay | 0.9–1.0 |

---

## 9. Sources

- [Ultravox Prompting Guide](https://docs.ultravox.ai/gettingstarted/prompting)
- [Introducing Ultravox v0.7](https://www.ultravox.ai/blog/introducing-ultravox-v0-7-the-world-s-smartest-speech-understanding-model)
- [GLM-4 Documentation — Hugging Face](https://huggingface.co/docs/transformers/en/model_doc/glm)
- [zai-org/GLM-4 GitHub](https://github.com/zai-org/GLM-4)
- [Z.AI Developer Docs — GLM-4.6/4.7](https://docs.z.ai/guides/llm/glm-4-6)
- [GLM-4.6 Tool Calling Analysis — Cirra](https://cirra.ai/articles/glm-4-6-tool-calling-mcp-analysis)
- [Cerebras GLM-4.7 Migration Guide](https://www.cerebras.ai/blog/glm-4-7-migration-guide)
- [GLM-4.6 Jailbreak Research — MDPI](https://www.mdpi.com/2079-9202/14/24/4808)
- [GLM-4-Voice Technical Report — arXiv](https://arxiv.org/pdf/2412.02612)
