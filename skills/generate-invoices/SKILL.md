---
disable-model-invocation: true
name: generate-invoices
description: Deprecated — renamed to /invoices. Auto-fires only on the literal "/generate-invoices" invocation; delegates to the invoices skill.
---

<!-- OPENAI-ADAPTER:START -->
## OpenAI host binding

Before acting, read `../../references/openai-portability.md`. That file translates
host-specific tools, agents, artifacts, scheduling, connectors, and config-root
access for ChatGPT and Codex. It overrides concrete Claude/Cowork tool names only;
the workflow, safety gates, and output contract in this skill remain canonical.
<!-- OPENAI-ADAPTER:END -->


# generate-invoices (deprecated alias)

Renamed to `/invoices`. This skill exists only so the old command name keeps
working. Print a one-line deprecation notice ("`/generate-invoices` is now
`/invoices` — this alias will be removed in a future version.") and then follow
`../../commands/invoices.md` (via `commands/generate-invoices.md`, which delegates
there) as the canonical workflow. Do not duplicate the workflow here.
