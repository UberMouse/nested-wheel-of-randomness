# Nested Wheel of Randomness

A concentric two-wheel spinner for choosing restaurants. The outer ring picks a food genre, then the inner ring auto-spins to pick a restaurant within that genre.

Replaces the two-step process of spinning [wheelofnames.com](https://wheelofnames.com) twice (once for genre, once for restaurant) with a single gameshow-style experience.

## Usage

Open `index.html` in Chrome or Firefox. Works from `file://` or any static host.

1. Click **Configure** to add genres and restaurants
2. Click any slice on the wheel to spin
3. The genre wheel spins and lands on a genre
4. The wheel animates — the genre ring shrinks inward, revealing a restaurant ring in the center with its own pointer
5. The restaurant ring auto-spins and lands on a restaurant
6. Result overlay shows the pick with options to spin again or try a different restaurant
7. "Spin Again" collapses back to the single-ring genre wheel before re-spinning

## Display

Designed for a 4K TV viewed at a distance. The wheel fills `min(98vw, 98vh)` with no maximum cap. The header overlays the top-left corner so it doesn't steal vertical space from the wheel.

## Data

All configuration is stored in `localStorage` under the key `wheel-of-randomness`. To seed test data, paste JSON into the browser console:

```js
localStorage.setItem('wheel-of-randomness', JSON.stringify({
  "version": 1,
  "genres": [
    {
      "id": "a1b2c3d4e5f6",
      "name": "Mexican",
      "enabled": true,
      "restaurants": [
        { "id": "m001a1b2c3d4", "name": "Taqueria El Sol", "enabled": true },
        { "id": "m002a1b2c3d4", "name": "Chipotle", "enabled": true }
      ]
    }
  ]
}));
location.reload();
```

Or use DevTools > Application > Local Storage to edit the value directly.

Both genres and restaurants have an `enabled` flag. Disabled items are hidden from the wheel but preserved in config for re-enabling later.

## Spin Physics

The spin animation uses a custom easing function modeled on physical friction:

```
progress(t) = 1 - (1 - t^a)^b
```

- **`a`** (1.2-1.5): Controls ramp-up speed. Low values = fast acceleration.
- **`b`** (3.0-4.5): Controls deceleration tail. Higher = longer slowdown.

The velocity curve is a single smooth hump: rises from zero, peaks at ~10% of the duration, then continuously decelerates to zero. This means ~90% of each spin is the slowdown phase.

Each spin randomizes: duration (5-8s), curve shape (`a` and `b`), number of full rotations (3-5), and landing position within the target slice. About 30% of spins land near a slice edge for "will it cross over?!" tension.

## Architecture

Single `index.html` file containing all HTML, CSS, and JS. No build step, no dependencies, no framework. Total size is ~1000 lines.

### Code sections (in order)

| Section | Purpose |
|---------|---------|
| HTML comment block | Architecture overview, tuning history, edge cases |
| `<style>` | Dark theme, screen routing, wheel/config/overlay layout |
| Store | localStorage CRUD with in-memory cache |
| Audio Engine | Web Audio API tick and celebration sounds |
| Wheel Renderer | SVG annular sector generation, text labels with stroke outline, colors, dynamic layout, inner pointer |
| Layout Animation | `animateLayout()` — rAF interpolation of ring radii for transitions |
| Spin Mechanics | rAF loop, parametric easing, tick sound integration |
| State Machine | Spin flow control, screen transitions, layout mode management |
| Config Screen | Accordion UI, event delegation, inline rename |
| Navigation | Button event bindings |
| Init | Load data, render initial state |

### Key design decisions

- **rAF loop over WAAPI**: Web Animations API can't express the asymmetric easing curve needed for the gameshow feel. The rAF loop also lets tick sounds be computed inline (from the current angle) instead of parsing `getComputedStyle` matrices.
- **Single easing formula over piecewise**: `1-(1-t^a)^b` is smooth everywhere with no stitching artifacts. Piecewise approaches (tried and rejected) create visible velocity kinks at phase boundaries.
- **No velocity floor**: A minimum speed clamp was tried and rejected. It creates a constant-speed cruise phase followed by a hard stop, which looks and feels unphysical.
- **Drama via landing position, not easing tricks**: Overshoot/undershoot effects that require re-acceleration look wrong. Instead, landing near slice edges achieves the same tension while the velocity only ever decreases.
- **SVG over Canvas**: Easier hit testing (click on `<path>`), no manual redraw needed, CSS transforms handle rotation.
- **Golden angle colors**: `hsl(i * 137.508 % 360, 85%, 58%)` maximizes hue separation regardless of item count. High saturation (85%) pops against the dark background.
- **Dynamic single→two-ring layout**: The wheel starts as a single fat genre ring. After the genre spin lands, the ring animates inward (via rAF path re-rendering at interpolated radii) to reveal the restaurant ring. This is more dramatic than showing both rings at all times, and avoids the inner ring being empty/confusing before a genre is selected.

## Browser Support

- `crypto.getRandomValues` (works on `file://`)
- Web Audio API (`AudioContext`)
- CSS `min()`, `inset`, `backdrop-filter`
- SVG `transform-origin` (must be `0 0` absolute, not `50% 50%`)

## Deployment

Drop `index.html` on any static host, or enable GitHub Pages on the repo root. No build step required.
