---
name: seo-content-marketing-skill-factory
description: SEO and content marketing automation suite with keyword research, technical audits, SERP analysis, and content workflows
triggers:
  - "analyze SEO for this site"
  - "run a technical SEO audit"
  - "generate a content brief for keyword"
  - "find keyword opportunities"
  - "check competitor backlink gaps"
  - "create an SEO content calendar"
  - "audit page speed for SEO"
  - "build a local SEO report"
---

# 📈 SEO & Content Marketing Skill Factory

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to perform comprehensive SEO and content marketing tasks using a command-based interface derived from the claude-code-skill-factory framework. It provides 10 specialized commands and 5 multi-step workflows for keyword research, technical audits, content strategy, and competitive analysis.

## What This Project Does

The SEO & Content Marketing Skill Factory provides:

- **Keyword Research & Clustering** — Deep semantic analysis with intent mapping
- **Technical SEO Audits** — Crawl budget, Core Web Vitals, schema validation
- **Content Audits** — Quality scoring, duplication detection, cannibalization reports
- **Competitive Analysis** — Gap analysis for backlinks, topics, and featured snippets
- **Content Strategy** — AI-generated briefs, calendars, and refresh workflows
- **SERP Monitoring** — Rank tracking with volatility alerts
- **Link Building** — Prospecting and outreach automation
- **Local SEO** — NAP consistency and citation audits

## Installation

### Manual Installation

```bash
# Clone or copy the skill into Claude's skills directory
mkdir -p ~/.claude/skills/seo-content-marketing-skill-factory
cp -r . ~/.claude/skills/seo-content-marketing-skill-factory/

# In a Claude Code session, register the skill:
/read ~/.claude/skills/seo-content-marketing-skill-factory/SKILL.md
```

### Verification

After installation, verify commands are available:

```bash
# List available SEO commands
/help | grep -E "(keyword|seo|content|serp)"
```

## Core Commands

### `/keyword-research`

Performs deep keyword clustering and opportunity scoring with SERP intent mapping.

**Usage:**

```bash
/keyword-research "coffee machines"
/keyword-research "saas analytics" --location US --depth deep
```

**Parameters:**
- `<seed_keyword>` — Primary keyword or topic (required)
- `--location` — Geographic target (default: US)
- `--depth` — Analysis depth: `quick`, `standard`, `deep` (default: standard)
- `--output` — Output format: `md`, `json`, `csv` (default: md)

**Output Structure:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "coffee machines"          ║
╠══════════════════════════════════════════════════╣
║  Collecting seed keywords …  [██████████] 100%   ║
║  Analyzing SERP intent …     [████████░░]  80%   ║
║  Clustering by topic …       [██████░░░░]  60%   ║
╚══════════════════════════════════════════════════╝

┌────────────────────────┬────────┬──────┬───────────┬──────────┐
│ Keyword                │ Volume │ KD   │ Intent    │ Priority │
├────────────────────────┼────────┼──────┼───────────┼──────────┤
│ best coffee machines   │ 22,000 │   42 │ Commercial│  🔴 High │
│ coffee machine reviews │ 18,000 │   38 │ Commercial│  🔴 High │
│ espresso machine       │ 90,000 │   67 │ Commercial│  🟠 Med  │
│ drip coffee maker      │ 14,000 │   29 │ Commercial│  🟢 Easy │
└────────────────────────┴────────┴──────┴───────────┴──────────┘
```

### `/content-audit`

Full-site content quality scoring with duplication and cannibalization detection.

**Usage:**

```bash
/content-audit https://example.com
/content-audit https://example.com --scope full --min-quality 60
```

**Parameters:**
- `<url>` — Target domain or URL (required)
- `--scope` — Audit scope: `page`, `section`, `full` (default: full)
- `--min-quality` — Minimum quality threshold 0-100 (default: 50)
- `--output` — Output format: `md`, `json`, `html` (default: md)

**Example Implementation:**

```python
# Example: Content quality scoring logic
def calculate_content_quality(page_data):
    """
    Calculate SEO content quality score (0-100)
    """
    score = 0
    
    # Word count (0-25 points)
    word_count = page_data.get('word_count', 0)
    if word_count >= 1500:
        score += 25
    elif word_count >= 800:
        score += 15
    elif word_count >= 300:
        score += 8
    
    # Heading structure (0-15 points)
    has_h1 = page_data.get('has_h1', False)
    h2_count = page_data.get('h2_count', 0)
    if has_h1 and h2_count >= 3:
        score += 15
    elif has_h1:
        score += 8
    
    # Meta data (0-20 points)
    title_length = len(page_data.get('title', ''))
    meta_desc = page_data.get('meta_description', '')
    if 50 <= title_length <= 60:
        score += 10
    elif 30 <= title_length <= 70:
        score += 5
    if 150 <= len(meta_desc) <= 160:
        score += 10
    elif 120 <= len(meta_desc) <= 170:
        score += 5
    
    # Readability (0-15 points)
    readability = page_data.get('flesch_reading_ease', 0)
    if 60 <= readability <= 70:
        score += 15
    elif 50 <= readability <= 80:
        score += 10
    
    # Media richness (0-15 points)
    image_count = page_data.get('images_with_alt', 0)
    has_video = page_data.get('has_video', False)
    if image_count >= 3 and has_video:
        score += 15
    elif image_count >= 2:
        score += 10
    
    # Internal linking (0-10 points)
    internal_links = page_data.get('internal_links', 0)
    if internal_links >= 5:
        score += 10
    elif internal_links >= 2:
        score += 5
    
    return min(score, 100)
