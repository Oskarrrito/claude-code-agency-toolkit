---
name: build-offer
description: Interactive conversation through 5 offer decisions, then scores the offer on the Value Equation. Walks you through delivery model, the ONE thing, offer specifics, guarantee, and pricing. Scores all 4 Value Equation dimensions and identifies fixes. Reads niche data from business.md, writes completed offer and scores back to business.md. Handles both service-based and membership/community models.
user-invocable: true
---

# /build-offer

Build your complete offer through 5 decisions. Interactive conversation, not a form. This skill adds 1 section to business.md: **Offer details**.

## Input

- Read `business.md` for: niche, target client, prospect data, competitor pricing, founder context
- If business.md is missing niche data, stop and tell the user to run `/find-niche` first
- **Shortcut check:** If business.md already has an "Offer details" section with delivery model, deliverables, guarantee, and price filled in, present the existing offer and ask: "You already have an offer in business.md. Want to refine it, or lock it as-is?" If they say lock, skip to Step 6. If they say refine, walk through the 5 decisions but pre-fill their existing answers.

## How this skill works

This is a conversation, not a batch job. Walk the user through each decision one at a time. Ask the question, wait for their answer, then move to the next decision. Do not dump all 5 questions at once.

Apply humanizer rules from `.claude/rules/humanizer.md` to all generated copy. No Tier 1/Tier 2 banned words.

## Decision 1: Delivery model

Ask: "How do you want to deliver your service? Five options:"

**Agency** (do-it-for-them): You handle everything. Client pays, you deliver. Fastest path to first dollar. Best for most first-time founders.

**Coaching** (teach-them-to-do-it): You guide, they execute. Scales better but needs proof first. Harder to sell without results.

**Software** (build-a-tool): You build something they use on their own. Highest upside, longest build time. Warning: a lot of technical founders spend 2 weeks building instead of selling. If this is your first business, flag the risk.

**Hybrid** (mix of two or more): Examples: agency setup + coaching ongoing. Done-for-you delivery + software tool access. Agency for month 1, transition to coaching month 2.

**Membership/Community** (recurring access to a system, community, or library): Members pay monthly or annually for ongoing access to content, community, coaching calls, tools, or a combination. Think: a system they stay inside, not a one-off service you deliver. Examples: course + community + live calls, skill library + group coaching, content engine + peer accountability.

If the user picks **membership/community**, set a flag. Decisions 2, 4, and 5 will branch to membership-specific questions.

If the user doesn't know: "Start with agency. You can evolve to coaching or hybrid after 2-3 clients. Agency gives you the fastest feedback loop because you're doing the work and seeing what clients actually need."

Wait for their answer before proceeding.

## Decision 2: The ONE thing

### For agency, coaching, software, or hybrid models:

Ask: "What's the ONE thing your client absolutely cannot do themselves? The thing they'd pay for even if everything else disappeared?"

This is the question most founders skip. They list 8 deliverables instead of finding the core. Push them to pick one.

If they give multiple things, ask: "If you could only deliver ONE of those, which one would the client still pay for?" Keep asking until they land on a single core deliverable.

### For membership/community models:

The standard "what can't they do themselves?" question breaks down here. Members could technically learn everything on their own. The answer is usually "the system as a whole," which is too vague.

Instead, ask two questions:

1. "What would members lose if they tried to do this alone? What's the thing they can't replicate by watching free YouTube videos or buying a $20 course?"
2. "What makes paying \$X/month worth it vs. doing it for free?"

