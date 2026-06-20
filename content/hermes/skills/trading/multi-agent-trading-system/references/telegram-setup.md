# Multi-Agent Telegram Setup Guide

## Prerequisites
- Hermes Agent v0.16.0+ installed
- Telegram BotFather access (@BotFather)
- API keys for all profiles (GLM-5.2)

## Step-by-Step Setup

### 1. Create Profiles
```bash
# Create three specialized profiles
hermes profile create trader
hermes profile create coder
hermes profile create news
```

### 2. Configure Profiles
```bash
# Set PATH for profile access
export PATH="$HOME/.local/bin:$PATH"

# Configure GLM-5.2 for all profiles
trader config set model.provider zai
trader config set model.default glm-5.2
trader config set gateway.auth bot

coder config set model.provider zai
coder config set model.default glm-5.2
coder config set gateway.auth bot

news config set model.provider zai
news config set model.default glm-5.2
news config set gateway.auth bot

# Set API keys (using shared Z.AI key)
trader config set GLM_API_KEY 6e2e14...nyFk
coder config set GLM_API_KEY 6e2e14...nyFk
news config set GLM_API_KEY 6e2e14...nyFk
```

### 3. Customize Personalities
Edit SOUL.md files for each profile with role-specific personality settings.

### 4. Create Telegram Bots
In Telegram:
1. Contact @BotFather
2. Create bots:
   ```
   /newbot → GoldTraderBot
   /newbot → CodeAssistantBot
   /newbot → NewsCollectorBot
   ```
3. Copy bot tokens and store securely

### 5. Set Up Gateway Services
```bash
# Install gateways (handles interactive prompts automatically)
trader gateway install
coder gateway install
news gateway install

# Start gateways in background
nohup trader gateway run > /tmp/trader-gateway.log 2>&1 &
nohup coder gateway run > /tmp/coder-gateway.log 2>&1 &
nohup news gateway run > /tmp/news-gateway.log 2>&1 &
```

### 6. Configure Bot Tokens
```bash
# Set Telegram bot tokens for each profile
trader config set TELEGRAM_BOT_TOKEN 'your_gold_bot_token'
coder config set TELEGRAM_BOT_TOKEN 'your_code_bot_token'
news config set TELEGRAM_BOT_TOKEN 'your_news_bot_token'
```

### 7. Verify Status
```bash
# Check gateway status
trader gateway status
coder gateway status
news gateway status

# Monitor logs
tail -f /tmp/trader-gateway.log
tail -f /tmp/coder-gateway.log
tail -f /tmp/news-gateway.log
```

## Common Issues and Solutions

### Gateway Not Starting
- **Issue**: Gateway service not installed
- **Fix**: Run `trader gateway install` again
- **Check**: Ensure PATH is set: `export PATH="$HOME/.local/bin:$PATH"`

### GLM API Key Not Found
- **Issue**: API key not configured in profile
- **Fix**: Set via `trader config set GLM_API_KEY <your_key>`
- **Verify**: Check with `trader status | grep GLM`

### Telegram Bot Not Responding
- **Issue**: Bot token not set, incorrect, or rejected by server
- **Fix**: Verify token comes from @BotFather and test with `curl -s "https://api.telegram.org/botTOKEN/getMe" | grep -o '"ok":true'`
- **Debug**: Check gateway logs for "InvalidToken" or "rejected by server" errors
- **Critical**: Tokens MUST come from @BotFather, not generated elsewhere

### Token Rejected by Server (CRITICAL)
- **Issue**: `ERROR gateway.platforms.telegram: The token '...' was rejected by the server`
- **Root Cause**: Token not from @BotFather, bot deleted, or token expired
- **Fix**: Recreate bots via @BotFather and use new tokens only
- **Prevention**: Always validate tokens with API test before setting in config

### Deprecated .env Settings
- **Issue**: `⚠ Deprecated .env settings detected: TERMINAL_CWD=/root found in .env`
- **Fix**: Remove deprecated entries and set in config.yaml:
  ```bash
  trader config set terminal.cwd /root
  coder config set terminal.cwd /root
  news config set terminal.cwd /root
  ```
  Then remove `TERMINAL_CWD` from .env files

### User Permission Issues
- **Issue**: Unauthorized users denied access
- **Fix**: Set allowed users in profile config:
  ```bash
  trader config set TELEGRAM_ALLOWED_USERS '8675116758'
  coder config set TELEGRAM_ALLOWED_USERS '8675116758'
  news config set TELEGRAM_ALLOWED_USERS '8675116758'
  ```

### Gateway Restart Problems
- **Issue**: Refusing to restart from inside gateway process
- **Fix**: Use external shell: `pkill -f "gateway"` then restart
- **Background**: Use `terminal(background=true)` for proper process tracking

### Profile PATH Issues
- **Issue**: Profile commands not found
- **Fix**: Always export PATH before profile commands
- **Pattern**: `export PATH="$HOME/.local/bin:$PATH" && <profile> <command>`

## Automation Scripts

### Setup Script
```bash
#!/bin/bash
# setup_multiagent_telegram.sh
export PATH="$HOME/.local/bin:$PATH"

# Create profiles and configure
hermes profile create trader
hermes profile create coder  
hermes profile create news

# Configure all profiles
for profile in trader coder news; do
    $profile config set model.provider zai
    $profile config set model.default glm-5.2
    $profile config set gateway.auth bot
    $profile config set GLM_API_KEY 6e2e14...nyFk
done

# Install gateways
for profile in trader coder news; do
    $profile gateway install
done
```

### Health Check Script
```bash
#!/bin/bash
# check_agents.sh
export PATH="$HOME/.local/bin:$PATH"

echo "=== Multi-Agent Health Check ==="
for profile in trader coder news; do
    echo "📊 $profile Status:"
    if $profile gateway status > /dev/null 2>&1; then
        echo "   ✅ Gateway running"
        echo "   📋 Log: /tmp/${profile}-gateway.log"
    else
        echo "   ❌ Gateway not running"
    fi
done
```

## Testing the System

### Test Individual Agents
1. Send messages to each bot in Telegram:
   - @GoldTraderBot: "วิเคราะห์ตลาดทอง"
   - @CodeAssistantBot: "แก้ bug ใน trading bot"
   - @NewsCollectorBot: "หาข่าว Fed ให้"

### Test Coordination
From main chat:
```
Trader มาวิเคราะห์ทองให้
Coder มาแก้โค้ดนี้
News มาหาข่าว XAU/USD ให้
```

## Maintenance

### Restart Gateways
```bash
# Stop all gateways
pkill -f 'gateway'

# Start all gateways
nohup trader gateway run > /tmp/trader.log 2>&1 &
nohup coder gateway run > /tmp/coder.log 2>&1 &
nohup news gateway run > /tmp/news.log 2>&1 &
```

### Update Profiles
```bash
# Update model or configuration
trader config set model.default glm-5.2-updated
coder config set gateway.auth bot
news config set model.provider zai
```

### Backup Configurations
```bash
# Copy profile configurations
cp -r ~/.hermes/profiles/trader /backup/trader-profile-$(date +%Y%m%d)
cp -r ~/.hermes/profiles/coder /backup/coder-profile-$(date +%Y%m%d)
cp -r ~/.hermes/profiles/news /backup/news-profile-$(date +%Y%m%d)
```

## Security Considerations

- Store bot tokens securely (consider environment variables)
- Use different API keys for different profiles if possible
- Monitor access logs for suspicious activity
- Regularly rotate bot tokens and API keys
- Keep profile configurations backed up