---
name: seo-content-marketing-claude-skill-suite
description: SEO & Content Marketing command suite with keyword research, content audits, technical SEO, and structured workflows for Claude Code agents
triggers:
  - "run an SEO audit on this website"
  - "generate a content brief for this keyword"
  - "analyze competitor content gaps"
  - "check technical SEO issues"
  - "create a keyword research report"
  - "build a content calendar"
  - "find backlink opportunities"
  - "audit page speed for SEO"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides Claude Code agents with expertise in SEO & Content Marketing tasks, derived from the `alirezarezvani/claude-code-skill-factory` scaffold. It delivers 10 specialized commands and 5 multi-step workflows with consistent structured output for keyword research, content audits, SERP analysis, technical SEO, and content strategy.

## What This Project Does

The SEO & Content Marketing Skills Suite enables AI coding agents to:

- **Keyword Research**: Deep clustering, opportunity scoring, SERP intent mapping
- **Content Audits**: Quality scoring, duplication detection, cannibalization reports
- **Technical SEO**: Crawl budget analysis, Core Web Vitals, schema validation
- **Competitor Analysis**: Backlink gaps, topic gaps, featured snippet opportunities
- **Content Creation**: AI-generated briefs with outlines, NLP terms, word targets
- **SERP Monitoring**: Rank tracking, volatility alerts, CTR optimization
- **Link Building**: Prospect lists with DA/DR filters, outreach templates
- **Performance**: Page speed diagnostics mapped to ranking impact
- **Local SEO**: NAP consistency, GBP optimization, citation audits
- **Planning**: Data-driven editorial calendars from search demand

All commands provide structured, actionable output with progress tracking, findings tables, and prioritized action plans.

## Installation

### Clone into Claude Skills Directory

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/

# Clone this skill suite
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git \
  ~/.claude/skills/seo-content-marketing-suite/

# Or download and extract
cd ~/.claude/skills/
curl -L https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo/archive/main.tar.gz | tar xz
mv r04-alirezarezvani-claude-code-skill-factory-seo-main seo-content-marketing-suite
```

### Register in Claude Code

In a Claude Code session:

```
/read ~/.claude/skills/seo-content-marketing-suite/SKILL.md
```

Or add to your project's `.claude/skills/` directory for automatic loading.

## Core Commands

### 1. Keyword Research

Performs deep keyword clustering, opportunity scoring, and SERP intent mapping.

```bash
/keyword-research <target>
/keyword-research "saas marketing" --cluster --export csv
```

**Output Structure:**

```
╔════════════════════════════════════════════════════╗
║  Keyword Research — "saas marketing"               ║
╠════════════════════════════════════════════════════╣
║  Fetching keywords …      [██████████] 100%  ✓     ║
║  Clustering terms …       [██████████] 100%  ✓     ║
║  Scoring opportunities …  [██████████] 100%  ✓     ║
╚════════════════════════════════════════════════════╝

┌──────────────────────┬────────┬──────┬────────┬──────────┐
│ Keyword              │ Volume │ KD   │ Intent │ Priority │
├──────────────────────┼────────┼──────┼────────┼──────────┤
│ saas marketing       │  8,100 │   45 │ Info   │  🟢 High │
│ b2b saas marketing   │  1,600 │   38 │ Info   │  🟢 High │
│ saas content mkt     │    720 │   29 │ Info   │  🟡 Med  │
│ saas marketing tools │    590 │   52 │ Comm   │  🟠 Low  │
└──────────────────────┴────────┴──────┴────────┴──────────┘
```

**Configuration:**

- `--cluster`: Enable semantic clustering
- `--intent`: Add SERP intent analysis
- `--export <format>`: csv, json, md

### 2. Content Audit

Full-site content quality scoring, duplication check, and cannibalization detection.

```bash
/content-audit --scope full --output md
/content-audit --url https://example.com/blog --export report.md
```

**Example Output:**

```
╔════════════════════════════════════════════════════╗
║  Content Audit — example.com                       ║
╠════════════════════════════════════════════════════╣
║  Crawling pages …       [████████░░]  80%  124/155 ║
║  Analyzing content …    [██████████] 100%     ✓    ║
║  Detecting duplicates … [██████████] 100%     ✓    ║
╚════════════════════════════════════════════════════╝

