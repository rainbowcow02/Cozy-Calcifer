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

### 3. Shadow + Highlight Dynamic Movement ✅ Done

**What it does**
The body's shadow and highlight layers have their own sense of movement — subtle independent drift in the idle state, and enhanced physical reaction during jumps and spins.

**Behavior**
- Idle: each layer drifts with the virtual light source (two overlapping sine waves, ~90s non-repeating orbit) and follows cursor parallax — shadows flee the light, highlights chase it
- During bounce: shadows and highlights behave with distinct physical weight across all 4 bounce phases

**Technical approach (as built)**
- Virtual light source orbits the character using two overlapping sine waves at distinct frequencies — creates organic, non-repeating motion
- 5 parallax layers (2 shadow, 1 accent, 2 highlight) each have unique `cursor`, `light`, `bloom`, and `wrap` values
- `getBounceScale()` now returns `phase` (1–4), `phaseP` (0–1 eased progress), and `bdir` (lean direction) for phase-aware behavior
- **Inertia lag:** shadows lerp at 0.025 (heavy/slow), highlights at 0.38 (light/snappy) — the gap between them sells the sense of mass
- **Phase 1 (squish):** shadow balloons outward 60%, highlight compresses 30%
- **Phase 2 (airtime):** shadow shrinks 22% (body lifts away), highlight blooms 18%
- **Phase 3 (landing):** shadow slams back and overshoots, highlight bounces back from 118% → 104%
- **Phase 4 (spring):** both layers settle toward neutral with a residual overshoot
- **Lateral drift:** during lean bounces, shadow drifts in lean direction, highlight pulls away (opposite)
- **Landing flash:** 10-frame highlight scale spike fires on phase 2→3 transition — reads as a pulse of light on impact
- `wrapRot` tilts each layer to conform to the sphere surface (shadows and highlights tilt opposite directions)

---

## Tier 2 — Reactivity

These features make the character respond to user presence and actions.

---

### 4. Squash & Stretch on Cursor Proximity

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

### 5. Reaction to Particle Spawns ✅ Done

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

### 6. Load-In Jump Animation

**What it does**
On page load, the character does a quick excited jump to animate in — a "ta-da!" entrance rather than just appearing.

**Behavior**
- Single bounce auto-triggers on first frame
- Float and parallax systems slightly delayed so the jump reads clearly before settling into idle drift

**Technical approach**
- Reuse existing `bounceT` / `bounceStyle` system — auto-trigger without waiting for a click

---

### 7. Rotating Load-In Greeting

**What it does**
The character says a short cozy greeting on load, cycling through a few options each visit — before the user even clicks anything.

**Behavior**
- Pool of ~5 greetings in Calcifer's cozy/casual voice (i.e. "hey there 🌸", "oh, you're here! ☕", "hi hi hi 🌼", "yo! 😎", "what's cookin'? 🔥", "*ੈ✩༺hello༻*ੈ✩" )
- Randomly selected each load
- Auto-triggers ~800ms after page load so the entry jump completes first
- Typewriter animation, then bubble auto-closes — no options panel shown
- Options only appear when user clicks the character intentionally

---

### 8. Response Animation (Talking State)

**What it does**
When the character is responding to a chat option, it plays a bigger, more expressive animation — swaying side-to-side or an excited talking motion — so the response feels grand and alive.

**Behavior**
- Gentle side-to-side sway and/or subtle scale pulse while the typewriter is running
- Different from the idle float — more expressive, more present
- Clears when the chat bubble auto-closes

**Technical approach**
- Add `talkingState` boolean flag
- When true, layer a sin-wave X sway on top of normal float in the animation loop

---

## Tier 3 — Polish

Refinements that add depth and visual cohesion once the core feel is solid.

---

### 9. Colored Particles ✅ Done

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

### 10. Star Color Differentiation

**What it does**
Star1 and star2 are currently too similar in color. Make them visually distinct so each feels like its own type.

**Behavior**
- Star1: brighter, cooler — white-yellow or light blue/silver shimmer
- Star2: warmer, richer — amber or orange-gold

**Technical approach**
- Adjust `hue-rotate` and `brightness` filter values for `star1` and `star2` in `PARTICLE_FILTER`

---

### 11. Petal Independent Wobble ✅ Done

**What it does**
The petals (scalloped edge layer) oscillate on their own slow frequency, as if catching a gentle breeze independently of the body.

**Behavior**
- Subtle rotation (±1.8°) on a dual-frequency sway — two overlapping sine waves so motion never feels mechanical
- Petals lean slightly toward the cursor (bias added to sway center)
- Subtle parallax shift opposite to cursor — petals feel like they sit on a slightly deeper plane
- Slow breath scale pulse (~75s period) layered on top
- Always running in the background; most noticeable at rest

**Technical approach (as built)**
- Petal layers grouped in `#petal-group` div with `transform-origin: 50% 78%` so sway pivots near the base
- `petalSway = sin(floatT * 0.018) * 1.8 + sin(floatT * 0.011) * 0.9 + swayBias` — dual-frequency rock with cursor lean
- `petalParX/Y = -lerpX/Y * 0.07` — subtle parallax shift opposite to cursor for depth illusion
- `petalBreath = 1 + sin(floatT * 0.0133 + π * 0.7) * 0.022` — slow scale pulse desynced from body float

