# X Pilot Skills

Official Agent Skills for [X Pilot](https://www.x-pilot.ai): course and explainer videos, digital human
avatars and narration. Each skill calls the X Pilot API or the remote MCP server at `https://mcp.x-pilot.ai/mcp`.

| Skill | What it does |
| --- | --- |
| `xpilot-audio` | Find and preview X Pilot voices and produce spoken MP3 audio when the user requests narration, voiceover, or text-to-speech using X Pilot. |
| `xpilot-avatar` | Find, create and inspect X Pilot digital humans from uploaded images or training videos when the user asks for an X Pilot presenter. |
| `xpilot-course-video` | Create or revise educational and explainer videos with X Pilot, including the brief, style and duration choice, source uploads, optional outline review, asynchronous polling, preview handoff and MP4 export. |

## Install

```bash
# Any agent (Cursor, Claude Code, Codex and 40+ others); add -g for a user-level install
npx skills add x-pilot-ai/skills --skill xpilot-course-video --agent cursor

# GitHub CLI v2.90+
gh skill install x-pilot-ai/skills xpilot-course-video
```

Agent ids: `cursor`, `claude-code`, `codex`. Use `--skill '*'` for all skills.

Claude Code can also install the full plugin (all Skills plus MCP):

```text
/plugin marketplace add x-pilot-ai/skills
/plugin install x-pilot@x-pilot-skills
```

## Connect

Add the MCP server `https://mcp.x-pilot.ai/mcp` to your agent and sign in with your X Pilot account when prompted,
or set `XPILOT_API_KEY` from the [developer console](https://app.x-pilot.ai/developers).
Usage is billed to your X Pilot credits.

[Documentation](https://app.x-pilot.ai/developers) · [Support](https://www.x-pilot.ai/support) ·
[Privacy policy](https://www.x-pilot.ai/privacy) · [Terms of service](https://www.x-pilot.ai/terms) ·
[License](./LICENSE)
