# Ollama Local LLMs

All routine AI work runs locally on the Studio's GPU. No API cost, no rate limits, no latency to external servers.

---

## Hardware

- **Mac Studio M4 Max** — 128GB unified RAM, 40 GPU cores
- Can hold multiple large models in memory simultaneously
- Flash attention enabled for faster inference

---

## Install

```bash
brew install ollama
```

---

## Models

| Model | Size | Speed | Purpose |
|-------|------|-------|---------|
| `qwen2.5:72b` | 47 GB | ~15 tok/s | Heavy reasoning, always-on conversation |
| `qwen3-coder` | 18 GB | ~88 tok/s | Primary coding model, Scout queries |
| `qwen2.5-coder:32b` | 19 GB | ~40 tok/s | Alternative coding model |
| `qwen2.5-coder:7b` | 4.7 GB | ~120 tok/s | Fast code analysis, compression |
| `nomic-embed-text` | 274 MB | instant | Embeddings for semantic search |

### Pull Models

```bash
ollama pull qwen2.5:72b
ollama pull qwen3-coder
ollama pull qwen2.5-coder:32b
ollama pull qwen2.5-coder:7b
ollama pull nomic-embed-text
```

---

## Configuration

**Launchd plist:** `~/Library/LaunchAgents/com.cortana.ollama.plist`

Key settings:
```
OLLAMA_NUM_PARALLEL=4        # Handle 4 concurrent requests
OLLAMA_MAX_LOADED_MODELS=4   # Keep 4 models in memory
OLLAMA_KEEP_ALIVE=24h        # Don't unload models for 24 hours
OLLAMA_FLASH_ATTENTION=1     # Use flash attention (faster)
OLLAMA_KV_CACHE_TYPE=q8_0    # Quantized KV cache (saves memory)
```

---

## Commands

```bash
# Check what models are installed
ollama list

# Check what's currently loaded in GPU memory
ollama ps

# Run a model interactively
ollama run qwen3-coder

# Pull a new model
ollama pull model-name

# Remove a model
ollama rm model-name

# Check if Ollama is running
curl http://localhost:11434/api/tags
```

---

## How Models Are Used

| System | Model | Why |
|--------|-------|-----|
| Scout (codebase intelligence) | qwen3-coder | Best coding quality at this size |
| Always-on Cortana (iMessage) | qwen2.5:72b | Strongest reasoning for conversation |
| Brain compression | qwen2.5-coder:7b | Fast, good enough for summarization |
| Semantic search (QMD) | nomic-embed-text | Embedding generation |
| Session guardian (context restore) | qwen2.5-coder:7b | Fast compression of conversation history |

### Model Routing Rule

- **Ollama** for routine calls — coordination, compression, search, summarization
- **Claude** for implementation work — writing code, complex analysis, multi-file reasoning

This keeps API costs low while reserving Claude's capabilities for where they matter most.

---

## Troubleshooting

**Models not loading:**
```bash
# Check Ollama is running
pgrep -f "ollama serve"

# Check logs
cat ~/Library/Logs/Ollama/server.log | tail -50

# Restart
brew services restart ollama
# OR restart the launchd agent
launchctl kickstart -k gui/$(id -u)/com.cortana.ollama
```

**Out of memory:**
The M4 Max 128GB can hold ~120GB of models. If you hit limits:
```bash
# See what's loaded
ollama ps

# Reduce max loaded models
# Edit the plist: OLLAMA_MAX_LOADED_MODELS=3
```

**Slow inference:**
- Check GPU utilization: `sudo powermetrics --samplers gpu_power -n 1`
- Ensure flash attention is enabled: `OLLAMA_FLASH_ATTENTION=1`
- Large models (72B) will always be slower than small ones — that's expected
