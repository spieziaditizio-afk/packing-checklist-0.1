# Outbound Workflow Animation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `workflow-animation.html`, a self-contained, offline-capable browser animation of the complete 10-step Sevenum outbound process.

**Architecture:** One HTML file. A static SVG scene of the warehouse plan, plus `<g>` groups for every moving actor. A single `requestAnimationFrame` clock advances one variable `t`; every actor's position is a pure function of `t`, so any seek — forwards or backwards — renders a correct frame. Steps are entries in a `STEPS` registry, each with `reset()` and `render(localT)`.

**Tech Stack:** Plain HTML, inline CSS, inline SVG, vanilla JS. No build step, no framework, no runtime CDN dependency except an optional web font with a full system fallback.

**Spec:** `docs/superpowers/specs/2026-09-13-outbound-workflow-animation-design.md`

## Global Constraints

- **Single file.** Everything inline in `workflow-animation.html` at the repo root. This is not merely convention: the file is opened by double-click over `file://`, where ES module imports are blocked by CORS. Splitting into modules would stop it opening at all.
- **No runtime CDN dependency** except the font. The file must render and animate with the network unavailable. No GSAP, no Lottie, no Tailwind, no jQuery.
- **Font:** `DM Sans` with a complete system fallback stack: `'DM Sans', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif`. Nothing else is fetched.
- **Theme:** dark, background `#0d1117`, matching `outbound-checklist.html`.
- **On-screen language: English.** All captions, titles, labels. `WBO` is never expanded.
- **Desktop only.** No responsive breakpoints, matching the app.
- **`viewBox="0 0 1600 900"`**, scaled to fit the window.
- **Total runtime 165 s.** Per-step durations live in one `DURATIONS` array and nothing else reads a hard-coded time.
- **Line endings: LF.** This repo's markdown and this new file are LF. After any tool-assisted write, verify with `tr -cd '\r' < <file> | wc -c` — it must print `0`.
- **Commits:** the user has **not** authorized commits in this session. Each task's commit step is written out, but ask before running it.

---

## File Structure

One file, `workflow-animation.html`, internally ordered as:

| Region | Responsibility |
|---|---|
| `<style>` | Page chrome, control bar, caption area, scrub bar. No animation lives here. |
| `<svg id="stage">` | Static scene (racks, lanes, zones) authored once; empty `<g>` hosts for actors, legend, app panel, document panel, elevation. |
| `/* ══ LAYOUT ══ */` | All geometry constants. Single source of coordinates. |
| `/* ══ PALETTE ══ */` | All colours, including both cone families. |
| `/* ══ BUILD ══ */` | Functions that construct actor nodes once at load. Never called per frame. |
| `/* ══ CLOCK ══ */` | `t`, `seek()`, `play()`, `pause()`, rAF loop, control wiring. |
| `/* ══ STEPS ══ */` | The `STEPS` registry: ten `{id, act, title, caption, reset, render}` entries. |
| `/* ══ TEST HOOK ══ */` | `window.__anim` — used by Playwright verification, harmless in production. |

Section-comment banners (`/* ══ NAME ══ */`) match the convention in `outbound-checklist.html`, where they are load-bearing for navigation.

---

## Verification strategy — read this before Task 1

This repo has no test runner and the deliverable is an animation, so "run the tests" is
not available. What *is* available is better than manual eyeballing, and it comes directly
from the spec's central decision:

**Because `render(t)` is a pure function of `t`, the scene at any instant is assertable.**

Each task therefore ends with executable checks driven through Playwright against a local
server, using an in-page test hook:

```js
/* ══ TEST HOOK ══ */
window.__anim = {
  seek,                        // (ms) => void, same function the scrub bar uses
  totalMs: () => TOTAL_MS,
  stepAt: (ms) => stepIndexAt(ms),
  state                        // () => serializable snapshot, defined in Task 2
};
```

Serve the repo and drive it:

```bash
cd "C:/Users/aspiezia/Downloads/packing checklist 0.1"
python -m http.server 8777
# then: http://localhost:8777/workflow-animation.html
```

`file://` is blocked by the Playwright MCP server, so verification always goes over
`http://localhost:8777`. Confirm the port is free first with
`netstat -ano | grep ':8777.*LISTENING'`, and kill the server when done.

Three checks recur and are referred to by name in later tasks:

- **DETERMINISM:** `seek(T)` twice yields identical `state()`.
- **SEEK-SAFETY:** `seek(T)`, then `seek(T2)` for some later `T2`, then `seek(T)` again
  yields the same `state()` as the first `seek(T)`. This is the one that catches stranded
  actors, and it is the reason `reset()` exists.
- **CLEAN CONSOLE:** zero console errors across the run.

---

### Task 1: Static scene — warehouse plan

**Files:**
- Create: `workflow-animation.html`

**Interfaces:**
- Consumes: nothing.
- Produces: `LAYOUT` (geometry constants), `PALETTE` (colours), and a DOM containing
  `#stage`, `#g-actors`, `#g-legend`, `#g-panel`, `#g-elevation`, all empty `<g>` hosts.

- [ ] **Step 1: Create the file with page chrome and an empty stage**

