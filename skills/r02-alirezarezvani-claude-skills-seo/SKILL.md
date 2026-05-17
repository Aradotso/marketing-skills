```yaml
---
name: r02-alirezarezvani-claude-skills-seo
description: SEO & Content Marketing skill suite with keyword research, content audits, technical SEO, and competitor analysis commands
triggers:
  - "run an SEO audit on this site"
  - "perform keyword research for"
  - "analyze content gaps for"
  - "check technical SEO issues"
  - "create a content brief for"
  - "run a competitor backlink analysis"
  - "generate an SEO content calendar"
  - "audit page speed for SEO"
---

# 📈 SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides **10 specialized SEO and content marketing commands** and **5 multi-step workflows** derived from [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills). It focuses on keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy automation with structured output and progress tracking.

---

## What This Project Does

Provides a command-based interface for:
- **Keyword research** with clustering, intent mapping, and opportunity scoring
- **Content audits** detecting duplication and cannibalization
- **Technical SEO** analysis (Core Web Vitals, schema, crawl budget)
- **Competitor analysis** for backlink gaps and topic opportunities
- **Content briefs** with NLP terms and optimization targets
- **Automated workflows** for full SEO sprints and content pipelines

All commands output structured data with progress bars, severity-coded findings, and prioritized action plans.

---

## Installation

### Clone the Skill Suite

```bash
# Clone to your Claude skills directory
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo.git

# Or clone directly into a named directory
git clone https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo.git r02-seo-skills
```

### Register with Claude Code

In a Claude Code session:

```bash
/read ~/.claude/skills/r02-alirezarezvani-claude-skills-seo/SKILL.md
```

Or reference directly in your project:

```bash
# Add to project .claudeconfig
{
  "skills": [
    "~/.claude/skills/r02-alirezarezvani-claude-skills-seo"
  ]
}
```

---

## Core Commands

### `/keyword-research`

Performs deep keyword analysis with clustering, search intent mapping, and opportunity scoring.

**Usage:**

```bash
/keyword-research <target_topic> [--depth shallow|medium|deep] [--market US|UK|global]
```

**Example:**

```bash
/keyword-research "organic dog food" --depth deep --market US
```

**Output Structure:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  organic dog food           ║
╠══════════════════════════════════════════════════╣
║  Fetching seed keywords …    [██████████] 100%   ║
║  Clustering by intent …      [████████░░]  80%   ║
║  Scoring opportunities …     [███████░░░]  70%   ║
╚══════════════════════════════════════════════════╝

┌────────────────────────┬────────┬────┬───────────┬──────────┐
│ Keyword                │ Volume │ KD │ Intent    │ Priority │
├────────────────────────┼────────┼────┼───────────┼──────────┤
│ organic dog food       │ 22,200 │ 45 │ Commercial│  🔴 High │
│ best organic dog food  │  8,100 │ 38 │ Commercial│  🔴 High │
│ organic dog food brands│  5,400 │ 42 │ Info      │  🟠 Med  │
│ grain free organic     │  3,600 │ 35 │ Commercial│  🟡 Med  │
└────────────────────────┴────────┴────┴───────────┴──────────┘
```

**Options:**
- `--depth`: Controls keyword expansion (`shallow` = 50, `medium` = 200, `deep` = 500+ keywords)
- `--market`: Geographic targeting for search volume data
- `--export`: Export to CSV/JSON (`--export json`)

---

### `/content-audit`

Scans site content for quality issues, duplication, and keyword cannibalization.

**Usage:**

```bash
/content-audit [--scope full|sample] [--url <domain>] [--output md|json]
```

**Example:**

```bash
/content-audit --scope full --url https://example.com --output md
```

**Output:**

```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  example.com                   ║
╠══════════════════════════════════════════════════╣
║  Crawling pages …        [██████████] 100%  1,204║
║  Analyzing content …     [████████░░]  80%    963║
║  Detecting duplicates …  [███████░░░]  70%    843║
╚══════════════════════════════════════════════════╝

📊 Summary:
  Total pages scanned:     1,204
  Thin content (<300w):      147  🔴
  Duplicate content:          23  🟠
  Cannibalization issues:     12  🟠
  Missing meta desc:         302  🟡
  
🔴 Critical Issues (12):
┌──────────────────────────────┬─────────────────┬──────────┐
│ URL                          │ Issue           │ Action   │
├──────────────────────────────┼─────────────────┼──────────┤
│ /products/organic-treats     │ Cannibalization │ Merge    │
│ /blog/organic-dog-treats     │ Cannibalization │ Merge    │
│ /about-us                    │ Thin content    │ Expand   │
└──────────────────────────────┴─────────────────┴──────────┘
```

