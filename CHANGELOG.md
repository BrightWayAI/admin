# Changelog

All notable changes to time-tracking are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/). Versions match `plugin.json`.

## [0.5.0] — Billing register and accountant handoff (2026-10-15)

### Added
- **Billing register** — new generic schema (`references/register-schema.md`) for Customers, Projects, Line Items (billable catalog), Cost Assignments, and a Billing Packets log. Backend is user-chosen: local CSV/markdown files under `<config-root>/plugins/admin-register/` (default) or Airtable via connector. Schema lives in the plugin; data never does.
- `/seed-register` (new command + skill) — one-time or occasional re-run that pulls customers, projects/sub-customers, and the product/service catalog from a configured accounting connector (QuickBooks first, structured to add others later), reads signed contracts from a configured location, and drafts register records for user confirmation. Never guesses a rate, term, or accounting name — unclear fields are marked `NEEDS CONFIRMATION`.
- `/setup-notice` (new command + skill) — manual trigger for the new-customer/project setup-notice workflow, built on new shared logic at `references/setup-notice-generation.md`. Also invoked automatically by `clients:project-setup` when a contract executes (clients 0.8.1+).
- `/setup-time` Step 7.5 — captures register backend, accounting connector, contract location, accountant recipients, packet format, prep/send days, and a legacy-invoice-documents toggle. Offers `ops:register-schedules` registration using the captured prep/send days.

### Changed
- **`/invoices` reworked into a billing-packet generator.** Default output is now an accountant handoff packet built from the billing register — one section per invoice (customer/project exact accounting-system names, terms, period, line items, totals), a not-billing-this-period list with reasons, and an hours-log attachment for hourly lines. A validation pass runs first and surfaces missing/duplicate installments, hours against fixed-fee projects, lines past a project's end date, register/time-log name mismatches, and unconfirmed rates at the top of the draft, before the packet body. Still draft-only; still requires explicit approval before sending and before marking time-log rows `invoiced`.
- The prior client-facing invoice-document path (grouping by client's billing model, handing off to `anthropic-skills:invoice` / the local invoice template) is preserved behind `--legacy-documents` or the `Legacy invoice documents` user-context setting — no longer the default.
- `references/user-context.template.md` gains `## Billing register` and `## Billing packet / accountant handoff` sections.

### Scope
- Out of scope, per design: transaction coding, payment tracking, sending invoices to clients.

## [0.4.1] — OpenAI host adapter (2026-09-14)

### Added
- Native Codex/ChatGPT plugin manifest, durable `AGENTS.md` entrypoint, and an explicit OpenAI capability/degradation contract.
- GPT-discoverable skill aliases for canonical command workflows and read-only Codex role bindings where this plugin ships agents.
- Shared config-root resolution compatible with Cortex and Claude; all GPT tests use repository fixtures or temporary directories only.

## [0.4.0] — pipeline-analyst ownership moved to growth (2026-09-15)

### Changed
- `pipeline-analyst` now lives in the `growth` plugin (moved from `ops`). README updated to reference `growth` and report "Growth Engine not installed; pipeline analysis skipped" when growth isn't present.

## [0.3.0] — Renamed to Admin; /invoices primary command (2026-09-15)

Renamed from `time-tracking` to `admin` (display name: Admin) as part of the
2026-09-15 Nucleus plugin rename. Old plugin ID/repo name redirects; see
marketplace catalog.

### Changed
- `/generate-invoices` renamed to `/invoices` as the primary command. `/generate-invoices` remains as a deprecated thin alias that delegates to `/invoices`.

## [0.2.8] — Delivery integration cleanup (2026-09-15)

### Changed
- Replaced retired project-setup dependencies and direct Cowork identity paths with canonical Delivery and Cortex paths.
- Moved invoice template customizations to user-owned config-root state.

## [0.2.7] — Codex adapter synchronization (2026-09-15)

### Fixed
- Synchronized the Codex manifest with the current plugin version.

## [0.2.6] — Skill auto-invocation audit (2026-09-15)

Nucleus Operating Model Refactor Phase 3 step 3.7. Ritual and side-effecting
skills marked `disable-model-invocation: true` so they only run on explicit
invocation, not loose natural-language matching — the model can still be
asked to run them by name. Read-mostly, low-stakes, or high-frequency
conversational skills are left auto-invocable. Marketplace-wide this brings
model-invocable skills from ~81 to 27, under the ≤30 target audited with
`/skill-doctor`.

### Changed
- Marked `disable-model-invocation: true` on: `generate-invoices`, `setup`, `setup-time`.

## [0.2.5] — Identity/voice moved to memory/me/ (2026-09-15)

### Changed
- Path references updated from `<config-root>/identity.md` / `<config-root>/voice.md` to `<config-root>/memory/me/identity.md` / `<config-root>/memory/me/voice.md`, per the Nucleus Operating Model Refactor Phase 2 scopes restructure (identity/voice are personal, not org-shared facts). No behavior change beyond the path.

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