📊 Summary Metrics:
• Total pages: 155
• Quality score: 73/100 ⚠
• Duplicate content: 12 pages 🔴
• Cannibalization: 5 clusters 🟠
• Thin content (<300w): 23 pages 🟡

🔍 Top Issues:

┌─────────────────────┬────────┬──────────┬──────────┐
│ Issue               │ Pages  │ Severity │ Impact   │
├─────────────────────┼────────┼──────────┼──────────┤
│ Duplicate title     │     18 │  🔴 High │  -15 pts │
│ Missing H1          │     12 │  🔴 High │  -12 pts │
│ Thin content        │     23 │  🟠 Med  │   -8 pts │
│ No internal links   │      9 │  🟡 Low  │   -3 pts │
└─────────────────────┴────────┴──────────┴──────────┘
```

**Options:**

- `--scope`: `full`, `path`, `single`
- `--output`: `md`, `json`, `html`
- `--threshold`: Minimum quality score (default: 60)

### 3. Technical SEO Audit

Crawl budget, Core Web Vitals, schema markup, and indexability analysis.

```bash
/technical-seo https://example.com
/technical-seo --depth 3 --export json
```

**Checks Performed:**

- Robots.txt validation
- XML sitemap structure
- Canonical tags
- Hreflang implementation
- Schema.org markup
- Core Web Vitals (LCP, FID, CLS)
- Mobile usability
- HTTPS/security headers

**Example Findings:**

```
✓ Robots.txt: Valid, allows all critical paths
✗ Sitemap: Missing lastmod timestamps (155 URLs)
⚠ Canonicals: 12 pages missing canonical tags
✓ Schema: Product markup valid on 89% of pages
✗ Core Web Vitals: LCP failing on 23% of pages
✓ Mobile: 98% pages mobile-friendly
⚠ HTTPS: Mixed content warnings on 5 pages
```

### 4. Competitor Gap Analysis

Backlink gap, topic gap, and featured snippet opportunity identification.

```bash
/competitor-gap --target example.com --competitors competitor1.com,competitor2.com
/competitor-gap --focus backlinks --export csv
```

**Analysis Types:**

1. **Backlink Gap**: Domains linking to competitors but not to you
2. **Topic Gap**: Keywords competitors rank for that you don't
3. **Snippet Gap**: Featured snippets held by competitors

**Output:**

```
🔗 Backlink Gap (Top 20 opportunities):

┌──────────────────────┬────┬────┬──────────┬──────────┐
│ Referring Domain     │ DR │ TF │ You      │ Comp 1   │
├──────────────────────┼────┼────┼──────────┼──────────┤
│ industry-blog.com    │ 67 │ 45 │  ✗ None  │  ✓ 3 lnk │
│ tech-review.co       │ 72 │ 51 │  ✗ None  │  ✓ 5 lnk │
│ saas-directory.io    │ 58 │ 38 │  ✗ None  │  ✓ 1 lnk │
└──────────────────────┴────┴────┴──────────┴──────────┘

📄 Topic Gap (Top keywords to target):

┌──────────────────────┬────────┬──────┬──────────┐
│ Keyword              │ Volume │ KD   │ Comp Pos │
├──────────────────────┼────────┼──────┼──────────┤
│ saas metrics         │  3,600 │   42 │  Pos 3   │
│ churn rate calc      │  1,200 │   35 │  Pos 5   │
│ mrr vs arr           │    880 │   28 │  Pos 2   │
└──────────────────────┴────────┴──────┴──────────┘
```

### 5. Content Brief Generator

AI-generated SEO content brief with outline, NLP terms, and word count targets.

```bash
/content-brief "saas marketing strategies"
/content-brief --keyword "b2b content marketing" --format md
```

**Brief Components:**

```markdown
# Content Brief: SaaS Marketing Strategies

## Target Keyword
- Primary: `saas marketing strategies` (Vol: 1,900, KD: 48)
- Secondary: `b2b saas marketing`, `saas growth tactics`

## Search Intent
**Informational** — Users want comprehensive guides and actionable strategies

## Target Metrics
- Word count: 2,400–2,800 words
- Reading level: Grade 10–12
- Internal links: 5–7
- External links: 3–5 authoritative sources

## Outline

