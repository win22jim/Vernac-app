# Vernac MCP tools — reference

A summary of the **87 tools** Vernac 1.2's MCP endpoint exposes, grouped by
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
  writes, the granted folder and the hand-off command, `notifications`
  (`enabled`, `authorization`), `onDeviceFit` (`available`, or why Apple
  Intelligence can't shorten text on this Mac), `safeToRelaunch`.
- **`set_notifications`** — `enabled` (**required**). The person's "Notify when
  transfers finish" switch: a macOS notification when a job finishes while
  they're away from Vernac's window, or when `request_screenshot_folder` waits
  on them.
  Counts and the app's name only; never prompts (provisional until the person
  chooses Show Banners). Local.
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
- **`get_controls`** — field policies and overrides, glossary (and
  `glossaryProtection`: `masked` or `native`), exclusions, source locale.
- **`check_compliance`** — local 2.3.7 / 5.2.5 / over-limit / required /
  untranslated / wrong-language scan of every locale. `severity`, `locale`,
  `rule` (`wrongLanguage` is on-device language identification per paragraph;
  never a blocker; keywords only ever a heads-up; sl-SI is not checked).
- **`analyze_keywords`** — keyword budget per locale with a tightened
  `suggestion`. `locale`, `platform`, `only_improvable`.

## Listing — edit and control (local)

- **`set_field`** — `field`, `locale` (**required**), `value` (string or null),
  `platform`. Always marked needs-review. Limits in characters: name/subtitle 30,
  keywords 100, promotional text 170, description/what's new 4000, URLs 255.
- **`revert_field`** — `field`, `locale` (**required**), `platform`.
- **`revert_all`** — `scope`: `listing` (default), `game_center`,
  `screenshots`, `app_previews`, `creative_assets`, `game_center_images`,
  `everything`. Refused while a job runs (an upload un-stages what lands; a
  translation job stages as it goes): follow or cancel it first.
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

- **`translate`** — `locale` (**required**), `platform`, `overwrite`,
  `fit_overflows`. One language. With `fit_overflows`, each overflow is
  shortened inside the call (up to ~2 minutes a field on a slow model).
- **`translate_all`** (job) — `locales`, `platforms` / `platform`, `overwrite`,
  `fit_overflows`, `wait_seconds`. Per-language counts; `needsDownload` lists
  languages whose model isn't installed.
