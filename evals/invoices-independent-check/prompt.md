---
max_turns: 10
allowed_tools: [Read, Glob, Grep, Skill]
---

Dry run: load the `/invoices` instructions with the Skill tool (loading them runs
nothing), then walk Step 4.5 against this exact scenario without executing any step.

The packet for this month has two invoices. Invoice A (Acme, hourly): 12 hours at $200,
amount $2,400. Invoice B (Birch, fixed fee): "Installment 2 of 3", $5,000. The cortex
verifier returns: Invoice A's rate passes, Invoice A's quantity fails (the time log has
10.5 billable hours for the period, not 12), and Invoice B fails because the Billing
Packets log shows installment 2 of 3 was already sent last month.

Tell me what you checked yourself, what you show me, and what can still be sent.
