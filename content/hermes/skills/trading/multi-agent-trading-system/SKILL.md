---
name: multi-agent-trading-system
category: trading
description: |

  Deploy and manage a team of specialized AI agents for comprehensive trading operations.
  
  This skill enables creating multiple specialized agents (trader, coder, news) with distinct responsibilities,
  supporting both centralized management and direct agent commands via Telegram bots.
  
  Key capabilities:
  - Role-based agent architecture
  - Centralized command coordination
  - Direct agent communication via Telegram
  - Thai language preferences for trading contexts
  - GLM-5.2 model optimization
  - Profile isolation for security and focus

triggers:
  - "สร้าง agent ทีม"
  - "multi-agent setup"
  - "trading team"
  - "specialized agents"
  - "create trader coder news"
  - "deploy bot team"

steps:
  1. Create specialized profiles with distinct roles
  2. Configure model and authentication settings with proper PATH setup
  3. Set up Telegram bot integration with individual bot tokens
  4. Define command patterns and workflows with Thai language preferences
  5. Test agent communication and coordination via both central and direct modes
  6. Deploy gateways in background mode with individual logging
  7. Verify system health and troubleshoot connectivity issues

dependencies:
  - hermes-agent: For profile management
  - claw-trade-linux-migration: For trading bot integration
  - web: For market data access

workflow:
  - Central Manager Mode: Main chat coordinates all agents
  - Direct Mode: Telegram bots respond directly to commands
  - Hybrid Mode: Agents work collaboratively on complex tasks

examples:
  - "สร้างทีม agent สำหรับ trading"
  - "Deploy multi-agent trading system"
  - "Setup trader + coder + news agents"
---

# Multi-Agent Trading System

## Role
Deploy and manage a team of specialized AI agents for comprehensive trading operations.
## Core Architecture (Updated June 2026)

```
🧠 default (ผู้จัดการสูงสุด / Orchestrator)
├── Trader Agent (Gold Trading Focus)
├── Coder Agent (System Development)
├── News Agent (Market Intelligence)
└── System Agent (Backend Monitor — silent)
```

**Roles:**
- **default** = Central orchestrator. Receives all commands, delegates to sub-agents via `delegate_task` (max 3 concurrent), or sends via Telegram bot. User can also bypass default and talk directly to any agent's Telegram bot.
- **system** = Backend monitor (was named `daemon`). Checks all gateways, dashboards, VPS health. Renamed from `daemon` → `system` June 2026.
- **trader/coder/news** = Domain specialists. Run independently; user can command them directly.

**delegate_task limitations:** Sub-agents run synchronously within the current turn. They WILL be interrupted if:
- The user sends a new message before completion
- The model response takes too long (timeout ~200s)
- Any out-of-band event occurs
For reliable async work, use `cronjob` or terminal(background=true) instead of delegate_task for long-running tasks.

## Key Principles
- **Role Separation**: Each agent has specific expertise and responsibilities
- **Flexible Command**: Support both centralized coordination and direct commands
- **Thai Language**: Prioritize Thai for trading discussions while maintaining English technical terms
- **Silent Monitoring**: NO Telegram notifications. Alerts are passive log-only. User says "ลำคาญ" to constant pings.
- **OOM Protection**: Every agent gateway runs with systemd auto-restart. See Gateway Resilience section.

## Agent Profiles

### Trader Agent (`trader`)
- **Focus**: XAU/USD analysis, trading bot management, risk assessment
- **Language**: Thai for trading discussions, English for technical terms
- **Skills**: Market analysis, bot monitoring, technical analysis

### Coder Agent (`coder`)  
- **Focus**: System development, automation, debugging
- **Language**: Technical English with Thai explanations
- **Skills**: Coding, deployment, system administration

### News Agent (`news`)
- **Focus**: Market intelligence, news research, sentiment analysis
- **Language**: Thai for local market discussions
- **Skills**: Web research, data analysis, information curation

### System Agent (`system`)
- **Focus**: Backend monitoring, gateway health, VPS status
- **Language**: Technical only
- **Skills**: Health checks, log monitoring, silent alerting
- **History**: Renamed from `daemon` profile June 2026. Daemon profile deleted.

## Command Patterns

### Central Manager Mode (Main Chat)
```
"Trader มาวิเคราะห์ทองให้"
"Coder มาแก้โค้ดนี้"  
"News มาหาข่าว XAU/USD ให้"
```

### Direct Mode (Telegram Bots)
```
@GoldTraderBot: "วิเคราะห์ตลาดทอง"
@CodeAssistantBot: "แก้ bug ใน trading bot"
@NewsCollectorBot: "หาข่าว Fed ให้"
```

## Setup Workflow

### 1. Profile Creation
```bash
hermes profile create trader
hermes profile create coder  
hermes profile create news
```

### 2. Configuration
```bash
# Configure GLM-5.2 for all profiles
trader config set model.provider zai
trader config set model.default glm-5.2
trader config set gateway.auth bot

# Repeat for coder and news profiles
```

### 3. Personality Customization
Edit `SOUL.md` for each profile to set language preferences and role focus.

### 4. Telegram Bot Setup
- Create 3 bots via @BotFather
- Run setup script: `./setup_telegram_bots.sh`
- Start gateways: `trader gateway start`, `coder gateway start`, `news gateway start`

