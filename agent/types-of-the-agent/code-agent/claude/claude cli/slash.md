# Slash Commands

## ⭐ Featured

A few commands worth knowing well before the rest.

### `/plan [description]`

Enter **plan mode** — read-only investigation that produces a step-by-step plan before any edit happens. Best for any non-trivial change.

```
/plan migrate the legacy auth middleware to the new session API
/plan refactor the billing module to support per-seat pricing
```

**Why:** prevents wasted edits. Claude reads first, proposes a plan, you approve or redirect, *then* it changes code. Toggle is also Shift+Tab.

---

### `/deep-research <question>`

Fan out parallel web searches, cross-check sources, return a cited report. For factual questions where one search isn't enough.

```
/deep-research what are the current state-of-the-art techniques for LLM agent memory in 2026?
/deep-research compare CRDT vs OT for collaborative editors — adoption, tradeoffs, recent advances
```

**Why:** one Claude turn would do one search. `/deep-research` runs many in parallel and synthesizes with citations.

---

### `/batch <instruction>`

Decompose a large change into 5–30 parallel subagents in git worktrees, each opening its own PR. For sweeping refactors and migrations.

```
/batch add error handling to every API endpoint in src/routes/
/batch convert all class components in src/ui/ to function components
/batch update every README in packages/* to follow the new template
```

**Why:** scales work past what one conversation context can hold. Each subagent works in isolation.

---

### `/code-review [ultra] [--fix] [--comment] [target]`

Multi-agent review of the current diff across correctness, security, performance, and style.

```
/code-review                       # local review of current branch
/code-review ultra                 # cloud review (deeper, billed)
/code-review --fix                 # apply findings
/code-review --comment 1234        # post inline comments on PR #1234
```

**Why:** independent reviewers catch things you and the author missed. Pair with `/simplify` for cleanup-only.

---

### `/goal <condition>`

Set a session goal Claude keeps working toward until met. Best paired with `/loop` for polling or `/background` to detach.

```
/goal all tests pass and lint is clean
/goal CI on this PR is green
/loop 5m              # then
/goal the deploy to staging finished
```

**Why:** turns a one-shot prompt into a "keep going until done" loop. Cleared on `/exit` or `/clear`.

---

### `/compact [hint]`

Summarize conversation history to free context when the bar in `/context` turns red.

```
/compact
/compact keep the auth refactor decisions; drop the unrelated debugging
```

**Why:** lets long sessions continue without losing the thread. Optional hint focuses what to preserve. Same-session-only — the compacted summary vanishes on `/exit`.

---

### `/handoff [path]`

Write a structured session summary to disk for the *next* session (you tomorrow, a teammate, or another agent) to pick up from.

```
/handoff                                 # writes to a default location
/handoff adr/handoff_$(date +%F).md      # explicit path
```

Output is forward-looking: current state, decisions made (with reasons), open questions, next-step suggestions, gotchas. Not a transcript — a *briefing*.

**Why:** compression saves a session about to overflow; `/handoff` saves whatever survives `/exit`. Use at end-of-day, before a context switch, when handing a project to another agent, or whenever you made a non-obvious decision you don't want future-you to re-derive.

---

### `/compact` vs `/handoff` — quick comparison

| | `/compact` | `/handoff` |
| --- | --- | --- |
| **Purpose** | Free context tokens in the *current* session | Capture state for the *next* session |
| **After it runs** | Conversation continues with compacted history | Conversation usually ends; artifact persists on disk |
| **Direction** | Backward — "what happened so far" | Forward — "where we are, what's next" |
| **Audience** | Same agent, same task | Future session (you / teammate / fresh agent) |
| **Persistence** | Lives in conversation buffer; gone on `/exit` | Markdown / memory file on disk |
| **Fidelity** | Lossy; system picks what to keep | Curated; you choose what survives verbatim |
| **Trigger** | Token budget pressure | Session boundary or decision capture |

Both summarize the conversation — `/compact` so you can keep talking, `/handoff` so you can stop. Healthy multi-day projects use both:

```
day 1 → work → /compact (mid-session) → more work → /handoff → /exit
day 2 → start session → read handoff → work → /compact → … → /handoff → /exit
```

`/compact` keeps each day's session alive past the token wall. `/handoff` bridges between days.

---

## Session control & context

| Command | What it does |
| --- | --- |
| `/clear [name]` | Start a fresh conversation. Previous stays in `/resume` picker. |
| `/resume [session]` | Resume a prior conversation by ID or via picker. Alias `/continue`. |
| `/context [all]` | Visualize token usage as a colored grid. |
| `/compact [hint]` | Summarize history to free context (this session). Optional focus hint. |
| `/handoff [path]` | Write a forward-looking summary to disk for the next session. |
| `/rewind` | Roll back to a checkpoint or selected message. Aliases `/checkpoint`, `/undo`. |
| `/recap` | One-line session summary. |
| `/exit` | Exit the CLI. Alias `/quit`. |

## Memory & project setup

| Command | What it does |
| --- | --- |
| `/init` | Initialize a project with `CLAUDE.md`. |
| `/memory` | Edit `CLAUDE.md`, toggle auto-memory, view auto-memory entries. |
| `/add-dir <path>` | Add another working directory to the session. |

## Model, reasoning & speed

