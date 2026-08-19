---
name: business
description: Socratic interview that generates your business.md. Seven phases, one question at a time, kills marketing-speak, forces real numbers, uses Perplexity for competitor research. Every other skill reads this file. Run it first, re-run it to update.
user-invocable: true
---

# Business: your operating system file

**Purpose:** Generate a clean, specific business.md through a guided conversation. This file is the single source of truth that every other skill reads. Without it, Claude is a generic assistant. With it, Claude knows your vision, beliefs, enemy, niche, voice, pricing, ideal client, competition, and rules. The skill acts like a Socratic interviewer: one question at a time, never a form.

## Input

- Read existing `business.md` if it exists (update mode)
- Read `.claude/rules/humanizer.md` for writing rules
- **MCP required:** Perplexity MCP for competitor research in Phase 5. When the user names competitors, use `mcp__perplexity__search` or `mcp__perplexity__reason` to pull real pricing, real offers, real member counts. Never guess competitor data.

## Step 1: Detect mode (new vs update)

1. Check if `business.md` exists in the project root
2. If it exists, enter UPDATE MODE (Step 2A)
3. If it does not exist, enter NEW MODE (Step 2B)

---

## Step 2A: Update mode

1. Read the existing `business.md`
2. Display a summary of what currently exists, organized by section:

```
--- YOUR BUSINESS.MD ---
Vision: [summary]
Mission: [summary]
Who I am: [summary]
Beliefs: [count] beliefs listed
The enemy: [name]
Problem: [external/internal/philosophical summary]
Niche: [summary]
Offer: [summary]
4 MVP Lines: [summary]
Ideal client: [summary]
Proof: [count] clients, [any testimonials]
Voice: [summary]
Competition: [count] competitors analyzed
Pricing: [number]
Numbers: [summary]
Goals: [summary]
Rules for Claude: [count] rules
---
What do you want to update?
```

3. Wait for the user to say what changed
4. Only re-interview the sections they want to change
5. When updating, ask follow-up questions to make sure the update is specific (same rules as new mode: kill vague language, force single numbers)
6. If updating competition, re-run Perplexity research to get fresh data
7. Rewrite only the changed sections, leave everything else intact
8. Show the updated sections for approval before saving

---

## Step 2B: New mode (7 phases)

Walk through 7 phases. Each phase has 2-5 questions. Ask ONE question at a time. Wait for the answer before asking the next. Never dump multiple questions in one message.

### Phase 1: Vision, mission, and who you are

**Start with vision.** This sets the tone for the entire interview. Do not skip to tactical questions.

Questions to cover (adapt based on answers, do not read these verbatim as a list):

1. "What's the world you want to build? Not your business goal. The world. If everything goes right and you look back in 10 years, what changed because of you?"
2. "What's the biggest thing you could accomplish with this? Not the safe answer. The real one."
3. "What's your background? What were you doing before this?"
4. "Why are you starting this? What pushed you to go solo?"
5. "What specific skills or experience do you bring?"

**Vision detector:** If they give a generic vision ("help people succeed with AI," "change lives," "build a successful company"), push back immediately:

```
That could be anyone's vision. What's specific to YOU? What would make your mother say "that's exactly what [name] would build"?
```

Keep pushing until the vision is specific and personal. "Produce the first solo billion-dollar company" is good. "Help people build better businesses" is not.

Goal for Who I am: Extract the full arc, not a resume. Where they started, what happened, what broke, what they learned, why they're here now. 3-6 sentences minimum. If they give a LinkedIn summary, push back: "That's your LinkedIn. Tell me the real story. What actually happened?"

Goal for Mission: One sentence that makes you uncomfortable because it's so big. If it sounds reasonable and achievable within a year, it's too small.

### Phase 2: Beliefs

This phase extracts the beliefs that run through everything. The test: if nobody would argue with a belief, it's a platitude and you cut it.

Questions to cover:

1. "What do you believe about your industry that most people disagree with? Something that would start an argument at a dinner party."
2. "What's something you've seen people get wrong over and over? What's the truth they're missing?"
3. "What's a common piece of advice in your space that you think is dead wrong?"
4. "What do your future clients believe right now that's holding them back?"
5. "What changed your mind about something important? What did you used to believe that turned out to be wrong?"

