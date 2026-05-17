---
name: seo-content-marketing-skill-suite
description: Comprehensive SEO and content marketing command suite with keyword research, technical audits, SERP analysis, and content optimization workflows
triggers:
  - analyze SEO performance for this site
  - run a keyword research report
  - audit technical SEO issues
  - create a content brief for this topic
  - check competitor SEO gaps
  - generate a content calendar
  - perform a full content audit
  - optimize page speed for SEO
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An AI-powered SEO and content marketing command suite derived from `shanraisshan/claude-code-best-practice`. Provides structured workflows for keyword research, technical SEO audits, content optimization, competitor analysis, and strategic planning with consistent UI/UX patterns.

## What This Skill Does

This skill suite enables AI coding agents to perform comprehensive SEO and content marketing analysis through:

- **Keyword Research** — clustering, intent mapping, opportunity scoring
- **Content Audits** — quality analysis, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema validation
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — AI-generated briefs, editorial calendars, refresh workflows
- **SERP Monitoring** — rank tracking, volatility alerts, CTR optimization
- **Link Building** — prospect identification, outreach templates, authority campaigns

Each command follows a 5-step interaction pattern with structured output, progress tracking, and actionable recommendations.

## Installation

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo.git

# Copy to Claude skills directory
cp -r r15-shanraisshan-claude-code-best-practice-seo ~/.claude/skills/seo-content-marketing/

# Register in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Quick Setup

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Navigate and clone
cd ~/.claude/skills
git clone https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo.git seo-content-marketing
```

## Core Commands

### Keyword Research

Perform deep keyword analysis with clustering and SERP intent mapping.

```bash
/keyword-research <target_keyword>

# Examples
/keyword-research "project management software"
/keyword-research "vegan recipes" --cluster by-intent
/keyword-research "best laptops" --output json
```

**Output Structure:**
- Primary keyword metrics (volume, difficulty, CPC)
- Semantic cluster map
- Search intent breakdown (informational/commercial/transactional)
- Long-tail opportunity scores
- Related questions and PAA data

### Content Audit

Full-site content quality assessment with cannibalization detection.

```bash
/content-audit --scope <full|sample|url-list>

# Examples
/content-audit --scope full
/content-audit --scope sample --limit 100
/content-audit --url-list urls.txt --output md
```

**Metrics Analyzed:**
- Content quality scores (uniqueness, depth, readability)
- Keyword cannibalization patterns
- Thin content identification
- Orphaned page detection
- Duplicate content percentage

### Technical SEO Audit

Comprehensive technical health check with Core Web Vitals analysis.

```bash
/technical-seo <domain>

# Examples
/technical-seo example.com
/technical-seo https://example.com --check-mobile
/technical-seo example.com --format report --export pdf
```

**Audit Points:**
- Crawlability and indexability
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- XML sitemap health
- Robots.txt analysis
- Canonical tag verification
- Mobile-friendliness score

### Competitor Gap Analysis

Identify content and backlink opportunities by comparing with competitors.

```bash
/competitor-gap <your-domain> <competitor-domains...>

# Examples
/competitor-gap example.com competitor1.com competitor2.com
/competitor-gap example.com competitor.com --focus backlinks
/competitor-gap example.com competitor.com --type topic-gap
```

**Gap Types:**
- Backlink gap (domains linking to competitors but not you)
- Topic gap (keywords competitors rank for)
- Featured snippet opportunities
- Content format gaps

### Content Brief Generator

AI-generated SEO content briefs with NLP term extraction.

```bash
/content-brief <topic> [--format <outline|full|minimal>]

# Examples
/content-brief "how to start a blog"
/content-brief "best CRM software 2026" --format full
/content-brief "vegan meal prep" --include-nlp-terms
```

**Brief Includes:**
- Target keyword and variants
- Recommended word count range
- H2/H3 outline structure
- NLP terms to include
- Competitor content analysis
- Target SERP features

### SERP Monitoring

Track keyword rankings with volatility alerts and CTR recommendations.

```bash
/serp-monitor <keyword-list>

