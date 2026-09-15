# Admin

Calendar-driven time tracking and invoice generation for solo consultants and small agencies.

The pain: you do the work, your calendar reflects it, and then you either spend an hour at month-end manually reconstructing billable time, or you forget and undercount. This plugin closes the loop. Pull calendar daily/weekly, classify by client, log to a simple file, generate invoices at month-end.

## What it does

1. **`/track-time`** (daily or weekly) — pulls calendar events, classifies each as billable / non-billable / unknown by client, prompts for confirmation, and logs to `<config-root>/time-log.csv` by default.
2. **`/invoices`** (monthly) — reads the time log, groups by client, calculates totals against your billing model, drafts invoices. Hands off to your existing invoice tool (e.g., `anthropic-skills:invoice`) or generates docx/pdf directly. (`/generate-invoices` is a deprecated alias that delegates here.)

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

- **Client Success** — provides engagement data (client name, contract value, deliverables, start/end dates). When configured, Admin imports billing context rather than re-asking.
- **`cortex`** — captures time-related observations ("Acme calls always run 15 min over") into memory.
- **`growth`** — pipeline-analyst can surface deal value alongside billable time for a "where am I making money vs. spending time" view; if not installed, note "Growth Engine not installed; pipeline analysis skipped."
- **Anthropic skills (`invoice`, `docx`)** — `/invoices` can hand off to these for final invoice document production.

Works without companions, but the integrations make billing more accurate.

## What's inside

```
.claude-plugin/plugin.json
commands/
  track-time.md             Daily/weekly calendar → time-log
  invoices.md               Monthly time-log → invoice drafts
  generate-invoices.md      Deprecated alias → /invoices
  setup-time.md             Interview
skills/
  track-time/SKILL.md       Auto-fires on time-tracking phrases
  invoices/SKILL.md         Auto-fires on invoice phrases
  setup/SKILL.md            Auto-fires on setup phrases
references/
  user-context.template.md  Structure (committed)
  user-context.md           Your config (gitignored, created by setup)
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
