---
name: thumbnail-gen
description: "Generate thumbnail concepts with visual direction. 5 concepts using 6-checkpoint click psychology. Use before filming or after scripting."
argument-hint: "[video folder name or topic]"
user-invocable: true
---

# Thumbnail generator: 5 concepts with click psychology

Generate 5 thumbnail concepts with full visual direction for a YouTube video. Each concept uses trigger elements from click psychology research and pairs with a specific title option.

## Input

- `$ARGUMENTS`: a video folder name (e.g., `v2-setup`) or a topic string + target avatar description
- If a folder name is provided, read `content/youtube/{folder}/02_OUTLINE.md` for context
- If no argument provided, ask: "Give me a video folder name or a topic + who the video is for."

## Step 1: Extract the video's core promise

**If folder provided:**
Read `content/youtube/{folder}/02_OUTLINE.md`. Pull:
- Working title(s)
- Target viewer (avatar)
- Main promise or transformation
- Key proof points or numbers

**If topic + avatar provided:**
Use the raw input directly. Ask for missing info if neither title nor avatar is clear.

Build a one-line summary:

```
VIDEO CORE
-----------
Topic: {topic}
Avatar: {who this is for}
Promise: {what result the viewer gets}
Proof: {specific numbers, timeframes, or credentials that back the promise}
```

## Step 2: Define the desire loop

Every clicked thumbnail triggers a desire loop: the viewer feels a pain, sees a desired outcome, and spots a hint that this video holds the answer.

```
DESIRE LOOP
------------
Pain: {what the viewer is stuck on or afraid of}
Desire: {what outcome they want}
Solution hint: {what the thumbnail implies without giving it away}
```

Keep each line under 15 words. The solution hint should create curiosity, not explain the method.

## Step 3: Generate 5 thumbnail concepts

For each concept, define all 6 fields below. Every concept must be distinct from the others in composition type and primary trigger.

### The 7 trigger categories

Pick 3 triggers per concept from this list:

| # | Trigger | What it does |
|---|---------|-------------|
| 1 | Color contrast | Bright vs dark, complementary colors, or neon on black. Pops in a feed of muted thumbnails. |
| 2 | Large face with emotion | Human faces grab attention. Exaggerated expressions (shock, excitement, frustration) outperform neutral. |
| 3 | Unusual graphic element | Something that looks wrong or out of place. Broken objects, impossible scales, visual metaphors. |
| 4 | Large numbers | Dollar amounts, percentages, day counts. Numbers with commas or dollar signs read faster than words. |
| 5 | Red circles/arrows | Direct the eye to a specific spot. Works when pointing at something surprising or hidden. |
| 6 | Aesthetic imagery | Clean, minimal, high-production look. Works for premium positioning. |
| 7 | Design collage | Multiple images arranged in a grid or overlap. Shows "a lot" at a glance. Good for comparison or collection videos. |

### Concept template

For each of the 5 concepts, output:

```
CONCEPT {N}: {short concept name}
----------------------------------
Composition: {symmetrical | asymmetrical | A/B split}
  - Symmetrical: subject centered, text above or below
  - Asymmetrical: subject on one side, text/graphic on the other (rule of thirds)
  - A/B split: left vs right comparison (before/after, old way vs new way)

Trigger 1: {category name} - {how it applies}
Trigger 2: {category name} - {how it applies}
Trigger 3: {category name} - {how it applies}

Text overlay: "{exact text on thumbnail}"
  - Max 5 words. Complements the title, does NOT repeat it.
  - Must be readable at 120x90px (mobile browse size). No thin fonts.

Face/emotion: {expression direction}
  - e.g., "wide eyes + open mouth (genuine shock)" or "slight smirk, raised eyebrow (confidence)"
  - If no face needed, write "No face - graphic only" and explain why

Title pairing: "{which title option from the outline works best with this thumbnail}"
  - Explain in one sentence why this title + thumbnail combo works

Visual notes: {1-3 sentences on colors, layout, props, or background}
```

## Step 4: Platform specs

Output these specs once (they apply to all 5 concepts):

```
PLATFORM SPECS
---------------
Resolution: 1280x720 minimum (recommended 1920x1080)
Aspect ratio: 16:9
File format: JPG or PNG, under 2MB
Safe zone: keep all text and key elements within center 80% (edges get cropped on some devices)
Mobile test: all text must be legible at 120x90px thumbnail size
Font minimum: 48px at 1920x1080 for primary text, 36px for secondary
Contrast rule: text must have a drop shadow, stroke, or background box if placed over a photo
```

## Step 5: Output Canva/Figma brief

For each concept, output a brief that a designer (or Canva/Figma user) can follow:

```
DESIGN BRIEF: Concept {N}
---------------------------
Canvas: 1920x1080
Background: {solid color hex / gradient / photo description}
Layer 1 (back): {background element}
Layer 2 (mid): {subject photo or graphic, position, size}
Layer 3 (front): {text overlay, font suggestion, color, position}
Layer 4 (accent): {arrows, circles, badges, or other trigger elements}

Font suggestions:
  - Primary text: {font name}, {weight}, {size}px, {color hex}
  - Secondary text (if any): {font name}, {weight}, {size}px, {color hex}

Color palette:
  - Background: {hex}
  - Primary text: {hex}
  - Accent: {hex}
  - Optional secondary: {hex}
```

## Step 6: AI image generation (optional)

If the Recraft API key is available in the environment (`RECRAFT_API_KEY`), generate draft thumbnail images:

