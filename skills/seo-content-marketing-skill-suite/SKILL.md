---
name: seo-content-marketing-skill-suite
description: SEO & Content Marketing command suite with keyword research, content audits, SERP analysis, technical SEO checks and link prospecting
triggers:
  - analyze SEO performance for this site
  - run a content audit on my website
  - find keyword opportunities for my target
  - check technical SEO issues and page speed
  - build a content calendar from search demand
  - analyze competitor backlink gaps
  - generate an SEO content brief
  - monitor SERP rankings and volatility
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

**r04-alirezarezvani-claude-code-skill-factory-seo** is a specialized command suite for SEO professionals and content marketers. It provides 10 domain-specific slash commands and 5 multi-step workflows for keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy automation.

Derived from [alirezarezvani/claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory), this suite extends the skill scaffolding framework with structured output UI, progress tracking, and prioritized action plans.

---

## Installation

### Method 1: Clone to Skills Directory

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Clone the repository
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git \
  ~/.claude/skills/seo-content-marketing

# Register in Claude Code
# In your Claude Code session:
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Method 2: Manual Copy

```bash
# Copy the skill directory into your project
cp -r /path/to/r04-alirezarezvani-claude-code-skill-factory-seo ./skills/

# Load the skill
/read ./skills/r04-alirezarezvani-claude-code-skill-factory-seo/SKILL.md
```

---

## Core Commands

### 1. Keyword Research

**Trigger:** `/keyword-research <target>`

Performs deep keyword clustering, opportunity scoring, and SERP intent mapping.

**Options:**
- `--depth <shallow|deep|comprehensive>` — analysis depth (default: deep)
- `--min-volume <number>` — minimum monthly search volume
- `--max-difficulty <number>` — maximum keyword difficulty (0-100)
- `--intent <informational|commercial|transactional>` — filter by search intent
- `--output <json|md|csv>` — output format (default: md)

**Example:**

```bash
/keyword-research "project management software" --depth comprehensive --min-volume 500 --output md
```

**Expected Output Structure:**

```markdown
# Keyword Research: project management software

## Summary
- Total keywords found: 247
- High-opportunity (low comp, high vol): 34
- Content gaps identified: 12
- Recommended clusters: 8

## Top Opportunities

| Keyword | Volume | Difficulty | Intent | Opportunity Score |
|---------|--------|------------|--------|-------------------|
| project management tools for small teams | 2,400 | 28 | Commercial | 94/100 |
| free project management software | 8,100 | 67 | Informational | 76/100 |
| project tracking templates | 1,900 | 22 | Informational | 91/100 |

## Keyword Clusters

### Cluster 1: Getting Started (Informational)
- project management basics (3,600 vol, 24 diff)
- what is project management software (1,300 vol, 18 diff)
- project management for beginners (880 vol, 21 diff)

**Content Recommendation:** Ultimate guide format, 2,500+ words

---
```

### 2. Content Audit

**Trigger:** `/content-audit [domain]`

Full-site content quality scoring, duplicate detection, and cannibalization analysis.

**Options:**
- `--scope <full|partial|urls>` — audit scope
- `--urls <file.txt>` — specific URLs to audit
- `--metrics <quality,duplication,performance>` — comma-separated metrics
- `--threshold <number>` — minimum quality score (0-100)

**Example:**

```bash
/content-audit example.com --scope full --metrics quality,duplication,performance
```

**Expected Output:**

