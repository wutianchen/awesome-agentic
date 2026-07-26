# `.omc/` Directory Reference

When Oh-My-Claude (OMC) runs in a repo, it creates a `.omc/` directory at the project root to hold runtime state, accumulated knowledge, and inter-agent communication. It's the OMC equivalent of `.git/` — machine-managed, never hand-edited, and **never committed**.

## TL;DR

| Aspect | Value |
| --- | --- |
| **Scope** | Per-project (one `.omc/` per repo root) |
| **Lifetime** | Survives sessions; persists until you delete it |
| **Authored by** | OMC hooks, skills, and MCP tools — not you |
| **Commit to git?** | **No.** Add `.omc/` to `.gitignore`. Some files contain absolute paths, session IDs, and per-machine cache. |
| **Safe to delete?** | Yes — OMC recreates what it needs. You lose accumulated project memory, plans, research, and session-replay history. |

## Top-level layout

```
.omc/
├── project-memory.json     ← cached facts about the project (tech stack, build cmds, conventions)
├── notepad.md              ← agent-managed working notes (created on first write)
├── plans/                  ← saved plans from /planner, /ralplan, etc.
├── research/               ← deep-research / deepsearch outputs
├── logs/                   ← agent logs (created on demand)
└── state/                  ← runtime state for the active session(s)
    ├── agent-replay-<sessionId>.jsonl
    ├── hud-stdin-cache.json
    ├── idle-notif-cooldown.json
    └── sessions/
        └── <sessionId>/    ← cross-agent shared state for one session
```

The set of files you'll actually see depends on which OMC features have run in your repo. A fresh setup may only contain `project-memory.json` and `state/`.

---

## File-by-file

### `project-memory.json`

**What it is:** A cached snapshot of project facts OMC has learned. Read on session start so subagents don't re-discover build/test commands, languages, and conventions every run.

**Why it exists:** Re-scanning the repo on every turn is expensive. Caching the result lets specialists like `executor` and `test-engineer` start with project context already in hand.

**Shape (from this repo):**

```json
{
  "version": "1.0.0",
  "lastScanned": 1781961022253,
  "projectRoot": "<absolute path>",
  "techStack": { "languages": [], "frameworks": [], "packageManager": null, "runtime": null },
  "build":      { "buildCommand": null, "testCommand": null, "lintCommand": null, "devCommand": null, "scripts": {} },
  "conventions":{ "namingStyle": null, "importStyle": null, "testPattern": null, "fileOrganization": null }
}
```

**How it's used:** Written by `mcp__project_memory_write`. Read by agents at the start of work. You'd update it by running OMC's project-memory scan (or via `project_memory_add_note` / `add_directive` MCP tools), not by hand.

**Why not commit:** Contains an absolute path and a host-machine timestamp. Some entries can be repo-relative knowledge, but the file as a whole is per-machine cache.

---

### `notepad.md`

**What it is:** An agent-managed scratchpad — short-lived working notes the orchestrator can re-read mid-task.

**Why it exists:** Long tasks need a place to jot intermediate findings ("found three places to update, currently at #2") that survive across subagent invocations within a session.

**How it's used:** Written via `notepad_write_manual` / `_priority` / `_working` MCP tools. Read via `notepad_read`. Pruned by `notepad_prune` when it grows. Stats by `notepad_stats`.

**Scope:** Project-local; persists across sessions but is intended as ephemeral.

---

### `plans/`

**What it is:** Saved plans produced by planning skills (`/planner`, `/ralplan`, etc.).

**Why it exists:** Lets you (and the agent) refer back to a previously-approved plan, resume an interrupted execution, or diff old vs. new strategies.

**How it's used:** Plans are written as markdown files when the planner skill commits to an approach. Listed via `state_list_active`. The orchestrator may read the active plan to keep execution on rails.

---

### `research/`

**What it is:** Outputs from research skills — `/deep-research`, `/deepsearch`, and the OMC `scientist` agent.

**Why it exists:** Research output is often long. Saving it on disk avoids re-running expensive web sweeps and lets later subagents cite earlier findings without paying their token cost.

**How it's used:** Reports are written as markdown or JSON. Indexed by the wiki tools (`wiki_ingest`) if you want them searchable.

---

### `logs/`

**What it is:** Agent execution logs — when OMC chooses to write them.

**Why it exists:** Debugging multi-agent runs needs an audit trail. Logs are also a sensor (in harness-engineering terms) — the human's outer feedback loop reads them when something goes wrong.

**How it's used:** Written automatically by hooks during long-running modes (ralph, ultrawork). Inspected via `trace_summary` / `trace_timeline` MCP tools.

---

### `state/` — runtime state

The active-session machinery. Everything here is throwaway after a session ends, but OMC keeps a few items around for resume/replay.

#### `state/agent-replay-<sessionId>.jsonl`

JSONL log of every subagent invocation in this session — agent type, duration, success/failure. One line per event:

```json
{"t":0,"agent":"ad5e2d7","agent_type":"unknown","event":"agent_stop","success":true,"duration_ms":0}
```

Powers the `trace_*` MCP tools (timeline, summary) and lets the orchestrator answer "what have I done so far this session?" without scanning the whole transcript.

#### `state/hud-stdin-cache.json`

Cached input for the OMC HUD (status display). Captures `session_id`, `transcript_path`, `cwd`, current `effort` setting, etc. — basically the data the HUD needs to render without re-querying the harness every tick.

Per-session, per-machine. Useless after the session ends; safe to delete anytime.

#### `state/idle-notif-cooldown.json`

Throttle file for idle notifications — records the last time OMC sent an idle-ping so it doesn't spam you:

