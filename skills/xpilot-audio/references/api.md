# Audio API

REST requires `XPILOT_API_BASE_URL` (Node API root, without /v1) and `XPILOT_API_KEY` in the environment. Use `Authorization: Bearer` and parse the `data` response. MCP uses the console's actual /mcp URL with OAuth or Key.

- GET /v1/voices?voice_version=1&type=public&language=en&page=1&limit=20: paginated public directory. `type=private` lists owned clones. Use the `_id` returned by the catalog, not a vendor ID. Fixed audition clips come from item `samples`.
- GET /v1/voices/{voice_id}?voice_version=1: public or owned voice details, including samples.
- POST /v1/voices/speech: body `{ "text": "Welcome to the lesson.", "voice_id": "<catalog_id>", "voice_version": 1, "locale": "en-US" }`. Voice I only; `voice_version=2` returns `403 voice_version_unavailable`. Optional speed 0.5–2. Text 1–1,000 characters. Returns `{audio_url, format, voice_id, voice_version, locale, billing_basis, cost_credits}`; audio_url can be a data URL.
- POST /v1/voices/clone: 202 accepted. Body `{ "name": "My clone", "audio_url": "<url returned by POST /v1/assets>" }`. Voice I only. The URL must belong to the user's uploads. Poll GET /v1/voices/{voice_id}.
- DELETE /v1/voices/{voice_id}: delete an owned clone. Public catalog voices cannot be deleted.

Every write needs an agent-generated stable UUID: REST `Idempotency-Key` header, MCP `idempotency_key` argument (8–128 letters, digits, `_`, `.`, `:` or `-`). Same key with different input returns 409. Pending or failed dispatches are not repeated automatically.

MCP tools: list_voices, get_voice, generate_audio, clone_voice, delete_voice. For 429 wait Retry-After; for 401 reconnect; for 404 check ownership and voice version. Do not copy private data URLs into public storage.
