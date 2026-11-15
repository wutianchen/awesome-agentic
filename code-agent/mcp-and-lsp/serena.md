# Serena

Notes as of 2026-11-15. [Serena](https://github.com/oraios/serena) (oraios, about 31k stars) is "a powerful MCP toolkit for coding, providing semantic retrieval and editing capabilities — the IDE for your agent".

**In one line:** LSP is a protocol, and Serena is a tool built on it. Serena runs language servers underneath and turns what they know into tools a coding agent can call over [MCP](README.md#mcp).

## LSP vs Serena

| | LSP | Serena |
|---|---|---|
| What it is | a standard ([Language Server Protocol](https://microsoft.github.io/language-server-protocol/)) between editors and language servers | an MCP server for coding agents |
| Used by | editors such as VS Code and Neovim | agents such as Claude Code |
| Unit of work | positions: "what is at line 42, column 17?" | symbol names: `find_symbol("UserService/login")` |
| Setup | you run the server, open files, track positions | Serena starts the right language servers for you |
| Extras | none | project onboarding and memories (`.serena/memories`) |

Serena's main tools:

* **search** — `get_symbols_overview`, `find_symbol`, `find_declaration`, `find_implementations`, `find_referencing_symbols`
* **edit** — `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`, `rename_symbol`, `safe_delete_symbol` (edit a whole function without matching exact text)
* **checks** — `get_diagnostics_for_file`
* **memory** — `write_memory`, `read_memory`, `list_memories`, `onboarding`

## ty vs Serena

[ty](https://github.com/astral-sh/ty) (Astral, makers of uv and ruff) is an extremely fast Python type checker and language server written in Rust. It's in beta (`0.0.x`, no stable API yet).

| | ty | Serena |
|---|---|---|
| Level | a language server and type checker — the engine | an agent toolkit that drives language servers — the dashboard |
| Languages | Python only | many, each through its own language server |
| Gives you | type errors (`ty check`), go-to-definition, completions, hover help, auto-import | symbol-level search, editing, and memory for agents |

They complement each other. In principle ty could be the Python engine under Serena; Serena's README doesn't say which Python server it uses or whether ty can be selected, so check its docs. For agents, the simplest use of ty is running `ty check` after edits; any agent with a shell can do that.

## Ways to give an agent LSP abilities

* **Serena** — top consideration: symbol-level editing and memory, best for large codebases
* **Claude Code's built-in `LSP` tool** — direct language-server queries, nothing to install
* **OMC's `lsp_*` tools** — go-to-definition, references, rename, diagnostics, code actions (see [oh-my-claude](../omc/oh-my-claude.md))
* smaller MCP alternatives, by GitHub stars: [mcp-language-server](https://github.com/isaacphi/mcp-language-server), [lsp-mcp](https://github.com/jonrad/lsp-mcp)

## Adding expertise on top (e.g. OO design)

Serena can't make a language server smarter. It gives the model the facts it needs to apply expertise you write down. Split it into three parts:

1. **Facts — the language server, through Serena.** Class overviews, who implements a base class or `Protocol`, who calls a method. That's the raw material for judgments like "this class has 40 callers across 12 modules" or "one subclass ignores the base contract".
2. **Expertise — instructions you write.**
   * a skill, e.g. `oo-review/SKILL.md`, with rules and a procedure that uses Serena's tools
   * Serena memories for project conventions, e.g. "repositories return domain objects, never ORM rows"
   * `CLAUDE.md` / `AGENTS.md` for short rules that always apply
3. **Enforcement — deterministic tools.** The model can be inconsistent, so rules that must always hold should fail a check:
   * ty: classes really satisfy their `Protocol` / ABC contracts; overrides have compatible signatures
   * ruff or pylint rules, or a small custom pylint checker: class size, inheritance depth
   * [import-linter](https://github.com/seddonym/import-linter): layering, e.g. "domain must not import infrastructure"

Example skill:

```markdown
---
name: oo-review
description: Review Python class design. Use when adding or changing classes, inheritance, or interfaces.
---
1. Use Serena get_symbols_overview on the changed modules.
2. For each changed base class or Protocol, use find_implementations and check that
   every subclass honors the base contract (Liskov).
3. Use find_referencing_symbols to judge coupling before moving or splitting a class.
4. Rules: composition over inheritance; inheritance depth ≤ 2; interfaces as typing.Protocol; ...
5. Run `ty check` and fix type errors before reporting.
```

Serena supplies the facts, the skill supplies the judgment, and ty and linters enforce the rules. The same `SKILL.md` format works in tau, but only agents with Serena connected get its tools.

## In this repo

The `.serena/` folder at the repo root holds Serena's project config and memories. It is neither tracked nor ignored by git; `project.local.yml` usually holds machine-specific settings, so it likely belongs in `.gitignore`.
