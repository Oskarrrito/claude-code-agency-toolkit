---
name: find-niche
description: Interactive niche finder. 4 warm-up questions, network assessment, 5-minute niche test, 3-source data collection (Reddit + YouTube comments + YouTube search suggestions), Perplexity validation (scored /20), outputs 4 MVP Lines and first 5 warm messages. Saves everything to business.md.
argument-hint: "[optional: rough niche idea, e.g. 'dental clinics Austin' or 'AI service business']"
user-invocable: true
---

# Find and validate your niche

**Purpose:** Walk you through niche selection with real data. This is the first skill I run. It writes the 4 MVP Lines and network strategy to business.md, which every other skill reads from.

**Required tools:** Apify MCP (Reddit scraping), YouTube MCP (comment scraping + search suggestions), Perplexity MCP (market validation)

## Input

- $ARGUMENTS (optional): a rough niche idea or blank
- business.md: read if it exists (may have prior context)

## Step 1: Load context and check for shortcut

1. Read `business.md` if it exists (may contain background from setup)
2. Read `.claude/rules/humanizer.md` for writing rules
3. If $ARGUMENTS contains a niche idea, note it but still run the full Q&A

**Shortcut check:** If business.md already has a niche defined (look for "My niche" or "4 MVP Lines" sections), tell the user: "You already have a niche in business.md: '[niche from file]'. Want to skip straight to validation (Step 5), or start fresh?" If they choose to skip, jump to Step 5 with the existing niche.

## Step 2: Four warm-up questions (interactive, one at a time)

Ask these one at a time. Wait for each answer before asking the next. Do not batch them.

1. "What work have you done in the last 5 years? Not job titles. What did you actually DO?"
2. "Who did you help, and what specific problem did you solve?"
3. "What result did they get? Numbers if you have them."
4. "If you charged for that, what would someone pay?"

Store all four answers. Use them to draft a candidate niche statement.

If their description sounds like someone stuck in learning mode instead of shipping, call it out plainly: "That sounds like learning-mode territory. Let's figure out who you actually helped, not what you learned."

## Step 3: Network size assessment

Ask: "How many people do you know professionally? Rough number is fine."

Branch based on answer:

- **200+ contacts:** Label as LARGE network. Strategy: first clients come from people you already know. Bias niche selection toward where the network is strongest. Referrers count (past clients, colleagues who know 10+ potential buyers).
- **50-200 contacts:** Label as MEDIUM network. Strategy: mix of warm and cold. Bias niche toward where at least some contacts are potential buyers or referrers.
- **Under 50 contacts:** Label as SMALL network. Strategy: cold outreach is the main channel. Bias niche toward prospects that are easy to find (local businesses on Google Maps, active LinkedIn communities, industry directories).

Save the network size label and strategy to use in later steps.

## Step 4: Draft niche statement and 5-minute niche test

From the warm-up answers plus network assessment, draft a candidate niche statement in one sentence (e.g., "I serve independent dental clinics doing $500K-$2M in annual revenue" or "I serve solo founders who know AI tools but have zero clients").

Then run the 5-minute niche test. Two versions based on network size:

- **Large or medium network:** "Name 5 real people you could text right now who either (a) need this themselves, OR (b) know 10+ people who do. First names and how you know them." (Referrers count for large networks. A past client who can send you 5 warm intros is as good as a direct prospect.)
- **Small network:** "Find 5 real businesses matching this niche on Google Maps or LinkedIn right now. Give me names."

Wait for their answer.

**If they cannot name or find 5:** Tell them the niche is too vague. Help them narrow it. Ask clarifying questions. Repeat the test until they can name 5.

**If they can name 5:** Those 5 become the first warm outreach targets. Save them.

## Step 5: 3-source data collection

This step collects real market data from three sources. Pick which sources to use based on niche type:

**For online niches** (creators, SaaS founders, AI enthusiasts, digital services): Reddit + YouTube comments are primary. Google Maps is for local niches only.

**For local niches** (dental clinics, restaurants, real estate): Google Maps is primary. Reddit is secondary.

### Source 1: Reddit scrape (Apify)

Run `fetch-actor-details` first to check the input schema before calling any actor.

