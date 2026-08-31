# dhaaga (AR Rakhi) — Project Context & Retrospective

**Status:** Live, launched Raksha Bandhan 2026-08-28. Ongoing polish since.
**Live URL:** https://dhaaga-send-an-ar-rakhi.vercel.app
**Repo:** `sushreya-lgtm/ar-rakhi` on GitHub (personal account, deliberately kept off work infra)

## What it is

Send an AR Rakhi that ties itself onto the recipient's wrist live, through their phone camera —
no app install. Sender picks a design + writes a note on the home page, gets a shareable link;
recipient opens it, points the camera at their wrist, and the Rakhi anchors and ties on live via
in-browser hand tracking (MediaPipe HandLandmarker, WebAssembly, fully client-side).

## Stack

- Static HTML/CSS/JS, three pages (`index.html`, `rakhi.html`, `privacy.html`), no build step,
  no framework, no backend, no database. All sender/recipient data lives in the link's URL query
  string — nothing is stored anywhere.
- Hosted on **Vercel** (migrated from GitHub Pages on launch day for real cache control — GH
  Pages edge-cached for 10min, Vercel serves `max-age=0, must-revalidate`). GitHub Pages
  deployment (`sushreya-lgtm.github.io/ar-rakhi/`) still exists but is not the canonical URL.
- Google Analytics (GA4, `G-VN25QNCQLH`) on the home page only — tracks page views, nothing else.
  Disclosed in `privacy.html`.
- Fonts: Martian Mono (site default), Cormorant Garamond italic (headings/emphasis/notes),
  Rozha One (the "Raksha Bandhan" reveal-screen lettering — chosen specifically for its
  Devanagari-adjacent, full/traditional letterforms).

## Deploy process (manual — no CI/CD)

```
git add -A && git commit -m "..." && git push
npx vercel --prod --yes
```
Then **always** re-point the alias — Vercel keeps regenerating `ar-rakhi.vercel.app` as a
side effect of deploying from this directory (tied to the git repo name, not the project's
actual name; root cause never fully confirmed, just worked around every time):
```
npx vercel alias set <new-deployment-url> dhaaga-send-an-ar-rakhi.vercel.app
npx vercel alias rm ar-rakhi.vercel.app --yes
```
The Vercel CLI auth token has occasionally returned a transient "Not authorized" on first
attempt — retrying the same `vercel --prod --yes` immediately has always resolved it.

## LOCKED INVARIANTS — do not change without reading this section

**The tying-hand animation's rotation system** (`rakhi.html`, search `FIXED_HAND_ROTATION`).
This broke and got "fixed" wrong **four separate times** before landing correctly. The rule,
also written directly in the code:

- `hands-pair.png` is one flat image containing both hands drawn as a "V" converging near the
  top. That V only reads as one cohesive gesture within a few degrees of **0° (upright) or 180°
  (upside-down)**. At any other rotation — including 45°, which was tried and failed — the two
  hands visually split apart along the arm.
- The hand's rotation is `FIXED_HAND_ROTATION = 0`, a **constant**, never derived from live
  wrist tracking. This is deliberate: no rotation-matching scheme can make a flat 2D asset look
  right at every live angle, so the fix is to never attempt it.
- The travel direction (`elbowTravelDir`) is **derived from that same fixed rotation** — 0°
  naturally maps to "enters from the bottom," which is both the safe rotation AND a direction
  she approved. Don't decouple these two again.
- The Rakhi + string are drawn at a **blended rotation** (`handOffRotation` in
  `drawTyingAnimation`): during ENTER they match the hand's fixed angle (so it looks genuinely
  carried, not floating independently pre-rotated), then snap to the true live wrist angle over
  `SNAP_MS` (~350ms) right as it's placed, timed with the existing haptic pulse. This is the
  actual mechanism for "it detects the wrist and turns to fit."
- If any of this needs to change (e.g., she wants a different entry direction), the constraint
  above is non-negotiable — pick a new fixed rotation only if it stays within ~45° of 0 or 180,
  and re-derive the travel direction from it via the same formula, don't hardcode a separate one.

**DOM ↔ canvas capture drift.** "Save this moment" redraws the reveal screen from scratch onto
canvas (`drawCaptureOverlay`), because `canvas.toBlob()` can't see separate DOM/CSS overlays.
This has drifted out of sync with the live CSS **multiple times** (dangler top position, caption
clearance, desktop-only padding/font sizes). Any future visual change to `#revealMark`'s CSS
needs the matching canvas code checked — grep `drawCaptureOverlay` and read the whole function,
don't assume a CSS-only change is safe.

**Mobile vs. desktop are different designs on purpose**, not the same thing scaled:
- Danglers (`assets/dangler-ornaments.png`): **desktop only**. Removed from mobile because two
  colorful ornaments + a colorful board + the colorful Rakhi itself read as too busy on a small
  screen. Mobile gets extra subtle sparkles instead (Rakhi stays the visual hero). The image
  itself is only fetched on desktop-width viewports (`window.matchMedia`) — don't remove that
  gating even if you re-enable danglers on mobile, or it wastes bandwidth on phones for nothing.
- The reveal board (placard) uses `width: fit-content` (hugs its own text) with a responsive
  `clamp()` font size — not a fixed width. This was a real bug once: a fixed max-width + fixed
  font-size + `white-space: nowrap` doesn't shrink to fit, it just overflows uncontained.

## Known open items, not guessed at further

- **Wrist-size auto-detection accuracy**: fundamentally limited by MediaPipe's sparse 21-point
  hand model, not by anything in this app's code. Confidence/accuracy genuinely degrades at
  wrist angles the model wasn't trained as heavily on. Manual pinch-to-resize/drag-to-reposition
  exists specifically as the correction for this — it is not a bonus feature, it's the intended
  answer. Don't keep tuning `WRIST_WIDTH_RATIO` hoping for a universal fix; there isn't one.
- **Desktop board centering**: was reported as off-center once; switched centering from
  `left:50%/transform` to flex-centering on the parent (more robust technique either way), but
  the original root cause was never confirmed — the actual bug found and fixed at the same time
  (text overflow from a fixed-width cap) may have been the whole story, or may not have been.
  Watch for this recurring.
- **Canvas capture's dot-and-dash marks** (the small teal flourish next to "Happy") render at a
  fixed mobile size even in a desktop capture — a few pixels of a small decorative detail, flagged
  and consciously not chased further given diminishing returns.

## Retrospective — read this before a big feature push

**What caused the most wasted rounds**: the tying-hand animation, by a wide margin (see LOCKED
INVARIANTS above). The root pattern: fixes were verified by checking that the *math* was
internally consistent (same shift variable, same rotation feeding the same formula), which is
not the same thing as verified by seeing the result — and there is no browser available in this
environment, only reasoning and whatever evidence is provided. Confidently-stated fixes that
were actually still guesses happened repeatedly on this specific system.

**What fixed it**: two things. First, an explicit "don't build it, let's brainstorm it" pass
*before* writing code — the actual correct architecture got designed in conversation, not
through trial and error. Second, real screenshots/recordings of the specific broken *moment*
(not the aftermath/steady-state) — every fix made from real evidence landed correctly on the
first try; every fix made from theory alone needed at least one more round.

**For next time**: if something visual is wrong and the fix isn't obvious from reading the code,
say so explicitly and ask for a recording of the exact broken moment, or propose a brainstorm
pass, rather than shipping another confident-sounding guess. If a system has failed a visual fix
twice already, that's the signal to stop guessing, not to guess more carefully.
