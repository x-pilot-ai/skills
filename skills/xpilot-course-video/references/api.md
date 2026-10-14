# Video API workflow

Use the deployed Node API root from the console as `XPILOT_API_BASE_URL` (without /v1) and `XPILOT_API_KEY` from the environment. REST uses `Authorization: Bearer`; responses are under `data`, errors under `error` with `code`, `message` and, for 400, `param` and `allowed`. MCP uses the console's actual /mcp URL with OAuth + S256 PKCE or an API Key. Installation alone does not authorize generation. Times are Unix seconds.

## REST routes

### Account

| Method | Path | Purpose |
|---|---|---|
| GET | /v1/account/credits | Shared balance and plan |
| GET | /v1/account/capacity | Concurrency slots and credential monthly budget |

### Video Agent

| Method | Path | Purpose |
|---|---|---|
| GET | /v1/video-agents/conversations | Owned conversations (`limit`, `offset`) |
| POST | /v1/video-agents/conversations | Create conversation and start the first message |
| GET | /v1/video-agents/conversations/{conversation_id} | Conversation summary |
| POST | /v1/video-agents/conversations/{conversation_id} | Send a message, confirm or revise the outline |
| GET | /v1/video-agents/conversations/{conversation_id}/messages | Messages (poll here) |
| POST | /v1/video-agents/conversations/{conversation_id}/stop | Stop the running message |
| DELETE | /v1/video-agents/conversations/{conversation_id} | Delete conversation |

### Renders (MP4 export)

| Method | Path | Purpose |
|---|---|---|
| GET | /v1/video-agents/conversations/{conversation_id}/renders | List exports |
| POST | /v1/video-agents/conversations/{conversation_id}/renders | Start export (Idempotency-Key required) |
| GET | /v1/video-agents/conversations/{conversation_id}/renders/{render_id} | Poll export |
| DELETE | /v1/video-agents/conversations/{conversation_id}/renders/{render_id} | Delete export record |

### Brand kits, assets, voices, avatars

| Method | Path | Purpose |
|---|---|---|
| GET / POST | /v1/brand-kits | List (`limit`, `offset`) or create |
| GET / PUT / DELETE | /v1/brand-kits/{brand_kit_id} | Read, update, delete |
| GET | /v1/assets | Owned uploads (`limit`, `offset`) |
| POST | /v1/assets | Multipart upload, field `files`, max 5 — see [uploads.md](uploads.md) |
| GET | /v1/voices | Voice catalog for `voice_id` |
| GET | /v1/avatars | Owned presenters for `avatar_id` |

List endpoints return `{items, total, limit, offset}`.

## Create a conversation

`POST /v1/video-agents/conversations` with `Idempotency-Key: <uuid>`:

```json
{
  "prompt": "# Solar energy\nAudience: beginners...\n\n## How panels work\n...",
  "require_outline_confirmation": false,
  "video_config": {
    "target_language": "en",
    "video_duration": "short",
    "video_style": "science_explainer",
    "voice_id": "<voice_id>",
    "voice_version": 1,
    "avatar_id": "<selectable_avatar_id>",
    "brand_kit_id": "<brand_kit_id>",
    "enable_initial_ai_video": false
  },
  "files": [
    {
      "asset_id": "<asset_id>",
      "files_role": "primary"
    }
  ]
}
```

Only `prompt` is required (≤20,000 characters).

`video_config` fields:

| Field | Values | Default |
|---|---|---|
| target_language | ISO 639-1 or BCP-47 (en, zh, zh-CN, ja, de, fr, es…) | en |
| video_duration | short (1–3 min), deep (3–10 min), long (10–20 min) | short |
| video_style | auto, clean_lecture, science_explainer, professional_training, product_showcase, cinematic_story | auto |
| voice_id | id from GET /v1/voices | picked for the language |
| voice_version | 1 or 2; requires voice_id | 1 |
| locale | voice locale such as en-US; requires voice_id | — |
| avatar_id | id from GET /v1/avatars with `selectable: true` (otherwise 409 `avatar_not_ready`; unknown id 404 `avatar_not_found`) | none |
| brand_kit_id | id from GET /v1/brand-kits | none |
| enable_initial_ai_video | boolean | false |

