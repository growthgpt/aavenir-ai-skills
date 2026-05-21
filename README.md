# Aavenir AI Skill Ecosystem

> Enterprise legal & procurement intelligence skills that work inside Claude, Codex, Cursor, OpenAI Agents, Cline, and any MCP-compatible AI assistant.

Aavenir builds AI-powered Contract Lifecycle Management (CLM) and Source-to-Pay (S2P) workflows on ServiceNow. This repository packages Aavenir's domain expertise as **portable AI skills** — drop-in capability modules that give any modern AI assistant the same contract, procurement, and obligation reasoning that powers the Aavenir platform.

A skill is not a product. It is a distribution asset. Once installed, the skill loads automatically whenever a user starts working on a relevant task — and outputs Aavenir-grade structured analysis without anyone learning a new tool.

---

## What's in the box

| Skill | Purpose | Primary surface |
|-------|---------|------------------|
| [`aavenir-contract-intelligence`](skills/contract-intelligence) | Contract review, clause extraction, risk scoring, obligation pulling, missing-clause detection, contract Q&A | Legal, CLM admins |
| [`aavenir-procurement-intelligence`](skills/procurement-intelligence) | RFx analysis, supplier scoring, vendor comparison, sourcing recommendations | Procurement, sourcing |
| [`aavenir-negotiation-copilot`](skills/negotiation-copilot) | Redline suggestions, clause rewrite, fallback positions, legal-language simplification | Legal, sales ops |
| [`aavenir-obligation-monitor`](skills/obligation-monitor) | Post-signature obligation extraction, deadline tracking, renewal alerts, escalation triggers | Contract ops, compliance |
| [`aavenir-avy`](skills/avy) | Unified Aavenir assistant — routes natural-language questions across all four domain skills | All users |

Each skill ships four files:

```
skill-name/
├── SKILL.md         # Frontmatter + behavior contract loaded by the host AI
├── prompt.md        # Drop-in system prompt for direct LLM API calls
├── examples.json    # Few-shot examples calibrated by Aavenir CLM experts
└── schema.json      # JSON Schema for structured output (parses cleanly into Aavenir)
```

---

## Why this matters

The contract & procurement workflow is fragmenting across surfaces — legal teams paste clauses into ChatGPT, sales ops asks Cursor to draft an MSA, procurement runs RFP analysis in Claude. Without a shared spine, every team gets a different answer.

These skills enforce a single Aavenir-defined contract taxonomy, risk model, and output schema across every assistant the user touches. The output goes back into Aavenir Contractflow, Obligationflow, RFPflow, Invoiceflow, and AVY with **zero translation**.

---

## Install (Claude Code)

The repo is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugins). Install it directly from GitHub — no clone, no zip, no manual copy.

```
/plugin marketplace add growthgpt/aavenir-ai-skills
/plugin install aavenir-ai-skills@aavenir
/reload-plugins
```

That's it. Use the plugin in any Claude Code session — mention a contract, RFP, vendor, or obligation and the matching skill auto-triggers.

### Choose **user scope** when prompted

Claude Code offers three install scopes:

- **User** (default, recommended) — installed for you across every project. The skill activates from any folder.
- **Project** — installed for everyone who clones this repo. Blocks the skill from reading files outside the current project folder.
- **Local** — installed for you in this repo only.

For Aavenir skills, pick **user scope**. You usually want to review contracts that live in `Downloads`, `Documents`, or Dropbox — project scope would block that.

To switch later: `/plugin uninstall aavenir-ai-skills@aavenir`, then `/plugin install aavenir-ai-skills@aavenir` from your home directory.

### Picking up updates

When the marketplace publishes a new version, run `/reload-plugins` to apply changes without restarting. Auto-update is on by default for third-party marketplaces — check the **Marketplaces** tab in `/plugin` to toggle.

## Install (other surfaces)

- **Claude (Code · Desktop · claude.ai)** — `/plugin marketplace add growthgpt/aavenir-ai-skills` — see [`docs/claude.md`](docs/claude.md)
- **Cursor** — `.cursor/rules/*.mdc` Project Rules with MDC frontmatter — see [`docs/cursor.md`](docs/cursor.md)
- **OpenAI Codex / Codex CLI** — `AGENTS.md` routing block + optional MCP via `~/.codex/config.toml` — see [`docs/codex.md`](docs/codex.md)
- **OpenAI Agents SDK** — `Agent(name=..., output_type=PydanticModel)` + `handoff()` for AVY routing — see [`docs/openai-agents.md`](docs/openai-agents.md)
- **Cline / Roo Code / other VS Code agents** — `.clinerules/` folder, also auto-detects `AGENTS.md` — see [`docs/cline.md`](docs/cline.md)
- **Any MCP-compatible client** — drop the skills as MCP server tools; `schema.json` becomes the I/O contract

---

## Output contract

Every skill returns JSON conforming to its `schema.json`. Every JSON payload carries:

```json
{
  "skill": "aavenir-contract-intelligence",
  "version": "1.0.0",
  "generated_at": "2026-05-20T14:23:00Z",
  "confidence": "high",
  "result": { ... }
}
```

This means an Aavenir Flow or any downstream system can deserialize output deterministically without prompt-by-prompt parsing.

---

## Compatibility

| Host | Status | Install path |
|------|--------|--------------|
| Claude Code | Native plugin | `/plugin marketplace add growthgpt/aavenir-ai-skills` |
| Claude Desktop / claude.ai | Native skills | Settings → Capabilities → Skills → upload `.zip` per skill |
| Cursor | Native rules | `.cursor/rules/*.mdc` with MDC frontmatter |
| OpenAI Codex CLI | Convention | `AGENTS.md` (project) + `~/.codex/AGENTS.md` (global) |
| OpenAI Agents SDK | Native | `Agent(output_type=…)` + `handoff()` |
| OpenAI Responses / Assistants API | Direct | `prompt.md` + `schema.json` via `response_format` |
| Cline / Roo Code | Native rules | `.clinerules/` folder (also reads `AGENTS.md`) |
| Any MCP client | Native | `schema.json` becomes the tool I/O contract |
| ServiceNow + Aavenir AVY | Native | Aavenir Flows consume `schema.json` directly |

---

## License

Apache-2.0. Aavenir, Contractflow, Obligationflow, RFPflow, Invoiceflow, and AVY are trademarks of Aavenir Inc.
