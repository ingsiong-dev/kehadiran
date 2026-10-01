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

## Rollback

`clasp update-deployment <deploymentId> -V 24` puts the ungated app back (its `doGet()` ignores
the token, so the iframe URL this page builds still loads it - but then anyone with the plain
link is inside again).
