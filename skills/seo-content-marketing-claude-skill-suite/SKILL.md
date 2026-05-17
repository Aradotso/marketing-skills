---
name: seo-content-marketing-claude-skill-suite
description: SEO & Content Marketing command suite for keyword research, content audits, SERP analysis, technical SEO, and content strategy workflows
triggers:
  - analyze keywords for SEO
  - run a content audit on this site
  - perform technical SEO analysis
  - create an SEO content brief
  - check competitor gap analysis
  - generate a content calendar
  - audit page speed for SEO
  - run local SEO audit
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This suite provides AI coding agents with 10 specialized SEO and content marketing commands plus 5 multi-step workflows. Adapted from vincenthopf/My-Claude-Code, it delivers keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy tools with structured output and progress tracking.

## What This Project Does

A command-line skill suite for AI agents performing SEO and content marketing tasks:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema markup, indexability
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — briefs, calendars, refresh workflows, AI pipelines

All commands output structured tables, progress bars, and prioritized action plans.

## Installation

### Clone and Register

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/

# Clone into skills directory
git clone https://github.com/Splitflucover93/r12-vincenthopf-my-claude-code-seo.git \
  ~/.claude/skills/seo-content-marketing/

# In Claude Code session, load the skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Verify Installation

Check available commands:

```bash
/help seo
```

You should see all 10 commands listed.

## Core Commands

### 1. Keyword Research

Perform deep keyword clustering with opportunity scoring.

```bash
# Basic keyword research
/keyword-research "marketing automation"

# With options
/keyword-research "content marketing" --depth full --export csv

# Multiple seed keywords
/keyword-research "seo,backlinks,serp" --cluster intent
```

**Output Structure:**
```
┌──────────────────────┬────────┬──────┬────────────┬──────────┐
│ Keyword              │ Volume │ KD   │ Intent     │ Priority │
├──────────────────────┼────────┼──────┼────────────┼──────────┤
│ marketing automation │ 22,200 │ 68   │ Commercial │ 🔴 High  │
│ email automation     │ 14,800 │ 52   │ Commercial │ 🟠 Med   │
│ workflow automation  │  8,100 │ 45   │ Info       │ 🟡 Low   │
└──────────────────────┴────────┴──────┴────────────┴──────────┘
```

### 2. Content Audit

Full-site content quality analysis with duplication and cannibalization checks.

```bash
# Full site audit
/content-audit --scope full --output md

# Specific path
/content-audit --path /blog --format json

# Export results
/content-audit --scope full --export results.csv
```

**Options:**
- `--scope` : `full` | `path` | `urls`
- `--output` : `md` | `json` | `html`
- `--export` : file path for CSV export

**Example Output:**
```
Content Quality Breakdown:
  Excellent (90-100): 24 pages (12%)
  Good (70-89):      89 pages (44%)
  Fair (50-69):      67 pages (33%)
  Poor (0-49):       22 pages (11%)

Issues Found:
  🔴 Thin content (<300 words):     18 pages
  🔴 Duplicate titles:               7 pages
  🟠 Keyword cannibalization:       12 page clusters
  🟡 Missing meta descriptions:     34 pages
```

### 3. Technical SEO Audit

Comprehensive technical SEO diagnostics.

```bash
# Full technical audit
/technical-seo https://example.com

# Specific checks
/technical-seo https://example.com --checks crawl,vitals,schema

# With depth limit
/technical-seo https://example.com --depth 500
```

**Check Categories:**
- `crawl` — robots.txt, sitemap, crawl budget
- `vitals` — Core Web Vitals (LCP, FID, CLS)
- `schema` — structured data validation
- `mobile` — mobile-friendliness
- `index` — indexability issues

### 4. Content Brief Generator

AI-generated SEO content briefs with NLP terms and structure.

```bash
# Generate brief
/content-brief "email marketing best practices"

# With competitor analysis
/content-brief "seo checklist" --competitors 5

# Specify word count target
/content-brief "link building guide" --words 2500
```

