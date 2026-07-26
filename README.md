# awesome-agentic

Curated notes on agentic LLM tooling.

* [Agentic Coding](code-agent/README.md) — code agents, meta-frameworks, benchmarks, open standards

## Layers of Using LLM

```
PROVIDER API ─ Anthropic · OpenAI · Google GenAI
│              (+ gateways: OpenRouter, LiteLLM)
│
├─ FRAMEWORK ─ LangChain / LangGraph · Vercel AI SDK
│  └─→ your general LLM app (RAG, pipelines, non-code agents)
│
├─ YOUR OWN LOOP (+ provider tool runner)
│  └─→ your custom agent, full control
│
└─ AGENT HARNESS (library) ─ Claude Agent SDK · Codex SDK ·
   │                         pi SDK / RPC · tau AgentHarness
   │
   └─ CODE AGENT (app) ─ claude code · codex · opencode · pi · tau
      │
      └─ ORCHESTRATION ─ oh-my-claudecode · oh-my-codex

HOSTED AGENTS ─ provider runs the harness + deployment, no local
install: Claude Managed Agents · Codex cloud · Devin · Jules
```

| Layer | What it is | Examples |
|---|---|---|
| 1. Provider API / SDK | raw model access — native tool calling, streaming, structured outputs; you own the loop | Anthropic SDK, OpenAI SDK, Google GenAI SDK; gateways (OpenRouter, LiteLLM) aggregate many providers behind one layer-1 API |
| 2. Framework (branch, not foundation) | provider abstraction + graph/loop orchestration for your own general LLM apps (RAG, pipelines, non-code agents); an *alternative* to layer 3, not a substrate for it — no code agent is built on it | LangChain / LangGraph, Vercel AI SDK |
| 3. Agent harness (library) | prebuilt loop + tools, embeddable in your own app; you host and deploy | Claude Agent SDK (Claude Code as a library: file tools, bash, sessions, subagents), Codex SDK, OpenAI Agents SDK, pi SDK / RPC, tau `AgentHarness` |
| 4. Code agent (application) | finished tool: loop + tools + TUI + sessions | claude code, codex, opencode, pi, tau, Gemini CLI, Aider |
| 5. Orchestration | workflows/skills/multi-agent layered on top of an existing agent | oh-my-claudecode, oh-my-codex |
| 6. Hosted agents | provider supplies the harness **and** the deployment: managed loop, per-session sandbox, sessions/scheduling — no local app | Claude Managed Agents (CMA), OpenAI Codex cloud, Devin, Google Jules, GitHub Copilot coding agent |

Two independent questions separate the layers: **who supplies the harness** (the agent loop + tools + context management) and **who supplies the deployment** (the infra it runs on). Layers 1–2: you build both. Layers 3–5: they supply the harness, you still host. Layer 6: they supply both.

|  | you deploy | provider deploys |
|---|---|---|
| **you build the loop** | 1–2 (raw API, frameworks) | — |
| **they supply the loop** | 3 (harness libraries) · 4–5 (finished agents + add-ons) | 6 (hosted agents) |

### Agent harness vs code agent — interface, not internals

The two are easy to conflate because most vendors ship both from one engine. But the boundary is defined by the **consumption interface**, not by what's under the hood: **a harness you `import`; a code agent you type into.**

* an **agent harness** is consumed by *your code*. You call something like `query(prompt, options)`, get back a stream of events (text, tool calls, permission requests), and decide how to render them, when to invoke the agent, and what happens around it. You're building a product; the harness is a component inside it — your process, your deployment, your UI
* a **code agent** is consumed by *a human at a keyboard*. Every decision the harness leaves open has been made: TUI, keybindings, slash commands, interactive permission prompts, session picker, rendering

≈ **libcurl vs curl**: the same HTTP engine, once as a library your program links, once as a binary for your shell.


### Caveats

* the branches off layer 1 compete for the same slot in *your* stack — raw layer 1 (+ tool runner), a layer-2 framework, or a layer-3 harness; you pick one. pi/tau/opencode sit directly on provider APIs; tau even reimplements a thin provider abstraction internally (`tau_ai`) rather than importing one
* the boundary between 1 and 3 is blurring — provider SDKs now ship mini-loops themselves (e.g. Anthropic's Tool Runner, `client.beta.messages.tool_runner`, runs the request → execute → loop cycle over tools you define, but with no built-in tools or sandbox)
* the agent loop is ~20% of a code agent; the other 80% is harness engineering (edit reliability, bash sandboxing, sessions, compaction, prompt caching) — layer 2 frameworks don't help there
* generic wrappers fight provider-specific features (`cache_control` breakpoints, adaptive thinking) — code agents prefer layer 1 directly
* layer 6 ≈ "layers 1–4 as a service" is close but not exact: hosted agents have no TUI (layer 4's defining surface) — you talk to them over an API/event stream, and they add things no local layer has (managed sandbox, credential vaults, scheduling)

### Cross-cutting standards

These cut across layers rather than living in one:

* **MCP** (Model Context Protocol) — standard for exposing tools/resources to any agent; consumed at layers 1, 3, 4, and 6
* **Skills** — packaged task instructions loaded on demand; work in code agents (layer 4), harnesses (layer 3), and hosted agents (layer 6)
* **AGENTS.md / CLAUDE.md** — repo-level agent instructions; read by most layer-4 agents regardless of vendor