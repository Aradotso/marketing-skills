---
name: seo-content-marketing-skill-factory
description: SEO & content marketing skill suite with keyword research, content audits, technical SEO analysis, and AI-driven content workflows
triggers:
  - "help me with SEO analysis"
  - "run a keyword research report"
  - "audit this website's content"
  - "check technical SEO issues"
  - "analyze competitor SEO gaps"
  - "generate an SEO content brief"
  - "create a content marketing strategy"
  - "track SERP rankings"
---

# 📈 SEO & Content Marketing Skill Factory

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill suite provides AI-powered SEO and content marketing capabilities through a structured command interface. Derived from `alirezarezvani/claude-code-skill-factory`, it delivers 10 specialized commands and 5 multi-step workflows for keyword research, content audits, technical SEO, and content strategy.

## What This Project Does

The SEO & Content Marketing Skill Factory provides:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplicate detection, cannibalization reports
- **Technical SEO** — crawl budget, Core Web Vitals, schema, indexability
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — AI briefs, editorial calendars, data-driven planning
- **Rank Tracking** — SERP monitoring with volatility alerts
- **Link Building** — prospecting, outreach templates, authority building
- **Local SEO** — NAP consistency, GBP optimization, citation audits

All commands output structured, actionable reports with visual progress tracking.

## Installation

### Clone the Skill Suite

```bash
# Clone into Claude Code skills directory
git clone https://github.com/JaguarPillage/r04-alirezarezvani-claude-code-skill-factory-seo.git \
  ~/.claude/skills/seo-content-marketing

# Or copy into your preferred skills location
cp -r /path/to/r04-alirezarezvani-claude-code-skill-factory-seo \
  ~/.claude/skills/seo-content-marketing
```

### Register with Claude Code

In a Claude Code session:

```bash
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Verify Installation

```bash
# List available commands
/help seo

# Test a basic command
/keyword-research --help
```

## Core Commands

### `/keyword-research`

Deep keyword clustering with opportunity scoring and SERP intent analysis.

```bash
# Basic keyword research
/keyword-research "digital marketing tools"

# With advanced options
/keyword-research "email marketing" \
  --volume-min 1000 \
  --difficulty-max 50 \
  --intent commercial \
  --output json

# Cluster keywords by topic
/keyword-research --cluster \
  --input keywords.csv \
  --format table
```

**Output Structure:**
- Keyword clusters with semantic grouping
- Search volume and difficulty scores
- SERP intent classification (informational/commercial/transactional/navigational)
- Opportunity score (volume vs. difficulty)
- Related questions and PAA data

### `/content-audit`

Full-site content quality analysis with duplicate detection.

```bash
# Audit entire domain
/content-audit https://example.com --scope full

# Audit specific section
/content-audit https://example.com/blog \
  --scope section \
  --min-words 500

# Export detailed report
/content-audit https://example.com \
  --export csv \
  --include-metrics \
  --check-cannibalization
```

**Output Structure:**
- Content quality scores (0-100)
- Thin content detection (<300 words)
- Duplicate/near-duplicate pages
- Keyword cannibalization report
- Missing meta tags and title issues
- Internal linking opportunities

### `/technical-seo`

Comprehensive technical SEO audit covering crawlability, performance, and indexability.

```bash
# Full technical audit
/technical-seo https://example.com

# Focus on Core Web Vitals
/technical-seo https://example.com --focus cwv

# Check indexability issues
/technical-seo https://example.com \
  --check-robots \
  --check-sitemap \
  --check-canonicals
```

**Checks Performed:**
- Crawl budget optimization
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Robots.txt and sitemap analysis
- Canonical tag implementation
- Mobile-friendliness
- HTTPS and security headers
- Structured data errors

### `/competitor-gap`

Identify SEO opportunities through competitor analysis.

```bash
# Analyze competitor gaps
/competitor-gap \
  --your-domain example.com \
  --competitors competitor1.com,competitor2.com

# Backlink gap analysis
/competitor-gap \
  --domain example.com \
  --competitors competitor.com \
  --focus backlinks \
  --min-dr 30

# Content gap analysis
/competitor-gap \
  --domain example.com \
  --competitors competitor.com \
  --focus content \
  --export keywords.csv
```

**Output Structure:**
- Keywords competitors rank for (but you don't)
- Backlink opportunities (sites linking to competitors)
- Featured snippet opportunities
- Topic coverage gaps
- DA/DR comparison metrics

### `/content-brief`

Generate AI-powered SEO content briefs.

```bash
# Create content brief
/content-brief "best project management software 2024"

