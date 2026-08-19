---
name: yt-script
description: Writes the full longform script, evaluates it against 29 viral criteria, improves failures, and syncs the outline. Self-contained pipeline (write -> eval -> improve -> final output). Use when requesting a video script, YouTube script, longform script, or script evaluation.
argument-hint: "[video folder name, e.g. v2-setup]"
disable-model-invocation: true
context: fork
agent: general-purpose
---

# Longform script

One skill, full pipeline: write the longform script from the outline, evaluate against 29 viral criteria, fix failures, sync the outline, output final passing script.

**Audience:** me (the speaker). This is the speaking script I read and deliver on camera.

## Input

- Folder name: $ARGUMENTS (e.g., v2-setup, v3-niche)
- Video folder path: `content/youtube/{folder_name}/`

---

## Step 1: Load context

1. Find the video folder: `content/youtube/{folder_name}/`
2. Read `02_OUTLINE.md` (required, fail if missing)
3. Read `01_RESEARCH.md` for facts, numbers, and research angles
4. Read `business.md` for accurate product/service names and structure
5. Read `.claude/rules/humanizer.md` for writing rules

---

## Step 2: Write the script

Convert the outline into a full spoken script. Every word I will say goes in this file. Scene directions go inline.

Work section by section through the outline's script outline. For each section:

1. Take the outline's summary and expand it into full spoken dialogue
2. Carry over scene directions from the outline (blockquotes and bullets)
3. Expand any demo Q&A with narration between each pair
4. Hit the word count target (see Length targets below)
5. Include all facts/numbers from research

### Output format

```markdown
# V{N} {Topic}: Script

---

## {start time}-{end time} -- {Section name} (talking head)

{Spoken dialogue. What I say to camera.}

---

## {start time}-{end time} -- {Section name}

> SCREEN SHARE: {what's visible}

{Narration...}

> {Short screen direction, 3-8 words}

{More narration...}

- Action I do while talking

{Narration continues...}

**I type:** "command"

**Claude asks:** "question"

**I type:** "answer"

**Claude outputs:** "response"

{Narration explaining why this matters...}

---

## {start time}-{end time} -- Close + CTA (talking head)

> TALKING HEAD

{Recap, CTA, next video teaser...}

---
```

### Recording zones

Every script follows a 3-zone structure. Two camera switches total.

| Zone | Type | Content |
|------|------|---------|
| A | Talking head | Hook, problem statement, system intro |
| B | Screen share | ALL demos, back to back |
| C | Talking head | CTA, recap, next video teaser |

Mark zone transitions:
- Zone A: add `(talking head)` to section header. No screen directions.
- Zone B: first section starts with `> SCREEN SHARE: {what's visible}`. No zone label on headers.
- Zone C: add `(talking head)` to section header. Starts with `> TALKING HEAD`.

**No micro-switching.** Within a zone, never flip between talking head and screen share. Zone B is pure screen share from start to finish. Zone A and C are pure talking head. All demos go in Zone B. Mid-roll CTA stays in Zone B as voice-over (screen shows dashboard or demo results). Close CTA goes in Zone C.

### Scene direction format

Two types of inline directions within screen share zones.

**Blockquotes `>` = silent screen change.** I don't read this aloud. The screen just changes. Keep them SHORT (3-8 words). I glance at the direction and know where to be.

```markdown
> Split: setup guide + Antigravity

> Browser: antigravity.google

> Flash: completed business.md, 12 sections

> .env.example open

> Terminal: 4 MCP connections confirmed

> Claude Code: clean slate
```

**Good directions** tell me where to be in 3-8 words.
**Bad directions** describe every UI element on screen. I will see the screen myself.

Bad: `> Antigravity terminal with empty workspace, Claude Code panel closed, file explorer on left showing no files`
Good: `> Antigravity: empty workspace`

**When to cut a direction entirely:**
- After `**I type:**` commands (I will see the result on my screen)
- After `**Claude outputs:**` lines (the output format already describes it)
- After action bullets that describe getting to a state (the bullet IS the direction)

