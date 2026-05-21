# Using Aavenir Skills with Cursor

Cursor 2026 supports custom AI behavior through [**Project Rules**](https://cursor.com/docs/context/rules) — markdown files with frontmatter under `.cursor/rules/`. The Aavenir skills install as MDC rule files, one per skill, with the right activation strategy for each.

> Cursor's legacy `.cursorrules` single-file format still works but is superseded by `.cursor/rules/`. New projects should use rules; the patterns below assume the modern path.

## Quick install

From the root of any project where you want Aavenir skills available:

```bash
mkdir -p .cursor/rules

for skill in contract-intelligence procurement-intelligence negotiation-copilot obligation-monitor avy; do
  cp aavenir-ai-skills-ecosystem/skills/$skill/prompt.md \
     .cursor/rules/aavenir-$skill.mdc
done
```

Then add frontmatter to each file so Cursor knows when to attach the rule (see below).

## Rule activation strategies

Cursor supports four rule types. Use the matching frontmatter for each Aavenir skill:

### Agent-requested (recommended for skill prompts)

The AI decides when to load the rule based on the `description`. Best fit for domain skills — they activate only when the conversation hits the trigger.

```markdown
---
description: Enterprise contract review and risk analysis. Use when reviewing an MSA, NDA, SOW, SaaS agreement, DPA, or any legal text. Triggers on words like indemnify, term, termination, warranty, liability, governing law, force majeure.
---

(paste contents of skills/contract-intelligence/prompt.md here)
```

### Auto-attached (file-scoped)

Auto-load the skill whenever the user opens a file matching a glob — useful for repos where contract files live in a known folder.

```markdown
---
description: Aavenir contract analysis for files in /contracts and /legal directories
globs: contracts/**/*, legal/**/*.{md,txt,pdf}
---

(skill prompt)
```

### Always-apply (workspace-wide)

Pin a rule into every conversation in this project. Reserve for the AVY router in workspaces where Aavenir is the dominant domain.

```markdown
---
description: Aavenir AVY — unified router across contracts, procurement, negotiation, obligations
alwaysApply: true
---

(contents of skills/avy/prompt.md)
```

### Manual

Reference via `@aavenir-contract-intelligence` in chat. No frontmatter trigger fields.

## Suggested rule set

| File | Frontmatter | Source |
|------|-------------|--------|
| `.cursor/rules/aavenir-contract-intelligence.mdc` | `description` (agent-requested) | `skills/contract-intelligence/prompt.md` |
| `.cursor/rules/aavenir-negotiation-copilot.mdc` | `description` (agent-requested) | `skills/negotiation-copilot/prompt.md` |
| `.cursor/rules/aavenir-procurement-intelligence.mdc` | `description` (agent-requested) | `skills/procurement-intelligence/prompt.md` |
| `.cursor/rules/aavenir-obligation-monitor.mdc` | `description` (agent-requested) | `skills/obligation-monitor/prompt.md` |
| `.cursor/rules/aavenir-avy.mdc` | `alwaysApply: true` *(optional)* | `skills/avy/prompt.md` |

Pulled from each skill's `SKILL.md` frontmatter, the `description` is already calibrated to trigger the right rule.

## User Rules (global preferences)

For preferences that apply across every project (e.g., "always return structured JSON conforming to schema.json"), use Cursor's **User Rules** in Settings → Cursor → Rules for AI. These live in your Cursor profile, not in any repo.

## Chat usage

In any Cursor chat, you can also pull a skill into context explicitly:

```
@.cursor/rules/aavenir-contract-intelligence.mdc

Review this MSA and flag risks: <paste contract>
```

Or with `Cmd/Ctrl + K` over a highlighted clause:

```
Use the negotiation-copilot rule to rewrite this clause from the customer side. Give me three positions.
```

## MCP integration (optional)

Cursor 2026 supports MCP servers natively. To expose the Aavenir skills as MCP tools that Cursor's agent can call (rather than as rules in context), drop an entry into `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "aavenir-skills": {
      "command": "node",
      "args": ["./mcp/aavenir-skills.mjs"],
      "env": { "AAVENIR_SCHEMA_DIR": "./skills" }
    }
  }
}
```

Rules and MCP are complementary — rules give Cursor your prompt context; MCP gives it callable tools that return structured JSON conforming to each skill's `schema.json`.

## Limitations and tips

- **Keep rules under 500 lines.** Long skill prompts should be split into multiple composable rules rather than a single monolithic file.
- **Cursor does not enforce JSON Schema natively.** Configure your downstream consumer to validate against `schema.json`, or use MCP tool calls for guaranteed structure.
- **Rules are version-controlled.** `.cursor/rules/` should be committed so the whole team gets the same Aavenir behavior.
