# Studio MCP Tools Reference

If the built-in Roblox Studio MCP server is connected (see the game's earlier
setup: Assistant → Manage MCP Servers → "Enable Studio as MCP server"), these
tools let you act directly on the live Studio session instead of only
generating files blind. Prefer these over guessing at the project's structure.

## Explore before you edit

- `search_game_tree` — explore the instance hierarchy (filter by path/type/
  keyword). Use this first on any non-trivial task to see what actually
  exists rather than assuming a folder layout.
- `inspect_instance` — full detail on one instance (properties, attributes,
  children summary). Use before editing a script/object you haven't read yet.
- `script_search` — fuzzy-find a script by name.
- `script_grep` — search a string/pattern across all scripts (use this the
  same way you'd grep a codebase for callers/dependents before editing
  something shared).
- `script_read` — read a script (whole or a line range) via dot-notation path.

## Editing

- `multi_edit` — apply one or more edits to a script in one operation; creates
  the script if the path doesn't exist yet. Requires a `datamodel_type`
  (Edit/Client/Server) — pick the one matching where the script actually
  lives/runs.

## Running code directly

- `execute_luau` — run Luau in Studio and get the result/error back. This is
  the fastest way to verify a hypothesis (e.g. print a property, test a
  calculation) instead of reasoning about it purely from reading code.

## Playtesting and verification

- `get_studio_state` — current play state and available datamodel types.
- `start_stop_play` — start/stop playtesting.
- `get_console_output` — read the Studio output log (check for
  warnings/errors after a change, before declaring it done).
- `screen_capture` — grab the current viewport, optionally from a custom
  camera angle — useful to visually confirm a UI/visual change actually looks
  right rather than just compiling.
- `character_navigation`, `user_keyboard_input`, `user_mouse_input` — simulate
  a player during playtesting to exercise a flow end-to-end (e.g. walk to an
  NPC, press a key, click a UI button) rather than only unit-testing logic in
  isolation.
- `subagent` (types: `explore`, `playtest`) — hand off a self-contained
  investigation or gameplay-scenario verification to a specialized subagent
  when the task is complex enough to benefit from a dedicated pass.

## Assets

- `generate_mesh`, `generate_material`, `generate_procedural_model`,
  `wait_job_finished` — AI-generate 3D content when the task calls for new
  assets rather than existing ones.
- `search_asset`, `insert_asset` — find and insert existing Creator Store/
  Inventory assets by ID rather than generating from scratch when something
  suitable likely already exists.
- `upload_image`, `store_image` — get images (e.g. a reference for procedural
  model generation) into Roblox's asset pipeline.

## Docs, on demand

- `http_get` — fetch Roblox's own documentation (Engine API, Creator docs,
  Cloud API, performance guides) with keyword search. Use this per
  `self-research-protocol.md` whenever unsure of current API behavior.
- `skill` — retrieve curated best-practice material for specific topics
  (debugging, device simulation, doc search).

## Multiple Studio instances

- `list_roblox_studios` — list connected Studio windows (name, ID, place ID).
  Every tool call takes a `studio_id` — look this up first if more than one
  Studio window might be open, rather than assuming which one is targeted.
