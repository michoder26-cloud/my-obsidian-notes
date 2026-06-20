## Session summary (2026‑06‑16)
- User provided Gemini API key `AQ.Ab8...Q` and wanted Gemini CLI to work on a headless VPS.
- Initial attempts `gemini -p "hello"` timed‑out and returned `Invalid auth method selected.`.
- Root cause: leftover OAuth configuration (`~/.gemini/google_accounts.json`) and `settings.json` still set to `oauth-personal`.
- Fix steps applied:
  1. Delete `~/.gemini/google_accounts.json`.
  2. (Optionally) wipe `~/.gemini/*`.
  3. Create new `settings.json` with `selectedType: "api-key"`.
  4. Export `GEMINI_API_KEY` and persist in `~/.bashrc`.
  5. Verify with `gemini -p "hello"` – should return a short answer.
- Added common pitfalls and verification script.
- Recorded that VNC processes in MT5 container are unrelated to Gemini CLI and should remain running for trading bot.
