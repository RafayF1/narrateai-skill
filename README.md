# NarrateAI — AI Video Workflows for MCP

Workflow instructions that teach AI assistants how to use NarrateAI's MCP tools for video narration, transcription, translation, dubbing, and document generation.

## What is this?

When you connect the [NarrateAI MCP server](https://pypi.org/project/narrateai-mcp/) to your AI assistant, it gets access to video processing tools. These workflow files teach the AI *how* to use those tools effectively — when to ask for context, which voice to suggest, how to chain tools together, etc.

**Without workflows:** The AI can call the tools but may skip important steps (like asking for context, which dramatically improves narration quality).

**With workflows:** The AI follows best practices, asks the right questions, and handles complex multi-step flows like translate-then-re-narrate.

## Quick Start

### 1. Install the MCP server

```bash
pip install narrateai-mcp
```

### 2. Get an API key

Sign up at [narrateai.app](https://narrateai.app) and purchase MCP credits.

### 3. Choose your platform

| Platform | File to use | Where to put it |
|----------|------------|----------------|
| **Cursor** | `cursor/SKILL.md` | `.cursor/skills/narrateai-video-workflows/SKILL.md` |
| **Claude Code** | `claude/CLAUDE.md` | Project root as `CLAUDE.md` |
| **Claude Desktop** | `claude/project-knowledge.md` | Project Settings → Add Knowledge |
| **Windsurf** | `other-ides/INSTRUCTIONS.md` | `.windsurfrules` in project root |
| **JetBrains AI** | `other-ides/INSTRUCTIONS.md` | AI Assistant custom instructions |
| **Google AI / Gemini** | `other-ides/INSTRUCTIONS.md` | System prompt or context |
| **Any MCP client** | `workflows.md` | Your platform's context/instructions |

### 4. Configure MCP

Every platform needs the same MCP server config (adjust the format for your platform):

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

See your platform's file for exact config location and format.

## What You Can Do

- **Narrate silent videos** — AI writes a script and generates voiceover
- **Transcribe speech** — Extract text from videos with existing audio
- **Translate transcripts** — Between 11+ languages
- **Dub videos** — Clone the speaker's voice in another language
- **Generate documents** — Tutorials, guides, product docs from screencasts
- **Edit before narrating** — Review and refine AI scripts before producing video
- **Batch process** — Up to 5 videos in parallel
- **Browse video library** — Access and re-use past videos

## Repository Structure

```
├── README.md                      ← You are here
├── workflows.md                   ← Full workflow reference (platform-agnostic)
├── cursor/
│   └── SKILL.md                   ← Cursor skill file (YAML frontmatter)
├── claude/
│   ├── CLAUDE.md                  ← Claude Code project file
│   └── project-knowledge.md       ← Claude Desktop project knowledge
└── other-ides/
    └── INSTRUCTIONS.md            ← Windsurf, JetBrains, Gemini, others
```

## Available Voices

- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko

## Links

- [NarrateAI](https://narrateai.app) — Sign up and get API credits
- [narrateai-mcp on PyPI](https://pypi.org/project/narrateai-mcp/) — Install the MCP server
- [MCP Protocol](https://modelcontextprotocol.io/) — Learn about Model Context Protocol
