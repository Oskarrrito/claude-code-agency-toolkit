---
name: kill-scale
description: YouTube Ads decision engine. Takes campaign metrics, applies kill/scale/hold rules with YouTube-specific thresholds (view rate, CPV, earned actions), outputs clear action per campaign with reasoning.
argument-hint: Paste campaign metrics or path to metrics file
user-invocable: true
---

# Kill / scale / hold decision engine

Feed this skill your YouTube campaign numbers and get a clear action for every campaign: kill it, scale it, or hold it.

## Step 1: Load metrics

Accept metrics in one of these formats:

1. **Pasted table**: campaign names with metrics
2. **CSV file path**: file in `temp/`
3. **From daily-monitor**: reference a recent `/daily-monitor` output

Required per campaign:
- Campaign name
- Spend (total)
- Views
- View rate (skippable campaigns)
- Clicks / CTR
- Conversions (leads or clients)
- CPA (cost per conversion)

Optional:
- Days running
- Impressions
- CPV (cost per view)
- Earned views / earned subscribers
- Frequency (for remarketing)

## Step 2: Set decision thresholds

Defaults (override if you have your own):

```
Target CPA: service_price x 0.33 (from business.md)
Minimum spend before judging: $100 or 2x target CPA, whichever is higher
View rate floor (skippable): 15%
CTR floor (in-feed discovery): 0.3%
Frequency ceiling (remarketing): 3.0
```

Ask: "Based on your service price, your target CPA is ${target_cpa}. Correct?"

## Step 3: Apply decision rules

For each campaign, run in order:

### Rule 1: Insufficient data
If spend < minimum threshold:
- Action: HOLD
- Reason: "Not enough data. Spent ${spend} of ${minimum} minimum."

### Rule 2: Zero results
If spend >= minimum AND conversions = 0:
- Action: KILL
- Reason: "Spent ${spend} with zero conversions."

### Rule 3: CPA check
If CPA > 2x target CPA:
- Action: KILL
- Reason: "CPA of ${cpa} is more than double target of ${target_cpa}."

If CPA > 1.5x target CPA:
- Action: HOLD (with warning)

If CPA <= target CPA:
- Passes. Continue.

### Rule 4: View rate check (skippable campaigns only)
If view rate < 15%:
- Add flag: "View rate of {N}% means most viewers skip immediately. Creative needs work."

### Rule 5: CTR check (discovery campaigns only)
If CTR < 0.3% after 5,000+ impressions:
- Add flag: "CTR of {N}% is below floor. Thumbnail or title isn't working."

### Rule 6: Frequency check (remarketing only)
If frequency > 3.0:
- Add flag: "Frequency of {N} means audience is seeing ads too often."

### Rule 7: Scale decision
If campaign passed CPA check AND no kill flags:

Check volume threshold:
- Budget $2,000+/month: need 50+ conversions over 14 days
- Budget under $2,000/month: need 15+ conversions over 30 days

If threshold met:
- Action: SCALE
- Recommendation: "Increase daily budget by 15-20%."

If CPA under target but volume threshold NOT met:
- Action: HOLD (approaching SCALE)
- Recommendation: "Keep running. Check again when conversions reach ${threshold}."

## Step 4: Output

```
Campaign              | Action | CPA     | View Rate | CPV   | Reason
---------------------|--------|---------|-----------|-------|---------------------------
Competitor Pre-roll   | SCALE  | $180    | 32%       | $0.04 | CPA under target, strong view rate
Discovery - AI Service| HOLD   | $310    | N/A       | $0.06 | CPA above target but only 8 days of data
Remarketing - Warm    | KILL   | N/A     | 45%       | $0.03 | $120 spent, zero conversions

SUMMARY
  Total spend: $450
  Blended CPA: $225
  Kill: 1 campaign ($120 freed)
  Scale: 1 campaign
  Hold: 1 campaign
  
  TOP MOVE: Kill remarketing (audience pool too small at 340 visitors).
  Redirect $120 to competitor pre-roll which is converting at $180 CPA.
```

Save to `temp/{YYYY-MM-DD}_kill_scale_decisions.md`.

## Step 5: Self-check

1. Every campaign has exactly one action: KILL, SCALE, or HOLD
2. Every action has a reason citing specific numbers
3. No campaign judged with insufficient spend
4. SCALE never suggests more than 30% budget increase
5. YouTube-specific metrics (view rate, CPV) included where applicable
6. No humanizer banned words

## Error handling

- **Missing metrics:** List which fields are missing. Ask user to fill in.
- **No target CPA:** Walk through: "What do you charge? If $3,000 service with 20% close rate, target CPA = $1,000."
- **Single campaign:** Still run analysis but note: "One campaign means you can't reallocate budget. Consider adding a second campaign type."
- **All HOLD:** "None have enough data yet. Increase budgets or wait 3-5 more days."

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
