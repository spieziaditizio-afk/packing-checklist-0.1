# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file (~2200 lines) HTML web app for outbound checklist verification at Arrow Electronics' Sevenum NL warehouse — pre-shipment prep of pallets before an order ships. The operator scans/types delivery info, pallets and per-box quantities, then exports a print/PDF/CSV report.

`outbound-checklist.html` is the whole app (renamed from `packing-check-list.html`; internal `localStorage` keys still use the legacy `arrowPacking*` prefix — see Constants). There is no build step, no `node_modules`, no framework. Opens directly in Chrome (double-click or `start outbound-checklist.html`). External deps load via CDN at runtime:

- jsPDF + jspdf-autotable (PDF export)
- Google Fonts (DM Sans / DM Mono)

`Resumen de la app cycle count.txt` is historical notes about a separate app (cycle-count) that previously lived in this repo. Not authoritative; the cycle-count code is gone.

## Workflow

- **Run**: open the HTML file in Chrome. No server needed.
- **Edit**: change the file, reload the browser (Ctrl+R). State persists in `localStorage` keyed by the active session (URL hash `#s=…`). A bare reload (no `#s=` in the address bar) always assigns a fresh blank session rather than resuming the last one — append `#s=default` (or whichever id you were using) to resume in-progress state instead. Clear all sessions via DevTools or remove keys matching `arrowPackingV1:*`.
- **Production scanner**: Zebra **DS3678** (cordless rugged 1D/2D imager, connects via Bluetooth cradle in HID Keyboard Wedge mode). Suffix is configurable via 123Scan / programming barcodes — the app accepts both Enter (CR) and Tab so it's tolerant of any unit's configuration without having to touch the hardware. Other scannable inputs (operator name, delivery#) inherit the same handler.
- **Test the scanner pipeline without a Zebra**: just type and press Enter — the app no longer distinguishes typed vs scanned (the timing heuristic produced too many false positives in field testing). Every valid Enter (or Tab) gets a green flash + beep, period. Test the wrong-scan branch by typing letters into a Qty field; test the wrong-PN branch by entering a PN that doesn't match the delivery PN.
- **Test multi-session**: append `#s=<anything>` to the URL, or click 🆕 "New Order" (topbar) / "+ New Order" (📑 dropdown) — both flush-save the current session and jump to a fresh blank one in the same tab. Each session is independent; opening two browser windows on the same file with different hashes works side-by-side.
- **No tests, no lint, no CI.** Verification is manual in-browser.

## Architecture inside the single file

```
Lines 1–447    : <style> block (all CSS, dark theme #0d1117)
Lines 448–2200 : <script> block, organized by section comments /* ══ NAME ══ */
```

### Domain model

A **delivery** has: operator, delivery#, Pick Label PN, destination (Europe ≤180cm / Export ≤160cm).

Each **pallet** has: type (`KP|EP|BP|GP|NP` → dimensions via `PTYPE` map), weight, height, plus two sub-sections:

1. **Pick Labels** = target quantities. Sum of `Pick 1`, `Pick 2`… rows = target pieces for the pallet. Recently renamed from "PL" / "Expected".
2. **Box Verification** = actual verified quantities. Operator picks a mode in the section header:
   - **Scan mode** with sub-toggle `Shared PN` (one PN field + per-box qty rows) vs `PN per box` (PN+qty pair per row).
   - **Visual mode** = quantity groups (N boxes × M pcs each). For fast entry of many small uniform boxes — no per-box scan.

The validation strip at the bottom of each pallet aggregates: `Pick PN | Box PN | PN Check | Target (pcs) | Verified (pcs) | Boxes` + a status badge.

**Empty rows never count.** Pick rows, box rows and per-box rows all guard with `if (qty > 0)` before incrementing. If you add a new counting path, follow the same pattern in **all five aggregators** — they read live DOM independently and must stay in sync:
1. `updatePalletSummary` — per-pallet badge + validation strip
2. `updateDeliverySummary` — sidebar Live Summary (delivery total)
3. `buildPrintHTML` — browser print preview
4. `doExportPDF` — jsPDF + autoTable
5. `buildExportData` — CSV exporter

### Central input pipeline

Every scannable `<input>` wires `onkeydown="onKeyDown(event, this)"` and `oninput="handleQtyInput(...)"` (or PN equivalent). The flow on **Enter or Tab** (the Zebra DS3678 used at the packing stations can be configured to emit either CR or Tab as its keystroke suffix via 123Scan; intercepting both lets us tolerate either configuration without touching the hardware. `Shift+Tab` is passed through to the browser for backward navigation):

```
onKeyDown
  ├── wrong-scan check (el._wrongScan flag set by handleQtyInput when PO format /\d[a-zA-Z]/ detected)
  ├── wrong-PN check (value on a PN field ≠ delivery PN, applies whether typed or scanned)
  ├── markAccepted(el) — beep + transient green flash (only visible feedback that input was registered)
  ├── auto-add new pick-row / box-row / perbox-row when on the LAST row → focus new row, return
  └── autoAdvance(el) → focuses the next visible input across the whole pallet block
```

There is **no persistent typed-vs-scanned visual state**. The old `markScanned` / `markTyped` / `tagStateOf` / `applyTagState` / scan-tag UI was removed after field testing — the timing heuristic (`< 50ms` between keystrokes) misclassified slower Zebras as typed input, producing visual discrepancies without changing any validation logic. Per-pallet status now relies entirely on the validation strip (`MATCH` / `MISMATCH` / `PENDING`).

Adding a new row type? It needs the same triple-check before reaching `autoAdvance` to avoid the cursor jumping out of its section. Pattern: when `el.closest('.your-row') === wrap.lastElementChild`, call your `addYourRow(pid)`, focus its input, and `return`.

`handleQtyInput` strips non-digits + leading zeros (`A00250` → `250`) and flags `el._wrongScan` if the input looks like a country code (no digits at all) or a PO (`/\d[a-zA-Z]/`).

### Persistence & multi-session

Each in-progress delivery is its own "session", keyed by `localStorage['arrowPackingV1:<sessionId>']`. The active session ID lives in the URL hash (`#s=<id>`), making sessions bookmarkable and openable in parallel browser windows for true side-by-side use. The 📑 dropdown in the topbar lists/switches/deletes sessions; `hashchange` triggers a full page reload to swap loaded state cleanly.

**The app always opens blank.** `ensureFreshSessionOnBareOpen()` runs first in `DOMContentLoaded` (before `migrateLegacyState()`/`loadState()`): if the URL has no `#s=<id>` at all, it silently assigns a new random id via `history.replaceState` (no reload, no hashchange) before anything reads `getSessionId()`. So double-clicking the file or opening a fresh tab never resumes an old in-progress order — only an explicit link (a bookmarked `#s=default` or `#s=s_xxxx`) does. `startNewOrder()` (🆕 "New Order" in the topbar, and "+ New Order" in the 📑 dropdown — both call it) flushes the current session via `saveState()` immediately, bypassing the debounce, then calls `setActiveSession(makeSessionId())` to jump to a new blank one in the same tab. Nothing is ever deleted by either path — old sessions stay listed in 📑 until explicitly removed via `deleteSession()`.

`saveStateDebounced()` (400ms debounce) writes via `getStorageKey()` → the active-session key. `loadState()` rehydrates via `restoreState()`. `migrateLegacyState()` (called once at DOMContentLoaded) moves any pre-multisession state from the unprefixed `arrowPackingV1` key into `arrowPackingV1:default`.

The serialized shape includes per-input tag state (`scanned` / `typed` / `wait`) so the green/amber/gray badges survive reloads. When adding a new field, both `serializeState` and `restoreState` must handle it — they are mirror functions.

### Random PN Audit tool

`/* ══ RANDOM PN AUDIT ══ */` — standalone spot-check modal (`#audit-modal`, opened via the 🎲 Audit PN button in `.top-bar-inner`). The operator scans random boxes from anywhere in the delivery in sequence; each scan is checked against the Pick Label PN (`#delivery-pn`) if set, otherwise the session's first scan becomes the reference. Deliberately ephemeral — never reads/writes pallet DOM state, not persisted, resets every time it's opened. Reuses `stripPNPrefix`, `beepOk`/`beepError`, and the `scan-flash-ok`/`scan-wrong` CSS classes rather than duplicating scan-feedback logic.

### Audio & welcome splash

`/* ══ AUDIO FEEDBACK ══ */` defines all sounds via Web Audio (no files). Every sound funnels through `_tone(freq, opts)`, which is gated by `isMuted()` (persisted to `localStorage['arrowPackingMutedV1']`). The mute toggle 🔊/🔇 lives in the topbar. Status-transition chimes (`chimeMatch`, `chimeMismatch`, `fanfareComplete`, `warnHeight`) fire on rising edges only — tracked via `_lastPalletStatus`, `_lastDeliveryDone`, `_lastHeightExceed` maps so chimes don't fire on every keystroke or on initial state load.

The quick-guide splash (`#welcome-splash`) auto-opens on first ever load (`localStorage['arrowPackingWelcomeSeenV1']` flag). The `?` button in the topbar re-opens it anytime without clearing the flag.

### Outputs

Two active output paths (Print UI + legacy code), all read live DOM (not the localStorage state):

| Output | Function | Notes |
|---|---|---|
| Browser Print | `doPrint` → `buildPrintHTML` | Injects into `#print-root` div (NOT an iframe), then calls `window.print()`. Browser dialog lets user choose printer or Save as PDF. |
| PDF download | `doExportPDF` | **UI button removed** — function still exists in JS. jsPDF + autoTable, A4. |
| CSV download | `doExportCSV` → `buildExportData` | **UI button removed** — function still exists in JS. One row per actual box; UTF-8 BOM for Excel. |

`doExportPDF` duplicates box-counting logic from `buildPrintHTML`. If restoring PDF export, sync both.

**Print pagination gotcha:** `html` and `body` both have `height:100%;overflow:hidden` for the app's scroll layout. This clips `#print-root` to viewport height during print, causing only 1 page to output regardless of content. The `@media print` block must override with `html,body{height:auto!important;overflow:visible!important;display:block!important;}`. Layout is A4 portrait via an explicit `@page{size:A4 portrait;margin:10mm;}` rather than relying on the printer/browser default.

**Per-pallet pagination is JS-driven, not left to the browser alone.** `.pr-pallet-block{page-break-inside:avoid}` stops a pallet's card from being sliced mid-table, but relying on that alone can leave a large, unexplained blank gap at the bottom of a page whenever a pallet gets pushed to the next one. `applyPrintPagination()` (in the `/* ══ PRINT ══ */` section, called from `doPrint()` right before `window.print()`) simulates the browser's own pagination in JS — it measures real rendered heights (`getBoundingClientRect()`) of the header and every `.pr-pallet-block` against the A4 content box (`PRINT_CONTENT_WIDTH_PX`/`PRINT_CONTENT_HEIGHT_PX`, converted from the same `@page` mm values), and whenever a pallet doesn't fit in what's left of the current page: (1) adds the `.pr-force-break` class to it (`page-break-before:always`, so the real break always lands exactly where we predicted, not wherever the browser's own layout happens to decide), and (2) if the leftover space was more than `PRINT_GAP_RATIO` (25%) of a page, inserts a `.pr-gap-note` ("Pallet N continues on next page") into that gap so it doesn't read as a printing error. Because of this, `doPrint()` briefly lays `#print-root` out off-screen (`position:fixed;left:-10000px`) at the real print width before the actual print pass — that's required for the height measurements to reflect real print typography/wrapping (on-screen width is otherwise the full app viewport, not A4). If you change pallet-block markup/CSS in a way that affects its rendered height, this still works automatically since it measures live layout — but if you change `@page`'s margin or size, update the matching `PRINT_PAGE_*_MM` constants next to `applyPrintPagination()` too.

Per-pallet block order in `buildPrintHTML`: pallet-hdr → pn-line → per-box cards (conditional, `showBoxDetail`) → breakdown table → summary table. Pre-compute conditional HTML as a variable before the `html +=` template literal rather than appending separately after.

## Constants you'll touch often

- `PTYPE`: pallet code → dimensions in cm (e.g. `EP:'120×80'`). Current types: `KP|EP|BP|GP|NP|AP`. `AP:'140×105'` (Amphenol supplier pallets).
- `HEIGHT_LIMITS`: `{ europe: 180, export: 160 }`. Drives the "exceeds max" warning on the height input.
- `STORAGE_BASE` / `SESSION_PREFIX`: `'arrowPackingV1'` / `'arrowPackingV1:'` — bump the `V1` suffix if you change the serialized shape in a backwards-incompatible way (also requires migrating existing `:default`, `:s_xxx` keys). Kept the `arrowPacking` name even after the app was renamed to "Outbound Checklist" — renaming it would orphan every operator's saved session.
- `STORAGE_VERSION`: numeric guard inside the saved JSON; `restoreState` refuses to load when the file's version differs.
- `WELCOME_KEY`, `MUTE_KEY`, `'arrowPackingPrintBoxDetailV1'`, `OPERATOR_KEY`: global (not per-session) UI preferences in localStorage. Adding a new global pref? Create `loadXxxOpts()` / `saveXxxOpts()` and call `loadXxxOpts()` in DOMContentLoaded.
- `OPERATOR_KEY` (`'arrowPackingOperatorNameV1'`): remembers whatever name was last typed into Operator, independent of session — since a plain webpage can't read the signed-in Windows/AD identity, this is what stands in for "auto-fill from the Windows session": each person's own Windows/Edge login has its own browser storage, so their name comes back automatically without ever contacting AD. `loadRememberedOperator()` (called in DOMContentLoaded, before `loadState()`) prefills the field; a session's own saved operator (restored by `restoreState`) still wins over it when present. `rememberOperatorName()` fires on every Operator `oninput`.

## UI language

The UI strings are **English** (finalized in commit `7db6dd7`). The user/operator is Spanish-speaking but the app text stays in English — don't translate UI strings to Spanish unless explicitly asked.

## Conventions

- The file is hand-maintained as one big document; section comments `/* ══ NAME ══ */` are load-bearing for navigation — keep them when adding new function groups.
- Pallet IDs (`pid`) are interpolated into every element ID (`box-pn-${pid}`, `pick-rows-${pid}`, etc.). Always use template literals against the pallet's `pid`, never hard-code IDs.
- HTML inserted via template strings; user-controlled values go through `esc()` (l.449) before interpolation in print/PDF paths.
- **PN prefix stripping**: `stripPNPrefix(scanned, delivPN)` normalizes both values (uppercase, `_`→`-`) and strips any prefix left of the delivery PN if the scanned value ends with it (e.g. `1PASD123` → `ASD123`). Called in `onKeyDown` at Enter/Tab time, never on keystroke. Never blocks on mismatch.
- **PN uppercase-on-input**: `uppercaseInPlace(el)` rewrites a PN field's value to uppercase in place (cursor position preserved) on every `oninput` — called from `onDeliveryPNInput`, `onBoxPNInput`, `onPerBoxPNInput`, and the audit modal's scan input. Any new PN field needs the same call (the Zebra sometimes sends lowercase depending on symbology config).
