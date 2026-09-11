# VastuLens — Install On Your Phone, or Deploy to GitHub Pages

## Latest round — voice commands now actually stop
Root cause: the only on/off control for voice commands lived on the
Dial and Lens tabs — so switching to Design, Matrix, or Guide left the
microphone listening in the background with no visible indication it
was still on and no button anywhere in sight to turn it off. Fixed
two ways:
- **Voice commands now stop automatically** the moment you navigate
  to any tab other than Dial or Lens — not paused-and-resumed, fully
  stopped, so coming back to Dial/Lens always shows an honest "Off"
  rather than a silently-still-listening mic.
- **The on-state is now unmistakable**: the toggle button turns red
  and gently pulses, with its label reading "🔴 Voice commands: On
  (tap to stop)" instead of a plain "On" that was easy to miss.
Verified with a real test (not just "does it crash"): forcing voice
commands into their "on" state and then simulating a tap to the
Matrix tab confirms the button reverts to "Off" — the actual behavior
being fixed, checked directly.

## Latest round — the main compass dial redesigned to match your photos
The Dial tab's actual compass face (not just the readout text below it)
is now a set of concentric rings, closest first to what's in your
reference photos:
- **Outer ring**: degree tick marks, numbered every 30°.
- **16-point compass names** (N, NNE, NE, ENE...).
- **The 32-point Vastu Purusha pada ring** (VASTU_MANDALA_32 — the
  same researched, source-cited data added last round), color-coded
  by cardinal side.
- **Innermost**: Brahma's cross at the centre with Aryama/Vivaswan/
  Mitra/Bhudhar named around it, plus the live current-sector
  highlight and heading ticks — this inner ring is exactly the old
  dial's entire display, just resized to sit inside the new outer
  rings rather than filling the whole canvas, so nothing about its
  tested behavior changed, only where it's drawn.
The whole card turns together as you turn the phone, exactly as
before — only the fixed pointer at the very top stays unrotated. The
canvas grew from 280px to 340px (with responsive CSS scaling on
narrower phones) to fit this much text without it turning to mush;
32-pada labels are abbreviated to 4 letters where the full name won't
fit at a legible size, with the full name always available in the
readout text directly below the dial and via the Center tab's
full-size tappable version.

**Caught by the test suite, again:** two things this time. First, the
new dial code initially crashed the moment `drawDial` was called
directly (a gap in the *test harness's* canvas stub, not the app —
`createRadialGradient` had never been stubbed because no earlier test
called `drawDial` directly; fixed the stub, not the app). Second, an
intermittent single test failure that didn't reproduce on a second or
third run — logged as a flake rather than quietly ignored, and
confirmed stable at 108/108 across three consecutive runs before
shipping.

## Previous round — real Vastu Purusha Mandala research, wired everywhere
You asked for the Dial to be as informative as your physical compass
product, backed by real research from your three uploaded manuscripts.
Here's exactly what was and wasn't possible, stated plainly:

**What got checked, and how:**
- **Brihat Samhita** (P.S. Sastri's 1946 translation) — this PDF has a
  genuine OCR text layer, so it could be searched directly. Found
  Adhyaya 53 ("Vastuvidya"), which gives the full 45-deity enumeration
  (13 inner + 32 outer), the quadrant groupings, and which part of the
  House-God's own body each pada corresponds to.
- **Vishwakarma Vidya Prakash** — restates the same 45 deities, same
  order, same groupings, in places almost verbatim. Used as a direct
  cross-check on the Brihat Samhita reading.
- **Samarangana Sutradhara** — this PDF is a pure page-scan with no
  text layer at all (confirmed via `pdffonts`), so it couldn't be
  searched or quoted. Its published synopsis of contents was checked
  visually instead: Ch. 11 covers the 64/81/100-pada site plans and
  Ch. 14 is specifically about assigning deities to the House-God's
  limbs — independent confirmation of the same tradition, honestly
  labeled as "checked for context," not "extracted."

**What got built from that research:**
- **Dial tab**: shows the Vastu Purusha pada name (e.g. "Sikhi, E
  side") plus its body-limb correspondence whenever the compass is set
  to 32 points.
- **Center tab**: a new, tappable **81-pada Vastu Purusha Mandala
  diagram** — Brahma at the centre, the 8 immediately surrounding
  deities, and the 32 outer names, drawn as equal segments and aligned
  precisely onto this app's own existing N1-8/E1-8/S1-8/W1-8 compass
  convention (which already matched your physical compass photos).
  Tap any segment for its deity and limb correspondence.
- **Matrix tab**: the live heading readout now always shows the
  current Vastu Purusha pada alongside the 8-direction reading.
- **Design tab**: every tagged room (OCR, manual, compass, or voice)
  now shows which pada it falls under.
- **Lens AR overlay**: the live compass tick labels show pada names
  too, when in 32-point mode.
- **Guide tab**: a full sources card, naming exactly which manuscript
  supports which claim, and stating the one genuine interpretive
  choice honestly — the 81-square mandala is a *square* diagram;
  drawing it as 32 equal wedges around a circular compass (same as
  the paper compass this was modeled on) is a reasonable alignment,
  not a claim that the two geometries are mathematically identical.
  This stays a clearly separate, complementary layer from the existing
  8-direction deity/planet table (Kubera/Ishaan/Indra/Agni) used by
  the Matrix tab's checker — the two are never merged into one
  confusing table.

**Caught by the test suite before it shipped:** the initial diagram
code crashed on load — it referenced the mandala data before that
data had finished initializing (a classic "temporal dead zone" bug),
which would have silently broken the whole page. Also caught: two
compass-orientation mistakes in the diagram's own geometry (south and
west sides were initially swapped/reversed). All fixed, and the app's
test suite (now 104 checks) verifies the actual drawn geometry — not
just that the code runs, but that the true-north cell, the NE corner
cell, and all 32 names land where they're supposed to.

## Previous round — AI detections now stick, plus voice commands
**The real bug behind "AI detection isn't sticking":** there was no
path at all from a live AI detection into permanent storage. The
detection banner was always a transient read — it recomputed every
~1.5s and quietly reset after 12s of losing sight of the object.
Nothing about it was ever being saved anywhere, so of course it never
"stuck" — there was no save step to begin with. Fixed:
- A **"Save this detection"** button now appears whenever AI confirms a
  room, writing straight into the same permanent store (`profile.
  areaDirections`) that Dial/Matrix/manual-tag all already share.
  Once saved, it stays saved regardless of what the camera sees
  afterward — nothing auto-overwrites a save; only an explicit new
  save or a voice command changes it.

**Voice commands**, using the standard Web Speech API already built
into Chrome on Android (no external service, nothing new to load):
- A **Voice commands** toggle on both the Lens and Dial tabs.
- **"Save kitchen" / "save toilet" / "tag entrance"** — saves that room
  at whatever direction the phone is facing right now.
- **"Confirm" / "yes" / "that's right"** with no room named — locks in
  whatever the AI overlay currently shows (the exact "it changed away
  from sink, tell it to stick" case), at your current heading.
- **"Remove toilet" / "clear kitchen"** — removes a saved tag by voice.
- Works in Dutch too for the room name itself ("bewaar de keuken",
  "verwijder de badkamer") — it reuses the exact same English+Dutch
  room-name matching the OCR pipeline already uses, one vocabulary for
  the whole app. Command verbs (save/remove/confirm) also now
  recognize a few common Dutch equivalents (opslaan, bewaar, verwijder,
  klopt, ja).
- Spoken confirmation talks back ("Saved kitchen facing east.") via
  speech synthesis, with the same text also shown on screen for silent
  environments or unsupported browsers.

**The saved-spaces map now shows ideal directions, not just current
status** — every space you've tagged (Design tab, Dial/Matrix "stand
and point", Lens AI+voice — all of it, unified, since they all write
to the same store) that isn't already in an ideal spot now shows which
of the 8 directions actually would be ideal for that room type, pulled
from the exact same rule table the verdict itself comes from — so it
can never disagree with the verdict shown right next to it.

## Previous round — fully fused AR overlay + AI detection overhaul
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

