# Avatar API

Use the configured Node root and environment API Key with Bearer authentication. All responses are under `data`. REST and MCP delegate to the platform's avatar service and entitlement/billing rules.

- GET /v1/avatars: `{items, total, quota}`. Items include `selectable`, `preview_image_url`, `preview_video_url`, `created_at` and `updated_at` (Unix seconds).
- GET /v1/avatars/{avatar_id}: owned avatar with trainingStatus, availability, selectable, preview_image_url and preview_video_url.
- POST /v1/avatars: 202 accepted. Body `{ "name": "Presenter", "avatar_type": "human_video", "asset_id": "<asset_id>" }`, or `"video_url"` when the user supplied the link. For `virtual_character`, pass an image `asset_id` or `image_url` plus `gender` (`Male` or `Female`). Requires the platform avatar entitlement. Do not send a consent field. Poll GET until preview_* fields populate.
- DELETE /v1/avatars/{avatar_id}: 202 accepted. Poll until the avatar disappears from the list.

Every write needs an agent-generated stable UUID: REST `Idempotency-Key` header, MCP `idempotency_key` argument (8–128 letters, digits, `_`, `.`, `:` or `-`). A repeat while pending returns 409; a completed repeat returns the same resource; different input with the same key returns 409. No automatic retraining after failed or unknown dispatch.

MCP tools: list_avatars, get_avatar, create_avatar, delete_avatar. Upload training sources with REST `POST /v1/assets`; there is no MCP upload tool.

Private resources stay in the owning account. `trainingStatus=ready` does not imply `selectable=true`: a ready sample video must also exist. Only selectable avatars work as `video_config.avatar_id`.