### 1. Introduction (150–200w)
- Hook: Current state of SaaS competition
- What this guide covers
- Who it's for

### 2. Core SaaS Marketing Channels (800–1,000w)
#### 2.1 Content Marketing
- Blog strategy for SaaS
- SEO-driven content calendar
- Case studies & whitepapers

#### 2.2 Product-Led Growth
- Free trial optimization
- In-app messaging
- Referral programs

[... continues with full outline ...]

## NLP Terms to Include
Must include (10+): `customer acquisition`, `churn rate`, `lifetime value`, 
`product-market fit`, `conversion funnel`, `freemium model`, `retention`, 
`onboarding`, `MRR`, `CAC`

Recommended (5+): `PLG`, `bottom-up sales`, `land and expand`, `usage-based`

## Competitor Analysis
Top 3 ranking pages:
1. hubspot.com/marketing/saas (2,850w, Pos 1)
2. neilpatel.com/blog/saas-marketing (3,200w, Pos 2)
3. cxl.com/blog/saas-marketing-guide (2,600w, Pos 3)

## Visual Assets
- Strategy framework diagram
- Channel comparison table
- Funnel optimization infographic
```

### 6. SERP Monitor

Daily rank tracking with volatility alerts and CTR optimization tips.

```bash
/serp-monitor --keywords keywords.csv --frequency daily
/serp-monitor --url example.com/blog --alert slack
```

**Tracking Dashboard:**

```
╔════════════════════════════════════════════════════╗
║  SERP Monitor — example.com                        ║
║  Last updated: 2026-05-11 09:00 UTC                ║
╠════════════════════════════════════════════════════╣

📈 Performance Summary (Last 7 days):

┌──────────────────────┬─────────┬──────────┬─────────┐
│ Metric               │ Current │ Δ 7d     │ Status  │
├──────────────────────┼─────────┼──────────┼─────────┤
│ Avg. Position        │    12.3 │  ↑ +2.1  │  🟢 Up  │
│ Keywords in Top 10   │      37 │  ↑ +5    │  🟢 Up  │
│ Total Impressions    │ 45,200  │  ↑ +8.2% │  🟢 Up  │
│ CTR                  │   3.8 % │  ↓ -0.3% │  🟡 -   │
└──────────────────────┴─────────┴──────────┴─────────┘

🔔 Alerts:

⚠ High volatility detected for "saas pricing" (Pos 5→14→7)
✓ "content marketing roi" entered Top 3 (Pos 8→3)
🔴 "email marketing tools" dropped out of Top 10 (Pos 9→15)
```

### 7. Link Prospecting

Quality backlink prospect list with DA/DR filters and outreach templates.

```bash
/link-prospecting --topic "saas marketing" --min-dr 40
/link-prospecting --export prospects.csv --template outreach
```

**Prospect Criteria:**

- Domain Rating (DR) / Domain Authority (DA)
- Traffic estimate
- Relevance score
- Contact availability
- Outreach difficulty

**Output:**

```
🎯 Link Prospects: SaaS Marketing

Filters: DR ≥40, Relevance ≥7/10, Traffic ≥5k/mo

┌──────────────────────┬────┬─────────┬──────────┬──────────┐
│ Domain               │ DR │ Traffic │ Relevnce │ Contact  │
├──────────────────────┼────┼─────────┼──────────┼──────────┤
│ saas-review.com      │ 68 │  45k/mo │   9/10   │  ✓ Email │
│ marketing-blog.io    │ 62 │  32k/mo │   8/10   │  ✓ Form  │
│ growth-hacks.co      │ 54 │  18k/mo │   8/10   │  ✓ Email │
│ tech-insights.net    │ 48 │  12k/mo │   7/10   │  ⚠ None  │
└──────────────────────┴────┴─────────┴──────────┴──────────┘

📧 Outreach Template Generated:
→ /output/outreach-template-saas-marketing.md
```

### 8. Page Speed SEO Analysis

Render-blocking diagnostics, LCP/CLS/FID analysis mapped to ranking impact.

```bash
/page-speed-seo https://example.com/page
/page-speed-seo --mobile --export report.json
```

**Metrics Analyzed:**

- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)
- Time to First Byte (TTFB)
- Total Blocking Time (TBT)
- Render-blocking resources

**Ranking Impact Mapping:**

```
🚀 Page Speed SEO — example.com/page