```html
<meta charset="UTF-8"/>
<title>Outbound Workflow — Sevenum</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700&display=swap">
<style>
:root{
  --bg:#0d1117; --surface:#161b22; --line:#30363d;
  --ink:#e6edf3; --muted:#8b949e; --accent:#58a6ff;
}
html,body{margin:0;height:100%;background:var(--bg);color:var(--ink);
  font-family:'DM Sans',-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif;}
#wrap{display:flex;flex-direction:column;height:100%;}
#stage{flex:1;width:100%;display:block;}
#bar{border-top:1px solid var(--line);background:var(--surface);padding:10px 16px;}
</style>
<div id="wrap">
  <svg id="stage" viewBox="0 0 1600 900" preserveAspectRatio="xMidYMid meet">
    <g id="g-static"></g>
    <g id="g-actors"></g>
    <g id="g-panel"></g>
    <g id="g-elevation"></g>
    <g id="g-legend"></g>
  </svg>
  <div id="bar"></div>
</div>
<script>
</script>
```

- [ ] **Step 2: Add the LAYOUT and PALETTE constants**

Inside `<script>`:

```js
/* ══ LAYOUT ══ */
const LAYOUT = {
  aisle:   { x0:40, y:70, h:140, groupW:105, gap:6, rackW:48, rackGap:3, count:10 },
  cross:   { y0:240, y1:360 },        // the lane the EPT travels
  drop:    { y:262 },                 // top edge of a pallet parked at an aisle mouth
  packing: { x:1190, y:230, w:230, h:200 },
  office:  { x:1440, y:230, w:120, h:110 },
  staging: { x:1190, y:470, w:330, h:350 },
  docks:   { x:1530, y:470, w:50,  h:350 },
  pallet:  { w:72, h:52 }
};

/* ══ PALETTE ══ */
const PALETTE = {
  floor:'#161b22', rack:'#1f2937', rackLine:'#30363d',
  zone:'#11161d',  zoneLine:'#30363d',
  ink:'#e6edf3',   muted:'#8b949e',
  vehicle:'#f5c518',                       // EPT and VNA are both yellow
  wood:'#8b5e34',  box:'#c9a227', label:'#f9fafb',
  strap:'#0b0e13', seal:'rgba(255,255,255,.14)',
  cone:  { blue:'#2563eb', yellow:'#eab308', red:'#dc2626', green:'#16a34a' },
  disc:  { black:'#111827', white:'#f9fafb', blue:'#2563eb', yellow:'#eab308',
           red:'#dc2626', purple:'#7c3aed', pink:'#ec4899', orange:'#f97316' }
};
```

- [ ] **Step 3: Generate the ten aisles, racks 54 down to 35**

Rack numbers descend left to right. Aisle *i* (0-based, left to right) holds racks
`54 - 2i` on the left and `53 - 2i` on the right.

```js
const SVGNS = 'http://www.w3.org/2000/svg';
function el(tag, attrs) {
  const n = document.createElementNS(SVGNS, tag);
  for (const k in attrs) n.setAttribute(k, attrs[k]);
  return n;
}
function text(x, y, str, opts = {}) {
  const n = el('text', { x, y, fill: opts.fill || PALETTE.muted,
    'font-size': opts.size || 12, 'text-anchor': opts.anchor || 'middle',
    'font-weight': opts.weight || 500 });
  n.textContent = str;
  return n;
}

function buildAisles(host) {
  const A = LAYOUT.aisle;
  for (let i = 0; i < A.count; i++) {
    const gx = A.x0 + i * (A.groupW + A.gap);
    const leftNo = 54 - 2 * i, rightNo = 53 - 2 * i;
    const inner = (A.groupW - (A.rackW * 2 + A.rackGap)) / 2;
    [[leftNo, gx + inner], [rightNo, gx + inner + A.rackW + A.rackGap]].forEach(([no, rx]) => {
      host.appendChild(el('rect', { x:rx, y:A.y, width:A.rackW, height:A.h,
        fill:PALETTE.rack, stroke:PALETTE.rackLine, rx:3 }));
      host.appendChild(text(rx + A.rackW / 2, A.y - 8, String(no), { size:13 }));
    });
  }
}
```

- [ ] **Step 4: Draw the cross aisle, packing, office, staging and docks**

```js
function zone(host, r, label) {
  host.appendChild(el('rect', { x:r.x, y:r.y, width:r.w, height:r.h,
    fill:PALETTE.zone, stroke:PALETTE.zoneLine, rx:6 }));
  host.appendChild(text(r.x + r.w / 2, r.y + 20, label, { size:13, fill:PALETTE.ink, weight:700 }));
}

function buildStatic() {
  const host = document.getElementById('g-static');
  host.appendChild(el('rect', { x:0, y:0, width:1600, height:900, fill:PALETTE.floor }));
  buildAisles(host);
  const C = LAYOUT.cross;
  host.appendChild(el('rect', { x:0, y:C.y0, width:1600, height:C.y1 - C.y0,
    fill:'#12171f', stroke:PALETTE.zoneLine }));
  host.appendChild(text(600, C.y1 - 10, 'CROSS AISLE', { size:11 }));
  zone(host, LAYOUT.packing, 'PACKING');
  zone(host, LAYOUT.office,  'OFFICE');
  zone(host, LAYOUT.staging, 'STAGING');
  zone(host, LAYOUT.docks,   'DOCKS');
}
buildStatic();
```

