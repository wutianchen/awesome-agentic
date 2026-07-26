# Oh-My-Claude

Oh-My-Claude (OMC) is a plugin that layers multi-agent orchestration, specialized subagents, persistent state, and pre-built skills on top of Claude Code. The CLI itself is unchanged — OMC ships as commands/skills/hooks/MCP tools through the standard plugin system.

## Claude Code CLI vs Oh-My-Claude

### High-level

| | Claude Code CLI | Oh-My-Claude (OMC) |
| --- | --- | --- |
| **What it is** | The base agentic terminal app | A plugin layered on top of Claude Code |
| **Install** | `npm i -g @anthropic-ai/claude-code` | `/plugin install` from the OMC marketplace |
| **Setup** | None | Say "setup omc" or `/oh-my-claudecode:omc-setup` |
| **Configuration** | `~/.claude/settings.json`, `CLAUDE.md` | Same files + OMC's own `.omc/` state dir |
| **Removal** | Uninstall the CLI | Disable/uninstall the plugin; CLI keeps working |

### Agents & delegation

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Subagents available** | `claude`, `Explore`, `Plan`, `general-purpose`, `statusline-setup` | All of the above + 18 OMC specialists: `analyst`, `architect`, `code-reviewer`, `code-simplifier`, `critic`, `debugger`, `designer`, `document-specialist`, `executor`, `explore`, `git-master`, `planner`, `qa-tester`, `scientist`, `security-reviewer`, `test-engineer`, `tracer`, `verifier`, `writer` |
| **Routing** | You / the model pick an agent ad-hoc | OMC's CLAUDE.md tells the model to delegate by task type: code → `executor` (opus for complex), planning → `planner`, etc. |
| **Verification** | Manual — you ask for it | Built-in: a separate `verifier` pass before claiming completion; authoring and review are kept in separate lanes |

### Skills & workflows

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Bundled skills** | `/init`, `/review`, `/security-review`, `/code-review`, `/simplify`, `/run`, `/verify`, `/deep-research`, `/batch`, `/loop`, `/schedule`, `/claude-api`, etc. | All of those + OMC tier-0 workflows: `autopilot`, `ultrawork`, `ralph`, `team`, `ralplan`, `ccg`, `deep-interview`, `ai-slop-cleaner`, `deepsearch`, plus `omc-reference` |
| **Trigger style** | Slash command or description match | Same — invoke with `/oh-my-claudecode:<name>`. Plus keyword auto-detection: `"autopilot"` triggers autopilot, `"ulw"` triggers ultrawork, `"ralph"` triggers ralph, `"ultrathink"` deepens reasoning |
| **Magic keywords** | `ultrathink` (single-turn high effort) | Same + many more: `cancelomc`, `ralph`, `team`, `deslop`, `deep-analyze`, `tdd`, `deepsearch`, `ccg`, `ralplan` |

### State, memory & context

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Persistent memory** | `CLAUDE.md`, auto-memory (`~/.claude/projects/<proj>/memory/`) | Same + `.omc/notepad.md`, `.omc/project-memory.json`, `.omc/plans/`, `.omc/research/`, `.omc/logs/` |
| **Session state** | Resume by ID | Resume + `.omc/state/sessions/{sessionId}/` for cross-agent shared state |
| **Cross-session shared memory** | None built-in | `shared_memory_*` MCP tools for read/write/list/delete across runs |
| **Wiki / knowledge base** | None | `wiki_add/ingest/query/lint` MCP tools for in-repo searchable notes |
| **Notepad** | None | `notepad_*` tools for ephemeral working notes the agent can re-read |

### Code-navigation tools

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Symbol-aware search** | Grep + `Explore` agent | All of that + LSP-backed MCP tools: `lsp_goto_definition`, `lsp_find_references`, `lsp_hover`, `lsp_diagnostics`, `lsp_rename`, `lsp_workspace_symbols`, `lsp_code_actions` |
| **Structural code search/edit** | Grep / Edit | `ast_grep_search`, `ast_grep_replace` for AST-aware refactors |
| **Python REPL** | Bash with `python -c` | `python_repl` MCP tool with persistent kernel |

### Hooks & control flow

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Hooks** | User-configured in `settings.json` | Same + OMC ships default hooks injecting `<system-reminder>` markers (`hook success: Success`, `[MAGIC KEYWORD: ...]`, `The boulder never stops` for ralph/ultrawork) |
| **Persistence in prompt** | None — every prompt is fresh | `<remember>` (7 days) and `<remember priority>` (permanent) tags |
| **Kill switches** | None | Env vars `DISABLE_OMC`, `OMC_SKIP_HOOKS` (comma-separated) |
| **Cancellation** | Ctrl-C interrupts current turn | `/oh-my-claudecode:cancel` (or saying "cancelomc") ends an execution mode like ralph/ultrawork |

### Mental model

| | Claude Code CLI | Oh-My-Claude |
| --- | --- | --- |
| **Posture** | Generalist single-agent loop with optional subagent delegation | Orchestrator-and-specialists: main loop routes, subagents execute, verifier checks |
| **Model routing** | Manual via `/model` | Conventionalized: `haiku` for lookups, `sonnet` standard, `opus` for architecture / deep analysis |
| **Tier-0 workflows** | None as a named concept | `autopilot`, `ultrawork`, `ralph`, `team`, `ralplan` — multi-phase orchestrations with cancellation semantics |
| **Verification** | Optional | Required by convention before claiming completion |

### When each fits

| Use case | Pick |
| --- | --- |
| One-off edit, quick question, scripting Claude Code in CI | **CLI** |
| Long, structured engineering tasks where you'd otherwise hand-coordinate subagents | **OMC** |
| Codebases with rich LSP support where symbol-aware navigation pays off | **OMC** (LSP MCP tools) |
| Maximum control / minimum surprise from defaults | **CLI** (no hooks, no auto-routing) |
| Teams that want a shared opinionated harness | **OMC** (delegation rules explicit in CLAUDE.md) |

### TL;DR

Plain Claude Code CLI = the engine. OMC = an opinionated harness on top: more specialist subagents, more skills, more MCP tooling (LSP, AST, shared memory, wiki, notepad), conventional model/agent routing, and a verification discipline baked into the workflow. You can switch between them freely — OMC adds, it doesn't replace.