Core Web Vitals Status: ⚠ Needs Improvement

┌──────────────┬──────────┬──────────┬──────────┬──────────┐
│ Metric       │ Score    │ Target   │ Status   │ SEO Δ    │
├──────────────┼──────────┼──────────┼──────────┼──────────┤
│ LCP          │  3.8 s   │  <2.5 s  │  🟠 Slow │  -5 pts  │
│ FID          │   85 ms  │ <100 ms  │  ✓ Good  │    —     │
│ CLS          │  0.18    │  <0.1    │  🔴 Poor │  -8 pts  │
│ TTFB         │  1.2 s   │  <0.8 s  │  🟠 Slow │  -3 pts  │
└──────────────┴──────────┴──────────┴──────────┴──────────┘

Estimated Ranking Impact: -16 positions

🔧 Critical Fixes (Ranked by Impact):

1. 🔴 Reduce CLS (0.18 → <0.1) — +8 ranking pts
   • Add width/height to images (12 affected)
   • Preload web fonts
   • Reserve ad slot space

2. 🟠 Improve LCP (3.8s → 2.5s) — +5 ranking pts
   • Optimize hero image (2.1 MB → ~200 KB)
   • Remove render-blocking CSS (3 files)
   • Enable lazy loading below fold

3. 🟡 Reduce TTFB (1.2s → 0.8s) — +3 ranking pts
   • Enable server-side caching
   • Use CDN for static assets
```

### 9. Local SEO Audit

NAP consistency, Google Business Profile optimization, and local citation audit.

```bash
/local-seo "Business Name" --location "New York, NY"
/local-seo --citations --export csv
```

**Audit Components:**

```
📍 Local SEO Audit — Business Name (New York, NY)

1️⃣ NAP Consistency Check:

✓ Google Business Profile: ✓ Consistent
✗ Yelp: Phone mismatch (555-0100 vs 555-0101)
✓ Facebook: ✓ Consistent
⚠ YellowPages: Address variant ("St" vs "Street")

Consistency Score: 75% (⚠ Needs work)

2️⃣ Google Business Profile:

✓ Verified: Yes
✓ Category: Primary + 2 secondary
⚠ Photos: 12 (target: 20+)
✗ Posts: Last post 45 days ago (target: weekly)
✓ Reviews: 4.6★ (87 reviews)
⚠ Q&A: 3 unanswered questions

3️⃣ Citation Audit (Top 50 directories):

┌──────────────────────┬──────────┬──────────┬──────────┐
│ Directory            │ Listed   │ NAP      │ Action   │
├──────────────────────┼──────────┼──────────┼──────────┤
│ Google Business      │  ✓ Yes   │  ✓ Match │    —     │
│ Bing Places          │  ✓ Yes   │  ✓ Match │    —     │
│ Yelp                 │  ✓ Yes   │  ✗ Phone │  Fix     │
│ YellowPages          │  ✓ Yes   │  ⚠ Addr  │  Fix     │
│ Foursquare           │  ✗ No    │    —     │  Add     │
└──────────────────────┴──────────┴──────────┴──────────┘

📊 Local Pack Ranking (Target keywords):
• "plumber new york": Not in pack
• "emergency plumber nyc": Position 8 (⚠ Just outside)
• "24 hour plumber manhattan": Position 3 (✓ In pack)
```

### 10. Content Calendar Generator

Data-driven editorial calendar built from search demand and seasonality.

```bash
/content-calendar --topics "saas,marketing,sales" --months 3
/content-calendar --keywords keywords.csv --export calendar.csv
```

**Calendar Structure:**

```
📅 Content Calendar — Q2 2026 (May–Jul)

Strategy: Search demand + seasonality + competitor gap filling

Week of May 12, 2026
┌─────────────────────────────────────────────────────────┐
│ Mon: [Blog] "SaaS Pricing Models Explained"             │
│      KW: saas pricing (Vol: 1,600, KD: 42)              │
│      Format: 2,500w guide | Est: 8h                     │
│                                                          │
│ Wed: [Video] "How to Calculate SaaS Metrics"            │
│      KW: saas metrics (Vol: 3,600, KD: 38)              │
│      Format: 12min tutorial | Est: 6h                   │
│                                                          │
│ Fri: [Infographic] "SaaS Sales Funnel Stages"           │
│      KW: saas sales funnel (Vol: 720, KD: 35)           │
│      Format: Visual + 800w post | Est: 5h               │
└─────────────────────────────────────────────────────────┘

