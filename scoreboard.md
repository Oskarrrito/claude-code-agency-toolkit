---
name: scoreboard
description: Creates 6 Supabase tables tracking your ACDC sales pipeline. Volume, Response, Booking, Show-up, Close, Cash. Plain English data entry, weekly summaries, channel breakdowns. Run once for setup, then anytime to log data.
user-invocable: true
---

# Scoreboard: pipeline tracking system

**Purpose:** Build and maintain the 6-table pipeline tracking system in Supabase. One continuous chain from first outreach message to first dollar collected. After setup, log data in plain English and Claude handles the SQL. This skill is the data layer. /scoreboard creates tables and handles all data entry. /mission-control reads from these tables and visualizes the pipeline. No dashboards live here.

**Why it matters:** Every Monday, do a weekly review of your Scoreboard. Track what you shipped, not what you learned. "Did you run your outreach engine today?" is the daily question. The Scoreboard answers it with data.

## Input

- Read `business.md` for: niche, channels used, current pipeline status
- MCP: Supabase (create tables, insert/read/update data)
- $ARGUMENTS (optional): plain English data entry string (e.g., "sent 10 LinkedIn DMs today, got 2 responses")

## Step 1: Detect mode

1. Read `business.md`
2. Check Supabase connection via MCP
3. Check if the 6 scoreboard tables already exist in Supabase

**If Supabase MCP is not connected:**
Tell the user: "Supabase MCP isn't connected. I need it to create your scoreboard tables. Go to your Claude Code MCP settings and add Supabase, then run /scoreboard again." Stop here. Do not attempt to create local files as a substitute.

**If tables do not exist:** Enter SETUP MODE (Step 2)
**If tables exist and $ARGUMENTS provided:** Enter DATA ENTRY MODE (Step 3)
**If tables exist and no $ARGUMENTS:** Enter STATUS MODE (Step 4)

---

## Step 2: Setup mode (run once)

Create the 6 tables in order. Each table links to the previous one via foreign key, forming a single chain from first message to collected payment.

### Table 1: volume (9 columns)

Tracks every outreach message sent.

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| channel | text | linkedin_dm, cold_email, warm_intro, community, voice_note, referral, other |
| recipient_name | text | who you messaged |
| recipient_company | text | their business name (nullable) |
| message_type | text | warm, cold, follow_up |
| personalized | boolean | true if custom-written, false if template |
| notes | text | any context (nullable) |
| response_id | uuid, foreign key | links to response table (nullable, filled when reply comes in) |

### Table 2: response (8 columns)

Tracks every reply received.

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| volume_id | uuid, foreign key | links back to the volume entry that triggered this |
| sentiment | text | positive, neutral, negative |
| response_text | text | what they said (summary) |
| next_step | text | send_pricing, book_call, follow_up, dead |
| booking_id | uuid, foreign key | links to booking table (nullable) |
| notes | text | any context (nullable) |

### Table 3: booking (8 columns)

Tracks every call scheduled.

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| response_id | uuid, foreign key | links back to the response that led to this booking |
| call_date | date | when the call is scheduled |
| call_time | time | what time |
| platform | text | zoom, google_meet, phone, in_person |
| confirmed | boolean | did they confirm? |
| showup_id | uuid, foreign key | links to showup table (nullable) |

### Table 4: showup (6 columns)

