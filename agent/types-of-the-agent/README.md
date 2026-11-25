# Types of Agents

Notes as of 2026-11-25. The examples are well-known products from general knowledge, not freshly checked.

All agents share the same core loop (model → tools → result → repeat, see [agent-loop.md](../../agent-loop.md)). What separates them is the tools and environment each is given, which is the capability formula in [Agent Capabilities](../README.md#agent-capabilities).

```text
                      same core loop (model → tools → result → repeat)
                                          │
  ┌────────┬──────────┬──────────┬────────┼────────┬──────────┬─────────┬────────┬──────────┐
  code     personal   research   computer data     ops/SRE    support   workflow voice      embodied
  (files,  (chat,     (search,   -use     (SQL,    (alerts,   (CRM,     (SaaS    (audio     (robot,
   shell)   calendar)  documents) (screen) notebooks) logs)   tickets)  triggers) stream)   sensors)
```

## By job

| Type | What it does | Main tools / environment | Examples |
|---|---|---|---|
| **Code agent** | reads, edits, runs and tests code in a repo | files, shell, git, LSP | Claude Code, Codex, opencode, pi, tau, OpenHands — see [code-agent](../../code-agent/README.md) |
| **Personal assistant** | lives on your devices or in chat apps; handles email, calendar, reminders and errands | chat channels, calendar, email, local files | OpenClaw, Hermes Agent, Khoj, [OpenJarvis](../../jarvis.md) |
| **Research agent** | searches the web or documents over many steps and writes a cited report | web search, page fetch, document stores | ChatGPT and Gemini Deep Research, Perplexity |
| **Computer-use / browser agent** | operates a GUI by seeing the screen, clicking and typing | screenshots, mouse, keyboard, browser | Claude computer use, OpenAI Operator, Claude in Chrome, browser-use |
| **Data / analytics agent** | queries databases, runs notebooks, builds charts | SQL, notebooks, BI tools | Hex, Julius |
| **Ops / SRE agent** | watches alerts, investigates incidents, runs runbooks | metrics, logs, traces, cloud APIs | Grafana Sift, incident.io AI, PagerDuty agents |
| **Customer-facing agent** | support, sales or booking conversations backed by business systems | CRM, ticketing, knowledge base | Intercom Fin, Sierra, Decagon |
| **Workflow / automation agent** | runs multi-step business processes across SaaS apps on triggers or schedules | app connectors, triggers, schedules | Zapier and n8n agents, Salesforce Agentforce |
| **Voice agent** | real-time spoken conversation, such as phone calls | streaming audio, telephony | OpenAI Realtime, Vapi, ElevenLabs agents |
| **Embodied agent** | controls a robot or acts in a simulation | sensors, actuators, simulators | Gemini Robotics, NVIDIA GR00T |

## By structure

| Type | How it works | Examples |
|---|---|---|
| **Single agent** | one loop, one set of tools | a plain Claude Code session |
| **Multi-agent / orchestrator** | a lead agent assigns work to specialist agents and merges results | [oh-my-claudecode](../../code-agent/omc/oh-my-claude.md), Claude Code subagents |
| **Background / ambient agent** | runs without a person present, triggered by events or a schedule | CI fixers, scheduled digests, Codegen background agents — see [codegen.md](../../codegen.md) |
| **Interactive / human-in-the-loop agent** | a person approves or steers each step | Claude Code with permission prompts |

## By where it runs

| Type | Where | Examples |
|---|---|---|
| **Local** | your laptop | Claude Code, omp, OpenClaw |
| **Hosted** | the provider's cloud sandbox | Claude Managed Agents, Codex cloud, Devin, [Codegen](../../codegen.md), OpenHands Cloud |
| **Embedded** | inside your own app, via an agent SDK | Claude Agent SDK, OpenAI Agents SDK |

See the [layers](../../README.md#layers-of-using-llm) for how local, hosted and embedded agents relate.
