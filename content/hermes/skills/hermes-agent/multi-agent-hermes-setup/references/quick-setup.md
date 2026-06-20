# Multi-Agent Quick Reference

## Essential Commands

### Environment Setup
```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Profile Creation
```bash
hermes profile create trader
hermes profile create coder
hermes profile create news
```

### Model Configuration
```bash
trader config set GLM_API_KEY "6e2e14...nyFk"
trader config set model.provider zai
trader config set model.default glm-5.2
```

### Telegram Bot Configuration

> ⚠️ **CRITICAL:** Do NOT pass Telegram tokens directly in shell commands!
> The system censors `\d+:...` patterns. See `token-censor-workaround.md`.

```bash
# Configure gateway auth mode
trader config set gateway.auth bot

# Set allowed users (your Telegram user ID, get it from @userinfobot)
trader config set TELEGRAM_ALLOWED_USERS '8675116758'
# OR allow all users (less secure):
export GATEWAY_ALLOW_ALL_USERS=true
```

### Gateway Management
```bash
# Start (use terminal background mode)
terminal(background=true, notify_on_complete=true) trader gateway run

# Check status
trader gateway status

# Stop (kill all processes)
pkill -f "trader gateway"

# List all running gateways
trader gateway list
```

## Bot Token Testing
```bash
# Read token from env and test
export $(grep TELEGRAM_BOT_TOKEN /root/.hermes/profiles/trader/.env)
curl -s "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getMe" | grep '"ok":true'
```

## Common Patterns

### Starting Gateways
```bash
# CORRECT: Using Hermes background tool
terminal(background=true, notify_on_complete=true) trader gateway run

# Troubleshooting: If things hang, kill and restart
pkill -f "trader gateway"
```

### Path Setup
```bash
# Essential before any profile commands
export PATH="$HOME/.local/bin:$PATH"
```

### Verification
```bash
# Check all gateways
trader gateway list

# Individual status
trader gateway status
coder gateway status
news gateway status
```

## Troubleshooting

### "No messaging platforms enabled"
Set TELEGRAM_ALLOWED_USERS or GATEWAY_ALLOW_ALL_USERS=true

### "Token was rejected by the server"
The .env file has the wrong token. Re-write using Python (see token-censor-workaround.md)

### "Gateway already running"
Kill first: `pkill -f "profile gateway"`
Then restart.

### Deprecated TERMINAL_CWD warning
Move terminal.cwd from .env to config.yaml:
```bash
profile config set terminal.cwd /root
```
Then remove TERMINAL_CWD from .env:
```bash
sed -i '/TERMINAL_CWD/d' /root/.hermes/profiles/profile/.env
```