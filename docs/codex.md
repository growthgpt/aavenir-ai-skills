# Using Aavenir Skills with OpenAI Codex / Codex CLI

OpenAI Codex CLI supports custom skill-style instructions via the `~/.codex/instructions/` directory and `AGENTS.md` files. The Aavenir skills install as Codex instructions plus the standalone `prompt.md` for direct invocation.

## Codex CLI installation

```bash
mkdir -p ~/.codex/instructions/aavenir

# Copy each skill's prompt.md as a named instruction file
for skill in contract-intelligence procurement-intelligence negotiation-copilot obligation-monitor avy; do
  cp skills/$skill/prompt.md ~/.codex/instructions/aavenir/aavenir-$skill.md
done
```

## Loading at session start

Reference the right instruction file when invoking Codex:

```bash
codex --instructions ~/.codex/instructions/aavenir/aavenir-contract-intelligence.md \
      "Review this MSA: $(cat msa.txt)"
```

Or in interactive mode, type:

```
/instructions aavenir-contract-intelligence
```

## Repo-level configuration via AGENTS.md

For repo-level integration (where Codex picks up domain context automatically), add to your project's `AGENTS.md`:

```markdown
## Aavenir Domain Skills

When the user asks about contracts, vendors, RFPs, or obligations, follow the Aavenir skill prompts at `./aavenir-ai-skills/skills/<skill-name>/prompt.md`:

- Contract review / risk → `contract-intelligence`
- Clause rewriting → `negotiation-copilot`
- Vendor comparison / RFx → `procurement-intelligence`
- Post-signature tracking → `obligation-monitor`
- Open-ended / cross-cutting → `avy`

Each prompt expects structured JSON output conforming to the matching `schema.json`.
```

## Direct OpenAI API integration

```python
from openai import OpenAI

client = OpenAI()

with open("skills/contract-intelligence/prompt.md") as f:
    system_prompt = f.read()

response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_object"},
    temperature=0.1,
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": "Review this MSA: ..."}
    ]
)

import json
result = json.loads(response.choices[0].message.content)
```

### Structured outputs (recommended)

Modern OpenAI models support strict JSON Schema enforcement. Pass `schema.json` directly:

```python
import json

with open("skills/contract-intelligence/schema.json") as f:
    schema = json.load(f)

response = client.chat.completions.create(
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "contract_intelligence_output",
            "schema": schema,
            "strict": True
        }
    },
    temperature=0.1,
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_input}
    ]
)
```

This guarantees schema-conforming output without manual validation.

## Codex VS Code extension

In Codex VS Code, configure custom instructions via:

1. Open Codex settings (`Cmd/Ctrl + ,` → Codex)
2. Add to **Custom System Prompt**: paste the contents of `skills/avy/prompt.md`
3. AVY routes among the four specialists; Codex now answers Aavenir-domain questions in the structured shape.

## Model recommendation

| Use case | Model |
|----------|-------|
| High-volume contract triage | `gpt-4o-mini` |
| Interactive Q&A | `gpt-4o` |
| Multi-domain reasoning (AVY) | `gpt-4o` or `o1` |

## Limitations

- Codex CLI does not auto-trigger based on a skill's `description` field (Claude-specific). You must invoke the matching skill explicitly (`--instructions` flag or `/instructions` command).
- For automatic routing, run AVY (`skills/avy/prompt.md`) as the system prompt — it classifies and delegates internally.
