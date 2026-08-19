---
name: budget-calculator
description: Calculates YouTube Ads budget allocation by campaign type (in-stream, discovery, remarketing, Shorts). Takes monthly budget and service price, outputs daily budgets, expected views/leads, break-even analysis, and LTV:CAC ratio across 3 scenarios.
argument-hint: "[monthly budget, e.g. $1000]"
user-invocable: true
disable-model-invocation: true
context: fork
agent: general-purpose
---

# YouTube Ads budget calculator

Calculates how to split YouTube ad spend across campaign types, projects lead volume and revenue, and shows break-even points. Outputs 3 scenarios: conservative, moderate, aggressive.

## Input

- $ARGUMENTS: monthly ad budget (e.g., "$1000", "2000", "$500/month")
- Parse the number from the input. Strip dollar signs, commas, "/month" text.
- If no budget provided, ask: "What's your monthly YouTube ad budget? (e.g., $1000)"
- Check if the business has both monthly and annual pricing. If so, calculate LTV scenarios for both.

## Step 1: Load context

1. Parse the monthly budget from $ARGUMENTS.
2. Read `business.md` for:
   - Service price (per project or monthly retainer)
   - Average client lifetime (months, if retainer)
   - Target number of clients
   - Current close rate (if mentioned, else use default 20%)
3. If `business.md` is missing, ask: "What's your service price?"

## Step 2: Determine budget tier

| Tier | Budget | Campaigns |
|---|---|---|
| Tier 1 | $500-999/month | In-stream only (competitor targeting) |
| Tier 2 | $1,000-1,999/month | In-stream (70%) + In-feed discovery (30%) |
| Tier 3 | $2,000+/month | In-stream (50%) + Discovery (25%) + Remarketing (15%) + Shorts (10%) |

If budget is under $500/month, warn: "At this budget, you won't get enough data. Focus on organic outreach and YouTube content first."

## Step 3: Set YouTube Ads benchmarks

| Metric | Skippable In-Stream | In-Feed Discovery | Remarketing | Shorts |
|---|---|---|---|---|
| CPV / CPM | $0.03-0.08 CPV | $0.04-0.10 CPV | $0.02-0.06 CPV | $3-5 CPM |
| View rate | 20-35% | N/A (click-to-play) | 30-50% | N/A (non-skip) |
| CTR | 0.5-1.5% | 2-4% | 1-3% | 0.1-0.5% |
| Landing page conv. | 5-10% | 8-15% | 10-20% | 3-8% |

Default close rate: 20% for YouTube traffic (higher intent than social media).

## Step 4: Calculate 3 scenarios

For each scenario (conservative, moderate, aggressive), compute:

### Conservative
- CPV: $0.08 (in-stream), $0.10 (discovery)
- Landing page conv: 5% (in-stream), 8% (discovery), 10% (remarketing)
- Close rate: 15%

### Moderate
- CPV: $0.05 (in-stream), $0.06 (discovery)
- Landing page conv: 7% (in-stream), 12% (discovery), 15% (remarketing)
- Close rate: 20%

### Aggressive
- CPV: $0.03 (in-stream), $0.04 (discovery)
- Landing page conv: 10% (in-stream), 15% (discovery), 20% (remarketing)
- Close rate: 25%

For each scenario, calculate:

```
Monthly budget = {input}
Campaign split = based on tier

Per campaign type:
  Monthly spend = budget x allocation %
  Daily budget = monthly spend / 30
  Views/month = monthly spend / CPV
  Clicks/month = views x CTR
  Leads/month = clicks x landing page conv rate
  
Total leads/month = sum across campaign types
Booked calls = total leads x show_up_rate (default 65%)
Clients won = booked calls x close_rate

Revenue per client = {service_price}
Monthly revenue from ads = clients won x revenue per client
LTV per client = service_price x avg_lifetime_months (retainer) or service_price (project)

Total CAC = budget / clients won
LTV:CAC ratio = LTV / CAC

Break-even = budget / (revenue per client x close_rate x show_up_rate x avg_conv_rate)
ROAS = monthly revenue from ads / monthly budget
```

