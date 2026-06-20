# Auxiliary Model Configuration

## Problem

By default, Hermes sets `auxiliary.*.provider: auto` for all auxiliary tasks.
The `auto` provider may route to Google/Gemini if `GOOGLE_API_KEY` or `GEMINI_API_KEY`
is set, or if no other provider is detected.

Users who do NOT want Google dependency must explicitly set every auxiliary task
to their preferred provider.

## All Auxiliary Tasks (13 total)

| Task | Purpose |
|------|---------|
| `vision` | Image analysis |
| `web_extract` | Web content extraction |
| `compression` | Context compression |
| `skills_hub` | Skill hub operations |
| `approval` | Command approval (smart mode) |
| `mcp` | MCP server interactions |
| `tts_audio_tags` | TTS audio tag generation |
| `triage_specifier` | Task triage |
| `kanban_decomposer` | Kanban task decomposition |
| `profile_describer` | Profile description generation |
| `curator` | Skill curator operations |
| `monitor` | Monitoring tasks |
| `title_generation` | Session title generation |

## Fix: Set All to Ollama Cloud

```bash
export PATH="$HOME/.local/bin:$PATH"

for task in vision web_extract compression skills_hub approval mcp tts_audio_tags triage_specifier kanban_decomposer profile_describer curator monitor title_generation; do
  hermes config set auxiliary.$task.provider ollama-cloud
  hermes config set auxiliary.$task.model glm-5.2
done
```

## Fix: Set All to Z.AI

```bash
for task in vision web_extract compression skills_hub approval mcp tts_audio_tags triage_specifier kanban_decomposer profile_describer curator monitor title_generation; do
  hermes config set auxiliary.$task.provider zai
  hermes config set auxiliary.$task.model glm-5.2
done
```

## Also Check TTS

The `tts.gemini` config section references `gemini-2.5-flash-preview-tts`.
Switch TTS to `edge` (free, no Google dependency):

```bash
hermes config set tts.provider edge
```

## Verification

```bash
hermes status | grep -i "google\|gemini"
# Should show nothing or all ✗
```