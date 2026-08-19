---
name: weekly-ad-report
description: Aggregates YouTube Ads data into a weekly report. Week-over-week changes, best/worst campaign, spend, blended CPA, earned actions. Adds organic channel growth from YouTube MCP.
argument-hint: Paste this week's YouTube campaign metrics
user-invocable: true
---

# Weekly YouTube Ads report

One report, every Monday. Paste your numbers, get a formatted summary with week-over-week trends, winners, losers, and one priority action.

## Step 1: Load this week's data

Accept data as:
1. **Pasted metrics**: campaign-level data for the week
2. **CSV path**: file in `temp/`
3. **Google Ads export**: raw data, parse and normalize

Required per campaign:
- Campaign name
- Spend
- Views
- View rate (skippable campaigns)
- Clicks / CTR
- Conversions (leads)
- CPA
- Earned views (if available)
- Earned subscribers (if available)

## Step 2: Load previous report

Check `temp/` for `*_weekly_ad_report.md`. If found, extract last week's totals for comparison.

If none exists: "First weekly report. No week-over-week data yet."

## Step 3: Pull organic channel data from YouTube MCP

Call YouTube MCP:
1. `youtube_analytics_overview` for the past 7 days (views, watch time, subscribers gained)
2. `youtube_analytics_traffic_sources` to see how much traffic came from "YouTube advertising"
3. `youtube_analytics_top_videos` to check if any organic videos got a boost from ad traffic

If YouTube MCP not connected, skip and note it.

## Step 4: Calculate aggregates

### Campaign totals
- Total spend
- Total views
- Total clicks
- Total leads
- Blended CPA (total spend / total leads)
- Total earned views
- Total earned subscribers

### Week-over-week changes
If previous data available, calculate % change for:
- Spend, leads, CPA, view rate, earned actions
- Label direction correctly (CPA down = good)

### Best and worst
- Best campaign: lowest CPA with $50+ spend
- Worst campaign: highest CPA with $50+ spend

## Step 5: Generate report

```
WEEKLY YOUTUBE ADS REPORT
Week of {Monday} to {Sunday}
Generated: {today}

OVERVIEW
Total Spend:      ${total}     ({wow_change})
Total Views:      {N}          ({wow_change})
Total Leads:      {N}          ({wow_change})
Blended CPA:      ${cpa}       ({wow_change})
Earned Views:     {N}          (free bonus from ad viewers)
Earned Subs:      +{N}         (free subscribers from ad viewers)

CAMPAIGN BREAKDOWN
Campaign                | Spend   | Views  | View Rate | Leads | CPA     | Earned
-----------------------|---------|--------|-----------|-------|---------|--------
Competitor Pre-roll     | $210    | 4,800  | 31%       | 5     | $42.00  | 620
Discovery - AI Service  | $120    | 2,100  | N/A       | 7     | $17.14  | 0
Remarketing             | $45     | 980    | 42%       | 2     | $22.50  | 85

HIGHLIGHTS
Best campaign:  Discovery - AI Service ($17.14 CPA)
Worst campaign: Competitor Pre-roll ($42.00 CPA)
Top earned:     Competitor Pre-roll (620 earned views)

ORGANIC CHANNEL HEALTH (YouTube MCP)
Channel views this week:    {N} ({wow_change})
Watch time (hours):         {N} ({wow_change})
New subscribers:            +{N}
Ad traffic to channel:      {N} views from YouTube Ads traffic source
Top organic video boosted:  "{title}" (+{N} views above average)

WEEK-OVER-WEEK TRENDS
Metric          | Last Week | This Week | Change
----------------|-----------|-----------|--------
Spend           | ${N}      | ${N}      | {+/-N%}
Leads           | {N}       | {N}       | {+/-N%}
CPA             | ${N}      | ${N}      | {+/-N%}
View Rate (avg) | {N}%      | {N}%      | {+/-N%}

THIS WEEK'S PRIORITY
{One specific, actionable recommendation based on the data.
Not three things. One. The highest-impact move.}
```

## Step 6: Save

Save to `temp/{YYYY-MM-DD}_weekly_ad_report.md`.

## Step 7: Self-check

1. Math is correct (total spend = sum of campaign spends)
2. Week-over-week direction labeled correctly
3. Best/worst only include campaigns with $50+ spend
4. Campaigns sorted by CPA ascending
5. Priority action is specific and actionable
6. Earned actions included where available
7. Organic channel health section included (if YouTube MCP available)
8. No humanizer banned words
9. Report is scannable in under 3 minutes

## Error handling

- **No earned actions data:** Skip those columns. Note: "Earned actions not available in the pasted data. Check Google Ads > Columns > Earned Views to add this metric."
- **Zero conversions all campaigns:** "Zero leads this week across all campaigns. Total spend: ${N}. Check: is your landing page converting organic traffic? If not, the ad isn't the problem."
- **Missing previous report:** Skip week-over-week. Note that next week will have baseline.
- **YouTube MCP not connected:** Skip organic channel health section. Note: "Connect YouTube MCP to see how paid ads affect your organic channel growth."

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Rebuilt for YouTube Ads only. Added YouTube MCP organic channel health. Added earned actions tracking. |
