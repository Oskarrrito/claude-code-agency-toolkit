---
name: yt-research
description: Runs viral research for a YouTube video. Scrapes YouTube and X for trending content, pulls transcripts, extracts angles, and presents packaging suggestions. Produces 01_RESEARCH.md with key takeaways at top and angle-first structure. Use when starting a new YouTube video or requesting video research.
argument-hint: "[video folder name, e.g. v2-setup]"
disable-model-invocation: true
context: fork
agent: general-purpose
---

# YouTube video research

**Audience:** me (the founder recording the video). This is a speaker-facing research brief.

Produce `01_RESEARCH.md` for a new YouTube video. The goal: find successful angles from viral content and show how to apply them to package my content.

## Core principle

I already have the content. The research finds how to PACKAGE it. Learn from successful patterns first. Layer differentiation on top. The output should let me review each angle's application and decide: use it or skip it.

## Input

- Folder name: $ARGUMENTS (e.g., v2-setup, v3-niche)
- Video folder path: `content/youtube/{folder_name}/`

Extract the video number (V2, V3, etc.) and topic from the folder name. For example, `v2-setup` -> V2, Setup. `v3-niche-accelerator` -> V3, Niche Accelerator.

**Rule:** The video number extracted from $ARGUMENTS is the single source of truth. The document title and all internal references must use this exact video number. Before saving, grep the output for any mismatched video ID and fix it.

Create the folder `content/youtube/{folder_name}/` if it doesn't already exist.

## Step 1: Load context

1. Read `business.md` for brand angle and narrative arc
2. Check `research/INDEX.md` for existing research on the topic
3. Read `.claude/rules/humanizer.md` for writing rules

## Step 2: Identify what I'm packaging

Before any scraping, write a brief summary of MY content for this video. What do I already have? What am I trying to present? This becomes the "What we're packaging" section.

## Step 3: YouTube scrape

Use Apify (`api-ninja/youtube-search-scraper`) to scrape YouTube. Run `fetch-actor-details` first to check the input schema.

Run 2-3 searches with different keyword combinations related to the topic:
- Search 1: primary topic keywords (e.g., "Claude Code setup")
- Search 2: broader angle keywords (e.g., "one person business AI agent")
- Search 3: contrarian/niche keywords (e.g., "Claude Code mistakes")

Settings: `uploadDate: "month"`, `videoDepthDetails: "basic"`, `sortBy: "relevance"`, `maxResults: 30` per search.

Save raw results to `temp/{YYYY-MM-DD}_{topic}_scrape.json`.

## Step 4: Score and rank videos

