---
description: Admin. Configure time-tracking for your billing models, calendar tagging conventions, and invoice preferences. Captures clients with rates, categories, rounding rules, and invoice templates. Re-run anytime to add or update clients.
---

# /setup-time

Short interview that captures what time-tracking needs to classify your calendar correctly and bill correctly.

---

## Step 0 — Resolve plugin config root

Resolve explicit override → `CORTEX_CONFIG_ROOT` → `~/.cortex/config-root` →
legacy pointer → default. Request access only to the resolved directory. If no root
has been intentionally configured, route to Cortex setup instead of creating a
plugin-specific pointer.

### Read shared identity

Read `<config-root>/memory/me/identity.md`. If populated, pre-fill company name, your name (for invoices), time zone, primary calendar. If missing, offer `/setup-identity` first or proceed inline.

For the rest of this document, **`<config-root>`** refers to the resolved path. This plugin's config file lives at **`<config-root>/plugins/admin.user-context.md`** and the time log at **`<config-root>/time-log.csv`**.

---

## Step 1 — Check existing config

Read `<config-root>/plugins/admin.user-context.md`. Populated → ask whether to update or restart. Missing → fresh interview.

If `<config-root>/plugins/clients.user-context.md` has populated engagements, offer:
> "I see active Client Success engagements: [list]. Want to import those as billing-tracked clients? I'll just need rates and billing models for each."

If yes → use Client Success data as the client list, ask for billing models per client. If no → capture clients fresh in Section 2.

---

## Step 2 — Clients and billing models

For each active client, capture:

- **Client name** (matches the name in your CRM and Client Success config)
- **Client domain(s)** (for attendee-domain matching, e.g., `acme.com`)
- **Project tag** in calendar (e.g., `[ACME]` prefix in event titles, or a project name)
- **Billing model** — pick one:
  - **Hourly** → ask: hourly rate ($/hr); rounding (15 min / 30 min / actual)
  - **Retainer** → monthly amount; included hours cap (if any); overage rate ($/hr); cap behavior (warn / hard-stop / auto-bill overage)
  - **Flat-fee project** → total contract value; billing schedule (monthly / milestone / 50-50 / 100% upfront / 100% on completion)
- **Active period** (start date / end date or "open-ended")
- **Status** (active / on-hold / closed)
- **Billing contact** (email + name + address for invoices)

Repeat per client. Walk through one at a time — don't bulk-ask.

---

## Step 3 — Categories

Default categories for the time-log:
- `client-work` (billable client engagement)
- `internal` (team meetings, admin)
- `bizdev` (outreach, networking, prospecting)
- `learning` (research, courses, reading)
- `personal` (slipped into work calendar)

Ask: any custom categories? Anything you want to add or rename? (Some users add `creative` for content production, `mentoring` for advisory time, etc.)

---

## Step 4 — Calendar tagging conventions

How do you mark events in your calendar today? Pick what applies:

- **Attendee domain matching** — Acme employees email from `@acme.com` → I match those emails to the Acme Corp client
- **Title prefix** — `[ACME] Discovery call` → I match `[ACME]` to the Acme Corp client
- **Calendar color** — yellow = client A, blue = client B, etc.
- **Description tag** — first line of description has `client: acme`
- **Combination** — most users use 2 of these

Capture which method(s) apply per client.

---

## Step 5 — Default block rounding and prep/recap padding

- **Rounding rule** — round each event to nearest: 15 min / 30 min / actual minutes (default 15)
- **Prep/recap padding for client calls** — automatic add for prep time before and recap after a meeting (default: 0 min — disabled). If enabled, specify minutes (e.g., 15 min prep + 15 min recap).

Note: padding is opinionated. Some clients accept it, others don't. Default off.

---

## Step 6 — Invoice preferences

- **Invoice template** — immutable default at `references/templates/invoice-template.md`; customized copies live at `<config-root>/plugins/admin/templates/invoice-template.md`
- **Net terms** — net 0 (due on receipt) / net 7 / net 14 / net 30 (default net 14)
- **Invoice numbering** — start number, format (e.g., `INV-2026-001`, `BWA-25-04`, etc.)
- **Tax** — do you charge sales tax / GST / VAT? If yes: rate and basis
- **Delivery method** — email / client portal / Drive folder upload
- **From address** — your billing address (or "from identity" if `<config-root>/memory/me/identity.md` has it)
- **Payment instructions** — Stripe link / wire / check / ACH details

---

## Step 7 — Time-log location

- **File path** (default `<config-root>/time-log.csv`) — anywhere you want, but keep it stable so the schema stays consistent

---

## Step 8 — Write the config

Populate `<config-root>/plugins/admin.user-context.md` with everything captured. Use the `references/user-context.template.md` structure.

---

## Step 9 — Confirm and offer next step

Summarize. Offer:

> "Setup done. Try `/track-time` for yesterday — see how the classification goes. Refine setup later if anything's off."

---

## Behavior rules

- One section at a time. Don't flood.
- Capture clients one at a time — important to get the rates right.
- Idempotent. Re-running adds new clients or updates existing ones.
- If Client Success is configured, prefer importing client data from there to avoid drift.
