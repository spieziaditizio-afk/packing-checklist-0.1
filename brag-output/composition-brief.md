# Hyperframes Composition Brief: Outbound Checklist

## Objective
Create a short launch-style brag video for Outbound Checklist, the pre-shipment pallet/box verification tool used on Arrow Electronics' Sevenum NL warehouse floor.

## Output
- Composition directory: `brag-output/composition/`
- Rendered video: `brag-output/brag.mp4`
- Format: landscape — 1920x1080
- Duration: ~19.5 seconds

## Source Material
- Project root: `c:\Users\aspiezia\Downloads\packing checklist 0.1`
- Primary files read: `outbound-checklist.html` (full single-file app — CSS `:root` theme vars, topbar, welcome splash, pallet Box Progress bar, validation strip, scan toast, audit modal), `CLAUDE.md` (architecture/domain notes)
- Product name: Outbound Checklist
- Tagline / strongest claim: "Over-target is a hard stop, not a warning." — reaching the target advances the operator to the next pallet; exceeding it freezes the run with a centered toast instead of letting the excess ride silently into the next pallet.
- Key UI or visual moment to recreate: the pallet's Box Progress bar sweeping through its live color states (idle → low → mid → high → done, with a green completion pulse) directly beneath the validation strip, and the centered "⚠ OVER TARGET" toast with a hard red lock state.
- Copy that must appear verbatim:
  - "Outbound Checklist"
  - "✓ MATCH" / "⚠ OVER TARGET"
  - "One box too many. Almost shipped." (hook line, /brag original, not app copy)

## Creative Direction
- Tone preset: app-store
- Creative direction: a real internal ops tool, not a toy — feature-card clean, confident, warehouse-floor practical
- Interpretation: title-case labels, one capability per scene, clean slide/crossfade transitions (0.35-0.45s), no aggression or humor — confidence comes from showing the mechanism working, not from performing
- Angle: engineering confidence made visible — a scan pipeline disciplined enough to catch a mis-pick in real time, shipped as a single portable HTML file
- Hook: dark canvas, hook line types out, hard cut to the app's topbar sliding in
- Outro / punchline: "Outbound Checklist. One file. Zero build. Every box accounted for."
- Avoid:
  - Generic SaaS language ("streamline your workflow" etc.)
  - Abstract filler visuals — every scene must show the actual product UI
  - Any unrelated visual redesign of the app's real palette/typography

## Visual Identity
- Background: `#0d1117` (page), `#010409` (deep, topbar gradient start), `#161b22` (surface/card)
- Text: `#e6edf3` (primary), `#8b949e` (secondary)
- Accent: `#38bdf8` (cyan — active tab, focus ring, progress mid-state), `#10b981` (green — MATCH, progress done/completion pulse), `#f87171` / `#dc2626` (red — MISMATCH, over-target lock)
- Display font: DM Sans, 600 weight
- Body font: DM Sans; DM Mono for numeric values, badges, labels
- Visual references from the project: pallet tab strip (cyan active fill), Box Progress bar with its 6-state color sweep, validation strip badge (✓ MATCH / ✗ MISMATCH / PENDING), centered scan toast pattern, Arrow wordmark (white on dark)

## Storyboard
Use the storyboard in `brag-output/brag-plan.md` as the creative contract.

Scene summary:
1. Hook — 3.7s — hook line types out on black, hard cut to "Outbound Checklist" title card with topbar sliding in
2. Reveal: the scan pipeline — 3.7s — Pick Label + Box Verification scans land, Box Progress bar fills live through its color states, Verified/Target counter climbs
3. Highlight — continuous cross-pallet scanning — 3.66s — pallet 1 completes (green pulse), focus auto-jumps to pallet 2's tab and scan field
4. Highlight — the hard stop — 4.75s — an over-scan flashes the field red, centered "⚠ OVER TARGET" toast slams in, progress bar locks red
5. Outro — 3.67s — bar settles green, validation strip flips to ✓ MATCH, report preview slides up, final product card + Arrow wordmark

