# pi — local metadata under `~/.pi`

What [pi](https://pi.dev/) stores on disk and what each file is for. Backed by the [official docs](https://pi.dev/docs/latest); observed layout from a real install (pi 0.82.x, macOS).

```
~/.pi/
└── agent/
    ├── auth.json          credentials (0600) — written by /login
    ├── settings.json      global settings
    ├── models.json        user-authored custom models / providers (optional — never auto-created)
    ├── models-store.json  cached provider model catalogs (offline use)
    ├── bin/               vendored tool binaries: rg, fd
    └── sessions/          session history, one folder per working directory
        └── --Users-me-Projects--/
            └── 2026-07-26T14-32-55-267Z_<uuid>.jsonl
```

| File / dir | Who writes it | What it is |
|---|---|---|
| `auth.json` | `/login` | OAuth tokens + API keys per provider |
| `settings.json` | pi (via `/model`, theme changes, …) | global defaults; project-level override exists |
| `models.json` | **you** (by hand) | custom providers / model overrides |
| `models-store.json` | pi (catalog refresh) | cached model lists per provider, with `etag` / `lastModified` |
| `bin/` | pi installer | bundled `rg` + `fd` so search tools work without host installs |
| `sessions/` | pi (every turn) | JSONL session trees, grouped by cwd |

## `auth.json` — credentials ([docs: providers](https://pi.dev/docs/latest/providers))

Written by `/login`, created with `0600` permissions. Two entry shapes:

* **OAuth** (Claude Pro/Max, ChatGPT, GitHub Copilot, OpenRouter, …): `{type, access, refresh, expires}` — tokens auto-refresh on expiry
* **API key**: `{"type": "api_key", "key": "sk-…"}` — values support `$ENV_VAR` interpolation and `!command` shell resolution

Credential resolution order: CLI flag → `auth.json` → provider env var (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, …) → custom provider keys from `models.json`. Note: `auth.json` **beats** env vars.

Secret — never commit or copy this file.

## `settings.json` — global settings ([docs: settings](https://pi.dev/docs/latest/settings))

Global config for all projects; `.pi/settings.json` inside a project overrides it (nested objects are merged). Fields seen in practice:

```json
{
  "lastChangelogVersion": "0.82.1",
  "theme": "dark",
  "defaultProvider": "openrouter",
  "defaultModel": "moonshotai/kimi-k2.7-code",
  "defaultThinkingLevel": "medium"
}
```

Other supported keys: `compaction.{enabled,reserveTokens,keepRecentTokens}`, `retry.{enabled,maxRetries,baseDelayMs}`, `httpProxy`, `shellPath` / `shellCommandPrefix` / `npmCommand`, `sessionDir`, `enabledModels` (model-cycling list), `externalEditor`, `quietStartup`, `defaultProjectTrust`, and resource lists (`packages`, `extensions`, `skills`, `prompts`, `themes`).

## `models.json` — custom models / providers ([docs: models](https://pi.dev/docs/latest/models), [custom provider](https://pi.dev/docs/latest/custom-provider))

**Not auto-created** — pi only reads it if you write it. Reloaded every time the `/model` selector opens (Ctrl+L), no restart needed. Defines new providers or overrides registry models; an entry with the same `id` as a built-in shadows it.

```json
{
  "providers": {
    "openrouter": {
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKey": "$OPENROUTER_API_KEY",
      "api": "openai-completions",
      "models": [
        {
          "id": "moonshotai/kimi-k2.7-code",
          "name": "Kimi K2.7 Code (capped 8k)",
          "reasoning": true,
          "input": ["text", "image"],
          "contextWindow": 262144,
          "maxTokens": 8192,
          "cost": { "input": 0.82, "output": 3.75, "cacheRead": 0.15, "cacheWrite": 0 }
        }
      ]
    }
  }
}
```

Key model fields: `id` (what the API receives), `name` (UI label), `contextWindow`, `maxTokens` (**max output tokens pi will request**), `cost` (per-MTok), `reasoning`, `input`.

Practical use: capping `maxTokens`. The registry entry for a model may set `maxTokens` = full context window, so pi requests `max_tokens = context − prompt` (e.g. 234K) — on OpenRouter that fails the pre-flight credit check with a 402 ("requires more credits, or fewer max_tokens") on small balances. An override with `maxTokens: 8192` fixes it.

## `models-store.json` — cached model catalogs ([docs: providers](https://pi.dev/docs/latest/providers))

Per-provider cache of the model catalog, refreshed from configured providers and kept "for offline use". Shape: `{<provider>: {models: [...], checkedAt, lastModified, etag}}` — e.g. anthropic (15 models), openrouter (276 models). This is where built-in model metadata (`contextWindow`, `maxTokens`, `cost`) comes from when you pick a model without a `models.json` override. Cache — safe to inspect, pointless to edit (gets refreshed).

## `sessions/` — session trees ([docs: sessions](https://pi.dev/docs/latest/sessions))

One subfolder per working directory (path encoded with dashes), one JSONL file per session, named `<ISO timestamp>_<uuid>.jsonl`.

Sessions are **trees, not linear logs**: every entry has an `id` and `parentId`, and the current position is the active leaf — so you can jump to any prior point and branch without a new file. Entry types observed in a real session:

| type | payload |
|---|---|
| `session` | header: `cwd`, `version`, `timestamp` — first line of the file |
| `message` | one user/assistant/tool turn |
| `model_change` | `provider` + `modelId` switch mid-session |
| `thinking_level_change` | `thinkingLevel` switch |

The docs list further types: labels, compactions, branch summaries, extension entries.

Navigation: `pi -c` (continue last), `pi -r` (session browser), `pi --session <path|id>`; in-TUI `/resume` (picker), `/tree` (jump to any point), `/fork` (new file from an earlier message), `/clone` (duplicate active branch).

## `bin/` — vendored tools

Standalone binaries pi ships so its search tools work regardless of what's on the host: `rg` (ripgrep 15.2.0) and `fd` (10.4.2), native executables for the install platform.

## References — official documentation

* [Docs index](https://pi.dev/docs/latest)
* [Quickstart](https://pi.dev/docs/latest/quickstart) — install, authentication, first session
* [Using Pi](https://pi.dev/docs/latest/usage) — interactive mode, slash commands, CLI reference
* [Keybindings](https://pi.dev/docs/latest/keybindings) — default shortcuts (Ctrl+L model selector, Ctrl+D exit)
* [Settings](https://pi.dev/docs/latest/settings) — `settings.json`, project-level overrides
* [Providers](https://pi.dev/docs/latest/providers) — `/login`, `auth.json`, env vars, catalog caching
* [Custom Models](https://pi.dev/docs/latest/models) — `models.json` model entries
* [Custom Providers](https://pi.dev/docs/latest/custom-provider) — `models.json` provider entries
* [Sessions](https://pi.dev/docs/latest/sessions) — JSONL tree format, `/resume`, `/tree`, `/fork`, `/clone`
* [Extensions](https://pi.dev/docs/latest/extensions) · [Skills](https://pi.dev/docs/latest/skills) · [Prompt Templates](https://pi.dev/docs/latest/prompt-templates) · [Themes](https://pi.dev/docs/latest/themes)
* [SDK](https://pi.dev/docs/latest/sdk) · [RPC Mode](https://pi.dev/docs/latest/rpc) · [JSON Event Stream](https://pi.dev/docs/latest/json) — programmatic access (the layer-3 doors)
