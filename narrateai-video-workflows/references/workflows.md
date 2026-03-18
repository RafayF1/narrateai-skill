# NarrateAI MCP — Workflow Reference

This document describes every workflow available through the NarrateAI MCP server. It is platform-agnostic — the same tools and logic apply whether you're using Cursor, Claude, Windsurf, or any other MCP-compatible AI assistant.

## Before You Start

- **Stdio mode (local):** Processing tools poll internally (every 25s, up to 15 min). Do NOT manually poll unless the tool returns `status: timeout`.
- **HTTP/remote mode:** Processing tools return a `job_id` immediately. You MUST poll — see Polling Rules below.
- Always preserve and return `job_id` and `db_job_id` from responses — users may need them later.
- When a tool returns `video_url`, present it as a clickable download link.
- **Local files in HTTP/remote mode:** The server cannot access local files. Use `get_upload_url` first, upload via curl, then pass the returned `temp_file_path` as `video_source`.

### Polling Rules (Critical — Read This)

Video processing takes **1–5 minutes** depending on video length and processing type.

1. **Poll with `get_job_result(job_id)` every 20–30 seconds.** Do not stop after one or two polls.
2. **Poll at least 6–8 times** (covering ~3 minutes) before considering a job stuck or failed.
3. **A job at 10–30% progress is NORMAL.** It means the video was downloaded and is being processed. Do NOT interpret low progress as an error, download failure, or source URL issue.
4. **NEVER diagnose source/download problems from progress percentage alone.** If `status` is `processing`, the video is being worked on successfully. Only report failure when `status` is explicitly `failed` with an error message.
5. **NEVER suggest the user re-upload, switch hosting providers, or use a different URL** while a job is still `processing`. Wait for completion or explicit failure.
6. **If a poll returns `status: processing`**, say "Still processing — I'll check again shortly" and poll again. Do not speculate about problems or offer alternatives.
7. Only report failure if `status: failed` with an error message, OR after 10+ polls (~5 minutes) with zero progress change.

## Available Voices

When a voice is needed and the user hasn't specified one, ask them to choose:

**AI Voices:**
- **male1** (default, fastest generation)
- **female1** (default, fastest generation)
- **female2**, **female3**, **female4**
- **male2**, **male3**

**Voice Cloning:**
Users can provide an audio file (local path or URL) for voice cloning. Ask: "Would you like an AI voice or your own voice sample for cloning?"

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko — and others via Whisper fallback.

---

## Workflow 1: Generate a Narration Script (Text Only)

**Trigger:** User wants a timed narration script from a silent video. NOT for extracting existing speech — use Workflow 3 for that.

**Tool:** `generate_narration_script`

### Steps

1. Get the video source from the user (URL or local file path).
2. Ask for language if not English (default: en).
3. **MUST ASK:** "Can you describe what this video shows? Context helps the AI write a much better narration script." Do not skip this.
4. Call `generate_narration_script` with `video_source`, `language`, and `manual_context`.
5. Present the returned transcript segments (each has `start_time`, `end_time`, `text`).

### After Completion

1. Ask: "Would you like to edit any of the narration segments before continuing?" If yes, make changes through conversation, then call `update_transcript` with the edited segments and the `job_id`.
2. Ask: "Would you like to turn this into a full narrated video with voiceover?" If yes, proceed to Workflow 2b.

---

## Workflow 2a: Narrate a Silent Video (Full Video, One Shot)

**Trigger:** User explicitly wants a narrated video file with AI voiceover from a silent video.

**Tool:** `narrate_video_full`

### Steps

1. Get the video source (URL or local path).
2. Ask for language if not specified (default: en).
3. Ask which voice they want (see Available Voices). Do not skip this.
4. **MUST ASK:** "Can you describe what this video shows? Context helps the AI write a much better narration." Do not skip this.
5. Call `narrate_video_full` with `video_source`, `language`, `voice_type`, and `manual_context`.
6. When completed, present the `video_url` as a download link and show transcript summary.

---

## Workflow 2b: Transcript to Full Video (Two-Step)

**Trigger:** User has a transcript (from Workflow 1 or a previous session) and wants the full narrated video.

**Tool:** `continue_to_full_video`

### Steps

1. You need the `job_id` from the transcript step.
2. **MUST ASK:** "Which voice would you like?" (see Available Voices). Do not skip this.
3. Call `continue_to_full_video` with `job_id`, `voice_type`, and `db_job_id`.
4. When completed, present the `video_url` as a download link.

---

## Workflow 3: Transcribe a Video with Existing Speech

**Trigger:** User has a video with spoken audio (podcast, interview, meeting) and wants speech-to-text.

