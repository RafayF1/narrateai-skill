# NarrateAI MCP — Claude Code

Drop this file in your project root as `CLAUDE.md`. When the NarrateAI MCP server is connected, Claude Code will follow these instructions automatically.

## MCP Server Setup

```bash
pip install narrateai-mcp
```

Add to your MCP config (`.mcp.json` in project root or `~/.claude/mcp.json` globally):

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

Get your API key at [narrateai.app](https://narrateai.app).

---

## What NarrateAI Does

NarrateAI adds AI video narration, transcription, translation, and dubbing to your workflow. All tools are available through the MCP server.

## Rules

- All processing tools poll internally in stdio mode. Do NOT manually poll unless `status: timeout`.
- Always preserve and return `job_id` and `db_job_id`.
- When a tool returns `video_url`, present it as a clickable download link.
- Do NOT use NarrateAI tools for video editing, trimming, compression, or format conversion.

## Available Voices

Ask the user to choose when a voice is needed:
- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko

## Tools & When to Use Them

### Narration (silent video → voiceover)
- `generate_narration_script` — Text-only script. **MUST ASK** for video context before calling.
- `narrate_video_full` — Full narrated video in one shot. **MUST ASK** for voice and context.
- `continue_to_full_video` — Turn an existing script into a narrated video. **MUST ASK** for voice.
- `narrate_batch` — Up to 5 videos at once.
- `batch_generate_scripts` — Scripts for multiple videos.

### Transcription (video with speech → text)
- `transcribe_video` — Speech-to-text. **MUST ASK** for source language (do not guess).
- `batch_transcribe` — Multiple videos.

### Translation
- `translate_video` — Transcribe + translate a new video.
- `translate_existing_video` — Translate a previously processed video's transcript (synchronous).

### Dubbing (voice cloning)
- `dub_video_full` — Clone speaker's voice into another language. **MUST ASK** about background music.
- `batch_dub` — Multiple videos.

### Documents
- `generate_document` — Generate guides, tutorials, docs from a video. Types: `user_onboarding`, `tutorial_guide`, `feature_showcase`, `business_overview`, `product_documentation`. **MUST ASK** which type.

### Library & Editing
- `list_videos` — Browse user's past videos (paginated).
- `update_transcript` — Edit script segments before narrating. Send ALL segments, not just changed ones.

### Utilities
- `get_job_result` — Check job status.
- `get_upload_url` — Get signed URL for uploading local files (remote mode only).
- `abandon_job` — Cancel a running job.

## Key Workflows

**Narrate a silent video:**
Get video → ask for context → ask for voice → `narrate_video_full` → return `video_url`

**Transcribe speech:**
Get video → ask for language → `transcribe_video` → return transcript

**Dub into another language:**
Get video → ask source/target language → ask about background music → `dub_video_full` → return `video_url`

**Script → edit → narrate (two-step):**
`generate_narration_script` → show transcript → offer edits → `update_transcript` if edited → ask for voice → `continue_to_full_video`

**Translate & re-narrate an existing video:**
`list_videos` → `translate_existing_video` → offer edits → `update_transcript` with `reset_for_reprocessing=True` → ask for voice → `continue_to_full_video`

**Generate docs from a video:**
Get video → ask document type → `generate_document` → return markdown. Offer synced transcript after.

## Common Errors

| Error | Fix |
|-------|-----|
| "API key required" | Set `NARRATEAI_API_KEY` in MCP config |
| "Video file not found" | Verify file path |
| 429 / Rate limit | User needs more credits at narrateai.app |
