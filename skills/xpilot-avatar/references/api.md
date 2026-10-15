# Avatar API

Use the configured Node root and environment API Key with Bearer authentication. All responses are under `data`. REST and MCP delegate to the platform's avatar service and entitlement/billing rules.

Avatar object (list items, detail and create response):

```json
{
  "id": "<avatar_id>",
  "name": "Presenter",
  "avatar_type": "human_video",
  "status": "ready",
  "training_status": "ready",
  "preview_status": "ready",
  "selectable": true,
  "stage": null,
  "preview_image_url": "https://…",
  "preview_video_url": "https://…",
  "error": null,
  "created_at": 1791849600,
  "updated_at": 1791849680
}
```

- `status`: overall state — `draft`, `generating`, `ready`, `failed`, `deleting` or `migration_required`.
- `training_status`: `pending`, `generating`, `ready` or `failed`. `preview_status`: `not_requested`, `pending`, `generating`, `ready` or `failed`.
- `selectable`: `true` only when the avatar can be used as `video_config.avatar_id`.
- `stage`: step in progress (`queued`, `generating_training_video`, `provider_processing`, `generating_preview`, `deleting_provider_asset`), or `null` when idle.
- `error`: `{code, message}` when something failed, otherwise `null`. `created_at` / `updated_at` are Unix seconds.

- GET /v1/avatars: `{items, total, quota: {used, limit}}`; items are avatar objects.
- GET /v1/avatars/{avatar_id}: one avatar object.
- POST /v1/avatars: 202 accepted. Body `{ "name": "Presenter", "avatar_type": "human_video", "asset_id": "<asset_id>" }`, or `"video_url"` with a caller-supplied public http(s) link. For `virtual_character`, pass an image `asset_id` or public `image_url` plus `gender` (`Male` or `Female`). Third-party URLs are downloaded and archived to the caller's owned OSS/Attachment storage before training; private-network URLs, invalid types and oversized files are rejected. Requires the platform avatar entitlement. Do not send a consent field. Returns the avatar object; poll GET until `status` is `ready` or `failed`.
- DELETE /v1/avatars/{avatar_id}: 202 accepted with `{id, status: "deleting"}`. Poll until the avatar disappears from the list.
- POST /v1/avatars/{avatar_id}/videos: 202 accepted. Avatar `training_status` must be `ready`. Body is exactly one speech source: `{ "text": "Welcome to the course.", "voice_id": "<voice_id>" }` (≤5,000 characters, Voice I; optional `locale`, `speed` 0.5–2), or `{ "asset_id": "<audio_asset_id>" }` / `{ "audio_url": "https://..." }` (public audio ≤20 MB, archived to owned storage first). Speech up to 5 minutes. Billed per rendered second at the digital-human video rate; returns 402 `insufficient_credits` when the balance cannot cover the estimate. Returns `{id, avatar_id, status, source, format: "webm", has_alpha: true, ...}`.
- GET /v1/avatars/{avatar_id}/videos/{video_id}: `status` is `queued`, `processing`, `completed` or `failed`. When completed: `video_url` (transparent WebM, VP9 with alpha), `duration_seconds`, `width`, `height`, `cost_credits`. Failed jobs carry `error {code, message}` and are not charged. Poll at least 5 seconds apart.

## Optional callback instead of polling

POST /v1/avatars and POST /v1/avatars/{avatar_id}/videos (MCP `create_avatar`, `create_avatar_video`) accept an optional `callback_url` (public HTTPS on port 443, ≤2,048 characters) and `callback_id` (letters, digits, `_ . : -`, ≤128; requires `callback_url`). When the job settles X Pilot POSTs `{id, type, created_at, data}` signed with `X-XPilot-Signature: t=<unix>,v1=<hex>` = HMAC-SHA256(workspace callback secret, `t + "." + raw body`); the secret is in Developer console → Webhooks.

- `avatar.completed`: training finished and the preview sample (if requested) is ready. `avatar.failed`: training or the sample failed. `data`: `avatar_id`, `status` (same value as the avatar object, normally `ready` or `failed`), `training_status`, `preview_status`, `selectable`, `preview_image_url`, `preview_video_url`, `error`, `error_code`, `callback_id`.
- `avatar-video.completed` / `avatar-video.failed`. `data`: `avatar_id`, `video_id`, `status`, `video_url`, `format`, `has_alpha`, `duration_seconds`, `width`, `height`, `cost_credits`, `error`, `error_code`, `callback_id`.

Account webhook endpoints can subscribe to the same four events. States are checked about every 30 seconds, so events can lag up to a minute. Deliveries retry up to 6 times; deduplicate by `id`. The callback is part of the idempotency fingerprint. Only use a callback URL the user gave you; otherwise keep polling.

Every write needs an agent-generated stable UUID: REST `Idempotency-Key` header, MCP `idempotency_key` argument (8–128 letters, digits, `_`, `.`, `:` or `-`). A repeat while pending returns 409; a completed repeat returns the same resource; different input with the same key returns 409. No automatic retraining after failed or unknown dispatch.

MCP tools: list_avatars, get_avatar, create_avatar, delete_avatar, create_avatar_video, get_avatar_video. Upload training sources and speech audio with REST `POST /v1/assets`; there is no MCP upload tool. An avatar video occupies one concurrency slot until it completes or fails.

Private resources stay in the owning account. `training_status=ready` does not imply `selectable=true`: a ready sample video must also exist. Only selectable avatars work as `video_config.avatar_id`.
