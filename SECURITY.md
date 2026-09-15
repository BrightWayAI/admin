# Security Policy

## What this plugin does with your data

Time Tracking pulls calendar events to classify billable time and produce monthly invoices. Read-only against the calendar; writes a local time-log file plus invoice drafts.

**Reads:**
- **Calendar** — events in the target window (yesterday / last week / specified range). Captures: date, start/end, duration, title, attendees, description.
- **Delivery settings** (if configured) — engagement data (client names, contract values, project tags) for billing context.
- **Time log** — `<config-root>/time-log.csv` by default for `/generate-invoices` and deduplication.
- **Plugin settings and templates** — `<config-root>/plugins/time-tracking.user-context.md`, optional user template override, and the immutable bundled starter.
- **Shared private profile** — `<config-root>/memory/me/identity.md` (read-only).

**Writes:**
- **Time log** — `<config-root>/time-log.csv` by default (append-only by `/track-time`; `/generate-invoices` flips the `invoiced` flag from `false` to `true` for billed rows). The user can manually edit the CSV.
- **Plugin settings** — `<config-root>/plugins/time-tracking.user-context.md` (after `/setup-time`).
- **Invoice drafts** — produced inline for review. Optionally handed to `anthropic-skills:invoice` or `docx` for final document creation; this plugin doesn't write the final docx.

**Does not:**
- **Modify calendar events.** Strictly read-only.
- **Send invoices.** Drafts only; user reviews and sends manually (or via their billing tool).
- **Modify CRM.** No CRM writes.
- **Track payment status.** Once an invoice is generated, the plugin doesn't follow up on payment.
- **Send time-log data anywhere.** The log lives at its configured local path.

## Where data lives

- Immutable plugin references inside the installed plugin directory.
- Time log as plain CSV at `<config-root>/time-log.csv` by default. **This file contains client names, hours, descriptions, and dollar amounts — back it up like other sensitive financial records.**
- Shared identity (read-only) at `<config-root>/memory/me/identity.md`.

## What gets sent off your machine

- Whatever your authorized calendar connector sends when `/track-time` reads events.
- Nothing else from this plugin.

## Privacy note about the time log

The time log is **local plain-text CSV**. It contains:
- Client names
- Project names
- Time durations
- Brief descriptions of work (your input, ≤120 chars)
- Billable status

It does **not** contain message content, contact emails, or external integration tokens.

If you fork this plugin or share your machine, treat the configured time-log CSV as confidential.

## Supported versions

| Version | Supported |
|---------|-----------|
| 0.1.x   | Yes       |

## Reporting a vulnerability

Report privately via GitHub Security Advisories:

https://github.com/BrightWayAI/time-tracking/security/advisories/new

Do not open a public issue for security concerns. We aim to respond within 5 business days.