**Tool:** `transcribe_video`

### Steps

1. Get the video source.
2. Ask what language is spoken. Required — do not guess, do not default to English.
3. Call `transcribe_video` with `video_source` and `source_language`.
4. Present the transcript segments.

**Key Distinction:** Workflow 1 generates AI narration for silent videos. This workflow extracts existing speech.

---

## Workflow 4a: Translate a Video Transcript (New Upload)

**Trigger:** User uploads a video with speech and wants a translated transcript.

**Tool:** `transcribe_and_translate`

### Steps

1. Get the video source.
2. Ask for source language (spoken in the video). Required.
3. Ask for target language. Required.
4. Call `transcribe_and_translate` with `video_source`, `source_language`, `target_language`.
5. Present the translated transcript segments.

---

## Workflow 4b: Translate an Existing Video's Transcript

**Trigger:** User wants to translate a previously processed video's transcript (has a job_id).

**Tool:** `translate_existing_video`

### Steps

1. Get the `job_id` of the completed video.
2. Ask for source language of the current transcript.
3. Ask for target language.
4. Call `translate_existing_video` with `job_id`, `source_language`, `target_language`.
5. Returns immediately (synchronous). Present the translated transcript.

---

## Workflow 5: Dub a Video into Another Language

**Trigger:** User wants a dubbed video — original speaker's voice cloned and speaking another language.

**Tool:** `dub_video_full`

### Steps

1. Get the video source.
2. Ask for source language. Required.
3. Ask for target language. Required.
4. Ask: "Does the video have background music you want to keep?" Required — do not assume.
5. Call `dub_video_full` with all four parameters.
6. When completed, present the `video_url` as a download link.

---

## Workflow 6: Generate a Document from a Video

**Trigger:** User wants a written document, guide, tutorial, or docs from a video.

**Tool:** `generate_document`

### Document Types

Ask the user which type before calling:
- **user_onboarding** — Step-by-step onboarding guide (default)
- **tutorial_guide** — Tutorial/how-to guide
- **feature_showcase** — Feature showcase document
- **business_overview** — Business overview document
- **product_documentation** — Product documentation

### Steps

1. Get the video source.
2. Ask which document type. Do not skip this.
3. Ask for language if not English (default: en).
4. Optionally ask for context.
5. Call `generate_document` with `video_source`, `document_type`, `language`, `manual_context`.
6. When completed (`status: document_ready`), present the `document_markdown`.

### After Completion

The response includes `has_synced_transcript: true` and a `transcript` array. Offer: "I also have a synced transcript for this video — would you like it?"

---

## Workflow 7: Batch Narration (Multiple Videos)

**Trigger:** User has multiple videos to narrate at once.

**Tool:** `narrate_batch`

### Steps

1. Collect all video sources (max 5).
2. Ask which voice — once, applies to all.
3. Ask for language if not English.
4. Ask about context: "(a) same for all, (b) different per video, or (c) skip?"
   - Same: pass as `manual_context`.
   - Different: pass as `contexts_json` — JSON array matching video order, `""` for no context.
5. Call `narrate_batch` with `video_sources_json`, `voice_type`, `language`, and context.
6. **Stdio:** Returns final results with `video_url` per video.
7. **HTTP:** Returns `job_id` per video. Poll each with `get_job_result` until `completed`. Do NOT call `continue_to_full_video`.
8. Present results per video.

---

## Workflow 8: Batch Script Generation

**Trigger:** Multiple silent videos, scripts only (no audio).

**Tool:** `batch_generate_scripts`

### Steps

1. Collect video sources (max 5).
2. Ask for language.
3. Ask about context (same as Workflow 7).
4. Call `batch_generate_scripts`.
5. Present transcript segments per video.

---

## Workflow 9: Batch Transcription

**Trigger:** Multiple videos with existing speech to transcribe.

**Tool:** `batch_transcribe`

### Steps

1. Collect video sources (max 5).
2. Ask what language is spoken. Required.
3. Call `batch_transcribe` with `video_sources_json` and `source_language`.
4. Present transcript per video.

---

## Workflow 10: Batch Dubbing

**Trigger:** Multiple videos to dub.

**Tool:** `batch_dub`

### Steps

1. Collect video sources (max 5).
2. Ask for source language, target language, and preserve background music. All required.
3. Call `batch_dub` with all parameters.
4. Present `video_url` per video.

---

## Workflow 11: Edit Transcript Before Narration

**Trigger:** User has a script from Workflow 1 and wants to edit before narrating.

**Tool:** `update_transcript`

### Steps

