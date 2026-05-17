---
name: seo-content-marketing-skill-suite
description: SEO & Content Marketing command suite for keyword research, content audits, technical SEO, competitor analysis, and content strategy automation
triggers:
  - "run an SEO audit on this site"
  - "help me with keyword research"
  - "analyze competitor SEO gaps"
  - "create a content brief for this topic"
  - "check technical SEO issues"
  - "build a content calendar"
  - "find backlink opportunities"
  - "audit page speed for SEO"
---

# SEO & Content Marketing Skill Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to perform comprehensive SEO and content marketing tasks through a structured command-line interface. Derived from `alirezarezvani/claude-code-skill-factory`, it provides 10 specialized commands and 5 multi-step workflows for keyword research, content audits, SERP analysis, technical SEO, and content strategy.

## What This Project Does

The SEO & Content Marketing Skills Suite provides:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget analysis, Core Web Vitals, schema validation
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — AI-generated briefs, editorial calendars, refresh workflows
- **Local SEO** — NAP consistency, GMB optimization, citation audits
- **Link Building** — prospect discovery, outreach template generation
- **Performance Monitoring** — rank tracking, SERP volatility, CTR optimization

All commands use structured output with progress tracking, findings tables, and prioritized action plans.

## Installation

### Clone the Skill

```bash
# Create Claude Code skills directory if it doesn't exist
mkdir -p ~/.claude/skills/

# Clone the skill suite
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git \
  ~/.claude/skills/seo-content-marketing-suite/

cd ~/.claude/skills/seo-content-marketing-suite/
```

### Register in Claude Code

In a Claude Code session:

```bash
/read ~/.claude/skills/seo-content-marketing-suite/SKILL.md
```

Or manually load:

```bash
# Add to your Claude Code configuration
echo "seo-content-marketing-suite" >> ~/.claude/skills/active_skills.txt
```

## Core Commands

### `/keyword-research`

Deep keyword clustering and opportunity scoring with SERP intent mapping.

**Usage:**

```bash
# Basic keyword research
/keyword-research "marketing automation"

# With advanced options
/keyword-research "saas analytics" --depth comprehensive --export csv

# Multiple seed keywords
/keyword-research "email marketing, marketing automation, lead nurturing"
```

**Options:**
- `--depth` — `quick | standard | comprehensive` (default: standard)
- `--min-volume` — minimum monthly search volume (default: 50)
- `--export` — output format: `md | csv | json` (default: md)
- `--intent` — filter by intent: `informational | commercial | transactional | navigational`

**Output Structure:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "marketing automation"     ║
╠══════════════════════════════════════════════════╣
║  Gathering keywords …     [██████████] 100%  ✓   ║
║  Analyzing SERP intent …  [██████████] 100%  ✓   ║
║  Clustering topics …      [██████████] 100%  ✓   ║
╚══════════════════════════════════════════════════╝

┌──────────────────────────────┬────────┬────────┬──────────────┬──────────┐
│ Keyword                      │ Volume │ KD     │ Intent       │ Priority │
├──────────────────────────────┼────────┼────────┼──────────────┼──────────┤
│ marketing automation software│ 12,100 │ 67     │ Commercial   │ 🔴 High  │
│ best marketing automation    │  8,900 │ 58     │ Commercial   │ 🔴 High  │
│ marketing automation tools   │  6,700 │ 62     │ Commercial   │ 🟠 Med   │
│ what is marketing automation │  4,400 │ 34     │ Informational│ 🟡 Low   │
└──────────────────────────────┴────────┴────────┴──────────────┴──────────┘

📊 Clusters Found: 4
   - Automation Software (23 keywords)
   - Email Automation (18 keywords)
   - CRM Integration (12 keywords)
   - Best Practices (15 keywords)
```

### `/content-audit`

Full-site content quality score, duplication check, and cannibalization report.

**Usage:**

```bash
# Basic content audit
/content-audit https://example.com

# Comprehensive audit with export
/content-audit https://example.com --scope full --output json

