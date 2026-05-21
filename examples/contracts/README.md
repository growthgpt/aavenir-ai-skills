# Sample contracts

Paste-ready contract drafts to exercise the Aavenir AI skills end-to-end. Each sample is intentionally engineered to surface specific behaviors from one or more skills.

| File | Type | What it demonstrates | Best skill to try |
|------|------|----------------------|-------------------|
| [`01-msa-risky.md`](01-msa-risky.md) | Master Services Agreement | Uncapped indemnity, no liability cap, vague termination, weak data terms — heavy risk surface | `aavenir-contract-intelligence` → `aavenir-negotiation-copilot` |
| [`02-saas-clean.md`](02-saas-clean.md) | SaaS Subscription Agreement | Mostly market-standard, 2 medium risks, rich post-signature obligations (SLA, renewal notice, audit, breach notification) | `aavenir-obligation-monitor` |
| [`03-mutual-nda.md`](03-mutual-nda.md) | Mutual NDA | Fast clean-pass review with one minor gap | `aavenir-contract-intelligence` |

## How to use

After installing the plugin (`/plugin install aavenir-ai-skills@aavenir`), open any Claude Code session and paste the contract body with a prompt like:

```
Review this contract and flag risks.
[paste contract text]
```

The matching skill auto-triggers and returns structured JSON conforming to the skill's `schema.json`. The same paste works in Claude Desktop, Cursor, Codex CLI, Cline, and any MCP-compatible client that has the skill loaded.

### Try the negotiation copilot

After running contract-intelligence on `01-msa-risky.md`, follow up with:

```
Draft redlines for the uncapped indemnity and add a liability cap.
```

### Try the obligation monitor

After running contract-intelligence on `02-saas-clean.md`, follow up with:

```
Extract every post-signature obligation with deadlines and alert windows.
```

## Disclaimer

These samples are synthetic, illustrative drafts written for skill demonstration only. They are not legal advice and should not be used as templates for executed agreements without review by qualified counsel.
