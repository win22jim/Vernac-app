---
name: vernac
description: >-
  Expert guidance for driving Vernac — a macOS app that localizes an App Store
  Connect listing across languages — through its Model Context Protocol (MCP)
  tools. Use this whenever the user wants to localize, translate, or edit their
  App Store listing (app name, subtitle, description, keywords, promotional text,
  what's-new), their screenshots, or their Game Center achievements/leaderboards;
  add a language to their listing; review exactly what would change; or publish
  listing changes to their App Store Connect draft. Also use when the user
  mentions "Vernac", or has Vernac's MCP server connected and asks about App
  Store Connect localization.
---

# Driving Vernac

Vernac is a native macOS app that reads a developer's own App Store Connect
metadata, lets them translate and edit every localizable field, and writes it to
their **editable draft**. When Vernac's MCP endpoint is connected, you can drive
all of that for the user with a fixed set of tools (names prefixed by the
connection, e.g. `vernac`). This skill tells you how to do it well and safely.

## The prime directives (never break these)

1. **Never submit the app for review.** Vernac has no submit capability and you
   must not try to add one. Your job ends at writing to the **draft**; the user
   always does the final "Submit for Review" themselves in App Store Connect.
2. **Never publish without showing the diff and getting the user's explicit OK.**
   Always run a dry run first, show the user exactly what will change, and only
   execute the write after they confirm.
3. **Preserve by default.** Only change the fields the user actually asked you to
   change. Vernac writes only modified fields; don't touch anything else.
4. **Act only on the app the user named.** Confirm the loaded app is the one they
   mean before editing — the account may contain many apps.
5. **Respect the user's content.** Translate meaning, not word-for-word. Keep
   brand names, product names, and URLs intact. Flag anything you're unsure of.

## The core workflow

```
list_apps → load_app → get_status / get_listing → (edit / translate) →
get_diff → publish (dry run) → [user confirms] → publish (with code)
```

1. **`list_apps`** — list the account's apps; find the one the user means and get
   its `app_id`.
2. **`load_app`** with that `app_id` — loads its metadata as the working
   baseline. It returns the locales, platforms, per-locale status, and whether
   the app has Game Center content. **Load once** at the start (see the
   important note below).
3. **Read** the current state: `get_status` for per-locale progress,
   `get_listing` for every field's value / character count / limit / lock state,
   `get_controls` for locks and exclusions, `get_game_center` for Game Center
   text.
4. **Edit / translate** (see the sections below).
5. **`get_diff`** — the exact set of changes a publish would write. Show this to
   the user.
6. **`publish` with no confirmation code** — a **dry run**. It returns the plan
   plus a short confirmation code, and writes nothing.
7. Show the user the plan and **ask them to confirm.**
8. **`publish` with the confirmation code** — executes the write to the draft.
   Report exactly what was written and anything that failed.

### ⚠️ Load once, then stage and publish

`load_app` re-fetches from App Store Connect and **replaces the working state,
discarding any unpublished edits.** So: load the app **once**, make all your
edits, then publish. Do **not** call `load_app` again in the middle to "re-check"
— you'll lose staged work. (If the user genuinely wants to abandon pending edits
and reload, `load_app` accepts a `discard: true` flag; use it only on purpose.)

## Editing text

- **`set_field`** sets one listing field for one locale: `field`, `locale`, and
  `value` (omit `value` to clear). For an app that targets more than one
  platform, pass `platform`. Fields: `name`, `subtitle`, `description`,
  `keywords`, `promotionalText`, `whatsNew`, `marketingURL`, `supportURL`.
- **`revert_field`** / **`revert_all`** undo staged edits.
- The **app name is locked** ("do not localize") by default, and shouldn't be
  translated unless the user explicitly wants a localized name. Locks are managed
  with `set_lock`.

### Character limits (App Store Connect counts CHARACTERS)

Every field has the same limit in every language — CJK/Arabic/Thai characters
count as 1 each, so non-Latin scripts are not penalized:

| Field | Limit | | Field | Limit |
|---|---|---|---|---|
| App Name | 30 | | Promotional Text | 170 |
| Subtitle | 30 | | Description | 4000 |
| Keywords | 100 | | What's New | 4000 |

**Count before you write.** Subtitle and keywords are the ones that overflow when
translated (especially German and French). Vernac won't publish an over-limit
field — trim it to fit. For **keywords**, use comma-separated terms with no
spaces after the commas, no duplicates, and localize the *concepts* real
developers in that language would search (you may keep universal tokens like
`i18n`/`l10n`).

## Translating

You have two paths, and the best results usually combine them:

- **You (the AI) translate directly**, then `set_field` per locale. This is best
  for the short, brand-critical fields (subtitle, keywords, promotional text) and
  for any language. Produce natural, market-appropriate marketing copy — never a
  literal translation. Use Apple's official localized terminology for platform
  concepts.
- **`translate`** uses Apple's **on-device** translator to stage a first draft for
  a locale (`locale`, optional `platform`, optional `overwrite`). It's great for
  long-form fields but tends to overflow the short ones and marks everything
  "needs review." After running it, **read the result and polish it** — treat it
  as a draft, not a final.

**Rule of thumb:** draft the long fields with `translate`, write the short
marketing-critical fields yourself, and always review before publishing.

### Adding a language

To localize into a locale the app doesn't have yet, call **`add_locale`** with
the App Store locale code (e.g. `de-DE`, `es-ES`, `pt-BR`, `zh-Hans`), then
`set_field` its fields. Publishing creates the new localization on App Store
Connect.

## Game Center

If the app has Game Center content, you can localize achievement and leaderboard
text with `set_gc_field` / `revert_gc_field`, create entities with
`add_gc_achievement` / `add_gc_leaderboard` / `add_gc_leaderboard_set`, and edit
attributes with `update_gc_*` / `set_gc_achievement_points`. Read the current
state first with `get_game_center` to get entity ids.

## Things App Store Connect will reject (handle gracefully)

- **What's New on a first release.** Release notes only exist for updates; App
  Store Connect rejects `whatsNew` on an initial version. Vernac blocks it so the
  rest of the fields still publish — don't try to force it.
- **A field that changed on App Store Connect since load** ("server changed").
  Someone edited it outside Vernac. Tell the user, reload the app, review the
  diff again, then publish — this avoids overwriting their outside edit.
- **Over-limit fields.** Reported as blocked; trim to fit, then publish.

## Snapshots

Before a large batch of edits you can take a `take_snapshot` (a local restore
point); `list_snapshots` / `restore_snapshot` bring it back. A publish also takes
one automatically.

## Getting exact tool parameters

The connection exposes the authoritative tool list and JSON schemas via the
MCP `tools/list` — consult it if you need the precise parameters for a tool.
A reference summary is in [references/mcp-tools.md](references/mcp-tools.md).

## Good habits

- Start by telling the user what you're about to do, in one line.
- After edits, summarize per language (what changed, any fields still over limit
  or needing review) before you publish.
- When you publish, report the real result: how many fields were written and any
  that failed, with the reason. Never claim success you didn't verify.
- If the user hasn't connected Vernac's MCP yet, point them to Vernac's Welcome
  screen (or menu-bar item), which shows a copy-ready config for Claude.