## Step 5: Output

Print directly to conversation:

```
## YouTube Ads budget analysis: ${budget}/month

**Service:** {service name}
**Price:** ${service_price} per {project/month}
**Budget tier:** Tier {N}

---

### Campaign allocation

| Campaign | Monthly spend | Daily budget | Format |
|----------|--------------|--------------|--------|
| In-stream (competitor targeting) | ${amount} | ${daily}/day | Skippable pre-roll |
| In-feed discovery | ${amount} | ${daily}/day | Search/browse placement |
| Remarketing | ${amount} | ${daily}/day | Non-skippable/Shorts |
| Shorts | ${amount} | ${daily}/day | Vertical feed |
| **Total** | **${budget}** | **${daily_total}/day** | |

---

### Scenario comparison

| Metric | Conservative | Moderate | Aggressive |
|--------|-------------|----------|------------|
| Total views/month | {N} | {N} | {N} |
| Total clicks/month | {N} | {N} | {N} |
| Total leads/month | {N} | {N} | {N} |
| Clients won | {N} | {N} | {N} |
| Revenue from ads | ${rev} | ${rev} | ${rev} |
| Total CAC | ${cac} | ${cac} | ${cac} |
| LTV:CAC ratio | {X}:1 | {X}:1 | {X}:1 |
| ROAS | {X}x | {X}x | {X}x |
| Break-even | {N} leads | {N} leads | {N} leads |

---

### What the numbers mean

**Conservative:** {1-2 sentences}
**Moderate:** {1-2 sentences}
**Aggressive:** {1-2 sentences}

---

### Recommendations

1. **Minimum viable budget:** ${min}/month
2. **Campaign priority:** {which campaign type first and why}
3. **Testing phase:** First $200-300 tests 3-5 ad variations before scaling
4. **Scale trigger:** Once CPA is under ${target_cpa}, increase daily budget 15% every 3 days
5. **Kill trigger:** If CPA exceeds ${kill_cpa} after spending ${kill_spend}, pause and rewrite

---

### LTV:CAC health check

| Ratio | Verdict |
|-------|---------|
| Below 1:1 | Losing money. Fix offer pricing or ad efficiency before scaling. |
| 1:1 to 2:1 | Breaking even. Improve landing page and close rate. |
| 2:1 to 3:1 | Healthy. Scale gradually. |
| 3:1 to 5:1 | Strong. Scale faster. |
| Above 5:1 | You're under-spending. Increase budget. |

Your moderate ratio: **{X}:1** = {verdict}
```

Also save to `temp/{YYYY-MM-DD}_budget_analysis.md`.

## Rules

1. All math must be shown. Round only final display values.
2. Pull service price from business.md. Do not assume.
3. If budget under $500/month, warn about insufficient data.
4. If service price under $500, warn about difficulty with paid ads at that price point.
5. Target CPA derivation: service_price x 0.33.
6. No banned words from humanizer rules in commentary.
7. Present numbers plainly. "$12,000" not "an impressive $12,000."
8. YouTube-specific: include earned actions note ("YouTube ads also generate free views and subscribers beyond what you pay for. These aren't included in the projections but improve your numbers over time.")

## Quality checklist

- [ ] Monthly budget parsed correctly
- [ ] business.md read and service price extracted
- [ ] Budget tier determined correctly
- [ ] Campaign allocation matches tier rules
- [ ] All 3 scenarios computed with YouTube-specific benchmarks
- [ ] Full funnel: spend > views > clicks > leads > calls > clients > revenue
- [ ] CAC, LTV:CAC, and ROAS calculated
- [ ] Break-even expressed in leads
- [ ] Plain-English interpretation per scenario
- [ ] Recommendations with scale/kill triggers
- [ ] Low-budget warning if under $500
- [ ] Output saved to temp/
- [ ] No banned words in commentary

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Rebuilt for YouTube Ads only (removed Meta/Google Search) |