**Scraper:** Use `parseforge/reddit-posts-scraper` or equivalent Reddit scraper.

**When using Reddit, always specify target subreddits. Global search returns noise.** If the first scrape uses broad search terms and returns irrelevant posts, document the failure in the output AND the fix. You learn from the process, not just the result.

For each niche type, target specific subreddits:

| Niche type | Target subreddits |
|---|---|
| AI services / automation | r/ClaudeAI, r/SaaS, r/EntrepreneurRideAlong, r/Entrepreneur |
| Web development / design | r/webdev, r/freelance, r/web_design |
| Marketing / ads | r/PPC, r/digital_marketing, r/marketing |
| Local business services | r/smallbusiness, r/sweatystartup |
| E-commerce | r/ecommerce, r/shopify, r/FulfillmentByAmazon |

Settings: pull 20-50 results from 2-3 target subreddits. Sort by top/hot.

**What to look for in Reddit data:**
- Posts with high upvotes showing the problem your niche solves
- Comments expressing frustration, confusion, or "I wish someone would just do this for me"
- Price points mentioned (what people pay, what people charge)
- Signals that people are stuck learning instead of shipping (asking "how do I start" for the 50th time)

### Source 2: YouTube comment scraping (YouTube MCP)

**Tool:** `mcp__youtube__youtube_list_comments`

This is a primary data source for online niches. YouTube comments are the highest-signal prospect data because every comment expressing a pain point is a person who self-identified as a potential customer.

**How to run it:**
1. Search for 2-3 competitor videos in your niche using `mcp__youtube__youtube_search` (pick videos with 50K+ views and lots of comments)
2. Scrape 20-30 comments from each video using `mcp__youtube__youtube_list_comments` with the video ID
3. Pull the commenter's username, comment text, and like count

**What to look for in comments:**
- People stuck learning without shipping ("I need to stop procrastinating," "great tutorial, now I just need to start")
- Pain points stated directly ("I still can't figure out how to get clients")
- Fear of commoditization ("clients will expect this for $100 now")
- Overwhelm behavior ("haven't finished the last video and you already released a new one")

**In the output, quote real comments with usernames.** Add analysis in parentheses after each quote explaining why this person is (or represents) a prospect. This is the strongest validation signal.

### Source 3: YouTube search suggestions (YouTube MCP)

**Tool:** `mcp__youtube__youtube_search_suggestions`

Run search suggestions for 3 keywords related to your niche. This shows what people actually search for on YouTube.

**How to interpret results:**
- High suggestion count (5+ suggestions) = active demand, people searching for this
- Low suggestion count (1-2 suggestions) = demand gap, nobody is teaching this yet
- Compare "learning" keywords vs "selling/getting clients" keywords. If learning keywords have 6+ suggestions but selling keywords have 1-2, that's the classic gap: people search for learning, not for selling. Your content fills what nobody is addressing.

### Source selection for local niches

| Niche type | Scraper | What to pull |
|---|---|---|
| Local businesses (dental, restaurant, etc.) | Google Maps scraper (Apify) | Names, addresses, websites, ratings, review counts, phone numbers |
| E-commerce | Shopify store scraper or similar | Store names, products, traffic estimates |
| B2B services | LinkedIn scraper | Decision-makers, company size, recent activity |

Settings: pull 20-50 results. Filter to your target geography if applicable.

### Save scrape data

Save all raw results to `temp/{YYYY-MM-DD}_niche-prospects-{niche-keyword}.json`.

### Error handling

If a tool is not connected, say which one is missing and what it costs you. Proceed with whatever sources are available.

- **Apify MCP not connected:** "Apify MCP isn't connected. I can't pull Reddit data. Connect it and re-run this step, or I'll validate with YouTube data plus Perplexity only."
- **YouTube MCP not connected:** "YouTube MCP isn't connected. I can't scrape competitor comments or search suggestions. That's a big miss for online niches. Connect it and re-run, or continue with Reddit plus Perplexity."
- **Both missing:** "Neither Apify nor YouTube MCP is connected. I'll validate with Perplexity only, but the data will be weaker. Connect at least one and re-run for real prospect data."

