---
name: xpilot-course-video
description: Create or revise educational and explainer videos with X Pilot, including the brief, style and duration choice, source uploads, optional outline review, asynchronous polling, preview handoff and MP4 export. Use when the user wants an X Pilot course, training, product or explainer video.
---

# X Pilot course video

Turn the user's goal into one video and return its real conversation_id, message state and preview.

## 1. Connection

- Prefer a connected X Pilot MCP server when its tools are present. For REST, read [references/api.md](references/api.md) and use `XPILOT_API_BASE_URL` and `XPILOT_API_KEY` from the environment.
- Installing this Skill does not authorize an account. Never invent a server address and never ask for keys in chat.
- With no authorized connection, prepare the brief and say clearly that generation has not started.

## 2. Settle four things before generating

| Item | Ask the user? | Maps to |
|---|---|---|
| Audience and goal | Yes | First lines of `prompt` |
| Language | Yes, unless obvious | `video_config.target_language` (cannot change later) |
| Length | Yes, unless obvious | `video_config.video_duration`: `short` 1–3 min, `deep` 3–10 min, `long` 10–20 min |
| Style | No, default `auto` | `video_config.video_style` |

Optional, only when the user wants them: `voice_id` (+ `voice_version`) from `list_voices`, `avatar_id` from `list_avatars` (must be `selectable: true`), `brand_kit_id` from `list_brand_kits`, `enable_initial_ai_video`. `video_config` is fixed at creation; a follow-up message cannot change it. Unknown fields or values fail with 400 and list the allowed values.

## 3. Write the prompt

Read [references/prompt-guide.md](references/prompt-guide.md) and pick the closest video type.

- Use a `#` title and several `##` sections. Each section becomes roughly one scene; keep one video to about 12 scenes and split bigger topics into a series.
- Write the real content: facts, numbers, formulas, dates and ordered steps. Do not name visual components ("use a bar chart"); the agent chooses visuals.
- Do not invent statistics. If the user did not give a number, leave it out or ask.

## 4. Upload source files

Read [references/uploads.md](references/uploads.md).

- Each `files` item uses either an owned `asset_id` or a public http(s) `url` supplied by the user. X Pilot archives third-party URLs to the user's storage before generation. REST multipart `POST /v1/assets` remains available for direct uploads; there is no MCP upload tool.
- Put `files_role` inside each item: `"primary"` when that file is main content/visual material, `"reference"` when it is background only. Omit it for that item when the agent should decide.
- Never search for, guess or invent file URLs, and never use private-network URLs.

## 5. Outline mode

- Default: omit `require_outline_confirmation`. The video is generated straight through. Do not send `outline_action`.
- When the user asks to see the outline first, create with `require_outline_confirmation: true`. When the message shows `status: "paused"` and `next_action: "confirm_outline"`, show `outline` (title, scenes, estimated duration) to the user, then:
  - user approves: `send_message` with `outline_action: "confirm"` and no prompt;
  - user wants changes: `send_message` with `outline_action: "revise"` and the changes in `prompt`.
- `next_action: "reply"` means the assistant asked a question. Relay it and send the user's answer as `prompt`.

## 6. Submit, poll, deliver

- Call `check_credits` and `check_capacity` first. Billing matches the X Pilot app: a reserve is held when the message starts (tell the user the `cost_credits` returned on create or send), released at the end, then actual usage is charged. Longer videos cost more; there is no per-message cap.
- Generate one stable `idempotency_key` (UUID) per operation and reuse it only when retrying the exact same input.
- Poll `list_messages` no faster than every 5 seconds and respect Retry-After. `status` is queued, running, stopping, paused, completed, failed or stopped; completed, paused, failed and stopped end that message. A timeout or disconnect is not completion.
- Deliver only service URLs (`preview_video_url`), with conversation_id and message_id. A preview is not an MP4: call `create_render` (with its own `idempotency_key`) and poll `get_render` when the user needs a file. Report the returned `progress` (0–100) when present; if it is `null`, report the status without inventing a percentage.
- To edit a finished video, `send_message` with a concrete instruction ("shorten scene 2, add a worked example").
- Stop with `stop_conversation` only when the user asks; incurred cost may still be charged.

## 7. Never

Top up, enable automatic charging, expand permissions, or ask the user for scopes, budgets or retry keys. Never report failed or unfinished runs as done.
