---
name: ad-script
description: Generates YouTube ad scripts in 3 formats (30s skippable pre-roll, 15s non-skippable, 60s Shorts) from your best organic videos and testimonials. Uses YouTube MCP to find top-performing content. Outputs scripts with timestamps and visual direction.
user-invocable: true
---

# YouTube ad script generator

Create ad scripts from your best organic YouTube content. Three formats: 30-second skippable pre-roll, 15-second non-skippable, and 60-second Shorts vertical.

## Step 1: Load context

1. Read `marketing/offer_structure.md` for the current offer, pricing, and positioning. If missing, check `business.md`.
2. Read `marketing/testimonials/` for all available testimonials and proof points. If missing, ask: "I need at least 2 client results to write ad scripts. Paste them here."
3. Read `.claude/rules/humanizer.md` for writing rules (all scripts must pass these).

## Step 2: Find your best organic content via YouTube MCP

1. Call `youtube_analytics_top_videos` to get videos ranked by watch time and views.
2. For the top 5 videos, call `youtube_analytics_retention` to check audience retention curves.
3. The video with the highest average percentage viewed = your best ad candidate.
4. Call `youtube_get_transcript` on the top 3 videos to extract the hooks and key moments.

If YouTube MCP is not connected, ask: "Which of your YouTube videos performs best? Paste the title and a summary of what it covers."

## Step 3: Extract hooks and proof points

From organic content and testimonials, pull:

- The strongest opening hooks (first 15 seconds of top videos)
- Specific client results (dollar amounts, timelines, before/after)
- Counterintuitive claims or contrarian angles
- Identity statements ("Built for solo founders who...")
- Process reveals ("Here's exactly how...")

List all extracted hooks and proof points before writing scripts.

## Step 4: Write the 30-second skippable pre-roll script

Constraints: 30 seconds = 70-80 words. Skip button appears at 5 seconds.

```
[0-2s] HOOK: Pattern interrupt. Bold claim, striking stat, or direct question.
       No logos. No intros. No "hey guys."
[2-5s] BRAND + PROOF: Connect the hook to your expertise. One credential or result.
       Must establish who you are BEFORE the skip button appears.
[5-15s] VALUE: Teach something specific. A framework, a number, a story.
        This is the part that makes them keep watching instead of skipping.
[15-25s] OFFER: What they get. Be specific. Pricing optional.
[25-30s] CTA: "Click the link below" or "check the description."
         Show URL on screen for full 3 seconds.
```

Rules:
- The 0-5s section must work as a standalone micro-ad (many will skip after 5s)
- No banned words from humanizer.md
- Every script needs at least one specific number
- CTA must be direct, not "learn more"
- Conversational tone. This plays before someone else's video. Match YouTube energy.

## Step 5: Write the 15-second non-skippable script

Constraints: 15 seconds = 35-40 words. No skip button. Viewer watches everything.

```
[0-3s] HOOK: One sentence. Grab attention immediately.
[3-9s] PROOF: One specific result or transformation.
[9-13s] BRIDGE: Connect proof to the viewer's situation.
[13-15s] CTA: Tell them what to do. One action.
```

Rules:
- Every word earns its place. Cut anything that isn't working.
- Audio clarity matters (plays before videos, often at volume)
- CTA is spoken AND shown on screen

## Step 6: Write the 60-second Shorts vertical script

Constraints: 60 seconds max but aim for 30-45 seconds. Vertical 9:16. Non-skippable in Shorts feed.

```
[0-2s] HOOK: Start mid-action. No title cards. No slow builds.
       First 2 seconds decide if they swipe away.
[2-15s] PROBLEM: Call out the frustration or situation. Be specific.
[15-35s] SOLUTION: Show or explain the fix. One clear thing.
[35-45s] PROOF: Result, testimonial, or before/after.
[45-55s] CTA: Direct ask. "Link in bio" or "Follow for more."
```

Rules:
- On-screen text is required (many watch Shorts with sound off)
- Fast cuts. High energy. No slow pacing.
- Native-feeling beats polished production
- Do NOT letterbox a horizontal video. This must be vertical-first.

## Step 7: Add visual direction notes

For each script, include 3-5 bullet points:
- What the viewer sees at each beat (talking head, screen recording, text overlay, b-roll)
- On-screen text overlays (what text, when, duration)
- End card layout (CTA placement, URL display)

## Step 8: Word count and timing check

For each script:
1. Count total words
2. Estimate timing at 2.5 words/second
3. If 30s script exceeds 80 words, trim
4. If 15s script exceeds 40 words, trim
5. If Shorts script exceeds 120 words, trim
6. Flag any sentence longer than 15 words

## Step 9: Save output

Save to `temp/{YYYY-MM-DD}_ad_scripts.md` with all three scripts, visual notes, word counts, and source video info.

## Example output (30s skippable)

```
30-SECOND SKIPPABLE PRE-ROLL
Source: "How I Built a $4,200/Mo AI Service" (your top video by retention)
Word count: 74 words (29.6 seconds at 2.5 wps)

[0-2s]
"I built a $4,200 per month business with one tool."
(On screen: $4,200/mo text overlay, dark background)

[2-5s]
"No coding background. Zero clients when I started. Just Claude Code."
(On screen: talking head, close-up)

[5-15s]
"Here's what I did. I picked one niche, built one offer in a weekend,
and sent 30 DMs. Client number one paid $1,400 on day nine."
(On screen: talking head with Stripe notification b-roll)

[15-25s]
"I put the exact system on my site. One tool. One offer.
Real clients from week one. No fluff."
(On screen: platform preview, testimonial flash)

[25-30s]
"Click below to see it."
(On screen: CTA button animation, site URL visible for 3s)

VISUAL NOTES:
- Open on close-up talking head, slight push-in
- $4,200/mo as large text overlay at 1s, fades at 3s
- Stripe notification as quick b-roll at 8s
- Platform UI preview at 18s (2 seconds)
- End with CTA lower-third, URL visible for full 3s
```

## Self-check

1. All three scripts written (30s, 15s, Shorts)
2. Word counts verified and within limits
3. Every script includes at least one specific number
4. No humanizer banned words
5. 0-5s section of skippable script works as standalone hook
6. Shorts script is designed for vertical, not repurposed horizontal
7. Visual notes included for each script
8. Source video identified (from YouTube MCP or user input)
9. Output saved to temp/ with date prefix

## Error handling

- **No YouTube MCP connection:** Ask user to name their best-performing video and describe the first 30 seconds.
- **No testimonials found:** Ask: "I need at least one specific client result. Paste a testimonial or result with numbers."
- **User wants more variations:** Write the 3 base scripts first. Then offer: "Want me to write A/B variants with different hooks?"
- **Script runs over time limit:** Auto-trim, show what was cut, explain why.

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
