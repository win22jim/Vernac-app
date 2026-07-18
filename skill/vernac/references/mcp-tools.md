# Vernac MCP tools — reference

A summary of the tools Vernac's MCP endpoint exposes, grouped by purpose. The
endpoint is a **fixed, closed set** — it cannot run arbitrary code, shell
commands, or scripts.

**The authoritative source is the live `tools/list`** on the connection — call it
for exact parameter names, types, and required fields. Reads return JSON (in a
text block); edits return a small confirmation with the affected field's fresh
state. Every mutation is staged in Vernac's working state until you publish.

## Discovery

- **`list_apps`** — list the connected account's apps (name, bundle id, app id).
  Call this first to get an `app_id`.
- **`load_app`** — load one app's metadata as the working baseline.
  - `app_id` (required)
  - `discard` (optional, default false) — discard any unpublished staged edits.
    Loading is refused when edits are pending unless this is `true`, so you don't
    silently lose work. **Load once; then stage and publish.**

## Reads

- **`get_listing`** — every listing field for a locale, with value, character
  count, limit, lock state, and change status. Optional `locale`, `platform`.
- **`get_status`** — per-locale completion status (not started / in progress /
  needs review / complete). Optional `locale`.
- **`get_diff`** — the exact set of changes a publish would write. No parameters.
- **`get_controls`** — locks, excluded locales/platforms, and the source locale.
- **`get_game_center`** — Game Center entities and their localizable text, with
  ids, values, source, change, and status.

## Listing edits

- **`set_field`** — set one field for one locale. `field`, `locale` (required);
  `value` (omit/null to clear); `platform` (required on a multi-platform app);
  `origin` (optional). Fields: `name`, `subtitle`, `description`, `keywords`,
  `promotionalText`, `whatsNew`, `marketingURL`, `supportURL`.
- **`revert_field`** — undo the staged edit for one field/locale.
- **`revert_all`** — undo all staged edits.

## Control

- **`add_locale`** — add a new App Store locale to localize into (`locale`, e.g.
  `de-DE`). Publishing creates the localization on App Store Connect.
- **`set_lock`** — lock/unlock a field as "do not localize" (`field`, `locale`,
  `locked`). The app name is locked by default.
- **`set_locale_excluded`** — include/exclude a locale (`locale`, `excluded`).
- **`set_platform_excluded`** — include/exclude a platform (`platform`,
  `excluded`).
- **`set_source_locale`** — set which locale is the translation source
  (`locale`).

## Translation

- **`translate`** — stage on-device (Apple Translation) drafts for a locale.
  `locale` (required); `platform` (on a multi-platform app); `overwrite`
  (optional, also replace fields the target already has). Skips locked fields and
  URLs; over-limit results are reported, not staged; everything staged is marked
  "needs review". Treat the output as a draft and review it.
- **`sync_localization`** — copy a locale's version-level fields from one platform
  to the others (for a multi-platform app).

## Game Center

- **`set_gc_field`** / **`revert_gc_field`** — edit/undo Game Center text
  (achievement name/before/after, leaderboard name/description/suffixes, set
  name).
- **`add_gc_achievement`** / **`add_gc_leaderboard`** / **`add_gc_leaderboard_set`**
  — create new Game Center entities (with their source-locale text).
- **`update_gc_achievement`** / **`update_gc_leaderboard`** /
  **`update_gc_leaderboard_set`** — edit entity attributes.
- **`set_gc_achievement_points`** — set an achievement's point value (respecting
  the app's total points budget).

## Media

- **`upload_media`** — screenshot / Game Center image upload. Because a sandboxed
  Mac app can't read arbitrary paths handed to it over MCP, this is generally a
  redirect to do the upload in the app UI (where you pick the file). Follow the
  tool's guidance.

## Snapshots

- **`take_snapshot`** — capture a local restore point of the current working
  state.
- **`list_snapshots`** — list saved snapshots.
- **`restore_snapshot`** — restore a snapshot.

## Review & publish (draft only — never submits)

- **`publish`** — with **no** `confirmation_code`, this is a **dry run**: it
  returns the plan and a short confirmation code and writes nothing. Call it
  again **with** that `confirmation_code` to execute the write to the editable
  draft. Any edit made after the dry run invalidates the code (re-run the dry
  run). Reports what was written and any per-field failures.
- **`add_for_review`** — staging helper; **does not** submit the app. The final
  Submit for Review is always done by the user in App Store Connect.
