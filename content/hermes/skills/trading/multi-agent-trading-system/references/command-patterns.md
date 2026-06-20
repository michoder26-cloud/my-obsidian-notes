# Command Patterns and Workflow Examples

## Command Structure

### Central Manager Mode (Main Chat)
Commands originate from a central management chat and are distributed to specialized agents.

#### Trading Analysis Commands
```
"Trader มาวิเคราะห์ทองให้"
  → Trader agent analyzes XAU/USD market
  → May request News agent for market intelligence
  → May request Coder agent for automation

"Trader ตรวจบอท"
  → Trader agent reviews trading bot performance
  → Analyzes recent trades and strategy effectiveness
  → Provides optimization recommendations

"Trader วิเคราะห์ความเสี่ยง"
  → Risk assessment for current positions
  → Market regime analysis
  → Risk management recommendations
```

#### Development Commands
```
"Coder มาแก้โค้ดนี้"
  → Coder agent analyzes code issues
  → Provides debugging solutions
  → Suggests code improvements

"Coder สร้างระบบใหม่"
  → Develops new trading systems
  → Implements automation scripts
  → Sets up infrastructure

"Coder ปรับปรุง performance"
  → Optimizes system performance
  → Implements best practices
  → Improves code maintainability
```

#### Research Commands
```
"News มาหาข่าว XAU/USD ให้"
  → Researches gold market news
  → Analyzes economic indicators
  → Provides market intelligence

"News วิเคราะห์ sentiment"
  → Analyzes market sentiment
  → Tracks social media trends
  → Provides sentiment analysis

"News ติดตาม Fed"
  → Monitors Federal Reserve announcements
  → Analyzes monetary policy impacts
  → Provides economic analysis
```

### Direct Agent Mode (Telegram Bots)
Individual Telegram bots respond directly to specialized commands.

#### Gold Trader Bot (@GoldTraderBot)
```
"วิเคราะห์ตลาดทอง"
  → Technical analysis of XAU/USD
  → Current market conditions
  → Trading recommendations

"ดูบอททำงานอย่างไร"
  → Trading bot status report
  → Recent trade history
  → Performance metrics

"ปรับพารามิเตอร์"
  → Strategy parameter optimization
  → Risk settings adjustment
  → Bot configuration updates

"หยุดเทรด"
  → Temporarily halt trading
  → Close current positions
  → Emergency stop functionality

"เริ่มเทรดใหม่"
  → Resume trading operations
  → Reinitialize bot settings
  → Start new trading session
```

#### Code Assistant Bot (@CodeAssistantBot)
```
"แก้ bug ใน trading bot"
  → Debug existing trading code
  → Fix runtime errors
  → Improve error handling

"สร้าง alert system"
  → Implement notification system
  → Create custom alerts
  → Set up monitoring

"เขียน script ใหม่"
  → Develop automation scripts
  → Create utility functions
  → Implement new features

"optimization โค้ด"
  → Code performance optimization
  → Memory usage improvement
  → Execution speed enhancement
```

#### News Collector Bot (@NewsCollectorBot)
```
"หาข่าว Fed ให้"
  → Federal Reserve news monitoring
  → Monetary policy analysis
  → Economic impact assessment

"ติดตามทอง"
  → Gold market news aggregation
  → Mining company updates
  → Commodity price tracking

"วิเคราะห์ sentiment"
  → Market sentiment analysis
  → Social media monitoring
  → News impact assessment

"สรุปข่าว"
  → Daily news digest
  → Key highlights summary
  → Actionable insights
```

## Complex Multi-Agent Workflows

### Trading Strategy Development Workflow
```
1. Main Chat: "Coder มาสร้าง strategy ใหม่"
   ↓
2. Coder Agent: Develops trading algorithm
   ↓
3. Main Chat: "Trader มาทดสอบ strategy"
   ↓
4. Trader Agent: Backtests and validates strategy
   ↓
5. Main Chat: "News มาตรวจสอบ market conditions"
   ↓
6. News Agent: Provides current market context
   ↓
7. Main Chat: "Coder มาimplement strategy"
   ↓
8. Coder Agent: Deploys optimized strategy
```

### Market Analysis Workflow
```
1. Main Chat: "Trader มาวิเคราะห์ทองให้"
   ↓
2. Trader Agent: Technical analysis
   ↓
3. Main Chat: "News มาหาข่าว Fed ให้"
   ↓
4. News Agent: Economic news analysis
   ↓
5. Main Chat: "Trader ปรับกลยุทธ์"
   ↓
6. Trader Agent: Updates trading strategy
   ↓
7. Main Chat: "Coder มาทำ alert"
   ↓
8. Coder Agent: Implements notifications
```

