# New customer/project setup-notice generation

Shared procedure for drafting billing-register records and the accountant
setup-sheet email for a new or newly-signed engagement. Invoked by:

- `admin:setup-notice`, run manually against a chosen project.
- `clients:project-setup` Step 6.4/6.5, automatically when a contract executes
  and `admin` is installed.

Both callers follow the same steps below; only the trigger and the surrounding
confirmation flow differ.

## Inputs

- The executed contract/SOW (finance-zone fields only — billing model, rate(s),
  contract value, cap/NTE, invoicing cadence, payment terms).
- Customer accounting name, billing contact, and billing address (ask the user if
  not already known).
- The register backend and location from `admin.user-context.md`.
- Accountant recipients from `admin.user-context.md`'s
  `## Billing packet / accountant handoff` section.

## Step 1 — Draft register records

From the contract's finance-only fields, draft:

- One **Customer** record (reuse an existing one if the customer already has
  register entries — never create a duplicate).
- One **Project** record.
- One or more **Line Item** records — one per distinct billable thing the
  contract calls for (e.g. separate line items for a retainer vs. a one-time
  setup fee).
- **Cost Assignment** records, if the contract or the user names a
  subcontractor/vendor to charge to this project.

Follow `references/register-schema.md` exactly. Any field that can't be sourced
with confidence (an accounting product/service name that doesn't obviously match
anything already in the register or the accounting system, an ambiguous billing
cadence) is written as `NEEDS CONFIRMATION`, never guessed.

## Step 2 — Confirm with the user

Show the drafted records before writing anything. The user can edit, confirm, or
reject each. Nothing is written to the register backend until confirmed.

## Step 3 — Write to the register

Write confirmed records to the configured backend (local files or Airtable). Never
write to the repo; never write finance data into a shared/project-safe surface
(cortex client node, calendar descriptions, etc.) — this output is finance-zone
only, same boundary `clients:project-setup` already enforces for its own writes.

## Step 4 — Generate the setup sheet

A single document (markdown, suitable for pasting into an email body or attaching)
with:

- Customer name and billing contact
- Project name (exact accounting-system name)
- Terms
- Contract value
- Billing schedule
- Full line-item catalog: accounting product/service, invoice description, unit,
  rate
- Cost assignments
- Start date
- First expected invoice (date + what it will cover, derived from the billing
  schedule)

Any `NEEDS CONFIRMATION` fields are called out at the top of the setup sheet, not
buried in the table.

## Step 5 — Draft the accountant email

Draft (never send) an email to the accountant recipients from
`admin.user-context.md`, with the setup sheet as the body or attachment per the
configured packet format. Present it to the user for explicit approval before any
send action.

## What this does NOT do

- Never sends the email itself.
- Never writes to an accounting system directly — this produces the inputs a
  human (the user, or the accountant) acts on.
- Never invents a rate, accounting name, or term it isn't confident about.
