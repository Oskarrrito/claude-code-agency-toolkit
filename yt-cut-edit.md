---
name: yt-cut-edit
description: Automated talking-head cut editor. Removes retakes, false starts, and silences. Burns YouTube-style semi-transparent subtitles. Uses Deepgram for transcription and Claude Code for AI cut decisions + subtitle corrections - no Anthropic API key needed. Run from any video project folder with footage/ and 03_SCRIPT.md.
argument-hint: "[video folder name, e.g. v2-setup] [--speed 1.1] [--sample 60]"
---

# /yt-cut-edit

Full pipeline: raw footage → Deepgram transcription → Claude Code cut decisions → ffmpeg edit → subtitle generation → Claude Code corrections → final burned video.

**No Anthropic API key.** Claude Code IS the AI. You read the transcript directly and make decisions. Python handles only technical work (Deepgram, ffmpeg, silence detection).

---

## Step 1: Load context

1. Parse arguments from `$ARGUMENTS`:
   - Folder name: first non-flag argument (e.g., `v2-setup`)
   - `--speed N`: playback speed multiplier (default: `1.10`)
   - `--sample N`: render only first N seconds of final output. Store as `SAMPLE_SECONDS = N`. If not provided, `SAMPLE_SECONDS = None` (full video).

2. Set paths:
   ```
   PROJECT_DIR = content/youtube/{folder_name}/
   FOOTAGE_DIR = content/youtube/{folder_name}/footage/
   OUTPUT_DIR  = content/youtube/{folder_name}/output/
   SCRIPT_PATH = content/youtube/{folder_name}/03_SCRIPT.md
   ```

3. Find footage file: first `.mp4` in `footage/` sorted alphabetically. Fail with clear message if not found.

4. Create `output/` directory if it doesn't exist.

5. Load `DEEPGRAM_API_KEY`:
   - Check environment variable `DEEPGRAM_API_KEY`
   - If not set, read `.env` in project root - look for line `DEEPGRAM_API_KEY=...`
   - Fail with message "Set DEEPGRAM_API_KEY in .env" if not found.

6. Run `ffprobe` to get video duration:
   ```bash
   ffprobe -v quiet -show_entries format=duration -of csv=p=0 "{footage_file}"
   ```
   Store as `SOURCE_DURATION`.

---

## Step 2: Transcribe with Deepgram

**Skip this step if `output/transcript_dg.json` already exists** - print "Using cached transcript" and continue.

Write `output/transcribe.py` with this content, substituting `{footage}`, `{output_dir}`, `{source_duration}`, `{api_key}`:

```python
#!/usr/bin/env python3
"""Extract audio and transcribe with Deepgram Nova-3."""
import subprocess, requests, json, os, sys

FOOTAGE = "{footage}"
OUT_DIR = "{output_dir}"
API_KEY = "{api_key}"
SOURCE_DURATION = {source_duration}

audio_path = os.path.join(OUT_DIR, "audio.wav")
print("Extracting audio...")
r = subprocess.run([
    "ffmpeg", "-y", "-i", FOOTAGE, "-t", str(SOURCE_DURATION),
    "-vn", "-ar", "16000", "-ac", "1",
    "-af", "highpass=f=200,lowpass=f=3000,afftdn=nf=-25,loudnorm",
    audio_path
], capture_output=True, text=True)
if r.returncode != 0:
    print("Audio extract error:", r.stderr[-500:]); sys.exit(1)
print("Audio extracted.")

print("Transcribing with Deepgram Nova-3...")
with open(audio_path, "rb") as f:
    audio = f.read()

resp = requests.post(
    "https://api.deepgram.com/v1/listen",
    headers={"Authorization": f"Token {API_KEY}", "Content-Type": "audio/wav"},
    params={
        "model": "nova-3", "language": "en", "smart_format": "true",
        "punctuate": "true", "utterances": "true",
        "words": "true", "filler_words": "true"
    },
    data=audio, timeout=300
)
if resp.status_code != 200:
    print(f"Deepgram error {resp.status_code}:", resp.text[:500]); sys.exit(1)

dg = resp.json()
out = os.path.join(OUT_DIR, "transcript_dg.json")
with open(out, "w") as f:
    json.dump(dg, f, indent=2)

words = dg["results"]["channels"][0]["alternatives"][0]["words"]
utts  = dg["results"]["utterances"]
print(f"✓ Transcribed: {len(words)} words, {len(utts)} utterances → {out}")
```

