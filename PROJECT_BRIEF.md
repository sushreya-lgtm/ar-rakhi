# dhaaga (AR Rakhi) — Project Brief

**Status:** Live for Rakhi 2026-08-28. Ongoing — more designs and fixes will keep landing after launch.

## What it is
Send an AR Rakhi that ties itself onto the recipient's wrist live, through their phone camera —
no app install. Sender picks a design + adds a note on the home page, gets a shareable link;
recipient opens it on their phone, points the camera at their wrist, and the Rakhi anchors and
ties on live via in-browser hand tracking.

## Stack (locked)
- Plain static web app, no build step, no native app, no Expo.
- AR = `@mediapipe/tasks-vision` HandLandmarker running client-side via WebAssembly. Detects wrist
  + hand landmarks from the live camera feed; a canvas overlay anchors, rotates, and scales the
  Rakhi + tying-hands illustration to match, frame by frame.
- Rakhi art = her real illustrated PNGs (not AI-generated, not 3D).
- **Zero backend, by design.** All sender data (recipient name, note, sender name, design choice)
  lives entirely in the shareable link's URL query string — no account, no database, no server
  logic. This was reconsidered once (to fix long notes → long links) and deliberately reverted:
  she chose to keep it simple over adding Firestore. Don't resurface that unless she raises it.

## Pages
- `index.html` — logo, hero, steps, Rakhi picker, note form, link generator, footer.
- `rakhi.html` — the AR camera experience (scan → tie-in animation → steady state → save/replay).
- `privacy.html` — honest policy matching the app's actual (zero) data collection.

## Current designs
Two: **Floral & Radiant**, **Sleek Evil Eye**. A "More designs on the way" note sits under the
picker on the home page — more will be added over time, no code changes needed beyond adding a
new card + composite asset + `DESIGNS` entry in `rakhi.html`.

## Deploy
GitHub Pages, `sushreya-lgtm/ar-rakhi` (personal GitHub, kept off work infra deliberately — this
is a personal gift project). HTTPS required for camera access, which Pages gives for free.
Live at **https://sushreya-lgtm.github.io/ar-rakhi/**

Push to `main`, GitHub Pages auto-deploys. Note: Pages sits behind a CDN with a few minutes of
edge caching — if a change doesn't appear to be live, verify with `curl` before assuming it's a
code bug.

## Not in v1 (revisit later if it becomes worth it)
- Account system / saved history of sent Rakhis.
- 3D modeled Rakhis.
- More than one Rakhi design entry point per link (one design per link, chosen at send time).

## Known gap
Everything is verified via a `?debug=1` fabricated-hand-pose harness (see `rakhi.html`) plus real
device screenshots/recordings she's sent — there's no automated way to click through the native
camera-permission dialog, so any new AR-facing change should still get a real phone check before
being called fully done.
