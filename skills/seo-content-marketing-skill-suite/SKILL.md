---
name: seo-content-marketing-skill-suite
description: SEO & content marketing automation with keyword research, content audits, SERP analysis, technical SEO checks and workflow orchestration
triggers:
  - analyze keywords for SEO
  - run content audit on website
  - check technical SEO issues
  - generate content brief for topic
  - analyze competitor SEO gaps
  - monitor SERP rankings
  - create content calendar from search data
  - audit local SEO setup
---

# SEO & Content Marketing Skill Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides structured SEO and content marketing automation derived from best-practice workflows. It offers 10 specialized commands and 5 multi-step workflows for keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy planning.

## What This Project Does

The suite automates common SEO and content marketing tasks through a consistent command interface:

- **Keyword Research**: Clustering, opportunity scoring, SERP intent mapping
- **Content Audits**: Quality scoring, duplication detection, cannibalization reports
- **Technical SEO**: Crawl budget, Core Web Vitals, schema markup, indexability
- **Competitor Analysis**: Backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy**: Brief generation, editorial calendars, refresh workflows
- **Monitoring**: Rank tracking, volatility alerts, CTR optimization

All commands use structured output with progress tracking, findings tables, action plans, and follow-up recommendations.

## Installation

```bash
# Clone into Claude skills directory
mkdir -p ~/.claude/skills/
cp -r . ~/.claude/skills/seo-content-marketing-skill-suite/

# Or install from repository
git clone https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo.git \
  ~/.claude/skills/seo-content-marketing-skill-suite/
```

**Register in Claude Code session:**

```bash
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md
```

## Core Commands

### Keyword Research

Deep keyword analysis with clustering and opportunity scoring:

```bash
/keyword-research <target>

# Examples
/keyword-research "project management software"
/keyword-research --domain example.com --export csv
/keyword-research "saas analytics" --volume-min 1000
```

**Output structure:**
- Keyword clusters by search intent
- Volume, difficulty, and opportunity scores
- SERP feature analysis (featured snippets, PAA, etc.)
- Priority ranking with traffic potential

### Content Audit

Full-site content quality and performance analysis:

```bash
/content-audit --scope <full|partial> --output <md|csv|json>

# Examples
/content-audit --scope full --output md
/content-audit --url https://example.com/blog --depth 2
/content-audit --check-duplicates --cannibalization-report
```

**Key metrics analyzed:**
- Content quality scores (thin content detection)
- Duplicate/near-duplicate detection
- Keyword cannibalization matrix
- Missing metadata (titles, descriptions, H1)
- Internal linking opportunities

### Technical SEO Audit

Infrastructure and technical optimization checks:

```bash
/technical-seo <domain>

# Examples
/technical-seo example.com
/technical-seo --check-schema --validate-hreflang
/technical-seo --core-web-vitals --mobile-first
```

**Audit areas:**
- Crawl budget optimization (robots.txt, XML sitemaps)
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Canonical and hreflang setup
- Mobile-friendliness and indexability

### Competitor Gap Analysis

Identify opportunities from competitor comparison:

```bash
/competitor-gap <your-domain> <competitor-domain>

# Examples
/competitor-gap example.com competitor.com
/competitor-gap --backlinks --topics --featured-snippets
/competitor-gap mysite.com rival.com --export report.md
```

