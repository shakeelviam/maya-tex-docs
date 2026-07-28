# Maya Tex — Staff Document Collection (staging pipeline)

Staff never touch the master sheet. They fill a sleek English form on their phones; every
submission lands in a separate **staging sheet** as *Pending*. HR reviews it there, approves the
good ones, and syncs them into the master with one click — with a full before/after audit trail.
The master cannot be corrupted from the outside because the public form can only ever *append* to
the staging sheet.

```
 Staff phone ──▶ Form (GitHub Pages) ──▶ Web App ──▶ STAGING sheet (Submissions = Pending)
                                                          │
                                              HR reviews & sets Approved
                                                          │
                                        "Sync approved to master"  ──▶  MASTER sheet + audit log
```

**Files**
- `Staging.gs` — bound to the **staging** sheet. Runs the form's web app *and* the two HR menu actions.
- `index.html` — the form, hosted free on GitHub Pages.

All dates are **dd-mm-yyyy** throughout (sheet cells, review screen, audit log).

---

## 1 · Create the pieces

1. **Staging sheet.** Make a new, blank Google Sheet — call it *Maya Tex — Staging*. This is the only
   sheet staff data flows through. (The `Roster` and `Submissions` tabs get created automatically.)
2. **Drive folder** for the Civil ID copies; copy its ID from the URL
   (`drive.google.com/drive/folders/`**`THIS_PART`**).
3. **A shared secret** — one long random string; you'll paste it in two places below.

## 2 · Install the backend (`Staging.gs`)

1. Open the **staging** sheet → **Extensions ▸ Apps Script**. Delete the sample, paste all of `Staging.gs`.
2. Fill **CONFIG** at the top:
   - `MASTER_ID` is already your master's ID; `MASTER_GID` is already its first tab.
   - `SHARED_SECRET` → your random string.
   - `DRIVE_FOLDER_ID` → the folder from step 1.
3. **Deploy ▸ New deployment ▸** (gear) **Web app** · *Execute as*: **Me** · *Who has access*: **Anyone** ▸ Deploy.
   Approve the permission prompt (Sheets + Drive). Copy the **`/exec`** URL.
4. Reload the staging sheet once — a **Maya Tex HR** menu appears.

> Changed `Staging.gs` later? Deploy ▸ **Manage deployments** ▸ edit ▸ Version **New version** ▸ Deploy.
> Same URL, live code. Editing without redeploying does nothing — the classic gotcha.

## 3 · Publish the form (`index.html`)

1. Edit the two lines at the bottom of `index.html`:
   ```js
   const API   = "PASTE_YOUR_APPS_SCRIPT_EXEC_URL";   // the /exec URL
   const TOKEN = "CHANGE_ME_TO_A_LONG_RANDOM_STRING"; // same as SHARED_SECRET
   ```
2. New GitHub repo ▸ add `index.html` ▸ **Settings ▸ Pages ▸** deploy from `main` / root.
3. Share the resulting `https://<user>.github.io/<repo>/` link with staff.

---

## Running a collection round

1. **Maya Tex HR ▸ 1 · Refresh roster from master.** Pulls every current name, role, join date and any
   existing document values into the `Roster` tab. Do this at the start (and any time master changes)
   so the typeahead and the pre-filled values are current.
2. **Staff submit.** They search their name (as printed on the Civil ID), confirm it's them
   (designation + join date), fill/correct their documents, optionally attach the Civil ID copy, submit.
   Each one appends to `Submissions` as **Pending**.
3. **Review** the `Submissions` tab. Sanity-check a row, then set its **Status** dropdown to **Approved**
   (or **Rejected** to ignore). Nothing has touched master yet.
4. **Maya Tex HR ▸ 2 · Sync approved rows to master.** Applies every *Approved* row to the matching
   master row, formats dates dd-mm-yyyy, writes a before→after line per field into the master's
   `Form Submissions Log`, files the Civil ID copy link into a `Civil ID Copy` column, and flips each
   row to **Synced**. Re-runnable safely — only *Approved* rows are processed.

## Why this is the safe version

- **Master is write-protected from the outside.** The web app only appends to `Submissions`; it never
  opens the master. Worst case, someone spams staging — which you review before syncing.
- **Human gate.** Nothing reaches master without an HR *Approved*.
- **Never wipes data.** Blank fields are skipped on sync; only filled values overwrite.
- **Name collisions handled.** Each roster entry carries its exact master row, captured at refresh time,
  so two similar names can't cross-write. Sync also re-checks the name against the row and skips (with a
  note) if the roster has drifted — refresh and re-sync fixes it.
- **Real dates, dd-mm-yyyy.** Your expiry "days left" formulas keep working; IDs stored as text so no
  lost leading zeros.
- **Full audit trail** in the master's `Form Submissions Log`, plus the whole `Submissions` history in staging.

## Optional hardening
The form's shared secret lives in public JavaScript, so it deters bots/casual traffic, not a determined
person who finds the URL — but since that person can still only append to staging, master stays safe.
If you want tighter access to staging itself, deploy the web app under a separate Google account that
isn't shared on the master, or add a per-employee PIN step later (easy once every Civil ID is filled in).
Say the word and I'll add the PIN gate.