Unknown fields or values return 400 `invalid_parameter` with `param` (for example `video_config.video_style`) and `allowed`. `video_config` and `require_outline_confirmation` are create-only; sending them on a follow-up returns 400 `unsupported_parameter`.

Billing is the same as the X Pilot app: the platform holds a reserve when the message starts, releases it when the message ends, then charges actual usage. If the balance is short, the charge stops at 0 balance; it never goes negative. There is no per-message cap; `max_cost_credits` returns 400 `unsupported_parameter`. 402 `insufficient_credits` means the balance cannot cover the reserve; 402 `budget_exceeded` means the credential monthly budget is used up.

The response is 202 with `conversation_id`, `message_id`, `status` and `cost_credits` (the reserve held for this message). After the message settles, its `cost_credits` is the actual charge.

## Optional callback instead of polling

Create and send also accept `callback_url` (public HTTPS on port 443, ≤2,048 characters) and `callback_id` (letters, digits, `_ . : -`, ≤128; requires `callback_url`). When that message ends, X Pilot POSTs `{id, type: "video-agent.<status>", created_at, data}` where `data` matches the message fields plus `callback_id`. The request carries `X-XPilot-Signature: t=<unix>,v1=<hex>` = HMAC-SHA256(workspace callback secret, `t + "." + raw body`); the secret is in Developer console → Webhooks. Deliveries retry up to 6 times and every attempt is listed in the console. Private or http addresses return 400 `invalid_webhook_url` / `unsafe_webhook_url` with `param: "callback_url"`. A callback only covers the message started by that request — pass it again when confirming an outline. Only use a callback URL the user gave you; otherwise keep polling.

## Poll messages

`GET .../messages` returns `{items, total}`. Message fields: `conversation_id`, `message_id`, `status`, `next_action`, `outline`, `created_at`, `error`, `query`, `answer`, `message_files`, `preview_video_url`, `cost_credits`.

- `status`: queued, running, stopping, paused, completed, failed, stopped. completed, paused, failed and stopped end that message.
- `next_action` (only when paused): `confirm_outline` or `reply`.
- `outline` (once planned): `title`, `total_scenes`, `estimated_duration_seconds`, `visual_style`, `confirmed`, `scenes: [{index, title, intent, type, estimated_duration_seconds}]`.

Conversation `status` and `next_action` follow its latest message. Poll at least 5 seconds apart and respect Retry-After on 429.

## Follow-up messages

`POST /v1/video-agents/conversations/{conversation_id}` with a new Idempotency-Key. Returns 409 `conversation_busy` while a message runs.

| Situation | Body |
|---|---|
| next_action = confirm_outline, user approves | `{"outline_action": "confirm"}` |
| next_action = confirm_outline, user wants changes | `{"outline_action": "revise", "prompt": "Merge scenes 3 and 4"}` |
| next_action = reply | `{"prompt": "<user's answer>"}` |
| completed, user wants edits | `{"prompt": "Shorten scene 2", "files": [...]}` |

`files` is also accepted on follow-ups. Put `files_role` inside each file item.

## Renders

`POST .../renders` needs `Idempotency-Key` and accepts `message_id` (default: latest message with a preview) and `force` (re-export). The same key returns the same render; a different body with the same key returns 409. Without a preview it returns 409 `preview_not_ready`. Render `status`: queued, running, completed, failed, superseded. Poll `GET .../renders/{render_id}` until `video_url` is set.

## Idempotency

The agent creates one UUID per new operation and reuses it only to retry the same input. REST sends it as `Idempotency-Key`; MCP write tools take `idempotency_key` (8–128 letters, digits, `_`, `.`, `:` or `-`). Same key with different input returns 409 `idempotency_conflict`.

## MCP tools

- Account: check_credits, check_capacity
- Video Agent: list_conversations, get_conversation, create_conversation, send_message, list_messages, stop_conversation, delete_conversation
- Renders: list_renders, create_render, get_render, delete_render
- Brand kits: list_brand_kits, get_brand_kit, create_brand_kit, update_brand_kit, delete_brand_kit
- Assets: list_assets (upload via REST `POST /v1/assets`)
- Voices: list_voices, get_voice, generate_audio, clone_voice, delete_voice
- Avatars: list_avatars, get_avatar, create_avatar, delete_avatar

MCP arguments mirror the REST bodies. Invalid arguments return JSON-RPC -32602 with the failing path.