```markdown
# Content Audit: example.com

## Crawl Summary
- Total pages crawled: 1,204
- Pages analyzed: 1,180
- Issues found: 247
- Audit duration: 12m 34s

## Quality Distribution

🔴 Critical (0-40): 23 pages
🟠 Poor (41-60): 89 pages
🟡 Fair (61-75): 234 pages
🟢 Good (76-100): 834 pages

## Top Issues

| Issue | Pages Affected | Severity | Impact |
|-------|----------------|----------|--------|
| Thin content (<300 words) | 127 | 🔴 High | Low rankings, high bounce |
| Duplicate title tags | 45 | 🟠 Medium | Indexation confusion |
| Keyword cannibalization | 34 | 🔴 High | Split ranking authority |
| Missing meta descriptions | 302 | 🟡 Low | Lower CTR |

## Action Plan

### Quick Wins (1-2 weeks)
- [ ] Add meta descriptions to 302 pages
- [ ] Consolidate 12 duplicate product pages
- [ ] Expand 34 thin content pages to 800+ words

### Medium-term (1-2 months)
- [ ] Resolve 34 cannibalization clusters
- [ ] Update 89 poor-quality pages
- [ ] Implement canonical tags on 67 near-duplicate pages
```

### 3. Technical SEO Audit

**Trigger:** `/technical-seo [domain]`

Crawl budget analysis, Core Web Vitals, schema markup validation, and indexability checks.

**Options:**
- `--checks <all|speed|schema|crawl|indexability>` — specific checks to run
- `--device <desktop|mobile|both>` — device type for speed tests
- `--output-format <detailed|summary>` — report detail level

**Example:**

```bash
/technical-seo example.com --checks all --device both --output-format detailed
```

**Expected Output:**

```markdown
# Technical SEO Audit: example.com

## Core Web Vitals (Mobile)

| Metric | Value | Status | Threshold |
|--------|-------|--------|-----------|
| LCP (Largest Contentful Paint) | 2.1s | 🟢 Good | <2.5s |
| FID (First Input Delay) | 87ms | 🟡 Needs Improvement | <100ms |
| CLS (Cumulative Layout Shift) | 0.18 | 🟠 Poor | <0.1 |

## Indexability Issues

🔴 **Critical**
- 23 pages blocked by robots.txt but linked internally
- 12 pages with noindex but in sitemap.xml
- 5 orphan pages (no internal links)

🟠 **Medium**
- 67 redirect chains (3+ hops)
- 34 pages with slow server response (>600ms)

## Schema Markup

✓ Valid schema on 892 pages
✗ Missing schema on 312 pages
⚠ Schema errors on 18 pages

**Recommended schema types:**
- Product schema for 156 product pages
- FAQ schema for 78 support pages
- BreadcrumbList for all pages

## Action Checklist

1. Fix CLS issues (replace dynamic ads with fixed dimensions)
2. Remove noindex from 12 sitemap URLs
3. Add redirects for 23 blocked pages or remove internal links
4. Implement breadcrumb schema site-wide
5. Break 67 redirect chains into single 301s
```

### 4. Competitor Gap Analysis

**Trigger:** `/competitor-gap <your-domain> <competitor-domains...>`

Backlink gap, topic gap, and featured snippet opportunity analysis.

**Options:**
- `--analysis <backlinks|content|snippets|all>` — gap analysis type
- `--min-dr <number>` — minimum domain rating for backlink sources
- `--limit <number>` — max results per category

**Example:**

```bash
/competitor-gap example.com competitor1.com competitor2.com --analysis all --min-dr 30
```

**Expected Output:**

```markdown
# Competitor Gap Analysis

## Backlink Gap

**Competitors rank with these links (you don't have):**

| Referring Domain | DR | Competitors Linking | Suggested Page |
|------------------|-----|---------------------|----------------|
| industry-news.com | 72 | competitor1, competitor2 | /blog/industry-trends |
| software-review-site.com | 68 | competitor1 | /product-comparison |
| expert-blog.com | 54 | competitor2 | /case-studies |

**Action:** 34 link prospects identified → `/link-prospecting` for outreach templates

## Content Gap

**Topics competitors rank for (you don't):**

| Topic Cluster | Competitor Pages | Est. Traffic | Difficulty |
|---------------|------------------|--------------|------------|
| Project management methodologies | 12 | 8,400/mo | 42 |
| Remote team collaboration | 8 | 5,200/mo | 38 |
| Agile vs Waterfall comparison | 5 | 3,100/mo | 51 |

## Featured Snippet Opportunities

You rank #2-5 for these queries with featured snippets:

| Query | Your Position | Snippet Type | Current Holder |
|-------|---------------|--------------|----------------|
| what is agile project management | #3 | Paragraph | competitor1.com |
| project management templates | #2 | List | competitor2.com |
| kanban vs scrum | #4 | Table | competitor1.com |

**Optimization:** Target paragraph snippets with 40-60 word definitions in H2 sections
```

