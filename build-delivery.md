---
name: build-delivery
description: "Builds your complete delivery system in Claude Code. Reads delivery model from business.md, branches into 5 paths (Agency, Coaching, Hybrid, Software, Membership), creates project folders, CLAUDE.md hierarchy, templates, delivery-specific skills, onboarding emails, reporting template, and testimonial request."
user-invocable: true
---

# Build your delivery product

**Purpose:** Turn your offer into a real delivery system. Not a document describing what you'll do. The actual folder structure, templates, CLAUDE.md files, and custom skills you use to deliver. This is where "learning" turns into "shipping."

## Input

- Read business.md for: delivery model, niche, offer, price, guarantee
- $ARGUMENTS: none expected
- No MCP servers required. This skill creates local files and folders.

**Error handling:** If business.md is missing or doesn't have a delivery model, tell the user: "I need your delivery model in business.md first. Run /build-offer to define your offer and delivery model, then come back here." Stop.

## Step 1: Load context

1. Read `business.md` (required: delivery model, niche, offer details)
2. Read `.claude/rules/humanizer.md` for writing rules
3. Identify the delivery model from business.md. It should be one of: Agency, Coaching, Hybrid, Software, Membership.
4. If the delivery model isn't clear, ask: "What's your delivery model? Are you doing the work for clients (agency), teaching them (coaching), a mix of both (hybrid), building a tool (software), or running a membership (coaching + community + software)?"

## Step 2: Conversation to understand specifics

Before building anything, have a short conversation (3-5 questions) to understand the details. The questions depend on the delivery model:

**For Agency:**
- "What type of agency are you running?" (options: web development, marketing/content, automation, data/analytics, design, niche vertical)
- "What does a typical deliverable look like?"
- "What tools do you use to deliver?"
- If niche vertical: "What industry?" (restaurant, dental, real estate, med spa, e-commerce, or other)

**For Coaching:**
- "Group or 1:1?"
- "How many weeks is your program?"
- "What transformation do you promise by the end?"

**For Hybrid:**
- "Which two models are you combining?" (Agency + Coaching, Agency + Software, Coaching + Software)
- "Which parts fall under each model?"

**For Membership:**
- "What's the core transformation? What does a member look like before and after?"
- "How many phases or stages does the membership have?"
- "What's the mix? Coaching calls, community, software/tools, content library?"
- "Do you have a custom platform or are you using Skool/Circle/Teachable?" (Note: building a custom platform vs renting Skool is a positioning choice. If you're building custom, that's a differentiator worth calling out.)
- "Do you have milestones for members? (e.g., levels, titles, certifications)" If not, suggest creating them. Clear progression keeps members engaged and gives them something to aim for beyond the content.

**For Software:**
- Display the warning first: "Most micro-SaaS products earn under $500/month. Median time to $1K MRR is 12-18 months. Successful solo software founders usually have years of domain experience before building. This path is not recommended as a starting point. Are you sure? If you have fewer than 5 agency clients in this niche, go Agency first."
- If they continue: "What are your core features? Maximum 3 for MVP."
- "What tech stack?" (default: Next.js + Supabase + Vercel)

## Step 3: Build the folder structure

Create a `delivery/` folder in your project. The structure depends on the path.

### Agency path

Determine the agency sub-type from Step 2 and create the matching structure:

```
delivery/
  CLAUDE.md                    # Agency operating rules
  _templates/
    onboarding/
      intake_form.md
      welcome_email.md
      kickoff_checklist.md
    proposals/
    contracts/
    reports/
      weekly_report.md
      monthly_report.md
  _shared/
    brand_guidelines/
    sops/
  clients/
    [example-client]/
      CLAUDE.md                # Client context (auto-populated from intake)
      _admin/
      _brand/
        style_guide.md
        logos/
        colors.md
        voice_guide.md
      projects/
        [example-project]/
          CLAUDE.md            # Project-specific instructions
          brief.md
          src/
          deliverables/
          feedback/
      retainer/
        [YYYY-MM]/
  _archive/
```

Generate agency sub-type specific skills based on the conversation:

| Agency type | Skills to generate |
|---|---|
| Web development | /client-site-scaffold, /responsive-check, /seo-audit |
| Marketing/content | /content-brief, /write-blog, /repurpose, /content-calendar |
| Automation | /audit-workflow, /build-chatbot, /automation-test |
| Data/analytics | /dashboard-build, /weekly-briefing, /data-clean |
| Design (code-adjacent) | /design-to-code, /style-guide-generate |
| Niche vertical | Industry-specific skills (see below) |