| Command | What it does |
| --- | --- |
| `/model [name]` | Switch model. Press `s` to switch session only. |
| `/effort [level\|auto]` | Set reasoning effort: `low`, `medium`, `high`, `xhigh`, `max`, `ultracode`. |
| `/fast [on\|off]` | Toggle fast mode. |

Typing `ultrathink` in a prompt sets effort to high for that single turn.

## Permissions & config

| Command | What it does |
| --- | --- |
| `/permissions` | Add / remove allow-ask-deny rules. Alias `/allowed-tools`. |
| `/config` | Open settings UI (theme, model, output style, extended thinking, auto-compact). Alias `/settings`. |
| `/hooks` | View hook configurations. |
| `/statusline` | Configure the status line. |
| `/keybindings` | Open / create keybindings config. |

## Code review & quality

| Command | What it does |
| --- | --- |
| `/code-review [level] [--fix] [--comment] [target]` | Review diff for bugs and cleanups. `ultra` runs cloud review. |
| `/simplify [target]` | Cleanup-only review (reuse, simplification, efficiency, abstraction). |
| `/security-review` | Scan pending changes for injection, auth, data-exposure issues. |
| `/review [PR]` | Review a PR locally. |
| `/diff` | Open interactive diff viewer. |

## Testing & verification

| Command | What it does |
| --- | --- |
| `/run` | Launch the app and observe a change running. |
| `/verify` | Build, run, observe — confirm the change works. |
| `/run-skill-generator` | Teach `/run` and `/verify` how to build/launch this project. |

## Plugins & MCP

| Command | What it does |
| --- | --- |
| `/plugin` | Open plugin manager (Discover / Installed / Marketplaces / Errors tabs). |
| `/plugin install <name>@<marketplace>` | Install a plugin. |
| `/plugin enable / disable / uninstall <name>@<marketplace>` | Lifecycle ops. |
| `/plugin marketplace add <source>` | Add marketplace (GitHub `owner/repo`, git URL, local path, remote URL). |
| `/plugin marketplace list / update / remove` | Marketplace ops. |
| `/reload-plugins` | Reload all plugins without restarting. |
| `/mcp` | Manage MCP server connections and OAuth. |

## Skills & agents

| Command | What it does |
| --- | --- |
| `/skills` | List skills; toggle visibility, sort by token cost. |
| `/reload-skills` | Re-scan skill/command directories. |
| `/agents` | Manage subagent configurations. |
| `/background [prompt]` | Detach session as a background agent. Alias `/bg`. |
| `/tasks` | List & manage background tasks. Alias `/bashes`. |

## Planning & automation

| Command | Scope | What it does |
| --- | --- | --- |
| `/plan [description]` | Session | Enter plan mode. |
| `/goal [condition\|clear]` | Session | Set a goal Claude keeps working toward. Cleared on `/exit` or `/clear`. |
| `/loop [interval] [prompt]` | Session | Run a prompt on an interval (or self-paced). Alias `/proactive`. |
| `/schedule [description]` | Cloud | Create cloud-scheduled routines. Persists across sessions. Alias `/routines`. |
| `/batch <instruction>` | Session → repo | Decomposes into parallel subagents in worktrees; each opens a PR (artifacts persist). |
| `/deep-research <question>` | Session | Fan out web searches and synthesize a cited report. |
| `/ultraplan <prompt>` | Cloud | Draft a plan in an ultraplan session for browser review. |
| `/workflows` | Session | View workflow progress; pause / resume / save. |

## Utilities & info

| Command | What it does |
| --- | --- |
| `/help` | Show help and commands. |
| `/status` | Version, model, account, connectivity. |
| `/doctor` | Diagnose installation; press `f` to fix. |
| `/usage` | Cost & plan-usage stats. Aliases `/cost`, `/stats`. |
| `/copy [N]` | Copy last (or Nth) assistant response to clipboard. |
| `/export [filename]` | Export conversation as plain text. |
| `/rename [name]` | Rename the session. |
| `/release-notes` | View the changelog. |

## Account, IDE & integrations

| Command | What it does |
| --- | --- |
| `/login` / `/logout` | Auth. |
| `/ide` | Manage IDE integration. |
| `/install-github-app` | Set up the Claude GitHub Actions app. |
| `/install-slack-app` | Install the Claude Slack app. |
| `/remote-control` | Make this session available for remote control from claude.ai. Alias `/rc`. |
| `/teleport` | Pull a web session into the terminal. Alias `/tp`. |
| `/autofix-pr [prompt]` | Watch current branch's PR; push fixes on CI fail / review comments. |
| `/desktop` | Continue session in Claude Code Desktop. Alias `/app`. |

## Display & UI

| Command | What it does |
| --- | --- |
| `/theme` | Change color theme. |
| `/color [color\|default]` | Set prompt-bar color. |
| `/tui [default\|fullscreen]` | Set terminal UI renderer. |
| `/focus` | Toggle focus view (fullscreen only). |
| `/voice [hold\|tap\|off]` | Toggle voice dictation. |


## References

- Commands — https://code.claude.com/docs/en/commands
- Skills — https://code.claude.com/docs/en/skills
- Plugins — https://code.claude.com/docs/en/discover-plugins
- Plugin marketplaces — https://code.claude.com/docs/en/plugin-marketplaces
- Hooks — https://code.claude.com/docs/en/hooks
- CLI reference — https://code.claude.com/docs/en/cli-reference
- MCP prompts as commands — https://code.claude.com/docs/en/mcp#use-mcp-prompts-as-commands
