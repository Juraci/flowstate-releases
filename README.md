# flowstate-releases
Public binary releases for FlowState (source is private). Hosts the .pacman/.deb artifacts consumed by the flowstate-bin AUR package.

## Claude Code skills

### save-it-offline

[`skills/save-it-offline/`](skills/save-it-offline) — a [Claude Code](https://claude.com/claude-code) skill that researches the official documentation for a technology (optionally at a specific version) and saves it into your FlowState vault as linked nodes with offline content copies, via the FlowState MCP server.

Install by copying it into your personal skills directory:

```bash
mkdir -p ~/.claude/skills
cp -r skills/save-it-offline ~/.claude/skills/
```

Then, in a session with the FlowState MCP connected:

```
/save-it-offline <technology> [version]   # e.g. /save-it-offline zod 3
```

Requires FlowState's MCP server to be enabled (Settings → MCP) and connected to Claude Code; the skill checks this before doing anything. When no version is given it resolves the latest stable release.