**Niche vertical skills by industry:**
- Restaurant: /respond-reviews, /social-posts, /monthly-metrics
- Dental: /respond-reviews, /patient-content, /gmb-recommendations (Claude writes optimization recommendations; you apply them in your Google Business Profile dashboard)
- Real estate: /listing-description, /market-report, /social-content
- Med spa: /treatment-content, /respond-reviews, /promo-campaign
- E-commerce: /product-description, /email-campaign, /competitor-watch

### Coaching path

```
delivery/
  curriculum/
    program-v1/
      week-01/
        lesson.md
        worksheet.md
        resources/
      week-02/
      ...
    templates/
      session-agenda.md
      intake-form.md
      goal-setting-worksheet.md
      progress-report-template.md
      testimonial-request.md
    frameworks/
  clients/
    active/
      [example-client]/
        intake.md
        sessions/
          YYYY-MM-DD.md
        progress/
        deliverables/
        correspondence/
    completed/
    prospects/
  content/
    social/
    email/
    lead-magnets/
    testimonials/
      raw/
      formatted/
  operations/
    onboarding/
    scheduling/
    financials/
    sops/
```

Generate coaching-specific skills:
- `/session-prep [client-name]`: reads intake form, previous session notes, action items. Generates prep brief.
- `/session-followup [client-name]`: takes bullet point notes, generates polished follow-up email with summary, action items, deadlines.
- `/progress-report [client-name]`: compares intake goals to current status. Doubles as testimonial and case study material.
- `/coaching-content`: mines session notes and client questions for content ideas (LinkedIn posts, newsletter topics, lead magnets).

### Hybrid path

```
delivery/
  agency/
    clients/
    templates/
    deliverables/
  coaching/
    curriculum/
    clients/
    sessions/
  shared/
    brand/
    sops/
    case-studies/
```

Generate skills from both component models. The agency side gets the agency skills matching the sub-type. The coaching side gets the coaching skills. Both share the case-studies folder.

### Membership path

```
delivery/
  CLAUDE.md                          # Program operating rules
  coaching/
    curriculum/
      phase-1/
        week-01/
          lesson.md
          worksheet.md
          resources/
        week-02/
        ...
    templates/
      session-agenda.md
      intake-form.md
      goal-setting-worksheet.md
      progress-report-template.md
      testimonial-request.md
    frameworks/
  software/
    skills/
      phase-1/                       # Skills for each phase
      phase-2/
      phase-3/
      always-available/
    skill-templates/
      SKILL-TEMPLATE.md
  shared/
    brand/
      voice-guide.md
      visual-identity.md
    sops/
      onboarding-flow.md
      call-hosting.md
      community-moderation.md
      skill-release-process.md
    case-studies/
  community/
    welcome-sequence/
    discussion-prompts/
    weekly-challenges/
  operations/
    scheduling/
    financials/
    member-tracking/
```

Generate membership-specific skills:
- `/session-prep`: reads member intake, previous call notes. Generates prep brief.
- `/session-followup`: takes bullet point notes, generates follow-up email with summary and action items.
- `/progress-report [member-name]`: compares intake goals to current status. Doubles as testimonial material.
- `/community-content`: mines call notes and member questions for content ideas.

**Member journey map:** Create a narrative document (`delivery/member-journey.md`) that walks through the full member experience from signup to completion. Include:
1. Before joining (what pattern are they escaping?)
2. Week 1 (the first actions the system forces)
3. Week 2-3 (pipeline, first money, first real progress)
4. Month 2+ (expansion, content, scale)
5. Ongoing (community, cadence, milestones)

**Milestone progression:** If business.md defines revenue milestones (first client, $5K/mo, $10K/mo, $30K MRR), map them to the member journey. Each milestone triggers a notification and a short congrats message. Include this in the CLAUDE.md and member journey doc.

### Software path

```
delivery/
  CLAUDE.md
  src/
    app/
    components/
    lib/
      db/
      api/
      auth/
  public/
  docs/
    spec.md
    roadmap.md
  tests/
```

Walk through: core features (max 3 for MVP), tech stack (default Next.js + Supabase + Vercel), initial user flow. Scaffold the project and build the first feature.

## Step 4: Create the CLAUDE.md hierarchy

This is the critical piece. Create CLAUDE.md files at each level:

**Agency-level CLAUDE.md (delivery/CLAUDE.md):**
- Agency name and mission
- Quality standards and code linting rules
- Default tech stack
- Naming conventions
- SOPs reference

**Client-level CLAUDE.md (delivery/clients/[name]/CLAUDE.md):**
- Client name, industry, target audience
- Brand colors, fonts, voice
- Contact info and communication preferences
- Active projects list
- Key dates and deadlines

**Project-level CLAUDE.md (delivery/clients/[name]/projects/[project]/CLAUDE.md):**
- Project scope and deliverables
- Tech stack for this project
- Design references
- Deadline and milestones
- Review/feedback process

