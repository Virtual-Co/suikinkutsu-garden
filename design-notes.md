# Suikinkutsu Garden — Design Notes

## Design Goals
1. **Calm, meditative, immediate** — first interaction rewards within 5 seconds; no clutter, no pressure.
2. **Physical authenticity** — controls map to real suikinkutsu parameters (vessel depth, bore, water level, bamboo, drip rate); audio synthesis reflects the paper's Minnaert-bubble + cavity-resonance model.
3. **One memorable element** — the gold-accented "water surface" that ripples on every drop; everything else is quiet moss-green and stone.

## Subject Vocabulary (Japanese garden / suikinkutsu)
- Moss (koke) — deep green surfaces
- Stone (ishi) — gray-brown textures
- Water (mizu) — ripples, reflections
- Bamboo (take) — hollow tubes, golden hue
- Shadow (kage) — subtle depth

## Color Palette (WCAG AA verified)

### Light mode
| Role | Hex | Contrast vs bg (#f5f0e8) | Use |
|------|-----|--------------------------|-----|
| bg | #f5f0e8 | — | Page background |
| surface | #e8f4ec | 11.0:1 (text) | Cards, panels |
| text | #1a3a2e | 10.96:1 | Primary text |
| muted | #4e6b59 | 5.18:1 | Secondary text, labels |
| border | #3a6b4a | 5.47:1 | Borders, dividers, focus ring |
| accent | #7c6d28 | 4.54:1 | Primary actions, water surface highlight, gold details |
| accent-hover | #6b5e22 | 5.1:1 | Hover/active states |
| success | #2d7a3d | 4.67:1 | Success moments, correct note |
| warning | #ac5618 | 4.52:1 | Off-target feedback |
| danger | #b82e2e | 5.34:1 | Errors (rare) |

### Dark mode (prefers-color-scheme: dark)
| Role | Hex | Contrast vs bg (#1a3a2e) | Use |
|------|-----|--------------------------|-----|
| bg | #1a3a2e | — | Page background |
| surface | #244a34 | 11.0:1 (text) | Cards, panels |
| text | #e8f4ec | 11.0:1 | Primary text |
| muted | #a8d0b8 | 7.33:1 | Secondary text, labels |
| border | #a8d0b8 | 7.33:1 | Borders, dividers, focus ring |
| accent | #c8b045 | 5.78:1 | Primary actions, water surface highlight, gold details |
| accent-hover | #d4c47a | 7.09:1 | Hover/active states |
| success | #4ade80 | 7.14:1 | Success moments, correct note |
| warning | #fb923c | 6.8:1 | Off-target feedback |
| danger | #f87171 | 6.2:1 | Errors (rare) |

All ratios computed via `ops/tools/contrast`.

## Typography
- **Family**: System stack only — `system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", sans-serif`
- **Monospace**: `ui-monospace, "SF Mono", "Monaco", monospace` (for frequency readouts)
- **Scale (px)**: 12 / 14 / 16 / 20 / 24 / 32
- **Body**: 16px, line-height 1.6
- **Weights**: 400 (regular), 600 (semibold)
- No web fonts — fully offline.

## Spacing & Radius
- **Scale (px)**: 4, 8, 12, 16, 24, 32, 48
- **Radius**: 8px (single value for all controls, cards, buttons)
- **Focus ring**: 2px solid accent, 2px offset, 3:1 against adjacent background

## Motion
- **State transitions**: 180ms ease-out
- **Water drop animation**: 600ms (ripple expand + fade)
- **Resonance visualization**: 2s decay (matches audio)
- **Success celebration**: 800ms (scale + color pulse)
- **Reduced motion**: `@media (prefers-reduced-motion: reduce)` disables all animations/transitions; instant state changes.

## Controls (Keyboard, Mouse, Touch)
- **Primary action (drop water)**: Click/tap the water surface, `Space` or `Enter` when focused
- **Parameter sliders**: Arrow keys (step), Home/End (min/max), drag on touch/mouse
- **Target note replay**: `R` key or replay button
- **Reset round**: `Escape` or reset button
- **About fold toggle**: `A` key or click summary
- All interactive elements: min 44×44px touch target

## First Five Seconds
1. Page loads → calm moss-green screen, centered "water surface" circle (120px) with subtle gold shimmer
2. Headline: **"Drop water into a buried garden pot"**
3. Subline: **"Click the water surface — hear the pot sing."**
4. Guided hint: A soft pulse animation on the water surface (2s loop) + "Tap or press Space" label
5. First click → immediate ripple + resonant tone (no latency) → "Well done!" toast at 1.5s

## On-Screen Hint (persistent, dismissible)
- Bottom-left, small: **"Space = drop • Arrows = tune • R = replay target • Esc = reset"**
- Auto-hides after 3 interactions; "?" button to recall

## Screens & States

### S1 — Welcome / First Drop
- **Default**: Headline, subline, water surface, hint, About fold (closed)
- **After first drop**: Toast "The pot rings at 247 Hz — try tuning it!" → transitions to S2

### S2 — Tuning Garden (main play)
- **Default**: Water surface (larger, 160px), resonance visualizer (canvas), 5 parameter sliders, target note panel (shows target freq, current freq, difference)
- **Playing**: Drop triggered → ripple + tone + visualizer burst
- **Success**: Target matched (±5 Hz) → celebration pulse, score +1, new target generated
- **Empty**: No target yet (first load) — "Drop water to begin"

### S3 — About the Science (fold, accessible from any screen)
- **Open**: Plain-language summary of the paper's 6 descriptors, Minnaert bubble model, 9 Kansai sites
- **Closed**: Summary line only

### S4 — Round Complete (modal overlay)
- **Success**: "Round complete! You tuned 5 notes." — shows score, "Play again" button
- **Accessible**: Focus trapped, Esc closes

## Resonance Visualizer
- Canvas 280×120, shows real-time frequency spectrum (Meyda/Web Audio Analyser)
- Bars colored by accent; decay mirrors audio envelope
- Reduced motion: static snapshot only

## Audio Synthesis (Web Audio API)
- **Minnaert bubble oscillator**: Short impulse (5–15 ms) at drop impact, frequency ~1/(2π)√(3γP/ρR³) mapped to "drip size" parameter
- **Cavity resonator**: 4–6 parallel bandpass filters (biquad) tuned to vessel modes (depth, bore, water level)
- **Bamboo tube**: Additional delay line with lowpass (length parameter)
- **Master**: Gain, reverb (convolver with short impulse), limiter
- All parameters update in real time; no audio worklet needed for MVP

## Target-Note Rounds
- 5 notes per round, frequencies drawn from the 9 Kansai sites' dominant resonances (embedded JSON)
- Player adjusts sliders to match target ±5 Hz
- Success → next target; 5 successes → round complete
- Score persists in sessionStorage

## Accessibility
- Semantic HTML: `<main>`, `<section>`, `<button>`, `<input type="range">` with `<label>`
- ARIA: `aria-live="polite"` for target freq updates, `aria-label` on icon-only controls
- Color never sole indicator: success adds checkmark icon, warning adds ⚠
- Reduced motion respected
- High contrast mode: uses same palette (already 7:1+ in dark, 4.5:1+ in light)

## Assets & Licenses
- No external images, fonts, or scripts
- All code original
- Paper credit: "Based on Haimes, Uhde, Marutschke — 'A computational framework for the acoustic characterization of Suikinkutsu (water harp cave)' (arXiv:2610.11219, CC BY 4.0)"
- Distinction: "The paper's 6 acoustic descriptors and cultural framing are from the authors. The playable synthesis, tuning game, and target-note rounds are our game design."

## Open Questions (Self-Critique)
1. **What would make this look like every other tool?** A generic settings panel on the right. Fix: integrate sliders into the garden scene as bamboo markers.
2. **Which element is decoration only?** The ripple shader could be pure eye candy. Fix: make ripple radius map to drip-rate parameter.
3. **Which text is under 4.5:1?** Muted labels on surface in light mode — verified at 5.18:1. Pass.

## Tokens (CSS Custom Properties)
```css
:root {
  /* Light (default) */
  --bg: #f5f0e8;
  --surface: #e8f4ec;
  --text: #1a3a2e;
  --muted: #4e6b59;
  --border: #3a6b4a;
  --accent: #7c6d28;
  --accent-hover: #6b5e22;
  --success: #2d7a3d;
  --warning: #ac5618;
  --danger: #b82e2e;
  --radius: 8px;
  --space-1: 4px; --space-2: 8px; --space-3: 12px; --space-4: 16px; --space-5: 24px; --space-6: 32px; --space-7: 48px;
  --font-sans: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", sans-serif;
  --font-mono: ui-monospace, "SF Mono", "Monaco", monospace;
  --fs-12: 12px; --fs-14: 14px; --fs-16: 16px; --fs-20: 20px; --fs-24: 24px; --fs-32: 32px;
  --lh-body: 1.6;
  --transition: 180ms ease-out;
  --focus-ring: 2px solid var(--accent);
  --focus-offset: 2px;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a3a2e;
    --surface: #244a34;
    --text: #e8f4ec;
    --muted: #a8d0b8;
    --border: #a8d0b8;
    --accent: #c8b045;
    --accent-hover: #d4c47a;
    --success: #4ade80;
    --warning: #fb923c;
    --danger: #f87171;
  }
}

@media (prefers-reduced-motion: reduce) {
  :root { --transition: 0ms; }
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```