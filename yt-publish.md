---
name: yt-publish
description: "Publish video to YouTube with optimized metadata. Generates title, description, tags from your script and uploads via YouTube MCP."
argument-hint: "[video folder name, e.g. v2-setup]"
user-invocable: true
---

# YouTube publish

Generate optimized metadata (title, description, tags) from your video's script and outline, then upload to YouTube via the YouTube MCP server.

## Input

- `$ARGUMENTS`: video folder name (e.g., v2-setup, v3-niche)
- Video folder path: `content/youtube/{folder_name}/`

Parse the video number from the folder name. For example, `v2-setup` means V2. `v5-ads` means V5.

## Step 1: Load context

Read these files from `content/youtube/{folder_name}/`:

1. `02_OUTLINE.md` (required, fail if missing). Extract: title options, section structure, hook angle.
2. `03_SCRIPT.md` (required, fail if missing). Extract: full script content, section headers, timestamps if present.
3. `01_RESEARCH.md` (optional). Extract: keyword angles, competitor titles, thumbnail text.

If either required file is missing, fail with: "Missing {filename} in content/youtube/{folder_name}/. Run the previous pipeline step first."

Also read `business.md` for brand voice and channel positioning.

## Step 2: Pick the best title

Review all title options from `02_OUTLINE.md`. Score each against these 5 criteria:

| Criteria | Weight | What to check |
|---|---|---|
| Curiosity gap | 25% | Does it create a question the viewer needs answered? |
| Specificity | 25% | Does it include a specific number, timeframe, or result? |
| Low competition overlap | 20% | Is it different from competitor titles in 01_RESEARCH.md? |
| Search intent match | 15% | Would someone searching for this topic click this? |
| Under 60 characters | 15% | YouTube truncates titles on mobile around 60 chars |

Pick the highest-scoring title. If two titles tie, pick the shorter one.

Print the scoring:

```
TITLE SELECTION
---------------
Option A: "{title}" - Score: {N}/100
  Curiosity: {score}  Specificity: {score}  Unique: {score}  Search: {score}  Length: {score}
Option B: "{title}" - Score: {N}/100
  ...

SELECTED: "{winning title}"
```

## Step 3: Generate description

Build the description in this exact structure:

```
{Hook line: 1 sentence that restates the title's promise with more detail}

{2-3 sentence summary of what the viewer will learn. Be specific about outcomes.}

TIMESTAMPS:
{Generate from script section headers. Format: MM:SS Section name}
00:00 {First section}
{MM:SS} {Second section}
{MM:SS} {Third section}
...

RESOURCES:
{Links mentioned in the script. If none, include channel link.}

#tags #go #here
```

Rules for description:
- Hook line must NOT repeat the title word-for-word. Rephrase the same idea with added specificity.
- Timestamps must match the actual script sections. Do not invent sections.
- Keep the description under 5,000 characters (YouTube limit).
- No banned words from humanizer rules in the description.

## Step 4: Generate tags

Generate 15-20 tags. Three categories:

**Primary tags (5-7):** Exact match for what the video is about.
Example: "claude code tutorial", "claude code setup", "ai coding assistant"

**Secondary tags (5-7):** Related topics viewers also search for.
Example: "one person business", "ai automation", "solo founder tools"

**Long-tail tags (3-5):** Specific phrases from the script or research.
Example: "how to set up claude code for business", "mcp server tutorial 2026"

Rules:
- No single-word tags (too broad, ignored by algorithm)
- No tags over 30 characters
- Check 01_RESEARCH.md for keyword ideas from competitor videos
- Include the main topic keyword in at least 3 tags

## Step 5: Upload via YouTube MCP

Use `mcp__youtube__youtube_upload_video` with these parameters:

```
title: {selected title}
description: {generated description}
tags: {comma-separated tag list}
categoryId: "28" (Science & Technology) or "22" (People & Blogs) depending on content
privacyStatus: "private" (always upload as private first, promote to public manually)
```

The video file path: look for the final rendered video in the folder. Check for these files in order:
1. `content/youtube/{folder_name}/output/final_broll.mp4`
2. `content/youtube/{folder_name}/output/final_motion.mp4`
3. `content/youtube/{folder_name}/output/cut_edit.mp4`

Use the first one that exists. If none exist, fail with: "No rendered video found in output/. Run /yt-cut-edit first."