**Options:**
- `--scope full`: Crawl entire site (default: sample 100 pages)
- `--output md`: Export findings as Markdown report
- `--min-words`: Set thin content threshold (default: 300)

---

### `/technical-seo`

Comprehensive technical SEO audit covering crawl budget, Core Web Vitals, schema markup, and indexability.

**Usage:**

```bash
/technical-seo <domain> [--mobile-first] [--export pdf]
```

**Example:**

```bash
/technical-seo https://example.com --mobile-first
```

**Output:**

```
╔══════════════════════════════════════════════════╗
║  Technical SEO Audit  —  example.com             ║
╠══════════════════════════════════════════════════╣
║  Checking robots.txt …       [██████████] 100% ✓ ║
║  Validating sitemap …        [██████████] 100% ✓ ║
║  Measuring Core Web Vitals … [████████░░]  80%   ║
║  Scanning schema markup …    [██████░░░░]  60%   ║
╚══════════════════════════════════════════════════╝

🔴 Critical (3):
  • 147 pages blocked by robots.txt but in sitemap
  • LCP > 2.5s on 34% of pages
  • No SSL certificate on www subdomain

🟠 Warning (8):
  • 23 broken internal links
  • Missing canonical tags on 89 pages
  • CLS issues on mobile (0.15 avg)

🟢 Passed (12):
  ✓ HTTPS redirect working
  ✓ Schema markup present (Organization, Article)
  ✓ Mobile-friendly test passed
```

**Key Checks:**
- Crawlability (robots.txt, sitemap, internal links)
- Indexability (noindex tags, canonicals, redirects)
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Mobile usability

---

### `/competitor-gap`

Analyzes competitor backlink profiles, topic coverage, and featured snippet opportunities.

**Usage:**

```bash
/competitor-gap --domain <your-domain> --competitors <domain1,domain2,domain3>
```

**Example:**

```bash
/competitor-gap --domain example.com --competitors competitor1.com,competitor2.com
```

**Output:**

```
📊 Backlink Gap Analysis:

Competitor Backlinks Your Site Doesn't Have:
┌────────────────────────┬────────┬────┬──────────────┐
│ Domain                 │ Links  │ DR │ Opportunity  │
├────────────────────────┼────────┼────┼──────────────┤
│ healthline.com         │    45  │ 92 │  🔴 Critical │
│ petmd.com              │    32  │ 78 │  🔴 Critical │
│ akc.org                │    28  │ 85 │  🟠 High     │
└────────────────────────┴────────┴────┴──────────────┘

📝 Topic Gap Analysis:

Keywords Competitors Rank For (You Don't):
┌──────────────────────────┬────────┬────────────────┐
│ Keyword                  │ Volume │ Ranking Site   │
├──────────────────────────┼────────┼────────────────┤
│ dog food allergies       │ 12,100 │ competitor1.com│
│ puppy nutrition guide    │  8,900 │ competitor2.com│
│ senior dog diet          │  6,700 │ competitor1.com│
└──────────────────────────┴────────┴────────────────┘
```

**Options:**
- `--competitors`: Comma-separated list of up to 5 domains
- `--focus backlinks|topics|snippets`: Narrow analysis scope
- `--export csv`: Export prospect list

---

### `/content-brief`

Generates AI-powered SEO content brief with outline, NLP terms, word count targets, and internal linking suggestions.

**Usage:**

```bash
/content-brief "<target_keyword>" [--url <reference_url>] [--format markdown]
```

**Example:**

```bash
/content-brief "best organic dog food" --format markdown
```

**Output:**