```

### `/technical-seo`

Comprehensive technical SEO audit covering crawlability, performance, and indexability.

**Usage:**

```bash
/technical-seo https://example.com
/technical-seo https://example.com --check-all --fix-suggestions
```

**Parameters:**
- `<url>` — Target domain (required)
- `--check-all` — Run all checks including deep crawl (default: false)
- `--fix-suggestions` — Include code fix examples (default: true)

**Example Check Implementation:**

```javascript
// Example: Core Web Vitals checker
async function checkCoreWebVitals(url) {
  const config = {
    api_key: process.env.PAGESPEED_API_KEY,
    url: url,
    strategy: 'mobile'
  };
  
  const response = await fetch(
    `https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=${encodeURIComponent(config.url)}&strategy=${config.strategy}&key=${config.api_key}`
  );
  
  const data = await response.json();
  const metrics = data.lighthouseResult.audits;
  
  return {
    lcp: {
      value: metrics['largest-contentful-paint'].numericValue,
      score: metrics['largest-contentful-paint'].score,
      passing: metrics['largest-contentful-paint'].score >= 0.9
    },
    fid: {
      value: metrics['max-potential-fid'].numericValue,
      score: metrics['max-potential-fid'].score,
      passing: metrics['max-potential-fid'].score >= 0.9
    },
    cls: {
      value: metrics['cumulative-layout-shift'].numericValue,
      score: metrics['cumulative-layout-shift'].score,
      passing: metrics['cumulative-layout-shift'].score >= 0.9
    }
  };
}
```

### `/competitor-gap`

Identify backlink, topic, and featured snippet opportunities by comparing to competitors.

**Usage:**

```bash
/competitor-gap https://example.com --competitors competitor1.com,competitor2.com
/competitor-gap https://example.com --competitors competitor.com --gap-type all
```

**Parameters:**
- `<url>` — Your domain (required)
- `--competitors` — Comma-separated competitor domains (required)
- `--gap-type` — Analysis type: `backlink`, `topic`, `snippet`, `all` (default: all)

### `/content-brief`

Generate AI-powered SEO content briefs with outlines, NLP terms, and word count targets.

**Usage:**

```bash
/content-brief "how to start a podcast"
/content-brief "saas pricing strategies" --competitors 3 --serp-depth 20
```

**Parameters:**
- `<target_keyword>` — Primary keyword (required)
- `--competitors` — Number of top-ranking pages to analyze (default: 5)
- `--serp-depth` — SERP positions to analyze (default: 10)
- `--format` — Brief format: `standard`, `detailed`, `outline-only` (default: standard)

**Example Brief Output:**

```markdown
# Content Brief: "how to start a podcast"

## Target Metrics
- **Primary Keyword:** how to start a podcast
- **Search Volume:** 33,100/month
- **Keyword Difficulty:** 42
- **Search Intent:** Informational (How-to)
- **Target Word Count:** 2,200-2,800 words

## Top-Ranking Content Analysis
- Average word count: 2,487
- Average headings: 12
- Common media: Equipment photos, step diagrams, video tutorials