# Specific section audit
/content-audit https://example.com/blog --scope section
```

**Options:**
- `--scope` — `quick | section | full` (default: section)
- `--output` — `md | json | csv` (default: md)
- `--min-words` — minimum word count for analysis (default: 100)
- `--check-duplicates` — enable duplicate content detection (default: true)

**Output Example:**

```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  example.com                   ║
╠══════════════════════════════════════════════════╣
║  Crawling pages …         [████████░░]  82%  247 ║
║  Analyzing content …      [██████░░░░]  60%  181 ║
║  Checking duplicates …    [███░░░░░░░]  30%   90 ║
╚══════════════════════════════════════════════════╝

┌─────────────────────────┬─────────┬────────┬──────────┬─────────┐
│ Page                    │ Words   │ Quality│ Issues   │ Status  │
├─────────────────────────┼─────────┼────────┼──────────┼─────────┤
│ /blog/seo-guide         │  2,847  │  92/100│ None     │ ✓ Good  │
│ /products/analytics     │  1,203  │  78/100│ Thin     │ ⚠ Fair  │
│ /about-us               │    342  │  45/100│ Thin, Dup│ ✗ Poor  │
└─────────────────────────┴─────────┴────────┴──────────┴─────────┘

🔴 Critical Issues (12 pages)
   - Duplicate content detected
   - Keyword cannibalization on "marketing automation"

🟠 Medium Issues (34 pages)
   - Thin content (<300 words)
   - Missing or weak meta descriptions
```

### `/technical-seo`

Crawl budget analysis, Core Web Vitals, schema markup, and indexability audit.

**Usage:**

```bash
# Basic technical audit
/technical-seo https://example.com

# Comprehensive with specific checks
/technical-seo https://example.com --checks vitals,schema,crawl --export json
```

**Options:**
- `--checks` — comma-separated: `vitals,schema,crawl,mobile,security` (default: all)
- `--export` — `md | json | csv` (default: md)
- `--depth` — max crawl depth (default: 5)

**Output Structure:**

```
┌──────────────────────┬──────────┬──────────┬──────────┐
│ Check                │ Status   │ Score    │ Issues   │
├──────────────────────┼──────────┼──────────┼──────────┤
│ Core Web Vitals      │ Pass     │  95/100  │ 0        │
│ Mobile-Friendly      │ Pass     │  98/100  │ 0        │
│ HTTPS/Security       │ Pass     │ 100/100  │ 0        │
│ Schema Markup        │ Warning  │  67/100  │ 12       │
│ Crawl Budget         │ Fail     │  42/100  │ 45       │
│ Indexability         │ Warning  │  78/100  │ 8        │
└──────────────────────┴──────────┴──────────┴──────────┘

🔴 Critical (3)
   - 45 pages blocked by robots.txt but in sitemap
   - Redirect chain depth >3 on 12 pages
   - Orphaned pages: 23

🟠 Warnings (15)
   - Missing schema on product pages
   - Slow LCP on 8 mobile pages (>2.5s)
```

### `/competitor-gap`

Backlink gap, topic gap, and featured-snippet opportunity analysis.

**Usage:**

```bash
# Basic competitor gap analysis
/competitor-gap https://example.com --competitors competitor1.com,competitor2.com

# Focused analysis
/competitor-gap https://example.com \
  --competitors competitor1.com,competitor2.com \
  --analysis backlinks,keywords,featured-snippets
```

**Options:**
- `--competitors` — comma-separated competitor domains (required)
- `--analysis` — `backlinks | keywords | featured-snippets | all` (default: all)
- `--export` — `md | csv | json` (default: md)

### `/content-brief`

AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Usage:**

```bash
# Generate content brief
/content-brief "best marketing automation software"

# With target competitors
/content-brief "best marketing automation software" \
  --analyze hubspot.com,marketo.com,activecampaign.com \
  --format detailed
```

**Options:**
- `--analyze` — competitor URLs to analyze (comma-separated)
- `--format` — `quick | standard | detailed` (default: standard)
- `--export` — `md | json` (default: md)

**Output Example:**

```markdown
# Content Brief: "best marketing automation software"

## Target Metrics
- Primary Keyword: best marketing automation software
- Search Volume: 12,100/mo
- Keyword Difficulty: 67
- Recommended Word Count: 2,800-3,200 words
- Content Type: Comparison / Listicle

## Search Intent
Commercial investigation — users comparing solutions before purchase

## Recommended Outline
1. Introduction (150-200 words)
2. What is Marketing Automation Software? (200-250 words)
3. Top 10 Marketing Automation Software [Main Section] (1,800 words)
   - For each tool: Overview, Key Features, Pricing, Pros/Cons