**Output Includes:**
- Primary and secondary keywords
- Search intent analysis
- Recommended headings structure
- NLP/LSI terms to include
- Word count targets
- Competitor content gaps

### 5. Competitor Gap Analysis

Identify backlink, topic, and featured snippet opportunities.

```bash
# Full gap analysis
/competitor-gap https://yoursite.com --competitors "competitor1.com,competitor2.com"

# Specific gap types
/competitor-gap https://yoursite.com --type backlinks --competitors "competitor.com"

# Export opportunities
/competitor-gap https://yoursite.com --competitors "comp.com" --export gaps.json
```

**Gap Types:**
- `backlinks` — linking domains they have, you don't
- `topics` — content topics they rank for
- `snippets` — featured snippet opportunities
- `keywords` — ranking keyword gaps

### 6. SERP Monitoring

Daily rank tracking with volatility alerts.

```bash
# Monitor keywords
/serp-monitor --keywords "keyword1,keyword2,keyword3"

# Load from file
/serp-monitor --file keywords.txt

# Set alert threshold
/serp-monitor --keywords "brand term" --alert-drop 3
```

**Tracked Metrics:**
- Current position
- Position change (day/week/month)
- SERP volatility score
- CTR optimization opportunities
- Featured snippet status

### 7. Link Prospecting

Generate quality backlink prospect lists with filtering.

```bash
# Find prospects
/link-prospecting "marketing blogs" --min-da 30

# Industry-specific
/link-prospecting --industry saas --type "guest-post,resource-page"

# Export with outreach templates
/link-prospecting "tech publications" --export prospects.csv --templates
```

**Filters:**
- `--min-da` : minimum Domain Authority
- `--min-dr` : minimum Domain Rating
- `--type` : prospect type (guest-post, resource-page, broken-link, etc.)
- `--industry` : industry filter

### 8. Page Speed SEO Audit

Performance diagnostics mapped to ranking impact.

```bash
# Audit page speed
/page-speed-seo https://example.com/page

# Mobile-specific
/page-speed-seo https://example.com --device mobile

# Batch audit
/page-speed-seo --file urls.txt --export speed-report.json
```

**Analyzes:**
- LCP (Largest Contentful Paint)
- FID (First Input Delay)
- CLS (Cumulative Layout Shift)
- Render-blocking resources
- SEO ranking impact score

### 9. Local SEO Audit

NAP consistency, Google Business Profile, and local citation audit.

```bash
# Local SEO audit
/local-seo "Business Name, City, State"

# With citation check
/local-seo "Acme Corp, Austin, TX" --citations 50

# Export report
/local-seo "Company Name" --export local-audit.pdf
```

**Checks:**
- NAP (Name, Address, Phone) consistency
- Google Business Profile optimization
- Local citation presence and accuracy
- Review signals
- Local schema markup

### 10. Content Calendar

Data-driven editorial calendar from search demand and seasonality.

```bash
# Generate calendar
/content-calendar --topics "seo,content marketing" --months 3

# With search trends
/content-calendar --topics "email marketing" --months 6 --trends

# Export to calendar format
/content-calendar --topics "social media" --export calendar.ics
```

## Workflows (Multi-Step)

### Full SEO Sprint

Complete 12-step SEO audit and optimization workflow.

```bash
/workflows:full-seo-sprint https://example.com --scope full
```

**Steps:**
1. Technical audit
2. Keyword research
3. Competitor gap analysis
4. Content audit
5. Keyword mapping
6. Content gaps identification
7. Quick wins list
8. Content refresh plan
9. New content plan
10. Technical fixes prioritization
11. Link building strategy
12. Measurement framework

### Launch SEO

Pre-launch SEO checklist with validation.

```bash
/workflows:launch-seo https://staging.example.com
```

**Validates:**
- Canonical tags
- Hreflang (if international)
- XML sitemap
- Robots.txt
- Schema markup
- Page speed
- Mobile responsiveness
- Index prevention removal

### Content Refresh

Identify and refresh underperforming pages.

