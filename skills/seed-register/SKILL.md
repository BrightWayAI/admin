---
disable-model-invocation: true
name: seed-register
description: Admin. Seed the billing register from your accounting system (QuickBooks first) and signed contracts. Auto-fires on "/seed-register", "import my customers from QuickBooks", "bootstrap my billing register", "load my contracts into the register". One-time or occasional re-run, never guesses a rate or term.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/seed-register.md` for the full workflow.

## When this skill fires

- User runs `/seed-register` directly
- User says: "import my customers from QuickBooks", "bootstrap my billing register", "load my contracts into the register"

## Pre-flight

Confirm `<config-root>/plugins/admin.user-context.md` has a billing register backend configured. If missing, route to `/setup-time` first.

## What this skill is NOT for

- Writing to the accounting system — read-only against it.
- Guessing rates, terms, or accounting names — unclear fields are marked `NEEDS CONFIRMATION`.
