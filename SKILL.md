---
name: roblox-master-dev
description: >
  Build and polish Roblox experiences (games) in Luau to the standard of a veteran
  (10-15+ year) Roblox developer — clean client/server architecture, buttery-smooth
  feel ("juice"), professional UI/UX, and solid performance on mobile and low-end
  devices. Use this skill whenever the user is working in Roblox Studio, writing
  Luau/Lua scripts, building a Roblox game/place/experience, designing Roblox UI
  (ScreenGui, Frames, ViewportFrames), setting up RemoteEvents/RemoteFunctions,
  doing DataStore/data-saving work, optimizing a laggy Roblox game, adding effects/
  animations/sound/camera work, or asks to "make my Roblox game better," "make it
  feel more professional," "add polish," "improve the game 100x," or similar — even
  if they don't say the word "skill" or name a specific system. Also use this when
  the user is uncertain how something works in Roblox/Luau and needs the agent to
  look it up rather than guess (see the Self-Research Protocol reference).
compatibility: Roblox Studio open on the user's machine, ideally with the built-in
  Roblox Studio MCP server connected (script_read/multi_edit/execute_luau/etc.) so
  changes can be read and applied directly. Works without MCP too — Claude can just
  write .lua/.luau files (e.g. for a Rojo project) instead.
---

# Roblox Master Dev

## Why this skill exists

Most AI-generated Roblox code is functionally correct and looks like it. It works,
but a player can feel the difference in the first ten seconds: parts pop into
existence with no tween, UI has default Roblox gray buttons with no hover state,
damage numbers just... appear, and there's no camera shake, no sound feedback, no
easing anywhere. A dev with 10-15 years on the platform doesn't write more code —
they write code that *respects timing, feedback, and edge cases* by default,
because they've been burned by skipping those things before.

This skill exists to make that experience the default output, not something the
user has to ask for line by line. Treat "make it work" and "make it feel good" as
the same request, always — never ship the flat version and wait to be asked to add
juice.

## Core operating principle: research before you guess

Roblox's API surface, best practices, and even service names change over time, and
Luau has quirks (metatables, `task` library vs `wait`, `Instance.new` two-argument
form deprecated, etc.) that are easy to get subtly wrong. If you are not
**certain** — not "pretty sure," certain — how something works (an API's exact
signature, whether a property still exists, the current recommended pattern for
something like proximity prompts or physics, a rate limit, a security caveat),
stop and research it before writing code. Read `references/self-research-protocol.md`
for exactly how to do this with the tools available (Studio MCP's `http_get`/`skill`
tools, the DevForum, WebSearch, or the Creator Docs). Guessing at Roblox APIs
produces code that looks plausible and throws a runtime error the first time it
runs — that failure mode is worse than the extra 30 seconds of looking it up.

## Workflow

1. **Understand the ask and the current state.** If Studio MCP is connected, use
   `search_game_tree` and `inspect_instance` to see what already exists before
   writing anything — don't assume the project's structure. If working from files
   (Rojo-style), read the relevant scripts fully first, per the user's own
   standing rule: read before editing, grep for callers/dependents before editing.
2. **Pick the right reference(s) below** for the part of the game you're touching.
   Don't load all of them for a small task — a script bugfix doesn't need the UI
   reference open.
3. **Write it like a senior dev would**, meaning: correct architecture (see
   `references/architecture.md`), then layer in feel and polish by default (see
   `references/polish-and-juice.md`) — not as an afterthought.
4. **Check performance and mobile behavior** before calling it done, per
   `references/performance-optimization.md`. "Works on my PC" is not done.
5. **Verify.** Prefer `execute_luau` (if MCP connected) to actually run a snippet
   and confirm behavior, or `get_console_output` after a playtest
   (`start_stop_play`), over declaring something fixed on inspection alone. Name
   the verification tier honestly: executed / read-only trace / unverified.
6. **Report concisely**: what changed (paths / instance names), how it was
   verified, and any risk or follow-up left open — no restating the request, no
   "I've successfully..." preambles.

## Reference index — read the one(s) relevant to the task

| File | Read this when... |
|---|---|
| `references/architecture.md` | Setting up or refactoring client/server structure, module scripts, RemoteEvents/Functions, data flow, folder layout |
| `references/scripting-patterns.md` | Writing gameplay logic, Luau idioms, common bugs/pitfalls, `task` library, signals, object pooling |
| `references/polish-and-juice.md` | Anything player-facing: tweens, camera work, sound, particles, hit feedback, UI motion — the "15-year dev feel" checklist |
| `references/ui-ux-design.md` | Building ScreenGuis, menus, HUDs, inventories — layout, responsiveness across screen sizes, accessibility, professional visual language |
| `references/performance-optimization.md` | The game is laggy, mobile performance, memory leaks, streaming, part counts, script profiling |
| `references/data-and-security.md` | DataStores, player data saving, remote event validation, anti-exploit, monetization (Developer Products/Gamepasses) |
| `references/self-research-protocol.md` | Any time you're not 100% sure an API, property, or pattern is correct/current — read this first, not last |
| `references/mcp-tools.md` | Reference for what the Studio MCP tools do and when to use each one (script_read, execute_luau, subagent, playtest tools, etc.) |

## The "100x improvement" mandate

When asked to improve an existing game/system "100x," "make it perfect," or
similar, don't just patch the one bug mentioned. Do a pass across all of these
dimensions and report what you touched:

- **Feel**: transitions, easing, camera, hit-stop/screen-shake, sound on every
  meaningful action, loading states instead of pop-in
- **Visual consistency**: one font pairing, one color palette, consistent corner
  radii/stroke widths across all UI, not mixed defaults
- **Robustness**: nil-checks on player/character references, remote validation
  server-side (never trust the client), reconnection/respawn handling
- **Performance**: no per-frame allocations in hot loops, StreamingEnabled
  considerations, part counts, unnecessary `Heartbeat`/`RenderStepped` connections
  left running
- **Onboarding**: does a brand-new player understand what to do in the first 10
  seconds, or is there a wall of unexplained UI

Flag which of these you actually changed vs. which are out of scope for the
current request, in 3 lines or fewer — don't silently expand scope on a small ask,
but do surface what's missing as a one-line note per the user's standing
preference on unrelated problems.
