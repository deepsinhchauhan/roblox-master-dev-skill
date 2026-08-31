# Architecture

A veteran Roblox dev's codebase is boring in the best way: predictable folder
layout, a clear line between client and server, and no gameplay-critical logic
that trusts the client. New scripts should slot into this shape rather than
inventing a new one per feature.

## Folder layout (Rojo-style, maps 1:1 to the DataModel)

```
ServerScriptService/
  Services/          -- one ModuleScript per game system (DataService, ShopService, CombatService...)
  init.server.luau    -- boots services in a defined order
ServerStorage/
  -- server-only assets/data that must never reach the client (drop tables, secrets)
ReplicatedStorage/
  Shared/             -- ModuleScripts used by BOTH client and server (constants, types, pure logic)
  Remotes/            -- RemoteEvents/RemoteFunctions, organized by feature, never loose in one folder
  Assets/             -- models/meshes referenced by both sides
StarterPlayer/
  StarterPlayerScripts/
    Controllers/      -- one ModuleScript per client system (CameraController, HudController...)
  StarterCharacterScripts/
StarterGui/
```

Why: when everything gameplay-critical lives in `Services` and is only ever
*triggered* by remotes (never directly mutated by client code), there is no
special-casing later for "wait, can an exploiter call this directly?" — the
answer is already "yes, and the service validates for it" by construction.

## Client/server split

- **Server owns truth.** Player currency, inventory, health, cooldowns — all
  server-side state, server-side authority. The client only ever *requests*
  changes via a RemoteEvent/RemoteFunction and *renders* whatever the server
  confirms.
- **Never trust remote arguments.** Every RemoteEvent handler on the server
  re-validates: is this the right type, is the player allowed to do this right
  now (cooldown/state check), is the referenced instance actually theirs. A
  client can fire any remote with any arguments at any time — treat every remote
  as a public API endpoint, because that's what it is.
- **Batch/throttle remotes.** Don't fire a RemoteEvent every frame for continuous
  things (e.g. a joystick position) — throttle to a fixed rate (e.g. every
  0.1s or on `Heartbeat` with an accumulator) or use unreliable remotes
  (`UnreliableRemoteEvent`) for fire-and-forget cosmetic data like VFX cues.
- **Naming convention**: prefix remotes with the verb the client is requesting
  (`RequestPurchase`, `RequestEquip`) so it's clear from the name alone that the
  server should validate, not just apply.

## ModuleScript patterns

- One module = one responsibility. A module returning a table of unrelated
  helper functions ("Utils") is a smell — split by domain (`TableUtil`,
  `MathUtil`, `StringUtil` are fine because each is a real domain).
  Prefer classes when a module represents an entity (Weapon, Enemy, Shop) using a
  standard Luau OOP pattern (`local Class = {}; Class.__index = Class`).
- Return the module table itself for singletons/services; return a constructor
  function (`.new()`) for anything instanced (per-player, per-tool, per-enemy).
- Use `--!strict` at the top of modules where feasible and real Luau types
  (`type Weapon = { damage: number, ... }`) — this catches whole classes of bugs
  (nil-arithmetic, wrong argument order) before runtime, and a senior dev's
  codebase leans on this instead of scattered `assert`s.

## Data flow example (a shop purchase)

1. Client fires `Remotes.Shop.RequestPurchase:FireServer(itemId)`.
2. `ShopService` handler: validate `itemId` exists in the catalog, validate the
   player has enough currency (server-side balance, not anything sent by
   client), validate the player doesn't already own it if it's a one-time item.
3. On success: mutate server-side data via `DataService`, then fire
   `Remotes.Shop.PurchaseConfirmed:FireClient(player, itemId)` so the client can
   update its UI and play a success animation/sound.
4. On failure: fire a `PurchaseDenied` remote with a reason code (not a raw
   string to translate ad hoc) so the client can show the right message.

This round-trip — request, validate, confirm/deny, render — is the shape of
almost every player action in a well-built Roblox game. Reuse it rather than
inventing a bespoke pattern per feature.
