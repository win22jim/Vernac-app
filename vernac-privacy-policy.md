# Privacy Policy for Vernac

**Effective Date:** July 18, 2026
**Last Updated:** September 30, 2026

This Privacy Policy explains how Vernac ("the app", "we", "us") handles your data. Vernac is a native macOS app that helps App Store developers localize their own App Store Connect listing metadata across languages. The app is published by Jeremy Littlewood ("the developer").

## Summary

**Vernac does not collect, transmit, or sell any of your personal data to the developer or to any third party. The developer operates no servers and receives nothing from the app.** Vernac talks to two things, both on your behalf and with your explicit setup: Apple's App Store Connect API (using your own credentials), and Apple's on‑device Translation framework (which runs locally on your Mac). Your credentials and your content stay on your Mac and travel only between your Mac and Apple.

If that summary is all you need, you can stop reading here. The rest of this document explains the details.

## 1. What the App Stores on Your Mac

Vernac keeps very little on disk, all of it local to your Mac:

- **Your App Store Connect API key** (Key ID, Issuer ID, and the `.p8` private key), stored in the macOS **Keychain**.
- **A local access token** for the app's optional MCP endpoint (see Section 4), also stored in the Keychain.
- **Your settings and per‑app controls** — such as how each field is handled (localize, leave as‑is, or the same in every language), which locales or platforms you've excluded, your do‑not‑translate terms, your chosen source language, and the app window size — stored in the standard macOS preferences (`UserDefaults`).
- **A read‑only bookmark to the one media folder you choose** (or open in Vernac) for screenshots, app previews and Game Center images, so Vernac can read it again after a relaunch. It lets Vernac read that folder and nothing else; Vernac never writes to it. Forget the folder in Vernac to remove it.

Vernac does **not** maintain its own database of your listing content. When you open an app, Vernac fetches its current metadata **live** from App Store Connect and holds your edits in a temporary in‑memory workspace. Those edits — and any screenshots, app previews or images you stage — exist only until you publish or upload them to your App Store Connect draft (or discard them, or quit Vernac). Vernac uses **no iCloud, no CloudKit, and no cross‑device sync** — nothing you do in Vernac is replicated anywhere off your Mac by the app.

## 2. What the App Does NOT Collect

The app does **not** collect, transmit, or process any of the following:

- Personal identifiers (name, email address, phone number, government ID, etc.)
- Advertising or tracking identifiers (IDFA, device fingerprints, etc.)
- Location, contacts, calendar, photos, microphone, or camera data
- Health, fitness, or biometric data
- Your sales, revenue, or financial information
- Usage analytics, crash telemetry, or feature‑engagement data
- Any data sent to the developer's servers — because the developer operates none

The app contains **no** third‑party analytics SDKs, **no** advertising SDKs, **no** crash‑reporting SDKs, **no** attribution SDKs, and **no** tracking of any kind.

## 3. App Store Connect Integration

Vernac's core function requires the App Store Connect API so it can read and write your own app's localizable metadata. This requires you to provide your own App Store Connect API key.

- Your API key (Key ID, Issuer ID, and `.p8` private key) is stored in your Mac's **Keychain**, encrypted by the operating system. It is marked **device‑only and non‑synchronizable** (`kSecAttrAccessibleWhenUnlockedThisDeviceOnly`), so it is **not** copied to your iCloud Keychain or any other device.
- The key is used only to sign short‑lived JSON Web Tokens (JWTs) that authenticate requests sent **directly from your Mac to Apple's official `api.appstoreconnect.apple.com` endpoint**.
- Your API key is never transmitted to the developer, never stored in plain text, never written to logs, and never sent to any third party.
- Vernac writes only to your **editable/draft** App Store Connect version. **It never submits your app for review** — there is no submit function in the app; the final "Submit for Review" is always an action you take yourself in App Store Connect.

## 4. Model Context Protocol (MCP) Endpoint

Vernac can host a local Model Context Protocol (MCP) endpoint so that an AI assistant such as Claude — or any MCP‑compatible client running on your own Mac — can drive the very same features Vernac offers, on your behalf.

- The endpoint binds **only to the loopback interface of your Mac** (`http://127.0.0.1:<port>/mcp`, default port 8376). It does **not** accept connections from your local network or the internet, and the developer cannot reach it.
- Every request must present a **per‑install access token** that Vernac generates randomly and stores in your Keychain; requests without it are rejected. This token only unlocks the local endpoint — it is **not** your App Store Connect key, and your App Store Connect credentials are never exposed to the MCP client.
- The endpoint exposes a **fixed, published set of tools** and cannot run arbitrary code, shell commands, or scripts.
- Any data an MCP client receives is governed by **that client's** own privacy policy. For example, if you use a cloud‑hosted AI assistant, the text it reads from your listing is handled under that assistant's terms, not this one. You choose which client to connect and what to ask it to do.

You can change the port from Vernac's menu‑bar item.

## 5. On‑Device Translation

When you use Vernac's built‑in translation, it uses Apple's **on‑device Translation framework**. Translation runs locally on your Mac; the text of your listing is not sent to the developer, and language availability is checked on‑device. (Some languages may prompt macOS to download a language model from Apple the first time you use them; that download is between your Mac and Apple.)

## 6. Diagnostic Logs

Vernac writes limited diagnostic information — for example, about App Store Connect API calls and sync/connection events — to your Mac's standard system log (Apple's unified logging). These logs stay on your Mac, are accessible only to you, and are not transmitted to the developer.

## 7. Children's Privacy

Vernac is a professional developer tool intended for adults who publish apps on the App Store. It is not directed at children under the age of 13 and does not knowingly collect any personal information from anyone.

## 8. Data Retention and Deletion

Because the developer has no access to your data, the developer cannot delete data on your behalf. You are in control:

- **Your API key and access token:** Remove your key from within Vernac, or delete the corresponding items from Keychain Access. Deleting the app removes its access to them.
- **Your settings:** Deleting the app removes its local preferences from your Mac.
- **Your listing content:** Lives in **your** App Store Connect account, under your control at all times. Vernac only ever writes to your editable draft; you can review, change, or discard anything in App Store Connect directly.

## 9. Security

Vernac relies on Apple‑provided security primitives:

- App Store Connect credentials and the MCP access token are stored in the macOS Keychain, device‑only.
- All App Store Connect traffic is TLS‑encrypted and goes directly to Apple's official endpoint.
- The MCP endpoint binds to the loopback interface only and requires a per‑install token.
- Vernac is sandboxed and requests only the minimum capabilities it needs (read‑only access to the files and the one folder you choose or open in Vernac, outgoing network to Apple, and its own loopback server).

No system is perfectly secure. If you discover a security vulnerability in Vernac, please contact the developer at the email address in Section 11 before disclosing it publicly.

## 10. Changes to This Policy

If this Privacy Policy is updated, the new version will be published at the same URL as this document, and the "Last Updated" date at the top will change. Material changes will also be noted in the app's release notes for the version in which they take effect.

## 11. Contact

If you have questions about this Privacy Policy, contact:

**Jeremy Littlewood**
Email: pacific.61-roaring@icloud.com

The developer typically responds within a few business days.
