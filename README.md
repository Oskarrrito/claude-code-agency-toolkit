# NiteOps Skills

Claude Code skills I run my automation business on. Every skill in this repo is one I use myself, day to day. If it's here, it ships.

## About

I'm Oskar. I run NiteOps out of Dublin. NiteOps is my automation partner brand for agencies. I ship the same MCP + Claude Code stack I use for my own outbound, delivery, and YouTube pipeline. These 28 skills are the operator manual behind that work.

## Tech stack

- **Claude Code** on **Antigravity** (Google's VS Code fork)
- **MCP servers:** Supabase (data), Apify (scraping), Perplexity (research), Stripe (payments)
- **Cold email:** Instantly (with the Instantly MCP)
- **Transactional email:** Resend
- **Web:** Next.js on Vercel
- **Video transcription:** Deepgram
- **Video render:** Remotion + ffmpeg

## The skills, grouped

### Business setup

- **business.md** Socratic interview that writes your `business.md` (vision, niche, offer, voice, competition, rules for Claude). Every other skill reads this file.
- **find-niche.md** 4 warm-up questions, network size test, scrapes Reddit + YouTube comments + search suggestions, Perplexity validates the niche (scored /20), writes 5 warm messages to send today.
- **build-offer.md** 5-decision walkthrough (delivery model, the ONE thing, deliverables, guarantee, price) then scores the offer on the Value Equation and identifies the weakest lever.
- **build-landing-page.md** Reads business.md, audits an existing page or generates a Next.js one with anti-AI design rules and humanizer copy. Deploys to Vercel.
- **build-delivery.md** Builds the full delivery folder (Agency, Coaching, Hybrid, Software, or Membership) with CLAUDE.md hierarchy and delivery-specific skills.
- **build-product.md** Same as build-delivery plus baked-in operations (onboarding emails, reporting template, testimonial request).
- **positioning-agent.md** Scrapes competitor YouTube channels, maps their avatars, finds content gaps, writes your positioning doc.
- **competitor-spy.md** Deep dive on individual competitor channels (top content, messaging, gaps, targeting recs).

### Outreach

- **build-pipeline.md** Refreshes warm network, scrapes cold leads via Apify (Google Maps + LinkedIn), activates Instantly, writes multi-channel sequences (Observation/Offer/Soft Ask), scales to 30-50 daily touches.
- **outreach-engine.md** Daily driver. Scrapes, uploads to Instantly, writes today's DMs and voice notes, tracks streak. 10 minutes, nothing sends without approval.
- **scoreboard.md** Creates 6 Supabase tables (volume, response, booking, showup, close, cash). Log in plain English, Claude writes the SQL.
- **mission-control.md** Personal admin dashboard. Reads Supabase and Instantly analytics, builds an HTML dashboard you keep open next to Claude Code.
- **close-deal.md** Sales cycle in 3 modes: prep-call (Perplexity research + 20-min timed script), sales-coach (5-dimension call review), after-close (case study, testimonial request, referral asks).
- **daily-monitor.md** Quick YouTube Ads check (2-3x per week). Pasted data, kill/scale/hold call per campaign.
- **kill-scale.md** Decision engine for YouTube ad campaigns. Rules-based kill/scale/hold with reasoning.
- **weekly-ad-report.md** Weekly YouTube Ads aggregation. WoW trends, best/worst campaign, one priority action.
- **budget-calculator.md** YouTube Ads budget split across campaign types with 3 scenarios (conservative/moderate/aggressive), CAC, LTV, break-even.
- **ad-script.md** Ad script generator (30s skippable, 15s non-skippable, 60s Shorts) pulled from your best organic content via YouTube MCP.

### YouTube pipeline

- **yt-research.md** Scrapes YouTube + X for viral content, pulls transcripts, extracts angles you can copy, drops a packaging brief.
- **yt-outline.md** Turns research into a video outline (title options, thumbnail concepts, timed script outline, visual inserts, CTA placements, recording zones).
- **yt-script.md** Writes the full script, scores it against 29 viral criteria, auto-improves failures, syncs the outline.
- **yt-cut-edit.md** Full auto edit. Deepgram transcript, cut decisions, ffmpeg cuts + speed-up, burned subtitles.
- **yt-motion.md** Inserts Remotion infographic cutaways (AppleNotes, XPost, NumbersHighlight, SpecCard, and more) into the cut video.
- **yt-broll.md** Multi-source B-roll (local library, YouTube via yt-dlp, X posts via Remotion, web scroll capture via Playwright).
- **yt-publish.md** Picks the best title from the outline, writes description + tags, uploads via YouTube MCP.
- **yt-shorts.md** Repurposes long-form into 3-5 vertical Shorts with rewritten hooks and ffmpeg extraction commands.
- **thumbnail-gen.md** Generates 5 thumbnail concepts using 7 click-psychology triggers with layer-by-layer design briefs.
- **lego-analysis.md** Breaks 5-10 competitor videos into 7 structural bricks. Shows which patterns to copy and which to make your own.

## License

MIT recommended. Fork it, run it, change it.
