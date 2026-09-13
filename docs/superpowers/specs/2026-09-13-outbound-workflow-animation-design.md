# Outbound workflow animation — design

## Context

There is no shared picture of what the Sevenum outbound process actually is. The
`outbound-checklist.html` app covers one slice of it (steps 4–5 below) and the
`outbound-checklist-launch.html` presentation explains that app, but nothing shows the
whole chain: how a pallet gets from a rack face to a truck, and where the app sits in
that chain.

This spec covers a browser-viewable animation of the complete 10-step flow, built as a
single self-contained HTML file. It is training and reference material. It is **not** an
MP4 and it does not read live data from the app.

## Deliverable

`workflow-animation.html` at the repo root. Opens by double-click in Chrome. No build
step, no `node_modules`, no framework — same constraints as the rest of this repo.

### Non-goals

- Not a video file. No MP4/WebM export.
- Not interactive training (no quizzes, no branching, no score).
- Does not read `localStorage`, the app's state, or any real delivery.
- Not responsive below desktop width. The app itself declares "Desktop only — no
  responsive breakpoints"; this follows the same rule. Target is a desktop browser or a
  wall screen.

## Warehouse geometry

Confirmed by the user, 2026-09-13.

20 racks numbered **35–54**, forming **10 Very Narrow Aisles** of two facing racks each.
Viewed from the front, the lower number of a pair is on the right, and aisles run left to
right in descending order:

```
 54│53   52│51   50│49   48│47   46│45   44│43   42│41   40│39   38│37   36│35
 ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓   ▓▓│▓▓
 ══════════════════════ cross aisle ══════════════════════
    ▣R  ▣R  ▣R        ▣B  ▣B          ──►  PACKING  ──►  STAGING ──► DOCKS
```

Left to right this reads 54, 53, 52, 51 … 36, 35.

Fixed areas on the plan:

- **Aisle mouths** — where the VNA sets pallets down, in the cross aisle. A VNA aisle is
  too narrow for an EPT to enter, so pallets are always deposited at the mouth.
- **Cross aisle** — the horizontal lane the EPT travels.
- **Packing station** — right-hand end of the cross aisle. Weighing, measuring, the app,
  printing, strapping and sealing all happen here.
- **Staging zone** — right of packing, adjacent to the loading docks. Pallets wait here
  overnight.
- **Docks** — along the far right edge. Trucks load the next day.
- **Office** — small area off the packing station; destination of the document packet.

### Confirmed process facts

Both explicitly confirmed by the user, 2026-09-13. Neither is an open assumption.

1. **The VNA deposits at the aisle mouth, not inside the aisle.** A VNA aisle is too
   narrow for the EPT to enter, so the handover between the two vehicles has to happen in
   the cross aisle. This is why step 1 and step 2 are separate steps at all.
2. **Strapping and sealing (step 8) happen at the packing station**, not at staging and
   not at the aisle.

No open assumptions remain in this spec.

## Actors and colour language

| Element | Treatment |
|---|---|
| VNA truck | Yellow. Moves in and out of aisle mouths, carries the pallet elevated. |
| EPT (electric pallet truck) | Yellow. Travels the cross aisle at floor level, has a weight readout. |
| Order A marker | Red 30 cm cone + white 6 cm disc stacked on its tip. The order the video follows. 3 pallets. |
| Order B marker | Blue 30 cm cone, no disc. Decoy order, 2 pallets, never collected on camera. |

### Cones — two families, combinable

Confirmed by the user, 2026-09-13, with product photos. This is not one set of coloured
cones but **two different objects**, and the distinction has to survive into the drawing.

| Family | Object | Colours |
|---|---|---|
| Large | 30 cm training cone, traffic-cone shape with holes in the sides | blue, yellow, red, green (4) |
| Small | 6 cm marker disc, flat dome, 20 cm across | black, white, blue, yellow, red, purple, pink, orange (8) |

Either family can mark an order on its own, **or the two combine: the 6 cm disc sits on
top of the 30 cm cone's tip**, reading as one two-colour piece.

That combination is the point of the whole system and the video must demonstrate it, not
just mention it. Four large colours alone would run out the moment five orders share the
aisle; 4 large × 8 small plus the singles is a vocabulary big enough for a real day's
work. A viewer who takes away "each order gets a colour" has learned the wrong rule — the
rule is "each order gets a *marker*, which may be one piece or two stacked".

