# MCP & LSP

## MCP

* aws
* kubernetes
* prometheus
* github
* [Ponytail](https://github.com/DietrichGebert/ponytail)


## LSP

Language Server Protocol

#### LSP for agents

* [Serena](serena.md) — MCP toolkit that turns language servers into symbol-level tools for agents; also compares Serena with LSP and ty

#### Python

* [ty](https://github.com/astral-sh/ty)
* [pylsp](https://github.com/python-lsp/python-lsp-server)


#### Helm

tbd


## Automated refactoring

Deterministic, type-aware code transformations. An agent runs a recipe instead of editing files one by one, so large changes come out exact and repeatable.

* [OpenRewrite](https://docs.openrewrite.org/) — mass refactoring engine maintained by Moderne. It runs prepackaged recipes for framework migrations, security fixes, and style consistency over a Lossless Semantic Tree (LST), a type-aware model of the code that keeps formatting. Parsers for Java (strongest), Kotlin, Groovy, JavaScript/TypeScript, Python, C#, and config files such as Maven, Gradle, YAML, and Docker. Tiered licensing: the framework is open source, but not every recipe is. Moderne is the commercial platform that runs recipes across many repos