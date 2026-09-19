# Brag Plan: Outbound Checklist

## What is this app?
A single 2,200-line HTML file that a warehouse operator opens by double-click — no server, no build — to scan pallets and boxes against pick-label quantities before a shipment leaves the dock, and it hard-stops the instant a scan goes one piece over target.

## The angle
This isn't a SaaS demo — it's a real tool built for the floor at Arrow Electronics' Sevenum warehouse. The angle is engineering confidence: a scan pipeline disciplined enough to catch a mis-pick in real time, shipped as one portable file. The hook is the product's own hardest rule made visible: over-target isn't a warning, it's a hard stop.

## Hook (first 2-3 seconds)
Dark GitHub-style canvas (#0d1117). A single line types out: "One box too many. Almost shipped." Cut hard to the app opening — cyan (#38bdf8) glow, "Outbound Checklist" title card.

## Key moments (the middle)
- The scan pipeline in motion: a Pick Label field takes a scan, flashes green, the pallet's Box Progress bar climbs through its live color states (idle → low → mid → high).
- Continuous cross-pallet scanning: the moment a pallet's progress bar completes (green pulse), focus jumps straight to the next pallet's tab and scan field — no mouse, no clicking around.
- The over-target hard stop: a scan pushes the count past target, the field flashes red, a centered toast reads "⚠ OVER TARGET", and the progress bar locks into its red over-state. The run stops cold instead of riding into the next pallet.

## Outro / punchline
Progress bar settles back to full green, the validation strip flips to "✓ MATCH," and a print/report preview slides up. Final card: "Outbound Checklist. One file. Zero build. Every box accounted for."

## User flow worth showing
1. Entry: operator scans the delivery's Pick Label PN, then the Pick Label quantities that set the pallet's target.
2. Key action: operator scans boxes in Box Verification — progress bar fills live, focus auto-advances across pallets, one over-scan triggers the hard stop.
3. Result: pallet status flips to ✓ MATCH and the print-ready report is generated.

## Tone
- Preset: app-store
- Creative direction: a real internal ops tool, not a toy — feature-card clean, confident, warehouse-floor practical
- Interpretation: title-case labels, one capability per scene, clean slide transitions, no aggression or jokes — the product earns attention by being genuinely well-engineered, not by performing.

## Format: landscape — 1920x1080
## Duration: ~19.5 seconds

## Visual identity (from the project)
- Background: #0d1117 (page), #010409 (deep/topbar gradient start)
- Accent: #38bdf8 (cyan, active tab/progress/focus states); #10b981 (green, MATCH/success/completion pulse); #f87171 / #dc2626 (red, MISMATCH/over-target)
- Text: #e6edf3 (primary), #8b949e (secondary)
- Display font: DM Sans (600 weight for titles)
- Body font: DM Sans; DM Mono for labels, badges, numeric values
- Strongest visual element: the pallet Box Progress bar's live color-state sweep (p-idle → p-low → p-mid → p-high → p-done, with a completion glow pulse) and the centered over-target toast

## Share copy (draft)
We built a warehouse checklist that physically stops you the instant you scan one box too many — as a single HTML file with zero build step. Outbound Checklist, shipped.

## Audio direction
- Role: sparse professional accents over a warm, mid-tempo business bed
- Music: `happy-beats-business-moves-vol-9-by-ende-dot-app.mp3` (114.84 BPM) — confident, clean, not corporate-bland
- Music treatment: starts under the hook at low volume, rises slightly into Scene 2, ducks briefly under the over-target stop beat for the SFX to read clearly, recovers for the outro, fades out over the final ~0.8s
- Music cue guidance: preset read from `happy-beats-business-moves-vol-9-by-ende-dot-app.music-cues.md`. Target strong cues near 3.70s (Scene 1→2 cut), 7.40s (Scene 2→3), 11.06s (Scene 3→4, progress-complete pulse), 15.28-15.81s (the over-target stop lands on/just after a strong beat for impact). Sequential progress-bar fill in Scene 2 rides the beat grid loosely rather than snapping every element to it.
- Audio-reactive treatment: subtle — the progress bar's glow may breathe slightly with the beat during fills, nothing else reacts to music energy
- SFX posture: moderate, motion-matched, professional restraint — a scan/click tick per scan event, a distinct alert tone for the over-target toast, a soft confirmation chime on the final MATCH
- Audio-coupled moments: scan ticks as Pick Label / Box quantities land, a sharper interface tone the instant the over-target toast appears, a short positive chime when the validation strip flips to MATCH
- Restraint rule: no waveform visualizations, no music swelling over the over-target moment (that beat needs clarity, not drama), never let SFX collide with the alert tone

## Storyboard

### Scene 1 — Hook — 3.7s
Full-bleed dark canvas (#0d1117). A single line types out character by character: "One box too many. Almost shipped." Cut on the last character to the app's topbar sliding into view — cyan glow icon, "Outbound Checklist" title, Arrow wordmark small and white.
Sequential/interaction: yes — the hook line types out character by character with key-tick sound
Audio intent: quiet tension, a held breath before the reveal
Audio-coupled idea: typing sound synced to each character; music enters low under the last word
Music: business-moves bed, low volume, entering
Transition mood: hard → Scene 2

### Scene 2 — Reveal: the scan pipeline — 3.7s
A Pick Label field receives a scan (green flash + tick sound), then a Box Verification row does the same. The pallet's Box Progress bar fills live, sweeping through its color states as pieces are verified. A small "Verified / Target" counter climbs in DM Mono.
Sequential/interaction: yes — three scan events land in sequence, each with its own flash + tick, progress bar advancing after each
Audio intent: mechanical confidence, a working system
Audio-coupled idea: scan tick per event, counter ticks upward in sync with the bar's advance
Music: bed continues, slight rise in presence
Transition mood: clean slide → Scene 3

### Scene 3 — Highlight: continuous cross-pallet scanning — 3.66s
Pallet 1's progress bar reaches full and pulses green ("p-done"). Focus visibly jumps: Pallet 2's tab activates (cyan fill), its scan field gets the focus ring, all without a cursor click. Small caption: "Finishes one pallet. Jumps to the next."
Sequential/interaction: yes — completion pulse → tab switch → focus ring, shown as one continuous beat
Audio intent: a satisfying snap of momentum
Audio-coupled idea: a soft chime on the completion pulse, a distinct "switch" tone on the tab change
Music: strong beat aligned near the completion pulse
Transition mood: clean slide → Scene 4

### Scene 4 — Highlight: the hard stop — 4.75s
A scan lands one piece past target. The field flashes red instantly, a centered toast slams in: "⚠ OVER TARGET" with sub-text "N pcs over target." The progress bar locks red. Caption: "Scan one too many? It stops. Now — not at the dock."
Sequential/interaction: yes — the over-scan event triggers flash → toast → bar lock as one fast, unmissable sequence
Audio intent: a clean, unambiguous alert — not alarming, but impossible to miss
Audio-coupled idea: music ducks briefly, a single clear interface alert tone on the toast's arrival, no other sound competing
Music: ducked under the alert tone, resumes after
Transition mood: hard cut → Scene 5

### Scene 5 — Outro — 3.67s
The over-target field resolves, the progress bar settles to full green, the validation strip flips to "✓ MATCH." A print/report preview slides up briefly behind it. Final card: "Outbound Checklist." / "One file. Zero build. Every box accounted for." Arrow wordmark small, bottom corner.
Sequential/interaction: yes — bar settle → MATCH flip → report slide-up, in that order
Audio intent: quiet resolution, earned calm
Audio-coupled idea: soft confirmation chime on the MATCH flip
Music: recovers from the duck, fades out over the last ~0.8s
Transition mood: soft → end

**Music mood for this video:** upbeat, confident, business-clean — never playful or joke-y
**Audio summary:** A low, confident business-moves bed carries the whole video, rising slightly through the scan/auto-advance beats, ducking cleanly for one unambiguous alert tone at the hard-stop moment, then recovering to fade out under a quiet, resolved outro.