**Bullets = narrated action.** I read this while doing the action on screen. Conversational tone. Use bullets for:
1. Physical actions (opening files, clicking panels, typing commands)
2. Output navigation (scrolling through results, moving to a different section)
3. Any mouse/keyboard action the viewer needs to follow

```markdown
- Click the Claude Code icon in the left sidebar
- Type "/find-niche" in the Claude Code input
- Scroll down to the "Competitor Pricing" section
- Click on the "Market Size" tab
- Scroll to the 3 recommended fixes
```

Bullets are required pacing breaks. Demo sections must NOT flow as continuous text. When I navigate output (read business results, then scroll to pricing, then scroll to MVP lines), each scroll is a bullet.

Rules:
- In talking head zones (A, C): no blockquotes or bullets needed. I'm talking to camera.
- In screen share zones (B): scene-setting directions use `>`. Every physical action gets a `- ` bullet.
- Never use `[SCREEN:]`, `[REMOTION:]`, `[Type:]`, `[Click:]`, `[Show:]`, `[Open:]`, `[Press:]`, `[Switch to:]` prefix syntax.
- Never use `[ brackets ]` for scene directions. Always use `> blockquotes`.
- Never put VIS IDs, animation names, or post-production overlay markers in directions.
- Keep directions to 3-8 words. If it takes more, you're describing too much.

### Demo Q&A format

When I demo a skill or type into Claude Code, use this exact format:

```markdown
**I type:** "/find-niche"

**Claude asks:** "What niche do you want to research?"

**I type:** "dental clinics Austin TX"

**Claude outputs:** "Good. That's specific enough for Claude to work with."
```

Between each Q&A pair, add 1-2 sentences of narration explaining what's happening or why it matters. Don't stack 5 Q&A pairs without any narration between them.

### Hook structure

The hook opens with a result flash showing the end result before the walkthrough:

```markdown
## 0:00-0:30 -- Hook (talking head)

> Flash: {end result, 3-5 words}

{Strongest claim from the result. Specific, not hype.}

{Call out viewer's current state or the noise they're dealing with.}

{Data point from research (specific numbers).}

{Lower the barrier: "No coding. Just follow along."}
```

### CTA format

CTA is email opt-in. "Want the full roadmap? Drop your email, link in the description." Teases the next video. Never directs to a specific product URL or starter kit download.

```markdown
## {time}-{time} -- Close + CTA (talking head)

> TALKING HEAD

{Recap the system in one paragraph. What they now have.}

{What else is worth knowing. Tease scope.}

{Email opt-in: "Want the full roadmap? Link in the description. Drop your email, I'll send you everything." + next video teaser.}
```

---

## Step 3: Pre-shoot checklist

Scan all `>` scene directions in the script and generate a preparation checklist. Insert it between the title and the first `---` divider.

### What to scan for

| Scene direction pattern | Checklist category |
|---|---|
| `> SCREEN SHARE:`, `> Split:`, `> Browser:` | **Screens to have ready** |
| `> .env`, `> business.md`, `> SKILL.md`, any file shown on screen | **Files to verify** |
| `**I type:** "/skill-name"` (live demos in Zone B) | **Skills to pre-test** |
| Apify scrape, Perplexity research, Supabase query, Stripe checkout | **API connections to confirm** |
| Prospect names, database rows, pre-existing outputs referenced | **Demo data needed** |
| `> Flash:` directions (pre-captured visuals, not live) | **Flashes to prepare** |

### Rules

- Only include categories that have items. Skip empty categories.
- Each item is a `- [ ]` checkbox.
- Keep items specific and actionable ("Antigravity with project folder open" not "Have IDE ready").
- Flashes are pre-captured screenshots or recordings (2-3 seconds). They are NOT live demos.
- Skills to pre-test = any `/skill` run LIVE on camera during Zone B. Run once before filming to verify output.
- If the script is pure talking head with only Flash directions, the checklist is just "Screens" and "Flashes".

### Output location

