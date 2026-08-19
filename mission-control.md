---
name: mission-control
description: Your personal admin dashboard. Reads your Supabase data and Instantly MCP analytics, builds a visual control center you keep open alongside Claude Code. Outreach pipeline, scoreboard metrics, channel performance, bottleneck finder. You build what you need.
user-invocable: true
---

# Mission control: your personal admin dashboard

**Purpose:** This is your admin page. It pulls data from Supabase (scoreboard tables) and Instantly MCP (email campaign analytics), combines everything into one visual dashboard, and opens it in your browser. Outreach stats, pipeline numbers, channel breakdowns, bottleneck identification, and weekly priorities. You keep it open on the right side of your screen, Claude Code on the left. The idea is simple: build what you need yourself. This gives you the skeleton. You make it your own.

## Input

- Read `business.md` for: niche, offer, current goals, weekly targets
- MCP: Supabase (read from all 6 scoreboard tables: volume, response, booking, showup, close, cash)
- MCP: Instantly (read campaign analytics: open rates, reply rates, bounce rates, campaign-level stats)
- $ARGUMENTS (optional): "compare [date]" to compare against a specific week, or "channel [name]" to filter by channel

## Step 1: Check prerequisites

1. Read `business.md`
2. Verify Supabase MCP is connected
3. Verify Instantly MCP is connected (optional but recommended)
4. Verify the 6 scoreboard tables exist

**If Supabase MCP is not connected:**
Tell the user: "Supabase MCP isn't connected. I need it to read your scoreboard data. Connect Supabase MCP in your Claude Code settings, then run /mission-control again." Stop here.

**If Instantly MCP is not connected:**
Tell the user: "Instantly MCP isn't connected. You can still run Mission Control with Supabase data only, but you'll miss email campaign analytics. Connect Instantly MCP to get the full picture." Continue without Instantly data.

**If scoreboard tables don't exist:**
Create them now. Run the Scoreboard setup (6 Supabase tables: volume, response, booking, showup, close, cash). Don't ask the user to run a separate command.

**If tables exist but are empty (no data):**
Enter ZERO DATA PROJECTION MODE. Do NOT stop. Instead, generate projections based on business.md targets and industry benchmarks. Show what the dashboard WILL look like after Week 1 data comes in. This gives you a target to aim for on Day 0 instead of a dead end.

Display: "Your scoreboard tables are empty. Generating projected dashboard based on your business.md targets and benchmark conversion rates. These projections get replaced with real numbers as you log data. Tell me what happened in plain English and I'll log it for you."

---

## Step 2: Pull data from Supabase and Instantly

### Supabase data

Query all 6 tables for two time ranges:

**This week:** Monday 00:00 through current timestamp
**Last week:** Previous Monday 00:00 through previous Sunday 23:59

For each table, pull:
- Total count of entries
- Breakdown by channel (volume and response tables)
- Breakdown by outcome (close table: won/lost/pending)
- Sum of amounts (cash table)

If $ARGUMENTS includes a channel filter, apply it to all queries.

### Instantly data

If Instantly MCP is connected, pull:
- Active campaigns and their status
- Open rates per campaign
- Reply rates per campaign
- Bounce rates per campaign
- Total emails sent (this week vs last week)
- Leads contacted vs leads replied

Merge Instantly email data into the channel breakdown. Cold email volume from Instantly supplements manual tracking in Supabase.

### Zero data projection mode

If tables are empty (no volume entries), skip the Supabase queries and instead:
1. Read business.md for: revenue goal, weekly targets, channel list, current client count
2. Use benchmark conversion rates from the benchmark table in Step 3
3. Project Week 1-4 numbers based on the targets in business.md
4. Show the revenue progress bar (see Step 3a)
5. Show milestone projections (see Step 3b)
6. Make it clear these are projections that get replaced by real data

---

## Step 3: Calculate metrics

### Raw numbers

```
This Week          | Last Week          | Change
Volume:    [count] | Volume:    [count] | [+/- count] ([+/- %])
Response:  [count] | Response:  [count] | [+/- count] ([+/- %])
Booking:   [count] | Booking:   [count] | [+/- count] ([+/- %])
Show-up:   [count] | Show-up:   [count] | [+/- count] ([+/- %])
Close:     [count] | Close:     [count] | [+/- count] ([+/- %])
Cash:      $[amt]  | Cash:      $[amt]  | [+/- $amt]  ([+/- %])
```

### MRR (Monthly Recurring Revenue)

