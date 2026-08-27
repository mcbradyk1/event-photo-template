# Spin up a new event (~5 minutes)

Do this each time you take on a new event. Assumes the **one-time setup** in
[SETUP.md](SETUP.md) is done (template repo exists, shared service account +
PAT + Apps Script are ready).

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

## 4. Point the upload broker at this event

- **Per-event Apps Script copy:** deploy a copy of `code.gs`, set its
  `PHOTO_FOLDER_ID` to this event's folder ID and `GITHUB_REPO` to
  `mcbradyk1/eventname`, add `http://…github.io` (your Pages origin) to
  `ALLOWED_ORIGINS`, deploy, and copy the `/exec` URL.
- **Central hub Apps Script:** just add this event's `eventId → folderId` to the
  allow-list; the `/exec` URL stays the same for all events.

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

Both are independent — deleting one event never affects another. The shared
service account, PAT, and Apps Script stay put for the next event.

## Per-event checklist (copy/paste)

```
[ ] Use template → repo named "eventname"
[ ] Settings → Pages → Source: GitHub Actions
[ ] Drive folder created + shared with service account
[ ] Folder ID copied
[ ] Secrets: GDRIVE_FOLDER_ID, GDRIVE_SA_FILE (+ booth pair if used)
[ ] Apps Script pointed at this folder/repo → /exec URL
[ ] config.js: eventName, subtitle, eventDate, siteDomain, colors, endpoint
[ ] Deploy finished, first sync run, test photo uploaded
[ ] photoboothSign.html printed
```
