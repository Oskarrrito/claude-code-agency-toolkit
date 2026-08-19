---
name: close-deal
description: Full sales cycle in 3 modes. prep-call: research prospect via Perplexity, generate timed 20-min script with price anchoring. sales-coach: analyze call transcript, score 5 dimensions. after-close: generate case study, testimonial request, referral asks.
argument-hint: "[mode] [details, e.g. 'prep-call Dr. Sarah Chen Bright Smile Dental']"
---

# Close the deal (3 modes)

**Purpose:** Support the full sales cycle. Before the call: research and prep. After the call: coaching and feedback. After the close: extract maximum value (case study, testimonial, referrals).

## Input

- $ARGUMENTS: mode + details. Three valid modes:
  - `prep-call [prospect name and business]`
  - `sales-coach [transcript file path or "notes"]`
  - `after-close [client name]`
- Read business.md for: offer, price, delivery model, niche, case studies
- If no mode specified, ask: "Which mode? (1) Prep a sales call, (2) Coach me on a call I just had, (3) After-close assets for a client I just signed."

## Step 1: Load context

1. Read `business.md` (required for all modes). All pricing, guarantee, case studies, and competitor data comes from business.md. Do not restate full lists of case studies in the script. Reference "proof from business.md" and pull the 1-2 most relevant examples for this specific prospect.
2. Read `.claude/rules/humanizer.md` for writing rules
3. Determine mode from $ARGUMENTS or ask

---

# MODE 1: PREP CALL

Run this 15-30 minutes before every sales call.

## Step 2a: Research the prospect

Use Perplexity MCP to research the prospect's business. Query should cover:

- Company website (services, pricing, team size, recent changes)
- Google reviews (rating, volume, common complaints, response patterns)
- Recent news or press mentions
- LinkedIn activity (recent posts, job changes, company updates)
- Local competitors (who else serves their area, how they compare)

**Error handling:** If Perplexity MCP is not connected, tell the user: "Perplexity MCP isn't connected. I can still build the call script from what you tell me, but the research will be thin. Connect Perplexity and re-run, or give me everything you know about this prospect." Ask for manual input and proceed.

## Step 3a: Generate 3 personalized observations

Distill the research into exactly 3 observations. These are specific things to mention on the call that show homework was done.

Rules for observations:
- Must be specific and verifiable (not "I love your business")
- Reference numbers, dates, or concrete details
- Each one should surface a potential pain point

Example: "I noticed your Google rating dropped from 4.6 to 4.2 since January, and most of the new negative reviews mention wait times."

## Step 4a: Build the 20-minute timed call script

Structure the script into 4 timed sections:

### Rapport (0:00-2:00)
- 2-3 opener options based on research
- Personal or specific, not generic
- Example: "I saw you just opened your second location, congrats. How's the transition going?"

### Discovery (2:00-10:00)
- 5-7 questions designed to surface pain
- Questions built from the research, not generic
- Goal: get the prospect talking about problems in their own words
- Example: "Your reviews mention follow-up communication. Is that something your team handles manually right now?"

### Solution (10:00-15:00)
- Bridge from the pain they described to the service
- Format: "You mentioned X. Here's exactly how I'd handle that."
- If case studies exist in business.md, pull a before/after comparison

### Price presentation (15:00-20:00)
- Price anchoring math (calculated below)
- The 3 most likely objections, customized to this prospect
- Instruction: "Say your price out loud five times before the call. Standing up."

## Step 5a: Calculate price anchoring

1. **Hours saved per month.** Estimate based on the prospect's current manual process.
2. **Hourly rate of the person doing that work.** Office manager, owner, etc.
3. **Revenue impact.** Additional revenue from the service (more leads, more reviews, etc.).
4. **Monthly value created:** Hours saved x hourly rate + revenue impact.
5. **Your price:** 30-50% of monthly value (the sweet spot).
6. **Client ROI:** For every $1 spent, $X returned.

Format the anchor script:
1. State the problem cost
2. Add the hidden cost
3. Name the total
4. Present the price (then: "Pause. Don't explain. Don't justify. Don't fill the silence.")
5. Present the stacked guarantee (both layers from business.md). After the price anchor, this is the closer. Layer 1 removes risk. Layer 2 removes the "what if it takes longer" fear. Present both, then ask for the sale.
6. Frame the ROI

### Belief-objection reframe

When the prospect says "I don't think this works" or "I don't believe people who need help can succeed," reframe by naming the pattern they're describing: "You're describing what happens when people stay in learning mode without shipping. Learning feels safe. Shipping feels scary. That's exactly why the system forces shipping. More information is not the answer. A forcing function is." The reframe works because it names the trap they're stuck in, not the objection. It shifts the conversation from "does this work?" to "are you going to keep researching, or actually ship?"

## Step 6a: Output the prep document

