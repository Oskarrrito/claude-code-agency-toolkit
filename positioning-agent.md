---
name: positioning-agent
description: "Scrapes top channels in your niche via YouTube MCP, maps avatars, finds content gaps, and outputs a positioning doc with statement, avatar profile, content pillars, and video ideas. Saves to business.md."
argument-hint: "[competitor channel URLs or niche keywords]"
user-invocable: true
---

# Positioning agent

Map the competitive field in your YouTube niche. Scrape top channels, map their avatars and content patterns, find gaps, and produce a positioning doc you can use as a filter for every video you make.

## Input

- `$ARGUMENTS`: 5-10 competitor YouTube channel URLs, separated by spaces or commas. If no argument provided, ask: "Give me 5-10 YouTube channel URLs in your niche."

## Step 1: Load context

1. Read `business.md` for niche, backstory, expertise, and target avatar
2. Read `.claude/rules/humanizer.md` for writing rules

If `business.md` is missing or has no avatar section, stop and tell the user: "Run `/business` first to set up your niche and avatar. The positioning agent needs that context."

## Step 2: Resolve channel list

**If URLs provided:**
Extract channel IDs from each URL. YouTube channel URLs can be:
- `youtube.com/@handle`
- `youtube.com/channel/CHANNEL_ID`

**If niche keywords provided:**
Use `mcp__youtube__youtube_search` to find relevant channels:
```
query: "{keywords}"
maxResults: 20
order: "relevance"
type: "channel"
```

Select the top 15-20 channels by subscriber count and relevance.

**Expand with related channels:**
For each provided channel, search YouTube for related content to find additional channels the user may have missed. Target: 15-20 total channels for analysis.

Print the channel list:
```
CHANNELS TO ANALYZE ({N} total)
-------------------------------
1. @{handle} | {subscribers} subscribers | {videos} videos
2. ...
```

## Step 3: Scrape channel data

For each channel, pull the last 50 videos using YouTube MCP. Collect:
- Video titles
- Descriptions (first 200 chars)
- View counts
- Like counts
- Comment counts
- Publish dates
- Video duration

Save raw data to `temp/{YYYY-MM-DD}_positioning_{niche_slug}.json`.

## Step 4: Analyze content patterns

For each channel, map:
- **Topics covered most often** (group by keyword clusters in titles)
- **Formats used** (tutorials, case studies, opinions, vlogs, shorts, live streams)
- **Breakout videos** (3x or more the channel's average views). Note the exact title and view count.
- **Underperformers** (below 50% of channel average). Note the topic.
- **Upload frequency** (videos per week/month)
- **Consistency** (longest gap between uploads in last 6 months)

## Step 5: Map each channel's avatar

For each channel, infer from the content, titles, and engagement patterns:
- Who is this channel talking to? (role, experience level, situation)
- What level? (beginner, intermediate, advanced)
- What problem does the channel solve?
- What language does the audience use? (pull from comments if available)

This tells you exactly who is already being served and who is being ignored.

## Step 6: Identify gaps

Look for four types of gaps:

1. **Topics with high search demand but low channel supply.** Use `mcp__youtube__youtube_search_suggestions` for niche keywords. Compare what people search for against what already exists.
2. **Audience segments that are not targeted.** Every channel talks to beginners, but nobody serves the person who's 6 months in and stuck.
3. **Angles that nobody is taking.** Everyone teaches "how to get clients." Nobody talks about "what happens after you get 5 clients and can't deliver."
4. **Oversaturated topics to avoid.** If 15 out of 20 channels have a "how to start" video with similar view counts, that's a crowded space.

Score each gap on a 1-10 scale based on demand (search volume, comment requests) vs. supply (how many channels cover it).

## Step 7: Cross-reference with YouTube search demand

Check YouTube search suggestions for niche keywords using `mcp__youtube__youtube_search_suggestions`. Compare what people are searching for against what already exists. The gap between demand and supply is where the first 10 videos live.

## Step 8: Generate the positioning doc

Produce the final output with these sections:

```
=== POSITIONING STATEMENT ===

{One sentence: who you help, how, and why you're different.
Pull from business.md avatar + gap analysis.}

=== AVATAR PROFILE ===

Name: {representative name}
Role: {role and situation}
Revenue: {current income range}
Background: {tools tried, money spent, current state}

Problem they describe: "{exact language they'd use}"

What they search on YouTube:
- {5-8 search queries}

What they click on:
- {4-5 patterns}

What they ignore:
- {4-5 patterns}

=== CONTENT PILLARS ===

{5 pillars, each with:}
1. {PILLAR NAME} ({what it covers})
   - {example video title 1}
   - {example video title 2}
   Gap score: {N}/10. {Why this gap exists.}

=== FIRST 10 VIDEO IDEAS (ranked by gap opportunity) ===

{10 video ideas, each with:}
 N. "{Title}"
    ({pillar}, gap {N}/10)

=== ENEMY RECOMMENDATION ===

Your enemy: "{name the enemy in one phrase}"

Why: {2-3 sentences on why this enemy resonates with the avatar}

=== COMPETITIVE FIELD SUMMARY ===

Channels analyzed: {N}
Average subscriber count: {N}
Average views per video: {N}

Key finding: {2-3 sentences on the biggest opportunity}

Positioning saved to business.md.
Competitor analysis saved to:
temp/{YYYY-MM-DD}_positioning-{slug}.json
```

## Step 9: Save to business.md

Append the positioning statement, avatar profile, content pillars, enemy, and first 10 video ideas to the user's `business.md` file. If a previous positioning section exists, archive it under `## Previous positioning ({date})` and replace with the new one.

## Comparison with previous runs

If a previous positioning section exists in `business.md`, compare:
- New competitors that appeared
- Channels that went quiet or changed direction
- Shifting search trends
- Whether the user's content is filling the gaps identified last time

Include a brief "What changed since last run" section in the output.

## Error handling

- YouTube MCP not connected: ask user to check connection, or use Apify YouTube scraper as fallback
- Channel URL not found: skip it, note it, continue with remaining channels
- Fewer than 5 channels resolved: ask for more URLs before proceeding
- business.md missing: stop and tell user to run `/business` first
- No clear gaps found: report the finding honestly. If the niche is saturated, say so.

## Self-check

1. At least 15 channels analyzed
2. Positioning statement is one sentence
3. Avatar profile includes search behavior and click patterns
4. Exactly 5 content pillars with gap scores
5. Exactly 10 video ideas ranked by gap opportunity
6. Enemy recommendation included
7. Competitive field summary with channel count and averages
8. Raw data saved to temp/ with date prefix
9. Positioning saved to business.md
10. No Tier 1/2 banned words from humanizer rules

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. 9-step pipeline: load context, resolve channels, scrape data, analyze patterns, map avatars, identify gaps, cross-reference search trends, generate positioning doc, save to business.md. |
