---
description: Admin. Generate a billing packet for the accountant/bookkeeper — not client-facing invoices. For each active project, applies its billing-register line items and billing rules to the period (default current month), pulls hours from the time log for hourly lines, validates before drafting, and produces one section per invoice plus an hours-log attachment, ready for your accountant to key into the accounting system. Drafts the handoff email; sends only on explicit approval. The old client-facing invoice-document path (anthropic-skills:invoice / docx) is still available behind `--legacy-documents` or the `Legacy invoice documents` setting, but is no longer the default. Formerly `/generate-invoices` (kept as a deprecated alias).
---

# /invoices

Monthly (or ad hoc) billing packet generation. Turns the register + time log into
complete, correct inputs for your accountant — not final invoices sent to clients.

---

## Step G — Config-root guard (run first)

Run cortex's `python3 scripts/cortex_cli.py root-status`. If it exits non-zero, **stop** and show its sentence verbatim; write nothing anywhere else and do not ask for folder access more than once per session. If cortex is not installed, require that the resolved config root is an existing directory containing `memory/`; otherwise stop and point to `/start-nucleus` (in a cloud session: "Your Nucleus files are on your Mac and this chat can't see them yet. In the Claude desktop app, use + → Add folder and pick ~/cortex-root, then send any message.").

## Step 0 — Preflight

Read `<config-root>/plugins/admin.user-context.md`. Require:
- Billing register backend configured (if not, route to `/setup-time` then
  `/seed-register`)
- Accountant recipients + packet format under
  `## Billing packet / accountant handoff`

Read the billing register (Customers, Projects, Line Items, Cost Assignments,
Billing Packets log) from the configured backend.

If `--legacy-documents` is passed, or `## Billing packet / accountant handoff`'s
`Legacy invoice documents` is `on`, also run the old client-facing invoice-document
path from `references/templates/invoice-template.md` — see **Legacy path** at the
bottom. It runs in addition to, never instead of, the billing packet below unless
the register itself isn't configured, in which case it's the only thing this
command can do.

---

## Step 1 — Determine the period and validate

Default period: current month. `/invoices 2026-10` for a specific month.
`/invoices --adhoc <project>` for a single project, any period boundary the
project's own schedule implies.

Before drafting anything, validate the register + time log and surface issues at
the top of the output, not buried:

- **Missing or duplicate installments** — an `installment_x_of_n` line item
  whose `n` has already been billed, or whose prior installments are missing
- **Hours against fixed-fee projects** — time-log rows tagged to a project whose
  billing model isn't `hourly`/`per_unit`
- **Lines past the project end date** — a line item billing in a period after
  its project's `end_date`
- **Names that don't match the register** — a time-log `client`/`project` value
  with no matching register record, or an `accounting_product_service` that
  doesn't match the catalog exactly
- **Unconfirmed rates** — any line item or field still marked
  `NEEDS CONFIRMATION`

Each issue blocks only the affected line, not the whole packet — unaffected
projects still draft normally. List every blocked line with its reason.

---

## Step 2 — Determine what bills this period

For each active Project:

- **`monthly_during_term`** — bills if the period falls within
  `start_date`–`end_date`
- **`installment_x_of_n`** — bills if this period is the next unbilled
  installment (check the Billing Packets log for prior installments)
- **`actual_hours`** — bills if the time log has billable, unflagged hours
  against this project for the period
- **`on_milestone_acceptance`** — bills only if the user confirms the milestone
  was accepted this period (ask)

For `actual_hours` lines, pull time-log rows where `client`/`project` match,
`billable = true`, `invoiced = false`, and `date` falls in the period.

