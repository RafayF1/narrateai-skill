# NarrateAI Skill

An AI Skill that teaches agents how to use [NarrateAI](https://narrateai.app) for video processing — narration, transcription, translation, dubbing, and document generation. Works with Cursor, Claude, and any MCP-compatible agent.

## What is NarrateAI?

NarrateAI turns silent videos into narrated content using AI. Upload a screen recording, product demo, or tutorial — and get back a fully narrated video with AI voiceover, a synced transcript, translated subtitles, dubbed audio in another language, or a structured document.

## What This Skill Does

This skill gives your AI agent step-by-step instructions for all NarrateAI workflows:

| Workflow | What it does |
|---|---|
| **Narrate (transcript)** | Generate a timed narration script from a silent video |
| **Narrate (full video)** | Produce a narrated video with AI voiceover |
| **Transcribe** | Extract speech-to-text from a video with existing audio |
| **Translate** | Translate a video's transcript to another language |
| **Dub** | Clone the speaker's voice and dub into another language |
| **Generate document** | Create a tutorial, guide, or article from a video |

## Setup

### 1. Get an API Key

Sign up at [narrateai.app](https://narrateai.app) and purchase an API credits package from the dashboard.

### 2. Install the MCP Server

Add this to your MCP configuration:

**Cursor** (Settings → Tools & MCP → New MCP Server):

```json
{
  "mcpServers": {
    "narrateai": {
      "command": "uvx",
      "args": [
        "narrateai-mcp"
      ],
      "env": {
        "NARRATEAI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "narrateai": {
      "command": "uvx",
      "args": ["narrateai-mcp"],
      "env": {
        "NARRATEAI_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

### 3. Add the Skill

**For Cursor:** Copy the `.cursor/skills/` folder from this repo into your project:

```bash
cp -r .cursor/skills/narrateai-video-workflows /path/to/your/project/.cursor/skills/
```

Cursor will automatically discover the skill and use it when relevant.

**For Claude:** Place the `SKILL.md` file in your project root or reference it in your Claude project instructions.

## Available Tools

Once the MCP server is connected, your agent has access to these tools:

| Tool | Description |
|---|---|
| `narrate_video_transcript` | Transcript only — timed script from a silent video |
| `narrate_video_full` | Full narrated video with AI voiceover |
| `continue_to_full_video` | Continue from transcript to full video (two-step) |
| `transcribe_video` | Speech-to-text from a video with existing audio |
| `translate_video` | Translate a video transcript (new upload) |
| `translate_existing_video` | Translate an already-processed video's transcript |
| `dub_video_full` | Full auto-dubbing with voice cloning |
| `generate_document` | Generate a markdown document from a video |
| `get_job_result` | Check status of a running job |
| `abandon_job` | Cancel a running job |

## Supported Voices

- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko — and others via Whisper fallback.

## Example Usage

Ask your AI agent:

- *"Narrate this screen recording with a female voice"*
- *"Transcribe this podcast — it's in Spanish"*
- *"Dub this English video into French"*
- *"Generate a tutorial guide from this demo video"*

The agent will use the skill to pick the right tool, ask for any missing info (voice, language, document type), and handle the entire workflow.

## Links

- [NarrateAI App](https://narrateai.app)
- [MCP Server on PyPI](https://pypi.org/project/narrateai-mcp/)
- [Cursor Documentation](https://docs.cursor.com)
- [Claude MCP Guide](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)

## License

MIT
