# admin user context (TEMPLATE)

_Run `/setup-time` to generate your real `<config-root>/plugins/admin.user-context.md` (gitignored)._

_Last updated: [filled by setup]_

## Clients and billing
[Per active client]

### [Client name]
- **Domains:** [comma-separated, for attendee matching]
- **Project tag:** [e.g., [ACME] prefix in event titles]
- **Billing model:** [hourly / retainer / flat-fee project]
- **Hourly rate:** $[/hr] (if hourly or overage)
- **Retainer:** $[/month] with [N] hrs cap (if retainer)
- **Flat fee total:** $[amount] over [duration] (if project)
- **Billing schedule:** [monthly / milestone / 50-50 / 100%]
- **Active period:** [start date – end date or open-ended]
- **Status:** [active / on-hold / closed]
- **Billing contact:** [name, email, address]

## Categories
- client-work
- internal
- bizdev
- learning
- personal
[+ user-customizable]

## Calendar tagging conventions
- **Method(s) used:** [domain matching / title prefix / color / description tag / combination]
- **Per-client mappings:** [how to map each client]

## Block rules
- **Rounding:** [15 min / 30 min / actual]
- **Prep/recap padding:** [enabled? minutes]

## Invoice preferences
- **Template:** references/templates/invoice-template.md
- **Net terms:** [net 0 / 7 / 14 / 30]
- **Numbering format:** [e.g., INV-2026-001]
- **Starting number:** ...
- **Tax:** [rate, basis, jurisdictions]
- **Delivery method:** [email / portal / Drive]
- **From address:** [or "from identity"]
- **Payment instructions:** [Stripe link / wire / ACH / check]

## Time-log location
- **File:** [default `<config-root>/time-log.csv`]