Run it:
```bash
python3 output/transcribe.py
```

If it fails: check API key, check ffmpeg is installed, check network.

---

## Step 3: Claude Code makes cut decisions

**This is you (Claude Code) working directly - no API call.**

1. Read `output/transcript_dg.json` and list all utterances. Use the absolute path you set in Step 1 for OUTPUT_DIR:
   ```bash
   python3 -c "
   import json
   out = '/absolute/path/to/output'  # substitute actual OUTPUT_DIR path
   with open(out + '/transcript_dg.json') as f: dg = json.load(f)
   for i, u in enumerate(dg['results']['utterances']):
       print(f'[{i}] {u[\"start\"]:.1f}-{u[\"end\"]:.1f}s: {u[\"transcript\"]}')
   "
   ```

2. Also read `03_SCRIPT.md` if it exists - compare utterances against the script to identify retakes more accurately (speaker says the same scripted sentence more than once; keep the last/cleanest version).

3. Make keep/cut decisions using these rules:

   **CUT these:**
   - False starts: speaker begins a sentence, stops, restarts with the same content
   - Retakes: same sentence said again (cut ALL earlier attempts, keep the LAST/BEST version)
   - Filler utterances: "here", "success case", "one more", "this is a real one", "let me try again", standalone affirmatives
   - Incomplete thoughts abandoned mid-sentence
   - Stumbling/incoherent phrasing ("It it will burn through a month. It will burn through a day.")
   - Self-corrections where the clean version appears later

   **KEEP these:**
   - Complete, clear sentences
   - Final/best version of any retaken content
   - All meaningful content, even if brief
   - Transitions between topics

   **Be aggressive.** A tight video beats a complete one.

4. Write decisions to `output/ai_decisions.json`. Every utterance id must appear exactly once:
   ```json
   [
     {"id": 0, "action": "keep"},
     {"id": 1, "action": "cut", "reason": "false start - restarted sentence"},
     {"id": 2, "action": "cut", "reason": "retake of utterance 3"}
   ]
   ```

5. Validate before saving: count total decisions equals total utterances. Print summary: `{N} keep, {N} cut`.

---

## Step 4: Cut video and generate raw subtitle cards

Write `output/process.py`, substituting `{footage}`, `{output_dir}`, `{source_duration}`, `{speed}`:

