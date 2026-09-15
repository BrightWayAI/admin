# Time Log Schema

The time-log lives at `<config-root>/time-log.csv` (or wherever your user-context's `time-log location` points).

Plain CSV — portable, readable, hand-editable, backup-friendly.

## Schema

```csv
date,start,end,duration_min,client,project,category,billable,description,invoiced
```

### Field definitions

| Field | Type | Required | Notes |
|---|---|---|---|
| `date` | YYYY-MM-DD | yes | Date the work happened |
| `start` | HH:MM (24h) | yes | Start time in user's time zone |
| `end` | HH:MM (24h) | yes | End time |
| `duration_min` | int | yes | Minutes (after rounding rule applied) |
| `client` | string | yes | Matches a client name in user-context, or `internal`, or `unknown` |
| `project` | string | optional | Project name within the client; blank if N/A |
| `category` | string | yes | One of the categories from user-context (`client-work`, `internal`, `bizdev`, `learning`, `personal`, etc.) |
| `billable` | boolean | yes | `true` or `false` |
| `description` | string | yes | What the time was for (≤120 chars; the calendar event title + key context) |
| `invoiced` | boolean | yes | `false` until `/invoices` runs and bills it; then `true` |

### Quoting rules

Standard CSV quoting:
- Fields containing commas must be wrapped in double quotes
- Internal double quotes are doubled (`""`)
- Newlines inside fields are not allowed — use spaces instead

### Example rows

```csv
2026-05-08,09:00,10:30,90,Acme Corp,AI Op Model,client-work,true,"Stakeholder interview — COO",false
2026-05-08,10:30,11:00,30,Acme Corp,AI Op Model,client-work,true,"Synthesis notes from interview",false
2026-05-08,11:00,11:30,30,internal,,bizdev,false,"LinkedIn outreach batch",false
2026-05-08,14:00,16:00,120,Beta Inc,Custom Agents,client-work,true,"Build session — alpha agent",false
```

### Append-only

`/track-time` only appends. It never modifies existing rows except:
- `/invoices` flips `invoiced` from `false` to `true` after billing the row.
- Manual edits by the user (in a CSV editor or text editor) are fine — the plugin doesn't enforce structure beyond reading what's there.

### Backups

The file is plain text — back up however you back up `~/Documents/Claude/`. Cortex memory and identity also live in that directory, so a single backup covers your whole marketplace plugin state.

### Future fields (v0.2+)

Possible additions:
- `tags` — comma-separated free-form tags for filtering (e.g., "deep-work,strategy")
- `payment_status` — once paid, mark `paid` (would shift the plugin into AR tracking territory; currently out of scope)
- `attendees` — JSON list of attendee emails for richer reporting

For now, keep it minimal. Plugin reads exactly the schema above.
