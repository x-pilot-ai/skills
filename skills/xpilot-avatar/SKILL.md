---
name: xpilot-avatar
description: Find, create and inspect X Pilot digital humans from uploaded images or training videos when the user asks for an X Pilot presenter.
---

Use connected MCP first. For REST read [references/api.md](references/api.md). Installation does not authorize access; credentials stay in client settings or environment variables.

## Source rules

- Prefer a file in the user's own account: find it with `list_assets` or upload it with REST multipart `POST /v1/assets` (images ≤10 MB, video ≤60 MB), then pass its `asset_id`.
- Use `image_url` / `video_url` only when the user gives you that http(s) link themselves. Never search for, guess or invent a source URL, and never use a link to someone else's likeness without the user saying they have the right to use it.

## Workflow

- List avatars with `list_avatars` and inspect `get_avatar` before choosing a presenter.
- Create with `create_avatar`: name, avatar_type (`human_video` with a training video, or `virtual_character` with an image plus gender `Male` or `Female`), and the source from the rules above. Generate a stable UUID `idempotency_key` internally. Do not send a consent field or ask the user for API permissions or budgets.
- Save the avatar ID and poll `get_avatar` about every five seconds, respecting Retry-After, until `preview_image_url` / `preview_video_url` appear or training reaches a terminal state. Report pending, failed and migration-required states honestly; never recreate or retrain to hide a failure.
- An avatar is usable in a video only when `selectable` is `true`. `trainingStatus=ready` alone is not enough. Deliver the real preview URLs returned by the resource.
- Delete owned avatars with `delete_avatar` only when the user asks.

Do not clone a voice or publish a video as a side effect. For a full video with this presenter, use the course video Skill and pass the avatar ID as `video_config.avatar_id`; the video request fails with 409 `avatar_not_ready` until the avatar is selectable.
