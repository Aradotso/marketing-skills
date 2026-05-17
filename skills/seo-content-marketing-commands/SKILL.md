---
name: seo-content-marketing-commands
description: Structured SEO and content marketing command suite with keyword research, content audits, SERP analysis, and automated workflows
triggers:
  - run an SEO audit on this website
  - perform keyword research for this topic
  - analyze content gaps against competitors
  - generate an SEO content brief
  - check technical SEO issues
  - create a content marketing calendar
  - find backlink opportunities
  - audit page speed for SEO impact
---

# SEO & Content Marketing Commands

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive command suite for SEO and content marketing workflows, derived from wshobson/commands. Provides 10 specialized commands and 5 multi-step workflows with structured output for keyword research, content audits, SERP analysis, technical SEO, and content strategy.

## What This Project Does

This skill suite transforms SEO and content marketing tasks into repeatable, structured commands that deliver:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema markup validation
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — AI-generated briefs, editorial calendars, refresh workflows
- **Link Building** — prospect lists, outreach templates, authority campaigns

All commands follow a consistent 5-step pattern: scope confirmation → live analysis → findings → action plan → next steps.

## Installation

```bash
# Clone into Claude skills directory
mkdir -p ~/.claude/skills
cp -r /path/to/r10-wshobson-commands-seo ~/.claude/skills/seo-content-marketing/

# Load in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

For other AI agents (Cursor, Codex):

```bash
# Add to agent config
export AGENT_SKILLS_PATH="$HOME/.claude/skills/seo-content-marketing"
```

## Core Commands

### Keyword Research

```bash
/keyword-research <target> [options]
```

**Options:**
- `--depth <shallow|medium|deep>` — analysis depth (default: medium)
- `--country <code>` — market locale (default: US)
- `--output <json|md|csv>` — output format

**Example:**
```bash
/keyword-research "project management software" --depth deep --country UK
```

**Output Structure:**
```
┌─────────────────────────┬────────┬──────┬─────────┬──────────┐
│ Keyword                 │ Volume │ Diff │ Intent  │ Priority │
├─────────────────────────┼────────┼──────┼─────────┼──────────┤
│ project management tool │ 18 100 │   32 │ Commercial │  ⭐⭐⭐  │
│ best project software   │  9 900 │   45 │ Informational │  ⭐⭐   │
│ pm tools for teams      │  2 400 │   28 │ Commercial │  ⭐⭐⭐  │
└─────────────────────────┴────────┴──────┴─────────┴──────────┘

📊 Clusters Found: 12 | Total Volume: 147,200 | Avg Difficulty: 34
```

### Content Audit

```bash
/content-audit [options]
```

**Options:**
- `--scope <full|sample|urls>` — audit scope
- `--urls <file>` — path to URL list
- `--min-quality <0-100>` — quality threshold

**Example:**
```bash
/content-audit --scope full --min-quality 60 --output md
```

**Detection Logic:**
```python
# Quality scoring algorithm
def calculate_content_quality(page):
    score = 0
    
    # Title optimization (20 pts)
    if page.title and 30 <= len(page.title) <= 60:
        score += 20
    elif page.title:
        score += 10
    
    # Meta description (15 pts)
    if page.meta_desc and 120 <= len(page.meta_desc) <= 160:
        score += 15
    elif page.meta_desc:
        score += 8
    
    # Content depth (25 pts)
    word_count = len(page.body.split())
    if word_count >= 1500:
        score += 25
    elif word_count >= 800:
        score += 15
    elif word_count >= 300:
        score += 8
    
    # Heading structure (15 pts)
    if page.has_h1 and page.h2_count >= 3:
        score += 15
    elif page.has_h1:
        score += 8
    
    # Internal links (10 pts)
    if page.internal_links >= 5:
        score += 10
    elif page.internal_links >= 2:
        score += 5
    
    # Images with alt text (10 pts)
    if page.images_with_alt / max(page.total_images, 1) >= 0.9:
        score += 10
    elif page.images_with_alt / max(page.total_images, 1) >= 0.5:
        score += 5
    
    # Readability (5 pts)
    if 50 <= page.flesch_score <= 70:
        score += 5
    
    return score
