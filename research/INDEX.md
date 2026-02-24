# Ultravox Research Documentation Index

Synthesised technical research on the Ultravox voice AI API — compiled February 22, 2026 from official documentation.

---

## Files in This Research Package

### 1. `TECHNICAL_RESEARCH.md` — Comprehensive Reference (1100+ lines)
Deep-dive implementation guide for developers building on Ultravox.

**Sections:**
- Architecture Overview (audio-native, no-ASR design)
- Call Configuration & API Parameters
- System Prompt & Persona Configuration
- Voice Expression & Prosody Control (4-level hierarchy)
- Tool Calling & Function Invocation
- Client ↔ Server Message Protocol (full type reference)
- WebSocket Setup & JoinURL
- Call Stages (Dynamic Multi-Phase)
- Advanced Features (Inline Instructions, Call State, RAG)
- SDK & Client Integration
- Ultravox vs ElevenLabs ConvAI comparison
- Known Issues & Edge Cases
- Production Implementation Checklist
- Quick Start Code Snippets (JS + Python)

**Best for:** Deep dives, implementation planning, debugging specific features

### 2. `QUICK_REFERENCE.md` — One-Pager
Fast lookup during development and production monitoring.

**Sections:**
- Voice Expression Control Hierarchy
- Core API Call Parameters (Minimal Example)
- Default Configuration Checklist
- Message Protocol Quick Lookup
- WebSocket vs REST vs SDK Tradeoff
- Tool Response Types
- ElevenLabs + Ultravox Integration
- Call State Management
- Call Stages
- RAG / Corpus Integration
- Common Pitfalls & Solutions (with agent PATCH gotchas)
- Production Monitoring (curl commands)
- API Quick Facts

**Best for:** Quick reference during development, production monitoring, configuration validation

---

## Key Findings Summary

### What is Ultravox?
Ultravox is an **audio-native voice AI platform** that:
- Processes speech directly without ASR (Automatic Speech Recognition)
- Preserves paralinguistic context (tone, emotion, inflection)
- Uses **GLM 4.6** as default model (switched from Llama 3.3 70B on Dec 22, 2025)
- Priced at **$0.05/minute** (deciminute billing)
- Offers **unlimited concurrency** on paid plans
- Supports "bring-your-own-telephony" (BYOT) and "bring-your-own-TTS" (BYOT)

### Core Capabilities
1. **Voice Expression Control:** 4-level hierarchy (prompt → temperature → text format → TTS tags)
2. **Tool Calling:** Built-in tools + custom HTTP/client tools with state persistence
3. **Message Protocol:** JSON-based bidirectional data + audio streaming
4. **Connection Methods:** WebRTC (SDK), WebSocket (server-to-server), Telephony (SIP)
5. **Advanced Features:** Call stages, inline instructions, tool state, RAG integration

---

## Quick Navigation

### If You Need To...

**Understand Ultravox architecture**
→ `TECHNICAL_RESEARCH.md` §1 (Architecture Overview)

**Set up a basic call**
→ `QUICK_REFERENCE.md` "Core API Call Parameters"

**Control voice expression**
→ `TECHNICAL_RESEARCH.md` §4 (Voice Expression & Prosody)
→ Or `QUICK_REFERENCE.md` "Voice Expression Control Hierarchy"

**Implement tool calling**
→ `TECHNICAL_RESEARCH.md` §5 (Tool Calling)

**Understand the message protocol**
→ `TECHNICAL_RESEARCH.md` §6 (Message Protocol)
→ Or `QUICK_REFERENCE.md` "Message Protocol Quick Lookup"

**Use WebSocket integration**
→ `TECHNICAL_RESEARCH.md` §7 (WebSocket Setup)
→ Code example: `TECHNICAL_RESEARCH.md` §14

**Implement multi-phase conversations**
→ `TECHNICAL_RESEARCH.md` §8 (Call Stages)

**Troubleshoot common issues**
→ `QUICK_REFERENCE.md` "Common Pitfalls & Solutions"

**Monitor production**
→ `QUICK_REFERENCE.md` "Production Monitoring"

**Compare Ultravox vs ElevenLabs**
→ `TECHNICAL_RESEARCH.md` §11

---

## Message Types Reference

### Client → Server
- `user_text_message` — Send user text with urgency
- `forced_agent_message` — Force agent action
- `client_tool_result` — Return tool result
- `set_output_medium` — Switch voice/text
- `hang_up` — End call
- `ping` — Measure latency

### Server → Client
- `state` — Agent state (idle/listening/thinking/speaking)
- `transcript` — Speech-to-text
- `client_tool_invocation` — Agent calls tool
- `call_started` — Call initialized
- `playback_clear_buffer` — WebSocket interrupt handling
- `pong` — Response to ping

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

---

## ElevenLabs Integration Quick Facts

| Setting | Value |
|---------|-------|
| Recommended Model | `eleven_turbo_v2_5` |
| Speed Range | 0.5–2.0 |
| Stability | 0.0–1.0 |
| Expressive Tags | `[excited]`, `[slow]`, `[whispers]`, `[laughs]`, `[sighs]` |
| Known Issues | Occasional slurring (May 2025+) |

---

## Research Sources

**Primary Documentation:**
- GitHub Repository: https://github.com/fixie-ai/ultradox
- Official Docs: https://docs.ultravox.ai/
- API Console: https://app.ultravox.ai/

**Research Date:** February 22, 2026
**Ultravox Version Covered:** v0.7 (default), v0.6 (legacy)
