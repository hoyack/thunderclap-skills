# Thunderclap skills

Agent skills for [Thunderclap](#about-thunderclap), a service that turns a brief into a finished
short-form video (LLM shot planning → Higgsfield/Kling clips → ElevenLabs narration and music →
ffmpeg render → signed webhook), exposed as a REST API and an MCP server. A skill is a folder with a `SKILL.md` (YAML
front matter `name` + `description`, then instructions) and optional `references/`,
`scripts/`, `assets/`. The description decides when an agent loads the skill; the body and
references are read only when it does.

## Layout

One directory per **skill set**, one directory per **skill** inside it:

```
.
├── README.md                     this file
└── mcp/                          using Thunderclap through its MCP server
    └── thunderclap-video/
        ├── SKILL.md              brief → generate_video → poll → deliver; rules, costs, failures
        └── references/
            ├── styles.md         the 14 usable styles (+3 to avoid), from the service's style catalog
            └── fields.md         every generate_video field, from the service's API model
```

## Connecting

The skills drive Thunderclap's MCP server, so connect it first:

- **Endpoint:** `https://<thunderclap-host>/mcp` (Streamable HTTP)
- **Auth:** header `Authorization: Bearer <VIDEO_API_KEY>` (no OAuth; the client must send a
  static header)

```bash
# Claude Code
claude mcp add --transport http thunderclap https://<thunderclap-host>/mcp \
  --header "Authorization: Bearer $VIDEO_API_KEY"
```

```json
// clients with a JSON config for remote servers (Cursor, VS Code, ...)
{"mcpServers": {"thunderclap": {"url": "https://<thunderclap-host>/mcp",
  "headers": {"Authorization": "Bearer <VIDEO_API_KEY>"}}}}
```

Clients that only start local (stdio) servers, such as the Claude Desktop config file, can
bridge with `npx -y mcp-remote https://<thunderclap-host>/mcp --header "Authorization:Bearer ${VIDEO_API_KEY}"`.

## Installing a skill

- **Claude Code**: clone this repository, then link or copy a skill folder into
  `~/.claude/skills/` (all projects) or `<project>/.claude/skills/`:
  ```bash
  git clone https://github.com/hoyack/thunderclap-skills.git
  ln -s "$PWD/thunderclap-skills/mcp/thunderclap-video" ~/.claude/skills/thunderclap-video
  ```
- **Claude.ai / Claude Desktop**: zip the skill folder (the folder itself at the zip root)
  and upload it under Settings → Capabilities → Skills.
- Other agents: give them `SKILL.md` as instructions and make the `references/` files
  readable.

`references/styles.md` and `references/fields.md` mirror the service's style catalog and API
model; they are regenerated when those change.

## Roadmap (proposed skill sets)

| set | skill | why |
|---|---|---|
| `mcp/` | `thunderclap-video` | **done**: make, track, resume videos via MCP |
| `writing/` | `narration-script` | write a voiceover that fits a target length (~2.5 words/s), one beat per clip; the most common cause of a bad video is a script that doesn't fit |
| `writing/` | `storyboard` | turn a brief into a per-clip scene plan (≤4 clips × ≤15 s, chained frames) and, when wanted, the `on_screen_text` lines |
| `formats/` | one skill per style family (explainers, character/mascot, live-action/testimonial, music/kinetic) | prompt patterns, pitfalls and example requests per style; one skill per style family keeps 14 styles manageable |
| `assets/` | `brand-assets` | prepare end-card logos (1920×1080 / 1080×1920 PNG, safe margins) and reference images for consistent characters/presenters |
| `ops/` | `thunderclap-operator` | deploy, configuration changes, resume jobs, redeliver callbacks, Higgsfield reconnect |

Each new skill should come with a few test prompts (dry runs that build the
`generate_video` arguments without calling it, so testing costs nothing).

## About Thunderclap

Thunderclap is a self-hosted FastAPI service. These skills document its public interface
(MCP tools and request fields); the service itself lives in a separate repository.

## License

MIT
