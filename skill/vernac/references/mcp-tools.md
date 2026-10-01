# Vernac MCP tools — reference

A summary of the **79 tools** Vernac 1.1's MCP endpoint exposes, grouped by
purpose. The endpoint is a **fixed, closed set**: it cannot run code, shell
commands or scripts, and no parameter is ever interpreted as one.

**The authoritative source is the live `tools/list`** on the connection — exact
parameter names, types, enums, ranges and required fields. Arguments are checked
against that schema before a tool runs: an unknown parameter, a wrong type or a
bad enum value does nothing and returns every problem plus the real parameters.
Enum values are forgiving about case and separators (`macos` → `MAC_OS`).

Conventions used below: **required** parameters are marked; everything else is
optional. Platforms: `IOS`, `MAC_OS`, `TV_OS`, `VISION_OS`. Game Center kinds:
`achievement`, `leaderboard`, `leaderboardSet`, `activity`, `challenge`.

- **Dry run → code** — writes App Store Connect. Without `confirmation_code` it
  writes nothing and returns the plan plus a code; repeat the **same arguments**
  with the code to write exactly that. A stale plan writes nothing.
- **Job** — runs inside Vernac and answers within `wait_seconds` (0–110,
  default 45) with the report under `result` or a `jobId`. Follow with
  `get_transfer`; never repeat the call.
- **Local** — changes only what Vernac has staged or its settings.

## Status & discovery

- **`get_vernac_status`** — call first. `ready` and each issue, the loaded app,
  unpublished work by kind (incl. `heldEdits`), running jobs and multi-request
  writes, the granted folder and the hand-off command, `safeToRelaunch`.
- **`list_accounts`** — stored API keys (key id, issuer id, which is active).
  Read-only; the person switches accounts in Vernac.
- **`list_apps`** — the active account's apps (`id`, name, bundle id, primary
  locale).
- **`load_app`** — `app_id` (**required**), `discard`. Refused while work is
  staged (unless `discard: true`) or a job runs. Load once.
- **`refresh`** — re-read the loaded app, keeping staged work. Returns `settled`
  and `stillPending`.

## Listing — read

- **`review_queue`** — start here: outstanding work as counts — empty required
  fields, values awaiting review, over-limit values, missing optionals, Game
  Center text and required images, missing screenshot types, staged media.
  `platform`, `locale`, `include_ready`, `detail`.
- **`get_listing`** — values with change, origin, policy, limit and length.
  `locale` / `locales`, `platform`, `fields`, `changed_only`, `values` (text is
  included only for scoped reads by default).
- **`get_status`** — per-locale status. `locale`.
- **`get_diff`** — what `publish` would write (listing + Game Center text).
  `locales`, `scope` (`listing` / `gameCenter`), `detail` (from → to text).
- **`get_controls`** — field policies and overrides, glossary, exclusions, source
  locale.
- **`check_compliance`** — local 2.3.7 / 5.2.5 / over-limit / required /
  untranslated scan of every locale. `severity`, `locale`, `rule`.
- **`analyze_keywords`** — keyword budget per locale with a tightened
  `suggestion`. `locale`, `platform`, `only_improvable`.

## Listing — edit and control (local)

- **`set_field`** — `field`, `locale` (**required**), `value` (string or null),
  `platform`. Always marked needs-review. Limits in characters: name/subtitle 30,
  keywords 100, promotional text 170, description/what's new 4000, URLs 255.
- **`revert_field`** — `field`, `locale` (**required**), `platform`.
- **`revert_all`** — `scope`: `listing` (default), `game_center`,
  `screenshots`, `app_previews`, `game_center_images`, `everything`.
- **`set_field_policy`** — `field`, `policy` (**required**: `localize`, `lock`
  = leave as-is, `mirror` = same in every language), `locale`, `platform`. App
  Name, Privacy Policy URL, Support URL and Marketing URL default to `mirror`.
  Edits a policy doesn't allow are held, not deleted.