# Examples
/serp-monitor "keyword1, keyword2, keyword3"
/serp-monitor --file keywords.txt --frequency daily
/serp-monitor --compare-to last-week
```

**Monitoring Features:**
- Daily rank positions
- SERP feature tracking (featured snippets, PAA, local pack)
- Volatility scoring
- CTR optimization suggestions
- Competitor movement alerts

### Link Prospecting

Generate quality backlink prospect lists with DA/DR filtering.

```bash
/link-prospecting <niche> [--min-da <score>]

# Examples
/link-prospecting "marketing blogs" --min-da 40
/link-prospecting "tech news sites" --min-da 50 --output csv
/link-prospecting "food bloggers" --location US --include-email
```

**Output Includes:**
- Domain authority / Domain rating
- Traffic estimates
- Niche relevance score
- Contact information
- Outreach email templates

### Page Speed SEO

Performance audit mapped to ranking impact factors.

```bash
/page-speed-seo <url>

# Examples
/page-speed-seo https://example.com
/page-speed-seo https://example.com/page --mobile
/page-speed-seo https://example.com --compare-to competitor.com
```

**Analysis Points:**
- Render-blocking resources
- Largest Contentful Paint (LCP)
- Cumulative Layout Shift (CLS)
- First Input Delay (FID)
- SEO ranking impact assessment
- Prioritized fix recommendations

### Local SEO Audit

NAP consistency check and Google Business Profile optimization.

```bash
/local-seo <business-name> [--location <city>]

# Examples
/local-seo "Joe's Pizza" --location "New York"
/local-seo "Smith Dental" --check-citations
/local-seo "ABC Plumbing" --gbp-optimize
```

**Audit Components:**
- NAP (Name, Address, Phone) consistency across directories
- Google Business Profile completeness
- Local citation audit
- Review sentiment analysis
- Local pack ranking factors

### Content Calendar

Data-driven editorial calendar based on search demand and seasonality.

```bash
/content-calendar --months <number> [--topics <list>]

# Examples
/content-calendar --months 3 --topics "SEO, content marketing"
/content-calendar --months 6 --auto-suggest
/content-calendar --year 2026 --industry tech --export google-sheets
```

**Calendar Features:**
- Search demand forecasting
- Seasonal trend alignment
- Topic clustering
- Publishing frequency recommendations
- Content format suggestions

## Multi-Step Workflows

### Full SEO Sprint

12-step comprehensive SEO campaign from audit to implementation.

```bash
/workflows:full-seo-sprint <domain> --scope <full|quick>

# Steps:
# 1. Technical audit
# 2. Content audit
# 3. Backlink profile analysis
# 4. Competitor gap analysis
# 5. Keyword opportunity mapping
# 6. Priority issue identification
# 7. Quick win recommendations
# 8. Content refresh list
# 9. New content topics
# 10. Technical fix roadmap
# 11. Link building strategy
# 12. 90-day action plan
```

### Launch SEO

Pre-launch SEO checklist and validation.

```bash
/workflows:launch-seo <staging-url>

# Validates:
# - Canonical tags
# - Hreflang implementation
# - XML sitemap generation
# - Robots.txt configuration
# - Schema markup
# - Meta tags
# - Page speed baseline
# - Mobile responsiveness
# - Indexability settings
```

### Content Refresh

Identify and optimize underperforming content.

```bash
/workflows:content-refresh --min-age <months> --max-position <rank>

# Process:
# 1. Identify declining pages (rankings dropped >5 positions)
# 2. Content quality assessment
# 3. Keyword intent re-alignment
# 4. Competitor content gap analysis
# 5. Refresh recommendations (update vs rewrite vs consolidate)
# 6. Optimization checklist per page
```

### Authority Building

End-to-end digital PR and link building campaign.

```bash
/workflows:authority-building <niche> --duration <months>

# Campaign includes:
# 1. Link-worthy asset ideation
# 2. Prospect list generation
# 3. Outreach template creation
# 4. Relationship mapping
# 5. Follow-up sequences
# 6. Success metrics tracking
# 7. ROI reporting
```

### AI Content Pipeline

Automated content production from keyword to publication.

```bash
/workflows:ai-content-pipeline --input <keyword-list>

