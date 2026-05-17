---
name: seo-content-marketing-skill-suite
description: SEO & content marketing command suite for keyword research, content audits, technical SEO, competitor analysis, and workflow automation
triggers:
  - "help me with keyword research"
  - "run an SEO audit on this site"
  - "analyze competitor content gaps"
  - "create an SEO content brief"
  - "check technical SEO issues"
  - "build a content calendar"
  - "find backlink opportunities"
  - "optimize page speed for SEO"
---

# SEO & Content Marketing Skill Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides **10 specialized SEO and content marketing commands** and **5 multi-step workflows** for keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy automation. Derived from `alirezarezvani/claude-code-skill-factory` with focus on structured output, progress tracking, and actionable recommendations.

## What This Project Does

- **Keyword Research**: Deep clustering, opportunity scoring, SERP intent mapping
- **Content Audits**: Quality scoring, duplication detection, cannibalization reports
- **Technical SEO**: Crawl budget, Core Web Vitals, schema markup, indexability
- **Competitor Analysis**: Backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy**: Briefs, calendars, refresh workflows, AI content pipelines
- **SERP Monitoring**: Rank tracking, volatility alerts, CTR optimization
- **Link Building**: Prospecting, outreach templates, authority campaigns
- **Local SEO**: NAP consistency, Google Business Profile optimization

## Installation

### Method 1: Manual Installation

```bash
# Clone to Claude Code skills directory
mkdir -p ~/.claude/skills/
cd ~/.claude/skills/
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git seo-content-marketing-skill-suite

# Register with Claude Code
# In a Claude Code session, run:
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md
```

### Method 2: Direct Copy

```bash
# Copy the skill directory to your agent's skills folder
cp -r /path/to/r04-alirezarezvani-claude-code-skill-factory-seo ~/.claude/skills/seo-content-marketing-skill-suite/
```

### Verification

After installation, test with:
```bash
/keyword-research --help
```

## Core Commands

### 1. Keyword Research

```bash
# Basic keyword research
/keyword-research "marketing automation"

# Advanced with filters
/keyword-research "SaaS marketing" --min-volume 500 --max-difficulty 40 --intent commercial

# Export results
/keyword-research "content marketing" --output csv --export keywords.csv
```

**Expected Output Structure:**
```
╔════════════════════════════════════════════════╗
║  Keyword Research  —  "marketing automation"   ║
╠════════════════════════════════════════════════╣
║  Analyzing seed keyword …     [██████████] 100% ║
║  Clustering variations …      [██████████] 100% ║
║  Scoring opportunities …      [████████░░]  80% ║
╚════════════════════════════════════════════════╝

┌──────────────────────────┬────────┬────────┬──────────┬──────────┐
│ Keyword                  │ Volume │ Diff.  │ Intent   │ Score    │
├──────────────────────────┼────────┼────────┼──────────┼──────────┤
│ marketing automation     │ 12,100 │   45   │ Comm.    │  🟢 85   │
│ best marketing automation│  3,600 │   38   │ Comm.    │  🟢 92   │
│ marketing automation tool│  2,900 │   42   │ Comm.    │  🟡 78   │
└──────────────────────────┴────────┴────────┴──────────┴──────────┘
```

### 2. Content Audit

```bash
# Full site audit
/content-audit https://example.com --scope full

# Specific section
/content-audit https://example.com/blog --scope section

# With duplicate detection
/content-audit https://example.com --check-duplicates --similarity-threshold 85

# Export report
/content-audit https://example.com --output md --export audit-report.md
```

**Options:**
- `--scope`: `full`, `section`, `page`
- `--check-duplicates`: Enable duplicate content detection
- `--similarity-threshold`: % similarity to flag (default: 80)
- `--output`: `md`, `json`, `csv`, `html`

### 3. Technical SEO Audit

