---
name: outreach-engine
description: Daily multi-channel outreach system. Scrapes leads via Apify, sends cold email campaigns via Instantly, writes LinkedIn DMs and voice note scripts for manual send, and reports pipeline health through Mission Control.
argument-hint: "[optional: 'warm', 'full', or 'cold' to override auto-detect]"
user-invocable: true
---

# Outreach engine: daily multi-channel system

**Purpose:** This is the daily driver. Run it every morning. Claude scrapes leads, loads them into Instantly campaigns, writes your LinkedIn messages, checks yesterday's numbers, and tells you exactly what to do today. 10 minutes. Nothing sends without your approval.

Three systems work together:
1. **APIFY** (scrape) pulls fresh leads through MCP
2. **INSTANTLY** (send) runs cold email campaigns through MCP
3. **MISSION CONTROL** (analyze) shows your pipeline health via /mission-control

## Input

- $ARGUMENTS (optional): "warm", "full", or "cold" to override auto-detection
- Read `business.md` for: niche, offer, voice, pricing, guarantee, case studies, network size, outreach history, current mode. All business data comes from business.md. Do not restate it in header sections.
- Read `.claude/rules/humanizer.md` for writing rules
- Read outreach tracking files if they exist (pipeline, sent log, response log, Instantly campaign IDs)
- Read `contacts.md` for network size and lead quality
- MCP: Apify (lead scraping), Instantly (cold email campaigns, analytics), YouTube (comment mining)

---

## Step 1: Load context and detect mode

1. Read `business.md`
2. Read `.claude/rules/humanizer.md`
3. Read any existing outreach tracking files
4. Check Instantly MCP for active campaign stats (open rate, reply rate, bounce rate)
5. Determine mode:

**Auto-detection logic:**
- If contacts.md has <10 warm contacts: COLD/COMMUNITY MODE (skip warm entirely)
- If business.md shows week 1-2 or fewer than 70 total messages sent: WARM MODE
- If business.md shows week 3+ and 70+ messages sent: FULL MODE
- If $ARGUMENTS specifies "warm", "full", or "cold": override auto-detection

**Cold/Community mode (zero or tiny network):** Skip warm outreach entirely. Start with cold email via Instantly, community engagement, and YouTube comment mining from Day 1. You have nobody warm to message. Don't pretend you do. Pull leads from Apify scrapes and competitor YouTube comments.

**Warm mode (Week 1-2):** 10 warm messages per day. No cold. Conversations, not pitches.

**Full mode (Week 3+):** Multi-channel with graduated volume:
- Week 3: 10 warm + 20 cold = 30/day
- Week 4+: 10 warm + 30-40 cold = 40-50/day

**Email list broadcast (Day 1 action for large networks):** If you have an email list of 500+ people, the first outreach engine run should include a broadcast email to the hot segment (opened 3+ emails recently). Highest-ROI action on Day 1.

Display the detected mode, current streak, and Instantly campaign stats.

---

## Step 2: Check yesterday's results

Pull data from Instantly MCP (campaign analytics) and local outreach tracking files. Display:

```
--- YESTERDAY'S RESULTS ---
Sent: [count] messages ([warm count] warm, [cold count] cold via Instantly)
Opens: [count]/[cold count] cold emails opened ([percent]%) [from Instantly]
Replies: [count] ([positive count] positive, [neutral count] neutral, [negative count] negative)
Calls booked: [count]
Follow-ups due today: [count]
Instantly campaign health: [open rate]% open, [reply rate]% reply, [bounce rate]% bounce
```

If a positive response came in overnight (check Instantly MCP for new replies), flag it immediately. Draft a reply. Positive responses get priority over all new outreach.

If a question came in (pricing, details, etc.), draft a reply that moves toward a call booking.

---

## Step 3: Find fresh leads

### Apify MCP scraping (primary lead source for cold)

Use Apify MCP to scrape fresh prospects. Always run `fetch-actor-details` first to check input schema before calling any actor.

**Key actors by channel:**

