# Patch Osodo Okomo — Portfolio Website

> Personal portfolio for **Patch Osodo Okomo**, IT Support & Systems Analyst based in Nairobi, Kenya.
> Live at: [https://patchosodo.github.io](https://patchosodo.github.io)

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Performance Optimisations](#performance-optimisations)
- [Font Strategy](#font-strategy)
- [Color Theme](#color-theme)
- [Sections](#sections)
- [Local Development](#local-development)
- [Deployment](#deployment)
- [Updating Content](#updating-content)
- [Browser Support](#browser-support)

---

## Overview

A production-grade, single-page portfolio website built with vanilla HTML, CSS, and JavaScript — no frameworks, no dependencies, no build tools required. Designed around a **precision dark** aesthetic with a green accent theme, monospaced terminal elements, and smooth scroll-reveal animations.

Built for performance on low-bandwidth connections (Safaricom/Airtel mobile networks in Kenya) with a target First Contentful Paint under 1 second.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Markup | Semantic HTML5 |
| Styles | Vanilla CSS3 — extracted and minified |
| Scripts | Vanilla ES5 JavaScript — minified and deferred |
| Fonts | Montserrat (self-hosted) · IBM Plex Mono · Syne (Google Fonts CDN) |
| Hosting | GitHub Pages |
| Version Control | Git / GitHub Desktop |

No npm. No webpack. No React. Just files.

---

## Project Structure

```
PatchOsodo.github.io/
│
├── index.html                          # Main HTML shell — no inline CSS or JS
│
└── assets/
    ├── css/
    │   └── style.min.css               # All styles, minified
    │
    ├── js/
    │   └── main.min.js                 # All scripts, minified — loaded with defer
    │
    └── fonts/
        ├── fonts.css                   # @font-face declarations for self-hosted fonts
        └── montserrat-v31-latin-regular.woff2   # Self-hosted body font (latin subset)
```

---

## Performance Optimisations

### CSS — Preload for Fastest First Paint
```html
<link rel="preload" href="assets/css/style.min.css" as="style">
<link rel="stylesheet" href="assets/css/style.min.css">
```
`rel="preload"` tells the browser to fetch the stylesheet at the highest priority during HTML parsing — it arrives before the render tree needs it, eliminating render-blocking delay on repeat visits.

### JS — Deferred, Non-Blocking
```html
<script src="assets/js/main.min.js" defer></script>
```
`defer` fetches the script in parallel with HTML parsing but only executes after the DOM is fully built. Zero render-blocking JavaScript. No `DOMContentLoaded` wrapper needed.

### Fonts — Non-Blocking Load
```html
<link rel="stylesheet" href="..." media="print" onload="this.media='all'">
```
The `media="print"` trick loads the Google Fonts CSS without blocking the render path. `display=swap` in the font URL prevents invisible text during font load (FOIT).

### Animations — GPU Compositor Only
All animations use only `opacity` and `transform` — properties handled entirely on the GPU compositor thread. No `width`, `height`, `top`, `left`, or `margin` animations that would trigger layout recalculation.

### Scroll Reveal — Single Observer
One `IntersectionObserver` instance watches all `.reveal` elements. Each element is `unobserve()`d immediately after it becomes visible — the observer never watches elements that are already done.

### RAF Loop — Paused When Hidden
The custom cursor `requestAnimationFrame` loop calls `cancelAnimationFrame` when the browser tab is hidden (`visibilitychange` event) and resumes when the tab regains focus — conserves battery on mobile Chrome.

### Custom Cursor — Fine Pointer Devices Only
```javascript
if (window.matchMedia('(pointer:fine)').matches) { ... }
```
The cursor and `cursor:none` body style only activate on devices with a precise pointer (mouse/trackpad). Touch screens and stylus devices never run this code.

---

## Font Strategy

### Current Setup (Phase 1 — CDN)
Two of three font families are loaded from Google Fonts CDN. Only 4 weights total are requested — down from a potential 8+, roughly halving the font payload.

```
IBM Plex Mono  — weights 400, 600  (nav, terminal, tags, code elements)
Syne           — weights 700, 800  (headings, section titles, hero name)
Montserrat     — weight  400       (body text — self-hosted)
```

### Self-Hosted Fonts (Phase 2 — Full Self-Host)
To eliminate the Google Fonts dependency entirely and remove the external DNS lookup:

1. Download the required `.woff2` files from [Google Webfonts Helper](https://gwfh.mranftl.com)
2. Place them in `assets/fonts/`
3. Replace the Google `<link>` tags in `index.html` with:
```html
<link rel="stylesheet" href="assets/fonts/fonts.css">
```
4. Add preload hints for the two critical above-the-fold fonts:
```html
<link rel="preload" href="assets/fonts/syne-v22-latin-800.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="assets/fonts/ibm-plex-mono-v19-latin-regular.woff2" as="font" type="font/woff2" crossorigin>
```

The `@font-face` declarations with `unicode-range` for Latin-only subsets are already written in `assets/fonts/fonts.css` — just drop the woff2 files in and swap the link tags.

### Why unicode-range?
Each `@font-face` block includes a `unicode-range` descriptor limiting the font to Latin characters used in English. The browser only downloads a font file if a character from its range is actually used on the page — cuts file size ~60% vs a full font download.

---

## Color Theme

The site uses CSS custom properties for the entire color system. All accent colors reference `--green` — changing the theme requires updating three values in `:root` inside `style.min.css`.

```css
:root {
  --green:     #22c55e;                    /* Primary accent — emerald green */
  --green-dim: rgba(34, 197, 94, 0.12);   /* Subtle background tints */
  --green-mid: rgba(34, 197, 94, 0.35);   /* Borders, mid-opacity elements */

  --bg:   #090d12;   /* Page background — deep navy */
  --bg2:  #0e1520;   /* Section alternate background */
  --bg3:  #131d2b;   /* Card backgrounds */
  --text: #e8edf5;   /* Primary text */
  --muted:#6b7a90;   /* Secondary/muted text */
  --sub:  #8a96a8;   /* Tertiary text */
  --teal: #00c9a7;   /* Terminal prompt color */
}
```

To change the accent color: update `--green`, `--green-dim`, and `--green-mid`. Everything cascades automatically — no other changes needed.

---

## Sections

| Section | ID | Description |
|---|---|---|
| Hero | `#hero` | Name, title, CTA buttons, terminal widget |
| Stats | `.stats-bar` | 4 key metrics — years, users, uptime, resolution rate |
| About | `#about` | Bio paragraphs + technology stack tags |
| Experience | `#experience` | Timeline of roles at Sidian Bank |
| Skills | `#skills` | 6 skill cards with descriptions and tech tags |
| Projects | `#projects` | 4 production project cards |
| Certifications | `#certifications` | 6 credential cards |
| Contact | `#contact` | Email, phone, LinkedIn, GitHub links |

---

## Local Development

No build tools required. Open directly in a browser:

```bash
# Option 1 — just open the file
# Double-click index.html in Explorer

# Option 2 — serve locally (avoids any CORS issues with font files)
# Python 3
python -m http.server 8000
# Then open http://localhost:8000

# Node (if installed)
npx serve .
# Then open http://localhost:3000
```

> **Note:** Self-hosted fonts may not load when opening `index.html` directly as a `file://` URL due to browser security restrictions on local font files. Use a local server (Option 2) if fonts appear to fall back to system fonts.

---

## Deployment

This site is hosted on **GitHub Pages** from the `main` branch root.

### Publishing a Update via GitHub Desktop

1. Make changes to files locally
2. Open **GitHub Desktop**
3. Review changed files in the left panel
4. Write a commit message describing what changed
5. Click **Commit to main**
6. Click **Push origin**
7. Wait ~60 seconds — changes are live at [https://patchosodo.github.io](https://patchosodo.github.io)

### GitHub Pages Settings
- **Source:** Deploy from branch
- **Branch:** `main`
- **Folder:** `/ (root)`

Settings location: Repository → Settings → Pages

---

## Updating Content

All content lives directly in `index.html`. No CMS, no database.

### Common Updates

**Change a job title or description:**
Open `index.html`, find the relevant section by its `id` (e.g., `id="experience"`), edit the text directly.

**Add a new project:**
Copy an existing `.project-card` div block inside `#projects`, paste it after the last card, update the number, name, description, and tags.

**Add a new skill tag:**
Find the relevant `.stack-tags` div inside `#about` and add:
```html
<span class="tag">Your New Tag</span>
```
Use `class="tag green"` to highlight it in the accent color.

**Update contact details:**
Find `id="contact"` and update the `href` values on the `.contact-link` anchors.

**Change the accent color:**
Open `assets/css/style.min.css`, find `:root` at the top, update `--green`, `--green-dim`, and `--green-mid`.

**Add a new font weight:**
Add a new `@font-face` block to `assets/fonts/fonts.css` with the correct `font-weight` value and the downloaded `.woff2` file path.

---

## Browser Support

| Browser | Support |
|---|---|
| Chrome 90+ | ✅ Full |
| Firefox 88+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Edge 90+ | ✅ Full |
| Mobile Chrome (Android) | ✅ Full |
| Mobile Safari (iOS) | ✅ Full |
| IE 11 | ❌ Not supported |

CSS features used: Custom properties, CSS Grid, `clamp()`, `mask-image`, `clip-path`, `IntersectionObserver`. All are baseline-supported in modern browsers. IE11 is not targeted.

The `prefers-reduced-motion` media query is respected — all animations and transitions are disabled for users who have enabled this accessibility setting in their OS.

---

## Author

**Patch Osodo Okomo**
IT Support & Systems Analyst · Nairobi, Kenya

- Email: okomopatchosodo@gmail.com
- LinkedIn: [linkedin.com/in/osodopatch](https://linkedin.com/in/osodopatch)
- GitHub: [github.com/PatchOsodo](https://github.com/PatchOsodo)

---

*Built with precision. No frameworks harmed.*
