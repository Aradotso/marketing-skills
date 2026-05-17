---
name: seo-content-marketing-skill-suite
description: SEO & content marketing automation with keyword research, technical audits, SERP analysis, and content strategy workflows
triggers:
  - "run an SEO audit on this site"
  - "do keyword research for this topic"
  - "analyze competitor content gaps"
  - "create a content brief with SEO optimization"
  - "check technical SEO issues"
  - "build a content calendar based on search demand"
  - "analyze SERP rankings and opportunities"
  - "find backlink prospects for outreach"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive command-line toolkit for SEO professionals and content marketers. Provides 10 specialized slash commands and 5 multi-step workflows for keyword research, technical audits, competitive analysis, and content strategy. Derived from [alirezarezvani/claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory) with enhanced structured output and domain-specific tooling.

## What This Project Does

Automates SEO and content marketing workflows:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema markup validation
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — AI-generated briefs, editorial calendars, refresh workflows
- **SERP Monitoring** — rank tracking, volatility alerts, CTR optimization
- **Link Building** — prospect discovery, outreach templates, authority scoring

All commands use a consistent structured-output UI with progress tracking, prioritized findings, and actionable recommendations.

## Installation

### Clone into Claude Code Skills Directory

```bash
# Standard Claude Code skills location
mkdir -p ~/.claude/skills
cd ~/.claude/skills

# Clone this repository
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git seo-marketing

# Register the skill in your Claude Code session
# In Claude Code:
/read ~/.claude/skills/seo-marketing/SKILL.md
```

### Manual Installation

```bash
# Download and extract
curl -L https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo/archive/main.zip -o seo-marketing.zip
unzip seo-marketing.zip
mv r04-alirezarezvani-claude-code-skill-factory-seo-main ~/.claude/skills/seo-marketing
```

### Environment Setup

Create a `.env` file in your project root:

```bash
# API Keys (use your own providers)
SERP_API_KEY=your_serpapi_key_here
AHREFS_API_KEY=your_ahrefs_key_here
SEMRUSH_API_KEY=your_semrush_key_here

# Optional: Custom Configuration
SEO_CRAWL_DEPTH=3
SEO_MAX_PAGES=5000
SEO_USER_AGENT="SEO-Audit-Bot/1.0"
```

## Core Commands

### 1. Keyword Research

Discovers keyword opportunities with clustering and intent analysis.

```bash
# Basic usage
/keyword-research "project management software"

# With advanced options
/keyword-research "project management software" --country us --depth 3 --min-volume 100

# Export to CSV
/keyword-research "project management software" --output csv --file keywords.csv
```

**Output Structure:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research — "project management software" ║
╠══════════════════════════════════════════════════╣
║  Fetching seed keywords …    [██████████] 100%   ║
║  Clustering by intent …      [██████████] 100%   ║
║  Scoring opportunities …     [██████████] 100%   ║
╚══════════════════════════════════════════════════╝

┌─────────────────────────────┬────────┬────────┬──────────┬──────────┐
│ Keyword                     │ Volume │ KD     │ Intent   │ Priority │
├─────────────────────────────┼────────┼────────┼──────────┼──────────┤
│ project management software │ 33,100 │ 72     │ Commercial│ 🔴 High │
│ best project management tool│ 18,200 │ 58     │ Commercial│ 🟠 Med  │
│ free project management app │ 12,400 │ 42     │ Commercial│ 🟢 Quick│
│ project management tutorial │  8,900 │ 34     │ Info     │ 🟢 Quick│
└─────────────────────────────┴────────┴────────┴──────────┴──────────┘
```

### 2. Content Audit

Analyzes existing content for quality, duplication, and optimization opportunities.

```bash
# Full site audit
/content-audit --scope full --domain example.com

# Specific section
/content-audit --scope /blog --domain example.com

# With cannibalization detection
/content-audit --scope full --domain example.com --check-cannibalization
```

**Configuration Options:**

```javascript
// .seo-config.json
{
  "contentAudit": {
    "minWordCount": 300,
    "maxDuplicationThreshold": 0.15,
    "qualityFactors": ["readability", "images", "links", "freshness"],
    "excludePaths": ["/admin", "/api"]
  }
}
```

### 3. Technical SEO Audit

Comprehensive technical health check.

```bash
# Full technical audit
/technical-seo example.com

# Focus on specific issues
/technical-seo example.com --check "core-web-vitals,schema,indexability"

# Generate PDF report
/technical-seo example.com --report pdf --output audit-report.pdf
```

**Programmatic Usage (Python):**

```python
from seo_suite import TechnicalAudit

audit = TechnicalAudit(
    domain="example.com",
    user_agent=os.getenv("SEO_USER_AGENT"),
    max_pages=5000
)

# Run audit
results = audit.run(
    checks=["crawlability", "core_web_vitals", "schema", "mobile"]
)

# Access findings
for issue in results.critical_issues:
    print(f"🔴 {issue.type}: {issue.description}")
    print(f"   Affected URLs: {len(issue.urls)}")
    print(f"   Fix: {issue.recommendation}\n")