| Actor | Use case | Cost |
|---|---|---|
| compass/crawler-google-places | Local businesses (dentists, restaurants, med spas) | ~$4/1K leads + email enrichment |
| harvestapi/linkedin-profile-scraper | B2B decision makers | ~$4/1K profiles, optional email |
| harvestapi/linkedin-post-comments | Commenters on niche-relevant posts | check actor details |
| apify/instagram-scraper | Service businesses on Instagram | check actor details |
| snipercoder/bulk-linkedin-email-finder | Email finding for LinkedIn leads | ~$1/1K emails |

**Scraping workflow:**
1. Run `fetch-actor-details` for the chosen actor
2. Configure input based on the niche from business.md
3. Run the actor via `call-actor`
4. Save results to `temp/{YYYY-MM-DD}_leads-{niche}.json`
5. Deduplicate against existing prospect files
6. Score each lead on niche fit (drop anyone below 5/10)
7. Pull 10-20 fresh qualified leads per session

**Warm mode lead sources:** Pull from your existing contact list, pipeline file, or the 5 prospects from /find-niche. If those are exhausted, suggest:
- Second-degree connections (ask existing contacts for intros)
- Community members (LinkedIn groups, Reddit, local meetups)
- Recent LinkedIn connections

**YouTube comment mining (all modes):** Use YouTube MCP to find prospects in competitor video comments. Search for comments on videos in your niche where people express intent: "I want to start," "how do I get clients," "is this worth it." These are self-identified prospects. Reply with something helpful (not a pitch), then DM anyone who engages back. Target: top 5 comments per day.

**Error handling:**
- If Apify MCP is not connected in full mode: "Apify MCP isn't connected. I can write messages for your existing pipeline, but I can't scrape new leads. Connect Apify for fresh prospects, or add leads manually to your pipeline file." Proceed with existing leads.
- If Instantly MCP is not connected: "Instantly MCP isn't connected. I'll write all messages as copy-paste ready. Connect Instantly to automate cold email campaigns." Proceed in manual mode.

---

## Standard prospect schema

All Apify actor outputs must be normalized to this schema before use. Different actors return different field names. Map them to this standard format so every downstream step (message writing, Instantly upload, scoreboard logging) works consistently.

| Field | Type | Notes |
|---|---|---|
| first_name | text | prospect's first name |
| last_name | text | prospect's last name |
| email | text | primary email address |
| company | text | business or company name |
| title | text | job title or role |
| source | text | which Apify actor or channel produced this lead |
| niche_fit_score | numeric (1-10) | how well they match the ideal client profile from business.md |
| observation | text | the specific thing you noticed about their business (feeds into Observation/Offer/Soft Ask) |
| custom_var_1 | text | flexible field for Instantly custom variables (e.g., Google rating, review count) |
| custom_var_2 | text | flexible field for Instantly custom variables (e.g., location, team size) |

**Mapping instructions for common actors:**

- **compass/crawler-google-places:** business name -> company, owner/contact -> first_name + last_name, email -> email, rating + review count -> custom_var_1 + custom_var_2
- **harvestapi/linkedin-profile-scraper:** firstName -> first_name, lastName -> last_name, headline -> title, companyName -> company, email (if enriched) -> email
- **harvestapi/linkedin-post-comments:** commenterName -> first_name + last_name, commenterHeadline -> title, commenterProfileUrl -> source
- **YouTube comment mining:** authorDisplayName -> first_name, channelId -> source, comment text -> observation

After scraping, run the normalization before writing any messages or uploading to Instantly.

---

## Step 4: Write personalized messages and load into channels

This is the multi-channel sequence. Each prospect gets touched across multiple channels over 14 days.

### Multi-channel sequence per prospect

| Day | Channel | Method | Who sends |
|---|---|---|---|
| Day 1 | LinkedIn | Profile view + connect request with note | You (manual, Claude writes the note) |
| Day 2 | Email | Email 1 via Instantly campaign | Instantly (auto) |
| Day 4 | LinkedIn | DM or voice note | You (manual, Claude writes the script) |
| Day 7 | Email | Email 2 follow-up via Instantly | Instantly (auto) |
| Day 14 | Email | Breakup email via Instantly | Instantly (auto) |