# With custom parameters
/content-brief "email marketing guide" \
  --target-words 2500 \
  --audience "small business owners" \
  --tone professional \
  --include-outline

# Generate from keyword list
/content-brief \
  --keywords keywords.csv \
  --batch \
  --output briefs/
```

**Brief Includes:**
- Target keyword and semantically related terms
- Recommended word count
- Content outline with H2/H3 structure
- NLP terms to include
- Questions to answer (from PAA)
- Competitor content analysis
- Internal linking suggestions

### `/serp-monitor`

Track rankings with volatility alerts and CTR optimization.

```bash
# Monitor keywords
/serp-monitor \
  --domain example.com \
  --keywords keywords.csv \
  --frequency daily

# Generate ranking report
/serp-monitor \
  --domain example.com \
  --date-range "2024-01-01:2024-01-31" \
  --export report.pdf

# Check SERP features
/serp-monitor \
  --keyword "project management" \
  --analyze-features
```

**Tracking Features:**
- Daily rank tracking (positions 1-100)
- Volatility alerts for rank changes >5 positions
- SERP feature tracking (featured snippets, PAA, local pack)
- CTR estimation and optimization tips
- Competitor movement alerts

### `/link-prospecting`

Find quality backlink opportunities with outreach templates.

```bash
# Find link prospects
/link-prospecting \
  --topic "content marketing" \
  --min-da 30 \
  --max-results 100

# Filter by link type
/link-prospecting \
  --niche "saas" \
  --link-types "guest-post,resource-page" \
  --export prospects.csv

# Include outreach templates
/link-prospecting \
  --topic "digital marketing" \
  --generate-templates \
  --personalize
```

**Output Structure:**
- Prospect domain with DA/DR scores
- Contact information (email, social)
- Link type classification
- Relevance score
- Personalized outreach templates
- Follow-up sequence

### `/page-speed-seo`

Diagnose page speed issues mapped to ranking impact.

```bash
# Analyze page speed
/page-speed-seo https://example.com

# Focus on specific metrics
/page-speed-seo https://example.com/page \
  --focus lcp,cls \
  --device mobile

# Generate optimization report
/page-speed-seo https://example.com \
  --full-audit \
  --export optimization-plan.md
```

**Analysis Includes:**
- Core Web Vitals scores
- Render-blocking resources
- LCP optimization opportunities
- CLS layout shift sources
- FID/INP interaction delays
- Image optimization recommendations
- JavaScript execution time
- Ranking impact estimation

### `/local-seo`

Local SEO audit with NAP consistency and GBP optimization.

```bash
# Local SEO audit
/local-seo \
  --business "Acme Coffee Shop" \
  --location "Seattle, WA"

# NAP consistency check
/local-seo \
  --check-citations \
  --business-name "Acme Coffee" \
  --export inconsistencies.csv

# GBP optimization
/local-seo \
  --gbp-url "https://g.page/acme-coffee" \
  --analyze-profile \
  --competitor-compare
```

**Audit Components:**
- NAP consistency across directories
- Google Business Profile optimization
- Local citation audit (Yelp, Facebook, etc.)
- Schema LocalBusiness markup
- Review management analysis
- Local pack ranking factors

### `/content-calendar`

Data-driven editorial calendar from search demand and seasonality.

```bash
# Generate content calendar
/content-calendar \
  --topics "email marketing,crm,sales automation" \
  --months 3 \
  --frequency weekly

# Seasonal planning
/content-calendar \
  --industry ecommerce \
  --include-seasonality \
  --start-date 2024-06-01 \
  --export calendar.csv

# From keyword research
/content-calendar \
  --keywords keywords.csv \
  --format google-sheets \
  --assign-to team.json
```

**Calendar Features:**
- Search demand-based topic scheduling
- Seasonal trend integration
- Content type recommendations (blog, video, infographic)
- Target keywords per piece
- Publishing frequency optimization
- Team assignment and deadlines

## Workflows (Multi-Step Processes)

### `full-seo-sprint`

Comprehensive 12-step SEO sprint from audit to implementation.

```bash
/workflows:full-seo-sprint \
  --domain example.com \
  --scope full \
  --duration 4-weeks
