---
workflow: product-launch-video
flow: automation
storyboard: yes
message: "Outbound Checklist turns pre-shipment pallet verification into a fast, mistake-proof scan workflow."
destination: youtube
aspect: 1920x1080
language: en
audience: warehouse operators and ops leads evaluating or onboarding to the tool
length: 60-75s
angle: feature walkthrough — the real user flow, not an abstract pitch
narration: yes
---

## Intent

A show-it-as-is product tour of Outbound Checklist, a single-file warehouse app
used at Arrow Electronics' Sevenum NL site to verify pallets/boxes against pick
labels before a shipment leaves the dock. The video should feel like watching
someone competent use the actual tool: confident, practical, no stock-photo
marketing gloss. It follows the real operator flow — scan the delivery PN and
Pick Label quantities, scan boxes with the live progress bar, watch the app
auto-advance across pallets, see the over-target hard stop catch a mis-scan,
then the validation strip flips to MATCH and a report exports. Close on the
fact that it's one portable HTML file with zero build step.

## Assets

No external assets. Source is `outbound-checklist.html` (this repo) — the UI is
recreated from its actual CSS tokens, copy, and behavior (documented in this
project's `CLAUDE.md`), not screen-captured, since the source is a local file,
not a live URL.

## Customizations

- Show the over-target hard-stop toast and progress-bar lock as the signature
  "safety feature" beat — this is the app's most distinctive behavior.
- Callout card for "single HTML file, zero build step, works offline" near the
  close.
- English UI/narration (per this repo's convention: UI strings and deliverables
  stay in English even though the operator/user is Spanish-speaking).

## Notes

- Visual identity pulled directly from `outbound-checklist.html`'s `:root` CSS
  vars: bg `#0d1117` / `#010409` / `#161b22`, cyan accent `#38bdf8`, success
  green `#10b981`, error red `#f87171`/`#dc2626`, text `#e6edf3`/`#8b949e`,
  fonts DM Sans (body/display) + DM Mono (labels/values).
- A separate 19.5s hype/brag cut of this same app already exists at
  `brag-output/brag.mp4` in the repo root — this is a different, longer,
  narrated feature-walkthrough deliverable, not a re-cut of that one.
- No HeyGen sign-in assumed; narration should fall back to an offline TTS
  engine if not signed in (checked at Setup).