### 5. SEO Content Brief Generator

**Trigger:** `/content-brief <target-keyword>`

AI-generated SEO content brief with outline, NLP terms, word count targets, and competitor analysis.

**Options:**
- `--competitors <urls>` — comma-separated competitor URLs to analyze
- `--intent <informational|commercial|transactional>` — search intent
- `--format <blog-post|landing-page|product-page>` — content type

**Example:**

```bash
/content-brief "how to choose project management software" \
  --competitors "competitor1.com/guide,competitor2.com/article" \
  --intent informational \
  --format blog-post
```

**Expected Output:**

```markdown
# Content Brief: "how to choose project management software"

## Target Metrics
- **Primary Keyword:** how to choose project management software
- **Search Volume:** 2,400/mo
- **Keyword Difficulty:** 34
- **Search Intent:** Informational (buying-guide)
- **Target Word Count:** 2,200-2,800 words
- **Target Reading Level:** Grade 8-10

## SERP Analysis

**Current Top 3:**
1. competitor1.com/guide (2,847 words, DR 68, 12 backlinks)
2. competitor2.com/article (1,923 words, DR 54, 8 backlinks)
3. industry-blog.com/post (3,104 words, DR 71, 34 backlinks)

**Common Elements:**
- Comparison tables (feature matrix)
- Pros/cons lists for each software type
- Pricing breakdowns
- Use-case scenarios
- FAQ section (8-12 questions)

## Recommended Outline

### H1: How to Choose Project Management Software: Complete 2026 Guide

### H2: What is Project Management Software?
- Brief definition (40-60 words for snippet optimization)
- Types of PM software (bullet list)

### H2: 7 Key Factors When Choosing PM Software
- H3: Team Size & Scalability
- H3: Budget & Pricing Models
- H3: Feature Requirements (task tracking, time tracking, reporting)
- H3: Integration Capabilities
- H3: Ease of Use & Learning Curve
- H3: Mobile Access & Remote Work Features
- H3: Security & Compliance

### H2: Popular Project Management Software Compared
- [Comparison table with 5-7 tools]

### H2: How to Evaluate PM Software (Step-by-Step)
1. Assess your team's needs
2. Set a budget range
3. Create a feature checklist
4. Test free trials
5. Check integration compatibility
6. Read user reviews
7. Make your decision

### H2: Common Mistakes to Avoid
- Bullet list of 5-7 mistakes

### H2: FAQ
- 10 questions based on "People Also Ask" SERP feature

### H2: Conclusion & Next Steps
- Summary + CTA

## NLP Terms to Include

**Entity Coverage:**
- Asana, Trello, Monday.com, Jira, ClickUp (competitor tools)
- Gantt chart, Kanban board, sprint planning (features)
- Small business, enterprise, remote team (audience segments)

**Semantic Keywords (use 60%+ of these):**
- project management tool selection
- software comparison
- team collaboration platform
- task management features
- pricing plans
- free trial
- integration options
- user interface
- reporting capabilities

## Content Requirements

- [ ] Include at least one comparison table
- [ ] Add 2-3 custom screenshots or diagrams
- [ ] Write 40-60 word definition for snippet optimization
- [ ] Create FAQ section with schema markup
- [ ] Internal link to product comparison page
- [ ] External link to 2-3 authoritative sources (e.g., PMI.org)
- [ ] Add jump links for long sections
- [ ] Mobile-friendly formatting (short paragraphs, subheadings every 200-300 words)

## Meta Tags

**Title (58 chars):** How to Choose Project Management Software [2026 Guide]

**Meta Description (155 chars):** Complete guide to choosing the right project management software for your team. Compare features, pricing, integrations & more. Updated 2026.

## Next Steps

1. Use `/ai-content-pipeline` to draft this brief
2. Run `/serp-monitor` to track ranking progress after publish
3. Set up `/link-prospecting` campaign to build backlinks
```

