---
description: Admin. Manually generate a new-customer/project setup notice for the accountant — drafts billing-register records from a signed contract, produces a setup sheet, and drafts (never sends) the handoff email. The same logic clients:project-setup runs automatically when a contract executes; use this command when you need to run it by hand (e.g. a contract signed before admin was installed, or a correction).
---

# /setup-notice

Draft a new-customer/project setup notice for the accountant, outside the
automatic `clients:project-setup` flow.

---

## Step 0 — Preflight

Read `<config-root>/plugins/admin.user-context.md`. Require:
- Billing register backend configured (if not, route to `/setup-time`)
- Accountant recipients configured under `## Billing packet / accountant handoff`
  (if not, ask for them now and offer to save)

Ask which project this is for, and where the signed contract lives (default:
the contract location from user-context).

---

## Step 1 — Generate

Follow `references/setup-notice-generation.md` Steps 1–5 exactly: draft register
records → confirm with user → write to register backend → generate the setup
sheet → draft the accountant email.

---

## Step 2 — Output

Show the confirmed register records written, the setup sheet, and the drafted
email. Ask whether to send the email now (explicit approval required — never
send by default).

---

## Behavior rules

- Never writes a register record without user confirmation.
- Never sends the email without explicit approval, separate from the confirmation
  in Step 1.
- Never guesses a rate, accounting name, or term — marks `NEEDS CONFIRMATION`
  per `references/register-schema.md`.