Score formula (internal, don't include in output):
Score = (views / subscribers) x (28 / days_since_publish)

Select top 8-12 videos that are most relevant to the topic. Prioritize:
- High overperformance (small channels with big reach)
- Direct topic overlap with the video
- Different angles/framings (variety, not duplicates)

## Step 5: X/Twitter scrape

Use Apify to scrape X/Twitter for relevant posts. Search for topic-related keywords. Look for:
- High bookmark counts (save-for-later = high intent)
- Setup guides, tutorials, hot takes
- Posts from notable figures in the space

Save raw results to `temp/{YYYY-MM-DD}_{topic}_x_scrape.json`.

Select top 7-10 posts with highest engagement and relevance.

## Step 6: Pull transcripts and extract angles

For the top 5-8 YouTube videos, pull transcripts using `youtube_get_transcript` (no quota cost).

For each video and X post, extract the ANGLE:
- What framing made this work? (not what topic, but what ANGLE on the topic)
- Pull the relevant transcript excerpt (hook, key analogy, turning point)
- Write a specific application: how do I apply this angle to package my V2/V3/etc. content?

An angle is not "they talked about skills." An angle is "they framed skills as the answer to a specific problem, using a tax analogy to make it click." The angle is the packaging, not the topic.

## Step 7: Thumbnail text extraction

While analyzing YouTube videos, collect thumbnail patterns. For each top-performing video, note:
- What text appears on the thumbnail (exact words, size, color)
- Layout (face position, terminal/screen placement, text placement)
- What pattern it follows (revenue number, before/after, competence gap, completeness)

In the output, include a `## Thumbnail angles` section with 3 concrete thumbnail text options for the video, each citing which source video's pattern it follows.

## Step 8: Synthesize key takeaways

Review all extracted angles. Write 6-8 key takeaways that tell me exactly how to package this video. Each takeaway should:
- Name the pattern
- Cite the proof (which video/post, what numbers)
- State the application in one sentence

These go at the TOP of the document. I read these first.

## Output format

Write to `{video_folder}/01_RESEARCH.md`:

```markdown
# V{N} {Topic}: Research

Scraped: {YYYY-MM-DD} | Sources: Apify YouTube scraper, Apify Tweet scraper, YouTube transcript API | Filter: last 30 days

---

## Key takeaways for V{N}

{6-8 numbered takeaways. Each one: pattern name + proof + application. This is the executive summary. I read this first and know exactly how to approach the video.}

---

## What we're packaging

{Brief summary of MY content for this video. What skills, what modules, what I'm showing. Bullet list. This is the content the angles will wrap around.}

---

## YouTube angles

{For each angle extracted from YouTube videos:}

### Angle N: "{short name}" -- {2-3 word description}

**Source:** {Creator}, "{Title}" | {views}, {overperformance}x | {video_id}

**Transcript:** "{Relevant excerpt from the video, the hook, key analogy, or turning point that makes the angle work.}"

**Application:**
{2-4 bullets showing exactly how I'd apply this angle to my video. Specific sections, specific lines, specific framing. I review this and decide.}

---

{Repeat for each angle. Aim for 6-10 YouTube angles.}

## X/Twitter angles

{For each angle extracted from X/Twitter:}

### X-Angle N: "{short name}" -- {2-3 word description}

**Source:** @{handle} | {likes} likes / {bookmarks} bookmarks

**Post:** "{Content excerpt}"

**Application:**
{2-4 bullets showing how I'd apply this to my video.}

---

{Repeat for each angle. Aim for 5-8 X angles.}

## Thumbnail angles

{3 thumbnail text/layout options. Each one cites which source video's pattern it follows.}

| Version | Text overlay | Layout | Source pattern |
|---------|-------------|--------|---------------|
| A | {exact text, size, color} | {face/terminal/split position} | {which video, views} |
| B | {exact text, size, color} | {face/terminal/split position} | {which video, views} |
| C | {exact text, size, color} | {face/terminal/split position} | {which video, views} |

---

## Adopted angles

{Empty table to fill after reviewing. This gets populated before outline stage.}

| # | Angle | Source | Applied to section | Notes |
|---|-------|--------|-------------------|-------|
| | | | | |

---

## Reference data

### YouTube ({N} videos scored)

| Rank | Creator | Title | Score | Views | Video ID |
|------|---------|-------|-------|-------|----------|
{Condensed reference table. All scored videos.}

### X/Twitter ({N} posts)

| # | Author | Topic | Likes | Bookmarks |
|---|--------|-------|-------|-----------|
{Condensed reference table. All scraped posts.}
```

## Content accuracy rules

Facts, statistics, and claims in the research must be accurate and sourced. Do not invent numbers or attribute quotes without a source. Product names, service names, and pricing referenced in the video must match `business.md`.

## Writing rules

1. **AI Humanizer:** All banned words, phrases, and patterns from `.claude/rules/humanizer.md` are strictly prohibited.
2. **Sentence length variation:** Never write 4+ consecutive sentences with the same word count (within +/-2 words).
3. **Dollar range formatting:** Use spelled-out style: "$3,000 to $10,000" (not "$3,000-$10,000").
4. **Anti-hype:** Persuade with specifics and data, not adjectives. "This does X" not "This is amazing."

## Self-check (run before saving)

1. Key takeaways section exists at the TOP of the document (before "What we're packaging")
2. Each YouTube angle has: Source (with video ID), Transcript excerpt, and Application bullets
3. Each X/Twitter angle has: Source (with handle + engagement), Post excerpt, and Application bullets
4. No scoring methodology section in the output (scoring is internal only)
5. No "why it works" or "the angle:" explanations. Just: source, transcript, application
6. Adopted angles table exists (empty, to fill later)
7. Reference data tables at the bottom (condensed)
8. Video number from folder name matches everywhere in the document
9. No Tier 1/2 banned words (delve, tapestry, landscape, leverage, robust, etc.)
10. No banned phrases ("worth noting", "in today's", "game-changer", etc.)
11. Minimum 6 YouTube angles with transcripts
12. Minimum 5 X/Twitter angles with post excerpts
13. Thumbnail angles section exists with 3 options, each citing a source pattern
14. Title format: `# V{N} {Topic}: Research` (not `# V{N} Research: {Topic}`)
15. File saved as `01_RESEARCH.md` (not `1_research.md`)
16. Product/service names in "What we're packaging" match business.md

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-26 | Pipeline alignment. File naming: `01_RESEARCH.md`. Folder path: `content/youtube/{folder_name}/`. Title format: `V{N} {Topic}: Research`. Argument-hint matches outline/script skills. Added: `humanizer.md` context load, content accuracy rules, writing rules, self-checks. |
| 2026-03-25 | Full rewrite. New angle-first structure: key takeaways at top, source -> transcript -> application format, no scoring methodology in output, no "why it works" explanations. Added: YouTube angles, X/Twitter angles, adopted angles table, condensed reference data. |