- [ ] **Step 5: Verify rack order and geometry in the browser**

Start the server, open `http://localhost:8777/workflow-animation.html`, then evaluate:

```js
() => [...document.querySelectorAll('#g-static text')]
        .map(t => t.textContent)
        .filter(s => /^\d+$/.test(s))
```

Expected, exactly: `["54","53","52","51","50","49","48","47","46","45","44","43","42","41","40","39","38","37","36","35"]`

This is the one thing that was wrong in the first sketch shown to the user, so it gets its
own assertion rather than a glance.

- [ ] **Step 6: Verify CLEAN CONSOLE, then commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: warehouse plan scene for outbound workflow animation"
```

---

### Task 2: Clock, controls, and the seekable step registry

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: `LAYOUT`, `PALETTE`, `el`, `text` from Task 1.
- Produces:
  - `DURATIONS: number[]` — ten step lengths in seconds.
  - `STEPS: {id:number, act:1|2, title:string, caption:string, start:number, end:number, reset:()=>void, render:(localT:number)=>void}[]`
  - `seek(ms:number):void`, `play():void`, `pause():void`
  - `state():object` — the snapshot every later task extends.
  - `window.__anim` test hook.

- [ ] **Step 1: Add durations, easing helpers and the step registry skeleton**

```js
/* ══ CLOCK ══ */
const DURATIONS = [18, 22, 14, 18, 28, 14, 14, 13, 11, 13];   // seconds, sums to 165
const TITLES = [
  'Pallets staged at the aisle',      'Weighing and pallet type',
  'Height, and the row order',        'Loading the order into the app',
  'Continuous scanning',              'WMS registration and printing',
  'Assembling the document packet',   'Strapping and sealing',
  'Delivery labels and packing list', 'Staging for tomorrow\u2019s truck'
];

const clamp = (v, a, b) => v < a ? a : v > b ? b : v;
const lerp  = (a, b, p) => a + (b - a) * p;
const ease  = p => p < .5 ? 2 * p * p : 1 - Math.pow(-2 * p + 2, 2) / 2;
// Progress of a sub-beat inside a step: 0 before `from`, 1 after `to`, eased between.
const beat  = (localT, from, to) => ease(clamp((localT - from) / (to - from), 0, 1));

let _acc = 0;
const STEPS = DURATIONS.map((secs, i) => {
  const start = _acc; _acc += secs * 1000;
  return { id:i + 1, act: i < 5 ? 1 : 2, title:TITLES[i], caption:'',
           start, end:_acc, reset(){}, render(){} };
});
const TOTAL_MS = _acc;
```

- [ ] **Step 2: Implement seek — the seek-safety contract**

`seek` is the whole reason the animation is scrubbable. It replays every prior step's
`reset()` so no actor can be left where a step never put it.

```js
let t = 0, playing = false, _raf = null, _last = 0;

function stepIndexAt(ms) {
  const c = clamp(ms, 0, TOTAL_MS);
  for (let i = 0; i < STEPS.length; i++) if (c < STEPS[i].end) return i;
  return STEPS.length - 1;
}

function seek(ms) {
  t = clamp(ms, 0, TOTAL_MS);
  const idx = stepIndexAt(t);
  // Rebuild scene state from scratch: every step up to and including the current one
  // re-asserts its starting conditions, then only the current one advances.
  for (let i = 0; i <= idx; i++) STEPS[i].reset();
  STEPS[idx].render(t - STEPS[idx].start);
  paintChrome(idx);
}
```

- [ ] **Step 3: Add the rAF loop and controls**

```js
function frame(now) {
  if (!playing) return;
  const dt = _last ? now - _last : 0;
  _last = now;
  seek(t + dt);
  if (t >= TOTAL_MS) { pause(); return; }
  _raf = requestAnimationFrame(frame);
}
function play()  { if (playing) return; playing = true; _last = 0; _raf = requestAnimationFrame(frame); }
function pause() { playing = false; if (_raf) cancelAnimationFrame(_raf); _raf = null; }
```

The control bar holds a play/pause button, a restart button, a caption line showing
`Act N · Step M — <title>`, and a scrub bar. The scrub bar is a `<div>` with ten tick
marks positioned at `STEPS[i].start / TOTAL_MS`, and a bracket spanning steps 1–5 labelled
`ACT I — FLOOR & VERIFICATION` and steps 6–10 labelled `ACT II — DOCUMENTS & DISPATCH`.
Clicking a tick calls `seek(STEPS[i].start)`. Clicking anywhere on the bar seeks
proportionally. `paintChrome(idx)` updates the caption text and the playhead position.

- [ ] **Step 4: Add the test hook and the state snapshot**

`state()` starts minimal and is extended by every later task. Keep it serializable —
Playwright has to return it across the bridge.

```js
/* ══ TEST HOOK ══ */
function state() {
  return {
    t: Math.round(t),
    step: stepIndexAt(t) + 1,
    playing
    // later tasks add: pallets, markers, vehicles, panel, elevation
  };
}
window.__anim = { seek, play, pause, state, totalMs: () => TOTAL_MS, stepAt: stepIndexAt };
```

- [ ] **Step 5: Verify the clock boundaries**

```js
() => {
  const a = window.__anim;
  return {
    total: a.totalMs(),                 // expect 165000
    atZero: a.stepAt(0) + 1,            // expect 1
    at17999: a.stepAt(17999) + 1,       // expect 1
    at18000: a.stepAt(18000) + 1,       // expect 2
    at164999: a.stepAt(164999) + 1,     // expect 10
    atEnd: a.stepAt(165000) + 1         // expect 10
  };
}
```

All six must match the expectations in the comments. The 17999/18000 pair is the real
check: an off-by-one in `stepIndexAt` makes the caption lag the scene by one step and is
invisible to the eye.

- [ ] **Step 6: Verify DETERMINISM and SEEK-SAFETY**

```js
() => {
  const a = window.__anim, J = JSON.stringify;
  a.seek(50000); const first = J(a.state());
  a.seek(50000); const again = J(a.state());
  a.seek(120000);
  a.seek(50000); const back  = J(a.state());
  return { deterministic: first === again, seekSafe: first === back };
}
```

Both must be `true`. Re-run this exact check at the end of every remaining task — it is
the regression test for the entire architecture.

- [ ] **Step 7: Commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: seekable clock, controls and step registry"
```

