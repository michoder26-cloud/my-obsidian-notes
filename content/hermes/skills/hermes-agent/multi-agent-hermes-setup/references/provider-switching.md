# Provider Switching — Complete Workflow

## Problem
When switching model providers (e.g. Z.AI → Ollama Cloud), old API keys
remain in `.env` files and cause 401 authentication errors. The gateway
tries to use the old provider's credentials, fails, and bots stop responding.

## Symptoms
- Gateway logs show: `HTTP 401: token expired or incorrect`
- Bots connected to Telegram but don't respond to messages
- `ps aux` shows gateway running, but logs show auth errors

## Fix: Switch All Profiles to New Provider

### 1. Update .env for EACH profile (remove old keys, add new)

```bash
# For each profile: trader, coder, news
profile="trader"  # repeat for each

env_file="/root/.hermes/profiles/$profile/.env"

# Remove old provider keys
sed -i '/^GLM_API_KEY=/d' "$env_file"

# Add new provider key (use Python if key contains special chars)
echo "OLLAMA_API_KEY=your_new_key_here" >> "$env_file"

# Verify
cat "$env_file"
```

### 2. Update config.yaml for EACH profile

```bash
# Set provider, model, and base_url
hermes --profile trader config set model.provider ollama-cloud
hermes --profile trader config set model.default glm-5.2
hermes --profile trader config set model.base_url "https://ollama.com/v1"
# Repeat for coder, news
```

### 3. Kill old gateways and restart

```bash
# Kill ALL gateways
pkill -f "gateway run"
sleep 2

# Restart each profile's gateway
# Use terminal(background=true) for each one
```

### 4. Verify

```bash
# Check processes running
ps aux | grep "gateway run" | grep -v grep

# Check logs for auth errors
tail -30 /root/.hermes/profiles/trader/logs/gateway.log
# Look for: "✓ telegram connected" (good)
# Look for: "HTTP 401" (bad — key still wrong)
```

## Key Lessons

1. **Check processes FIRST** — before giving any advice, run `ps aux | grep "gateway run" | grep -v grep`
2. **Old keys cause 401** — always remove old provider keys from .env when switching
3. **Use ACTIONS not ADVICE** — when user reports bots not responding, check processes, read logs, fix .env, restart gateways — all within the same turn
4. **All profiles need updating** — don't just update one profile, update ALL of them
5. **GLM-5.2 has no vision** — when using GLM-5.2, vision_analyze will fail with "this model does not support image input". Use a different model for vision tasks or inform user.