```python
#!/usr/bin/env python3
"""
Cut video: apply AI decisions + silence removal + speed.
Generate utterance-boundary phrase subtitle cards.
"""
import json, subprocess, re, os, sys

FOOTAGE = "{footage}"
OUT_DIR = "{output_dir}"
SOURCE_DURATION = {source_duration}
SPEED = {speed}
GAP_KEEP = 0.12
SILENCE_NOISE = -35
SILENCE_MIN   = 0.40
MAX_CHARS = 52
PHRASE_GAP = 0.40
MIN_CARD_DUR = 0.8

WORD_FIXES = [
    (r'\bclothecode\b','Claude Code'),(r'\bclothcode\b','Claude Code'),
    (r'\bclosed code\b','Claude Code'),(r'\bcloak to code\b','Claude Code'),
    (r'\bclotakote\b','Claude Code'),(r'\bcloakcode\b','Claude Code'),
    (r'\bcloud code\b','Claude Code'),
    (r'\bantigravity\b','Antigravity'),
    (r'\bsmall test\b','smoke test'),(r'\bin older\b','in order'),
    (r'\bweb scrapping\b','web scraping'),
    (r'\bcode called max\b','Claude Max'),(r'\bcalled max\b','Claude Max'),
]
def fix(t):
    for p, r in WORD_FIXES: t = re.sub(p, r, t, flags=re.IGNORECASE)
    return t

with open(OUT_DIR + '/transcript_dg.json') as f: dg = json.load(f)
with open(OUT_DIR + '/ai_decisions.json') as f: decisions = json.load(f)

words_raw  = dg['results']['channels'][0]['alternatives'][0]['words']
utterances = dg['results']['utterances']
words = [{'word': fix(w['word']), 'start': w['start'], 'end': w['end']} for w in words_raw]

# Validate all utterances have a decision
decision_ids = {d['id'] for d in decisions}
missing = [i for i in range(len(utterances)) if i not in decision_ids]
if missing:
    print(f"WARNING: {len(missing)} utterances have no decision, defaulting to KEEP: {missing}")
    for m in missing:
        decisions.append({"id": m, "action": "keep"})

# Bad zones from cut decisions
bad_zones = []
for d in decisions:
    if d['action'] == 'cut':
        u = utterances[d['id']]
        bad_zones.append((u['start'] - 0.05, u['end'] + 0.05))

# Detect intra-utterance retakes in kept utterances
def find_intra_retake(utt_words, min_ngram=3):
    texts = [w['word'].lower().strip('.,!?') for w in utt_words]
    n = len(texts)
    for ng in range(min(7, n//2), min_ngram-1, -1):
        for i in range(n - ng):
            seq = texts[i:i+ng]
            for j in range(i+1, n-ng+1):
                if texts[j:j+ng] == seq:
                    return utt_words[i]['start'], utt_words[j]['start']
    return None

for d in decisions:
    if d['action'] == 'keep':
        u = utterances[d['id']]
        uw = [w for w in words if u['start']-0.1 <= w['start'] <= u['end']+0.1]
        if len(uw) >= 6:
            r = find_intra_retake(uw)
            if r: bad_zones.append((r[0]-0.05, r[1]+0.05))

bad_zones.sort()
merged = []
for bz in bad_zones:
    if merged and bz[0] <= merged[-1][1]:
        merged[-1] = (merged[-1][0], max(merged[-1][1], bz[1]))
    else:
        merged.append(list(bz))
bad_zones = [tuple(b) for b in merged]

# Silence detection
def detect_silences(src, dur):
    cmd = ["ffmpeg","-i",src,"-t",str(dur),
           "-af",f"silencedetect=noise={SILENCE_NOISE}dB:duration={SILENCE_MIN}",
           "-f","null","/dev/null"]
    r = subprocess.run(cmd, capture_output=True, text=True)
    ss = [float(x) for x in re.findall(r"silence_start: ([\d.]+)", r.stderr)]
    se = [float(x) for x in re.findall(r"silence_end: ([\d.]+)",   r.stderr)]
    return list(zip(ss, se))

silences = detect_silences(FOOTAGE, SOURCE_DURATION)
first_word = words[0]['start'] if words else 0
keep = []
cursor = max(0, first_word - 0.15)
for ss, se in silences:
    if ss < cursor: continue
    if ss > cursor: keep.append((cursor, ss))
    cursor = se - GAP_KEEP
if cursor < SOURCE_DURATION:
    keep.append((cursor, SOURCE_DURATION))

def subtract(segs, bads):
    result = []
    for ks, ke in segs:
        parts = [(ks, ke)]
        for bs, be in bads:
            np2 = []
            for s, e in parts:
                if be <= s or bs >= e: np2.append((s, e))
                elif bs <= s and be >= e: pass
                elif bs > s and be < e: np2.extend([(s,bs),(be,e)])
                elif bs <= s: np2.append((be, e))
                else: np2.append((s, bs))
            parts = np2
        result.extend(parts)
    return [(s, e) for s, e in result if e - s > 0.08]

keep = subtract(keep, bad_zones)
total = sum(e-s for s,e in keep)
print(f"Keep: {len(keep)} segments, {total:.1f}s → {total/SPEED:.1f}s at {SPEED}x speed")

# ffmpeg cut edit
n = len(keep)
fp = []
for i,(s,e) in enumerate(keep):
    fp.append(f"[0:v]trim=start={s:.4f}:end={e:.4f},setpts=PTS-STARTPTS[v{i}];")
    fp.append(f"[0:a]atrim=start={s:.4f}:end={e:.4f},asetpts=PTS-STARTPTS[a{i}];")
fp.append("".join(f"[v{i}]" for i in range(n)) + f"concat=n={n}:v=1:a=0[vcat];")
fp.append("".join(f"[a{i}]" for i in range(n)) + f"concat=n={n}:v=0:a=1[acat];")
fp.append(f"[vcat]setpts={1/SPEED:.4f}*PTS[vout];")
fp.append(f"[acat]atempo={SPEED}[aout]")

ff = OUT_DIR + '/filter.txt'
with open(ff,"w") as f: f.write("\n".join(fp))
cut_path = OUT_DIR + '/cut_edit.mp4'
print("Rendering cut edit...")
r = subprocess.run([
    "ffmpeg","-y","-i",FOOTAGE,"-filter_complex_script",ff,
    "-map","[vout]","-map","[aout]",
    "-c:v","hevc_videotoolbox","-q:v","65","-tag:v","hvc1","-c:a","aac","-b:a","192k", cut_path
], capture_output=True, text=True)
if r.returncode != 0: print("ERROR:", r.stderr[-2000:]); sys.exit(1)
print("Cut edit done.")

# Map word timestamps to cut timeline
def orig_to_cut(t, keep_segs, speed):
    acc = 0.0
    for s, e in keep_segs:
        if t <= s: break
        if t <= e: acc += (t - s); break
        acc += (e - s)
    return acc / speed

def word_ok(ws, we, bads):
    return not any(ws < be and we > bs for bs, be in bads)

def get_utt_id(w_start, utts):
    for i, u in enumerate(utts):
        if u['start'] - 0.1 <= w_start <= u['end'] + 0.1:
            return i
    return -1

good_words = []
for w in words:
    if word_ok(w['start'], w['end'], bad_zones) and w['start'] < SOURCE_DURATION:
        good_words.append({
            'word': w['word'],
            'cut_start': orig_to_cut(w['start'], keep, SPEED),
            'cut_end':   orig_to_cut(w['end'],   keep, SPEED),
            'utt_id':    get_utt_id(w['start'], utterances),
        })

print(f"Good words: {len(good_words)}")

# Build phrase-level subtitle cards (utterance boundaries = forced breaks)
phrases = []
buf = []
buf_chars = 0
prev_utt_id = -1

for w in good_words:
    word_text = w['word']
    gap = (w['cut_start'] - buf[-1]['cut_end']) if buf else 0
    utt_changed = (w['utt_id'] != prev_utt_id and prev_utt_id != -1 and buf)
    should_break = (
        utt_changed or
        (gap > PHRASE_GAP and buf) or
        (buf_chars + len(word_text) + 1 > MAX_CHARS and buf)
    )
    if should_break:
        phrases.append(buf); buf = []; buf_chars = 0
    buf.append(w)
    buf_chars += len(word_text) + 1
    prev_utt_id = w['utt_id']
if buf: phrases.append(buf)

# Build raw subtitle entries
raw_subs = []
for phrase in phrases:
    text = fix(' '.join(w['word'] for w in phrase))
    t0 = phrase[0]['cut_start']
    t1 = phrase[-1]['cut_end']
    raw_subs.append({'text': text, 'start': t0, 'end': t1})

# Auto-split any card over MAX_CHARS at last space before the limit
split_subs = []
for sub in raw_subs:
    text = sub['text']
    if len(text) <= MAX_CHARS:
        split_subs.append(sub)
        continue
    # Find split point: last space at or before MAX_CHARS
    split_at = text.rfind(' ', 0, MAX_CHARS)
    if split_at == -1:
        split_subs.append(sub)  # can't split, keep as is
        continue
    # Split timing: use midpoint as approximation
    mid_t = sub['start'] + (sub['end'] - sub['start']) * (split_at / len(text))
    split_subs.append({'text': text[:split_at], 'start': sub['start'], 'end': mid_t})
    split_subs.append({'text': text[split_at+1:], 'start': mid_t, 'end': sub['end']})
raw_subs = split_subs

# Merge very short cards with adjacent card
merged_subs = []
for sub in raw_subs:
    dur = sub['end'] - sub['start']
    if (merged_subs and dur < MIN_CARD_DUR and
        sub['start'] - merged_subs[-1]['end'] < 0.3 and
        len(merged_subs[-1]['text']) + 1 + len(sub['text']) <= MAX_CHARS + 10):
        merged_subs[-1]['text'] += ' ' + sub['text']
        merged_subs[-1]['end'] = sub['end']
    else:
        merged_subs.append(dict(sub))

out = OUT_DIR + '/subtitle_cards_raw.json'
with open(out, "w") as f: json.dump(merged_subs, f, indent=2)
print(f"Raw subtitle cards: {len(merged_subs)} → {out}")
print("\nPREVIEW (check for text issues before Step 5):")
for i, s in enumerate(merged_subs):
    flag = " *** LONG" if len(s['text']) > MAX_CHARS else ""
    print(f"  [{i:02d}] [{s['start']:.2f}-{s['end']:.2f}s] \"{s['text']}\" ({len(s['text'])}c){flag}")
```