```

**Sprint Steps:**
1. Initial technical audit
2. Content quality assessment
3. Keyword research and mapping
4. Competitor gap analysis
5. On-page optimization plan
6. Content creation roadmap
7. Link building strategy
8. Technical fixes prioritization
9. Implementation tracking
10. Performance monitoring
11. Iteration planning
12. Reporting and recommendations

### `launch-seo`

Pre-launch SEO checklist and validation.

```bash
/workflows:launch-seo \
  --domain newsite.com \
  --staging-url staging.newsite.com \
  --launch-date 2024-06-15
```

**Checklist:**
- Canonical tag validation
- Hreflang implementation (if international)
- XML sitemap generation and submission
- Robots.txt configuration
- 301 redirect mapping (if migration)
- Meta tags and schema markup
- Mobile-friendliness verification
- Page speed baseline
- Analytics and Search Console setup

### `content-refresh`

Identify and refresh underperforming pages.

```bash
/workflows:content-refresh \
  --domain example.com \
  --min-age 12-months \
  --rank-drop 5-positions
```

**Process:**
1. Identify pages with ranking declines
2. Analyze current content quality
3. Research updated keyword intent
4. Generate refresh recommendations
5. Create updated content briefs
6. Track post-refresh performance

### `authority-building`

End-to-end digital PR and link building campaign.

```bash
/workflows:authority-building \
  --domain example.com \
  --target-da 50 \
  --budget $5000 \
  --duration 6-months
```

**Campaign Steps:**
1. Authority baseline assessment
2. Link-worthy asset creation
3. Prospect list building
4. Outreach campaign execution
5. Relationship nurturing
6. Link acquisition tracking
7. Authority metric monitoring

### `ai-content-pipeline`

Automated content pipeline from keyword to publish.

```bash
/workflows:ai-content-pipeline \
  --topics topics.csv \
  --output-dir content/ \
  --auto-publish false \
  --review-required true
```

**Pipeline Stages:**
1. Keyword research and selection
2. AI content brief generation
3. Draft creation (with AI assistance)
4. SEO optimization
5. Quality review
6. Image and media addition
7. Internal linking
8. Publishing and indexing
9. Performance tracking

## Configuration

### Environment Variables

```bash
# API credentials (store in .env)
export SEO_SERP_API_KEY="${YOUR_SERP_API_KEY}"
export SEO_BACKLINK_API_KEY="${YOUR_BACKLINK_API_KEY}"
export SEO_KEYWORD_API_KEY="${YOUR_KEYWORD_TOOL_API_KEY}"
export SEO_GSC_CREDENTIALS="${PATH_TO_GSC_JSON}"
export SEO_GA_CREDENTIALS="${PATH_TO_GA_JSON}"

# Default settings
export SEO_DEFAULT_LOCATION="United States"
export SEO_DEFAULT_LANGUAGE="en"
export SEO_OUTPUT_FORMAT="markdown"
export SEO_PROGRESS_DISPLAY="verbose"
```

### Configuration File

Create `~/.claude/skills/seo-content-marketing/config.yaml`:

```yaml
# API Configuration
apis:
  serp:
    provider: "serpapi" # or "dataforseo", "semrush"
    rate_limit: 100
  backlinks:
    provider: "ahrefs" # or "moz", "majestic"
    rate_limit: 50
  keywords:
    provider: "google-keyword-planner"
    fallback: "ubersuggest"

# Default Settings
defaults:
  location: "United States"
  language: "en"
  device: "desktop"
  search_engine: "google.com"
  
# Output Preferences
output:
  format: "markdown" # or "json", "csv", "pdf"
  verbosity: "detailed" # or "summary", "quiet"
  progress_bars: true
  color_coding: true

# Thresholds
thresholds:
  keyword_difficulty_high: 70
  keyword_difficulty_medium: 40
  content_quality_good: 75
  page_speed_good: 90
  da_quality_link: 30

# Workflow Settings
workflows:
  auto_export: true
  checkpoint_saves: true
  notification_webhooks:
    - "${SLACK_WEBHOOK_URL}"
```

## Common Patterns

### Pattern 1: New Site SEO Setup

```bash
# Step 1: Technical foundation
/technical-seo https://newsite.com \
  --export technical-audit.md

# Step 2: Keyword research
/keyword-research \
  --seed-keywords "product category 1, product category 2" \
  --volume-min 500 \
  --export keywords.csv

# Step 3: Content planning
/content-calendar \
  --keywords keywords.csv \
  --months 6 \
  --export editorial-calendar.csv

# Step 4: Pre-launch checklist
/workflows:launch-seo \
  --domain newsite.com \
  --launch-date 2024-07-01
