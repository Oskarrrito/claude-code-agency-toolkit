---
name: yt-broll
description: Finds multi-source B-roll (YouTube video, X/Twitter posts via Remotion XPost, web scroll-capture via Playwright) and inserts as cutaway clips. Claude identifies 3-5 moments from script/subtitles, fetches content, and assembles final_broll.mp4. Run after /yt-motion or /yt-cut-edit.
argument-hint: "[folder-name]  e.g. v2-setup"
---

# /yt-broll

Identifies moments where real visual B-roll adds context. Fetches from multiple sources and inserts as full-screen cutaways. Audio from the talking head continues throughout.

**Source types and rendering methods:**
- `local` clip from the local `videoasset/` library (me working, events, meetings, cinematic, testimonials, screen recordings). **Always check local assets first** before searching externally.
- `youtube` - video clip from YouTube, downloaded via yt-dlp, 5% zoom-in crop
- `twitter` - rendered via Remotion `XPost` component (realistic X dark mode UI with animated like count). NEVER screenshot X.com - it shows loading spinners or Cloudflare blocks.
- `web` - scroll video capture via Playwright (120 frames, 30fps, smoothstep easing). NEVER take static screenshots - always produce scrolling video.

---

## Step 1: Load context

1. Parse argument: folder name (e.g., `v2-setup`)

2. Set absolute paths:
   ```
   PROJECT_DIR  = {absolute_path}/content/youtube/{folder_name}/
   OUTPUT_DIR   = {absolute_path}/content/youtube/{folder_name}/output/
   ENGINE_DIR   = {absolute_path}/content/youtube/_engine/
   BASE_VIDEO   = OUTPUT_DIR/final_motion.mp4
   ASSET_DIR    = {absolute_path}/videoasset/
   BROLL_DIR    = OUTPUT_DIR/broll/
   PLAN_FILE    = OUTPUT_DIR/broll_plan.json
   FINAL_OUT    = OUTPUT_DIR/final_broll.mp4
   ```
   Derive `{absolute_path}` as the project root. Verify by checking that `{absolute_path}/content/youtube/` exists. If it doesn't, walk up from the current working directory until you find the directory containing `content/youtube/`, or fail with a clear message.
   If `final_motion.mp4` doesn't exist, try `final.mp4`, then `cut_edit.mp4`.

3. Verify BASE_VIDEO exists. Fail with clear message if not found.

4. Create broll directory:
   ```bash
   mkdir -p "{OUTPUT_DIR}/broll"
   ```

---

## Step 2: Identify B-roll moments

**This is you (Claude Code) working directly - no API call.**

1. Read `output/subtitle_cards_raw.json` (for timestamps). **If `subtitle_cards_raw.json` doesn't exist**, fall back to `03_SCRIPT.md` and estimate timestamps at ~150 words per minute. If neither file exists, fail with a clear message: "Cannot plan B-roll: no subtitle_cards_raw.json or 03_SCRIPT.md found in {PROJECT_DIR}."
2. Read `03_SCRIPT.md` if it exists
3. **Read `videoasset/ASSET_INDEX.md`** to know what local clips are available

Look for moments where B-roll would add visual proof or context. **Always check local assets first** before searching externally. For each moment, pick the **most authentic source**:

| Content type | source_type | Rendering method |
|---|---|---|
| Me working/typing/coding | `local` | Trim from `videoasset/03_Broll/working/` |
| Me at event/stage/networking | `local` | Trim from `videoasset/03_Broll/events/` |
| Meeting/group/teaching scene | `local` | Trim from `videoasset/03_Broll/meetings/` |
| Cinematic/contemplative shot | `local` | Trim from `videoasset/03_Broll/cinematic/` |
| Screen recording/demo | `local` | Trim from `videoasset/02_Screen_Recordings/` |
| Testimonial/social proof | `local` | Trim from `videoasset/04_Social_Proof/` |
| Stripe dashboard, real app UI | `local` | Trim from `videoasset/04_Social_Proof/screenshots/` (convert to clip) |
| Tool/product demo (no local match) | `youtube` | yt-dlp download + 5% zoom crop |
| Named person speaking on video | `youtube` | yt-dlp download + 5% zoom crop |
| Specific tweet/X post | `twitter` | Remotion `XPost` component (dark mode UI) |
| Website/product page | `web` | Playwright scroll video capture |
| SaaS product UI | `web` | Playwright scroll video capture |