So the two orders on camera deliberately use one of each mode:

- **Order A** — the order the video follows — **red 30 cm cone with a white 6 cm disc on
  top**. The combination gets the most screen time because it is the harder idea.
- **Order B** — the decoy — **blue 30 cm cone, no disc**.

The legend shows both families complete: all four large colours and all eight small ones.

One collision to preserve through any redesign: **yellow appears in both cone families and
is also the colour of the EPT and the VNA.** A yellow cone beside a yellow truck is the
one pairing a viewer can genuinely misread, which is why neither on-camera order uses
yellow. The legend's yellow chips stay yellow regardless — the colour is real and changing
it would misrepresent the system. If a chip proves confusing, give it an outline.
| Pallet | Wooden base, stacked boxes, a pick label sheet on top. |
| Strap | Black bands. |
| Seal | Transparent wrap, drawn as a light translucent overlay. |
| Delivery label | Small white adhesive rectangle. One per pallet, on the pallet's **short front face**, stuck on top of the transparent seal. |
| Packing list envelope | Adhesive envelope. First pallet of the row only, on that same short front face, alongside its delivery label. |

Order B exists for one reason: it is what makes the cone's purpose legible. In step 2 the
EPT collects the red pallets and drives past the blue ones. With only one order on the
floor, the cone is decoration that the narration has to explain instead of demonstrate.

## Timeline structure

One clock, one scrub bar, ten markers, grouped into two acts.

- **Act I — Floor & Verification** (steps 1–5)
- **Act II — Documents & Dispatch** (steps 6–10)

Acts are a visual grouping on the scrub bar and a heading in the caption area. They are
not separate files and not separate timelines.

### Steps

| # | Act | On screen |
|---|---|---|
| 1 | I | VNA brings out 3 pallets of Order A to aisle mouths, plus 2 of Order B. Each pallet carries a pick label and its order's marker on top of the boxes — Order A's red cone with the white disc stacked on top, Order B's plain blue cone. The legend of both cone families is introduced here. |
| 2 | I | EPT collects each red pallet, weight appears on its readout, weight and pallet type are hand-written onto the pick label, pallet is hauled to packing. Blue pallets are passed over and stay put. |
| 3 | I | Height is measured with a tape, written onto each pick label, and the pallets are numbered 1-2-3 in row order. |
| 4 | I | Pick labels are collected in row order. App panel slides in: delivery no., destination, PN scan, then per pallet type / weight / height / pick quantities. |
| 5 | I | Continuous scanning. Scanner moves box to box across pallet 1; on reaching target the app jumps to pallet 2 by itself; on pallet 3 a scan exceeds the target and the real over-target stop fires. Then the report is printed. |
| 6 | II | WMS: the transaction number of each pick label is registered. Printer emits one delivery label per pallet, one packing list for the order, and the WBO. |
| 7 | II | Documents are stacked in order and clipped: pick labels, then the outbound checklist print, then the packing list copy, then the WBO. The clipped packet goes to the office. |
| 8 | II | Other workers strap each pallet with black plastic, then seal with transparent wrap. |
| 9 | II | **Elevation view** (see below). A delivery label is stuck on the short front face of every pallet, over the seal. On pallet 1 only, an adhesive envelope holding the packing list goes on that same front face beside the label. |
| 10 | II | Pallets are moved to the staging zone by the docks and lined up. A caption states loading happens the next day. |

Step 5 is the payoff of Act I and must show the over-target stop, since that behaviour now
exists in the app (implemented 2026-09-13). The toast wording on screen must match the
app's real toast: `⚠ OVER TARGET` above `Pallet 3 · <verified> / <target> pcs (+<over>)`.

### Pacing

Starting durations, in seconds. These live in one `DURATIONS` table and are the only knob
for re-pacing the piece; no animation code reads a hard-coded time.

| Step | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Seconds | 18 | 22 | 14 | 18 | 28 | 14 | 14 | 13 | 11 | 13 | **165** (2:45) |

Steps 2 and 5 get the most time because they carry the two ideas a viewer is most likely
to get wrong: that the cone colour decides which pallets get collected, and that scanning
runs continuously across pallets until something stops it.

### On-screen language

