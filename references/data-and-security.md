# Data & Security

## DataStores (player save data)

- Wrap every `DataStore:GetAsync`/`:UpdateAsync`/`:SetAsync` call in `pcall` —
  DataStore calls can and do fail (throttling, outages) and an unhandled
  failure can silently lose player progress.
- Prefer `:UpdateAsync` over `:GetAsync` + `:SetAsync` for anything that reads
  then writes — `UpdateAsync` handles the read-modify-write atomically and
  avoids race conditions from concurrent saves (e.g. a player joining two
  servers in quick succession, or a server crash mid-save).
- Save on `PlayerRemoving` and periodically during the session (not only at
  the end) so a server crash doesn't lose the whole session's progress.
- Use `BindToClose` to attempt a final save when the server is shutting down,
  with a reasonable timeout — don't let it hang the shutdown indefinitely.
- Version/schema your saved data (a `DataVersion` field) so future changes to
  the save shape can migrate old data instead of breaking on load.
- Never store data that should be authoritative (currency, inventory) only in
  a client-readable place — server-side memory + DataStore is the source of
  truth; the client only ever sees a copy for rendering.

## Remote security (assume every client is hostile)

- Every RemoteEvent/RemoteFunction handler validates on the server: correct
  argument types, sane value ranges, that the requesting player is actually
  allowed to do this right now (owns the item, has the currency, isn't on
  cooldown, is in the right game state).
- Never trust a client-sent price, damage amount, or position for anything
  that affects another player or persistent state — recompute or look up the
  authoritative value server-side instead of accepting what the client sent.
- Rate-limit remotes that could be spammed (a purchase button mashed, an
  attack fired every frame) — track last-fired time per player server-side and
  ignore/penalize calls that are too frequent.
- Sanity-check physics-adjacent client reports (e.g. a client claiming "I hit
  this enemy at this position") against server-known state (distance,
  line-of-sight, cooldown) rather than trusting it outright — this is how
  speed-hacks and aimbots are mitigated at the gameplay-logic level.

## Monetization (Developer Products / Gamepasses)

- Always process `MarketplaceService.ProcessReceipt` idempotently and return
  `Enum.ProductPurchaseDecision.PurchaseGranted` only after the reward has
  actually been durably applied (saved) — if the callback errors or the
  server doesn't confirm, Roblox will retry the receipt, so handle duplicate
  delivery gracefully (check "already granted" before granting again for
  non-consumable rewards).
- Verify gamepass ownership server-side via
  `MarketplaceService:UserOwnsGamePassAsync` at the moment it matters (e.g. on
  join, or before granting a perk) rather than caching an ownership check
  client-side and trusting it later.

## When unsure

DataStore and monetization APIs have real edge cases (throttling limits,
receipt retry semantics, gamepass caching behavior) that are easy to get
subtly wrong and expensive to get wrong in production (lost player data, lost
revenue, or granting purchases for free). If genuinely unsure about current
API behavior here, this is exactly the kind of thing to verify via
`self-research-protocol.md` before shipping — don't guess on money or save
data.