```bash
/workflows:content-refresh --min-age 180 --max-position 20
```

**Process:**
1. Identify declining pages
2. Analyze content gaps vs. top-rankers
3. Generate refresh briefs
4. Prioritize by traffic potential
5. Create refresh schedule

### Authority Building

End-to-end digital PR and link-building campaign.

```bash
/workflows:authority-building --industry saas --target-links 50
```

**Stages:**
1. Competitor backlink analysis
2. Prospect identification
3. Content asset planning
4. Outreach template creation
5. Campaign tracking setup

### AI Content Pipeline

Automated content creation workflow.

```bash
/workflows:ai-content-pipeline --keywords "keyword1,keyword2" --auto-publish false
```

**Pipeline:**
1. Keyword research
2. Brief generation
3. Draft creation
4. SEO optimization
5. Quality check
6. Publishing (if `--auto-publish true`)

## Configuration

### Environment Variables

Create `.env` file in skill directory:

```bash
# API Keys (optional, for enhanced features)
SEO_API_KEY=your_api_key_here
SERP_API_KEY=your_serp_api_key

# Default Settings
DEFAULT_COUNTRY=US
DEFAULT_LANGUAGE=en
DEFAULT_DEVICE=desktop

# Output Preferences
OUTPUT_FORMAT=md
EXPORT_PATH=./reports/

# Alert Thresholds
RANK_DROP_ALERT=3
VOLATILITY_THRESHOLD=0.5
```

### Skill Configuration

Edit `~/.claude/skills/seo-content-marketing/config.yaml`:

```yaml
# Command defaults
defaults:
  keyword_research:
    depth: full
    cluster_method: intent
    export_format: csv
  
  content_audit:
    scope: full
    min_word_count: 300
    quality_threshold: 70
  
  technical_seo:
    checks: [crawl, vitals, schema, mobile, index]
    depth: 1000
  
  page_speed:
    device: both
    metrics: [lcp, fid, cls]

# API endpoints (if using external services)
apis:
  serp_api: ${SERP_API_KEY}
  seo_tool: ${SEO_API_KEY}

# Output settings
output:
  format: md
  show_progress: true
  export_path: ./reports/
```

## Real-World Examples

### Example 1: Complete Site Audit

```bash
# Step 1: Technical foundation
/technical-seo https://example.com --export tech-audit.json

# Step 2: Content analysis
/content-audit --scope full --export content-audit.csv

# Step 3: Keyword opportunities
/keyword-research "main topic" --depth full --export keywords.csv

# Step 4: Competitor gaps
/competitor-gap https://example.com --competitors "comp1.com,comp2.com"

# Step 5: Action plan generation
# (AI will synthesize all reports into prioritized action plan)
```

### Example 2: Content Strategy for New Site

```bash
# Research keywords
/keyword-research "industry topic" --cluster intent --export keywords.csv

# Analyze competitors
/competitor-gap --competitors "leader1.com,leader2.com" --type topics

# Generate content calendar
/content-calendar --topics "topic1,topic2,topic3" --months 6 --export calendar.ics

# Create first 5 briefs
/content-brief "keyword 1"
/content-brief "keyword 2"
# ... etc
```

### Example 3: Recovery from Traffic Drop

```bash
# Identify affected pages
/serp-monitor --alert-drop 5 --period 30days

# Audit those pages
/content-audit --urls "url1,url2,url3"

# Run refresh workflow
/workflows:content-refresh --urls "url1,url2,url3"

# Check technical issues
/technical-seo --urls "url1,url2,url3" --checks vitals,index
```

## Common Patterns

### Pattern 1: Monthly SEO Health Check

```bash
#!/bin/bash
# monthly-seo-check.sh

SITE="https://example.com"
DATE=$(date +%Y-%m)
EXPORT_DIR="./reports/$DATE"

mkdir -p "$EXPORT_DIR"

/technical-seo "$SITE" --export "$EXPORT_DIR/technical.json"
/content-audit --scope full --export "$EXPORT_DIR/content.csv"
/serp-monitor --file keywords.txt --export "$EXPORT_DIR/rankings.json"
/page-speed-seo "$SITE" --export "$EXPORT_DIR/speed.json"

echo "Monthly SEO check complete. Reports in $EXPORT_DIR"
```

