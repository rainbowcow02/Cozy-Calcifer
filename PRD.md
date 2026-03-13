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

### 2. Button On/Off States ✅ Done

**What it does**
Buttons visually reflect whether their particle type is currently active (particles present on screen).

**Behavior**
- A button appears "on" when its particles are orbiting the character
- A button appears "off" (default state) when no particles of that type exist
- Toggle is driven directly by click — each click flips the state

**Technical approach (as built)**
- `btn.classList.toggle('active')` on each `.spark-btn` click
- `.spark-btn.active .btn-bg`: background shifts from `#8897F4` to `#12227B` (dark navy fill)
- Active and inactive states are purely CSS-driven; no separate tracking set needed

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

### 4. Reaction to Particle Spawns ✅ Done

**What it does**
When a button is clicked, the character performs one of four expressive bounce animations in sequence, cycling with each click.

**Behavior**
- Triggered on every button click (toggle on or off)
- Four states cycle in order: lean-right → lean-left → axel (forward spin) → axel-reverse (backward spin) → repeat
- Each state has 4 phases: wind-up → launch → fall/overshoot → spring to rest
- Right/left: ~24 frames (~400ms) — leans and tips with squash & stretch
- Axel / axel-reverse: ~41 frames (~680ms) — dramatic wind-up, full 360° spin, clean landing

**Technical approach (as built)**
- `BOUNCE_STYLES = ['right', 'left', 'axel', 'axel-reverse']` cycles via `bounceStyleIdx % 4`
- Each style shares a 4-phase easing system (`ein`, `eout`, `eio`) applied to `sx`, `sy`, `dy`, `rot`
- Axel styles use same arc/timing as each other; `axelDir` flips the sign of all rotation values for reverse
- `getBounceScale()` called each frame; transform composed onto `#flower-wrapper` with float + drift
- Face lags the body via a secondary lerp (`faceOffY`) for a subtle secondary motion feel

---

## Tier 3 — Polish

Refinements that add depth and visual cohesion once the core feel is solid.

---

### 5. Colored Particles ✅ Done

**What it does**
Each particle type gets a thematic color from the design palette, tinting the orbiting sparkles so they feel intentional and part of the visual system.

**Color mapping**

| Particle | Color | Palette var |
|----------|-------|-------------|
| Heart | Peach/coral | `#F4A896` |
| Star1 | Pale yellow | `#F5D878` |
| Star2 | Rich gold | `#F0C040` |
| Swirl | Periwinkle | `#7B8FD6` |
| Crits | Confetti (random per shape) | `#7B8FD6`, `#F4A896`, `#F5D878`, `#F0C040`, `#C4C8DC`, `#9AA5D8` |

**Technical approach (as built)**
- All particle SVGs share a pale-yellow base fill (`#FEE69A`)
- Color is applied via CSS `filter` on each `<img>`: `hue-rotate()` shifts hue, `brightness()` adjusts lightness
- `PARTICLE_FILTER` map: heart `hue-rotate(323deg)`, star1 `brightness(0.94)`, star2 `brightness(0.82)`, swirl `hue-rotate(178deg) brightness(0.88)`
- Crits particles are dynamically built SVGs — each of the 4 shapes gets a random color from `CONFETTI_COLORS` on every spawn, creating a confetti effect

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
