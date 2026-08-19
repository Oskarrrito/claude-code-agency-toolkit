---
name: lego-analysis
description: "Break competitor videos into 7 structural bricks. Analyzes 5-10 videos and outputs which patterns to copy and which to make original."
argument-hint: "[video URLs or channel handle]"
user-invocable: true
---

# Lego analysis: 7-brick video breakdown

Break competitor YouTube videos into 7 structural bricks. Analyze 5-10 videos, build a comparison matrix, and identify which patterns appear so often they are safe to copy and which have enough variation that you should make them your own.

## Input

- `$ARGUMENTS`: one or more YouTube video URLs, a channel handle (e.g., @mkbhd), or a search query
- If multiple URLs, separate with spaces or commas
- If no argument provided, ask: "Give me video URLs, a channel handle, or a topic to search."

## Step 1: Resolve video IDs

**If URLs provided:**
Extract video IDs from each URL. YouTube URLs can be:
- `youtube.com/watch?v=VIDEO_ID`
- `youtu.be/VIDEO_ID`
- `youtube.com/shorts/VIDEO_ID`

**If channel handle provided:**
Use `mcp__youtube__youtube_search` to find the channel's top videos:
```
query: "channel:{handle}"
maxResults: 10
order: "viewCount"
```

Select the top 8-10 videos by view count. Skip any under 5 minutes (those are shorts or intros, not full content).

**If search query provided:**
Use `mcp__youtube__youtube_search`:
```
query: "{search query}"
maxResults: 15
order: "relevance"
```

Filter to videos over 5 minutes. Select top 8-10 by view count.

Print the video list:

```
VIDEOS TO ANALYZE ({N} total)
------------------------------
1. "{Title}" by {Creator} | {views} views | {duration} | {video_id}
2. ...
```

## Step 2: Pull transcripts

For each video, use `mcp__youtube__youtube_get_transcript`:
```
videoId: "{video_id}"
```

If a transcript is unavailable (private, no captions), skip that video and note it:
```
SKIPPED: "{Title}" - no transcript available
```

You need at least 5 transcripts to proceed. If fewer than 5, ask the user for more video URLs.

## Step 3: Identify the 7 bricks

For each video, analyze the transcript and identify these 7 structural bricks:

| Brick | What to look for | Typical location |
|---|---|---|
| 1. Hook | First 15-30 seconds. How do they grab attention? | 0:00-0:30 |
| 2. Intro/Setup | How do they frame the video's promise? | 0:30-2:00 |
| 3. First main point | What is the first substantive section? How do they teach it? | 2:00-5:00 |
| 4. Transition method | How do they move between sections? | Between sections |
| 5. Middle structure | How is the bulk of the video organized? (list, story, demo, comparison) | 5:00-end minus 2 min |
| 6. Proof format | How do they prove their points? (data, demo, case study, screen share) | Throughout |
| 7. Outro/CTA | How do they end? What do they ask the viewer to do? | Last 1-2 minutes |

For each brick, write:
- **Type:** a 2-5 word label (e.g., "bold claim hook," "numbered list structure," "screen share demo")
- **Transcript excerpt:** 1-3 sentences that show this brick in action
- **Duration:** approximate timestamp range

## Step 4: Build the comparison matrix

Create a matrix with videos as rows and bricks as columns:

```
BRICK COMPARISON MATRIX
========================

| Video | Hook | Intro | First point | Transition | Middle | Proof | Outro |
|-------|------|-------|-------------|------------|--------|-------|-------|
| V1: "{short title}" | bold claim | personal story | screen demo | "next up..." | tutorial steps | live demo | sub + link |
| V2: "{short title}" | question | problem setup | framework | visual cut | numbered list | data slide | community CTA |
| V3: "{short title}" | result tease | credential | walkthrough | chapter marker | chronological | before/after | course pitch |
| ... | ... | ... | ... | ... | ... | ... | ... |
```

## Step 5: Find the 5 most common patterns

Count how many times each brick type appears across all videos. The 5 most common patterns are "safe to copy" because the audience expects them.

For each of the top 5:
- Name the pattern
- Count: appears in X of Y videos
- Why it works (1 sentence from the transcripts)
- How to apply it to your video

```
COPY THESE (5 most common patterns)
-------------------------------------

1. {Brick}: {Pattern name}
   Frequency: {X}/{Y} videos
   Example: "{transcript excerpt from highest-performing video}"
   Apply: {how to use this in your video}

2. ...
```

## Step 6: Find the 2 with most variation

The 2 bricks with the most variation across videos are where you should be original. If everyone does the same hook, copy the hook. If hooks vary wildly, that is your chance to stand out.

Measure variation: count unique brick types per position. The positions with the most unique types have the most variation.

