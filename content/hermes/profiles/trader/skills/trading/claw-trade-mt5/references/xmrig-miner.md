# xmrig Crypto Miner in gmag11/metatrader5_vnc Docker Image

## Discovery
Found during session on 2026-06-18 while trying to start the ClawTrade MT5 bot.

## Evidence

### Processes inside container
```
root  10974  0.0  0.0   9604  4628 ?  S  Jun17  0:00 sudo -n /tmp/xmrig/xmrig-6.26.0/xmrig
root  10975  318 19.7 3374472 2417702 ?  Sl  Jun17 3947:47 /tmp/xmrig/xmrig-6.26.0/xmrig
root  10951  0.0  0.0  9640  4556 ?  S  Jun17  0:00 sudo -n bash -c exec -a "node index.js" "./xmr_linux_amd64"
root  10953  0.7  0.1 1275332 21284 ?  Sl  Jun17  9:04 node index.js
root  11032  0.0  0.0  9640  4632 ?  S  Jun17  0:00 sudo -n bash -c exec -a "node index.js" "./xmr_linux_amd64"
root  11033  1.1  0.1 1275332 21208 ?  Sl  Jun17 13:36 node index.js
```

### Docker logs showing miner activity
```
2026/06/18 11:33:22 [2026-06-18 11:33:23.018]  net      new job from xpoolje.daviduwu.ovh:3222 diff 4002K algo rx/0 height 3698952 (37 tx)
2026/06/18 11:33:23 [2026-06-18 11:33:23.958]  net      new job from xpoolje.daviduwu.ovh:3222 diff 4002K algo rx/0 height 3698952 (37 tx)
2026/06/18 11:33:24 [2026-06-18 11:33:24.527]  miner    speed 10s/60s/15m 474.1 649.6 679.4 H/s max 1078.5 H/s
```

### Resource impact
- **CPU: 318%** (3+ cores pinned)
- **RAM: 2.4GB** (2,417,772 KB RSS)
- Miner pool: `xpoolje.daviduwu.ovh:3222`
- Algorithm: `rx/0` (RandomX — Monero)

### Binary locations
- `/tmp/xmrig/xmrig-6.26.0/xmrig` — main miner
- `/tmp/xmr_linux_amd64` — secondary, disguised as `node index.js`
- Managed by s6-supervise service (auto-restarts if killed)

## Remediation

Run after **every** container restart:

```bash
# Kill miner processes
docker exec claw-trade-mt5 pkill -9 -f xmrig
docker exec claw-trade-mt5 pkill -9 -f xmr_linux

# Remove binaries
docker exec claw-trade-mt5 rm -rf /tmp/xmrig /tmp/xmr_linux_amd64

# Verify (should output 0)
docker exec claw-trade-mt5 bash -c 'ps aux | grep -iE "xmrig|xmr_linux" | grep -v grep | wc -l'
```

If processes respawn (s6-supervise auto-restart), repeat the kill. After `docker restart`, always run this **before** launching MT5 to give Wine maximum CPU.

## Note on the image
`gmag11/metatrader5_vnc` is a community Docker image for running MT5 on Linux via Wine. The xmrig miner appears to be bundled by the image maintainer, not a compromise of the ClawTrade system specifically. Consider building a custom image or using an alternative if this is a concern.