**Analysis includes:**
- Backlink gap (DA/DR quality backlinks they have)
- Topic/keyword gap (ranking keywords you're missing)
- Featured snippet opportunities
- Content format gaps

### Content Brief Generation

AI-generated SEO content briefs:

```bash
/content-brief <topic>

# Examples
/content-brief "how to choose project management software"
/content-brief "b2b email marketing" --word-count 2000
/content-brief --target-keyword "seo tools" --serp-top-10
```

**Brief components:**
- Target keyword and related NLP terms
- Recommended outline with H2/H3 structure
- Word count targets based on SERP analysis
- Questions to answer (PAA integration)
- Internal linking recommendations

### SERP Monitoring

Track rankings with volatility alerts:

```bash
/serp-monitor <domain>

# Examples
/serp-monitor example.com --keywords keyword-list.csv
/serp-monitor --daily --alert-threshold 3
/serp-monitor mysite.com --ctr-analysis
```

**Tracking features:**
- Daily rank tracking for target keywords
- Volatility alerts (position changes > threshold)
- CTR optimization recommendations
- SERP feature wins/losses

### Link Prospecting

Quality backlink prospect identification:

```bash
/link-prospecting <niche>

# Examples
/link-prospecting "saas marketing"
/link-prospecting --da-min 30 --dr-min 25
/link-prospecting "project management" --templates
```

**Output:**
- Prospect list with DA/DR scores
- Contact information discovery
- Outreach email templates
- Relevance scoring

### Page Speed SEO Analysis

Performance diagnostics mapped to ranking impact:

```bash
/page-speed-seo <url>

# Examples
/page-speed-seo https://example.com
/page-speed-seo --mobile --desktop --lighthouse
/page-speed-seo mysite.com/page --render-blocking
```

**Diagnostics:**
- Render-blocking resources
- LCP, CLS, FID measurements
- Image optimization opportunities
- Critical path analysis
- Ranking impact estimation

### Local SEO Audit

Local search optimization checks:

```bash
/local-seo <business-name> <location>

# Examples
/local-seo "Acme Coffee" "Seattle, WA"
/local-seo --nap-consistency --gbp-optimize
/local-seo mybusiness seattle --citations
```

**Audit areas:**
- NAP (Name, Address, Phone) consistency
- Google Business Profile optimization
- Local citation audit
- Review management recommendations
- Local schema markup

### Content Calendar

Data-driven editorial planning:

```bash
/content-calendar --months <n>

# Examples
/content-calendar --months 3 --keywords keyword-list.csv
/content-calendar --seasonality --search-demand
/content-calendar Q1-2026 --export calendar.csv
```

**Calendar features:**
- Search demand forecasting
- Seasonality analysis
- Topic distribution
- Publishing frequency recommendations
- Content type mix

## Multi-Step Workflows

### Full SEO Sprint

12-step comprehensive SEO campaign:

```bash
/workflows:full-seo-sprint <target> --scope <full|focused>

# Example
/workflows:full-seo-sprint example.com --scope full
```

**Sprint steps:**
1. Technical SEO audit
2. Content audit
3. Competitor gap analysis
4. Keyword research and mapping
5. Content strategy planning
6. On-page optimization
7. Internal linking optimization
8. Schema markup implementation
9. Page speed optimization
10. Link building campaign
11. Monitoring setup
12. Reporting dashboard

### Launch SEO Checklist

Pre-launch SEO validation:

```bash
/workflows:launch-seo <domain>

# Example
/workflows:launch-seo newsite.com --checklist
```

**Checklist items:**
- Canonical URL setup
- Hreflang tags (if multi-language)
- XML sitemap submission
- Robots.txt configuration
- Google Search Console setup
- Analytics tracking
- Core Web Vitals baseline

### Content Refresh Workflow

Recover lost rankings through content updates:

```bash
/workflows:content-refresh <domain>

# Example
/workflows:content-refresh example.com --underperforming-threshold 50
```

**Workflow:**
1. Identify underperforming pages (rank drops)
2. Analyze current vs. top-ranking content
3. Generate refresh recommendations
4. Prioritize by traffic recovery potential
5. Track post-refresh performance

### Authority Building Campaign

End-to-end link building and digital PR:

```bash
/workflows:authority-building <niche>

# Example
/workflows:authority-building "saas marketing" --duration 90
```

**Campaign stages:**
1. Link gap analysis
2. Prospect identification
3. Content asset creation (linkable assets)
4. Outreach campaign execution
5. Relationship management
6. Link acquisition tracking

### AI Content Pipeline

Automated content production workflow:

```bash
/workflows:ai-content-pipeline <keyword-list>

# Example
/workflows:ai-content-pipeline keywords.csv --auto-publish false
```

**Pipeline stages:**
1. Keyword selection and prioritization
2. Content brief generation
3. AI-assisted draft creation
4. SEO optimization
5. Quality review
6. Publishing (manual approval or automated)

## Configuration

Create `.seo-config.json` in your project root:

```json
{
  "defaultDomain": "example.com",
  "apiKeys": {
    "searchConsole": "${GOOGLE_SEARCH_CONSOLE_API_KEY}",
    "semrush": "${SEMRUSH_API_KEY}",
    "ahrefs": "${AHREFS_API_KEY}",
    "moz": "${MOZ_API_KEY}"
  },
  "thresholds": {
    "minWordCount": 300,
    "minKeywordDensity": 0.5,
    "maxKeywordDensity": 2.5,
    "minDA": 20,
    "minDR": 15,
    "coreWebVitalsLCP": 2.5,
    "coreWebVitalsFID": 100,
    "coreWebVitalsCLS": 0.1
  },
  "workflows": {
    "fullSeoSprint": {
      "duration": 90,
      "checkpoints": [30, 60, 90]
    }
  },
  "reporting": {
    "format": "markdown",
    "includeCharts": true,
    "exportPath": "./seo-reports/"
  }
}
```

**Environment variables:**

```bash
export GOOGLE_SEARCH_CONSOLE_API_KEY="your-key"
export SEMRUSH_API_KEY="your-key"
export AHREFS_API_KEY="your-key"
export MOZ_API_KEY="your-key"
```

## Command Output Format

All commands follow a consistent 5-step structure:

```
① Scope Confirmation  — verify target and options
② Live Analysis       — progress bar with real-time updates
③ Findings Table      — structured results sorted by severity
④ Action Plan         — prioritized recommendations with time estimates
⑤ Next Steps          — suggested follow-up commands
```

**Severity indicators:**
- 🔴 Critical (immediate action required)
- 🟠 High (address within 1 week)
- 🟡 Medium (address within 1 month)
- 🟢 Low (nice-to-have improvements)

## Common Patterns

### Keyword Research → Content Brief → Publish

```bash
# 1. Research keywords
/keyword-research "saas analytics tools" --export keywords.csv

# 2. Generate content brief for top opportunity
/content-brief "best saas analytics tools 2026" --word-count 2500

# 3. Create draft (use brief output)
# ... content creation ...

# 4. Optimize before publish
/technical-seo --check-schema --validate-metadata
```

### Site Audit → Fix → Monitor

```bash
# 1. Full technical audit
/technical-seo example.com --scope full --output audit-report.md

# 2. Content quality check
/content-audit --scope full --cannibalization-report

# 3. Implement fixes based on priorities
# ... technical improvements ...

# 4. Set up monitoring
/serp-monitor example.com --keywords target-keywords.csv --daily
```

### Competitor Analysis → Gap Filling

```bash
# 1. Identify gaps
/competitor-gap mysite.com competitor.com --backlinks --topics

# 2. Prospect for similar backlinks
/link-prospecting "my-niche" --da-min 30 --similar-to competitor.com

# 3. Find missing topic opportunities
/keyword-research --competitor-keywords competitor.com --not-ranking mysite.com

# 4. Create content calendar
/content-calendar --months 6 --gap-keywords gap-list.csv
```

## Troubleshooting

### "Crawl rate limited"

Reduce crawl speed in config:

```json
{
  "crawl": {
    "delayMs": 1000,
    "maxConcurrent": 3
  }
}
```

### "API quota exceeded"

Commands fall back to free methods when API keys missing. For full features:

```bash
# Check which APIs are configured
grep -r "API_KEY" .seo-config.json

# Set missing keys
export SEMRUSH_API_KEY="your-key"
```

### "Core Web Vitals data unavailable"

Requires public site with sufficient traffic:

```bash
# Use Lighthouse instead
/page-speed-seo https://example.com --lighthouse --mobile
```

### "Keyword data not found"

Try broader match or synonyms:

```bash
# Use broader match type
/keyword-research "project management" --match-type broad

# Include synonyms
/keyword-research "pm software" --synonyms true
```

### "Workflow checkpoint failed"

Resume from last successful step:

```bash
# Check workflow state
cat .seo-workflows/full-seo-sprint-state.json

# Resume from step N
/workflows:full-seo-sprint example.com --resume-from 5
```

## Integration Examples

### Export to Google Sheets

```bash
/keyword-research "my topic" --export csv
# Then import CSV to Google Sheets via Apps Script
```

### Slack Alerts for Rank Changes

```bash
# Monitor with webhook
/serp-monitor example.com \
  --webhook "${SLACK_WEBHOOK_URL}" \
  --alert-threshold 3
```

### CI/CD SEO Testing

```bash
# Pre-deploy SEO validation
/technical-seo staging.example.com \
  --exit-code-on-errors \
  --critical-only
```

## Performance Notes

- **Keyword research**: ~30s for 100 keywords
- **Content audit**: ~2-5 min per 1000 pages
- **Technical SEO**: ~1-3 min for typical site
- **Competitor gap**: ~2-4 min depending on site size

Use `--scope partial` and `--depth` limits for faster results during development.

## Further Documentation

- Full command reference: See `/commands/` directory
- Workflow definitions: See `/workflows/` directory
- Original project: [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- Example reports: See `/examples/` directory
