# VastuLens — Install On Your Phone, or Deploy to GitHub Pages

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

## Updating it later

**Option A:** replace `index.html` on your phone with a newer version
the same way you installed it, then Add to Home Screen again for a
fresh icon.

**Option B:** edit the file(s) in the GitHub repo (or upload a new
version) and commit — GitHub Pages redeploys automatically within a
minute or two, no separate publish step.