```bash
# Complete technical audit
/technical-seo https://example.com

# Specific checks
/technical-seo https://example.com --checks crawlability,schema,vitals

# Mobile-first audit
/technical-seo https://example.com --device mobile --vitals

# With fix suggestions
/technical-seo https://example.com --suggest-fixes
```

**Available Checks:**
- `crawlability`: Robots.txt, sitemap, internal linking
- `schema`: Structured data markup validation
- `vitals`: Core Web Vitals (LCP, FID, CLS)
- `indexability`: Meta robots, canonical tags, hreflang
- `security`: HTTPS, mixed content, security headers

### 4. Competitor Gap Analysis

```bash
# Backlink gap
/competitor-gap --type backlinks --competitors competitor1.com,competitor2.com

# Topic gap
/competitor-gap --type topics --competitors competitor1.com,competitor2.com --your-domain example.com

# Featured snippet opportunities
/competitor-gap --type snippets --competitors competitor1.com,competitor2.com
```

**Output Structure:**
```json
{
  "gap_type": "backlinks",
  "your_domain": "example.com",
  "competitors": ["competitor1.com", "competitor2.com"],
  "opportunities": [
    {
      "domain": "authority-site.com",
      "links_to_competitors": 3,
      "links_to_you": 0,
      "domain_authority": 72,
      "priority": "high",
      "outreach_angle": "Guest post on topic X"
    }
  ]
}
```

### 5. SEO Content Brief Generator

```bash
# Generate brief from target keyword
/content-brief "email marketing best practices"

# With custom parameters
/content-brief "SaaS pricing strategies" --target-length 2500 --competitors 5 --include-faq

# Export as template
/content-brief "content marketing ROI" --template markdown --export brief.md
```

**Brief Structure:**
```markdown
# Content Brief: Email Marketing Best Practices

## Target Keyword
Primary: "email marketing best practices"
Secondary: ["email campaign tips", "effective email marketing"]

## Search Intent
Informational + Commercial

## Target Word Count
2,200 - 2,500 words

## Recommended Structure
1. Introduction (150-200 words)
2. Core Best Practices (1,200 words)
   - Segmentation strategies
   - Subject line optimization
   - Personalization tactics
3. Tools & Resources (400 words)
4. Common Mistakes (300 words)
5. Conclusion & CTA (150 words)

## NLP Terms to Include
- subscriber engagement
- open rate optimization
- A/B testing
- deliverability
...

## Competitor Analysis
Top 5 ranking pages analyzed for structure and coverage
```

### 6. SERP Monitoring

```bash
# Track rankings
/serp-monitor --keywords keywords.csv --domain example.com

# Daily tracking with alerts
/serp-monitor --keywords keywords.csv --frequency daily --alert-threshold 3

# Export rank report
/serp-monitor --keywords keywords.csv --output json --export rankings.json
```

**Keywords CSV Format:**
```csv
keyword,target_url,device
marketing automation,https://example.com/marketing-automation,desktop
best CRM software,https://example.com/crm-guide,mobile
```

### 7. Link Prospecting

```bash
# Find backlink prospects
/link-prospecting "marketing automation" --min-da 40 --type guest-post

# Resource page opportunities
/link-prospecting "marketing tools" --type resource-page --export prospects.csv

# Broken link building
/link-prospecting --type broken-links --competitor competitor.com
```

**Prospect Types:**
- `guest-post`: Sites accepting guest contributions
- `resource-page`: Curated resource/tool lists
- `broken-links`: 404s on competitor sites
- `unlinked-mentions`: Brand mentions without links

### 8. Page Speed SEO Audit

```bash
# Full speed audit
/page-speed-seo https://example.com

# Mobile-focused
/page-speed-seo https://example.com --device mobile --priority vitals

# With fix scripts
/page-speed-seo https://example.com --generate-fixes
```

**Output includes:**
- Render-blocking resources
- LCP elements and optimization
- CLS contributors
- FID improvement suggestions
- Impact on SEO rankings

### 9. Local SEO Audit

