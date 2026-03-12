# NarrateAI MCP — Claude Desktop Project Knowledge

Copy this into your Claude Desktop Project as "Project Knowledge" (Project Settings → Add Knowledge).

## What This Does

NarrateAI lets you narrate silent videos, transcribe speech, translate transcripts, dub videos with voice cloning, and generate documents from videos — all through conversation.

## MCP Server Setup

Install: `pip install narrateai-mcp`

Add to Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on Mac, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):

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

Get your API key at narrateai.app. Restart Claude Desktop after adding.

## How to Use

Just ask naturally:
- "Narrate this video with the female1 voice: https://example.com/demo.mp4"
- "Transcribe this Spanish podcast: podcast.mp4"
- "Dub this English video into French, keep the background music"
- "Create a tutorial guide from this screencast"
- "Translate my existing Messi video to Hindi and re-narrate it"

## Available Voices

When narrating, choose a voice:
- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Available Tools

| Tool | What it does |
|------|-------------|
| `generate_narration_script` | AI script for a silent video (text only) |
| `narrate_video_full` | Full narrated video with voiceover |
| `continue_to_full_video` | Turn a script into a narrated video |
| `transcribe_video` | Speech-to-text from video with audio |
| `translate_video` | Transcribe + translate a new video |
| `translate_existing_video` | Translate a previously processed video |
| `dub_video_full` | Voice-cloned dubbing into another language |
| `generate_document` | Guide/tutorial/docs from a video |
| `list_videos` | Browse your video library |
| `update_transcript` | Edit script before narrating |
| `get_job_result` | Check processing status |
| `narrate_batch` | Narrate up to 5 videos at once |
| `batch_transcribe` | Transcribe multiple videos |
| `batch_dub` | Dub multiple videos |

## Important Rules

1. **Always ask for context** before narrating: "What does this video show?" — improves quality dramatically.
2. **Always ask for voice** before narrating — don't default silently.
3. **Always ask for language** before transcribing — don't guess.
4. **Always ask about background music** before dubbing.
5. When a tool returns `video_url`, present it as a clickable link.
6. Preserve `job_id` and `db_job_id` — the user may need them.
7. Do NOT use these tools for video editing, trimming, or compression.

## Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko

## Document Types

When generating documents, ask which type: `user_onboarding`, `tutorial_guide`, `feature_showcase`, `business_overview`, `product_documentation`.
