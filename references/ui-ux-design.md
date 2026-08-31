# UI/UX Design for Roblox

## Layout that survives every device

Roblox players span phone, tablet, console, and desktop with wildly different
aspect ratios. A senior dev never hardcodes pixel offsets/sizes for anything
meant to be seen on all platforms.

- Use `UDim2` scale components (the 0-1 fractional part) for sizing/position
  relative to the screen, and offset (pixel part) only for fine-tuning that
  should stay constant (e.g. a fixed icon size inside a scaling container).
- Use `UIAspectRatioConstraint` for elements that must keep proportions (square
  icons, card layouts).
- Use `UIListLayout`/`UIGridLayout` + `UIPadding` instead of manually
  positioning siblings — manual positioning breaks the instant a screen size
  changes or an item is added/removed.
- Test (or at least reason through) both a 16:9 desktop and a tall phone
  aspect ratio for any new HUD element — does it overlap the chat, the topbar,
  or get cut off at the bottom on a notched phone?
- Respect `GuiService:GetGuiInset()` / safe-area considerations so UI doesn't
  sit under the Roblox topbar.

## Visual hierarchy

- Establish clear primary/secondary/tertiary actions on any screen (one bold
  primary button, not five equally-weighted buttons).
- Use size and color weight, not just position, to signal importance —
  a shop's "Buy" button should visually outweigh a "Cancel" nearby.
- Group related controls with consistent spacing (an 8px/16px/24px spacing
  scale is a reasonable default) rather than ad hoc gaps.

## Input handling

- Support both mouse/keyboard, touch, and gamepad if the game targets mobile/
  console — use `UserInputService` to detect the last input type and adapt
  hints (e.g. show "Tap to jump" vs "Press Space") rather than assuming PC.
- Make touch targets large enough (roughly 44x44 px minimum) — a button sized
  for a mouse cursor is often too small to reliably tap.
- Always provide a visible focus/selection state for gamepad navigation if
  supporting console (`GuiObject.Selectable`, `SelectionImageObject`).

## Feedback states every interactive element needs

Hover (PC only), press/active, disabled, loading, error/success. A button that
only has a default and a click handler with no visual press feedback feels
unresponsive even if it functions correctly — see `polish-and-juice.md`.

## Accessibility basics

- Sufficient contrast between text and background (don't rely on color alone
  to convey state — pair a color change with an icon/text change too, for
  colorblind players).
- Avoid pure flashing/strobing effects for error states.
- Keep critical text legible at small sizes on mobile — test font sizes at the
  smallest supported screen scale, not just on a desktop preview.

## Structure of a typical HUD build

```
ScreenGui (ResetOnSpawn = false, IgnoreGuiInset as needed)
  MainFrame (UIListLayout or manually laid out zones)
    TopBar (currency, level, settings)
    ActionBar (abilities/hotbar, bottom-anchored)
    Notifications (top-anchored, stacking via UIListLayout, auto-dismiss)
  Modals/ (Shop, Inventory, Settings — each its own Frame, toggled, tweened
           in/out, only one open at a time via a shared "currently open modal"
           state to avoid stacked overlapping UI)
```

`ResetOnSpawn = false` on the main ScreenGui plus manual re-parenting/state
management on respawn is the standard pattern — avoids UI flicker/reset every
time the character dies.
