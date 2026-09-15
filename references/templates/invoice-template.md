# Invoice Template

_Edit this template to match your firm's invoice format. The starter is generic._

When `/invoices` runs, it produces a structured invoice draft using this template. Replace `[bracketed]` fields with actual values.

---

```
INVOICE

Invoice #: [INV-YYYY-NNN]
Issue Date: [YYYY-MM-DD]
Due Date: [YYYY-MM-DD]

---

FROM
[Your Name / Your Company]
[Address line 1]
[Address line 2]
[Email] · [Phone if applicable]

TO
[Client Name]
[Billing contact name]
[Address]
[Email]

---

DESCRIPTION
[Engagement name, e.g., "AI Operating Model & Governance — April 2026"]

[Optional: 1-line scope reference, e.g., "Per SOW dated 2026-03-15"]

---

LINE ITEMS

| Date       | Description                                | Hours / Qty | Rate     | Amount   |
|------------|--------------------------------------------|-------------|----------|----------|
| [date]     | [description]                              | [hrs]       | $[/hr]   | $[amt]   |
| [date]     | [description]                              | [hrs]       | $[/hr]   | $[amt]   |
| ...        | ...                                        | ...         | ...      | ...      |
|            |                                            |             |          |          |
|            | **Subtotal**                               |             |          | $[X.XX]  |
|            | **Tax** ([rate]%, if applicable)           |             |          | $[X.XX]  |
|            | **TOTAL DUE**                              |             |          | **$[X.XX]** |

For retainer clients, replace the line items with:
| Date Range          | Description                  | Amount   |
|---------------------|------------------------------|----------|
| [month]             | [Retainer per agreement]     | $[amt]   |

For flat-fee project clients with milestone billing:
| Milestone           | Description                  | Amount   |
|---------------------|------------------------------|----------|
| [Milestone name]    | [What was delivered]         | $[amt]   |

---

PAYMENT INSTRUCTIONS
[Stripe link / wire details / ACH / check payable to / etc. — pulled from user-context]

Net [N] from issue date.

---

NOTES
[Any custom notes — e.g., "Includes prep + recap for all meetings per agreement" or "Phase 2 deliverables included; Phase 3 to be billed at completion."]

Thank you for your business.
```

---

## Customizing this template

Common edits:
- **Brand styling** — if your firm has brand colors, font choices, etc., the docx skill (when called by `/invoices`) can apply them on conversion. Adjust the brand-pass logic in your firm's `ops` user-context if applicable.
- **Tax rules** — different jurisdictions and product types have different rules. Edit the line-item formula and tax line accordingly.
- **Multi-currency** — if you bill in multiple currencies, add currency to the user-context's per-client config and update the template to render the currency code.
- **Logo / signature image** — handled at docx-conversion time, not in this markdown template.

The plugin produces structured invoice data per client; this template defines the shape of that data. The actual document (.docx, .pdf) is produced by your invoice skill or whatever tool you use for final delivery.
