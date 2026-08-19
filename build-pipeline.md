---
name: build-pipeline
description: Refreshes warm network from Week 1-2, scrapes cold leads via Apify (Google Maps and LinkedIn), activates cold email infrastructure via Instantly MCP, generates multi-channel outreach sequences using Observation/Offer/Soft Ask framework, and scales to 30-50 daily touches.
user-invocable: true
---

# /build-pipeline

Scale your outreach pipeline. Refresh warm network, scrape cold leads, activate cold email, write first messages, and hit 30-50 touches per day.

## Input

- Read `business.md` for: niche, ideal client profile, delivery model, offer details, price, guarantee, case studies, competitor data, location (if local business). All pricing, proof, and guarantee language comes from business.md. Do not restate business.md content in strategy sections.
- Read `outreach/` folder for existing warm network map and any previous prospect files
- Read `contacts.md` for warm network size and lead quality
- If business.md is missing niche or ideal client, stop and tell the user to run `/find-niche` first

### MCP connection check

Before running the pipeline, verify the user has these MCP servers connected:

1. **Apify MCP** (for scraping cold leads). If not connected, warn the user and skip Step 3.
2. **Instantly MCP** (for cold email campaigns). Connect at `mcp.instantly.ai`. You need your API key from the Instantly dashboard (Settings > Integrations > API). If not connected, Step 4 still works but you'll manage campaigns manually through the Instantly web dashboard.
3. **Supabase MCP** (for scoreboard and tracking). If not connected, skip scoreboard updates but continue with the pipeline.
4. **YouTube MCP** (optional, for comment mining). Only needed if you want to mine competitor video comments for prospects.

If any required MCP is missing, tell the user exactly what to add and where to find the credentials. Don't just skip silently.

### Zero network detection

Check contacts.md for network size:
- **LARGE (50+ warm contacts):** Run Steps 1-6 as written.
- **SMALL (<50 contacts) or contacts.md has only cold leads:** Skip Step 1 (warm outreach). Go directly to cold email + community engagement + Apify scrape. You have nobody warm to message. Don't pretend you do.
- **ZERO (no contacts.md or empty):** Skip Steps 1-2 entirely. Start with Step 3 (Apify cold lead scrape) + community engagement plan. Use YouTube comment mining as a lead source (see Step 3).

## Step 1: Refresh and expand warm network

You mapped your warm network in Week 1. Now it needs refreshing with 2 weeks of new activity.

Ask these questions:

1. "Who replied to your Week 1-2 outreach messages? List anyone who responded, even if they said no."
2. "Did you run any free pilots? If so, who? These are your strongest referral sources now."
3. "Did anyone mention other people you should talk to? ('You should talk to my friend Dave' = warm lead with intro path)"
4. "Any new contacts from communities, events, or LinkedIn engagement?"

Combine answers with the existing warm network map (if it exists in `outreach/`).

### Score every contact

Score each person into one of three buckets:

- **Direct prospect:** They fit the ideal client profile from business.md. Message them first.
- **Warm referral source:** They don't need your service, but they know people who do. Ask for intros.
- **Not relevant right now:** Park them. Don't spend outreach energy here.

For each contact, recommend a channel: email, LinkedIn DM, text, voice note, or phone call. Base this on the relationship (close friend = text, professional contact = LinkedIn, former colleague = email).

### Weave in proof assets

If you have testimonials or pilot results from Week 1-2, note which warm messages should include them. A message to a direct prospect that includes "I helped Dr. Torres go from 52 to 180 Google reviews in 90 days" converts differently than one without proof.

Save refreshed network to: `outreach/network-map-refreshed.md`

## Step 2: Small network expansion

Check the contact count. If you have fewer than 30 contacts after the refresh:

Generate an accelerated community-building plan:

1. **Identify 5 online communities** where your target clients spend time (Facebook groups, Slack channels, Discord servers, Reddit, niche forums). Be specific based on your niche from business.md.

2. **Create a 7-day engagement plan:**
   - Days 1-3: Comment on posts, answer questions, contribute value. Zero self-promotion.
   - Days 4-5: Start conversations. DM people whose problems match your offer.
   - Days 6-7: Share a quick win or insight from your work. Soft DMs to interested people.