**For each moment, specify:**
- `source_type`: `local`, `youtube`, `twitter`, or `web`
- `timestamp`: use the `end` time of a subtitle card (never mid-sentence)
- `duration`: 4-5 seconds
- `description`: exactly what should be shown
- For `local`: `asset_path` (relative path within `videoasset/`, e.g. `03_Broll/working/work-desk-coding-monitor-mic.mp4`) and `start_offset` (seconds, default 0)
- For `youtube`: `search_query` for YouTube MCP search, and `start_offset` (seconds into the source video to begin the clip, default 0)
- For `twitter`: `tweet_props` object (see XPost format below)
- For `web`: `url` (the actual URL to scroll-capture)

**XPost tweet_props format:**
```json
{
  "handle": "@username",           // REQUIRED
  "displayName": "Display Name",   // REQUIRED
  "text": "The tweet content text", // REQUIRED
  "likes": 6426,                   // REQUIRED
  "reposts": 1823,                 // REQUIRED - omitting breaks the XPost layout
  "bookmarks": 10211,              // REQUIRED
  "timestamp": "Mar 28, 2026",    // REQUIRED - omitting breaks the XPost layout
  "verified": false,               // optional (default false)
  "avatarColor": "#1D9BF0"        // optional (default #1D9BF0)
}
```
All seven fields (handle, displayName, text, likes, reposts, bookmarks, timestamp) are **REQUIRED**. `reposts` and `timestamp` are the most commonly forgotten and will cause rendering errors if omitted.

**Rules:**
- 3-5 moments max - quality over quantity
- Timestamps must be at subtitle card end boundaries
- For `twitter`: find a real tweet via Apify. Extract the actual text, author, and engagement numbers. Put them in `tweet_props`.
- For `web`: pick URLs that won't block headless browsers (avoid claude.ai, x.com - use anthropic.com instead of claude.ai, etc.)
- For `youtube`: be specific about what should be on screen

Write `output/broll_plan.json`:
```json
[
  {
    "id": "founder-coding",
    "source_type": "local",
    "timestamp": 8.5,
    "duration": 4,
    "start_offset": 5,
    "description": "Founder coding at desk with monitor and mic",
    "asset_path": "03_Broll/working/work-desk-coding-monitor-mic.mp4",
    "clip_path": null
  },
  {
    "id": "claude-code-demo",
    "source_type": "youtube",
    "timestamp": 22.0,
    "duration": 4,
    "start_offset": 0,
    "description": "Claude Code running in a terminal",
    "search_query": "Claude Code terminal demo 2024",
    "url": null,
    "clip_path": null
  },
  {
    "id": "fomo-tweet",
    "source_type": "twitter",
    "timestamp": 40.0,
    "duration": 5,
    "description": "Tweet about Claude Code hype",
    "tweet_props": {
      "handle": "@username",
      "displayName": "Name",
      "text": "Tweet text here",
      "likes": 5000,
      "reposts": 1200,
      "bookmarks": 8000,
      "timestamp": "Mar 28, 2026",
      "verified": false,
      "avatarColor": "#1D9BF0"
    },
    "clip_path": null
  },
  {
    "id": "product-page",
    "source_type": "web",
    "timestamp": 60.0,
    "duration": 4,
    "description": "The Anthropic website",
    "url": "https://www.anthropic.com",
    "clip_path": null
  }
]
```

Print each planned cutaway: `[{i}] {timestamp:.1f}s - {source_type}: "{description}"`

---

## Step 3: Fetch content by source type

### 3a: Local asset sources (`source_type == "local"`)

No fetching needed. The `asset_path` in the plan points to a file inside `videoasset/`. Verify the file exists. No API calls, no downloads.

The `download_broll.py` script handles trimming: `ffmpeg -ss {start_offset} -i {asset_path} -t {duration} -vf scale=1920:1080:... clip_path`

**Asset library quick reference** (from `videoasset/ASSET_INDEX.md`):

