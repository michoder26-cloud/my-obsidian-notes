# mt5linux RPyC Integration Quirks

## Architecture
mt5linux connects to a Python 3.9 (32-bit) process running inside Wine in the Docker container.
Communication is via RPyC (Remote Python Call) on port 8001. The host-side `mt5linux` package
wraps RPyC calls to make the remote MetaTrader5 API feel like a local import.

## The numpy Namespace Bug (CRITICAL)

### Symptom
```
ERROR:orchestrator:Error in trading loop: name 'np' is not defined
Remote Traceback:
  File "...rpyc/core/service.py", line 159, in eval
    return eval(text, self.namespace)
  File "<string>", line 1, in <module>
NameError: name 'np' is not defined
```

### Root Cause
RPyC uses `eval()` on the Wine side to execute code sent from the host. The RPyC `SlaveService`
namespace only imports what `mt5linux.MetaTrader5.__init__()` tells it to:
```python
self.__conn.execute("import MetaTrader5 as mt5")
self.__conn.execute("import datetime")
```
numpy is installed in Wine Python but NOT imported into the RPyC namespace. When `mt5.order_send()`
or other MT5 functions internally reference numpy (or when RPyC serialization triggers eval of
code containing `np`), it fails.

### Fix
In `mt5linux/metatrader5.py` (host-side site-packages), add to `__init__`:
```python
self.__conn.execute("import numpy as np")
```
After: `self.__conn.execute("import datetime")`

### Why it only manifests on order_send
Analysis and data fetch don't trigger this because they use simpler RPyC calls. `order_send`
involves more complex serialization (TradeRequest namedtuple, result objects) that triggers
RPyC eval paths referencing numpy types.

## account_info Return Type

`MT5Connector.get_account_info()` converts the RPyC netref to a dict in both code paths:
- If `isinstance(info, (tuple, list))` → returns dict with indexed keys
- Else (netref/namedtuple) → returns dict with attribute-based keys

Always returns dict or None. Orchestrator code must use `acc_info['balance']`, not `acc_info.balance`.

## RPyC netref objects
mt5linux returns `rpyc.core.netref.builtins.AccountInfo` (and similar) objects. These behave like
namedtuples and support attribute access (`.balance`, `.login`, etc.) but:
- `isinstance(info, (tuple, list))` returns False
- `isinstance(info, dict)` returns False
- `hasattr(info, 'balance')` returns True

So `get_account_info()` goes to the `else` branch and converts to dict via `info.balance` etc.

## tz_convert error (non-critical)
```
ERROR:mt5_connector:Error fetching MT5 historical data: tz_convert() takes exactly 2 positional arguments (1 given)
```
Pandas 3.x on host vs pandas 1.x in Wine causes a timezone conversion API mismatch. The bot
falls back to Yahoo Finance (`GC=F`) for historical data. Not a blocker — just means historical
data comes from Yahoo instead of MT5 directly.

## Manual connection test
```python
import mt5linux
mt5 = mt5linux.MetaTrader5(host='localhost', port=8001)
mt5.initialize(login=LOGIN, password=PASSWORD, server=SERVER)
# terminal_info() returns None if MT5 not logged in
# account_info() returns None if not connected
# Both return objects when properly connected
```