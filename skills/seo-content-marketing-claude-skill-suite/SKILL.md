---
name: seo-content-marketing-claude-skill-suite
description: SEO & content marketing automation suite with keyword research, content audits, technical SEO, SERP analysis and AI content workflows
triggers:
  - "analyze SEO performance for this site"
  - "generate keyword research and content brief"
  - "audit technical SEO and page speed issues"
  - "find competitor content gaps and backlink opportunities"
  - "create data-driven content calendar"
  - "check Core Web Vitals and crawl budget"
  - "build AI content pipeline from keyword to publish"
  - "run full SEO audit and action plan"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

**r06-alirezarezvani-claude-code-tresor-seo** is a comprehensive SEO and content marketing automation suite that provides 10 specialized commands and 5 multi-step workflows for keyword research, content audits, technical SEO analysis, SERP monitoring, and AI-powered content creation. Derived from [alirezarezvani/claude-code-tresor](https://github.com/alirezarezvani/claude-code-tresor), it delivers structured, actionable insights with visual progress tracking and prioritized action plans.

---

## Installation

### Basic Setup

```bash
# Clone the repository
git clone https://github.com/LairLightningDerrick/r06-alirezarezvani-claude-code-tresor-seo.git

# Navigate to directory
cd r06-alirezarezvani-claude-code-tresor-seo

# Copy to Claude skills directory
mkdir -p ~/.claude/skills/
cp -r . ~/.claude/skills/seo-content-marketing/
```

### Register with Claude Code

In a Claude Code session:

```bash
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

Or manually load the skill context:

```bash
# From project root
cat SKILL.md | pbcopy  # macOS
# Then paste into Claude Code conversation
```

---

## Core Commands

### 1. Keyword Research

Deep keyword clustering with SERP intent mapping and opportunity scoring.

```bash
/keyword-research "ecommerce analytics tools"
```

**Options:**

```bash
/keyword-research <target> \
  --country us \
  --language en \
  --volume-min 100 \
  --difficulty-max 60 \
  --intent commercial \
  --output json
```

**Output structure:**

- Keyword clusters by topic
- Search volume + trend data
- Keyword difficulty (0-100)
- SERP intent classification (informational/commercial/transactional/navigational)
- Opportunity score (volume ÷ difficulty)
- Related questions and "People Also Ask"

**Example response:**

```
┌─────────────────────────┬────────┬────────┬──────────────┬─────────┐
│ Keyword                 │ Volume │ Diff.  │ Intent       │ Opp.    │
├─────────────────────────┼────────┼────────┼──────────────┼─────────┤
│ ecommerce analytics     │  8 100 │     45 │ Commercial   │  180.0  │
│ shopify analytics       │  4 400 │     38 │ Commercial   │  115.8  │
│ google analytics setup  │  2 900 │     52 │ Informational│   55.8  │
│ conversion tracking     │  1 600 │     41 │ Commercial   │   39.0  │
└─────────────────────────┴────────┴────────┴──────────────┴─────────┘
```

---

### 2. Content Audit

Full-site content quality analysis with duplication and cannibalization detection.

```bash
/content-audit --scope full --output md
```

**Options:**

```bash
/content-audit \
  --scope full|section|url \
  --target /blog \
  --min-words 300 \
  --check-duplicates \
  --check-cannibalization \
  --export csv
```

**Checks:**

- Missing/duplicate title tags and meta descriptions
- Thin content (word count < threshold)
- Keyword cannibalization (multiple pages targeting same keyword)
- Content freshness (last updated date)
- Internal linking structure
- Image alt text coverage
- Readability scores

**Example findings:**

```
🔴 CRITICAL (12 pages)
  - 8 pages with duplicate title tags
  - 4 pages with <300 words

🟠 HIGH PRIORITY (34 pages)
  - 21 pages missing meta description
  - 13 pages with cannibalization issues

🟡 MEDIUM (67 pages)
  - 45 pages not updated in 12+ months
  - 22 pages with low internal link count
```

---

### 3. Technical SEO Audit

Crawl budget, Core Web Vitals, schema markup, and indexability analysis.

```bash
/technical-seo
```

**Options:**

```bash
/technical-seo \
  --url https://example.com \
  --user-agent googlebot \
  --check-vitals \
  --check-schema \
  --check-mobile \
  --depth 5
```

**Audits:**

- Robots.txt and sitemap.xml validation
- HTTP status codes (301/302/404/500)
- Canonical tag implementation
- Hreflang for multilingual sites
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation (JSON-LD)
- Mobile-friendliness
- HTTPS implementation
- Page speed (FCP, TTI, TBT)

**Sample output:**

```json
{
  "core_web_vitals": {
    "lcp": { "value": 2.1, "rating": "good", "threshold": 2.5 },
    "fid": { "value": 85, "rating": "good", "threshold": 100 },
    "cls": { "value": 0.08, "rating": "good", "threshold": 0.1 }
  },
  "indexability": {
    "total_pages": 1204,
    "indexable": 1180,
    "blocked_robots": 18,
    "noindex": 6
  },
  "schema_coverage": {
    "pages_with_schema": 892,
    "valid_schema": 874,
    "errors": 18
  }
}
```

---

### 4. Competitor Gap Analysis

Backlink gap, topic gap, and featured snippet opportunities.

```bash
/competitor-gap https://example.com --competitors competitor1.com,competitor2.com
```

**Analysis includes:**

- Keywords competitors rank for (that you don't)
- Backlink sources competitors have (that you don't)
- Featured snippet opportunities
- Content topics with high competitor coverage
- Domain authority comparison

**Example gap report:**

```
KEYWORD GAP (Top 20 opportunities)
┌──────────────────────────┬─────────┬──────────────┐
│ Keyword                  │ Volume  │ Comp. Rank   │
├──────────────────────────┼─────────┼──────────────┤
│ best crm for startups    │   3 600 │ #3, #5       │
│ sales automation tools   │   2 900 │ #4, #7       │
│ pipeline management      │   1 800 │ #2, #6       │
└──────────────────────────┴─────────┴──────────────┘

BACKLINK GAP (High-authority domains)
  - techcrunch.com (DR 93) → competitor1.com
  - forbes.com (DR 94) → competitor1.com, competitor2.com
  - entrepreneur.com (DR 91) → competitor2.com
```

---

### 5. Content Brief Generation

AI-generated SEO content brief with outline, NLP terms, and word count targets.

```bash
/content-brief "how to choose crm software"
```

**Options:**

```bash
/content-brief <keyword> \
  --intent informational \
  --serp-analysis \
  --include-questions \
  --include-nlp-terms \
  --target-length 2000
```

**Brief includes:**

- Primary and secondary keywords
- Recommended word count (based on top 10 SERP)
- Content outline (H2/H3 structure from top-ranking pages)
- NLP terms and entities to include
- Questions to answer
- Internal linking suggestions
- Media recommendations (images, videos, charts)

**Sample brief:**

```markdown
# Content Brief: How to Choose CRM Software

## Target Keyword
- Primary: how to choose crm software (1,900/mo, KD 42)
- Secondary: best crm for small business, crm selection criteria

## Recommended Length
2,200 words (avg. of top 10: 2,187 words)

## Outline (from SERP analysis)
1. What is CRM Software?
2. Key Features to Look For
   - Contact management
   - Sales pipeline tracking
   - Integration capabilities
   - Mobile access
3. How to Evaluate CRM Options
   - Define your requirements
   - Consider team size
   - Budget planning
4. Top CRM Solutions Comparison
5. Implementation Best Practices

## NLP Terms to Include
contact management, sales pipeline, lead tracking, customer data,
integration, automation, reporting, mobile app, cloud-based

## Questions to Answer
- What are the must-have CRM features?
- How much should I budget for CRM?
- What's the difference between cloud and on-premise CRM?
```

---

### 6. SERP Monitor

Daily rank tracking with volatility alerts and CTR optimization tips.

```bash
/serp-monitor --keywords keywords.csv --frequency daily
```

**Configuration:**

```bash
# Create keywords.csv
keyword,target_url,location
"ecommerce analytics","/products/analytics","United States"
"shopify reporting","/blog/shopify-guide","United States"
```

**Tracking output:**

```
┌────────────────────────┬──────┬──────┬────────┬──────────┐
│ Keyword                │ Rank │ Prev │ Change │ CTR Est. │
├────────────────────────┼──────┼──────┼────────┼──────────┤
│ ecommerce analytics    │    4 │    6 │ ▲ +2   │  8.2%    │
│ shopify reporting      │    8 │    7 │ ▼ -1   │  3.1%    │
│ conversion tracking    │   12 │   11 │ ▼ -1   │  1.8%    │
└────────────────────────┴──────┴──────┴────────┴──────────┘

🎯 CTR Optimization Tips:
  - "ecommerce analytics" (#4): Test year in title (2024/2025)
  - "shopify reporting" (#8): Add power word to meta description
```

---

### 7. Link Prospecting

Quality backlink prospect discovery with outreach templates.

```bash
/link-prospecting --topic "marketing automation" --min-dr 40
```

**Filters:**

```bash
/link-prospecting \
  --topic "marketing automation" \
  --min-dr 40 \
  --max-dr 80 \
  --link-type guest-post|resource-page|broken-link \
  --country us \
  --limit 100
```

**Prospect output:**

```csv
domain,dr,topic_relevance,contact_email,outreach_angle
marketingland.com,78,0.92,editor@marketingland.com,guest-post
contentmarketinginstitute.com,76,0.89,submissions@cmi.com,expert-roundup
hubspot.com/marketing,94,0.87,blog@hubspot.com,data-study
```

**Included outreach templates:**

- Guest post pitch
- Resource page addition
- Broken link replacement
- Expert quote/roundup
- Data/study promotion

---

### 8. Page Speed SEO Analysis

Render-blocking resource analysis, Core Web Vitals diagnosis, and ranking impact assessment.

```bash
/page-speed-seo https://example.com/page
```

**Metrics analyzed:**

```
Performance Score: 78/100

Core Web Vitals:
  LCP: 2.8s  ⚠ (target: <2.5s)  -3 ranking positions est.
  FID: 120ms ⚠ (target: <100ms) -1 ranking positions est.
  CLS: 0.05  ✓ (target: <0.1)

Render-blocking resources (8):
  - /css/main.css (124 KB) → defer non-critical CSS
  - /js/analytics.js (87 KB) → async load
  - /js/jquery.min.js (91 KB) → consider removal

Optimization recommendations:
  1. Enable text compression (saves 312 KB)
  2. Serve images in WebP format (saves 890 KB)
  3. Eliminate render-blocking resources (saves 1.2s LCP)
  4. Reduce JavaScript execution time (saves 840ms)
```

---

### 9. Local SEO Audit

NAP consistency, Google Business Profile optimization, and citation analysis.

```bash
/local-seo --business "Coffee Shop Name" --location "Seattle, WA"
```

**Checks:**

- NAP (Name, Address, Phone) consistency across directories
- Google Business Profile completeness
- Local citation count and quality
- Schema markup (LocalBusiness)
- Review count and rating
- Local keyword rankings
- Competitor local pack analysis

**Sample report:**

```
Google Business Profile Score: 72/100
  ✓ Business name, address, phone
  ✓ Business hours
  ✓ Categories (primary + 4 secondary)
  ⚠ Only 12 photos (recommend 25+)
  ⚠ Missing Q&A content
  ✗ No posts in last 30 days

NAP Consistency: 84% (42/50 citations)
  Issues found on:
    - yelp.com (wrong phone number)
    - yellowpages.com (old address)

Citation Opportunities (15 high-priority):
  - Apple Maps
  - Bing Places
  - Better Business Bureau
```

---

### 10. Content Calendar Generation

Data-driven editorial calendar from search demand and seasonality.

```bash
/content-calendar --topics topics.txt --months 3
```

**Input format (topics.txt):**

```
email marketing tips
marketing automation workflows
lead generation strategies
social media marketing
```

**Output:**

```
┌────────┬─────────────────────────────┬────────┬──────────┬──────────┐
│ Date   │ Topic                       │ Volume │ Trend    │ Intent   │
├────────┼─────────────────────────────┼────────┼──────────┼──────────┤
│ Jun 5  │ Summer email campaigns      │  3 200 │ ▲ +45%   │ Info     │
│ Jun 12 │ Marketing automation ROI    │  1 800 │ ▲ +12%   │ Comm     │
│ Jun 19 │ Lead magnet ideas           │  2 400 │ → stable │ Info     │
│ Jun 26 │ Social media calendar tools │  1 600 │ ▲ +8%    │ Comm     │
└────────┴─────────────────────────────┴────────┴──────────┴──────────┘

Seasonality insights:
  - "email marketing" peaks in January, September
  - "social media marketing" steady demand year-round
  - "lead generation" spikes Q4 (budget planning season)
```

---

## Multi-Step Workflows

### Full SEO Sprint

Complete 12-step SEO audit and action plan.

```bash
/workflow:full-seo-sprint https://example.com --scope full
```

**Steps:**

1. Technical SEO audit (crawl, indexability, speed)
2. On-page SEO analysis (titles, meta, headings)
3. Content quality audit
4. Keyword research and mapping
5. Competitor gap analysis
6. Backlink profile audit
7. Local SEO check (if applicable)
8. Core Web Vitals measurement
9. Mobile-friendliness test
10. Schema markup validation
11. Prioritized action plan generation
12. Quick wins vs. long-term roadmap

**Progress tracking:**

```
╔══════════════════════════════════════════════════╗
║  Full SEO Sprint — example.com                   ║
╠══════════════════════════════════════════════════╣
║  Step  1/12: Technical audit     [████████████] ✓║
║  Step  2/12: On-page analysis    [████████████] ✓║
║  Step  3/12: Content audit       [████████░░░░] 75%║
║  Step  4/12: Keyword research    [░░░░░░░░░░░░]  0%║
╚══════════════════════════════════════════════════╝
```

---

### Launch SEO Workflow

Pre-launch SEO validation checklist.

```bash
/workflow:launch-seo https://staging.example.com
```

**Validation steps:**

```
□ Robots.txt configured (disallow staging, allow production)
□ Sitemap.xml generated and submitted
□ Canonical tags pointing to production URLs
□ Hreflang tags (if multilingual)
□ 301 redirects mapped (if migration)
□ Google Analytics + Search Console installed
□ Core Web Vitals passing
□ Mobile-friendly test passed
□ SSL certificate installed
□ Structured data implemented and validated
□ Meta tags (title, description) on all pages
□ Image alt text present
□ No broken links (internal/external)
□ Page speed optimized (score >80)
□ Social meta tags (OG, Twitter Cards)
```

---

### Content Refresh Workflow

Identify and optimize underperforming content.

```bash
/workflow:content-refresh --age 180 --traffic-drop 30
```

**Process:**

1. Identify pages with traffic drop >30% in last 6 months
2. Analyze SERP changes (new competitors, intent shift)
3. Check for keyword cannibalization
4. Review content freshness and accuracy
5. Generate refresh recommendations:
   - Update statistics and examples
   - Add new sections based on SERP analysis
   - Improve internal linking
   - Optimize title/meta for CTR
   - Add or update media
6. Track post-refresh performance

**Sample refresh plan:**

```markdown
## Page: /blog/email-marketing-tips (published 2022-03-15)

Traffic: 2,400/mo → 1,100/mo (-54%)
Ranking: #3 → #8 for "email marketing tips"

SERP Changes:
  - 3 new comprehensive guides published in 2024
  - Average word count increased from 1,800 to 2,600
  - Video content now appearing in 6/10 results

Refresh Actions:
  1. Expand from 1,850 to 2,800 words
  2. Add 2024 statistics and examples
  3. Create embedded video (tips overview)
  4. Add sections:
     - AI-powered email personalization
     - Privacy regulations (iOS Mail Privacy Protection)
  5. Update title: "17 Email Marketing Tips" → "21 Email Marketing Tips That Work in 2024"
  6. Improve internal links to related guides
```

---

### Authority Building Workflow

End-to-end digital PR and link-building campaign.

```bash
/workflow:authority-building --niche "B2B SaaS" --duration 90
```

**Campaign structure:**

```
Week 1-2: Research & Planning
  □ Identify high-authority targets (DR 60+)
  □ Analyze competitor backlink profiles
  □ Create data/research asset
  □ Design visual assets (infographics, charts)

Week 3-6: Content Creation
  □ Publish original research/study
  □ Create expert roundup content
  □ Develop comprehensive guides
  □ Guest post pitches (10 targets)

Week 7-10: Outreach
  □ Personalized outreach (50 contacts)
  □ Follow-up sequence (3 emails)
  □ Track responses and placements
  □ Monitor backlink acquisition

Week 11-12: Amplification
  □ Promote placements on social
  □ Email existing network
  □ Repurpose content (LinkedIn, Medium)
  □ Measure DR/traffic impact
```

---

### AI Content Pipeline

Automated keyword-to-publish content workflow.

```bash
/workflow:ai-content-pipeline --input keywords.csv --auto-publish false
```

**Pipeline stages:**

```
Input: keywords.csv
  ↓
① Keyword Analysis
  - Search volume, difficulty, intent
  - SERP analysis for top 10
  ↓
② Brief Generation
  - Outline from top-ranking content
  - NLP terms and entities
  - Word count target
  ↓
③ AI Draft Creation
  - Generate content sections
  - Include statistics and examples
  - Optimize for target keywords
  ↓
④ SEO Optimization
  - Title tag (55-60 chars, keyword-front)
  - Meta description (150-160 chars, CTA)
  - Heading structure (H1, H2, H3)
  - Internal linking suggestions
  - Image alt text generation
  ↓
⑤ Quality Check
  - Plagiarism scan
  - Readability score (Flesch-Kincaid)
  - Keyword density check
  - Fact-checking pass
  ↓
⑥ Review & Publish
  - Human review (if --auto-publish false)
  - Schedule publication
  - Submit to Search Console
  - Track initial rankings
```

**Configuration:**

```bash
# config/ai-pipeline.yaml
model: gpt-4
temperature: 0.7
max_tokens: 3000
auto_publish: false
human_review_required: true
plagiarism_threshold: 15
readability_target: 60  # Flesch Reading Ease
keyword_density_max: 2.5
```

---

## Configuration

### Environment Variables

```bash
# Required for data providers
export SEMRUSH_API_KEY=your_semrush_key
export AHREFS_API_KEY=your_ahrefs_key
export SERPAPI_KEY=your_serpapi_key

# Optional for enhanced features
export OPENAI_API_KEY=your_openai_key
export GOOGLE_SEARCH_CONSOLE_CLIENT_ID=your_gsc_client_id
export GOOGLE_SEARCH_CONSOLE_CLIENT_SECRET=your_gsc_secret
export SCREAMING_FROG_LICENSE=your_sf_license
```

### Configuration File

Create `~/.seo-skills/config.yaml`:

```yaml
# Default settings
defaults:
  country: us
  language: en
  output_format: markdown
  progress_display: true

# Data sources (priority order)
data_sources:
  keyword_data: semrush  # or ahrefs, google_keyword_planner
  backlink_data: ahrefs  # or semrush, moz
  serp_data: serpapi     # or semrush, ahrefs

# Command-specific settings
keyword_research:
  min_volume: 100
  max_difficulty: 60
  cluster_threshold: 0.7

content_audit:
  min_word_count: 300
  freshness_threshold_days: 365
  cannibalization_similarity: 0.85

technical_seo:
  crawl_depth: 5
  user_agent: googlebot
  respect_robots_txt: true
  follow_redirects: true

# Workflow settings
workflows:
  full_seo_sprint:
    include_local_seo: auto  # auto, true, false
    parallel_execution: true
  
  ai_content_pipeline:
    auto_publish: false
    require_human_review: true
    plagiarism_check: true
```

---

## Common Usage Patterns

### Pattern 1: New Site SEO Setup

```bash
# Step 1: Technical foundation
/technical-seo --url https://newsite.com

# Step 2: Keyword research
/keyword-research "main product category" --export csv

# Step 3: Create content calendar
/content-calendar --topics keywords.csv --months 6

# Step 4: Generate first batch of briefs
for keyword in $(cat priority-keywords.txt); do
  /content-brief "$keyword" --output md
done
```

### Pattern 2: Competitor Analysis Deep-Dive

```bash
# Identify gaps
/competitor-gap https://yoursite.com \
  --competitors comp1.com,comp2.com,comp3.com \
  --export json

# Extract specific opportunities
jq '.keyword_gap[] | select(.volume > 1000 and .difficulty < 50)' gaps.json

# Create content plan from gaps
/content-calendar --keywords gap-keywords.csv --months 3
```

### Pattern 3: Content Recovery

```bash
# Find underperforming content
/content-audit --scope full --min-traffic-drop 30

# Analyze specific pages
for url in $(cat underperforming.txt); do
  /serp-monitor --url "$url" --analyze-changes
done

# Generate refresh plans
/workflow:content-refresh --urls underperforming.txt --batch 10
```

### Pattern 4: Monthly SEO Reporting

```bash
#!/bin/bash
# monthly-seo-report.sh

DOMAIN="example.com"
REPORT_DIR="reports/$(date +%Y-%m)"
mkdir -p "$REPORT_DIR"

# Technical health
/technical-seo --url https://$DOMAIN > "$REPORT_DIR/technical.md"

# Rankings
/serp-monitor --keywords tracked.csv > "$REPORT_DIR/rankings.csv"

# Content performance
/content-audit --scope full > "$REPORT_DIR/content-audit.md"

# Backlink growth
/competitor-gap https://$DOMAIN \
  --focus backlinks \
  > "$REPORT_DIR/backlinks.md"

# Combine into single report
cat "$REPORT_DIR"/*.md > "$REPORT_DIR/monthly-report-$(date +%Y-%m).md"
```

---

## Troubleshooting

### Issue: "API rate limit exceeded"

**Cause:** Too many requests to SEO data provider (SEMrush, Ahrefs, etc.)

**Solution:**

```bash
# Check current rate limits
/debug:rate-limits

# Enable request caching
export SEO_SKILLS_CACHE_ENABLED=true
export SEO_SKILLS_CACHE_TTL=86400  # 24 hours

# Or reduce parallel requests
/keyword-research <target> --parallel 1
```

### Issue: "Crawl blocked by robots.txt"

**Cause:** Target site blocks automated crawlers

**Solution:**

```bash
# Check robots.txt
curl https://example.com/robots.txt

# Use different user agent
/technical-seo --url https://example.com --user-agent "Mozilla/5.0"

# Or skip crawl and use API data only
/technical-seo --url https://example.com --mode api-only
```

### Issue: "Schema validation errors"

**Cause:** Invalid JSON-LD structured data

**Solution:**

```bash
# Validate specific schema
/technical-seo --url https://example.com/page \
  --check-schema \
  --schema-type Article \
  --verbose

# Get detailed error messages
{
  "errors": [
    {
      "type": "MISSING_REQUIRED_FIELD",
      "field": "datePublished",
      "message": "Article schema requires datePublished"
    }
  ],
  "fix": "Add <meta property=\"article:published_time\" content=\"2024-01-15\">"
}
```

### Issue: "Keyword data unavailable"

**Cause:** API key missing or keyword not in database

**Solution:**

```bash
# Verify API configuration
/debug:check-apis

# Fall back to alternative data source
/keyword-research "target keyword" --source google_keyword_planner

# Or use approximate data
/keyword-research "target keyword" --allow-estimates
```

### Issue: "Content brief generation fails"

**Cause:** Insufficient SERP data or API errors

**Solution:**

```bash
# Retry with manual SERP analysis
/content-brief "keyword" --serp-manual --top 5

# Or provide custom outline
/content-brief "keyword" --outline custom-outline.md

# Skip SERP analysis entirely
/content-brief "keyword" --skip-serp-analysis
```

---

## Advanced Features

### Custom Scoring Algorithms

Override default opportunity scoring:

```yaml
# config/scoring.yaml
keyword_opportunity:
  formula: "(volume / difficulty) * intent_multiplier"
  intent_multiplier:
    transactional: 1.5
    commercial: 1.2
    informational: 1.0
    navigational: 0.5

content_quality:
  weights:
    word_count: 0.2
    readability: 0.15
    internal_links: 0.15
    external_links: 0.1
    freshness: 0.2
    engagement: 0.2
```

### Webhook Integration

Send results to external systems:

```bash
# config/webhooks.yaml
webhooks:
  - event: audit_complete
    url: https://your-system.com/api/seo-audit
    headers:
      Authorization: "Bearer ${WEBHOOK_TOKEN}"
    
  - event: ranking_change
    url: https://slack.com/api/webhook/${SLACK_WEBHOOK_ID}
    condition: "abs(rank_change) >= 3"
```

### CI/CD Integration

```yaml
# .github/workflows/seo-checks.yml
name: SEO Quality Checks

on:
  pull_request:
    paths:
      - 'content/**'
      - 'pages/**'

jobs:
  seo-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install SEO Skills
        run: |
          git clone https://github.com/LairLightningDerrick/r06-alirezarezvani-claude-code-tresor-seo.git
          cd r06-alirezarezvani-claude-code-tresor-seo
          npm install  # if applicable
      
      - name: Check meta tags
        run: |
          /technical-seo --url ${{ github.event.pull_request.head.repo.html_url }} \
            --check-meta \
            --fail-on-error
      
      - name: Validate schema
        run: |
          /technical-seo --url ${{ github.event.pull_request.head.repo.html_url }} \
            --check-schema \
            --fail-on-error
```

---

## Integration Examples

### Export to Google Sheets

```bash
# Install gsheets dependency
pip install gspread oauth2client

# Export keyword research
/keyword-research "topic" --export gsheets --sheet "SEO Research"
```

### Import from Google Search Console

```bash
# Authenticate
/integrations:gsc-auth

# Import performance data
/integrations:gsc-import \
  --property https://example.com \
  --start-date 2024-01-01 \
  --end-date 2024-03-31 \
  --dimension page \
  --output csv
```

### Notion Database Sync

```yaml
# config/integrations.yaml
notion:
  token: ${NOTION_TOKEN}
