# X-Wing — A Rebel Flight

![X-Wing cinematic scroll experience](./screenshots/hero.png)

> A cinematic, scroll-driven 3D X-Wing experience. Watch the ship dive out of deep space, fill the frame, whip past the lens, then vanish into a distant spiral galaxy.

**Live:** https://franknoris.github.io/Space-Landing-Page/

---

## What it is

A single-page, scroll-driven cinematic built with **Three.js** and **GSAP ScrollTrigger**. The X-Wing flies a scripted path tied directly to scroll position: as you scroll down, the ship approaches, whips past the camera, and rockets into a procedural spiral galaxy. Scroll back up and the entire sequence reverses — including the bank, the camera roll, and the ship's rotation.

Autoplay mode runs the full flight on page load in ~10 seconds, and hands control back to you the moment you touch the wheel.

---

## Features

- **Scroll-driven flight choreography** — 3 phases (approach → whip past → into the void), scrubbed 1:1 to scroll with GSAP ScrollTrigger
- **Real 3D scene** — 5,200 twinkling stars, 40 rotating asteroids, exponential fog, ACES tonemapping
- **Procedural spiral galaxy** — 34,000 GPU points arranged in 4 log-spiral arms + a warm core bulge, animated by a custom vertex shader with **real differential rotation** (inner stars orbit faster than outer ones)
- **Cinematic camera** — always tracks the ship; rolls sideways during the whip-past for a handheld-dogfight feel; FOV punch on flyby; subtle shake at the moment of crossing
- **Idle drift** — after the scroll hits 100%, the ship keeps flying deeper into the galaxy on its own, then smoothly unwinds if you scroll back
- **Autoplay with replay** — auto-runs the flight on load; any user input instantly takes over
- **Typography system** — Space Grotesk + Inter; character-by-character hero reveal, word-mask section headings, blur-in paragraphs, floating kickers
- **Reduced-motion aware** — respects `prefers-reduced-motion` (autoplay is skipped, camera shake is disabled)
- **Fully responsive** — from ultrawide to mobile
- **Performance-conscious** — capped DPR, `lagSmoothing`, single-pass render loop

---

## Tech stack

| | |
|---|---|
| 3D | [Three.js](https://threejs.org/) r170 (ES modules via importmap) |
| Animation | [GSAP 3.12](https://gsap.com/) + ScrollTrigger |
| Fonts | Google Fonts — Space Grotesk, Inter |
| Hosting | GitHub Pages |
| Build | None — pure HTML/CSS/JS, single file |

---

## Project structure

```
.
├── index.html                 # Everything: markup, styles, and the full 3D scene
├── assets/
│   └── model.glb              # Optional — real X-Wing model (see "Using a real model")
├── screenshots/
│   └── hero.png               # README preview image
└── README.md
```

Everything lives in `index.html`. No build step, no bundler, no dependencies to install.

---

## Running locally

Because `GLTFLoader` fetches the model over HTTP, you can't just double-click `index.html` if you're using the `.glb`. Serve the folder:

```bash
# Python
python -m http.server 8000

# Node
npx serve .

# Or VS Code → Live Server extension → "Open with Live Server"
```

Then visit **http://localhost:8000**.

If you're only using the placeholder primitives, double-clicking `index.html` works fine.

---

## Deploying

The repo is already set up for **GitHub Pages**:

1. **Settings → Pages** → Source: `Deploy from a branch`
2. Branch: `main`, folder: `/ (root)`
3. Save. Your site goes live at `https://<username>.github.io/<repo>/` in ~1 minute.

Updates deploy automatically on every `git push`.

> ⚠️ GitHub Pages is case-sensitive and serves from a subfolder. Always use **relative paths** (`./assets/model.glb`), never absolute (`/assets/model.glb`).

---

## Customization

### Flight timing

Near the top of the `<script type="module">` block:

```js
const AUTOPLAY_SECONDS = 10;   // how long autoplay runs
```

Phase pacing lives inside `buildScrollTimeline()` — each `.to()` call has a `duration` and a `position` (start time) that map directly to scroll progress 0→1. Change the durations and keep them summing to `1.0`.

### Scene density

```js
const STAR_COUNT     = 5200;   // in createStars()
const ASTEROID_COUNT = 40;     // in createAsteroids()
```

The galaxy is built from three point clouds in `createGalaxy()` — bulge (9k), arms (18k), haze (7k). Tweak `RADIUS`, `branches`, `spin`, and `armWidth` to reshape the spiral.

### Colors

All accent colors are CSS variables at the top of `<style>`:

```css
:root {
  --accent:     #7ad0ff;   /* cool blue — kickers, headings */
  --accent-hot: #ff9a55;   /* warm orange — engine glow, finale */
}
```

### Using a real `.glb` model

The scene ships with a placeholder X-Wing built from Three.js primitives. To swap in a real model:

1. Drop your `.glb` into `assets/`
2. Uncomment the `GLTFLoader` import at the top of the module script
3. Replace `createXWingPlaceholder()` with the loader block

Everything downstream animates the outer `xwing` group, so no flight code needs to change. If the model faces the wrong way, adjust `rotationY` (`0`, `Math.PI`, `±Math.PI/2`) until the nose faces the camera at rest.

---

## How the scroll choreography works

Scroll progress (0 → 1) maps to three linear phases:

| Phase | Progress | What happens |
|---|---|---|
| **Approach** | 0.00 → 0.35 | Ship dives in from z ≈ −100 straight at the lens |
| **Whip Past** | 0.35 → 0.60 | Ship crosses the camera plane, banking hard; camera rolls sideways |
| **Into the Void** | 0.60 → 1.00 | Ship arcs up and away, shrinking into the galaxy |

Every phase tween uses `ease: 'none'` (linear), so velocity is **constant within a phase** and only "gear-shifts" at the boundaries — no ease-in/out = no stalling at phase transitions. The velocity graph accelerates monotonically: `349 → 388 → 670` units per scroll-progress.

The camera always calls `camera.lookAt(ship)`, then `camera.rotateZ(roll)` — so the whip-pan and the cinematic bank are emergent from the ship's world position, not manually keyed.

---

## Browser support

Modern evergreen browsers with WebGL2:

- Chrome / Edge 90+
- Safari 15+
- Firefox 90+

Requires ES modules and CSS `backdrop-filter` (for the badge UI). If `backdrop-filter` is missing, the UI still works — just without the frosted-glass effect.

---

## License

MIT — do what you want. Star Wars and the X-Wing design are trademarks of Lucasfilm / Disney; this is a fan project for personal and portfolio use only.

---

## Credits

- 3D — [Three.js](https://threejs.org/)
- Animation — [GSAP](https://gsap.com/) by GreenSock
- Type — [Space Grotesk](https://fonts.google.com/specimen/Space+Grotesk) & [Inter](https://fonts.google.com/specimen/Inter)
- Built with a lot of scrolling back and forth