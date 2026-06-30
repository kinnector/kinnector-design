# Kinnector UI Design System (`DESIGN-SYSTEM.md`)

> This document is the technical specification layer that bridges the narrative vision of `DESIGN-NOVEL.md` with the implementation reality of `FRONTEND.md`. It defines every design token, material specification, animation contract, layer architecture, and component rule that an engineer needs to faithfully build the Kinnector interface. It does not contain application logic or routing. It does not prescribe Svelte component structure (see `FRONTEND.md`). It prescribes *what things look, feel, and move like*, with enough precision that no design decision need be made at build time.

---

## 1. Foundational Principles

Three rules govern every decision in this system:

1. **The weather state machine is the single source of visual truth.** Every colour, blur value, animation speed, and material property is a downstream consequence of one of five weather states. Nothing is styled in isolation.
2. **Compositor-only animations.** Only `transform` and `opacity` may be animated directly. `filter`, `backdrop-filter`, `box-shadow`, and `background` transitions are achieved through opacity cross-fades of pre-rendered pseudo-elements, not direct property animation.
3. **Text is always sovereign (with quarantine exception).** No atmospheric effect — condensation, displacement, blur, lightning flash — may reduce text legibility. The typography layer is always topmost and always opaque. *Exception:* During the `frost` state, the text wrapper opacity of the quarantined card scales inversely with the frost growth (`opacity: calc(1 - var(--frost-progress, 0))`) to simulate freezing under the ice.

---

## 2. The Weather State Machine

The entire visual system is controlled by a single CSS custom property set on the `:root` element. The Svelte application reads the current threat level from the `kinnect-agent` API and updates these variables. All visual components cascade from them.

### 2.1 State Definitions

| ID | Name | Trigger Condition |
|---|---|---|
| `clear` | Clear / Sunny | System healthy. No active threats. Normal operation. |
| `overcast` | Overcast / Humid | Anomalous but unconfirmed activity. Unusual I/O or network behaviour. |
| `rain` | Light Rain | Minor confirmed threat signal. Unusual process activity flagged. |
| `storm` | Thunderstorm | Active EDR threat confirmed and under analysis. |
| `frost` | Frost / Quarantine | Threat successfully blocked and quarantined. Awaiting user acknowledgment. |

### 2.2 The CSS Variable API

These are the variables updated by the weather orchestrator. All component styles reference these variables exclusively — no component hardcodes a weather-dependent value.

```
--weather-state          string       'clear' | 'overcast' | 'rain' | 'storm' | 'frost'
--sky-top                color        Top of the background sky gradient
--sky-bottom             color        Bottom of the background sky gradient
--fog-opacity            number       0.0 – 1.0  (fog layer global alpha)
--fog-rise               number       0.0 – 1.0  (how far fog rises up the viewport)
--cloud-speed            duration     CSS duration string, e.g. '30s', '12s'
--cloud-opacity          number       0.0 – 1.0
--alert-light-color      color        The primary non-neutral light source during threat states
--alert-light-opacity    number       0.0 – 1.0
--light-source-x         percentage   Horizontal position of the primary light source
--light-source-y         percentage   Vertical position of the primary light source
--glass-bg               color        rgba() value for glass card background
--glass-blur             string       e.g. 'blur(20px) saturate(180%)'
--glass-border-alpha     number       Alpha of the 1px prism border at rest (0.0 – 1.0)
--glass-moisture         number       0.0 – 1.0  (drives condensation pseudo-element opacity)
--status-pulse-speed     duration     Breath cycle duration for the status dot
--status-pulse-color     color        Current status dot colour
--surface-sweep-trigger  integer      Increment this value to trigger a sweep (Svelte reactive)
--canvas-rain-active     0 | 1        Signal to the canvas layer to start/stop rain simulation
--canvas-frost-active    0 | 1        Signal to the canvas layer to start/stop frost drawing
--screen-shake-active    0 | 1        Triggers the viewport shake keyframe
```

### 2.3 State Value Table

#### State: `clear`

| Variable | Day Value | Night Value |
|---|---|---|
| `--sky-top` | `#e0f2fe` | `#090d16` |
| `--sky-bottom` | `#f0fdfa` | `#02040a` |
| `--fog-opacity` | `0.28` | `0.35` |
| `--fog-rise` | `0.20` | `0.25` |
| `--cloud-speed` | `30s` | — (stars; no clouds) |
| `--cloud-opacity` | `0.30` | `0` |
| `--alert-light-color` | — | — |
| `--alert-light-opacity` | `0` | `0` |
| `--light-source-x` | `18%` | `20%` |
| `--light-source-y` | `8%` | `10%` |
| `--glass-bg` | `rgba(255,255,255, 0.44)` | `rgba(12,18,30, 0.64)` |
| `--glass-blur` | `blur(20px) saturate(185%)` | `blur(24px) saturate(170%)` |
| `--glass-border-alpha` | `0.55` | `0.10` |
| `--glass-moisture` | `0` | `0` |
| `--status-pulse-speed` | `4s` | `4s` |
| `--status-pulse-color` | `#10b981` | `#10b981` |
| `--canvas-rain-active` | `0` | `0` |
| `--canvas-frost-active` | `0` | `0` |