```markdown
# V{N} {Topic}: Script

## Pre-shoot checklist

### Screens to have ready
- [ ] Antigravity with project folder open
- [ ] Browser tab: dashboard

### Skills to pre-test (run once before filming)
- [ ] `/find-niche` returns scored results

### Flashes to prepare
- [ ] /find-niche output scored 17/20

---

## 0:00-... -- Hook (talking head)
```

---

## Step 4: Basic validation

After writing the full script, run these checks:

1. **All sections covered.** Every section from the outline appears in the script. Count `##` headings and compare against outline sections.
2. **Hook has result flash.** First section opens with `> Flash: ...` showing the end result.
3. **Scene directions correct.** Search for `[ ` (bracket with space) to catch old-format directions. All scene directions must use `> ` blockquote format. Also search for `[SCREEN:`, `[Type:`, `[Click:`, `[Show:`, `[REMOTION:`, `[Open:`, `[Press:`, `[Switch to:`, `VIS-`, `Animation:`. None should appear.
4. **CTA is email opt-in.** No "download starter kit", no direct product URL. "Want the full roadmap? Drop your email, link in the description." + next video teaser.
5. **Demo Q&A complete.** Every skill demo has `**I type:**` and either `**Claude asks:**` or `**Claude outputs:**` with narration between pairs.
6. **Word count.** Count total spoken words (exclude scene directions in blockquotes). Compare against target. If under minimum, expand the thinnest sections with specifics, examples, or data.
7. **Banned word scan.** Search for all Tier 1 and Tier 2 banned words from humanizer.md. Replace with plain alternatives.
8. **Headings.** All section headings use `##` with time ranges. Sentence case only. Each section ends with `---`. Zone A and C headers include `(talking head)`. Zone B headers have no zone label.
9. **Direction length.** Spot-check 5 blockquote directions. Each should be 3-8 words. If any are longer, shorten or cut.

Fix any failures before proceeding to Step 5.

---

## Step 5: Viral evaluation

Score the script against all 6 categories below. Each criterion is PASS or FAIL. A script passes at 29/29.

Print the full scorecard before making any changes.

---

### Category A: Hook (0:00 to first section break)

**A1. Result flash.** Hook opens with `> Flash: {result}` or equivalent blockquote. The viewer sees the end result before the walkthrough begins.
- PASS: Result flash present in first 3 lines of the hook section
- FAIL: No result flash, or it appears after spoken dialogue

**A2. Hook pattern.** The hook uses one of these 7 proven patterns:

| # | Pattern | What it does |
|---|---------|-------------|
| 1 | Macro timeline pressure | Opens with where the economy/industry is going. Creates urgency. |
| 2 | Authority-backed contrarian | Cites a respected figure, then contradicts or extends their argument. |
| 3 | Result flash + speed claim | Shows the end result, then states how fast it happened. |
| 4 | "For normal people" qualifier | Immediately disqualifies technical complexity. Expands audience. |
| 5 | Pain gap opener | Names the exact problem the viewer has right now. |
| 6 | Revenue + disbelief | States a revenue number that seems too good, then grounds it. |
| 7 | "Watch me do it" live-build | Commits to doing the entire thing on camera. No cuts. |

- PASS: Hook clearly matches at least one pattern
- FAIL: Hook is generic ("In this video I'll show you...") or doesn't match any pattern

**A3. Specific claim.** The hook's opening statement is specific, not vague. Contains a number, a timeframe, a result, or a named thing.
- PASS: "47 minutes," "$30K/month," "3 skills," etc.
- FAIL: "amazing results," "changed everything," "really powerful"

**A4. Viewer state callout.** The hook addresses where the viewer is right now. What they're doing wrong, what they're stuck on, or what noise they're dealing with.
- PASS: Explicit callout of viewer's current situation
- FAIL: Skips straight to the solution without naming the problem

**A5. Data point.** The hook includes at least one research-backed number. Not a claim. A fact with a source (even implied).
- PASS: Research-backed number present
- FAIL: No data, or data is vague ("studies show")

**A6. Barrier lowering.** The hook lowers the barrier to entry. Tells the viewer they don't need coding, a team, money, or experience.
- PASS: Explicit barrier-lowering statement
- FAIL: No accessibility signal