**English**, matching the app. Narration captions, step titles and all labels are English.
`WBO` is shown as-is, not expanded.

## Technical architecture

### No external dependencies at runtime

Everything is inline: SVG shapes, CSS, JS. The animation must render correctly with the
network unavailable, because it will be opened on warehouse machines.

The one concession is the font: `DM Sans` is referenced with a complete system fallback
stack, so it matches the app's typography when online and degrades to a system face
offline without the layout collapsing. Nothing else is fetched. No GSAP, no Lottie, no
Tailwind — a CDN dependency here means a training video that one day does not play.

### Deterministic, seekable clock

This is the central design decision, and it dictates everything else.

A single `requestAnimationFrame` loop advances one variable, `t`, in milliseconds. Every
moving element's position is a **pure function of `t`** — not a chain of CSS transitions
or queued animations. Consequences:

- Dragging the scrub bar to any point renders the correct scene at that point.
- Dragging **backwards** works. A chained-animation approach would leave actors stranded
  wherever their last transition happened to end.
- Rendering is idempotent: calling `render(t)` twice with the same `t` produces the same
  frame and no accumulated state.

Structure:

```
STEPS = [ { id, act, title, caption, start, end, reset(), render(localT) }, … ]
```

- `render(localT)` sets attributes on **persistent** SVG nodes. It never creates or
  destroys nodes per frame — that would leak and make scrubbing expensive.
- `reset()` establishes the scene state at that step's start. It is called on **any**
  seek, for the target step and every step before it, so a backward scrub cannot leave a
  pallet in a position that step never produced.

### Scene

A single SVG with `viewBox="0 0 1600 900"`, scaled to fit the window. Dark theme on
`#0d1117` to match the app.

Steps 1–3, 8 and 10 are pure top-down plan. Steps 4–7 slide in a side panel showing the
app screen or the documents, because at those moments the thing that matters is on the
computer or on paper, not on the floor.

**Step 9 is the exception and needs its own view.** Labels and the envelope go on the
pallet's short front face, and a top-down plan cannot show a vertical face at all — from
above, a label on the front and a label on the side look identical. Drawing step 9 in plan
would teach the wrong placement, which is the one thing a training video must not do. So
step 9 cuts to a **front elevation** of the three pallets standing side by side, short
faces toward the viewer, and the labels land on those faces. The plan view returns for
step 10.

This is a view change, not a second scene graph: the elevation is a separate `<g>` in the
same SVG, hidden outside step 9, and it obeys the same `reset()` / `render(localT)`
contract as everything else.

### Controls

Play/pause, restart, a scrub bar with the two acts marked and ten step ticks, and a
caption line naming the current step. Clicking a tick seeks to that step.

## Verification

Manual, in-browser — consistent with the rest of this repo, which has no test runner.

1. Open in Chrome; capture a frame from each of the ten steps and confirm each shows what
   the step table says it shows.
2. Step 1 specifically: confirm the 30 cm cone and the 6 cm disc are distinguishable by
   **shape**, not only by colour — a tall cone against a flat dome — and that Order A's
   stacked marker reads as one two-piece object rather than two unrelated things.
3. Step 9 specifically: confirm the frame is a front elevation, that every pallet carries
   a delivery label on its short front face, and that the packing list envelope appears on
   pallet 1 and on no other pallet. (The short face is the 80 cm side of a 120×80 EP, and
   the correspondingly narrower side of the other `PTYPE` sizes.)
4. Scrub backwards from step 10 to step 1 and confirm no actor is stranded — specifically
   that the pallets return to the aisle mouths and the markers reappear, and that the step
   9 elevation is hidden again once the playhead leaves it.
5. Load with the network blocked (DevTools offline) and confirm the scene renders and
   animates.
6. Zero console errors across a full play-through.

## Risks

- **Yellow reads as "vehicle".** Yellow is both a real cone colour and the colour of the
  EPT and VNA. The video avoids the collision by following the red and blue orders, but
  the legend still shows a yellow cone chip. If that chip proves confusing on screen it
  needs an outline, not a different colour — the colour is real and changing it would
  misrepresent the system.
- **Ten steps is a long piece.** Past roughly three minutes it stops being watchable in a
  briefing. Mitigation: per-step durations are constants in one table, so the whole piece
  can be re-paced without touching any animation code.