### 6. SERP Monitor

**Trigger:** `/serp-monitor <keywords-file|keyword>`

Daily rank tracking with volatility alerts and CTR optimization recommendations.

**Options:**
- `--domain <domain>` — your domain to track
- `--frequency <daily|weekly>` — monitoring frequency
- `--device <desktop|mobile|both>` — device type
- `--location <country-code>` — geographic location

**Example:**

```bash
/serp-monitor keywords.txt --domain example.com --frequency daily --device both
```

**Expected Input (keywords.txt):**

```
project management software
best project management tools
free project management app
agile project management
kanban board software
```

**Expected Output:**

```markdown
# SERP Monitor Report: example.com

## Ranking Summary (Last 7 Days)

| Keyword | Current | Previous | Change | Volatility | CTR |
|---------|---------|----------|--------|------------|-----|
| project management software | #4 | #5 | ↑1 🟢 | Low | 8.2% |
| best project management tools | #8 | #7 | ↓1 🔴 | Medium | 3.1% |
| free project management app | #12 | #15 | ↑3 🟢 | High | 1.4% |
| agile project management | #6 | #6 | — | Low | 5.7% |
| kanban board software | #3 | #2 | ↓1 🔴 | Medium | 11.3% |

## Alerts

🔴 **High Volatility Detected**
- "free project management app" moved from #15 to #9 to #12 (3-day period)
- Likely cause: SERP feature changes (People Also Ask box added)

🟠 **CTR Opportunity**
- "best project management tools" at #8 has 3.1% CTR (expected: 5-6%)
- **Action:** Rewrite title tag to include year and power words

## CTR Optimization Suggestions

**Keyword:** best project management tools (#8, 3.1% CTR)

Current title: `Best Project Management Tools | Example.com`

Suggested titles:
- `11 Best Project Management Tools in 2026 (Expert Tested)`
- `Best Project Management Software: Top 11 Tools Compared [2026]`
- `The 11 Best Project Management Tools for Teams (2026 Guide)`

**Expected CTR improvement:** +2-3 percentage points

## Next Steps

1. Update title tag for "best project management tools"
2. Monitor "free project management app" for SERP feature optimization
3. Set up featured snippet tracking for "agile project management"
```

### 7. Link Prospecting

**Trigger:** `/link-prospecting <topic|url>`

Quality backlink prospect discovery with DA/DR filters and outreach templates.

