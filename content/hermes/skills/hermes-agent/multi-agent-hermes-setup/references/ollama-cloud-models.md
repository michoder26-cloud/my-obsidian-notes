# Ollama Cloud Models Reference

## Provider Configuration
- **Provider name:** `ollama-cloud`
- **Base URL:** `https://ollama.com/v1`
- **API Key env var:** `OLLAMA_API_KEY`
- **Endpoint:** `https://ollama.com/v1/models` (OpenAI-compatible)

## Setup
```bash
# Set API key (use Python to avoid censoring)
python3 -c "
key = 'your_ollama_cloud_key'
with open('/root/.hermes/.env', 'a') as f:
    f.write(f'OLLAMA_API_KEY=*** + key + '\n')
"

# Configure profile
hermes config set model.provider ollama-cloud
hermes config set model.base_url "https://ollama.com/v1"
hermes config set model.default glm-5.2
```

## Available Models (as of 2026-06-18)

### Flagship / Large
| Model ID | Notes |
|----------|-------|
| `glm-5.2` | GLM 5.2 — excellent Thai support, recommended default |
| `glm-5.1` | GLM 5.1 |
| `glm-5` | GLM 5 |
| `glm-4.7` | GLM 4.7 |
| `minimax-m3` | MiniMax M3 — 428B MoE, 1M context |
| `minimax-m2.7` | MiniMax M2.7 |
| `minimax-m2.5` | MiniMax M2.5 |
| `minimax-m2.1` | MiniMax M2.1 |
| `deepseek-v4-pro` | DeepSeek V4 Pro |
| `deepseek-v4-flash` | DeepSeek V4 Flash |
| `deepseek-v3.2` | DeepSeek V3.2 |
| `deepseek-v3.1:671b` | DeepSeek V3.1 671B |
| `kimi-k2.7-code` | Kimi K2.7 Code — coding specialist |
| `kimi-k2.6` | Kimi K2.6 |
| `kimi-k2.5` | Kimi K2.5 |
| `mistral-large-3:675b` | Mistral Large 3 675B |
| `nemotron-3-ultra` | Nemotron 3 Ultra |
| `nemotron-3-super` | Nemotron 3 Super |
| `nemotron-3-nano:30b` | Nemotron 3 Nano 30B |
| `qwen3.5:397b` | Qwen 3.5 397B |

### Coder / Specialist
| Model ID | Notes |
|----------|-------|
| `qwen3-coder:480b` | Qwen3 Coder 480B |
| `qwen3-coder-next` | Qwen3 Coder Next |
| `devstral-2:123b` | Devstral 2 123B |
| `devstral-small-2:24b` | Devstral Small 2 24B |
| `gpt-oss:120b` | GPT-OSS 120B |
| `gpt-oss:20b` | GPT-OSS 20B |

### Small / Fast
| Model ID | Notes |
|----------|-------|
| `gemini-3-flash-preview` | Gemini 3 Flash Preview |
| `gemma4:31b` | Gemma 4 31B |
| `gemma3:27b` | Gemma 3 27B |
| `gemma3:12b` | Gemma 3 12B |
| `gemma3:4b` | Gemma 3 4B |
| `ministral-3:14b` | Ministral 3 14B |
| `ministral-3:8b` | Ministral 3 8B |
| `ministral-3:3b` | Ministral 3 3B |
| `rnj-1:8b` | RNJ-1 8B |

## Switching Models
```bash
# CLI
hermes model                    # interactive picker
hermes config set model.default minimax-m3   # direct

# Per-profile
trader config set model.default glm-5.2
coder config set model.default qwen3-coder:480b
news config set model.default deepseek-v3.2

# In Telegram chat
/model minimax-m3
```

## Verify Connection
```bash
# List models from API
curl -s "https://ollama.com/v1/models" \
  -H "Authorization: Bearer $OLLAMA_API_KEY" | python3 -m json.tool

# Quick test
hermes status | grep -E "Model|Provider"
```