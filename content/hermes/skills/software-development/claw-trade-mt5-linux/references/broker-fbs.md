# FBS Broker — MT5 Connection Reference

## Server Names
| Account Type | Server Name |
|-------------|-------------|
| Demo | `FBSTradestone-Demo` |
| Real | `FBSTradestone-Real` or `FBS-Real` |

## Symbol Names
Gold symbols on FBS:
- `XAUUSDc` ✅ (primary — used by this bot)
- `XAUUSD` (sometimes available)

Check Market Watch (Ctrl+M) in MT5 for the exact symbol name for your account type.

## Login Flow
1. Open MT5 via VNC (http://SERVER_IP:3000)
2. File → Login to Trade Account
3. Enter: Login ID (number), Password, Server name
4. Ensure "Enable algo trading" is checked (Tools → Options → Expert Advisors)

## Cent vs Standard
FBS Demo accounts are typically Standard. `$10,450.96` balance is the demo starting amount.
