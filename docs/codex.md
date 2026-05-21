# Using Aavenir Skills with OpenAI Codex / Codex CLI

OpenAI Codex CLI (2026) reads instructions from [`AGENTS.md`](https://developers.openai.com/codex/guides/agents-md) files — one canonical convention that's also auto-detected by Cline, Cursor, and other agentic tools. The Aavenir skills install as repo-level `AGENTS.md` content for routing, plus `prompt.md` files for direct programmatic use.

## Codex CLI: project-level setup (recommended)

In any repo where you want Aavenir context available, add an `AGENTS.md` at the root:

```markdown
# Aavenir Domain Skills

When the user asks about contracts, vendors, RFPs, suppliers, or obligations, follow the matching Aavenir skill prompt under `./aavenir-ai-skills-ecosystem/skills/<skill>/prompt.md`:

- Contract review, risk, extraction → `contract-intelligence`
- Clause rewriting, redlining → `negotiation-copilot`
- Vendor comparison, RFx scoring, TCO → `procurement-intelligence`
- Post-signature obligation tracking → `obligation-monitor`
- Cross-cutting or unclear → `avy` (router)

Each prompt expects structured JSON output conforming to the matching `schema.json`. Return JSON only, conforming to the schema, without prose preamble.
```

Codex walks from the git root to the current directory, concatenating every `AGENTS.md` it finds. Files closer to your working directory override earlier guidance.

## Codex CLI: global setup

To make Aavenir skills available across every repo (without copying `AGENTS.md` each time), add the same routing block to `~/.codex/AGENTS.md`. Project-level files override it where present.

If you need a global file that takes precedence over project-level files (rare — for safety overrides like "never auto-commit"), put it in `~/.codex/AGENTS.override.md` instead.

## Codex CLI: config.toml

`~/.codex/config.toml` holds CLI-level settings — model choice, MCP servers, sandbox policy. To expose the Aavenir skills as MCP tools Codex can call:

```toml
model = "gpt-5.5"
sandbox_mode = "workspace-write"

[mcp_servers.aavenir-skills]
command = "node"
args = ["./mcp/aavenir-skills.mjs"]
env = { AAVENIR_SCHEMA_DIR = "./skills" }
```

Project-trusted folders can override this with `.codex/config.toml`.

## Direct API integration (Codex / Responses)

For programmatic use, load `prompt.md` and call the Responses API with `json_schema` for guaranteed structured output:

```python
from openai import OpenAI
import json, pathlib

client = OpenAI()
skill = pathlib.Path("skills/contract-intelligence")

system_prompt = (skill / "prompt.md").read_text()
schema = json.loads((skill / "schema.json").read_text())

response = client.responses.create(
    model="gpt-5.5",
    instructions=system_prompt,
    input="Review this MSA: ...",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "contract_intelligence_output",
            "schema": schema,
            "strict": True
        }
    }
)

result = response.output_parsed  # Validated against schema
```

For agent orchestration (AVY routing across all five skills), prefer the OpenAI Agents SDK — see [`docs/openai-agents.md`](openai-agents.md).

## Codex VS Code extension

In the Codex VS Code extension, set the system prompt under Codex settings → Custom Instructions, and paste either:

- The contents of `skills/avy/prompt.md` (lets AVY route across all five specialists), or
- The contents of a single skill prompt if the workspace is domain-specific

Codex will treat those instructions as the assistant baseline for the workspace.

## Model recommendations (2026)

| Use case | Model |
|---|---|
| High-volume contract triage | `gpt-5-mini` |
| Interactive Q&A | `gpt-5.5` |
| Multi-domain reasoning (AVY) | `gpt-5.5` or `o3` |
| Negotiation rewrites (creative) | `gpt-5.5` at `temperature: 0.3` |

## Limitations

- Codex CLI does not auto-trigger from a skill's `description` field — that's a Claude-specific behavior. Either rely on the `AGENTS.md` routing block, or run AVY (`skills/avy/prompt.md`) as the system prompt for automatic delegation.
- Combined `AGENTS.md` content is capped at 32 KiB by default. If you stack multiple skill prompts inline, raise `project_doc_max_bytes` in `config.toml` or reference prompts by path instead of inlining them.