#### State: `overcast`

| Variable | Day Value | Night Value |
|---|---|---|
| `--sky-top` | `#8896a8` | `#141920` |
| `--sky-bottom` | `#6b7a8d` | `#0d1016` |
| `--fog-opacity` | `0.55` | `0.60` |
| `--fog-rise` | `0.50` | `0.55` |
| `--cloud-speed` | `20s` | `24s` |
| `--cloud-opacity` | `0.60` | `0.55` |
| `--alert-light-opacity` | `0` | `0` |
| `--light-source-x` | `50%` | `50%` |
| `--light-source-y` | `30%` | `30%` |
| `--glass-bg` | `rgba(240,244,248, 0.50)` | `rgba(14,20,32, 0.70)` |
| `--glass-blur` | `blur(22px) saturate(160%)` | `blur(26px) saturate(150%)` |
| `--glass-border-alpha` | `0.35` | `0.08` |
| `--glass-moisture` | `0.30` | `0.35` |
| `--status-pulse-speed` | `4s` | `4s` |
| `--status-pulse-color` | `#10b981` | `#10b981` |
| `--canvas-rain-active` | `0` | `0` |
| `--canvas-frost-active` | `0` | `0` |

#### State: `rain`

| Variable | Day Value | Night Value |
|---|---|---|
| `--sky-top` | `#4a5568` | `#0c1018` |
| `--sky-bottom` | `#2d3748` | `#080b12` |
| `--fog-opacity` | `0.70` | `0.72` |
| `--fog-rise` | `0.70` | `0.75` |
| `--cloud-speed` | `14s` | `18s` |
| `--cloud-opacity` | `0.80` | `0.75` |
| `--alert-light-opacity` | `0` | `0` |
| `--light-source-x` | `50%` | `50%` |
| `--light-source-y` | `50%` | `50%` |
| `--glass-bg` | `rgba(220,228,236, 0.52)` | `rgba(14,20,32, 0.72)` |
| `--glass-blur` | `blur(22px) saturate(140%)` | `blur(26px) saturate(130%)` |
| `--glass-border-alpha` | `0.30` | `0.07` |
| `--glass-moisture` | `0.65` | `0.70` |
| `--status-pulse-speed` | `4s` | `4s` |
| `--status-pulse-color` | `#10b981` | `#10b981` |
| `--canvas-rain-active` | `1` | `1` |
| `--canvas-frost-active` | `0` | `0` |

#### State: `storm`

| Variable | Day Value | Night Value |
|---|---|---|
| `--sky-top` | `#1c1c2e` | `#050508` |
| `--sky-bottom` | `#0a0a14` | `#020204` |
| `--fog-opacity` | `0.85` | `0.88` |
| `--fog-rise` | `0.90` | `0.95` |
| `--cloud-speed` | `7s` | `9s` |
| `--cloud-opacity` | `0.95` | `0.90` |
| `--alert-light-color` | `#d97706` | `#b45309` |
| `--alert-light-opacity` | `0.70` | `0.80` |
| `--light-source-x` | `50%` | `50%` |
| `--light-source-y` | `85%` | `85%` |
| `--glass-bg` | `rgba(20,22,30, 0.68)` | `rgba(10,10,18, 0.78)` |
| `--glass-blur` | `blur(24px) saturate(130%)` | `blur(28px) saturate(120%)` |
| `--glass-border-alpha` | `0.15` | `0.12` |
| `--glass-moisture` | `0.90` | `0.95` |
| `--status-pulse-speed` | `0.8s` | `0.8s` |
| `--status-pulse-color` | `#10b981` | `#10b981` |
| `--canvas-rain-active` | `1` | `1` |
| `--canvas-frost-active` | `0` | `0` |

#### State: `frost`

Inherits the preceding storm state sky and fog values (the storm is receding), with the following applied to the quarantined card specifically. The global rain canvas continues at reduced intensity while the storm withdraws.