```markdown
# Content Brief: Best Organic Dog Food

## Target Keyword
**Primary:** best organic dog food  
**Volume:** 8,100/mo | **Difficulty:** 38 | **Intent:** Commercial

## SEO Requirements
- **Target Word Count:** 2,400 - 2,800 words
- **Target Reading Level:** Grade 8-10
- **Content Type:** Comparison / Buyer's Guide

## Recommended Outline

### H1: Best Organic Dog Food in 2026 (Top 10 Brands Reviewed)

### H2: What Makes Dog Food "Organic"? (USDA Standards)
  - H3: Organic vs. Natural vs. Grain-Free
  - H3: USDA Certification Requirements

### H2: Top 10 Organic Dog Food Brands (Comparison Table)
  - H3: #1 - Brand A (Best Overall)
  - H3: #2 - Brand B (Best Budget)
  [... continue for all 10]

### H2: How to Choose Organic Dog Food (Buying Guide)
  - H3: Ingredient Quality Checklist
  - H3: Life Stage Considerations

### H2: FAQ
  - Is organic dog food worth the cost?
  - What certifications should I look for?

## NLP Terms to Include (TF-IDF Analysis)
- organic ingredients (12-15 mentions)
- USDA certified (8-10 mentions)
- grain-free (6-8 mentions)
- human-grade (5-7 mentions)
- limited ingredient (4-6 mentions)

## Internal Linking Opportunities
- Link to: /blog/dog-nutrition-guide (anchor: "nutritional requirements")
- Link to: /blog/reading-dog-food-labels (anchor: "ingredient label")
- Link from: /category/dog-food (anchor: "organic options")

## SERP Analysis
Current Top 10 avg word count: 2,654 words  
Images per article: 8-12  
Videos embedded: 40% of results  
Featured snippet: Comparison table (opportunity!)
```

---

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization recommendations.

**Usage:**

```bash
/serp-monitor --keywords <keyword_file.txt> [--frequency daily|weekly]
```

**Example:**

```bash
/serp-monitor --keywords my-keywords.txt --frequency daily
```

**Input file format** (`my-keywords.txt`):

```
organic dog food
best organic dog food
grain free organic dog food
```

**Output:**

```
📊 SERP Monitor Report — 2026-05-11

┌──────────────────────────┬──────────┬──────────┬────────┬──────────┐
│ Keyword                  │ Current  │ Previous │ Change │ Volatility│
├──────────────────────────┼──────────┼──────────┼────────┼──────────┤
│ organic dog food         │    #7    │    #8    │   ↑1   │  🟢 Low  │
│ best organic dog food    │    #12   │    #9    │   ↓3   │  🔴 High │
│ grain free organic       │    #5    │    #5    │   →0   │  🟢 Low  │
└──────────────────────────┴──────────┴──────────┴────────┴──────────┘

🔴 Alert: "best organic dog food" dropped 3 positions
   Likely cause: New competitor content published 2 days ago
   Action: Refresh content, add comparison table

💡 CTR Optimization Opportunities:
  • "organic dog food" — rank #7 but CTR only 2.1%
    Suggested title: Add year "... in 2026" + power word "Ultimate"
    Expected CTR lift: +40%
```

---

### `/link-prospecting`

Generates qualified backlink prospect lists with DA/DR filtering and outreach templates.

**Usage:**

```bash
/link-prospecting --topic "<topic>" --min-dr <score> [--export csv]
```

**Example:**

```bash
/link-prospecting --topic "organic pet food" --min-dr 30 --export csv
```

**Output:**

```
🔗 Link Prospecting Results — organic pet food

Found 47 prospects (DR 30+)

┌────────────────────────────┬────┬────┬──────────────┬─────────────┐
│ Domain                     │ DR │ DA │ Content Type │ Outreach    │
├────────────────────────────┼────┼────┼──────────────┼─────────────┤
│ pethealthnetwork.com       │ 68 │ 72 │ Resource pg  │ 🟢 Template1│
│ dogfoodadvisor.com         │ 75 │ 79 │ Review       │ 🟢 Template2│
│ thesprucepets.com          │ 82 │ 85 │ Roundup      │ 🔴 Template3│
└────────────────────────────┴────┴────┴──────────────┴─────────────┘

📧 Outreach Template #1 (Resource Page Addition):

Subject: Quick addition to your [TOPIC] resource page

Hi [NAME],

I noticed your excellent resource page on [URL]. I recently published
a comprehensive guide on [YOUR_TOPIC] that your readers might find valuable:

[YOUR_URL]

It covers [KEY_POINTS] with original data from [SOURCE].

Would you consider adding it to your resource list?

Thanks,
[YOUR_NAME]
```

