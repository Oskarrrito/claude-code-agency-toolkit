---
name: yt-motion
description: Inserts Remotion infographic cutaways at relevant moments in a talking-head YouTube video. Claude identifies 3-5 moments, picks real UI templates (AppleNotes, XPost) or data templates (NumbersHighlight, SpecCard), renders clips, and wires them into TalkingHeadEdit's infographics array. Run after /yt-cut-edit.
argument-hint: "[folder-name]  e.g. v2-setup"
---

# /yt-motion

Reads the script and subtitle cards, identifies where animated infographics would clarify the content, renders them with Remotion, and inserts them into the TalkingHeadEdit composition via the `infographics` array in `motion_props.json`.

**No external AI calls.** Claude Code IS the AI - you identify the moments and write the props.

**Two rendering paths:**
1. **Inline infographics** - rendered as part of TalkingHeadEdit via the `infographics` array in `motion_props.json`. Clips are stored in `_engine/public/videos/infographics/` and referenced as relative paths.
2. **Standalone assembly** - rendered separately and assembled via ffmpeg (fallback if TalkingHeadEdit is not used).

**Prefer path 1** (inline) when the project uses TalkingHeadEdit (has `subtitle_cards_raw.json`).

---

## Step 1: Load context

1. Parse argument: folder name (e.g., `v2-setup`)

2. Set absolute paths:
   ```
   PROJECT_DIR  = {absolute_path}/content/youtube/{folder_name}/
   OUTPUT_DIR   = {absolute_path}/content/youtube/{folder_name}/output/
   ENGINE_DIR   = {absolute_path}/content/youtube/_engine/
   PUBLIC_INFOGRAPHICS = ENGINE_DIR/public/videos/infographics/
   ```
   Derive `{absolute_path}` from the current working directory (the project root).

3. Verify `cut_edit.mp4` and `subtitle_cards_raw.json` exist in OUTPUT_DIR. Fail with clear message if missing.

4. **Verify `gen_motion_props.py` exists in OUTPUT_DIR.** If it doesn't exist yet, the agent creates it from the `cut_edit.mp4` metadata and `subtitle_cards_raw.json`. This is the normal path for first-time runs. The generated script must read subtitle cards, compute video duration via ffprobe, and output `motion_props.json` with the `videoSrc`, `cards`, and `videoDurationSecs` fields that TalkingHeadEdit expects.

5. Create infographics directory:
   ```bash
   mkdir -p "{ENGINE_DIR}/public/videos/infographics"
   ```

---

## Step 2: Identify infographic moments

**This is you (Claude Code) working directly - no API call.**

1. Read `output/subtitle_cards_raw.json` - note timestamps and content of each card
2. Read `03_SCRIPT.md` if it exists

Match script/subtitle content against these specific patterns. **Prefer real UI templates** over abstract data templates when the content maps to a real-world interface.

### Pattern-matching rules (what triggers an infographic)

| Content pattern | Template | Example |
|---|---|---|
| Lists of 3+ items | `SpecCard`, `AppleNotes`, or `TextSlide` | Feature lists, tool lists, setup steps |
| Numbers, prices, statistics | `NumbersHighlight` | $200/month, 19 skills, $30,000 earn |
| Comparisons, before/after, good vs bad | `Comparison` or `AppleNotes` | Bad setup vs good setup |
| Step-by-step instructions | `ProcessFlow` | Setup steps with arrows |
| Named frameworks or mental models | `Framework` | Stacked concept blocks |
| Tweet/social proof | `XPost` | Real tweet with engagement stats |
| Title + key bullets (no better fit above) | `TextSlide` | Section header with key points |

### Disambiguation table (when content could match multiple templates)

| Situation | Use | Not |
|---|---|---|
| Checklist items (done/not done) | `AppleNotes` | `Comparison` |
| Social quote or tweet | `XPost` | `TextSlide` |
| Long description comparison (2+ sentences per side) | `Comparison` | `AppleNotes` |
| Short checklist comparison (single words/phrases) | `AppleNotes` | `Comparison` |
| Feature list with label:value pairs | `SpecCard` | `TextSlide` |
| Simple bullet list (no labels) | `TextSlide` | `SpecCard` |

### Skip rules (do NOT place an infographic here)

- Rhetorical questions or emotional beats (let the speaker land it)
- Content already shown via B-roll clip
- Fewer than 15 seconds since the previous infographic ended
- Moments with no concrete data (opinions, anecdotes, transitions)