---

### Category B: Recording zones and section structure

**B1. Zone structure.** The script follows 3-zone pattern: Zone A (talking head) -> Zone B (screen share: ALL demos) -> Zone C (talking head: close). Two camera switches total.
- PASS: Clear 3-zone structure. No micro-switching. All demos in Zone B.
- FAIL: Zones missing, unmarked, or talking head/screen share mixed within a zone.

**B2. Screen share specificity.** Every blockquote `>` direction in Zone B is 3-8 words and tells me where to be.
- PASS: All directions short (3-8 words) and specific
- FAIL: Any vague direction, any direction longer than ~10 words, or redundant directions after Q&A format lines

**B3. Result-first demos.** Every skill demo section opens with `> Flash: {output}` before showing the process.
- PASS: All demo sections show result first
- FAIL: Any demo section starts with process before result

**B4. Output navigation bullets.** Demo sections that walk through Claude's output use `- Scroll to {section}` or `- Open {file}` bullets as pacing breaks.
- PASS: All demo sections with output walkthrough have bullets
- FAIL: Any demo output walkthrough reads as a wall of text

**B5. Scene direction format.** Blockquotes `> ` for silent screen changes. Bullets `- ` for narrated actions. No `[ bracket ]` format. No old-style markers.
- PASS: All scene directions use `> blockquote` format, all are 3-8 words
- FAIL: Any `[ bracket ]` directions or old-style markers found

**B6. Demo Q&A format.** Skill demos use `**I type:**`, `**Claude asks:**`, `**Claude outputs:**` with 1-2 sentences of narration between each pair.
- PASS: All demos have narration between Q&A pairs
- FAIL: Any stacked Q&A without narration, or missing format markers

**B7. Action bullets in demos.** Every physical action I perform on screen gets its own `- ` bullet.
- PASS: All significant screen actions have bullets
- FAIL: Actions described in narration prose instead of bullets

---

### Category C: CTA

**C1. Email opt-in.** CTA directs viewers to drop their email. Never "download the starter kit," never a direct product URL, never "subscribe" as the primary CTA.
- PASS: Email opt-in CTA present
- FAIL: Subscribe, download, or product URL as primary CTA

**C2. Data-backed framing.** Mid-roll or close CTA references specific research or numbers behind the offer.
- PASS: Data-backed reference present in at least one CTA
- FAIL: No data tie

**C3. Next video teaser.** Close CTA previews the next video in the series.
- PASS: Next video mentioned
- FAIL: No next video teaser (exception: last video in series can reference V1 or the full series)

**C4. CTA placement.** At least 2 CTA placements: one mid-roll (after a proof/demo section) and one at close.
- PASS: 2+ CTA moments
- FAIL: Only one CTA, or CTA before any proof is shown

---

### Category D: Language quality

**D1. Banned words.** Search for all Tier 1 and Tier 2 words from humanizer.md.
- PASS: Zero banned words in spoken dialogue (words inside `Claude outputs:` are exempt)
- FAIL: Any banned word in spoken dialogue
- Report: List every violation with line number and replacement

**D2. Banned phrases.** Search for: "In today's," "In the realm of," "It's worth noting," "game-changer," "best-in-class," "push the boundaries," "the future looks bright," "as we move forward," "Furthermore," "Moreover," "Additionally," "In conclusion."
- PASS: Zero banned phrases
- FAIL: Any banned phrase found

**D3. Sentence length variation.** Check for 4+ consecutive sentences with similar word counts (within +/-2 words).
- PASS: No runs of 4+ uniform-length sentences
- FAIL: Any uniform run found

**D4. Anti-hype check.** Search for empty adjective claims: "amazing," "incredible," "powerful," "game-changing," "revolutionary," "mind-blowing," "insane" (as praise).
- PASS: No empty adjective hype
- FAIL: Any hype adjective without a specific number or fact attached

**D5. Filler words.** Search for scripted filler: "so," at sentence starts, "basically," "you know," "right?" as rhetorical tags, "actually" as a hedge.
- PASS: No scripted fillers (exception: "so" mid-sentence is fine)
- FAIL: Any scripted filler found

