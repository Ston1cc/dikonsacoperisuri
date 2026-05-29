# Acoperișuri Dikons — Codebase Guide

## Project Overview

**Acoperișuri Dikons** is a marketing website for a roofing company based in Chișinău, Moldova.
It combines a 5-page SPA marketing site with an interactive 3D roof configurator built on Three.js.

- **Stack:** Vanilla JavaScript, HTML5, CSS3 (custom properties), Three.js v0.160.0
- **Build system:** None — plain static files, no npm, no bundler, no transpiler
- **Languages used in content:** Romanian
- **Deployment:** Any static host (Netlify, GitHub Pages, Vercel, etc.)

---

## File Structure

```
/
├── index.html           # Marketing website (5-page SPA, ~1 250 lines)
├── configurator.html    # 3D roof configurator wizard (~1 450 lines)
├── img/                 # Brand + portfolio photos (~1.5 MB)
│   ├── logo.jpg
│   ├── hero.jpg
│   └── proj-*.jpg       # 6 portfolio reference photos
└── tex/                 # PBR texture maps for 3D materials (~198 MB)
    ├── *.jpg            # 18 roofing material swatch images
    ├── sindrila/        # Bituminous shingle PBR maps (Color, Normal, Roughness, AO, Displacement)
    └── tigla/           # Modular tile PBR maps (same set)
```

> The `.blend`, `.mtlx`, `.usdc`, `.tres` files inside `tex/` are source/export files from
> Blender/Godot and are **not loaded by the web app** — do not reference them in HTML/JS.

---

## index.html — Marketing SPA

### Pages (client-side routing via `showPage(id)`)

| Page ID      | Romanian label | Content |
|--------------|----------------|---------|
| `home`       | Acasă          | Hero, stats ribbon, configurator iframe, value props, pricing |
| `services`   | Servicii       | 6 service cards, 4-step process |
| `portfolio`  | Portofoliu     | 9 filterable project cards |
| `about`      | Despre         | History, values, team |
| `contact`    | Contact        | Form + location/hours |

### Design System

All visual tokens are CSS custom properties on `:root`:

```css
--bg:          #F7F4F0   /* page background (light) */
--dark:        #1A1714   /* primary text */
--orange:      #E8621A   /* brand accent — CTAs, highlights */
--sand:        #D9CEC3   /* borders, dividers */
--mid:         #8A7F74   /* secondary text */
--white:       #FFFFFF
--card:        #FEFDFC   /* card surface */
--orange-soft: #FFF3EC   /* tinted background areas */
```

**Fonts** (Google Fonts, preconnected):
- `'Syne'` — display/headings, weight 700–800
- `'Outfit'` — body text, weight 300–600

Dark mode: toggled by adding `.dark` class to `<html>`. The vars are overridden inside `:root.dark`.

### Responsive Breakpoints

- `900px` — main layout collapse (nav, grids)
- `600px` — compact adjustments (font sizes, padding)
- Scroll-reveal animations are **disabled** on `≤768px` (Intersection Observer skipped)

---

## configurator.html — 3D Roof Configurator

### Architecture

| Concern | Implementation |
|---------|----------------|
| 3D rendering | Three.js v0.160.0 (CDN) — WebGL, soft shadows, ACES filmic tone mapping |
| Geometry | `GeoBuilder` class — procedural mesh generation, no pre-built models |
| Materials | PBR (`MeshStandardMaterial`) — color, normal maps, roughness from `tex/` |
| Camera | Orbit controls (mouse drag, pinch zoom, touch pan) — custom, no OrbitControls import |
| State | Global `S` object holds all wizard choices |
| Theme | Inherits `dark`/`light` from `<html>` class; scene background changes dynamically |

### 5-Step Wizard

1. **Shape** — gable (2-pante), hip (4-pante), L-shape, T-shape
2. **Volumetry** — width, depth, overhang, wing dimensions (L/T only)
3. **Material & Color** — Țiglă Modulară (6 colors) or Sindrila Bituminoasă (8 colors)
4. **Drainage** — gutters/downspouts toggle + 6 color options
5. **Estimate** — price formula `area × 59€` to `area × 59€ × 1.35`; contact form