### Placement rules

- Pick 3-5 moments max. Quality over quantity.
- Pick timestamps BETWEEN subtitle cards (not mid-sentence). Use the `end` time of a card as the `timestamp`.
- Duration: 5-6 seconds per infographic.
- Props must have real content from the video, don't invent data.
- **All templates render at 1920x1080 and must fill the full screen.** No small centered windows with dark borders.

### Template props formats

**`AppleNotes`** (macOS Notes checklist UI):
```json
{
  "title": "Claude Code Setup",
  "sections": [
    {
      "heading": "Bad Setup",
      "items": [
        {"text": "Claude doesnt know your business", "checked": true},
        {"text": "No access to your tools and data", "checked": true}
      ]
    },
    {
      "heading": "Good Setup",
      "items": [
        {"text": "Business context loaded via CLAUDE.md", "checked": false},
        {"text": "All tools connected via MCP servers", "checked": false}
      ]
    }
  ]
}
```

**`XPost`** (X/Twitter dark mode post):
```json
{
  "handle": "@username",
  "displayName": "Display Name",
  "text": "The tweet content",
  "likes": 6426,
  "reposts": 1823,
  "bookmarks": 10211,
  "timestamp": "Mar 28, 2026",
  "verified": false,
  "avatarColor": "#1D9BF0"
}
```

**`NumbersHighlight`** (count-up numbers):
```json
{
  "title": "The ROI",
  "numbers": [
    {"value": 200, "prefix": "$", "suffix": "/mo", "label": "Claude Max", "color": "#C15F3C"},
    {"value": 30000, "prefix": "$", "label": "Earn per project", "color": "#22C55E"},
    {"value": 107, "prefix": "$", "suffix": "/mo", "label": "Tools you skip", "color": "#3B82F6"}
  ]
}
```
**Important:** prefix and suffix are rendered as separate smaller spans - never concatenated into the number string. This prevents overflow.

**`SpecCard`** (feature grid):
```json
{
  "title": "19 Built-In Skills",
  "specs": [
    {"label": "Research", "value": "Topic + case study", "highlight": false},
    {"label": "Terminal", "value": "Claude Code OS", "highlight": true}
  ]
}
```

**`Comparison`** (two panels):
```json
{
  "heading": "Setup makes the difference",
  "left": {"title": "Bad Setup", "items": ["point", "point"], "color": "#EF4444"},
  "right": {"title": "Good Setup", "items": ["point", "point"], "color": "#22C55E"}
}
```

**`ProcessFlow`** (numbered steps with arrows):
```json
{
  "title": "5 Setup Steps",
  "steps": [
    {"number": 1, "label": "Install", "description": "npm install -g @anthropic/claude-code"},
    {"number": 2, "label": "Auth", "description": "claude login"}
  ]
}
```

**`Framework`** (stacked blocks):
```json
{
  "title": "The Speed Framework",
  "pieces": [
    {"label": "Earn", "description": "Money as fast as possible", "color": "#C15F3C"},
    {"label": "Invest", "description": "Fund your next move", "color": "#3B82F6"}
  ]
}
```

**`TextSlide`** (title + bullets):
```json
{
  "title": "What You Get",
  "subtitle": "Everything in one tool",
  "bullets": ["Research", "Outreach", "Payments"]
}
```

---

## Step 3: Render infographic clips

**Naming conventions:**
- `{NN}` = zero-padded two-digit index: 00, 01, 02, ...
- `{id}` = kebab-case slug from template name + title (e.g., `applenotes-claude-code-setup`, `numbershighlight-the-roi`)

### Cache/skip logic

Before rendering, check if clips already exist in `_engine/public/videos/infographics/`. If any clips exist, **delete them all first** (they are project-specific and stale from a previous run):

```bash
rm -f "{ENGINE_DIR}/public/videos/infographics/"*.mp4
```

### Render using file-based props (not inline JSON)

Inline `--props='{...}'` breaks when prop values contain single quotes (e.g., "doesn't"). Use a temp file instead.

For each planned infographic:

1. Write the props JSON to a temp file:
   ```bash
   cat > "{OUTPUT_DIR}/infographic_{NN}_props.json" << 'PROPEOF'
   {json_props}
   PROPEOF
   ```

