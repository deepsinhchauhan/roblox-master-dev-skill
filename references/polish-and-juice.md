# Polish & "Juice" — the 15-year-dev feel

"Juice" is the layer of feedback between an action happening and the player
perceiving it. A veteran dev adds this by default, not when asked. If a system
you're building or touching doesn't have an entry here covered, ask: *what
does the player see, hear, and feel in the 200ms after this happens?*

## Motion — nothing should just "appear" or "disappear"

- Use `TweenService` for any state change a player will notice: UI opening/
  closing, a button press, a value changing, an object appearing. Default to
  `Enum.EasingStyle.Quad` or `.Back`/`.Elastic` for UI (never `Linear` for
  anything player-facing — linear motion reads as robotic).
- Standard durations: micro-interactions (button hover/press) ~0.1-0.15s, panel
  open/close ~0.25-0.35s, big reveals ~0.5s. Faster than this feels twitchy;
  slower feels sluggish.
- Stagger related elements instead of animating a whole group at once (e.g. a
  list of inventory items fading/scaling in 0.03s apart each) — this alone is
  one of the highest-impact, lowest-effort changes for "feeling professional."

## Camera and screen feedback

- Screen shake on impactful moments (taking damage, explosions, landing from a
  fall) — small magnitude, short duration, decaying over time, never applied to
  the actual CFrame directly but as an offset so it composes with normal camera
  movement.
- Camera FOV kick (brief FOV increase) on speed boosts/dashes reads as "fast"
  far more effectively than the raw speed number.
- Hit-stop (briefly slowing or freezing time for a few frames on a heavy hit)
  sells weight in combat — used sparingly, it's very effective.

## Audio — every meaningful action gets a sound

- UI: hover, click, open, close, error/denied, success/purchase, notification.
- Gameplay: footsteps (varied by surface material if feasible), hits (varied,
  not the exact same sound every time — pick from 2-3 variants with slight
  pitch randomization `SoundId` pool + `PlaybackSpeed` jitter), pickups, level-
  up/reward stingers.
- Silence is a bug in an interactive system, not a neutral default — if an
  action has no sound, that's usually an oversight, not a choice.

## Visual feedback for numbers/state

- Damage/heal numbers as floating, tweened, fading `BillboardGui`/2D labels,
  not just a health bar tick — players want to *see* the number.
- Cooldowns rendered as a radial or linear fill (`ImageLabel` with
  `Image.ImageRectSize`-style radial mask, or a simple frame width tween) on
  the ability icon itself, not just a disabled gray button with no timer.
- Currency/XP changes: animate the counter counting up/down over ~0.3-0.5s
  rather than snapping instantly — a snapping number reads as broken/laggy even
  when it isn't.

## Loading and transitions

- Never let the player stare at a frozen or empty screen. Use a loading screen
  with actual progress (asset preload via `ContentProvider:PreloadAsync`) for
  anything that takes more than ~0.5s, or at minimum a spinner/skeleton state.
- Fade transitions (a full-screen black `Frame` tweened in alpha) between major
  state changes (teleporting, respawning, entering a new area) mask any pop-in
  and feel intentional rather than jarring.

## Particles and effects

- Use `ParticleEmitter`/`Beam`/`Trail` for anything moving fast or impactful
  (projectiles, dashes, explosions) rather than relying on the base part alone.
- Keep effect lifetimes short and pooled (see scripting-patterns.md) — a
  particle system that never gets cleaned up is both a visual bug (effects
  pile up) and a performance bug.

## Consistency is part of polish

- One color palette, one font (or a deliberate 2-font pairing: one for
  headers, one for body), one corner-radius value, one stroke-width value
  across ALL UI in the game. Mixed defaults (some buttons rounded, some
  square; some using the Roblox default gray) is the single most common
  "obviously AI-made / amateur" tell — fix this even if not explicitly asked,
  and mention it was fixed.
- Reuse one small UI component library (a `Button` module that applies hover/
  press states and sound consistently) instead of hand-rolling each button's
  behavior inline — this is what makes 50 buttons in a game all feel the same.