Tracks every call that actually happened (or didn't).

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| booking_id | uuid, foreign key | links back to the booking |
| showed_up | boolean | did they show? |
| call_notes | text | what happened on the call (nullable) |
| close_id | uuid, foreign key | links to close table (nullable) |

### Table 5: close (16 columns)

Tracks every deal won or lost, plus churn tracking for recurring clients.

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| showup_id | uuid, foreign key | links back to the showup |
| outcome | text | won, lost, pending |
| service | text | what you sold |
| price | numeric | deal value |
| start_date | date | when work begins (nullable) |
| contract_type | text | monthly, project, retainer (nullable) |
| objections | text | what pushback they had (nullable) |
| proposal_sent | boolean | did you send a formal proposal? |
| signed | boolean | did they sign? |
| canceled | boolean | did the client cancel? Defaults to false |
| cancel_date | date | when they canceled (nullable) |
| cancel_reason | text | why they canceled (nullable) |
| months_retained | numeric | how many months they stayed before canceling (nullable) |
| cash_id | uuid, foreign key | links to cash table (nullable) |

### Table 6: cash (7 columns)

Tracks every dollar collected. Not invoiced. Collected.

| Column | Type | Notes |
|---|---|---|
| id | uuid, primary key | auto-generated |
| created_at | timestamptz | defaults to now() |
| close_id | uuid, foreign key | links back to the close |
| amount | numeric | actual dollars received |
| payment_method | text | stripe, bank_transfer, cash, venmo, other |
| invoice_number | text | your invoice reference (nullable) |
| received_date | date | when the money actually hit your account |

### After creating all 6 tables:

1. Enable Row Level Security on each table
2. Run a smoke test: insert a test row into the volume table, verify it exists, then delete it
3. Verify foreign key relationships by checking that each table's FK column references the correct parent table
4. Display results:

```
--- SCOREBOARD SETUP COMPLETE ---
Tables created: 6/6
  volume (9 columns)     OK
  response (8 columns)   OK
  booking (8 columns)    OK
  showup (6 columns)     OK
  close (16 columns)     OK
  cash (7 columns)       OK
Smoke test: passed
Foreign keys: verified

Your pipeline chain: Volume -> Response -> Booking -> Show-up -> Close -> Cash

You can now log data by running /scoreboard with a plain English description.
Example: "/scoreboard sent 5 LinkedIn DMs today to dentists"
```

**If some tables already exist:** Only create the missing ones. Do not drop or recreate existing tables. Tell the user which ones already existed and which were created.

### After setup, display revenue milestones

After the setup confirmation, show the revenue milestone tracker:

```
--- REVENUE MILESTONES ---
Track when you hit each level:
  First paying client
  $5K/month
  $10K/month
  $30K MRR

Mark these in your weekly review as you hit them.
```

---

## Step 3: Data entry mode

Log data in plain English. Claude parses the input and inserts the right rows into the right tables.

### Parsing rules

The user might say any of the following. Claude must handle all of them:

**Volume entries (outreach sent):**
- "Sent 10 LinkedIn DMs today"
- "Cold emailed 15 dentists in Austin"
- "Warm intro to Lisa at Peak Dental through Jake"
- "Left 3 voice notes for prospects from yesterday's list"
- "Sent follow-ups to everyone who didn't respond last week (8 total)"

Parse into: channel, count, message_type (warm/cold/follow_up), personalized flag, recipient details if given.

**Response entries (replies received):**
- "Dr. Kim responded on LinkedIn, asked about pricing. Positive."
- "Got 2 responses from cold emails. One positive (wants a call), one neutral (asked for more info)."
- "Jake's referral Lisa emailed back. She's interested."

Parse into: link to volume entry if identifiable, sentiment, response summary, next step.

**Booking entries (calls scheduled):**
- "Booked a call with Dr. Kim for Thursday 2pm on Zoom"
- "Lisa wants to meet Friday morning, Google Meet"

Parse into: link to response entry, call date, time, platform, confirmed status.

**Show-up entries:**
- "Dr. Kim showed up, great call, she wants a proposal"
- "Lisa no-showed"

Parse into: link to booking, showed_up boolean, call notes.

**Close entries:**
- "Dr. Kim signed! $997/month, starting Monday"
- "Lost the deal with the pizza shop. Said too expensive."

Parse into: link to showup, outcome, service, price, objections.

**Cash entries:**
- "Dr. Kim paid first month, $997 via Stripe"
- "Got the bank transfer from Lisa, $1200"

Parse into: link to close, amount, payment method, received date.

### Batch entry

If you give a batch ("Sent 10 LinkedIn DMs today"), create 10 volume entries with the same channel, date, and message_type. Set recipient_name to "batch_entry_1" through "batch_entry_10" unless specific names are given. Set personalized to false for batch entries unless they say otherwise.

### Confirmation

After parsing, show what will be inserted before inserting:

```
--- LOGGING ---
Volume: 10 LinkedIn DMs (cold, batch)
Response: 2 replies (1 positive, 1 neutral)
Booking: 1 call (Dr. Kim, Thu 2pm, Zoom)

Insert these? (yes/adjust)
```

Wait for confirmation. Then insert and confirm:

```
--- LOGGED ---
10 volume entries added
2 response entries added
1 booking entry added

Totals this week: 47 sent, 6 responses, 2 calls booked
Outreach streak: 3 days (target: 7 consecutive)
```

### Linking entries

When you mention a name that matches an existing entry (e.g., "Dr. Kim responded" and there's a volume entry for Dr. Kim), automatically link them via foreign key. If the match is ambiguous (multiple Dr. Kims), ask which one.

---

## Step 4: Status mode

When you run /scoreboard with no arguments and tables already exist, show a quick status:

```
--- SCOREBOARD STATUS ---
This week (Mon-Sun):
  Volume:    [count] sent ([warm] warm, [cold] cold, [followup] follow-up)
  Response:  [count] replies ([positive] positive, [neutral] neutral)
  Booking:   [count] calls scheduled
  Show-up:   [count] calls completed ([no-show] no-shows)
  Close:     [count] deals ([won] won, [lost] lost, [pending] pending)
  Cash:      $[amount] collected

Outreach streak: [count] consecutive days with outreach logged
Milestone: [most recent hit] (next: [next milestone])

All time:
  Total sent: [count]
  Total revenue: $[amount]
  Clients: [count]

Log data: just tell me what happened in plain English.
Example: "sent 8 DMs on LinkedIn, got 1 response from a dentist"
```

### Outreach streak tracking

Track consecutive days where at least one outreach entry was logged. A day counts if any volume entry has a `created_at` timestamp on that date. The streak resets to 0 if a day has no volume entries.

Display the streak in every status check and after every data entry. The streak is the daily accountability metric. "Did you run your outreach today?" The streak answers it.

---

## Channel tracking

Every volume entry has a channel field. The valid channels are:

- `linkedin_dm` - direct messages on LinkedIn
- `cold_email` - outbound email to people who don't know you
- `warm_intro` - messages to people in your network or referred
- `community` - posts/replies in Slack, Reddit, Discord, Facebook groups
- `voice_note` - audio messages sent via DM
- `referral` - warm introductions from existing contacts/clients
- `other` - anything that doesn't fit the above

If the user mentions a channel not on this list, map it to the closest one. If unclear, ask.

### Instantly MCP integration (cold email)

For users on Instantly for cold email, the Instantly MCP auto-tracks opens and replies. Those metrics feed into the scoreboard without manual entry. But bookings, show-ups, and closes still need manual entry here because Instantly has no visibility past the reply stage. The scoreboard is the single source of truth for the full pipeline. Instantly just supplements the cold_email channel with engagement data that would be tedious to log by hand.

Channel data is what makes /mission-control useful. Without it, those skills can't compare channel performance. Stress this on first setup:

```
Make sure to mention the channel when you log outreach. "Sent 10 DMs" is
less useful than "sent 10 LinkedIn DMs." The channel data powers your
growth analysis later.
```

---

## Weekly summary

If you ask for a weekly summary (or it's Friday), pull data from the current week and display:

```
--- WEEK OF [date range] ---
Volume by channel:
  LinkedIn DM:  [count] sent, [response count] responses ([rate]%)
  Cold email:   [count] sent, [response count] responses ([rate]%)
  Warm intro:   [count] sent, [response count] responses ([rate]%)
  Voice note:   [count] sent, [response count] responses ([rate]%)

Pipeline:
  [count] sent -> [count] responses -> [count] booked -> [count] showed -> [count] closed -> $[amount]

Conversion rates:
  Response rate:  [percent]%
  Booking rate:   [percent]%
  Show-up rate:   [percent]%
  Close rate:     [percent]%

vs Last week:
  Volume: [change]
  Responses: [change]
  Cash: [change]
```

---

## Error handling

**Supabase MCP not connected:** Stop immediately. Tell the user to connect it. Do not create local files as a fallback. The whole point of the scoreboard is structured data in Supabase that other skills can query.

**Table creation fails:** Show the exact error. Common issues: permissions (needs service role for DDL), table name conflicts, RLS policy issues. Tell the user what to check in their Supabase dashboard.

**Data entry fails:** Show what failed and what succeeded. If 8 of 10 batch entries went through and 2 failed, confirm the 8 and retry the 2. Do not silently drop entries.

**Ambiguous input:** When the plain English is unclear ("logged some messages"), ask for specifics: "How many? What channel? Warm or cold?" Do not guess.

---

## Self-check before finishing

1. All 6 tables exist in Supabase with correct column counts (volume: 9, response: 8, booking: 8, showup: 6, close: 16, cash: 7)
2. Foreign key chain is intact: volume -> response -> booking -> showup -> close -> cash
3. RLS enabled on all tables
4. Smoke test passed (insert + read + delete)
5. Data entries confirmed before inserting (never insert without approval)
6. Channel field populated on every volume entry
7. Foreign key links created when names match existing entries
8. Weekly totals displayed after every data entry session
9. Outreach streak displayed after every data entry and status check
10. Revenue milestone shown in status mode (most recent hit + next target)
11. No em dashes anywhere in the output
12. Plain English parsing handles all example formats listed above

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Clarified scoreboard as data layer: /scoreboard creates tables and handles data entry, /mission-control reads and visualizes. Added Instantly MCP integration note for cold email channel (opens and replies auto-tracked, bookings/showups/closes still manual). Added churn tracking to close table (canceled, cancel_date, cancel_reason, months_retained). Close table now 16 columns. |
| 2026-03-30 | Full rewrite. Added: three operating modes (setup, data entry, status), complete table schemas with all columns and types, plain English data entry parser with examples for every table, batch entry support, entry linking via name matching, confirmation flow before any insert, weekly summary with channel breakdown and conversion rates, channel tracking spec with valid channel list, status mode for quick pipeline view. Added revenue milestone tracking and outreach streak tracking (consecutive days with outreach logged). |