# Export
audit.export("audit-results.json", format="json")
```

### 4. Competitor Gap Analysis

Identifies content and backlink opportunities from competitors.

```bash
# Analyze multiple competitors
/competitor-gap --domain example.com --competitors "competitor1.com,competitor2.com"

# Focus on specific gap types
/competitor-gap --domain example.com --competitors "competitor1.com" --type "keywords,backlinks"

# Export opportunities
/competitor-gap --domain example.com --competitors "competitor1.com" --output opportunities.csv
```

**JavaScript/Node.js Usage:**

```javascript
const { CompetitorAnalysis } = require('seo-suite');

const analysis = new CompetitorAnalysis({
  domain: 'example.com',
  competitors: ['competitor1.com', 'competitor2.com'],
  apiKey: process.env.AHREFS_API_KEY
});

// Find keyword gaps
const keywordGaps = await analysis.findKeywordGaps({
  minVolume: 500,
  maxDifficulty: 60
});

// Find backlink opportunities
const backlinkGaps = await analysis.findBacklinkGaps({
  minDR: 40,
  limit: 100
});

// Generate report
const report = analysis.generateReport({
  keywordGaps,
  backlinkGaps,
  format: 'markdown'
});

console.log(report);
```

### 5. Content Brief Generation

Creates data-driven SEO content briefs.

```bash
# Generate brief for target keyword
/content-brief "how to manage remote teams"

# Include competitor analysis
/content-brief "how to manage remote teams" --analyze-top 10

# Custom word count target
/content-brief "how to manage remote teams" --target-words 2500 --format md
```

**Generated Brief Structure:**

```markdown
# Content Brief: "how to manage remote teams"

## Target Metrics
- **Target Word Count:** 2,200–2,500 words
- **Search Volume:** 8,900/mo
- **Keyword Difficulty:** 42/100
- **Primary Intent:** Informational + How-to

## Recommended Outline
1. Introduction (150 words)
2. Challenges of Remote Team Management (400 words)
3. Essential Tools for Remote Teams (500 words)
4. Communication Best Practices (450 words)
5. Building Team Culture Remotely (400 words)
6. Performance Tracking & Accountability (350 words)
7. Conclusion & Action Steps (150 words)

## NLP Terms to Include
- asynchronous communication
- time zone management
- virtual team building
- remote collaboration tools
[... 20+ more terms]

## Competitor Analysis
Top 3 ranking pages analyzed:
- Page 1: 2,340 words, 12 images, DA 68
- Page 2: 1,890 words, 8 images, DA 72
- Page 3: 2,650 words, 15 images, DA 65
```

### 6. SERP Monitoring

Track rankings with volatility detection.

```bash
# Daily rank check
/serp-monitor --keywords "keyword1,keyword2,keyword3" --domain example.com

# Historical comparison
/serp-monitor --keywords "keyword1" --domain example.com --compare 30d

# Alert on significant changes
/serp-monitor --keywords "keyword1" --domain example.com --alert-threshold 3
```

### 7. Link Prospecting

Find high-quality backlink opportunities.

```bash
# Find prospects for topic
/link-prospecting "project management" --min-dr 40 --limit 100

# With outreach templates
/link-prospecting "project management" --generate-templates

# Export to CRM format
/link-prospecting "project management" --output csv --fields "domain,dr,email,template"
```

### 8. Page Speed SEO

Analyze performance issues impacting rankings.

```bash
# Full performance audit
/page-speed-seo https://example.com/page

# Focus on Core Web Vitals
/page-speed-seo https://example.com/page --check cwv

# Mobile-specific analysis
/page-speed-seo https://example.com/page --device mobile
```

### 9. Local SEO Audit

Optimize for local search.

```bash
# Full local SEO check
/local-seo "Business Name" --city "New York" --category "restaurant"

# NAP consistency check
/local-seo "Business Name" --check nap-consistency

# Citation audit
/local-seo "Business Name" --check citations --output missing-citations.csv
```

### 10. Content Calendar

Generate data-driven editorial calendars.

```bash
# 90-day calendar
/content-calendar --topics "project management,remote work" --days 90

# Seasonal optimization
/content-calendar --topics "tax preparation" --seasonal --year 2024

# Export to Google Sheets
/content-calendar --topics "fitness" --days 60 --export gsheets
```

## Multi-Step Workflows

### Full SEO Sprint

12-step comprehensive optimization workflow:

```bash
/workflows:full-seo-sprint example.com --scope full --duration 30d
```

**Workflow Steps:**
1. Technical audit baseline
2. Keyword research & mapping
3. Competitor gap analysis
4. Content audit & prioritization
5. On-page optimization
6. Content brief generation
7. Schema markup implementation
8. Internal linking optimization
9. Page speed improvements
10. Link building campaign
11. Tracking setup
12. 30-day performance review

### Launch SEO Checklist

Pre-launch validation:

```bash
/workflows:launch-seo example.com --checklist full
```

Validates:
- Canonical tags
- Hreflang (if applicable)
- Robots.txt
- XML sitemap
- Schema markup
- Core Web Vitals
- Mobile-friendliness
- SSL/HTTPS
- Redirect chains

### Content Refresh Workflow

Recover lost rankings:

```bash
/workflows:content-refresh --domain example.com --threshold -5
```

Identifies pages that:
- Dropped 5+ positions in last 90 days
- Have high-volume target keywords
- Are recoverable with optimization

Generates refresh plan with prioritization.

### Authority Building Campaign

End-to-end link building:

```bash
/workflows:authority-building --domain example.com --niche "saas" --duration 90d
```

**Campaign Steps:**
1. Competitor backlink analysis
2. Prospect discovery & scoring
3. Outreach template generation
4. Contact finding
5. Outreach execution tracking
6. Follow-up scheduling
7. Acquisition reporting

### AI Content Pipeline

Automated content production:

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --output-dir ./content
```

