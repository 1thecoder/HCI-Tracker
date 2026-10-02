# Usability Tracker & Analytics Dashboard

A JavaScript usability tracking engine for an HCI assignment. It captures real
user interaction, turns it into usability metrics, and presents the results in
two places:

1. **A floating tracker panel** that sits on the page being tested — live stats,
   insights and three heatmap overlays.
2. **A full analytics dashboard** — a separate console page with charts,
   funnels, a click map and the complete event log.

Built on the sample tracker provided in class, extended with frustration
detection, a scored report, and a visualisation layer. No libraries, no build
step, no network requests.

---

## Quick start

Open a terminal in this folder and start a local server:

```bash
python -m http.server 5500
```

| Page | URL |
|------|-----|
| Demo site (the page being tested) | <http://localhost:5500/index.html> |
| Analytics dashboard | <http://localhost:5500/dashboard.html> |

Click around the demo page for a few seconds, then open the dashboard. A local
server is needed because the code is split into separate script files.

---

## Files

| File | What it does |
|------|--------------|
| `index.html` | Demo site (a cafe ordering page) with planted usability problems |
| `dashboard.html` | The analytics console |
| `tracker/tracker-core.js` | Capture engine, metrics, persistence and export. No UI. |
| `tracker/tracker-heatmap.js` | Canvas heatmap and click-marker renderer |
| `tracker/tracker-ui.js` | The floating panel on the page under test |
| `tracker/tracker-ui.css` | Panel styling (light and dark) |
| `dashboard/palette.js` | The validated chart colour palette |
| `dashboard/charts.js` | SVG chart library — area, line, donut, bar, funnel, scatter, gauge |
| `dashboard/dashboard.js` | Dashboard application |
| `dashboard/dashboard.css` | Dashboard styling (light and dark) |

The layers are independent. `tracker-core.js` runs on its own if you only want
the data; the heatmap, the panel and the dashboard are each optional on top.

---

## Using it on any page

Add these before `</body>`:

```html
<link rel="stylesheet" href="tracker/tracker-ui.css">
<script src="tracker/tracker-core.js"></script>
<script src="tracker/tracker-heatmap.js"></script>
<script src="tracker/tracker-ui.js"></script>
```

Order matters: the core must load first.

---

## What it records

| Event | Captured when |
|-------|---------------|
| `click` | Any click, with page coordinates, element, label and whether the target was interactive |
| `mousemove` | Pointer position, sampled every 120ms |
| `scroll` | Scroll depth as a percentage of the page |
| `keypress` | That a key was pressed, and which element had focus |
| `focus` | A form field gained focus |
| `submit` | A form was submitted |
| `hesitation` | The pointer rested on one element for more than 2 seconds |
| `rageclick` | 3 or more clicks within 40px and 1 second |
| `deadclick` | A click on a non-interactive element that produced no DOM change |
| `resize` / `visibility` | Window resized, or the tab hidden or shown |

### Derived metrics

- **Session score (0–100)** — one headline number for session health
- **Click accuracy** — percentage of clicks that hit an interactive element
- **Click outcomes** — hit a control / missed / no response / frustrated
- **Max scroll depth** — how far down the page the user actually got
- **Mouse travel** — total pointer distance in pixels
- **Most clicked elements** — ranked
- **Insights** — the above translated into plain-language findings

---

## The floating panel

Open it with the purple **Tracking** pill at the bottom right, or press
<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>U</kbd>. Drag the header to move it;
<kbd>Esc</kbd> closes it.

- **Overview** — score gauge, stat cards, a 60-second activity sparkline, an
  interaction-quality bar, scroll depth, insights and the most-clicked list.
- **Heatmap** — three overlays: a click heatmap, a movement heatmap, and
  numbered click markers with an optional path. Blue is a normal click, grey
  missed the target, orange is a dead click, red is a rage click.
- **Events** — a live log, newest first, filterable by type.
- **Export** — open the full dashboard, download JSON or CSV, copy to the
  clipboard, or print a formatted summary to the console.

The overlay is a full-page canvas with `pointer-events: none`, so the page
stays usable while it is on screen.

---

## The analytics dashboard

`dashboard.html` reads the sessions the tracker saved in this browser. It
updates live while the demo page is open in another tab.

**Filter row** — one row above everything it scopes: a session picker and a
time range. Every chart below re-renders against the same slice.

| Chart | Form | Why that form |
|-------|------|---------------|
| Session health score | Radial gauge | One headline number, so it is a figure not a chart |
| Interaction activity | Area chart | Change over time, one series |
| Scroll depth over time | Line chart | Change over time, one series, one axis |
| Page reach funnel | Ordinal bars | Ordered stages, so an ordinal ramp |
| Event type mix | Donut | Part-to-whole at a glance, capped at six segments |
| Most clicked elements | Horizontal bars | Magnitude comparison, one series one colour |
| Click outcomes | Segmented bar | Part-to-whole with status meaning |
| Where clicks landed | Scatter plot | Position on the page |
| Raw event log | Table | The data behind every chart |

