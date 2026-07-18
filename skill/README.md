# Vernac Claude skill

A downloadable skill that makes Claude an expert at driving **Vernac** — so you
can localize your App Store Connect listing just by asking. It teaches Claude
Vernac's workflow, its safety rules (draft‑only, never submits), the character
limits, a good translation strategy, and every MCP tool.

It's generic: it works for **any** developer's apps, not a specific one.

## What you need

1. **Vernac**, running on your Mac, connected to your App Store Connect account.
2. **Vernac's MCP endpoint connected to Claude.** Vernac's Welcome screen (and
   its menu‑bar item) shows a copy‑ready config for both Claude Code and Claude
   Desktop, pre‑filled with your endpoint and access token. Add it to your
   client first.
3. **This skill**, installed into Claude (below).

## Install

The skill is the `vernac/` folder next to this README (`SKILL.md` plus a
`references/` folder).

### Claude Code (CLI, VS Code, JetBrains)

Copy the `vernac` folder into your skills directory:

- **For all your projects** (personal):
  ```
  ~/.claude/skills/vernac/
  ```
- **For one project** (share it with a team via the repo):
  ```
  <your-project>/.claude/skills/vernac/
  ```

So `SKILL.md` ends up at `~/.claude/skills/vernac/SKILL.md`. That's it — Claude
Code discovers it automatically. You can invoke it explicitly with `/vernac`, or
just start asking Claude to localize your listing and it will use it.

Quick copy (personal install):
```bash
mkdir -p ~/.claude/skills
cp -R vernac ~/.claude/skills/vernac
```

### Claude Desktop / claude.ai

Add it as a skill in your Claude settings' Capabilities/Skills area (zip the
`vernac` folder if an upload is requested). Then connect Vernac's MCP endpoint as
described on Vernac's Welcome screen.

## Try it

Once Vernac's MCP is connected and the skill is installed, ask Claude things
like:

- "Load my app in Vernac and show me which languages still need work."
- "Translate my listing into German, French, and Spanish, then show me the diff."
- "Add Japanese and localize the description and keywords — keep the app name in English."
- "Trim any subtitle that's over the 30‑character limit."
- "Review everything that would change, then publish it to my draft."

Claude will always show you the diff and ask before writing anything, and it will
never submit your app — that final step is always yours in App Store Connect.

## Support

Questions: **pacific.61-roaring@icloud.com**
