# VPS Service Recovery

Diagnosing and recovering a dead service on a remote VPS (SSH-accessible Linux).

## When to Use

- A service the user expects to be running is unreachable (HTTP 503, connection refused, etc.)
- User reports a URL/dashboard/server is down
- You notice no process is listening on an expected port

## Troubleshooting Steps

1. **Browser check**: Navigate to the URL. Note the error (e.g., `ERR_CONNECTION_REFUSED`).
2. **Check listening ports**: `ss -tlnp | grep <port>` — if empty, nothing is listening.
3. **Check for process**: `ps aux | grep <service_name>` — look for the process. If found but port is closed, it's hung (restart needed). If not found, it died.
4. **Restart**: Run the service's start command in the background with Hermes tracking:
   ```
   terminal(background=true, command="<start_command>", notify_on_complete=false)
   ```
5. **Verify**: `ss -tlnp | grep <port>` should show the port listening. Test with `curl http://127.0.0.1:<port>/`.
6. **Log output**: Check process output via `process(action='log', session_id='<id>')` or `process(action='poll', ...)`.

## Common Pitfalls

- Services running without systemd/cron watchdogs will die silently. Always recommend a watchdog (cron check or systemd unit) for any long-running service the user depends on.
- If the service uses a Python venv, use the venv's python: `path/to/.venv/bin/python3`
- Services started via `terminal(background=true)` in Hermes will NOT survive Hermes restart — they need proper daemonization (systemd) or a cron-based watchdog.
- Port `ss -tlnp` may show the process but the service isn't actually responding (hung process). In that case, kill and restart.
