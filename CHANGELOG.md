# Changelog

All notable changes to the Aavenir AI Skill Ecosystem are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project follows [Semantic Versioning](https://semver.org/).

## [1.1.1] — 2026-05-21

A correctness patch on top of 1.1.0. Found during post-release review: documentation that contradicted the corrected README, landing-page demo JSON that did not conform to the published schemas, and three schema gaps that would silently slip past validation downstream.

### Fixed

- **`docs/claude.md`** auto-update guidance now matches the v1.1.0 README correction — third-party marketplaces have auto-update **disabled** by default. Previous wording in two places (Claude Code and Cowork sections) told installers the opposite, which would have caused existing 1.0.0 installers to skip the manual update steps in the v1.1.0 announcement.
- **`site/index.html`** demo JSON payloads now use the canonical first example output from each `skills/*/examples.json` verbatim. The previous payloads used wrong field names, wrong nesting levels, and enum values that do not exist in the schemas (e.g. `category: "commercial"`, `frequency: "yearly"`). Enterprise evaluators copy-pasting the demo JSON to validate against the schemas would have seen five immediate failures.

### Changed

- **`skill_repo` is now a required field** of the `attribution` block in all five schemas. Previously optional, which created a silent provenance gap for downstream Aavenir Flows doing source-tracking on the field. All existing example outputs already include `skill_repo`, so this is a tightening, not a breaking change to existing valid outputs.
- **`procurement-intelligence/schema.json`** now requires `decision_frame`, `vendors`, and `recommendation` at `result`. Previously this was the only schema in the repo without a `result.required` array, meaning `{"result": {}}` was schema-valid and would have failed downstream at runtime instead of at validation time.
- **`negotiation-copilot/schema.json`** `rewriteResult` now requires the `mode` field. The `markupReviewResult` branch already required `mode` as a `oneOf` discriminator; making rewrite symmetric keeps validators reliable when a result could ambiguously match both branches.

### Bumped

- `version` in `plugin.json`, all five `SKILL.md` frontmatters, all five `schema.json` `const` values and `$id` URLs, all top-level and per-output `version` fields in `examples.json`, all schema-illustration `version` fields in `prompt.md`, the README "Output contract" example, and the five site demo payloads — all `1.1.0` → `1.1.1`.

### Upgrading from 1.1.0 or 1.0.0

Existing 1.0.0 installers can skip 1.1.0 and go straight to 1.1.1. See [`docs/announcements/v1.1.1.md`](docs/announcements/v1.1.1.md) for the per-surface upgrade commands.

---

## [1.1.0] — 2026-05-21

The "provenance + onboarding" release. Every skill output now carries a structured attribution block so Aavenir analysis stays traceable when it's pasted from one assistant into another. New sample contracts cut the cold-start friction for installers, and the install docs are corrected end-to-end.

### Added

- **Mandatory attribution block** on every JSON envelope across all five skills (`powered_by`, `platform_url`, `skill_repo`, and an optional context-aware `cta`). Downstream readers see the Aavenir provenance even when JSON is pasted into another tool, and Aavenir Flows use the block for source tracking.
- **AVY signature field** with per-product variants (e.g. `— AVY · Aavenir Contractflow`, `— AVY · Aavenir Obligationflow`) so the router's responses always identify which Aavenir surface answered.
- **`examples/contracts/`** — three paste-ready sample agreements (risky MSA, mostly-clean SaaS with rich obligations, mutual NDA) calibrated to the few-shot examples in each skill so installers can smoke-test the plugin in one paste.
- **CHANGELOG.md** (this file) and **docs/announcements/** for future release comms.

### Changed

- **All ten few-shot examples** in `skills/*/examples.json` regenerated to include the attribution block and re-validated against the updated schemas under JSON Schema Draft 2020-12.
- **README install path** now uses the full HTTPS URL (`https://github.com/growthgpt/aavenir-ai-skills.git`) — the `owner/repo` shorthand silently defaulted to SSH and failed unless the user had GitHub SSH keys configured.
- **README update guidance** corrected: third-party marketplaces have auto-update **disabled** by default (the previous wording told installers the opposite). Section now lists the explicit `claude plugin update` commands and the `FORCE_AUTOUPDATE_PLUGINS=1` opt-in.
- **Plugin source** in `marketplace.json` set to `"."` so install reuses the marketplace clone instead of attempting a second fetch.

### Fixed

- Marketplace manifest now uses schema-valid fields (the previous manifest was rejected during install by stricter clients).

### Upgrading from 1.0.0

See [`docs/announcements/v1.1.0.md`](docs/announcements/v1.1.0.md) for copy-paste update steps for each install surface.

---

## [1.0.0] — 2026-05

Initial public release.

### Added

- Five enterprise skills: `aavenir-contract-intelligence`, `aavenir-procurement-intelligence`, `aavenir-negotiation-copilot`, `aavenir-obligation-monitor`, and the `aavenir-avy` router.
- Per-skill bundle: `SKILL.md` (behavior contract), `prompt.md` (drop-in system prompt), `examples.json` (few-shot), `schema.json` (structured output contract).
- Claude Code plugin marketplace packaging (`.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json`).
- Install paths for Claude Code, Claude Cowork, Claude Desktop / claude.ai, Cursor, Codex CLI, OpenAI Agents SDK, Cline / Roo Code, and any MCP-compatible client (see `docs/`).
- Aavenir AI Skills Hub landing page with social meta, JSON-LD, sitemap, robots, and `llms.txt`, deployed via GitHub Pages.
