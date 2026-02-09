# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-file web app (`index.html`) — a concentric two-wheel spinner for choosing restaurants. No build step, no dependencies, no framework. ~1200 lines of HTML+CSS+JS.

Open `index.html` directly in any bwrowser (works from `file://`).

## Architecture

Everything lives in `index.html` in this order:

1. **HTML comment block** — detailed architecture notes, tuning history, edge cases
2. **`<style>`** — dark theme via CSS custom properties, screen routing (`.screen.active`), wheel/config/overlay layout
3. **Store** — localStorage CRUD with in-memory cache (`_data`). Key: `wheel-of-randomness`. Schema: `{ version, genres: [{ id, name, enabled, restaurants: [{ id, name, enabled }] }] }`
4. **Audio Engine** — Web Audio API. Tick (1200Hz square, 30ms) on slice crossings. Celebration (C5→E5→G5→C6 sine sequence) on result.
5. **Wheel Renderer** — SVG annular sectors in a 600×600 viewBox centered at origin. Two `<g>` elements (`#outer-wheel`, `#inner-wheel`) independently rotatable. Golden angle color distribution. Text has black stroke outline for readability. Inner pointer shown in two-ring mode.
6. **Layout Animation** — `animateLayout()` interpolates ring radii via rAF to transition between single-ring and two-ring modes.
7. **Spin Mechanics** — rAF loop with parametric easing `1-(1-t^a)^b`. Winner pre-computed; animation is visual only. Landing position biased toward edges 60% of the time for drama.
8. **State Machine** — `Machine` object with states: `IDLE → OUTER_SPINNING → OUTER_LANDED → INNER_SPINNING → RESULT` and `CONFIG`. All clicks gated on state.
9. **Config Screen** — Event delegation on `#genre-list` via `data-action` attributes. Accordion expand/collapse. Inline rename via `<input>` change events.
10. **Init** — `Store.load()`, `renderOuterWheel()`, `updateWheelVisibility()`

### Key constraints

- **SVG coordinate system**: 0° = 12 o'clock (top), angles increase clockwise. `polarToCart` offsets by -90°.
- **Single-item ring**: Must use two-semicircle full annulus path with `fill-rule: evenodd` (SVG can't draw a full circle as one arc).
- **`transform-origin` on SVG `<g>`**: Must be `0 0` absolute, not `50% 50%`.
- **Easing tuning**: `a` in 1.2–1.5, `b` in 3.0–4.5. Values outside these ranges were tested and rejected (see HTML comment block for history). Do not add velocity floors or piecewise easing — both were tried and produce visible artifacts.
- **Layout sizing**: `min(98vw, 98vh)` with no max cap — designed for 4K TV at distance.
