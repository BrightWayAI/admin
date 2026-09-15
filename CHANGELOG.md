# Changelog

All notable changes to time-tracking are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [0.2.4] — OpenAI host adapter (2026-09-14)

### Added
- Native Codex/ChatGPT plugin manifest, durable `AGENTS.md` entrypoint, and an explicit OpenAI capability/degradation contract.
- GPT-discoverable skill aliases for canonical command workflows and read-only Codex role bindings where this plugin ships agents.
- Shared config-root resolution compatible with Cortex and Claude; all GPT tests use repository fixtures or temporary directories only.

## [0.2.3] — Platform-agnostic Step 0 (2026-05-12)

### Changed
- **Setup command Step 0 now platform-agnostic.** Every `request_cowork_directory(...)` call is conditional: "In Cowork, call `request_cowork_directory(...)`. In Claude Code (or any environment with direct filesystem access), no mount is needed." Same plugin source works in both runtimes.

### Why this matters
Phase 0 of SECOND-BRAIN-V2-SPEC. Removes the implicit Cowork-only assumption so Claude Code users do not hit unsupported tool calls during setup.

## [0.2.0] — Config-root refactor

### Changed
- **Plugin config and time-log moved to a user-chosen folder.** Reads/writes now go to `<config-root>/plugins/time-tracking.user-context.md` and `<config-root>/time-log.csv` via the pointer at `~/Documents/.claude-plugin-config-root`.
- **`/setup-time` Step 0 bootstraps the config root** and reads shared identity. Offers to migrate legacy `~/Documents/Claude/time-log.csv` during first-time bootstrap.
- **`/track-time` and `/generate-invoices` updated** to read/write the new paths.
- **User-facing prompts debranded** for fork-friendliness.

## [0.1.0] — Initial release

### Added
- Calendar-driven time tracking (`/track-time`) — pulls calendar events from a window (default yesterday or last week), classifies each as billable / non-billable per client (using attendee domain matching, title prefix, or project tag from setup), prompts for confirmation, appends to `~/Documents/Claude/time-log.csv`.
- Monthly invoice generation (`/generate-invoices`) — reads the time log, groups by client, applies each client's billing model (hourly / retainer / flat-fee project), drafts invoices using a user-editable template. Optionally hands off to `anthropic-skills:invoice` for final docx production. Marks rows as invoiced to prevent double-billing.
- `/setup-time` interview — captures clients with billing models, calendar tagging conventions, rounding rules (15 min / 30 min / actual), categories, invoice preferences (template, net terms, numbering, tax, delivery method, payment instructions). Optionally imports client list from `project-setup` if installed.
- Time log lives at `~/Documents/Claude/time-log.csv` as plain CSV — portable, hand-editable, backup-friendly.
- Schema documented in `references/time-log-schema.md` with privacy guidance.
- Companion plugin support: `project-setup` (engagement data), `claude-cortex` (memory of time-related observations), `core-ops` (pipeline-analyst for revenue-vs-time view).