```
MAKE THESE YOUR OWN (2 most varied bricks)
--------------------------------------------

1. {Brick}: {N} different approaches found
   Approaches: {list the unique types}
   Recommendation: {which approach fits your style and why}

2. {Brick}: {N} different approaches found
   Approaches: {list the unique types}
   Recommendation: {which approach fits your style and why}
```

## Step 7: Output the full analysis

Save to `temp/{YYYY-MM-DD}_{topic}_lego_analysis.md` and print the summary:

```markdown
# Lego analysis: {topic}

Date: {YYYY-MM-DD}
Videos analyzed: {N}
Source: {channel handle / search query / manual URLs}

---

## Brick comparison matrix

{matrix from Step 4}

---

## Copy these (5 most common patterns)

{from Step 5, with transcript excerpts and applications}

---

## Make these your own (2 most varied bricks)

{from Step 6, with approaches and recommendations}

---

## Per-video breakdown

### Video 1: "{Title}" by {Creator}

| Brick | Type | Transcript excerpt | Timestamp |
|---|---|---|---|
| Hook | {type} | "{excerpt}" | 0:00-0:20 |
| Intro | {type} | "{excerpt}" | 0:20-1:30 |
| First point | {type} | "{excerpt}" | 1:30-4:00 |
| Transition | {type} | "{excerpt}" | - |
| Middle | {type} | "{excerpt}" | 4:00-12:00 |
| Proof | {type} | "{excerpt}" | throughout |
| Outro | {type} | "{excerpt}" | 12:00-13:15 |

### Video 2: ...

{Repeat for each video}

---

## Reference

| # | Title | Creator | Views | Duration | Video ID |
|---|-------|---------|-------|----------|----------|
| 1 | {title} | {creator} | {views} | {duration} | {id} |
| ... | ... | ... | ... | ... | ... |
```

## Example output (abbreviated)

```
COPY THESE (5 most common patterns)
-------------------------------------

1. Hook: Bold claim with number
   Frequency: 7/9 videos
   Example: "This one tool replaced my entire tech stack." (from @techcreator, 1.2M views)
   Apply: open with a specific result number tied to Claude Code

2. Middle: Step-by-step tutorial structure
   Frequency: 8/9 videos
   Example: walks through each setting screen by screen with narration
   Apply: use numbered steps for the setup walkthrough, match viewer expectation

3. Proof: Live screen share demo
   Frequency: 6/9 videos
   Example: shows real terminal output with actual code running
   Apply: demo the actual MCP server connection live, do not use slides

4. Transition: "Next up..." verbal bridge
   Frequency: 6/9 videos
   Example: "Now that we have that set up, the next thing you need is..."
   Apply: simple verbal transitions between setup steps, no fancy cuts needed

5. Outro: Subscribe + specific next video recommendation
   Frequency: 7/9 videos
   Example: "If you want to see what I built with this, watch this video next."
   Apply: end with a pointer to the next video in the series

MAKE THESE YOUR OWN (2 most varied bricks)
--------------------------------------------

1. Intro: 6 different approaches found
   Approaches: personal story, credential flex, problem statement, "most people do X wrong",
               viewer promise, cold open (no intro)
   Recommendation: use "most people do X wrong" for setup videos. It creates tension and
                   positions you as someone who knows the right way. Matches our contrarian angle.

2. First point: 5 different approaches found
   Approaches: "why this matters" context, jump straight to demo, comparison with alternatives,
               prerequisite checklist, origin story
   Recommendation: jump straight to demo. Setup videos lose viewers in the first 2 minutes
                   if they do not see the tool yet. Show Claude Code running within 90 seconds.
```

## Error handling

- YouTube MCP not connected: ask user to paste video URLs, then use Apify YouTube scraper as fallback
- No transcript available for a video: skip it, note it, continue with remaining videos
- Fewer than 5 transcripts: ask for more video URLs before proceeding
- Channel handle not found: suggest checking the spelling or providing direct video URLs
- Very short videos (under 3 minutes): skip or note that not all 7 bricks may be present
- Transcript in non-English language: note the language, attempt analysis, flag lower confidence

## Self-check

1. At least 5 videos analyzed with full 7-brick breakdown
2. Comparison matrix includes all videos and all 7 bricks
3. Exactly 5 "copy" patterns identified with frequency counts
4. Exactly 2 "make original" bricks identified with variation counts
5. Each pattern includes a transcript excerpt from an actual video
6. Each pattern includes a specific application recommendation
7. Per-video breakdown table exists for every analyzed video
8. Reference table at the bottom with video IDs
9. Output saved to temp/ with date prefix
10. No Tier 1/2 banned words from humanizer rules

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. 7-brick framework, comparison matrix, 5 copy + 2 original recommendation structure. |
