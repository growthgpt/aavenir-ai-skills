# Using Aavenir Skills with Claude

The Aavenir AI Skill Ecosystem is **natively supported** across all Claude surfaces: Claude Code (CLI), Claude Desktop, claude.ai web, and the Anthropic API.

## Claude Code (CLI)

### Install as a plugin

```bash
# Clone into your Claude plugins directory
git clone https://github.com/aavenir/aavenir-ai-skills.git \
  ~/.claude/plugins/aavenir-ai-skills

# Enable in your Claude settings (or use /plugin enable)
```

The skills auto-discover. In any Claude Code session, mention a contract, RFP, vendor, or obligation and the matching skill activates.

### Install individual skills

If you only need one capability:

```bash
mkdir -p ~/.claude/skills
cp -r skills/contract-intelligence ~/.claude/skills/aavenir-contract-intelligence
```

Restart Claude Code or run `/skills reload`.

### Verify

```
> What skills do I have?

> Review this MSA: <paste contract>
```

Claude will invoke `aavenir-contract-intelligence` automatically.

---

## Claude Desktop / claude.ai (web)

### Install as packaged skills

1. Open Claude.ai or Claude Desktop
2. Navigate to **Skills** in the sidebar
3. Click **Add Skill**
4. Upload the skill folder as a `.zip` (e.g., `contract-intelligence.zip` containing `SKILL.md`, `prompt.md`, `schema.json`, `examples.json`)

Repeat for each of the five skills.

### Verify

Start a new chat and paste a contract — `aavenir-contract-intelligence` will trigger from its description.

---

## Anthropic API (direct integration)

For programmatic use (e.g., embedding into your own application or the Aavenir platform), load `prompt.md` as the system prompt:

```python
from anthropic import Anthropic

client = Anthropic()

with open("skills/contract-intelligence/prompt.md") as f:
    system_prompt = f.read()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=4096,
    system=system_prompt,
    messages=[
        {"role": "user", "content": "Review this MSA: ..."}
    ]
)

# response.content[0].text is JSON conforming to schema.json
import json
result = json.loads(response.content[0].text)
```

### With prompt caching

These prompts are designed to be cached. Aavenir's production deployment uses prompt caching for system + few-shot examples (loaded from `examples.json`):

```python
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=4096,
    system=[
        {
            "type": "text",
            "text": system_prompt,
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": user_input}
    ]
)
```

Cache hit reduces per-call cost by ~90% for the system prompt portion.

### Structured output validation

Every skill's output conforms to its `schema.json`. Validate before consuming:

```python
import jsonschema

with open("skills/contract-intelligence/schema.json") as f:
    schema = json.load(f)

jsonschema.validate(instance=result, schema=schema)
```

---

## Recommended models

| Use case | Model |
|----------|-------|
| Contract review at scale (batched) | `claude-haiku-4-5-20251001` |
| Interactive contract Q&A | `claude-sonnet-4-6` |
| Complex multi-domain reasoning (AVY) | `claude-opus-4-7` |
| Negotiation rewrites (creative variation) | `claude-sonnet-4-6` at `temperature: 0.3` |

---

## MCP integration

The skills are MCP-compatible. To expose them as MCP tools (e.g., for the Aavenir AVY assistant in ServiceNow):

```json
{
  "mcpServers": {
    "aavenir-skills": {
      "command": "node",
      "args": ["./mcp-server/index.js"],
      "env": {
        "AAVENIR_SKILLS_PATH": "./skills"
      }
    }
  }
}
```

Each skill exposes one MCP tool (`aavenir-contract-intelligence`, etc.) with `schema.json` as the input/output schema.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Skill doesn't trigger | Verify SKILL.md frontmatter is loaded. Check `description` field is present. |
| Output is not JSON | Lower `temperature` to 0.0–0.2. Verify you used `prompt.md` (not `SKILL.md`) for direct API calls. |
| Output doesn't match schema | Run schema validation; surface the validation error back to the model in a follow-up turn. |
| Wrong skill triggered | Sharpen `description` triggers in SKILL.md (see `aavenir-avy` for routing logic). |
