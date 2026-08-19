---
name: yt-outline
description: Creates a video outline from research. Reads 01_RESEARCH.md, produces 02_OUTLINE.md with title options, thumbnail concepts, angles applied, clean script outline, visual inserts, CTA placements, cross-video references, recording guideline, and screen layout. Use when requesting a video outline, video concept, or video structure.
argument-hint: "[video folder name, e.g. v2-setup]"
disable-model-invocation: true
context: fork
agent: general-purpose
---

# YouTube video outline

**Audience:** me (the speaker). This is a speaker-facing doc. The script outline is clean narrative that I read to feel the flow. Screen recording directions go in the Recording guideline section at the bottom, not inside the script outline.

Produce `02_OUTLINE.md` for a YouTube video.

## Input

- Folder name: $ARGUMENTS (e.g., v2-setup, v3-niche)
- Video folder path: `content/youtube/{folder_name}/`

## Step 1: Load context

1. Find the video folder: `content/youtube/{folder_name}/`
2. Read `01_RESEARCH.md` from that folder (required, fail if missing)
3. Read `business.md` for accurate product/service names and structure
4. Read `.claude/rules/humanizer.md` for writing rules

## Step 2: Determine video identity

From the research file, extract:
- Video number (V1, V2, etc.)
- Topic area
- Target length
- Funnel position (which video comes before/after)
- Core principle (the one-sentence editorial thesis)

## Step 3: Build the outline

Write to `{video_folder}/02_OUTLINE.md` with the following structure. Every section is required.

### Output structure