Run it:
```bash
python3 output/process.py
```

---

## Step 5: Claude Code corrects subtitles (thorough double-check)

**This is you (Claude Code) working directly - no API call.**

1. Read `output/subtitle_cards_raw.json`
2. Read `03_SCRIPT.md` - use as a REFERENCE, not a strict match

**This step is critical. Review EVERY single card. Do not skip any.**

**Important: the script is a guide, not gospel.** The speaker often adds improvised explanations, examples, or tangents that aren't in the script. These additions are intentional and must be kept as-is. Only use the script to:
- Identify correct spelling of names, products, and technical terms
- Verify numbers and statistics
- Fix clear Deepgram mishears where the script shows what was actually said
- Do NOT delete or flag content that isn't in the script - it was said on purpose

**Text corrections (fix mishears):**
- "clothecode", "closed code", "cloak to code", "cloud code", "claw the cord" → "Claude Code"
- "clothe codes", "closed codes" → "Claude Code's"
- "clothe revenue", "clothes revenue" → "Claude's revenue"
- "code called max", "called max" → "Claude Max"
- "antigravity" → "Antigravity"
- "web scrapping" → "web scraping"
- "in older" → "in order"
- "small test" → "smoke test"
- Compare against script for any other mishears (names, numbers, technical terms)

