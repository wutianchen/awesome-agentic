# Agentic Coding

* [Claude Code Plugins: Orchestration and Automation](https://github.com/wshobson/agents)
* [Understand Anything](https://github.com/Egonex-AI/Understand-Anything)


## Layers of Using LLM

```
┌─────────────────────────────────────────────────┐
│ 5. meta-orchestration    oh-my-claudecode       │  ≈ oh-my-zsh on zsh
├─────────────────────────────────────────────────┤
│ 4. coding agent    claude code · pi · tau       │  ≈ the finished app (IDE, browser)
├─────────────────────────────────────────────────┤
│ 3. agent harness   Claude Agent SDK             │  ≈ web framework you embed (Flask)
├─────────────────────────────────────────────────┤
│ 2. orchestration fw   LangChain   (optional)    │  ≈ ORM
├─────────────────────────────────────────────────┤
│ 1. provider API    Anthropic / OpenAI SDK       │  ≈ DB driver / raw SQL
└─────────────────────────────────────────────────┘
  6. hosted agents (CMA) = layers 1–4 run by the provider   ≈ SaaS vs self-host
```

| Layer | What it is | Examples | Traditional-app analogy |
|---|---|---|---|
| 1. Provider API / SDK | raw model access — native tool calling, streaming, structured outputs; you own the loop | Anthropic SDK (tool runner), OpenAI SDK | DB driver / raw SQL |
| 2. Orchestration framework (optional) | provider abstraction + graph/loop orchestration; general LLM apps (RAG, pipelines), not coding-agent specific | LangChain / LangGraph | ORM — convenient generic layer, leaky for vendor-specific features |
| 3. Agent harness (library) | prebuilt loop + tools, embeddable in your own app; you host | Claude Agent SDK, pi SDK / RPC, tau `AgentHarness` | web framework you embed (Flask / Express) |
| 4. Coding agent (application) | finished tool: loop + tools + TUI + sessions | claude code, codex, opencode, pi, tau | finished app (IDE, browser) |
| 5. Meta-orchestration | workflows/skills layered on top of an existing agent | oh-my-claudecode, oh-my-codex | oh-my-zsh on zsh; browser extensions |
| 6. Hosted agents | provider runs the loop + sandbox + deployment | Claude Managed Agents (CMA) | managed SaaS (Heroku / Vercel) vs self-hosting |

* higher layer ≠ built on the layer below — pi/tau/opencode skip layer 2 and sit directly on provider APIs
* the agent loop is ~20% of a coding agent; the other 80% is harness engineering (edit reliability, bash sandboxing, sessions, compaction, prompt caching) — layer 2 frameworks don't help there
* generic wrappers fight provider-specific features (`cache_control` breakpoints, adaptive thinking) — coding agents prefer layer 1 directly

## Frameworks

* claude code
* codex
* [opencode](https://opencode.ai/)
* [openrouter](https://openrouter.ai/)
* [Claw Code](https://github.com/ultraworkers/claw-code)
* [pi coding agent](https://pi.dev/)
* [tau](https://github.com/huggingface/tau)


Framework upon Framework

#### [oh-my-claude](https://github.com/Yeachan-Heo/oh-my-claudecode)

https://github.com/Yeachan-Heo/oh-my-claudecode


#### [oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex)

https://yeachan-heo.github.io/oh-my-codex-website/


#### [ph-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)

tbd

## LLM

#### Benchmark

* [SWE-bench](https://www.swebench.com/)



## Open Standards

* skills: /skills