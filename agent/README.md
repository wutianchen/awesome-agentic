# Agent


## Agent Capabilities
An **agent capability** is something an agent can actually do. It isn't just the model's intelligence: it's the model's skill combined with the tools, knowledge and permissions the agent has been given.

```text
        capability  =  model skill  ×  tools  ×  knowledge  ×  permission
                         (can it       (does it    (does it     (is it
                          reason/       have a      know the     allowed
                          plan?)        hand?)      context?)    to act?)

 ┌──────────────── Agent ────────────────┐
 │  Model ── reasons, plans, writes code │
 │    │                                  │
 │    ├─ Tools      read/edit files, bash, web, MCP servers
 │    ├─ Knowledge  CLAUDE.md, skills, memory, LSP/code index
 │    └─ Policy     sandbox, approvals, allowed commands
 └───────────────────────────────────────┘
```

If any factor is zero, the capability is zero. A brilliant model with no shell tool can't run tests.

### Example: "fix the failing test"

| Needed capability | Comes from | Missing it means |
|---|---|---|
| read code | file tools or [Serena](../code-agent/mcp-and-lsp/serena.md)/LSP | it guesses |
| run tests | bash tool | it can't verify the fix |
| know the project rules | CLAUDE.md or a skill | wrong style or test command |
| change files | write permission | it can only suggest a patch |
| check CI | GitHub MCP server | it stops at local tests |

The same model in Claude Code with these pieces can fix the bug. In a plain chat window it can only describe a fix.

### How agents gain capabilities

* **Tools and [MCP](../techniques/protocol-mcp/README.md) servers** let it act, for example Grafana, AWS, or code bundles
* **Skills** give it know-how, for example a skill that teaches it to delegate work to tau
* **Subagents** give it parallel, specialised work
* **Permissions and sandboxing** decide which of those capabilities it may actually use

In one line: **capability = what the agent can do and is allowed to do, given its model, tools, context and permissions.**

### Two ways to build the capabilities

1. LLM traning (inside the model)
2. Harness engineering (outside the model)