**Capitalization - fix ALL of these:**
- `I` (standalone pronoun) - ALWAYS capitalize. Check every card.
- `AI`, `ROI`, `MCP`, `API`, `SaaS`, `CRM`, `IDE`, `DM`, `CEO` - always uppercase
- Product names: `Claude Code`, `Antigravity`, `Supabase`, `Apify`, `Stripe`, `Perplexity`, `Resend`, `Instantly`, `Cursor`, `FutureFlow`
- Person names: `Mark Cuban`, `Codie Sanchez`, `Dan Martell`, `Hormozi`, `Koning`
- Framework names: capitalize first letters (`Value Equation`, `Observation/Offer/Soft Ask`, `ACA`)
- Skill names: use slash format (`/find-niche`, `/build-offer`, `/outreach-engine`, `/scoreboard`, `/business`)
- Sentence starts: capitalize first word of every card

**Punctuation:**
- Add periods at sentence ends
- Add commas for natural pauses
- Add colons/question marks where script uses them

**Numbers:**
- Cross-check all dollar amounts, percentages, and counts against the script

**Do NOT rewrite** what was said. Only fix mishears, capitalization, punctuation, and numbers.

Write `output/subtitle_corrections.json` - include ALL cards that need ANY change:
```json
[
  {"id": 0, "text": "corrected text here"},
  {"id": 7, "text": "corrected longer card text"}
]
```

Print: `Applied N corrections`.

---

## Step 6: Burn subtitles with ffmpeg + ASS

Uses ffmpeg with ASS subtitles for semi-transparent pill-style captions. Hardware-accelerated encoding via `hevc_videotoolbox` on Apple Silicon. Renders in 2-3 minutes regardless of video length.

Write `output/burn_subs.py`, substituting `{output_dir}`:

```python
#!/usr/bin/env python3
"""
Generate ASS subtitles with semi-transparent pill background + burn onto cut_edit.mp4.
Uses hevc_videotoolbox for hardware-accelerated encoding.
"""
import json, os, re, subprocess, sys

OUT_DIR = "{output_dir}"
CUT_VIDEO = OUT_DIR + "/cut_edit.mp4"
ASS_PATH = OUT_DIR + "/captions.ass"
FINAL_PATH = OUT_DIR + "/final_motion.mp4"

with open(OUT_DIR + "/subtitle_cards_raw.json") as f:
    cards = json.load(f)

try:
    with open(OUT_DIR + "/subtitle_corrections.json") as f:
        corrections = {c['id']: c['text'] for c in json.load(f)}
    for i, card in enumerate(cards):
        if i in corrections:
            card['text'] = corrections[i]
except FileNotFoundError:
    pass

WORD_FIXES = [
    (r'\bclothecode\b','Claude Code'),(r'\bclothcode\b','Claude Code'),
    (r'\bclosed code\b','Claude Code'),(r'\bcloak to code\b','Claude Code'),
    (r'\bclotakote\b','Claude Code'),(r'\bcloakcode\b','Claude Code'),
    (r'\bcloud code\b','Claude Code'),(r'\bclothe codes?\b','Claude Code'),
    (r'\bclaw the cord\b','Claude Code'),(r'\bcloud to code\b','Claude Code'),
    (r'\bClothe revenue\b',"Claude's revenue"),
    (r'\bantigravity\b','Antigravity'),
    (r'\bsmall test\b','smoke test'),(r'\bin older\b','in order'),
    (r'\bweb scrapping\b','web scraping'),
    (r'\bcode called max\b','Claude Max'),(r'\bcalled max\b','Claude Max'),
]
def fix(t):
    for p, r in WORD_FIXES: t = re.sub(p, r, t, flags=re.IGNORECASE)
    return t

for card in cards:
    card['text'] = fix(card['text'])

for i in range(len(cards) - 1):
    cards[i]['end'] = cards[i+1]['start']

def secs_to_ass(t):
    h = int(t // 3600)
    m = int((t % 3600) // 60)
    s = t % 60
    return f"{h}:{m:02d}:{s:05.2f}"

# ASS subtitle file
# BorderStyle=4 = opaque box (pill effect)
# Font: Inter Medium 64px, white text, 90% opaque black background
# BorderStyle=3 = opaque box with rounded outline, Outline=4 for border-radius
# Bold=0 for medium weight
ass_content = r"""[Script Info]
Title: Cut Edit Captions
ScriptType: v4.00+
PlayResX: 1920
PlayResY: 1080
WrapStyle: 0

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Default,Inter Medium,64,&H00FFFFFF,&H000000FF,&H19000000,&H19000000,0,0,0,0,100,100,0.5,0,3,4,0,2,60,60,80,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
"""

for card in cards:
    start = secs_to_ass(card['start'])
    end = secs_to_ass(card['end'])
    text = card['text'].replace('\\', '\\\\').replace('{', '\\{').replace('}', '\\}')
    ass_content += f"Dialogue: 0,{start},{end},Default,,0,0,0,,{text}\n"

with open(ASS_PATH, "w") as f:
    f.write(ass_content)
print(f"ASS subtitles: {len(cards)} cards")

# Write captions.srt for reference/re-editing
srt_path = OUT_DIR + "/captions.srt"
with open(srt_path, "w") as f:
    for i, c in enumerate(cards):
        def ts(t):
            h,m,s = int(t//3600), int((t%3600)//60), t%60
            return f"{h:02d}:{m:02d}:{s:06.3f}".replace('.', ',')
        f.write(f"{i+1}\n{ts(c['start'])} --> {ts(c['end'])}\n{c['text']}\n\n")

# Burn subtitles with hardware encoding
print("Rendering final video with hardware encoding...")
cmd = [
    "ffmpeg", "-y", "-i", CUT_VIDEO,
    "-vf", f"ass='{ASS_PATH}'",
    "-c:v", "hevc_videotoolbox", "-q:v", "65", "-tag:v", "hvc1",
    "-c:a", "aac", "-b:a", "192k",
    FINAL_PATH
]
r = subprocess.run(cmd, capture_output=True, text=True)
if r.returncode != 0:
    print("hevc_videotoolbox failed, falling back to libx264...")
    cmd[-7:-4] = ["-c:v", "libx264", "-preset", "fast", "-crf", "18"]
    del cmd[cmd.index("-tag:v"):cmd.index("-tag:v")+2] if "-tag:v" in cmd else None
    r = subprocess.run(cmd, capture_output=True, text=True)
    if r.returncode != 0:
        print("ERROR:", r.stderr[-2000:]); sys.exit(1)

dur = subprocess.run(
    ["ffprobe","-v","quiet","-show_entries","format=duration","-of","csv=p=0", FINAL_PATH],
    capture_output=True, text=True
).stdout.strip()
print(f"✓ Final video: {FINAL_PATH} ({float(dur):.1f}s)")
print(f"✓ Captions: {srt_path}")
```

