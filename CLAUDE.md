# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Overview

This is the **StackOne agent-plugins marketplace** — a Claude Code plugin marketplace (and Skills-CLI catalog) of StackOne agent plugins for integration infrastructure (200+ connectors, MCP, A2A, SDKs) plus `stackone-defender`, a prompt-injection defense hook. It is primarily Markdown skills and JSON manifests; only `stackone-defender` ships executable code. Marketplace structure follows "one plugin per skill."

## Commands

Most of this repo is content (skills/manifests) with no build step. There is **no root package manager**; the only Node package lives in `plugins/security/stackone-defender` (npm, `package.json`, `type: module`, Node >= 22). Use **npm** there — do not introduce another package manager.

- `npx --yes @anthropic-ai/claude-code plugin validate .` — validate `marketplace.json` and all `plugin.json` manifests (this is exactly what CI `.github/workflows/validate.yml` runs on every push).
- `npm test` — run defender tests (`node --test tests/*.test.mjs`); run from `plugins/security/stackone-defender/` after `npm install`.

Install/use of the marketplace itself (from the README):
- `/plugin marketplace add stackonehq/agent-plugins` then `/plugin install <name>@stackone-agent-plugins` — Claude Code.
- `npx skills add stackonehq/agent-plugins[@<plugin>]` — any agent (Cursor, Codex, Windsurf, etc.).

## Architecture

```
.claude-plugin/marketplace.json   # Marketplace manifest — lists all 7 plugins; pluginRoot ./plugins
plugins/
  integrations/                   # 6 content-only plugins (Markdown skills)
    stackone-platform/            #   API keys, accounts, logs, webhooks
    stackone-connect/             #   account linking (Connect Sessions, Hub component)
    stackone-agents/              #   build agents via TS/Python SDK, MCP, A2A
    stackone-connectors/          #   connector/action discovery
    stackone-cli/                 #   custom connector dev & deploy
    stackone-unified-connectors/  #   schema-based unified connectors
  security/
    stackone-defender/            # the only plugin with executable code
.github/workflows/                # validate.yml (manifest validation), release-please.yml
```

Each plugin directory contains:
- `.claude-plugin/plugin.json` — plugin manifest (name, version, author, keywords).
- `README.md`.
- `skills/<skill-name>/SKILL.md` — the skill, with YAML frontmatter (`name`, `description`, `license`, `compatibility`, `metadata`). Detailed lookup tables live under `skills/<skill-name>/references/*.md`, loaded on demand.

`stackone-defender` additionally has:
- `hooks/hooks.json` — registers a `PostToolUse` hook matching `Bash|Read|WebFetch|WebSearch|Monitor|ReadMcpResourceTool|mcp__.*` that runs `scripts/scan-tool-result.mjs`.
- `scripts/` — `scan-tool-result.mjs` (hook entry point), `defender-daemon.mjs` + `defender-daemon.config.json` (local ML classifier daemon using `@huggingface/transformers` / `onnxruntime-node` / `fasttext.wasm`).
- `package.json`, `tests/` — Node test runner specs plus `tests/fixtures/` (benign / realistic / tricky sample tool outputs).

## Conventions

- **Versions are managed by release-please** (`.release-please-config.json`, `.release-please-manifest.json`); plugin/marketplace versions are kept in lockstep (currently `3.0.0`). Do not hand-bump versions outside that flow.
- Skills **teach workflows and point to live docs** (`docs.stackone.com`, `docs.stackone.com/llms.txt`) rather than duplicating frequently-changing API reference; keep new content in that style.
- Adding a plugin: create `plugins/<category>/<name>/` with a `.claude-plugin/plugin.json` and `skills/<name>/SKILL.md`, then add an entry to `.claude-plugin/marketplace.json`. Validate with the `plugin validate` command above.
- License is MIT.