3. **Suggest 3 local networking events or meetups** relevant to the niche and location (if location is in business.md). Check for industry meetups, chamber of commerce events, BNI groups.

4. **Build a LinkedIn daily engagement list:** 10 profiles to engage with daily (comment on their posts, react, start conversations). Pick people who are either prospects or connectors.

If you have 30+ contacts, skip this step and move to Step 3.

Save expansion plan to: `outreach/small-network-plan.md` (only if triggered)

## Step 3: Scrape cold leads via Apify

Connect to Apify through MCP and pull live prospect data. Run `fetch-actor-details` first to check the input schema before calling any actor.

### Source selection based on niche

**Google Maps** (for local businesses): dentists, HVAC, fitness studios, restaurants, salons, clinics, anything with a physical location and Google listing. Use the Google Maps scraper actor.

**LinkedIn** (for B2B services): agencies, consultants, SaaS founders, marketing directors, anyone with a LinkedIn profile and a business problem. Use the LinkedIn scraper actor.

**Online communities** (for creator/membership niches): YouTube comment scrapers, Reddit scrapers, Discord/Skool community members. For niches where prospects don't have Google listings or LinkedIn profiles but ARE active in online communities. Use Apify actors for YouTube comment scraping (find people commenting on competitor videos with intent signals like "I want to start" or "how do I get clients"), Reddit post/comment scrapers (find people asking for help in niche subreddits), and community member scrapers where available. These prospects self-identify by the questions they ask and the communities they join.

Choose the source based on the niche in business.md. If the niche is local (e.g., "dental clinics in Austin"), use Google Maps. If B2B (e.g., "SaaS companies with 10-50 employees"), use LinkedIn. If the niche is creators, coaches, or membership businesses where prospects live in online communities, use the online communities path. If multiple apply, scrape multiple sources.

### YouTube comment mining (additional lead source)

If you have a YouTube channel (even small), mine competitor video comments for prospects. Use YouTube MCP to find comments on competitor videos where people say things like "I want to start" or "how do I get clients." These are direct prospects who already raised their hand. Reply to the top 5 per day with something helpful (not a pitch), then DM the ones who engage back.

### Email list segmentation (if you have an email list)

