# Context Management — Compaction and Handoff

Two distinct mechanisms keep long-running Claude Code work productive: **compaction** (within a session) and **handoff** (across sessions). They look like cousins because both produce a "summary," but they solve opposite problems.

| | Compaction | Handoff |
| --- | --- | --- |
| Scope | Within one session | Across sessions |
| Triggered by | Token threshold (or `/compact`) | User command (`/handoff`) |
| Output lives | In the running conversation buffer | On disk (markdown file) |
| After it runs | Conversation continues | Conversation usually ends |
| Direction | Backward — "what happened" | Forward — "what's next" |
| Persistence | Gone on `/exit` | Survives indefinitely |
| Goal | Don't run out of tokens | Don't lose context across the session boundary |

---

## Compaction (within a session)

Compaction automatically summarizes older conversation context when approaching the context window limit, replacing stale messages with concise summaries. This keeps the active context focused — long contexts degrade model performance even before hitting the token cap.

### How it triggers (in Claude Code CLI)

1. The system monitors input token count each turn
2. When tokens exceed a threshold (default: 150,000 tokens, minimum: 50,000), compaction kicks in
3. Claude generates a `<summary>` of the conversation so far
4. A compaction content block is inserted into the assistant's response
5. On the next turn, all messages before the compaction block are dropped — the summary replaces them

### How to use compaction

You don't need to do anything — Claude Code handles compaction automatically. But you can influence it:

1. `/compact` — manually trigger compaction at any time (useful to free up context before a complex task)
2. `/compact <instructions>` — compact with custom instructions, e.g. `/compact preserve all file paths and function signatures`
3. `CLAUDE.md` files — content in CLAUDE.md is treated as the system prompt and survives compaction (it's re-injected each time). Put critical context here.

### What compaction does NOT do

- **Does not survive `/exit`.** The compacted summary is part of the live conversation; once you quit, the buffer is gone. `claude --continue` reloads the JSONL transcript, which by then contains the compaction-block-replaced version of history — lossy, ordered chronologically, with all the noise of false starts.
- **Does not export anywhere.** No file is written, no memory entry is created. If you want the conversation's findings to outlive the session, compaction alone won't get them out.
- **Does not let you control fidelity.** The instruction hint is a soft preference, not a guarantee. Specific decisions ("we removed the kubectl fallback because of dict-vs-int shape mismatch") routinely get collapsed into vague phrases ("we simplified the probe logic").
- **Does not help across machines, agents, or people.** The JSONL transcript is local. A teammate, a different agent (Codex / Gemini), or even your own laptop after a re-image won't see it.

---

## Handoff (across sessions)

`/handoff` writes a structured session summary to disk for the **next session** to pick up from. The next reader might be you tomorrow, a teammate, a different agent (Claude → Codex / Gemini), or a fresh Claude session on a different machine.

### How it differs from compaction

Same primitive (summarize the conversation) used in opposite directions:

- Compaction operates **backward in time** — looks at what happened, drops what's least relevant, keeps the conversation alive for *this* session.
- Handoff operates **forward in time** — looks at what *matters going forward*, organizes by decisions / open questions / next steps, persists for *the next* session.

A useful mental model: compaction is the agent telling itself "I can keep going with less detail." Handoff is the agent telling future-readers "here's what you need to know to resume."

### What's in a good handoff document

Not a transcript. A briefing. Sections typically include:

```markdown
## State
- Branch, uncommitted changes, build status, test status
- Where things stand on the active task

## Decisions made (and why)
- The non-obvious choices that shaped the code
- Reasons in enough detail that future-you doesn't re-derive them

## Next session
- The exact next 2–5 actions to take
- Anything currently blocked, and what unblocks it

## Gotchas
- Hidden constraints, brittle behavior, things that look fine but aren't
- Tests that should run but currently don't, and why
```

The persistence story matters: a markdown file lives in git (or `~/.claude/projects/.../memory/`), survives reboots, transfers across machines, is grep-able years later, and works as onboarding documentation for humans too.

### When to use it

Reach for `/handoff` when:

- You're stopping for the day and want tomorrow's session to start productively
- You're switching laptops, environments, or operating systems
- A teammate is going to pick up the project
- You're switching agents (Claude → Codex / Gemini)
- A non-obvious decision was made that future-you should not have to re-derive
- The session is about to hit context exhaustion AND will need to continue across days (handoff is more durable than another `/compact`)

### When NOT to bother

If the task is going to be finished in the current session, or if `claude --continue` tomorrow on the same machine is genuinely enough — skip the handoff. The artifact has a small but real cost (a few minutes of attention to author it well).

---

## `/compact` vs `/handoff` vs `claude --continue` — choosing between them

These three are the main controls over long-running Claude Code work. Most projects need all three, used at different moments.

| Situation | Best tool |
| --- | --- |
| Token bar in `/context` is turning red, want to keep working | `/compact` |
| Made a non-obvious decision worth preserving past `/exit` | `/handoff` (+ optional memory entry) |
| Coming back to the same project tomorrow on the same machine, single-day task | `claude --continue` — handoff optional |
| Coming back after >1 week, or across machines | `/handoff` is the durable bridge; `--continue` may not be available or replays too much noise |
| Switching from Claude to Codex / Gemini | `/handoff` (markdown is portable; JSONL isn't) |
| Teammate is picking up the project | `/handoff` (committed to the repo, optionally) |
| Auditing "why did we decide X?" months later | `/handoff` is grep-able; `--continue` requires scrolling a long transcript |

### Why `claude --continue` is not a full substitute for handoff

`claude --continue` reloads the prior session's full JSONL transcript. It works well within the same machine + reasonably short timeframe. Its limits:

1. **Local-only.** The JSONL is at `~/.claude/projects/<hash>/<uuid>.jsonl`. Switch machines and it's gone.
2. **Replays everything, noise included.** A 4-hour day of debugging detours all comes back into context tomorrow — burning the day-2 budget on history that's no longer relevant.
3. **Chronological, not prioritized.** Decisions get scattered across the timeline; tomorrow-you has to reconstruct conclusions from the journey.
4. **Auto-compaction is lossy by then.** Long sessions get compacted internally; `--continue` replays the lossy summary, not the original detail.

A handoff document costs ~1k tokens to read and is curated by importance. A long replayed transcript costs ~50–100k tokens and is structured by time.

### The healthy multi-day workflow

```
day 1 → work → /compact (when token bar reds) → more work → /handoff → /exit
day 2 → claude --continue   (optional, free, gives full context)
          + read the handoff (forces re-orientation in 30 seconds)
        → work → /compact → … → /handoff → /exit
day 3 → …
```

`/compact` keeps each day's session alive past the token wall. `/handoff` bridges between days. `claude --continue` replays the previous session verbatim *when* the prior session is still on the same machine and small enough to be useful.

---

## Where handoff fits in the broader persistence toolkit

Compaction and handoff aren't the only context-engineering tools. The full set, ordered from most ephemeral to most durable:

| Mechanism | Lifetime | Best for |
| --- | --- | --- |
| Auto-compaction | Until `/exit` | Token budget pressure, automatic |
| `/compact <hint>` | Until `/exit` | Targeted in-session summary |
| `/context` (just inspecting) | Single point in time | Diagnosing where tokens are going |
| `/recap` | Single message | Quick "where are we" within a session |
| `claude --continue` / `/resume` | Until the JSONL is deleted | Same-machine resume |
| `/handoff` (markdown to disk) | Indefinite | Cross-session, cross-machine, cross-agent |
| Memory entries (`memory/*.md`) | Indefinite | Stable facts about user, preferences, project conventions |
| `CLAUDE.md` | Indefinite, re-injected every session | Critical project context that should never be lost |
| Wiki / ADR documents | Indefinite, shared | Long-form decisions, design rationale |
| Git commits | Indefinite, shared | The change itself plus the "why" in the message |

Compaction and `--continue` cover within-session and same-machine-next-day cases respectively. Handoff covers everything beyond that — and feeds the rest of the durable layer (memory, CLAUDE.md, wiki, commits) by surfacing what's worth promoting from one session into the long-term knowledge base.

---

## Source of Information

* https://platform.claude.com/docs/en/build-with-claude/compaction
* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
* See also: [[../claude cli/slash.md]] — `/compact`, `/handoff`, `/context`, `/recap`, `/resume` reference.