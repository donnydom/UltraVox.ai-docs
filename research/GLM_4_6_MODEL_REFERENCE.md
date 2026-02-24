# GLM-4.6 Model Reference for Ultravox v0.7

**Research Date:** February 2026
**Ultravox Version:** v0.7 (default model switched to GLM-4.6 on December 22, 2025)
**Sources:** Fixie AI blog, Hugging Face model cards, Zhipu AI documentation, academic papers

---

## Executive Summary

Ultravox v0.7 switched its default LLM backbone from **Llama 3.3 70B** to **GLM-4.6** on December 22, 2025. This change was driven by two primary user requests: better instruction following and more reliable tool calling. GLM-4.6 is a 357-billion-parameter Mixture-of-Experts model by Zhipu AI, with only 28.72B parameters active per inference pass. It brings a 200K context window, MIT licensing, and substantially improved agentic capabilities over its predecessor.

**Key fact:** GLM-4.6 is a pure text LLM. Ultravox wraps it with an audio encoder + projector to create the audio-native pipeline. GLM-4.6 never "hears" audio directly — it receives projected audio embeddings.

---

## 1. Model Identity

| Attribute | Value |
|-----------|-------|
| **Full Name** | GLM-4.6 |
| **Developer** | Zhipu AI (Z.AI) / THUDM |
| **License** | MIT (open-source, commercial use allowed) |
| **Release Date** | September 30, 2025 |
| **Hugging Face** | `zai-org/GLM-4.6` |
| **Ultravox HF** | `fixie-ai/ultravox-v0_7-glm-4_6` |
| **Default in Ultravox** | From December 22, 2025 |
| **Model string** | `ultravox-v0.7` |

---

## 2. Architecture

### 2.1 Model Type: Sparse Mixture-of-Experts (MoE)

| Parameter | Value |
|-----------|-------|
| **Total Parameters** | 357 billion |
| **Active Parameters (per forward pass)** | 28.72 billion |
| **Expert Layers** | 160 experts per MoE layer |
| **Active Experts** | 8 of 160 per token |
| **Context Window** | 200,000 tokens |
| **Maximum Output** | 128,000 tokens |

**Key architectural features:**
- **Grouped Query Attention (GQA):** Reduces memory footprint, improves long-range attention
- **QK-Norm:** Stabilises attention logits
- **Sparse Expert Routing:** Learned router activates 8 of 160 experts per token — quality of a large model at cost of a smaller one
- **30% token reduction** vs GLM-4.5 for equivalent tasks

### 2.2 Training

| Aspect | Details |
|--------|---------|
| **Training Data** | 10 trillion tokens |
| **Primary Languages** | Chinese and English (balanced) |
| **Supported Languages** | 26 languages |
| **Post-Training** | Multi-stage: supervised fine-tuning + RLHF |

---

## 3. Ultravox Audio-Native Architecture

GLM-4.6 is the reasoning backbone. Ultravox adds the audio layer on top:

```
[Raw Audio Input — 16kHz mono]
        ↓
[Audio Encoder]
  Whisper-medium encoder + Llama 3.1-8B backbone
  Converts audio waveform → high-dimensional audio embeddings
  (NO transcription — this is not ASR)
        ↓
[UltravoxProjector — trainable adapter]
  Maps audio embeddings → GLM-4.6 token embedding space
  Audio becomes "soft tokens" GLM-4.6 can reason over
        ↓
[GLM-4.6 LLM backbone]
  Processes: soft audio tokens + system prompt + conversation history
  Activates 8 of 160 experts per token (28.72B active params)
  Outputs: text response + optional tool calls
        ↓
[Response streamed to TTS / client]
```

### 3.1 What "No ASR" Actually Means

Traditional voice AI:
```
Audio → [ASR: speech-to-text] → Text → [LLM] → Text → [TTS] → Audio
```
Information lost at ASR stage: tone, inflection, emotion, timing, paralinguistic cues.

Ultravox v0.7:
```
Audio → [Encoder → Projector → GLM-4.6] → Text → [TTS] → Audio
```
No transcription step. Audio embeddings fed directly into LLM token space. Paralinguistic context preserved in embedding space (future roadmap: native emotion/tone understanding).

### 3.2 GLM-4.6's Role in the Pipeline

GLM-4.6 handles:
- **Reasoning** over the audio soft tokens + text context
- **Tool calling** (native JSON-structured output, up to 128 functions)
- **Response generation** (text streamed to TTS)
- **Instruction following** (persona adherence, constraint enforcement)
- **Conversation state management** via call state + tool state