### Warm messages: ACA framework

For warm contacts (people you actually know or have interacted with):

- **Acknowledge:** Reference something specific about the person or your relationship
- **Connect:** Genuine observation connecting their situation to something relevant (their work, a shared experience, a recent activity)
- **Ask:** A question (not a pitch). "Got 5 minutes for a quick question?" or "Do you know anyone who struggles with [problem]?"

Each warm message: under 80 words, casual tone, sounds like a real person texting a contact. No banned words from humanizer.md.

### Cold emails: loaded into Instantly

For cold prospects scraped via Apify:

**Email 1 (Day 2):**
- **Observation:** Something specific about their business. A dentist with 4 new 1-star reviews. A restaurant that stopped responding to reviews. Concrete and verifiable.
- **Offer:** One sentence connecting your service to that observation.
- **Soft ask:** A question, not a pitch. "Worth 15 minutes this week?"

**Email 2 (Day 7):** Short follow-up. Reference Email 1. Add one new data point or angle. Same soft ask.

**Breakup email (Day 14):** Final touch. Brief, no pressure. "Figured this wasn't the right time. If [problem] comes back up, happy to chat."

Each cold email: under 100 words. Checked against humanizer rules. Must read like personal research went into it.

**Instantly campaign setup (via Instantly MCP):**
1. Create a campaign for the current batch (or add to existing niche campaign)
2. Upload leads with: email, first name, last name, company, custom variables (observation, offer angle)
3. Set the 3-email sequence with Day 2 / Day 7 / Day 14 timing
4. Verify deliverability settings are active (warmup, sending limits)
5. Activate the campaign after approval

### LinkedIn connect notes (Day 1, manual)

Short. 300 characters max (LinkedIn limit). Reference something specific. No pitch. Examples:

> "Hey [Name], saw your post about [topic]. Running into the same thing with my clients. Would be great to connect."

> "Hi [Name], noticed [specific thing about their business]. Interesting approach. Connecting in case it makes sense to swap notes."

### LinkedIn DMs and voice note scripts (Day 4, manual)

**DMs:** Slightly longer than connect notes. Follow the Observation/Offer/Soft Ask pattern but warmer since they already accepted the connection.

**Voice note scripts (top 2-3 prospects daily):** 30-second scripts. Voice notes hit 15-40% response rates. Script should sound natural, not rehearsed. Include what to say and what tone to use.

Example voice note script:
> "Hey [Name], this is [your name]. I was looking at [their business] and noticed [observation]. I actually help [niche] with [service], and I had a quick idea for you. No pressure at all, but if you've got 15 minutes this week I'd love to share it. Either way, cool stuff you're building."

---

## Step 5: Suggest engagement actions

Find 3-5 engagement opportunities:
- LinkedIn posts to comment on (from prospects or industry figures)
- Community threads to answer (Reddit, forums, Facebook groups)
- Google Q&A sections to respond in
- YouTube comments to reply to (from Step 3 mining)

These are visibility plays. Show up in someone's notifications before you DM them. That makes the DM land differently.

For each opportunity, draft a comment or response. Short, helpful, specific.

---

## Step 6: Present the full plan for approval, then execute

Display everything in one block:

