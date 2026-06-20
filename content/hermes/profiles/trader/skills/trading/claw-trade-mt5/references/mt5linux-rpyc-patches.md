# mt5linux RPyC Patches — Fixes for Order Execution Errors

Two bugs discovered during live trading startup (2026-06-18 session). Both must be applied or `mt5.order_send()` will fail.

## Bug 1: `NameError: name 'np' is not defined`

### Root cause

`mt5linux/metatrader5.py` (the host-side library) connects via RPyC and seeds the Wine-side Python namespace with only:

```python
self.__conn.execute("import MetaTrader5 as mt5")
self.__conn.execute("import datetime")
```

When `mt5.order_send(request)` serializes the order dict, RPyC's `eval()` on the Wine side references `np` (numpy) — which was never imported into the RPyC namespace. The result:

```
ERROR:orchestrator:Error in trading loop: name 'np' is not defined
========= Remote Traceback (1) =========
  File "...\rpyc\core\service.py", line 159, in eval
    return eval(text, self.namespace)
  File "<string>", line 1, in <module>
NameError: name 'np' is not defined
```

### Fix

Patch the installed `mt5linux/metatrader5.py` to add numpy import:

**File:** `<venv>/lib/python3.11/site-packages/mt5linux/metatrader5.py`

```diff
         self.__conn = rpyc.classic.connect(host, port)
         self.__conn._config["sync_request_timeout"] = timeout
         self.__conn.execute("import MetaTrader5 as mt5")
         self.__conn.execute("import datetime")
+        self.__conn.execute("import numpy as np")
```

Find the exact path:
```bash
python3 -c "import mt5linux; print(mt5linux.__file__)"
# → /usr/local/lib/hermes-agent/venv/lib/python3.11/site-packages/mt5linux/metatrader5.py
```

### Verify

```python
python3 -c "
import sys; sys.path.insert(0,'/root/Claw_Trade/src')
import mt5linux, os
from dotenv import load_dotenv; load_dotenv('/root/Claw_Trade/.env')
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
mt5.initialize(login=int(os.getenv('MT5_LOGIN')), password=os.getenv('MT5_PASSWORD'), server=os.getenv('MT5_SERVER'))
request = {
    'action': mt5.TRADE_ACTION_DEAL, 'symbol': 'XAUUSDc', 'volume': 0.01,
    'type': mt5.ORDER_TYPE_BUY, 'price': mt5.symbol_info_tick('XAUUSDc').ask,
    'sl': 0.0, 'tp': 0.0, 'deviation': 20, 'magic': 234000, 'comment': 'test',
    'type_time': mt5.ORDER_TIME_GTC, 'type_filling': mt5.ORDER_FILLING_IOC,
}
result = mt5.order_send(request)
print('Result:', result)
"
# If retcode=10009 → fix is working
```

### Must re-patch after

- `pip install --upgrade mt5linux` — overwrites the file
- Recreating the venv

---

## Bug 2: `'dict' object has no attribute 'balance'`

### Root cause

`mt5_connector.py`'s `get_account_info()` (lines 257-298) **always returns a dict** — both the tuple/list branch (lines 270-281) and the namedtuple branch (lines 284-295) construct and return a dict.

But `orchestrator.py` line 677 used attribute access:

```python
# BROKEN:
balance = acc_info.balance if acc_info is not None else 10000.0
```

This crashes with `'dict' object has no attribute 'balance'` when the bot tries to size a position.

### Fix

**File:** `/root/Claw_Trade/src/orchestrator.py` ~line 676

```diff
             if self.mode == "LIVE":
                 acc_info = self.mt5_connector.get_account_info()
-                balance = acc_info.balance if acc_info is not None else 10000.0
+                if acc_info is not None:
+                    balance = acc_info['balance'] if isinstance(acc_info, dict) else getattr(acc_info, 'balance', 10000.0)
+                else:
+                    balance = 10000.0
             else:
                 balance = self.backtester.current_balance
```

### Verify

```bash
cd /root/Claw_Trade && python3 -c "
import sys; sys.path.insert(0,'src')
from dotenv import load_dotenv; load_dotenv()
from mt5_connector import MT5Connector
conn = MT5Connector(); conn.connect()
info = conn.get_account_info()
print('type:', type(info), 'is dict:', isinstance(info, dict))
print('balance:', info['balance'] if isinstance(info, dict) else info.balance)
"
# → type: <class 'dict'>  is dict: True  balance: 10302.18
```

---

## Patch application sequence (for cold-start)

1. Clear Python caches: `find /root/Claw_Trade/src -name __pycache__ -type d -exec rm -rf {} +`
2. Also clear mt5linux cache: `find <venv>/.../mt5linux -name __pycache__ -type d -exec rm -rf {} +`
3. Apply both patches
4. Kill any running bot process
5. Restart `main.py live --confirm`
6. Verify log shows `✅ MT5 Order executed successfully!` (not `NameError` or `'dict' object has no attribute`)