| I need... | Look in |
|---|---|
| Me typing/coding at desk | `03_Broll/working/work-desk-coding-monitor-mic.mp4` or `work-hoodie.mp4` |
| Me at event/on stage | `03_Broll/events/event-stage-keynote.mp4` |
| Meeting/teaching scene | `03_Broll/meetings/meeting-group-cafe-presenting.mp4` |
| Cinematic/contemplative | `03_Broll/cinematic/broll-suit-window-contemplative.mp4` |
| Screen recording demos | `02_Screen_Recordings/screen-vibe-coding-*.mp4` |
| Testimonial clips | `04_Social_Proof/testimonials/` |
| Code review pair programming | `03_Broll/meetings/broll-code-review-large-screen.mp4` |

### 3b: YouTube sources (`source_type == "youtube"`)

1. Call `mcp__youtube__youtube_search` with `search_query`
2. Review top 3-5 results. Pick best match:
   - Relevance to description
   - View count (higher = more credible)
   - Duration (prefer 1-10 minute videos)
   - Official/known channels preferred
3. Set `url` = `https://youtu.be/{video_id}` in the plan

### 3c: Twitter/X sources (`source_type == "twitter"`)

1. Search Apify for a Twitter scraper:
   ```
   mcp__apify__search-actors with query "twitter scraper"
   ```
2. Call `mcp__apify__fetch-actor-details` to get the input schema.
3. Run the actor to search for relevant tweets. Pick the best one (high engagement, credible account).
4. Extract the actual tweet data: author name, handle, text, likes, reposts, bookmarks.
5. Fill in `tweet_props` in broll_plan.json with real data.

**The `download_broll.py` script renders twitter items via Remotion XPost - no screenshot needed.**

**IMPORTANT:** The Remotion composition MUST be `"XPost"` (not `"TweetCard"` or any other name). The `tweet_props` object MUST include `reposts` and `timestamp` fields or the render will fail.

### 3d: Web sources (`source_type == "web"`)

The `url` is already specified in the plan.

**Avoid URLs that block headless browsers:**
- `claude.ai` → use `www.anthropic.com/claude` instead
- `x.com` → use Remotion XPost instead
- Any site with aggressive Cloudflare/bot protection

**The `download_broll.py` script uses Playwright scroll capture - no static screenshots.**

---

## Step 4: Write and run download_broll.py

Write `output/download_broll.py`, substituting `{output_dir}` and `{engine_dir}` with absolute paths:

```python
#!/usr/bin/env python3
"""Fetch B-roll clips from multiple sources: YouTube, Twitter (Remotion), Web (scroll capture)."""
import json, subprocess, os, time, shutil

OUTPUT_DIR = "{output_dir}"
BROLL_DIR  = OUTPUT_DIR + "/broll"
PLAN_FILE  = OUTPUT_DIR + "/broll_plan.json"
ENGINE_DIR = "{engine_dir}"
ASSET_DIR  = "{absolute_path}/videoasset"
CACHE_DIR  = "{absolute_path}/content/youtube/_broll_cache"

os.makedirs(CACHE_DIR, exist_ok=True)

with open(PLAN_FILE) as f:
    plan = json.load(f)

def video_capture_scroll(url, clip_path, duration, fps=30):
    """Capture a scrolling video of a web page via Playwright."""
    try:
        from playwright.sync_api import sync_playwright
        import tempfile, shutil

        frames_dir = tempfile.mkdtemp(prefix="broll_frames_")
        n_frames = int(duration * fps)

        with sync_playwright() as p:
            browser = p.chromium.launch()
            page = browser.new_page(viewport={"width": 1280, "height": 720})
            page.goto(url, wait_until="domcontentloaded", timeout=30000)
            time.sleep(2)

            max_scroll = page.evaluate("document.body.scrollHeight - window.innerHeight")
            target_scroll = max_scroll * 0.30  # slow scroll - 30% of page

            for i in range(n_frames):
                t = i / n_frames
                eased = t * t * (3 - 2 * t)  # smoothstep
                scroll_y = int(target_scroll * eased)
                page.evaluate(f"window.scrollTo(0, {scroll_y})")
                page.screenshot(path=f"{frames_dir}/{i:04d}.png", full_page=False)

            browser.close()

        r = subprocess.run([
            "ffmpeg", "-y",
            "-framerate", str(fps),
            "-i", f"{frames_dir}/%04d.png",
            "-vf", "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black",
            "-c:v", "libx264", "-preset", "fast", "-crf", "18",
            "-pix_fmt", "yuv420p", clip_path
        ], capture_output=True, text=True)

        shutil.rmtree(frames_dir)
        return r.returncode == 0
    except Exception as e:
        print(f"  Scroll capture error: {e}")
        return False


def extract_video_id(url):
    """Extract YouTube video ID from various URL formats."""
    if "youtu.be/" in url:
        return url.split("youtu.be/")[1].split("?")[0]
    if "v=" in url:
        return url.split("v=")[1].split("&")[0]
    return None


for i, item in enumerate(plan):
    src = item.get('source_type', 'youtube')
    clip_path = f"{BROLL_DIR}/{i:02d}_{item['id']}.mp4"
    duration = item['duration']
    start_offset = item.get('start_offset', 0)

    # Skip if clip already exists and is valid (local cache)
    if os.path.exists(clip_path) and os.path.getsize(clip_path) > 50000:
        item['clip_path'] = clip_path
        print(f"[{i}] Cached: {clip_path}")
        continue

    if src == 'local':
        # Local asset: trim from videoasset/ library
        asset_path = item.get('asset_path')
        if not asset_path:
            print(f"[{i}] Skipping local - no asset_path set")
            continue
        full_asset = os.path.join(ASSET_DIR, asset_path)
        if not os.path.exists(full_asset):
            print(f"[{i}] Local asset not found: {full_asset}")
            continue
        print(f"[{i}] Local: trimming {asset_path} from {start_offset}s...")
        r = subprocess.run([
            "ffmpeg", "-y", "-ss", str(start_offset), "-i", full_asset, "-t", str(duration),
            "-vf", (
                "scale=1920:1080:force_original_aspect_ratio=decrease,"
                "pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black"
            ),
            "-c:v", "libx264", "-preset", "fast", "-crf", "18",
            "-an", clip_path
        ], capture_output=True, text=True)
        if r.returncode != 0:
            print(f"  ffmpeg error: {r.stderr[-300:]}")
            continue

    elif src == 'youtube':
        if not item.get('url'):
            print(f"[{i}] Skipping youtube - no url set")
            continue

        # Check cross-project B-roll cache by video_id
        video_id = extract_video_id(item['url'])
        cache_file = os.path.join(CACHE_DIR, f"{video_id}_{start_offset}s_{duration}s.mp4") if video_id else None
        if cache_file and os.path.exists(cache_file) and os.path.getsize(cache_file) > 50000:
            shutil.copy2(cache_file, clip_path)
            item['clip_path'] = clip_path
            print(f"[{i}] Cache hit (cross-project): {cache_file}")
            continue

        raw_path = f"{BROLL_DIR}/{i:02d}_{item['id']}_raw.mp4"
        print(f"[{i}] YouTube: downloading {item['url']}...")
        r = subprocess.run([
            "yt-dlp",
            "-f", "bestvideo[height<=1080][ext=mp4]+bestaudio/best[height<=1080]",
            "--merge-output-format", "mp4", "--no-playlist",
            "-o", raw_path, item['url']
        ], capture_output=True, text=True)
        if r.returncode != 0 or not os.path.exists(raw_path):
            print(f"  Download failed: {r.stderr[-300:]}")
            continue
        r = subprocess.run([
            "ffmpeg", "-y", "-ss", str(start_offset), "-i", raw_path, "-t", str(duration),
            "-vf", (
                "scale=iw*1.05:ih*1.05,"
                "crop=1920:1080:(iw-1920)/2:(ih-1080)/2,"
                "scale=1920:1080"
            ),
            "-c:v", "libx264", "-preset", "fast", "-crf", "18",
            "-c:a", "aac", "-b:a", "192k", clip_path
        ], capture_output=True, text=True)
        if r.returncode != 0:
            print(f"  ffmpeg error: {r.stderr[-300:]}")
            continue
        os.remove(raw_path)

        # Copy to cross-project cache
        if cache_file:
            shutil.copy2(clip_path, cache_file)
            print(f"  Cached to: {cache_file}")

    elif src == 'twitter':
        # Render via Remotion XPost component - realistic X dark mode UI
        tweet_props = item.get('tweet_props')
        if not tweet_props:
            print(f"[{i}] Twitter: no tweet_props - skipping")
            continue
        print(f"[{i}] Twitter: rendering XPost infographic...")
        props_json = json.dumps(tweet_props)
        frames = int(duration * 30) - 1
        r = subprocess.run([
            "npx", "remotion", "render", "src/index.ts", "XPost", clip_path,
            f"--props={props_json}",
            f"--frames=0-{frames}", "--codec=h264"
        ], capture_output=True, text=True, cwd=ENGINE_DIR)
        if r.returncode != 0:
            print(f"  Remotion render error: {r.stderr[-500:]}")
            continue

    elif src == 'web':
        # Always scroll-capture - never static screenshot
        if not item.get('url'):
            print(f"[{i}] Skipping web - no url set")
            continue
        print(f"[{i}] Web: scroll-capturing {item['url']}...")
        if not video_capture_scroll(item['url'], clip_path, duration):
            print(f"  Scroll capture failed")
            continue

    else:
        print(f"[{i}] Unknown source_type: {src}")
        continue

    item['clip_path'] = clip_path
    print(f"  ✓ {clip_path}")

# Save updated plan
with open(PLAN_FILE, "w") as f:
    json.dump(plan, f, indent=2)

downloaded = sum(1 for c in plan if c.get('clip_path') and os.path.exists(c['clip_path']))
print(f"\n{downloaded}/{len(plan)} clips ready")
```