Calculate from the close table using actual client prices, not hardcoded membership prices. Sum monthly_price for active monthly clients. For annual clients, divide their annual_price by 12. If monthly_price/annual_price columns don't exist, fall back to summing the deal_value or amount column divided by the contract length. Display as the headline metric at the top of the dashboard. Show MRR trend (this month vs last month).

Active means: outcome = 'won' AND (canceled = false OR canceled IS NULL). Use contract_type to distinguish monthly vs annual. If contract_type is not set, treat as monthly by default. MRR goes above the raw numbers block, right after the page header.

### Conversion rates

Calculate the rate between each adjacent stage:

- **Response rate:** responses / volume (how many messages get a reply)
- **Booking rate:** bookings / responses (how many replies become calls)
- **Show-up rate:** showups / bookings (how many booked calls actually happen)
- **Close rate:** closes (won) / showups (how many calls become deals)
- **Cash rate:** cash entries / closes (how many deals actually get paid)

### Benchmark comparison

Color-code each conversion rate against these benchmarks:

| Metric | Green (healthy) | Amber (borderline) | Red (problem) |
|---|---|---|---|
| Weekly volume | 20+ messages | 10-19 messages | Under 10 messages |
| Response rate (cold) | 5%+ | 2-5% | Under 2% |
| Response rate (warm) | 20%+ | 10-20% | Under 10% |
| Booking rate | 30%+ | 15-30% | Under 15% |
| Show-up rate | 70%+ | 50-70% | Under 50% |
| Close rate | 20%+ | 10-20% | Under 10% |
| Cash rate | 90%+ | 70-90% | Under 70% |
| Instantly open rate | 50%+ | 30-50% | Under 30% |
| Instantly reply rate | 5%+ | 2-5% | Under 2% |
| Instantly bounce rate | Under 3% | 3-5% | Over 5% |

For response rate, use the appropriate benchmark based on the channel mix. If you send mostly cold, use cold benchmarks. If mostly warm, use warm benchmarks. If mixed, calculate separate rates for warm and cold channels.

### Step 3a: Revenue progress bar

Read business.md for the revenue goal (e.g., $30K MRR). Read the current revenue from close table data. Display a progress bar:

```
Revenue: $[current] / $[goal]
[██████████░░░░░░░░░░░░░░░░░░░░] [percent]%
```

This goes at the top of the dashboard, right after the raw numbers. The progress bar is the macro view. Everything else is the micro view of how to get there.

### Step 3b: Milestone projections

Based on current conversion rates (or benchmarks if in zero data mode), estimate when you'll hit each revenue milestone:

```
--- MILESTONES ---
First paying client:    [hit / projected by Day X]
$5K/month:              [hit / projected by Day X]
$10K/month:             [hit / projected by Day X]
$30K MRR:               [hit / projected by Day X]
```

Revenue is the metric that matters. Track these every Monday alongside the pipeline numbers.

---

## Step 4: Identify the bottleneck

The bottleneck is the single conversion rate that is furthest below its benchmark. Not the lowest rate (close rate is naturally lower than show-up rate). The one with the biggest gap between actual and target.

**Bottleneck scoring:**
For each metric, calculate: (benchmark midpoint - actual rate) / benchmark midpoint. The highest score is the bottleneck.

Example: If response rate is 3% against a 7.5% benchmark midpoint, the gap score is (7.5 - 3) / 7.5 = 0.60. If close rate is 15% against a 25% midpoint, the gap score is (25 - 15) / 25 = 0.40. Response rate is the bigger problem.

**Instantly-specific bottleneck detection:** If Instantly data is available and bounce rate is over 5%, flag it as a separate warning: "Your email deliverability is hurting you. [bounce_rate]% bounce rate means your emails aren't landing. Fix your list hygiene before sending more." If open rate is under 30%, flag subject lines. If reply rate is under 2% but open rate is fine, flag email copy.

**Special case: volume under 20/week.** If weekly volume is under 20 messages, volume IS the bottleneck regardless of conversion rates. Conversion rates are unreliable at low volume. Display: "Your sample size is too small for conversion rates to mean anything. Send more messages first. Target: 20+ per week." Planning without shipping is the trap. The outreach engine exists to break it.

Display the bottleneck clearly:

```
--- BOTTLENECK ---
[Stage name]: [actual rate]% (benchmark: [target]%)
[One sentence explaining what this means in plain language]
```

---

## Step 5: Generate 3 action items

Based on the bottleneck and the data, generate exactly 3 specific action items for this week. Not generic advice. Specific actions with numbers.