4. How to Choose the Right Platform (300 words)
5. Comparison Table (visual)
6. FAQ Section (200 words)
7. Conclusion & CTA (150 words)

## NLP/LSI Terms (Top 20)
- email marketing
- lead nurturing
- CRM integration
- workflow automation
- drip campaigns
[...]

## Competitor Analysis
Top 3 ranking pages average:
- Word count: 3,124
- Images: 12
- Internal links: 18
- External links: 8
```

### `/serp-monitor`

Daily rank tracking report with volatility alerts and CTR optimization tips.

**Usage:**

```bash
# Monitor rankings
/serp-monitor --keywords keywords.csv --domain example.com

# Single keyword check
/serp-monitor --keyword "marketing automation" --domain example.com
```

### `/link-prospecting`

Quality backlink prospect list with DA/DR filters and outreach templates.

**Usage:**

```bash
# Find link prospects
/link-prospecting "marketing automation" \
  --min-dr 40 \
  --types guest-post,resource-page,broken-link

# Export with outreach templates
/link-prospecting "saas analytics" \
  --min-dr 50 \
  --export csv \
  --include-templates
```

**Options:**
- `--min-dr` — minimum Domain Rating (default: 30)
- `--types` — prospect types: `guest-post,resource-page,broken-link,unlinked-mention`
- `--export` — `md | csv` (default: md)
- `--include-templates` — add outreach email templates (default: false)

### `/page-speed-seo`

Render-blocking, LCP, CLS, FID diagnosis mapped to ranking impact.

**Usage:**

```bash
# Analyze page speed
/page-speed-seo https://example.com/page

# With mobile focus
/page-speed-seo https://example.com/page --device mobile --export json
```

### `/local-seo`

NAP consistency, Google Business Profile optimization, and local citation audit.

**Usage:**

```bash
# Local SEO audit
/local-seo "Business Name" --location "City, State"

# With citation check
/local-seo "Acme Plumbing" \
  --location "Austin, TX" \
  --check-citations \
  --export csv
```

### `/content-calendar`

Data-driven editorial calendar built from search demand and seasonality.

**Usage:**

```bash
# Generate 3-month calendar
/content-calendar --topics "marketing automation, email marketing" \
  --months 3 \
  --export csv

# With keyword research integration
/content-calendar --seed-keywords keywords.csv \
  --months 6 \
  --frequency weekly
```

## Workflows (Multi-Step)

### `full-seo-sprint`

12-step comprehensive SEO sprint: audit → keyword map → content plan → technical fixes.

**Usage:**

```bash
/workflows:full-seo-sprint https://example.com --duration 4-weeks
```

**Steps:**
1. Technical audit
2. Content audit
3. Keyword research
4. Competitor gap analysis
5. On-page optimization plan
6. Content refresh priorities
7. New content strategy
8. Link building plan
9. Local SEO (if applicable)
10. Implementation timeline
11. Measurement framework
12. Reporting dashboard setup

### `launch-seo`

Pre-launch SEO checklist with canonical, hreflang, and sitemap validation.

**Usage:**

```bash
/workflows:launch-seo https://staging.example.com --launch-date 2026-06-01
```

### `content-refresh`

Identify and refresh underperforming pages to recover lost rankings.

**Usage:**

```bash
/workflows:content-refresh https://example.com --min-drop 5-positions
```

### `authority-building`

End-to-end digital PR and link-building campaign workflow.

**Usage:**

```bash
/workflows:authority-building --topic "marketing automation" --duration 3-months
```

### `ai-content-pipeline`

Keyword → brief → draft → optimize → publish automation pipeline.

**Usage:**

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --output-dir ./content
```

## Configuration

### Environment Variables

```bash
# API Keys (if using external SEO tools)
export AHREFS_API_KEY=your_ahrefs_key
export SEMRUSH_API_KEY=your_semrush_key
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=path/to/credentials.json

# Default Settings
export SEO_SUITE_DEFAULT_EXPORT=json
export SEO_SUITE_MAX_CRAWL_DEPTH=5
export SEO_SUITE_MIN_KEYWORD_VOLUME=50
```

### Config File

Create `~/.seo-suite/config.yaml`:

```yaml
defaults:
  export_format: md
  min_keyword_volume: 50
  max_crawl_depth: 5
  
api_integrations:
  ahrefs:
    enabled: true
    rate_limit: 100
  semrush:
    enabled: true
    rate_limit: 50
  google_search_console:
    enabled: true
    
output:
  progress_bars: true
  color: true
  save_logs: true
  log_dir: ~/.seo-suite/logs
```

## Common Patterns

### Complete Site Audit Workflow

```bash
# Step 1: Technical foundation
/technical-seo https://example.com --export json > tech-audit.json

# Step 2: Content analysis
/content-audit https://example.com --scope full --output json > content-audit.json

# Step 3: Keyword opportunities
/keyword-research "primary topic" --depth comprehensive --export csv > keywords.csv

# Step 4: Competitor intelligence
/competitor-gap https://example.com \
  --competitors competitor1.com,competitor2.com \
  --export json > gap-analysis.json

# Step 5: Generate action plan
/workflows:full-seo-sprint https://example.com --use-existing-audits
```

### Content Production Pipeline

```bash
# 1. Build content calendar
/content-calendar --seed-keywords keywords.csv --months 3 --export csv > calendar.csv

# 2. Generate briefs for each topic
while IFS=, read -r keyword; do
  /content-brief "$keyword" --format detailed --export md > "briefs/${keyword}.md"
done < keywords.csv

# 3. Create drafts (with AI assistance)
# ... content creation process ...

# 4. Optimize before publish
/page-speed-seo https://staging.example.com/new-article
```

### Link Building Campaign

```bash
# 1. Find prospects
/link-prospecting "your topic" \
  --min-dr 40 \
  --types guest-post,resource-page \
  --include-templates \
  --export csv > prospects.csv

# 2. Launch authority building workflow
/workflows:authority-building \
  --topic "your topic" \
  --duration 3-months \
  --prospects prospects.csv
```

## Troubleshooting

### Command Not Found

```bash
# Verify skill is registered
cat ~/.claude/skills/active_skills.txt | grep seo-content-marketing-suite

# Re-register if needed
/read ~/.claude/skills/seo-content-marketing-suite/SKILL.md
```

### Slow Crawl Performance

```bash
# Reduce crawl depth
/technical-seo https://example.com --depth 3

# Or limit scope
/content-audit https://example.com --scope section
```

### API Rate Limits

```bash
# Check rate limit configuration
cat ~/.seo-suite/config.yaml | grep rate_limit

# Adjust in config or use delays
export SEO_SUITE_API_DELAY=1000  # milliseconds between requests
```

### Missing Dependencies

```bash
# Ensure required tools are installed
pip install -r requirements.txt  # if project uses Python
npm install                       # if project uses Node.js
```

### Export Format Issues

```bash
# Force specific format
/keyword-research "topic" --export json | jq '.'  # validate JSON

# Or use markdown for human-readable output
/keyword-research "topic" --export md > report.md
```

## Advanced Usage

### Integrating with CI/CD

```yaml
# .github/workflows/seo-monitoring.yml
name: SEO Monitoring
on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run SEO Audit
        run: |
          /technical-seo https://example.com --export json > seo-report.json
          /serp-monitor --keywords keywords.csv --domain example.com
```

### Custom Scripts

```bash
#!/bin/bash
# weekly-seo-report.sh

DOMAIN="example.com"
OUTPUT_DIR="./reports/$(date +%Y-%m-%d)"

mkdir -p "$OUTPUT_DIR"

# Run audits
/technical-seo "$DOMAIN" --export json > "$OUTPUT_DIR/technical.json"
/content-audit "$DOMAIN" --scope full --output json > "$OUTPUT_DIR/content.json"
/serp-monitor --keywords keywords.csv --domain "$DOMAIN" > "$OUTPUT_DIR/rankings.md"

# Generate summary
echo "SEO Report for $DOMAIN - $(date)" > "$OUTPUT_DIR/summary.md"
echo "Reports generated in $OUTPUT_DIR"
```

## Related Skills

- **Content Writing** — for implementing content briefs
- **Web Development** — for implementing technical SEO fixes
- **Analytics & Reporting** — for measuring SEO performance
- **Social Media Marketing** — for content distribution

## Support

For issues specific to this skill suite:
- GitHub: https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo
- Original framework: https://github.com/alirezarezvani/claude-code-skill-factory