```
=== CALL PREP: [Prospect Name], [Business Name] ===
Call time: [if known]

--- RESEARCH ---
Business: [name, location]
Google: [stars, review count]
Signal: [key finding]
Website: [key observations]
Team: [size estimate]
Competitors: [count within area, avg rating]

--- 3 OBSERVATIONS ---
1. [specific, verifiable observation]
2. [specific, verifiable observation]
3. [specific, verifiable observation]

--- TIMED SCRIPT ---
[00:00-02:00] RAPPORT
[2-3 opener options]

[02:00-10:00] DISCOVERY
Q1: [research-based question]
Q2: [research-based question]
Q3-Q7: [additional questions]

[10:00-15:00] SOLUTION
[Bridge from pain to service]

[15:00-20:00] PRICE
Anchor: $[monthly value] (problem cost)
Your price: $[your price]
ROI: $[X] returned per $1 spent

--- TOP 3 OBJECTIONS ---
1. "[likely objection]" -> [response]
2. "[likely objection]" -> [response]
3. "[likely objection]" -> [response]

--- PRACTICE ---
Stand up. Say "$[price]" five times.
```

---

# MODE 2: SALES COACH

Run after a sales call with a transcript or notes.

## Step 2b: Get the transcript

Ask: "Paste your transcript, give me the file path, or jot your notes here. Transcripts give better coaching, but bullet points work too."

If they provide a file path, read it. If they paste text, use that. If they have notes, work with what they give.

## Step 3b: Score 5 dimensions

Analyze the transcript/notes and score each dimension. Provide a score and specific evidence from the call for each.

### 1. Discovery depth
- Did they uncover a specific pain with a dollar or hour figure attached?
- Or did they stay surface-level?
- Score: 1-10

### 2. Talk ratio
- What percentage did they talk vs. the prospect?
- Target: user 30-40%, prospect 60-70%
- If user talked more than 50%, they pitched too early
- Score: 1-10

### 3. Objection handling
- When the prospect pushed back, how did they respond?
- Did they fold, get defensive, or acknowledge and redirect?
- Identify each objection and rate the response
- Score: 1-10

### 4. Price presentation
- Did they anchor before stating the price?
- Did they pause after?
- Did they immediately offer a discount nobody asked for?
- Score: 1-10

### 5. Close attempt
- Did they actually ask for the sale?
- "So, want to move forward?" is better than "Let me send you a proposal"
- Score: 1-10

## Step 4b: Provide specific feedback

For each dimension, give actionable feedback tied to exact moments in the call. Not "do better at discovery." More like:

"You asked 'How are you handling reviews?' and the prospect said 'We try to respond when we can.' You moved on. This was the moment to dig deeper: 'How many hours a week does that actually take?' You left money on the table by not quantifying the pain."

For objections, identify the exact moment and provide an alternative response.

## Step 5b: Pattern tracking

If /close-deal sales-coach has run before and previous call analyses exist in business.md or delivery files, compare across calls. Identify recurring patterns:

- Do they consistently skip ROI framing?
- Do they always rush discovery?
- Do they handle the same objection the same weak way every time?

Patterns are more valuable than individual feedback.

## Step 6b: Output

```
=== SALES COACH: [Prospect Name] ===
Date: [YYYY-MM-DD]

--- SCORES ---
Discovery depth:     [score]/10
Talk ratio:          [score]/10 (you: [X]%, prospect: [Y]%)
Objection handling:  [score]/10
Price presentation:  [score]/10
Close attempt:       [score]/10
TOTAL:               [total]/50

--- FEEDBACK ---
[Specific, moment-by-moment feedback for each dimension]

--- PATTERNS ---
[Cross-call patterns if multiple analyses exist]

--- TOP 2 IMPROVEMENTS FOR NEXT CALL ---
1. [Specific action]
2. [Specific action]
```

---

# MODE 3: AFTER CLOSE

Two separate triggers. The signup sequence fires immediately when the deal closes. The first-result sequence fires later, after the client gets their first win. Do not combine them.

## Trigger A: Signup sequence (fires at signup)

Run this the moment the deal closes.

### Step 2c-A: Gather client data

Read from `delivery/clients/[name]/` if it exists. If not, ask for:
- What was the problem before?
- What did you do?
- What's the agreed scope and timeline?

### Step 3c-A: Generate welcome assets and mark the first client milestone

#### First client milestone

When you close your first paying client, mark it. Not a ceremony, just an honest checkpoint. Revenue changed. Post about it if you want. Then get back to work on delivery.

Rough milestones worth tracking:
- First paying client
- $5K/month
- $10K/month
- $30K MRR

#### Welcome message

Draft a welcome message covering:
1. Congrats and excitement about working together
2. What to expect in the first 48 hours (onboarding steps, access, first deliverable timeline)
3. Any prep work they need to do before you start

#### "Before" story capture

Ask the new client to write 2-3 sentences about where they are right now. This becomes the "Before" snapshot for a future case study. Prompt them: "Before we start, I'd like to capture where things stand today. In 2-3 sentences, describe your current situation with [the problem your service solves]. This helps us measure progress and tell the story later." Save their response in `delivery/clients/[name]/before-story.md`.