**Action item rules:**
- Each action starts with a verb
- Each action includes a specific number or target
- Each action is tied to the identified bottleneck
- Actions should be doable within one week

**Examples by bottleneck:**

If the bottleneck is **volume (under 20/week):**
1. "Increase daily outreach from [current daily average] to [current + 5]. Block 30 minutes every morning."
2. "Add a second channel. You're only using [current channel]. Try [suggested channel] for 10 messages this week."
3. "Run /outreach-engine every day this week. Your streak is [current streak]. Target: 5 consecutive days."

If the bottleneck is **response rate:**
1. "Rewrite your opening line. Your current first line is getting [rate]% responses. Test 3 variations on the next 30 messages."
2. "Switch [count] of your cold messages to personalized. Your personalized messages get [personal rate]% vs [template rate]% for templates."
3. "Add voice notes to your top 3 prospects each day. Voice notes typically get 15-40% response rates."

If the bottleneck is **booking rate:**
1. "Change your CTA from [current CTA] to a direct calendar link. Remove friction."
2. "Follow up on the [count] positive responses that haven't booked yet. Send a 'still interested?' message today."
3. "Offer two specific time slots instead of 'when works for you.' Specific options get more bookings."

If the bottleneck is **show-up rate:**
1. "Send a confirmation message 24 hours before every booked call. You have [count] calls booked this week."
2. "Send a 15-minute reminder the day of. Include the meeting link again."
3. "Add a 'looking forward to it' message right after booking. People who get an immediate confirmation are more likely to show."

If the bottleneck is **close rate:**
1. "Prep a call script before your next call. Run /close-deal prep-call mode."
2. "Review your last [count] call notes. Identify the moment the deal stalled (price objection, timing, scope)."
3. "Practice your price delivery. Say your price out loud 10 times: '$[price] per month.' No flinching."

If the bottleneck is **Instantly deliverability (bounce > 5%):**
1. "Clean your email list. Remove all addresses that bounced in the last 2 weeks."
2. "Verify your next batch of leads before importing to Instantly. Use a verification tool."
3. "Reduce daily send volume by 30% until bounce rate drops under 3%."

---

## Step 6: Generate the HTML dashboard

Create a clean, dark-themed HTML file that displays everything from Steps 3-5. The dashboard should be functional and readable, not fancy.

### Dashboard layout

```
+--------------------------------------------------+
| MISSION CONTROL          [date range] [channel]   |
+--------------------------------------------------+
| MRR: $[amount]          [+/- $change] vs last mo |
+--------------------------------------------------+
| OUTREACH ENGINE                                   |
| Instantly: [sent] sent, [open_rate]% open,       |
|   [reply_rate]% reply, [bounce_rate]% bounce     |
| Manual: [DMs sent] DMs, [other] other channels   |
+--------------------------------------------------+
| SCOREBOARD                                        |
| VOLUME | RESPONSE | BOOKING | SHOWUP | CLOSE | CASH
| [num]  | [num]    | [num]   | [num]  | [num] | $[amt]
| [+/-]  | [+/-]    | [+/-]   | [+/-]  | [+/-] | [+/-]
+--------------------------------------------------+
| Conversion rates (color-coded pills)              |
| [rate]% -> [rate]% -> [rate]% -> [rate]% -> [rate]%
+--------------------------------------------------+
| BOTTLENECK: [stage]                               |
| [explanation]                                     |
+--------------------------------------------------+
| THIS WEEK'S PRIORITIES                            |
| 1. [action item]                                  |
| 2. [action item]                                  |
| 3. [action item]                                  |
+--------------------------------------------------+
| CHANNEL BREAKDOWN                                 |
| Instantly:    [sent] sent, [resp] responses [rate]%
| LinkedIn DM:  [sent] sent, [resp] responses [rate]%
| Warm intro:   [sent] sent, [resp] responses [rate]%
+--------------------------------------------------+
| CASH: $[this week] this week / $[this month] MTD  |
+--------------------------------------------------+
| REVENUE PROGRESS: $[current] / $[goal] [bar] [%] |
+--------------------------------------------------+
| MILESTONES                                        |
| First client: [status]  $5K/mo: [status]          |
| $10K/mo: [status]  $30K MRR: [status]             |
+--------------------------------------------------+
```

### HTML requirements

