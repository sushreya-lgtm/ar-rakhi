# AR Rakhi — Project Brief

**Deadline:** Rakhi is 2026-08-28 — this needs to be live and shareable by tomorrow.
**Type:** General shareable tool (anyone can open it, pick a Rakhi, personalize, send a link to anyone).

## What it is
A "Digital Bouquet"-style emotional micro-gift, but AR: sender picks an illustrated Rakhi from a
gallery and adds a name/note; recipient opens the link on their phone, points the camera at their
wrist, and the Rakhi anchors live onto their wrist via browser-based hand tracking.

## Stack decision (locked, don't re-litigate mid-build)
- **No native app, no Expo.** Expo Go can't run real ARKit/ARCore hand tracking (needs a custom
  EAS dev-client build — days, not hours) and forces an install, which kills the "just a link"
  emotional-gift use case.
- **Plain web app**, same pattern as `Social Projects/mateo-bouquet` (single-file-ish static
  HTML/CSS/JS, no build step, deployed as a link via Unravel Powers).
- **AR = MediaPipe hand-landmark detection in-browser** (`@mediapipe/tasks-vision` HandLandmarker,
  loaded via CDN, runs client-side via WebAssembly). Detects the wrist landmark from the live
  camera feed; a canvas overlay anchors + rotates the chosen Rakhi graphic to it in real time.
  No native AR APIs involved — this is what makes "AR on a link, no install" possible.
- **Rakhi art = illustrated 2D/layered PNGs**, not 3D models. A sculpted 3D asset pipeline is not
  achievable by tomorrow; a well-designed flat illustration that tracks and rotates with the wrist
  reads as magic without that cost. Revisit 3D later if this becomes a bigger thing.

## v1 scope (tonight)
1. **Sender flow:** landing page → pick 1 of ~3-4 starter Rakhi designs → optional name/note →
   generate a shareable link.
2. **Recipient flow:** open link → camera permission prompt (with plain-language privacy note) →
   point at wrist → Rakhi anchors and animates on (thread-tie shimmer) → screenshot-able moment →
   sender's name/note shown.
3. Mobile-first only (this is a phone-camera experience; desktop can show a "open on your phone"
   message).

## Explicitly NOT in v1 (say so if asked, don't silently build it)
- Account system / saved history of sent Rakhis.
- 3D modeled/cloth-simulated Rakhis.
- Multiple hands / tying-animation physics.
- Native app version.

## Assets
Starter Rakhi illustrations generated via `generate_image` (Unravel Powers), stored in `assets/`.
Sushreya can swap in her own art later — nothing in the AR anchoring logic should assume a specific
image beyond "transparent PNG, wrist-anchored, front-facing."

## Security
Hand-tracking runs entirely client-side (WebAssembly in-browser) — camera video is never uploaded
or stored anywhere. State this plainly on the camera-permission screen. Run the mandatory security
scan before any deploy, same as every other live link out of this project (per
`~/Desktop/AI project/process/DESIGN_TO_CODE_INITIALIZATION.md` Phase 0 spirit, scoped down since
this isn't a dev-handoff project).

## Deploy
Deploy via Unravel Powers (`mcp__unravel-powers__deploy`), same as other live-preview links from
this account. Track link expiry — this is meant to be alive through the Rakhi weekend, not forever.
