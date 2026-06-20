# Thai Language Guidelines for Trading Agents

## Language Preferences by Context

### Trading Discussions (Thai Preferred)
- **Market Analysis**: Use natural Thai for all trading discussions
- **Price Discussions**: Thai with technical terms (XAU/USD, pip, support/resistance)
- **Risk Management**: Thai for risk discussions (ความเสี่ยง, การจัดการความเสี่ยง)
- **Strategy Planning**: Mix of Thai and English technical terms

**Examples:**
```
"วิเคราะห์ตลาดทองวันนี้"
"XAU/USD ทำกำไรอย่างไร"
"ความเสี่ยงในการเทรดคืออะไร"
"กลยุทธ์golden hourทำงานยังไง"
```

### Technical Discussions (English Preferred)
- **Code Development**: Primarily English with Thai comments
- **API Interactions**: English (API calls, database queries)
- **System Configuration**: English (config files, environment variables)
- **Debugging**: English with Thai explanations

**Examples:**
```python
# Code example with Thai comments
def calculate_risk(amount, risk_percent):
    """
    คำนวณความเสี่ยงตามเปอร์เซ็นต์ที่ตั้งไว้
    """
    risk_amount = amount * (risk_percent / 100)
    return risk_amount

# API calls in English
mt5.initialize()
mt5.symbol_select("XAUUSDc", True)
```

### Bot Commands (Thai for Trading, English for Technical)
- **Telegram Bot Commands**: Thai for trading interactions
- **System Commands**: English for technical operations
- **Configuration Commands**: Mix of both

**Examples:**
```
# Trading commands (Thai)
@GoldTraderBot: "วิเคราะห์ตลาด"
@GoldTraderBot: "ปรับพารามิเตอร์"
@GoldTraderBot: "หยุดเทรด"

# Technical commands (English)
@CodeAssistantBot: "debug this code"
@CodeAssistantBot: "optimize performance"
@CodeAssistantBot: "create alert system"
```

## Agent-Specific Language Guidelines

### Trader Agent Language
```markdown
# Language Style
- Primary: Thai for all trading discussions
- Secondary: English for technical terms and concepts
- Tone: Professional, analytical, risk-aware

# Key Phrase Examples
"วิเคราะห์ XAU/UD วันนี้"
"ความเสี่ยงปัจจุบันอยู่ที่ [percentage]"
"กลยุทธ์ที่เหมาะสมคือ [strategy]"
"ทองกำลังอยู่ใน [regime] mode"
"golden hour ของทองคือ [time_range]"

# Technical Terms (Keep in English)
- XAU/USD
- Support/Resistance
- RSI
- MACD
- Pips
- Lot size
- Margin
- Stop loss
- Take profit
```

### Coder Agent Language
```markdown
# Language Style
- Code: English (variables, functions, classes)
- Comments: Mix of Thai and English
- Explanations: Thai with technical terms
- Tone: Technical, precise, problem-solving

# Code Examples
```python
def calculate_gold_price_change(current_price, previous_price):
    """
    คำนวณการเปลี่ยนแปลงราคาทองคำ
    Returns percentage change
    """
    change = ((current_price - previous_price) / previous_price) * 100
    return change

# Thai comments explaining technical decisions
# ใช้ RSI 14 เพื่อตรวจสอบ overbought/oversold
# ตั้งค่า stop loss ที่ 2% จาก entry point
```

# Technical Explanations (Thai)
"เราใช้ algorithm นี้เพื่อ [purpose]"
"ปัญหานี้แก้ได้ด้วย [solution]"
"Code นี้ทำงานได้ดีเพราะ [reason]"
```

### News Agent Language
```markdown
# Language Style
- News Summary: Thai for local market context
- Technical Analysis: Thai with English terms
- Sentiment Analysis: Thai for market mood
- Tone: Objective, analytical, timely

# Key Phrase Examples
"ข่าว Fed ส่งผลต่อทองอย่างไร"
"Market sentiment เป็น [positive/negative]"
"การเปลี่ยนแปลงนโยบายเงินตรา"
"เหตุการณ์ geopolitical ที่ส่งผล"
"ข่าว mining company ล่าสุด"

# Technical Terms (Mix)
- Fed (กำหนดนโยบาย)
- Interest rates (อัตราดอกเบี้ย)
- Inflation (เงินเฟ้อ)
- Geopolitical (การเมืองระหว่างประเทศ)
- Mining company (บริษัทขุดทอง)
```

