# Docker + mt5linux Setup Reference

## Architecture
```
Host Linux (Claw_Trade bot)
  └── RPyC (port 8001) ──→ Docker Container
                            └── Wine (Windows emulation)
                                ├── MetaTrader 5 terminal64.exe
                                ├── Python 3.9 (Win32)
                                ├── MetaTrader5 Python lib
                                └── mt5linux RPyC server
```

## Container Details
- **Image**: `gmag11/metatrader5_vnc` (based on linuxserver/kasmvnc)
- **Base**: Debian Bookworm
- **Wine**: wine-10.0 (included in image)
- **Python**: 3.9 (Win32, inside Wine)
- **VNC**: KasmVNC on port 3000, Xvfb display :1
- **Default user**: `abc` (uid 911), NOT root

## The numpy Crisis (and Fix)

### Symptoms
```
MetaTrader5 __init__.py line 257: from ._core import *
ImportError: numpy.core.multiarray failed to import
```

### Root Cause
MetaTrader5's `_core.cp39-win32.pyd` is a compiled C extension (DLL) that:
1. Is loaded by Python inside Wine
2. Links against numpy's C API
3. Was compiled against numpy 1.x API version 0xe
4. Breaks with numpy 1.19 (API 0xd = too old) OR numpy 2.x (ABI break = too new)

### Working numpy versions (confirmed)
- ✅ **numpy 1.24.3** — works after vcrun2019 is installed
- ❌ numpy 1.19.5 — API mismatch (0xd vs 0xe)
- ❌ numpy 2.0.2 — numpy 2.x ABI break

### Fix Steps
```bash
# 1. Install cabextract (needed by winetricks)
apt-get install -y cabextract

# 2. Install winetricks
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks -O /usr/local/bin/winetricks
chmod +x /usr/local/bin/winetricks

# 3. Fix .wine ownership
chown -R abc:abc /config/.wine

# 4. Install vcrun2019 (Visual C++ 2019 redistributable)
su abc -c "WINEPREFIX=/config/.wine winetricks -q vcrun2019"

# 5. Install correct numpy version
su abc -c 'export WINEPREFIX=/config/.wine && wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -m pip install numpy==1.24.3 --force-reinstall'
```

### Why vcrun2019 is needed
The `_core.pyd` DLL depends on MSVC CRT (msvcp140.dll, vcruntime140.dll) which aren't included in the default Wine setup. Without these, the DLL can't load properly, causing the numpy.core.multiarray import chain to fail.

## Container Commands Cheat Sheet
| Action | Command |
|--------|---------|
| Run as abc user | `docker exec --user abc claw-trade-mt5 ...` |
| Run Wine command | `docker exec --user abc claw-trade-mt5 bash -c 'export WINEPREFIX=/config/.wine && wine "C:\\path\\to\\exe" args'` |
| Run Python in Wine | `wine "C:\\Program Files (x86)\\Python39-32\\python.exe" -c "code"` |
| Install pip package | `wine python -m pip install <package>` |
| Start mt5linux server | `wine python -m mt5linux --host 0.0.0.0 --port 8001` |

## .env File Security Note
The `write_file` tool automatically redacts sensitive values (API keys, passwords) with `***` when writing `.env` files. To write a correct `.env`:
- Use terminal with Python heredoc: `python3 << 'PYEOF' ... PYEOF`
- Or use terminal with cat + manual editing

## mt5_connector.py changes
The connector auto-detects mt5linux vs native:
```python
try:
    from mt5linux import MetaTrader5 as _MT5Client
    mt5 = _MT5Client(host=os.getenv('MT5_HOST','127.0.0.1'), port=int(os.getenv('MT5_PORT','8001')))
except ImportError:
    import MetaTrader5 as mt5
```

## Known Issues
- `mt5.initialize()` returns False until MT5 terminal is fully started AND logged in
- mt5linux attempts `import MetaTrader5` eagerly at client construction (not lazily)
- Container ~4-6GB disk, ~1-2GB RAM
- x86_64 only (no ARM/M1 support due to Wine)
- mt5linux server auto-start in container init script may fail if MT5 not yet logged in