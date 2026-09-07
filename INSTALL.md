# VastuLens — Install On Your Phone, or Deploy to GitHub Pages

## Latest round — fully fused AR overlay + AI detection overhaul
Two things, both on the Lens tab:

**AI object detection was silently starving, not "broken."** The old
code required 50%+ confidence and only recognized 10 specific COCO
object classes across 4 room types — real phone-camera conditions
(angle, distance, partial occlusion, lighting) routinely score
legitimate objects at 35-50%, so most of what the camera actually saw
was thrown away before you ever found out about it. Also worth saying
plainly: there is no official, verified TensorFlow.js model that
classifies "what type of room is this" the way COCO-SSD classifies
"is there a chair here" — bolting on some unverified third-party "room
classifier" from a random CDN would look like a fix while being fake,
so that's not what this is. Instead:
- Confidence floor lowered to 35%, and matching now covers ~20+ COCO
  classes (bowls, cups, cutlery, dining tables, chairs, potted plants,
  vases, clocks, books, toothbrushes, hair dryers, and more), each
  weighted by how strong a real signal it actually is for a given room
  — one detected oven is worth more than one detected cup, and several
  weaker objects can now add up to a confirmed room the way one strong
  object alone used to be required to.
- **You can now see exactly what the model sees, every ~1.5s, live** —
  a "Currently seeing: chair (72%), potted plant (58%)" line updates
  continuously regardless of whether a room type has been confirmed
  yet. Previously the banner said nothing at all until a full verdict
  was confirmed, which is exactly why this looked like total silence
  even in frames where it was correctly detecting real objects.

**Fully fused AR overlay, live, on the camera itself:** the compass
tick-ribbon and sector labels were already drawn directly over the
video (this app already had a real, working AR compass — not a
separate screen); what it was missing was the object-detection half.
Added: a live bounding box + confidence label drawn around every
object the model currently sees, positioned correctly against the
actual visible video (accounting for `object-fit:cover` cropping,
which a naive scale-by-ratio would have misaligned), color-coded
green for high-confidence detections. Combined with the existing
compass ticks, sector labels, heading readout, and the Vastu verdict
banner — all layered directly over the live feed — this is now one
coherent live AR HUD rather than a camera feed with separate text
blocks stacked below it.

## Previous round — the layering mess, OCR hangs, and compass sync
Your screenshot showed the real bug clearly: three unrelated coordinate
systems (the photo, the compass zone-overlay pie, and your typed room
boxes) were all being drawn on the same canvas with no relationship to
each other, producing exactly that unreadable overlap. Fixed that and
the underlying gaps behind your other three complaints:

- **The overlap is gone.** When a floor-plan photo is shown, the 2D
  view is the photo plus its own OCR markers and zone overlay only.
  When you hide the photo, the view switches to your typed room-box
  diagram instead. They're never drawn on top of each other again.
- **"Pin and correct" hanging.** Root cause: nothing stopped more than
  one OCR pass from running at once — rotating/flipping quickly, or
  tapping the OCR button again while a read was still in progress,
  could stack up multiple Tesseract runs competing for the same CPU
  core, which is exactly what reads as the app freezing. There's now a
  guard: only one OCR pass runs at a time, with a clear "already
  reading, please wait" message instead of silently piling up.
- **A real "change direction" control.** Every tagged room (OCR,
  manual, or compass) now has a **Remove** button right next to it —
  previously the only way to change one was to add a new tag that
  superseded the old (which stayed on screen, struck through, possibly
  reading as "I can't get rid of the wrong one").
- **The actual "sync with the compass" feature, using your phone's real
  sensor:**
  - **Calibrate photo's north** — point your phone the way the photo's
    top edge actually faces in real life and tap once; every OCR word
    and manual tap on that photo is now measured against your real
    compass reading instead of assuming the image was drawn north-up.
    Rotating the photo afterward keeps the calibration in sync
    automatically; flipping it (mirroring) requires recalibrating,
    since a mirror can't be corrected with a simple rotation.
  - **Stand at the Brahmasthan and point** — a second, photo-free way
    to tag a direction: stand at/near your home's center, pick what
    you're facing (entrance, kitchen, etc.), point the phone at it,
    and tap Save. This records your phone's live compass bearing
    directly, with no image, no tap position, no orientation
    assumption at all. For "is this actually the kitchen" as a second
    opinion before you tag it, use the Lens tab's existing live-camera
    AI object detection, then come back and save the direction here.
  Both calibration and stand-and-point reuse the exact same
  battle-tested compass state the Dial tab already keeps live in the
  background (it's continuously updated regardless of which tab is
  open) — no separate sensor plumbing was needed for this, which is
  also why it works immediately.

## Previous round of fixes
Ran the app's own logic through an automated test harness (44 checks:
compass sector math, English+Dutch OCR label mapping, dimension
parsing, PDOK coordinate parsing, bearing geometry, and the full
room→direction→Vastu-status pipeline using the app's built-in sample
floor plan) plus a manual code-review pass. Found and fixed two real
bugs on top of the previous round of fixes:
- **Removed a duplicate, dead copy of the dimension-parsing function**
  (`parseDimensionText` was defined twice; the first copy was silently
  unreachable dead code — harmless in practice since JS always used the
  second, but confusing and a genuine defect to clean up).
- **Removed a hardcoded, non-required API key on the NOAA magnetic-
  declination lookup** used by the Dial tab's true-north correction.
  That parameter is for NOAA's own usage-tracking survey, not
  authentication, isn't documented as required, and every published
  working example of that endpoint omits it — removing it eliminates a
  needless point of failure independent of this app.
