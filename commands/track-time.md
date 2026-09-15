---
description: Pull calendar events from a time window (default: yesterday or last week), classify each as billable / non-billable / unknown by client, prompt for confirmation, and append to the time log at <config-root>/time-log.csv. Run daily (5–10 min) or weekly (15–20 min). Reconstructing time at month-end takes hours; this turns it into a few minutes a day.
---

# /track-time

Pull calendar → classify → log billable time. Closes the gap between "what you actually did" and "what you'll bill for."

---

## Step 0 — Preflight

Read `<config-root>/plugins/admin.user-context.md`. If missing, route to `/setup-time` and stop.

Also read shared identity at `<config-root>/memory/me/identity.md` (calendar tool,
time zone). If `<config-root>/plugins/clients.user-context.md` exists, read it for
active engagement data (client names, contract values, project tags).

Extract from this plugin's user-context:
- Active clients + billing models (hourly rate / retainer / flat fee)
- Tagging conventions (attendee domains / title prefix / project tags)
- Default block rounding (15 / 30 / actual)
- Categories (client-work / internal / bizdev / learning / etc.)
- Time-log file path (default `<config-root>/time-log.csv`)

---

## Step 1 — Determine the window

Default windows:
- **Daily mode** (most common) — yesterday's calendar, working hours window
- **Weekly mode** — last Monday–Friday
- **User-specified** — `/track-time [start-date] [end-date]` — use those

Announce: "Tracking time for [window]."

---

## Step 2 — Pull the calendar

Query the calendar for the window. Capture each event:
- Date, start, end, duration (minutes)
- Title
- Attendees (with domains)
- Description / notes
- Recurring? (mark in case the user wants to bulk-handle)

Skip events that are clearly non-billable in the user's setup:
- All-day blocks like "vacation," "holiday," "sick"
- Pure internal meetings if user-context excludes those (per Categories config)

---

## Step 3 — Classify each event

For each event, determine:

**A. Client / project assignment**

Match against active clients via the user's tagging convention:

- **Attendee domain matching** — if any attendee email domain matches a known client domain → that client
- **Title prefix** — if title starts with `[ClientName]` or `ClientName:` → that client
- **Project tag** — if event description contains a known project tag → that client/project
- **Calendar / category color** — if user-context maps colors to clients

If multiple matches: prefer most-specific (project tag > title prefix > attendee domain).
If no match: classify as `unknown` and prompt the user to assign at confirmation time.

**B. Category**

- `client-work` — direct client engagement (calls, deep work on deliverables, prep)
- `internal` — internal team meetings, admin
- `bizdev` — outreach, prospecting, networking
- `learning` — research, reading, courses (usually non-billable)
- `personal` — anything personal that leaked into work calendar
- `unknown` — needs user input

**C. Billable flag**

Default rules (configurable per user-context):
- `client-work` → billable = true (unless client is on a flat fee — then track but don't aggregate hours)
- `internal` / `bizdev` / `learning` / `personal` → billable = false

**D. Duration adjustments**

- Apply rounding rule from user-context (15 min / 30 min / actual)
- For client meetings: include 5-15 min for prep + recap if not already on calendar (per user-context default)

---

## Step 4 — Confirm with the user

Present the classified list as a table for review:

```
**Time tracked for [window]** — [N] events, [total hours] hrs total

| Date  | Start | Duration | Client       | Project       | Cat         | Bill? | Description                   |
|-------|-------|----------|--------------|---------------|-------------|-------|-------------------------------|
| 5/7   | 9:00  | 90 min   | Acme Corp    | AI Op Model   | client-work | ✓     | Stakeholder interview — COO   |
| 5/7   | 11:00 | 30 min   | Acme Corp    | AI Op Model   | client-work | ✓     | Synthesis notes               |
| 5/7   | 14:00 | 60 min   | unknown      | -             | unknown     | ?     | "Working session"             |
| 5/7   | 15:30 | 30 min   | -            | -             | bizdev      | -     | LinkedIn outreach             |

**[N] needs assignment** — let me know how to classify the unknown rows.
**[Total billable: X.XX hrs]**
```

For unknown rows, ask one at a time: "5/7 14:00, 60 min, 'Working session' — which client/project?" Use a short choice list of active clients + "internal" + "skip."

After all classifications: "Look right? (Y to log, edit to change, skip to drop a row)"

---

## Step 5 — Append to time-log

Once confirmed, append rows to `<config-root>/time-log.csv`. Format per `references/time-log-schema.md`:

```csv
date,start,end,duration_min,client,project,category,billable,description,invoiced
```

Don't overwrite existing rows. Don't write duplicates — check for existing rows in the same window with the same start time + client.

If the file doesn't exist, create it with the header row.

---

## Step 6 — Output

Brief confirmation:

```
Logged [N] entries.

This [window]:
- Billable: [X.XX hrs] across [M clients]
- Internal: [Y.YY hrs]
- Bizdev: [Z.ZZ hrs]

This month so far:
- Billable: [running total per client]

Run /invoices on [next-month-date] to bill what's logged.
```

---

## Behavior rules

- **Don't over-classify.** "Unknown" is a valid answer. The point is capture, not perfection.
- **Honor recurring events.** If a recurring meeting was just classified as "Acme Corp, billable," offer "apply to all future instances of this recurring event?" so the user doesn't re-classify weekly.
- **Append-only.** Never modify existing rows in the time-log. If the user wants to correct a past entry, they edit the CSV directly or use a future `/edit-time-entry` command.
- **5-minute capture target.** Daily run should take 5 min once classification is mostly automated. Weekly run should take 15-20 min.

## Edge cases

- **Calendar empty for the window** — say so, log nothing, recommend running again after meetings happen.
- **All events look unknown** — likely the tagging convention isn't right. Recommend re-running `/setup-time` to refine.
- **Time conflicts** (two events at the same time) — flag and let the user pick which counts.
- **Travel days / wall-to-wall meetings** — fine, log them all; the totals will reflect reality.

## What this is NOT for

- **Real-time tracking** (start/stop timers). The plugin is calendar-driven — you record events, then classify them after.
- **Detailed work logs** (every commit / file edit). It's calendar-event resolution.
- **Project management.** The plugin tracks time, not progress on deliverables.
