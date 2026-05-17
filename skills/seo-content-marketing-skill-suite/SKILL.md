---
name: seo-content-marketing-skill-suite
description: SEO & content marketing command suite with keyword research, content audits, SERP analysis, technical SEO checks, and automated workflows
triggers:
  - analyze keyword opportunities for this content
  - run a full SEO audit on this site
  - create an SEO content brief
  - check technical SEO issues
  - generate a content calendar
  - analyze competitor SEO gaps
  - monitor SERP rankings
  - find backlink opportunities
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The SEO & Content Marketing Skills Suite is a specialized collection of commands and workflows derived from the claude-code-skill-factory framework. It provides 10 domain-specific commands for SEO analysis, content optimization, and marketing automation, plus 5 multi-step workflows for comprehensive SEO campaigns.

**Key capabilities:**
- Keyword research with clustering and intent mapping
- Full-site content audits with quality scoring
- Technical SEO analysis (Core Web Vitals, schema, crawlability)
- Competitor gap analysis (backlinks, topics, featured snippets)
- AI-powered content brief generation
- SERP monitoring and rank tracking
- Link prospecting and outreach automation
- Local SEO optimization
- Data-driven content calendar creation

All commands use structured output with progress tracking, findings tables, prioritized action plans, and next-step suggestions.

## Installation

### Local Installation

```bash
# Clone to Claude Code skills directory
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git seo-content-marketing

# Activate in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Project Integration

```bash
# Add to project .claude/skills
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git seo-content-marketing
```

## Core Commands

### /keyword-research

Performs deep keyword analysis with clustering, search volume, competition scoring, and SERP intent mapping.

```bash
# Basic usage
/keyword-research "project management software"

# With options
/keyword-research "project management software" --min-volume 1000 --max-difficulty 40 --output json

# Multiple seed keywords
/keyword-research "project management, task tracking, team collaboration" --cluster true
```

**Output structure:**
```
┌─────────────────────────────┬────────┬──────────┬────────┬─────────┐
│ Keyword                     │ Volume │ KD       │ Intent │ Cluster │
├─────────────────────────────┼────────┼──────────┼────────┼─────────┤
│ project management software │ 22,000 │ 58 (Med) │ Comm   │ Core    │
│ best project management app │  8,100 │ 42 (Med) │ Comm   │ Core    │
│ free project management     │  5,400 │ 35 (Low) │ Comm   │ Price   │
│ project management tools    │ 18,000 │ 61 (Med) │ Info   │ Core    │
└─────────────────────────────┴────────┴──────────┴────────┴─────────┘
```

### /content-audit

Full-site content quality scoring with duplication checks and keyword cannibalization detection.

```bash
# Full site audit
/content-audit --scope full --output md

# Specific section
/content-audit --scope /blog --min-score 50

# Export results
/content-audit --scope full --export content-audit-report.csv
```

**Metrics tracked:**
- Content quality score (0-100)
- Word count vs. target
- Readability score
- Keyword optimization
- Internal linking depth
- Duplicate content detection
- Cannibalization risk

### /technical-seo

Comprehensive technical SEO audit covering crawlability, indexability, Core Web Vitals, and structured data.

```bash
# Full technical audit
/technical-seo https://example.com

# Specific checks
/technical-seo https://example.com --checks "vitals,schema,mobile"

# With crawl limit
/technical-seo https://example.com --max-pages 500
```

**Checks performed:**
```
Core Web Vitals:
  ✓ LCP: 1.8s (Good)
  ⚠ CLS: 0.15 (Needs Improvement)
  ✓ FID: 45ms (Good)

Crawlability:
  ✓ Robots.txt valid
  ✗ 23 pages blocked by robots.txt
  ⚠ Sitemap contains 1,204 URLs but only 1,180 discovered

Schema Markup:
  ✓ Organization schema present
  ⚠ 45% of articles missing Article schema
  ✗ No FAQ schema on product pages
```

### /competitor-gap

Analyzes backlink gaps, topic coverage gaps, and featured snippet opportunities vs. competitors.

```bash
# Basic competitor analysis
/competitor-gap https://yoursite.com https://competitor1.com,https://competitor2.com

# Focus on specific gaps
/competitor-gap https://yoursite.com https://competitor1.com --gap-type backlinks,topics

