# Vernac Support

Welcome to the Vernac support page. Vernac is a native macOS app for App Store developers: it loads your app's localizable App Store Connect metadata, lets you translate and edit every field side‑by‑side with your source language, shows you exactly what will change, and writes it to your editable draft — in every language, without the copy‑paste.

If your question isn't answered below, email **pacific.61-roaring@icloud.com** — the developer reads every message personally.

## Contact

- **Email:** pacific.61-roaring@icloud.com
- **Response time:** Typically within 1–2 business days. Critical issues (failed writes, connection problems, anything touching your live listing) are prioritized.
- **What to include:** Your Vernac version, your macOS version, the app/locale you were working on, and a description of what you were doing when the issue occurred. Screenshots help.

## What Vernac Does (and Doesn't) Do

- Vernac writes **only** to your **editable/draft** App Store Connect version.
- Vernac **never submits your app for review.** There is no submit button — the final "Submit for Review" is always yours to do in App Store Connect.
- Vernac is **preserve‑by‑default:** only the fields you actually change are ever written. Everything else is left exactly as it is.
- Before anything is written, Vernac shows you a **field‑by‑field diff** and asks you to confirm.

## Getting Started

### 1. Connect App Store Connect
Vernac needs an App Store Connect API key to read and write your own metadata.

1. In your browser, go to **App Store Connect → Users and Access → Integrations (Keys)**.
2. Generate an API key with the **App Manager** role (or higher) — this role can edit app metadata.
3. Download the `.p8` private key file (Apple lets you download it only once) and note the **Key ID** and **Issuer ID**.
4. In Vernac, click **Connect App Store Connect**, enter your Issuer ID and Key ID, and choose the `.p8` file.

Your key is stored in your Mac's Keychain and is used only to talk to Apple. See the [Privacy Policy](vernac-privacy-policy.md).

### 2. Load an app
Pick an app from the sidebar. Vernac fetches its current listing, screenshots, and Game Center content live from App Store Connect. Each locale shows a clear status: **not started, in progress, needs review, complete**.

### 3. Localize
For each language, edit fields side‑by‑side with your source text. You can:
- **Translate on‑device** with Apple's Translation framework (one click, private, instant).
- **Bring your own AI assistant** via the MCP endpoint for the short, brand‑critical fields.
- **Type or paste** anything by hand.

Live character counts and limit warnings keep you inside App Store Connect's limits.

### 4. Review and push
Open **Review changes** to see the exact diff of everything that will be written. Confirm, and Vernac writes it to your **draft**. You then do the final Submit yourself in App Store Connect.

## Explore Without Connecting

Not ready to connect a key? Choose **Explore with sample data** on the welcome screen. Vernac loads three fictional sample apps so you can try every feature — editing, translating, status tracking, Game Center, the review diff — with no App Store Connect account and none of your real data.

## Translation: On‑Device vs. an AI Assistant

Vernac gives you two complementary ways to translate.

