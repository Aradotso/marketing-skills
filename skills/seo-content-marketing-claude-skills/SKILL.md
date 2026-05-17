---
name: seo-content-marketing-claude-skills
description: SEO & content marketing command suite for keyword research, content audits, technical SEO, and content strategy workflows
triggers:
  - help me with keyword research and SEO analysis
  - I need to audit my website content for SEO
  - analyze competitors and find content gaps
  - create an SEO content brief
  - run a technical SEO audit on my site
  - build a content calendar based on search data
  - find backlink opportunities for my website
  - optimize page speed for better rankings
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite extends Claude Code with 10 specialized SEO and content marketing commands plus 5 multi-step workflows. It provides structured keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy planning with consistent progress tracking and actionable outputs.

## What This Project Does

The SEO & Content Marketing Skills Suite is a collection of slash commands and workflows designed to help AI coding agents assist with:

- **Keyword research** — clustering, opportunity scoring, SERP intent mapping
- **Content audits** — quality scoring, duplication detection, cannibalization analysis
- **Technical SEO** — crawl budget, Core Web Vitals, schema validation
- **Competitor analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content creation** — AI-generated briefs, outlines, NLP term extraction
- **Rank tracking** — SERP monitoring with volatility alerts
- **Link building** — prospect discovery, DA/DR filtering, outreach templates
- **Local SEO** — NAP consistency, GBP optimization, citation audits
- **Content planning** — data-driven editorial calendars

All commands follow a 5-step interaction pattern: scope confirmation → live analysis → findings table → action plan → next steps.

## Installation

### Quick Install

```bash
# Clone the repository
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git

# Copy to Claude skills directory
cp -r r04-alirezarezvani-claude-code-skill-factory-seo ~/.claude/skills/seo-content-marketing/

# Register in Claude Code session
# In Claude Code:
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Manual Setup

1. Download or clone the repository
2. Place the skill files in your Claude Code skills directory
3. Reference the SKILL.md file in a Claude Code session to load commands

## Core Commands

### `/keyword-research`

Performs deep keyword clustering and opportunity scoring with SERP intent mapping.

**Usage:**
```bash
/keyword-research <target_keyword>
/keyword-research "content marketing" --country us --language en
/keyword-research "saas analytics" --depth comprehensive --output json
```

**Options:**
- `--country` — target country code (default: us)
- `--language` — target language (default: en)
- `--depth` — analysis depth: quick | standard | comprehensive
- `--output` — output format: md | json | csv

**Example workflow:**
```bash
# Step 1: Analyze seed keyword
/keyword-research "email marketing automation"

# Agent will:
# ① Confirm target and options
# ② Display progress bar while fetching data
# ③ Show keyword clusters in table format
# ④ Provide prioritized action plan
# ⑤ Suggest next command (/content-brief)
```

**Expected output structure:**
```
┌──────────────────────────────┬────────┬──────┬────────────┬──────────┐
│ Keyword                      │ Volume │ KD   │ Intent     │ Priority │
├──────────────────────────────┼────────┼──────┼────────────┼──────────┤
│ email marketing automation   │ 14 800 │ 68   │ Commercial │  🟢 High │
│ best email automation tools  │  8 100 │ 45   │ Commercial │  🟢 High │
│ email drip campaigns         │  5 400 │ 52   │ Informational │ 🟡 Med │
│ automated email sequences    │  3 600 │ 41   │ Informational │ 🟢 High │
└──────────────────────────────┴────────┴──────┴────────────┴──────────┘
```

### `/content-audit`

Full-site content quality scoring, duplication check, and cannibalization report.

**Usage:**
```bash
/content-audit --scope full --output md
/content-audit --url https://example.com --depth 3
/content-audit --focus cannibalization
```

**Options:**
- `--scope` — audit scope: full | sample | urls
- `--url` — target URL or domain
- `--depth` — crawl depth (default: 3)
- `--focus` — specific audit: quality | duplication | cannibalization | all
- `--output` — output format: md | json | csv

**Example:**
```bash
/content-audit --url https://myblog.com --scope full

# Returns:
# - Pages with thin content (<300 words)
# - Duplicate title tags and meta descriptions
# - Keyword cannibalization clusters
# - Content quality scores (0-100)
# - Priority fix list
```

### `/technical-seo`

Crawl budget analysis, Core Web Vitals audit, schema markup validation, and indexability checks.

**Usage:**
```bash
/technical-seo --url https://example.com
/technical-seo --url https://example.com --check vitals,schema,crawl
/technical-seo --url https://example.com --output json
```

**Options:**
- `--url` — target URL or domain (required)
- `--check` — specific checks: vitals | schema | crawl | index | all
- `--output` — output format: md | json | csv

**Example output:**
```
Core Web Vitals:
┌──────────┬─────────┬────────┬────────┐
│ Metric   │ Current │ Target │ Status │
├──────────┼─────────┼────────┼────────┤
│ LCP      │ 2.1s    │ <2.5s  │ ✓ Pass │
│ FID      │ 85ms    │ <100ms │ ✓ Pass │
│ CLS      │ 0.18    │ <0.1   │ ✗ Fail │
└──────────┴─────────┴────────┴────────┘