## Audio
- Audio role: sparse professional accents over a warm, mid-tempo business bed
- Audio arc: bed enters low under the hook, rises slightly through the scan/auto-advance beats, ducks cleanly for one unambiguous alert tone at the hard-stop moment, recovers, fades out under the outro
- Music: `happy-beats-business-moves-vol-9-by-ende-dot-app.mp3`
- Music treatment: start ~0.15-0.2 volume under the hook, rise to ~0.3-0.35 through scenes 2-3, duck to ~0.1 for ~0.5s around the over-target alert, recover to ~0.3, fade to 0 over the last ~0.8s
- Music cue guidance: bundled preset at `<skill-dir>/assets/music/cues/happy-beats-business-moves-vol-9-by-ende-dot-app.music-cues.md` / `.json` (114.84 BPM). Candidate strong-cue targets near 3.70s (scene 1→2 cut), 7.40s (scene 2→3), 11.06s (scene 3→4, completion pulse) — pick from the preset's actual `strongCues` array within ±0.15s of these. Use 1-3 locks total; treat the rest as natural timing.
- Audio-reactive treatment: subtle — the Box Progress bar's glow/presence may breathe slightly with RMS during scene 2's fill; no waveform/equalizer visuals, no strobing
  - **Implementation note:** skipped in the delivered composition — no per-frame audio-data extraction was run. The completion-pulse glow on `#pallet-card` (7.4s) is a plain timeline tween, not audio-reactive. Documented per the skill's fallback allowance rather than blocking the render.
- Audio-coupled moments:
  - Scene 1 hook line — types out character by character, keypress ticks (randomized from `keyboard/`)
  - Scene 2 scan events — a click/tick per scan landing, synced to each flash
  - Scene 3 completion pulse + tab switch — a positive chime on the pulse, a distinct switch tone on the tab change
  - Scene 4 over-target toast — one clear interface alert tone the instant it appears, music ducked so it reads cleanly, no competing SFX
  - Scene 5 MATCH flip — a soft confirmation chime
- SFX selection guidance: app-store energy — a consistent light layer, `interface/drop_*` or `interface/click_*` per scan/feature moment, one clear alert cue for the over-target moment, `impact/impactBell_heavy_000` (or similar) on the final MATCH/outro payoff. Keep SFX at 0.65-0.75 volume per the tone's guidance; the alert tone can sit slightly hotter for clarity.
- SFX analysis guidance: read `<skill-dir>/assets/sfx/sfx-analysis.md` before final selection; prefer low/medium HF-risk files since this video repeats scan-tick sounds several times.
- Exact SFX choice: Hyperframes should choose exact filenames, timestamps, density, and volume based on the implemented animation.
- Audio files: copy the chosen music (and any Hyperframes-selected SFX) into `brag-output/composition/assets/`

## Hyperframes Instructions
Load the composition-building Hyperframes domain skills — `hyperframes-core` (composition contract + `data-*` timing), `hyperframes-animation` (motion), `hyperframes-creative` (design spec, beats, audio-reactive), `hyperframes-keyframes` (seek-safe keyframes), and `hyperframes-cli` (lint/check/render). `/brag` is its own workflow: do not enter the `hyperframes` entry-point intent interview and do not route into its generic promo / launch-video workflow. Prefer native Hyperframes conventions over anything in `/brag`.

Requirements:
- Show at least one real UI, copy, or visual element from the source project (the Box Progress bar, validation strip, and over-target toast are the required centerpiece).
- Keep all text readable in the final render.
- Keep the video within 15-25 seconds.
- Include the planned music/SFX layer — not disabled, not documented as silent.
- Treat `/brag` audio notes as guidance, not a fixed cue sheet. Choose SFX after the visual animation exists.
- Treat music cue metadata as optional timing hints; ignore cues that hurt readability, scene pacing, or the product story.
- Major reveals may move toward nearby strong cues within about 0.15s. Smaller entrances may align to nearby beat points within about 0.10s. Use only 1-3 strong cue locks in this 19.5s video.
- Use SFX to support motion and interaction per the moment→sound heuristics in `audio.md`.
- Honor the planned music treatment (ducking under the alert, fade-out under the outro) using the best Hyperframes-supported implementation.
- Consider the Hyperframes audio-reactive workflow for a subtle glow/presence treatment on the progress bar during scene 2; skip and document if extraction is unavailable.
- Use local assets for audio (copy into `composition/assets/`) and any required runtime/media dependencies.
- Run `hyperframes check` before render — it is brag's single gate.
