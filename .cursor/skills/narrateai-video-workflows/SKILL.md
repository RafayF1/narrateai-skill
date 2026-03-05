---
name: narrateai-video-workflows
description: >
  NarrateAI video processing workflows: narration, transcription, translation, dubbing,
  and document generation via the NarrateAI MCP server. Use when user wants to narrate a
  silent video, add voiceover, transcribe speech from a video, translate a video transcript,
  dub a video into another language, generate a timed script, generate a document/article/guide
  from a video, or check on a video processing job. Requires the narrateai MCP server to be
  connected. Do NOT use for general video editing, trimming, compression, format conversion,
  or non-NarrateAI tasks.
metadata:
  author: NarrateAI
  version: 1.1.0
  mcp-server: narrateai
  category: video-processing
---

# NarrateAI Video Workflows

## Important

- All processing tools poll internally (every 25s, up to 15 minutes). Do NOT manually poll with `get_job_result` unless the tool returns `status: timeout`.
- When a tool returns `status: timeout`, tell the user the job is still running and use `get_job_result(job_id)` to check back.
- Always preserve and return the `job_id` and `db_job_id` from responses — users may need them later.
- When a tool returns `video_url`, present it as a clickable download link.

## Available Voices

When a voice is needed and the user has not specified one, ask them to choose:
- **chatterbox** (default) — natural, expressive
- **female1**, **female2**, **female3**, **female4**
- **male2**, **male3**

## Supported Languages

en, es, fr, de, it, pt, ru, zh, yue, ja, ko — and others via Whisper fallback.

---

## Workflow 1: Narrate a Silent Video (Transcript Only)

**Trigger:** User wants a timed script, transcript, or text-only narration from a silent video (no spoken audio in the source).

**Tool:** `narrate_video_transcript`

### Steps

1. Get the video source from the user (URL or local file path).
2. Ask for language if not English (default: en).
3. Optionally ask if they want to provide context about the video for better narration.
4. Call `narrate_video_transcript` with `video_source`, `language`, and `manual_context`.
5. Present the returned transcript segments to the user (each segment has `start_time`, `end_time`, `text`).

### After Completion

Ask the user: "Would you like to turn this transcript into a full narrated video with voiceover?" If yes, proceed to Workflow 2b.

---

## Workflow 2a: Narrate a Silent Video (Full Video, One Shot)

**Trigger:** User explicitly wants a narrated video file with AI voiceover from a silent video.

**Tool:** `narrate_video_full`

### Steps

1. Get the video source (URL or local path).
2. Ask for language if not specified (default: en).
3. Ask which voice they want (see Available Voices above). Do not skip this.
4. Optionally ask for context about the video.
5. Call `narrate_video_full` with `video_source`, `language`, `voice_type`, and `manual_context`.
6. When completed, present the `video_url` as a download link and show transcript summary.

---

## Workflow 2b: Transcript to Full Video (Two-Step)

**Trigger:** User has already received a transcript (from Workflow 1 or a previous session) and now wants the full narrated video.

**Tool:** `continue_to_full_video`

### Steps

1. You need the `job_id` from the transcript step.
2. Ask which voice they want if not specified.
3. Call `continue_to_full_video` with `job_id` and `voice_type`.
4. When completed, present the `video_url` as a download link.

---

## Workflow 3: Transcribe a Video with Existing Speech

**Trigger:** User has a video that already has spoken audio (podcast, interview, meeting, lecture) and wants speech-to-text.

**Tool:** `transcribe_video`

### Steps

1. Get the video source.
2. Ask what language is spoken in the video. This is required — do not guess, do not default to English. Ask the user.
3. Call `transcribe_video` with `video_source` and `source_language`.
4. Present the transcript segments.

### Key Distinction

This is different from Workflow 1. Workflow 1 generates narration for a silent video. This workflow extracts existing speech from a video that already has audio.

---

## Workflow 4a: Translate a Video Transcript (New Upload)

**Trigger:** User uploads a video with speech and wants a translated transcript (e.g., "translate this Spanish video to English").

**Tool:** `translate_video`

### Steps

1. Get the video source.
2. Ask for source language (language spoken in the video). Required.
3. Ask for target language (language to translate into). Required.
4. Call `translate_video` with `video_source`, `source_language`, `target_language`.
5. Present the translated transcript segments.

---

## Workflow 4b: Translate an Existing Video's Transcript

**Trigger:** User wants to translate the transcript of a video they already processed with NarrateAI (they have a job_id).

**Tool:** `translate_existing_video`

### Steps

1. Get the `job_id` of the completed video.
2. Ask for source language of the current transcript. For narrated videos, this is typically the narration language. For dubbed videos, this is the dubbed language.
3. Ask for target language.
4. Call `translate_existing_video` with `job_id`, `source_language`, `target_language`.
5. This returns immediately (synchronous, no polling needed). Present the translated transcript.

---

## Workflow 5: Dub a Video into Another Language

**Trigger:** User wants a fully dubbed video — the original speaker's voice cloned and speaking another language. Phrases like "dub this video", "make it speak French", "auto-dub".

**Tool:** `dub_video_full`

### Steps

