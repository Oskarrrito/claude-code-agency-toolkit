---
name: daily-monitor
description: YouTube Ads performance check. Takes pasted Google Ads data, compares against kill/scale/hold thresholds, outputs 3-line summary per campaign with recommendations. Also checks organic channel health via YouTube MCP. Run 2-3x per week.
user-invocable: true
disable-model-invocation: true
context: fork
agent: general-purpose
---

# YouTube Ads monitor

Quick check on YouTube ad campaign performance. Pulls key metrics, compares against thresholds, outputs a 3-line summary with kill/scale/hold recommendation per campaign. Adds organic channel data from YouTube MCP.

## Input

User pastes data from Google Ads dashboard. One dashboard, one platform.

On invocation, ask the user to paste their data (see Step 2).

## Step 1: Load context

1. Read `business.md` for service price (to calculate target CPA).
2. Check for previous monitor reports in `temp/` matching `*_daily_monitor.md`. If found, read the most recent one for trend comparison.
3. Set target CPA: `service_price x 0.33`.
4. If `business.md` is missing, ask: "What's your service price? I need it to set your target CPA."

## Step 2: Get campaign data

Ask the user:

```
Paste your YouTube campaign data from Google Ads:

CAMPAIGN NAME | SPEND | VIEWS | VIEW RATE | CLICKS | CTR | LEADS | COST/CONV | EARNED VIEWS
Competitor Pre-roll | $150 | 3,200 | 28% | 48 | 1.5% | 4 | $37.50 | 420
Discovery - AI Service | $80 | 1,800 | N/A | 72 | 4.0% | 6 | $13.33 | 0

Or paste a screenshot from your Google Ads dashboard.
```

Parse the data. If unstructured, extract columns. Ask for clarification only if genuinely ambiguous.

## Step 3: Calculate metrics

For each campaign:

| Metric | Formula |
|---|---|
| CPV | spend / views |
| CTR | clicks / views x 100 (or clicks / impressions for discovery) |
| CPL | spend / leads |
| CPA | spend / conversions |
| View rate | views / impressions x 100 (skippable only) |

If a metric can't be calculated (0 leads), mark as "N/A."

## Step 4: Check organic channel health via YouTube MCP

Call these YouTube MCP tools to add to the paid data:

1. `youtube_analytics_daily` - check if daily views and watch time are trending up (paid traffic spillover)
2. `youtube_analytics_traffic_sources` - check if "YouTube advertising" traffic source is growing
3. `youtube_get_channel` - current subscriber count for tracking growth

If YouTube MCP is not connected, skip this step and note it.

## Step 5: Apply decision thresholds

### KILL
- CPA exceeds 2x target CPA AND spend exceeds 2x target CPA (enough data)
- Zero conversions after $100+ spend
- View rate below 15% on skippable ads after 5,000+ impressions (creative is broken)
- CTR below 0.3% on in-feed discovery after 5,000+ impressions

### SCALE
- CPA under target CPA for 3+ consecutive check-ins
- View rate above 25% on skippable (strong creative)
- Budget increase: 15-20% per step, not 2x overnight

### HOLD
- CPA within 30% of target
- Less than 3 check-ins worth of data
- Just made a change (wait 3-5 days)

### REFRESH CREATIVE
- View rate dropping 20%+ from baseline on skippable
- Same creative running 6+ weeks
- CPV rising while views stay stable

## Step 6: Output

```
## YouTube Ads check-in: {YYYY-MM-DD}

**Target CPA:** ${target_cpa} | **Service price:** ${service_price}

### Summary
1. {Total spend and lead count}
2. {Best performing campaign and why}
3. {Biggest issue and recommended action}

---

### Campaign breakdown

| Campaign | Spend | Views | View Rate | Clicks | CTR | Leads | CPL | CPA | Verdict |
|----------|-------|-------|-----------|--------|-----|-------|-----|-----|---------|
| {name} | ${spend} | {N} | {N}% | {N} | {N}% | {N} | ${N} | ${N} | **HOLD** |

---

### Organic channel health (via YouTube MCP)
- Daily views trend: {up/down/flat} ({N} avg daily views this week)
- Subscriber change: +{N} this week
- Ad traffic spillover: {N} earned views from ad campaigns

---

### Actions
**KILL:** {if any}
**SCALE:** {if any}
**HOLD:** {if any}
**REFRESH:** {if any}

---

### Trend (vs previous check-in)
| Metric | Previous | Current | Direction |
|--------|----------|---------|-----------|
| Total spend | ${N} | ${N} | {up/down/flat} |
| Total leads | {N} | {N} | {up/down/flat} |
| Avg CPA | ${N} | ${N} | {up/down/flat} |
```

Save to `temp/{YYYY-MM-DD}_daily_monitor.md`.

## Rules

1. Never recommend scaling with under 5,000 impressions per campaign.
2. Target CPA = service_price x 0.33. You can override.
3. The 3-line summary must cite specific numbers.
4. YouTube-specific: always include view rate for skippable campaigns and earned views if available.
5. Organic channel health section is a bonus, not required if MCP isn't connected.
6. All commentary must pass humanizer rules.
7. If previous reports exist, check for consecutive patterns (3+ check-ins trending same direction = actionable signal).

## Quality checklist

- [ ] Target CPA calculated from business.md
- [ ] Data ingested and parsed correctly
- [ ] YouTube-specific metrics included (CPV, view rate, earned views)
- [ ] Kill/scale/hold thresholds applied correctly
- [ ] 3-line summary is specific with numbers
- [ ] Organic channel health checked via YouTube MCP (if available)
- [ ] Previous report checked for trend comparison
- [ ] Report saved to temp/
- [ ] No banned words in commentary

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Rebuilt for YouTube Ads only. Added YouTube MCP organic health check. Changed from daily to 2-3x/week cadence. |