---

### 12. Particle Lifespan (Spawn → Float → Die)

**What it does**
Particles have a finite lifespan — they appear, orbit for a while, then gently fade out and are replaced. Calm and organic, like particles quietly cycling through.

**Behavior**
- Each particle lives long enough to complete several full orbits (~5–15 seconds) before fading
- New particles continuously spawn to replace dying ones while the button is active
- Feels like gentle turnover, not flickery disappearing

**Technical approach**
- Add `lifespan` (300–900 frames) and `age` counter to the `Particle` class
- When `age >= lifespan`, mark as dying to trigger existing fade-out
- Spawn loop replaces dying particles when button is active

---

### 13. Confetti Independent Movement (Galaxy Cluster)

**What it does**
The confetti cluster becomes its own little universe — the cluster as a whole orbits the flower, and inside it, individual pieces swirl around independently like planets in the abyss.

**Behavior**
- The cluster center orbits the flower as a whole (slow, drifting)
- Each confetti piece orbits/wanders within the cluster with its own angle, radius, and speed
- The overall system drifts around the character while individual pieces stay alive and in motion

**Technical approach**
- Two-level motion: cluster center position (relative to flower) + per-particle position (relative to cluster center)
- Each `crits` particle stores its own cluster-relative orbital params

---

### 14. Focus Mode (Clean View)

**What it does**
A toggle to hide or shrink the action bar, leaving just the character in a minimal, distraction-free view. Great for ADHD body doubling focus sessions.

**Behavior**
- Small toggle on the edge of the action bar collapses it (slides off screen or shrinks to a dot)
- Character still floats, reacts to mouse, and chat still works
- Clicking the dot/edge restores the action bar

---

### 15. Particle Parameter Controls (Sliders)

**What it does**
Expose particle behavior as user-controllable sliders — letting you dial in exactly how busy or calm the particle system feels.

**Controls**
- **Count:** How many particles of each type orbit
- **Speed:** How fast each particle moves
- **Entropy:** How chaotic/random the paths are (wobble amplitude + drift)

**Design**
- Collapsible panel or drawer, visible in edit mode
- Hidden in Focus Mode (Feature 14)

---

## Parking Lot — Future Ideas

Features that are desirable but TBD on timing or approach.

---

### Facial Expressions

**What it does**
Character has more than one emotion — happy, surprised, sleepy, etc. — so it's not the same face forever.

**Note:** Would require additional SVG face variants or a way to swap face slices. Current 3-slice face (forehead / eye-band / chin) is designed for blink only. Architecture TBD.

---

### AI Character Responses

**What it does**
Dynamic, personality-driven responses via Claude API instead of the current static response pools.

**Note:** Current pool has 7 responses per topic (focus, break, tired). Can expand manually in the meantime. Full AI integration is a separate architecture conversation — involves Claude API, a defined character voice/personality, and prompt design.

---

## Opportunity — Desktop Companion

The next major evolution: transform Cozy Calcifer from an animated browser character into a living desktop companion. Inspired by the Digimon concept of a personal digital friend who is always with you — not a productivity tool, but a *presence* that makes working feel less lonely.

This is a separate future initiative, distinct from the web animation work above.

---

### Vision

A floating, always-visible companion who lives in the corner of your Mac screen. Transparent window, never blocking your work. It feels alive because it already is — the animations, personality, and reactivity already exist. The goal is to make it *real* by getting it off the browser and onto your desktop.

---

### Core pillars

**1. Desktop presence**
- Electron app: transparent, frameless, always-on-top Mac window
- Lives in a corner of the screen, never in the way
- Accessible at a glance without switching apps or tabs

**2. Speech bubbles — on demand, not intrusive ✅ Done**

*What was built:* Tap/hover the character to reveal an action bar of prompt buttons. Pick one and the character responds in a styled speech bubble with a typewriter animation.

- Action bar appears on interaction with labeled prompt buttons; CSS handles scaling and responsive layout
- Character responds in a rounded speech bubble with a custom tail; tail tip anchored by `bottom` so it stays fixed as the bubble grows
- Response text animates in sentence-by-sentence with breathing pauses between sentences
- Bubble expands vertically to fit multi-line responses; width matches the action bar
- Mobile-responsive: action bar fits within 30px side padding; character shifts down to make room

*Still needed:* Real Claude API responses (currently placeholder text); defined character personality/voice

**3. Activity states with props**
- The character visually reflects what you're both doing together
- States: working (tiny laptop), reading (open book), writing (notebook + pencil), on a break (steamy mug)
- Props are custom SVGs in the same painterly style as the existing character art
- State change is triggered intentionally — you tell your companion what you're doing
- Animations subtly shift per state (more settled on break, more alert while working)

**4. Body doubling for ADHD**
- The core emotional value: not feeling alone while you work
- The companion is *with* you — present, engaged, in the same state as you
- Celebrates wins (bounce animations + confetti already exist for this)
- Gentle presence during hard stretches — no nagging, just company

---

### Open questions to resolve
- What is the character's name and defined personality/voice?
- What prompt categories/labels go on the action bar buttons?
- How are activity states triggered — manual selection, or automatic detection?
- Which Claude model and prompt system handles character responses?

---

## Out of Scope

- New character parts or SVG assets
- Sound / audio
- Native mobile app (mobile-responsive web view is now supported)
