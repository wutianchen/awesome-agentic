# The Loop

> An agent is a model calling tools in a loop. Everything else is elaboration.

```python
while True:
    response = model(context)              # frozen function: text in, text out
    if response.stop_reason != "tool_use":
        break                              # model decided it's done
    result = execute(response.tool_calls)  # act on the world
    context += response + result           # observe the consequence
```

## Why a single call is not an agent

* one model call is a pure function — it cannot act, observe, or self-correct
* one call = one guess; the loop = iterative error-correction
* agency is not in the weights — it emerges from iterating a frozen function against an environment that pushes back

## Three properties the loop creates

| Property | Mechanism | Consequence |
|---|---|---|
| feedback | tool results re-enter context | model sees consequences (compiler error, test output) and adapts — why coding is the killer domain: cheap, honest feedback |
| working memory | each iteration appends to context | the transcript *is* the agent's state |
| inverted control flow | model decides next action + termination (`stop_reason`) | the line between agent and workflow — in a pipeline *you* hardcode the steps |

## The API contract

* the API is stateless → the loop must live client-side somewhere; someone resends full history every turn
* `stop_reason: "tool_use"` = the API's half of the loop contract ("your turn to execute")
* `end_turn` = model-decided termination; `pause_turn` = server-side tool loop hit its iteration cap, re-send to resume
* parallel tool calls: one iteration may carry multiple `tool_use` blocks — execute all, return all results in one message


## Harness = the loop + life support

The loop is ~20% of a coding agent; the other 80% keeps the loop productive over hours. Every harness feature maps to a loop problem:

| Loop problem | Harness feature |
|---|---|
| context overflows | compaction / context editing |
| resending history is expensive | prompt caching (prefix match) |
| actions are dangerous | permissions, sandboxing, confirmation gates |
| state must survive restarts | sessions (JSONL, branching), memory files |
| one context can't hold the task | subagents = loops spawning loops |
| loop runs away | max iterations, task budgets, effort |
| loop stalls (text-only intent, asks permission) | autonomy prompts, outer-loop retries |

## Extensions of the basic loop

* **thinking between iterations** — adaptive/interleaved thinking: reason about the last tool result before the next action
* **subagents** — an iteration whose "tool" is another whole loop; fan-out for parallel independent work
* **multi-agent** — loops messaging loops (coordinator/worker threads)
* **verifier loops** — separate fresh-context loop grades the worker loop's output (writer/reviewer, outcome rubrics)
* **outer loops** — harness-level: rerun the agent until tests pass / rubric satisfied (ralph, ultraqa, CMA outcomes)

## Key insight

The loop is trivial; the moat was never the loop. It's everything that keeps the loop running well for hours — which is why minimal agents (pi, tau) are possible, and why they're readable: the essential part is small.

## References

* [What I learned building an opinionated and minimal coding agent](https://mariozechner.at/posts/2025-11-30-pi-coding-agent/) — Zechner on pi
* [huggingface/tau](https://github.com/huggingface/tau) — the loop as a teaching codebase (`tau_agent`)
* [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) — Anthropic on workflows vs agents
