---
name: seo-content-marketing-commands
description: SEO & content marketing command suite with keyword research, content audits, SERP analysis, and technical SEO workflows
triggers:
  - "run an SEO audit on this website"
  - "generate a content brief for target keywords"
  - "analyze competitor content gaps"
  - "create a content calendar based on search trends"
  - "check technical SEO issues and page speed"
  - "find backlink opportunities for our site"
  - "audit content for keyword cannibalization"
  - "build a keyword research cluster"
---

# SEO & Content Marketing Commands

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive suite of 10 specialized SEO commands and 5 multi-step workflows for keyword research, content audits, SERP analysis, technical SEO, and content strategy. Derived from wshobson/commands architecture with structured output, progress tracking, and actionable recommendations.

## What This Does

This skill suite provides production-ready commands for:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema validation, indexability
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Operations** — briefs, calendars, refresh workflows, publishing pipelines
- **Local SEO** — NAP consistency, Google Business Profile optimization, citations
- **Performance** — page speed diagnostics mapped to ranking impact

All commands output structured, prioritized action plans with time estimates.

## Installation

```bash
# Clone the repository
git clone https://github.com/Plateeocondense/r10-wshobson-commands-seo.git

# Move to your Claude skills directory
mkdir -p ~/.claude/skills
cp -r r10-wshobson-commands-seo ~/.claude/skills/

# Or install as a project dependency
npm install r10-wshobson-commands-seo
```

For Claude Code or Cursor integration:

```bash
# Register the skill
/read ~/.claude/skills/r10-wshobson-commands-seo/SKILL.md
```

## Core Commands

### `/keyword-research`

Deep keyword clustering with opportunity scoring and SERP intent analysis.

```bash
# Basic keyword research
/keyword-research "project management software"

# With filters and output format
/keyword-research "SaaS tools" \
  --min-volume 500 \
  --max-difficulty 40 \
  --intent commercial \
  --output json

# Multi-seed clustering
/keyword-research "CRM, customer management, sales software" \
  --cluster-threshold 0.7 \
  --output md
```

**Output Structure:**
- Keyword clusters (primary, secondary, long-tail)
- Search volume & difficulty scores
- SERP intent classification
- Opportunity score (volume × intent / difficulty)
- Recommended action per cluster

### `/content-audit`

Full-site content quality analysis with duplication and cannibalization detection.

```bash
# Site-wide audit
/content-audit https://example.com --scope full

# Specific section
/content-audit https://example.com/blog \
  --scope section \
  --min-word-count 300

# Focus on cannibalization
/content-audit https://example.com \
  --check cannibalization \
  --similarity-threshold 0.8
```

**Output:**
- Content inventory table (URL, word count, quality score, last updated)
- Duplicate/thin content flagged 🔴
- Cannibalization pairs with similarity %
- Priority refresh list
- Consolidation recommendations

### `/technical-seo`

Comprehensive technical SEO audit covering crawlability, performance, and markup.

```bash
# Full technical audit
/technical-seo https://example.com

# Focus areas
/technical-seo https://example.com \
  --checks "crawl,performance,schema" \
  --output detailed

# Mobile-first audit
/technical-seo https://example.com \
  --user-agent mobile \
  --check-vitals
```

**Checks:**
- Crawl budget analysis (robots.txt, XML sitemap, internal links)
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation (JSON-LD, Microdata)
- Indexability issues (canonical, noindex, redirects)
- Mobile usability
- HTTPS/security headers

### `/competitor-gap`

Identify content and backlink opportunities vs. competitors.

```bash
# Content gap analysis
/competitor-gap \
  --your-site example.com \
  --competitors "competitor1.com,competitor2.com,competitor3.com" \
  --gap-type content

# Backlink gap
/competitor-gap \
  --your-site example.com \
  --competitors "competitor1.com,competitor2.com" \
  --gap-type backlinks \
  --min-dr 30

# Featured snippet opportunities
/competitor-gap \
  --your-site example.com \
  --competitors "competitor1.com" \
  --gap-type featured-snippets
```

