# Using Aavenir Skills with Claude

The Aavenir AI Skill Ecosystem is **natively supported** across every Claude surface: Claude Code (CLI), Claude Cowork (team workspaces in Claude Desktop), Claude Desktop / claude.ai web, and the Anthropic API. The same `.claude-plugin/marketplace.json` powers Claude Code and Cowork; claude.ai web uses skill `.zip` uploads.

## Claude Code (CLI)

The repo ships as a Claude Code plugin marketplace. Install directly from GitHub — no clone or zip download needed.

### Install

```
/plugin marketplace add https://github.com/growthgpt/aavenir-ai-skills.git
/plugin install aavenir-ai-skills@aavenir
/reload-plugins
```

> Use the full HTTPS URL. The `growthgpt/aavenir-ai-skills` GitHub shorthand defaults to SSH cloning and fails with `Permission denied (publickey)` unless you have GitHub SSH keys configured.

When `/plugin install` asks for scope, pick **user scope** (default). Project scope blocks the skill from reading files outside the current project folder — most contract review work needs access to `Downloads`, `Documents`, or Dropbox.

You can also install locally from a clone (useful for plugin development):

```
/plugin marketplace add /path/to/aavenir-ai-skills-ecosystem
```

Tip: after typing `/plugin marketplace add ` (with a trailing space), drag the folder onto the terminal to autofill the path.

### Verify

```
/plugin                            # opens the manager; confirm aavenir-ai-skills is installed
Review this MSA: <paste contract>  # contract-intelligence auto-triggers
```

Claude will invoke `aavenir-contract-intelligence` automatically based on the SKILL.md trigger description.

### Updates

Claude Code disables auto-update for third-party marketplaces by default ([docs](https://code.claude.com/docs/en/discover-plugins)), so you will **not** pick up new releases automatically. To update on demand:

```
claude plugin marketplace update aavenir
claude plugin update aavenir-ai-skills@aavenir
/reload-plugins
```

To opt into auto-update for every session, set `export FORCE_AUTOUPDATE_PLUGINS=1` in your shell profile — or toggle it in the `/plugin` → **Marketplaces** tab.

---

## Claude Cowork (Claude Desktop, team workspaces)

Claude Cowork is Anthropic's collaborative workspace product (rolled out Feb 2026) that runs inside Claude Desktop. It uses the **same plugin marketplace system as Claude Code** — same `marketplace.json`, same plugin folder — just installed through a GUI instead of slash commands.

### Prerequisites

- [Claude Desktop](https://claude.com/download) installed (macOS or Windows).
- Claude Cowork access on your Anthropic plan. Cowork appears as a workspace inside Claude Desktop once enabled on your account.

### Install

1. **Open Claude Desktop** and switch to the Cowork workspace.
2. Open **Settings → Plugins** (or click the plugins icon in the Cowork sidebar). Cowork browses the same catalog as [claude.com/plugins](https://claude.com/plugins).
3. Click **Add marketplace** and paste:
   - `https://github.com/growthgpt/aavenir-ai-skills` (full HTTPS URL — recommended)
   - `growthgpt/aavenir-ai-skills` (GitHub `owner/repo` shortcut — only works if you have GitHub SSH keys configured locally; otherwise it fails with `Permission denied (publickey)`)
4. From the marketplace catalog, **install `aavenir-ai-skills`**. Choose **user scope** when prompted.
5. Reload the workspace (Cowork has a reload control next to the installed plugin list).

### Verify

In any Cowork conversation:

```
Review this MSA and flag any liability cap or auto-renewal risks: <paste contract>
```

`aavenir-contract-intelligence` auto-triggers — same SKILL.md `description` routing that drives Claude Code.

### Why scope matters in Cowork

Cowork conversations are scoped to a workspace. **User scope** means the plugin is available across every workspace you're a member of. **Project/workspace scope** restricts the plugin to one workspace — useful for Aavenir-specific team workspaces, but for personal review work pick user scope.

### Updates

Cowork uses Claude Code's plugin system under the hood, which disables auto-update for third-party marketplaces by default. You will **not** see new versions automatically. To update:

1. Open **Settings → Plugins** in Cowork.
2. Find `aavenir-ai-skills` and click the three-dot menu → **Check for updates** (or click the marketplace name to open the marketplace detail view and refresh from there).
3. When **Update** activates on the plugin card, click it.

If the Update button stays greyed even after a refresh, the local marketplace clone may be cached. Remove the marketplace entirely, quit Cowork (⌘Q), reopen, then re-add the marketplace and reinstall. That forces a clean fetch from GitHub.

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