```

### Technical SEO Audit

```bash
/technical-seo [domain] [options]
```

**Options:**
- `--checks <all|critical|performance>` — check types
- `--crawl-depth <number>` — max crawl depth
- `--user-agent <string>` — crawler user agent

**Example:**
```bash
/technical-seo example.com --checks all --crawl-depth 3
```

**Checks Performed:**
```javascript
const technicalChecks = {
  indexability: [
    'robots.txt syntax',
    'meta robots tags',
    'canonical tags',
    'XML sitemap presence',
    'noindex pages in sitemap'
  ],
  
  performance: [
    'Core Web Vitals (LCP, FID, CLS)',
    'render-blocking resources',
    'image optimization',
    'browser caching',
    'compression (gzip/brotli)'
  ],
  
  structure: [
    'schema markup (JSON-LD)',
    'breadcrumb navigation',
    'pagination handling',
    'hreflang implementation',
    'internal linking structure'
  ],
  
  security: [
    'HTTPS implementation',
    'mixed content warnings',
    'security headers',
    'certificate validity'
  ]
};
```

### Competitor Gap Analysis

```bash
/competitor-gap <domain> --competitors <domains> [options]
```

**Options:**
- `--competitors <comma-separated>` — competitor domains
- `--gap-type <backlink|topic|snippet>` — analysis type
- `--min-dr <number>` — minimum domain rating

**Example:**
```bash
/competitor-gap mysite.com \
  --competitors competitor1.com,competitor2.com \
  --gap-type topic \
  --min-dr 30
```

**Topic Gap Algorithm:**
```python
def find_topic_gaps(target_site, competitor_sites):
    """Identify topics competitors rank for that target doesn't"""
    
    gaps = {
        'quick_wins': [],      # Low difficulty, high volume
        'content_ideas': [],   # Ranked by multiple competitors
        'defensive': []        # Target ranks poorly, competitors rank well
    }
    
    for competitor in competitor_sites:
        # Get ranking keywords
        comp_keywords = get_organic_keywords(competitor)
        target_keywords = get_organic_keywords(target_site)
        
        # Find gaps
        missing = comp_keywords - target_keywords
        
        for kw in missing:
            metrics = get_keyword_metrics(kw)
            
            # Quick wins: volume > 500, difficulty < 30
            if metrics['volume'] > 500 and metrics['difficulty'] < 30:
                gaps['quick_wins'].append({
                    'keyword': kw,
                    'volume': metrics['volume'],
                    'difficulty': metrics['difficulty'],
                    'competitor_position': metrics['comp_rank']
                })
            
            # Multiple competitors ranking
            ranking_comps = count_ranking_competitors(kw, competitor_sites)
            if ranking_comps >= 2:
                gaps['content_ideas'].append({
                    'keyword': kw,
                    'volume': metrics['volume'],
                    'competitors_ranking': ranking_comps
                })
    
    return gaps
```

### Content Brief Generation

```bash
/content-brief <topic> [options]
```

**Options:**
- `--target-words <number>` — target word count
- `--intent <informational|commercial|transactional>` — search intent
- `--competitors <number>` — top competitors to analyze

**Example:**
```bash
/content-brief "best email marketing tools 2024" \
  --target-words 2500 \
  --intent commercial \
  --competitors 10
```

**Brief Template:**
```markdown
# Content Brief: {topic}

## Target Metrics
- **Primary Keyword:** {main_keyword}
- **Search Volume:** {monthly_volume}
- **Keyword Difficulty:** {difficulty}/100
- **Target Word Count:** {target_words}
- **Content Type:** {content_type}

## SERP Analysis
Top 10 competitors analyzed:
- Average word count: {avg_words}
- Common content angles: {angles}
- Media elements: {images}, {videos}, {infographics}

## Outline
{h1}
  {h2}
    - {h3}
    - {h3}
  {h2}
    - {h3}

## NLP Terms (must include)
{nlp_terms_list}

## Questions to Answer
{people_also_ask}

## Internal Linking Opportunities
{suggested_internal_links}