## Required H2 Sections (in order)
1. Choose Your Podcast Topic and Format
2. Select Podcast Recording Equipment
3. Choose Podcast Hosting Platform
4. Record Your First Episode
5. Edit Your Podcast Audio
6. Create Podcast Artwork and Description
7. Submit to Podcast Directories
8. Promote Your Podcast

## NLP Terms to Include
- podcast hosting
- RSS feed
- microphone
- audio editing software
- iTunes/Apple Podcasts
- episode artwork
- podcast description
- recording software

## Content Angle
Focus on beginner-friendly, step-by-step guidance with equipment recommendations at multiple price points.
```

### `/serp-monitor`

Track daily rankings with volatility alerts and CTR optimization recommendations.

**Usage:**

```bash
/serp-monitor --keywords keywords.csv
/serp-monitor --domain example.com --auto-detect-keywords
```

### `/link-prospecting`

Generate quality backlink prospect lists with domain authority filtering and outreach templates.

**Usage:**

```bash
/link-prospecting "sustainable fashion" --min-da 30 --max-results 50
/link-prospecting "tech startups" --min-da 40 --type guest-post,resource
```

**Parameters:**
- `<topic>` — Target topic or niche (required)
- `--min-da` — Minimum Domain Authority (default: 20)
- `--type` — Prospect types: `guest-post`, `resource`, `broken-link`, `unlinked-mention`
- `--max-results` — Maximum prospects to return (default: 100)

### `/page-speed-seo`

Diagnose render-blocking resources, LCP, CLS, FID issues mapped to ranking impact.

**Usage:**

```bash
/page-speed-seo https://example.com/page
/page-speed-seo https://example.com --device mobile --fix-code
```

### `/local-seo`

Audit NAP consistency, Google Business Profile optimization, and local citations.

**Usage:**

```bash
/local-seo "Acme Plumbing, Austin TX"
/local-seo --business-name "Acme Plumbing" --location "Austin, TX" --full-audit
```

### `/content-calendar`

Generate data-driven editorial calendars from search demand and seasonality.

**Usage:**

```bash
/content-calendar --niche "fitness" --months 3
/content-calendar --keywords keywords.csv --start-date 2024-06-01
```

## Multi-Step Workflows

### `full-seo-sprint`

12-step comprehensive SEO sprint from audit to implementation.

**Usage:**

```bash
/workflows:full-seo-sprint https://example.com --duration 4weeks
```

**Steps:**
1. Technical audit
2. Content audit
3. Competitor analysis
4. Keyword research & clustering
5. Site architecture review
6. Content gap analysis
7. Prioritized action plan
8. Quick wins implementation
9. Content optimization
10. Link building campaign
11. Performance monitoring setup
12. Progress report

### `launch-seo`

Pre-launch SEO checklist with canonical, hreflang, and sitemap validation.

**Usage:**

```bash
/workflows:launch-seo https://staging.example.com --launch-date 2024-06-15
```

### `content-refresh`

Identify and refresh underperforming pages to recover rankings.

**Usage:**

```bash
/workflows:content-refresh https://example.com --min-decline 5positions
```

### `authority-building`

End-to-end digital PR and link-building campaign.

**Usage:**

```bash
/workflows:authority-building --niche "sustainable tech" --target-links 50
```

### `ai-content-pipeline`

Automated keyword → brief → draft → optimize → publish pipeline.

**Usage:**

```bash
/workflows:ai-content-pipeline --keywords keywords.csv --auto-publish false
```

## Configuration

### Environment Variables

```bash
# API Keys
export SEMRUSH_API_KEY="your_semrush_key"
export AHREFS_API_KEY="your_ahrefs_key"
export PAGESPEED_API_KEY="your_pagespeed_key"
export OPENAI_API_KEY="your_openai_key"

# Default Settings
export SEO_DEFAULT_LOCATION="US"
export SEO_OUTPUT_FORMAT="md"
export SEO_CRAWL_DELAY="1000"  # milliseconds
export SEO_MAX_CONCURRENT_REQUESTS="5"
```

### Configuration File

Create `~/.seo-skill/config.yml`:

```yaml
defaults:
  location: US
  output_format: md
  crawl_delay_ms: 1000
  max_concurrent: 5