```bash
# Check for API key
echo $RECRAFT_API_KEY
```

If available, generate a draft image for each of the top 3 concepts using the Recraft API. The prompt should describe the thumbnail composition, not the video content. Save outputs to `content/youtube/{folder}/thumbnails/` or `temp/` if no folder.

If the API key is not available, skip this step and note:
```
RECRAFT: No API key found. Skipping image generation.
Manual creation: use the design briefs above in Canva or Figma.
```

## Step 7: Save and summarize

Save the full output to:
- `content/youtube/{folder}/05_THUMBNAILS.md` if a folder was provided
- `temp/{YYYY-MM-DD}_{topic}_thumbnails.md` if working from a topic

Print a summary:

```
THUMBNAIL CONCEPTS COMPLETE
-----------------------------
Concepts generated: 5
Top recommendation: Concept {N} - {reason in one sentence}
Strongest hook combo: "{title}" + Concept {N} thumbnail
Next step: create in Canva/Figma using the design briefs, or run with Recraft if API key is set up
```

## Example output (abbreviated, showing 2 of 5 concepts)

Topic: "How I Got 4 Clients in 7 Days Without Cold Calling"

```
VIDEO CORE
-----------
Topic: Getting clients fast without cold outreach
Avatar: Solo service provider, 0-3 clients, hates cold DMs and feels salesy
Promise: 4 paying clients in one week using warm methods
Proof: 4 clients, 7 days, zero cold calls

DESIRE LOOP
------------
Pain: sending hundreds of cold messages that get ignored
Desire: clients coming to them (or at least warm conversations that convert)
Solution hint: there's a method that skips the cold outreach grind entirely

CONCEPT 1: The Scoreboard
----------------------------------
Composition: asymmetrical
  - Face on left third, scoreboard graphic on right two-thirds

Trigger 1: Large numbers - "$" amounts or "4 clients" displayed large on a scoreboard graphic
Trigger 2: Large face with emotion - genuine surprised expression, like "I can't believe that worked"
Trigger 3: Unusual graphic element - a literal sports scoreboard showing "Day 1: 0 to Day 7: 4"

Text overlay: "no cold DMs"
  - 3 words, complements the title which already says "4 clients in 7 days"

Face/emotion: eyebrows raised, slight open mouth, head tilted. Genuine surprise, not fake shock.

Title pairing: "How I Got 4 Clients in 7 Days Without Cold Calling"
  - Title has the numbers and timeframe. Thumbnail adds the visual proof (scoreboard)
    and the emotional "I didn't expect this to work" reaction.

Visual notes: Dark background (#1a1a2e). Scoreboard is a bright green on dark panel,
  styled like a basketball score display. Face lit from the screen side with a green tint
  to suggest the scoreboard is the light source. Text "no cold DMs" in white with red
  strikethrough over "cold DMs" for emphasis.

CONCEPT 3: The Inbox (A/B split)
----------------------------------
Triggers: Color contrast (gray vs green), Large numbers ("4" huge on right), Design collage (stacked notification bubbles)
Text overlay: "0 sent"
Face/emotion: No face - graphic only. The split comparison IS the hook.
Title pairing: "How I Got 4 Clients in 7 Days Without Cold Calling"
Visual notes: Hard vertical split. Left: gray, empty chat. Right: bright #f0fdf4, four notification
  bubbles with green dots. "4" overlaps the split line in bold yellow (#FBBF24).

(Concepts 2, 4, 5 follow the same template structure)

DESIGN BRIEF: Concept 1
---------------------------
Canvas: 1920x1080 | Background: #1a1a2e
Layer 1: radial gradient #1a1a2e to #0d0d1a | Layer 2: face, left third (x:100, y:140, h:800)
Layer 3: "no cold DMs" top-right (x:1050, y:180) | Layer 4: scoreboard right two-thirds
Fonts: Montserrat 800/72px #FFF + JetBrains Mono 700/96px #22C55E
Colors: bg #1a1a2e, text #FFF, accent #22C55E, strikethrough #EF4444

THUMBNAIL CONCEPTS COMPLETE
-----------------------------
Concepts generated: 5
Top recommendation: Concept 1 - scoreboard visual is specific and unusual, face adds human connection
Strongest hook combo: "How I Got 4 Clients in 7 Days Without Cold Calling" + Concept 1 thumbnail
Next step: create in Canva/Figma using the design briefs, or run with Recraft if API key is set up
```

## Error handling

- Outline file not found: ask user for the video topic and avatar directly, proceed without file
- No title options in outline: generate 3 title suggestions first, then pair thumbnails to them
- Recraft API call fails: log the error, skip image generation, continue with design briefs
- Folder name doesn't match any directory: check for close matches (e.g., `v2` vs `v2-setup`), suggest correction
- Topic too vague (e.g., "AI"): ask for the specific angle, avatar, and promise before generating concepts

## Self-check

1. Exactly 5 concepts generated, each with all 6 fields filled
2. Each concept uses a different composition type or primary trigger (no two concepts identical)
3. All text overlays are 5 words or fewer
4. Every concept includes a title pairing with a one-sentence explanation
5. Platform specs section is present
6. Design brief exists for each concept with layer-by-layer instructions
7. Desire loop is defined before concepts (pain + desire + solution hint)
8. Example output section is present with at least 2 concepts shown
9. Output saved to the correct location (folder or temp/)
10. No Tier 1/2 banned words from humanizer rules

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. 5-concept framework with 7 trigger categories, desire loop, design briefs, optional Recraft generation. |
