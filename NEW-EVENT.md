# Spin up a new event (~5 minutes)

Do this each time you take on a new event. Assumes the **one-time setup** in
[SETUP.md](SETUP.md) is done (template repo exists, shared service account +
PAT + Apps Script `code.gs` are ready).

The result is a site at **`mcbradyk1.github.io/eventname`** with its own Drive
folder, fully isolated from every other event.

> Replace **`eventname`** below with the actual slug you want in the URL — keep
> it lowercase, no spaces (e.g. `smith-wedding`, `jones-grad-2027`).

---

## 1. Create the repo from the template

1. On the template repo, click **Use this template → Create a new repository**.
2. Owner: **mcbradyk1**. Repository name: **`eventname`** (this becomes the URL).
3. Create it, then in the new repo: **Settings → Pages → Source: GitHub Actions.**

The site will live at `https://mcbradyk1.github.io/eventname/` once it deploys.

## 2. Make the Drive folder

1. In your Google Drive, create a folder (name it after the event).
2. **Share it with your shared service account's `client_email`** (from SETUP
   step 2), role **Viewer**.
3. Open the folder and copy its **ID** from the URL:
   `drive.google.com/drive/folders/`**`<THIS_PART>`**.

> Optional booth: if this event has a photo booth, make a **second** folder the
> same way and note its ID too.

## 3. Add the repo secrets

In the new repo: **Settings → Secrets and variables → Actions → New repository
secret.** Add:

| Secret | Value |
|--------|-------|
| `GDRIVE_FOLDER_ID` | the folder ID from step 2 |
| `GDRIVE_SA_FILE` | paste the **same** service-account JSON you use for every event |

Booth only (if used): also add `BOOTH_DRIVE_FOLDER_ID` and `BOOTH_DRIVE_SA_FILE`
(the SA JSON is the same file again).

## 4. Deploy the upload broker (Apps Script) — ~3 min

You keep one master copy of `code.gs`. Each event gets its **own deployment** of
it so the events stay isolated. Follow this exactly:

1. Go to **[script.google.com](https://script.google.com) → New project** (or
   open your master `PhotoUploader` project and **Make a copy**).
2. Paste in your `code.gs`. Near the top, change **these two constants only**:

   ```js
   const PHOTO_FOLDER_ID = 'PASTE_THIS_EVENTS_FOLDER_ID';   // from step 2
   const ALLOWED_ORIGINS = ['https://mcbradyk1.github.io'];  // your Pages origin
   ```

3. **Project Settings (gear icon) → Script Properties → Add script property**,
   twice:

   | Property | Value |
   |----------|-------|
   | `GITHUB_PAT` | your shared PAT (from SETUP step 3) |
   | `GITHUB_REPO` | `mcbradyk1/eventname` |

4. **Deploy → New deployment → gear icon → Web app.** Set:
   - **Execute as:** *Me*
   - **Who has access:** *Anyone*
   - Click **Deploy**, authorize when prompted, and **copy the `/exec` URL.**

5. In the Apps Script editor, run **`installGalleryFlushTrigger`** once
   (Run menu → select the function → Run) to install the 5-minute debounce.

> The `/exec` URL is public by design (guests' browsers call it) but keep it out
> of screenshots. It goes into `config.js` next.

## 5. Edit `config.js` (the only file you touch)

Open `config.js` in the new repo and set:

```js
eventName:  "Smith & Jones",              // the big title
subtitle:   "Share Your Photos",
eventDate:  "June 14th, 2027",
siteDomain: "mcbradyk1.github.io/eventname",   // <-- match the repo name!

theme: { primary: "#7a8f6a", secondary: "#8b7db8" },   // pick two colors

features: { photoUpload:true, guestGallery:true, photobooth:false },

endpoints: { photoUpload: "https://script.google.com/…/exec" },  // from step 4
```

Commit. That rebrands every page and wires up uploads.

### Changing fonts (optional)

The default titles use **Alex Brush** (bundled locally in `fonts/`). To use a
different font for a specific event, pull it from **Google Fonts by name** — no
files to download. In `config.js`'s `theme` block:

1. Add the family name(s) to `googleFonts`.
2. Reference them in `scriptFont` (titles) and/or `bodyFont` (everything else).

```js
theme: {
  googleFonts: ["Playfair Display", "Lora"],          // pulled from Google Fonts
  scriptFont:  "'Playfair Display', Georgia, serif",  // titles
  bodyFont:    "'Lora', Georgia, serif",              // body text
  // ...colors unchanged
}
```

**Two rules:** (a) any family named in `scriptFont`/`bodyFont` must also appear
in `googleFonts` — *except* system fonts like Georgia/Arial, which need no
entry; (b) spell the family exactly as shown on
[fonts.google.com](https://fonts.google.com). Leave `googleFonts: []` to keep
the local Alex Brush default (zero extra network requests).

**Wedding-friendly pairings** (title / body — copy a row):

| Vibe | `scriptFont` (title) | `bodyFont` (body) | `googleFonts` |
|------|----------------------|-------------------|---------------|
| Default (bundled) | `'Alex Brush', cursive` | `'Georgia', serif` | `[]` |
| Classic & elegant | `'Playfair Display', serif` | `'Lora', serif` | `["Playfair Display","Lora"]` |
| Romantic script | `'Great Vibes', cursive` | `'EB Garamond', serif` | `["Great Vibes","EB Garamond"]` |
| Modern & clean | `'Cormorant Garamond', serif` | `'Montserrat', sans-serif` | `["Cormorant Garamond","Montserrat"]` |
| Soft & handwritten | `'Parisienne', cursive` | `'Nunito Sans', sans-serif` | `["Parisienne","Nunito Sans"]` |
| Bold & timeless | `'Cinzel', serif` | `'Crimson Text', serif` | `["Cinzel","Crimson Text"]` |

## 6. Go live

1. The **Deploy** workflow runs on push; wait for it to finish (Actions tab).
2. Kick a first sync: **Actions → "Sync Guest Gallery" → Run workflow.**
3. Open `mcbradyk1.github.io/eventname` and upload a test photo to confirm the
   whole loop works.
4. Print the sign: open **`photoboothSign.html`** and print it — the QR points
   at `mcbradyk1.github.io/eventname`.

---

## Handing off & deleting later

When the event's over and you've given the couple their photos:

1. **Delete the Google Drive folder** (removes the originals).
2. **Delete the GitHub repo** (removes the site instantly).
3. **Delete the event's Apps Script deployment** (or the whole script project).

All independent — deleting one event never affects another. The shared service
account, PAT, and master `code.gs` stay put for the next event.

## Per-event checklist (copy/paste)

```
[ ] Use template → repo named "eventname"
[ ] Settings → Pages → Source: GitHub Actions
[ ] Drive folder created + shared with service account
[ ] Folder ID copied
[ ] Secrets: GDRIVE_FOLDER_ID, GDRIVE_SA_FILE (+ booth pair if used)
[ ] Apps Script: copy code.gs, set PHOTO_FOLDER_ID + ALLOWED_ORIGINS
[ ] Apps Script: Script Properties GITHUB_PAT + GITHUB_REPO
[ ] Apps Script: deploy Web app (Me / Anyone) → copy /exec URL
[ ] Apps Script: run installGalleryFlushTrigger once
[ ] config.js: eventName, subtitle, eventDate, siteDomain, colors, endpoint
[ ] (optional) config.js: googleFonts + scriptFont/bodyFont
[ ] Deploy finished, first sync run, test photo uploaded
[ ] photoboothSign.html printed
```