Every chart has a **Chart / Table** toggle, so no value is reachable by colour
or hover alone. Hovering any mark shows a tooltip; keyboard focus shows the
same.

---

## Chart design decisions

The palette was **validated with a script, not chosen by eye**:

- **Categorical** (event types) — six hues in a fixed slot order, never cycled.
  Light mode: worst adjacent colour-blind ΔE 9.1, normal-vision ΔE 19.6. Dark
  mode: all six checks pass at ≥ 3:1 contrast. Three light-mode hues fall below
  3:1, so those charts ship direct labels and a table view as the relief.
- **Ordinal** (the funnel) — one hue, light to dark, monotone lightness with
  visible steps.
- **Sequential** (single-series charts) — one hue, never a value ramp across
  nominal categories.
- **Status** (click outcomes) — reserved good / warning / serious / critical
  steps, never reused as a series colour, and always paired with a text label.

Other rules followed: no dual-axis charts anywhere; solid hairline gridlines,
never dashed; a 2px surface gap between stacked segments rather than borders;
direct labels only on the endpoint or extreme, never on every point;
proportional figures on hero numbers and `tabular-nums` only in table columns
and axis ticks.

---

## Testing it on the demo page

Three usability problems are planted in the demo page deliberately:

1. **Two fake buttons.** The grey "Add to cart" boxes on Cold Brew and Chai
   Latte are `<div>` elements styled to look like buttons. Clicking them
   repeatedly produces **dead clicks** and then a **rage click**.
2. **A tiny tap target.** The "see full nutritional information" link is 10px
   and low contrast, so clicks near it miss and drag accuracy down.
3. **Content below the fold.** The order form is far down the page, so scroll
   depth shows whether a user ever reached it.

Verified behaviour: a scripted session of 81 events produced 12 clicks — 6 that
hit a control, 1 missed, 4 dead and 1 rage — a score of 43 ("Poor"), 90% scroll
depth, and a click heatmap with a red hotspot on the fake button directly
beside a cool blue blob on the real one.

---

## JavaScript API

```js
UsabilityTracker.getLogs()           // every captured event
UsabilityTracker.getSummary()        // metrics object
UsabilityTracker.getScore()          // { value, grade, tone }
UsabilityTracker.getInsights()       // plain-language findings
UsabilityTracker.getTopElements(5)   // most-clicked elements
UsabilityTracker.getClickBreakdown() // { ok, missed, dead, rage, total }
UsabilityTracker.getActivitySeries() // events bucketed over time

UsabilityTracker.exportJSON()        // JSON string
UsabilityTracker.exportCSV()         // CSV string
UsabilityTracker.downloadJSON()      // triggers a file download
UsabilityTracker.downloadCSV()

UsabilityTracker.showPanel()         // open the floating panel
UsabilityTracker.hidePanel()
UsabilityTracker.showHeatmap('clicks')  // 'clicks' | 'movement' | 'markers' | 'off'

UsabilityTracker.saveSession()         // write a snapshot for the dashboard
UsabilityTracker.getStoredSessions()   // every saved session
UsabilityTracker.clearStoredSessions()
UsabilityTracker.reset()               // clear the current session

UsabilityTracker.onEvent(function (event) { /* stream events live */ });
```

Settings live in `UsabilityTracker.config` (sample rates, rage-click
thresholds, the memory cap).

---

## Design notes

**Separation of concerns.** Capture, visualisation and interface are separate
files with one job each. The core never touches the DOM for display, so the
panel and the dashboard can both be removed without affecting the data.

**The tracker ignores itself.** Everything in the panel carries a `data-ut-ui`
attribute, and all listeners skip those elements. This also matters for
dead-click detection: the panel's own DOM updates would otherwise look like the
page responding to every click.

**Bounded memory.** The live log is capped at 8000 events, dropping the oldest
*mousemove* samples first so clicks and problem events are never lost. Saved
snapshots keep every meaningful event but thin the movement stream to 400
evenly spread samples, and only the six newest sessions are kept.

**Live refresh never interrupts reading.** The dashboard re-renders only when
the data actually changed, and never while the pointer or keyboard focus is
inside the board — so a refresh cannot pull a chart out from under a hover or
reset a table you just opened.

**Heatmap technique.** Each point is drawn as a greyscale radial blob so
overlapping activity accumulates in the alpha channel. The canvas is then
recoloured by mapping alpha through a blue-green-yellow-red palette, in
horizontal strips to keep memory use low on tall pages.

**Privacy.** Keystrokes are counted but never recorded — only navigation keys
(Tab, Enter, Escape, Backspace) are named, everything else logs as `character`.
Input values are replaced with a placeholder such as `[email field]`. Nothing
leaves the browser: there is no network request anywhere in the tracker or the
dashboard.

---

## Accessibility

Full keyboard access with visible focus rings, a skip link, ARIA roles on tabs
and panels, labelled icon buttons, hit targets larger than the marks they
cover, a table view for every chart, and support for `prefers-reduced-motion`
and `prefers-color-scheme`. The dashboard also has an explicit light/dark
toggle that overrides the system setting.