---

### Task 3: Actors — pallets, both cone families, vehicles, legend

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: `LAYOUT`, `PALETTE`, `el`, `text`, `state` from Tasks 1–2.
- Produces:
  - `buildPallet(id, opts) => {g, setPos(x,y), setMarker(visible), setStrap(on), setSeal(on), setLabel(on), setEnvelope(on), setPickLabel(on)}`
  - `buildCone(coneColour, discColour|null) => SVGGElement` — the two-family marker.
  - `buildVehicle(kind) => {g, setPos(x,y), setCarrying(bool), setWeight(kg|null)}` for
    `kind` of `'vna'` or `'ept'`.
  - `PALLETS` — five pallet handles: `A1, A2, A3` (Order A), `B1, B2` (Order B).
  - `state()` extended with `pallets` and `markers`.

- [ ] **Step 1: Build the marker — cone plus optional disc**

The two families must differ by **shape**, not only colour: a tall triangular cone against
a flat wide dome. At plan scale the disc is drawn wider than the cone's base is, which is
true to the real objects (20 cm across versus a 30 cm tall cone) and makes the stack read
as one two-colour piece.

```js
function buildCone(coneColour, discColour) {
  const g = el('g', {});
  // 30 cm cone: tall triangle plus base slab, with the side holes suggested by two dots.
  g.appendChild(el('polygon', { points:'0,-18 7,0 -7,0', fill:coneColour }));
  g.appendChild(el('rect', { x:-10, y:0, width:20, height:3, rx:1, fill:coneColour }));
  g.appendChild(el('circle', { cx:0, cy:-6, r:1.6, fill:'rgba(0,0,0,.45)' }));
  g.appendChild(el('circle', { cx:0, cy:-12, r:1.3, fill:'rgba(0,0,0,.45)' }));
  if (discColour) {
    // 6 cm disc resting on the cone's tip: flat, wide, unmistakably not a cone.
    g.appendChild(el('ellipse', { cx:0, cy:-20, rx:11, ry:3.4, fill:discColour,
      stroke:'rgba(0,0,0,.35)', 'stroke-width':.8 }));
  }
  return g;
}
```

- [ ] **Step 2: Build the pallet**

Every decoration is created once and toggled with the `hidden` attribute. Nothing is
created or destroyed during playback.

```js
function buildPallet(id, markerCone, markerDisc) {
  const P = LAYOUT.pallet;
  const g = el('g', { 'data-pallet':id });
  g.appendChild(el('rect', { x:0, y:0, width:P.w, height:P.h, rx:3, fill:PALETTE.wood }));
  for (let r = 0; r < 2; r++) for (let c = 0; c < 3; c++)
    g.appendChild(el('rect', { x:6 + c * 21, y:6 + r * 21, width:18, height:18, rx:2,
      fill:PALETTE.box, stroke:'rgba(0,0,0,.35)' }));

  const pickLabel = el('rect', { x:P.w - 20, y:4, width:16, height:11, rx:1.5, fill:PALETTE.label });
  const strap = el('g', {}); strap.setAttribute('hidden', '');
  [18, 46].forEach(x => strap.appendChild(el('rect', { x, y:0, width:4, height:P.h, fill:PALETTE.strap })));
  const seal = el('rect', { x:0, y:0, width:P.w, height:P.h, rx:3, fill:PALETTE.seal });
  seal.setAttribute('hidden', '');
  const marker = buildCone(markerCone, markerDisc);
  marker.setAttribute('transform', `translate(${P.w / 2}, 6)`);

  g.append(pickLabel, strap, seal, marker);

  const api = {
    g, id, x:0, y:0, marker,
    setPos(x, y) { api.x = x; api.y = y; g.setAttribute('transform', `translate(${x},${y})`); },
    setMarker(v) { marker.toggleAttribute('hidden', !v); },
    setPickLabel(v) { pickLabel.toggleAttribute('hidden', !v); },
    setStrap(v) { strap.toggleAttribute('hidden', !v); },
    setSeal(v) { seal.toggleAttribute('hidden', !v); }
  };
  return api;
}
```

