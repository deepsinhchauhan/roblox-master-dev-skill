# Performance Optimization

Roblox runs on everything from a flagship gaming PC to a five-year-old
Android phone. "Runs smoothly on my machine" is not a standard — build with
the low end in mind by default.

## Scripting hot paths

- Never allocate tables/instances inside a `Heartbeat`/`RenderStepped`/
  `Stepped` connection or a tight loop — allocate once outside and mutate.
  Per-frame garbage collection pressure is one of the top causes of stutter.
- Avoid unnecessary `Heartbeat`/`RenderStepped` connections left running for
  things that only need to react to events. If a system can be event-driven
  (see scripting-patterns.md, Signals over polling), it should be — every
  active per-frame connection is fixed overhead, always.
- Cache expensive lookups (`WaitForChild` chains, `:FindFirstChild` results,
  `game:GetService(...)`) in locals/module-level variables rather than
  re-resolving them every call.
- Prefer `table.create(n)` when you know an array's final size up front, to
  avoid repeated table resizing.
- Debounce/throttle anything that could fire rapidly from player input or
  physics touch events (`Touched` can fire many times per second) — a raw,
  unguarded `Touched` handler is a classic source of duplicate-trigger bugs
  and wasted work.

## Instance / part count

- Use `MeshPart`/unions sparingly for large structures — high part counts hurt
  both rendering and physics. Where possible, model complex static geometry as
  fewer, larger meshes instead of hundreds of unioned blocks.
- Set `Anchored = true` on anything that doesn't need physics — every
  unanchored part is simulated by the physics engine even if it never visibly
  moves.
- Use `StreamingEnabled` for large worlds so distant parts aren't loaded/
  simulated client-side; be mindful that scripts relying on far-away instances
  need to handle them not yet existing (`WaitForChild` with timeout, or design
  around `Instance.Parent` being nil until streamed in).

## Rendering

- Keep texture/decal resolutions reasonable — oversized textures on small
  objects waste memory and load time for no visible gain.
- Use `LevelOfDetail`-style thinking manually: simplify or hide detail on
  distant objects if the game has large view distances.
- Limit the number of live `ParticleEmitter`s/`Beam`s active at once — a
  combat scene with unbounded particle spawn will drop frames on lower-end
  devices first; cap concurrent effects and reuse/pool them (see
  polish-and-juice.md + scripting-patterns.md, object pooling).

## Networking

- Don't replicate more than needed: mark not-yet-relevant instances/parts
  appropriately, and avoid firing remotes with large payloads at high
  frequency (throttle continuous data as described in architecture.md).
- Batch related state updates into a single remote firing/table payload
  instead of many small separate remote calls in the same frame.

## Diagnosing a "laggy game" report

1. Ask (or infer from context) whether it's client-side stutter, server lag,
   or network latency — the fixes differ completely.
2. If Studio MCP is connected, use `get_studio_state`/`get_console_output`
   and consider a `subagent` playtest run to reproduce and observe, rather
   than guessing at the cause from static code reading alone.
3. Check for the usual suspects in order of likelihood: unthrottled
   `Heartbeat`/`Touched` handlers, unpooled instance creation in a loop,
   unanchored physics parts in large numbers, oversized textures/meshes,
   and remote spam.
4. Fix the actual bottleneck found, not the first thing that looks
   suspicious — profile/observe before rewriting, per the user's standing
   rule to fix the bug, not the symptom.