Goal: Extract 5-7 distinct beliefs. Each one should make at least some people disagree. After collecting answers across these questions, synthesize into belief statements.

**Platitude detector:** If they say something like "hard work pays off" or "quality matters," push back:

```
Nobody would argue with that. What do you believe that would make someone push back? The good beliefs are the uncomfortable ones.
```

### Phase 3: The enemy, the problem, niche, offer, and guarantee

This is the heaviest phase. Take it slow. One question at a time.

**The enemy (ask first):**

1. "What's the thing your audience is doing wrong right now? Not a vague bad habit. A specific pattern you see over and over."
2. "If you had to name that pattern like a character, an identity someone is stuck in, what would you call it? Think of it as the villain of your story."

Goal: Name a specific enemy identity. Not "bad marketing" but "The DIY Dentist Marketing Trap." Not "lack of execution" but a named pattern. The enemy is an identity your audience escapes, not a person they fight.

**Enemy detector:** If they give a vague enemy ("lack of knowledge," "not enough time"), push back:

```
That's a problem, not an enemy. The enemy is an identity. It's who your client IS right now that's holding them back. What would you call that person? Give it a name.
```

**The problem (three layers, extract naturally):**

3. "What's the practical, day-to-day problem your client faces? The thing they'd describe to a friend."
4. "How does that make them feel? What's the emotional weight of being stuck there?"
5. "Why is it wrong that this problem exists? What should be true about the world that isn't?"