# Pipeline stages:
# 1. Keyword prioritization
# 2. Brief generation
# 3. AI draft creation
# 4. SEO optimization (NLP terms, internal links, meta)
# 5. Quality review checklist
# 6. Publishing schedule
# 7. Performance tracking setup
```

## Configuration

### Environment Variables

Store API keys and configuration in environment variables:

```bash
# SEO tool integrations
export AHREFS_API_KEY="your_ahrefs_key"
export SEMRUSH_API_KEY="your_semrush_key"
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS="path/to/credentials.json"
export SCREAMING_FROG_LICENSE="your_license_key"

# AI content generation
export OPENAI_API_KEY="your_openai_key"
export ANTHROPIC_API_KEY="your_anthropic_key"

# Reporting
export GOOGLE_ANALYTICS_VIEW_ID="your_ga_view_id"
export GOOGLE_SHEETS_API_CREDENTIALS="path/to/sheets_credentials.json"
```

### Config File

Create `.seo-skills-config.json` in project root:

```json
{
  "default_domain": "example.com",
  "output_format": "markdown",
  "export_directory": "./seo-reports",
  "keyword_research": {
    "min_volume": 100,
    "max_difficulty": 70,
    "default_country": "US"
  },
  "technical_audit": {
    "check_mobile": true,
    "core_web_vitals_threshold": "good",
    "max_crawl_depth": 5
  },
  "content_audit": {
    "min_word_count": 300,
    "max_duplicate_percentage": 10,
    "quality_score_threshold": 60
  },
  "workflows": {
    "full_seo_sprint_duration_days": 90,
    "content_refresh_min_age_months": 6,
    "authority_building_outreach_volume": 50
  }
}
```

## Common Usage Patterns

### Quarterly SEO Review

```bash
# 1. Run full technical audit
/technical-seo example.com --export pdf

# 2. Content performance analysis
/content-audit --scope full --sort-by traffic-decline

# 3. Keyword opportunity refresh
/keyword-research --cluster by-intent --filter gaps

# 4. Competitor movement check
/competitor-gap example.com competitor1.com competitor2.com

# 5. Generate action plan
/workflows:full-seo-sprint example.com --scope quick
```

### New Content Campaign

```bash
# 1. Research topic cluster
/keyword-research "main topic keyword" --cluster semantic

# 2. Generate briefs for top 5 keywords
for keyword in keyword1 keyword2 keyword3 keyword4 keyword5; do
  /content-brief "$keyword" --format full
done

# 3. Create editorial calendar
/content-calendar --months 3 --topics "keyword cluster theme"

# 4. Set up tracking
/serp-monitor --file target-keywords.txt --frequency daily
```

### Recovery from Rankings Drop

```bash
# 1. Identify affected pages
/serp-monitor --compare-to last-month --filter declined

# 2. Content quality check
/content-audit --url-list declined-urls.txt

# 3. Technical issues scan
/technical-seo example.com --focus affected-pages

# 4. Competitor analysis
/competitor-gap example.com --keywords affected-keywords.txt

# 5. Refresh strategy
/workflows:content-refresh --url-list declined-urls.txt
```

## Output Formats

All commands support multiple output formats:

```bash
# Markdown (default)
/command --output md

# JSON for programmatic use
/command --output json

# CSV for spreadsheet import
/command --output csv

# PDF report
/command --export pdf --output-dir ./reports

