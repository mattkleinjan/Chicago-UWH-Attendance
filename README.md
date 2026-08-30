# Chicago UWH — Attendance & Dues Tracker

A single-page web app for tracking underwater hockey practice attendance and
who owes what. Works as an installable home-screen app on iOS (and any other
device), with data synced live across everyone who has it open. This app was
built using Claude.

**Live site:** https://mattkleinjan.github.io/Chicago-UWH-Attendance/

## Features

### Today (open to everyone, no passcode)
- Mark attendance for any date by tapping names on the roster.
- Once attendance for a date is saved, the checklist **locks** to prevent
  accidental changes — tap "Update attendance" to unlock it, make changes,
  then save again.
- **+ Add visitor** opens a popup to check in a one-off visiting player —
  either linking to a previous visitor (so they connect to their existing
  balance) or adding someone brand new on the spot.
- The default date always uses your device's local calendar day (fixed a
  bug where it could roll over to the next day depending on time zone).

### Balances
- **Everyone** can look up their own balance via a name dropdown — no
  passcode needed. A **"Visitors"** option at the bottom of that dropdown
  shows a read-only summary of every visitor's balance.
- Selecting a name shows sessions attended (all-time and this calendar
  year), itemized **other charges**, and **payment history** (most recent 5
  by default, with a "Show all" option).
- **Admins** (passcode) additionally see a full list of every member with
  tools to record payments, add one-off charges, and a **"See details"**
  button per person to view, edit, or delete individual charge/payment
  entries.
- Admin toolbar also has: **Export balances (CSV)**, **Recent payments**
  (a club-wide payment log with its own CSV export, useful for reconciling
  that payments were recorded accurately), **Manage roster**, and
  **Guests → Visitors**.

### Top Attendance
- A leaderboard of practices attended for the current calendar year, open
  to everyone. Ties are handled properly — equal attendance counts share
  the same rank/medal instead of being split alphabetically.

### Roster (admin only, reached via "Manage roster" on Balances)
- Add/remove members, toggle student pricing per person.
- **Bulk import past attendance** — paste a block of text
  (`YYYY-MM-DD: Name, Name, Name` per line) to backfill practices from
  before the club started using this app. Skips unmatched names safely and
  merges with existing dates rather than duplicating them.

### Visitors (admin only, reached via "Visitors" on Balances)
- Track one-off visiting players separately from the permanent roster —
  they don't appear on the attendance checklist by default, Top Attendance,
  or the roster list.
- Add a visitor, mark them as a **student** (applies the student rate
  automatically), and **edit their name** any time via the pencil icon.
- Once linked to a practice (via "+ Add visitor" on the Today tab), their
  attendance fee is charged automatically — no need to manually log a
  charge per practice. "Add charge" is for genuinely one-off things like
  equipment.
- Visitors can stay on this list indefinitely — there's no need to remove
  them once they've paid. If you do remove someone, their payment, charge,
  and attendance history is **kept**, not deleted — only their entry on the
  active list is removed.

### History
- Every past practice, who attended (roster members and visitors, tagged
  separately), and the total collected.
- Admins can delete a session entry and export the full log as CSV
  (including a Visitors column).

### Student pricing
- A separate practice fee for members/visitors flagged as students, set in
  the header next to the regular fee.

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
  
