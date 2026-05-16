---
name: r16-voltagent-seo-content-marketing-agent-skills
description: SEO & content marketing agent skill suite with keyword research, technical audits, SERP analysis, and content strategy automation
triggers:
  - "help me with SEO analysis"
  - "run a content audit on this site"
  - "do keyword research for this topic"
  - "check technical SEO issues"
  - "analyze SERP competitors"
  - "create an SEO content brief"
  - "build a content calendar"
  - "find backlink opportunities"
---

# 📈 SEO & Content Marketing Agent Skills

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides specialized SEO and content marketing capabilities derived from the VoltAgent/awesome-agent-skills framework. It includes 10 domain-specific commands and 5 multi-step workflows for keyword research, technical SEO audits, competitor analysis, content strategy, and link building.

## What This Skill Does

- **Keyword Research** — Cluster keywords, score opportunities, map search intent
- **Content Audits** — Quality scoring, duplication detection, cannibalization analysis
- **Technical SEO** — Crawl budget, Core Web Vitals, schema markup validation
- **Competitor Analysis** — Backlink gaps, topic gaps, SERP feature opportunities
- **Content Strategy** — AI-generated briefs, editorial calendars, refresh workflows
- **Local SEO** — NAP consistency, GBP optimization, citation audits
- **Link Building** — Prospect discovery, outreach automation, authority building

## Installation

```bash
# Clone the skill suite
git clone https://github.com/Gravityaespot/r16-voltagent-awesome-agent-skills-seo.git

# Install in Claude Code skills directory
mkdir -p ~/.claude/skills/
cp -r r16-voltagent-awesome-agent-skills-seo ~/.claude/skills/seo-marketing/

# Or copy the SKILL.md directly
cp SKILL.md ~/.claude/skills/seo-marketing.md
```

**Register the skill in your agent session:**

```bash
# In Claude Code or compatible agent environment
/read ~/.claude/skills/seo-marketing/SKILL.md
```

## Core Commands

### Keyword Research

Analyze keyword opportunities with clustering, intent mapping, and difficulty scoring.

```bash
# Basic keyword research
/keyword-research "sustainable fashion"

# With advanced options
/keyword-research "sustainable fashion" \
  --country US \
  --language en \
  --volume-min 100 \
  --difficulty-max 50 \
  --intent commercial,informational \
  --output json
```

**Output structure:**
- Primary keyword metrics (volume, CPC, difficulty)
- Keyword clusters by topic/intent
- SERP feature opportunities
- Seasonality trends
- Competitive landscape scoring

### Content Audit

Full-site content quality analysis with actionable recommendations.

```bash
# Basic content audit
/content-audit https://example.com

# Scoped audit with filters
/content-audit https://example.com \
  --scope /blog \
  --min-words 300 \
  --check-duplicates \
  --check-cannibalization \
  --output markdown
```

**Audit includes:**
- Thin content detection (<300 words)
- Duplicate content fingerprinting
- Keyword cannibalization matrix
- Content quality scores (readability, structure, entities)
- Orphan page identification
- Internal linking opportunities

### Technical SEO Audit

Comprehensive technical health check with Core Web Vitals.

```bash
# Full technical audit
/technical-seo https://example.com

# Specific checks
/technical-seo https://example.com \
  --checks crawlability,mobile,speed,schema \
  --include-sitemap \
  --check-robots \
  --verify-https
```

**Checks performed:**
- Crawlability (robots.txt, noindex, canonicals)
- Core Web Vitals (LCP, FID, CLS)
- Mobile-friendliness (viewport, tap targets)
- Schema markup validation
- Sitemap integrity
- HTTPS/security headers
- Redirect chains
- Broken links

### Competitor Gap Analysis

Identify content and backlink opportunities vs. competitors.

```bash
# Competitor gap analysis
/competitor-gap \
  --target https://example.com \
  --competitors https://competitor1.com,https://competitor2.com \
  --type backlinks,content,features

# Backlink gap only
/competitor-gap \
  --target example.com \
  --competitors competitor1.com,competitor2.com \
  --type backlinks \
  --min-dr 30 \
  --output csv
```