- **`suggest_fit`** — `locale`, `field` (**required**: `name`, `subtitle`,
  `keywords`, `promotionalText`), `platform`. Up to three shorter versions of a
  value over its limit, by Apple Intelligence ON THIS MAC (macOS 26+), each
  checked by Vernac — within the limit, glossary terms kept, no price or Apple
  product wording where App Review refuses it, in the right language — with its
  `length` and a `backTranslation` into the source language. Stages nothing:
  stage your pick with `set_field`. No prompt parameter. `{unavailable: reason}`
  where it can't run. `fit_overflows: true` on `translate` / `translate_all`
  stages the first passing version instead (`fitted`, origin `shortened`); an
  overflow left says why — `fitUnavailable` (couldn't run) or `fitError` /
  `fitErrors` (ran and failed, e.g. `busy`: try again shortly); with neither,
  nothing passed. For the App Name, the app's own name isn't a term it must keep
  — put any word of it that must stay in the glossary.
- **`translate_gc`** (job) — `locales`, `overwrite`, `wait_seconds`. Game
  Center text.
- All three report `strategy` (`highFidelity` / `lowLatency`, macOS 26.4+;
  `mixed` across a job's languages), and — only when non-zero —
  `glossaryFallbacks` (re-translated with masking because a term changed) and
  `glossaryMissed` (staged values still missing a do-not-translate term: fix
  them with `set_field` / `set_gc_fields`).

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

## Media slots

Screenshots, app previews and creative assets live in App Store Connect's **App
Asset Library**: an asset shown in a slot — a language and a **placement group**
(`IPHONE_DYNAMIC_ISLAND_MEDIUM_PROFILE`, `IPHONE_DUO_PROFILE`, `IPAD_13_PROFILE`,
`MAC_PROFILE`…), read at run time from App Store Connect's catalog. Tools take
`group` / `groups`; the deprecated `display_type(s)` and `preview_type(s)` still
work as aliases. A slot that would end up over its group's `maxCount` makes any
upload dry run `blocked`, with no code. Removing media removes **placements**
(the files stay in the library) unless `delete_unused_assets` is set.

- **`get_media_specs`** — what each slot takes now, from App Store Connect's own
  catalog: sizes, `maxCount`, `requirement` (`required`,
  `requiredWhenFamilyUsed`, `requiredFrom` with its date), legacy aliases.
  `platform`, `placement_types`, `detail` (each spec: file types, alpha, largest
  file, video rules). Works before an app is loaded.

## Screenshots

- **`request_screenshot_folder`** — opens a picker for a person (prefer
  `open -g -b com.jeremylittlewood.Vernac "<folder>"`). `change`, `wait_seconds`.
- **`list_screenshots`** — the folder (`folderPath`, `layout`,
  `expectedSubfolders`) and compact counts; `detail`, `locale` / `locales`,
  `platform`.
- **`get_screenshots`** — what App Store Connect holds, in order, with placement
  ids. `locales`, `platform`, `groups`, `detail`.
- **`import_screenshots`** (local) — `paths` (in order), `platform`, `group`,
  `detail`.
- **`unstage_screenshots`** (local) — `paths`, or `locale` + `group`
  (+ `platform`), or `all`.
- **`reorder_staged_screenshots`** (local) — `locale`, `group`, `order`
  (**required**; positions, paths or staged ids), `platform`.
- **`upload_screenshots`** (dry run → code, job) — `replace` (every new file
  goes up first; the slot is swapped only then, and put back if that fails),
  `delete_unused_assets` (with `replace`), `locales`, `platform`, `groups`,
  `wait_seconds`, `detail`.
- **`delete_screenshots`** (dry run → code) — `locale` (**required**),
  `groups` or `ids`, `delete_unused_assets`, `platform`.
- **`reorder_screenshots`** (dry run → code) — `locale`, `group`, `order`
  (**required**; every live id once), `platform`.
- **`set_screenshot_shared`** (local) — `group`, `shared` (**required**).

## App previews

- **`get_app_previews`** — what App Store Connect holds. `locales`, `platform`,
  `groups`, `detail`.
- **`list_app_previews`** — videos in the folder, checked against their group's
  specs. `platform`, `detail`, `locale` / `locales`.
- **`import_app_previews`** (local) — `paths`, `platform`, `group`.
- **`unstage_app_previews`** (local) — `paths`, or `locale` + `group`
  (+ `platform`), or `all`.
- **`reorder_staged_app_previews`** (local) — `locale`, `group`, `order`
  (**required**), `platform`.
- **`upload_app_previews`** (dry run → code, job) — `replace`,
  `delete_unused_assets` (with `replace`), `locales`, `platform`, `groups`,
  `wait_seconds`, `detail`.
- **`delete_app_previews`** (dry run → code) — `locale` (**required**),
  `groups` or `ids`, `delete_unused_assets`, `platform`.
- **`reorder_app_previews`** (dry run → code) — `locale`, `group`, `order`
  (**required**), `platform`.
- **`set_app_preview_poster_frame`** — `path` (a staged video: local) or
  `asset_id` (deprecated `preview_id`) + `locale` (an uploaded video: dry run →
  code; refused once the video is approved); `time_code` (`HH:MM:SS:FF`) or
  `seconds` (+ `frame`), or `reset`.

## Creative assets (iOS: product page header, search results)

One image or video per language and `kind` (`header`, `search_results`; `both`
for either) on the editable iOS version — an upload always replaces.

- **`get_creative_assets`** — what each language holds and has staged.
  `locales`, `kind`, `detail`.
- **`import_creative_assets`** (local) — every file of a creative size in the
  language folders, or `paths`; `kind`, `detail`. A universal size stages as both.
- **`unstage_creative_assets`** (local) — `paths`, or `locales` and/or `kind`,
  or `all`.
- **`upload_creative_assets`** (dry run → code, job) — `locales`, `kind`,
  `delete_unused_assets`, `wait_seconds`, `detail`.
- **`delete_creative_assets`** (dry run → code) — `locales` (**required**),
  `kind`, `delete_unused_assets`.

## Transfers

- **`get_transfer`** — `job_id` (**required**), `wait_seconds`, `detail`.
  State, progress, the report once finished, and `notified: true` when the
  person was sent a notification about it (settled before the reply; a
  starting call that sees its job finish says it too).
- **`list_transfers`** — `running_only`.
- **`cancel_transfer`** — `job_id` (**required**). Stops at the next item; a
  video stops while it goes up (its reservation is deleted; it stays staged),
  and a language waiting on macOS's download prompt is let go of at once.

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