```

### Pattern 2: Recover Lost Rankings

```bash
# Step 1: Identify affected pages
/serp-monitor \
  --domain example.com \
  --date-range "2024-01-01:2024-03-31" \
  --filter rank-drop \
  --export declining-pages.csv

# Step 2: Content audit
/content-audit https://example.com \
  --pages declining-pages.csv \
  --detailed

# Step 3: Refresh workflow
/workflows:content-refresh \
  --pages declining-pages.csv \
  --priority high
```

### Pattern 3: Competitor Overtake Strategy

```bash
# Step 1: Gap analysis
/competitor-gap \
  --your-domain example.com \
  --competitors top-competitor.com \
  --export gaps.json

# Step 2: Keyword targeting
/keyword-research \
  --import gaps.json \
  --filter opportunity-score \
  --sort desc

# Step 3: Content briefs
/content-brief \
  --keywords opportunity-keywords.csv \
  --batch \
  --competitor-analysis

# Step 4: Link building
/link-prospecting \
  --competitor-backlinks top-competitor.com \
  --min-dr 30 \
  --export prospects.csv
```

### Pattern 4: Local Business SEO

```bash
# Step 1: Local audit
/local-seo \
  --business "Acme Plumbing" \
  --location "Austin, TX" \
  --export local-audit.md

# Step 2: Citation building
/local-seo \
  --build-citations \
  --directories yelp,facebook,yellowpages \
  --nap "Acme Plumbing, 123 Main St, Austin TX 78701, (512) 555-1234"

# Step 3: GBP optimization
/local-seo \
  --optimize-gbp \
  --add-posts \
  --respond-reviews

# Step 4: Local content
/content-calendar \
  --local-focus "Austin plumbing" \
  --include-local-keywords \
  --months 3
```

### Pattern 5: Enterprise Content Audit

```bash
# Step 1: Crawl and categorize
/content-audit https://enterprise.com \
  --scope full \
  --categorize-by-type \
  --export full-audit.csv

# Step 2: Identify issues
/content-audit https://enterprise.com \
  --find-duplicates \
  --find-cannibalization \
  --find-thin-content \
  --export issues.csv

# Step 3: Prioritize fixes
/content-audit \
  --import issues.csv \
  --score-by-traffic \
  --export prioritized-fixes.csv

# Step 4: Track implementation
/content-audit \
  --track-changes \
  --baseline full-audit.csv \
  --current-check \
  --export progress-report.md
```

## Real Code Examples

### Using Commands Programmatically

```python
# Python example: Automate keyword research
import subprocess
import json
import pandas as pd

def run_keyword_research(seed_keywords, output_file):
    """Run keyword research command and parse results."""
    cmd = [
        "/keyword-research",
        ",".join(seed_keywords),
        "--volume-min", "1000",
        "--output", "json"
    ]
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True
    )
    
    data = json.loads(result.stdout)
    
    # Convert to DataFrame for analysis
    df = pd.DataFrame(data['keywords'])
    df.to_csv(output_file, index=False)
    
    return df

# Use the function
keywords = run_keyword_research(
    ["content marketing", "seo tools", "link building"],
    "keyword-data.csv"
)

# Filter high-opportunity keywords
opportunities = keywords[
    (keywords['opportunity_score'] > 70) &
    (keywords['difficulty'] < 50)
]

print(f"Found {len(opportunities)} high-opportunity keywords")
```

### Workflow Integration

```javascript
// Node.js example: Content pipeline automation
const { exec } = require('child_process');
const util = require('util');
const execPromise = util.promisify(exec);

async function contentPipeline(topics) {
    const results = {
        keywords: [],
        briefs: [],
        calendar: null
    };
    
    // Step 1: Keyword research for each topic
    for (const topic of topics) {
        const { stdout } = await execPromise(
            `/keyword-research "${topic}" --output json`
        );
        results.keywords.push(JSON.parse(stdout));
    }
    
    // Step 2: Generate content briefs
    const allKeywords = results.keywords.flat();
    for (const keyword of allKeywords.slice(0, 10)) {
        const { stdout } = await execPromise(
            `/content-brief "${keyword.term}" --output json`
        );
        results.briefs.push(JSON.parse(stdout));
    }
    
    // Step 3: Create editorial calendar
    const { stdout } = await execPromise(
        `/content-calendar --topics ${topics.join(',')} --months 3 --output json`
    );
    results.calendar = JSON.parse(stdout);
    
    return results;
}