- [ ] **Step 3: Instantiate the five pallets with the spec's markers**

Order A is a red cone **with a white disc stacked on it**. Order B is a plain blue cone.
Neither uses yellow, because yellow is also the vehicle colour.

```js
const PALLETS = {
  A1: buildPallet('A1', PALETTE.cone.red,  PALETTE.disc.white),
  A2: buildPallet('A2', PALETTE.cone.red,  PALETTE.disc.white),
  A3: buildPallet('A3', PALETTE.cone.red,  PALETTE.disc.white),
  B1: buildPallet('B1', PALETTE.cone.blue, null),
  B2: buildPallet('B2', PALETTE.cone.blue, null)
};
Object.values(PALLETS).forEach(p => document.getElementById('g-actors').appendChild(p.g));
```

- [ ] **Step 4: Build the two yellow vehicles**

```js
function buildVehicle(kind) {
  const g = el('g', { 'data-vehicle':kind });
  const w = kind === 'vna' ? 46 : 38, h = kind === 'vna' ? 30 : 22;
  g.appendChild(el('rect', { x:0, y:0, width:w, height:h, rx:4, fill:PALETTE.vehicle,
    stroke:'rgba(0,0,0,.4)' }));
  g.appendChild(text(w / 2, h / 2 + 4, kind.toUpperCase(), { size:10, fill:'#1a1a1a', weight:700 }));
  const readout = text(w / 2, -6, '', { size:11, fill:PALETTE.ink, weight:700 });
  g.appendChild(readout);
  const api = {
    g, x:0, y:0, weight:null,
    setPos(x, y) { api.x = x; api.y = y; g.setAttribute('transform', `translate(${x},${y})`); },
    setWeight(kg) { api.weight = kg; readout.textContent = kg == null ? '' : `${kg} kg`; }
  };
  return api;
}
const VNA = buildVehicle('vna'), EPT = buildVehicle('ept');
```

- [ ] **Step 5: Build the legend showing both families complete**

The legend documents the real system, not just what is on camera: all four 30 cm colours
and all eight 6 cm colours, each labelled with its family and size.

```js
function buildLegend() {
  const host = document.getElementById('g-legend');
  host.appendChild(text(40, 840, '30 cm CONES', { size:11, anchor:'start', fill:PALETTE.muted, weight:700 }));
  Object.entries(PALETTE.cone).forEach(([name, c], i) => {
    const k = buildCone(c, null);
    k.setAttribute('transform', `translate(${60 + i * 74}, 878)`);
    host.appendChild(k);
    host.appendChild(text(60 + i * 74, 893, name, { size:9 }));
  });
  host.appendChild(text(400, 840, '6 cm DISCS', { size:11, anchor:'start', fill:PALETTE.muted, weight:700 }));
  Object.entries(PALETTE.disc).forEach(([name, c], i) => {
    host.appendChild(el('ellipse', { cx:420 + i * 74, cy:876, rx:11, ry:3.4, fill:c,
      stroke:'rgba(255,255,255,.25)', 'stroke-width':.8 }));
    host.appendChild(text(420 + i * 74, 893, name, { size:9 }));
  });
}
buildLegend();
```

- [ ] **Step 6: Extend `state()`**

```js
// inside state(), add:
pallets: Object.fromEntries(Object.values(PALLETS).map(p =>
  [p.id, { x:Math.round(p.x), y:Math.round(p.y), marker:!p.marker.hasAttribute('hidden') }])),
vehicles: { vna:{ x:Math.round(VNA.x), y:Math.round(VNA.y) },
            ept:{ x:Math.round(EPT.x), y:Math.round(EPT.y), weight:EPT.weight } }
```

- [ ] **Step 7: Verify the marker shapes are distinguishable**

```js
() => {
  const a1 = document.querySelector('[data-pallet="A1"]');
  const b1 = document.querySelector('[data-pallet="B1"]');
  return {
    A1_hasCone:  !!a1.querySelector('polygon'),
    A1_hasDisc:  !!a1.querySelector('ellipse'),   // expect true  — red cone + white disc
    B1_hasCone:  !!b1.querySelector('polygon'),
    B1_hasDisc:  !!b1.querySelector('ellipse'),   // expect false — plain blue cone
    legendCones: document.querySelectorAll('#g-legend polygon').length,  // expect 4
    legendDiscs: document.querySelectorAll('#g-legend ellipse').length   // expect 8
  };
}
```

Expected: `A1_hasCone` true, `A1_hasDisc` true, `B1_hasCone` true, `B1_hasDisc` **false**,
4 legend cones, 8 legend discs. The `B1_hasDisc: false` assertion is what stops the two
orders quietly becoming the same marker.