```bash
# NAP consistency check
/local-seo --business "Acme Marketing" --check nap-consistency

# Google Business Profile optimization
/local-seo --business "Acme Marketing" --check gbp-optimization

# Citation audit
/local-seo --business "Acme Marketing" --check citations --export citations.csv
```

**NAP Format:**
```json
{
  "name": "Acme Marketing Inc.",
  "address": "123 Main St, Suite 100, Austin, TX 78701",
  "phone": "+1-512-555-0123",
  "inconsistencies": [
    {
      "source": "yelp.com",
      "issue": "Phone number differs: +1-512-555-0199",
      "fix": "Update to +1-512-555-0123"
    }
  ]
}
```

### 10. Content Calendar Generator

```bash
# Generate calendar from keyword research
/content-calendar --keywords keywords.csv --months 3

# Seasonal content planning
/content-calendar --topic "email marketing" --seasonal --months 12

# Export to Google Sheets format
/content-calendar --keywords keywords.csv --output sheets --export calendar.xlsx
```

**Calendar Output:**
```
┌────────────┬─────────────────────────┬──────────┬──────────┬──────────┐
│ Pub Date   │ Topic                   │ Keyword  │ Volume   │ Priority │
├────────────┼─────────────────────────┼──────────┼──────────┼──────────┤
│ 2026-06-01 │ Summer Email Campaigns  │ 2,400    │ Seasonal │  🔴 High │
│ 2026-06-08 │ Email Segmentation 101  │ 1,900    │ Evergreen│  🟡 Med  │
│ 2026-06-15 │ Subject Line A/B Tests  │ 1,200    │ Evergreen│  🟢 Low  │
└────────────┴─────────────────────────┴──────────┴──────────┴──────────┘
```

## Multi-Step Workflows

### 1. Full SEO Sprint (12 Steps)

```bash
/workflows:full-seo-sprint https://example.com --scope full
```

**Steps:**
1. Technical SEO audit
2. Content audit
3. Keyword gap analysis
4. Competitor benchmarking
5. Backlink profile analysis
6. On-page optimization recommendations
7. Content refresh priorities
8. New content opportunities
9. Link building roadmap
10. Technical fixes prioritization
11. Implementation timeline
12. KPI tracking setup

### 2. Pre-Launch SEO Checklist

```bash
/workflows:launch-seo https://new-site.com --checklist full
```

**Validates:**
- Canonical tags
- Hreflang (if multi-language)
- Sitemap.xml generation and submission
- Robots.txt configuration
- Schema markup
- Core Web Vitals
- Mobile responsiveness
- Analytics/tracking setup

### 3. Content Refresh Workflow

```bash
/workflows:content-refresh https://example.com --identify underperforming --threshold 30
```

**Process:**
1. Identify pages with ranking decline
2. Analyze content gaps vs. competitors
3. Update NLP term coverage
4. Refresh statistics/data
5. Improve internal linking
6. Re-optimize meta tags
7. Schedule re-crawl request

### 4. Authority Building Campaign

```bash
/workflows:authority-building --topic "marketing automation" --duration 90
```

**Campaign Phases:**
1. Identify linkable assets
2. Prospect outreach targets
3. Create outreach templates
4. Track outreach progress
5. Monitor link acquisition
6. Measure authority growth

