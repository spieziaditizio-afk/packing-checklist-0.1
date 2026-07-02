# Print report improvements — design

## Context

`packing-check-list.html`'s Browser Print output (`doPrint` → `buildPrintHTML`) currently has four gaps identified from field use:

1. Nothing stops an operator from printing a report with missing pallet data (type/weight/height) or with pick-vs-verified quantity mismatches.
2. The printed report shows only the aggregate Pick Label total, not the individual Pick 1 / Pick 2 / … quantities.
3. On a black & white printer, pallets are hard to tell apart — the current `.pr-pallet-hdr` navy bar reduces to a flat gray band with no strong separator between pallets.
4. The `Per-Box Detail` chips (`.pr-box-card`) are large enough that few fit per row, wasting page space.

All four changes are scoped to the print path only (`buildPrintHTML`, its `@media print` CSS, and a new pre-print validation gate). No changes to on-screen UI, PDF export, or CSV export are in scope.

## 1. Print-blocking validation modal

**Behavior:** `doPrint()` runs a new `validatePrintReady()` before touching `#print-root`. It inspects every `.pallet-block` and, per pallet, checks:

- Pallet type is selected (`getPType(pid)` truthy)
- Weight is entered (`weight-${pid}` non-empty)
- Height is entered (`height-${pid}` non-empty)
- Pick Label total > 0 (sum of `#pick-rows-${pid} .pick-row input`)
- Verified total > 0 (sum of scanned/visual box quantities, same logic already in `buildPrintHTML`)
- Pick total === Verified total (no mismatch)

A pallet that is completely untouched (no type/weight/height/quantities at all) still fails every applicable check above — there is no special-case exemption for empty pallets. An operator who added a pallet card but doesn't need it must remove it via the existing "✕ Remove" button rather than leave it blank.

**If any pallet fails any check:** printing is blocked. A new modal (`#print-block-modal`), styled to match the existing `#welcome-splash` (same dark overlay, centered card, fonts, colors), opens listing the issues grouped by pallet number, e.g.:

```
⚠ No se puede imprimir — faltan datos

Pallet 1: falta altura
Pallet 2: falta tipo de pallet · cantidades no coinciden (Pick: 100 · Verificado: 80)
Pallet 3: sin datos cargados

[Entendido]
```

`doPrint()` returns immediately in this case (no `buildPrintHTML()` call, no `window.print()`). If validation passes, `doPrint()` proceeds exactly as today.

**Not in scope:** CSV/PDF export are unaffected — only Browser Print gets this gate, per the current ask.

## 2. Pick Label quantities table in print

`buildPrintHTML` gains a new small table per pallet, inserted after the `pr-pn-line` div and before the (existing) `Per-Box Detail` chips and `Box Breakdown by Quantity` table:

```
Pick Label Breakdown
┌─────────────┬───────┐
│ Pick Label  │  Qty  │
├─────────────┼───────┤
│ Pick 1      │  50   │
│ Pick 2      │  30   │
└─────────────┴───────┘
```

- One row per pick row with qty > 0 (rows with empty/zero qty are skipped — same convention as every other aggregator in this file).
- Label text is the row's position (`Pick 1`, `Pick 2`, …), matching the on-screen `.scan-row-num` label.
- If a pallet has zero qualifying pick rows, render the same "No data entered" placeholder row style already used in the Box Breakdown table.
- Reuses `pr-table` / `pr-breakdown-lbl` classes for visual consistency with the tables directly below it — no new table styling needed.

## 3. Pallet separation — black number tab

Each pallet's print block is restructured so a solid black tab containing a large white pallet number sits to the left, spanning the full height of that pallet's content (title bar through summary table), with the existing content boxed in a thin border to its right.

**Structure change:** everything currently emitted per-pallet — from `.pr-pallet-hdr` through `.pr-summary-tbl` — gets wrapped in:

```html
<div class="pr-pallet-block">
  <div class="pr-pallet-tab">1</div>
  <div class="pr-pallet-content">
    <!-- existing .pr-pallet-hdr, .pr-pn-line, pick table, box cards, breakdown table, summary table -->
  </div>
</div>
```

- `.pr-pallet-block{display:flex;margin-top:14px;}`
- `.pr-pallet-tab{background:#000;color:#fff;font-weight:bold;font-size:20px;min-width:26px;display:flex;align-items:center;justify-content:center;padding:4px;-webkit-print-color-adjust:exact;print-color-adjust:exact;}`
- `.pr-pallet-content{flex:1;border:1px solid #999;border-left:none;}`

No changes to the internal navy `.pr-pallet-hdr` bar or badge — only the new outer wrapper changes.

**Pagination:** per the existing documented gotcha (CLAUDE.md), no `page-break-inside:avoid` is added to `.pr-pallet-block` itself — only the existing per-table `avoid` rules remain. This preserves today's auto-pagination behavior across multi-pallet, multi-page reports.

## 4. Smaller Per-Box Detail chips

Within the existing `@media print` block:

| Property | Before | After |
|---|---|---|
| `.pr-box-cards` grid column min-width | `62px` | `46px` |
| `.pr-box-card` padding | `5px 5px` | `3px 4px` |
| `.pr-box-card-qty` font-size | `11px` | `9.5px` |
| `.pr-box-card-pn` font-size | `6.5px` | `6px` |

Targets roughly 8-9 chips per row (moderate density) instead of the current handful, while keeping qty and PN legible.

## Out of scope

- PDF export (`doExportPDF`) and CSV export (`buildExportData`) are untouched — the validation gate and layout changes are print-only, per the requests.
- No changes to on-screen (non-print) styling or the pallet editing UI.
- No change to what counts as a "match/mismatch" pallet status elsewhere in the app (sidebar, per-pallet badge) — only the print validation gate and print layout are new.