2. Render with the temp file:
   ```bash
   cd "{ENGINE_DIR}" && npx remotion render src/index.ts {template} \
     "public/videos/infographics/{NN}_{id}.mp4" \
     --props="$(cat '{OUTPUT_DIR}/infographic_{NN}_props.json')" \
     --frames=0-{last_frame} --codec=h264 2>&1 | tail -3
   ```

3. Clean up the temp file:
   ```bash
   rm -f "{OUTPUT_DIR}/infographic_{NN}_props.json"
   ```

Where `last_frame = duration_seconds * 30 - 1` (e.g., 5s = frames 0-149).

Run each render sequentially. Print: `Rendered [{i}] {template} -> {path}`

If any render fails: print error, skip that cutaway, continue.

---

## Step 4: Wire infographics into gen_motion_props.py

This is the critical step. The `gen_motion_props.py` script must include the `infographics` array in the output JSON.

**Idempotent edit:** Read the existing `output/gen_motion_props.py`. If an `infographics = [...]` block already exists, **REPLACE it entirely** with the new list. If none exists, **ADD it** immediately before the `props = {` line. Either way, ensure `"infographics": infographics` is present in the props dict.

The infographics list looks like:

```python
infographics = [
    {
        "videoSrc": "videos/infographics/00_applenotes-setup-comparison.mp4",
        "timestamp": 20.4,   # seconds in cut timeline
        "duration":  6.0,
    },
    {
        "videoSrc": "videos/infographics/01_numbershighlight-nineteen-skills.mp4",
        "timestamp": 45.7,
        "duration":  5.0,
    },
]
```

Then ensure `"infographics": infographics` is in the props dict:

```python
props = {
    "videoSrc": "videos/cut_edit.mp4",
    "cards": [...],
    "videoDurationSecs": duration_secs,
    "infographics": infographics,
}
```

The `videoSrc` paths are relative to `_engine/public/`.

---

## Step 5: Render final_motion.mp4

This overwrites the subtitle-only version from yt-cut-edit. The final file contains both subtitles and infographic overlays.

Run `gen_motion_props.py` to rebuild the props JSON, then render via TalkingHeadEdit:

```bash
cd "{OUTPUT_DIR}/.." && python3 output/gen_motion_props.py
```

Then render:
```bash
cd "{ENGINE_DIR}" && PROPS=$(cat "{OUTPUT_DIR}/motion_props.json") && \
  npx remotion render src/index.ts TalkingHeadEdit \
  "{OUTPUT_DIR}/final_motion.mp4" \
  --props="$PROPS" --codec=h264 2>&1 | tail -4
```

### Failure handling

If the final render fails:
1. Validate `motion_props.json` is valid JSON (run `python3 -m json.tool "{OUTPUT_DIR}/motion_props.json"`).
2. If JSON is invalid, re-run `gen_motion_props.py` and check for syntax errors in the infographics block.
3. Retry the render **once**. If it fails again, report the error and stop.

---

## Step 6: Report

```
═══════════════════════════════════════
  YT-MOTION COMPLETE
═══════════════════════════════════════
  Project:     {folder_name}
  Cutaways:    {N} infographics inserted
  Templates:   {list of templates used}
  Output:      output/final_motion.mp4  ({duration:.0f}s)

  Infographics:
  {for each: timestamp - template - title}
═══════════════════════════════════════
```

---

## Pipeline order

```
/yt-cut-edit {folder}   →  cut_edit.mp4 + final.mp4 (cut + subtitles)
/yt-motion {folder}     →  final_motion.mp4          (+ Remotion infographics via TalkingHeadEdit)
/yt-broll {folder}      →  final_broll.mp4           (+ B-roll cutaways)
```

---

## Design rules

See `.claude/rules/motion-graphics-design.md` for the full spec. Key points:
- All templates fill 1920x1080 - no small centered cards with dark borders
- Number prefix/suffix as separate spans (36px) from the main number (72px)
- Spring config: `{ damping: 200, mass: 0.5, stiffness: 180 }` for titles, `{ damping: 14, mass: 0.6, stiffness: 160 }` for card entrances (slight overshoot)
- Step colors cycle: `[#C15F3C, #3B82F6, #22C55E, #F59E0B, #8B5CF6]`
- All templates include premium visual treatment (see below)

---

## Premium visual standards

All templates now include premium effects matching the the premium reference video reference quality. These are built into every template component and render automatically. No extra props needed.

### Effects in all data templates (NumbersHighlight, SpecCard, Comparison, ProcessFlow, Framework, TextSlide):