Week of May 19, 2026
┌─────────────────────────────────────────────────────────┐
│ Mon: [Pillar] "Complete Guide to B2B Content Marketing" │
│      KW: b2b content marketing (Vol: 2,400, KD: 51)     │
│      Format: 4,000w pillar + 5 clusters | Est: 16h      │
│                                                          │
│ Thu: [Case Study] "How [Client] Reduced Churn 40%"      │
│      KW: reduce saas churn (Vol: 590, KD: 44)           │
│      Format: 1,800w case study | Est: 10h               │
└─────────────────────────────────────────────────────────┘

📊 Monthly Targets:
• Posts: 12 (4/week)
• Total words: 28,000
• Target traffic: +15% MoM
• Avg. time investment: 6h/post
```

## Multi-Step Workflows

### Full SEO Sprint

12-step comprehensive SEO sprint from audit to execution.

```bash
/workflows:full-seo-sprint https://example.com --duration 4weeks
```

**Sprint Steps:**

1. Technical SEO audit
2. Content audit
3. Competitor gap analysis
4. Keyword research & mapping
5. On-page optimization priorities
6. Content refresh plan
7. New content pipeline
8. Internal linking structure
9. Backlink acquisition strategy
10. Local SEO (if applicable)
11. Tracking & monitoring setup
12. 30/60/90 day roadmap

**Example Progress:**

```
╔════════════════════════════════════════════════════╗
║  SEO Sprint — example.com                          ║
║  Week 1/4 — Foundation & Analysis                  ║
╠════════════════════════════════════════════════════╣
║  ✓ Step 1: Technical audit complete                ║
║  ✓ Step 2: Content audit complete                  ║
║  → Step 3: Competitor analysis [████░░░░░░] 40%    ║
║  ⏳ Step 4: Keyword research (pending)              ║
╚════════════════════════════════════════════════════╝
```

### Launch SEO Checklist

Pre-launch SEO validation with canonical, hreflang, and sitemap checks.

```bash
/workflows:launch-seo https://staging.example.com --production https://example.com
```

**Checklist Items:**

```
✅ Pre-Launch SEO Checklist

🔍 Technical Setup
☐ Robots.txt configured (allow crawling in prod, block in staging)
☐ XML sitemap generated and submitted
☐ Canonical tags pointing to production URLs
☐ Hreflang tags (if multilingual)
☐ 301 redirects from old URLs mapped
☐ 404 page customized with search/nav
☐ SSL certificate installed and HTTPS enforced

📄 On-Page Essentials
☐ All pages have unique title tags (<60 chars)
☐ All pages have unique meta descriptions (<160 chars)
☐ H1 tags present and unique on all pages
☐ Image alt text added (focus on key images)
☐ Internal linking structure reviewed
☐ Schema.org markup implemented (Organization, Product, etc.)

⚙️ Performance & UX
☐ Core Web Vitals passing (LCP <2.5s, FID <100ms, CLS <0.1)
☐ Mobile-friendly test passed
☐ Page speed >80 (mobile + desktop)
☐ No render-blocking resources in critical path

🔗 Off-Page Setup
☐ Google Search Console verified
☐ Google Analytics 4 installed
☐ Bing Webmaster Tools verified
☐ Google Business Profile created (if local)
☐ Social media profiles linked

📊 Tracking & Monitoring
☐ Rank tracking setup for target keywords
☐ Conversion tracking configured
☐ Uptime monitoring enabled
☐ Crawl budget monitoring (enterprise sites)
```

### Content Refresh Workflow

Identify and refresh underperforming pages to recover rankings.

```bash
/workflows:content-refresh --min-age 6months --rank-drop 5
```

**Workflow Phases:**

1. **Identify**: Pages that dropped >5 positions in last 6 months
2. **Analyze**: What competitors updated, new SERP features
3. **Prioritize**: By traffic loss potential
4. **Refresh**: Update stats, add sections, improve depth
5. **Republish**: Update date, re-promote
6. **Monitor**: Track ranking recovery

**Example Report:**

```
📉 Content Refresh Candidates (Top 10 by impact)

