# cursor-david-harness

DAVID agent harness for Cursor — organizational memory + MR review across all repos.

## What this plugin does

**Two always-on rules** that apply to every agent session in every repo:

| Rule | What it does |
|------|-------------|
| `brain-harness` | Brain-first protocol: recall at session start, remember at session end, feedback on every memory. Brings organizational memory to all repos. |
| `hard-rules` | Universal TypeScript standards: no `any`, no `throw`, no `console.log`, no `process.env`, typed clients for service calls. |

**MR review system** — 4 specialized personas for GitLab merge request review:

| Persona | Skill | Focus |
|---------|-------|-------|
| Code Reviewer | `/mr-code-review` | Logic, performance, tests, patterns |
| Compliance Auditor | `/mr-compliance-audit` | VARA, ISO 27001, SOC 2 with section refs |
| Security Scanner | `/mr-security-scan` | SAST (CWE), SCA, secrets, false positive learning |
| Standards Guardian | `/mr-standards-guard` | ADR citations, proto breaking, incident history |

**Entry points:**
- Agent: `@mr-reviewer Review MR !123 in cfd-bank/saas`
- Command: `/review-mr cfd-bank/saas 123`
- Skill: `/mr-review-orchestrator` (for direct skill invocation)

## How it applies to all repos

The `brain-harness` and `hard-rules` rules have `alwaysApply: true`. When the plugin is installed (or set as **Required** via Team Marketplace), these rules inject into the model context for every chat session — regardless of which repo is open.

Skills, agents, and commands are available everywhere too. A dev working in `payments-service` can run `@mr-reviewer` even though `payments-service` has no DAVID-specific `.cursor/` config.

## Prerequisites

- **DAVID brain-mcp** running (local or production)
- **GitLab access token** with `api` scope

## Environment variables

```bash
BRAIN_MCP_API_KEY=<brain-mcp API key>
BRAIN_MCP_URL=http://localhost:5053/mcp    # optional, defaults to localhost
GITLAB_TOKEN=<GitLab personal/project/group access token>
GITLAB_API_URL=https://gitlab.umo.tech/api/v4  # optional
```

For Cursor Cloud Agents, add these in the Secrets tab.

### API key via env (required for `mcp.json` interpolation)

Cursor resolves **only** the documented form `${env:VAR}` in `url` and `headers` — not `${VAR}` and not shell-style `$VAR`. See [Cursor MCP — Config interpolation](https://cursor.com/docs/context/mcp#config-interpolation).

The plugin’s `mcp.json` uses:

- `"Authorization": "Bearer ${env:BRAIN_MCP_API_KEY}"`

**Important:** On macOS/Linux, **GUI Cursor does not load `~/.zshrc`**. Exporting the key only in zsh leaves the IDE process without `BRAIN_MCP_API_KEY`, so interpolation expands to empty and brain-mcp returns `missing_token`.

Set the variable where the **Cursor app** inherits it, for example:

- **macOS:** `~/.zshenv` (loaded for non-interactive contexts), or launch Cursor from a terminal where the var is set, or set a user-level env via your org’s MDM / `launchctl` (see Apple docs for `LaunchAgents` / environment).
- **Linux (desktop):** `~/.profile`, systemd user environment, or `/etc/environment` — whichever applies to how you start Cursor.
- **Override:** Duplicate the same `headers` block in **`~/.cursor/mcp.json`** (global) or **`.cursor/mcp.json`** (project) after confirming `${env:BRAIN_MCP_API_KEY}` works in **Output → MCP** (no empty Bearer).

After changing env vars, restart Cursor or **MCP: Restart All MCP Servers**.

### OAuth (Google) vs API key

- **Bearer API key:** brain-mcp validates the plaintext key against configured API key hashes — use `${env:BRAIN_MCP_API_KEY}` as above.
- **OAuth:** Cursor follows MCP OAuth discovery against `https://mcp.umo.dev`. The access token sent to `/mcp` must be one **brain-mcp accepts** (DAVID **control-auth** JWT with audience `david-mcp`, per internal OAuth flow — see `docs/modules/knowledge/guides/mcp-client-setup.md`). If you see **“OAuth tokens saved”** then **401**, Cursor completed a browser login but the token is not accepted by brain-mcp (wrong issuer/audience, or exchange failed). That is resolved on the **DAVID auth / brain-mcp** side (redirect URI `cursor://anysphere.cursor-mcp/oauth/callback` registered in the OAuth client, token endpoint issuing the expected JWT). API key auth does not depend on that path.

Using a **static `Authorization` header** (API key) is the supported path for Cursor until OAuth end-to-end is verified against production control-auth.

## Install

### Team marketplace (recommended — applies to all team members)

1. Push this directory to a GitHub repo (e.g., `umo-tech/cursor-david-harness`)
2. Cursor Dashboard → Settings → Plugins → Team Marketplaces → Import
3. Paste the GitHub repo URL
4. Set as **Required** for distribution group **"All"**
5. Every team member gets it on next Cursor restart

### Local testing

```bash
ln -s /path/to/cursor-david-harness ~/.cursor/plugins/local/cursor-david-harness
# Restart Cursor (Developer: Reload Window)
```

## Plugin contents

```
cursor-david-harness/
├── .cursor-plugin/plugin.json
├── mcp.json                            ← brain-mcp + GitLab MCP
├── rules/
│   ├── brain-harness.mdc              ← alwaysApply: true
│   ├── hard-rules.mdc                 ← alwaysApply: true
│   └── mr-review-protocol.mdc        ← findings format (active during reviews)
├── skills/
│   ├── mr-review-orchestrator/SKILL.md
│   ├── mr-code-review/SKILL.md
│   ├── mr-compliance-audit/SKILL.md
│   ├── mr-security-scan/SKILL.md
│   └── mr-standards-guard/SKILL.md
├── agents/
│   └── mr-reviewer.md
├── commands/
│   └── review-mr.md
└── README.md
```

## MCP handling

The plugin bundles `david-brain` and `gitlab` MCP server configs. If a dev already has these configured under different names (e.g., `user-david`), both coexist — the skills reference tools by name (`david_recall`, `david_remember`), not by server name.

For devs who don't have brain configured, the plugin provides it automatically.
