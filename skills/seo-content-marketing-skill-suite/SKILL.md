---
name: seo-content-marketing-skill-suite
description: SEO & content marketing automation commands for keyword research, content audits, technical SEO, competitor analysis, and content strategy workflows
triggers:
  - "help me with SEO keyword research"
  - "audit this site for SEO issues"
  - "analyze competitor content gaps"
  - "generate an SEO content brief"
  - "check technical SEO problems"
  - "create a content calendar"
  - "find backlink opportunities"
  - "optimize page speed for SEO"
---

# 📈 SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Derived from [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), this skill suite provides 10 specialized SEO and content marketing commands plus 5 multi-step workflows with structured output and visual progress tracking.

## What This Project Does

A command-line skill suite for AI coding agents that automates:

- **Keyword research** with clustering and SERP intent mapping
- **Content audits** with quality scoring and cannibalization detection
- **Technical SEO** analysis (Core Web Vitals, schema, indexability)
- **Competitor gap analysis** (backlinks, topics, featured snippets)
- **Content brief generation** with NLP terms and word targets
- **SERP monitoring** with rank tracking and CTR optimization
- **Link prospecting** with DA/DR filtering and outreach templates
- **Page speed SEO** diagnosis mapped to ranking impact
- **Local SEO** audits (NAP consistency, GBP optimization)
- **Content calendar** generation from search demand data

All commands use a consistent 5-step interaction pattern with visual progress bars, findings tables, prioritized action plans, and next-step suggestions.

## Installation

### Clone the Skill

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Clone into skills directory
git clone https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo.git \
  ~/.claude/skills/seo-content-marketing-skill-suite

# Or copy manually
cp -r /path/to/this/repo ~/.claude/skills/seo-content-marketing-skill-suite/
```

### Register with Claude Code

In a Claude Code session:

```bash
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md
```

Or add to your project's `.claude/config.json`:

```json
{
  "skills": [
    "~/.claude/skills/seo-content-marketing-skill-suite"
  ]
}
```

## Core Commands

### `/keyword-research`

Deep keyword clustering with opportunity scoring and SERP intent mapping.

**Usage:**

```bash
# Basic keyword research
/keyword-research "cloud storage solutions"

# With filters
/keyword-research "saas tools" --volume-min 500 --difficulty-max 60

# Export to CSV
/keyword-research "marketing automation" --output csv --file keywords.csv
```

**Options:**
- `--volume-min <number>` — Minimum monthly search volume
- `--volume-max <number>` — Maximum monthly search volume
- `--difficulty-min <number>` — Minimum keyword difficulty (0-100)
- `--difficulty-max <number>` — Maximum keyword difficulty (0-100)
- `--intent <type>` — Filter by intent: informational, commercial, transactional, navigational
- `--cluster` — Group keywords into thematic clusters
- `--output <format>` — Output format: table (default), csv, json, md

**Output Example:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "cloud storage solutions"  ║
╠══════════════════════════════════════════════════╣
║  Fetching seed keywords …     [██████████] 100% ✓║
║  Analyzing SERP intent …      [██████████] 100% ✓║
║  Clustering keywords …        [██████████] 100% ✓║
║  Calculating opportunity …    [██████████] 100% ✓║
╚══════════════════════════════════════════════════╝

┌────────────────────────────┬────────┬────────┬──────────────┬──────────┐
│ Keyword                    │ Volume │ Diff   │ Intent       │ Opp Score│
├────────────────────────────┼────────┼────────┼──────────────┼──────────┤
│ cloud storage solutions    │ 12 400 │     58 │ Commercial   │    🟢 82 │
│ best cloud storage 2026    │  8 900 │     62 │ Commercial   │    🟢 78 │
│ cloud storage comparison   │  4 200 │     54 │ Commercial   │    🟢 81 │
│ secure cloud storage       │  3 800 │     51 │ Commercial   │    🟢 84 │
│ what is cloud storage      │  6 700 │     42 │ Info         │    🟡 68 │
└────────────────────────────┴────────┴────────┴──────────────┴──────────┘
```

### `/content-audit`

Full-site content quality analysis with duplication and cannibalization detection.

**Usage:**

```bash
# Audit entire site
/content-audit --scope full

# Audit specific section
/content-audit --scope "/blog/*" --min-quality 60

# Export report
/content-audit --scope full --output md --file audit-report.md
```

