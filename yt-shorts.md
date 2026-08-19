---
name: yt-shorts
description: "Repurpose long-form video into 3-5 YouTube Shorts. Finds hookable moments and generates cut instructions."
argument-hint: "[video folder name, e.g. v2-setup]"
user-invocable: true
---

# YouTube Shorts repurposer: long-form to vertical

Scan a long-form video script for moments that work as standalone YouTube Shorts. Rank by hook strength, rewrite for vertical-first delivery, and output ffmpeg commands to extract and crop each clip.

## Input

- `$ARGUMENTS`: a video folder name (e.g., `v2-setup`)
- The folder must contain `content/youtube/{folder}/03_SCRIPT.md`
- If no argument provided, ask: "Give me a video folder name (e.g., v2-setup)."

## Step 1: Read the script

Read `content/youtube/{folder}/03_SCRIPT.md`.

If the file doesn't exist, check for `content/youtube/{folder}/02_OUTLINE.md` as a fallback and warn:
```
WARNING: No 03_SCRIPT.md found. Using 02_OUTLINE.md instead.
Shorts will be approximate. Rerun after the script is written for accurate timestamps.
```

Parse the script and build a section map:

```
SCRIPT MAP
-----------
Total sections: {N}
Estimated duration: {N} minutes (based on ~150 words/minute speaking rate)
Sections:
  1. {section name} (~{word count} words, ~{duration} seconds)
  2. {section name} (~{word count} words, ~{duration} seconds)
  ...
```

## Step 2: Scan for short-form candidates

Go through every section and paragraph of the script. Flag any passage that meets ALL 4 criteria:

| Criterion | What to check |
|---|---------|
| Standalone | Makes sense without prior context. A new viewer should understand it cold. |
| Hook in 2 seconds | The first sentence grabs attention (question, bold claim, surprising fact, or direct challenge). |
| Complete point | Delivers one full idea with a payoff in under 60 seconds (~150 words). |
| Shock Score | Contains something surprising, counterintuitive, or emotionally charged. Rate 1-5. |

For each candidate, extract:

```
CANDIDATE {N}
--------------
Section: {which script section this comes from}
Start phrase: "{first 10 words of the passage}"
End phrase: "{last 10 words of the passage}"
Word count: {N} (~{N} seconds at 150 wpm)
Standalone: {yes/no + brief reason}
Hook strength: {1-5, where 5 = impossible to scroll past}
Shock Score: {1-5, where 5 = "wait, what?"}
Combined score: {hook + shock, max 10}
```

Identify at least 7-10 raw candidates. More is fine. Fewer means the script may not have enough standalone moments (note this).

## Step 3: Rank and select top 5

Sort candidates by Combined Score (descending). Break ties by preferring:
1. Shorter clips (under 45 seconds beat 45-60 seconds)
2. Clips from different sections (variety across the video)
3. Clips with a natural punchline ending (not trailing off)

Print the ranked list:

```
RANKED CANDIDATES
------------------
Rank | Section | Hook strength | Shock Score | Combined | Duration | First words
1    | {sect}  | {N}/5         | {N}/5       | {N}/10   | ~{N}s    | "{first 8 words}..."
2    | ...
3    | ...
4    | ...
5    | ...
---  | --- below the cut ---
6    | ...
7    | ...
```

Select the top 3-5 for full processing. Default to 5 if all candidates score 6/10 or higher. Drop to 3 if only 3 candidates score above 5/10.

## Step 4: Generate short-form scripts

For each selected candidate, rewrite the opening for vertical-first delivery. Shorts viewers decide in under 2 seconds. The first sentence must hit immediately.

### Rewrite rules

1. **Kill the preamble.** Remove any setup, context, or "so basically..." lead-ins. Start with the hook.
2. **Front-load the tension.** The first sentence should be the most surprising or provocative part.
3. **Punchline structure.** Build to a single clear payoff at the end. Cut anything after the payoff.
4. **Trim to 55 seconds max.** At 150 wpm, that's ~137 words. If the original is longer, cut the middle, not the hook or punchline.
5. **Add a retention loop.** If the short ends with a resolved point, add a one-line open loop: a question or teaser that makes the viewer check the channel.

For each short, output:

```
SHORT {N}: {working title}
===========================
Source section: {section name from script}
Estimated timestamp: {MM:SS - MM:SS in the long-form video}
Duration: ~{N} seconds ({N} words)

--- SCRIPT (vertical-first rewrite) ---

{Full rewritten script text.}

--- FIRST FRAME TEXT ---

"{Caption/hook text overlaid on the first frame, max 8 words}"

--- END ---
```

### First frame text rules

- Max 8 words
- Must create a reason to keep watching (question, bold claim, or "wait for it" implication)
- Should NOT summarize the short. It should tease.
- All caps is fine for 3 words or fewer. Mixed case for 4-8 words.

## Step 5: Generate ffmpeg extraction commands

For each short, generate the ffmpeg command to extract and crop from the long-form video.

Check for the long-form video file in this order (use the first one found):
```
content/youtube/{folder}/output/final_broll.mp4
content/youtube/{folder}/output/final_motion.mp4
content/youtube/{folder}/output/cut_edit.mp4
```

If none exist, use a placeholder path and note it.

### Crop options (provide both, let user pick)