| Variable | Value |
|---|---|
| `--canvas-frost-active` | `1` (on the quarantined card's canvas) |
| `--canvas-rain-active` | `0.4` (reduced spawn rate, storm fading) |
| `--glass-moisture` | Transitions toward `0` as sky clears |
| `--frost-progress` | `0.0` to `1.0` (drives card crystal growth and text opacity decay) |
| `--status-pulse-speed` | `4s` (recovers to calm breath) |
| `--status-pulse-color` | `#10b981` (recovers to healthy green) |

---

## 3. Color Tokens

### 3.1 Fixed Palette (Weather-Independent)

| Token | Value | Usage |
|---|---|---|
| `--color-accent` | `#10b981` | Status dot, timeline nodes, active tab indicator |
| `--color-accent-dim` | `rgba(16,185,129, 0.25)` | Accent glow halos |
| `--color-alert-amber` | `#d97706` | Storm state primary light source (day) |
| `--color-alert-amber-night` | `#b45309` | Storm state primary light source (night) |
| `--color-danger-soft` | `rgba(239,68,68, 0.18)` | Danger button pulsing glow |
| `--color-frost-crystal` | `#dbeafe` | Frost crystal arm colour |
| `--color-frost-ice` | `rgba(200,225,255, 0.82)` | Quarantine card ice overlay |
| `--color-lightning-day` | `rgba(253,224,71, 0.18)` | Lightning flash background bloom (day) |
| `--color-lightning-night` | `rgba(200,220,255, 0.22)` | Lightning flash background bloom (night) |
| `--color-moonlight` | `#e8eef5` | Night mode primary light source colour |
| `--color-sunlight` | `#fefce8` | Day mode primary light source colour |

### 3.2 Typography Tokens

| Token | Day Value | Night Value |
|---|---|---|
| `--text-primary` | `#1e293b` | `#f1f5f9` |
| `--text-secondary` | `#475569` | `#cbd5e1` |
| `--text-muted` | `#64748b` | `#94a3b8` |
| `--text-data` | `#334155` | `#e2e8f0` |
| `--text-alert` | `#78350f` | `#fde68a` |
| `--text-shadow-backlit` | none | `0 0 12px rgba(255,255,255,0.08)` |

---

## 4. Typography

### 4.1 Font Families

All fonts must be bundled as WOFF2 assets. No external font requests are permitted (air-gapped deployment requirement).

| Role | Family | Fallback Stack |
|---|---|---|
| UI / Headings | `Inter` | `system-ui, -apple-system, sans-serif` |
| Data / Logs / Telemetry | `JetBrains Mono` | `'Cascadia Code', 'Fira Code', monospace` |

### 4.2 Type Scale

| Token | Size | Weight | Line Height | Letter Spacing | Usage |
|---|---|---|---|---|---|
| `--type-h1` | `28px` | `700` | `36px` | `-0.02em` | Page titles, window headers |
| `--type-h2` | `20px` | `600` | `28px` | `-0.01em` | Card section headers |
| `--type-h3` | `16px` | `600` | `24px` | `0` | Sub-headers, group labels |
| `--type-body` | `14px` | `400` | `20px` | `0` | Standard body copy |
| `--type-label` | `12px` | `500` | `16px` | `0.01em` | Metadata, timestamps, badges |
| `--type-mono` | `13px` | `400` | `20px` | `0` | All telemetry, paths, PIDs, hashes |
| `--type-mono-sm` | `11px` | `400` | `16px` | `0` | Inline code, compressed log rows |

### 4.3 Typography Rules

- Headings are always fully opaque. `opacity: 1` is enforced.
- In dark mode, headings carry `text-shadow: var(--text-shadow-backlit)`.
- Data grid text uses `--text-data` and `opacity: 0.88` to soften without losing contrast.
- Alert text during `storm` state carries `text-shadow: 0 0 8px var(--color-alert-amber)`.
- No weather effect, displacement filter, or blur layer may be placed above the typography layer.

---

## 5. The Glass Material

### 5.1 Base Glass Properties

Glass cards use three stacked mechanisms: the background fill, the `backdrop-filter`, and the micro-grain texture.

```
Background:       var(--glass-bg)
Backdrop filter:  var(--glass-blur)
Border:           1px prism gradient (see §5.2)
Border radius:    12px (cards); 16px (modals)
Shadow:           see §5.3
Noise texture:    SVG feTurbulence overlay (see §5.4)
Moisture overlay: ::before pseudo-element (see §5.5)
```

### 5.2 Prism Border

The 1px border on every glass card is a `linear-gradient` simulating a physical beveled glass edge catching the primary light source. The gradient angle is computed from `--light-source-x` and `--light-source-y`.

The gradient always has this structure: bright specular highlight at the edge nearest the light source → faint translucent middle → near-invisible far edge.

**Clear/Day (light source upper-left → angle ~315deg):**
```
linear-gradient(315deg,
  rgba(255,255,255, 0.05) 0%,
  rgba(255,255,255, 0.55) 40%,
  rgba(255,255,255, 0.55) 60%,
  rgba(255,255,255, 0.05) 100%
)
```

**Storm (light source lower-center, amber tint):**
```
linear-gradient(180deg,
  rgba(217,119,6, 0.05) 0%,
  rgba(255,255,255, 0.12) 50%,
  rgba(217,119,6, 0.35) 100%
)
```

**On lightning flash:** The prism border gradient angle and intensity updates instantly via a JS-driven CSS variable. It reverts after `200ms`.

**On hover:** The border alpha increases by `+0.25` at the edge nearest the cursor, tracked via `--hover-light-x` and `--hover-light-y` CSS variables.

### 5.3 Shadow Stack

```
Day / Clear:
  box-shadow:
    0 4px 6px rgba(0,0,0, 0.04),
    0 10px 32px rgba(0,0,0, 0.06),
    0 1px 0 rgba(255,255,255, 0.60) inset

Night / Clear:
  box-shadow:
    0 4px 8px rgba(0,0,0, 0.35),
    0 12px 40px rgba(0,0,0, 0.45),
    0 1px 0 rgba(255,255,255, 0.08) inset

Storm (any):
  box-shadow:
    0 4px 8px rgba(0,0,0, 0.60),
    0 16px 48px rgba(0,0,0, 0.70),
    0 1px 0 rgba(217,119,6, 0.20) inset
```

### 5.4 Micro-Grain Noise Texture

An SVG `feTurbulence` filter applied as a zero-opacity background overlay on every glass card simulates the microscopic rough surface of real sandblasted glass. This texture does not animate.

```
type:           fractalNoise
baseFrequency:  0.68
numOctaves:     4
stitchTiles:    stitch
Overlay opacity: 0.032 (day) / 0.048 (night)
Blend mode:     overlay
```

The SVG filter definition (`id="glass-grain"`) is injected once in the application root. Individual cards reference it via `filter: url(#glass-grain)` on a `::after` pseudo-element with `pointer-events: none`.

### 5.5 Moisture / Condensation Overlay

A `::before` pseudo-element on each glass card has its opacity bound directly to `--glass-moisture`.

```
Position:    absolute, inset 0
Background:  radial-gradient(ellipse at 50% 0%,
               rgba(255,255,255, 0.18) 0%,
               rgba(255,255,255, 0.04) 60%,
               transparent 100%)
Opacity:     calc(var(--glass-moisture) * 0.9)
Filter:      blur(4px)
Pointer-events: none
```

---

## 6. Layer Architecture (Z-Index Stack)

| Layer | Z-Index | Contents |
|---|---|---|
| Background / Sky | `0` | Sky gradient fill |
| Cloud blobs | `1–5` | Animated radial gradient blobs |
| Star field | `3` | Dark-mode only; radial-gradient texture |
| Fog layer | `8–12` | Volumetric fog element |
| Rain canvas | `15` | 2D canvas — droplet simulation |
| Glass cards | `20–30` | `.glass-card` components |
| Navigation chrome | `35` | Sidebar, top bar |
| Alert spotlight | `40` | Storm amber radial gradient, cursor-tracked |
| Quarantine frost canvas | `45` | Per-card canvas overlay — frost crystal growth |
| Modal backdrop | `50` | Blur/freeze overlay behind modal |
| Modal / Threat Viewer | `55` | `.glass-modal` component |
| Surface sweep | `60` | Diagonal wiper beam pseudo-element |
| Toast notifications | `70` | Short-lived status messages |
| Screen glare | `80` | Topmost ambient reflection gradient |

No component may exceed z-index `79`. Screen glare is always `80`.

---

## 7. The Canvas Layer Specification

### 7.1 Rain Canvas (`#rain-canvas`)

**Position:** Fixed, full-viewport, z-index `15`. `pointer-events: none`.
**Active when:** `--canvas-rain-active` is `1` (or fractional for fading).
**Frame rate:** 60 fps during active weather; paused when `--canvas-rain-active` is `0`.

**Droplet system parameters:**

| Parameter | `rain` State | `storm` State |
|---|---|---|
| Max simultaneous droplets | `80` | `200` |
| Spawn rate | `0.5` / frame | `3.0` / frame |
| Min droplet radius | `2px` | `2px` |
| Max static droplet radius | `8px` | `12px` |
| Growth rate (before streak) | `+0.08px/frame` | `+0.15px/frame` |
| Streak threshold radius | `7px` | `10px` |
| Gravity (streak acceleration) | `0.12 px/frame²` | `0.18 px/frame²` |
| Max streak velocity | `4 px/frame` | `7 px/frame` |
| Horizontal drift | `±0.3 px/frame` | `±0.5 px/frame` |
| Merge distance threshold | `10px` center-to-center | `10px` |
| Track evaporation time | `4000ms` | `2500ms` |
| Track opacity at birth | `0.80` | `0.85` |

**Droplet rendering:**

- **Static bead:** Rendered using canvas `globalCompositeOperation = 'exclusion'` or `difference` to invert the background colors, then overlaying a filled circle of `rgba(200,220,240, 0.25)` day / `rgba(160,190,220, 0.20)` night.
- **Specular highlight:** Small filled ellipse offset `(r*0.3, r*-0.3)` from center, radius `r*0.35`, colour `rgba(255,255,255, 0.70)`.
- **Amber raindrop highlight (storm night only):** Additional specular at `(r*-0.2, r*0.4)`, colour `rgba(217,119,6, 0.45)`, radius `r*0.25` — tungsten alert source reflected.
- **Streak tracks:** A `clearRect` path + thin `rgba(255,255,255, 0.55)` stroke on the frosted glass. Track opacity decays via a separate timer.

**SVG Displacement Map — Background Distortion:**

The rain canvas distorts the background layers (sky, fog, cloud blobs) only. It does not distort DOM text.

```
Filter:           feDisplacementMap
Source:           background layer composite (z-index 0–12)
Map source:       A secondary low-resolution canvas (1/4 viewport resolution)
                  encoding droplet positions as red-channel values
xChannelSelector: R
yChannelSelector: G
Scale:            proportional to droplet radius, max 18px
                  formula: scale = droplet.radius * 1.5
Update rate:      Every 4 frames (15 fps is sufficient for displacement)
Activation:       Applied only when --canvas-rain-active is 1
                  Scale transitions to 0 over 800ms on deactivation
```

### 7.2 Frost Crystal Canvas (`#frost-canvas-[card-id]`)

One canvas is created per quarantined card, absolutely positioned over the card, matching its dimensions, at z-index `45`.

**Crystal growth algorithm (recursive fractal tree):**

```
Seed points:   Distributed along card edges at irregular intervals
               (~1 seed per 40px of edge length, randomised ±20px)
Primary arms:  Grow perpendicular to the edge inward
               Length: 25–60% of the card's narrower dimension
Branch angle:  ±18° – ±42° from parent arm (randomised per branch)
Branch length: 60–75% of parent arm
Max recursion: 4 levels deep
Growth speed:  2px advance per frame per arm tip
Colour:        rgba(219,234,254, 0.72) at base → rgba(255,255,255, 0.90) at tips
Line width:    1.5px (base) → 0.5px (4th-level branches)
```

**Ice overlay (drawn simultaneously with crystal arms):**
```
Fill:           rgba(200,225,255, 0.55)
Filter:         blur(2px)
Applied over:   Full card bounds
Opacity target: 0.78 at full frost
Duration:       2000ms from trigger to full frost
```

---

## 8. Animation Contracts

### 8.1 Duration Tokens

| Token | Value | Usage |
|---|---|---|
| `--dur-instant` | `100ms` | Button press, bevel flash on lightning |
| `--dur-fast` | `200ms` | Button hover, bevel intensity |
| `--dur-standard` | `350ms` | Most micro-interactions |
| `--dur-enter` | `400ms` | Elements entering the screen |
| `--dur-exit` | `250ms` | Elements leaving the screen |
| `--dur-modal` | `500ms` | Modal rise / lower |
| `--dur-sweep` | `2200ms` | Weather fog-front transition |
| `--dur-frost` | `2000ms` | Frost crystal full growth |
| `--dur-dissolve-out` | `300ms` | Tab pane dissolve to steam |
| `--dur-dissolve-in` | `400ms` | Tab pane materialise from depth |
| `--dur-breath` | `var(--status-pulse-speed)` | Status dot pulse cycle |
| `--dur-surface-sweep` | `900ms` | Wiper beam diagonal pass |

### 8.2 Easing Tokens

| Token | Value | Usage |
|---|---|---|
| `--ease-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | General UI transitions |
| `--ease-enter` | `cubic-bezier(0.16, 1, 0.30, 1)` | Elements entering (spring-out) |
| `--ease-exit` | `cubic-bezier(0.4, 0, 1, 1)` | Elements leaving (ease-in) |
| `--ease-physical` | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` | Modal, fog front — things with weight |

### 8.3 Keyframe Specifications

**Status Dot — Calm Breath** (clear / overcast / rain states)
```
0%, 100%:  scale(1.00), opacity 0.80, box-shadow 0 0 4px  --color-accent
50%:       scale(1.12), opacity 1.00, box-shadow 0 0 14px --color-accent
Duration:  --status-pulse-speed (4s), ease-in-out, infinite
```

**Status Dot — Urgent Double-Pulse** (storm state)
```
0%:        scale(1.00), opacity 1.0, box-shadow 0 0 16px --color-accent
15%:       scale(1.18), opacity 1.0, box-shadow 0 0 28px --color-accent
30%:       scale(1.00), opacity 0.6, box-shadow 0 0 6px  --color-accent
45%:       scale(1.14), opacity 1.0, box-shadow 0 0 22px --color-accent
60%,100%:  scale(1.00), opacity 0.6, box-shadow 0 0 4px  --color-accent
Duration:  0.8s, ease-in-out, infinite
```
The double-beat (two swells per cycle) simulates an elevated heartbeat without a strobe effect.

**Tab Pane — Dissolve Out**
```
0%:    opacity 1, translateY(0),    blur(0)
100%:  opacity 0, translateY(-3px), blur(6px)
Duration: --dur-dissolve-out, --ease-exit, fill-mode: forwards
```

**Tab Pane — Rise In**
```
0%:    opacity 0, translateY(4px), blur(4px)
100%:  opacity 1, translateY(0),   blur(0)
Duration: --dur-dissolve-in, --ease-enter, fill-mode: forwards
```

**Modal — Rise**
```
0%:    translateY(100%), scale(0.96), opacity 0
100%:  translateY(0),    scale(1.00), opacity 1
Duration: --dur-modal, --ease-physical, fill-mode: forwards
```

**Modal — Lower**
```
0%:    translateY(0),    scale(1.00), opacity 1
100%:  translateY(80px), scale(0.98), opacity 0
Duration: 350ms, --ease-exit, fill-mode: forwards
```

**Window — Open**
```
0%:    opacity 0, scale(0.97), blur(6px)
100%:  opacity 1, scale(1.00), blur(0)
Duration: --dur-enter, --ease-enter
```

**Window — Close**
```
0%:    opacity 1, scale(1.00)
100%:  opacity 0, scale(0.98)
Duration: --dur-exit, --ease-exit
```

**Surface Sweep (Wiper)**
```
0%:    translate(-110%, -110%)
100%:  translate(110%, 110%)
Duration: --dur-surface-sweep, linear
```
The sweep element is a diagonal pseudo-element spanning 120% of the card width at a 30° angle, filled with
`linear-gradient(120deg, transparent → rgba(255,255,255,0.55) → transparent)`. The movement scales across both X and Y coordinates to achieve a true upper-left to lower-right sweep.

**Lightning Flash — Background**

Lightning is not a CSS `animation` on a fixed schedule. It is triggered by a JS function that applies a short-lived CSS class to the sky container and removes it after the animation completes. Interval between flashes is randomised: storm state = 3–7 seconds between triggers.

```
0%:      brightness(1.00)
8%:      brightness(1.35), inject --color-lightning-[day/night] bloom
12%:     brightness(1.00)
16%:     brightness(1.28)
20%,100%: brightness(1.00)
Duration: 400ms, triggered by JS class injection
```

On each lightning trigger: the prism border gleam direction snaps to the flash origin and reverts after `200ms`. One screen-shake keyframe is also triggered on the root layout wrapper.

**Screen Shake**
```
0%,100%: translateX(0)
20%:     translateX(-2px)
40%:     translateX(2px)
60%:     translateX(-1.5px)
80%:     translateX(1px)
Duration: 280ms, ease-in-out
Applied to: root layout wrapper, once per lightning flash
```

**Fog Front — Weather State Transition**
```
0%:    translateX(-140vw)
100%:  translateX(140vw)
Duration: --dur-sweep, --ease-physical
```
The fog front is a fixed-position element, 140vw wide, z-index `9999`. CSS class swap at the 50% midpoint via JS `setTimeout` at `duration / 2`.

**New Event Row — Vapor Materialise**
```
0%:    opacity 0, translateY(4px)
100%:  opacity 1, translateY(0)
Duration: 350ms, --ease-enter, fill-mode: forwards
```
Applied to each new row entering the telemetry log stream.

**Star Twinkle (Night Clear State)**
```
0%, 100%: opacity 0.30
50%:      opacity 1.00
Duration: 3s – 6s (randomized per star via `animation-delay` and `animation-duration` inline styles)
Easing:   ease-in-out, infinite
```
Twinkling applies to the star elements on the star field layer to create a natural, organic shimmering effect.

---

## 9. Component Specifications

### 9.1 Glass Card

```
Background:       var(--glass-bg)
Border:           1px solid — prism gradient (see §5.2)
Border-radius:    12px
Backdrop-filter:  var(--glass-blur)
Box-shadow:       per-state shadow stack (§5.3)
Padding:          16px standard; 24px for featured/large cards
Noise overlay:    ::after pseudo-element, SVG grain filter (§5.4)
Moisture overlay: ::before pseudo-element (§5.5)
```

On hover:
```
Prism border alpha: +0.25 on nearest edge
Transition:         var(--dur-fast) var(--ease-standard)
```

### 9.2 Primary Action Button

```
Background:    var(--color-accent)
Color:         white
Border:        none
Border-radius: 8px
Padding:       8px 16px
Font:          var(--type-body), weight 500
Box-shadow:    0 0 0 0 rgba(16,185,129, 0)  (at rest)
```

Hover:
```
box-shadow: 0 0 16px 2px rgba(16,185,129, 0.35)
Transition: box-shadow var(--dur-fast) var(--ease-standard)
```

Active / pressed:
```
transform:  translateY(1px)
box-shadow: 0 0 6px 0 rgba(16,185,129, 0.20) inset
Transition: transform var(--dur-instant), box-shadow var(--dur-instant)
```

### 9.3 Secondary Glass Button

```
Background:    rgba(255,255,255, 0.06)
Border:        1px solid rgba(255,255,255, calc(var(--glass-border-alpha) * 0.6))
Border-radius: 8px
Padding:       8px 16px
Color:         var(--text-secondary)
```

Hover:
```
Background: rgba(255,255,255, 0.18)
Color:      var(--text-primary)
```

Active / pressed:
```
A localized ripple from the pointer position:
  ::after pseudo, scale(0)→scale(2.5), opacity(0.4)→opacity(0), 400ms
```

### 9.4 Danger / Destructive Button

```
Background:  rgba(234,88,12, 0.12)
Border:      1px solid rgba(234,88,12, 0.35)
Color:       #fdba74
Border-radius: 8px
Padding:     8px 16px
Animation:   Slow amber-red box-shadow pulse, 2.4s ease-in-out infinite
             0%/100% → box-shadow: 0 0 6px 0 rgba(234,88,12, 0.10)
             50%     → box-shadow: 0 0 18px 4px rgba(234,88,12, 0.28)
```

Hover:
```
Background:  rgba(239,68,68, 0.22)
Parent card: --glass-bg alpha temporarily reduced by 0.10
```

### 9.5 Ghost Text Button

```
Background:  transparent
Border:      none
Color:       var(--text-secondary)
Padding:     4px 2px
```

Hover — underline draw:
```
::after:      absolute, bottom -1px, height 1px, background var(--color-accent)
transform:    scaleX(0) → scaleX(1)
transform-origin: center
Transition:   transform var(--dur-standard) var(--ease-enter)
```

### 9.6 Status Indicator Dot

```
Width/Height: 8px
Border-radius: 50%
Background:   var(--status-pulse-color)
Animation:    clear/overcast/rain → status-breath keyframe
              storm               → status-alert-pulse keyframe

Ripple halo: ::after pseudo, border-radius 50%,
             border 1.5px solid var(--status-pulse-color), opacity 0
             scale(1)→scale(2.8), opacity(0.5)→opacity(0)
             2s ease-out infinite, delayed 400ms from main pulse
```

### 9.7 Scrollbar

```
Width:       6px (vertical and horizontal)
Track:       transparent
Thumb rest:  opacity 0, transition 200ms
Thumb scroll: rgba(255,255,255, 0.22) day / rgba(255,255,255, 0.18) night
              border-radius: 99px
Thumb hover: rgba(255,255,255, 0.38)
```

### 9.8 Modal (Threat Alert Viewer)

```
Width:         min(720px, 92vw)
Max-height:    80vh
Border-radius: 16px
Background:    var(--glass-bg) — deepest glass treatment (storm state values)
Backdrop-filter: blur(32px) saturate(200%)
Box-shadow:    storm shadow stack (§5.3)
Entry:         modal-rise keyframe
Exit:          modal-lower keyframe

Backdrop overlay (over dashboard, behind modal):
  position:          fixed, inset 0
  background:        rgba(0,0,0, 0.35)
  backdrop-filter:   blur(24px)
  z-index:           50
  Entry:             opacity 0→1, 350ms ease-out
  Exit:              opacity 1→0, 50ms ease-in (to simulate instant thaw upon resolution)
```

### 9.9 Telemetry Event Timeline

```
Timeline rail:  1.5px wide, positioned 6px from left
Background:     linear-gradient(to bottom,
                  var(--color-accent) 0%,
                  rgba(16,185,129, 0.25) 35%,
                  rgba(16,185,129, 0.05) 70%,
                  transparent 100%)

Timeline node:  7px circle, background var(--color-accent)
                box-shadow: 0 0 8px rgba(16,185,129, 0.50)
                Hover: scale(1.4), 200ms ease

New entry:      event-vapor-in keyframe
```

---

## 10. Alert Spotlight (Storm State Only)

A full-viewport fixed element tracks the cursor during `storm` state, emitting tungsten amber light.

```
Position:   fixed, inset 0, pointer-events none, z-index 40
Background: radial-gradient(circle 380px at var(--cursor-x) var(--cursor-y),
              rgba(var(--color-alert-amber-rgb), var(--alert-light-opacity)),
              transparent 100%)
Blend mode: overlay
Opacity:    0 in non-storm states; transitions to 1 over 600ms on storm entry
```

Cursor position written to `--cursor-x` / `--cursor-y` only while `--weather-state` is `storm`. JS handler added on storm entry, removed on exit.

---

## 11. Screen Glare Layer

A single fixed element at z-index `80`, always present:

```
Position:    fixed, inset 0, pointer-events none
Background:  linear-gradient(135deg,
               rgba(255,255,255,0) 0%,
               rgba(255,255,255,0.022) 35%,
               rgba(255,255,255,0.038) 50%,
               rgba(255,255,255,0.018) 65%,
               rgba(255,255,255,0) 100%)
Animation:   Very slow gradient position drift, 45s ease-in-out infinite
             background-position oscillates between 0%/0% and 100%/100%
```

Opacity values are near-imperceptible by design. The effect is felt, not seen.

---

## 12. Spacing and Layout

### 12.1 Base Grid

All spacing uses an **8px base grid**. Only these values are permitted for margin, padding, and gap:

| Token | Value |
|---|---|
| `--space-1` | `4px` |
| `--space-2` | `8px` |
| `--space-3` | `12px` |
| `--space-4` | `16px` |
| `--space-5` | `24px` |
| `--space-6` | `32px` |
| `--space-7` | `48px` |
| `--space-8` | `64px` |

### 12.2 Border Radius Tokens

| Token | Value | Usage |
|---|---|---|
| `--radius-sm` | `6px` | Badges, tags, small controls |
| `--radius-md` | `8px` | Standard buttons, form inputs |
| `--radius-lg` | `12px` | Glass cards |
| `--radius-xl` | `16px` | Modals |
| `--radius-full` | `9999px` | Status dots, pill badges, scrollbar thumbs |

---

## 13. Accessibility and Safe Mode

### 13.1 Reduced Motion

When `prefers-reduced-motion: reduce` is detected:
- All CSS `animation` properties are set to `none`.
- The rain canvas is not activated regardless of `--canvas-rain-active`.
- The frost canvas is not activated regardless of `--canvas-frost-active`.
- Weather state changes apply CSS variable updates immediately (no fog-front sweep).
- Static glassmorphism is preserved. All text and data remains fully legible.

### 13.2 Safe Mode

Activated by `data-safe-mode="true"` on the `<html>` element. Set by the Svelte app when `kinnect-agent` reports a GPU-constrained environment.

- `backdrop-filter` removed. Glass cards use a flat opaque fill.
- Noise texture filter not applied.
- Rain and frost canvas layers not activated.
- Screen glare layer removed.
- Surface sweep animation still runs (CSS only; no GPU cost).
- Weather state machine continues operating. Color and typography tokens respond normally.

### 13.3 Colour Contrast Targets

| Text Token | Background | WCAG Target |
|---|---|---|
| `--text-primary` | `--glass-bg` (day) | ≥ 7:1 |
| `--text-primary` | `--glass-bg` (night) | ≥ 8:1 |
| `--text-muted` | `--glass-bg` | ≥ 4:1 |
| `--text-alert` | storm `--glass-bg` | ≥ 5:1 |

---

## 14. Performance Budget

| Resource | Limit |
|---|---|
| CSS compositor-only animations | No limit |
| JS main thread per frame | ≤ 2ms |
| Rain canvas frame budget | ≤ 3ms at 200 droplets, 60fps |
| Simultaneous `backdrop-filter` elements | ≤ 8 |
| Active SVG filter nodes | ≤ 4 (grain + fog displacement + 2 spare) |
| Frost canvas instances | ≤ 3 simultaneous; oldest replaced above 3 |
| GPU memory (canvas textures) | ≤ 64 MB |

**Throttle triggers** — when `kinnect-agent` reports CPU load ≥ 80%:
1. Rain canvas drops from 60fps to 15fps.
2. New droplet spawning suspends; existing droplets finish their trajectory.
3. Animated cloud blobs switch to `animation-play-state: paused`.
4. Status dot and timeline animations remain active (compositor-thread; zero CPU cost).

---

*For the narrative sensory description of the design, see `DESIGN-NOVEL.md`.*
*For Svelte component structure, file organisation, air-gapped bundling, and the agent API communication layer, see `FRONTEND.md`.*
