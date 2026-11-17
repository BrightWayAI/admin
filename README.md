# Admin

Calendar-driven time tracking and month-end billing packets for solo consultants and small agencies.

The pain: you do the work, your calendar reflects it, and then you either spend an hour at month-end manually reconstructing billable time, or you forget and undercount. This plugin closes the loop. Pull calendar daily/weekly, classify by client, log to a simple file, and hand your accountant a complete, checked billing packet at month-end.

## What it does

1. **`/track-time`** (daily or weekly) — pulls calendar events, classifies each as billable / non-billable / unknown by client, prompts for confirmation, and logs to `<config-root>/time-log.csv` by default.
2. **`/invoices`** (monthly) — builds a billing packet for your accountant or bookkeeper, not client-facing invoices. For each active project it applies the billing register's line items and rules to the period, pulls hours from the time log for hourly lines, and produces one section per invoice plus an hours-log attachment. It drafts the handoff email and sends only when you approve. The packet is checked against the billing register and time log by the cortex `verifier` agent before you approve it; a failed line is blocked and shown with the evidence. Client-facing invoice documents are still available with `--legacy-documents`. (`/generate-invoices` is a deprecated alias.)
3. **`/seed-register`** (once, or to refresh) — fills the billing register from your accounting system (customers, projects, product and service catalog) and your signed contracts.
4. **`/setup-notice`** (per new client or project) — drafts billing-register records from a signed contract, produces a setup sheet for your accountant, and drafts the handoff email. Never sends. Clients's `/project-setup` runs the same logic at kickoff.

## Install

Recommended: via the [BrightWayAI marketplace](https://github.com/BrightWayAI/nucleus).

```
/plugin marketplace add BrightWayAI/nucleus
/plugin install admin@nucleus
```

## First-time setup

Run `/setup-time`. The interview captures:

- **Clients and billing models** — for each active client: hourly rate / retainer / project flat fee
- **Categories** — billable client work, internal time, business development, learning, etc.
- **Calendar tagging conventions** — how you mark events as billable in your calendar (attendee domain matching / event title prefix / project tag)
- **Default block rounding** — round to 15 min, 30 min, or actual
- **Invoice preferences** — template, due-on-receipt vs net-30, where to deliver (email / portal / Drive folder)

Saved to `<config-root>/plugins/admin.user-context.md`.

## Companion plugins

- **the clients plugin** — provides engagement data (client name, contract value, deliverables, start/end dates). When configured, Admin imports billing context rather than re-asking.
- **`cortex`** — captures time-related observations ("Acme calls always run 15 min over") into memory.
- **`growth`** — pipeline-analyst can surface deal value alongside billable time for a "where am I making money vs. spending time" view; if not installed, note "Growth not installed; pipeline analysis skipped."
- **Anthropic skills (`invoice`, `docx`)** — used only by `/invoices --legacy-documents` for client-facing invoice documents.

Works without companions, but the integrations make billing more accurate.

## What's inside

```
.claude-plugin/plugin.json
commands/
  track-time.md             Daily/weekly calendar → time-log
  invoices.md               Monthly register + time-log → billing packet
  seed-register.md          Fill the billing register from accounting + contracts
  setup-notice.md           New client/project setup notice for the accountant
  generate-invoices.md      Deprecated alias → /invoices
  setup-time.md             Interview
skills/
  <one per command>/SKILL.md  Fires the matching command from plain language
  setup/SKILL.md            Auto-fires on setup phrases
references/
  user-context.template.md  Structure (committed)
  register-schema.md        The billing register format
  setup-notice-generation.md  How setup notices are built
  time-log-schema.md        Documentation of the time-log file format
  templates/
    invoice-template.md     Immutable starter invoice template
```

## The time-log file

Lives at `<config-root>/time-log.csv` by default (configurable in setup). Plain CSV for portability:

```csv
date,start,end,duration_min,client,project,category,billable,description,invoiced
2026-05-08,09:00,10:30,90,Acme Corp,AI Op Model,client-work,true,"Stakeholder interview with COO",false
2026-05-08,10:30,11:00,30,Acme Corp,AI Op Model,client-work,true,"Synthesis notes from interview",false
2026-05-08,11:00,11:30,30,Internal,BD,bizdev,false,"LinkedIn outreach batch",false
```

Hand-editable. Backup-friendly. Works offline. The plugin reads + appends; never destroys.

<!-- OPENAI-SUPPORT:START -->
## ChatGPT and Codex

Admin ships as a native OpenAI plugin as well as a Claude plugin. In
ChatGPT desktop Local Work, enable **Admin** and ask naturally or mention
`@Admin`. In Codex, use natural language or the namespaced skills exposed
by the plugin. Claude slash-command names in this README remain workflow aliases.

All hosts resolve the same `<config-root>` used by Cortex, so Claude, ChatGPT desktop,
and Codex can share identity, voice, memory, and per-plugin settings without copying
them. The installed plugin directory is read-only at runtime. See
[`references/openai-portability.md`](references/openai-portability.md) for capability
mapping, connector checks, permissions, and honest degraded behavior.

Import the full catalog from
[`BrightWayAI/nucleus`](https://github.com/BrightWayAI/nucleus); Nucleus is the master
marketplace, while each plugin remains independently installable.
<!-- OPENAI-SUPPORT:END -->


## License

MIT.