| Option | When to use | Filter |
|---|---|---|
| A: Center crop | Subject centered | `crop=ih*9/16:ih:(iw-ih*9/16)/2:0,scale=1080:1920` |
| B: Blurred pad | Subject off-center | `[0:v]scale=1080:-2[s];[0:v]scale=1080:1920,boxblur=20:20[bg];[bg][s]overlay=0:(1920-oh)/2[out]` |

Base command structure:
```bash
ffmpeg -i "{input}" -ss {start} -to {end} -vf "{filter}" -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k "{output}"
```
For Option B, use `-filter_complex` instead of `-vf` and add `-map "[out]" -map 0:a`.

Output path: `content/youtube/{folder}/output/short_{N}_{slug}.mp4` (slug = lowercase title with hyphens).

## Step 6: Output the runnable bash script

Combine all ffmpeg commands into a single `extract_shorts.sh`:

```bash
#!/bin/bash
# Shorts extraction for: {folder} | Generated: {YYYY-MM-DD}
set -e
INPUT="{input_path}"
OUTDIR="content/youtube/{folder}/output"
mkdir -p "$OUTDIR"
echo "Short 1: {title}..." && ffmpeg -i "$INPUT" -ss {start} -to {end} -vf "{filter}" -c:v libx264 -preset medium -crf 18 -c:a aac -b:a 192k "$OUTDIR/short_1_{slug}.mp4"
# ... repeat for each short
echo "Done. {N} shorts extracted."
```

Save to `content/youtube/{folder}/output/extract_shorts.sh`.

## Step 7: Save and summarize

Save full analysis to `content/youtube/{folder}/04_SHORTFORM_SCRIPTS.md` (if exists, use `_v2.md`).

Print summary with: video name, candidates scanned, shorts generated, total duration, one-line per short (title + duration + hook), path to bash script, and next steps (review scripts, run extraction, add captions via `/yt-cut-edit`, upload via `/yt-publish`).

## Example output (abbreviated, showing 2 of 5 shorts)

Folder: `v2-setup`

```
SCRIPT MAP: 8 sections, ~14 min total
RANKED: 1. "aha" moment (10/10, ~52s) | 2. Common mistakes (8/10, ~48s) | 3. Why Claude Code (7/10, ~55s) | 4. MCP connection (7/10, ~44s) | 5. Hook (7/10, ~38s)

SHORT 1: The moment it clicked
===========================
Source section: The "aha" moment
Estimated timestamp: 10:20 - 11:12
Duration: ~52 seconds (130 words)

--- SCRIPT (vertical-first rewrite) ---

I asked Claude Code to build my entire landing page. Copy, design, deployment. Everything.

It took eleven minutes.

Not eleven hours. Not a weekend. Eleven minutes. I sat there and watched it write the headline, pick the color palette, set up the hosting, and push it live.

The page wasn't perfect. I changed the headline later and swapped one image. But the structure, the layout, the copy flow? Better than what I paid a freelancer two thousand dollars for last year.

That's the moment I stopped thinking of Claude Code as a coding tool. It's not. It's a business operator that happens to write code.

If you're still building things by hand, you're not saving money. You're burning time.

--- FIRST FRAME TEXT ---

"11 minutes. Entire landing page."

--- END ---

SHORT 2: The biggest beginner mistake
===========================
Source section: Common mistakes
Estimated timestamp: 12:15 - 13:03
Duration: ~48 seconds (120 words)

--- SCRIPT (vertical-first rewrite) ---

The number one reason beginners quit Claude Code in the first week? They ask it to do too much at once.

They type a paragraph-long prompt, hit enter, and get garbage back. Then they say the tool doesn't work.

Here's the fix. One instruction at a time. "Create a landing page with a headline and a signup form." Done. "Add a Stripe checkout button." Done. "Connect it to my Supabase database." Done.

Three prompts. Three minutes. Small prompts, big results.

--- FIRST FRAME TEXT ---

"Why beginners quit (and the fix)"

--- END ---

(Shorts 3-5 follow the same format)

SHORTS EXTRACTION COMPLETE: 5 shorts, ~237s total
Bash script: content/youtube/v2-setup/output/extract_shorts.sh
```

## Error handling

- Script file not found: check for outline as fallback, warn about reduced accuracy
- Script too short (under 5 minutes / 750 words): note that fewer than 3 shorts may be possible, proceed with what exists
- No candidates score above 5/10: output all candidates with scores and let the user pick manually
- Video file not found for ffmpeg: use placeholder path, print a note to update the path before running
- Existing `04_SHORTFORM_SCRIPTS.md`: save as `_v2.md` (or `_v3.md`, etc.) instead of overwriting

## Self-check

1. Script map includes all sections with word counts and estimated durations
2. At least 7 raw candidates identified (or noted if the script doesn't support that many)
3. Ranked list sorted by Combined Score with ties broken correctly
4. 3-5 shorts selected based on score thresholds
5. Each short has a full vertical-first rewrite (not just a copy of the original)
6. Each short is under 60 seconds / 150 words
7. First frame text is 8 words or fewer for every short
8. ffmpeg commands include both center-crop and padded options
9. Runnable bash script saved to output folder
10. Full analysis saved to `04_SHORTFORM_SCRIPTS.md` (or versioned variant)
11. No Tier 1/2 banned words from humanizer rules

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. 4-criteria candidate scan, 10-point scoring, vertical-first rewrites, dual ffmpeg crop options, bash extraction script. |