🔴 Critical: CLS exceeds threshold
→ Check layout shifts in hero section
→ Reserve space for ad units
```

### `/competitor-gap`

Backlink gap, topic gap, and featured snippet opportunity analysis.

**Usage:**
```bash
/competitor-gap --yours https://mysite.com --theirs https://competitor.com
/competitor-gap --yours mysite.com --theirs competitor1.com,competitor2.com --type backlinks
```

**Options:**
- `--yours` — your domain (required)
- `--theirs` — competitor domain(s), comma-separated (required)
- `--type` — gap type: backlinks | topics | snippets | all
- `--output` — output format: md | json | csv

### `/content-brief`

AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Usage:**
```bash
/content-brief "how to choose email marketing software"
/content-brief "saas pricing strategies" --target-kw "saas pricing models" --depth comprehensive
```

**Options:**
- `--target-kw` — primary target keyword
- `--depth` — brief detail level: quick | standard | comprehensive
- `--format` — output format: md | json | docx

**Example brief structure:**
```markdown
# Content Brief: How to Choose Email Marketing Software

## Target Keyword
- Primary: "how to choose email marketing software"
- Volume: 2,400/mo | KD: 38 | Intent: Informational

## Recommended Structure
1. Introduction (150-200 words)
2. Key Features to Look For (800 words)
   - Automation capabilities
   - Segmentation options
   - Analytics and reporting
3. Pricing Considerations (400 words)
4. Top Recommendations (600 words)
5. Conclusion (150 words)

## Target Word Count: 2,100-2,400 words

## NLP Terms to Include
- email campaigns, subscriber list, open rate, click-through rate,
  A/B testing, deliverability, SMTP, ESP, marketing automation

## Competitor Analysis
- competitor1.com/article — 2,850 words, DA 68
- competitor2.com/guide — 1,920 words, DA 71
```

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization tips.

**Usage:**
```bash
/serp-monitor --keywords keywords.csv --url mysite.com
/serp-monitor --keywords "keyword1,keyword2,keyword3" --frequency daily
```

**Options:**
- `--keywords` — CSV file or comma-separated list
- `--url` — domain to track
- `--frequency` — check frequency: daily | weekly | monthly

### `/link-prospecting`

Quality backlink prospect discovery with DA/DR filters and outreach templates.

**Usage:**
```bash
/link-prospecting --niche "marketing automation" --min-da 40
/link-prospecting --niche "saas" --type guest-post --output csv
```

**Options:**
- `--niche` — target niche or topic (required)
- `--type` — link type: guest-post | resource-page | broken-link | all
- `--min-da` — minimum domain authority (default: 30)
- `--output` — output format: md | json | csv

### `/page-speed-seo`

Render-blocking diagnostics, LCP/CLS/FID analysis mapped to ranking impact.

**Usage:**
```bash
/page-speed-seo --url https://example.com/page
/page-speed-seo --url https://example.com --device mobile
```

**Options:**
- `--url` — target URL (required)
- `--device` — device type: desktop | mobile | both

### `/local-seo`

NAP consistency check, Google Business Profile optimization, local citation audit.

**Usage:**
```bash
/local-seo --business "Acme Marketing" --location "Austin, TX"
/local-seo --gmb-url https://business.google.com/... --audit citations
```

**Options:**
- `--business` — business name
- `--location` — city, state/region
- `--gmb-url` — Google Business Profile URL
- `--audit` — focus area: nap | gmb | citations | all

### `/content-calendar`

Data-driven editorial calendar built from search demand and seasonality.

**Usage:**
```bash
/content-calendar --niche "email marketing" --months 6
/content-calendar --topics topics.csv --start 2026-06 --format csv
```

**Options:**
- `--niche` — target niche
- `--topics` — CSV of topics/keywords
- `--months` — planning horizon (default: 3)
- `--start` — start month (YYYY-MM)
- `--format` — output format: md | json | csv | gcal

## Workflows

Workflows are multi-step processes that chain multiple commands together.

### `full-seo-sprint`

12-step SEO sprint: audit → keyword map → content plan → technical fixes.

**Usage:**
```bash
/workflows:full-seo-sprint https://example.com --scope full
```

**Steps:**
1. Technical SEO audit
2. Content audit
3. Keyword research
4. Competitor gap analysis
5. Content brief generation
6. Page speed diagnostics
7. Schema markup validation
8. Internal linking analysis
9. Content calendar creation
10. Link prospecting
11. Priority fix list
12. Implementation roadmap

### `launch-seo`

Pre-launch SEO checklist with canonical, hreflang, and sitemap validation.

**Usage:**
```bash
/workflows:launch-seo https://newsite.com
```

### `content-refresh`

Identify and refresh underperforming pages to recover lost rankings.

**Usage:**
```bash
/workflows:content-refresh --url https://example.com --threshold -20%
```

### `authority-building`

End-to-end digital PR and link-building campaign workflow.

**Usage:**
```bash
/workflows:authority-building --niche "B2B SaaS" --duration 90
```

### `ai-content-pipeline`

Keyword → brief → draft → optimize → publish automation pipeline.

**Usage:**
```bash
/workflows:ai-content-pipeline --keywords keywords.csv --output-dir ./content/
```

## Configuration

All commands support environment variables for API keys and default settings:

```bash
# Set API credentials (example for hypothetical integrations)
export SEO_TOOL_API_KEY=your_key_here
export SERP_API_KEY=your_key_here
export BACKLINK_API_KEY=your_key_here

