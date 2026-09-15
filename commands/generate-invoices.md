---
description: Generate monthly invoices from the time log. Reads <config-root>/time-log.csv for the target month, groups by client, calculates totals against each client's billing model, and produces invoice drafts. Hands off to the anthropic-skills:invoice or docx skill for final document generation, or outputs structured invoice data inline. Run on the 1st of the month for the prior month.
---

# /generate-invoices

Monthly invoice generation. Closes the calendar → time-log → money loop.

---

## Step 0 — Preflight

Read `<config-root>/plugins/time-tracking.user-context.md`. Extract:
- Active clients + billing models (hourly rate / retainer / flat fee)
- Invoice preferences (template, due-on-receipt vs net-30, delivery channel)
- Time-log file path (default `<config-root>/time-log.csv`)

If `<config-root>/plugins/delivery.user-context.md` exists, also read it for current contract values and engagement names.

---

## Step 1 — Determine the period

Default: previous month (e.g., if today is May 1, target = April).

User can override: `/generate-invoices 2026-04` for April 2026.

Announce: "Generating invoices for [Month YYYY]."

---

## Step 2 — Read the time log

Open `<config-root>/time-log.csv`. Filter to:
- Rows where `date` is within the target month
- Rows where `billable` = true
- Rows where `invoiced` = false (not already billed)

Group by `client`.

---

## Step 3 — Calculate per client

For each client:

### Hourly billing
- Sum `duration_min` for the month
- Convert to hours (round per user-context rounding rule)
- Multiply by hourly rate from user-context
- Itemize line items by project (or by week if many entries)

### Retainer
- Confirm retainer amount from user-context (flat monthly fee)
- Note hours used vs. retainer cap (if cap exists)
- If hours > cap: flag overage, calculate per overage rate (if defined)

### Flat fee project
- Reference Delivery's contract value
- If invoicing is milestone-based: identify which milestone closed this period and bill accordingly
- Otherwise: bill the agreed-upon monthly portion (e.g., contract / engagement-duration months)

---

## Step 4 — Draft invoices

For each client, prefer the user-owned
`<config-root>/plugins/time-tracking/templates/invoice-template.md`; otherwise use
the immutable `references/templates/invoice-template.md` default.

Structure:
- Invoice number (auto-increment from last invoiced number — track in user-context or derive from time-log)
- Date issued + due date (per net terms)
- Client info (name, billing email/address — from Delivery or time-tracking user-context)
- Your info (from `<config-root>/memory/me/identity.md`)
- Line items (per Step 3 calculation)
- Subtotal, taxes if applicable, total
- Payment terms + payment instructions (from user-context)
- Notes / scope reference (e.g., "Per SOW dated [date]")

---

## Step 5 — Hand off or output

The plugin doesn't produce final PDF/docx itself. Two options:

### Option A — Hand off to anthropic-skills:invoice (recommended if user has it)

If the user has `anthropic-skills:invoice` available, pass the structured invoice data and let that skill produce the final branded document. Mention to user:

> "Drafts ready. Want me to hand off to the `invoice` skill to generate branded .docx files? (Y/N)"

### Option B — Output structured data inline

If no invoice skill is available, output the structured invoice data as markdown for the user to review and copy into their billing tool of choice (QuickBooks, Wave, Stripe, manual emails).

---

## Step 6 — Confirm and update time-log

After invoices are drafted (and ideally sent), prompt:

> "Mark these time-log rows as invoiced? (Y/N) — sets `invoiced=true` so they're not double-billed next month."

If yes: update the corresponding rows in `<config-root>/time-log.csv`. This is the only mutation the plugin makes to existing rows.

---

## Step 7 — Output summary

```
Invoices generated for [Month YYYY]:

| Client      | Hours / Type | Amount     | Due       |
|-------------|--------------|------------|-----------|
| Acme Corp   | 24 hrs hourly| $X,XXX     | [date]    |
| Beta Inc    | Retainer     | $X,XXX     | [date]    |
| Gamma Co    | Milestone 2  | $X,XXX     | [date]    |
                                            
**Total: $XX,XXX**

Time-log: [N] rows marked invoiced.
[If anthropic-skills:invoice was used] Drafts in [Drive folder / local path]. Review before sending.
```

---

## Behavior rules

- **Never double-bill.** Always check `invoiced` flag before including a row.
- **Surface anomalies.** If a client has unusual hours (e.g., 80 hrs in a month vs. typical 30), flag for review before generating.
- **Honor billing model.** Don't generate hourly invoices for retainer clients, even if the time log shows hours.
- **Don't auto-send.** Always require user confirmation before invoices go out. The plugin produces drafts; the user reviews and sends.
- **Idempotent.** Running `/generate-invoices` twice for the same month should produce the same output (because rows already marked `invoiced=true` are skipped).

## Edge cases

- **Time-log file missing or empty** — say so, recommend running `/track-time` first.
- **Client in time-log but not in user-context** — flag, ask user to add the client to setup before invoicing.
- **Hours = 0 for an active retainer client** — still generate the retainer invoice; flag in notes that no hours were tracked (which may be fine for retainer or may indicate forgotten time).
- **Mid-month engagement starts/ends** — pro-rate flat fees if user-context says so; otherwise use full month.

## What this is NOT for

- **Sending invoices.** The plugin drafts; the user sends (manually or via their billing tool).
- **Payment tracking.** Once invoiced, the plugin doesn't track payment status. Use your accounting tool for that.
- **Tax calculation.** Surface taxable amounts per user-context, but don't compute tax tables for jurisdictions.