**D6. Dollar formatting.** Dollar ranges in spoken scripts must be spelled out: "$3,000 to $10,000" not "$3,000-$10,000".
- PASS: All dollar ranges spelled out
- FAIL: Any hyphenated dollar range found

---

### Category F: Viral meta-patterns

**F1. System reveal over tutorial.** The video frames itself as "here's my system" not "here's how to use a tool."
- PASS: Script reads as a system reveal or live demonstration
- FAIL: Script reads as a step-by-step tutorial about tool features

**F2. Business-first positioning.** Results and business outcomes appear before tool explanations.
- PASS: Business outcomes mentioned before or alongside tool features
- FAIL: Tool features dominate without business context

**F3. Honest limits.** At least one moment of honesty about what the system can't do or where it fails.
- PASS: At least one honest limitation mentioned
- FAIL: Everything presented as perfect

**F4. Specificity over hype.** Spot-check 3 claims. Each should have a specific number, example, or data source.
- PASS: Claims backed by specifics
- FAIL: Claims are generic praise

---

### Category G: Outline sync check

**G1. Section coverage.** Every section in `02_OUTLINE.md` has a corresponding `##` heading in `03_SCRIPT.md`.
- PASS: Section count matches
- FAIL: Missing or extra sections

**G2. CTA consistency.** The CTA format in the script matches the outline's CTA placements table.
- PASS: Script CTA matches outline CTA table
- FAIL: Mismatch between script and outline CTA

---

### Scorecard format

```
================================================
  V{N} LONGFORM SCRIPT EVALUATION
================================================

Category A: Hook                    {X}/6
  A1 Result flash                   PASS/FAIL
  A2 Hook pattern                   PASS/FAIL  ({pattern name})
  A3 Specific claim                 PASS/FAIL
  A4 Viewer state callout           PASS/FAIL
  A5 Data point                     PASS/FAIL
  A6 Barrier lowering               PASS/FAIL

Category B: Zones & structure       {X}/7
  B1 Zone structure (A->B->C)        PASS/FAIL
  B2 Screen share specificity       PASS/FAIL
  B3 Result-first demos             PASS/FAIL
  B4 Output navigation bullets      PASS/FAIL
  B5 Scene direction format         PASS/FAIL
  B6 Demo Q&A format                PASS/FAIL
  B7 Action bullets in demos        PASS/FAIL

Category C: CTA                     {X}/4
  C1 Email opt-in                   PASS/FAIL
  C2 Data-backed framing            PASS/FAIL
  C3 Next video teaser              PASS/FAIL
  C4 CTA placement                  PASS/FAIL

Category D: Language quality         {X}/6
  D1 Banned words                   PASS/FAIL
  D2 Banned phrases                 PASS/FAIL
  D3 Sentence length variation      PASS/FAIL
  D4 Anti-hype                      PASS/FAIL
  D5 Filler words                   PASS/FAIL
  D6 Dollar formatting              PASS/FAIL

Category F: Viral meta-patterns      {X}/4
  F1 System reveal > tutorial       PASS/FAIL
  F2 Business-first positioning     PASS/FAIL
  F3 Honest limits                  PASS/FAIL
  F4 Specificity over hype          PASS/FAIL

Category G: Outline sync             {X}/2
  G1 Section coverage               PASS/FAIL
  G2 CTA consistency                PASS/FAIL

================================================
  TOTAL: {X}/29
  STATUS: PASS (29/29) / FAIL ({list failing criteria})
================================================
```

After the scorecard, list every FAIL with:
- Criterion code (e.g., D1)
- Line number(s) in the script
- What's wrong
- Suggested fix

---

## Step 6: Improve

For each FAIL:

1. Edit `03_SCRIPT.md` to fix the issue
2. Re-check the criterion to confirm it now passes
3. If the fix changes section structure, timing, or CTA content, note it for Step 7

Do NOT change sections that already pass. Only touch what fails.

**Maximum 3 improvement rounds.** If the script still has failures after 3 rounds, flag them and move on.

