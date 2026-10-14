---
name: xpilot-audio
description: Find and preview X Pilot voices and produce spoken MP3 audio when the user requests narration, voiceover, or text-to-speech using X Pilot.
---

Prefer a connected X Pilot MCP client. For REST, read [references/api.md](references/api.md); use configured environment credentials, never ask for keys in chat.

## Voice versions

- The developer API synthesizes and clones with **Voice I only** (`voice_version=1`, the default). Voice II is not available on the developer API; when the user needs it, point them to the X Pilot platform.
- Video generation is separate: a video's `video_config.voice_id` may use either version returned by `list_voices`.

## Workflow

- Find voices with `list_voices` (`type=public` for the catalog, `type=private` for the user's own clones), filtered by language. Use a returned voice ID; never guess provider IDs or clone another person's voice.
- Check readiness with `get_voice`. For pending or failed voices, explain the status instead of silently substituting another voice.
- For a fixed audition clip, play `samples` from list or detail. There is no separate preview endpoint.
- Call `generate_audio` with text, voice_id and optional locale and speed. Limit each request to 1,000 characters; split longer scripts on paragraph boundaries and label the clips. Do not promise one merged file.
- To clone a Voice I voice, find an audio file with `list_assets` or upload one with REST multipart `POST /v1/assets` (field `files`; wav, mp3, m4a, aac, ogg or flac, ≤20 MB), then call `clone_voice` with that owned audio URL. Delete owned clones with `delete_voice`.
- Generate a stable UUID `idempotency_key` for each write and reuse it only when retrying the same input. Never ask the user for retry keys, budgets or limits. A pending or unknown result is not permission to resubmit with a new key.

## Delivery

Return the real `audio_url`, format and voice used. A data URL contains the MP3 bytes; save or play it with the client's media tools. Never fabricate a hosted download URL or claim a clone finished before `get_voice` shows it ready.

Standalone TTS follows the platform TTS policy with no separate developer surcharge. Video and avatar workflows bill differently.
