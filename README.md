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

```bash
# Option A — clone as a plugin
git clone https://github.com/growthgpt/aavenir-ai-skills.git \
  ~/.claude/plugins/aavenir-ai-skills

# Option B — install individual skills
cp -r skills/contract-intelligence ~/.claude/skills/aavenir-contract-intelligence
```

Then in any Claude Code session: ask anything about a contract, RFP, vendor, or obligation. The relevant skill auto-triggers.

## Install (other surfaces)

- **OpenAI Agents / Assistants API** — see [`docs/openai-agents.md`](docs/openai-agents.md)
- **Codex / Codex CLI** — see [`docs/codex.md`](docs/codex.md)
- **Cursor** — see [`docs/cursor.md`](docs/cursor.md)
- **Claude (Desktop, Code, claude.ai)** — see [`docs/claude.md`](docs/claude.md)
- **MCP-compatible clients** — load `prompt.md` as the system prompt; consume `schema.json` for structured output

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

| Host | Status |
|------|--------|
| Claude Code (CLI) | Native |
| Claude.ai (web) | Native |
| Claude Desktop | Native |
| OpenAI Assistants / Responses API | Via `prompt.md` |
| Codex CLI | Via `prompt.md` |
| Cursor | Via `.cursorrules` + `prompt.md` |
| Cline / Roo Code | Via custom modes + `prompt.md` |
| MCP clients | Native (uses `schema.json` for tool I/O) |
| ServiceNow + Aavenir AVY | Native (Aavenir Flows consume `schema.json`) |

---

## License

Apache-2.0. Aavenir, Contractflow, Obligationflow, RFPflow, Invoiceflow, and AVY are trademarks of Aavenir Inc.