- **`set_glossary`** — `terms` (**required**; replaces the list).
- **`set_locale_excluded`** — `locale`, `excluded` (**required**). Edits held.
- **`set_platform_excluded`** — `platform`, `excluded` (**required**).
- **`set_source_locale`** — `locale` (null = primary).
- **`add_locale`** — `locale`; omit it to list the addable locales.
- **`remove_locale`** — `locale` (**required**): an added, unpublished language
  and everything staged in it.
- **`sync_localization`** — `locale`, `from_platform` (**required**),
  `to_platform`.

## Translation (on device, staged for review)

- **`translate`** — `locale` (**required**), `platform`, `overwrite`. One
  language.
- **`translate_all`** (job) — `locales`, `platforms` / `platform`, `overwrite`,
  `wait_seconds`. Per-language counts; `needsDownload` lists languages whose
  model isn't installed.
- **`translate_gc`** (job) — `locales`, `overwrite`, `wait_seconds`. Game
  Center text.

## Game Center — read and text

- **`get_game_center`** — entities, attributes, set members in order, text in
  one locale, image status, point budget, `localeCoverage`, and `next` for
  activities/challenges. `locale`, `kinds`, `entity_ids`, `detail`, `images`,
  `read_images`, `limit`, `offset`.
- **`set_gc_field`** — `entity_id`, `field`, `locale` (**required**), `value`.
- **`set_gc_fields`** — `values` (**required**; up to 1000
  `{entity_id, field, locale, value}`).
- **`revert_gc_field`** — `entity_id`, `field`, `locale` (**required**).
- **`set_gc_field_lock`** — `field`, `locked` (**required**), `entity_ids` or
  `kinds`, `locales`.

## Game Center — create and edit (dry run → code)

- **`add_gc_achievement`** — `reference_name`, `vendor_identifier` (permanent),
  `points` (0–100), `show_before_earned`, `repeatable`, `locale`, `name`,
  `before_earned_description`, `after_earned_description` (all **required**);
  `activity_properties`.
- **`add_gc_leaderboard`** — `reference_name`, `vendor_identifier`,
  `default_formatter`, `submission_type`, `score_sort_type`, `locale`, `name`
  (**required**); score range, recurrence, `visibility`, `activity_properties`.
- **`add_gc_leaderboard_set`** — `reference_name`, `vendor_identifier`,
  `locale`, `name` (**required**).
- **`add_gc_activity`** — `reference_name` (≤ 40), `vendor_identifier`,
  `locale`, `name` (**required**); `play_style`, player counts,
  `supports_party_code`, `properties`, `fallback_url`, `description`,
  `leaderboard_ids`, `achievement_ids`. Created with a draft version.
- **`add_gc_challenge`** — `reference_name`, `vendor_identifier`,
  `leaderboard_id`, `locale`, `name` (**required**); `repeatable` (needs a
  recurring leaderboard), `description`.
- **`set_gc_achievement_points`** — `entity_id`, `points` (**required**);
  enforces the 1000-point total.
