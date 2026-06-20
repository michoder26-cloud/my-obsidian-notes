# 🚀 Setup Guide (5 Minutes)

## Step 1: Install Dependencies
```powershell
pip install -r requirements.txt
```

## Step 2: Get API Key
1. Go to https://openrouter.ai
2. Click "Sign Up" (free)
3. Copy API key from Settings

## Step 3: Configure .env File
Open `.env` file in this directory and replace:
```
OPENROUTER_API_KEY=sk-or-your-key-here
```

With your actual key:
```
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxx
```

**⚠️ Don't commit .env file to git** (already in .gitignore)

## Step 4: Test System
```powershell
python test_system.py
```

Should show:
```
✓ API Key found!
✓ requests installed
✓ yfinance installed
✓ beautifulsoup4 installed
✓ API connection successful!
✓ yfinance working!
✓ System is ready!
```

## Step 5: Run Analysis
```powershell
python stock_analysis_system.py
```

This will analyze GOOGL (you can change stocks in the code)

---

## 📊 What to Expect

Running `stock_analysis_system.py` will:
1. Fetch real stock data (~5 sec)
2. Analyze fundamentals with AI thinking (~15 sec)
3. Get news & sentiment (~5 sec)  
4. Generate recommendation with AI thinking (~30 sec)

**Total time: ~1 minute per stock**

Output includes:
- ✓ Current price & valuation
- ✓ P/E analysis (over/undervalued?)
- ✓ Financial health score
- ✓ News sentiment
- ✓ BUY/HOLD/SELL recommendation
- ✓ 12-month price target
- ✓ Investment thesis
- ✓ Risk factors
- ✓ AI thinking process (for transparency)

---

## ❓ Troubleshooting

**Error: "OPENROUTER_API_KEY not set"**
- Check .env file exists
- Verify API key is pasted correctly
- No quotes needed around the key

**Error: "401 Unauthorized"**
- API key is wrong
- Check openrouter.ai has credits
- Try creating new key

**Error: "Connection timeout"**
- Check internet connection
- Try again (API might be slow)

**Missing package errors**
- Run `pip install -r requirements.txt` again
- Check Python version >= 3.8

---

## 💡 Next Steps

1. **Change stocks**: Edit line at bottom of `stock_analysis_system.py`
   ```python
   stocks = ["GOOGL", "AAPL", "MSFT", "NVDA"]
   ```

2. **Disable expensive thinking** (faster & cheaper):
   Edit both Agent init functions, change `use_thinking=True` to `use_thinking=False`

3. **Analyze multiple stocks**: Change slice in for loop
   ```python
   for symbol in stocks[:5]:  # Analyze first 5 instead of 1
   ```

4. **Save results**: Capture JSON output to file for later review

---

Read `STOCK_ANALYSIS_README.md` for full documentation.