## Common Workflows

### Trading Analysis Workflow
1. Main chat: "Trader มาวิเคราะห์ทองให้"
2. Trader analyzes market conditions
3. News agent: "News มาหาข่าว Fed ให้" (if needed)
4. Coder agent: "Coder มาสร้าง alert system" (if automation needed)

### System Development Workflow  
1. Main chat: "Coder มาสร้าง trading bot"
2. Coder develops system
3. Trader: "Trader มาทดสอบ bot" 
4. News: "News มาตรวจสอบข่าว" (for data input)

## Thai Language Preferences
- **Trading discussions**: Use Thai naturally
- **Technical terms**: Keep in English (API, SQL, etc.)
- **Code comments**: Mix Thai and English for clarity
- **Bot responses**: Thai for trading, English for technical

## Agent Coordination
- **Main chat**: Coordinates complex multi-agent tasks
- **Individual chats**: Direct agent specialization
- **Hybrid mode**: Agents collaborate within their domains

## Monitoring & Maintenance (Updated with Gateway Resilience)

### Gateway Auto-Restart (3-layer protection)
```
Layer 1: systemd user service (default gateway)
├── Restart=always, RestartSec=1, OOMScoreAdjust=-500
└── Survives OOM kill events

Layer 2: systemd system service (watchdog for 4 agents)
├── hermes-gateway-watchdog.service
├── Load threshold >8 skips cycle to prevent thrashing
└── 5-min cooldown prevents restart loops

Layer 3: Instant auto-restart daemon (no cooldown)
├── hermes-auto-restart.service
├── Checks every 10 seconds
├── Zero cooldown — instant restart on crash
└── MemoryMax=100M, CPUQuota=10% (very lightweight)
```

### Deployment Commands
```bash
# Start all 5 agents
systemctl --user start hermes-gateway.service  # default
hermes --profile trader gateway run            # or via watchdog

# Check all statuses at once
for p in default trader coder news system; do
  if [ "$p" = "default" ]; then
    pid=$(pgrep -f "hermes_cli.main gateway run" | grep -v "profile" | head -1)
  else
    pid=$(pgrep -f "hermes.*profile $p.*gateway" | head -1)
  fi
  [ -n "$pid" ] && echo "✅ $p ($pid)" || echo "❌ $p"
done

# Restart everything cleanly
sudo systemctl restart hermes-gateway-watchdog.service
sudo systemctl restart hermes-auto-restart.service
systemctl --user restart hermes-gateway.service
```

### Health Check
- Check gateway status: `trader gateway status`
- Monitor agent performance and response times
- Update skills and configurations as needed
- Backup profile configurations regularly
- **Watch load average**: if >20, check for mining malware in docker containers immediately

## Pitfalls to Avoid
- ❌ Don't mix agent responsibilities beyond their expertise
- ❌ Avoid technical jargon in Thai trading discussions  
- ❌ Don't overload agents with unrelated tasks
- ❌ Remember to test Telegram bot connectivity
- ❌ Maintain consistent model configurations across profiles
- ❌ Use tokens from @BotFather only (critical!)
- ❌ Ignore deprecated .env settings - must move to config.yaml
- ❌ Forget to set user permissions (TELEGRAM_ALLOWED_USERS)
- ❌ Try to restart gateways from inside running processes

## Quick Reference & Scripts
- [Health Check Script](scripts/health_check.sh) - Quick diagnostic for all agents
- [Complete Setup Script](scripts/setup_complete.sh) - Automated deployment of entire system
- [Troubleshooting Guide](references/troubleshooting-guide.md) - Detailed error solutions

## Key Patterns Discovered
- **Profile Isolation**: Each agent needs dedicated configuration with proper PATH setup (`export PATH="$HOME/.local/bin:$PATH"`)
- **Gateway Configuration**: Use `profile gateway install` then `nohup profile gateway run > /tmp/profile-gateway.log 2>&1 &` for background operation
- **Telegram Bot Integration**: Bot tokens must be set individually per profile using `profile config set TELEGRAM_BOT_TOKEN 'token'`
- **GLM API Management**: Shared API key across profiles via `profile config set GLM_API_KEY 'key'`
- **Command Patterns**: Thai commands for trading, mixed language for technical tasks, direct agent invocation pattern "[Agent] มา [Task]"

## Critical Lessons Learned
- **TOKEN VALIDATION CRITICAL**: Tokens MUST come from @BotFather. Always validate with `curl -s "https://api.telegram.org/botTOKEN/getMe" | grep -o '"ok":true'` before setting in config
- **DEPRECATED ENV SETTINGS**: Remove `TERMINAL_CWD` from .env files and set with `profile config set terminal.cwd /root`
- **USER PERMISSIONS**: Set `TELEGRAM_ALLOWED_USERS '8675116758'` for access control
- **GATEWAY RESTART**: Never restart from inside - use `pkill -f "gateway"` first, then restart
- **ERROR PATTERNS**: "Token rejected by server" means recreate via @BotFather immediately

## Lessons Learned
- Gateway installation handles interactive prompts automatically when run correctly
- Always verify gateway status with `profile gateway status` after setup
- Profile PATH must be set before running profile commands
- Bot tokens are required before Telegram integration will work
- Separate log files per agent help with debugging and monitoring