- **`update_gc_achievement`** / **`update_gc_leaderboard`** /
  **`update_gc_leaderboard_set`** / **`update_gc_activity`** /
  **`update_gc_challenge`** — `entity_id` (**required**) plus the attributes to
  change (reference name, flags, score configuration, recurrence, visibility,
  archived, properties, fallback URL, a challenge's leaderboard).
- **`set_gc_leaderboard_set_members`** — `set_id` or `entity_id`; exactly one of
  `leaderboard_ids` (full ordered list), `add` (+ `position`), `remove`.
- **`set_gc_activity_links`** — `activity_id` or `entity_id`;
  `add_leaderboards`, `remove_leaderboards`, `add_achievements`,
  `remove_achievements`.
- **`create_gc_version`** — `entity_id` (**required**): a new draft version of
  a live activity or challenge.
- **`delete_gc_entity`** — `entity_id` (**required**): only while never
  released.
- **`delete_gc_localization`** — `entity_id`, `locale` (**required**).

## Game Center images

- **`get_gc_images`** — what App Store Connect holds. `kinds`, `entity_ids`,
  `locales` (`default` for default images), `refresh`, `detail`.
- **`list_gc_images`** — files in the folder and what each maps to. `detail`,
  `locale` / `locales`.
- **`import_gc_images`** (local) — every convention file, or `paths`, or
  `mappings` (`{path, locale, entity_id | vendor_identifier, kind?}`); `detail`.
- **`unstage_gc_images`** (local) — `paths`, `images`, `entity_ids`, `locales`,
  `kinds` or `all`.
- **`upload_gc_images`** (dry run → code, job) — `kinds`, `entity_ids`,
  `locales`, `wait_seconds`, `detail`. New or replacing.
- **`delete_gc_images`** (dry run → code) — `images` (**required**;
  `[{entity_id, locale}]`).

## Screenshots

- **`request_screenshot_folder`** — opens a picker for a person (prefer
  `open -g -b com.jeremylittlewood.Vernac "<folder>"`). `change`, `wait_seconds`.
- **`list_screenshots`** — the folder (`folderPath`, `layout`,
  `expectedSubfolders`) and compact counts; `detail`, `locale` / `locales`,
  `platform`.
- **`get_screenshots`** — what App Store Connect holds, in order, with ids.
  `locales`, `platform`, `display_types`, `detail`.
- **`import_screenshots`** (local) — `paths` (in order), `platform`, `detail`.
- **`unstage_screenshots`** (local) — `paths`, or `locale` + `display_type`
  (+ `platform`), or `all`.
- **`reorder_staged_screenshots`** (local) — `locale`, `display_type`, `order`
  (**required**; positions, paths or staged ids), `platform`.
- **`upload_screenshots`** (dry run → code, job) — `replace`, `locales`,
  `platform`, `display_types`, `wait_seconds`, `detail`.
- **`delete_screenshots`** (dry run → code) — `locale` (**required**),
  `display_types` or `ids`, `platform`.
- **`reorder_screenshots`** (dry run → code) — `locale`, `display_type`,
  `order` (**required**; every live id once), `platform`.
- **`set_screenshot_shared`** (local) — `display_type`, `shared` (**required**).

## App previews

- **`get_app_previews`** — what App Store Connect holds. `locales`, `platform`,
  `preview_types`, `detail`.
- **`list_app_previews`** — videos in the folder, checked against Apple's rules.
  `platform`, `detail`, `locale` / `locales`.
- **`import_app_previews`** (local) — `paths`, `platform`, `preview_type`.
- **`unstage_app_previews`** (local) — `paths`, or `locale` + `preview_type`
  (+ `platform`), or `all`.
- **`reorder_staged_app_previews`** (local) — `locale`, `preview_type`, `order`
  (**required**), `platform`.
- **`upload_app_previews`** (dry run → code, job) — `replace`, `locales`,
  `platform`, `preview_types`, `wait_seconds`, `detail`.
- **`delete_app_previews`** (dry run → code) — `locale` (**required**),
  `preview_types` or `ids`, `platform`.
- **`reorder_app_previews`** (dry run → code) — `locale`, `preview_type`,
  `order` (**required**), `platform`.
- **`set_app_preview_poster_frame`** — `path` (a staged video: local) or
  `preview_id` + `locale` (an uploaded, `COMPLETE` video: dry run → code);
  `time_code` (`HH:MM:SS:FF`) or `seconds` (+ `frame`), or `reset`.

## Transfers

- **`get_transfer`** — `job_id` (**required**), `wait_seconds`, `detail`.
  State, progress, and the report once finished.
- **`list_transfers`** — `running_only`.
- **`cancel_transfer`** — `job_id` (**required**). Stops at the next item.

## Snapshots (listing text only)

- **`take_snapshot`** — `label` (**required**).
- **`list_snapshots`**.
- **`restore_snapshot`** — `id` (**required**). Saves the current state first
  (`backup`).

## Publish (dry run → code, job)

- **`publish`** — `confirmation_code`, `wait_seconds`, `detail`. Listing and
  Game Center **text** only; media goes through its own upload tool. Confirmed,
  it writes one request per localization; what lands is un-staged, what fails
  stays staged (`written`, `failures`, `stillStagedCount`, `readBack`).
  **Never submits** — the person submits in App Store Connect.