Run it:
```bash
python3 output/download_broll.py
```

**If Playwright is not installed:**
```bash
pip3 install playwright && python3 -m playwright install chromium
```

---

## Step 5: Assemble cutaways into final video

Write `output/assemble_broll.py`, substituting `{base_video}`, `{plan_file}`, `{output}` with absolute paths.

**IMPORTANT:** Output file is `final_broll.mp4` (not `final.mp4`) to avoid overwriting previous pipeline outputs.

```python
#!/usr/bin/env python3
"""Insert B-roll cutaways into base video (video replacement, audio continuous)."""
import json, subprocess, os, sys

BASE_VIDEO = "{base_video}"
PLAN_FILE  = "{plan_file}"
OUTPUT     = "{output}"

with open(PLAN_FILE) as f:
    plan = json.load(f)

plan = sorted(
    [c for c in plan if c.get('clip_path') and os.path.exists(c['clip_path'])],
    key=lambda x: x['timestamp']
)

if not plan:
    print("No B-roll clips available - nothing to insert")
    import shutil; shutil.copy2(BASE_VIDEO, OUTPUT)
    print(f"✓  Copied {BASE_VIDEO} → {OUTPUT}")
    sys.exit(0)

dur_str = subprocess.run(
    ["ffprobe","-v","quiet","-show_entries","format=duration","-of","csv=p=0", BASE_VIDEO],
    capture_output=True, text=True
).stdout.strip()
total_dur = float(dur_str)

inputs = ["-i", BASE_VIDEO]
for c in plan:
    inputs += ["-i", c['clip_path']]

filter_parts = []
vsegs = []
cursor = 0.0

for i, c in enumerate(plan):
    t0 = c['timestamp']
    t1 = min(t0 + c['duration'], total_dur)
    cidx = i + 1

    if t0 > cursor + 0.02:
        tag = f"tv{i}"
        filter_parts.append(f"[0:v]trim={cursor:.4f}:{t0:.4f},setpts=PTS-STARTPTS[{tag}]")
        vsegs.append(f"[{tag}]")

    tag = f"cv{i}"
    filter_parts.append(
        f"[{cidx}:v]scale=1920:1080:force_original_aspect_ratio=decrease,"
        f"pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black,"
        f"trim=0:{c['duration']:.4f},setpts=PTS-STARTPTS[{tag}]"
    )
    vsegs.append(f"[{tag}]")
    cursor = t1

if cursor < total_dur - 0.02:
    filter_parts.append(f"[0:v]trim={cursor:.4f}:{total_dur:.4f},setpts=PTS-STARTPTS[tv_end]")
    vsegs.append("[tv_end]")

n = len(vsegs)
filter_parts.append("".join(vsegs) + f"concat=n={n}:v=1:a=0[vout]")
filter_str = ";\n".join(filter_parts)

ff_file = OUTPUT.replace(".mp4", "_broll_filter.txt")
with open(ff_file, "w") as f:
    f.write(filter_str)

cmd = ["ffmpeg", "-y"] + inputs + [
    "-filter_complex_script", ff_file,
    "-map", "[vout]", "-map", "0:a",
    "-c:v", "libx264", "-preset", "fast", "-crf", "18",
    "-c:a", "copy", OUTPUT
]

print(f"Assembling {len(plan)} B-roll cutaways...")
r = subprocess.run(cmd, capture_output=True, text=True)
if r.returncode != 0:
    print("ERROR:", r.stderr[-3000:]); sys.exit(1)

os.remove(ff_file)

result_dur = subprocess.run(
    ["ffprobe","-v","quiet","-show_entries","format=duration","-of","csv=p=0", OUTPUT],
    capture_output=True, text=True
).stdout.strip()
output_dur = float(result_dur)

# Duration assertion: warn if output differs from input by more than 0.5s
dur_diff = abs(output_dur - total_dur)
if dur_diff > 0.5:
    print(f"WARNING: Output duration ({output_dur:.2f}s) differs from input ({total_dur:.2f}s) by {dur_diff:.2f}s (threshold: 0.5s)")

print(f"✓  {OUTPUT}  ({output_dur:.1f}s)")
```

