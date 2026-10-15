---
disable-model-invocation: true
name: setup-notice
description: Admin. Manually draft a new-customer/project setup notice for the accountant — billing-register records plus a setup sheet and draft handoff email. Auto-fires on "/setup-notice", "set up this project for billing", "draft the accountant setup email", "add this customer to the billing register". Normally runs automatically from clients:project-setup when a contract executes; use this for manual/corrective runs.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/setup-notice.md` for the full workflow.

## When this skill fires

- User runs `/setup-notice` directly
- User says: "set up this project for billing", "draft the accountant setup email", "add this customer to the billing register"

## Pre-flight

Confirm `<config-root>/plugins/admin.user-context.md` has a billing register backend configured. If missing, route to `/setup-time` first.

## What this skill is NOT for

- Sending the email. Drafts only; the user sends.
- Writing to an accounting system directly.
- Guessing rates or terms — unclear fields are marked `NEEDS CONFIRMATION`.