### 5. AI Content Pipeline

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --auto-publish false
```

**Pipeline:**
1. Keyword → Content brief
2. Brief → AI-generated draft
3. Draft → SEO optimization
4. Optimization → Quality check
5. Quality check → Editor review
6. Review → Publish (manual or auto)

## Configuration

Create `.seo-config.json` in your project root:

```json
{
  "default_domain": "https://example.com",
  "serp_api_key": "${SERP_API_KEY}",
  "ahrefs_api_key": "${AHREFS_API_KEY}",
  "semrush_api_key": "${SEMRUSH_API_KEY}",
  "google_psi_key": "${GOOGLE_PSI_KEY}",
  "output_format": "markdown",
  "export_directory": "./seo-reports",
  "tracking": {
    "rank_check_frequency": "daily",
    "alert_threshold": 3,
    "devices": ["desktop", "mobile"]
  },
  "audit_defaults": {
    "include_subdomains": false,
    "max_pages": 10000,
    "duplicate_threshold": 80,
    "content_min_words": 300
  }
}
```

**Environment Variables:**
```bash
export SERP_API_KEY="your_serpapi_key"
export AHREFS_API_KEY="your_ahrefs_key"
export SEMRUSH_API_KEY="your_semrush_key"
export GOOGLE_PSI_KEY="your_pagespeed_insights_key"
```

## Common Patterns

### Pattern 1: Monthly SEO Health Check

```bash
#!/bin/bash
# monthly-seo-check.sh

DOMAIN="https://example.com"
DATE=$(date +%Y-%m)
REPORT_DIR="./reports/${DATE}"

mkdir -p "${REPORT_DIR}"

# Technical audit
/technical-seo "${DOMAIN}" --output json --export "${REPORT_DIR}/technical.json"

# Content audit
/content-audit "${DOMAIN}" --scope full --output md --export "${REPORT_DIR}/content.md"

# Rank tracking
/serp-monitor --keywords keywords.csv --domain "${DOMAIN}" --output csv --export "${REPORT_DIR}/rankings.csv"

# Summary report
cat > "${REPORT_DIR}/summary.md" <<EOF
# SEO Health Check - ${DATE}

## Technical Issues
$(jq '.critical_issues | length' "${REPORT_DIR}/technical.json") critical issues found

## Content Performance
See content.md for detailed analysis

## Rankings
See rankings.csv for position changes
EOF
```

### Pattern 2: Competitive Intelligence Dashboard

```bash
#!/bin/bash
# competitor-intel.sh

COMPETITORS="competitor1.com,competitor2.com,competitor3.com"

# Backlink gap
/competitor-gap --type backlinks --competitors "${COMPETITORS}" --output json --export backlink-gap.json

# Topic gap
/competitor-gap --type topics --competitors "${COMPETITORS}" --output json --export topic-gap.json

# Featured snippet opportunities
/competitor-gap --type snippets --competitors "${COMPETITORS}" --output json --export snippet-opps.json

# Consolidate into dashboard
python3 build_dashboard.py backlink-gap.json topic-gap.json snippet-opps.json > dashboard.html
```

### Pattern 3: Content Production Pipeline

```python
# content_pipeline.py
import json
import subprocess

def generate_content_brief(keyword):
    result = subprocess.run(
        ['/content-brief', keyword, '--output', 'json'],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)

def generate_draft(brief):
    # Integration with AI writing tool
    # Uses brief structure and NLP terms
    pass

def optimize_for_seo(draft, brief):
    # Apply SEO recommendations
    # - Target word count
    # - NLP term inclusion
    # - Header structure
    # - Internal linking
    pass

# Workflow
keywords = ['email marketing tips', 'CRM best practices']
for keyword in keywords:
    brief = generate_content_brief(keyword)
    draft = generate_draft(brief)
    optimized = optimize_for_seo(draft, brief)
    # Save for editor review
```

## Troubleshooting

### Command Not Found

If commands aren't recognized:

```bash
# Verify installation
ls ~/.claude/skills/seo-content-marketing-skill-suite/

# Re-register skill
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md

# Check Claude Code skill registry
/skills:list
```

### API Rate Limits

If hitting rate limits on SERP/backlink APIs:

```json
{
  "rate_limiting": {
    "serp_api": {
      "max_requests_per_minute": 60,
      "retry_after": 60
    },
    "ahrefs_api": {
      "max_requests_per_minute": 30,
      "retry_after": 120
    }
  }
}
```

### Large Site Audits Timing Out

For sites with 10,000+ pages:

```bash
# Use sampling
/content-audit https://large-site.com --sample 1000 --strategy random