## Step 6: Perplexity validation (3 queries)

Use Perplexity MCP to research the niche. Split into two queries for better results:

**Query 1 (Market size + demand):** Ask about market size, growth rate, search volume, and whether people are actively looking for solutions in this niche.

**Query 2 (Competitor deep dive):** Research the top 5-10 competitors in this niche. For each: what they sell, price point, platform, member/student count, core positioning, and their weakness. Ask specifically about their pricing, delivery model, what tools they teach, and what gaps exist. This is not a surface scan. Dig into what they actually offer vs what they promise.

**Query 3 (Competitor positioning gaps):** Based on the competitors found in Query 2, ask: "What do none of these competitors offer? What complaints do their students have? Where is the gap in this market?"

Score each dimension 1-5:

1. **Demand:** Is there active search volume? Are people posting about this problem? Are communities talking about it? Use Reddit data and YouTube search suggestions to support the score.
2. **Competition:** How many competitors exist? What do they charge? What platform are they on? Is there a gap? Use Perplexity competitor data and platform pricing to support the score.
3. **Willingness to pay:** Do these people already buy services or courses? What's the existing budget range? Use Reddit mentions of spending and competitor pricing as anchors.
4. **Reachability:** Can you find and contact prospects easily? Are they on YouTube, Reddit, LinkedIn, Google Maps, in directories? Use the scrape results to support the score.

Total out of 20.

**Scoring bands:**
- 17-20: GO. Lock the niche.
- 13-16: MAYBE. Sharpen the angle (e.g., "dental clinics" scoring 13 might become "dental clinics with under 50 Google reviews" scoring 17). Re-score after sharpening.
- 10-12: WEAK. Suggest pivoting the angle or narrowing further.
- Below 10: NO. Go back to Step 2 and try a different direction.

**Error handling:** If Perplexity MCP is not connected: "Perplexity MCP isn't connected. I'll score based on the Reddit and YouTube data we already collected plus what I know. Scores will be less reliable. Connect Perplexity MCP for real market data." Provide estimates and flag them as estimates.

## Step 7: Lock the 4 MVP Lines

Once validation passes (score 13+), write the 4 MVP Lines.

Use the user's own language. Keep the framing plain and specific.

```
Line 1 (Niche): I serve [specific target audience] who [specific situation or pain].
Line 2 (Offer): I give them [specific deliverable/system] so they [specific measurable result].
Line 3 (Price): $[amount]/month. [One sentence ROI math showing why it's worth it.]
Line 4 (Delivery): Week 1: [first action]. Week 2: [second action]. Week 3: [first result]. Ongoing: [maintenance].
```

Review the lines with the user. Let them edit anything that doesn't sound like them. Once approved, these are locked.

## Step 8: Generate first 5 warm messages

Using the 5 people from the niche test (Step 4), generate a personalized conversation-starter message for each one.

Message format: casual, personal, under 80 words. NOT a sales pitch. A conversation opener. Use the user's natural voice, not corporate speak.

For each message, specify:
- Recipient name and relationship
- Suggested channel (text, LinkedIn DM, email, Instagram DM)
- The message text

Vary the messages based on relationship type:
- **Past client/student:** Reference their specific result, ask for referral or founding member status
- **Peer/colleague:** Reference shared experience, ask for feedback on the concept
- **Cold-ish contact (YouTube commenter, forum poster):** Reference the specific thing they said, offer the solution directly
- **Referrer (someone who knows the target audience):** Ask if they know anyone who fits the profile, not a direct sell

Present all 5 and ask: "Send these now? [Yes / Pick 3 / Pick 1]"

## Step 9: Write to business.md and save data