```
=== OUTREACH ENGINE - [YYYY-MM-DD] ===
Day [N] streak | [total] messages sent | [responses] responses | [calls] calls booked

--- INSTANTLY CAMPAIGN STATS ---
Active campaigns: [count]
Open rate: [percent]% | Reply rate: [percent]% | Bounce rate: [percent]%
Leads in sequence: [count] | Completed sequences: [count]

--- RESPONSE DRAFTS ---
[If any responses came in (Instantly replies or LinkedIn), show drafted replies first]

--- FRESH LEADS ([count] found, [qualified count] qualified) ---
[List with: name, business, rating/signal, niche fit score, source (Apify actor used)]

--- MESSAGES READY ---
Cold emails for Instantly: [count] (will auto-create campaign and upload leads)
LinkedIn connect notes: [count] (manual, copy-paste ready)
LinkedIn DMs: [count] (manual, copy-paste ready)
Voice note scripts: [count] (top prospects)
Warm messages: [count] (ACA framework)

--- ENGAGEMENT ACTIONS ---
[3-5 opportunities with drafted comments]

--- TODAY'S PLAN ---
Reply to [count] responses (drafted above)
Upload [count] leads to Instantly campaign "[campaign name]"
Send [count] warm messages (manual)
Send [count] LinkedIn connect requests (manual)
Send [count] LinkedIn DMs / voice notes (manual)
Complete [count] engagement actions
Follow up with [count] from previous days

Total touches today: [count]
Streak: [N] days

Approve? Or adjust?
```

Wait for the response. Options:
- "Approved" (proceed: upload leads to Instantly, activate campaign, display manual tasks)
- Adjust specific messages ("change the cold emails to focus on med spas")
- Remove specific prospects
- Ask to regenerate any section

**On approval:**
1. Use Instantly MCP to create/update the campaign
2. Upload qualified leads with custom variables
3. Set the email sequence (3 emails over 14 days)
4. Activate the campaign
5. Display all manual tasks (LinkedIn connect notes, DMs, voice notes) as copy-paste ready with platform noted for each
6. Update local tracking files with today's activity

Nothing sends without explicit approval.

---

## Signal check (after 70 personal + 700 cold)

When you hit these thresholds, automatically run a signal check:

**Response rates:**
- LinkedIn DM (warm): 10%+ = messaging works. Under 5% = rewrite first line and soften the ask.
- Cold email (Instantly): 2%+ = solid. Under 1% = check deliverability in Instantly first (warmup status, SPF/DKIM/DMARC, bounce rate), then rewrite subject lines.
- Zero responses across both channels = fundamentally broken. Trigger emergency protocol.

**Conversion rates:**
- Booking rate (response to call): 20%+ = good. Below 20% = CTA too vague.
- Show-up rate: 70%+ = good. Below 60% = add confirmation + 24h reminder + 1h reminder.
- Close rate: 20%+ = good. Below 15% = discovery too shallow, recommend /close-deal prep-call mode.

Pull Instantly analytics via MCP for accurate cold email metrics. Display the signal check results and identify the single weakest metric.

---

## Emergency protocol (0 responses after 50+ messages)

If you have sent 50+ messages and received zero responses:

1. Tell yourself: "Don't pivot the niche. Not yet. Your niche is not the problem at 50 messages."
2. Rewrite 3 variations of the core message (change first line, offer, and ask).
3. Create 3 separate Instantly campaigns, one per variation, with 20 leads each. Track which gets responses.
4. Suggest switching channels (email not working via Instantly? Try LinkedIn DMs. LinkedIn not working? Try communities).
5. Check Instantly deliverability: warmup status, bounce rate, spam complaints. If bounce rate is above 5%, the list quality is the problem, not the messaging.
6. Get a second opinion. Share your messages with someone you trust.

If 100+ messages with revised messaging and still zero: NOW examine niche, offer, or targeting. But not before 100.

---

## Streak tracking

Track and display in every output:
- Current streak (consecutive days running /outreach-engine)
- Total messages sent (warm and cold separately)
- Total responses received
- Calls booked
- Deals closed (if tracked)
- Instantly campaign stats: open rate, reply rate, bounce rate (pulled from Instantly MCP)

Update the tracking data in business.md or a dedicated outreach log file after each session.

### Streak persistence

Save streak data to business.md after each session. Read it at start of next session. Format: `Outreach streak: [count] days, last run: [YYYY-MM-DD]`. This line goes at the bottom of business.md under a `## Outreach tracking` section. If the section doesn't exist, create it. At the start of every /outreach-engine run, read business.md and check the last run date. If yesterday, increment the streak. If today (already ran), keep the streak. If older than yesterday, reset to 1.