# Or segment by directory
/content-audit https://large-site.com/blog --scope section
/content-audit https://large-site.com/products --scope section
```

### Export/Output Issues

If exports fail:

```bash
# Verify directory exists
mkdir -p ./seo-reports

# Check permissions
chmod 755 ./seo-reports

# Use absolute paths
/content-audit https://example.com --export /full/path/to/report.md
```

### Inaccurate Keyword Data

If keyword volumes seem off:

```bash
# Use multiple data sources
/keyword-research "target keyword" --sources semrush,ahrefs,google --average-volumes

# Specify location
/keyword-research "local service" --location "Austin, TX" --language en
```

## Integration Examples

### With Google Analytics

```python
# ga_integration.py
from google.analytics.data_v1beta import BetaAnalyticsDataClient
import subprocess
import json

def get_underperforming_pages(property_id):
    client = BetaAnalyticsDataClient()
    # Fetch pages with declining traffic
    # Return list of URLs

def refresh_content(urls):
    for url in urls:
        # Generate refresh recommendations
        result = subprocess.run(
            ['/workflows:content-refresh', url],
            capture_output=True,
            text=True
        )
        print(f"Refresh plan for {url}:\n{result.stdout}")

# Run monthly
underperforming = get_underperforming_pages("properties/123456")
refresh_content(underperforming)
```

### With Google Search Console

```python
# gsc_integration.py
from google.oauth2 import service_account
from googleapiclient.discovery import build
import subprocess

credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/webmasters.readonly']
)

service = build('searchconsole', 'v1', credentials=credentials)

# Get pages with impressions but low CTR
response = service.searchanalytics().query(
    siteUrl='https://example.com',
    body={
        'startDate': '2026-04-01',
        'endDate': '2026-04-30',
        'dimensions': ['page'],
        'rowLimit': 100
    }
).execute()

low_ctr_pages = [
    row['keys'][0] for row in response['rows']
    if row['ctr'] < 0.02  # Less than 2% CTR
]

# Optimize meta tags for low CTR pages
for page in low_ctr_pages:
    subprocess.run(['/technical-seo', page, '--optimize-meta'])
```

## Advanced Usage

### Custom Scoring Algorithms

Override default opportunity scoring:

```json
{
  "scoring": {
    "keyword_opportunity": {
      "volume_weight": 0.4,
      "difficulty_weight": 0.3,
      "intent_weight": 0.2,
      "trend_weight": 0.1
    },
    "content_quality": {
      "word_count_weight": 0.2,
      "readability_weight": 0.15,
      "nlp_coverage_weight": 0.25,
      "internal_links_weight": 0.15,
      "external_links_weight": 0.1,
      "freshness_weight": 0.15
    }
  }
}
```

### Batch Processing

```bash
# Process multiple domains
cat domains.txt | while read domain; do
  /technical-seo "$domain" --output json --export "reports/${domain//\//_}.json"
done

# Batch keyword research
cat keywords.txt | while read keyword; do
  /keyword-research "$keyword" --output csv --export "keywords/${keyword// /_}.csv"
done
```

## Best Practices

1. **Run Technical Audits Weekly**: Catch issues before they impact rankings
2. **Monitor Core Web Vitals Continuously**: Set up automated alerts for CLS/LCP/FID regressions
3. **Refresh Top Content Quarterly**: Keep top-ranking pages updated with latest data
4. **Track Competitor Movement Monthly**: Stay ahead of competitive content updates
5. **Generate Content Briefs Before Writing**: Ensure SEO-optimized structure from the start
6. **Export Reports for Stakeholders**: Use `--output md` or `--output html` for sharing
7. **Version Control Your Keyword Lists**: Track keyword strategy evolution over time

## License

MIT — free to use, modify, and distribute.

---

**Source Repository**: [JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo](https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo)

**Derived From**: [alirezarezvani/claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory)
