# 📊 4-Agent Stock Analysis System

**Advanced Fundamental Analysis using Claude Opus 4 + Extended Thinking via OpenRouter API**

## Architecture

```
Scraper Agent (yfinance + stockanalysis.com)
    ↓
Analyst Agent (Valuation, Health, Profitability) [Extended Thinking]
    ↓
Sentiment Agent (News, Ratings, Catalysts)
    ↓
Recommender Agent (BUY/HOLD/SELL + Price Target) [Extended Thinking]
```

## 🚀 Quick Start

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Set OpenRouter API Key
```bash
# Edit .env file (created in repo root)
OPENROUTER_API_KEY=sk-or-your-api-key-here
```

Get free API key: https://openrouter.ai

### 3. Test Setup
```bash
python test_system.py
```

### 4. Run Analysis
```bash
python stock_analysis_system.py
```

## 🤖 Agents Explained

### Agent 1: Scraper (No Thinking)
- Fetches real stock data from yfinance
- Extracts: Price, P/E, EPS, ROE, Market Cap, Debt, Equity, etc.
- Fast (~5 seconds)

### Agent 2: Analyst (Extended Thinking ✨)
- Deep fundamental analysis
- Assesses: Valuation, Financial Health, Profitability, Growth
- Returns: Overvalued/Fair/Undervalued verdict + Risk factors
- Slow but accurate (~15-30 seconds with thinking)

### Agent 3: Sentiment (No Thinking)
- Analyzes recent news sentiment
- Gets analyst consensus from yfinance
- Identifies insider activity patterns
- Fast (~5 seconds)

### Agent 4: Recommender (Extended Thinking ✨)
- Synthesizes all data from 3 agents
- Generates: BUY/HOLD/SELL + Confidence + Price Target
- Detailed investment thesis + exit conditions
- Deepest reasoning (~30-45 seconds with thinking)

## 💡 Key Features

✅ **Extended Thinking** — AI uses "thinking tokens" for deeper analysis  
✅ **Real Data** — Not simulated, uses live yfinance + web scraping  
✅ **Professional Analysis** — Mimics 20+ year investor perspective  
✅ **Risk Assessment** — Identifies key risks and exit conditions  
✅ **Confidence Scoring** — 0-100% confidence on recommendation  

## 📊 What You Get

Each analysis includes:

```
✓ Current Price & Valuation
✓ P/E Analysis (Undervalued? Overvalued?)
✓ Financial Health Score
✓ ROE & Profitability Assessment
✓ Latest News & Sentiment
✓ Analyst Consensus
✓ Final Recommendation (BUY/HOLD/SELL)
✓ 12-Month Price Target
✓ Upside/Downside Potential
✓ Key Investment Reasons
✓ Risk Factors to Watch
✓ Exit Conditions
✓ AI Thinking Process (for transparency)
```

## 🎯 Customize Stocks

Edit the bottom of `stock_analysis_system.py`:

```python
stocks = ["GOOGL", "AAPL", "MSFT", "NVDA", "TSLA"]
for symbol in stocks[:1]:  # Change slice to analyze more
    orchestrator.analyze_stock(symbol)
```

## 📈 Cost Estimation (OpenRouter)

Per stock analysis:
- Scraper: ~$0.01
- Analyst (with thinking): ~$0.30
- Sentiment: ~$0.05
- Recommender (with thinking): ~$0.50
- **Total per stock: ~$0.86**

Extended thinking costs ~3-5x more but provides deeper reasoning.

## 🔧 Disable Extended Thinking (Faster, Cheaper)

Edit `stock_analysis_system.py`:

```python
# In AnalystAgent.__init__()
use_thinking=False  # Change from True

# In RecommenderAgent.__init__()
use_thinking=False  # Change from True
```

This reduces cost by ~60% but loses deep reasoning capability.

## ❌ Troubleshooting

### "OPENROUTER_API_KEY not set"
```bash
# PowerShell
$env:OPENROUTER_API_KEY = "sk-or-..."
python stock_analysis_system.py
```

### "API Error: 401 Unauthorized"
- Check API key is correct
- Ensure account has credits on openrouter.ai

### "yfinance data not available"
- Some small cap stocks may have incomplete data
- Try major stocks: GOOGL, AAPL, MSFT, NVDA, TSLA

### "Connection timeout"
- Increase timeout in `Agent.call()` (currently 120 seconds)
- Check internet connection

## 🎓 Next Steps

1. **Integration with OpenClaw** — Add UI layer for chat interface
2. **Portfolio Analysis** — Analyze multiple stocks simultaneously
3. **Alert System** — Notify when price targets are hit
4. **Historical Tracking** — Store recommendations and measure accuracy
5. **Custom Metrics** — Add industry-specific analysis (tech vs pharma)

---

**Built with:** Python + OpenRouter API + Claude Opus 4 + Extended Thinking  
**For:** Professional stock analysis without Bloomberg/FactSet cost
