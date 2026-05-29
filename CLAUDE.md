# Garden — Claude Code Guide

## Project Overview

An interactive, offline-capable HTML gardening checklist for **Dallas, PA (Zone 6a, Back Mountain, Luzerne County)**. Users track weekly tasks from March through October, with checkboxes that persist via `localStorage`. No backend, no build tools, no dependencies.

---

## Repository Structure

```
Garden/
├── index.html                 # Primary application (serve this)
├── gardening-checklist.html   # Identical copy of index.html (GitHub Pages fallback)
├── gardening-checklist.md     # Markdown source — ground truth for content
└── CLAUDE.md                  # This file
```

`index.html` and `gardening-checklist.html` are byte-for-byte identical. When updating content, update **both** files.

---

## Technology Stack

| Layer | Choice |
|---|---|
| Markup | HTML5 (semantic: `<section>`, `<table>`, `<blockquote>`, `<ul>`) |
| Styles | CSS3 embedded in `<style>` — no external sheets |
| Logic | Vanilla ES5 JavaScript embedded in `<script>` |
| Persistence | Browser `localStorage` |
| Build | None — ship the file directly |
| Deploy | GitHub Pages (static hosting) |
| Tests | None — manual browser verification |

---

## index.html Internal Structure

The file is ~953 lines organized in three logical blocks:

1. **`<head>` + `<style>`** (lines 1–~150): HTML meta, embedded CSS (~55 rules). Earth-tone palette: greens `#8a9a5b`, golds `#c2a96e`, browns `#5a3e1b`. Max content width 900px.

2. **`<body>` content** (lines ~150–780): Semantic HTML. Three sections:
   - Crop reference table (14 vegetables/flowers)
   - Weekly focus sections (weeks 1–30+, each with a `<blockquote>` context note + `<ul>` checkbox list)
   - Progress sidebar (overall bar + per-month breakdown)

3. **`<script>` IIFE** (lines ~780–951): All JavaScript is wrapped in `(function () { ... })();` to avoid global namespace pollution. Only `window.resetSeason` is intentionally global (called from a `<button onclick>`).

---

## JavaScript Patterns

### localStorage Key Convention

```js
var PREFIX = 'garden-';
// Keys look like: garden-w3-task2
```

Checkbox IDs follow `w{weekNum}-task{taskIndex}` (e.g., `w3-task2`). The prefix namespaces garden data in `localStorage`.

### WEEK_MAP — Date-Range Lookup

```js
// [startMonth, startDay, endMonth, endDay] — months are 0-indexed (JS Date.getMonth())
var WEEK_MAP = {
  1:  [2,15, 2,21],  // March 15–21
  ...
  30: [9,15, 10,31]  // Sept 15 – Oct 31
};
```

`getCurrentWeekNum()` compares today's `Date` against `WEEK_MAP` to auto-highlight the current week.

### Month-to-Week Mapping (for progress bars)

Week numbers map to month labels via hard-coded ranges in `getMonthForCheckbox()`:

```
weeks 1–3   → MARCH
weeks 4–7   → APRIL
weeks 8–11  → MAY
weeks 12–15 → JUNE
weeks 16–19 → JULY
weeks 20–23 → AUGUST
weeks 24–27 → SEPTEMBER
weeks 28+   → OCTOBER
```

### Event Delegation

All checkbox changes use a single delegated listener on `document`:

```js
document.addEventListener('change', function (e) {
  if (e.target.type === 'checkbox' && e.target.id) { ... }
});
```

---

## Content Conventions

### Adding a New Week Section

Each week in the HTML follows this exact pattern:

```html
<h4 id="week-N">Week N: [Month Day–Day]</h4>
<blockquote>
  <p><strong>You are here:</strong> [context note, avg temps, etc.]</p>
</blockquote>
<ul>
  <li><label><input type="checkbox" id="wN-task1"> Task description</label></li>
  <li><label><input type="checkbox" id="wN-task2"> Task description</label></li>
</ul>
```

- `id="week-N"` on the `<h4>` is required for the current-week highlight scroll target.
- `id="wN-taskM"` on each checkbox drives localStorage persistence and progress counting.
- Tasks are 0-indexed in `WEEK_MAP` months (January = 0, March = 2).

### Updating the Crop Table

Edit the `<table>` near the top of `<body>`. No JavaScript references it; it is purely informational.

### Markdown Source

`gardening-checklist.md` is the canonical content source. For large content edits (e.g., adding new crops, reorganizing weeks), edit the Markdown first, then port changes to both HTML files.

---

## Development Workflow

### Editing

No build step. Open `index.html` directly in a browser or use any static server:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

### Syncing the Two HTML Files

After editing `index.html`, keep `gardening-checklist.html` in sync:

```bash
cp index.html gardening-checklist.html
```

### Deployment

Push to the remote; GitHub Pages serves `index.html` automatically. No CI pipeline exists.

```bash
git add index.html gardening-checklist.html
git commit -m "..."
git push -u origin <branch>
```

---

## Key Constraints

- **No external dependencies.** Do not add CDN links, npm packages, or `import` statements. The page must work fully offline.
- **ES5 only.** The JavaScript avoids `const`, `let`, arrow functions, template literals, and other ES6+ features to maximize browser compatibility without a transpiler.
- **Single-file principle.** CSS and JS stay embedded in `index.html`. Do not split into separate `.css` or `.js` files unless explicitly requested.
- **No backend.** All persistence is `localStorage`. There are no API calls or server-side requirements.
- **Print-safe.** The `@media print` block strips the sidebar and interactive controls. Changes to layout should be tested in print preview.

---

## Geographic / Domain Context

- **Location:** Dallas, Pennsylvania — Back Mountain region, Luzerne County
- **USDA Hardiness Zone:** 6a (avg minimum −10°F to −5°F)
- **Growing season:** March (seed starting) through October (final harvest/cleanup)
- **Reference:** Penn State Extension for regional planting dates
- **Crops tracked:** tomatoes, peppers, cucumbers, beans, peas, squash, basil, lettuce, beets, root vegetables, cosmos, scarlet flax, iris, sunflowers

When adding planting dates or frost guidance, use Penn State Extension resources for Zone 6a Northeast Pennsylvania.
