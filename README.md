# NarrateAI Skill

A skill that teaches Claude (and other AI assistants) how to use NarrateAI's MCP tools for video narration, transcription, translation, dubbing, and document generation.

## What You Can Do

- **Narrate silent videos** — AI writes a script and generates voiceover with natural voices
- **Transcribe speech** — Extract text from meetings, podcasts, lectures
- **Translate transcripts** — Between 11+ languages
- **Dub videos** — Clone the speaker's voice into another language
- **Generate documents** — Tutorials, guides, product docs from screencasts
- **Edit before narrating** — Review and refine AI scripts before producing the final video
- **Batch process** — Up to 5 videos in parallel

## Prerequisites

1. **MCP server installed:** `pip install narrateai-mcp`
2. **API key:** Sign up at [narrateai.app](https://narrateai.app) and purchase MCP credits

## Install

### Claude.ai

1. Download or clone this repo
2. Zip the `narrateai-video-workflows` folder
3. Go to **Claude.ai → Settings → Capabilities → Skills**
4. Click **Upload skill** and select the zip
5. Toggle the skill on

### Claude Code

Place the skill folder in your project or global skills directory:

```bash
# Project-level
cp -r narrateai-video-workflows /your-project/.claude/skills/

# Or global
cp -r narrateai-video-workflows ~/.claude/skills/
```

### Cursor

```bash
cp -r narrateai-video-workflows /your-project/.cursor/skills/
```

### Other IDEs (Windsurf, JetBrains, etc.)

Copy the contents of `narrateai-video-workflows/SKILL.md` into your IDE's AI rules or custom instructions file.

## MCP Server Configuration

After installing the skill, make sure the NarrateAI MCP server is connected. Add to your MCP config:

```json
{
  "mcpServers": {
    "narrateai": {
      "command": "narrateai-mcp",
      "env": {
        "NARRATEAI_API_BASE_URL": "https://api.narrateai.app",
        "NARRATEAI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

Config file locations:
- **Claude.ai:** MCP servers are configured in Settings → Extensions
- **Claude Code:** `.mcp.json` in project root or `~/.claude/mcp.json`
- **Cursor:** `.cursor/mcp.json` in project root
- **Windsurf:** `~/.codeium/windsurf/mcp_config.json`

## Usage

Once installed, just talk naturally:

> "Narrate this video with the male1 voice: demo.mp4"

> "Transcribe this Spanish meeting recording"

> "Dub this English video into French, keep the background music"

> "Generate a tutorial guide from this screencast"

> "Take my Messi video, translate it to Hindi, and re-narrate it"

The skill automatically activates when your request matches a NarrateAI workflow.

## Available Voices

- **male1** (default, fastest generation) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Skill Structure

```
narrateai-video-workflows/
├── SKILL.md                    # Main skill file (required)
└── references/
    └── workflows.md            # Detailed workflow reference (loaded on demand)
```

## Links

- [NarrateAI](https://narrateai.app) — Sign up and get API credits
- [narrateai-mcp on PyPI](https://pypi.org/project/narrateai-mcp/) — MCP server package
- [Anthropic Skills Guide](https://docs.anthropic.com/en/docs/agents-and-tools/skills) — How skills work