**Saved to:** `link-prospects-organic-pet-food-2026-05-11.csv`

---

### `/page-speed-seo`

Diagnoses page speed issues (LCP, CLS, FID) and maps them to SEO ranking impact.

**Usage:**

```bash
/page-speed-seo <url> [--device mobile|desktop]
```

**Example:**

```bash
/page-speed-seo https://example.com/blog/post --device mobile
```

**Output:**

```
⚡ Page Speed SEO Audit — example.com/blog/post (Mobile)

Core Web Vitals:
┌─────────────────┬─────────┬─────────┬────────┬──────────────┐
│ Metric          │ Current │ Target  │ Status │ SEO Impact   │
├─────────────────┼─────────┼─────────┼────────┼──────────────┤
│ LCP             │  3.8s   │  <2.5s  │  🔴 Fail│ High (rank -5)│
│ FID             │  45ms   │ <100ms  │  🟢 Pass│ None         │
│ CLS             │  0.18   │  <0.1   │  🟠 Warn│ Medium (-2)  │
└─────────────────┴─────────┴─────────┴────────┴──────────────┘

🔴 Critical Issues:

1. LCP = 3.8s (Target: <2.5s)
   Cause: Hero image (2.4 MB) not optimized
   Fix:
   ```html
   <!-- Before -->
   <img src="hero.jpg" />
   
   <!-- After -->
   <img src="hero.webp" 
        width="1200" height="630"
        loading="eager"
        fetchpriority="high" />
   ```
   Expected improvement: 1.8s → 2.1s

2. Render-blocking CSS (1.2s delay)
   Files: main.css (450 KB), fonts.css (89 KB)
   Fix:
   ```html
   <link rel="preload" href="main.css" as="style" />
   <link rel="stylesheet" href="main.css" media="print" 
         onload="this.media='all'" />
   ```
   Expected improvement: -0.9s

Estimated Ranking Impact After Fixes: +3 positions
```

---

### `/local-seo`

Local SEO audit covering NAP consistency, Google Business Profile optimization, and citation building.

**Usage:**

```bash
/local-seo --business "<name>" --location "<city, state>"
```

**Example:**

```bash
/local-seo --business "Organic Paws Pet Store" --location "Austin, TX"
```

**Output:**

```
📍 Local SEO Audit — Organic Paws Pet Store, Austin, TX

NAP Consistency Check:
┌────────────────────────────┬─────────────────────┬────────┐
│ Platform                   │ NAP Match           │ Status │
├────────────────────────────┼─────────────────────┼────────┤
│ Google Business Profile    │ ✓ Exact match       │  🟢    │
│ Yelp                       │ ✗ Phone mismatch    │  🔴    │
│ Facebook                   │ ✓ Exact match       │  🟢    │
│ YellowPages                │ ✗ Address shortened │  🟠    │
└────────────────────────────┴─────────────────────┴────────┘

🔴 Fix NAP Issues:
  Yelp: Update phone from (512) 555-0100 to (512) 555-0199
  YellowPages: Expand "123 Main" to "123 Main Street, Suite 5"

Google Business Profile Score: 68/100

Missing Elements:
  🔴 No posts in last 30 days (-15 pts)
  🟠 Only 12 reviews (-8 pts)
  🟠 Missing "Products" section (-5 pts)
  🟡 No Q&A responses (-4 pts)

Citation Opportunities (Top 20):
  1. Bing Places (DR 95) — Not claimed
  2. Apple Maps (DR 90) — Not listed
  3. Better Business Bureau (DR 88) — Not listed
  [... continue]
```

---

### `/content-calendar`

Generates data-driven editorial calendar based on search demand seasonality and keyword opportunities.

**Usage:**

```bash
/content-calendar --topic "<niche>" --months <number> [--export google-sheets]
```

**Example:**

```bash
/content-calendar --topic "organic dog food" --months 6 --export google-sheets
```

**Output:**