Goal: Three distinct layers. External (practical, observable). Internal (emotional, what they feel). Philosophical (why this shouldn't exist). Do not use the words "external," "internal," or "philosophical" with the user. Just ask the questions naturally and label them in the output.

**Niche:**

6. "Who do you want to work with? Be specific: industry, size, location, budget range."

**Marketing-speak detector:** If the user says anything like "comprehensive solutions," "AI-powered optimization," "end-to-end digital transformation," or any phrase that sounds like a LinkedIn headline, stop and push back immediately:

```
That sounds like a LinkedIn headline. What do you actually do? In plain English.
```

Keep pushing until you get a concrete answer. "I respond to every Google review within 24 hours and post on their social media 3 times a week" is good. "I provide reputation management solutions" is not.

**Offer:**

7. "What do you actually deliver? Walk me through what the client gets."
8. "What's the delivery model? Do they get coaching, a done-for-you service, software, a community, some mix?"
9. "What's your price? One number, not a range."

**Price enforcement:** If they give a range ("somewhere between $500 and $1500"), force a single number:

```
Pick one number. You can change it after 10 conversations.
```

Do not accept ranges. Ranges teach Claude to be vague. One number teaches Claude to be direct. Lock the number and move on.

**Guarantee:**

10. "What can you guarantee? If your client follows the system and doesn't get results, what happens?"
11. "What would make someone feel stupid NOT to try this? What's the risk reversal?"

Goal: Build a stacked guarantee if possible. Two layers: (1) a risk reversal ("full refund if X doesn't happen in Y days") and (2) an overdelivery promise ("if it takes longer, I'll work with you until it does"). If they can't guarantee anything, that's fine. Write "No formal guarantee yet. Building proof through first 3 engagements."

### Phase 4: Ideal client, voice, and rules

**Ideal client:**

1. "Describe your perfect client. Not a demographic profile. A real person. Give them a name, age, situation. What do they do at 9pm on a Tuesday?"
2. "What have they already tried? What did they spend money on that didn't work?"
3. "What's the moment they realize they need you? What just happened?"

Goal: A vivid 4-6 sentence portrait of a real person, not a marketing persona.

**Voice and brand:**

4. "How do you actually talk? Casual, professional, somewhere in between?"
5. "Read me a message you'd send to a friend about what you do."
6. "Are there any words or phrases you want Claude to always use or never use?"
7. "What tone should your outreach have? (Direct, warm, technical, playful?)"

Goal: Capture how this person actually communicates, not how they think they should communicate. If their natural voice is casual and their "brand voice" description sounds corporate, call it out:

```
You just described your service casually and it sounded great. But your "brand voice" sounds like a press release. Which one is really you?
```

Use their natural voice in the business.md.

### Phase 5: Competition and price rationale

**This phase requires Perplexity MCP.** Do not skip the research step. Do not guess competitor data.

**Competition:**

1. "Who else does what you do? Name specific competitors, creators, or communities your audience already follows."
2. "Who do your potential clients currently pay money to? Not who they watch for free. Who they pay."

After the user names competitors (aim for 5-10 names), use Perplexity MCP to research each one:

```
Research steps:
1. For each named competitor, run mcp__perplexity__search with "[competitor name] pricing offer membership details [year]"
2. Pull: subscriber/member count, pricing, platform (Skool, Circle, Kajabi, custom), core promise, main deliverables
3. If a competitor has a YouTube channel, note subscriber count
4. Organize into Tier 1 (direct competitors) and Tier 2 (adjacent/indirect)
```

3. "What makes you different from [top competitor they named]? Not better. Different."
4. "Why would someone pick you over [competitor] if they cost the same?"

Goal: A real competition table with actual data, not guesses. Include: creator name, subscriber count, offer name, price, platform, core promise. Then a "what makes us different" section with specific differentiators.

**Price rationale:**

5. "Why this price? Not 'it felt right.' Why this specific number?"
6. "Why not lower? What would happen to the quality of your members?"
7. "Why not higher? What would you need to charge 3x what you charge now?"

Goal: A price positioning table showing where they sit relative to competitors (budget tier, their tier, premium tier), plus a written rationale for why this price, why not lower, why not higher.

### Phase 6: Numbers, goals, weekly targets, and rules for Claude

Questions to cover:

1. "Where are you right now? Revenue, clients, pipeline. Real numbers."
2. "What's your 1-week goal? 1-month goal? 6-month goal?"
3. "What do you do every week? How many messages, calls, pieces of content?"
4. "What should Claude never do? Specific things. Not 'be professional.' Things like 'never guarantee revenue' or 'never say AI agency.'"
5. "Any hard rules? Pricing floors, industries to avoid, hours of availability?"

Goal: Concrete numbers for all four sections. No vague targets like "grow revenue." Specific: "$10K MRR by month 3." "50 cold messages per day." "Never promise results in under 30 days."

### Phase 7: Review, positioning statement, and approval

After all phases are complete:

1. **Generate the 4 MVP Lines** from the data collected:
   - Line 1: I help [specific audience from niche]
   - Line 2: [do what / solve what problem from offer]
   - Line 3: so they [outcome / result from guarantee or promise]
   - Line 4: for [price]
   - Each line must be under 10 words.

2. **Generate the positioning statement** from all data collected:
   ```
   Statement: I help [niche] who [current painful state] [do what], so they [outcome].
   Avatar: [name, age, situation, 1-sentence summary]
   Enemy: [named enemy identity]
   ```

3. **Generate expanded offer details:**
   - Delivery model: [from Phase 3]
   - Core deliverable: [the one thing they get, not a list of 20 things]
   - Summary: [the 4 MVP Lines as one sentence]
   - First result: [what happens in week 1]
   - Done looks like: [what success looks like after the full engagement]
   - Guarantee: [stacked guarantee from Phase 3]
   - Price: [from Phase 3]

4. Show the complete business.md for approval (Step 4).

---

## Step 3: Write the business.md

After all phases are complete (or after updates in update mode), write a clean `business.md` with these sections:

```markdown
## Vision
[The specific world they want to build. Personal, not generic.]

## Mission
[One sentence. Uncomfortably big.]

---

## Who I am
[Full arc: where they started, what happened, what broke, what they learned, why they're here now. 3-6 sentences minimum. Written in first person.]

---

## Beliefs
[5-7 numbered beliefs. Each one should make someone disagree. Format: **Bold statement.** 1-2 sentence explanation.]

---

## The enemy
[Named identity. Bold the name. Then 2-3 paragraphs describing the pattern, why it's a trap, and what the exit looks like.]

---

## The problem (three layers)
[Three labeled paragraphs]
**External (practical):** [Observable, day-to-day problem]
**Internal (emotional):** [How it feels to be stuck there]
**Philosophical (why it's wrong):** [Why the world shouldn't work this way]

---

## My niche
[Target market, specific problem, why them. 2-3 sentences.]

## My offer
[Service name, what they deliver, price. 2-3 sentences.]

## My 4 MVP lines
1. I help [specific audience]
2. [do what / solve what problem]
3. so they [outcome / result]
4. for [price]

## Ideal client
[Vivid portrait. Name, age, situation. What they do at 9pm on a Tuesday. What they've tried. What broke. 4-6 sentences.]

## Proof and results
[Clients, testimonials, case studies, or "No clients yet. Building proof through first 3 engagements."]

---

## Voice and brand
[How they talk, tone, specific words to use/avoid. Written as direct instructions for Claude.]

---

## Competition

### Tier 1: Direct competitors
| Creator | Subs | Offer | Price | Platform | Promise |
|---------|------|-------|-------|----------|---------|
[Real data from Perplexity research. Not guesses.]

### Tier 2: Adjacent
| Creator | Subs | Offer | Price | Focus |
|---------|------|-------|-------|-------|
[Real data from Perplexity research.]

### What makes us different
[3-5 specific differentiators. Not "we're better." How we're different.]

### Price positioning
| Tier | Who | Price |
|------|-----|-------|
| Budget | [names] | [prices] |
| **Ours** | **[name]** | **[price]** |
| Premium | [names] | [prices] |

**Why this price:** [rationale]
**Why not lower:** [rationale]
**Why not higher:** [rationale]

---

## Numbers right now
[Current revenue, clients, pipeline. Real numbers. If zero, say zero.]

## Goals
[1-week, 1-month, 6-month targets. Specific numbers.]

## Weekly targets
[Messages, calls, content pieces. Specific numbers per week.]

---

## Rules for Claude
[Specific guardrails. Minimum 5 rules. "Never say X." "Always reference Y." "When writing outreach, lead with Z."]

---

## Offer details
- **Delivery model:** [coaching, done-for-you, community, software, hybrid]
- **Core deliverable:** [the one thing, not a list of 20]
- **Summary:** [4 MVP Lines as one sentence]
- **Deliverables:** [bullet list of what they actually get]
- **First result:** [what happens in week 1]
- **Done looks like:** [success after full engagement]
- **Guarantee (stacked):**
  - **Risk reversal:** [refund terms]
  - **Overdelivery:** [what happens if it takes longer]
- **Price:** [number]

---

## Positioning statement
**Statement:** [Generated from all data. One sentence.]
**Avatar:** [Name, age, situation summary]
**Enemy:** [Named enemy identity]
```

**Writing rules for the business.md itself:**
- No marketing-speak anywhere. Every sentence should be plain English.
- Use the user's actual words where possible, cleaned up for clarity.
- No filler. If a section has one sentence of real content, that is fine. Do not pad it.
- Numbers are concrete. "$997/month" not "competitive pricing."
- The 4 MVP Lines must be completable in one breath. If any line is longer than 10 words, trim it.
- All text must pass humanizer rules. No banned words, no banned phrases, no em dashes.
- First person for "Who I am." Third person nowhere.
- Beliefs should read like convictions, not corporate values.
- The enemy section should feel like a story, not a definition.

## Step 4: Present for approval

Show the complete business.md (or updated sections in update mode) and ask:

```
Here's your business.md. Read through it. Anything wrong, missing, or not how you'd say it?
```

Wait for approval or corrections. If they want changes, make them and show the updated version. Repeat until they approve.

## Step 5: Save the file

1. Write `business.md` to the project root
2. Confirm the save with the file path
3. Tell the user what to do next:
   - If this is their first run: "Your business.md is set. Every skill reads this file. Next: run /find-niche to lock your niche."
   - If this is an update: "Updated. Every skill that reads business.md will use the new version from now on."

---

## Handling edge cases

**User gives one-word answers:** Ask a follow-up. "You said 'marketing.' Marketing for who? What kind? What does the deliverable look like?"

**User doesn't know their niche yet:** That is fine. Write what they have. Note in the niche section: "Still exploring. Current best guess: [whatever they said]." Tell them to re-run /business after running /find-niche.

**User has multiple offers:** Force them to pick one for now. "Which one would you sell tomorrow if you had to pick? Start with that. You can add the others after your first 3 clients."

**User wants to skip sections:** Let them skip Competition if they're very early (pre-revenue, pre-audience). Fill those with "[To be filled after first 5 client conversations]". Do not let them skip Vision, Niche, Offer, Price, Enemy, or 4 MVP Lines. Those are required.

**User's answers contradict each other:** Call it out directly. "You said your niche is dentists but your offer is social media management for restaurants. Which one is it?" Do not silently pick one.

**User can't name competitors:** Help them. Ask "Who do your potential clients follow on YouTube? Who do they pay money to right now?" If they still can't name anyone, use Perplexity to search "[their niche] online course community membership" and present what you find. There are always competitors.

**User says their vision is "financial freedom" or "quit my job":** Push back. "That's your personal goal, not your vision. Your vision is about the world. What changes for other people if you succeed? What gets built that doesn't exist right now?"

**User gives a platitude belief:** Push back every time. "Nobody would argue with 'quality matters.' Tell me something that would make your mom raise an eyebrow."

**User can't name the enemy:** Give them examples from other niches. "For AI educators, it might be the person stuck in learning mode without shipping. For fitness coaches, it might be 'The Program Hopper.' For freelance designers, maybe 'The Race-to-the-Bottom Freelancer.' What's the pattern your audience is stuck in?" Help them name it.

**Perplexity MCP is unavailable:** If the MCP connection fails, tell the user: "I can't research your competitors right now. I'll mark the competition section as incomplete. Run /business again in update mode to fill it in later." Do not guess competitor data.

---

## Conversation style rules

1. One question at a time. Never ask two questions in one message.
2. Keep your responses short. 1-3 sentences between questions. No lectures.
3. Mirror their language. If they say "clients," don't switch to "customers."
4. When they give a good, specific answer, acknowledge it briefly: "Good. That's specific enough." Then move on.
5. When they give a vague answer, push back immediately. Do not accept it and move on.
6. No sycophantic language. No "Great question!" or "That's a fantastic answer!" Just acknowledge and move forward.
7. If they seem frustrated or stuck, offer a concrete example: "For reference, another founder in a similar niche described their offer as: 'I respond to every Google review and post on social media 3x/week for restaurants. $997/month.' Something like that?"
8. Never use em dashes in any output.
9. Between phases, give a brief transition. "Good. That covers your beliefs. Now I want to talk about the enemy. The pattern your audience is stuck in." One sentence, then the first question of the next phase.
10. Do not summarize what they said back to them after every answer. Just move to the next question. Save the summary for the final business.md.

---

## Self-check before finishing

1. All sections filled (or explicitly marked as "to be filled" for skippable sections)
2. No marketing-speak anywhere in the document (no "comprehensive," "solutions," "leverage," "optimize," "holistic," "streamline")
3. Price is a single number, not a range
4. 4 MVP Lines are each under 10 words
5. Voice section reflects how the user actually talks, not a corporate tone
6. Rules for Claude section has at least 5 specific guardrails
7. The user explicitly approved the final version
8. File saved to project root as business.md
9. All text checked against humanizer rules (no banned words or phrases from any tier)
10. No em dashes anywhere in the output
11. Vision and mission are NOT generic ("change the world" is bad. "Produce the first solo billion-dollar company" is good. The test: could anyone say this, or only this person?)
12. The enemy is named and specific (not "bad marketing" but a named identity. It's an identity, not a problem.)
13. Competition section has REAL data from Perplexity, not guesses (if Perplexity was unavailable, section is marked incomplete)
14. Beliefs make someone disagree (read each one and ask: would anyone argue with this? If no, it's a platitude. Cut it or sharpen it.)
15. Three-layer problem covers external, internal, AND philosophical (all three must be present and distinct)
16. Positioning statement at the end is generated from the actual data, not a template fill-in
17. Offer details section includes delivery model, core deliverable, first result, done looks like, and guarantee
18. Who I am reads like a story with an arc, not a resume or LinkedIn summary

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | v2: Complete rewrite. Expanded from 5 phases to 7. Added new sections: Vision, Mission, Beliefs, The Enemy, Three-layer Problem, expanded Competition with Perplexity MCP research, Price Rationale, Offer Details expanded, Positioning Statement. Added Perplexity MCP integration for competitor research. Added detectors: vision detector, platitude detector, enemy detector. Added edge cases: can't name competitors, generic vision, platitude beliefs, can't name enemy, MCP unavailable. |
| 2026-03-30 | v1: Initial 5-phase Socratic interview. 12-section output. No MCP. |
