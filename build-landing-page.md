---
name: build-landing-page
description: "Reads business.md, audits or generates a landing page with anti-AI design rules and humanizer copy. Two modes: audit (existing page) or generate (new page). Stacked guarantee display required."
user-invocable: true
---

# /build-landing-page

Audit an existing landing page or build a new one from business.md. Anti-AI design rules enforced. Humanizer copy rules installed. The landing page is where readers become buyers.

## Mode selection

Ask the user: "Do you have an existing landing page, or are we building from scratch?"

- **Audit mode:** They have a page. Read it, score it, give specific fixes. Jump to Step 2 (audit).
- **Generation mode:** No page exists. Build one from business.md. Follow Steps 1-7.

If they provide a URL or component path, that's audit mode. If they say "build" or "create," that's generation mode.

## Input

- Read `business.md` for: service name, one-sentence description, target client, price, guarantee, key deliverables, delivery timeline, brand voice notes
- If business.md is missing offer details, stop and tell the user to run `/build-offer` first

## Step 1: Read business.md

Extract these fields from business.md:

- Service name
- One-sentence description
- Target client (who)
- Price (number and billing cycle)
- Guarantee (if set)
- Key deliverables (list)
- Delivery timeline
- Brand voice notes (if any)
- Founding rate (if applicable, show with full price crossed out)

If any of service name, target client, price, or deliverables are missing, stop and ask the user to run `/build-offer` first. Do not proceed with incomplete data.

## Step 2: Generate page copy

Write all page copy from the extracted business.md data. Apply humanizer rules from `.claude/rules/humanizer.md` to every line of copy. No Tier 1 or Tier 2 banned words. No banned phrases. Vary sentence length.

### Headline

Under 10 words. No hype. Names the exact result for the exact audience.

Good: "15 new 5-star reviews per month for dental clinics"
Bad: "Grow your business with AI"
Bad: "Unlock the power of reviews" (banned phrase)

### Subhead

One sentence that adds specificity. Extends the headline with detail.

### Service description

2-3 short paragraphs. What you do, how you do it, why it works. Written in the founder's voice from business.md.

### Benefit sections

Three sections. Short title (3-5 words) + 2 sentences each. Focus on client outcomes, not your process.

Good: "Patients find you first" (outcome)
Bad: "Our review management process" (process)

### Pricing section

Service name, price, what's included, timeline. If business.md has a founding rate, show the full price crossed out next to the founding rate.

### Guarantee section (REQUIRED if business.md has a stacked guarantee)

If business.md has a stacked guarantee, it MUST be prominently displayed. This is the #1 conversion lever. A stacked guarantee can move the value equation from 29/40 to 34/40. Do NOT bury it in FAQ or fine print.

Display format: Two visual blocks side by side.
- Block 1 (Risk reversal): The refund guarantee
- Block 2 (Overdelivery): The safety net beyond the refund

If business.md has no guarantee, ask the user: "What can you guarantee about your deliverables? Even 'Cancel anytime, no contracts' works." They can skip and add later.

### Platform differentiator check

If business.md mentions a custom platform (not Skool, not Circle, not Teachable), call it out on the page. Every competitor rents the same Skool apartment. Building your own platform is a real differentiator. Add a line near the platform preview or pricing section: "Built from scratch. Not rented from Skool."

### Portfolio section

Only generate this section for visual or creative services (design, video, content creation). Skip for non-visual services (consulting, coaching, SEO, review management, etc.).

### CTA logic

Choose CTA based on the price in business.md:

- Under $500/month: Stripe checkout button (direct purchase). Use `NEXT_PUBLIC_STRIPE_CHECKOUT_URL` env var.
- $500 to $1,000/month: Ask the user's preference: Stripe checkout or "Book a call" with Calendly.
- Over $1,000/month: "Book a call" linking to Calendly. No Stripe checkout for high-ticket. Use `NEXT_PUBLIC_BOOKING_URL` env var.

## Step 2B: Audit mode (existing page)

If auditing an existing page, read the page source or component and run this analysis:

1. **Section-by-section audit:** For each section, describe what's there, what's working, and what's broken. Be specific.
2. **Stacked guarantee check:** Is the full stacked guarantee from business.md displayed prominently? If not, flag as critical. This is the biggest conversion lever.
3. **Platform differentiator check:** If the user built a custom platform, is it called out? "Not Skool" is a real differentiator.
4. **Stats consistency:** Cross-check every number on the page against business.md. Mismatched stats destroy credibility.
5. **Anti-AI design checklist:** Run all 9 checks from Step 3.
6. **Humanizer audit:** Scan for banned words and phrases from humanizer.md.
7. **Copy recommendations:** Specific rewrites for the top 3-5 issues found.

Output format for audit mode: Section-by-section analysis, anti-AI score (N/9), humanizer verdict, then ranked list of specific fixes with priority (critical/high/medium/low).

## Step 3: Apply anti-AI design rules

Run every design decision through this checklist. All 9 items must pass before delivering the page.

