---
format: 1920x1080
duration: 68.5s
message: "Outbound Checklist turns pre-shipment pallet verification into a fast, mistake-proof scan workflow."
arc: Demo Loop (hook → product_intro → demo cycle 1 → demo cycle 2 → climax feature → benefit → differentiator → CTA)
audience: warehouse operators and ops leads evaluating or onboarding to the tool
mode: collaborative
music: none — no HeyGen credential available for BGM retrieval; narration (SCRIPT.md) still runs
---

## Video direction

- **Palette** (from `frame.md`, by role): canvas `#0d1117` (paper) / `#1c2433` (paper-2, card surfaces) · ink `#e6edf3` (text) · accent `#38bdf8` (cyan — interactive/focus/in-progress) · status success `#10b981` (MATCH/complete) · status error `#dc2626`/`#f87171` (over-target/mismatch) · text-secondary `#8b949e`. Never invented — these are the real app's own CSS custom properties.
- **Type**: display/body = DM Sans (frame.md's weight ramp); mono = DM Mono for every label, badge, and numeric value — matches the real app's actual typography split.
- **Motion grammar**: `power3` long-tail settle by default everywhere; entrances always `fromTo` (explicit from-state); reveals paced to the voiceover — nothing appears before its spoken cue, and each frame's back ~50% carries the payoff reveal, never the front ~25%. Overshoot (`back.out`) is reserved for exactly one beat in the whole film: the over-target toast slam in Frame 5 — the one moment the product itself is meant to feel sudden.
- **Held / breather frames**: Frame 5 (the over-target stop) is the deliberate long hold — the climax, held through its full line with no further development after the toast lands. Frame 8 (outro) is the closing held frame. Frames 6 and 7 are secondary calm beats (titlecard-reveal's one-move-then-hold contract). Frames 1–4 stay in continuous motion, timed to the VO, so the video's energy has real shape rather than being uniformly busy.
- **Timing note**: every frame's Scene windows below were authored against an earlier duration estimate; narration was since regenerated and each frame's `duration:` re-synced to its real spoken length. Treat each frame's Scene timestamps as **proportional guidance**, not literal seconds — scale them to the frame's actual `duration:` value and pace reveals to the real narration audio in `assets/voice/NN.wav`, not to the written numbers.
- **Negative list**: no slideshow front-loading (nothing dumps at t=0 beyond what the VO is saying then); no lazy breathing or idle camera drift (holds are stillness, at most `sine-wave-loop` low-amplitude jitter); no invented UI — every scan value, badge string, and toast copy in Frames 2–6 is verbatim from `outbound-checklist.html`; **no mouse cursor anywhere** — the real app is scanner/keyboard-driven (Enter/Tab), never mouse-driven, so Frames 3–4 use `device-surface-showcase`'s cursorless static-tour variant specifically because a cursor-driven blueprint (`cursor-ui-demo`) would misrepresent the product; no glow-heavy hero treatment beyond the one completion pulse the real app's own CSS already implements; caption band (bottom ~17%) stays clear on every frame.

## Locked

Sketch sheet (`storyboard.html` v1) approved as drawn — no frame changes requested.
Layout, hierarchy, and copy for all 8 frames are locked; Step 4 dresses these
layouts with real motion and design treatment, never redraws them.

## Frame 1 — One box too many

- scene: Full-bleed black canvas. A short, tense line types out, then cuts hard.
- duration: 7.893s
- transition_in: cut
- status: built
- voiceover: "One box too many on a pallet. Nobody catches it on the floor — it rides all the way to the truck before anyone knows."
- src: compositions/frames/01-hook.html
- type: hook
- persuasion: Pain validation
- beat: tension
- blueprint: kinetic-type-beats (Adapt — Hook escalation sub-shape)
- asset_candidates:
- sfx: impact-bass-1

narrativeRole: Opens on the cost of a missed scan in outcome language — no product name yet, just the stakes.
keyMessage: A mis-scan that isn't caught costs you at the dock, not on the floor.

Adapt: keep the signature hard-cut-in-place token swap; use three escalating phrases (the corpus typically runs two) since the line itself has three natural beats.
Scene 1 (0.0–1.6s): solid black field (`#05070a`). "One box too many." hard-cut FLASHES in dead-center, bold DM Sans, ink color — `discrete-text-sequence`. Nothing else on screen.
Scene 2 (1.6–2.8s): on the VO's "Nobody catches it —", beat 1 clears by hard cut and "Nobody catches it —" lands in accent cyan (`#38bdf8`) at the same center anchor — `discrete-text-sequence`.
Scene 3 (2.8–4.0s): on "until the dock," the final phrase hard-cuts in ink white and holds to the cut — settle only, no further movement.

## Frame 2 — Meet Outbound Checklist

- scene: Hard cut from black into the app's real topbar sliding into view — app icon, "Outbound Checklist" title, Arrow wordmark. Cyan glow accent.
- duration: 9.835s
- transition_in: zoom-through
- status: built
- voiceover: "This is Outbound Checklist — Arrow Electronics' pre-shipment verification tool, built to run scan by scan, right where the pallets get staged."
- src: compositions/frames/02-product-intro.html
- type: product_intro
- persuasion: Friction reduction
- beat: curiosity → clarity
- blueprint: compose
- asset_candidates:
- sfx: pop

narrativeRole: Names the product and plants it in the real, physical warehouse context (not an abstract SaaS dashboard).
keyMessage: This is a tool built for the floor, not a slide deck.

Compose: too short and too simple for a full blueprint reach — a plain entrance-then-settle, composed from the vocabulary. Centered framing, ~2 depth layers (topbar over app background).
Scene 1 (0.0–2.0s): black holds from Frame 1's exit. As the VO opens "This is Outbound Checklist —", the real topbar (app icon, "Outbound Checklist" title, Arrow wordmark) enters `fromTo({y:-26, autoAlpha:0}, {y:0, autoAlpha:1})` and settles on `power3` — a restrained entrance, no bounce.
Scene 2 (2.0–4.2s): as the VO says "scan by scan,", the pallet card and tab strip `fromTo` fade + slide in beneath the topbar on the same `power3` settle — `scale-swap-transition` style handoff from empty to populated.
Scene 3 (4.2–6.0s): on "right on the warehouse floor," hold static on the full app view — settle only.

## Frame 3 — Scan the pallet

- scene: The real pallet card — Pick Label PN field, Box Verification rows. Each scan lands with a green flash; the Box Progress bar climbs live as quantities register.
- duration: 8.213s
- transition_in: crossfade
- status: built
- voiceover: "Scan the Pick Label PN, then each box's quantity. Every valid scan flashes green and beeps the instant it lands."
- src: compositions/frames/03-scan-pipeline.html
- type: feature_showcase
- persuasion: Show-don't-tell proof
- beat: clarity → control
- blueprint: device-surface-showcase (Adapt — static-tour variant)
- asset_candidates:
- sfx: click

narrativeRole: First demo cycle — establishes the core scan-and-verify loop the rest of the video builds on.
keyMessage: Every scan gets instant, unambiguous feedback.

Adapt: keep the static-tour's defining trait — camera locked the whole clip, every state change is element-level, **no cursor at all** (the real trigger is an off-screen scanner, not a click) — and cast the app's own Pick Label / Box Verification rows as the "screens" that advance.
Scene 1 (0.0–2.0s): camera static. Pick Label PN field at rest ("AS-4471-B" already legible) — nothing pre-empted before the VO names it.
Scene 2 (2.0–4.3s): on "Scan the Pick Label.", the PN field border/background flashes green (0.15s in/out) — a plain color chord, `gsap-effects`.
Scene 3 (4.3–7.0s): on "Scan the quantities.", Box 1 then Box 2 flash green in the same way, 0.6s apart, while the progress bar fill advances stepwise (`0→6/9→9/9`, `scaleX` from a left origin) and the counter counts up in lockstep — `stat-bars-and-fills` + `counting-dynamic-scale`.
Scene 4 (7.0–10.0s): on "Every scan flashes green the second it lands," hold on the filled bar — settle only, no breathing.

## Frame 4 — It keeps moving with you

- scene: Progress bar reaches its target and pulses green; focus auto-switches to the next pallet's tab and scan field with no cursor click.
- duration: 10.197s
- transition_in: crossfade
- status: built
- voiceover: "Finish one pallet, and the app jumps straight to the next one's tab and scan field — an operator can work through a whole delivery without ever touching the mouse."
- src: compositions/frames/04-auto-advance.html
- type: feature_showcase
- persuasion: Feature-to-benefit translation
- beat: control → confidence
- blueprint: device-surface-showcase (Adapt — static-tour variant)
- asset_candidates:
- sfx: chime, click-soft

narrativeRole: Second demo cycle — shows the workflow compounds across a whole delivery, not just one pallet.
keyMessage: The app keeps pace with the operator; the operator never stops to drive it.

Adapt: same static-tour contract as Frame 3 (locked camera, no cursor) — the "screen advance" here is the tab strip's state step rather than a scan.
Scene 1 (0.0–2.5s): static camera on the completed Pallet-1 card (bar full, cyan) — nothing new yet; VO opens "Finish a pallet,".
Scene 2 (2.5–5.0s): on "and it jumps to the next one on its own —", the bar flips cyan→green with a one-shot completion glow (`ambient-glow-bloom`, finite, no loop) on the card; a beat later the tab strip crossfades its active state from Pallet 1 to Pallet 2 (background-color state step).
Scene 3 (5.0–8.0s): on "scan a whole delivery", Pallet 2's scan field gets a one-shot cyan focus-ring pulse (`asr-keyword-glow`) — the auto-advance's "no click" payoff, made visible.
Scene 4 (8.0–10.0s): on "without touching the mouse," hold — settle only.

## Frame 5 — It stops before the truck does

- scene: A scan pushes the count past target. The field flashes red, a centered "⚠ OVER TARGET" toast slams in, the progress bar locks red. The climax beat — held a moment longer than the others.
- duration: 10.368s
- transition_in: zoom-through
- status: built
- voiceover: "Scan one box too many, and it doesn't ride along quietly. It stops — right there, so the operator can find the double scan before it reaches the truck."
- src: compositions/frames/05-over-target-stop.html
- type: feature_showcase
- persuasion: Negative contrast
- beat: tension → relief
- blueprint: compose
- asset_candidates:
- sfx: error

narrativeRole: The signature differentiator — most tools would let the error ride to the next pallet or the print report; this one stops the run cold, on the spot.
keyMessage: Over-target is a hard stop, not a warning.

Compose: this is the film's one deliberately punctuated beat — the only place `back.out` overshoot is allowed (see Video direction).
Scene 1 (0.0–3.0s): static hold on Pallet 2's card mid-scan — nothing new yet, as the VO opens "Scan one box too many,".
Scene 2 (3.0–5.5s): on "and it doesn't ride along quietly.", Box 2's field flashes red (color chord) and the progress bar begins tracking toward red.
Scene 3 (5.5–7.0s): on "It stops —", the "⚠ OVER TARGET" toast spring-pops in dead-center (`spring-pop-entrance`, light overshoot — the one exception to the smooth-only rule) as the bar locks full red.
Scene 4 (7.0–12.0s): on "before it ever reaches the truck," everything HOLDS — the deliberate long beat; at most a subtle low-amplitude jitter (`sine-wave-loop`) on the toast's outline keeps it from reading dead. No further development.

## Frame 6 — One clear answer

- scene: The red state resolves; validation strip flips to "✓ MATCH" in green; a print/report preview card slides up beside it.
- duration: 7.765s
- transition_in: crossfade
- status: built
- voiceover: "Every pallet resolves to one clear answer — a green MATCH badge — and a print-ready report, ready for the paper trail."
- src: compositions/frames/06-match-report.html
- type: benefit_highlight
- persuasion: Risk reversal
- beat: relief → trust
- blueprint: titlecard-reveal (Adapt — Benefits variant)
- asset_candidates:
- sfx: chime

narrativeRole: Pays off the tension from Frame 5 — the workflow always lands on a clean, provable result.
keyMessage: The paper trail is automatic.

Adapt: keep the "one slide-up crossfade IS the one move" contract; cast it as the badge flip + report card arrival instead of two text lines.
Scene 1 (0.0–2.0s): the red state from Frame 5 is still resolving as this frame opens (the crossfade seam carries it); VO opens "Every pallet resolves to one answer —".
Scene 2 (2.0–4.5s): on "MATCH —", red clears, the bar settles green, and the badge text swaps PENDING → "✓ MATCH" (`discrete-text-sequence`) with a gentle ~95%→100% scale-up settle (`power3`) — the one restrained move.
Scene 3 (4.5–8.0s): on "with a print-ready report to prove it," the report-preview card slides up from the bottom-right corner and holds — no further development after it lands.

## Frame 7 — No server, no install

- scene: The live app UI pulls back / dissolves into a simple visual: a single file icon labeled "outbound-checklist.html" — double-click, opens instantly.
- duration: 8.405s
- transition_in: blur-crossfade
- status: built
- voiceover: "No server, no database, nothing to deploy. It's one HTML file — double-click it and start scanning in seconds."
- src: compositions/frames/07-single-file.html
- type: benefit_highlight
- persuasion: Friction reduction
- beat: ease
- blueprint: titlecard-reveal (Adapt)
- asset_candidates:

narrativeRole: The differentiator against a typical SaaS pitch — this tool imposes zero setup cost on the warehouse.
keyMessage: It's as portable as the pallet it's checking.

Adapt: single restrained reveal + hold, with the file icon standing in for the usual text lockup.
Scene 1 (0.0–2.0s): the live app UI (still visible from Frame 6, `blur-crossfade` seam) dims and blurs (`depth-of-field-blur`) as the VO opens "No server. No install.".
Scene 2 (2.0–4.5s): on "One file —", the app UI has fully dissolved to the graph-paper ground and a single file icon spring-pops in centered (`spring-pop-entrance`, restrained — no overshoot here), labeled "outbound-checklist.html" beneath in DM Mono.
Scene 3 (4.5–7.0s): on "open it and start scanning," the sub-line "no server · no install · double-click to open" fades up beneath and holds — settle only.

## Frame 8 — Every pallet, verified

- scene: Calm end card — the Outbound Checklist wordmark/title centered, tagline beneath, Arrow wordmark small.
- duration: 5.824s
- transition_in: crossfade
- status: built
- voiceover: "Outbound Checklist — every pallet, every box, verified before it ever leaves the dock."
- src: compositions/frames/08-outro.html
- type: cta
- persuasion: Value stacking
- beat: confidence
- blueprint: titlecard-reveal (Adapt — CTA/card-chain register)
- asset_candidates:

narrativeRole: Closes on the brand line, calm and declarative — no "sign up," this is an internal tool closing on adoption confidence.
keyMessage: Outbound Checklist. Every pallet, verified before it ships.

Adapt: monochrome, one card, hold-to-end — the CTA variant's contract minus the URL (an internal tool has no signup link).
Scene 1 (0.0–2.0s): black field holds (`crossfade` seam from Frame 7); on "Outbound Checklist.", the title `fromTo` fade+scale settles dead-center — gentle fade-in + subtle scale-up, `power3`.
Scene 2 (2.0–4.5s): on "Every pallet, verified before it ships.", the tagline slides up + fades in beneath (one slide-up crossfade — the same move-shape as Frame 6's badge flip, for a visual rhyme across the film).
Scene 3 (4.5–6.0s): the Arrow wordmark fades in small, bottom corner; the whole card holds to the final render frame — this is the film's one true exit (Part 3 of the motion doctrine), so nothing further moves.