# With keyword list
/competitor-gap https://yoursite.com https://competitor1.com --keywords keywords.txt
```

### /content-brief

Generates AI-powered SEO content briefs with outlines, NLP terms, word count targets, and internal linking suggestions.

```bash
# Generate brief
/content-brief "how to manage remote teams" --target-keyword "remote team management"

# With competitor analysis
/content-brief "project management best practices" --analyze-competitors 3

# Export to template
/content-brief "agile methodology guide" --template markdown --output brief.md
```

**Brief structure:**
```markdown
# Content Brief: Remote Team Management

## Target Metrics
- Primary keyword: "remote team management"
- Target word count: 2,400-2,800 words
- Target position: Top 3
- Content type: How-to guide

## Required Topics (H2/H3)
1. Remote Team Communication Tools
   - Video conferencing best practices
   - Async communication strategies
2. Building Remote Team Culture
   - Virtual team building activities
   - Recognition and feedback loops
3. Managing Remote Team Performance
   - KPIs and metrics for remote teams
   - Remote performance reviews

## NLP Terms to Include
- distributed teams (8-12 mentions)
- virtual collaboration (5-8 mentions)
- remote work policies (4-6 mentions)
...

## Internal Links (3-5)
- /blog/remote-work-tools
- /guide/team-communication
- /resources/remote-leadership
```

### /serp-monitor

Daily rank tracking with volatility alerts and CTR optimization recommendations.

```bash
# Start monitoring
/serp-monitor --keywords keywords.csv --frequency daily

# Check current positions
/serp-monitor --report today

# Historical comparison
/serp-monitor --report --compare 30d
```

### /link-prospecting

Generates quality backlink prospect lists with DA/DR filtering and outreach templates.

```bash
# Find link prospects
/link-prospecting "project management" --min-da 30 --max-results 50

# With specific link types
/link-prospecting "SaaS tools" --types "resource-page,roundup,guest-post"

# Export with templates
/link-prospecting "productivity apps" --export prospects.csv --include-templates
```

### /page-speed-seo

Diagnoses render-blocking resources, LCP, CLS, and FID issues mapped to ranking impact.

```bash
# Speed audit
/page-speed-seo https://example.com/page

# Mobile focus
/page-speed-seo https://example.com --device mobile

# With recommendations
/page-speed-seo https://example.com --priority high --output actionable.md
```

### /local-seo

NAP consistency checks, Google Business Profile optimization, and local citation audits.

```bash
# Local SEO audit
/local-seo "Business Name" --location "City, State"

# Citation audit
/local-seo "Business Name" --check citations --export citations.csv

# GBP optimization
/local-seo "Business Name" --optimize-gbp --category "Primary Category"
```

### /content-calendar

Creates data-driven editorial calendars based on search demand and seasonality.

```bash
# Generate 3-month calendar
/content-calendar --duration 3months --topics topics.txt

# With search trends
/content-calendar --duration 6months --include-trends --export calendar.csv

# Team-specific
/content-calendar --duration 1month --team-size 3 --format trello
```

## Workflows

### full-seo-sprint

12-step comprehensive SEO sprint covering audit, keyword mapping, content planning, and technical fixes.

```bash
/workflows:full-seo-sprint https://example.com --scope full
```

**Steps:**
1. Initial site crawl and indexation check
2. Technical SEO audit (Core Web Vitals, schema, mobile)
3. On-page SEO analysis
4. Keyword research and clustering
5. Content gap analysis
6. Competitor benchmarking
7. Backlink profile audit
8. Content quality scoring
9. Internal linking optimization
10. Priority fixes list generation
11. Content calendar creation
12. Implementation roadmap

### launch-seo

Pre-launch SEO checklist with canonical tags, hreflang, sitemap validation, and indexability checks.

```bash
/workflows:launch-seo https://staging.example.com --go-live-date 2026-06-01
```

### content-refresh

Identifies and refreshes underperforming content to recover lost rankings.

```bash
/workflows:content-refresh --min-position 11 --max-position 50 --priority declining
```

### authority-building

End-to-end digital PR and link-building campaign workflow.

```bash
/workflows:authority-building --target-da 50 --campaign-duration 6months
```

### ai-content-pipeline

Automated pipeline: keyword → brief → draft → optimize → publish.

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --output-dir ./content --auto-publish false
```

## Configuration