1. No purple-to-blue gradients anywhere. Use 2-3 colors max, pulled from brand or neutral defaults.
2. Layout is NOT the standard Hero > 3-cards > Testimonial > CTA formula. Break the pattern. Use asymmetric sections.
3. Font is NOT Inter or system default. Use one of: Space Grotesk, Clash Grotesk, or Satoshi for personality. Import via Google Fonts or Fontsource.
4. At least one asymmetric section. Not everything center-aligned.
5. Headline is under 10 words with zero Tier 1/Tier 2 banned AI words.
6. Copy contains specific numbers, not vague promises. Pull exact figures from business.md.
7. No stock imagery or AI-generated illustrations. If no real images exist, use clean typography and whitespace instead.
8. Texture or grain overlay at low opacity (5-15%) for depth. Subtle, not decorative.
9. Monospace font for data points and specific numbers (price, stats, timelines).

## Step 4: Build the Next.js page

Create a `site/` directory in the user's project root.

Requirements:
- Next.js with Tailwind CSS
- Responsive (phone and desktop). Test mobile layout has no horizontal scroll.
- Clean, minimal. No animations, no parallax, no hero images unless the user provides one.
- CTA link via environment variable: `NEXT_PUBLIC_STRIPE_CHECKOUT_URL` (Stripe) or `NEXT_PUBLIC_BOOKING_URL` (Calendly)
- Meta tags for social sharing: title, description, Open Graph image tag (even if no image yet)
- `.env.example` file with placeholder values for the CTA URL

If the user doesn't have a Stripe checkout link yet, use a placeholder URL and log a reminder: "Set NEXT_PUBLIC_STRIPE_CHECKOUT_URL when your Stripe checkout is ready. Don't let a missing payment link block the page."

## Step 5: Install humanizer rules

Copy the humanizer ruleset into the user's project:

1. Create `.claude/rules/` directory in the user's project if it doesn't exist
2. Copy the full humanizer.md content (4-tier banned word list, banned phrases, sentence structure patterns, formatting tells, voice and tone guidelines)
3. Confirm installation in the output

From this point forward, every piece of copy Claude writes for this project gets checked against humanizer rules automatically.

## Step 6: Deploy to Vercel

1. Initialize git repo if not already initialized
2. Push to GitHub
3. Vercel auto-deploys (or run `vercel --prod` if Vercel CLI is available)
4. Verify the live URL loads correctly
5. Test mobile layout
6. Test CTA button links to Stripe checkout or Calendly (or placeholder)
7. Return the live URL

If deployment fails, log the error and tell the user what to fix. Common issues: missing Node.js version, missing env vars, build errors.

## Step 7: Update business.md

Add the live URL to business.md so every other skill can reference it:

```
landing_page_url: https://[project].vercel.app
```

## Output

Print a summary in this format:

```
Landing page generated:
- Headline: "[exact headline text]"
- Subhead: "[exact subhead text]"
- Font: [font name]
- Layout: [brief layout description]
- Colors: [hex values for text, bg, accent]
- Grain overlay: [opacity]% [texture type]
- CTA: [Stripe/Calendly/Placeholder] via [env var name]
- Deployed: [live URL]

Humanizer rules installed:
- .claude/rules/humanizer.md (copied)
- 4-tier banned word list active
- Sentence structure checks enabled
- All future copy generation passes through humanizer

Anti-AI design checks: [N]/9 passed
[List any failures with fix instructions]
```

## Error handling

- business.md missing or empty: Stop. Tell the user to run `/build-offer` first.
- business.md missing price: Ask for a price before proceeding.
- business.md has stacked guarantee but page doesn't display it: Flag as critical. This is the #1 conversion lever.
- business.md missing guarantee: Ask once. If they skip, proceed without guarantee section.
- Vercel deployment fails: Log the error. Provide the fix. The page still exists locally.
- No Stripe link: Use placeholder. Remind them to set it up. Do not block the page.
- Font import fails: Fall back to Space Grotesk from Google Fonts. Never fall back to Inter.

## Self-check (run before delivering)

1. Headline is under 10 words
2. Zero Tier 1 banned words in any copy (delve, tapestry, landscape, leverage, robust, etc.)
3. Zero Tier 2 banned words in any copy (elevate, navigate, harness, unlock, foster, etc.)
4. Zero banned phrases ("In today's...", "game-changer", "best-in-class", etc.)
5. All 9 anti-AI design checks pass
6. Stacked guarantee from business.md is prominently displayed (if it exists)
7. Platform differentiator called out (if custom platform, not Skool)
8. CTA environment variable is set (even if placeholder)
9. Mobile layout has no horizontal scroll
10. business.md updated with live URL
11. Humanizer rules file exists in user's `.claude/rules/`
12. Font is NOT Inter or system default

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-30 | Initial version. Next.js landing page generator with anti-AI design and humanizer rules. Added: audit mode vs generation mode, stacked guarantee display REQUIRED, platform differentiator check ("not Skool"), stats consistency cross-check. |