- **Verified the Center tab's Dutch address lookup (PDOK)** against
  both requested test addresses — Saltholm 9, 2133 EA Hoofddorp and
  Maria Snelplantsoen 40, Amsterdam — by confirming both are real,
  registered Dutch addresses and that the query/response parsing
  (`parseWktPoint`, the free-text query construction) matches PDOK's
  current, documented API shape exactly. This sandbox's own network
  policy blocks live calls out to api.pdok.nl for me directly, so this
  was verified by API-contract review rather than a live call from
  here — worth actually tapping "Look up address" for both on your
  phone to confirm the live round-trip too.

## Previous round of fixes
- **Fixed the app-wide slowness/hanging.** The compass tab was running a
  60-times-a-second background render loop that never stopped, even while
  you were on a different tab (e.g. running OCR on the Design tab). It now
  only runs while the Dial or Lens screen is actually the one on screen.
- **Fixed OCR silently missing labels like "kitchen"/"entrance."** OCR now
  runs on a contrast-stretched version of your image (helps faint print
  and colored/handwritten annotations alike), shows live progress instead
  of a static message, and — this is the important part — now tells you
  what text it *did* see but couldn't map to a room type, instead of
  dropping it with no trace. If a label still doesn't map, tap the plan to
  mark it yourself.
- **Fixed the manual point-and-correct feature being unreachable.** It
  already existed in the code, but the tappable plan view only appeared
  after clicking "Build plan & 3D model" — which itself required manually
  adding room boxes first. It now appears as soon as you upload a floor
  plan photo, with no unrelated steps in the way.
- **Added offline/installable support** via a service worker (only active
  when hosted over HTTPS — GitHub Pages, Option B below) so a repeat visit
  still opens and still has the compass, Brahmasthan, Matrix, and NL Rules
  tabs working with no signal.

This folder has exactly 4 files — everything the app needs, nothing
else. It works both ways: installed straight from your phone's
storage (no hosting), or deployed to GitHub Pages if you want the
camera-based Lens tab working too. Same files, either path.

## Option A: Install directly on your phone (2 minutes, no hosting)

1. **Get the file onto your phone.** Send `index.html` to yourself any
   way you'd send a photo — email attachment, WhatsApp/Telegram "file"
   (not as a photo), Google Drive/Dropbox, USB cable, whatever's easiest.
   Keep the other 3 files (`manifest.json`, `icon-192.svg`,
   `icon-512.svg`) in the **same folder** as `index.html` on your phone
   — they're small icon/name files the app looks for next to itself.
2. **Open `index.html` with Chrome** (tap it in your Downloads app, or
   wherever you saved it — choose "Open with Chrome" if asked).
3. Tap the **⋮** menu in Chrome → **Add to Home Screen**. You now have
   a normal app icon that opens VastuLens full-screen, like any
   installed app.

## Option B: Deploy to GitHub Pages (free, ~5 minutes, camera works)

These exact 4 files are all you need — no build step, no extra
config files required.

1. Create a new repository on **github.com** (public repos get free
   Pages hosting).
2. On the repo page, **Add file → Upload files**, and drag in all 4
   files (`index.html`, `manifest.json`, `icon-192.svg`,
   `icon-512.svg`) — keep them at the repo root, not in a subfolder.
   Commit.
3. Go to **Settings → Pages**. Under **Build and deployment → Source**,
   choose **Deploy from a branch**. Under **Branch**, choose **main**
   and **/ (root)**, then **Save**.
4. Wait 1-2 minutes, then refresh that Settings page — it'll show a
   live URL: `https://<you>.github.io/<repo-name>/`.
5. Open that URL on your phone and **Add to Home Screen** the same way
   as Option A.

Optional nicety, not required: GitHub Pages runs everything through
Jekyll by default, which is harmless here (your 4 files aren't
anything Jekyll needs to transform) but adds a moment to each deploy.
If you want to skip that, add one more empty file to the repo named
`.nojekyll` (Add file → Create new file → name it `.nojekyll` → leave
it empty → commit).

## What works which way

Verified in a real browser before packaging this:

- **Dial, Center (Brahmasthan), Matrix, Design, and NL Rules tabs work
  fully either way** — direct from your phone's storage, or hosted.
- **The Lens tab's live camera overlay only works with Option B
  (hosted).** This isn't a bug or a missing file — Chrome only allows
  camera access on a page loaded over HTTPS or `localhost`, never on a
  plain local file, for anyone's page, not just this one. GitHub Pages
  serves over HTTPS automatically, which is what unlocks it.
- The compass itself (Dial tab) needs your phone's real orientation
  sensor either way — it won't show a reading in a desktop browser.

## Two features fetch a library from the internet at the moment you use them

The **3D view** (Design tab) and **AI room detection** (Lens tab) each
load an external library — Three.js and TensorFlow.js — from a CDN
right when you open them, not from a file in this folder. That's
normal (same as a site loading a font from Google), and it doesn't
need adding to this folder or to your repo. It does mean those two
specific features need the visiting phone to have an internet
connection *at that moment*, separately from the page just being
hosted. If there's no connection when you open them, they show a
plain fallback message instead of failing silently — everything
else (camera passthrough, compass, Brahmasthan, Matrix, NL Rules)
runs entirely on-device with no external calls at all once the page
itself has loaded.

## Updating it later

**Option A:** replace `index.html` on your phone with a newer version
the same way you installed it, then Add to Home Screen again for a
fresh icon.

**Option B:** edit the file(s) in the GitHub repo (or upload a new
version) and commit — GitHub Pages redeploys automatically within a
minute or two, no separate publish step.