## Communication Patterns

### Main Chat to Agents
```markdown
# Central Manager Commands (Thai)
"Trader มาวิเคราะห์ทองให้"
"Coder มาแก้โค้ดนี้"
"News มาหาข่าว Fed ให้"

# Agent Responses (Mixed)
"Trader: รับครับ! วิเคราะห์ตลาด XAU/USD โดยใช้ RSI และ MACD"
"Coder: ได้เลยครับ! จะแก้ bug ใน code ด้วยการปรับ algorithm"
"News: ได้ครับ! ข่าว Fed ล่าสุดว่า [summary in Thai]"
```

### Agent Coordination
```markdown
# Cross-Agent Communication
"Trader: ต้องข้อมูลจาก News เกี่ยวกับ Fed policy"
"News: ส่งข่าว Fed ไปให้ Trader แล้วครับ"
"Coder: รอข้อมูลจาก Trader เพื่อปรับ trading bot"

# Response Coordination
"Trader: ข้อมูลแล้วครับ กำลังวิเคราะห์..."
"News: ข่าวเพิ่มเติมมาแล้วครับ"
"Coder: พร้อมเขียน code ตามกลยุทธ์ที่ Trader วิเคราะห์แล้ว"
```

## Technical Documentation Language

### Code Comments (Bilingual)
```markdown
# Python Code Example
def manage_trading_positions(symbol, entry_price, stop_loss):
    """
    จัดการตำแหน่งการเทรด (Thai explanation)
    Manage trading positions based on risk parameters (English summary)
    
    Args:
        symbol (str): Trading symbol (e.g., XAUUSDc)
        entry_price (float): Entry price for the position
        stop_loss (float): Stop loss percentage
    
    Returns:
        dict: Position management results
    """
    # คำนวณ risk management parameters (Thai comment)
    risk_amount = calculate_risk(entry_price, stop_loss)
    
    # Check if position meets risk criteria (English comment)
    if risk_amount > MAX_RISK:
        return close_position(symbol)
    
    # ปรับตำแหน่งตาม market conditions (Thai comment)
    adjusted_position = adjust_position_size(symbol, market_volatility)
    
    return adjusted_position
```

### Configuration Files (English with Thai Comments)
```yaml
# Trading configuration
trading:
  # การเทรดทองคำ (Gold trading settings)
  symbol: "XAUUSDc"
  lot_size: 0.01
  risk_percent: 2.0
  
  # Golden hours for trading (ช่วงเวลาทอง)
  golden_hours:
    start: "09:00"
    end: "17:00"
  
  # Strategy parameters (พารามิเตอร์กลยุทธ์)
  rsi_period: 14
  macd_fast: 12
  macd_slow: 26
  macd_signal: 9
```

### Error Messages (Bilingual)
```markdown
# Error Handling
try:
    # ทำการเทรด (Trading operation)
    result = execute_trade(symbol, volume, direction)
except APIError as e:
    # แสดง error ในภาษาไทย
    print(f"เกิดข้อผิดพลาดในการเทรด: {e}")
    # Show error in English for debugging
    print(f"Trade API error: {e}")
    # แจ้งเตือนให้ Trader agent ทราบ
    notify_trader_agent("เกิดข้อผิดพลาดในการเทรด")
```

## Common Phrases and Vocabulary

### Trading Vocabulary (Mix)
```markdown
# Thai Terms
- ทองคำ (Gold)
- ราคาปิด (Close price)
- ราคาเปิด (Open price)
- ซื้อ/ขาย (Buy/Sell)
- กำไร/ขาดทุน (Profit/Loss)
- ความเสี่ยง (Risk)
- การจัดการความเสี่ยง (Risk management)

# English Terms (Keep as-is)
- Support/Resistance
- RSI (Relative Strength Index)
- MACD (Moving Average Convergence Divergence)
- Pip
- Lot
- Margin
- Leverage
- Volatility
```

