# kehadiran

Sign-in gate **and** full-page wrapper for the SMK Meradong "Sistem Rekod Kehadiran"
Google Apps Script web app.

Live app : https://script.google.com/macros/s/AKfycbwL7xyYOZmCL-KkCIee_k-fYl-k4DUZXq5FEHklz8qWFppBGtazTbi126LjiFRKHrj-6w/exec
Wrapper  : https://ingsiong-dev.github.io/kehadiran/

## Why this file is not just an iframe any more

It still embeds the app in an iframe, which is what suppresses Google's
"This application was created by a Google Apps Script user" bar - that bar is drawn only in
the **top-level** window, so the same URL inside a frame has none.

But the app **serves this school's attendance data** - every class's daily figures, the class
teacher list, the holiday list - and it used to accept anyone holding the link, so this page now
signs the teacher in *before* the app loads. This is the only place the Google account chooser
can be reached from: an Apps Script page runs in a sandboxed iframe and may not navigate to
`accounts.google.com`.

The backend stays the authority. `doGet()` needs `?token=<Google ID token>` and only then
renders the app, and every data function re-checks that token against the school's teacher
roster (the `DELIMA` tab) - because `google.script.run` is reachable from any page the script
serves, so a render gate alone would be walkable-around.

Two things must be true for a sign-in to work, and both are one-time:

1. `https://ingsiong-dev.github.io/kehadiran/` must be an **Authorized redirect URI** on the
   shared OAuth client (`314693319074-n5unk6cg17srqoe2fiu1nd67njaj5pp2`). It must match
   character for character, trailing slash included, or Google shows a raw
   `redirect_uri_mismatch` page. **Registered** (verified 01 Oct 2026).
2. The Apps Script project needs the `UrlFetchApp` scope, granted once by the owner running
   `ujianLogin` -> Allow in the editor. Until it is granted every token check fails and nobody
   can log in.

Already signed in to the portal or the e-RPH app on the same phone? This page adopts that
session, because all three share one Google client and one teacher roster.

## Deploying a change

Code changes go to the **Apps Script** project; page changes go here. An auth change needs
both. Never create a new deployment - that mints a new URL and breaks every link already sent
to teachers:

```powershell
cd ..\..\apps-script-project
clasp push ; clasp create-version "description" ; clasp update-deployment <deploymentId> -V <n>
```

then, in this folder, `git add index.html README.md ; git commit ; git push`
(GitHub Pages rebuilds in about a minute).

## The `versi` stamp

The gate prints `versi k<major>.<minor>-<date>` at the bottom. GitHub Pages serves this HTML
with `Cache-Control: max-age=600`, so a phone can be running a stale copy and the failure text
looks identical - **bump the stamp on every edit**, or it stops identifying anything.
Current: `k2.1-2026-10-01` (backend `v2.43`, deployment `@43`).

## The app's OWN login page is a copy of this one (02 Oct 2026)

Teachers who open the `/exec` link directly never see this page: `doGet()` finds no token and
renders `LogMasuk.html` from the Apps Script project instead. That page used to look like a
different product - a dark masthead, a filled navy button and three numbered steps - and his
report was a screenshot of it: "不要这样的".

`LogMasuk.html` is now a **deliberate copy of `#card-login`** here: same card, same 60px crest,
same centred `SISTEM KEHADIRAN / SMK MERADONG`, same WHITE OUTLINED button
("Log masuk dengan akaun Delima"), same version + developer footer. **Restyle one, restyle the
other** - verified by rendering both at the same viewport and comparing the boxes
(`_test\preview_logmasuk.py`; card 354px, button 310x44, crest 60px on both).

Its footer must show the **app** version (`<?= versi ?>` = `APP_VERSION`), never the `k2.x`
page stamp: the standing rule is that the visible version equals the deployed Apps Script
version. `_test\test-kehadiran-readonly.mjs` enforces that constant and is meant to be edited
on every deploy.

## Trap: an inline style beats the stylesheet

`.tblwrap.narrow` (Guru Kelas, and the monthly-average table) only works if the wrapper is
`display:inline-block` - that is what makes the box hug its table instead of stretching the
full width while the table sits 305px inside it. `renderGuru()` used to write
`style.display = 'block'`, and **an inline declaration always wins**, so the class was silently
defeated and the KELAS column sat a mile from the teacher's name ("太远", his report).
Setting `'inline-block'` there fixed it. Grep for `style.display` before trusting any
`.narrow` table.

## Language rule (01 Oct 2026, his request)

Every RUNTIME message is English - the status line ("Loading the attendance system…",
"Verifying account…", "Sign-in cancelled. Try again.") and the in-app notice. The designed card
copy (headings, "Sebab" box, numbered steps) stays Malay, like the rest of the app. Same split
in the app itself: system words English, report content Malay.

## Rollback

`clasp update-deployment <deploymentId> -V 68` puts back **v2.67**, before the Laporan
class-name fix (07 Oct 2026): the card's document tile + three action buttons occupy
120px, so at the old 134px column the name was left 29px and "U6STEM" printed as "U6…".
v2.68 raises the floor to 178px, never lets the name shrink below its own text
(`min-width:min-content`) and drops the buttons to a second line instead.

`clasp update-deployment <deploymentId> -V 67` puts back **v2.66**, the last version before
kehadiran became pass/fail in **two colours only** (07 Oct 2026, his request): one
`PASS_MARK` of 96.72 %, green at or above it and red below, on every attendance figure.
Before that the monthly class cards used four bands (95 / 90 / 85) and the daily bar three
(90 / 75), so the same class could be green on one tab and amber on the next.

`clasp update-deployment <deploymentId> -V 40` puts the previous version back (v2.40, before the
LogMasuk restyle and the Guru Kelas column fix). `-V 30` goes back before the name/PIN/English
changes; `-V 24` goes all the way to the ungated app (its `doGet()` ignores the token, so the
iframe URL this page builds still loads it - but then anyone with the plain link is inside
again). No rollback ever needs a new URL.