The answer usually falls into one of these categories:
- **Execution system** (pre-built tools, skills, templates that force action instead of more learning)
- **Accountability + community** (peer pressure, public tracking, someone watching)
- **Access to you** (live calls, direct feedback, personalized coaching)
- **Curated path** (they don't have to figure out what to do next, the system decides)

Push them to pick the ONE category that matters most. Everything else supports it.

Reference the prospect data from business.md (collected during `/find-niche`) to ground the conversation. "Your prospects said [pain point]. Which of those things you listed solves that specific pain?"

Wait for their answer before proceeding.

## Decision 3: Offer specifics

Ask: "What exactly does the client get? Be specific. Not 'better online presence.' What measurable thing changes for them?"

Push for outcomes over activities:
- If they say "I respond to reviews" (activity), ask: "And what result does that create for the client?"
- If they say "I manage their social media" (activity), ask: "What changes for the client because of that? More followers? More bookings? More DMs?"
- Keep asking until they reach a specific, measurable outcome.

Good: "15 new 5-star reviews per month"
Bad: "Better online presence"
Good: "3 qualified sales calls per week"
Bad: "Lead generation services"
Good: "First 5 paying clients in 90 days"
Bad: "Learn how to get clients"

Once you have the core deliverable (from Decision 2) and the specific outcome, list out the full offer:

- Core deliverable: [the ONE thing]
- Supporting deliverables: [2-5 specific things that support the core]
- First result: [what the client/member sees first and when]
- "Done" looks like: [specific end state or ongoing metric]

For memberships, "done" might be an ongoing state ("5 paying clients, working YouTube channel, and a content system running") rather than a single deliverable moment.

Wait for their confirmation before proceeding.

## Decision 4: Guarantee

### For agency, coaching, software, or hybrid models:

Ask: "What can you guarantee? Not the client's results. YOUR deliverables."

Present the three options:

**Deliverable guarantee:** "If I don't deliver [specific thing] by [specific date], you don't pay." Best for new founders. You control the deliverable, so you control the guarantee.

Example: "If you don't have 10+ new reviews in the first 30 days, you don't pay for month 1."

**Conditional guarantee:** "If you follow the process and don't see [result] in [timeframe], I'll work for free until you do." Requires client participation. Good for coaching models.

**No guarantee:** Fine if proof is strong (lots of testimonials, case studies). Not recommended for the first 3 clients when trust is zero.

### For membership/community models:

Memberships benefit from a **stacked guarantee** (two guarantees layered together). This is stronger than either one alone because it answers two separate fears.

Present the stacked guarantee structure:

**Layer 1: Risk reversal** (removes financial risk):
- "Run the full system for [30/60/90] days. If you don't get [specific result], full refund."
- "Try it for 30 days. If you don't see the value, cancel and get your money back."
- "Complete the first stage. If you don't have [result], full refund."

**Layer 2: Overdelivery** (removes the "what if I'm slow?" fear):
- "If you get results but it takes longer than [timeframe], I'll work with you 1:1 until you hit [target]. Free."
- "If you're doing the work but haven't hit [milestone] by [date], you get an extra month free."
- "Annual members who complete all stages without hitting [target] get a private strategy session."

Example stacked guarantee: "Run the full system for 90 days. If you don't get 5 paying clients, full refund. No questions. And if you get clients but it takes longer than 90 days, I'll work with you 1:1 until you hit 5. Free."

The stacked structure works because:
- Layer 1 answers "What if it doesn't work at all?" (refund)
- Layer 2 answers "What if it works but I'm slower than average?" (extra support)
- Together they make saying yes feel like zero risk.

Single guarantees also work for memberships:
- "Cancel anytime, no questions" (lowest friction, weakest)
- "30-day money-back guarantee" (standard, decent)

Push for the stacked version if they have any proof (testimonials, case studies, personal results).

Help the user construct their guarantee statement(s). Each should be one sentence a prospect can repeat back to a friend. If it takes a paragraph to explain, simplify it.

Wait for their answer before proceeding.

## Decision 5: Pricing

### For agency, coaching, software, or hybrid models:

Walk through value math with the user.

Ask: "What's one [unit of value] worth to your client?" The unit depends on the niche:
- Dental clinic: one new patient (\$3,000+ lifetime value)
- SaaS company: one qualified lead (varies by deal size)
- Restaurant: one new regular customer (\$2,000-5,000 annual value)
- Coach: one client transformation (hard to quantify, use competitor pricing)

Then: "How many [units] can you deliver per month?" Multiply. That's the value created.

"Charge 10-20% of the value you create. One number, not a range."

#### Graduated pricing progression (service models)

Walk the user through the full pricing ladder:

1. **Clients 1-2: Free pilots.** These are case studies, not charity. Deliver the full service for free in exchange for a detailed testimonial and permission to use the results as a case study. Set a clear timeline (2-4 weeks).

2. **Clients 3-5: Founding rate (50% off full price).** In exchange for a testimonial commitment. Lock BOTH numbers now: founding rate AND full price. You need to know your full price even if you're not charging it yet.

3. **Clients 6-10: 20-30% off full price.** By now you have proof. Discount shrinks.

4. **Client 11+: Full price.** No discounts. The proof speaks.

5. **\$800/month floor after 3 paying clients.** No one charges less than \$800/month once they have 3 clients and testimonials. If the value math suggests a lower price, raise the deliverables until the price makes sense.

Set ONE price (the full price). Set ONE founding rate. No ranges. No "it depends." A prospect should be able to ask "how much?" and get a number in under 3 seconds.

### For membership/community models:

Membership pricing works differently from service pricing. There's no "10-20% of value created" formula because the value is systemic, not per-unit.

Ask three questions:

1. **"What's your monthly price?"** This is what members pay each month. Anchor to competitors if you have them from business.md. Position relative to the market: budget (\$47-77/mo), mid-tier (\$99-149/mo), premium (\$149-199/mo).

2. **"What's your annual price, and what discount does that give?"** Annual pricing should give 30-50% off the monthly rate. This rewards commitment and reduces churn. Common patterns:
   - \$149/mo, \$997/yr (saves \$791, 44% discount)
   - \$99/mo, \$697/yr (saves \$491, 42% discount)
   - \$199/mo, \$1,497/yr (saves \$891, 37% discount)

3. **"What do annual members get that monthly members don't?"** Common: all stages unlocked immediately (monthly members are time-gated), bonus resources, priority access to calls. This is the incentive to go annual.

#### Value math for memberships

The formula is simpler: "If a member follows your system, what's the first dollar amount they could earn? How long does that take?"

If the answer is "first client worth \$1,500 in 30 days," then \$149/month is 10% of that value. One client pays for a full year of annual membership. That's the pitch.

#### Membership pricing progression

Instead of per-client pricing ladders, memberships use member-count milestones:

1. **Launch price:** Your starting price. This is the founding member rate. It gets locked in forever for early members.
2. **Post-[100] members:** Price increases for new members (e.g., \$149 -> \$199/mo). Founding members keep their rate.
3. **Post-[300] members:** Another increase (e.g., \$199 -> \$249/mo). Founding members still locked.
4. **Founding member lock:** Early members never see a price increase. This rewards early commitment and creates urgency ("price goes up at 100 members").

Walk the user through setting their launch price, first price increase trigger (member count), and what founding members get locked in at.

## Step 6: Update business.md

After all 5 decisions are confirmed, append the following section to business.md under "## Offer details":

### For service models (agency, coaching, software, hybrid):

```
## Offer details

- **Delivery model:** [agency/coaching/software/hybrid]
- **Core deliverable:** [the ONE thing]
- **Summary:** "[I help [who] do [what] so they get [result]]"
- **Deliverables:**
  - [core deliverable]
  - [supporting deliverable 1]
  - [supporting deliverable 2]
  - [supporting deliverable 3]
- **First result:** [what and when]
- **Done looks like:** [specific end state or ongoing metric]
- **Guarantee:** "[guarantee statement]"
- **Guarantee type:** [deliverable/conditional/none]
- **Full price:** [amount + billing cycle]
- **Founding rate:** [amount + billing cycle]
- **Pricing progression:**
  - Clients 1-2: Free pilots (testimonial + case study)
  - Clients 3-5: [founding rate] (50% off, testimonial commitment)
  - Clients 6-10: [discounted price] (20-30% off)
  - Client 11+: [full price]
- **Price floor:** $800/month (after 3 paying clients)
```

### For membership/community models:

```
## Offer details

- **Delivery model:** Membership/community [+ coaching/software/hybrid if applicable]
- **Core deliverable:** [the ONE thing]
- **Summary:** "[I help [who] do [what] so they get [result]]"
- **Deliverables:**
  - [core deliverable]
  - [supporting deliverable 1]
  - [supporting deliverable 2]
  - [supporting deliverable 3]
  - [supporting deliverable 4 if applicable]
- **First result:** [what and when]
- **Done looks like:** [specific ongoing state]
- **Guarantee (stacked):**
  - **Risk reversal:** "[refund guarantee statement]"
  - **Overdelivery:** "[overdelivery guarantee statement]"
- **Guarantee type:** stacked (risk reversal + overdelivery)
- **Monthly price:** [amount]/month
- **Annual price:** [amount]/year ([effective monthly], [discount %] off monthly)
- **Annual extras:** [what annual members get that monthly don't]
- **Founding rate:** [amount]/month locked forever
- **Pricing progression:**
  - Launch: [price] (founding member rate, locked)
  - Post-[N] members: [new price] for new members
  - Post-[N] members: [new price] for new members
  - Founding members: [price] locked forever
```

Do not overwrite existing business.md fields (niche, target client, etc.). Only add or update the Offer details section.

## Step 7: Score your offer on the Value Equation

After saving the offer to business.md, score it immediately. No need for a separate skill.

**Value = (Dream Outcome x Perceived Likelihood) / (Time Delay x Effort & Sacrifice)**

Two variables in the numerator make the offer more valuable. Two in the denominator make it less.

Score each dimension 1-10 based on the offer you just built:

1. **Dream Outcome:** How life-changing is the result? (9-10: they'd pay 10x. 1-2: barely notice.)
2. **Perceived Likelihood:** How confident is a cold prospect you'll deliver? Not you. Them. (9-10: 3+ case studies. 1-2: feels like a gamble.)
3. **Time Delay:** How fast is the first result? (9-10: first week. 1-2: 6+ months.)
4. **Effort & Sacrifice:** How much does the client do? (9-10: hand over credentials, done. 1-2: overhaul everything.) Guided effort with AI checklists scores higher than unguided.

Present all four scores at once with a 1-2 sentence explanation for each. Ask: "Agree? Disagree on any dimension?"

Add all four. Max is 40. Assign the band:
- **40+:** Strong. Ready for market. Minor tweaks while you sell.
- **25-39:** Fixable. Identify the weakest dimension and fix it before heavy outreach.
- **Under 25:** Rethink. Go back through the 5 decisions above.

**AI Multiplier:** If you use AI to deliver the service (AI handles responses, AI writes content, AI builds reports), add 5 points to Dream Outcome before calculating. AI delivery raises perceived value because the client gets speed and consistency a human freelancer can't match.

If the score is low, identify the weakest dimension and give 1 specific fix:
- Perceived Likelihood low? Do a free pilot to get proof. Record a 2-minute Loom showing the system working live.
- Time Delay low? What manual step could AI make instant?
- Effort low? Remove forms, calls, and decisions. Pre-fill everything.
- Dream Outcome low? Add a specific number to the outcome.

Append a "Value equation scores" section to business.md:

```
## Value equation scores

Scored: [YYYY-MM-DD]

| Dimension | Score | Notes |
|-----------|-------|-------|
| Dream Outcome | X/10 | [short note] |
| Perceived Likelihood | X/10 | [short note] |
| Time Delay | X/10 | [short note] |
| Effort & Sacrifice | X/10 | [short note] |
| **Total** | **X/40** | **[BAND]** |

- Weakest: [dimension name]
- Top fix: [specific action]
- AI Multiplier: [applied / not applicable]
```

## Output

Print a summary using markdown headers and tables:

```markdown
## Delivery model

**[Model name]:** [brief description]

- [How delivery works point 1]
- [How delivery works point 2]
- [How delivery works point 3 if needed]

## The ONE thing

"[Core deliverable in one sentence]"

## Offer

"I help [who] [do what] so they get [result]."

| Component | Detail |
|-----------|--------|
| Core deliverable | [the ONE thing] |
| Supporting deliverables | [deliverable 1], [deliverable 2], [deliverable 3] |
| First result | [what, by when] |
| Done looks like | [specific end state] |

## Guarantee

**[Guarantee type]:**

[For stacked guarantees, list both layers:]
1. **Risk reversal:** "[statement]"
2. **Overdelivery:** "[statement]"

[For single guarantees:]
"[guarantee statement]"

## Pricing

[For service models:]

| Tier | Price | Notes |
|------|-------|-------|
| Full price | [amount] | [billing cycle] |
| Founding rate (clients 3-5) | [amount] | + testimonial commitment |
| Price floor | $800/month | After 3 paying clients |

Pricing progression:
- Clients 1-2: Free pilots (case study + testimonial)
- Clients 3-5: [founding rate] (50% off)
- Clients 6-10: [discounted] (20-30% off)
- Client 11+: [full price]

[For membership models:]

| Tier | Price | Notes |
|------|-------|-------|
| Monthly | [amount]/mo | [details] |
| Annual | [amount]/yr | [effective monthly], [discount]% off |
| Founding rate | [amount]/mo | Locked forever |

Pricing progression:
- Launch: [price] (founding members locked)
- Post-[N] members: [new price] for new members
- Post-[N] members: [new price] for new members

**Value math:** [one sentence showing the math]

## Next action

business.md updated (Offer details and Value Equation scores sections added).
```

## Error handling

- business.md missing or empty: Stop. Tell user to run `/find-niche` first.
- business.md missing niche data: Stop. The offer builds on top of niche research.
- business.md already has offer details: Present them. Ask "refine or lock as-is?" Don't force them through 5 decisions again if the offer is already solid.
- User can't pick a delivery model: Default to agency. Explain why. Move forward.
- User picks membership but describes a service: Ask "Are clients paying you to do work for them, or paying for ongoing access to a system?" Clarify before proceeding.
- User can't identify the ONE thing: Ask "What did your prospects say was their biggest pain?" (from business.md prospect data). Work backward from the pain to the deliverable.
- User can't identify the ONE thing for membership: Ask "What would your members lose if you shut down tomorrow? The community? The tools? The calls? The system?" Whatever they'd miss most is the ONE thing.
- User gives vague deliverables: Keep pushing for specifics. "Better online presence" becomes "15 new 5-star reviews per month." "Learn AI" becomes "first 5 paying clients in 90 days." Do not accept vague answers.
- User wants to skip guarantee: That's fine. Set guarantee_type to "none" in business.md. Note that adding a guarantee later (after proof) is recommended.
- User's value math produces a service price under \$500/month: Check if deliverables can be expanded. If not, proceed but flag: "After 3 paying clients and testimonials, your floor is \$800/month."
- User sets membership price below \$47/month: Flag it. "Below \$47/month attracts tire-kickers who won't execute. The system requires commitment. Consider \$47-77 as your absolute floor."

## Self-check (run before delivering)

1. All 5 decisions are answered (delivery model, ONE thing, specifics, guarantee, price)
2. Delivery model is one of: agency, coaching, software, hybrid, membership/community
3. If membership, both monthly AND annual prices are set
4. If membership, stacked guarantee was presented as an option (user may still choose single)
5. Core deliverable is ONE specific thing, not a list
6. Offer summary is one sentence in "I help [who] do [what] so they get [result]" format
7. Guarantee is one sentence (per layer) a prospect can repeat to a friend
8. For services: both full price and founding rate are set (single numbers, not ranges)
9. For memberships: monthly, annual, and founding rate are all set
10. Pricing progression matches the model type (per-client ladder for services, member-count milestones for memberships)
11. business.md is updated with Offer details section
12. Zero Tier 1/Tier 2 banned words in any generated copy
13. Each decision was asked individually (not dumped as a batch)
14. Value Equation scores calculated and saved to business.md
15. Weakest dimension identified with a specific fix

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
