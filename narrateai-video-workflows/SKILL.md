---
name: narrateai-video-workflows
description: >
  NarrateAI video processing workflows via MCP: narrate silent videos with AI voiceover,
  transcribe speech-to-text, translate transcripts, dub videos with voice cloning, and
  generate documents from screencasts. Use when user wants to "narrate this video",
  "add voiceover", "transcribe this meeting", "translate this video", "dub into French",
  "generate a guide from this screencast", "create product docs from video", "edit the
  narration script", "show my videos", or "re-narrate in another language".
  Requires the narrateai MCP server to be connected.
  Do NOT use for general video editing, trimming, compression, format conversion,
  or non-NarrateAI tasks.
metadata:
  author: NarrateAI
  version: 2.0.0
  mcp-server: narrateai
  category: video-processing
  tags: [video, narration, transcription, translation, dubbing, voiceover, mcp]
  documentation: https://narrateai.app
---

# NarrateAI Video Workflows

## Important

- **Stdio mode (local):** Processing tools poll internally. Do NOT manually poll unless the tool returns `status: timeout`.
- **HTTP/remote mode:** Tools return a `job_id` immediately. Poll with `get_job_result(job_id)` every 25 seconds until complete.
- Always preserve and return `job_id` and `db_job_id` from responses — users need them for follow-up actions.
- When a tool returns `video_url`, present it as a clickable download link.
- **Local files in remote mode:** Use `get_upload_url` first, upload via curl, then pass `temp_file_path` as `video_source`.

For detailed workflow steps and edge cases, consult `references/workflows.md`.

## Available Voices

Ask the user to choose when a voice is needed — do not default silently:

**AI Voices:**
- **male1** (default, fastest generation)
- **female1** (default, fastest generation)
- **female2**, **female3**, **female4**
- **male2**, **male3**

**Voice Cloning:**
Users can provide their own audio file (local path or URL) for voice cloning. When a voice sample is provided, the narration uses the cloned voice instead of an AI preset. Ask: "Would you like to use an AI voice or provide your own voice sample for cloning?"

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko — and others via Whisper fallback.

## Decision Guide

| User wants... | Tool(s) | Key requirements |
|---|---|---|
| Narration script from a silent video (text only) | `generate_narration_script` | MUST ASK for video context |
| Narrated video with AI voiceover | `narrate_video_full` | MUST ASK for voice (AI or clone) and context |
| Turn an existing script into a video | `continue_to_full_video` | MUST ASK for voice (AI or clone) |
| Speech-to-text from video with audio | `transcribe_video` | MUST ASK for source language |
| Translated transcript (new video) | `transcribe_and_translate` | Source and target language required |
| Translate existing video's transcript | `translate_existing_video` | Synchronous, no polling |
| Dubbed video (voice cloning) | `dub_video_full` | MUST ASK about background music |
| Document/guide/tutorial from video | `generate_document` | MUST ASK for document type |
| Narrate multiple videos | `narrate_batch` | Max 5, ask voice once |
| Scripts for multiple videos | `batch_generate_scripts` | Max 5 |
| Transcribe multiple videos | `batch_transcribe` | Max 5 |
| Dub multiple videos | `batch_dub` | Max 5 |
| Edit narration before producing video | `update_transcript` | Send ALL segments |
| Browse past videos | `list_videos` | Paginated |
| Translate and re-narrate existing video | `list_videos` → `translate_existing_video` → `update_transcript` → `continue_to_full_video` | Use `reset_for_reprocessing=True` |

## Critical Rules

1. **Always ask for context** before narrating: "Can you describe what this video shows?" This dramatically improves narration quality. Never skip this.
2. **Always ask for voice** before narrating: "Would you like an AI voice (male1, female1, etc.) or your own voice sample for cloning?" Do not silently default.
3. **Always ask for source language** before transcribing. Do not guess.
4. **Always ask about background music** before dubbing. Do not assume.
5. **Always ask for document type** before generating documents.

## Workflow Quick Reference

