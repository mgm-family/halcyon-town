---
description: Bring a staged/future job section (drafted ahead of time in the Google Doc under a marked heading) live on the site — replacing its "COMING SOON" placeholder, or updating an already-live section with its next-chapter version.
---

# /reveal — bring a staged section live

Some upcoming FiveM server features (e.g. ディーラー, or a rebuilt 闇医者/闇メカニック for a later story chapter) get written into the Halcyon Town Google Doc well ahead of time, but must **not** appear on the live site — and must **not** be picked up by the daily "Halcyon Town doc content sync" Routine — until the user explicitly says the feature is ready. This command does that on-demand reveal for one section at a time.

**Argument**: which section to reveal (e.g. "ディーラー(高級車・ヘリ)", "闇医者", "闇メカニック"). If it's ambiguous which doc heading or which data file group is meant, ask before doing anything.

## Why staged content doesn't leak on its own

The daily sync Routine (and `/start`) only ever touch a fixed list of *exact* heading strings (サーバー規程, 公務員, PD, 押収品, EMS, 白メカニック, ディーラー(高級車・ヘリ), 有人JOB, 飲食店①, 飲食店②, ディーラー(普通車・ボート), 犯罪者, 闇医者, 闇メカニック, ギャング, 犯罪ルール, etc.). Anything else in the doc is invisible to it. So the convention for staging is:

- **Brand-new section, not live anywhere yet** (e.g. a future job with no current site presence): write it under a heading that does **not** match one of those exact strings — e.g. prefix it, like "【下書き】ディーラー(高級車・ヘリ)". The daily sync will never match that heading, so it stays inert until `/reveal` is run.
- **Replacing an already-live section** (e.g. 闇医者/闇メカニック getting a rebuilt chapter-5 version while the current one is still shown on the site): do **not** edit the live heading in place — write the new version under a *separate* draft heading (e.g. "【第5章下書き】闇医者") elsewhere in the doc, leaving the currently-live "闇医者" heading untouched so the daily sync keeps serving the current content until you're ready to switch over.

## Steps

1. **Get the doc.** Read it fresh with `read_file_content` (fall back to `download_file_content` with `exportMimeType: "text/plain"` + base64 decode if it looks truncated).
2. **Find the draft section** matching the requested argument — it will be under one of the marked/prefixed headings described above, not the plain public heading.
3. **Work out the target** in this repo's `data/*.json`:
   - ディーラー(高級車・ヘリ) → `data/civil.json`, the group with `"tag": "DEALER"` — replace its `html` (currently `<span class="badge-soon">COMING SOON…</span>`).
   - ディーラー(普通車・ボート) → `data/civilian.json`, the group with `"tag": "DEALER"` — same treatment.
   - 闇医者 / 闇メカニック → `data/crime.json`, the two groups with `"tag": "DARK"` (match by `title`) — replace that group's `html` with the new version.
   - Anything else (a job with no existing group at all) → add a new group to the appropriate file in a sensible position, following the accordion schema already used there (`tag`, `title`, `open: false`, `html`), and matching the existing HTML conventions (`<h4>`, `<ul><li>`, `<table><tr><th>/<td>`, `<strong>`, inline `style="color:var(--muted)"` for de-emphasized notes).
   - If a matching icon exists under `assets/logo-*.jpg` for this job, add/keep the group's `"icon"` field; otherwise leave it unset (per the convention already used — groups without provided artwork don't get one).
4. **Write faithful HTML** from the draft text — don't invent content beyond what the doc says.
5. **Sanity-check**: serve the repo (`python3 -m http.server` on a free port), open `index.html` with Playwright (`NODE_PATH=/opt/node22/lib/node_modules`, chromium at `/opt/pw-browsers/chromium`), click to the relevant tab, expand the revealed group, screenshot it. Stop the server after.
6. **Ship it**: branch (e.g. `reveal/<section-slug>`), commit, push, open a PR titled clearly as a reveal (not a routine content sync) describing what's being revealed and, if known, which story chapter it corresponds to. Do not merge it yourself unless the user has said they want it auto-merged — if unsure, ask.
7. **Tell the user about the doc cleanup**: once this is merged, if they want the daily sync Routine to keep this section up to date going forward, they should rename the doc heading to drop the draft marker (e.g. "【下書き】ディーラー(高級車・ヘリ)" → "ディーラー(高級車・ヘリ)") so it matches the Routine's normal mapping — otherwise future edits to that heading won't be picked up automatically. If a draft heading was staged *alongside* a still-differently-named live heading (the 闇医者/闇メカニック replace-in-place case), remind them to remove or repurpose the old live heading so the doc doesn't end up with two conflicting versions of the same section.
8. **Report back**: what was revealed, which file(s) changed, the PR link, and the doc-cleanup reminder from step 7.