┌──────────────────────┬──────┬────────┬──────────┬──────────┐
│ URL                  │ Rank │ Δ Rank │ Traffic  │ Priority │
├──────────────────────┼──────┼────────┼──────────┼──────────┤
│ /saas-metrics-guide  │   12 │  ↓ -8  │  -450/mo │  🔴 High │
│ /content-mkt-roi     │    9 │  ↓ -6  │  -320/mo │  🔴 High │
│ /email-best-practice │    7 │  ↓ -5  │  -180/mo │  🟠 Med  │
└──────────────────────┴──────┴────────┴──────────┴──────────┘

🔧 Recommended Updates for /saas-metrics-guide:

1. Add 2026 benchmarks (competitor added)
2. Expand "churn rate" section (+400 words)
3. Add calculator tool embed
4. Update 3 outdated screenshots
5. Add schema FAQ markup for 5 common questions
6. Improve internal linking to product pages
```

### Authority Building Campaign

End-to-end digital PR and link-building workflow.

```bash
/workflows:authority-building --duration 12weeks --target-links 50
```

**Campaign Phases:**

1. **Research**: Competitor backlinks, industry publications
2. **Content**: Linkable assets (data studies, tools, guides)
3. **Outreach**: Personalized pitches to prospects
4. **Relationships**: Build journalist/blogger relationships
5. **Monitoring**: Track placements and link acquisition
6. **Reporting**: ROI analysis and attribution

### AI Content Pipeline

Automated keyword → brief → draft → optimize → publish workflow.

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --auto-brief
```

**Pipeline Stages:**

```
📝 AI Content Pipeline

Stage 1: Keyword Selection
→ Input: keywords.csv (250 keywords)
→ Prioritization: Volume × (100 - KD) × Relevance
→ Output: Top 20 keywords for next 4 weeks

Stage 2: Brief Generation
→ For each keyword: /content-brief
→ Auto-generate outline, NLP terms, word targets
→ Human review: Approve/modify briefs

Stage 3: Draft Creation
→ AI-assisted writing (Claude/GPT-4)
→ Follow brief structure
→ Include required NLP terms
→ Target word count ±10%

Stage 4: Optimization
→ On-page SEO check (title, meta, headings)
→ Readability score (target: Grade 10-12)
→ Internal/external link insertion
→ Image optimization and alt text

Stage 5: Review & Publish
→ Human editor review
→ Plagiarism check
→ Publish to CMS
→ Submit to GSC for indexing
→ Add to SERP monitoring

📊 Pipeline Metrics (Last 30 days):
• Articles published: 18
• Avg. time per article: 4.2h
• Avg. ranking (30d): Position 12.8
• Traffic generated: +2,340 sessions
```

## Configuration

### Environment Variables

Set these in your environment or `.env` file:

```bash
# SEO APIs
export SEMRUSH_API_KEY=your_semrush_key
export AHREFS_API_KEY=your_ahrefs_key
export MOZ_ACCESS_ID=your_moz_id
export MOZ_SECRET_KEY=your_moz_secret

# Search Console
export GSC_CLIENT_ID=your_google_client_id
export GSC_CLIENT_SECRET=your_google_client_secret

# Analytics
export GA4_PROPERTY_ID=your_ga4_property
export GA4_CREDENTIALS_PATH=/path/to/credentials.json

# Monitoring
export SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK
export DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/YOUR/WEBHOOK

# Content AI
export OPENAI_API_KEY=your_openai_key
export ANTHROPIC_API_KEY=your_anthropic_key
```

### Skill Configuration File

Create `~/.claude/skills/seo-content-marketing-suite/config.yml`:

```yaml
# Default settings
defaults:
  export_format: markdown
  min_dr: 30
  keyword_volume_min: 100
  content_min_words: 800
  
# Command-specific settings
commands:
  keyword_research:
    default_cluster: true
    max_results: 200
    intent_analysis: true
    
  content_audit:
    quality_threshold: 60
    thin_content_limit: 300
    duplicate_similarity: 0.85
    
  technical_seo:
    max_depth: 5
    timeout: 300
    user_agent: "SEO-Audit-Bot/1.0"
    