### Narrate a Silent Video

1. Get video source (URL or local path).
2. Ask for language (default: en).
3. **MUST ASK:** "Can you describe what this video shows?"
4. **MUST ASK:** "Would you like an AI voice (male1, female1, etc.) or your own voice sample for cloning?"
5. Call `narrate_video_full` with `video_source`, `language`, `voice_type` (or `voice_sample` if cloning), `manual_context`.
6. Present `video_url` as a download link.

### Two-Step: Script Then Video

1. Call `generate_narration_script` (ask for context first).
2. Present transcript segments to user.
3. Offer to edit: "Would you like to change any segments?" If yes, apply changes and call `update_transcript` with full transcript.
4. Ask: "AI voice or your own voice sample?" Then call `continue_to_full_video` with `voice_type` or `voice_sample`.

### Transcribe Speech

1. Get video source.
2. **MUST ASK:** "What language is spoken?" (do not guess, do not default)
3. Call `transcribe_video`.

### Translate

- **New video:** `transcribe_and_translate` with source + target language.
- **Existing video:** `translate_existing_video` with `job_id` — returns immediately.

### Dub

1. Get source language, target language.
2. **MUST ASK:** "Keep background music?"
3. Call `dub_video_full`.

### Generate Document

Types: `user_onboarding`, `tutorial_guide`, `feature_showcase`, `business_overview`, `product_documentation`.

1. Get video source.
2. **MUST ASK:** which document type.
3. Call `generate_document`.
4. Present `document_markdown`. Offer synced transcript if available.

### Translate and Re-Narrate Existing Video

1. `list_videos` — find the video.
2. `translate_existing_video` — translate transcript.
3. Offer to edit segments.
4. `update_transcript` with `reset_for_reprocessing=True`.
5. **MUST ASK:** which voice.
6. `continue_to_full_video`.

### Batch Operations

All batch tools accept `video_sources_json` (max 5 videos). For `narrate_batch`, ask voice once and ask about context: same for all, per-video (`contexts_json`), or skip.

## Uploading Local Files (Remote Mode)

1. Call `get_upload_url(filename)` → returns `upload_url` and `temp_file_path`.
2. Upload: `curl -X PUT -H 'Content-Type: video/mp4' --upload-file 'video.mp4' '<upload_url>'`
3. Pass `temp_file_path` as `video_source`.

## Handling Timeouts

If `status: timeout`, use `get_job_result(job_id)`. Statuses: `processing`, `transcript_ready`, `document_ready`, `completed`, `failed`.

## Cancelling

Call `abandon_job(job_id)`.

## Common Errors

| Error | Fix |
|-------|-----|
| "API key required" | Set `NARRATEAI_API_KEY` in MCP config |
| "Video file not found" | Verify file path with user |
| 429 / Rate limit | User needs more credits at narrateai.app |
| "has_active_job" | Handled automatically — old job abandoned |

## Examples

### Example 1: Narrate a screencast

User: "I have a screen recording of our app demo, no voice. Can you narrate it?"

1. Ask: "What language?" → en
2. Ask: "Can you describe what the video shows?" → "It's a walkthrough of our new dashboard"
3. Ask: "AI voice or your own voice sample?" → "Use male1"
4. Call `narrate_video_full` with `voice_type=male1`, context
5. Return `video_url`

### Example 2: Transcribe a meeting

User: "Transcribe this meeting: meeting.mp4"

1. Ask: "What language is spoken?" → Spanish
2. Call `transcribe_video` with `source_language=es`
3. Present transcript

### Example 3: Dub into French

User: "Dub this English video into French"

1. Ask: "Keep background music?" → yes
2. Call `dub_video_full` with `source_language=en`, `target_language=fr`, `preserve_background_music=true`
3. Return `video_url`

### Example 4: What this skill is NOT for

- "Trim the first 10 seconds" → NOT NarrateAI (video editing)
- "Convert to MP4" → NOT NarrateAI (format conversion)
- "Compress this video" → NOT NarrateAI
