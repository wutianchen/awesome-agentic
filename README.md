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
| 2. Framework (branch, not foundation) | provider abstraction + graph/loop orchestration for your own general LLM apps (RAG, pipelines, non-code agents); an *alternative* to layer 3, not a substrate for it — no code agent is built on it | LangChain / LangGraph, Vercel AI SDK, [OpenJarvis](https://github.com/open-jarvis/OpenJarvis) (local-first personal AI agents, on-device models) |
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

## Landscape

Everything referenced in these notes, by kind. "Layer" refers to the table above.

| Kind | Name | Notes |
|---|---|---|
| Model provider | Anthropic · OpenAI · Google | first-party model APIs and SDKs (layer 1) |
| Model provider | [TypeSafe AI](https://typesafe.ai) | makes Jev, a decision model returning typed JSON; early access since 2026-09-15 — see [jev.md](jev.md) |
| Model gateway | [OpenRouter](https://openrouter.ai/) · LiteLLM | one API in front of many providers (layer 1) |
| Model gateway (third-party reseller) | [DefAPI](https://defapi.org/) | prepaid-credit gateway to OpenAI, Anthropic, Google, and TypeSafe models; run by KeyStorm LLC (Wyoming); its terms say provider brands "do not imply any endorsement", so it is not an official channel; few public reviews |
| Framework | LangChain / LangGraph · Vercel AI SDK | build your own general LLM apps (layer 2) |
| Framework | [OpenJarvis](https://github.com/open-jarvis/OpenJarvis) | local-first personal AI agents from Stanford — see [jarvis.md](jarvis.md) |
| Framework | [Letta](https://github.com/letta-ai/letta) | stateful agents with long-term memory |
| Agent harness | Claude Agent SDK · Codex SDK · OpenAI Agents SDK · pi SDK · tau `AgentHarness` | prebuilt loop + tools you embed (layer 3) |
| Code agent | claude code · codex · [opencode](https://opencode.ai/) · [pi](https://pi.dev/) · [tau](https://github.com/huggingface/tau) · Gemini CLI · Aider · [goose](https://github.com/aaif-goose/goose) · [Claw Code](https://github.com/ultraworkers/claw-code) | finished agents you type into (layer 4) |
| Orchestration | [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) · [oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex) · [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) · [wshobson/agents](https://github.com/wshobson/agents) | workflows and multi-agent add-ons on top of a code agent (layer 5) |
| Hosted agent | Claude Managed Agents · OpenAI Codex cloud · Devin · Google Jules · GitHub Copilot coding agent | provider runs harness and deployment (layer 6) |
| Personal agent | [OpenClaw](https://github.com/openclaw/openclaw) | assistant on your machine reached through chat apps; Gateway control plane with a themable Control UI |
| Personal agent | [Hermes Agent](https://github.com/NousResearch/hermes-agent) | self-improving agent that writes its own skills |
| Personal agent | [Khoj](https://github.com/khoj-ai/khoj) | self-hostable "second brain" with automations and research |
| Local AI stack | [LocalAI](https://github.com/mudler/LocalAI) · [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) · [Open WebUI](https://github.com/open-webui/open-webui) · Ollama | run or chat with models on your own hardware |
| Standard | MCP · Skills ([agentskills.io](https://agentskills.io/specification)) · AGENTS.md / CLAUDE.md | cross-layer conventions, see above |
| Technique | tool calling · ReAct · CodeAct · RAG | core agent patterns: call tools, reason-act loops, act by writing code, retrieve context |
| Technique | prompt caching · context compaction · subagents · sandboxing | harness engineering that makes long agent runs cheap, fit in context, and safe |
| Benchmark | [SWE-bench](https://www.swebench.com/) | code-agent evaluation |
| Analysis tool | [Understand Anything](https://github.com/Egonex-AI/Understand-Anything) | knowledge graphs of a codebase for agents and people |
| Organization | Nous Research · Hugging Face · OpenClaw Foundation · Stanford Hazy Research / Scaling Intelligence Lab | makers of Hermes Agent, tau, OpenClaw, and OpenJarvis |