### Step 4c-A: Output (signup sequence)

```
=== SIGNUP SEQUENCE: [Client Name] ===

--- WELCOME MESSAGE ---
[Welcome text with first-48-hours expectations]

--- "BEFORE" STORY CAPTURE ---
[Prompt for the client to describe their current situation]

--- MILESTONE ---
[If first client: mark it. Revenue changed. Move on.]
[If not first client: skip this section]
```

---

## Trigger B: First-result sequence (fires after first client win)

Run this after the client gets their first measurable result. Not at signup. After the result lands.

### Step 2c-B: Gather result data

Read from `delivery/clients/[name]/` if it exists. Pull the "before" story from `delivery/clients/[name]/before-story.md`. Ask for:
- What are the results so far? (specific numbers)
- How long did it take from start to this result?

### Step 3c-B: Generate case study

#### Case study format

Before / What / After / Timeline

```
BEFORE: [The problem, in the client's situation. Specific numbers. Pull from before-story.md.]
WHAT: [What was done. Actions, not process.]
AFTER: [Results. Specific numbers.]
TIMELINE: [How fast.]
```

If data is incomplete, create a template with blanks and tell what to fill in. Never fabricate metrics.

Save to `delivery/case-studies/[client-name].md` or suggest a location.

### Step 4c-B: Generate testimonial request

Draft a testimonial request. Not "can you write a testimonial?" Use the 3-question format:

1. What was your situation before we started?
2. What specific result have you seen so far?
3. What surprised you about working with me?

Include: "I'll draft something from your answers and send it back for your approval before publishing anything."

Timing instruction: "Send this within 48 hours of delivering a result. The numbers are fresh and the client is feeling the impact."

### Step 5c-B: Generate referral asks

Three versions of the same ask for different channels:

**Email version:** 4-5 sentences. Opens with specific result, thanks them, makes the ask. Uses "who else" (not "do you know anyone").

**LinkedIn DM version:** 3 sentences. Shorter, more casual.

**Text message version:** 2 sentences. Direct. For clients with a personal relationship.

Each version follows:
1. Reference the specific result delivered
2. Brief thanks
3. Ask: "Who else do you know who might need this?"
4. Make it easy: "Just an intro is all I need. I'll handle the rest."

Also generate a follow-up for 3 days later. The follow-up does NOT repeat the ask. It shares something useful (an article, a result update) and ends with a softer version of the referral request.

### Step 6c-B: Output (first-result sequence)

```
=== FIRST RESULT: [Client Name] ===

--- CASE STUDY ---
BEFORE: [situation, from before-story.md]
WHAT: [actions]
AFTER: [results]
TIMELINE: [duration]

Saved to: [file path]

--- TESTIMONIAL REQUEST ---
Subject: [subject line]
[Email body with 3 specific questions]

--- REFERRAL REQUEST (Email) ---
Subject: [subject line]
[4-5 sentences]

--- REFERRAL REQUEST (LinkedIn DM) ---
[3 sentences]

--- REFERRAL REQUEST (Text) ---
[2 sentences]

--- 3-DAY FOLLOW-UP ---
[Useful share + softer referral ask]
```

---

## Self-check before finishing

1. Correct mode was identified and executed
2. For prep-call: 3 observations are specific (not generic), script covers all 4 time blocks, price anchoring math is calculated, stacked guarantee presented as the closer (both layers)
3. For sales-coach: all 5 dimensions scored with evidence from the call, feedback references specific moments, belief-objection reframe suggested where relevant
4. For after-close: two separate triggers (signup sequence at close, first-result sequence after first win). Signup: welcome message with first-48-hours expectations, "before" story capture saved to delivery/clients/[name]/before-story.md, first-client milestone marked if applicable. First-result: case study has Before/What/After/Timeline using before-story.md, testimonial request uses 3-question format, referral asks use "who else" phrasing. Referral asks only in first-result sequence, never in signup sequence.
5. business.md read for context. Case studies and pricing referenced, not fully restated (pull 1-2 most relevant for this prospect)
6. No Tier 1/2 banned words from humanizer rules
7. No em dashes anywhere in the output

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Split Mode 3 (after-close) into two triggers. Signup sequence (fires at close): welcome message, first-48-hours expectations, "before" story capture for future case study. First-result sequence (fires after first client win): case study using before-story.md, testimonial request (3-question format), referral asks. Moved referral asks out of signup and into first-result only. |
| 2026-03-30 | Full rewrite. Restructured into 3 explicit modes (prep-call, sales-coach, after-close). Added: Perplexity research for prep-call, 3 personalized observations, 20-minute timed script with 4 sections, price anchoring math (5-step calculation), 5-dimension sales coaching, pattern tracking across calls, case study (Before/What/After/Timeline), question-based testimonial requests, 3-channel referral asks with follow-up. |
