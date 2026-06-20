---
name: gemini-cli-setup
title: Gemini CLI Setup and Authentication (API‑Key Mode)
description: Step‑by‑step guide for configuring Gemini CLI on headless servers (VPS) to use API‑Key authentication instead of OAuth, including common pitfalls and verification.
summary: |
  Configure Gemini CLI on a headless VPS to work with an API‑Key, remove stale OAuth credentials, and verify the setup.
---

# Overview
The Gemini CLI (`gemini`) supports two authentication methods:
- **OAuth (personal)** – requires a browser login, unsuitable for headless servers.
- **API‑Key** – works directly via the `GEMINI_API_KEY` environment variable.

For VPS or remote‑desktop setups, use the API‑Key method.

# Procedure
1. **Install the CLI** (skip if already present)
   ```bash
   npm install -g @google/gemini-cli@latest   # or use the bundled binary at ~/.hermes/node
   ```
2. **Delete any cached OAuth accounts**
   ```bash
   rm -f ~/.gemini/google_accounts.json
   ```
3. **(Optional) Clean the whole config directory**
   ```bash
   rm -rf ~/.gemini/*
   ```
4. **Create a fresh `settings.json` that selects API‑Key authentication**
   ```bash
   cat > ~/.gemini/settings.json <<'EOF'
   {
     "security": {
       "auth": {
         "selectedType": "api-key"
       }
     }
   }
   EOF
   ```
5. **Export your API key** (replace the placeholder with the real key)
   ```bash
   export GEMINI_API_KEY="AQ.Ab8...KQ"
   echo 'export GEMINI_API_KEY="AQ.Ab8...KQ"' >> ~/.bashrc   # persist for future sessions
   ```
6. **Verify the configuration**
   ```bash
   gemini -p "hello"   # should return a short answer, no browser prompt
   ```
   Expected output: a JSON or plain‑text reply, *not* `Invalid auth method selected.`

# Common Pitfalls & Fixes
- **`Invalid auth method selected.`** – caused by leftover OAuth settings. Ensure step 2‑4 are completed and restart the shell.
- **`Could not find a suitable web browser!`** – occurs when running `gemini login` on a headless machine. Use API‑Key mode instead.
- **Missing `~/.gemini/google_accounts.json`** – delete it as in step 2.

# Verification Script (optional)
Create `scripts/verify-gemini.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail
if [[ -z "${GEMINI_API_KEY:-}" ]]; then
  echo "GEMINI_API_KEY not set"
  exit 1
fi
gemini -p "test" | head -n 5
```
Make it executable and run to double‑check the setup.

# References
- `references/session-gemini-setup.md` – condensed transcript of this troubleshooting session.
- Official Gemini CLI docs: https://ai.google.dev/gemini-cli
