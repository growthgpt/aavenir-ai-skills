# Using Aavenir Skills with Cursor

Cursor supports custom AI behavior through `.cursorrules` (project-level) and custom modes (workspace-level). The Aavenir skills install as Cursor custom modes for one-click activation.

## Quick install (project-level)

Drop the Aavenir routing prompt into your project's `.cursorrules`:

```bash
cat skills/avy/prompt.md > .cursorrules
```

Now Cursor will route any contract/vendor/procurement question through AVY automatically.

## Custom modes (recommended)

Cursor's **Custom Modes** allow per-domain personalities. Create one per Aavenir skill.

### Setup

1. Open Cursor → `Cmd/Ctrl + Shift + P` → `Cursor: New Custom Mode`
2. For each skill, create a mode:

| Mode name | Prompt source | When to switch to it |
|-----------|---------------|----------------------|
| `Aavenir: Contract Review` | `skills/contract-intelligence/prompt.md` | Reviewing or analyzing a contract |
| `Aavenir: Negotiation` | `skills/negotiation-copilot/prompt.md` | Rewriting or redlining clauses |
| `Aavenir: Sourcing` | `skills/procurement-intelligence/prompt.md` | Comparing vendors or scoring RFPs |
| `Aavenir: Obligations` | `skills/obligation-monitor/prompt.md` | Post-signature tracking |
| `Aavenir: AVY` | `skills/avy/prompt.md` | Default — auto-routes to the right specialist |

3. Set `Aavenir: AVY` as your default mode for contract/procurement workspaces.

## Workspace-level .cursorrules

For repos where Aavenir is the dominant domain (e.g., the Aavenir Contractflow codebase, or a legal-ops workspace), use this `.cursorrules`:

```markdown
# Aavenir AI Skill Routing

When the user asks about contracts, vendors, RFPs, suppliers, or obligations:

1. Load the matching skill prompt from `./aavenir-ai-skills/skills/<skill>/prompt.md`
2. Use the structured JSON output shape defined in the matching `schema.json`
3. For ambiguous queries, default to `aavenir-avy` (universal router)

## Routing rules

- Contract review, risk, extraction → `aavenir-contract-intelligence`
- Clause rewriting, redlining, simplification → `aavenir-negotiation-copilot`
- Vendor comparison, RFx scoring, TCO → `aavenir-procurement-intelligence`
- Post-signature obligations, renewals → `aavenir-obligation-monitor`
- Cross-cutting or unclear → `aavenir-avy`

## Output

Always return JSON conforming to the skill's `schema.json`. Use the few-shot examples in `examples.json` as calibration.
```

## Chat usage

In any Cursor chat:

```
@aavenir-ai-skills/skills/contract-intelligence/prompt.md

Review this MSA and flag risks: <paste contract>
```

Cursor's `@` file referencing pulls the system prompt into context.

## Inline editing

Highlight a clause in any document, then `Cmd/Ctrl + K`:

```
Use aavenir-negotiation-copilot to rewrite this clause from the customer side. Give me three positions.
```

Cursor loads the negotiation prompt and produces calibrated alternatives.

## Limitations

- Cursor does not enforce JSON schema natively. Configure your downstream consumer to validate against `schema.json`.
- Mode-switching is manual unless you use `aavenir-avy` as the default — AVY auto-routes inside a single conversation.
