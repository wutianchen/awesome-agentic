# OpenClaw

Notes as of 2026-11-15, from OpenClaw's README and docs in its repo. OpenClaw was not installed or run for these notes.

[OpenClaw](https://github.com/openclaw/openclaw) ([openclaw.ai](https://openclaw.ai), [docs](https://docs.openclaw.ai)) — "Your assistant, on your devices, in your chats". An open-source AI assistant that runs on your own computer and meets you in the chat apps you already use.

* **Facts:** TypeScript, MIT, about 394k stars, started 2025-11
* **Governance:** developed by the OpenClaw Foundation, an independent 501(c)(3) that employs the core team and signs releases. Donors include Amazon, OpenAI, Red Hat, and the University of Michigan; "OpenAI is a donor, not an owner"
* **Origin:** built by Peter Steinberger and the community for "Molty", a space lobster assistant; it credits Mario Zechner, author of [pi](code-agent/pi/README.md), for support
* **Where it fits:** a personal agent — it runs a finished assistant (layer 4) on top of swappable models and agent harnesses (layers 1–3), see the [layers](README.md#layers-of-using-llm)

## Architecture

```text
 chat apps (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, …)
                     │
                     ▼
 ┌──────────────── Gateway (one daemon per host) ────────────────┐
 │ sessions · tools · events · channel connections · plugins     │
 │ typed WebSocket API, default 127.0.0.1:18789                   │
 └───────────────────────────────────────────────────────────────┘
     ▲                    ▲                       ▲
 control-plane clients  nodes                   external apps
 (Control UI, CLI, TUI, (macOS/iOS/Android/     (scripts, dashboards,
  macOS app)             headless: voice,        CI, IDE extensions)
                         camera, screen)
```

* **Gateway** — the local control plane. A single long-lived daemon that owns every messaging channel, keeps provider connections, and exposes a typed WebSocket API (requests, responses, server-push events)
* **Control-plane clients** — the web [Control UI](#control-ui), CLI, TUI, and macOS app all connect to the Gateway
* **Nodes** — companion apps on macOS, iOS, Android, or headless machines add voice, canvas, camera, screen, and device-local actions
* **Channels** — 20+ messaging services; DM-capable channels pair unknown senders by default
* **Models and harnesses are plugins** — Claude, Codex, or local models, swappable without changing anything else
* **Extensions** — tools, skills, and plugins built on the plugin SDK and shared through [ClawHub](https://clawhub.ai); [OpenJarvis](jarvis.md) reports about 13,700 community skills
* **Two shapes** — a *personal claw* (one person, own machine) or a *multiplayer claw* (one Gateway a team shares as a bot in its workspace chat); configuration is the only difference

**Privacy:** state, memory, and credentials stay on your hardware. Prompts go to the model providers and chat platforms you configure. By default OpenClaw itself sends only a daily version check; anonymous feature statistics are opt-in.

## Getting started

```bash
curl -fsSL https://openclaw.ai/install.sh | bash   # macOS / Linux / WSL2
openclaw onboard --install-daemon                   # only if installed via npm/pnpm/bun
openclaw gateway status
openclaw dashboard                                  # opens the Control UI
```

Onboarding verifies model access, creates the workspace, and configures the Gateway.

## Control UI

The web dashboard served by the Gateway. Customization goes from no-code to replacing the UI:

| Level | What you can change |
|---|---|
| Appearance | 11 built-in themes, fonts, accent color, background image, language, tab icon; ask the agent to design and apply a new theme, or import one from tweakcn |
| Layout | Chat, Details, and Discussion in resizable columns or merged into tabs; session dashboards with pinned widgets, including interactive MCP Apps |
| Plugins | add themes, a sidebar tab, or a widget via `api.session.controls.registerControlUiDescriptor(...)` |
| Fork the UI | MIT, source in `ui/`, rebuild with `pnpm ui:build` — you then maintain a fork of a fast-moving project |
| Your own frontend | keep OpenClaw as the backend and talk to the Gateway protocol from your own app |

**Limit to test early:** bundled plugins can ship a first-class view for their tab, while other plugins' tabs appear to open in a generic plugin-tab page. How much a third-party tab can render wasn't confirmed, and plugin APIs are marked experimental.

## Security

* **Tools run on the host** for the main session unless you configure sandboxing
* **Sandboxing is off by default.** Turn it on with `agents.defaults.sandbox` (global) or per agent. Backends include Docker (default), Podman, SSH, and OpenShell. The docs say it is "not a perfect security boundary" but materially limits filesystem and process access
* **Treat inbound messages as untrusted input.** Approve unknown senders with `openclaw pairing approve <channel> <code>`
* Keep the Gateway on localhost. Read the security guide and exposure runbook before exposing it remotely or connecting other users

## As a laptop AI control plane

OpenClaw is the top recommendation in these notes for an always-on control plane on a laptop, because the Gateway already provides a background daemon, web UI, CLI, sessions, and scheduling, and it doesn't lock you into one model or harness.

Plan for:

* **Security** — turn on sandboxing and keep the Gateway local-only before connecting chat channels
* **Sleep** — scheduled and long-running jobs stop when the lid closes; move the same Gateway to a small always-on machine if they must run reliably

When to pick something else:

* **writing your own control plane** — Claude Agent SDK + MCP + a small local service with SQLite
* **nothing may leave the laptop** — [OpenJarvis](jarvis.md) with Ollama
* **mostly orchestrating coding work** — Claude Code + [oh-my-claudecode](code-agent/omc/oh-my-claude.md)

## OpenClaw vs OpenJarvis

OpenClaw is a finished assistant and control plane; [OpenJarvis](jarvis.md) is a Stanford research framework for running agents efficiently on your own device.

| | OpenClaw | OpenJarvis |
|---|---|---|
| Goal | keep state and credentials local; any model, hosted or local | keep inference local; call the cloud only "when truly necessary" |
| Models | swappable plugins (Claude, Codex, local) | installer sets up Ollama and a starter model |
| How you use it | Gateway daemon; chat apps, Control UI, CLI, TUI, mobile apps | `jarvis` CLI, browser GUI, desktop app |
| Out of the box | one assistant extended with tools, skills, plugins | eight built-in agents (morning digest, deep research, monitor, …) |
| Unique strength | reach and plugin ecosystem | measures energy, latency, and cost; tunes skills from local traces |
| Backing | OpenClaw Foundation, about 394k stars | Stanford Hazy Research / Scaling Intelligence Lab, about 14k stars |
| Stack | TypeScript, MIT | Python, Apache-2.0 |

Pick OpenClaw for an assistant in your chat apps today; pick OpenJarvis if all inference must stay on the device or you want to measure and tune local agents. OpenJarvis can import OpenClaw's community skills.

## Related

* [jarvis.md](jarvis.md) — OpenJarvis and other personal agents (Hermes Agent, Khoj, …)
* [README Landscape](README.md#landscape) — where OpenClaw sits among other tools
