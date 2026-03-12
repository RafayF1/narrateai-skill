# NarrateAI MCP — Other IDEs & AI Assistants

Setup instructions for Windsurf, JetBrains AI Assistant, Google AI Studio (Project IDX / Gemini), and any MCP-compatible tool.

## General Setup

All platforms follow the same pattern:

1. **Install the MCP server:** `pip install narrateai-mcp`
2. **Get an API key** at [narrateai.app](https://narrateai.app)
3. **Configure your IDE's MCP settings** (see platform-specific sections below)
4. **Give your AI the workflow instructions** from `workflows.md` in this repo (paste as context, system prompt, or knowledge file — whatever your platform supports)

---

## Windsurf (Codeium)

Windsurf supports MCP servers via its configuration file.

Add to `~/.codeium/windsurf/mcp_config.json`:

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

Restart Windsurf after adding. The NarrateAI tools will appear in Cascade.

For workflow guidance, add the contents of `workflows.md` to your Windsurf Rules (`.windsurfrules` in your project root).

---

## JetBrains AI Assistant

JetBrains IDEs (IntelliJ, WebStorm, PyCharm, etc.) support MCP through the AI Assistant plugin.

1. Open **Settings → Tools → AI Assistant → MCP Servers**
2. Add a new server with:
   - **Command:** `narrateai-mcp`
   - **Environment variables:**
     - `NARRATEAI_API_BASE_URL` = `https://api.narrateai.app`
     - `NARRATEAI_API_KEY` = `your_api_key_here`

For workflow guidance, paste the contents of `workflows.md` into the AI Assistant's custom instructions.

---

## Google AI Studio / Project IDX / Gemini

If your platform supports MCP via stdio:

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

If your platform supports MCP via HTTP/SSE, point it to the hosted server:

```
URL: https://mcp.narrateai.app/sse
Header: x-api-key: your_api_key_here
```

For workflow guidance, upload `workflows.md` as context or system instructions.

---

## Any Other MCP-Compatible Tool

The NarrateAI MCP server works with any tool that supports the Model Context Protocol. Two connection modes:

### Stdio (local)

Run `narrateai-mcp` as a subprocess. Set these environment variables:
- `NARRATEAI_API_BASE_URL` — the NarrateAI API URL (e.g., `https://api.narrateai.app`)
- `NARRATEAI_API_KEY` — your API key

### HTTP/SSE (remote)

Connect to the hosted server:
- **SSE endpoint:** `https://mcp.narrateai.app/sse`
- **Authentication:** `x-api-key` header with your API key

### Providing Workflow Context

The MCP tools work on their own, but results are significantly better when your AI knows the full workflow instructions. Depending on your platform:

- **Rules/system prompt file** — paste contents of `workflows.md`
- **Knowledge/context upload** — upload `workflows.md` as a file
- **Conversation start** — paste the relevant sections at the beginning of your conversation

The key things the AI needs to know:
1. Ask for video context before narrating (improves quality)
2. Ask for voice choice before narrating (don't default silently)
3. Ask for language before transcribing (don't guess)
4. Ask about background music before dubbing
5. Present `video_url` as clickable download links

---

## Available Tools Quick Reference

| Tool | Description |
|------|-------------|
| `generate_narration_script` | AI script for silent video (text only) |
| `narrate_video_full` | Full narrated video with voiceover |
| `continue_to_full_video` | Turn script into narrated video |
| `transcribe_video` | Speech-to-text |
| `translate_video` | Transcribe + translate |
| `translate_existing_video` | Translate existing video's transcript |
| `dub_video_full` | Voice-cloned dubbing |
| `generate_document` | Docs/guides from video |
| `list_videos` | Browse video library |
| `update_transcript` | Edit script before narrating |
| `get_job_result` | Check job status |
| `get_upload_url` | Upload local files (remote mode) |
| `abandon_job` | Cancel a job |
| `narrate_batch` | Narrate up to 5 videos |
| `batch_generate_scripts` | Scripts for multiple videos |
| `batch_transcribe` | Transcribe multiple videos |
| `batch_dub` | Dub multiple videos |
