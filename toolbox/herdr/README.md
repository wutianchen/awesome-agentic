# herdr — architecture at a high level

[herdr](https://herdr.dev/) is a mouse-first terminal multiplexer (tmux/zellij alternative, `ctrl+b` prefix) with **native AI-agent awareness**: it detects code agents running in its panes, tracks their state (idle / working / blocked / done), and surfaces which project needs attention. In the layers model it sits *beside* code agents (layer 4) — it doesn't drive the model, it multiplexes and monitors the agents that do.

## Client/server model

```
┌ client (TUI) ──┐  attach / detach (ctrl+b q)
│  mouse-native  ├────────────┐
└────────────────┘            ▼
                   ┌ server (daemon) ─────────────────────┐
                   │  owns panes + process state          │
                   │  survives client disconnect          │
                   │                                      │
                   │  WORKSPACE (per project, sidebar)    │
                   │  └─ TAB (layout container)           │
                   │     └─ PANE (real terminal process)  │
                   │        └─ AGENT (detected + tracked) │
                   └──────────┬───────────────────────────┘
        NDJSON over unix socket / named pipe
                   ┌──────────┴───────────┐
              CLI wrappers          plugins / scripts
              (herdr pane …,        (raw socket API,
               herdr agent …)        event subscriptions)
```

* **Server**: background daemon that owns panes and process state; keeps running when the client detaches — the panes' processes (and agents) never stop.
* **Client**: the terminal UI attached to the server; purely a view. Detach with `ctrl+b q`, reattach with `herdr`.
* **Sessions**: named sessions are separate runtime namespaces (own server socket), isolating unrelated workflows.

## Entity hierarchy

| Entity | What it is |
|---|---|
| Workspace | top-level container per project; sidebar rolls up agent states across everything inside |
| Tab | layout container within a workspace (e.g. "agents", "logs", "server"); addressable from CLI/API |
| Pane | a real terminal process; split right/down, multiple per tab |
| Agent | a process herdr recognizes inside a pane and tracks through states: idle · working · blocked · done · unknown |

## Agent layer

The differentiating subsystem. Two detection paths:

* **Lifecycle-hook integrations** — agents with official integrations (pi, Kimi Code CLI, and others) report state authoritatively via hooks
* **Screen-manifest detection** — for everything else (20+ agents incl. Claude Code, Copilot CLI, Devin CLI), herdr reads the pane's live bottom-buffer snapshot and classifies it against **TOML detection manifests** (remote-updated, cached in server memory; locally overridable at `~/.config/herdr/agent-detection/<agent>.toml`)

Status **rolls up hierarchically**: pane → tab → workspace, so one blocked agent flags its whole workspace. "Blocked" classification is deliberately strict (only explicit approval/permission UI counts; unknown prompts default to idle). Debug detection decisions with `herdr agent explain`; connect straight to one agent's terminal with `herdr agent attach` (bypasses the main UI).

## Persistence model

Layered, strongest first:

1. **Detach/reattach** — server keeps running; processes never stop (the normal path)
2. **Snapshot restore** — after a full server stop, herdr restores the session *shape* (workspaces, tabs, panes, cwds, layout, focus) but panes reopen as fresh shells
3. **Native agent session restore** — integrated agents resume their own conversation via integration-reported session references after a restart
4. **Pane history replay** — replays saved output into restored panes; **off by default** (pane output can contain secrets), opt-in via `[experimental] pane_history = true`
5. **Live handoff** — `herdr update --handoff` migrates running panes to a new server binary without stopping them (experimental)

## Control plane — CLI + socket API

One protocol underneath: **newline-delimited JSON** over a unix domain socket (named pipe on Windows), default `~/.config/herdr/herdr.sock`. Request/response (`{id, method, params}` → `{id, result|error}`) plus long-lived **event subscriptions** (`events.subscribe`) pushing events for workspace/tab/pane/agent/worktree changes.

Methods are namespaced by domain: `server.*`, `workspace.*`, `tab.*`, `pane.*` (split, read, send_text, send_keys), `agent.*` (list, wait, prompt, start, explain), `layout.*`, `plugin.*`, `events.*`. The **CLI is a thin wrapper** over these calls (`herdr pane split`, `herdr agent wait --until done`) — the docs recommend the CLI for most automation and raw sockets only for direct control or event streams. Schema via `herdr api schema --json`.

This makes herdr scriptable as an orchestration surface: split a pane, start an agent, `agent.wait` until done, read the output — from any language.

## Plugin system

A plugin is a directory with a `herdr-plugin.toml` manifest plus commands herdr can launch — **language-agnostic** (bash, JS, Lua, Rust binary, anything argv-runnable). The manifest declares:

* **Actions** — invocable workflows (globally qualified as `plugin.id.action`, bindable to keys)
* **Event hooks** — commands run on events (payload arrives in `HERDR_PLUGIN_EVENT_JSON`)
* **Startup hooks** — one-shot init after session restore, once the API socket is ready

Plugins talk back to herdr through the injected environment (`HERDR_SOCKET_PATH`, `HERDR_BIN_PATH`, plus plugin/context IDs) — the whole CLI *is* the plugin API. No managed storage in v1 (plugins own their state under `HERDR_PLUGIN_STATE_DIR`). Distribution is GitHub-native: `herdr plugin install owner/repo`, with a marketplace auto-indexing repos tagged `herdr-plugin`; local dev via `herdr plugin link`.

## References — official documentation

* [Docs index](https://herdr.dev/docs/)
* [Quick start](https://herdr.dev/docs/quick-start/) — first session, workspaces/tabs/panes, running agents
* [Concepts](https://herdr.dev/docs/concepts/) — entity hierarchy, client/server, sessions
* [Agents](https://herdr.dev/docs/agents/) — detection (hooks vs manifests), status rollup, attach/wait/explain
* [Session state](https://herdr.dev/docs/session-state/) — persistence paths, restore, pane history, handoff
* [Socket API](https://herdr.dev/docs/socket-api/) — NDJSON protocol, method domains, events
* [Plugins](https://herdr.dev/docs/plugins/) — manifest, actions, hooks, runtime env
* [Marketplace](https://herdr.dev/docs/marketplace/) — GitHub-topic indexing, install/link
* [Configuration](https://herdr.dev/docs/configuration/) — settings, keybindings
