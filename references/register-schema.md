# Billing register schema

The billing register is the structured source of truth `/invoices` reads to build a
billing packet, and the thing `/seed-register` and `/setup-notice` populate. It is
**not** the time log — `time-log.csv` records hours actually worked; the register
records the billable catalog, contract terms, and who to bill.

The schema lives in the plugin. The data never does. Every field below is captured
by `/setup-time`, seeded by `/seed-register`, or drafted by `/setup-notice`, and
written only to the user's own `<config-root>`.

## Backend

Two supported backends, chosen in `/setup-time` and recorded in
`<config-root>/plugins/admin.user-context.md` under `## Billing register`:

- **Local files (default)** — one CSV per entity under
  `<config-root>/plugins/admin-register/`:
  `customers.csv`, `projects.csv`, `line-items.csv`, `cost-assignments.csv`,
  `billing-packets.csv`. Plain, portable, hand-editable — same philosophy as
  `time-log.csv`.
- **Airtable** — via the Airtable connector, if configured. The five entities map
  to five tables in a base the user names during setup. `/setup-time` records the
  base identifier in user-context; it is never written into plugin code, templates,
  or docs. If the connector isn't available when an Airtable backend is chosen,
  fall back to local files for that run and tell the user.

Every command that reads or writes the register resolves the backend from
user-context first — never assume local files.

## Entities

### Customers

| Field | Type | Required | Notes |
|---|---|---|---|
| `customer_id` | string | yes | Stable key, e.g. a slug |
| `accounting_name` | string | yes | Exact name as it exists in the accounting system — must match character-for-character, since the accountant matches on this |
| `billing_contact_name` | string | yes | |
| `billing_contact_email` | string | yes | |
| `billing_address` | string | yes | |
| `terms` | string | yes | e.g. "net 30" |
| `status` | string | yes | active / inactive |

### Projects

| Field | Type | Required | Notes |
|---|---|---|---|
| `project_id` | string | yes | Stable key |
| `accounting_name` | string | yes | Exact project / sub-customer name as it exists in the accounting system |
| `customer_id` | string | yes | FK → Customers |
| `contract_link` | string | no | Link or path to the executed contract/SOW, finance-zone location only |
| `po_number` | string | no | |
| `contract_value` | number | yes (unless hourly/per-unit with no cap) | |
| `billing_model` | enum | yes | `retainer` / `fixed_fee_installments` / `milestone` / `hourly` / `per_unit` |
| `start_date` | date | yes | |
| `end_date` | date | no | Open-ended if blank |
| `billing_schedule_summary` | string | yes | Plain-language summary, e.g. "3 equal installments, 1st/3rd/5th month" |
| `status` | string | yes | active / on-hold / closed |

### Line items (billable catalog)

| Field | Type | Required | Notes |
|---|---|---|---|
| `line_item_id` | string | yes | Stable key |
| `project_id` | string | yes | FK → Projects |
| `invoice_description` | string | yes | Exact wording that should appear on the invoice |
| `accounting_product_service` | string | yes | Must match an existing product/service in the accounting system exactly — never invented |
| `unit` | string | yes | e.g. "hour", "installment", "month", "unit" |
| `rate` | number | yes, unless marked `NEEDS CONFIRMATION` | |
| `billing_rule` | enum | yes | `monthly_during_term` / `installment_x_of_n` / `actual_hours` / `on_milestone_acceptance` |
| `installment_n` / `installment_of` | int | required if `installment_x_of_n` | |
| `active` | boolean | yes | |

### Cost assignments

| Field | Type | Required | Notes |
|---|---|---|---|
| `project_id` | string | yes | FK → Projects |
| `contractor_or_vendor` | string | yes | |
| `cost_type` | string | yes | e.g. "subcontractor", "software", "reimbursable expense" |
| `notes` | string | no | So the accountant charges the cost to the right project |

### Billing packets (log)

| Field | Type | Required | Notes |
|---|---|---|---|
| `packet_id` | string | yes | Stable key, e.g. period + sequence |
| `period` | string | yes | e.g. `2026-10` or an ad hoc label |
| `projects` | string | yes | Project ids included |
| `total` | number | yes | |
| `date_sent` | date | no | Blank until the user confirms send |
| `status` | enum | yes | `drafted` / `sent` / `partially_sent` |
| `notes` | string | no | |

## Traceability rule

Every line on a billing packet must trace back to a Line Item record, every Line
Item to a Project, every Project to a Customer. `/invoices` validates this before
drafting (see `commands/invoices.md` Step 1).

## `NEEDS CONFIRMATION`

`/seed-register` and `/setup-notice` never guess a rate, term, or accounting name.
Any field they can't source with confidence from the accounting system or the
signed contract is written as the literal string `NEEDS CONFIRMATION` and listed
separately for the user to resolve before the record is treated as active.
`/invoices` Step 1 validation blocks any line still carrying that value.