- [ ] **Step 8: Screenshot and commit (ask first)**

Take a screenshot and confirm by eye that the cone reads as tall and the disc as flat.
Shape distinctness is the one property here a DOM assertion cannot judge.

```bash
git add workflow-animation.html
git commit -m "feat: pallets, two cone families, vehicles and legend"
```

---

### Task 4: Act I steps 1–3 — the floor

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: everything from Tasks 1–3.
- Produces: `STEPS[0..2].reset/render` implemented; `DROP_X: number[]` (aisle-mouth x
  positions); `writeOn(handle, field, value)` helper for the hand-written pick-label fields.

- [ ] **Step 1: Compute the aisle-mouth drop positions**

```js
const A = LAYOUT.aisle;
const aisleCentre = i => A.x0 + i * (A.groupW + A.gap) + A.groupW / 2;
// Order A takes aisles 1, 3, 5; Order B takes aisles 7, 8. Spread so the EPT's route
// visibly passes Order B's pallets on its way to packing.
const DROP = {
  A1:{ ai:1 }, A2:{ ai:3 }, A3:{ ai:5 }, B1:{ ai:7 }, B2:{ ai:8 }
};
Object.values(DROP).forEach(d => { d.x = aisleCentre(d.ai) - LAYOUT.pallet.w / 2; d.y = LAYOUT.drop.y; });
```

- [ ] **Step 2: Step 1 — the VNA places five pallets**

`reset()` puts the world in its step-1 opening state. `render(localT)` drives five drop
beats plus the VNA's travel between them.

```js
STEPS[0].caption = 'The VNA brings each pallet out to the mouth of its aisle and sets it '
  + 'down with its pick label and its order marker on top.';
STEPS[0].reset = () => {
  Object.values(PALLETS).forEach(p => { p.setPos(-200, -200); p.setMarker(true);
    p.setPickLabel(true); p.setStrap(false); p.setSeal(false); });
  VNA.setPos(-200, -200); EPT.setPos(-200, -200); EPT.setWeight(null);
  showPanel(false); showElevation(false);
};
STEPS[0].render = (lt) => {
  const order = ['A1','A2','A3','B1','B2'];
  order.forEach((id, i) => {
    const from = i * 3200, to = from + 2600;
    const p = beat(lt, from, to);
    const d = DROP[id];
    if (p <= 0) return;
    // Pallet rides out of the aisle (from inside, above the mouth) down to its drop spot.
    PALLETS[id].setPos(d.x, lerp(LAYOUT.aisle.y + 40, d.y, p));
    if (p < 1) VNA.setPos(d.x - 4, lerp(LAYOUT.aisle.y + 40, d.y, p) - 34);
    else if (i === order.length - 1) VNA.setPos(-200, -200);
  });
};
```

- [ ] **Step 3: Step 2 — the EPT collects only Order A**

This step carries the idea the spec calls the hardest: the marker decides what gets
collected. The EPT drives the full length of the cross aisle, picks up the three red-coned
pallets, and passes the two blue-coned ones without stopping.

`render(localT)` runs three collect beats. Each beat: EPT travels to `DROP[id].x`, the
weight appears on its readout, `writeOn` reveals the weight and pallet type on the pick
label, then EPT and pallet travel together to the packing zone and the pallet parks in a
row inside it. Order B's pallets are never touched and stay at their drop positions for
the rest of the video.

```js
const PACK_SLOT = i => ({ x: LAYOUT.packing.x + 16, y: LAYOUT.packing.y + 40 + i * 56 });
const WEIGHTS = { A1:420, A2:365, A3:510 };
const PTYPES  = { A1:'EP', A2:'EP', A3:'BP' };
```

`writeOn(pallet, field, value)` appends a small `<text>` onto the pallet's pick label the
first time it is called for that field, and is idempotent — calling it again with the same
value is a no-op. Idempotence is required: `render` runs every frame and on every seek.

- [ ] **Step 4: Step 3 — height, then row numbering**

A tape graphic extends up each pallet at packing, the height is written onto the pick
label via `writeOn`, and then a large `1`, `2`, `3` badge fades onto the three pallets in
row order. Heights: `A1 165`, `A2 150`, `A3 172` cm — all under the Europe limit of 180,
so nothing here contradicts the app's `HEIGHT_LIMITS`.

- [ ] **Step 5: Verify the collection rule**

The single most important assertion in Act I: at the end of step 2, Order A is at packing
and Order B has not moved.

```js
() => {
  const a = window.__anim;
  a.seek(39500);                       // near the end of step 2
  const s = a.state();
  const inPacking = p => p.x > 1150;
  return {
    A1:inPacking(s.pallets.A1), A2:inPacking(s.pallets.A2), A3:inPacking(s.pallets.A3),
    B1:inPacking(s.pallets.B1), B2:inPacking(s.pallets.B2)
  };
}
```

Expected: `A1/A2/A3` all `true`, `B1/B2` both `false`.