```json
{ "lastSentAt": "2026-06-20T13:13:38.906Z" }
```

#### `state/sessions/<sessionId>/`

Per-session shared workspace for cross-agent communication. When one subagent writes something the next subagent needs (e.g. `planner` produces a plan that `executor` consumes), it lands here.

Backed by the `state_read` / `state_write` MCP tools.

---

## Scope summary

| Path | Lives in | Shared across sessions? | Shared across projects? | Commit? |
| --- | --- | --- | --- | --- |
| `.omc/project-memory.json` | Repo root | Yes | No | No |
| `.omc/notepad.md` | Repo root | Yes (intended ephemeral) | No | No |
| `.omc/plans/` | Repo root | Yes | No | No |
| `.omc/research/` | Repo root | Yes | No | No |
| `.omc/logs/` | Repo root | Yes | No | No |
| `.omc/state/sessions/<id>/` | Repo root | No (per session) | No | No |
| `.omc/state/agent-replay-<id>.jsonl` | Repo root | No (per session) | No | No |
| `.omc/state/hud-*.json`, `idle-*.json` | Repo root | No (transient) | No | No |
| Cross-project shared memory | `~/.claude/...` (not `.omc/`) | Yes | Yes | N/A |

## Does `.omc/` have user / local scopes like `.claude/`?

**No — `.omc/` is effectively project-scope only.** Unlike `.claude/`, there's no layered user/project/local config you author into.

### What actually exists on disk

| Path | Contents | Role | Equivalent in `.claude/` |
| --- | --- | --- | --- |
| `<repo>/.omc/` | `project-memory.json`, `state/`, `notepad.md`, `plans/`, `research/`, `logs/` | Per-project runtime state | `<repo>/.claude/` |
| `~/.omc/` | Empty `state/` dir on your machine | Fallback parking spot when OMC runs outside any detected project root | Loosely `~/.claude/` — but **not** a config layer |
| `~/.claude/.omc/` | `update-check.json` only | OMC plugin's self-update bookkeeping | None — plugin-internal |
| `<parent-dir>/.omc/` (e.g. `~/Projects/.omc/`) | `sessions/`, `state/` | Created when OMC's project-root detection lands on a parent dir of a repo (running `claude` from `~/Projects` instead of inside a repo) | None |
| `<repo>/.omc.local/` | **Does not exist** | — | `.claude/settings.local.json` has no OMC counterpart |

### Why no layered scopes

| `.claude/` | `.omc/` |
| --- | --- |
| **Config** — you edit it | **State** — OMC writes it |
| 3 scopes (user / project / local), shallow-merged | Effectively 1 scope (project) |
| Project layer designed for git sharing | Designed to be gitignored everywhere |
| Layering lets the team share policy while you keep tweaks out of git | Every `.omc/` is already per-machine; layering has no meaning |

The reason `.claude/` needs scopes is that *you author into it* — settings, hooks, permissions — and the team needs to share some while you keep others private. OMC's `.omc/` is machine-managed runtime state. There's nothing for layering to do.

### Where OMC's user-scope *behavior* actually lives

If you want "OMC settings that apply to every project," they live in the `~/.claude/` surface OMC is a plugin of — not in `~/.omc/`:

| Concern | File / mechanism |
| --- | --- |
| Personal OMC delegation rules, agent preferences | `~/.claude/CLAUDE.md` (global) |
| Cross-project shared memory | `shared_memory_*` MCP tools — stored under `~/.claude/plugins/data/oh-my-claudecode-omc/` |
| Per-project, cross-session structured memory | `~/.claude/projects/<project>/memory/` |
| OMC plugin install / update bookkeeping | `~/.claude/.omc/`, `~/.claude/plugins/` |
| Kill switches | Env vars: `DISABLE_OMC`, `OMC_SKIP_HOOKS` (comma-separated) |

### TL;DR

- `.omc/` = single project scope. The `~/.omc/` and `~/.claude/.omc/` you may see on disk aren't configuration scopes — they're a fallback parking spot and plugin-internal bookkeeping.
- For per-user OMC behavior, edit `~/.claude/CLAUDE.md` or use `shared_memory_*` MCP tools.
- For per-machine secrets/overrides at the Claude-Code layer, use `<repo>/.claude/settings.local.json` (a `.claude/` feature, not an OMC one).

## What it doesn't include

These live elsewhere, not in `.omc/`:

| Concern | Where |
| --- | --- |
| Auto-memory (`feedback`, `user`, `project`, `reference` types) | `~/.claude/projects/<project>/memory/` |
| Cross-session / cross-project shared memory | Accessed via `shared_memory_*` MCP tools; stored in OMC's plugin dir |
| Wiki knowledge base | `wiki_*` MCP tools; backed by SQLite in OMC's plugin dir |
| Claude Code settings | `~/.claude/settings.json` and `<repo>/.claude/settings.json` |
| Project context for the model | `<repo>/CLAUDE.md` |

## Should I gitignore it?

Yes. Add to `.gitignore`:

```
.omc/
```

The directory contains absolute paths, session IDs, cache timestamps, and machine-specific HUD/notification state. None of it is meant to be shared. If you want to *share* project knowledge with the team, put it in `CLAUDE.md` (human-curated) — not `.omc/project-memory.json` (machine-managed cache).

## Can I delete it?

Yes, anytime OMC isn't actively running. You'll lose:

- Cached project-memory scan (re-runs on next setup)
- Notepad contents
- Saved plans and research outputs
- Session replay history

You won't lose: code, auto-memory (lives in `~/.claude/`), CLAUDE.md, or settings.