1. **Dot grid background** - subtle radial-gradient dots at 40px spacing (rgba white 4% opacity). Creates depth without distraction.
2. **Gradient card backgrounds** - `linear-gradient(180deg, #1a1a22 0%, #141418 100%)` instead of flat bgCard. Adds subtle dimension.
3. **Glow orbs behind key numbers** - Absolutely positioned radial-gradient circles with blur(30px) at 20% color opacity. Used behind NumbersHighlight values and ProcessFlow step numbers.
4. **TextShadow on important text** - `0 0 20px color(40%)` on numbers, values, and panel headers. Soft luminance effect.
5. **BoxShadow with colored glow on cards** - `0 0 30px color(7%), 0 4px 16px rgba(0,0,0,0.4)`. Every card has a tinted ambient glow plus depth shadow.
6. **Animated accent lines under titles** - Growing width from 0 to 64px synced to title spring progress, with `boxShadow: 0 0 8px color(40%)` for glow.
7. **Gradient accent bars** - Top/bottom bars use `linear-gradient(90deg, solid, 30% fade)` instead of flat color. Top bars include boxShadow glow on highlighted cards.
8. **Entrance spring with slight overshoot** - Cards use `{ damping: 14, mass: 0.6, stiffness: 160 }` for organic feel. Titles still use the smooth config.

### Effects in UI templates:

- **XPost**: Subtle radial gradient background behind post (blue 4% opacity), avatar glow shadow, textShadow on engagement numbers colored by action type (green reposts, blue bookmarks, pink likes), liked heart glow.
- **AppleNotes**: Enhanced window depth shadow (0 24px 80px), checked items get yellow glow on checkbox, section headers have animated accent bars matching Apple Notes yellow, sidebar selected item has glow shadow.

### What this means for rendering:

- Templates produce premium output by default. No additional configuration needed.
- The effects use only inline React styles (no CSS files, no external dependencies).
- All effects are frame-synchronized via Remotion spring() and interpolate().
- hexToRgba() utility handles all transparent color calculations locally in each template.

---

## Quality checklist

- [ ] 3-5 moments identified - no more, no less
- [ ] All timestamps fall at card boundaries (not mid-sentence)
- [ ] Props contain real data from the video (nothing invented)
- [ ] Each Remotion clip rendered successfully to _engine/public/videos/infographics/
- [ ] gen_motion_props.py updated with `infographics` array (idempotent - replaced if already present)
- [ ] motion_props.json contains infographics array (not empty)
- [ ] final_motion.mp4 rendered via TalkingHeadEdit with infographics
- [ ] Output duration matches cut_edit.mp4 duration (plus or minus 0.5s)
- [ ] All UI templates fill full 1920x1080 (no small windows)
- [ ] All rendered clips have premium visual treatment (glow, gradient, depth)

---

## Change log

| Date | Changes |
|---|---|
| 2026-03-28 | Initial version. Remotion cutaway infographics, 6 templates, spring entrance style. |
| 2026-03-29 | v2: Added AppleNotes + XPost real UI templates. Infographics wired into TalkingHeadEdit via gen_motion_props.py infographics array. All templates fill full 1920x1080. Design rules reference. NumbersHighlight prefix/suffix split to prevent overflow. |
| 2026-03-29 | v3: Fixed from eval report. (1) Step 3 uses file-based props instead of inline JSON to avoid single-quote breakage. (2) Step 4 gen_motion_props.py edits are now idempotent (replace existing infographics block). (3) Step 2 has concrete pattern-matching rules, disambiguation table, and skip rules instead of vague "scan for content." (4) Step 1 fails early if gen_motion_props.py missing. Step 5 validates JSON and retries once on render failure. (5) Step 3 clears stale infographic clips before rendering. (6) Defined {NN} and {id} naming conventions. |
| 2026-03-29 | v4: Premium visual upgrade. All 8 templates upgraded to match the premium reference video quality: dot grid backgrounds, gradient card backgrounds, glow orbs behind numbers, textShadow on values/headers, boxShadow with colored glow on all cards, animated accent lines with glow, gradient accent bars, entrance spring with slight overshoot. XPost gets radial bg gradient + engagement number glow. AppleNotes gets enhanced window shadow + checked item glow + section header accent bars. Added "Premium visual standards" section and updated quality checklist. |
| 2026-08-19 | Sanitized and republished under NiteOps brand |
