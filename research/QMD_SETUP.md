# QMD Setup for Ultravox Docs

This repository is designed to be indexed with [qmd](https://github.com/tobi/qmd) — a fast local markdown search engine — so that Claude Code agents can search across all documentation efficiently.

## What is qmd?

qmd is a local BM25 + vector search engine for markdown files. Once you index this repo, Claude Code agents and you can run:

```bash
qmd search "GLM-4.6 tool calling"
qmd search "call stages vs inline instructions"
qmd search "ElevenLabs expressive tags"
qmd search "persona lock techniques"
```

Instead of reading 170+ files, agents get instant ranked results.

## Quick Setup

### 1. Install qmd (if not already installed)

```bash
# Requires Bun
bun install -g https://github.com/tobi/qmd
```

### 2. Add this repo as a collection

```bash
# After cloning to your local path:
qmd collection add /path/to/UltraVox.ai-docs --name ultravox-docs --mask "**/*.{md,mdx}"
```

### 3. Update index

```bash
qmd update
```

### 4. (Optional) Build semantic search embeddings

```bash
qmd embed  # Slow on first run — downloads local GGUF model
```

### 5. Verify

```bash
qmd search "GLM-4.6 architecture" -c ultravox-docs -n 5
```

---

## Recommended qmd Queries for This Repo

### Model & Architecture
```bash
qmd search "GLM-4.6 MoE architecture parameters" -c ultravox-docs
qmd search "Ultravox audio encoder whisper projector" -c ultravox-docs
qmd search "audio native no ASR pipeline" -c ultravox-docs
```

### Prompting
```bash
qmd search "system prompt structure voice agent" -c ultravox-docs
qmd search "GLM-4.6 instruction following temperature" -c ultravox-docs
qmd search "persona lock character break prevention" -c ultravox-docs
qmd search "tool calling function definition JSON schema" -c ultravox-docs
```

### API & Integration
```bash
qmd search "call stages new-stage responseType" -c ultravox-docs
qmd search "inline instructions deferResponse" -c ultravox-docs
qmd search "ElevenLabs externalVoice BYOT" -c ultravox-docs
qmd search "WebSocket message protocol client server" -c ultravox-docs
qmd search "queryCorpus RAG corpus" -c ultravox-docs
```

### Voice Persona Design
```bash
qmd search "5-layer persona architecture operator lock" -c ultravox-docs
qmd search "silent tool pattern persona continuity" -c ultravox-docs
qmd search "inactivity messages silence handling" -c ultravox-docs
qmd search "voice expression hierarchy prompt temperature TTS" -c ultravox-docs
```

---

## File Map (for direct reads)

| File | What's in it |
|------|-------------|
| `research/INDEX.md` | Navigation hub — start here |
| `research/TECHNICAL_RESEARCH.md` | Full API reference (calls, tools, WebSocket, stages) |
| `research/QUICK_REFERENCE.md` | One-pager for fast lookup |
| `research/GLM_4_6_MODEL_REFERENCE.md` | GLM-4.6 architecture, Ultravox pipeline, benchmarks |
| `research/GLM_4_6_PROMPTING_GUIDE.md` | Prompting best practices, ChatML, tool calling format |
| `research/VOICE_PERSONA_DESIGN.md` | 5-layer persona architecture, reusable template |
| `gettingstarted/` | Official Ultravox getting started docs |
| `api-reference/` | Official REST API reference (.mdx) |
| `agents/` | Official agent building docs |
| `tools/` | Official tools/RAG docs |
| `voices/` | Official voice configuration docs |
| `apps/` | Official WebSocket/SDK docs |
| `telephony/` | Official telephony/SIP docs |

---

## For Claude Code Agents

When building a new Ultravox voice agent, use this search order:

1. **Architecture question** → `qmd search "<question>" -c ultravox-docs`
2. **API parameter question** → Check `research/QUICK_REFERENCE.md` or `research/TECHNICAL_RESEARCH.md`
3. **Prompting question** → Check `research/GLM_4_6_PROMPTING_GUIDE.md`
4. **Persona design question** → Check `research/VOICE_PERSONA_DESIGN.md`
5. **Official spec deep dive** → Read the relevant `.mdx` file in `api-reference/` or `agents/`

Start with the `research/` folder — it's synthesised for agent consumption.
