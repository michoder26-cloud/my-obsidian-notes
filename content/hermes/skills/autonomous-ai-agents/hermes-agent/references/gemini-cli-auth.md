# Gemini CLI Authentication on Headless Servers

## Key Pitfall

**Gemini CLI (`@google/gemini-cli`) is NOT an MCP server** — it uses `--acp` (Agent Communication Protocol), not `--mcp`. It cannot be added to `mcp_servers` in `config.yaml`.

## Authentication Methods

### 1. OAuth Login (default) - Requires Browser
```bash
gemini auth login
```
**Problem:** Opens browser for Google OAuth — fails on headless VPS.

### 2. API Key (headless-friendly) - Recommended
```bash
# Get key from: https://aistudio.google.com/apikey
# Works with paid Google One AI Premium ($19.99/month) accounts

export GEMINI_API_KEY="AIzaSy..."
gemini -p "hello" --output-format json
```

## Paid Subscription Note

Users with Google One AI Premium ($19.99/month) **do NOT need separate API credits**. When creating an API key while logged into the paid account, the API calls consume the existing subscription quota.

## Docker Container Gotchas

- `jlesage/firefox` image: **Browser only, no terminal** — not suitable for auth via CLI
- Alpine-based containers use `apk add nodejs npm` (not apt)
- Ubuntu 16.04 (xenial) has old libc6 (< 2.28) — incompatible with Node 20.x
- Always install Gemini CLI in container: `npm install -g @google/gemini-cli`
- No native MCP mode — use ACP (`--acp`) for IDE integration instead

## Workflow for Subscription Users (No API Key)

If user insists on OAuth login without API key:

### Option A: NoMachine Desktop (Fastest)
1. VPS already has NoMachine running on port 4000
2. Connect via NoMachine client → get full Ubuntu desktop
3. Open Terminal in desktop → run `gemini auth login`
4. Use existing $19.99/month Google account for login

### Option B: Full VNC Desktop
Use Ubuntu 22.04+ based container with XFCE + noVNC:
```bash
docker run -d \
  --name=gemini-desktop \
  -p 6080:5800 -p 6900:5900 \
  -e VNC_PASSWORD=yourpass \
  -v /root/.gemini:/home/user/.gemini \
  --shm-size=512m \
  conpot/ubuntu-desktop:latest  # or linuxserver/kasm-desktop
```
Then: `docker exec --user root ...` to install node/npm/gemini-cli

## Workflow for Any User (API Key - Easiest)

1. Go to https://aistudio.google.com/apikey
2. Create API key while logged into Google account
3. Send API key to Hermes agent
4. Agent does: `echo 'GEMINI_API_KEY="...'' >> ~/.hermes/.env`
5. Use Gemini via:
   - Direct terminal: `gemini -p "prompt"`
   - Or `delegate_task` for complex tasks

## Post-Auth Usage Patterns

After auth (whichever method):

```bash
# Non-interactive prompt
gemini -p "Summarize this codebase" --output-format json

# Interactive mode
gemini

# With specific model
gemini -m gemini-2.5-pro-001 -p "Analyze this architecture"
```

## Integration with Hermes

- Hermes cannot use Gemini CLI as MCP server (use ACP or direct shell calls)
- Best pattern: `delegate_task(goal="Use Gemini CLI to...", toolsets=["terminal"])`
- Or use `terminal()` to run `gemini -p "..."` directly

## Troubleshooting

| Error | Fix |
|-------|-----|
| "Command not found: npm" | Alpine: `apk add nodejs npm` |
| "libc6 version mismatch" | Use Ubuntu 22.04+, not 16.04 |
| "Browser required for auth" | Use NoMachine or VNC desktop |
| "--mcp: unknown argument" | Gemini CLI uses `--acp`, not `--mcp` |