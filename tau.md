# tau and pi

Notes as of 2026-11-14, from each project's README, docs, and file layout. Neither tool was installed or run, and source code was only listed, not read.

* [tau](https://github.com/huggingface/tau) — Hugging Face, Python, MIT, about 3k stars, started 2026-06. Docs: [twotimespi.dev](https://twotimespi.dev/) (τ = 2π)
* [pi](https://github.com/earendil-works/pi) ([pi.dev](https://pi.dev/)) — Earendil Works, TypeScript, MIT, about 122k stars, started 2025-08

**Relationship:** tau is a Python port of pi's design, not built on pi. It describes itself as "a Python port of Pi's minimalist coding agent", and its dependencies don't include pi. It reimplements the same layers and copies several of pi's contracts by name. tau is a teaching project, meant to be read; pi is the production toolkit.

## tau's architecture

Three packages, each depending only on the one to its right:

```text
tau_coding  →  tau_agent  →  tau_ai
(the app)      (the brain)   (providers)
```

* **`tau_ai`** — turns each model provider into one provider-neutral stream. It has one module per provider (Anthropic, Google, Mistral, OpenAI-compatible, OpenAI Codex), plus retries, a model catalog, model limits, and a fake provider for tests
* **`tau_agent`** — the portable "brain": messages, tools, events, the agent loop, the harness (`AgentHarness`), and session primitives (JSONL storage, a session tree for branching, in-memory sessions). It must not import CLI, Rich, Textual, or config-loading code
* **`tau_coding`** — the coding app wrapped around the brain: CLI, Textual TUI, the `read` / `write` / `edit` / `bash` tools, provider config and OAuth logins, `AGENTS.md` and `.tau/` instructions, skills, prompt templates, themes, extensions, compaction, and on-disk sessions under `~/.tau/sessions/`. Local inference, such as the built-in llama.cpp extension, lives here, not in the core

The key boundary:

```text
AgentHarness  = reusable brain
CodingSession = coding-agent environment
TUI           = one possible frontend
```

Design rules from the README:

* **Events are the contract.** Providers, renderers, the TUI, and custom frontends meet at a typed event stream (`AgentStartEvent`, `MessageStart/Update/EndEvent`, `ToolExecutionStart/Update/EndEvent`, …). Frontends never see raw provider chunks. `tau_coding` adds app-level events on top, such as compaction, queue updates, and retries
* **Tools are ordinary typed functions:** a schema plus an async executor that returns a structured result
* **Sessions are append-only JSONL**, so context can be compacted without rewriting the record, and any point can be resumed or branched
* **Small layers:** each package has one job and can be read on its own

You can use tau as a library by building an `AgentHarness` with a provider, model, system prompt, and tools, which is why the [README](README.md) lists tau `AgentHarness` at layer 3 and tau at layer 4.

## How it matches pi

pi is a monorepo of packages. tau's three layers map onto pi's core three:

| Role | pi (TypeScript) | tau (Python) |
|---|---|---|
| Unified multi-provider LLM API | `pi-ai` | `tau_ai` |
| Agent runtime: loop, tool calling, state | `pi-agent-core` | `tau_agent` |
| Interactive coding agent CLI | `pi-coding-agent` | `tau_coding` |
| Terminal UI | `pi-tui`, pi's own library with differential rendering | Textual, a third-party library |
| Durable runtime, telemetry, MCP, evals, `chord` app runtime, and more | separate packages | no equivalent; sessions live in `tau_agent.session` |

Contracts tau copies from pi:

* **Message lifecycle events:** tau's docs call its message start/update/end events "Pi-compatible"
* **`finish_turn`:** a hook that runs after a turn's results are appended, "Pi's `finishTurn` scheduling contract with Python names and typed dataclasses"
* **Tool-batch termination:** a tool can end the run, but "as in Pi" a batch of tool calls stops only when every result opts in
* **The same ways to customize:** both are extended with skills, prompt templates, themes, and extensions, and both read project instructions from `AGENTS.md`

## Where they differ

* **Purpose:** tau is for learning how a coding agent is built and keeps a phase-by-phase build journal in `dev-notes/`. pi is a product you are meant to adapt to your workflow
* **Scope:** pi has 12 packages and several ways in (interactive, print/JSON mode, RPC, TypeScript SDK). tau has 3 packages, with a TUI, print mode, and a Python library
* **Ecosystem:** pi has far more users, and OpenClaw credits pi's author for support. tau is new and small
* **Philosophy:** pi deliberately skips sub-agents and plan mode, and has no built-in permission system; it says to containerize or sandbox it for stronger isolation. tau's matching choices weren't checked
