Watchdog: /usr/local/bin/gateway-watchdog.sh (5 profiles: trader/coder/news/system/security) + cron */2 gateway-monitor.sh. SILENT mode. All gateways via terminal(background=true).
§
Dashboard port 8080 (/tmp/start_dashboard.sh), Hermes port 9119 (/usr/local/bin/hermes-services.sh). Agent HQ pixel art at /agent-hq only; rest = clean dark theme.
§
Ollama Cloud: base_url=https://ollama.com/v1, key=d27299...WG. 43 models incl minimax-m3 (default), glm-5.2 (prev default), kimi-k2.6, qwen3-coder, deepseek. Switch: hermes --profile <name> config set model.default <id> then restart.
§
อัปเดต skill references/telegram-group-chat.md ของ multi-agent-hermes-setup ด้วยขั้นตอนล่าสุดที่ใช้จริง: disable privacy mode ทุก bot → getUpdates ผ่าน API โดย stop gateway ชั่วคราว → เขียน group ID ด้วย Python (ห้ามใช้ shell echo เพราะ censor token) → restart gateway ทุกตัวพร้อม stagger start. และเพิ่ม pitfall 'Stagger gateway starts: Starting 5 gateways simultaneously causes RAM spike → OOM'.
§
Pixel Agent Office at http://13.140.183.183:9120 — p5.js scene with pixel-agents-hq sprites + Hermes API. 4 agents walk between lounge (sofa) and desks. State machine: resting→to_desk→working→to_rest, driven by API 'working' flag. Key fixes: drawImage() not p5 image(); spriteScale not scale; signText not text param; line.text.includes(). server.py detects CLI work processes.
§
# DEFAULT = ORCHESTRATOR (ผู้จัดการสูงสุด)
- รับงาน → delegate ไป agent → ตรวจผล → สรุปให้ user (ไม่ทำเองหมด)
- 6 agents: default(orchestrator), trader, coder, news, system, security
- Max 3 concurrent delegate_task (Ollama Cloud limit)
- User: TK TF, Thai, concise, hates Telegram spam, trades XAU/USD
- Security first! VPS was xmrig-compromised → cleaned, C2 blocked
- Migration VPS→MiniPC: plan ready in memory, waiting user signal
§
Obsidian vault: /root/obsidian-vault/ (trading/system/cron-output/journal/security). User tried Syncthing but gave up (too complex). Used HTTP download instead. User main PC: Windows. Prefers SIMPLE solutions — when too hard, says "ไม่เอาแล้ว" and wants agent to do it for them.