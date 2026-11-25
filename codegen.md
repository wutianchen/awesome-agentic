# Codegen

Notes as of 2026-11-16, from four pages of Codegen's docs: [overview](https://docs.codegen.com/introduction/overview), [about](https://docs.codegen.com/introduction/about), [sandboxes](https://docs.codegen.com/sandboxes/overview) and [Claude Code](https://docs.codegen.com/capabilities/claude-code). The pricing, PR review, API and integration pages were not read, and Codegen was not used.

[Codegen](https://codegen.com) ([docs](https://docs.codegen.com)) is a platform to "run frontier code agents at scale". It calls itself "an AI coworker that can understand and solve coding challenges", working from your team's tools. Its stated mission is "to build fully-autonomous software engineering".

* **Maker:** Codegen, Inc., San Francisco
* **Where it fits:** a hosted agent (layer 6, see the [layers](README.md#layers-of-using-llm)). Codegen runs the agent and its sandbox, and you hand it tasks from Slack, GitHub or a ticket tracker. Comparable to Devin, Codex cloud and Jules
* **Licensing:** a SaaS platform. The GitHub app is "free to install", and the core library [codegen-sdk](https://github.com/codegen-sh/codegen-sdk) is open source. Pricing not checked

## How it works

```text
 Slack · GitHub · Linear · Jira · ClickUp · Monday · web · API/SDK · CLI
                              │  tasks, comments, follow-ups
                              ▼
 ┌──────────────── Codegen platform (cloud) ─────────────────┐
 │ agent (Codegen's own, or Claude Code)                      │
 │   └─ isolated sandbox: shell, files, processes, network    │
 │ integrations exposed to the agent via MCP                  │
 │ analytics: tokens, completion rates, integration usage     │
 └────────────────────────────────────────────────────────────┘
                              │
                              ▼
                 PRs, reviews, Slack/Linear replies
```

### Interfaces

* **Slack** — agents "report progress, and interact directly with your team in Slack channels", and can ask for clarification
* **GitHub** — a GitHub App: open PRs, "Leave PR Reviews", "Auto-Fix Broken Tests"
* **Ticket trackers** — Linear, Jira, ClickUp and Monday.com
* **Web** — sign in with GitHub at codegen.com. Sessions are listed at codegen.com/agents
* **Python SDK / API** — "advanced automation"
* **CLI** — `uv tool install codegen`, `codegen login`, `codegen claude "..."`. It can also "Run Background Agents from your Terminal"
* Other integrations: Figma, databases and custom MCP servers

### Sandboxes

Agents "operate within secure, isolated sandbox environments" and don't touch your machine or production.

* Shell and terminal access, a file system, running processes, and networking that "can be restricted"
* Files stay in place across follow-ups in the same context, such as later Slack messages or Linear comments on the same task
* Configurable: setup commands, repository secrets, image snapshots, a base Docker image, web preview, and a remote VS Code editor

### Claude Code inside Codegen

* Running `codegen claude "..."` starts Claude Code locally and logs the session to the cloud
* At codegen.com/settings/model you can "Configure Claude Code as your default agent to run in Codegen's secure cloud sandboxes"
* Connected integrations reach Claude through MCP
* Support for OpenAI Codex and Gemini CLI is planned
* The docs don't say which model or agent loop Codegen's own default agent uses

### Other

* SOC 2 Type I and II certified, with regular penetration tests
* An analytics dashboard shows token usage, task completion rates and integration usage
* Parallel runs aren't described explicitly. Background agents are

## Codegen vs oh-my-pi (omp)

They sit in different layers and aren't direct rivals. [oh-my-pi](https://github.com/can1357/oh-my-pi) ([omp.sh](https://omp.sh)) is a local code agent (layer 4), described as a "Coding agent with the IDE wired in". It is a "Fork of Pi by @mariozechner" (see [tau.md](tau.md) for pi).

| | Codegen | omp |
|---|---|---|
| Kind | hosted agent platform (layer 6) | local code agent (layer 4) |
| Maker | Codegen, Inc. (company) | Stencil Labs (open source) |
| Runs | cloud sandboxes, plus a CLI | your terminal on macOS, Linux or Windows; needs Bun |
| Interface | Slack, GitHub, ticket trackers, web, SDK, CLI | TUI; ACP so editors such as Zed can drive it. Print mode, RPC and SDK reported but not confirmed |
| Agent | own agent (unspecified) or Claude Code; Codex and Gemini CLI planned | own engine, TypeScript on an ~80k-line Rust core, 60+ providers |
| Strengths | team workflow: tickets to PRs, PR review, test autofix, background agents, analytics, SOC 2 | IDE depth: LSP on every write (14 ops), debugger via DAP (28 ops), subagents in isolated worktrees, AST edits, persistent Python |
| License and cost | SaaS; free GitHub app; open-source SDK; prices not checked | MIT, free; you pay model providers directly |
| Popularity | not checked | about 39k stars |

**When to use which:**

* **Codegen** — hand off tickets and Slack requests to agents in the cloud that return PRs, with nothing running on your laptop
* **omp** — a stronger hands-on local agent than pi or tau, with language-server checks and a real debugger built in. It fills the same slot as Claude Code with [oh-my-claudecode](code-agent/omc/oh-my-claude.md)

They can be combined: omp locally for interactive work, and a hosted service like Codegen for background tickets. For a laptop control panel, omp sits next to [OpenClaw](openclaw.md), while Codegen is a cloud service outside it.

## Related

* [openclaw.md](openclaw.md) — local control plane that can also drive Claude Code
* [tau.md](tau.md) — pi and its Python port
* [README Landscape](README.md#landscape)
