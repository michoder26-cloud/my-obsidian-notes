# VPS → Raspberry Pi 5 Migration Analysis

Recorded from a real assessment session (June 2026) for a 4-profile Hermes
deployment with ClawTrade (MT5 + Wine + Docker).

## Current VPS Resource Usage (Measured)

| Component | RAM | Notes |
|-----------|-----|-------|
| Hermes Gateways × 4 | ~1,048 MB | trader, coder, news, system (~170-230 MB each) |
| Dashboard (port 9119) | ~184 MB | |
| Pixel Agent Office (port 9120) | ~3 MB | Python HTTP server + p5.js frontend |
| ClawTrade Docker (MT5+Wine) | ~419 MB | heaviest Docker consumer |
| VS Code Remote Server | ~530 MB | only present when editing remotely |
| Docker daemon | ~84 MB | |
| systemd + misc | ~200 MB | |
| **Total (no VS Code)** | **~1.94 GB** | normal load |
| **Total (with VS Code)** | **~2.47 GB** | during code editing |

VPS specs: AMD EPYC 6C @ 2.0GHz, 12 GB RAM, NVMe 376 MB/s, no swap.

## Critical Constraint: MT5 on ARM

**MT5 Terminal (terminal64.exe) is x86_64 Windows software.**
It cannot run on Raspberry Pi 5 (ARM64) — Wine on ARM does not support
x86_64 Windows apps. Even box86/box64 translation layers are too slow
and unstable for MT5.

The `MetaTrader5` Python package also requires Windows — it does not
run on ARM Linux.

**Conclusion:** ClawTrade MUST stay on an x86_64 host (VPS or Mini PC).

## Raspberry Pi 5 vs VPS Comparison

| Aspect | VPS | Pi 5 | Impact |
|--------|-----|------|--------|
| CPU | EPYC 6C/6T @ 2.0GHz | Cortex-A76 4C/4T @ 2.4GHz | Pi ~30% weaker per-core |
| RAM | 12 GB | 4 or 8 GB | 8 GB recommended |
| Disk | NVMe 376 MB/s | SD: ~50 MB/s, USB3 SSD: ~400 MB/s | SD card unsuitable for 24/7 |
| Network | Datacenter ~1-5ms to broker | Home/office ~20-100ms+ | Critical for trading |
| Uptime | 99.9%+ | Power outage / ISP dropout risk | Needs UPS |
| Architecture | x86_64 | ARM64 | MT5 incompatible |

## Recommendations

### If goal is "save monthly VPS cost":

**Best option: Mini PC x86 (N100)** — e.g. Beelink EQ12
- Runs MT5 + Wine + Docker + all Hermes profiles
- 16GB RAM, NVMe SSD included
- ~3,500 THB one-time vs ~500-1,000 THB/month VPS
- Still has home-network latency/outage issues

### If using Pi 5 anyway:
- Buy **8 GB** version (~2,500 THB)
- Must use **USB3 SSD**, not SD card
- Must buy **UPS** (~800 THB)
- Must buy **active cooler** (fan + heatsink, ~300 THB)
- ClawTrade stays on a separate cheap x86 VPS (e.g. Hetzner CX11 ~150 THB/mo)
- Total one-time: ~4,900 THB

### RAM sizing for Pi 5 (Hermes only, no ClawTrade):
- Minimum: 4 GB (tight, ~1.7 GB used, little buffer)
- Recommended: 8 GB (comfortable, room for swap + cache)

## 3.5" LCD Touchscreen (ILI9486 480x320 SPI)

User asked about adding a 3.5" LCD for status display.

- SPI interface: slow refresh, fine for text/status, not video
- Resistive touch: usable but not smooth
- "For Pi 4B" labeling: usually works on Pi 5 (same 40-pin header) but
  verify driver compatibility with Pi 5's RP1 I/O chip
- Does NOT consume significant RAM (~10-20 MB framebuffer)
- Good for: showing gateway/bot status at a glance
- No use case on Mini PC x86 (no GPIO header)
- Buy AFTER deciding platform, not before

## Decision Matrix

| Option | Cost | MT5? | Network | Uptime | Verdict |
|--------|------|------|---------|--------|---------|
| Keep VPS | ~500-1k/mo | ✅ | ✅ fast | ✅ | safest |
| Pi 5 8GB + cheap VPS for MT5 | ~4.9k + ~150/mo | split | home | home | OK if fine with latency |
| Mini PC x86 N100 | ~3.5k one-time | ✅ | home | home | best value, all-in-one |