**Options:**
- `--scope <path>` — URL path pattern to audit (full, /section/*, /page)
- `--min-quality <number>` — Flag pages below quality threshold (0-100)
- `--check-duplicates` — Detect duplicate and near-duplicate content
- `--check-cannibalization` — Identify keyword cannibalization
- `--output <format>` — Output format: table, md, json, html

**Output Example:**

```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  example.com                   ║
╠══════════════════════════════════════════════════╣
║  Crawling pages …         [████████░░]  82%  245 ║
║  Analyzing quality …      [██████████] 100%  Done║
║  Checking duplicates …    [██████████] 100%  Done║
╚══════════════════════════════════════════════════╝

Quality Distribution:
🟢 High (80-100):    142 pages (58%)
🟡 Medium (60-79):    78 pages (32%)
🟠 Low (40-59):       21 pages ( 9%)
🔴 Very Low (0-39):    4 pages ( 2%)

Critical Issues:
┌───────────────────────────────────────────┬────────┬──────────┐
│ Page                                      │ Issue  │ Severity │
├───────────────────────────────────────────┼────────┼──────────┤
│ /blog/cloud-storage-guide                 │ Thin   │   🔴 High│
│ /products/storage-a vs /products/storage-b│ 89% dup│   🔴 High│
│ /blog/seo-tips + 3 other pages            │ Cannibal│  🟠 Med │
└───────────────────────────────────────────┴────────┴──────────┘
```

### `/technical-seo`

Technical SEO audit covering crawl budget, Core Web Vitals, schema markup, and indexability.

**Usage:**

```bash
# Full technical audit
/technical-seo

# Specific checks
/technical-seo --checks "core-web-vitals,schema,robots"

# CI/CD integration
/technical-seo --format json --threshold 85 --fail-on-critical
```

**Options:**
- `--checks <list>` — Comma-separated: core-web-vitals, schema, robots, sitemap, canonicals, hreflang, structured-data, mobile
- `--threshold <number>` — Pass/fail threshold score (0-100)
- `--fail-on-critical` — Exit code 1 if critical issues found
- `--format <type>` — Output: table, json, html

**Output Example:**

```
╔══════════════════════════════════════════════════╗
║  Technical SEO Audit  —  example.com             ║
╠══════════════════════════════════════════════════╣
║  Checking Core Web Vitals … [██████████] 100% ✓  ║
║  Validating schema markup … [██████████] 100% ✓  ║
║  Analyzing crawl budget …   [██████████] 100% ✓  ║
╚══════════════════════════════════════════════════╝

Overall Score: 🟢 87/100

┌──────────────────────────┬──────────┬──────────┐
│ Check                    │ Status   │ Score    │
├──────────────────────────┼──────────┼──────────┤
│ Core Web Vitals          │   ✓ Pass │   🟢 92  │
│ Schema Markup            │   ✓ Pass │   🟢 88  │
│ Mobile-Friendly          │   ✓ Pass │   🟢 96  │
│ Robots.txt               │   ✓ Pass │  🟢 100  │
│ XML Sitemap              │   ⚠ Warn │   🟡 75  │
│ Canonical Tags           │   ✗ Fail │   🔴 58  │
└──────────────────────────┴──────────┴──────────┘

Critical Issues:
• 47 pages missing canonical tags
• Sitemap contains 23 noindexed URLs
• 12 pages have conflicting canonical chains
```

### `/competitor-gap`

Backlink gap, topic gap, and featured-snippet opportunity analysis.

**Usage:**

```bash
# Compare against multiple competitors
/competitor-gap "example.com" --competitors "competitor1.com,competitor2.com"

# Focus on specific gap type
/competitor-gap "example.com" --competitors "rival.com" --gap-type backlinks

# Export opportunities
/competitor-gap "example.com" --competitors "rival.com" --output csv --file gaps.csv
```

**Options:**
- `--competitors <list>` — Comma-separated competitor domains
- `--gap-type <type>` — Focus: backlinks, topics, keywords, featured-snippets, all (default)
- `--min-value <number>` — Minimum opportunity value threshold
- `--output <format>` — Output: table, csv, json

### `/content-brief`

AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Usage:**

```bash
# Generate brief for target keyword
/content-brief "how to choose cloud storage"

# With custom parameters
/content-brief "saas pricing models" --word-count 2500 --tone professional --include-faqs

# Multiple keywords
/content-brief "best crm software" --secondary "crm comparison,crm features"
```

**Options:**
- `--word-count <number>` — Target word count (default: auto-calculated from SERP)
- `--tone <style>` — Content tone: professional, casual, technical, friendly
- `--include-faqs` — Add FAQ section based on "People Also Ask"
- `--include-stats` — Add data/statistics section
- `--output <format>` — Output: md (default), json, html

**Output Example:**

```markdown
# Content Brief: "how to choose cloud storage"

## Target Metrics
- **Primary Keyword:** how to choose cloud storage
- **Search Volume:** 3,800/mo
- **Keyword Difficulty:** 48/100
- **Target Word Count:** 2,200-2,500 words
- **Content Type:** Guide / How-to
- **Search Intent:** Informational → Commercial

## Recommended Outline

### 1. Introduction (150-200 words)
- Hook: Common cloud storage decision pain points
- Brief overview of selection criteria
- What readers will learn

### 2. Key Factors When Choosing Cloud Storage (400-500 words)
- Storage capacity and scalability
- Security and encryption standards
- Pricing models and cost comparison
- Integration with existing tools
- File sharing and collaboration features

### 3. Top Cloud Storage Providers Compared (500-600 words)
[Include comparison table]
- Provider A: strengths, weaknesses, best for
- Provider B: strengths, weaknesses, best for
- Provider C: strengths, weaknesses, best for

### 4. Security Considerations (300-400 words)
- End-to-end encryption
- Compliance certifications (SOC 2, GDPR, HIPAA)
- Two-factor authentication
- Data backup and recovery

### 5. Making Your Final Decision (200-300 words)
- Decision framework
- Free trial recommendations
- Migration considerations

## NLP Terms to Include
(Frequency targets based on top-10 SERP analysis)

High Priority (8-12 mentions):
- cloud storage provider
- file storage
- data security
- storage space
- backup solution

Medium Priority (4-7 mentions):
- encryption
- file sharing
- collaboration
- pricing plan
- free storage

## Questions to Answer
(From "People Also Ask")

1. What is the most secure cloud storage?
2. How much cloud storage do I need?
3. What is the cheapest cloud storage option?
4. Can I use multiple cloud storage services?
5. Is cloud storage safe for business documents?

## Internal Linking Opportunities
- Link to: /blog/cloud-storage-security-guide
- Link to: /compare/dropbox-vs-google-drive
- Link to: /pricing/cloud-storage-costs

## External Authority Links
- NIST cloud security standards
- Gartner cloud storage market report
- Industry compliance documentation
```

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization suggestions.

**Usage:**

```bash
# Monitor keyword rankings
/serp-monitor --keywords "keyword1,keyword2,keyword3"

# With alerts
/serp-monitor --keywords-file keywords.txt --alert-on-drop 3 --email user@example.com

# Historical comparison
/serp-monitor --keywords "target keyword" --compare-date 2026-04-01
```

**Options:**
- `--keywords <list>` — Comma-separated keywords to track
- `--keywords-file <path>` — File containing keywords (one per line)
- `--alert-on-drop <positions>` — Send alert if rank drops by N positions
- `--alert-on-gain <positions>` — Send alert if rank gains N positions
- `--email <address>` — Email for alerts (requires SMTP configuration)
- `--compare-date <YYYY-MM-DD>` — Compare against specific date

### `/link-prospecting`

Quality backlink prospect list with DA/DR filters and outreach templates.

**Usage:**

```bash
# Find link prospects for topic
/link-prospecting "cloud computing" --min-da 40 --type guest-post

# Resource page opportunities
/link-prospecting "marketing tools" --type resource-page --contacts

# Export with outreach templates
/link-prospecting "seo tools" --output csv --include-templates --file prospects.csv
```

**Options:**
- `--type <strategy>` — Prospecting type: guest-post, resource-page, broken-link, competitor-backlinks, unlinked-mentions
- `--min-da <number>` — Minimum Domain Authority (0-100)
- `--min-dr <number>` — Minimum Domain Rating (0-100)
- `--contacts` — Include contact information when available
- `--include-templates` — Add outreach email templates

### `/page-speed-seo`

Page speed analysis with render-blocking diagnosis and ranking impact assessment.

**Usage:**

```bash
# Analyze specific page
/page-speed-seo "https://example.com/slow-page"

# Batch analysis
/page-speed-seo --sitemap "https://example.com/sitemap.xml" --pages 50

# CI/CD integration
/page-speed-seo $DEPLOY_URL --threshold 85 --fail-on-threshold
```

**Options:**
- `--threshold <score>` — Minimum acceptable score (0-100)
- `--fail-on-threshold` — Exit code 1 if below threshold
- `--device <type>` — Test device: mobile, desktop, both (default)
- `--metrics <list>` — Focus metrics: lcp, fid, cls, fcp, ttfb, all (default)

### `/local-seo`

Local SEO audit covering NAP consistency, Google Business Profile optimization, and local citations.

**Usage:**

```bash
# Full local SEO audit
/local-seo --business "Example Business" --location "New York, NY"

# NAP consistency check
/local-seo --business "Example Business" --check nap-consistency

# Citation audit
/local-seo --business "Example Business" --check citations --min-da 30
```

**Options:**
- `--business <name>` — Business name
- `--location <city, state>` — Business location
- `--check <type>` — Check type: nap-consistency, gbp-optimization, citations, reviews, all (default)
- `--min-da <number>` — Minimum DA for citation sources

### `/content-calendar`

Data-driven editorial calendar built from search demand and seasonality.

**Usage:**

```bash
# Generate 3-month calendar
/content-calendar --topics "seo,content marketing,analytics" --months 3

# Include trend data
/content-calendar --topics "ecommerce" --months 6 --include-trends

# Export to project management tool
/content-calendar --topics "saas" --months 4 --output csv --file calendar.csv
```

**Options:**
- `--topics <list>` — Comma-separated topic areas
- `--months <number>` — Number of months to plan (1-12)
- `--posts-per-month <number>` — Target posts per month
- `--include-trends` — Include Google Trends seasonality data
- `--output <format>` — Output: table, csv, json, md

## Multi-Step Workflows

### `full-seo-sprint`

Complete 12-step SEO sprint from audit to implementation.

**Usage:**

```bash
/workflows:full-seo-sprint "example.com" --scope full
```

**Steps:**
1. Technical SEO audit
2. Content audit
3. Keyword research
4. Competitor gap analysis
5. Content opportunity mapping
6. Priority action plan generation
7. Content brief creation
8. Technical fix recommendations
9. Link-building strategy
10. Implementation timeline
11. KPI dashboard setup
12. Monitoring configuration

### `launch-seo`

Pre-launch SEO checklist with validation.

**Usage:**

```bash
/workflows:launch-seo "https://staging.example.com" --go-live "2026-06-15"
```

**Validates:**
- Canonical tags
- Hreflang (if multilingual)
- XML sitemap
- Robots.txt
- Schema markup
- Core Web Vitals
- Mobile-friendliness
- 301 redirects
- Analytics setup
- Search Console setup

### `content-refresh`

Identify and refresh underperforming pages.

**Usage:**

```bash
/workflows:content-refresh --ranking-drop 5 --timeframe 90days
```

### `authority-building`

End-to-end digital PR and link-building campaign.

**Usage:**

```bash
/workflows:authority-building --topic "fintech security" --target-links 50 --months 6
```

### `ai-content-pipeline`

Automated keyword → brief → draft → optimize → publish pipeline.

**Usage:**

```bash
/workflows:ai-content-pipeline --keywords "kw1,kw2,kw3" --auto-publish false
```

## Configuration

### Environment Variables

Create `.env` file in project root:

```bash
# API Keys (required for full functionality)
SEMRUSH_API_KEY=your_semrush_api_key_here
AHREFS_API_KEY=your_ahrefs_api_key_here
GOOGLE_SEARCH_CONSOLE_CREDENTIALS=path/to/credentials.json
GOOGLE_ANALYTICS_CREDENTIALS=path/to/credentials.json

# SERP API (choose one)
SERP_API_KEY=your_serpapi_key_here
# OR
SERPER_API_KEY=your_serper_api_key_here

# Page Speed
PAGESPEED_API_KEY=your_pagespeed_insights_key_here

# Optional: Email alerts
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=user@example.com
SMTP_PASSWORD=your_smtp_password_here

# Optional: Slack notifications
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Optional: Custom thresholds
DEFAULT_QUALITY_THRESHOLD=70
DEFAULT_SPEED_THRESHOLD=85
DEFAULT_MIN_DA=30
```

### Configuration File

Create `config.yml` in project root:

```yaml
defaults:
  output_format: table
  quality_threshold: 70
  speed_threshold: 85
  min_da: 30
  min_dr: 25

keyword_research:
  default_volume_min: 100
  default_difficulty_max: 70
  cluster_similarity_threshold: 0.75

content_audit:
  quality_factors:
    - word_count
    - readability
    - keyword_optimization
    - internal_links
    - external_links
    - image_optimization
  duplicate_threshold: 0.85

technical_seo:
  core_web_vitals:
    lcp_threshold: 2.5
    fid_threshold: 100
    cls_threshold: 0.1
  mobile_friendly_required: true
  https_required: true

link_prospecting:
  default_prospects_per_query: 50
  require_email_contact: false
  outreach_templates_dir: ./templates/outreach

content_calendar:
  default_posts_per_month: 8
  include_social_media: true
  lead_time_days: 14
```

## Real-World Examples

### Example 1: Complete Site SEO Audit

```bash
# Step 1: Technical audit
/technical-seo --checks all --format json > tech-audit.json

# Step 2: Content audit
/content-audit --scope full --check-duplicates --check-cannibalization --output md --file content-audit.md

# Step 3: Identify quick wins
/keyword-research --volume-min 500 --difficulty-max 40 --cluster --output csv --file quick-wins.csv

# Step 4: Generate action plan
/workflows:full-seo-sprint "example.com" --scope full
```

### Example 2: Launch New Blog Post

```bash
# Step 1: Research target keyword
/keyword-research "best project management software 2026" --volume-min 1000

# Step 2: Analyze competitors
/competitor-gap "yoursite.com" --competitors "competitor1.com,competitor2.com" --gap-type topics

# Step 3: Generate content brief
/content-brief "best project management software 2026" \
  --word-count 3000 \
  --tone professional \
  --include-faqs \
  --include-stats \
  --output md \
  --file brief-project-management.md

# Step 4: After writing, optimize page speed
/page-speed-seo "https://yoursite.com/blog/best-project-management-software" --device both

# Step 5: Set up monitoring
/serp-monitor --keywords "best project management software 2026" --alert-on-drop 3
```

### Example 3: Link Building Campaign

```bash
# Step 1: Find prospects
/link-prospecting "project management" \
  --type guest-post \
  --min-da 40 \
  --contacts \
  --include-templates \
  --output csv \
  --file prospects.csv

# Step 2: Find broken link opportunities
/link-prospecting "project management resources" \
  --type broken-link \
  --min-da 35 \
  --output csv \
  --file broken-links.csv

# Step 3: Execute full campaign
/workflows:authority-building --topic "project management" --target-links 50 --months 6
```

### Example 4: Content Refresh for Declining Rankings

```bash
# Step 1: Identify underperforming pages
/serp-monitor --keywords-file all-keywords.txt --compare-date 2026-01-01

# Step 2: Run content refresh workflow
/workflows:content-refresh --ranking-drop 5 --timeframe 90days

# Step 3: Generate updated briefs for flagged pages
# (workflow will output list of pages needing refresh)

# Step 4: Monitor recovery
/serp-monitor --keywords-file refreshed-pages-keywords.txt --alert-on-gain 2
```

### Example 5: CI/CD Integration

Add to `.github/workflows/seo-checks.yml`:

```yaml
name: SEO Checks

on:
  pull_request:
  push:
    branches: [main, staging]

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Technical SEO Check
        run: |
          /technical-seo \
            --checks "core-web-vitals,schema,mobile" \
            --format json \
            --threshold 85 \
            --fail-on-critical
        env:
          PAGESPEED_API_KEY: ${{ secrets.PAGESPEED_API_KEY }}
      
      - name: Page Speed Check
        run: |
          /page-speed-seo "${{ secrets.STAGING_URL }}" \
            --threshold 85 \
            --fail-on-threshold \
            --device both
        env:
          PAGESPEED_API_KEY: ${{ secrets.PAGESPEED_API_KEY }}
      
      - name: Content Quality Check
        run: |
          /content-audit \
            --scope full \
            --min-quality 70 \
            --output json > content-report.json
        continue-on-error: true
      
      - name: Upload SEO Reports
        uses: actions/upload-artifact@v3
        with:
          name: seo-reports
          path: |
            tech-audit.json
            content-report.json
```

## Common Patterns

### Pattern 1: Weekly SEO Health Check

```bash
#!/bin/bash
# weekly-seo-check.sh

SITE="example.com"
DATE=$(date +%Y-%m-%d)
REPORT_DIR="./reports/$DATE"

mkdir -p "$REPORT_DIR"

echo "Running weekly SEO health check for $SITE..."

# Technical check
/technical-seo --format json > "$REPORT_DIR/technical.json"

# Rank monitoring
/serp-monitor --keywords-file keywords.txt > "$REPORT_DIR/rankings.txt"

# Page speed spot check (homepage + top 10 pages)
/page-speed-seo "https://$SITE" --device both > "$REPORT_DIR/speed-home.txt"

# Content audit (weekly sample)
/content-audit --scope "/blog/*" --min-quality 70 > "$REPORT_DIR/content-sample.txt"

echo "✓ Weekly check complete. Reports in $REPORT_DIR"
```

### Pattern 2: Content Production Pipeline

```bash
#!/bin/bash
# content-pipeline.sh

# Input: CSV with target keywords
KEYWORDS_FILE="$1"

while IFS=, read -r keyword secondary_keywords
do
  echo "Processing: $keyword"
  
  # Generate brief
  SLUG=$(echo "$keyword" | tr ' ' '-' | tr '[:upper:]' '[:lower:]')
  
  /content-brief "$keyword" \
    --secondary "$secondary_keywords" \
    --word-count 2500 \
    --tone professional \
    --include-faqs \
    --output md \
    --file "briefs/${SLUG}-brief.md"
  
  echo "✓ Brief created: briefs/${SLUG}-brief.md"
  
done < "$KEYWORDS_FILE"

echo "✓ All briefs generated"
```

### Pattern 3: Competitor Monitoring Dashboard

```bash
#!/bin/bash
# competitor-dashboard.sh

SITE="example.com"
COMPETITORS="competitor1.com,competitor2.com,competitor3.com"

# Backlink gap
/competitor-gap "$SITE" \
  --competitors "$COMPETITORS" \
  --gap-type backlinks \
  --output json > backlink-gap.json

# Topic gap
/competitor-gap "$SITE" \
  --competitors "$COMPETITORS" \
  --gap-type topics \
  --output json > topic-gap.json

# Keyword gap
/competitor-gap "$SITE" \
  --competitors "$COMPETITORS" \
  --gap-type keywords \
  --min-value 500 \
  --output csv --file keyword-gap.csv

echo "✓ Competitor analysis complete"
```

## Troubleshooting

### API Rate Limits

If you hit rate limits:

```bash
# Add delay between requests
export API_RATE_LIMIT_DELAY=2000  # milliseconds

# Or use batch mode with built-in rate limiting
/keyword-research --batch keywords.txt --rate-limit 30  # 30 requests/min
```

### Missing API Keys

Commands will indicate which API keys are required:

```bash
$ /keyword-research "test"
❌ Error: SEMRUSH_API_KEY not found in environment
   Set via: export SEMRUSH_API_KEY=your_key_here
   Or add to .env file
```

### Large Site Audits Timing Out

For sites with 10,000+ pages:

```bash
# Use scope filters
/content-audit --scope "/blog/*" --limit 1000

# Or run in sections
/content-audit --scope "/blog/*" > blog-audit.txt
/content-audit --scope "/products/*" > products-audit.txt
/content-audit --scope "/resources/*" > resources-audit.txt
```

### Schema Validation Errors

Check specific schema types:

```bash
/technical-seo --checks schema --schema-types "Article,Product,Organization"
```

### SERP Data Freshness

Force fresh SERP data (bypasses cache):

```bash
/keyword-research "keyword" --fresh
/serp-monitor --keywords "keyword" --force-refresh
```

### Export Format Issues

If CSV exports are malformed:

```bash
# Use explicit encoding
/content-audit --scope full --output csv --encoding utf-8 --file audit.csv

# Or use JSON and convert
/content-audit --scope full --output json > audit.json
python -m json.tool audit.json | jq -r '.pages[] | [.url, .quality, .issues] | @csv' > audit.csv
```

## Integration with Other Tools

### Google Search Console

```bash
# Sync GSC data for more accurate rank tracking
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=/path/to/credentials.json

/serp-monitor --keywords-file keywords.txt --source gsc
```

### Screaming Frog

```bash
# Import Screaming Frog crawl
/content-audit --import-crawl screaming-frog-export.csv

# Or export audit for Screaming Frog
/technical-seo --export screaming-frog --file sf-import.csv
```

###