### Technical Vocabulary (English with Thai explanations)
```markdown
# Technical Terms
- API: ช่องทางการเชื่อมต่อระหว่างโปรแกรม (Application Programming Interface)
- Database: ฐานข้อมูลสำหรับเก็บข้อมูล (Storage system)
- Algorithm: วิธีการแก้ปัญหา (Problem-solving method)
- Backtesting: ทดสอบกลยุทธ์ด้วยข้อมูลย้อนหลัง (Historical testing)
- Optimization: การปรับปรุงให้ดีขึ้น (Improvement process)
```

## Quality Guidelines

### Language Consistency
- **Commands**: Always start with agent name in Thai
- **Responses**: Match the language context of the request
- **Technical terms**: Keep English terms standardized
- **Code**: Use English for all code elements

### Response Quality
- **Clarity**: Ensure meaning is clear in both languages
- **Accuracy**: Maintain technical accuracy while being language-appropriate
- **Completeness**: Provide all necessary information in appropriate language
- **Actionability**: Give clear, actionable instructions

### Cultural Appropriateness
- **Trading context**: Use appropriate Thai business terminology
- **Technical context**: Maintain international technical standards
- **Professional tone**: Keep professional across all interactions
- **Respect**: Show respect for both Thai and English contexts

## Implementation Examples

### Trading Bot Response Example
```
User: "Trader มาวิเคราะห์ทองให้"

Trader Agent Response:
"รับครับ! วิเคราะห์ตลาด XAU/USD โดยใช้:

📊 สถานะปัจจุบัน:
- ราคาปิด: $2,350.50
- RSI: 68.5 (ใกล้ overbought)
- MACD: สัญญาณเชิงบวก
- Volume: ปกติ

🎯 การวิเคราะห์:
- Market อยู่ในแนวโน้มขึ้น
- Golden hour: 09:00-17:00 (อยู่ในช่วงนี้)
- Risk level: ปกติ (2%)

💡 แนะนำ:
- ซื้อเปิดตำแหน่ง 0.01 lot
- Stop loss: 2,330 (ซื้อเกินไป)
- Take profit: 2,380 (ขายได้)

จะทำงานนี้ให้เสร็จหรือไม่ครับ?"
```

### Development Response Example
```
User: "Coder มาแก้ bug ใน trading bot"

Coder Agent Response:
"ได้เลยครับ! แก้ bug ใน trading bot:

🐛 ปัญหาที่พบ:
- Line 234: symbol_select failed
- Error: Symbol not found
- Cause: XAUUSDc ไม่มีใน symbol list

🛠️ แก้ไข:
1. ตรวจสอบ symbol name: XAUUSDc (ถูกต้อง)
2. เพิ่มการตรวจสอบ symbol ก่อน select
3. ใช้ error handling สำหรับกรณีนี้

📝 Code ที่แก้ไข:
```python
# เดิม
mt5.symbol_select("XAUUSDc", True)

# ใหม่
if mt5.symbol_info("XAUUSDc") is None:
    logger.error("Symbol XAUUSDc not found")
    return False
mt5.symbol_select("XAUUSDc", True)
```

ทดสอบแล้วใช้ได้ปกติครับ!"
```

### News Response Example
```
User: "News มาหาข่าว Fed ให้"

News Agent Response:
"ได้ครับ! ข่าว Fed ล่าสุด:

📰 ข่าวสำคัญ:
- Fed ตัดสินใจคง Fed rate ที่ 5.25-5.5%
- คำพูดของ Chair Powell: "inflation still above target"
- ตลาดตอบสนอง: USD อ่อนลง, gold ขึ้น

📊 ผลกระทบ:
- XAU/USD: ปรับตัวขึ้น $25 (+1.1%)
- Market sentiment: กลุ่มขาย (bearish)
- ทองคำ: คาดว่าจะยังคงแข็งแรง

💡 วิเคราะห์:
- Fed policy ยังคง hawkish
- ทองคำเป็น safe haven
- Golden hour: 09:00-17:00 ควรค้างตำแหน่ง

ส่งข่าวไปให้ Trader วิเคราะห์ต่อครับ!"
```