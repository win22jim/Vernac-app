---
name: vernac
description: >-
  Expert guidance for driving Vernac — a macOS app that localizes an App Store
  Connect app's listing, screenshots, app previews and Game Center — through its
  Model Context Protocol (MCP) tools. Use this whenever the user wants to
  localize, translate or edit their App Store listing (app name, subtitle,
  description, keywords, promotional text, what's new, URLs); add a language;
  stage, order, upload or replace screenshots or app previews; localize, create
  or edit Game Center achievements, leaderboards, leaderboard sets, activities or
  challenges and their images; check a listing for App Review problems; review
  exactly what would change; or write changes to their App Store Connect draft.
  Also use when the user mentions "Vernac", or has Vernac's MCP server connected
  and asks about App Store Connect localization.
---

# Driving Vernac

Vernac is a native macOS app that reads a developer's own App Store Connect
content, lets them translate and edit it, and writes it to their **editable
draft**. When its MCP endpoint is connected you can drive all of it with a fixed
set of 79 tools (prefixed by the connection name, e.g. `vernac`). Everything you
do shows up live in Vernac's window, and the person can see and undo it there.

## Connecting Vernac (setup)

If no `vernac` tools are available, Vernac isn't registered with this assistant
(or wasn't running when the session started). Tell the user how to fix it rather
than guessing:

- **Claude Code:** in Vernac, menu bar → **Copy for Claude Code** (or the Welcome
  guide), then run the copied command in Terminal. It has this shape, with the
  user's own port and token:

  ```
  claude mcp add --scope user --transport http vernac \
    http://127.0.0.1:8376/mcp \
    --header "Authorization: Bearer <token>"
  ```

  `--scope user` registers Vernac for **every** project. Without it, `claude mcp
  add` defaults to *local* scope — only the folder the command ran in.
- **Then restart the session.** An open Claude Code session doesn't pick up a new
  server. If Vernac is registered but wasn't running when the session began,
  reconnecting it from `/mcp` is enough.
- **Re-adding** (a new port or token): `claude mcp add` won't overwrite an
  existing entry, so `claude mcp remove --scope user vernac` first.
- **Claude Desktop** uses a `claude_desktop_config.json` entry bridged through
  `mcp-remote` (needs Node.js) — the arrow beside Vernac's copy button.

## Hard rules (never break these)

1. **Never submit.** Vernac has no submit or add-for-review capability — not for
   the listing, app previews, or a Game Center activity/challenge version — and
   you must not look for one. Your work ends at the draft; the person submits in
   App Store Connect.
2. **Dry run, show, then confirm.** Every tool that writes App Store Connect is a
   dry run without `confirmation_code`: it writes nothing and returns the exact
   plan plus a code. Show the person the plan and get an explicit yes before you
   repeat the call with the **same arguments** and the code. Vernac can't undo
   these writes.
3. **Never repeat a long-running call to retry it.** Follow it with
   `get_transfer` (see below). A repeated upload duplicates what already landed.
4. **Preserve by default.** Change only what the person asked for. Vernac writes
   only fields that actually change.
5. **Act only on the app the person named.** Confirm the loaded app before you
   edit; an account can hold many apps.
6. **Don't claim review.** Everything you (or the on-device translator) stage is
   marked needs-review, and only the person clears that, in Vernac. Never
   describe your text as reviewed or final.
7. **Respect the content.** Translate meaning, not words. Keep brand names,
   product names and URLs intact. Flag what you're unsure of.

## Start every session the same way

1. **`get_vernac_status`** — one cheap call: is Vernac `ready` (endpoint serving,
   App Store Connect connected) and, if not, each issue; which app is loaded;
   unpublished work by kind (`unpublished`); jobs still running; the granted
   folder; and `safeToRelaunch`. Call it before asking the person to relaunch
   or update Vernac — relaunching loses unpublished work.
2. **`list_apps` → `load_app`** with the `app_id`. `load_app` is refused while
   work is staged (the refusal names each kind and the tool that sends or clears
   it) and while a job runs. **Load once.** To re-read App Store Connect without
   losing staged work, call **`refresh`**. Only pass `discard: true` when the
   person really wants to abandon their unpublished work.
3. **Read narrow.** A whole listing is large. Start with **`review_queue`** (what
   is still outstanding, as counts), then `get_listing` scoped by `locales`,
   `fields` or `changed_only`; `get_diff` for what a publish would write;
   `check_compliance` and `analyze_keywords` (findings, not text);
   `get_game_center` narrowed by `kinds` or `entity_ids`. Reads are **compact by
   default**; pass `detail: true` only when you need the per-item rows or text.

Arguments are checked against each tool's schema before it runs. An unknown
parameter, a wrong type (a string where a list is declared) or a bad enum value
does nothing and returns every problem plus the tool's real parameters — fix the
call and send it again.

## Editing the listing

- **`set_field`** stages one field in one locale (`field`, `locale`, `value`;
  `platform` for a version-level field on a multi-platform app; omit `value` or
  pass `null` to clear). Fields: `name`, `subtitle`, `privacyPolicyURL`
  (shared by every platform) and `description`, `keywords`, `promotionalText`,
  `whatsNew`, `marketingURL`, `supportURL` (per platform).
- **`revert_field`** undoes one field; **`revert_all`** takes back a kind of
  unpublished work (`scope`: `listing` (default), `game_center`, `screenshots`,
  `app_previews`, `game_center_images` or `everything`).
- Each field has a **policy** (`set_field_policy`): `localize` (per language),
  `lock` — **leave as-is**, never written, and *not* a fallback to the primary
  language (App Store Connect doesn't inherit listing text) — or `mirror` — the
  source language's value in every language. **App Name, Privacy Policy URL,
  Support URL and Marketing URL default to `mirror`.** Don't translate the app
  name unless the person asks for a localized name.
- `add_locale` without `locale` lists the addable languages; with one it adds
  it (publish creates it on App Store Connect). `remove_locale` takes back an
  added, unpublished language with everything staged in it.
- Excluding a language (`set_locale_excluded`) or platform, or changing a policy,
  never deletes staged edits: they are **held** and come back when allowed again.

### Character limits (App Store Connect counts UTF-16 units)

The same in every language. A Latin, CJK or Arabic letter counts 1; a Hindi or Thai syllable with vowel marks, or an emoji, counts 2 or more — trust Vernac's `length`, which uses App Store Connect's unit, over your own count:

| Field | Limit | | Field | Limit |
|---|---|---|---|---|
| App Name | 30 | | Promotional Text | 170 |
| Subtitle | 30 | | Description | 4000 |
| Keywords | 100 | | What's New | 4000 |
| URLs | 255 | | | |

**Count before you stage.** Subtitle and keywords overflow in translation
(German and French especially); Vernac won't publish an over-limit field. For
keywords: comma-separated, no spaces after commas, no duplicates, and don't
repeat words already in the name or subtitle — `analyze_keywords` returns a
tightened suggestion you can stage with `set_field`.

## Translating

Two paths, best combined:

- **You translate**, then stage with `set_field` (or `set_gc_fields` for Game
  Center). Best for the short, brand-critical fields and for any language Apple
  can't translate on device. Write natural, market-appropriate copy using
  Apple's own localized terminology.
- **Vernac's on-device translator:** **`translate_all`** sweeps many languages
  and returns per-language **counts**, not prose; `translate` does one language
  and platform; **`translate_gc`** does Game Center text. They need Vernac's main
  window open (it can be in the background). A language whose on-device model
  isn't downloaded is **skipped** and listed under `needsDownload` — translate
  those yourself. `translate_all` and `translate_gc` run as jobs (below); `ok` is
  false if any language didn't translate. Re-running with `overwrite: false`
  fills only what is still empty.

Register the person's own brand and feature words with **`set_glossary`** first
— the on-device path masks them so they survive verbatim. Apple's product names
and the app's own name are always protected. Read back only what you want to
check, with `get_listing(locales: [...])`.

## Writing to App Store Connect

### Publish text

`publish` sends staged **text** only — listing and Game Center. Media goes
separately (below).

1. `get_diff` or `publish` with no code → the plan (compact; `detail: true`
   for the text) and a `confirmationCode`. Blocked changes are grouped by reason
   (over the limit, a non-editable version, What's New on a first release).
2. Show the person; wait for their yes.
3. `publish` with `confirmation_code` → it runs as a job and answers within
   `wait_seconds` (below). Vernac takes a "Before publish" snapshot, skips what
   is already live, and writes one request per localization.
4. Report the real result: `written`, `failures`, `stillStagedCount`. What
   landed is un-staged; what failed stays staged, and publishing again sends only
   that. If `readBack` is `"failed"`, the write stands — call `refresh`. `notes`
   (e.g. `recoveredExistingLocalization`) are not failures.

### Long-running calls are jobs

A confirmed `publish`, `upload_screenshots`, `upload_app_previews`,
`upload_gc_images`, and `translate_all` / `translate_gc` run as jobs inside
Vernac. Each answers within `wait_seconds` (default 45, at most 110) with the
report under `result`, or with a `jobId` while it carries on — your client timing
out loses nothing. Then:

- **`get_transfer`** `{job_id}` to follow it (it can wait up to 110 s per call);
  `list_transfers` shows every job; `cancel_transfer` stops one at the next item
  boundary (what landed stays; the rest stays staged).
- **Never repeat the original call.** A second one while the first runs starts
  nothing (`started: false`); repeating a finished upload would send it twice.
  Read the result, then run a fresh dry run for whatever is still staged.

The person sees the same job in Vernac's window and as a progress bar on its Dock
icon.

## Files: screenshots, app previews and Game Center images

All three come from **one folder**. The person is usually away, so hand it over
**unattended** from a shell:

```
open -g -b com.jeremylittlewood.Vernac "/absolute/path/to/folder"
```

macOS grants Vernac read access to exactly that folder as part of the open — no
dialog, nobody at the keyboard. It reaches the running Vernac (or launches it);
**never add `-n`**. Then call `list_screenshots` and check `folderPath`. The
grant persists across relaunches. Only without a shell, call
`request_screenshot_folder` (it opens a picker for a person to answer, waits
`wait_seconds`, and answers `pending` while nobody has chosen). A hand-off is
refused while an upload is reading from the current folder —
`get_vernac_status` reports `handOffRefused`.

Layout — the `fastlane deliver` shape, plus Game Center:

```
<folder>/<locale>/1.png, 2.png …                  screenshots
<folder>/<locale>/tour.mov                        app previews (.mov, .m4v, .mp4) beside them
<folder>/gamecenter/<locale>/<vendor_id>.png      Game Center images
<folder>/gamecenter/default/<vendor_id>.png       an activity's or challenge's default image
```

Tool paths are **relative to that folder**; absolute paths and `..` are refused.
Then **list → import (stages, sends nothing) → upload (dry run → code → job)**:

- **Screenshots:** `list_screenshots` → `import_screenshots` →
  `upload_screenshots`. The folder names the language, the pixel size picks the
  display type. At most 10 per slot. **Order matters:** position 1 is the hero
  image in search results; staged order is upload order (`paths` in list order,
  otherwise natural file-name order — `2.png` before `10.png`). Fix staged order
  with `reorder_staged_screenshots`, take files back with
  `unstage_screenshots`. An upload **adds** to a slot by default; `replace: true`
  clears each target slot first (the dry run's `willDelete` lists what goes).
  Scope any upload with `locales`, `platform`, `display_types`. Confirm what
  landed with `get_screenshots`; fix live order with `reorder_screenshots`;
  clear slots or single screenshots with `delete_screenshots`.
- **App previews:** `list_app_previews` → `import_app_previews` →
  `upload_app_previews`. Apple's rules are checked before staging: 15–30 s,
  ≤ 500 MB, ≤ 30 fps, H.264 or ProRes 422 HQ, and an **audio track** (a silent
  stereo AAC track is fine — without one App Store Connect's processing fails).
  At most 3 per slot. After upload, `PROCESSING` is normal for up to 24 hours.
  Set a poster frame with `set_app_preview_poster_frame` (`time_code`
  `"HH:MM:SS:FF"`, or `seconds` + `frame`): on a staged video by `path`, or on
  an uploaded one by `preview_id` once it is `COMPLETE` (dry run → code).
- **Game Center images:** `list_gc_images` → `import_gc_images` →
  `upload_gc_images`. 1024×1024 for achievements, leaderboards and sets;
  3840×2160 for activities and challenges; PNG/JPEG, RGB, no transparency. An
  upload **replaces** a language's existing image. An entity needs text in a
  language before it can have an image there — publish the text first.

Staged media is **never** part of `publish` — `review_queue`, `get_diff` and
the publish dry run count it so a release can't look finished when it isn't.

## Game Center

- **Read** with `get_game_center` (compact; `kinds`, `entity_ids`, `locale`,
  `limit` / `offset`). It reports the 1000-point achievement budget
  (`achievementPoints`), `localeCoverage` and, for activities and challenges,
  `next`.
- **Text:** `set_gc_field` / `set_gc_fields` (up to 1000 at once) /
  `revert_gc_field`, `set_gc_field_lock`, `translate_gc`. Staging text in a
  language an entity doesn't have is how you **add** that language — `publish`
  creates it.
- **Create and edit** (each a dry run → code): `add_gc_achievement`,
  `add_gc_leaderboard`, `add_gc_leaderboard_set`, `add_gc_activity`,
  `add_gc_challenge`, `set_gc_achievement_points`, `update_gc_achievement`,
  `update_gc_leaderboard`, `update_gc_leaderboard_set`, `update_gc_activity`,
  `update_gc_challenge`. A vendor identifier is **permanent**. The dry run flags
  high-impact changes on live entities (a reversed ranking, deleted scores,
  hiding, archiving) — call them out to the person.
- **Structure:** `set_gc_leaderboard_set_members` (members and order),
  `set_gc_activity_links`, `delete_gc_entity` (only while never released —
  otherwise archive), `delete_gc_localization`.
- **Activities and challenges are versioned.** Vernac writes only the newest
  version while it is editable. When `get_game_center` says
  `next: "create_gc_version"`, run `create_gc_version` (dry run → code) to make
  a draft first; players see it only after the person submits it in App Store
  Connect.

## Snapshots

`take_snapshot` / `list_snapshots` / `restore_snapshot` cover staged **listing
text** only. A restore first saves the current state as a restore point
(`backup`), so it can be undone. `publish` takes its own snapshot automatically.

## Things App Store Connect rejects (handle gracefully)

- **What's New on a first release** — blocked; the rest publishes.
- **App Store Connect changed since the dry run** — the code no longer matches
  and nothing is written. Call `refresh` (never `load_app`, which would discard
  staged work), review the new plan, and confirm again.
- **Over-limit fields** — blocked; trim and stage again.
- **A slot over its limit** — an app preview slot over 3 is blocked with no
  code; a screenshot slot over 10 is flagged `overLimit` in the dry run (it still
  returns a code, but App Store Connect refuses the extra screenshots). Either
  way, use `replace`, unstage some, or scope that slot out before confirming.

## Before a submission

Run `check_compliance` over every language: price or "free" wording outside the
description (Guideline 2.3.7), Apple product names in the name, subtitle or
keywords (5.2.5), over-limit and empty required fields, languages still
identical to the source. A *translation* can reintroduce a term the English copy
never had. Then `review_queue` for anything still outstanding.

## Exact parameters

The connection's `tools/list` is authoritative for names, types and required
fields. A grouped summary of all 79 tools is in
[references/mcp-tools.md](references/mcp-tools.md).

## Good habits

- Say what you're about to do in one line before you start.
- After edits, summarise per language (what changed, anything over the limit or
  awaiting review) before proposing a publish.
- Report real results — what was written, what failed and why, what is still
  staged. Never claim success you didn't read back.
- Before suggesting a relaunch, check `get_vernac_status.safeToRelaunch`.
