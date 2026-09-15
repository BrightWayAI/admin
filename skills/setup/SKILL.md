---
disable-model-invocation: true
name: setup
description: Configure time-tracking for your billing models, calendar tagging conventions, and invoice preferences. Auto-fires on "/setup-time", "set up time tracking", "configure my billing rates", "add a client to time tracking", "update my billing setup", or when /track-time or /generate-invoices reports user-context.md is missing.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


See `commands/setup-time.md` for the full interview.

## When this skill fires

- User runs `/setup-time` directly
- User says: "set up time tracking", "configure my billing", "add a client", "update my billing setup"
- The `/track-time` or `/generate-invoices` command reports user-context.md is missing → auto-route here

## Quick path

If the user wants minimum-viable defaults: capture one client with a single billing model and use defaults for everything else. Recommend running the full interview later when they have more clients to add.