### Pattern 2: Content Production Workflow

```bash
# 1. Research phase
/keyword-research "$TOPIC" --export keywords.csv

# 2. Brief creation (from top 10 keywords in CSV)
while IFS=, read -r keyword volume difficulty; do
  /content-brief "$keyword" --words 1500 --export "briefs/${keyword// /-}.md"
done < <(tail -n +2 keywords.csv | head -10)

# 3. Calendar creation
/content-calendar --file keywords.csv --months 3 --export schedule.ics
```

### Pattern 3: Link Building Campaign

```bash
# 1. Find prospects
/link-prospecting "$INDUSTRY blogs" --min-da 30 --type guest-post --export prospects.csv

# 2. Analyze competitor backlinks
/competitor-gap --type backlinks --competitors "$COMPETITORS" --export gaps.json

# 3. Create outreach templates
/link-prospecting --templates --export outreach-templates/

# 4. Track campaign
# (Monitor new backlinks over time)
```

## Troubleshooting

### Command Not Found

```bash
# Reload the skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md

# Verify installation path
ls -la ~/.claude/skills/seo-content-marketing/

# Check for syntax errors in config
cat ~/.claude/skills/seo-content-marketing/config.yaml
```

### Slow Performance on Large Sites

```bash
# Limit crawl depth
/technical-seo https://largesite.com --depth 500

# Audit specific paths only
/content-audit --path /blog --depth 2

# Use sampling for keyword research
/keyword-research "topic" --sample 1000
```

### API Rate Limits

If using external SEO APIs:

```bash
# Check rate limit status
echo $SEO_API_RATE_REMAINING

# Use batch mode with delays
/keyword-research --file keywords.txt --batch 10 --delay 2s

# Cache results
/keyword-research "topic" --cache 24h
```

### Export Failures

```bash
# Check write permissions
ls -la ./reports/

# Create export directory
mkdir -p ./reports/

# Specify full path
/content-audit --export /full/path/to/reports/audit.csv

# Use alternative format
/content-audit --output json > audit.json
```

### Incomplete Data

```bash
# Increase timeout for slow sites
/technical-seo https://slowsite.com --timeout 60s

# Retry failed requests
/content-audit --scope full --retry 3

# Use cached SERP data
/serp-monitor --cache 12h --keywords "term"
```

## Integration Examples

### CI/CD Pipeline Integration

```yaml
# .github/workflows/seo-check.yml
name: SEO Audit

on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday
  workflow_dispatch:

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Technical SEO Audit
        run: |
          /technical-seo ${{ secrets.SITE_URL }} \
            --export ./audit-results/technical.json
      
      - name: Check for Critical Issues
        run: |
          # Parse JSON and fail if critical issues found
          jq '.critical_issues | length' ./audit-results/technical.json | \
            awk '{if ($1 > 0) exit 1}'
      
      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: seo-audit-report
          path: ./audit-results/
```

### Slack Alert Integration

```bash
#!/bin/bash
# seo-alert.sh - Run daily and alert on rank drops

SLACK_WEBHOOK="${SLACK_WEBHOOK_URL}"

REPORT=$(/serp-monitor --alert-drop 3 --format json)

if echo "$REPORT" | jq -e '.alerts | length > 0' > /dev/null; then
  ALERT_COUNT=$(echo "$REPORT" | jq '.alerts | length')
  
  curl -X POST "$SLACK_WEBHOOK" \
    -H 'Content-Type: application/json' \
    -d "{\"text\": \"🚨 SEO Alert: $ALERT_COUNT keyword(s) dropped 3+ positions\"}"
fi
```

---

**Source:** https://github.com/Splitflucover93/r12-vincenthopf-my-claude-code-seo

**License:** MIT