# Google Sheets direct export
/command --export google-sheets --sheet-id SHEET_ID
```

## Structured Output Example

Every command displays progress and results in a consistent format:

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "project management"       ║
╠══════════════════════════════════════════════════╣
║  Fetching search volume …    [██████████] 100% ✓ ║
║  Clustering keywords …       [██████████] 100% ✓ ║
║  Analyzing SERP intent …     [████████░░]  80%   ║
╚══════════════════════════════════════════════════╝

PRIMARY KEYWORD METRICS
┌──────────────────────┬──────────┬────────────┬───────┐
│ Keyword              │ Volume   │ Difficulty │ Intent│
├──────────────────────┼──────────┼────────────┼───────┤
│ project management   │   74,000 │         72 │ Info  │
│ software             │          │            │       │
└──────────────────────┴──────────┴────────────┴───────┘

SEMANTIC CLUSTERS (Top 3)
┌────────────────────────────────────────────────────┐
│ 🔵 Tools & Software (23 keywords, avg vol: 12,400)│
│    • project management tools                      │
│    • best project management software              │
│    • free project management apps                  │
│                                                     │
│ 🟢 Methodology (18 keywords, avg vol: 8,200)      │
│    • project management methodology                │
│    • agile project management                      │
│    • waterfall vs agile                            │
│                                                     │
│ 🟡 Certification (15 keywords, avg vol: 5,600)    │
│    • PMP certification                             │
│    • project management professional               │
│    • how to become a project manager               │
└────────────────────────────────────────────────────┘

RECOMMENDED ACTIONS
✓ Quick Win   — Target "free project management apps" (low difficulty, high intent)
⚠ Medium Term — Create comparison content for "Tools & Software" cluster
🎯 Strategic  — Build authority content for certification queries

NEXT STEPS
→ /content-brief "free project management apps" --format full
→ /competitor-gap example.com --keywords tools-cluster.txt
→ /content-calendar --months 3 --topics "project management"
```

## Troubleshooting

### API Rate Limits

```bash
# If you hit rate limits, enable caching
export SEO_SKILLS_CACHE_ENABLED=true
export SEO_SKILLS_CACHE_TTL_HOURS=24

# Use sampling for large audits
/content-audit --scope sample --limit 500
/technical-seo example.com --max-pages 1000
```

### Slow Crawls

```bash
# Reduce crawl depth
/technical-seo example.com --max-depth 3

# Use parallel processing
export SEO_SKILLS_PARALLEL_REQUESTS=5

# Skip external resources
/technical-seo example.com --skip-external
```

### Missing Dependencies

```bash
# If command fails with "module not found"
# Check required integrations are configured
echo $AHREFS_API_KEY
echo $GOOGLE_SEARCH_CONSOLE_CREDENTIALS

# Verify API access
/test-connections --verbose
```

### Export Failures

```bash
# Check write permissions
ls -la ./seo-reports

# Set alternative export directory
/command --export-dir ~/Documents/seo-reports

# Use stdout if file export fails
/command --output json > output.json
```

## Integration Examples

### Export to Google Sheets

```bash
# Authenticate with Google Sheets API
export GOOGLE_SHEETS_API_CREDENTIALS="/path/to/credentials.json"

# Export directly
/keyword-research "topic" --export google-sheets --sheet-id "YOUR_SHEET_ID"
/content-audit --scope full --export google-sheets --create-new
```

### Continuous Monitoring

```bash
# Set up daily cron job
# crontab -e
0 9 * * * /path/to/claude --skill seo-content-marketing /serp-monitor --file keywords.txt --email-alerts admin@example.com
```

### Integrate with CI/CD

```bash
# Pre-deployment SEO check
#!/bin/bash
if /workflows:launch-seo https://staging.example.com --strict; then
  echo "SEO validation passed"
  exit 0
else
  echo "SEO issues detected, blocking deployment"
  exit 1
fi
```

## Best Practices

1. **Always verify domain ownership** before running audits on third-party sites
2. **Use sampling** for initial analysis of large sites (>10k pages)
3. **Export results** regularly to track progress over time
4. **Set up monitoring** for critical keywords immediately after optimization
5. **Review competitor analysis** monthly to stay ahead of market changes
6. **Refresh content audits** quarterly to catch quality decay early
7. **Validate technical fixes** after implementation using `/technical-seo`

## Resources

- **Source Repository**: https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo
- **Parent Project**: https://github.com/shanraisshan/claude-code-best-practice
- **License**: MIT

---

*Last updated: 2026-05-11*