```
📅 SEO Content Calendar — organic dog food (6 months)

May 2026:
┌────────┬──────────────────────────────┬────────┬──────────┬──────────┐
│ Date   │ Topic                        │ Volume │ Priority │ Type     │
├────────┼──────────────────────────────┼────────┼──────────┼──────────┤
│ May 5  │ Summer dog nutrition tips    │  8,900 │  🔴 High │ Guide    │
│ May 12 │ Organic treats for training  │  3,400 │  🟠 Med  │ Listicle │
│ May 19 │ Grain-free vs organic        │  5,600 │  🔴 High │ Compare  │
│ May 26 │ Memorial Day dog safety      │  2,100 │  🟡 Low  │ Seasonal │
└────────┴──────────────────────────────┴────────┴──────────┴──────────┘

June 2026 (Summer Spike — Volume +34%):
┌────────┬──────────────────────────────┬────────┬──────────┬──────────┐
│ Jun 2  │ Keeping dogs cool + hydration│ 12,300 │  🔴 High │ Guide    │
│ Jun 9  │ Raw organic dog food safety  │  6,700 │  🟠 Med  │ How-to   │
│ Jun 16 │ Best organic puppy food      │  9,800 │  🔴 High │ Review   │
│ Jun 23 │ Fourth of July dog treats    │  4,200 │  🟡 Low  │ Seasonal │
└────────┴──────────────────────────────┴────────┴──────────┴──────────┘

📊 Seasonality Insights:
  • June-August: +34% search volume (summer pet care peak)
  • October: +28% (holiday gifting starts)
  • January: +41% (New Year resolutions, diet changes)

Exported to: Google Sheets (link: https://docs.google.com/...)
```

---

## Workflows (Multi-Step)

### `full-seo-sprint`

12-step comprehensive SEO sprint from audit to implementation.

**Usage:**

```bash
/workflows:full-seo-sprint <domain> --scope full
```

**Steps:**

```
╔══════════════════════════════════════════════════╗
║  Full SEO Sprint  —  example.com                 ║
╠══════════════════════════════════════════════════╣
║  Step  1/12: Technical audit         [██████████]║
║  Step  2/12: Content audit            [████████░░]║
║  Step  3/12: Keyword research         [████░░░░░░]║
║  Step  4/12: Competitor gap           [░░░░░░░░░░]║
║  Step  5/12: Backlink analysis        [░░░░░░░░░░]║
║  Step  6/12: On-page optimization     [░░░░░░░░░░]║
║  Step  7/12: Content brief generation [░░░░░░░░░░]║
║  Step  8/12: Internal linking map     [░░░░░░░░░░]║
║  Step  9/12: Schema implementation    [░░░░░░░░░░]║
║  Step 10/12: Page speed fixes         [░░░░░░░░░░]║
║  Step 11/12: Content calendar         [░░░░░░░░░░]║
║  Step 12/12: Monitoring setup         [░░░░░░░░░░]║
╚══════════════════════════════════════════════════╝
```

**Final Deliverables:**
- Technical SEO audit report (PDF)
- Prioritized fix list with time estimates
- Keyword map (500+ terms clustered by intent)
- Content calendar (12 months)
- Schema markup templates
- Monitoring dashboard config

---

### `launch-seo`

Pre-launch SEO validation checklist.

**Usage:**

```bash
/workflows:launch-seo <domain>
```

**Checks:**

1. ✅ Robots.txt allows crawling of key pages
2. ✅ XML sitemap submitted to GSC
3. ✅ Canonical tags point to correct URLs
4. ✅ Hreflang tags (if multi-language)
5. ✅ 301 redirects from old URLs
6. ✅ HTTPS enabled + redirect working
7. ✅ Structured data validates
8. ✅ Page titles unique + optimized
9. ✅ Meta descriptions under 160 chars
10. ✅ Core Web Vitals pass
11. ✅ Mobile-friendly test pass
12. ✅ Google Analytics + GSC connected

**Output:**

```
🚀 Launch SEO Checklist — example.com

✅ Passed (10/12):
  ✓ Robots.txt configured correctly
  ✓ Sitemap submitted and indexed
  ✓ Canonical tags implemented
  ✓ HTTPS enabled
  [... continue]

🔴 Failed (2):
  ✗ Hreflang tags missing for /es/ pages
  ✗ LCP > 2.5s on 3 landing pages

⚠️  Do Not Launch Until:
  1. Add hreflang tags (est. 2 hours)
  2. Optimize LCP on /products, /about, /contact (est. 4 hours)

Re-run: /workflows:launch-seo example.com
```

