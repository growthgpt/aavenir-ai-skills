# Using Aavenir Skills with Cline (and other MCP clients)

[Cline](https://cline.bot) (2026) is a VS Code AI coding agent that reads project rules from a `.clinerules/` folder and natively supports MCP servers and AGENTS.md files. The Aavenir skills install as a set of routed Cline rules — one file per skill — plus an optional MCP server for structured tool calls.

## Quick install (project rules)

From the root of any project where you want Aavenir context available in Cline:

```bash
mkdir -p .clinerules

for skill in contract-intelligence procurement-intelligence negotiation-copilot obligation-monitor avy; do
  cp aavenir-ai-skills-ecosystem/skills/$skill/prompt.md \
     .clinerules/aavenir-$skill.md
done
```

Cline reads every `.md` and `.txt` file inside `.clinerules/` and combines them into a unified rule set. No restart needed — Cline picks up changes on the next message.

## Conditional rules (file-scoped activation)

For skills that should only attach when working in specific folders (e.g., contract review only in `/contracts`), add YAML frontmatter:

```markdown
---
paths:
  - "contracts/**"
  - "legal/**/*.md"
---

(contents of skills/contract-intelligence/prompt.md)
```

## Cross-tool compatibility

Cline auto-detects rules from neighboring tools, so if your repo already has any of these, Cline will read them without extra setup:

| File | Source |
|---|---|
| `.clinerules/` | Cline's native folder |
| `.cursor/rules/*.mdc` | Cursor — see [docs/cursor.md](cursor.md) |
| `.cursorrules` | Cursor legacy single-file |
| `AGENTS.md` | Codex / shared convention — see [docs/codex.md](codex.md) |
| `.windsurfrules` | Windsurf |

If you already shipped Aavenir as an `AGENTS.md` block for Codex, Cline will use that automatically — no duplication needed.

## MCP integration (structured tool calls)

To expose the Aavenir skills as MCP tools Cline's agent can call (rather than as rules in context), add an entry to Cline's MCP settings:

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

Each skill becomes one MCP tool (`aavenir-contract-intelligence`, etc.) with `schema.json` as the I/O contract. The agent can now call them as functions that return validated structured JSON.

Rules and MCP are complementary — rules give Cline the prompt context for *how* to reason; MCP gives it callable tools for *guaranteed* structured output.

## Memory Bank pattern

Cline's [Memory Bank](https://docs.cline.bot/) lets the agent maintain context across sessions via a structured set of files. For Aavenir-heavy workspaces, add the AVY router prompt to your Memory Bank's project brief — it'll be loaded at the start of every session.

## Other MCP clients

The same `.mcp.json` pattern works for any MCP-compatible client:

- **Roo Code** (Cline fork) — reads `.clinerules/` and `.roo/mcp.json`
- **Kilo Code** — reads `AGENTS.md` and MCP configs
- **Claude Desktop** — MCP config at `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)
- **Continue.dev** — uses `~/.continue/config.yaml`

For most of these, the Aavenir routing block in `AGENTS.md` is the cheapest distribution path — it's one file every modern agent reads.

## Limitations

- Cline rules are concatenated into one unified set, so keep individual rule files focused (under ~500 lines). Long skill prompts are fine, but layering five full skill prompts at once will eat context budget.
- For agentic workflows that need guaranteed structured output, prefer the MCP tool path over rules — the MCP server can enforce `schema.json` validation; rules cannot.