1. Get the video source.
2. Ask for source language (what language is currently spoken). Required.
3. Ask for target language (what language to dub into). Required.
4. Ask: "Does the video have background music you want to keep?" Set `preserve_background_music` accordingly. Required — do not assume.
5. Call `dub_video_full` with all four parameters.
6. When completed, present the `video_url` as a download link.

---

## Workflow 6: Generate a Document from a Video

**Trigger:** User wants a written document, article, guide, tutorial, or documentation based on a video. Phrases like "generate a document", "create a guide from this video", "write an article about this video", "product documentation from this screencast".

**Tool:** `generate_document`

### Document Types

Ask the user which type of document they want before calling:
- **user_onboarding** — Step-by-step onboarding guide (default)
- **tutorial_guide** — Tutorial/how-to guide
- **feature_showcase** — Feature showcase document
- **business_overview** — Business overview document
- **product_documentation** — Product documentation

### Steps

1. Get the video source (URL or local path).
2. Ask which document type they want (see list above). Do not skip this.
3. Ask for language if not English (default: en).
4. Optionally ask for context about the video.
5. Call `generate_document` with `video_source`, `document_type`, `language`, and `manual_context`.
6. When completed (`status: document_ready`), present the `document_markdown` to the user.

### After Completion

The response includes `has_synced_transcript: true` and a full `transcript` array. After delivering the document, offer: "I also have a synced transcript for this video — would you like it?" If yes, present the transcript segments from the same response (no additional API call needed).

### Key Distinction

This is NOT for narrated videos or voiceover. The output is a written markdown document, not a video file. The video is analyzed visually (keyframes + AI understanding) to produce the document.

---

## Handling Timeouts

If any tool returns `status: timeout`:

1. Tell the user: "The video is still processing. This can happen with longer videos."
2. Save the `job_id` from the response.
3. Use `get_job_result(job_id)` to check status.
4. Possible statuses:
   - `processing` — still working, check again later
   - `transcript_ready` — transcript is done (for narration/transcription workflows)
   - `document_ready` — document and transcript are done (for document generation workflow)
   - `completed` — video is done, `video_url` is available
   - `failed` — something went wrong, show `error_message` to user

## Cancelling a Job

If the user wants to cancel a running job, call `abandon_job(job_id)`.

## Common Errors

### "API key required"
The NarrateAI API key is not configured. User needs to set `NARRATEAI_API_KEY` in their MCP server config.

### "Video file not found"
Local file path doesn't exist. Verify the path with the user.

### 429 / Rate limit
User has hit their API usage limit. Check subscription tier in NarrateAI dashboard.

### "has_active_job"
The server handles this automatically — it abandons the old job and starts a new one. No user action needed.

## Examples

### Example 1: Narrate a silent screencast

User says: "I have a screen recording of our app demo, it has no voice. Can you narrate it?"

Actions:
1. Ask: "What language should the narration be in?" (user says English)
2. Ask: "Any context about the video I should know for better narration?"
3. Call `narrate_video_full` with `video_source`, `language=en`, `voice_type` (ask user), `manual_context`
4. Return `video_url` as download link

Result: User gets a narrated video file with AI voiceover.

### Example 2: Transcribe a podcast

User says: "Transcribe this podcast for me: podcast.mp4"

Actions:
1. Ask: "What language is spoken in the podcast?" (user says Spanish)
2. Call `transcribe_video` with `video_source=podcast.mp4`, `source_language=es`
3. Present timed transcript segments

Result: User gets a text transcript of the Spanish speech, as-is, not translated.

### Example 3: Dub a video into French

User says: "I want this English video dubbed into French"

Actions:
1. Ask: "Does the video have background music you want to keep?"
2. Call `dub_video_full` with `video_source`, `source_language=en`, `target_language=fr`, `preserve_background_music`
3. Return `video_url` as download link

Result: User gets a French-dubbed video using the original speaker's cloned voice.

### Example 4: Generate a product guide from a screencast

User says: "I have a demo video of our new feature. Can you create a product guide from it?"

Actions:
1. Ask: "What type of document would you like? Options: user_onboarding, tutorial_guide, feature_showcase, business_overview, product_documentation"
2. User says: "feature_showcase"
3. Ask: "Any context about the video I should know?"
4. Call `generate_document` with `video_source`, `document_type=feature_showcase`, `manual_context`
5. Present the `document_markdown` to the user
6. Offer: "I also have a synced transcript for this video — would you like it?"

Result: User gets a structured feature showcase markdown document, plus optionally the timed transcript.

### Example 5: What this skill is NOT for

User says: "Trim the first 10 seconds off this video" — Do NOT use NarrateAI tools. This is a video editing task.

User says: "Convert this video to MP4" — Do NOT use NarrateAI tools. This is format conversion.

User says: "Compress this video" — Do NOT use NarrateAI tools. This is not a NarrateAI workflow.

---

## Decision Guide

| User wants... | Use this workflow |
|---|---|
| Script/transcript from a silent video | Workflow 1 |
| Narrated video with voiceover from a silent video | Workflow 2a |
| Turn an existing transcript into a video | Workflow 2b |
| Speech-to-text from a video with audio | Workflow 3 |
| Translated transcript from a new video | Workflow 4a |
| Translate an already-processed video's transcript | Workflow 4b |
| Fully dubbed video in another language | Workflow 5 |
| Document, article, guide, or docs from a video | Workflow 6 |
