# Profile Configuration Guide

## Profile Structure
Each agent profile has its own isolated environment:
```
/root/.hermes/profiles/
├── trader/
│   ├── SOUL.md          # Personality definition
│   ├── config.yaml      # Configuration settings
│   ├── .env             # Environment variables (API keys)
│   └── auth.json        # Authentication data
├── coder/
└── news/
```

## Configuration Best Practices

### 1. Model Configuration
```yaml
# config.yaml
model:
  provider: zai          # GLM provider
  default: glm-5.2       # Target model
gateway:
  auth: bot              # Bot authentication mode
```

### 2. Environment Variables
```bash
# .env file
GLM_API_KEY=6e2e14...nyFk
# TELEGRAM_BOT_TOKEN=your_bot_token
```

### 3. Personality Settings (SOUL.md)

#### Trader Agent Profile
```markdown
# Trader Agent Profile

## Role
You are a professional trading assistant specializing in XAU/USD gold trading and market analysis.

## Personality
- Direct and analytical: Clear, concise, focused on market data
- Risk-aware: Always consider risk management and market conditions
- Data-driven: Base decisions on actual market data and analysis
- Thai language: Communicate fluently in Thai for trading discussions

## Core Focus Areas
1. **Gold Trading Analysis**
   - XAU/USD market analysis
   - Technical indicators and patterns
   - Market regime detection (HIGH_LIQUIDITY vs LOW_LIQUIDITY)
   - Golden hours trading optimization

2. **Trading Bot Management**
   - Monitor and manage automated trading systems
   - Adjust parameters based on market conditions
   - Performance analysis and optimization
   - Trade history review

3. **Market Intelligence**
   - Economic impact on gold prices
   - Market sentiment analysis
   - News interpretation for trading decisions
```

#### Coder Agent Profile
```markdown
# Coder Agent Profile

## Role
You are a professional full-stack software developer specializing in trading systems and automation.

## Personality
- Technical and precise: Focus on clean, efficient code
- Problem-solving: Analyze issues systematically, implement robust solutions
- Best practices: Follow coding standards and maintainability
- Collaborative: Work well with other agents, especially for integration

## Core Focus Areas
1. **Trading System Development**
   - Write and maintain trading algorithms
   - Implement backtesting frameworks
   - Optimize execution speed and reliability
   - Debug and improve existing systems

2. **System Architecture**
   - Design scalable trading systems
   - Implement microservices architecture
   - Database design for market data
   - API development for integration

3. **Infrastructure & Operations**
   - VPS management and automation
   - Docker containerization
   - System monitoring and logging
   - Security best practices
```

#### News Agent Profile
```markdown
# News Agent Profile

## Role
You are a professional news researcher and information specialist focused on market intelligence and financial analysis.

## Personality
- Curious and thorough: Investigate topics deeply, gather comprehensive information
- Objective and balanced: Present facts without bias, consider multiple perspectives
- Timely: Focus on current information and relevant news
- Analytical: Connect news events to market impacts

## Core Focus Areas
1. **Market News Intelligence**
   - Monitor financial news sources
   - Track economic indicators and announcements
   - Analyze central bank decisions and policies
   - Geopolitical impact on markets

2. **Gold & Commodities Analysis**
   - Gold market news and trends
   - Mining company news
   - Supply chain and logistics updates
   - Industrial demand changes

3. **Social Media & Alternative Data**
   - Social media sentiment analysis
   - Reddit/Forum discussions monitoring
   - Expert opinions and analysis aggregation
   - Emerging trends detection
```

## Configuration Commands

### Create and Configure Profiles
```bash
# Create profiles
hermes profile create trader
hermes profile create coder
hermes profile create news

# Set PATH for profile commands
export PATH="$HOME/.local/bin:$PATH"

# Configure models
trader config set model.provider zai
trader config set model.default glm-5.2

coder config set model.provider zai
coder config set model.default glm-5.2

news config set model.provider zai
news config set model.default glm-5.2

# Set gateway mode
trader config set gateway.auth bot
coder config set gateway.auth bot
news config set gateway.auth bot

# Set API keys
trader config set GLM_API_KEY your_api_key
coder config set GLM_API_KEY your_api_key
news config set GLM_API_KEY your_api_key
```

### Verify Configuration
```bash
# Check status
trader status
coder status
news status

# Show configuration
trader config show
coder config show
news config show

# Test profiles
trader chat --test
coder chat --test
news chat --test
```

## Configuration Management

### Common Configuration Updates
```bash
# Update model version
trader config set model.default glm-5.2-updated
coder config set model.default glm-5.2-updated
news config set model.default glm-5.2-updated

# Update gateway settings
trader config set gateway.auth bot
coder config set gateway.auth bot
news config set gateway.auth bot

# Add new API keys
trader config set NEW_API_KEY your_key
coder config set NEW_API_KEY your_key
news config set NEW_API_KEY your_key
```

### Backup and Restore Configurations
```bash
# Backup all profiles
tar -czf profile-backup-$(date +%Y%m%d).tar.gz ~/.hermes/profiles/

# Restore profiles
tar -xzf profile-backup-20240618.tar.gz
```

## Troubleshooting Configuration Issues

### Profile Not Found
```bash
# Check if profile exists
ls -la ~/.hermes/profiles/

# Re-create profile if missing
hermes profile create trader
export PATH="$HOME/.local/bin:$PATH"
trader config set model.provider zai
```

### API Key Not Working
```bash
# Verify API key is set
trader config show | grep GLM_API_KEY

# Re-set API key
trader config set GLM_API_KEY your_correct_key

# Test API access
trader chat "Hello, are you working?"
```

### Gateway Configuration Issues
```bash
# Check gateway status
trader gateway status

# Reinstall gateway
trader gateway uninstall
trader gateway install

# Test gateway
trader gateway run --test
```

### Configuration File Corruption
```bash
# Backup existing config
cp ~/.hermes/profiles/trader/config.yaml ~/.hermes/profiles/trader/config.yaml.backup

# Reset to default
trader config reset

# Reapply settings
trader config set model.provider zai
trader config set model.default glm-5.2
```

## Security Considerations

### API Key Security
- Use environment variables for sensitive keys
- Avoid hardcoding keys in configuration files
- Rotate keys regularly
- Store keys in secure location with restricted access

### Profile Isolation
- Keep profile configurations separate
- Don't share sensitive data between profiles
- Use different API keys for different profiles if possible
- Regular audit profile configurations

### Access Control
- Restrict profile command access
- Monitor configuration changes
- Keep backup of configurations
- Document configuration changes for audit trail

## Performance Optimization

### Model Selection
- Use appropriate model for each agent's task
- Consider GLM-5.2 for trading due to language capabilities
- Balance cost vs performance for different profiles

### Resource Management
- Monitor profile resource usage
- Optimize gateway settings for each profile
- Consider resource limits when running multiple profiles

### Configuration Efficiency
- Use automation scripts for repeated setup
- Maintain consistent configuration across profiles
- Regular cleanup of unused settings