## Step 6: Set thumbnail

Check for a thumbnail file:
1. `content/youtube/{folder_name}/thumbnail.png`
2. `content/youtube/{folder_name}/thumbnail.jpg`
3. `assets/thumbnails/{folder_name}.png`

If found, use `mcp__youtube__youtube_set_thumbnail` to upload it.

If not found, print: "No thumbnail file found. Add thumbnail.png to the video folder and run: /yt-publish {folder_name} --thumbnail-only"

## Step 7: Check optimal publish time

Use `mcp__youtube__youtube_analytics_daily` to pull the last 28 days of daily analytics.

Find the day of week and time window with the highest average views in the first 24 hours. Print a scheduling recommendation:

```
PUBLISH TIMING
--------------
Best day:  {day of week}
Best time: {time window, e.g. "2:00 PM to 4:00 PM PST"}
Based on:  last 28 days of daily performance data

The video is uploaded as PRIVATE. When ready to publish:
1. Go to YouTube Studio > Content > find this video
2. Click Edit > Visibility > Schedule
3. Set to: {recommended day}, {recommended time} PST
```

If analytics MCP fails, print: "Could not pull analytics. General recommendation: publish Tuesday-Thursday between 12:00 PM and 3:00 PM PST."

## Step 8: Summary

Print the full summary:

```
YT-PUBLISH COMPLETE
====================
Video:       V{N} - {topic}
Title:       {selected title}
Tags:        {N} tags
Thumbnail:   {set / not found}
Status:      PRIVATE (promote manually)
Video ID:    {id from upload response}

Description preview (first 3 lines):
  {first 3 lines of description}

Recommended publish:
  {day}, {time} PST

Next steps:
  1. Review title and description in YouTube Studio
  2. Add thumbnail if not set
  3. Schedule publish for {recommended time}
  4. Prepare community post for publish day
```

## Example metadata output

```
TITLE SELECTION
---------------
Option A: "I Built a $10K/Month Business With One AI Tool" - Score: 82/100
  Curiosity: 22  Specificity: 23  Unique: 18  Search: 12  Length: 7
Option B: "Claude Code Setup: Everything You Need in 30 Minutes" - Score: 78/100
  Curiosity: 15  Specificity: 20  Unique: 16  Search: 15  Length: 12
Option C: "The Only AI Tool Solo Founders Need (Full Setup)" - Score: 85/100
  Curiosity: 23  Specificity: 18  Unique: 20  Search: 14  Length: 10

SELECTED: "The Only AI Tool Solo Founders Need (Full Setup)"

Description:
  This is the complete setup guide for Claude Code, the AI coding assistant that
  replaced 6 tools in my business. You will see exactly how to install it,
  configure MCP servers, and build your first automation in under 30 minutes.

  TIMESTAMPS:
  00:00 Why one tool beats twenty
  02:15 Installing Claude Code
  05:30 Your first MCP server (Supabase)
  ...

Tags: claude code tutorial, claude code setup guide, ai tool for solo founders,
one person business ai, mcp server tutorial, claude code mcp setup, ...
```

## Error handling

- Required files missing (02_OUTLINE.md, 03_SCRIPT.md): fail with clear message naming the missing file
- No video file in output/: fail with message to run /yt-cut-edit first
- YouTube MCP not connected: save metadata to `temp/{YYYY-MM-DD}_{folder_name}_yt_metadata.md` instead, print "YouTube MCP not connected. Metadata saved. Copy-paste into YouTube Studio manually."
- Upload fails (API error): print the error, save metadata locally, suggest manual upload
- Analytics call fails: use default timing recommendation
- Title over 100 characters: truncate and warn

## Self-check

1. Title scored against all 5 criteria with visible scoring
2. Description has hook line, summary, timestamps, resources section
3. Timestamps match actual script sections (not invented)
4. 15-20 tags generated across 3 categories
5. Video uploaded as PRIVATE (never public by default)
6. Thumbnail set if file exists, clear message if not
7. Publish timing based on actual analytics data
8. Summary includes video ID and next steps
9. No Tier 1/2 banned words in title, description, or tags
10. Description under 5,000 characters

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. Title scoring, description generation with timestamps, tag generation, YouTube MCP upload, analytics-based timing. |
