# Upload rules

Upload with REST multipart `POST /v1/assets`, form field `files`, repeated for several files. There is no MCP upload tool; agents without REST access should ask the user to upload in X Pilot and then find the file with `list_assets`.

## Limits

| Kind | Extensions | Max per file | Use in a video |
|---|---|---|---|
| Document | pdf, docx, pptx, md, markdown, mdx | 10 MB | Main source or background facts |
| Image | jpg, jpeg, png, gif, webp, svg | 10 MB | Real product shots, photos, diagrams placed in matching scenes |
| Video | mp4, webm, mov | 60 MB | Real footage or product demos placed in matching scenes |
| Audio | wav, mp3, m4a, aac, ogg, flac | 20 MB | Voice cloning and avatars only; not a video attachment |

- At most 5 files per request and 5 files per message.
- Empty files and other extensions are rejected with 400 `invalid_parameter`. The `error` object carries `reason` (`too_many`, `unsupported_type`, `empty` or `too_large`), `file` and `max_mb`.
- The response `data` is a list of `{id, name, url, mimetype, created_at, updated_at}`.

## Attaching to a conversation

```json
{
  "prompt": "Turn the uploaded spec into a 3-minute overview for sales.",
  "files": [
    {
      "url": "https://cdn.example.com/product-spec.pdf",
      "files_role": "primary"
    }
  ],
  "video_config": { "target_language": "en", "video_duration": "short" }
}
```

- Each item needs an owned `asset_id` or a public http(s) `url` supplied by the user. Third-party URLs are downloaded and archived to the user's storage before use. `name` and `type` (`document`, `image`, `video`) are optional; type is inferred.
- Put `files_role` in each item: `primary` builds the outline and visuals from that file; `reference` uses that file only as background. Omit it on an item to let the agent decide.
- Never invent or guess URLs, and do not use private-network URLs.
- Files can also be attached to a follow-up `send_message`.
