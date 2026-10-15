---
description: Admin. One-time (or re-run) seeding of the billing register from your accounting system and signed contracts. Pulls customers, projects/sub-customers, and the product/service catalog through the configured accounting connector (QuickBooks first), reads signed contracts from the configured location, and drafts register records for your confirmation. Never guesses a rate or term — unclear fields are marked NEEDS CONFIRMATION. Writes only to your configured backend, never into the repo.
---

# /seed-register

Bootstrap the billing register from what already exists in your accounting
system and your signed contracts, instead of entering everything by hand.

---

## Step 0 — Preflight

Read `<config-root>/plugins/admin.user-context.md`. Require a billing register
backend to be configured (route to `/setup-time` if not). Confirm:
- Accounting connector (if any) — QuickBooks supported first; structured so
  others can be added without redesign
- Contract location

If no accounting connector is configured, say so and offer to proceed
contract-only (Step 2 below, skipping Step 1).

---

## Step 1 — Pull from the accounting system

If an accounting connector is configured:
- Pull the customer list → draft Customer records (accounting name exact,
  billing contact/address/terms as available; `NEEDS CONFIRMATION` for anything
  the connector doesn't expose).
- Pull projects/sub-customers (or jobs/classes, depending on the accounting
  system's terminology) → draft Project records.
- Pull the product/service list → this becomes the set of valid
  `accounting_product_service` values Line Items must match exactly.

This step never invents an accounting name — only names the connector actually
returns are used.

---

## Step 2 — Read signed contracts

Read contracts from the configured location. For each, extract finance-only
terms (billing model, rate(s), contract value, cap/NTE, invoicing cadence,
payment terms, start/end dates) and draft:
- Project fields not already covered by Step 1 (contract value, billing model,
  schedule, dates)
- Line Item records — invoice description, unit, rate, billing rule
- Cost Assignment records, if the contract names a subcontractor/vendor

Match each project to its accounting-system counterpart from Step 1 by name
where possible; if no confident match exists, mark the project's
`accounting_name` `NEEDS CONFIRMATION` rather than guessing.

---

## Step 3 — Confirm with the user

Present every drafted record, grouped by entity, with all `NEEDS CONFIRMATION`
fields called out at the top. Walk confirmations — don't bulk-accept rates or
terms the user hasn't reviewed.

---

## Step 4 — Write

Write confirmed records to the configured backend (local files or Airtable).
Never to the repo. Re-running `/seed-register` updates existing records by
matched key rather than duplicating them.

---

## Behavior rules

- Never guesses a rate, term, or accounting name — see
  `references/register-schema.md`'s `NEEDS CONFIRMATION` rule.
- Read-only against the accounting system and contracts; all writes go to the
  user's configured register backend only, after explicit confirmation.
- Idempotent — safe to re-run as new customers/projects/contracts appear.