## CTA Recommendations
{cta_suggestions}
```

### SERP Monitoring

```bash
/serp-monitor [keywords-file] [options]
```

**Options:**
- `--frequency <daily|weekly|monthly>` — check frequency
- `--alert-threshold <number>` — position change alert trigger
- `--device <desktop|mobile>` — device type

**Example:**
```bash
/serp-monitor keywords.txt --frequency daily --alert-threshold 3
```

**Monitoring Output:**
```
┌──────────────────────────┬──────────┬─────────┬────────┬───────────┐
│ Keyword                  │ Position │ Change  │ Volume │ Alert     │
├──────────────────────────┼──────────┼─────────┼────────┼───────────┤
│ project management       │     4    │  ▲ +2   │ 22 200 │ 🟢 Gained │
│ task management software │     8    │  ▼ -5   │ 12 100 │ 🔴 Lost   │
│ collaboration tools      │    12    │  ─ 0    │  8 100 │ 🟡 Stable │
└──────────────────────────┴──────────┴─────────┴────────┴───────────┘

🔔 Alerts: 2 positions gained ▲ | 1 position lost ▼ | CTR opportunity on rank 4
```

## Multi-Step Workflows

### Full SEO Sprint

```bash
/workflows:full-seo-sprint <domain> [options]
```

**12-Step Process:**
```yaml
steps:
  1: Technical audit (indexability, performance)
  2: Keyword research (opportunity mapping)
  3: Content audit (quality scoring)
  4: Competitor gap analysis
  5: Priority matrix generation
  6: Quick wins identification
  7: Content plan creation
  8: Technical fixes roadmap
  9: Link building strategy
  10: Implementation timeline
  11: KPI dashboard setup
  12: 30/60/90 day milestones
```

### Launch SEO Checklist

```bash
/workflows:launch-seo <domain> --pre-launch
```

**Pre-Launch Checks:**
```python
launch_checklist = {
    'critical': [
        'Remove noindex from production pages',
        'Submit XML sitemap to GSC',
        'Verify robots.txt allows crawling',
        'Check canonical tags point to production',
        'Validate hreflang implementation',
        'Test 301 redirects from old URLs',
        'Confirm HTTPS certificate valid'
    ],
    
    'important': [
        'Add Google Analytics tracking',
        'Set up GSC property',
        'Implement structured data',
        'Optimize Core Web Vitals',
        'Create 404 error page',
        'Set up redirect monitoring'
    ],
    
    'recommended': [
        'Add social meta tags (OG, Twitter)',
        'Create favicon and app icons',
        'Set up internal linking structure',
        'Optimize images (alt text, compression)',
        'Add breadcrumb navigation',
        'Test mobile responsiveness'
    ]
}
```

### Content Refresh Workflow

```bash
/workflows:content-refresh --min-age 180 --max-quality 60
```

**Refresh Criteria:**
```javascript
function identifyRefreshCandidates(pages) {
  return pages.filter(page => {
    const age = daysSince(page.published_date);
    const qualityScore = calculateQualityScore(page);
    const trafficTrend = getTrafficTrend(page.url, 90);
    
    // Refresh if:
    // - Content older than 6 months
    // - Quality score below 60
    // - Traffic declined >20% in 90 days
    // - Keyword positions dropped >5 spots
    
    return (
      age > 180 &&
      (qualityScore < 60 || 
       trafficTrend.decline > 0.20 ||
       trafficTrend.position_drop > 5)
    );
  }).sort((a, b) => {
    // Prioritize by opportunity (traffic potential × ease)
    const oppA = a.historical_traffic * (100 - a.quality_score);
    const oppB = b.historical_traffic * (100 - b.quality_score);
    return oppB - oppA;
  });
}
```

## Configuration

Commands can be configured via environment variables or config file:

```bash
# ~/.seo-commands/config.yaml
default_options:
  country: US
  language: en
  output_format: md
  
api_keys:
  google_search_console: ${GSC_API_KEY}
  semrush_api: ${SEMRUSH_API_KEY}
  ahrefs_api: ${AHREFS_API_KEY}
  
crawl_settings:
  max_depth: 3
  max_pages: 10000
  rate_limit: 2  # requests per second
  user_agent: "SEO-Commands-Bot/1.0"
  
quality_thresholds:
  min_content_score: 60
  min_word_count: 300
  max_duplicate_threshold: 0.15
