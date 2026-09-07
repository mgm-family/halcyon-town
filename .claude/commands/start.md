---
description: Re-launch the FiveM RP server homepage content sync for a new server (first-time full rewrite, then re-enable the daily diff check)
---

# /start — new-server content re-launch

This command is for when the current server ("Vulfayne City" / "VAY city") has been decommissioned and a **new server with a new Google Doc** has taken its place. The daily content-sync Routine was intentionally disabled on 2026-09-07 because the old source Google Doc (fileId `1gJu3XOIIuHXuEIPi4QAMEFK-KnoI-L1mAw4DpZGNqBE`) became inaccessible. Running `/start` is how the sync gets turned back on for the new doc.

**Argument**: the new Google Doc's share URL or bare fileId. If the user ran `/start` with no argument, ask them for it before doing anything else — do not guess or reuse the old fileId.

## What this repo looks like

- `index.html` — the homepage. Sidebar tabs: TOP / 物語(story) / 最新情報 / 規程(rules) / 参加(entry) / 補填(compensation) / 公務員(civil) / 民間(civilian) / 犯罪(crime). Only the last 6 are doc-driven.
- `data/rules.json`, `data/entry.json`, `data/compensation.json` — flat schema: `{ "blocks": [ { "num": "01"|null, "title": "..."|null, "html": "<ul>...</ul>..." }, ... ] }`
- `data/civil.json`, `data/civilian.json`, `data/crime.json` — accordion schema: `{ "accent": "blue"|"gold"|"vermillion", "groups": [ { "tag": "PD", "title": "...", "open": true|false, "html": "<h4>...</h4><p>...</p>..." }, ... ] }`
- `index.html` fetches these 6 files client-side at load (see the `CONTENT_TARGETS` array and `loadContent()` near the end of the file) — editing the JSON and pushing to `main` is all that's needed for GitHub Pages to show it.
- `top` and `story` are hand-written static HTML in `index.html` itself — **not** part of this sync. Leave them alone unless the user separately asks to rebrand them for the new server.

## Steps

1. **Get the new doc.** Parse the fileId out of the URL argument (the `.../document/d/<fileId>/edit` segment). Read its full content with the Google Drive connector: try `read_file_content` first; if it looks truncated (large docs sometimes cut off), use `download_file_content` with `exportMimeType: "text/plain"`, which returns base64 — decode it (`base64.b64decode(...).decode('utf-8')`) to get the complete text.

2. **This is a full rewrite, not a diff.** Unlike the daily sync (which only patches specific lines it finds changed), treat every one of the 6 files as being replaced from scratch based on the new doc's current content — the old content is from a different server and must not linger. Map doc headings to files the same way the daily sync does:
   - サーバー規程 → `rules.json` (each `★...`-style heading = one flat block, numbered "01", "02", ... in doc order)
   - 新規参加者の招待について → `entry.json` (flat, usually a single block)
   - 補填対応について → `compensation.json` (flat, usually a single block)
   - 公務員 and its sub-sections (PD / EMS / 白メカニック / ディーラー etc.) → `civil.json` (accordion, accent `"blue"`, one group per sub-section, first group `open: true`)
   - 有人JOB and its shops → `civilian.json` (accordion, accent `"gold"`)
   - 犯罪者 and its sub-sections (闇医者 / 闇メカニック / ギャング / 犯罪ルール etc.) → `crime.json` (accordion, accent `"vermillion"`)

   If the new doc's section headings differ from this list (new server, possibly reorganized rules), use your judgment to map them sensibly to the closest matching file/schema — ask the user if a whole section doesn't fit anywhere.

3. **Write faithful HTML**, matching the visual conventions already used in these files: `<ul><li>` for bullet rules, `<table><tr><th>/<td>` for tables, `<strong>` for emphasis, inline `style="color:var(--muted)"` for de-emphasized notes, `<h4>` for sub-headings inside an accordion body. Don't invent content beyond what the new doc says.

4. **Sanity-check before shipping**: start a local static server (`python3 -m http.server` on a free port), open `index.html` there with Playwright (`NODE_PATH=/opt/node22/lib/node_modules`, chromium at `/opt/pw-browsers/chromium`), click through all 6 rewritten tabs (and expand accordions), confirm nothing renders broken, screenshot each. Stop the server after.

5. **Ship it**: create a branch (e.g. `content-sync/new-server-launch`), commit, push, open a PR into `main` clearly titled as a full re-launch for the new server (not a daily incremental sync), and describe what the new doc's structure looked like and how it was mapped. Do not merge it yourself unless the user has told you (in this conversation) that they want every sync auto-merged — if unsure, ask.

6. **Re-arm the daily Routine.** Find the "Vulfayne City doc content sync" Routine (trigger_id `trig_01WawaRe6aa5V8MJP8mJyiPB` as of 2026-09-07, but confirm via `list_triggers` in case it changed) and call `update_trigger` with:
   - `enabled: true`
   - a new `prompt` — take its existing prompt as a template and swap in the new doc's URL/fileId everywhere the old one appears. Keep the rest of the logic (branch reuse, screenshot-then-Discord-webhook flow, "no changes → stay quiet" rule) the same.

7. **Report back**: what the new doc's structure looked like, which files were rewritten, the PR link, and confirmation that the daily Routine is re-enabled and pointed at the new doc.