GLM-4.6 does NOT handle:
- Audio encoding (that's Whisper-medium + Llama 3.1-8B)
- TTS (that's ElevenLabs, Cartesia, LMNT, or other external provider)
- WebRTC/WebSocket connection management (that's Ultravox infrastructure)

---

## 4. Why Ultravox Switched from Llama 3.3 70B

### 4.1 User-Requested Improvements

Per the [Ultravox v0.7 announcement](https://www.ultravox.ai/blog/introducing-ultravox-v0-7-the-world-s-smartest-speech-understanding-model):
1. **Better instruction following** — more reliable adherence to system prompt constraints
2. **More reliable tool calling** — accurate function invocation, fewer hallucinated arguments

### 4.2 Model Comparison

| Criterion | Llama 3.3 70B | GLM-4.6 | Winner |
|-----------|--------------|---------|--------|
| **Parameters** | 70B dense | 357B total / 28.72B active (MoE) | GLM-4.6 |
| **Context Window** | 128K | 200K | GLM-4.6 |
| **Instruction Following** | Strong | Substantially better (87.6% IFEval) | GLM-4.6 |
| **Tool Calling** | Capable | Specialized, up to 128 functions, low hallucination | GLM-4.6 |
| **Multilingual** | English-optimised | 26 languages, balanced Chinese+English | GLM-4.6 |
| **Reasoning** | Good | ~48.6% win rate vs Claude Sonnet 4 on CC-Bench | GLM-4.6 |
| **License** | Meta License | MIT | GLM-4.6 |
| **Inference speed (Ultravox)** | v0.6 baseline | 20% faster in v0.7 | GLM-4.6 |

### 4.3 Tool Calling Specifics

GLM-4.6 native tool calling:
- Outputs wrapped in `<tool_call>` / `</tool_call>` tags
- Supports up to **128 functions per call**
- Autonomously decides whether a tool call is necessary
- Refuses unrecognised tools (low hallucination rate)
- Compatible with MCP, LangChain, and standard agentic frameworks

---

## 5. Performance Benchmarks

| Benchmark | GLM-4.6 Result | Context |
|-----------|---------------|---------|
| **IFEval (instruction following)** | 87.6% | Approaches GPT-4-Turbo |
| **CC-Bench (real-world coding)** | 48.6% win rate vs Claude Sonnet 4 | Competitive frontier performance |
| **AIME 25** | Competitive | Math reasoning |
| **GPQA** | Competitive | Graduate-level reasoning |
| **Ultravox AIEWF benchmark** | Outperforms GPT Realtime, Gemini Live, Nova Sonic, Grok Realtime | Tool calling + instruction following in voice |

---

## 6. Deployment Options

### 6.1 Via Fixie AI API (Recommended)

```bash
# Create a call using default v0.7 (GLM-4.6)
curl -X POST https://api.ultravox.ai/api/calls \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"systemPrompt": "...", "model": "ultravox-v0.7"}'
```

**Pricing:** $0.05/minute (same as v0.6 — no price increase for upgrade)

### 6.2 Self-Hosting

- **Hugging Face:** `fixie-ai/ultravox-v0_7-glm-4_6`
- **Quantization:** AWQ and GGUF versions available
- **License:** MIT (GLM-4.6) — commercial self-hosting permitted

### 6.3 Staying on Llama 3.3 70B (Opt-out)

```json
{ "model": "ultravox-v0.6" }
// or
{ "model": "ultravox-v0.6-llama3.3-70b" }
```

**Note:** `fixie-ai/ultravox-qwen3-32b-preview` was deprecated on December 22, 2025.

---

## 7. Version History

| Date | Event |
|------|-------|
| Sept 30, 2025 | GLM-4.6 released by Zhipu AI (200K context, 357B params, MIT) |
| Dec 22, 2025 | Ultravox v0.7 released — GLM-4.6 becomes default model |
| Dec 2025 | GLM-4.7 released by Zhipu AI (not yet in Ultravox) |
| Feb 2026 | Current state |

---

## 8. Key Takeaways for Agent Developers

1. **GLM-4.6 is a frontier-scale reasoning model** running efficiently via MoE (28.72B active params despite 357B total)
2. **Audio understanding is Ultravox's layer** — GLM-4.6 handles reasoning, not audio
3. **Tool calling is a strength** — reliable, structured, low hallucination; use it aggressively
4. **200K context** — supports very long multi-turn conversations and large system prompts
5. **Instruction following improved significantly** — you need less scaffolding than with Llama
6. **MIT licensed** — self-hosting is viable for production at scale
7. **Multilingual** — strong Chinese+English, 24 additional languages

---

## 9. Sources

- [Introducing Ultravox v0.7 — Fixie AI Blog](https://www.ultravox.ai/blog/introducing-ultravox-v0-7-the-world-s-smartest-speech-understanding-model)
- [Ultravox How It Works](https://docs.ultravox.ai/gettingstarted/how-ultravox-works)
- [GLM-4.6 Hugging Face Model Card](https://huggingface.co/zai-org/GLM-4.6)
- [Fixie AI Ultravox v0.7 on HuggingFace](https://huggingface.co/fixie-ai/ultravox-v0_7-glm-4_6)
- [Z.AI Developer Docs — GLM-4.6](https://docs.z.ai/guides/llm/glm-4-6)
- [GLM-4.6 Tool Calling Analysis — Cirra](https://cirra.ai/articles/glm-4-6-tool-calling-mcp-analysis)
- [GLM-4-Voice Technical Report — arXiv](https://arxiv.org/abs/2412.02612)
- [Ultravox Changelog](https://docs.ultravox.ai/changelog/news)