Projects with nothing billing this period are listed separately with the reason
(e.g. "no accepted milestone this period", "outside active dates", "no hours
logged").

---

## Step 3 — Draft the packet

One section per invoice (one per Project that bills this period):

```
CUSTOMER: [accounting_name]
PROJECT: [accounting_name]
TERMS: [terms]
SERVICE PERIOD: [period]

| Product/Service          | Description                | Qty  | Rate    | Amount  |
|---------------------------|-----------------------------|------|---------|---------|
| [accounting_product_service] | [invoice_description]  | [qty]| $[rate] | $[amt]  |

**Invoice total: $[X,XXX]**
```

After all sections:

```
**Period total: $[XX,XXX]**

Not billing this period:
- [Project] — [reason]
- [Project] — [reason]
```

For every hourly line item included, attach a detailed hours log: date, person,
hours, description, project — so the accountant can mark time billable and apply
it against the right invoice.

---

## Step 4 — Draft the email

Draft (never send) an email to the accountant recipients, with the packet in the
configured format (email text / .xlsx attachment / shared register view — default
email + .xlsx). Attach the hours log for hourly lines.

---

## Step 4.5 — Independent check (v0.6.0+)

Before asking for approval, have the cortex `verifier` agent check the packet against
the billing register and the time log. Step 1 validates the inputs; this step checks
that the packet says what the inputs say.

1. **List the claims.** One per invoice line: customer, project, product/service,
   quantity, rate, amount, and service period. Add one claim per invoice total, one for
   the period total, and one per "not billing this period" reason.
2. **Cite sources.** The billing register and `time-log.csv` are files; cite them by
   path with the record or row. If the register was read through a connector, save the
   records used to a temporary file outside `<config-root>` and cite that.
3. **Invoke `verifier`** with the claims and sources, and with the Billing Packets log
   as reference material so an installment already billed is caught. Do not pass your
   working or the Step 1 issue list.
4. **Arithmetic is not the verifier's job.** Recompute every amount (quantity × rate),
   every invoice total, and the period total yourself and show the sums. Hours on an
   hourly line must equal the sum of the attached hours-log rows.
5. **Show the result at the top of the Step 5 draft**, next to the validation issues:
   the counts, then every `fail` and `unverifiable` line with the evidence. A failed
   line is blocked the same way a Step 1 issue blocks a line: the rest of the packet
   can still go, and the blocked line is listed with its reason.
6. **Do not silently fix.** A correction is the user's call; after it, re-check the
   changed lines.
7. **If cortex is not installed or the verifier fails,** print "Not independently
   checked: <reason>" at the top of the draft. Never state or imply it was checked.

Delete any temporary files when the step is done. A clean check does not approve the
send: Step 5 still requires explicit approval.

---

## Step 5 — Confirm and finalize

Show the full draft (validation issues, the Step 4.5 check result, packet, not-billing list, email). Ask for
explicit approval to send. On approval:

- Send the email.
- Write a Billing Packets record: period, projects included, total, date sent,
  status `sent`, notes.
- Mark the corresponding `time-log.csv` rows `invoiced = true` — **only after**
  the user confirms the packet was sent. This is the only place `/invoices`
  mutates existing rows.

If the user doesn't approve sending, write the Billing Packets record with status
`drafted` and leave time-log rows unmarked — safe to re-run.

---

## Behavior rules

- **Never double-bill.** `invoiced = true` rows and already-billed installments
  are always excluded.
- **Validate before drafting, not after.** Issues surface at the top, before the
  packet body.
- **Every line traces to a register record.** No line item is invented from the
  time log alone — if a time-log project has no register entry, it's a validation
  issue, not a silent invoice line.
- **Don't auto-send.** Drafts only; explicit approval required.
- **Idempotent.** Re-running for an already-sent period reproduces the same
  "nothing new to bill" result for already-billed lines.

## Out of scope

- Transaction coding, payment tracking, sending invoices to clients — those stay
  with the accountant and the accounting system.

## Legacy path — client-facing invoice documents

When enabled (`--legacy-documents` or the user-context setting), also run the
pre-register workflow: group time-log rows by client, apply each client's billing
model from `## Clients and billing`, and hand off to `anthropic-skills:invoice`
(or output structured markdown inline) using
`references/templates/invoice-template.md`. This produces client-facing invoice
documents, not an accountant packet, and does not touch the billing register. Off
by default — most users want the packet above, not self-generated invoices.