- [ ] **Step 6: Re-run DETERMINISM and SEEK-SAFETY, screenshot steps 1–3, commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: act I floor steps - staging, weighing, height and row order"
```

---

### Task 5: Act I steps 4–5 — the app panel and continuous scanning

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: Tasks 1–4.
- Produces: `showPanel(visible, kind)` where `kind` is `'app'` or `'docs'`;
  `buildAppPanel()`; `state()` extended with `panel`.

- [ ] **Step 1: Build the app panel**

A simplified rendering of `outbound-checklist.html` inside `#g-panel`: a dark card with a
delivery header (operator, delivery no., destination, part number), three pallet tabs, and
for the active tab a target/verified readout plus a progress bar. It slides in from the
right over the plan, it does not replace it.

- [ ] **Step 2: Step 4 — loading the order**

Fields fill in sequence: delivery no. `4500012345`, destination `Europe 180 cm`, part
number `ASD123`, then per pallet the type, weight and height already written on the pick
labels in steps 2–3, then the pick quantities. Targets: `P1 250`, `P2 180`, `P3 300`.

Every value shown here must equal the value written onto that pallet's pick label earlier.
Define them **once** in the `WEIGHTS`, `PTYPES`, `HEIGHTS` and `TARGETS` constants from
Task 4 and read them in both places — never retype a number into the panel.

- [ ] **Step 3: Step 5 — the continuous scan and the over-target stop**

Beats, in order:

1. Scanner moves box to box on pallet 1; the verified count climbs 0 → 250. Progress bar
   goes green at target.
2. **The app switches to tab 2 by itself.** Hold a caption on this: it is the behaviour
   the whole feature exists for.
3. Pallet 2 scans to 180. Auto-jump to tab 3.
4. Pallet 3 climbs to 250 of 300, then a scan of 65 takes it to 315 — over target.
5. The stop fires: bar turns red, and the app's real toast appears.

The toast must match `triggerOverTargetStop` in `outbound-checklist.html` exactly:

```
⚠ OVER TARGET
Pallet 3 · 315 / 300 pcs (+15)
```

Verify this string against the live app before hardcoding it — the app is the source of
truth, and a training video that shows a message the software does not produce is worse
than showing no message.

- [ ] **Step 4: Verify the counts and the toast**

```js
() => {
  const a = window.__anim;
  const at = ms => { a.seek(ms); return a.state().panel; };
  return { midP1:at(76000), afterP1:at(83000), overP3:at(99000) };
}
```

Assert: at `76000` the active tab is 1 and verified is between 0 and 250; at `83000` the
active tab has advanced to 2 without any click; at `99000` the active tab is 3, verified is
315, target is 300, and `toast` is the exact two-line string above.

- [ ] **Step 5: Re-run DETERMINISM and SEEK-SAFETY, screenshot, commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: app panel, continuous scanning and over-target stop"
```

---

### Task 6: Act II steps 6–7 — WMS, printing, document packet

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: Tasks 1–5.
- Produces: `buildDocsPanel()`; `DOC_ORDER: string[]`; `state()` extended with `docs`.

- [ ] **Step 1: Step 6 — WMS registration and printing**

The panel switches from the app view to a WMS view. A transaction number is typed against
each of the three pick labels. A printer then emits, in view: **three** delivery labels
(one per pallet), **one** packing list, and **one** WBO. The counts matter — one packing
list for the whole order, not one per pallet.

- [ ] **Step 2: Step 7 — the clipped packet, in the specified order**

```js
// The clip order is fixed by the process and must not be reordered for visual balance.
const DOC_ORDER = ['Pick labels (3)', 'Outbound checklist print', 'Packing list copy', 'WBO'];
```

The four items stack top to bottom in exactly this order, a metal clip animates onto the
top edge, and the packet then travels to the `OFFICE` zone — its destination, confirmed by
the user.

- [ ] **Step 3: Verify the stack order and destination**

```js
() => {
  const a = window.__anim;
  a.seek(127000);
  const s = a.state();
  return { order:s.docs.stack, destination:s.docs.destination, clipped:s.docs.clipped };
}
```

Expected: `order` deep-equals `DOC_ORDER`, `destination` is `'office'`, `clipped` is
`true`.

- [ ] **Step 4: Re-run DETERMINISM and SEEK-SAFETY, screenshot, commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: WMS registration, printing and document packet"
```

---

### Task 7: Act II steps 8–10 — strap, seal, labels in elevation, staging

**Files:**
- Modify: `workflow-animation.html`

**Interfaces:**
- Consumes: Tasks 1–6.
- Produces: `showElevation(visible)`; `buildElevation()`; `state()` extended with
  `elevation`.

- [ ] **Step 1: Step 8 — strapping then sealing, in that order**

For each of the three pallets at packing: `setStrap(true)` with the black bands wrapping
on, then `setSeal(true)` with the transparent film fading over the whole pallet. Strap
always precedes seal, and both precede any label — step 9 sticks labels **on top of** the
seal, which is only true if this ordering holds.

- [ ] **Step 2: Step 9 — cut to the front elevation**

This is the only step that leaves the plan view, and the spec explains why: from above, a
label on the front face and a label on the side are the same picture, so a plan view would
teach the wrong placement.

`buildElevation()` draws the three pallets face-on in `#g-elevation`, short faces toward
the viewer, hidden outside step 9. Each is a pallet base with a stack of boxes, the black
straps as vertical bands, and the seal as a translucent overlay.

