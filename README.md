# UMO Cursor Plugins

Internal Cursor team marketplace for UMO. Brain harness, MR review, organizational memory.

## Plugins

| Plugin | What it does |
|--------|-------------|
| [cursor-umo-brain](cursor-umo-brain/) | DAVID brain harness + 4-persona MR review + GitLab integration |

## Install (Team Marketplace)

1. Cursor Dashboard → Settings → Plugins → Team Marketplaces → Import
2. Paste this GitHub repo URL
3. Set plugins as Required or Optional per distribution group

## Adding a new plugin

1. Create `<plugin-name>/` directory with `.cursor-plugin/plugin.json`
2. Add rules, skills, agents, commands, mcp.json as needed
3. Add entry to `.cursor-plugin/marketplace.json` → `plugins` array
4. Push to main