Run it:
```bash
python3 output/assemble_broll.py
```

---

## Step 6: Report

```
═══════════════════════════════════════
  YT-BROLL COMPLETE
═══════════════════════════════════════
  Project:     {folder_name}
  Cutaways:    {N} B-roll clips inserted
  Output:      output/final_broll.mp4  ({duration:.0f}s)

  Clips used:
  {for each: timestamp - source_type - description}
═══════════════════════════════════════
```

---

## Pipeline order

```
/yt-cut-edit {folder}   →  cut_edit.mp4 + final.mp4 (cut + subtitles)
/yt-motion {folder}     →  final_motion.mp4          (+ Remotion infographics)
/yt-broll {folder}      →  final_broll.mp4           (+ B-roll cutaways)
```

---

## Available Remotion UI templates for B-roll

These are registered in `_engine/src/Root.tsx` and render at 1920x1080:

| Template | Use for | Key props |
|---|---|---|
| `XPost` | Twitter/X posts | handle, displayName, text, likes, reposts, bookmarks, timestamp, verified, avatarColor |
| `AppleNotes` | Checklist comparisons | title, sections[{heading, items[{text, checked}]}] |

**All UI templates fill the full 1920x1080 canvas.** No small centered windows.

Future templates to add as needed: `Terminal`, `GoogleSheets`, `Stripe`, `VSCode`.

---

## Quality checklist

- [ ] 3-5 moments identified with correct source_type per item
- [ ] YouTube items: YouTube MCP search run, video_id selected, url set
- [ ] Twitter items: Apify search run, tweet data extracted, tweet_props filled
- [ ] Twitter tweet_props includes ALL required fields (handle, displayName, text, likes, reposts, bookmarks, timestamp)
- [ ] Web items: URL confirmed accessible (no Cloudflare-blocked sites)
- [ ] Twitter rendered via Remotion XPost (NOT screenshotted)
- [ ] Web rendered as scroll video (NOT static screenshot)
- [ ] All clips generated and cached in broll/ folder
- [ ] assemble_broll.py ran without errors
- [ ] Output duration matches input duration (±0.5s)

---

## Change log

| Date | Changes |
|---|---|
| 2026-03-28 | v2: Multi-source support - youtube (yt-dlp), twitter (Apify + Playwright screenshot), web (Playwright screenshot) |
| 2026-03-29 | v3: Twitter renders via Remotion XPost (no more X.com screenshots). Web uses scroll video capture (no static screenshots). Added clip caching. Output renamed to final_broll.mp4. Full-screen UI templates. |
| 2026-03-29 | v4: XPost tweet_props required fields. YouTube start_offset. Cross-project _broll_cache/. Duration assertion. |
| 2026-03-29 | v5: Added `local` source type. Reads from `videoasset/` asset library (B-roll, screen recordings, testimonials). Local assets checked first before external search. ASSET_INDEX.md quick reference in Step 3a. download_broll.py handles local trim via ffmpeg. |
| 2026-08-19 | Sanitized and republished under NiteOps brand |