Run it:
```bash
python3 output/burn_subs.py
```

- Render time: 2-3 minutes for ANY length video (hardware encoding)
- Output: `output/final_motion.mp4`
  Note: yt-motion overwrites this file with the infographic-overlaid version. This is intentional -- the pipeline builds incrementally.
- Also outputs: `output/captions.srt` and `output/captions.ass`

---

## Step 7: Verify and report

```bash
ffprobe -v quiet -show_entries format=duration -of csv=p=0 output/final_motion.mp4
ffprobe -v quiet -show_entries format=duration -of csv=p=0 output/cut_edit.mp4
```

Print:

```
═══════════════════════════════════════
  YT-CUT-EDIT COMPLETE
═══════════════════════════════════════
  Project:    {folder_name}
  Source:     {source_duration:.0f}s raw footage
  Cut edit:   {cut_edit_duration:.0f}s (removed {removed:.0f}s, {pct:.0f}%)
  Final:      {final_duration:.0f}s with subtitles
  Speed:      {speed}x
  Subtitles:  {N} cards

  Output files:
    output/cut_edit.mp4        (cut video, no subtitles)
    output/captions.srt        (subtitle file, re-editable)
    output/captions.ass        (styled ASS subtitle file)
    output/final_motion.mp4    (final with burned subtitles)
    output/transcript_dg.json
    output/ai_decisions.json
═══════════════════════════════════════
```

---

## Cut decision reference

| Pattern | Action | Example |
|---|---|---|
| Exact sentence repeated | CUT first | "This setup exists so you can... this setup exists so you can" |
| Started, stopped, restarted | CUT false start | "This is the - okay so the way this works" |
| Standalone filler | CUT | "Here." / "Okay." / "One more." / "Success case." |
| Stumbled, incoherent | CUT | "It it will burn through a month. It will burn through a day." |
| Complete sentence | KEEP | |
| Last clean version | KEEP | Keep final take, cut all earlier attempts |

---

## Quality checklist

- [ ] No subtitle gaps visible during speech
- [ ] "Claude Code", "Claude Max", "Antigravity" spelled correctly throughout
- [ ] Subtitles stay at fixed bottom position (no jumping)
- [ ] No card text exceeds 55 chars
- [ ] Each subtitle card displays as one line
- [ ] Retakes cut cleanly with no audio pop
- [ ] Speed-up is natural, not robotic
- [ ] `output/captions.srt` written for re-editing
- [ ] All utterance IDs present in ai_decisions.json

---

## Change log

| Date | Eval score | Changes | Eval file |
|---|---|---|---|
| 2026-03-28 | 69% → 82% | Added --sample implementation, captions.srt output, WORD_FIXES re-apply in render, auto-split for long cards, script read in Step 3, utterance ID validation, subtitle correction structural guidance | temp/eval_yt-cut-edit_v2-setup_2026-03-28.md |
| 2026-03-28 | 82% → 97% | Fixed Step 3 relative path in python -c command (used absolute OUTPUT_DIR), fixed {sample_arg} ambiguous substitution → {sample_args_list} with explicit [] or ["-t","N"] guidance, removed unimplemented N_extra split instruction from Step 5 | temp/eval_yt-cut-edit_operational_2026-03-28.md |
| 2026-03-29 | -- | Replaced Remotion render (Step 6) with ffmpeg + ASS subtitles (2-3 min vs 3-4 hours). Switched cut edit encoder from libx264 to hevc_videotoolbox (hardware). Expanded Step 5 subtitle corrections with thorough double-check: all capitalization, punctuation, numbers, proper nouns, framework names, skill names. Added more WORD_FIXES patterns. | -- |
| 2026-08-19 | -- | Sanitized and republished under NiteOps brand | -- |