```markdown
# V{N} {Topic}: Outline

## Founder's notes

> -
> -
> -

---

## Video info

- **Publish date:** {date}
- **Target length:** {N} minutes
- **Funnel role:** {Where this sits in the video sequence}
- **Core principle:** {One-sentence editorial thesis. What should the viewer think/feel?}

---

## Title options

| # | Title | Pattern |
|---|-------|---------|
| A | {title} | {what pattern it uses} |
| B | {title} | {what pattern it uses} |
| C | {title} | {what pattern it uses} |

**Recommended: {letter}.** {1-2 sentences why, referencing research data.}

---

## Thumbnail concepts

| Version | Visual | Text overlay | Test angle |
|---------|--------|-------------|------------|
| A ({name}) | {description} | {text on thumbnail} | {what angle, reference source from research} |
| B ({name}) | {description} | {text on thumbnail} | {what angle, reference source} |
| C ({name}) | {description} | {text on thumbnail} | {what angle, reference source} |

---

## Angles applied (from research)

| # | Angle | Source | Where it appears |
|---|-------|--------|-----------------|
| 1 | {angle name} | {source with numbers} | {which section of the video} |
| ... | ... | ... | ... |

Map every viable angle from the research to a specific section of the video. Drop angles that don't fit. Each angle should appear at most once.

---

## Script outline

{I read this top to bottom during recording. Everything I need is here: what to say, what to type, when to show screen. Two types of inline directions: `> blockquotes` for silent screen changes (3-8 words max), `- bullets` for narrated actions I do while talking. Talking head is the default (no markers needed).}

### 0:00-{time} -- Hook (talking head)

> Flash: {end result}

{Strongest claim. Specific, not hype.}

{Call out viewer's current state.}

{Data point from research.}

{Lower the barrier.}

---

### {time}-{time} -- {Section name}

{What I say to camera. Written as readable prose I can speak naturally.}

> Short screen direction (3-8 words)

{More narration...}

- Action I do while talking (opening files, scrolling output, clicking)

{Narration continues...}

{For skill demo sections, include the full Q&A:}

**I type:** "{command}"

**Claude asks:** "{question}"

**I type:** "{exact answer I type}"

{1-2 sentences of narration explaining why this matters.}

**Claude outputs:** "{key output line}"

{After the Q&A flow ends and I navigate through the output, use bullets as pacing breaks:}

- Scroll through the business results

{Narration about what the output shows...}

- Scroll to competitor pricing

{Narration about pricing data...}

---

### {time}-{time} -- {Next section}

{Continue the narrative...}

---

### {time}-{time} -- Close + CTA (talking head)

> TALKING HEAD

{Recap the system in one paragraph.}

{Email opt-in CTA: "Want the full roadmap? Link in the description. Drop your email." + next video teaser. Never reference downloading a specific starter kit or a specific product URL.}

---

## Visual inserts (Remotion)

| ID | Timestamp | Description |
|----|-----------|-------------|
| V{N}-VIS-01 | {time} | {Animated graphic description} |
| ... | ... | ... |

{List all Remotion motion graphics needed. Architecture diagrams, org charts, pipeline flows, comparison graphics, recap animations.}

---

## CTA placements

| Timestamp | Type | Script |
|-----------|------|--------|
| {mid-roll time} | Mid-roll | {Email opt-in CTA: "Want the full roadmap? Link in the description."} |
| {end time} | End | {Email opt-in + next video teaser} |
| Pinned comment | Comment | {Short hook + email opt-in} |
| Description | Link | "Full Series" section linking related videos |

---

## Cross-video references

| Timestamp | Reference | Links to |
|-----------|-----------|----------|
| {time} | {natural reference to another video} | V{N} |
| ... | ... | ... |

{Each video should reference at least the previous and next video naturally within the script.}

---

## Recording guideline

Screen recording and capture directions for each section. Used during production, not part of the script I read.

### {Section name} ({time range})
- {What to show on screen}
- {What to click, type, demonstrate}
- {Camera vs screen share notes}

{Repeat for each section}

---

## Recording zones

Every video follows a 3-zone structure. Two camera switches total. Simple to shoot, simple to edit.

| Zone | Type | Content | Camera |
|------|------|---------|--------|
| A | Talking head | Hook, problem statement, theory, stories, frameworks | Face to camera. No screen share. |
| B | Screen share OR demonstration | Show your service in action, client results, tool walkthrough, before/after | Screen recording or slides. Voice-over only. |
| C | Talking head | CTA, recap, summary, next video teaser | Face to camera. No screen share. |

Ask: "What will you show on screen during Zone B? (e.g., client dashboard, before/after results, live demo of your service)"

Rules:
- **No micro-switching.** Within a zone, never flip between talking head and screen share. Zone B is pure screen share from start to finish. Zone A and C are pure talking head.
- **Zone transitions use blockquotes.** At the start of Zone B: `> SCREEN SHARE: {what's visible}`. At Zone C: `> TALKING HEAD`. Zone A adds `(talking head)` to the header.
- **All demos in one block.** Every demo goes into Zone B, back to back. No returning to talking head between demos.
- **Mid-roll CTA stays in Zone B.** Deliver the mid-roll CTA as voice-over while the screen shows something relevant (dashboard, demo results so far). No camera switch. Place it between two natural demo breaks.
- **Close CTA goes in Zone C.** The final CTA, recap, and next video teaser happen in Zone C (talking head).

In the outline, add a `## Recording zones` section that maps each script outline section to its zone (A, B, or C). This tells the production team exactly when to switch cameras.
```

## Rules

### Script outline rules

The script outline is a teleprompter draft. I read it top to bottom during recording. Everything I need (what to say, what to type, when to show screen) is inline. The script skill expands this into the full spoken script.

1. **Readable flow.** Every line is either something I say to camera, a blockquote (silent screen change, 3-8 words), or a bullet (narrated action). Nothing else.
2. **Blockquotes = silent screen changes.** I don't read this aloud. The screen just changes. Use `> ` format. Keep to 3-8 words. Good: `> Antigravity: empty workspace`, `> Split: setup guide + Antigravity`, `> Flash: niche report scored`. Bad: `> Antigravity terminal with empty workspace, Claude Code panel closed, file explorer showing no files`. Cut directions entirely after `**I type:**` or `**Claude outputs:**` lines (I see the result on screen).
3. **Bullets = narrated actions.** I read this while doing the action on screen. Use for physical actions AND output navigation. Good: `- Open the skills folder`, `- Scroll through the business results`, `- Scroll to competitor pricing`. Bullets are required pacing breaks. Demo sections must NOT flow as continuous text.
4. **No structured markers.** Never use `[SCREEN:]`, `[Screen:]`, `[REMOTION:]`, `[Type:]`, `[Click:]`, `[Show:]`, `[Open:]`, `[Press:]`, `[Switch to:]` prefix syntax.
5. **No VIS markers in script outline.** Never put VIS IDs in the script outline. No `[V3-VIS-04: diagram]`. VIS IDs belong only in the Visual inserts table and Recording guideline.
6. **Talking head is the default.** No marker needed for talking head sections. If there's no bracket or bullet, I'm talking to camera.
7. **Result flash first.** Every demo section opens with `> Flash: {output}`. The hook does the same. Viewer sees the payoff before the process.
8. **Script the full Q&A for demos.** Every demo includes the exact inputs I type. Use `**I type:**` for commands, `**Claude asks:**` for questions, `**Claude outputs:**` for results. Between each pair, 1-2 sentences of narration.
9. **Output navigation bullets.** After the Q&A flow ends in a demo section and I walk through the output, add `- Scroll to {section}` bullets between each output block. Each scroll is a pacing break where I read while the screen moves.
10. **No angle annotations.** Don't put "(Angle #3)" or similar. Angles are already mapped in the Angles applied table.

### CTA rules

1. **Email opt-in.** CTA is always "want the full roadmap? drop your email, link in the description." Direct viewers to leave their email to receive the roadmap. Never a product URL or download link.
2. **Next video teaser.** End CTA always previews the next video in the series.
3. **Pinned comment.** Short, curiosity-driven, email opt-in focused.

### Content accuracy rules

Apply the content accuracy rules from yt-research.md. Read that file for the authoritative reference.

### Title rules

- Titles should include the tool or theme name when it makes sense (e.g., "Claude Code")
- Include a revenue number ($30K/Month) when applicable
- ~60 characters
- Must create curiosity gap or disbelief
- Avoid clickbait the video can't deliver on

### Formatting

- Dollar ranges include dollar sign on both numbers: $500-$2,000, not $500-2,000
- AI Humanizer rules applied (no banned words: landscape, delve, comprehensive, showcase, navigate, etc.)
- Sentence case headings (capitalize first word and proper nouns only)

## Post-save validation

After saving `02_OUTLINE.md`, read the file back and verify:

1. **All required sections present:** Video info, Title options, Thumbnail concepts, Angles applied, Script outline, Visual inserts, CTA placements, Cross-video references, Recording guideline, Recording zones, Founder's notes.
2. **No old-format markers in script outline.** Search for `[SCREEN`, `[Screen:`, `[Type:`, `[Click:`, `[Show:`, `[REMOTION`, `[Open:`, `[Press:`, `[Switch to:`. None should appear. Also search for `[ ` (bracket with space) to catch old-format bracket directions. All scene directions must use `> ` blockquote format.
3. **No VIS IDs in script outline.** Search for `VIS-` in the Script outline section. None should appear. VIS IDs belong only in Visual inserts table and Recording guideline.
4. **No angle annotations in script outline.** Search for `(Angle #`, `(#`. None should appear in the Script outline section.
5. **Hook has result flash.** First section opens with `> Flash: ...`.
6. **Demo sections have output navigation bullets.** Every demo section that shows skill output must have `- Scroll to {section}` or `- Open {file}` bullets as pacing breaks. Demo sections must NOT flow as continuous text after the Q&A.
7. **CTA is email opt-in.** Search for "download", "starter kit", "subscribe" as primary CTA. CTA should be "drop your email" / "link in the description" for roadmap. Never a product URL or download link.
8. **Product/service names match business.md.** Cross-reference any names mentioned with `business.md`.
9. **Banned word scan.** Search for: landscape, delve, comprehensive, showcase, navigate, leverage, robust, tapestry, multifaceted, pivotal. Replace with plain alternatives.
10. **Script outline flows as readable narrative.** Each section is prose I can read to feel the video's arc.

If any check fails, fix and re-validate.

## Quality checklist

- [ ] Every angle from research is either mapped to a section or explicitly dropped
- [ ] Title options reference specific research data (view counts, patterns)
- [ ] Thumbnail concepts reference specific source patterns from research
- [ ] Script outline reads as a continuous narrative (no production markers)
- [ ] Hook opens with `> Flash: {result}` blockquote
- [ ] Scene directions use `> blockquotes` (3-8 words) for silent changes, `- bullets` for narrated actions
- [ ] No `[SCREEN:]`, `[Screen:]`, `[Type:]`, `[Click:]`, `[Show:]`, `[REMOTION:]` markers in script outline
- [ ] No `[ bracket ]` scene directions (all must use `> blockquote` format)
- [ ] No VIS IDs in script outline (only in Visual inserts table and Recording guideline)
- [ ] Demo sections have output navigation bullets (not continuous text)
- [ ] Demo sections use `**I type:**` / `**Claude asks:**` / `**Claude outputs:**` with narration between pairs
- [ ] Recording guideline has screen directions for every section with demos
- [ ] CTA is email opt-in ("drop your email, link in the description")
- [ ] Cross-video references feel natural within the narrative
- [ ] Visual inserts cover all key concepts that need animated graphics
- [ ] Total estimated length matches target
- [ ] AI Humanizer rules applied throughout

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