Append or update business.md with proper markdown headers (use ## headings, not === === format):

```markdown
## Network assessment

**Network size:** [Large/Medium/Small] (~[number] contacts)
**Strategy:** [1-2 sentence outreach strategy based on network size]

## Your 4 MVP lines

- **Line 1 (Niche):** [locked niche]
- **Line 2 (Offer):** [locked offer]
- **Line 3 (Price):** [locked price with ROI math]
- **Line 4 (Delivery):** [locked delivery timeline]

## Validation scorecard

| Dimension | Score | Notes |
|---|---|---|
| Demand | [score]/5 | [brief explanation with data source] |
| Competition | [score]/5 | [brief explanation with data source] |
| Willingness to pay | [score]/5 | [brief explanation with data source] |
| Reachability | [score]/5 | [brief explanation with data source] |
| **Total** | **[total]/20** | **[GO/MAYBE/WEAK/NO]** |

## Competitor deep dive

| Competitor | Price | Platform | What they sell | Weakness |
|---|---|---|---|---|
| [Name] | [price] | [platform] | [1-line description] | [what they don't do] |

**Market gap:** [1-2 sentences on what no competitor offers. This is where the positioning should attack.]

**Your edge:** [1-2 sentences on what you can do that competitors can't or won't.]

## Your first 5 warm messages

1. **[Name]** ([channel]) - [1-line summary of message approach]
2. ...
```

Also save the Apify/YouTube scrape results (already saved in Step 5).

## Step 10: Present next actions

End with a clear action list:

```
NEXT ACTIONS:
1. Send your first 5 messages NOW. Not tomorrow. Now.
2. Run /build-offer to structure your delivery model and lock pricing.
3. Set up cold email infrastructure (secondary domain + Instantly warmup).
4. Automate cold outreach with /outreach-engine next. For now: keep sending 10 warm messages per day.

business.md updated. Scrape saved to: temp/{date}_{file}.json
```

## Overthinking check

If at any point the user:
- Asks to restart from scratch after already having a valid niche
- Wants to test 3+ different niches before committing
- Spends more than 2 messages debating between similar options

Call it out: "You're researching instead of shipping. Pick one. Test it for a full week before you're allowed to switch. Pick the one that felt most natural and let's lock it."

## Output formatting rules

The full output should document the entire process, not just the final answers. Include:

1. **Warm-up answers** (quoted from the user, with commentary)
2. **Network assessment** (size, sources, strategy)
3. **Niche statement + 5-minute test results** (names listed, pass/fail)
4. **All 3 scrape sources with real data** (Reddit findings with upvote counts, YouTube comments with usernames and analysis, search suggestions with interpretation)
5. **Scraping lessons learned** (if a scrape failed or returned bad data, document what went wrong and how you fixed it. You learn from watching the process.)
6. **Perplexity validation** (both queries, all 4 scores with explanations, competitor pricing table with real names and numbers)
7. **Locked 4 MVP Lines**
8. **All 5 warm messages** (personalized, with channel and relationship context)
9. **business.md update section** (the exact content to append)
10. **Next actions**

Keep the tone direct and specific. Frame findings around real prospects and real gaps: Reddit posts showing the pain, YouTube comments showing people asking for help, search suggestions showing where demand goes unmet.

## Self-check before finishing

1. business.md has been updated with all sections above
2. 4 MVP Lines are specific (not vague like "I help businesses grow")
3. Validation scorecard has all 4 dimensions scored with explanations and data sources cited
4. All 5 warm messages are personalized (not generic templates)
5. Scrape results saved to temp/ with date prefix
6. Reddit scrape used specific subreddits (not global search)
7. YouTube comments scraped from 2-3 competitor videos with real quotes and usernames
8. YouTube search suggestions run for 3 keywords with interpretation
9. Competitor pricing includes real names, prices, platforms, and member counts
10. No Tier 1/2 banned words from humanizer rules
11. No em dashes anywhere in the output

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | v2: 3-source data collection (Reddit + YouTube comments + YouTube search suggestions). YouTube MCP now required. Reddit guidance: always specify target subreddits. YouTube comment scraping: 20-30 comments from 2-3 competitor videos. YouTube search suggestions: 3 keywords, low suggestion count = demand gap. Perplexity split into 2 queries. 5-minute test now allows referrers for large networks. Added shortcut: if business.md has niche defined, offer to skip to validation. business.md output uses markdown headers. |
| 2026-03-30 | v1: Full rewrite to match lesson doc. Added: interactive Q&A flow, network size assessment, 5-minute niche test, Perplexity validation, 5 warm message generation, overthinking check, MCP error handling. |
