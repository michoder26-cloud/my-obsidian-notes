# Windows Setup Guide - Claude Code x NotebookLM

## Status

✅ **Setup Complete** - Ready to use with Claude Code

- Dependencies installed: `uv sync` ✓
- MCP Server registered: `claude mcp list` ✓  
- Server startup verified ✓
- Authentication: ⏳ (see below)

---

## 🔐 Authentication on Windows

NotebookLM uses browser-based authentication. Due to a Windows Playwright subprocess limitation, use this workaround:

### Option 1: Browser-Based Manual Auth (Recommended)

1. **Open Chrome and go to NotebookLM:**
   ```
   https://notebooklm.google.com/
   ```

2. **Sign in with your Google account**

3. **Access or create a notebook** to ensure auth is complete

4. **Close the browser** - your cookies are saved automatically

This saves authentication cookies to: `C:\Users\TKTF\.notebooklm\browser_profile`

### Option 2: Generate Token via Google API (Advanced)

If you prefer to use a service account or API key instead:

1. Create a service account in Google Cloud Console
2. Download the service account JSON
3. Set environment variable: 
   ```powershell
   $env:NOTEBOOKLM_SERVICE_ACCOUNT_JSON = "C:\path\to\service_account.json"
   ```

---

## 🚀 Using Claude Code

### List Your Notebooks

```bash
claude
# Then in Claude Code session:
List my NotebookLM notebooks
```

### Available Commands

| Task | Example |
|------|---------|
| List notebooks | "Show me all my NotebookLM notebooks" |
| Create notebook | "Create a new notebook called 'Research'" |
| Add URL source | "Add this URL to notebook [id]: https://example.com" |
| Ask notebook | "In notebook [id], what is..." |
| Generate podcast | "Generate a podcast for notebook [id]" |
| Create slides | "Make a slide deck from notebook [id]" |

### Available Tools

- `list_notebooks` - List all notebooks
- `create_notebook` - Create new notebook
- `add_source_url` - Add website URL
- `add_source_text` - Add raw text
- `ask_notebook` - Ask questions about notebook sources
- `generate_audio_overview` - Podcast generation
- `generate_slide_deck` - PowerPoint slides
- `generate_mind_map` - Interactive mind map
- `generate_quiz` - Quiz questions
- `generate_flashcards` - Study cards
- And more...

---

## 🔧 Troubleshooting

### "Server disconnected" in Claude Code

1. Verify authentication worked:
   ```powershell
   Get-ChildItem "$env:USERPROFILE\.notebooklm" -Recurse
   ```
   Should show `browser_profile/` directory

2. Check if Chrome has saved cookies:
   ```powershell
   Get-ChildItem "$env:USERPROFILE\.notebooklm\browser_profile" -Recurse
   ```

3. Restart Claude Code:
   ```bash
   claude
   ```

### Server fails to authenticate

**Solution:**
1. Delete old cache: `Remove-Item "$env:USERPROFILE\.notebooklm\browser_profile" -Recurse -Force`
2. Visit https://notebooklm.google.com/ in Chrome again
3. Sign in and create/access a notebook
4. Restart Claude Code

### Check MCP Server Status

```powershell
cd C:\Users\TKTF\notebooklm-mcp
claude mcp list
```

Should show: `notebooklm - ✓ Connected` (after auth)

---

## 📝 Next Steps

1. **Authenticate:** Open https://notebooklm.google.com/ in Chrome
2. **Test:** Run `claude` and ask about your notebooks
3. **Use:** Start chatting with your data through Claude Code!

---

## 📚 Project Structure

```
notebooklm-mcp/
├── server.py            # MCP server with tools
├── pyproject.toml       # Dependencies
├── README.md            # Full documentation
├── WINDOWS_SETUP.md     # This file
└── .venv/              # Virtual environment
```

---

## 🔗 Resources

- [NotebookLM](https://notebooklm.google.com/)
- [Claude Code](https://claude.com/claude-code)
- [MCP Specification](https://modelcontextprotocol.io/)
- [FastMCP](https://gofastmcp.com/)