After all fixes, re-print the full scorecard showing the final score.

**Common fixes:**
- A1 fail: Add `> Flash: {result}` as first line of hook section
- B2 fail: Shorten blockquote directions to 3-8 words
- B6 fail: Add 1-2 narration sentences between stacked Q&A pairs
- D1 fail: Replace banned word with simple alternative (see humanizer.md Tier 4 table)
- D3 fail: Break up uniform sentence runs with a short or long sentence
- F3 fail: Add one honest limitation about the system

---

## Step 7: Sync outline (if script changed)

If Step 6 modified any of these in the script, update `02_OUTLINE.md` to match:

1. **Section names or order changed** -- Update the script outline section headings
2. **CTA wording changed** -- Update the CTA placements table
3. **New demo Q&A added** -- Add the Q&A to the corresponding outline section
4. **Hook restructured** -- Update the hook section in the outline's script outline
5. **Time ranges shifted** -- Update all time ranges in the outline

Only sync what changed. Don't rewrite the entire outline.

---

## Step 8: Final output

Save `03_SCRIPT.md` and print:

```
SCRIPT COMPLETE
  File: {path}
  Score: {X}/29
  Fixes applied: {N} edits
  Outline synced: Yes/No
  Failing criteria remaining: None / {list}
```

## Writing rules

1. **Voice:** 1st person ("I"), direct, blunt, data-driven. My natural voice. Not a presenter voice.
2. **Anti-hype:** Persuade with specifics and data, not adjectives. "This does X" not "This is amazing."
3. **Sentence rhythm:** Mix short and long. Two words is fine. So is a 40-word sentence. Humans are bursty.
4. **Hook:** Must grab in the first 5 seconds. Open with the most surprising or specific claim.
5. **Numbers:** Use concrete numbers from the research. "$599" not "affordable." "3,461 single-core" not "fast."
6. **No filler:** Cut "so," "basically," "you know," "right?" from the script. Those happen naturally on camera.
7. **AI Humanizer:** All banned words, phrases, and patterns from `.claude/rules/humanizer.md` are strictly prohibited.
8. **Sentence length variation:** Never write 4+ consecutive sentences with the same word count (within +/-2 words).
9. **Dollar range formatting:** Use spelled-out style: "$3,000 to $10,000" (not "$3,000-$10,000").
10. **Honest limits:** Include what the tool/product CAN'T do before what it CAN do. Builds trust.

## Length targets

| Video type | Minutes | Words |
|---|---|---|
| Standard | 10-14 | 1,800-2,500 |
| Deep dive | 15-20 | 2,500-3,500 |
| Long deep dive | 25-30 | 3,500-5,000 |
| Quick take | 6-9 | 1,000-1,600 |

## Content accuracy reference

Apply the content accuracy rules from yt-research.md. Read that file for the authoritative reference.

## Quality checklist

- [ ] Script file starts with `# V{N} {Topic}: Script` followed by `---`
- [ ] Every section from the outline is covered
- [ ] Section headings use `## {time}-{time} -- {name}` format, separated by `---`
- [ ] Zone A and C headers include `(talking head)`. Zone B headers have no zone label.
- [ ] Headings use sentence case
- [ ] Hook opens with `> Flash: {result}` blockquote
- [ ] All key facts/numbers from research included
- [ ] Scene directions use `> blockquotes` (3-8 words), NOT `[ brackets ]`
- [ ] Action bullets use `- ` for narrated actions
- [ ] No old-style markers: `[SCREEN:]`, `[Type:]`, `[Click:]`, no VIS IDs
- [ ] Demo sections use Q&A format with narration between pairs
- [ ] CTA is email opt-in with next video teaser
- [ ] Word count meets minimum for video type
- [ ] Viral eval: 29/29 (or failures flagged after 3 improvement rounds)
- [ ] Zero AI Humanizer violations
- [ ] Sentence length varies throughout
- [ ] Outline synced if script changed

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand. Removed Category E (module content accuracy) since it referenced program-specific structure. Total moved from /36 to /29. |