---

### `content-refresh`

Identifies and refreshes underperforming content to recover rankings.

**Usage:**

```bash
/workflows:content-refresh --url <domain> [--min-drop 3]
```

**Process:**

1. Identify pages that dropped ≥3 positions in last 90 days
2. Analyze why (outdated info, new competitors, technical issues)
3. Generate refresh checklist for each page
4. Prioritize by potential traffic recovery

**Example Output:**

```
🔄 Content Refresh Workflow — example.com

Pages Identified: 23 (dropped ≥3 positions)

Top Priority (Est. 1,200 visits/mo recovery):

📄 /blog/organic-dog-food-brands
   Current rank: #9 (was #3)
   Keyword: "organic dog food brands" (8,100/mo)
   Drop date: 2026-03-15
   
   Why it dropped:
   • Content last updated 2024-11-12 (18 months old)
   • Competitor published updated 2026 guide
   • Missing 3 new brands launched in 2025
   
   Refresh Checklist:
   ✅ Update publish date
   ✅ Add 3 new brands (with images + reviews)
   ✅ Refresh statistics (2026 data)
   ✅ Add comparison table
   ✅ Update FAQ section
   ✅ Re-optimize meta description
   ✅ Add internal links to new content
   
   Est. time: 3 hours
   Expected rank recovery: #3-4 (800 visits/mo)
```

---

### `authority-building`

End-to-end digital PR and link-building campaign workflow.

**Usage:**

```bash
/workflows:authority-building --domain <domain> --goal <number_links>
```

**Stages:**

```
1. Asset Audit
   → Identify linkable assets (data, tools, guides)
   
2. Content Gap Analysis
   → Find link-worthy topics you're missing
   
3. Prospect Research
   → Build list of 200+ DR 30+ targets
   
4. Outreach Campaign
   → Personalized templates for each segment
   
5. Relationship Building
   → Follow-up sequence + value-add touches
   
6. Link Monitoring
   → Track acquired links + anchor text distribution
```

---

### `ai-content-pipeline`

Automated keyword → brief → draft → optimize → publish pipeline.

**Usage:**

```bash
/workflows:ai-content-pipeline --topic "<topic>" --count <number>
```

**Example:**

```bash
/workflows:ai-content-pipeline --topic "organic dog treats" --count 5
```

**Pipeline Steps:**

```
For each article:

1. Keyword Research
   → Extract 1 primary + 3-5 secondary keywords
   
2. Content Brief Generation
   → Outline, NLP terms, word count target
   
3. AI Draft Creation
   → Generate 2,000-3,000 word draft
   
4. SEO Optimization
   → Add internal links, optimize headings, meta tags
   
5. Quality Check
   → Readability score, plagiarism check, fact verification
   
6. Publish + Index
   → Publish to CMS, submit to GSC for indexing
```

**Output:**

```
🤖 AI Content Pipeline — organic dog treats (5 articles)

Article 1/5: "Best Organic Dog Treats in 2026"
  ✅ Keywords identified
  ✅ Brief generated (2,600 word target)
  ⏳ Draft in progress (1,847/2,600 words)
  ⏸  Optimization pending
  ⏸  Publishing pending

Article 2/5: "Grain-Free vs Organic Dog Treats"
  ✅ Keywords identified
  ⏳ Brief in progress
  ...
```

---

## Configuration

### Environment Variables

Create `.env` in your project root:

```bash
# SEO Tool API Keys
SEMRUSH_API_KEY=your_semrush_key
AHREFS_API_KEY=your_ahrefs_key
MOZ_API_KEY=your_moz_key

# Google Services
GOOGLE_SEARCH_CONSOLE_CLIENT_ID=your_client_id
GOOGLE_SEARCH_CONSOLE_CLIENT_SECRET=your_client_secret
GOOGLE_ANALYTICS_VIEW_ID=your_view_id

# Page Speed APIs
PAGESPEED_INSIGHTS_API_KEY=your_psi_key
WEBPAGETEST_API_KEY=your_wpt_key

# Optional Integrations
SCREAMING_FROG_