Explain: "When you open a client folder in Claude Code, it already knows who they are, what they want, and how they communicate. No re-explaining. No copy-pasting brand guidelines."

## Step 5: Generate delivery-specific skills

Each generated skill should be a SKILL.md file saved to `delivery/_skills/` (or `.claude/skills/` if you prefer). Each skill file should include:

- YAML frontmatter (name, description)
- Clear input (what data it reads)
- Step-by-step instructions for Claude
- Output format
- Error handling

The skills are specific to your agency type, coaching structure, or hybrid combo. Not generic. A dental review agency gets /respond-reviews with dental-specific templates. A web dev agency gets /client-site-scaffold with the default tech stack.

## Step 6: Populate example content

Create one example client folder with realistic sample data so you can test the delivery flow:

- Example client CLAUDE.md filled with plausible data for the niche
- Example project brief
- Example deliverable (if applicable)

Tell the user: "Run through the full delivery flow as if this were a real client. Test your skills. See what works and what needs adjustment."

## Step 7: Update business.md

Append to business.md:

```
=== PRODUCT BUILT ===
Date: [YYYY-MM-DD]
Delivery model: [Agency/Coaching/Hybrid/Software]
Sub-type: [if agency, which kind]
Product folder: delivery/
Active skills: [list of generated skill names]
```

## Operations setup (built-in)

After the delivery folder is built, create an `operations/` folder with basic client operations. No separate skill needed.

### 3 onboarding emails

Generate three emails in `operations/templates/emails/`. Apply humanizer rules. Short, direct, zero fluff.

1. **welcome.md** (sent after payment): Welcome them, tell them what happens in the next 48 hours, ask them to fill out the intake form, mention the guarantee from business.md, include your contact info.
2. **intake.md** (sent day 1): Intake questions directly in the email (no external form needed). Questions match the delivery model: agency gets brand assets, credentials, project goals; coaching gets current situation, goal, obstacles, available time; hybrid combines both.
3. **kickoff.md** (sent after intake returned): Confirm receipt, outline week 1 plan, set expectations for first deliverable, specify when they'll hear from you next.

### Reporting template

Create `operations/templates/reports/` with a weekly or monthly report template based on the delivery model:
- Agency: weekly status (Fridays) + monthly analytics (1st of month)
- Coaching: post-session summary + midpoint progress report
- Hybrid/Membership: combination based on the specific mix

### Testimonial request

Create `operations/templates/testimonial-request.md`. Send at day 14-21 (after first deliverable, before any problems). Ask 3 questions:
1. "What was your situation before we started?"
2. "What specific result have you seen so far?"
3. "What would you tell someone considering this service?"

Claude drafts the testimonial from their answers. Send the draft back for approval. First 3 clients committed to a testimonial as part of the founding rate deal.

## Step 8: Present next actions

```
=== NEXT ACTIONS ===
1. Run through the delivery flow with the example client. Test each skill.
2. If agency: create a sample deliverable for your portfolio.
3. If coaching: run through week 1 of your curriculum.
4. If hybrid: test both delivery paths independently.
5. Update business.md with your product folder and active skills.
6. Move to Week 3 and start getting clients.
```

## Self-check before finishing

1. delivery/ folder created with correct structure for the chosen path
2. CLAUDE.md hierarchy created (agency/client/project levels for agency, program/client levels for coaching, program-level for membership)
3. At least 2-3 delivery-specific skills generated as SKILL.md files
4. Example client/member folder populated with realistic sample data
5. business.md updated with product info
6. Software path includes the warning about micro-SaaS failure rates
7. Membership path includes member journey map with milestones
8. Platform differentiator noted if custom platform (not Skool)
9. No Tier 1/2 banned words from humanizer rules
10. No em dashes anywhere in the output
11. operations/ folder created with 3 onboarding emails, reporting template, and testimonial request
12. Testimonial request uses 3-question format and is scheduled for day 14-21

## Change log

| Date | Changes |
|---|---|
| 2026-08-19 | Sanitized and republished under NiteOps brand |
| 2026-03-31 | Merged build-operations into build-product. Added "Operations setup (built-in)" section: 3 onboarding emails (welcome, intake, kickoff), reporting template (weekly/monthly by delivery model), testimonial request (3-question format, day 14-21). |
| 2026-03-30 | Full rewrite. 4-path branching (Agency with 6 sub-types, Coaching, Hybrid, Software), CLAUDE.md hierarchy (agency/client/project), agency sub-type specific folder structures and skills, niche vertical industry skills, coaching skills, hybrid dual-structure, software MVP scaffold with warning, example client population. Added membership delivery model path (coaching + community + software), member journey map with milestones, platform differentiator check (custom vs Skool). |
