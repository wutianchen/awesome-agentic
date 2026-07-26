# Adversarial Review

A multi-agent pattern where **multiple independent agents try to refute a finding** rather than confirm it. Only findings that survive refutation count.

## Why

One agent finding a bug = "I think this is a bug." Adversarial review turns it into "three skeptics couldn't disprove it." Filters out plausible-but-wrong findings — the kind LLMs are most likely to produce.

## Shape

```
Finder agent → "SQL injection at line 42"
        │
   ┌────┼────┬─────────┐
   ▼    ▼    ▼         ▼
 Refuter1  Refuter2  Refuter3    ← each independent; sees only the claim
   │         │         │             prompted to REFUTE
{real?}   {real?}   {real?}
   └────┬────┴─────────┘
        ▼
  ≥2 say "still stands" → keep; else drop
```

## Design rules

| Rule | Why |
| --- | --- |
| **Default to "refuted"** | Bias the refuter to skepticism — "unless you can prove this is real, mark refuted" |
| **Independent contexts** | Each refuter sees only the claim + code, not other refuters' opinions — no echo chamber |
| **Quorum / majority** | Single dissent isn't enough; need 2 of 3 (or 3 of 5) to keep |
| **Structured verdicts** | Force JSON output `{refuted: bool, reason: string}` so counting is mechanical |

## Two flavors

| Flavor | Setup | When to use |
| --- | --- | --- |
| **Redundant refute** | N identical "try to refute this" agents | Confidence score on a single hypothesis |
| **Perspective-diverse refute** | Each refuter gets a different lens (correctness, security, perf, does-it-reproduce) | Findings can fail in more than one way; redundancy alone misses some failure modes |

## Where it shows up

- **Code review pipelines** — finder pass produces candidates; adversarial verify cuts the false-positive rate before showing the user.
- **Bug-hunting fleets** — loop-until-dry finders + adversarial verify per finding.
- **Migration safety checks** — before applying a transform, N agents try to find a case where it breaks.
- **Citation verification** in research workflows — refuters re-fetch the source to confirm the quote actually says what was claimed.

## Workflow snippet

```js
const findings = await agent("review src/auth/ for bugs", {schema: BUGS})

const verified = await parallel(findings.bugs.map(b => () =>
  parallel(['correctness', 'security', 'repro'].map(lens => () =>
    agent(
      `Try to REFUTE this bug via the ${lens} lens. Default refuted=true if uncertain. Bug: ${b}`,
      {schema: VERDICT}
    )
  ))
  .then(votes => ({ bug: b, real: votes.filter(v => !v.refuted).length >= 2 }))
))

const real = verified.filter(v => v.real).map(v => v.bug)
```

## Tradeoffs

| Pro | Con |
| --- | --- |
| Dramatically lower false-positive rate | 3–5× the agent calls per finding |
| Catches plausible-sounding hallucinations | Slow if not parallelized |
| Independent perspectives surface failure modes redundancy misses | Equivalent-mutant problem — some refutations don't change the verdict |

## Related patterns

| Pattern | Direction | What it catches |
| --- | --- | --- |
| **Adversarial review** | One claim → many skeptics | False positives from a single proposer |
| **Judge panel** | Many proposals → many judges → synthesis | Picks the best among diverse approaches |
| **Completeness critic** | Final agent asks "what's missing?" | False negatives — uncovered modalities |
| **Mutation testing** | Break code, see if tests catch it | Weak test assertions |