### System Maintenance Workflow
```
1. Main Chat: "Coder มาตรวจสอบระบบ"
   ↓
2. Coder Agent: System diagnostics
   ↓
3. Main Chat: "Trader มาตรวจสอบ performance"
   ↓
4. Trader Agent: Performance analysis
   ↓
5. Main Chat: "Coder มาแก้ปัญหา"
   ↓
6. Coder Agent: Implements fixes
   ↓
7. Main Chat: "Trader มาทดสอบใหม่"
   ↓
8. Trader Agent: Validation testing
```

## Command Syntax Patterns

### Thai Command Patterns
- **Agent Invocation**: `[Agent] มา [Task]`
- **Request**: `[Agent] [Task] ให้`
- **Question**: `[Agent] [Question]`
- **Command**: `[Agent] [Action]`

### English Command Patterns (for technical tasks)
- **Request**: `[Agent] please [task]`
- **Command**: `[Agent] [action]`
- **Question**: `[Agent] can you [task]?`

### Code-Related Commands
- **Debug**: "Coder มาแก้โค้ดนี้"
- **Development**: "Coder มาสร้าง [feature]"
- **Optimization**: "Coder มาปรับปรุง [aspect]"
- **Maintenance**: "Coder มาตรวจสอบระบบ"

### Trading-Related Commands
- **Analysis**: "Trader มาวิเคราะห์ตลาด"
- **Management**: "Trader มาตรวจบอท"
- **Strategy**: "Trader มาปรับกลยุทธ์"
- **Risk**: "Trader มาตรวจสอบความเสี่ยง"

### News-Related Commands
- **Research**: "News มาหาข่าว [topic]"
- **Analysis**: "News มาวิเคราะห์ sentiment"
- **Monitoring**: "News มาติดตาม [topic]"
- **Summary**: "News มาสรุปข่าว"

## Response Templates

### Agent Acknowledgments
```
"[Agent] รับเรื่องครับ/ค่ะ มาทำงานตามคำสั่ง"
"[Agent] เข้าใจครับ/ค่ะ จะทำงานนี้ให้เสร็จ"
"[Agent] รออยู่ครับ/ค่ะ พร้อมทำงานตามคำสั่ง"
```

### Task Completion Messages
```
"งานนี้เสร็จแล้วครับ/ค่ะ [Agent]"
"ทำตามคำสั่งเสร็จเรียบร้อยครับ/ค่ะ"
"[Agent] ทำงานเสร็จแล้ว สามารถตรวจสอบได้"
```

### Coordination Messages
```
"[Agent] ต้องการข้อมูลจาก [Other Agent] เพื่อทำงานต่อ"
"[Agent] ส่งผลงานไปให้ [Other Agent] ต่อไป"
"[Agent] ร่วมกันทำงานนี้เสร็จแล้วครับ/ค่ะ"
```

## Error Handling Patterns

### When Agent Cannot Complete Task
```
"[Agent] ไม่สามารถทำงานนี้ได้ เนื่องจาก [reason]"
"[Agent] ต้องการข้อมูลเพิ่มเติมจาก [source]"
"[Agent] ขอข้อมูลเพิ่มเติมเพื่อทำงานต่อ"
```

### When Agent Needs Help
```
"[Agent] ต้องการความช่วยเหลือจาก [Other Agent]"
"[Agent] ขอความช่วยเหลือในการ [task]"
"[Agent] ส่งงานไปให้ [Other Agent] ต่อ"
```

### When Agent Completes Partially
```
"[Agent] ทำงานนี้บางส่วนเสร็จแล้ว ยังต้อง [remaining task]"
"[Agent] ทำได้เฉพาะ [completed part]"
"[Agent] ขอเวลาเพิ่มเพื่อทำงานให้เสร็จสมบูรณ์"
```

## Performance Metrics

### Response Time Targets
- **Simple commands**: < 30 seconds
- **Analysis tasks**: < 2 minutes
- **Development tasks**: < 5 minutes
- **Research tasks**: < 3 minutes

### Success Rate Targets
- **Direct commands**: > 95%
- **Complex tasks**: > 85%
- **Multi-agent coordination**: > 80%

### Quality Metrics
- **Accuracy**: > 90%
- **Completeness**: > 85%
- **Actionability**: > 90%

## Maintenance Commands

### System Health Checks
```
"Trader มาตรวจสอบสถานะระบบ"
"Coder มาตรวจสอบ performance"
"News มาตรวจสอบข่าวล่าสุด"
```

### Updates and Improvements
```
"Coder มาอัพเกรดระบบ"
"Trader มาปรับกลยุทธ์"
"News มาปรับ algorithm"
```

### Troubleshooting
```
"Trader มาแก้ปัญหา bot"
"Coder มา debug โค้ด"
"News มาตรวจสอบ data source"
```