```

## Environment Variables

```bash
# Required for full functionality
export GSC_API_KEY="your_key_here"
export SEMRUSH_API_KEY="your_key_here"

# Optional integrations
export AHREFS_API_KEY="your_key_here"
export SCREAMING_FROG_PATH="/path/to/screamingfrog"

# Configuration overrides
export SEO_COMMANDS_CONFIG="$HOME/.seo-commands/config.yaml"
export SEO_COMMANDS_OUTPUT_DIR="$HOME/seo-reports"
```

## Common Patterns

### Daily Monitoring Routine

```bash
#!/bin/bash
# daily-seo-check.sh

# Monitor key rankings
/serp-monitor priority-keywords.txt --frequency daily --alert-threshold 3

# Check technical health
/technical-seo mysite.com --checks critical --output json > daily-health.json

# Alert on critical issues
if grep -q '"severity":"critical"' daily-health.json; then
  echo "🔴 Critical SEO issues detected" | notify
fi
```

### Monthly Content Audit

```bash
# Full content audit with refresh recommendations
/content-audit --scope full --min-quality 60 --output md > audit-$(date +%Y%m).md

# Identify refresh candidates
/workflows:content-refresh --min-age 180 --max-quality 60 > refresh-plan.md
```

### Competitor Monitoring

```bash
# Weekly competitor gap check
/competitor-gap mysite.com \
  --competitors comp1.com,comp2.com,comp3.com \
  --gap-type topic \
  --output json > gaps-$(date +%Y%m%d).json
```

## Troubleshooting

### Command Not Found

```bash
# Verify skill is loaded
ls -la ~/.claude/skills/seo-content-marketing/

# Reload skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### API Rate Limits

```yaml
# Adjust rate limiting in config
crawl_settings:
  rate_limit: 1  # Reduce to 1 req/sec
  batch_size: 50  # Process in smaller batches
```

### Incomplete Crawl Data

```bash
# Increase crawl depth and timeout
/technical-seo mysite.com \
  --crawl-depth 5 \
  --timeout 120 \
  --max-pages 50000
```

### Low Quality Scores

Check scoring criteria alignment:

```python
# Review quality algorithm weights
QUALITY_WEIGHTS = {
    'title': 20,
    'meta_desc': 15,
    'content_depth': 25,
    'heading_structure': 15,
    'internal_links': 10,
    'image_optimization': 10,
    'readability': 5
}

# Adjust minimum thresholds if needed
MIN_WORD_COUNT = 300  # Lower for product pages
MIN_INTERNAL_LINKS = 2  # Lower for landing pages
```

## Integration Examples

### Google Search Console

```python
from google.oauth2 import service_account
from googleapiclient.discovery import build

# Authenticate
credentials = service_account.Credentials.from_service_account_file(
    os.getenv('GSC_CREDENTIALS_PATH'),
    scopes=['https://www.googleapis.com/auth/webmasters.readonly']
)

service = build('searchconsole', 'v1', credentials=credentials)

# Fetch performance data
response = service.searchanalytics().query(
    siteUrl='https://example.com',
    body={
        'startDate': '2024-01-01',
        'endDate': '2024-01-31',
        'dimensions': ['query', 'page'],
        'rowLimit': 25000
    }
).execute()
```

### Screaming Frog Integration

```bash
# Export crawl data for analysis
/path/to/screamingfrog --crawl https://example.com \
  --export-tabs "Internal:All" \
  --output-folder ./crawl-data/ \
  --headless

# Import into content audit
/content-audit --urls ./crawl-data/internal_all.csv
```

## Advanced Usage

### Custom Scoring Functions

```python
# Override default quality scorer
def custom_ecommerce_quality(page):
    score = 0
    
    # Product-specific criteria
    if page.has_price: score += 15
    if page.has_availability: score += 10
    if page.has_reviews and page.review_count >= 5: score += 20
    if page.has_schema_product: score += 15
    if page.image_count >= 3: score += 10
    if page.has_size_chart: score += 10
    if page.has_shipping_info: score += 10
    if page.has_return_policy: score += 10
    
    return score

# Use in audit
/content-audit --scope full --scorer custom_ecommerce_quality
```

This skill provides production-ready SEO and content marketing automation with structured, actionable output for common marketing workflows.
