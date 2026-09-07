# VastuLens — Install On Your Phone, or Deploy to GitHub Pages

## What changed since the previous version (full test pass)
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