# Set default options
export SEO_DEFAULT_COUNTRY=us
export SEO_DEFAULT_LANGUAGE=en
export SEO_OUTPUT_FORMAT=md
```

## Common Patterns

### Pattern 1: New Content Creation

```bash
# 1. Research keywords
/keyword-research "topic name" --depth comprehensive

# 2. Generate content brief
/content-brief "target keyword from research"

# 3. Add to content calendar
/content-calendar --topics "target keyword" --start 2026-06
```

### Pattern 2: Site Recovery

```bash
# 1. Full technical audit
/technical-seo --url https://mysite.com --check all

# 2. Content audit
/content-audit --url https://mysite.com --scope full

# 3. Identify content to refresh
/workflows:content-refresh --url https://mysite.com

# 4. Competitor analysis
/competitor-gap --yours mysite.com --theirs competitor.com
```

### Pattern 3: Monthly Reporting

```bash
# 1. Rank tracking
/serp-monitor --keywords monthly-keywords.csv --url mysite.com

# 2. Page speed check
/page-speed-seo --url https://mysite.com --device both

# 3. New link prospects
/link-prospecting --niche "your niche" --min-da 40
```

## Troubleshooting

### Command not recognized

**Problem:** `/keyword-research` returns "unknown command"

**Solution:**
```bash
# Ensure skill is loaded
/read ~/.claude/skills/seo-content-marketing/SKILL.md

# Or reload skills
/reload-skills
```

### API rate limits

**Problem:** "Rate limit exceeded" error

**Solution:**
- Wait for rate limit reset (usually 60 seconds)
- Use `--depth quick` for faster, lower-quota commands
- Set `SEO_RATE_LIMIT_DELAY=2000` (milliseconds) to slow requests

### Large site timeouts

**Problem:** Content audit timing out on large sites

**Solution:**
```bash
# Use sample mode
/content-audit --scope sample --url https://largesite.com

# Or limit crawl depth
/content-audit --scope full --depth 2 --url https://largesite.com
```

### Missing data in reports

**Problem:** Some metrics show "N/A" or missing

**Solution:**
- Verify API credentials are set: `echo $SEO_TOOL_API_KEY`
- Check that target URL is publicly accessible
- Use `--output json` to see raw data and error messages

### Output format issues

**Problem:** Tables not rendering properly

**Solution:**
```bash
# Try different output format
/keyword-research "topic" --output json

# Or export to file
/keyword-research "topic" --output csv > results.csv
```

## Advanced Usage

### Scripting Multiple Commands

```bash
#!/bin/bash
# monthly-seo-check.sh

SITE="https://mysite.com"
DATE=$(date +%Y-%m)

echo "Running monthly SEO audit for $SITE ($DATE)"

# Technical check
/technical-seo --url $SITE --output json > reports/$DATE-technical.json

# Content audit
/content-audit --url $SITE --scope full --output json > reports/$DATE-content.json

# Rank tracking
/serp-monitor --keywords keywords.csv --url $SITE > reports/$DATE-rankings.md

echo "Reports saved to reports/ directory"
```

### Custom Workflows

You can chain commands in your own workflows:

```bash
# Local business SEO package
/local-seo --business "My Business" --location "City, State" --audit all
/page-speed-seo --url https://mybusiness.com --device mobile
/content-audit --url https://mybusiness.com --focus quality
/link-prospecting --niche "local service" --type resource-page --min-da 30
```

## Integration Examples

### Export to Google Sheets

```bash
# Generate CSV output
/keyword-research "topic" --output csv > keywords.csv

# Import to Google Sheets using clasp or API
# (Requires separate Google Sheets integration)
```

### CI/CD Pipeline Integration

```yaml
# .github/workflows/seo-check.yml
name: Weekly SEO Audit
on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM
jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run technical SEO check
        run: |
          /technical-seo --url ${{ secrets.SITE_URL }} --output json > audit.json
      - name: Upload report
        uses: actions/upload-artifact@v2
        with:
          name: seo-audit
          path: audit.json
```

## Best Practices

1. **Start with quick analysis** — Use `--depth quick` for initial exploration, `--depth comprehensive` for final decisions
2. **Export results** — Always save reports with `--output json` or `csv` for historical tracking
3. **Batch operations** — Group similar commands to respect API rate limits
4. **Validate before acting** — Review all recommendations before implementing changes
5. **Track changes** — Use `/serp-monitor` to measure impact of optimizations

## Support & Community

- **Source repository:** https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo
- **Parent project:** https://github.com/alirezarezvani/claude-code-skill-factory
- **Issues:** Report bugs or request features via GitHub Issues
- **License:** MIT — free to use, modify, and distribute
