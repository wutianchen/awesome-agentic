# Configuration

Three scopes determine where each config file lives, who it applies to, and whether it gets committed.

## Scopes at a glance

| Scope | Lives in | Applies to | Checked into git? |
| --- | --- | --- | --- |
| **User** | `~/.claude/` | You, every project | No |
| **Project** | `<project>/.claude/`, `<project>/` | Anyone on the team | **Yes** |
| **Local** | `<project>/.claude/*.local.json`, `<project>/CLAUDE.local.md` | You, this project only | No (gitignored) |

Precedence (highest wins): **Enterprise → Local → Project → User**. Merging is shallow per top-level key — a list in a higher layer **replaces**, not merges, the same list in a lower layer.

---

## User scope — `~/.claude/`

Personal defaults across every project.

| File / dir | Purpose |
| --- | --- |
| `~/.claude/settings.json` | Global config: model, permissions, hooks, MCP servers, env. |
| `~/.claude.json` | **State** (not config) — managed by Claude Code. Auth, history, trust dialog state, per-project map. **Never commit.** |
| `~/.claude/CLAUDE.md` | Global context Claude reads in every project. |
| `~/.claude/commands/` | Personal slash commands. |
| `~/.claude/skills/` | Personal skills. |
| `~/.claude/agents/` | Personal subagent definitions. |
| `~/.claude/memory/` | Persistent memory store (per-project subdirs). |
| `~/.claude/plugins/` | Installed plugins. |
| `~/.claude/keybindings.json` | Custom key bindings. |
| `~/.claude/themes/` | Custom color themes. |

---

## Project scope — `<project>/.claude/` and `<project>/`

Team-shared, committed to the repo.

| File / dir | Purpose |
| --- | --- |
| `<project>/.claude/settings.json` | Team policy: shared permissions, hooks, MCP servers, model pin. |
| `<project>/CLAUDE.md` | Project context (architecture, conventions). |
| `<project>/AGENTS.md` | Alternative agent-instruction file (cross-tool standard). Often symlinked to/from `CLAUDE.md`. |
| `<project>/.claude/commands/` | Project slash commands. |
| `<project>/.claude/skills/` | Project skills. |
| `<project>/.claude/agents/` | Project subagent definitions. |
| `<project>/.mcp.json` | Project-declared MCP servers (each one requires trust approval the first time). |

---

## Local scope — machine-only overrides

Personal tweaks for one project; never committed.

| File | Purpose |
| --- | --- |
| `<project>/.claude/settings.local.json` | Override anything from project `settings.json` (e.g. extra `allow` rules you don't want to push). Auto-gitignored. |
| `<project>/CLAUDE.local.md` | Personal project notes Claude reads but the team doesn't. |

---

## `settings.json` — key reference

Same schema at user, project, and local scopes.

| Key | What it controls |
| --- | --- |
| `model` | Default model ID (`claude-opus-4-7`, `claude-sonnet-4-6`, etc.). |
| `permissions.allow` | Auto-approved tool calls. Pattern: `Tool(spec)`, e.g. `Bash(git status)`, `Edit(src/**)`, `Read`. |
| `permissions.ask` | Always prompt, even if elsewhere allowed. |
| `permissions.deny` | Always block. |
| `hooks` | Event-driven shell commands. Events: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `Notification`, `SessionStart`, `SessionEnd`. |
| `env` | Env vars injected into Claude Code's tool-execution shell. |
| `mcpServers` | Declarations for MCP servers to launch. |
| `enableAllProjectMcpServers` / `enabledMcpjsonServers` | Pre-approve MCP servers from `.mcp.json` without the trust prompt. |
| `apiKeyHelper` | Shell command that prints an API key to stdout (for rotating creds). |
| `outputStyle` | `default` / `explanatory` / `learning` / custom. |
| `statusLine` | Shell command for the bottom status line. Configure via `/statusline`. |
| `includeCoAuthoredBy` | Add `Co-Authored-By: Claude` to commits (default `true`). |

Permission pattern syntax:

| Pattern | Matches |
| --- | --- |
| `Bash(npm test)` | Exact command. |
| `Bash(git status:*)` | Prefix match. |
| `Edit(src/**)` | Glob on file paths. |
| `Read` | All calls to that tool. |

---

## `~/.claude.json` — user-scope state

| Key | Purpose |
| --- | --- |
| `userId` | Internal user ID. |
| `oauthAccount` | OAuth tokens if logged in via Anthropic account. |
| `numStartups` / `firstStartTime` | Counters and onboarding state. |
| `installMethod` | npm, brew, etc. |
| `autoUpdates` | Auto-update toggle. |
| `mcpServers` | Globally configured MCP servers (mirror of settings runtime state). |
| `tipsHistory` / `feedbackSurveyState` | UI display state. |
| `cachedChangelog` | Cached release notes. |
| **`projects`** | Map keyed by absolute project path; per-project history, trust state, MCP approvals. |

Per-project entry inside `projects`:

```json
"/Users/me/Projects/myrepo": {
  "allowedTools": ["Read", "Edit", "Bash(npm test)"],
  "history": [{ "display": "fix the auth bug" }],
  "mcpServers": { "postgres": { } },
  "enabledMcpjsonServers": ["filesystem"],
  "hasTrustDialogAccepted": true
}
```

State, not config. Edit only when you need to clear history, revoke trust (`hasTrustDialogAccepted: false`), or recover from corruption.

---

## How to edit

| Goal | Method |
| --- | --- |
| Interactive editor | `/config` |
| Scripted change | `claude config set <key> <value>` |
| Direct edit | Open the JSON file |
| Diagnose misconfig | `/doctor`, `claude --debug` |

---

## Gotchas

| Gotcha | Detail |
| --- | --- |
| **Shallow merge** | `permissions.allow` in a higher-precedence layer **replaces** the lower layer's list. Same for `hooks`. |
| **Unknown keys ignored** | Typos like `mcpServerss` fail silently. |
| **Hooks run as your user** | A freshly cloned repo's `.claude/settings.json` has the same blast radius as a `Makefile`. Audit before trusting. |
| **`CLAUDE.md` is not config** | It's context Claude reads. Different file, different purpose from `settings.json`. |
| **Never commit `~/.claude.json`** | Contains OAuth tokens. |
| **MCP servers overlap** | `settings.json` is the source of truth; `.claude.json` holds runtime trust state. `claude mcp list` shows what's actually loaded. |

---

## Minimal starter (`<project>/.claude/settings.json`)

```json
{
  "permissions": {
    "allow": [
      "Read", "Edit", "Write", "Grep", "Glob",
      "Bash(git status)", "Bash(git diff:*)", "Bash(git log:*)",
      "Bash(npm test)", "Bash(npm run build)"
    ],
    "ask": ["Bash(git push:*)", "Bash(rm:*)"],
    "deny": ["Bash(rm -rf /*)"]
  }
}
```

---

## References

- Settings — https://code.claude.com/docs/en/settings
- Hooks — https://code.claude.com/docs/en/hooks
- Permissions — https://code.claude.com/docs/en/iam
- MCP — https://code.claude.com/docs/en/mcp
- Memory — https://code.claude.com/docs/en/memory
