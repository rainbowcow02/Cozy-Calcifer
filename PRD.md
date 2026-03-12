# Cozy Calcifer — Character Animation PRD

A whimsical, living flower character with customizable traits in a WYSIWYG editor. This document outlines planned improvements to deepen the sense of life, reactivity, and polish.

The goal of this project is to learn about vibe coding as a non-technical Product Designer and explore what's possible to build.

---

## Tier 1 — Core Life

These features make the character feel alive at rest. Highest priority.

---

### 1. Breathing Pulse

**What it does**
The character's body slowly expands and contracts, as if inhaling and exhaling.

**Behavior**
- Scale oscillates between 1.0 and 1.025 on a ~4-second rhythm
- Slightly out of phase with the existing float animation so they never sync up perfectly
- Always running — visible even when the cursor is still

**Technical approach**
- Add a `breathT` counter to the existing `requestAnimationFrame` loop (alongside `floatT`)
- Apply `scale(1 + 0.025 * Math.sin(breathT * 0.0157))` to `#flower-wrapper`
- Phase-offset from vertical float by π/3 to desync the two rhythms

---

### 2. Button On/Off States

**What it does**
Buttons visually reflect whether their particle type is currently active (particles present on screen).

**Behavior**
- A button appears "on" when its particles are orbiting the character
- A button appears "off" (default state) when no particles of that type exist
- State updates automatically as particles are spawned or expire

**Technical approach**
- Add `activeTypes = new Set()` to track which types currently have live particles
- Update set membership after every spawn and every FIFO removal
- Toggle an `.active` CSS class on matching `.spark-btn` elements
- `.spark-btn.active .btn-box`: `filter: brightness(0.85) saturate(1.3)` + subtle inset shadow
- `.spark-btn.active`: suppress hover scale (button is already committed)

---

## Tier 2 — Reactivity

These features make the character respond to user presence and actions.

---

### 3. Squash & Stretch on Cursor Proximity

**What it does**
When the cursor drifts close to the character, it squishes slightly — widening and shortening — then pops back as the cursor moves away. Classic Disney principle that sells physical weight.

**Behavior**
- Proximity zone: within ~140px of the character center triggers the effect
- At closest: scaleX 1.06 / scaleY 0.94 (subtle, not cartoony)
- Transitions smoothly in and out — no snapping

**Technical approach**
- Compute distance from lerped mouse position to flower center each frame (mouse coords already available)
- Lerp a `squishFactor` toward target (lerp rate ~0.08) for smooth transitions
- Apply as additional scale on `#body-group`, compounding with existing tilt transform

---

### 4. Reaction to Particle Spawns

**What it does**
When a button is clicked and new particles appear, the character does a quick little bounce — a brief squish-then-stretch — as if excited by the new sparkles.

**Behavior**
- Triggered on every button click
- Quick arc: squish down, then stretch up, then settle back to neutral
- Duration: ~12 frames (~200ms) — punchy, not lingering

**Technical approach**
- On button click: set `bounceT = 12`
- Each frame while `bounceT > 0`: apply `scaleY(1 - 0.07 * Math.sin(π * (1 - bounceT / 12)))` to `#flower-wrapper`
- Decrement `bounceT` each frame; compound with breathing scale

---

## Tier 3 — Polish

Refinements that add depth and visual cohesion once the core feel is solid.

---

### 5. Colored Particles

**What it does**
Each particle type gets a thematic color from the design palette, tinting the orbiting sparkles so they feel intentional and part of the visual system.

**Color mapping**

| Particle | Color | Palette var |
|----------|-------|-------------|
| Heart | Peach | `#F4A896` |
| Star1 | Pale yellow | `#F5D878` |
| Star2 | Gold | `#F0C040` |
| Swirl | Periwinkle | `#7B8FD6` |
| Crits | Lavender gray | `#C4C8DC` |

**Technical approach**
- Add a colored overlay `<div>` positioned over each particle `<img>`, using the type's color at ~55% opacity with `mix-blend-mode: multiply`
- Supplement with `filter: drop-shadow(0 0 4px COLOR)` on the particle element for a soft glow

---

### 6. Petal Independent Wobble

**What it does**
The petals (scalloped edge layer) oscillate on their own slow frequency, as if catching a gentle breeze independently of the body.

**Behavior**
- Subtle rotation (±1.8°) and slight horizontal squeeze (±0.8%)
- ~5.2-second period — slower and distinct from the ~3.5s body float
- Always running in the background; most noticeable at rest

**Technical approach**
- Apply `rotate` + `scaleX` oscillation to the `bodyscalop` image element directly
- Use `breathT * 0.0121` for the period (reuses existing counter, different multiplier)
- `transform-origin: center` so the petal fans naturally from the center

---

## Out of Scope

- New character parts or SVG assets
- Sound / audio
- Mobile / touch interactions
- Any changes to the existing float, parallax, gaze, or blink systems