1. You have the transcript and `job_id` from Workflow 1.
2. User describes changes naturally ("change the second segment to...", "make the intro more exciting").
3. Apply changes to the transcript array yourself.
4. Call `update_transcript` with `job_id` and the full `transcript_json` (all segments, not just changed ones).
5. Ask: "More changes, or continue to the full narrated video?"
6. If continuing, proceed to Workflow 2b.

---

## Workflow 12: Browse Video Library

**Trigger:** User wants to see past videos, find a specific one, or act on an existing video.

**Tool:** `list_videos`

### Steps

1. Call `list_videos` to get the library (supports `page` and `per_page` for pagination).
2. Present results: filename, status, language, date.
3. Use the `id` from results as `job_id` for other tools (translate, check status, etc.).

---

## Workflow 13: Translate & Re-Narrate an Existing Video

**Trigger:** User has a narrated video and wants it re-narrated in another language with a chosen AI voice. Different from dubbing (Workflow 5) which clones the original speaker's voice.

**Tools:** `list_videos` → `translate_existing_video` → `update_transcript` → `continue_to_full_video`

### Steps

1. `list_videos` — find the video, get its `id`.
2. `translate_existing_video` — translate the transcript.
3. Present the translated transcript.
4. Ask: "Edit any segments before narrating?" If yes, edit through conversation.
5. `update_transcript` — save segments with `reset_for_reprocessing=True`.
6. **MUST ASK:** "Which voice?" Do not skip.
7. `continue_to_full_video` — generate the video.
8. Present `video_url`.

---

## Video Source Options

All processing tools accept a `video_source`. Supported formats:

| Source | Format | Notes |
|--------|--------|-------|
| Direct URL | `https://example.com/video.mp4` | Any publicly accessible URL |
| Google Drive | `https://drive.google.com/uc?export=download&id=FILE_ID` | Convert share link: extract `FILE_ID` from `drive.google.com/file/d/FILE_ID/view` |
| Dropbox | `https://dropbox.com/...?dl=1` | Change `?dl=0` to `?dl=1` for direct download |
| Local file | `/path/to/video.mp4` | Stdio mode only |
| Temp upload | `temp/user-id/filename.mp4` | Use upload flow below for remote mode |

## Uploading Local Files (Remote Mode)

When the MCP server runs remotely and the user provides a local file:

1. Call `get_upload_url(filename)` → returns `upload_url` and `temp_file_path`.
2. Upload: `curl -X PUT -H 'Content-Type: video/mp4' --upload-file '/path/to/video.mp4' '<upload_url>'`
3. Pass `temp_file_path` as `video_source` to any processing tool.

Not needed in stdio mode — tools read local files directly.

### Claude.ai / Web Sandbox Limitation

Web-based AI environments (like Claude.ai) block outgoing requests to `storage.googleapis.com`, so the curl upload cannot be run from within the sandbox.

**Workaround A — User runs curl locally:**
1. Call `get_upload_url(filename)` — the MCP call works fine.
2. Give the user the exact curl command to paste in their terminal.
3. Tell them: "Run this in your terminal. No output means success."
4. Once confirmed, proceed with `temp_file_path` as `video_source`.

**Workaround B — Public URL (easier):**
Ask the user to share the video via Google Drive, Dropbox, or any file host and provide the public URL. Pass that URL directly as `video_source` — no upload step needed.

---

## Handling Timeouts

If any tool returns `status: timeout`:
1. Tell the user the video is still processing.
2. Use `get_job_result(job_id)` to check.
3. Statuses: `processing`, `transcript_ready`, `document_ready`, `completed`, `failed`.

## Cancelling a Job

Call `abandon_job(job_id)`.

## Common Errors

| Error | Fix |
|-------|-----|
| "API key required" | Set `NARRATEAI_API_KEY` in MCP config |
| "Video file not found" | Verify the file path with the user |
| 429 / Rate limit | User hit their credit limit — purchase more at narrateai.app |
| "has_active_job" | Handled automatically — old job is abandoned |

---

## Decision Guide

| User wants... | Workflow |
|---|---|
| Narration script from a silent video (text only) | 1 |
| Narrated video with voiceover from a silent video | 2a |
| Turn an existing transcript into a video | 2b |
| Speech-to-text from a video with audio | 3 |
| Translated transcript from a new video | 4a |
| Translate an already-processed video's transcript | 4b |
| Fully dubbed video in another language | 5 |
| Document, article, guide from a video | 6 |
| Narrate multiple videos at once | 7 |
| Scripts for multiple silent videos | 8 |
| Transcribe multiple videos at once | 9 |
| Dub multiple videos at once | 10 |
| Edit narration script before making the video | 11 |
| Browse past videos (video library) | 12 |
| Translate & re-narrate with a chosen voice | 13 |