apis:
  semrush:
    enabled: true
    rate_limit: 100
  ahrefs:
    enabled: true
    rate_limit: 50
  pagespeed:
    enabled: true
    strategy: mobile

thresholds:
  quality_score_min: 60
  domain_authority_min: 20
  keyword_difficulty_max: 70
```

## Common Patterns

### Pattern 1: Full Site SEO Audit

```bash
# Step 1: Technical audit
/technical-seo https://example.com --check-all

# Step 2: Content audit
/content-audit https://example.com --scope full

# Step 3: Keyword opportunities
/keyword-research "main product" --depth deep

# Step 4: Competitive gaps
/competitor-gap https://example.com --competitors top3.com,leader.com
```

### Pattern 2: Content Strategy Development

```bash
# Research demand
/keyword-research "topic" --location US --output json > keywords.json

# Create briefs
/content-brief "primary keyword" --competitors 5

# Plan calendar
/content-calendar --keywords keywords.json --months 6

# Generate pipeline
/workflows:ai-content-pipeline --keywords keywords.json
```

### Pattern 3: Page Optimization

```bash
# Check current performance
/page-speed-seo https://example.com/page --device mobile

# Audit content quality
/content-audit https://example.com/page --scope page

# Get optimization brief
/content-brief "target keyword" --serp-depth 20
```

### Pattern 4: Link Building Campaign

```bash
# Find prospects
/link-prospecting "niche topic" --min-da 30 --type guest-post

# Analyze gaps
/competitor-gap https://example.com --gap-type backlink

# Run full campaign
/workflows:authority-building --niche "topic" --target-links 100
```

## Troubleshooting

### Command Not Found

If commands aren't recognized:

```bash
# Re-register the skill
/read ~/.claude/skills/seo-content-marketing-skill-factory/SKILL.md

# Verify installation
ls -la ~/.claude/skills/seo-content-marketing-skill-factory/
```

### API Rate Limiting

If you hit rate limits:

```yaml
# Adjust in config.yml
apis:
  semrush:
    rate_limit: 50  # Reduce from default
    
defaults:
  crawl_delay_ms: 2000  # Increase delay
  max_concurrent: 3  # Reduce concurrency
```

### Slow Performance

For large sites:

```bash
# Use targeted scopes
/content-audit https://example.com --scope section

# Limit depth
/keyword-research "topic" --depth quick

# Use sampling
/technical-seo https://example.com --sample-size 100
```

### Missing Dependencies

If analysis fails:

```bash
# Check required APIs are configured
env | grep -E "(SEMRUSH|AHREFS|PAGESPEED)_API_KEY"

# Verify network access
curl -I https://api.semrush.com/
```

### Output Format Issues

```bash
# Specify format explicitly
/keyword-research "topic" --output md

# For programmatic use
/keyword-research "topic" --output json | jq '.opportunities[] | select(.priority == "high")'
```

## Advanced Usage

### Scripting Multiple Audits

```bash
#!/bin/bash
# audit-sites.sh

SITES=(
  "https://site1.com"
  "https://site2.com"
  "https://site3.com"
)

for site in "${SITES[@]}"; do
  echo "Auditing $site..."
  /technical-seo "$site" --output json > "audit-$(basename $site).json"
  /content-audit "$site" --scope full --output json > "content-$(basename $site).json"
done
```

### Custom Quality Thresholds

```bash
# High-quality content only
/content-audit https://example.com --min-quality 80

# Aggressive keyword targeting
/keyword-research "topic" --max-difficulty 30
```

### Integration with CI/CD

```yaml
# .github/workflows/seo-check.yml
name: SEO Audit
on:
  pull_request:
    paths:
      - 'content/**'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run SEO checks
        run: |
          /technical-seo https://staging.example.com --output json > audit.json
          /content-audit https://staging.example.com --scope full --min-quality 70
```

## Best Practices

1. **Always specify output format** for programmatic workflows
2. **Use environment variables** for API keys — never hardcode
3. **Start with quick depth** for initial research, then go deep
4. **Sample large sites** to avoid timeout issues
5. **Review action plans** before bulk implementation
6. **Monitor rate limits** when running multiple commands
7. **Version control audits** to track improvements over time
8. **Combine commands** in workflows for comprehensive analysis

---

**Source:** [alirezarezvani/claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory)
