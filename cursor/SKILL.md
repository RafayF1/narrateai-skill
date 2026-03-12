---
name: narrateai-video-workflows
description: >
  NarrateAI video processing workflows: narration, transcription, translation, dubbing,
  document generation, transcript editing, and video library via the NarrateAI MCP server.
  Use when user wants to narrate a silent video, add voiceover, transcribe speech, translate
  a transcript, dub a video, generate a timed script, generate a document/article/guide from
  a video, edit a narration script, browse their video library, or check on a processing job.
  Requires the narrateai MCP server to be connected. Do NOT use for general video editing,
  trimming, compression, format conversion, or non-NarrateAI tasks.
metadata:
  author: NarrateAI
  version: 1.6.0
  mcp-server: narrateai
  category: video-processing
---

# NarrateAI Video Workflows — Cursor

## Setup

1. Install: `pip install narrateai-mcp`
2. Add to `.cursor/mcp.json`:

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

3. Copy this `SKILL.md` to `.cursor/skills/narrateai-video-workflows/SKILL.md` in your project.
4. Restart Cursor.

## How to Use

With the MCP server connected and this skill installed, just ask naturally:
- "Narrate this video with the chatterbox voice: demo.mp4"
- "Transcribe this Spanish meeting recording"
- "Dub this into French, keep the background music"
- "Generate a product guide from this screencast"

Cursor will automatically use the NarrateAI tools. See `workflows.md` in the repo root for the full workflow reference.

---

<!-- The full workflow reference is included below so Cursor's skill system has all context in a single file. -->

## Important

- **Stdio mode (local Cursor):** All processing tools poll internally (every 25s, up to 15 minutes). Do NOT manually poll unless the tool returns `status: timeout`.
- **HTTP/remote mode:** Tools return a `job_id` immediately. Poll with `get_job_result(job_id)` every 25 seconds.
- Always preserve and return `job_id` and `db_job_id` from responses.
- When a tool returns `video_url`, present it as a clickable download link.
- **Local files in HTTP/remote mode:** Use `get_upload_url` first, upload via curl, then pass `temp_file_path` as `video_source`.

## Available Voices

When a voice is needed and the user hasn't specified one, ask them to choose:
- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko — and others via Whisper fallback.

---

## Workflow 1: Generate a Narration Script (Text Only)

**Trigger:** User wants a timed narration script from a silent video. NOT for extracting existing speech — use Workflow 3.

**Tool:** `generate_narration_script`

1. Get the video source (URL or local file path).
2. Ask for language if not English (default: en).
3. **MUST ASK:** "Can you describe what this video shows? Context helps the AI write a much better narration script."
4. Call `generate_narration_script` with `video_source`, `language`, `manual_context`.
5. Present the transcript segments.
6. Ask: "Would you like to edit any segments?" If yes → `update_transcript`. Ask: "Continue to full narrated video?" If yes → Workflow 2b.

## Workflow 2a: Narrate a Silent Video (Full Video, One Shot)

**Tool:** `narrate_video_full`

1. Get the video source.
2. Ask for language (default: en).
3. Ask which voice (see Available Voices). Do not skip.
4. **MUST ASK:** "Can you describe what this video shows?"
5. Call `narrate_video_full` with `video_source`, `language`, `voice_type`, `manual_context`.
6. Present `video_url` as a download link.

## Workflow 2b: Transcript to Full Video (Two-Step)

**Tool:** `continue_to_full_video`

1. Need the `job_id` from Workflow 1.
2. **MUST ASK:** "Which voice would you like?"
3. Call `continue_to_full_video` with `job_id`, `voice_type`, `db_job_id`.
4. Present `video_url`.

## Workflow 3: Transcribe a Video with Existing Speech

**Tool:** `transcribe_video`

1. Get the video source.
2. Ask what language is spoken. Required — do not guess.
3. Call `transcribe_video` with `video_source`, `source_language`.
4. Present transcript segments.

## Workflow 4a: Translate a Video Transcript (New Upload)

**Tool:** `translate_video`

1. Get video source, source language, target language (all required).
2. Call `translate_video`.
3. Present translated transcript.

## Workflow 4b: Translate an Existing Video's Transcript

**Tool:** `translate_existing_video`

1. Get `job_id`, source language, target language.
2. Call `translate_existing_video`. Returns immediately (synchronous).
3. Present translated transcript.

## Workflow 5: Dub a Video into Another Language

**Tool:** `dub_video_full`

1. Get video source, source language, target language (all required).
2. Ask: "Keep background music?" Required — do not assume.
3. Call `dub_video_full`.
4. Present `video_url`.

## Workflow 6: Generate a Document from a Video

**Tool:** `generate_document`

Document types: `user_onboarding`, `tutorial_guide`, `feature_showcase`, `business_overview`, `product_documentation`.

1. Get video source.
2. Ask which document type.
3. Ask for language, optional context.
4. Call `generate_document`.
5. Present `document_markdown`. Offer the synced transcript if available.

## Workflow 7–10: Batch Operations

| Workflow | Tool | Max videos |
|----------|------|-----------|
| 7: Batch Narration | `narrate_batch` | 5 |
| 8: Batch Scripts | `batch_generate_scripts` | 5 |
| 9: Batch Transcription | `batch_transcribe` | 5 |
| 10: Batch Dubbing | `batch_dub` | 5 |

For batch narration: ask voice once, ask about context (same for all / per-video / skip). Pass `video_sources_json`, and `contexts_json` if per-video.

## Workflow 11: Edit Transcript Before Narration

**Tool:** `update_transcript`

1. User describes changes naturally. Apply them to the transcript array.
2. Call `update_transcript` with `job_id` and full `transcript_json` (all segments, not just changed ones).
3. Ask: "More changes, or continue to video?" → Workflow 2b.

## Workflow 12: Browse Video Library

**Tool:** `list_videos`

1. Call `list_videos` (supports `page`, `per_page`).
2. Present: filename, status, language, date.
3. Use the `id` as `job_id` for other tools.

## Workflow 13: Translate & Re-Narrate an Existing Video

**Tools:** `list_videos` → `translate_existing_video` → `update_transcript` → `continue_to_full_video`

1. Find the video with `list_videos`.
2. Translate with `translate_existing_video`.
3. Offer to edit. Save with `update_transcript` (`reset_for_reprocessing=True`).
4. **MUST ASK:** "Which voice?"
5. Generate with `continue_to_full_video`.
6. Present `video_url`.

---

## Decision Guide

| User wants... | Workflow |
|---|---|
| Narration script (text only) | 1 |
| Narrated video with voiceover | 2a |
| Turn transcript into video | 2b |
| Speech-to-text | 3 |
| Translated transcript (new video) | 4a |
| Translate existing video's transcript | 4b |
| Dubbed video | 5 |
| Document/guide from video | 6 |
| Narrate multiple videos | 7 |
| Scripts for multiple videos | 8 |
| Transcribe multiple videos | 9 |
| Dub multiple videos | 10 |
| Edit script before narrating | 11 |
| Browse video library | 12 |
| Translate & re-narrate | 13 |