**Options:**
- `--min-dr <number>` — minimum Domain Rating
- `--max-dr <number>` — maximum Domain Rating (avoid tier-1 sites you can't get)
- `--type <guest-post|resource-page|broken-link|unlinked-mention>` — prospecting method
- `--limit <number>` — max prospects to return

**Example:**

```bash
/link-prospecting "project management" --min-dr 30 --max-dr 70 --type guest-post --limit 50
```

**Expected Output:**

```markdown
# Link Prospecting: project management

## Prospect Summary
- Total prospects found: 127
- Filtered by DR 30-70: 48
- Guest post opportunities: 34
- Resource page opportunities: 14

## Top Prospects (Guest Posts)

| Domain | DR | Traffic | Contact | Notes |
|--------|-----|---------|---------|-------|
| pm-insights.com | 54 | 45K/mo | editor@pm-insights.com | Accepts guest posts, bio link allowed |
| business-tools-blog.com | 48 | 28K/mo | Form | Weekly guest post slot, 1,500+ words |
| startup-resources.io | 61 | 67K/mo | submissions@startup-resources.io | High editorial standards, 2-3 week review |

## Outreach Template (Guest Post)

**Subject:** Guest Post Idea: [Your Topic] for [Their Site]

---

Hi [Name],

I'm [Your Name] from [Your Company], and I've been following [Their Site] for [specific reason — e.g., your recent article on agile methodologies].

I'd love to contribute a guest post on **[specific topic]** — I noticed you haven't covered [angle] yet, and I think your audience would find it valuable.

**Proposed outline:**
- [H2 heading 1]
- [H2 heading 2]
- [H2 heading 3]

I've written for [Site 1], [Site 2], and [Site 3] (links: [URLs]).

Would this be a good fit for [Their Site]?

Best,
[Your Name]

---

## Resource Page Opportunities

| Page URL | DR | Anchor Context | Suggested Pitch |
|----------|-----|----------------|-----------------|
| industry-tools.com/resources | 58 | "Project management tools and guides" | Pitch your ultimate guide as a comprehensive resource |
| startup-toolkit.io/pm-resources | 52 | Lists 20+ PM tools | Suggest adding your tool with brief description |

## Next Steps

1. Export prospect list: `/export prospects.csv`
2. Set up email sequences in your outreach tool
3. Track responses in `/link-prospecting --track`
4. Follow up after 5-7 days if no response
```

### 8. Page Speed SEO Analysis

**Trigger:** `/page-speed-seo <url>`

Render-blocking resource detection, LCP/CLS/FID diagnosis mapped to ranking impact.

**Options:**
- `--device <desktop|mobile|both>` — device type
- `--throttling <none|3g|4g>` — network throttling
- `--report <summary|detailed>` — report detail level

**Example:**

```bash
/page-speed-seo https://example.com/product-page --device mobile --throttling 4g --report detailed
```

**Expected Output:**

```markdown
# Page Speed SEO: example.com/product-page (Mobile, 4G)

## Performance Score: 68/100 🟠

### Core Web Vitals

| Metric | Value | Status | Impact on Rankings |
|--------|-------|--------|-------------------|
| LCP | 3.8s | 🔴 Poor | **High** — LCP >2.5s negatively impacts mobile rankings |
| FID | 120ms | 🟡 Needs Improvement | Medium — May affect engagement metrics |
| CLS | 0.24 | 🔴 Poor | **High** — CLS >0.1 causes user frustration, increases bounce rate |

## Issues Ranked by SEO Impact

### 🔴 Critical (Fix Immediately)

**1. Render-Blocking Resources (LCP Impact: -1.2s)**
- `/css/main.css` (247 KB) — blocks rendering
- `/js/analytics.js` (89 KB) — blocks main thread
- **Fix:** Inline critical CSS, defer non-critical CSS, async analytics script

**2. Cumulative Layout Shift from Ads (CLS: 0.18)**
- Dynamic ad slots cause layout shifts
- **Fix:** Reserve fixed dimensions for ad slots (300x250, 728x90)

**3. Large Images Not Optimized (LCP Impact: -0.8s)**
- `hero-image.jpg` (1.2 MB) — uncompressed
- **Fix:** Compress to WebP, use `srcset` for responsive images

### 🟠 Medium Priority

**4. JavaScript Execution Time (FID Impact: +40ms)**
- Main thread blocked for 680ms during load
- **Fix:** Code-split large bundles, defer non-essential JS

**5. Server Response Time (Impact: -0.3s)**
- TTFB: 580ms (target: <200ms)
- **Fix:** Enable caching, use CDN, optimize database queries

### 🟡 Low Priority

**6. Unused CSS (Savings: 67 KB)**
- 34% of CSS unused on this page
- **Fix:** Remove unused Bootstrap components, critical CSS extraction

## Implementation Guide

### Quick Wins (1-2 days)

```html
<!-- BEFORE -->
<link rel="stylesheet" href="/css/main.css">
<script src="/js/analytics.js"></script>

<!-- AFTER -->
<style>
  /* Critical CSS inlined here (above-the-fold styles) */
  .hero { ... }
  .nav { ... }
</style>
<link rel="preload" href="/css/main.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="/css/main.css"></noscript>

<script async src="/js/analytics.js"></script>
```

```html
<!-- Fix layout shift from ads -->
<div class="ad-slot" style="min-height: 250px; width: 300px;">
  <!-- Ad code here -->
</div>
```

### Image Optimization

```bash
# Convert to WebP
cwebp hero-image.jpg -q 80 -o hero-image.webp

# Generate responsive sizes
convert hero-image.jpg -resize 800x hero-image-800w.jpg
convert hero-image.jpg -resize 1200x hero-image-1200w.jpg
convert hero-image.jpg -resize 1600x hero-image-1600w.jpg
```

```html
<picture>
  <source srcset="hero-image-800w.webp 800w,
                  hero-image-1200w.webp 1200w,
                  hero-image-1600w.webp 1600w"
          type="image/webp">
  <img src="hero-image.jpg"
       srcset="hero-image-800w.jpg 800w,
               hero-image-1200w.jpg 1200w,
               hero-image-1600w.jpg 1600w"
       sizes="(max-width: 800px) 100vw, 800px"
       alt="Project management dashboard"
       loading="lazy">
</picture>
```

### Server-Side Fixes

```nginx
# Nginx caching config
location ~* \.(jpg|jpeg|png|webp|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# Enable compression
gzip on;
gzip_types text/css application/javascript image/svg+xml;
```

## Estimated Impact

**After implementing fixes:**
- LCP: 3.8s → 2.1s (↓1.7s) 🟢
- FID: 120ms → 75ms (↓45ms) 🟢
- CLS: 0.24 → 0.08 (↓0.16) 🟢
- **Performance Score: 68 → 92**
- **Ranking Impact:** Potential +2-5 positions for mobile queries

## Next Steps

1. Implement critical CSS inlining
2. Fix ad layout shift
3. Optimize images to WebP
4. Re-test after changes: `/page-speed-seo <url> --device mobile`
5. Monitor Core Web Vitals in Search Console
```

### 9. Local SEO Audit

**Trigger:** `/local-seo <business-name> <location>`

NAP consistency check, Google Business Profile optimization, and local citation audit.

**Options:**
- `--gmb-profile <url>` — Google Business Profile URL
- `--competitors <names>` — comma-separated competitor names
- `--radius <miles>` — local search radius

**Example:**

```bash
/local-seo "Acme Plumbing" "Austin, TX" --gmb-profile https://g.page/acme-plumbing-austin --competitors "Best Plumbing,Quick Fix Plumbing"
```

**Expected Output:**

```markdown
# Local SEO Audit: Acme Plumbing (Austin, TX)

## Google Business Profile Score: 74/100 🟡

### Profile Completeness

| Element | Status | Impact |
|---------|--------|--------|
| Business name | ✓ Correct | — |
| Address | ✓ Verified | — |
| Phone | ✓ Consistent | — |
| Website | ✓ Linked | — |
| Hours | ✗ Missing holiday hours | Medium |
| Categories | ⚠ Only 1 of 10 used | High |
| Description | ⚠ 87 chars (recommend 750) | Medium |
| Photos | ⚠ 12 photos (recommend 50+) | High |
| Posts | ✗ No posts in 30 days | Medium |
| Q&A | ⚠ 3 unanswered questions | Low |
| Reviews | ✓ 47 reviews (4.6 stars) | — |

### Recommendations

**1. Add More Categories**
- Primary: Plumber ✓
- Suggested additions: Emergency plumbing, Water heater repair, Drain cleaning, Leak detection

**2. Expand Description (87 → 750 chars)**

Current:
> Acme Plumbing provides plumbing services in Austin, TX.

Suggested:
> Acme Plumbing is Austin's trusted emergency plumber, serving Travis County since 2005. We specialize in water heater repair, drain cleaning, leak detection, and sewer line replacement. Our licensed, insured plumbers are available 24/7 for emergency plumbing services. We proudly serve Austin, Round Rock, Cedar Park, and Pflugerville with same-day service. Family-owned, locally operated. Call (512) 555-1234 for fast, reliable plumbing repairs.

**3. Upload More Photos**
- Team photos: 3 (add 5-7 more)
- Before/after: 0 (add 10-15)
- Service photos: 6 (good)
- Vehicle/branding: 3 (add 2-3 more)

## NAP Consistency Audit

**Your NAP:**
- Name: Acme Plumbing
- Address: 123 Main St, Austin, TX 78701
- Phone: (512) 555-1234

### Citations Found: 34

| Source | NAP Match | Status |
|--------|-----------|--------|
| Yelp | ✓ Exact match | 🟢 Good |
| BBB | ⚠ Phone: (512) 555-1235 | 🟠 Fix |
| YellowPages | ✓ Exact match | 🟢 Good |
| HomeAdvisor | ⚠ Address: 123 Main Street | 🟡 Minor |
| Angi | ✗ Old address: 456 Oak Ave | 🔴 Critical |

**Inconsistencies found:** 7 citations with incorrect NAP

### Missing Citations (High-Priority)

- [ ] Nextdoor
- [ ] Thumbtack
- [ ] Porch
- [ ] Local.com
- [ ] Manta

## Competitor Comparison (3-Mile Radius)

| Business | GMB Reviews | Avg Rating | Photos | Posts/Month |
|----------|-------------|------------|--------|-------------|
| **Acme Plumbing** | 47 | 4.6 ⭐ | 12 | 0 |
| Best Plumbing | 89 | 4.8 ⭐ | 64 | 4 |
| Quick Fix Plumbing | 123 | 4.4 ⭐ | 38 | 2 |

**Gap Analysis:**
- You need 42 more reviews to match Quick Fix
- Upload 26 more photos to match Quick Fix
- Post 2-4 times/month to match competitors

## Review Strategy

**Current review velocity:** 2.1 reviews/month

**Suggested actions:**
1. Send review request emails to recent customers (template below)
2. Add QR code to invoices linking to GMB review page
3. Train technicians to ask for reviews after service
4. Respond to all reviews (currently 18 unanswered)

**Review Request Email Template:**

```
Subject: How was our plumbing service?

Hi [Customer Name],

Thanks for choosing Acme Plumbing for your recent [service type] in [city].

We'd love to hear about your experience! Your feedback helps us improve and helps other Austin homeowners find reliable plumbing services.

[Leave a Google Review →]

If there's anything we can do to improve, please reply to this email.

Thanks again!
[Your Name]
Acme Plumbing
(512) 555-1234
```

## Local Content Opportunities

**Keyword gaps (competitors rank, you don't):**
- "emergency plumber austin" — 1,200 local searches/mo
- "water heater repair austin" — 680 local searches/mo
- "austin plumber near me" — 2,100 local searches/mo

**Suggested content:**
1. Service area pages for each suburb (Round Rock, Cedar Park, etc.)
2. FAQ page targeting "plumber austin" + question keywords
3. Blog posts: "Common Plumbing Problems in Austin Homes" (local angle)

## Action Plan

### This Week
- [ ] Fix NAP inconsistencies on BBB and Angi
- [ ] Add 9 more categories to GMB
- [ ] Expand GMB description to 750 characters
- [ ] Answer 3 unanswered Q&A questions
- [
