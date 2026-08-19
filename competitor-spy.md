---
name: competitor-spy
description: Analyzes competitor YouTube channels via YouTube MCP. Finds their top content, messaging themes, content gaps, and targeting opportunities. Also checks Meta Ad Library via Apify for cross-platform intelligence.
argument-hint: Competitor channel name or URL
user-invocable: true
---

# Competitor channel spy

Research your competitors' YouTube channels and ad strategy. Find what content works for them, what gaps they're leaving, and which channels to target with your ads.

## Step 1: Get competitor info

1. If the user provided a YouTube channel name or URL, use it.
2. If the user provided a brand name, search for their channel.
3. If nothing provided, ask: "Which competitor should I research? Give me their YouTube channel name or URL."

Collect up to 5 competitors per run.

## Step 2: Research via YouTube MCP

For each competitor:

1. Call `youtube_search` with the competitor's brand name to find their channel.
2. Call `youtube_get_channel` to get channel stats (subscribers, total views, video count).
3. Call `youtube_list_videos` to get their recent videos (last 20-30).
4. For their top 5 videos by view count, call `youtube_get_video` for detailed stats.
5. For the top 3 videos, call `youtube_get_transcript` to analyze their messaging.

If YouTube MCP is not connected, ask the user to provide competitor channel URLs and use Apify web scraper as fallback.

## Step 3: Analyze competitor content

For each competitor, analyze:

### 3a. Channel health
- Subscriber count
- Upload frequency (weekly? monthly?)
- Average views per video (last 10 videos)
- Engagement rate (likes + comments / views)
- Channel growth trajectory

### 3b. Content themes
Read video titles, descriptions, and transcripts. Categorize into themes:
- Educational/how-to
- Case studies/results
- Behind-the-scenes/process
- Industry commentary/opinion
- Product/service promotion
- Client testimonials

Report the percentage split.

### 3c. Top-performing content
Identify the 3-5 best-performing videos (highest views relative to channel average). Note:
- What topic did they cover?
- What hook did they use in the title/thumbnail?
- How long is the video?
- What's the engagement rate?

### 3d. Messaging patterns
From transcripts, identify:
- Common hooks (first 30 seconds of videos)
- Key phrases they repeat
- Pain points they address
- Results/proof they cite
- CTAs they use

### 3e. Content gaps
What topics or angles are they NOT covering that you could own?
- Angles missing from their content
- Questions their audience asks in comments that they don't address
- Niches within the topic they skip

## Step 4: Check Meta Ad Library (optional cross-platform intel)

If the competitor runs Meta ads:
1. Search Apify actors for "meta ad library" or "facebook ad library."
2. Run `fetch-actor-details` first to check input schema.
3. Call the actor with competitor's page name.
4. Note: active ad count, dominant format, longest-running ad, messaging themes.

If no Meta ads found, note it and move on.

## Step 5: Generate targeting recommendations

Based on analysis:

1. **Channels to target:** Which competitor channels are worth targeting with your pre-roll ads? Prioritize channels with:
   - Active uploading (weekly+)
   - Audience overlap with your target (same niche)
   - Moderate to high view counts (enough volume for your ads to reach people)

2. **Custom intent keywords:** Based on competitor content themes and audience questions, suggest 10-20 keywords for custom intent audiences.

3. **Creative gaps:** Angles and formats your competitors aren't using that you can fill.

## Step 6: Save output

Save to `temp/{YYYY-MM-DD}_competitor_spy_{competitor_name}.md`

If multiple competitors, save one file per competitor plus a comparison summary:
`temp/{YYYY-MM-DD}_competitor_spy_summary.md`

## Example output

```
COMPETITOR CHANNEL ANALYSIS
Competitor: TechFounderAcademy
Source: YouTube MCP
Date: 2026-04-28

CHANNEL STATS
Subscribers:        12,400
Total videos:       87
Upload frequency:   2x/week
Avg views (last 10): 3,200
Engagement rate:    4.2%

TOP VIDEOS
1. "How I Got My First AI Client in 7 Days" - 28,000 views
2. "The $5,000/Month AI Service Nobody Talks About" - 19,000 views
3. "Why I Quit My Job to Build AI Services" - 15,000 views

CONTENT THEMES
Educational/how-to:     45%
Case studies/results:   25%
Industry opinion:       15%
Behind-the-scenes:      10%
Direct promotion:        5%

TOP HOOKS (from transcripts)
1. "Most people think you need to code to sell AI services..."
2. "I'm going to show you exactly what I did in week one..."
3. "$5,000 per month from a service nobody is talking about..."

CONTENT GAPS
- No content about using Claude Code specifically (they focus on ChatGPT)
- No content about outreach/cold DMs for client acquisition
- No content about pricing strategy for AI services
- Comments frequently ask about "how to handle scope creep" and no video answers it

TARGETING RECOMMENDATIONS
1. Target this channel for pre-roll ads (active, relevant audience, 3K+ avg views)
2. Custom intent keywords from their content:
   "AI service business", "sell AI services", "AI freelancing",
   "first AI client", "AI automation agency", "AI side hustle"
3. Creative angle to exploit: Claude Code specific content (they don't cover it)
4. Your differentiator: outreach and sales skills plus specific tool mastery vs general AI talk
```

## Self-check

1. Channel stats match what YouTube MCP returned
2. Content themes add up to ~100%
3. Top videos are correctly identified (highest views)
4. Content gaps are real (not just things they covered less)
5. Targeting recommendations are actionable
6. No humanizer banned words in analysis
7. Each competitor's report can stand alone

## Error handling

- **Channel not found:** "I couldn't find a YouTube channel for '{name}'. Try providing the exact channel URL."
- **YouTube MCP not connected:** Fall back to Apify web scraper. If that fails, ask user to paste channel data manually.
- **Channel has very few videos (<10):** Note it: "This channel only has {N} videos. Limited data for analysis. The content gap analysis may be less reliable."
- **Multiple channels with similar names:** List matches with subscriber counts and ask user to confirm.

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