### Price Calculation

```
area = roof surface area in m²
min_price = area × 59
max_price = area × 59 × 1.35    (VIP tier)
```

### Mobile Layout

- Desktop: `3D viewer | 400px config panel` side-by-side
- Mobile (≤900px): vertical stack — viewer on top (`42vh` tablets, `40vh` phones)
- Step indicator shrinks to `"1 / 5"` text on small screens
- Larger tap targets for all interactive controls

---

## Development Workflow

### Running locally

No build step needed — just serve the directory over HTTP:

```bash
# Python (built-in)
python3 -m http.server 8080

# Node (npx)
npx serve .
```

Open `http://localhost:8080` for the marketing site and
`http://localhost:8080/configurator.html` for the 3D tool.

> Opening `index.html` directly as a `file://` URL works for the marketing site but may
> block the configurator iframe due to browser CORS restrictions on local files.

### Making changes

- All CSS is embedded in `<style>` tags inside each HTML file — no separate stylesheet.
- All JavaScript is embedded in `<script>` tags — no separate JS files.
- Edit `index.html` for marketing copy/layout; edit `configurator.html` for the 3D tool.
- There is no hot-reload; refresh the browser after saving.

### Screenshot comparison workflow

When making visual changes (layout, spacing, color, typography):

1. **Screenshot** the rendered page:
   ```bash
   npx puppeteer screenshot index.html --fullpage
   # or use the browser DevTools → "Capture full size screenshot"
   ```
2. **Compare** against the reference image. Check:
   - Spacing and padding (measure in px)
   - Font sizes, weights, and line heights
   - Colors (exact hex values against design tokens above)
   - Alignment and positioning
   - Border radii, shadows, effects
   - Responsive behavior at 375px, 768px, 1280px
3. **Fix** every mismatch found.
4. **Re-screenshot** and repeat until no visible differences remain (target: ≤3px).

Always do at least **2 comparison rounds** before declaring a visual task done.

---

## Key Conventions

### CSS

- Use existing CSS custom properties (`--orange`, `--bg`, etc.) — never hardcode hex colors.
- All sizing uses `clamp()` for fluid typography; maintain this pattern for new headings.
- Do not introduce Tailwind or any CSS framework — the project uses hand-written custom CSS.

### JavaScript

- No framework, no TypeScript, no imports — plain ES2020 browser JS.
- The configurator's global state lives in the `S` object; add new wizard state there.
- `GeoBuilder` methods return `THREE.Group` — add new geometry methods following the same pattern.
- Event listeners are registered directly on DOM elements (no delegation library).

### Assets

- New project photos go in `img/` and should be ≤400 KB (compress before committing).
- Texture maps go in `tex/` (or a subdirectory per material); keep the PBR naming convention:
  `*_Color.jpg`, `*_NormalGL.jpg`, `*_Roughness.jpg`, `*_AmbientOcclusion.jpg`.
- Do **not** commit `.blend`, `.usdc`, or `.tres` files unless they replace existing ones.

### Content

- All user-visible text is in Romanian.
- Company details: phone `+373 78400731`, email `office@acoperisuridikons.md`,
  address `Str. Grădina Botanică 2/1, Chișinău`.
- Pricing: €59/m² (Standard) to €79.65/m² (VIP).

---

## What NOT to do

- Do not add a package.json / build step unless the project explicitly moves to a bundler.
- Do not split HTML into component files — keep each page self-contained.
- Do not introduce React, Vue, or any JS framework.
- Do not use Tailwind classes — the site uses its own design token system.
- Do not load 3D texture files that are `>5 MB` on page init; lazy-load them on material selection.
- Do not commit the raw PBR source files (`*.blend`, `*.usdc`, `*.mtlx`, `*.tres`) for new assets.
