# Self-Research Protocol

The single biggest difference between "AI slop" Roblox code and code a veteran
dev would ship is confidence calibration: a senior dev knows the boundary of
what they remember cold versus what they'd quickly check, and checks the
second category instead of guessing. Apply the same standard here.

## When to stop and research (not just write code)

- An API's exact method signature, argument order, or return type/shape you
  aren't certain of (e.g. does `TweenInfo.new` take a table or positional
  args; does this property still exist on this class in current Roblox).
- Whether a pattern is still the *current* recommendation — Roblox
  deprecates/replaces APIs over time (e.g. legacy `wait`/`spawn` vs the
  `task` library; older `Instance.new(className, parent)` two-arg form is
  discouraged in favor of setting `.Parent` after construction).
- Anything touching money, saved player data, or moderation/compliance rules
  (Developer Product receipts, DataStore limits, Roblox Terms of Use/Community
  Standards implications) — being wrong here has real consequences.
- A DevForum-only convention or a platform limit (rate limits, size limits,
  quota) that isn't something you'd reliably know precisely from training
  alone.
- The user describes a symptom you don't have a confident diagnosis for
  (e.g. "sometimes this remote just doesn't fire") — research known causes
  rather than guessing at the first plausible one.

## How to research, in priority order

1. **Studio MCP's built-in docs tools**, if connected: use the `skill` tool
   (retrieves curated best-practice/reference material for specific topics
   like debugging or performance) and `http_get` (fetches from the allowed
   Roblox documentation domains — Engine API reference, Creator docs, Cloud
   API, performance guides) with keyword search. These are the most direct,
   current source and don't require leaving the tool loop.
2. **WebSearch** for anything not covered by the above, or for current
   DevForum discussion of a specific edge case/bug — search with a specific,
   current-year query (e.g. "Roblox TweenService EasingStyle 2026" rather than
   a vague one) so results aren't stale.
3. **Read the actual project first** before assuming an API works a certain
   way in this specific codebase — a property or convention might be
   overridden/wrapped locally (e.g. a custom Signal module that isn't the
   built-in `BindableEvent`).

## What to do with uncertainty you can't resolve

If research genuinely doesn't turn up a confident answer (rare, but it
happens — very new API, ambiguous docs), say so plainly rather than
presenting a guess as fact: state what you tried, what you found, and what
your best-guess approach is, and flag it as unverified. This mirrors the
required verification-tier honesty (executed / traced / unverified) — an API
detail you didn't confirm is "unverified," not "should work."

## Don't over-research

This protocol is for genuine uncertainty, not every line of code. Common,
stable APIs (`TweenService`, `RemoteEvent:FireServer`, basic `CFrame` math,
`Players.PlayerAdded`) don't need a search each time — that would waste time
and tokens for no benefit. Calibrate: if you'd bet confidently on the answer,
proceed; if you're guessing at the shape of something, check it first.
