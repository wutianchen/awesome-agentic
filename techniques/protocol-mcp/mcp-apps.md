# MCP Apps

Notes as of 2026-11-17. The comparison is from general knowledge and not checked against the spec. Field names such as `_meta.ui.resourceUri` and the list of supporting hosts may have changed; check the [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview.md) before relying on them.

## MCP vs MCP Apps

**MCP Apps is an optional extension of MCP, not a separate protocol.** With plain [MCP](README.md), an agent calls a server's tools and gets back data as text or JSON. With MCP Apps, a tool can also return an interactive UI that the chat app shows inline.

| | MCP | MCP Apps |
|---|---|---|
| What it is | the core protocol for connecting AI apps to tools, resources and prompts | an official optional extension to MCP |
| A tool returns | text, JSON or images, read by the model | the same, plus a reference to an HTML interface that the host displays |
| Who uses the result | mainly the model | the model and the person, who can click, filter and fill in forms |
| How the UI is provided | no UI layer | the server publishes HTML as a `ui://` resource, and the tool's metadata points to it |
| Where it runs | the server runs as its own process, local or remote | the HTML runs in a sandboxed iframe inside the host |
| Communication | JSON-RPC between client and server | the same, plus JSON-RPC over `postMessage` between the iframe and the host, so the UI can call tools or update the chat |
| Host support | almost every agent: Claude Code, Codex, Cursor and others | only hosts that can render UI, such as Claude, ChatGPT, VS Code and Goose. Terminal agents fall back to the plain result |

### Example

A sales MCP server has a tool called `get_sales`:

* **plain MCP** — it returns a table of numbers, and the model summarizes it in text
* **MCP Apps** — it also returns a chart dashboard that appears in the conversation. Filtering the chart triggers further tool calls, without retyping prompts

### Origin

The community [MCP-UI](https://mcpui.dev) project and OpenAI's Apps SDK were merged into an official MCP extension, launched in Claude on 2026-01-26, so one app can work in several hosts.

### Where it matters in these notes

* **OpenClaw** — its Control UI can pin interactive MCP Apps as session widgets. This is a way to add your own views to a control panel without forking the UI, see [openclaw.md](../../openclaw.md#control-ui)
* **Claude Code** — it is a terminal agent, so you get the tool's data but not the visual UI
* **Security** — an app's HTML is untrusted code. Hosts sandbox it and require approval before it calls tools

## Popular MCP Apps

No usage rankings are published, so "popular" means the apps most often cited at launch and in the official examples. The partner list comes from news coverage and blogs, not from Anthropic's own announcement.

### Launch partners in Claude

Announced 2026-01-26. Reported lists have nine or ten partners.

| App | Category | What you get in the chat |
|---|---|---|
| Slack | messaging | draft and preview messages before sending |
| Figma | design | build diagrams in FigJam |
| Canva | design | design presentations |
| Asana | project management | project timelines and tasks |
| monday.com | project management | boards and work items |
| Amplitude | analytics | live analytics charts you can filter |
| Hex | analytics | data notebooks and charts |
| Box | files | browse and preview files |
| Clay | sales data | data enrichment and prospect tables |
| Salesforce | CRM | named by one outlet; another says its Agentforce, Data 360 and Customer 360 apps come later |

### Official example apps

From the [`ext-apps`](https://github.com/modelcontextprotocol/ext-apps) repo; useful as templates for your own app.

| Example | What it shows |
|---|---|
| `map-server` | interactive map |
| `threejs-server` | 3D scenes |
| `pdf-server` | document viewer |
| `system-monitor-server` | live system dashboard; a starting point for an [OpenClaw](../../openclaw.md) control-panel widget |
| `sheet-music-server` | music notation |
| ShaderToy | shader viewer |
| Wiki Explorer | Wikipedia browsing |
| Cohort Heatmap | analytics heatmap |

### Hosts

* **Claude** — web and desktop apps. Anyone can build an MCP App; only a listing in Claude's app directory needs Anthropic's review
* **ChatGPT** — has adopted MCP Apps for third-party apps. A third-party guide says it has been fully compatible since 2026-02-22; not checked against OpenAI's changelog
* **VS Code and Goose** — also cited as supporting it

## Sources

* [The New Stack: Anthropic extends MCP with an app framework](https://thenewstack.io/anthropic-extends-mcp-with-an-app-framework/)
* [Engadget: Claude now offers deeper integrations with apps like Canva and Slack](https://engadget.com/ai/claude-now-offers-deeper-integrations-with-apps-like-canva-and-slack-180000604.html)
* [The Decoder: Anthropic brings Asana, Figma, Slack and more into Claude](https://the-decoder.com/anthropic-brings-asana-figma-slack-and-more-tools-directly-into-claude-as-interactive-apps/)
* [Conversion (BR): Claude lança apps interativos](https://www.conversion.com.br/?p=52111)
* [Alpic: MCP Apps goes official, Claude and ChatGPT support](https://alpic.ai/blog/mcp-apps-goes-official-claude-chatgpt-support)
* [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview.md)
* [sunpeak: MCP App examples](https://sunpeak.ai/blogs/mcp-app-examples/)
* [MCP Playground: Claude MCP Apps full guide](https://mcpplaygroundonline.com/blog/claude-mcp-apps-full-guide)
