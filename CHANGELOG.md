# Changelog

All notable changes to the Aavenir AI Skill Ecosystem are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project follows [Semantic Versioning](https://semver.org/).

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