**Output:**
- Keywords competitors rank for (you don't)
- Backlink sources you're missing
- Featured snippet opportunities
- Content format gaps (video, tools, guides)

### `/content-brief`

AI-generated SEO content brief with structure, NLP terms, and optimization targets.

```bash
# Generate brief
/content-brief "how to use project management software"

# With specific target page
/content-brief "best CRM for small business" \
  --target-url example.com/crm-guide \
  --competitors "competitor1.com/crm,competitor2.com/best-crm"

# Custom outline depth
/content-brief "enterprise SaaS security" \
  --outline-depth 3 \
  --target-length 2500 \
  --format markdown
```

**Brief Includes:**
- Target keyword & variants
- Search intent analysis
- Recommended word count
- H2/H3 outline based on SERP analysis
- NLP terms to include (TF-IDF)
- Questions to answer (People Also Ask)
- Internal linking opportunities
- Media requirements (images, videos)

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization.

```bash
# Track keywords
/serp-monitor \
  --domain example.com \
  --keywords "keyword1,keyword2,keyword3" \
  --location "United States"

# With alerts
/serp-monitor \
  --domain example.com \
  --keywords-file keywords.txt \
  --alert-threshold 3 \
  --frequency daily
```

### `/link-prospecting`

Find quality backlink prospects with DA/DR filtering and outreach templates.

```bash
# Find prospects
/link-prospecting \
  --topic "project management" \
  --min-da 40 \
  --link-type "guest-post,resource-page"

# Generate outreach list
/link-prospecting \
  --competitor-backlinks competitor.com \
  --filter "dofollow" \
  --output csv
```

### `/page-speed-seo`

Performance diagnostics mapped to ranking impact.

```bash
# Analyze page speed
/page-speed-seo https://example.com/page

# With field data
/page-speed-seo https://example.com \
  --strategy mobile \
  --include-field-data

# Batch audit
/page-speed-seo --urls-file top-pages.txt \
  --threshold "good" \
  --output report.json
```

### `/local-seo`

NAP consistency, Google Business Profile optimization, local citations.

```bash
# Local audit
/local-seo \
  --business-name "Example Co" \
  --location "New York, NY" \
  --gmb-url "https://g.page/example"

# Citation check
/local-seo \
  --business-name "Example Co" \
  --check citations \
  --output csv
```

### `/content-calendar`

Data-driven editorial calendar based on search demand and seasonality.

```bash
# Generate calendar
/content-calendar \
  --keywords-file seeds.txt \
  --start-date 2026-06-01 \
  --duration 90 \
  --frequency weekly

# With trend analysis
/content-calendar \
  --topic "email marketing" \
  --include-trends \
  --format google-sheets
```

## Workflows (Multi-Step)

### `full-seo-sprint`

Complete 12-step SEO sprint from audit to execution plan.

```bash
/workflows:full-seo-sprint https://example.com \
  --scope full \
  --output sprint-plan.md
```

**Steps:**
1. Technical audit
2. Content inventory
3. Keyword gap analysis
4. Competitor benchmarking
5. Priority issues flagged
6. Keyword mapping to pages
7. Content refresh plan
8. New content briefs
9. Link building targets
10. On-page optimization checklist
11. Implementation timeline
12. Monitoring dashboard setup

### `launch-seo`

Pre-launch SEO validation checklist.

```bash
/workflows:launch-seo https://staging.example.com \
  --target-launch 2026-06-01
```

**Checks:**
- Canonical tags configured
- Hreflang for international (if applicable)
- XML sitemap generated & submitted
- Robots.txt configured
- 301 redirects mapped
- Schema markup validated
- Core Web Vitals passing
- Mobile usability verified

### `content-refresh`

Identify and refresh underperforming pages.

```bash
/workflows:content-refresh https://example.com \
  --traffic-drop-threshold 30 \
  --time-window 90days
```

**Process:**
1. Identify pages with traffic drops
2. Analyze SERP changes (new competitors, features)
3. Content freshness check
4. Keyword relevance audit
5. Generate refresh briefs
6. Prioritize by potential ROI

### `authority-building`

End-to-end digital PR and link building campaign.

```bash
/workflows:authority-building \
  --niche "SaaS project management" \
  --target-dr 50 \
  --duration 6months
```

**Campaign Steps:**
1. Link-worthy asset ideation (data studies, tools)
2. Competitor backlink analysis
3. Journalist/blogger prospect list
4. Outreach template creation
5. Pitch calendar
6. Follow-up automation setup
7. Link acquisition tracking

### `ai-content-pipeline`

Automated keyword → brief → draft → optimize → publish workflow.

```bash
/workflows:ai-content-pipeline \
  --keywords-file topics.csv \
  --output-dir ./content \
  --auto-publish false
```

**Pipeline:**
1. Keyword research & clustering
2. Brief generation per cluster
3. AI draft (with human review gate)
4. On-page optimization (meta, headers, internal links)
5. Image generation/sourcing
6. SEO validation (schema, readability)
7. Publish to CMS (optional)

## Configuration

Create a `.seo-commands.json` config file:

```json
{
  "defaults": {
    "domain": "example.com",
    "location": "United States",
    "language": "en",
    "output_format": "markdown"
  },
  "api_keys": {
    "semrush": "${SEMRUSH_API_KEY}",
    "ahrefs": "${AHREFS_API_KEY}",
    "screaming_frog": "${SF_LICENSE_KEY}",
    "google_search_console": "${GSC_CLIENT_ID}"
  },
  "thresholds": {
    "min_word_count": 300,
    "max_keyword_difficulty": 50,
    "min_domain_authority": 30,
    "core_web_vitals_threshold": "good"
  },
  "workflows": {
    "full_seo_sprint": {
      "depth": "comprehensive",
      "output": "markdown"
    }
  }
}
```

Environment variables:

```bash
export SEMRUSH_API_KEY="your_key_here"
export AHREFS_API_KEY="your_key_here"
export GSC_CLIENT_ID="your_client_id"
export GSC_CLIENT_SECRET="your_client_secret"
```

## Output Format Conventions

All commands follow this structure:

```
╔══════════════════════════════════════════════════╗
║  [COMMAND NAME]  —  [TARGET]                     ║
╠══════════════════════════════════════════════════╣
║  [Progress bars with percentages]                ║
╚══════════════════════════════════════════════════╝

┌─────────────────────┬──────────┬──────────┬──────────┐
│ [Findings Table]                                     │
└──────────────────────────────────────────────────────┘

🔴 Critical Issues (fix immediately)
🟠 Important Issues (fix this sprint)
🟡 Optimization Opportunities (backlog)
🟢 Passed Checks

📋 Action Plan:
  ✓ Quick Wins (< 1 day)
  ⏱ Medium-term (1-2 weeks)
  🎯 Strategic (> 1 month)

💡 Next Steps:
  Suggested follow-up commands
```

## Common Patterns

### Pattern: Monthly SEO Health Check

```bash
# Run on the 1st of each month
/technical-seo https://example.com --output monthly-tech-$(date +%Y%m).md
/content-audit https://example.com --scope changes --since 30days
/serp-monitor --domain example.com --keywords-file tracked.txt
```

### Pattern: New Content Creation

```bash
# Research → Brief → Draft workflow
/keyword-research "topic" --output keywords.json
/content-brief "primary keyword" --competitors auto --output brief.md
# (Human writes draft)
/technical-seo https://example.com/new-page --checks on-page
```

### Pattern: Competitor Analysis

```bash
# Full competitive intelligence
/competitor-gap \
  --your-site example.com \
  --competitors "comp1.com,comp2.com,comp3.com" \
  --gap-type all \
  --output competitor-analysis.json

/link-prospecting --competitor-backlinks comp1.com --min-da 40
```

### Pattern: Post-Launch Validation

```bash
/workflows:launch-seo https://newsite.com --checklist
/technical-seo https://newsite.com --checks "crawl,indexability,schema"
/page-speed-seo https://newsite.com --strategy both
```

## Troubleshooting

### "API rate limit exceeded"

Most SEO APIs have rate limits. Commands automatically respect limits, but for bulk operations:

```bash
# Add delay between requests
/keyword-research "topic" --rate-limit 1req/sec

# Or batch process
split -l 100 keywords.txt batch_
for file in batch_*; do
  /keyword-research --keywords-file $file --output ${file}.json
  sleep 60
done
```

### "Cannibalization not detected (false negative)"

Adjust similarity threshold:

```bash
# Lower threshold for more aggressive detection
/content-audit https://example.com \
  --check cannibalization \
  --similarity-threshold 0.6
```

### "Core Web Vitals data unavailable"

New pages lack field data. Use lab data:

```bash
/page-speed-seo https://example.com \
  --strategy mobile \
  --data-source lab
```

### "Competitor backlinks incomplete"

Combine multiple data sources:

```bash
/link-prospecting \
  --competitor-backlinks competitor.com \
  --sources "ahrefs,semrush,moz" \
  --deduplicate
```

## Integration Examples

### GitHub Actions (Monthly SEO Report)

```yaml
name: Monthly SEO Audit
on:
  schedule:
    - cron: '0 0 1 * *'  # 1st of month
jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run SEO commands
        env:
          SEMRUSH_API_KEY: ${{ secrets.SEMRUSH_API_KEY }}
        run: |
          /technical-seo https://example.com --output reports/tech-$(date +%Y%m).md
          /content-audit https://example.com --output reports/content-$(date +%Y%m).md
      - name: Commit reports
        run: |
          git add reports/
          git commit -m "Monthly SEO report $(date +%Y-%m)"
          git push
```

### Node.js Programmatic Usage

```javascript
const { keywordResearch, contentBrief } = require('r10-wshobson-commands-seo');

async function generateContentPipeline(seed) {
  // Research keywords
  const keywords = await keywordResearch(seed, {
    minVolume: 500,
    maxDifficulty: 40,
    clusterThreshold: 0.7
  });

  // Generate briefs for top clusters
  const briefs = await Promise.all(
    keywords.clusters.slice(0, 5).map(cluster =>
      contentBrief(cluster.primaryKeyword, {
        targetLength: 2000,
        competitors: 'auto',
        format: 'json'
      })
    )
  );

  return briefs;
}

generateContentPipeline('email marketing automation')
  .then(briefs => console.log(JSON.stringify(briefs, null, 2)));
```

### Python Script (Bulk Audit)

```python
import subprocess
import json
from pathlib import Path

def audit_sitemap(sitemap_url):
    """Audit all URLs in a sitemap."""
    # Fetch sitemap URLs
    result = subprocess.run(
        ['curl', '-s', sitemap_url],
        capture_output=True,
        text=True
    )
    
    # Extract URLs (simplified)
    urls = extract_urls_from_xml(result.stdout)
    
    # Audit each URL
    reports = []
    for url in urls[:50]:  # Limit to 50 for example
        audit = subprocess.run(
            ['/technical-seo', url, '--output', 'json'],
            capture_output=True,
            text=True
        )
        reports.append(json.loads(audit.stdout))
    
    return reports

if __name__ == '__main__':
    audits = audit_sitemap('https://example.com/sitemap.xml')
    Path('bulk-audit.json').write_text(json.dumps(audits, indent=2))
```

## Advanced Usage

### Custom Workflow Composition

Chain commands for complex workflows:

```bash
#!/bin/bash
# Full content refresh workflow

DOMAIN="example.com"
OUTPUT_DIR="./seo-refresh-$(date +%Y%m%d)"
mkdir -p $OUTPUT_DIR

# 1. Identify underperforming content
/content-audit https://$DOMAIN \
  --scope full \
  --output $OUTPUT_DIR/audit.json

# 2. Extract pages with traffic drops
jq '.pages[] | select(.traffic_change < -20) | .url' \
  $OUTPUT_DIR/audit.json > $OUTPUT_DIR/refresh-targets.txt

# 3. Generate refresh briefs
while read url; do
  /content-brief "$url" \
    --refresh-mode \
    --output $OUTPUT_DIR/briefs/$(basename $url).md
done < $OUTPUT_DIR/refresh-targets.txt

# 4. Technical check on refresh targets
/technical-seo \
  --urls-file $OUTPUT_DIR/refresh-targets.txt \
  --output $OUTPUT_DIR/tech-check.json

echo "✅ Refresh workflow complete. Results in $OUTPUT_DIR"
```

This skill enables comprehensive SEO and content marketing operations through consistent, actionable command interfaces with structured output for easy automation and reporting.
