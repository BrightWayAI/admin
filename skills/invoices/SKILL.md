---
disable-model-invocation: true
name: invoices
description: Admin. Generate the monthly (or ad hoc) billing packet for your accountant — applies the billing register's line items and billing rules to the period, pulls hours for hourly lines, validates before drafting, and drafts the accountant handoff email. Not client-facing invoices (that path still exists behind a legacy flag). Auto-fires on "/invoices", "generate the billing packet", "bill clients", "monthly invoicing", "send the accountant the packet", "create my invoices", or any phrase about producing billing inputs from logged time. Run on the configured prep day. Formerly "/generate-invoices" (kept as a deprecated alias skill).
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/invoices.md` for the full workflow.

## When this skill fires

- User runs `/invoices` directly (or the deprecated `/generate-invoices` alias)
- User says: "generate invoices", "bill clients", "monthly invoicing", "create my invoices", "send out invoices for [month]"
- A scheduled task triggers this (common pattern: 1st of month at 9am)

## Pre-flight

Confirm `<config-root>/plugins/admin.user-context.md` has a billing register backend configured. If missing, route to `/setup-time` then `/seed-register` first. (Legacy-documents mode only needs at least one client + billing model in `## Clients and billing`.)

Also confirm `<config-root>/time-log.csv` exists with rows for the target month. If empty, recommend `/track-time` first.

## What this skill is NOT for

- Sending invoices. The plugin drafts; the user sends.
- Payment tracking. Once invoiced, payment status is tracked elsewhere.
- Tax calculation. Surfaces taxable amounts; doesn't compute jurisdictional tax.