- **Built‑in Translate (Apple's on‑device translator):** private, instant, and great for the bulk of your copy — descriptions, promotional text, what's‑new. Everything it produces is marked **"needs review."** Short fields (subtitle, keywords) can overflow their character limit when translated, especially in German and French; Vernac flags anything over the limit and **won't** publish it until it fits.
- **An AI assistant via MCP:** best for short, brand‑critical copy (subtitle, keywords) where tone, nuance, and staying inside the limits matter most.

**Rule of thumb:** draft everything with Translate, polish the short marketing‑critical fields with an AI assistant, and always review before you publish.

### About the app name
By default Vernac **locks your app name** as "do not localize" — machine translation of a brand name is unreliable. Unlock a locale only if you deliberately want a localized name (common in some CJK markets).

### Character limits
App Store Connect counts **characters** (every CJK/Arabic/Thai character counts as 1 — non‑Latin scripts are not penalized), with the same limit in every language:

| Field | Limit |
|---|---|
| App Name | 30 |
| Subtitle | 30 |
| Keywords | 100 |
| Promotional Text | 170 |
| Description | 4000 |
| What's New | 4000 |

## Connecting an AI Assistant (MCP)

Vernac can host a local **Model Context Protocol** endpoint so an AI assistant like Claude can use every one of Vernac's tools on your behalf.

### How to connect
The Welcome screen (and Vernac's menu‑bar item) shows a **copy‑ready config** filled in with your endpoint and access token, for both Claude Code and Claude Desktop:

- **Claude Code:**
  ```
  claude mcp add --transport http vernac http://127.0.0.1:8376/mcp \
    --header "Authorization: Bearer YOUR_TOKEN"
  ```
- **Claude Desktop** (via the `mcp-remote` bridge, which needs Node.js):
  ```json
  {
    "mcpServers": {
      "vernac": {
        "command": "npx",
        "args": ["-y", "mcp-remote@latest", "http://127.0.0.1:8376/mcp",
                 "--allow-http", "--header", "Authorization: Bearer YOUR_TOKEN"]
      }
    }
  }
  ```

Then just ask your assistant to load your app and start localizing. To make Claude an expert at Vernac, install the [Vernac Claude skill](skill/).

### What the AI can do
The endpoint exposes a fixed set of tools covering discovery, reads, edits, translation, locale/platform control, Game Center, snapshots, and the review‑and‑publish flow. Examples:
- "Load my app and show me which languages still need work."
- "Translate my listing into German, French, and Japanese, then show me the diff."
- "Add Spanish (Mexico) and localize the description and keywords."
- "Trim any subtitle that's over the 30‑character limit."
- "Review everything that would change, then publish it to my draft."

Everything the assistant can do, you can also do by hand in the app — the MCP endpoint drives the same engine.

### Security
The endpoint binds **only** to `localhost`, requires your per‑install token, exposes a **fixed** tool set (no arbitrary code), and is never reachable from your network or the internet. Your App Store Connect key stays inside Vernac and is never shared with the assistant.

## Game Center

Beyond the store listing, Vernac localizes your **Game Center** achievements, leaderboards, and leaderboard sets — the parts most tools ignore — and can create new achievements, leaderboards, and sets, edit their attributes, and upload per‑locale achievement images.

## Screenshots

Vernac manages your screenshots per language and lets you reuse a shared set across locales. Screenshots are added from files you choose on your Mac.

## Troubleshooting

- **"Not connected" after an update or rebuild.** Reconnect from within Vernac (re‑select your `.p8`). Your key is safe in the Keychain; a reconnect re‑establishes access.
- **A field won't publish / is flagged over the limit.** It exceeds App Store Connect's character limit for that language. Shorten it (or ask an AI assistant to), and it will publish once it fits.
- **"What's New" won't write on a first release.** App Store Connect only accepts release notes for **updates**, not an initial version. Vernac blocks it for a first release so the rest of your fields still publish; add What's New on your next update.
- **"The app changed in App Store Connect since Vernac loaded it."** Someone (or another tool) edited the value on App Store Connect after Vernac loaded it. Reload the app, review the diff again, then publish — this protects you from overwriting an outside edit.
- **On‑device translation returns "not supported" for a language.** Language availability depends on your macOS version and grows with each update. For a language Apple can't translate on‑device, translate the text yourself (or with an AI assistant) and paste it in.
- **My AI assistant can't connect.** Confirm Vernac is running, the port matches (default 8376), and you pasted the exact `Authorization: Bearer` token from Vernac. Claude Desktop needs the `mcp-remote` bridge (and Node.js) to reach a local HTTP endpoint.

## Frequently Asked Questions

**Does Vernac ever submit my app?**
No. Vernac writes only to your editable draft and has no submit function. You always do the final Submit for Review yourself in App Store Connect.

**Can Vernac see or change my live (published) listing?**
No. Vernac writes to the editable/draft version only. Fields that can't be edited in the current state are shown as read‑only.

**Where do my App Store Connect credentials go?**
Into your Mac's Keychain, device‑only. They're used only to sign requests sent directly to Apple. They're never sent to the developer or any third party.

**Is there an iOS version?**
No. Vernac is a native macOS app.

**Does Vernac use iCloud?**
No. Vernac stores nothing in iCloud and does no cross‑device sync. Your listing content lives in your App Store Connect account; your key and settings live on your Mac.

**Do I have to use an AI assistant?**
No. Everything works entirely in the app. The MCP endpoint is an optional convenience for driving Vernac from an assistant.

**I found a bug or have a feature request. Where do I send it?**
Email **pacific.61-roaring@icloud.com**.

## System Requirements

- **Mac:** macOS 15.0 (Sequoia) or later
- **App Store Connect:** an API key with the App Manager role (or higher)
- **On‑device translation:** availability depends on your macOS version
- **AI assistant (optional):** any MCP‑compatible client; Claude Desktop additionally needs Node.js for the `mcp-remote` bridge

---

*Vernac is built and maintained by Jeremy Littlewood. Last updated July 18, 2026.*