// Run pipeline
contentPipeline(['email marketing', 'crm software', 'sales automation'])
    .then(results => {
        console.log(`Generated ${results.briefs.length} content briefs`);
        console.log(`Calendar has ${results.calendar.items.length} items`);
    })
    .catch(console.error);
```

### Batch Processing

```bash
#!/bin/bash
# Batch audit multiple domains

DOMAINS_FILE="domains.txt"
OUTPUT_DIR="audits"

mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
    echo "Auditing $domain..."
    
    # Technical SEO
    /technical-seo "$domain" \
        --export "$OUTPUT_DIR/${domain}-technical.json"
    
    # Content audit
    /content-audit "$domain" \
        --scope full \
        --export "$OUTPUT_DIR/${domain}-content.csv"
    
    # Page speed
    /page-speed-seo "$domain" \
        --export "$OUTPUT_DIR/${domain}-speed.json"
    
    echo "Completed $domain"
    sleep 5  # Rate limiting
    
done < "$DOMAINS_FILE"

echo "All audits complete. Results in $OUTPUT_DIR/"
```

## Troubleshooting

### API Rate Limits

```bash
# Check current rate limit status
/seo-config --check-limits

# Adjust rate limiting
export SEO_RATE_LIMIT_DELAY=2  # seconds between requests

# Use batch mode with delays
/keyword-research \
    --batch keywords.csv \
    --rate-limit 30  # requests per minute
```

### Large Site Audits

```bash
# For sites with 10,000+ pages, use progressive crawl
/content-audit https://largesite.com \
    --progressive \
    --max-pages 1000 \
    --checkpoint-every 250 \
    --resume-from checkpoint-750.json

# Or split by section
/content-audit https://largesite.com/blog --scope section
/content-audit https://largesite.com/products --scope section
```

### Missing Data

```bash
# Verify API credentials
echo $SEO_SERP_API_KEY  # should not be empty

# Test API connectivity
/seo-config --test-apis

# Use fallback providers
/keyword-research "test" \
    --provider-fallback ubersuggest,wordtracker
```

### Performance Issues

```bash
# Enable caching
export SEO_ENABLE_CACHE=true
export SEO_CACHE_TTL=86400  # 24 hours

# Reduce verbosity
/content-audit https://example.com \
    --verbosity summary \
    --no-progress-bars

# Use parallel processing
/content-audit https://example.com \
    --parallel 4  # concurrent workers
```

### Export Format Issues

```bash
# Force specific format
/keyword-research "topic" \
    --output json \
    --pretty

# Convert between formats
/seo-convert \
    --input report.json \
    --output report.csv \
    --format csv

# Validate output
/seo-validate report.json
```

## Advanced Usage

### Custom Scoring Models

```yaml
# Add to config.yaml
scoring:
  keyword_opportunity:
    formula: "(volume / 100) * (100 - difficulty) * intent_multiplier"
    intent_multipliers:
      commercial: 1.2
      transactional: 1.5
      informational: 0.8
      navigational: 0.5
  
  content_quality:
    weights:
      word_count: 0.2
      readability: 0.15
      keyword_usage: 0.25
      internal_links: 0.15
      external_links: 0.1
      multimedia: 0.15
```

### Webhook Notifications

```yaml
# Add to config.yaml
webhooks:
  on_audit_complete:
    url: "${SLACK_WEBHOOK_URL}"
    payload:
      text: "SEO audit complete for {{domain}}"
      attachments:
        - title: "Results Summary"
          fields:
            - title: "Pages Crawled"
              value: "{{pages_crawled}}"
            - title: "Issues Found"
              value: "{{issues_count}}"
  
  on_rank_change:
    url: "${DISCORD_WEBHOOK_URL}"
    threshold: 5  # positions
    payload:
      content: "Ranking alert: {{keyword}} moved {{change}} positions"
```

### Integration with Analytics

```bash
# Import Google Search Console data
/serp-monitor \
    --import-gsc \
    --date-range "last-90-days" \
    --export gsc-data.csv

# Combine with Analytics
/content-audit https://example.com \
    --include-ga-metrics \
    --metrics "sessions,bounce_rate,avg_time_on_page"

# Cross-reference
/keyword-research \
    --augment-with-gsc \
    --min-clicks 100
```

This skill provides comprehensive SEO and content marketing capabilities optimized for AI coding agents to help developers implement effective search optimization and content strategies.