- Dark background (#0A0A0A), light text (#E5E5E5)
- Sans-serif font stack (system fonts)
- Green (#22C55E) for healthy metrics, amber (#F59E0B) for borderline, red (#EF4444) for problems
- Conversion rate pills: small colored badges between each metric column
- Bottleneck section: highlighted with a red left border
- Action items: numbered list with checkbox styling
- Responsive: looks fine on a half-screen width (960px minimum)
- No external dependencies (no CDN links, no JavaScript frameworks)
- Self-contained single HTML file

### Save and open

1. Save the HTML file to `temp/mission-control-{YYYY-MM-DD}.html`
2. Open it in the default browser
3. Tell the user: "Dashboard is open. Put it on the right side of your screen, Claude Code on the left. When you see a red number, ask me about it."

---

## Customization: make it yours

This is YOUR dashboard. The layout above is the skeleton. You should add sections for whatever your business actually needs to track. Some ideas:

- **Client delivery status.** Track active projects, deadlines, and deliverables. Pull from a Supabase table you create.
- **Content calendar.** Upcoming videos, posts, or newsletters with dates and status.
- **Revenue projections.** Monthly recurring revenue, churn, lifetime value. Build the math yourself.
- **Ad spend tracker.** If you're running paid ads, add cost-per-lead and ROAS sections.
- **Custom KPIs.** Whatever number matters most to your business right now.

Tell Claude what you want to add. It reads your Supabase tables and Instantly data, builds the section, and drops it into the HTML. Every dashboard should look different because every business is different. If yours looks exactly like the template, you haven't made it yours yet.

---

## Handling edge cases

**Only one week of data:** Show the current week's numbers without week-over-week comparison. Replace the "Change" column with "N/A (first week)." Still calculate conversion rates and identify the bottleneck.

**Partial week:** If it's Tuesday and you only have 2 days of data, show what exists but note: "Partial week (2 of 7 days). Numbers will be more meaningful by Friday."

**Zero at a stage:** If any stage has zero entries, the conversion rate below it is 0%. But flag it differently: "No data at [stage] yet. This isn't a conversion problem, you haven't reached this stage. Focus on the stage before it."

**All metrics green:** Rare, but possible. Say: "Everything looks healthy. Keep your current volume and focus on consistency. Run /mission-control again Friday to confirm the trend holds." Give 3 action items focused on maintaining and incrementally improving.

**Instantly not connected:** Generate the dashboard without the Outreach section's Instantly data. Show manual tracking only. Add a note: "Connect Instantly MCP for email campaign analytics."

**User asks about a specific metric:** If $ARGUMENTS includes a question (e.g., "why is my response rate low"), skip the full dashboard generation and answer the specific question with data from Supabase and Instantly. Show the relevant numbers, compare channels, and give a specific recommendation.

---

## Self-check before finishing

1. All 6 Supabase tables queried successfully
2. Instantly MCP data pulled (or noted as unavailable)
3. This week and last week data pulled (or first-week mode noted)
4. All 5 conversion rates calculated
5. Conversion rates color-coded against correct benchmarks (including Instantly benchmarks if available)
6. Bottleneck identified using gap scoring (not just lowest rate)
7. Volume under 20 correctly overrides bottleneck detection
8. 3 action items are specific (verb + number + timeline), not generic
9. HTML dashboard generated, saved, and opened in browser
10. Outreach section includes Instantly data (if connected) and manual tracking
11. Scoreboard section shows all 6 metrics with week-over-week comparison
12. Channel breakdown combines Instantly and Supabase data
13. Revenue progress bar displayed with current/goal amount
14. Milestone statuses displayed (first client, $5K, $10K, $30K MRR)
15. Zero data projection mode works when tables are empty (does not stop)
16. No em dashes anywhere in the output or generated HTML

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Added MRR calculation to Step 3. MRR displayed as headline metric at top of dashboard with month-over-month trend. Uses close table churn columns (canceled, contract_type) to determine active clients. |
| 2026-03-31 | Concept update: Mission Control is now your personal admin dashboard. Added Instantly MCP as a data source (campaign open/reply/bounce rates). Added Outreach section to dashboard layout. Added Instantly-specific benchmarks and bottleneck detection. Added customization section: build your own sections (client delivery, content calendar, revenue projections, ad spend, custom KPIs). Updated channel breakdown to combine Instantly and Supabase data. |
| 2026-03-30 | Full rewrite. Added: prerequisite checks (Supabase connected, tables exist, data exists), week-over-week comparison with percent change, benchmark table with green/amber/red thresholds, bottleneck identification via gap scoring with volume override, 3 specific action items per bottleneck type with examples, full HTML dashboard spec with dark theme and layout, channel breakdown with per-channel response rates, edge case handling. Added zero data projection mode (generates projections from business.md when tables are empty instead of stopping), revenue progress bar, milestone projections. |
