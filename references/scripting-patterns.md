# Scripting Patterns & Luau Idioms

Practical, current (2026) Luau patterns. Where you're unsure whether one of
these is still accurate, verify per `self-research-protocol.md` rather than
trusting this list blindly — APIs do shift.

## Use `task` library, not legacy globals

- `task.wait(n)` instead of `wait(n)` — `wait` is throttled/deprecated in
  behavior and less precise.
- `task.spawn(fn, ...)` instead of `spawn(fn)` — runs immediately, not deferred
  to next resumption cycle, which avoids a class of race conditions.
- `task.delay(n, fn, ...)` instead of `delay(n, fn)`.
- `task.cancel(thread)` to cleanly cancel a spawned thread instead of leaving it
  dangling.

## Signals over polling

Don't poll a condition in a `while true do task.wait() end` loop when an event
exists. `Character.Humanoid.Died`, `Players.PlayerAdded`,
`Instance:GetPropertyChangedSignal("Property")`, and custom Signal
implementations (e.g. a small `BindableEvent` wrapper or a lightweight Signal
module) all exist so gameplay code reacts to state changes instead of checking
them every frame. Polling wastes CPU and introduces latency equal to the poll
interval.

## Connections must be cleaned up

Every `:Connect()` needs a matching `:Disconnect()` (or use
`Instance:GetPropertyChangedSignal` scoped to an object's lifetime with
`Destroying` to auto-clean). A connection left dangling on a destroyed
character/tool/UI element is a memory leak and, worse, a hidden bug where old
handlers fire after the object should be gone. Use a `Maid`/`Trove` pattern
(collect connections in a table, disconnect them all when the owner object is
destroyed) rather than tracking each one manually — this is the single most
common source of "why is this happening twice" bugs in Roblox codebases.

## Object pooling for anything spawned frequently

Bullets, hit-effects, damage-number labels, footstep particles — anything
created and destroyed many times per second should be pooled (a fixed set of
instances reused via `:Clone()` once, then hidden/shown and repositioned)
rather than `Instance.new()` + `:Destroy()` on every use. Constant
instantiation/destruction is a top cause of frame drops in combat-heavy games.

## Common pitfalls to actively check for

- **Nil character/humanoid access**: `player.Character` can be nil (not yet
  spawned, or mid-respawn). Always guard: `local char = player.Character; if
  not char then return end` before indexing into it.
- **Wrong `WaitForChild` usage**: `WaitForChild` without a timeout will yield
  forever if the instance never appears — always pass a timeout in
  production code and handle the nil-return case, or you get a script that
  silently hangs.
- **`FindFirstChild` vs `WaitForChild`**: use `FindFirstChild` when the instance
  might legitimately not exist yet and you want to check without blocking;
  `WaitForChild` when it's guaranteed to eventually exist and you're willing to
  yield.
- **Firing remotes before the client has loaded**: gate feature-critical
  remotes behind a "client ready" signal rather than assuming
  `PlayerAdded`/`CharacterAdded` means the client-side controller is already
  listening.
- **Using `pairs` on arrays where order matters**: `pairs` doesn't guarantee
  order; use `ipairs` (or a numeric `for i = 1, #t` loop) for sequential
  arrays.
- **Deep-nested `if` chains for state**: prefer explicit state machines
  (a state enum + a dispatch table) over deeply nested conditionals for things
  like character states (idle/running/attacking/stunned) — much easier to
  extend and debug.

## Debugging workflow

Prefer `execute_luau` (Studio MCP) to run a small snippet directly in the live
DataModel to confirm a hypothesis (e.g. print a property value) rather than
guessing from reading code alone. After a change, use `start_stop_play` to
playtest and `get_console_output` to check for warnings/errors before reporting
the task done — reading code is a trace, not a verification.
