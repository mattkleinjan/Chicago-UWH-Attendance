# Chicago UWH — Attendance & Dues Tracker

A single-page web app for tracking underwater hockey practice attendance and
who owes what. Works as an installable home-screen app on iOS (and any other
device), with data synced live across everyone who has it open. This app was
built using Claude.

**Live site:** https://mattkleinjan.github.io/Chicago-UWH-Attendance/

## Features

- **Mark attendance** — open to anyone, no passcode needed. Pick a date, tap
  who showed up.
- **Balances** — everyone can look up their own balance via a name dropdown.
  Admins (passcode) see the full list plus tools to record payments and add
  one-off charges (tournaments, equipment, etc).
- **Student pricing** — a separate practice fee for members flagged as
  students.
- **Top Attendance** — a leaderboard of practices attended for the current
  calendar year, with ties handled properly (equal attendance = equal rank).
- **History** — a log of every past practice, who attended, and (for admins)
  the ability to delete an entry.
- **Bulk import** — admins can paste a block of text to backfill past
  attendance from before the club started using this app.
- **Live sync** — all data is stored in Firebase Realtime Database, so every
  device sees the same information immediately.

## Admin passcode

Most editing actions (recording payments, adding charges, editing the
roster, editing fees, deleting history, exporting CSVs) are hidden behind a
simple passcode gate. Tap **"Unlock admin features"** in the header to enter
it; tap **"Lock"** to hide those controls again on that device.

**This is a soft gate** The Firebase rules are currently
open (anyone can read/write), so the passcode only prevents casual mistakes
and snooping through the app's UI — it doesn't stop someone from editing the
database directly. 

## Tech stack

Plain HTML/CSS/JavaScript — no build step, no framework, no dependencies to
install. Firebase is loaded via CDN script tags. This means you can edit
`index.html` directly in GitHub's web editor and redeploy just by committing.

- **Hosting:** GitHub Pages, serving `index.html` from the `main` branch
- **Data:** Firebase Realtime Database (free Spark plan)
- **Sync:** `firebase-database-compat` SDK, listening on the `chicago-uwh`
  path for live updates

## Deploying a change

1. Edit `index.html` (either locally, or directly in GitHub's file editor)
2. Commit the change
3. Wait ~1 minute for GitHub Pages to redeploy
4. Hard-refresh the site to pick up the new version (browsers cache
   aggressively)

## Firebase setup (for reference / if rebuilding from scratch)

Project: `chicago-uwh-attendance`

1. [Firebase console](https://console.firebase.google.com) → Realtime
   Database → Data tab has the database URL
2. Rules are scoped to a single path so they don't expire like default test
   mode does:
   ```json
   {
     "rules": {
       "chicago-uwh": {
         ".read": true,
         ".write": true
       }
     }
   }
   ```
3. The web app config (`firebaseConfig`) lives near the top of the
   `<script>` block in `index.html`

## Backups

A scheduled GitHub Action (`.github/workflows/backup.yml`) pulls a full copy
of the database once a week and commits it into the `backups/` folder as a
dated JSON file. No setup needed beyond having the workflow file in the repo
— check the **Actions** tab to see recent runs or trigger one manually.

If data is ever accidentally deleted or corrupted, restore by copying the
contents of the most recent `backups/backup-YYYY-MM-DD.json` file back into
the Firebase console (Realtime Database → Data → **⋮** → Import JSON, at the
`chicago-uwh` path).

## Known limitations

- No real user accounts — the "unlock" and per-player balance lookup are
  both convenience features, not authentication.
- Firebase database rules are open.
- Historical fee amounts aren't preserved for imported/bulk sessions —
  imports use whatever the fee is set to at the time of import.
  