Create `.seo-suite.yaml` in your project root:

```yaml
# SEO Suite Configuration
project:
  domain: example.com
  target_market: en-US
  competitor_domains:
    - competitor1.com
    - competitor2.com

api_keys:
  # Use environment variables
  serp_api: ${SERP_API_KEY}
  semrush_api: ${SEMRUSH_API_KEY}
  ahrefs_api: ${AHREFS_API_KEY}

defaults:
  keyword_research:
    min_volume: 100
    max_difficulty: 60
    cluster: true
  
  content_audit:
    min_quality_score: 60
    check_cannibalization: true
  
  technical_seo:
    max_pages_crawl: 1000
    check_mobile: true
    check_vitals: true

output:
  format: markdown
  include_progress: true
  export_csv: true
```

## Common Patterns

### Complete Site Audit Workflow

```bash
# 1. Run technical audit
/technical-seo https://example.com --output technical-audit.md

# 2. Content quality check
/content-audit --scope full --min-score 50 --output content-audit.md

# 3. Keyword gap analysis
/competitor-gap https://example.com https://competitor.com --gap-type topics

# 4. Generate action plan
/workflows:full-seo-sprint https://example.com --scope full
```

### Content Optimization Pipeline

```bash
# 1. Research target keyword
/keyword-research "target topic" --cluster true

# 2. Generate content brief
/content-brief "target topic" --target-keyword "primary keyword" --output brief.md

# 3. After writing, optimize
/page-speed-seo https://example.com/new-article --priority high

# 4. Monitor performance
/serp-monitor --keywords "primary keyword,secondary keyword" --frequency daily
```

### Link Building Campaign

```bash
# 1. Find prospects
/link-prospecting "industry topic" --min-da 30 --types "guest-post,resource" --export prospects.csv

# 2. Launch full campaign
/workflows:authority-building --target-da 50 --campaign-duration 3months

# 3. Monitor backlink growth
/competitor-gap https://example.com https://competitor.com --gap-type backlinks
```

## Troubleshooting

### Command not found

Ensure the skill is properly loaded:

```bash
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### API rate limits

Commands respect rate limits. Configure delays in `.seo-suite.yaml`:

```yaml
api_settings:
  rate_limit_delay: 1000  # milliseconds between requests
  max_retries: 3
```

### Large site crawls timing out

Limit crawl depth or page count:

```bash
/technical-seo https://example.com --max-pages 500 --max-depth 3
```

### Missing API credentials

Set environment variables before running commands:

```bash
export SERP_API_KEY=your_serp_api_key
export SEMRUSH_API_KEY=your_semrush_key
export AHREFS_API_KEY=your_ahrefs_key
```

### Output formatting issues

Specify output format explicitly:

```bash
/content-audit --scope full --format json --output report.json
```

## Integration Examples

### CI/CD Integration

```yaml
# .github/workflows/seo-check.yml
name: SEO Audit
on:
  push:
    branches: [main]

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Technical SEO Check
        run: |
          /technical-seo https://staging.example.com \
            --checks "vitals,mobile,schema" \
            --output seo-report.md
      - name: Upload Report
        uses: actions/upload-artifact@v2
        with:
          name: seo-report
          path: seo-report.md
```

### Scheduled Monitoring

```bash
# Add to crontab for daily monitoring
0 9 * * * /usr/local/bin/claude-code --skill seo-content-marketing \
  --cmd "/serp-monitor --report today" >> /var/log/seo-monitor.log
```

## Advanced Usage

### Custom Workflow Creation

Combine multiple commands into custom workflows:

```bash
# Create custom audit script
#!/bin/bash
echo "Starting custom SEO audit..."

/technical-seo $1 --output tech-audit.md
/content-audit --scope full --output content-audit.md
/competitor-gap $1 $2 --output gap-analysis.md
/content-calendar --duration 3months --export calendar.csv

echo "Audit complete. Check *-audit.md files for results."
```

### Batch Processing

```bash
# Process multiple URLs
while IFS= read -r url; do
  /page-speed-seo "$url" --output "speed-$(echo $url | md5sum | cut -d' ' -f1).md"
done < urls.txt
```

## Additional Resources

- Source repository: https://github.com/alirezarezvani/claude-code-skill-factory
- Issue tracker: https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo/issues
- License: MIT
