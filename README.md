# roblox-master-dev

A Claude Agent Skill that makes Claude build and polish Roblox experiences
(Luau/Roblox Studio) like a 10-15 year veteran Roblox developer — clean
client/server architecture, professional "juice" (feel/polish), solid UI/UX,
performance discipline, and a self-research protocol so it looks things up
instead of guessing at Roblox APIs it isn't sure about.

## What's in here

- `SKILL.md` — the main skill file (frontmatter + workflow). This is what
  Claude reads to know when and how to use the skill.
- `references/` — deep-dive docs Claude loads only when relevant:
  - `architecture.md` — client/server split, folder layout, remotes, module patterns
  - `scripting-patterns.md` — Luau idioms, common pitfalls, debugging workflow
  - `polish-and-juice.md` — motion, camera, audio, VFX, the "feels professional" checklist
  - `ui-ux-design.md` — responsive layout, input handling, accessibility
  - `performance-optimization.md` — hot-path rules, part counts, diagnosing lag
  - `data-and-security.md` — DataStores, remote validation, monetization
  - `self-research-protocol.md` — when/how Claude should look things up instead of guessing
  - `mcp-tools.md` — reference for the built-in Roblox Studio MCP server's tools

## Installing this skill

### Claude (claude.ai / Claude Desktop / Cowork)
Zip this folder as `roblox-master-dev.skill` (a `.skill` file is just a zip of
the skill directory) and use the "Save skill" flow, or drop the folder into
wherever your Claude setup loads local skills from.

### Claude Code
Copy this folder into your project's `.claude/skills/` directory (or your
global `~/.claude/skills/` directory), e.g.:

```bash
cp -r roblox-master-dev ~/.claude/skills/roblox-master-dev
```

Claude Code will pick it up automatically based on the `description` in
`SKILL.md`'s frontmatter, or you can invoke it directly with
`/roblox-master-dev`.

### Other tools that read Anthropic-style skills
Any tool that supports the Agent Skills format (a folder with a `SKILL.md`
containing YAML frontmatter + Markdown body, plus optional `references/`,
`scripts/`, `assets/`) should be able to load this folder as-is — check that
tool's docs for where it expects local skills to live.

## Recommended pairing

This skill assumes Roblox Studio's built-in MCP server is enabled (Studio →
Assistant → `…` → Manage MCP Servers → "Enable Studio as MCP server") and
connected to whatever client is using this skill, so Claude can read the live
DataModel, edit scripts, execute Luau, and playtest — not just write files
blind. It still works without MCP; it'll just produce `.lua`/`.luau` files
for you to drop into Studio or a Rojo project.

## License

MIT — see `LICENSE`.