```js
function showElevation(v) {
  document.getElementById('g-elevation').toggleAttribute('hidden', !v);
  document.getElementById('g-static').toggleAttribute('hidden', v);
  document.getElementById('g-actors').toggleAttribute('hidden', v);
}
```

Then: a delivery label fades onto the short front face of **each** pallet, and the packing
list envelope fades onto **pallet 1 only**, beside its label on that same face.

- [ ] **Step 3: Step 10 — back to plan, move to staging**

`showElevation(false)` restores the plan. The three pallets travel from packing to the
`STAGING` zone and line up in row order next to the docks. A closing caption states that
loading happens the next day.

- [ ] **Step 4: Verify the envelope lands on pallet 1 and nowhere else**

```js
() => {
  const a = window.__anim;
  a.seek(150000);                      // late in step 9
  const s = a.state();
  return { visible:s.elevation.visible, labels:s.elevation.labels,
           envelopes:s.elevation.envelopes };
}
```

Expected: `visible` true, `labels` is `['A1','A2','A3']` (all three), `envelopes` is
`['A1']` — exactly one. An envelope on more than one pallet is a process error the video
would teach as correct.

- [ ] **Step 5: Verify the elevation hides again on the way out**

```js
() => {
  const a = window.__anim;
  a.seek(150000); const during = a.state().elevation.visible;   // expect true
  a.seek(160000); const after  = a.state().elevation.visible;   // expect false
  a.seek(30000);  const before = a.state().elevation.visible;   // expect false
  return { during, after, before };
}
```

A stranded elevation group is exactly the failure mode `reset()` exists to prevent, so it
gets its own check in both directions.

- [ ] **Step 6: Re-run DETERMINISM and SEEK-SAFETY, screenshot, commit (ask first)**

```bash
git add workflow-animation.html
git commit -m "feat: strapping, sealing, front-elevation labelling and staging"
```

---

### Task 8: Full-run verification and pacing pass

**Files:**
- Modify: `workflow-animation.html`

- [ ] **Step 1: Capture one frame per step**

Seek to the midpoint of each of the ten steps, screenshot, and confirm against the spec's
step table that each frame shows what that step claims to show.

First get the ten midpoints in one evaluate:

```js
() => {
  const D = [18,22,14,18,28,14,14,13,11,13];
  let acc = 0;
  return D.map(s => { const mid = acc + s * 500; acc += s * 1000; return mid; });
}
```

Expected: `[9000, 29000, 47000, 63000, 86000, 107000, 121000, 134500, 146500, 158500]`.

Then, for each midpoint, evaluate `window.__anim.seek(<mid>)` and take a screenshot. Ten
seeks, ten screenshots, each checked against its row in the spec's step table.

- [ ] **Step 2: Backward scrub over the whole timeline**

```js
() => {
  const a = window.__anim, J = JSON.stringify;
  const marks = [5000, 30000, 60000, 90000, 110000, 130000, 150000, 164000];
  const forward = marks.map(m => { a.seek(m); return J(a.state()); });
  const backward = [...marks].reverse().map(m => { a.seek(m); return J(a.state()); }).reverse();
  return { identical: J(forward) === J(backward) };
}
```

Expected `identical: true`. This is the full-timeline version of SEEK-SAFETY and is the
single check most likely to catch a missing `reset()`.

- [ ] **Step 3: Offline check**

With DevTools network throttling set to Offline, hard-reload and confirm the scene renders
and animates. Only the font may degrade. If anything else breaks, a CDN dependency crept
in and must be inlined.

- [ ] **Step 4: CLEAN CONSOLE over a full play-through**

Call `play()`, let it run to the end, then read console errors. Expect zero.

- [ ] **Step 5: Pace review**

Watch it once end to end. If any step feels rushed or dead, adjust **only** the
`DURATIONS` array and re-run Step 2 of this task. No animation code should need touching —
if it does, a beat is reading a hard-coded time and that is a bug to fix.

- [ ] **Step 6: Confirm LF line endings, then commit (ask first)**

```bash
tr -cd '\r' < workflow-animation.html | wc -c    # must print 0
git add workflow-animation.html
git commit -m "feat: verification pass and pacing for workflow animation"
```

---

## Self-review notes

**Spec coverage.** Every spec section maps to a task: geometry and zones → Task 1; clock,
acts, pacing → Task 2; cone families, legend, vehicles → Task 3; steps 1–3 → Task 4;
steps 4–5 including the over-target stop → Task 5; steps 6–7 → Task 6; steps 8–10
including the elevation → Task 7; the spec's six verification points → Task 8, plus the
per-task checks.

**Placeholder scan.** Clean. Task 8 Step 1 originally carried an illustrative, non-runnable
code block; it has been replaced with a real midpoint computation and its expected output.

**Type consistency.** `showPanel(visible, kind)` is introduced in Task 5 but called from
`STEPS[0].reset` in Task 4. Define a no-op `showPanel`/`showElevation` pair in Task 3
alongside the other builders so Task 4 has something to call, and give them real bodies in
Tasks 5 and 7.