**Pipeline:**
1. Keyword → Brief generation
2. Brief → First draft (AI)
3. SEO optimization pass
4. Fact-checking & editing
5. Image optimization
6. Schema markup injection
7. Publish-ready export

## Configuration

### Global Configuration File

Create `.seo-config.json` in your project root:

```json
{
  "general": {
    "defaultCountry": "us",
    "defaultLanguage": "en",
    "userAgent": "SEO-Suite/1.0"
  },
  "crawling": {
    "maxDepth": 3,
    "maxPages": 5000,
    "respectRobotsTxt": true,
    "crawlDelay": 1000
  },
  "keywordResearch": {
    "minVolume": 50,
    "maxDifficulty": 80,
    "clusteringThreshold": 0.7
  },
  "contentAudit": {
    "minWordCount": 300,
    "qualityThresholds": {
      "excellent": 90,
      "good": 70,
      "fair": 50,
      "poor": 0
    }
  },
  "technicalSEO": {
    "coreWebVitals": {
      "lcp": 2.5,
      "fid": 100,
      "cls": 0.1
    }
  }
}
```

## Common Patterns

### 1. New Site Launch Checklist

```bash
# Pre-launch
/technical-seo newsite.com --report checklist
/workflows:launch-seo newsite.com

# Post-launch monitoring
/serp-monitor --keywords "brand,target-keyword" --domain newsite.com --frequency daily
```

### 2. Recovering from Traffic Drop

```bash
# Diagnose
/technical-seo affected-site.com --check indexability
/content-audit --scope full --domain affected-site.com

# Recovery
/workflows:content-refresh --domain affected-site.com --threshold -10
```

### 3. Monthly Reporting Automation

```bash
# Generate reports
/serp-monitor --keywords keywords.csv --domain client.com --compare 30d --output report-rankings.pdf
/technical-seo client.com --report pdf --output report-technical.pdf
/content-audit --scope full --domain client.com --output report-content.csv
```

### 4. Competitor Research Sprint

```bash
# Comprehensive competitive analysis
/competitor-gap --domain mysite.com --competitors "comp1.com,comp2.com,comp3.com"
/link-prospecting "industry topic" --competitors "comp1.com,comp2.com"
/keyword-research "industry topic" --analyze-competitors
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```bash
# Add delays between requests
export SEO_REQUEST_DELAY=2000  # milliseconds

# Use caching
/keyword-research "topic" --use-cache --cache-duration 7d
```

### Large Site Crawls Timing Out

```bash
# Reduce scope
/technical-seo bigsite.com --max-pages 1000 --depth 2

# Or use sampling
/content-audit --scope full --domain bigsite.com --sample 20
```

### Missing API Keys

All commands requiring external APIs will prompt for keys:

```bash
# Set via environment
export SERP_API_KEY=your_key
export AHREFS_API_KEY=your_key

# Or via config
echo '{"apiKeys": {"serpApi": "your_key"}}' > .seo-config.json
```

### Progress Bar Not Updating

If running in non-TTY environment:

```bash
# Disable interactive mode
/technical-seo example.com --no-progress --output json
```

## Advanced Usage

### Scripting Multiple Audits

```bash
#!/bin/bash
# audit-all-clients.sh

for domain in $(cat clients.txt); do
  echo "Auditing $domain..."
  /technical-seo $domain --output "reports/${domain}-tech.json"
  /content-audit --domain $domain --output "reports/${domain}-content.json"
  /serp-monitor --domain $domain --keywords "brand" --output "reports/${domain}-ranks.json"
done

# Aggregate results
/workflows:generate-client-report --input-dir reports/ --output client-dashboard.html
```

### CI/CD Integration

```yaml
# .github/workflows/seo-monitoring.yml
name: Daily SEO Monitoring

on:
  schedule:
    - cron: '0 9 * * *'  # 9 AM daily

jobs:
  seo-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Technical SEO Check
        run: |
          /technical-seo ${{ secrets.SITE_DOMAIN }} --output seo-report.json
      - name: Alert on Critical Issues
        run: |
          if grep -q '"severity":"critical"' seo-report.json; then
            echo "Critical SEO issues found!"
            exit 1
          fi
```

## License

MIT — free to use, modify, and distribute.

---

**Source:** Adapted from [alirezarezvani/claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory)