If you have an existing email list (even 100 people), segment it before blasting:
- **Hot (opened 3+ emails in last 30 days):** Message first. These people are paying attention.
- **Warm (opened at least 1 email in last 30 days):** Second wave. They know you but aren't engaged daily.
- **Cold (haven't opened in 30+ days):** Re-engagement sequence first, then pitch. Don't waste the list by pitching cold subscribers who forgot who you are.

### Data collected per prospect

Google Maps: business name, contact person, email, phone, website, location, Google rating, review count

LinkedIn: name, headline, company, company size, location, profile URL

### Scoring

Score every prospect on niche fit using the ideal client profile from business.md. The score reflects how much this prospect needs your service, not how successful they already are.

Example: A dental clinic with 42 reviews and a 3.8 star rating scores HIGHER than one with 500 reviews and 4.9 stars. The first one actually needs help.

Scoring criteria (weight by niche):
- Matches ideal client profile description (high weight)
- Shows signals of needing help: low review count, low rating, outdated website, small team (medium weight)
- Location match if local service (medium weight)
- Contact info available: email or phone found (low weight, but required for cold outreach)

### Cost note

A typical run costs about $3.50 for 200 leads. Scrape 200 businesses from Google Maps (~$1.50) + find contact info from websites (~$2.00). Monthly Apify spend for cold outreach is $4 to $10 depending on volume.

Save results to: `outreach/prospects-[YYYY-MM-DD].json`

## Step 4: Activate cold email infrastructure via Instantly MCP

Check the cold email setup. Ask: "Did you set up your Instantly accounts during Week 1 setup? Are your email accounts warming up?"

### If warmup is running (started 2+ weeks ago):

Accounts should be ready. Present the ramp schedule:

- Week 3: 5-10 cold emails per day (alongside continued warmup)
- Week 4: 15-20 cold emails per day
- Week 5+: 30-40 cold emails per day per address

### If warmup is NOT running:

Walk through the setup:
1. Buy a secondary domain (never send cold outreach from your primary domain)
2. Set up 2-3 email accounts on the secondary domain
3. Configure DNS authentication (SPF, DKIM, DMARC)
4. Start warmup in Instantly (takes 2-3 weeks before sending)
5. Note: Instantly costs $30/month for the Growth plan (5,000 active leads, unlimited email accounts)

### Instantly MCP connection

Instantly has an official MCP server at `mcp.instantly.ai`. Once connected, Claude Code can:

- **Create campaigns** directly from scraped lead lists
- **Upload leads** from Apify scrape results without manual CSV export
- **Check analytics** (open rates, reply rates, bounce rates) in real time
- **Manage sequences** (pause, resume, edit steps) without switching to the web dashboard
- **Pull campaign reports** for the daily outreach summary

To connect: add the Instantly MCP server during `/build-pipeline` setup. You need your API key from the Instantly dashboard (Settings > Integrations > API). Once connected, the pipeline runs end-to-end inside Claude Code: scrape leads with Apify, push them into Instantly campaigns, and check results, all without leaving the terminal.

If the Instantly MCP is not connected, you manage campaigns through the Instantly web dashboard manually. Claude Code still generates email copy and prospect lists, but you handle the upload and campaign creation yourself.

Log the ramp status and next action in the output.

## Step 5: Write first outreach messages (multi-channel sequences)

Generate the first batch of outreach messages. Apply humanizer rules from `.claude/rules/humanizer.md` to every message. No Tier 1/Tier 2 banned words. No banned phrases.

### Framework: Observation / Offer / Soft Ask

Every cold message follows this structure:

1. **Observation:** Something specific about their business that proves you looked. Not "I love your business." More like "I noticed your Google rating dropped from 4.6 to 4.2 since January." Pull real data from the scrape results where possible.

2. **Offer:** One sentence connecting your service to that observation.

3. **Soft ask:** A question, not a pitch. "Worth 15 minutes this week?" or "Want me to show you how?"

### Multi-channel outreach sequence

Cold prospects don't respond to a single touch on a single channel. Set up a 4-touch sequence that crosses platforms:

**Touch 1 (Day 1): LinkedIn connection request**
Send a personalized connection request. No pitch. Just the Observation line and a reason to connect. Example: "Saw your clinic's Google listing, looks like you're growing fast in [city]. Would love to connect."

**Touch 2 (Day 3): Cold email via Instantly**
First email follows the full Observation/Offer/Soft Ask framework. If Instantly MCP is connected, Claude Code creates the campaign and uploads the leads directly.

**Touch 3 (Day 6): LinkedIn DM**
If they accepted the connection request, send a short DM. Reference the email if you want ("sent you a quick note earlier"). Keep it to 2-3 sentences. Voice notes work here too (30-40% response rates).

**Touch 4 (Day 9): Follow-up email via Instantly**
Different angle from Touch 2. New observation, same offer. If Instantly MCP is connected, this is the second step in the Instantly sequence and fires automatically.

If a prospect responds at any point, stop the sequence and have a real conversation. Don't keep blasting someone who already replied.

### Message types to generate

**Cold email templates (3-5 variations):** Each one follows Observation/Offer/Soft Ask but with different observations and angles. Rotate these in Instantly.

**LinkedIn connection request templates (2-3 variations):** Short, specific, no pitch. Just enough to get accepted.

**LinkedIn DM templates (2-3 variations):** For after the connection is accepted. Warmer tone than cold email.

**Warm message templates (2-3 variations):** Adjusted tone for people you already know. No business pitch for your college friend. A catch-up message that naturally leads to what you're building.

**Voice note scripts (2 variations):** 30 seconds max. Reference something specific about the prospect. Voice notes get 30-40% response rates.

**Loom video outlines (1 template):** 60 seconds. Walk through the prospect's website or Google listing, point out improvements you'd make. Loom videos hit 15-20% response rates.

**Required lines in cold email templates:** Every cold email template MUST include the guarantee line and the differentiator that makes your offer distinct. Pull the exact guarantee from business.md (e.g., "5 clients in 90 days or full refund"). Include a one-line differentiator that separates you from the pack ("one tool, not twenty platforms" for automation offers). These two lines belong in the Offer section of the Observation/Offer/Soft Ask framework. The guarantee removes risk. The differentiator sparks curiosity.

Each message must read like you spent 5 minutes researching the person. No generic templates. Personalization placeholders where you fill in prospect-specific details: [business name], [specific observation], [specific metric].

## Step 6: Scale to 30-50 daily touches

Present the complete daily outreach plan:

### Daily breakdown

- **Warm messages (10/day):** Refreshed network contacts, referral asks, pilot follow-ups. These convert at 15-25%. Never stop warm outreach.
- **Cold emails via Instantly (20-40/day):** Scraped leads from Step 3, loaded into Instantly (via MCP or manual upload). Convert at 2-3% without proof, 5-8% with testimonials and case studies.
- **LinkedIn touches (5-10/day):** Connection requests + DMs as part of the multi-channel sequence. These warm up the cold email and boost reply rates.
- **Creative touches (3-5/day):** Voice notes, Loom videos, YouTube comment engagement. Voice notes hit 30-40% response rates.

**Combined target: 30-50 outreach touches per day.**

### Proof integration

If you have testimonials or case studies from Week 1-2 pilots, weave them into cold outreach. A cold email from someone with zero clients converts at 1%. A cold email with a specific result ("helped Dr. Torres go from 52 to 180 Google reviews in 90 days") converts at 5-8%.

Show exactly where to insert proof in each message template.

Save the daily plan to: `outreach/daily-outreach-plan.md`

## How to run it

1. Make sure MCP servers are connected (Apify, Instantly, Supabase, and optionally YouTube). Run `/build-pipeline` and it will check for you.
2. Have `business.md` and `contacts.md` ready. If either is missing or incomplete, the skill will tell you what to fix first.
3. The skill runs Steps 1-6 in order, skipping steps based on network size and setup status.
4. When the Instantly MCP is connected, campaigns get created directly from the pipeline output. No copy-pasting into dashboards.
5. Run `/outreach-engine` daily after initial setup to execute the plan and track progress.

## What to do after

- **Daily:** Run `/outreach-engine` to execute your daily 30-50 touches. It pulls fresh analytics from Instantly MCP, updates your scoreboard in Supabase, and tells you exactly who to message today.
- **Weekly:** Re-run `/build-pipeline` to refresh leads and adjust the sequence based on what's converting. If reply rates drop below 5%, the skill flags it and suggests copy changes.
- **When you land a client:** Update `business.md` with the new case study. The pipeline automatically works it into future outreach messages.

## Output

Print a summary in this format:

```
=== PIPELINE SCALE COMPLETE ===
Date: [YYYY-MM-DD]

MCP STATUS:
  Apify: [Connected / Not connected]
  Instantly: [Connected via MCP / Manual mode]
  Supabase: [Connected / Not connected]
  YouTube: [Connected / Not connected / Skipped]

WARM NETWORK (refreshed):
  Total contacts: [N] (up from [N] in Week 1)
  New since Week 1: [N] (referrals, community, pilots)
  Direct prospects: [N]
  Warm referral sources: [N]
  Pilot clients with testimonials: [N]
  Saved to: outreach/network-map-refreshed.md

[If small network expansion triggered:]
NETWORK EXPANSION:
  7-day engagement plan created
  Communities identified: [N]
  LinkedIn daily engagement list: 10 profiles
  Saved to: outreach/small-network-plan.md

COLD LEADS:
  Source: [Google Maps / LinkedIn / Both] ([location/niche])
  Scraped: [N] businesses
  After scoring: [N] qualified ([threshold] niche fit)
  Saved to: outreach/prospects-[date].json

COLD EMAIL STATUS:
  [If ready:] Secondary domain: [domain], warmup complete, ready to send
  [If not ready:] Setup instructions provided, warmup starts now (ready in 2-3 weeks)
  [If Instantly MCP connected:] Campaign created: [campaign name], [N] leads uploaded
  [If manual mode:] Prospect list exported, upload to Instantly dashboard
  Ramp plan: [current]/day this week, [next]/day next week, [target]/day week 3

MULTI-CHANNEL SEQUENCE:
  Touch 1 (Day 1): LinkedIn connection request
  Touch 2 (Day 3): Cold email via Instantly
  Touch 3 (Day 6): LinkedIn DM
  Touch 4 (Day 9): Follow-up email via Instantly

FIRST MESSAGES:
  Cold email templates: [N] variations
  LinkedIn connection templates: [N] variations
  LinkedIn DM templates: [N] variations
  Warm message templates: [N] variations
  Voice note scripts: [N]
  Loom video outline: 1

DAILY OUTREACH PLAN:
  Warm messages: 10/day
  Cold emails via Instantly: [N]/day (ramping to 30-40)
  LinkedIn touches: 5-10/day
  Creative touches: 3-5/day
  Total: [N]/day, scaling to 50/day by week [N]

[If proof assets exist:]
PROOF ASSETS:
  [testimonial or case study summary]
  Integrated into [N] message templates

Next step: Run /outreach-engine daily to execute this plan.
```

## Error handling

- business.md missing or empty: Stop. Tell user to run `/find-niche` first.
- business.md missing niche or ideal client: Stop. Cannot scrape leads without a target.
- Apify MCP not connected: Skip Step 3 (cold lead scrape). Proceed with warm network refresh and message generation. Remind user to connect Apify to unlock cold leads.
- Instantly MCP not connected: Fall back to manual mode. Generate email copy and prospect lists, but user handles campaign creation in the Instantly web dashboard. Don't block the pipeline.
- Supabase MCP not connected: Skip scoreboard updates. Everything else runs normally.
- No existing warm network map: Start fresh. Ask all warm network questions from scratch.
- User has zero contacts: Trigger Step 2 (small network expansion) automatically. Focus entirely on community engagement and LinkedIn warm-up.
- Instantly not set up: Provide setup instructions. Generate the message templates anyway (usable manually via email or LinkedIn).
- Apify scrape returns zero results: Check search terms and location. Try broader search. If still zero, suggest manually finding 20 prospects and entering them.

## Self-check (run before delivering)

1. Warm network is refreshed with Week 1-2 activity (not just the original list)
2. Every contact is scored into one of 3 buckets (direct prospect, warm referral, not relevant)
3. Cold leads are scored on niche fit, not business success
4. All message templates follow Observation/Offer/Soft Ask framework
5. Multi-channel sequence has 4 touches across LinkedIn and email
6. Zero Tier 1/Tier 2 banned words in any message copy
7. Zero banned phrases in any message copy
8. Voice note scripts are under 30 seconds when read aloud
9. Daily outreach plan adds up to 30-50 touches
10. Proof assets (if they exist) are integrated into message templates
11. All files saved to `outreach/` directory with correct names
12. Zero network path correctly triggered if contacts.md is SMALL or ZERO
13. Email list segmented (hot/warm/cold) if you have one
14. Business.md data (pricing, guarantee, case studies) referenced, not restated in strategy sections
15. MCP connection status reported in output summary
16. Instantly MCP integration tested (campaign creation, lead upload) if connected

---

## Change log

- **2026-08-19:** Sanitized and republished under NiteOps brand.
- **2026-03-31:** Added online communities as third niche path in Step 3 (YouTube comments, Reddit, Discord/Skool) for creator/membership niches where prospects lack Google listings or LinkedIn profiles. Added requirement in Step 5 that cold email templates MUST include the guarantee line and a one-line differentiator.
- **2026-03-31:** Major update. Added Instantly MCP server integration (mcp.instantly.ai) so Claude Code can create campaigns, upload leads, and check analytics directly. Added MCP connection check section (Apify, Instantly, Supabase, YouTube). Added multi-channel outreach sequence (LinkedIn connect > Email via Instantly > LinkedIn DM > Follow-up via Instantly). Added "How to run it" and "What to do after" sections.