If the streak breaks (skipped day), note it but don't shame. Just show the reset: "Streak: 1 day (reset from [previous streak])."

---

## Instantly MCP integration

The Instantly MCP server provides direct access to campaign management. Key operations:

**Campaign management:**
- Create new campaigns per niche or lead batch
- Upload leads with custom variables (first name, company, observation, offer angle)
- Set email sequences with timing delays (Day 2, Day 7, Day 14)
- Activate/pause campaigns

**Analytics (pull daily in Step 2):**
- Open rate per campaign
- Reply rate per campaign
- Bounce rate per campaign
- Lead status (contacted, replied, bounced, unsubscribed)
- Sequence completion stats

**Deliverability monitoring:**
- Warmup status of sending accounts
- Bounce rate alerts (flag if above 5%)
- Spam complaint tracking

If Instantly MCP is not connected, present all cold emails as copy-paste ready and tell the user: "Connect Instantly MCP to automate your cold email campaigns. Right now I'm writing everything for manual send."

---

## Gmail integration (optional, secondary)

If Gmail MCP is connected:
- Can send warm emails directly after approval
- Can check for responses automatically
- Tell the user which messages were sent via Gmail vs. which they need to send manually (LinkedIn DMs, text messages)

Gmail is for warm outreach only. Cold email goes through Instantly (better deliverability, built-in warmup, sequence automation).

If Gmail MCP is not connected, present warm emails as copy-paste ready with the platform noted for each.

---

## Self-check before finishing

1. Correct mode detected and displayed (warm, full, or cold/community)
2. Yesterday's results shown with Instantly campaign stats (or "Day 1, no prior data" if first run)
3. Fresh leads found and scored (warm mode: from pipeline, full mode: from Apify, cold/community mode: from scrapes + YouTube comments)
4. All messages follow the correct framework (ACA for warm, Observation/Offer/Soft Ask for cold)
5. Cold emails structured for Instantly upload (campaign name, lead data, sequence timing, custom variables)
6. Multi-channel sequence clear: Day 1 LinkedIn connect, Day 2 Email 1, Day 4 LinkedIn DM/voice note, Day 7 Email 2, Day 14 Breakup
7. Voice note scripts generated for top 2-3 prospects
8. Engagement actions suggested with drafted comments
9. Full plan presented for approval before any action
10. On approval: leads uploaded to Instantly, campaign activated, manual tasks displayed
11. Streak and totals displayed with Instantly analytics
12. All messages checked against humanizer rules (no banned words, no generic templates, no em dashes)
13. Zero network path correctly triggered if contacts.md has <10 warm contacts
14. Business.md data (pricing, guarantee, case studies) used naturally in messages but not restated in headers
15. Apify actors called with `fetch-actor-details` first, results saved to `temp/` with date prefix

---

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Added standard prospect schema (10 fields) with mapping instructions for common Apify actors. Added streak persistence: save streak data to business.md after each session, read at start of next session. |
| 2026-03-31 | Three-part system: Apify (scrape via MCP), Instantly (send via MCP), Mission Control (analyze via /mission-control). Added: multi-channel sequence per prospect (Day 1 LinkedIn connect, Day 2 Email 1 via Instantly, Day 4 LinkedIn DM/voice note, Day 7 Email 2, Day 14 Breakup), Instantly MCP integration for campaign creation, lead upload, sequence automation, and analytics, specific Apify actors with costs, deliverability monitoring via Instantly, A/B testing via separate Instantly campaigns in emergency protocol. Kept: ACA framework for warm, Observation/Offer/Soft Ask for cold, streak tracking, signal check thresholds, emergency protocol logic, mode auto-detection, approval flow. |
| 2026-03-30 | Full rewrite. Added two modes with auto-detection, ACA framework, Observation/Offer/Soft Ask, voice note scripts, graduated volume ramp, signal check, emergency protocol, streak tracking, engagement actions, approval flow, Gmail MCP integration. Added cold/community mode, YouTube comment mining, email list broadcast. |