**Analysis includes:**
- Backlink gap (unique referring domains)
- Topic gap (competitor content not covered)
- SERP feature gap (featured snippets, PAA, images)
- Keyword gap (ranking keywords you're missing)
- Content depth comparison

### SEO Content Brief Generation

AI-generated content briefs optimized for target keywords.

```bash
# Generate content brief
/content-brief "how to start a sustainable fashion brand"

# With custom parameters
/content-brief "how to start a sustainable fashion brand" \
  --word-count 2500 \
  --competitors 5 \
  --include-nlp-terms \
  --include-questions \
  --outline-depth 3 \
  --output markdown
```

**Brief includes:**
- Target keyword and variants
- Search intent analysis
- Recommended word count
- H2/H3 outline structure
- NLP entities to include
- People Also Ask questions
- Related searches
- Image/media recommendations
- Internal linking suggestions
- Meta title/description templates

### SERP Monitoring

Track rankings with volatility alerts and optimization tips.

```bash
# Setup SERP monitoring
/serp-monitor setup \
  --domain example.com \
  --keywords keywords.csv \
  --frequency daily \
  --competitors 3

# Generate ranking report
/serp-monitor report \
  --period 30d \
  --show-volatility \
  --show-features \
  --output dashboard
```

**Monitoring features:**
- Daily rank tracking (desktop + mobile)
- SERP volatility detection
- Featured snippet tracking
- CTR opportunity scoring
- Position change alerts
- Competitor movement tracking

### Link Prospecting

Discover high-quality backlink opportunities with outreach templates.

```bash
# Find link prospects
/link-prospecting \
  --topic "sustainable fashion" \
  --min-da 30 \
  --max-da 70 \
  --types blog,resource,directory \
  --country US \
  --limit 100

# With outreach templates
/link-prospecting \
  --topic "sustainable fashion" \
  --min-da 30 \
  --generate-templates \
  --personalization high \
  --output csv
```

**Prospect scoring:**
- Domain Authority / Domain Rating
- Relevance score (topical alignment)
- Traffic estimates
- Outreach difficulty
- Contact information
- Pre-written outreach templates

### Page Speed SEO Analysis

Diagnose speed issues mapped to ranking impact.

```bash
# Page speed SEO analysis
/page-speed-seo https://example.com/page

# With specific metrics
/page-speed-seo https://example.com/page \
  --device mobile \
  --metrics lcp,fid,cls,fcp,ttfb \
  --check-render-blocking \
  --check-images \
  --output report
```

**Analysis includes:**
- Core Web Vitals scoring
- Render-blocking resources
- Image optimization opportunities
- JavaScript execution time
- CSS delivery optimization
- Third-party script impact
- Server response time (TTFB)
- Ranking impact estimation

### Local SEO Audit

NAP consistency, GBP optimization, and citation building.

```bash
# Local SEO audit
/local-seo \
  --business "Example Coffee Shop" \
  --location "Seattle, WA" \
  --check-gbp \
  --check-citations \
  --check-reviews

# Citation discovery
/local-seo citations \
  --business "Example Coffee Shop" \
  --location "Seattle, WA" \
  --find-opportunities \
  --check-consistency \
  --output spreadsheet
```

**Audit includes:**
- Google Business Profile completeness
- NAP consistency across web
- Citation building opportunities
- Review sentiment analysis
- Local pack ranking factors
- Schema markup for local business
- Competitor local presence

### Content Calendar Generation

Data-driven editorial calendar from search demand.

```bash
# Generate content calendar
/content-calendar \
  --topics "sustainable fashion,eco-friendly clothing" \
  --period 3months \
  --frequency 2-per-week \
  --include-seasonality \
  --include-trends \
  --output calendar

# With keyword research integration
/content-calendar \
  --seed-keywords keywords.csv \
  --period 6months \
  --posts-per-month 8 \
  --mix informational:70,commercial:30 \
  --output google-sheets
```

**Calendar includes:**
- Topic selection by search volume
- Seasonal trend alignment
- Publishing date recommendations
- Target keyword for each piece
- Content type (guide, listicle, how-to)
- Estimated effort (hours)
- Priority scoring

## Workflows (Multi-Step)

### Full SEO Sprint

12-step comprehensive SEO audit and action plan.

```bash
/workflows:full-seo-sprint https://example.com \
  --scope full \
  --include-competitors 3 \
  --output-format report
```

**Workflow steps:**
1. Technical SEO audit
2. On-page content audit
3. Keyword research & gap analysis
4. Competitor analysis
5. Backlink profile audit
6. SERP feature opportunities
7. Content cannibalization check
8. Internal linking optimization
9. Schema markup recommendations
10. Speed & Core Web Vitals
11. Action plan generation (prioritized)
12. Progress tracking setup

### Launch SEO Checklist

Pre-launch SEO validation workflow.

```bash
/workflows:launch-seo https://staging.example.com \
  --production-url https://example.com \
  --check-redirects \
  --check-canonicals \
  --verify-sitemap
```

**Checklist includes:**
- Robots.txt validation
- Sitemap generation & submission
- Canonical tag verification
- Hreflang implementation (if multilingual)
- 301 redirect mapping
- Structured data validation
- Meta tags audit
- Mobile responsiveness
- Page speed baseline
- Analytics/tracking setup

### Content Refresh Workflow

Identify and optimize underperforming content.

```bash
/workflows:content-refresh https://example.com \
  --period 12months \
  --traffic-drop-threshold 30% \
  --ranking-drop-threshold 5
```

**Workflow steps:**
1. Identify declining pages (traffic/rankings)
2. Analyze SERP changes
3. Competitor content analysis
4. Content gap identification
5. Generate refresh recommendations
6. Create updated outlines
7. Track post-refresh performance

### Authority Building Campaign

End-to-end link building and digital PR.

```bash
/workflows:authority-building \
  --domain example.com \
  --target-dr 50 \
  --duration 6months \
  --budget 5000
```

**Campaign phases:**
1. Competitor backlink analysis
2. Link-worthy asset ideation
3. Prospect list building
4. Outreach template creation
5. Campaign execution tracking
6. Follow-up automation
7. Link acquisition monitoring
8. Impact measurement

### AI Content Pipeline

Automated keyword-to-publish content workflow.

```bash
/workflows:ai-content-pipeline \
  --keywords keywords.csv \
  --output-folder ./content \
  --review-before-publish \
  --seo-optimize
```

**Pipeline stages:**
1. Keyword selection & clustering
2. Search intent analysis
3. Brief generation
4. Content drafting (AI-assisted)
5. SEO optimization pass
6. Readability enhancement
7. Fact-checking & citations
8. Meta data generation
9. Internal linking suggestions
10. Publishing/scheduling

## Configuration

Create a configuration file for persistent settings:

```json
{
  "seo_config": {
    "default_country": "US",
    "default_language": "en",
    "api_keys": {
      "serp_api": "${SERP_API_KEY}",
      "ahrefs": "${AHREFS_API_KEY}",
      "semrush": "${SEMRUSH_API_KEY}",
      "screaming_frog": "${SCREAMING_FROG_API_KEY}"
    },
    "thresholds": {
      "thin_content_words": 300,
      "keyword_difficulty_high": 60,
      "min_domain_authority": 20,
      "core_web_vitals": {
        "lcp_good": 2.5,
        "fid_good": 100,
        "cls_good": 0.1
      }
    },
    "output": {
      "default_format": "markdown",
      "progress_bars": true,
      "color_output": true
    }
  }
}
```

Save as `~/.claude/skills/seo-marketing/config.json`

## Environment Variables

Set these for API integrations:

```bash
# SERP data
export SERP_API_KEY="your_key_here"

# SEO tool APIs
export AHREFS_API_KEY="your_key_here"
export SEMRUSH_API_KEY="your_key_here"
export MOZ_API_KEY="your_key_here"

# Crawling
export SCREAMING_FROG_API_KEY="your_key_here"

# Analytics
export GOOGLE_ANALYTICS_CREDENTIALS="/path/to/credentials.json"
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS="/path/to/credentials.json"
```

## Common Patterns

### Pattern 1: New Site SEO Setup

```bash
# 1. Run pre-launch checklist
/workflows:launch-seo https://newsite.com \
  --production-ready-check

# 2. Setup baseline tracking
/serp-monitor setup \
  --domain newsite.com \
  --keywords initial-keywords.csv

# 3. Generate first content batch
/content-calendar \
  --topics "core topics" \
  --period 3months \
  --posts-per-month 8
```

### Pattern 2: Declining Traffic Investigation

```bash
# 1. Run full technical audit
/technical-seo https://example.com \
  --scope full

# 2. Check for algorithmic issues
/content-audit https://example.com \
  --check-quality \
  --check-thin-content

# 3. Analyze competitor gains
/competitor-gap \
  --target example.com \
  --competitors competitor1.com,competitor2.com

# 4. Create recovery plan
/workflows:content-refresh https://example.com \
  --period 6months
```

### Pattern 3: Content Scale-Up

```bash
# 1. Research keyword opportunities
/keyword-research "industry topic" \
  --volume-min 500 \
  --difficulty-max 40 \
  --output csv

# 2. Generate content calendar
/content-calendar \
  --seed-keywords keywords.csv \
  --period 6months \
  --posts-per-month 16

# 3. Setup automated pipeline
/workflows:ai-content-pipeline \
  --keywords keywords.csv \
  --review-before-publish
```

### Pattern 4: Authority Building

```bash
# 1. Analyze current backlink profile
/competitor-gap \
  --target example.com \
  --competitors competitor1.com \
  --type backlinks

# 2. Find link prospects
/link-prospecting \
  --topic "your niche" \
  --min-da 30 \
  --limit 200

# 3. Launch campaign
/workflows:authority-building \
  --domain example.com \
  --target-dr 50 \
  --duration 3months
```

## Structured Output Format

All commands return structured output following this pattern:

```
╔══════════════════════════════════════════════════╗
║  [COMMAND NAME]  —  [TARGET]                     ║
╠══════════════════════════════════════════════════╣
║  [PROGRESS BAR]                                  ║
╚══════════════════════════════════════════════════╝

┌─────────────────────┬──────────┬──────────┬──────────┐
│ Metric              │ Current  │ Target   │ Status   │
├─────────────────────┼──────────┼──────────┼──────────┤
│ [Metric rows...]                                     │
└─────────────────────┴──────────┴──────────┴──────────┘

🔴 Critical Issues (Fix Immediately)
  • Issue 1 with impact details
  • Issue 2 with impact details

🟠 High Priority (This Week)
  • Issue 3 with impact details

🟡 Medium Priority (This Month)
  • Issue 4 with impact details

🟢 Low Priority / Enhancements
  • Issue 5 with impact details

Next Steps:
  1. [Recommended action]
  2. [Recommended action]

Suggested Follow-up Commands:
  /command-name [args]
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```bash
# Add delay between requests
/keyword-research "topic" --rate-limit 1req/sec

# Use cached data when available
/serp-monitor report --use-cache --cache-max-age 24h
```

### Large Site Audits

For sites with 10k+ pages:

```bash
# Use sampling for initial audit
/content-audit https://example.com \
  --sample-size 1000 \
  --sample-method stratified

# Or scope to specific sections
/content-audit https://example.com \
  --scope /blog,/products \
  --exclude /category,/tag
```

### Missing Data Sources

If API keys aren't configured:

```bash
# Run with mock data for testing
/keyword-research "topic" --mock-mode

# Or use limited free data
/competitor-gap \
  --target example.com \
  --competitors competitor.com \
  --data-source free
```

### Slow Performance

Speed up analysis:

```bash
# Reduce competitor count
/competitor-gap --competitors competitor1.com --max-competitors 1

# Skip expensive checks
/technical-seo https://example.com \
  --skip crawl,backlinks \
  --fast-mode

# Parallel processing
/content-audit https://example.com --parallel 4
```

## Integration Examples

### Export to Google Sheets

```bash
/keyword-research "topic" \
  --output google-sheets \
  --sheet-id "${GOOGLE_SHEET_ID}" \
  --credentials "${GOOGLE_CREDENTIALS_PATH}"
```

### Scheduled Reporting

```bash
# Setup daily ranking report via cron
0 8 * * * /usr/local/bin/claude-agent \
  "/serp-monitor report --email team@example.com"
```

### CI/CD Pre-Deploy Checks

```bash
# In deployment pipeline
/workflows:launch-seo https://staging.example.com \
  --production-url https://example.com \
  --fail-on-critical \
  --output ci-report
```

## Best Practices

1. **Start with Technical Foundation** — Fix crawlability and indexing before optimizing content
2. **Prioritize by Impact** — Use color-coded severity (🔴🟠🟡🟢) to focus on high-ROI fixes
3. **Validate Before Scaling** — Test content strategy with small batch before full calendar
4. **Track Everything** — Setup monitoring before making changes to measure impact
5. **Iterate Based on Data** — Review reports weekly, adjust strategy monthly

## Additional Resources

- [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) — Source framework
- SEO tool API documentation (Ahrefs, SEMrush, Moz)
- Google Search Central guidelines
- Core Web Vitals optimization guides

---

**License:** MIT — Free to use, modify, and distribute.
