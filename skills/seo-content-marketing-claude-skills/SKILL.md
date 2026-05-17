---
name: seo-content-marketing-claude-skills
description: SEO & content marketing command suite with keyword research, content audits, technical SEO, competitor analysis, and automated content workflows
triggers:
  - analyze SEO performance for this website
  - run a keyword research analysis
  - audit content for SEO issues
  - check technical SEO health
  - create an SEO content brief
  - analyze competitor keyword gaps
  - generate a content calendar based on search demand
  - monitor SERP rankings and volatility
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive SEO and content marketing command suite derived from [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills). Provides 10 specialized commands and 5 multi-step workflows for keyword research, content audits, technical SEO, SERP analysis, and content strategy — all with structured output and visual progress tracking.

## What This Project Does

This skill suite gives Claude Code structured commands for:

- **Keyword research** with clustering, intent mapping, and opportunity scoring
- **Content audits** including quality scoring, duplication detection, and cannibalization reports
- **Technical SEO** covering crawl budget, Core Web Vitals, schema markup, and indexability
- **Competitor analysis** for backlink gaps, topic gaps, and featured snippet opportunities
- **Content strategy** with AI-generated briefs, editorial calendars, and automated pipelines
- **Local SEO** for NAP consistency and Google Business Profile optimization
- **Performance monitoring** with rank tracking, volatility alerts, and CTR optimization

## Installation

### Clone into Claude Skills Directory

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/

# Clone the repository
git clone https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo.git \
  ~/.claude/skills/seo-content-marketing/

# Register the skill in Claude Code
# In a Claude Code session, run:
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Manual Installation

```bash
# Download and extract
curl -L https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo/archive/main.zip -o seo-skills.zip
unzip seo-skills.zip -d ~/.claude/skills/
mv ~/.claude/skills/r02-alirezarezvani-claude-skills-seo-main ~/.claude/skills/seo-content-marketing/
```

## Core Commands

### `/keyword-research`

Deep keyword clustering and opportunity scoring with SERP intent mapping.

**Usage:**

```bash
/keyword-research "project management software"
/keyword-research "ecommerce platform" --region US --language en
/keyword-research "fitness apps" --output json
```

**Options:**
- `--region <CODE>` — Target region (US, UK, CA, etc.)
- `--language <CODE>` — Target language (en, es, fr, etc.)
- `--scope <TYPE>` — Analysis depth: quick, standard, deep
- `--output <FORMAT>` — Output format: md, json, csv

**Output Structure:**

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "project management software"   ║
╠══════════════════════════════════════════════════╣
║  Fetching seed keywords …    [██████████] 100% ✓     ║
║  Clustering by intent …      [██████████] 100% ✓     ║
║  Scoring opportunities …     [████████░░]  80%       ║
╚══════════════════════════════════════════════════╝

┌─────────────────────────┬────────┬────┬──────────┬─────────┐
│ Keyword                 │ Volume │ KD │ Intent   │ Opp Score│
├─────────────────────────┼────────┼────┼──────────┼─────────┤
│ project management tool │ 12 400 │ 45 │ Commercial│   8.2   │
│ best PM software        │  8 100 │ 38 │ Commercial│   8.7   │
│ PM software for teams   │  6 200 │ 32 │ Commercial│   9.1   │
│ free project tracker    │  4 900 │ 28 │ Commercial│   7.8   │
└─────────────────────────┴────────┴────┴──────────┴─────────┘

ACTION PLAN:
🎯 Quick Wins (0-2 weeks)
  □ Target "PM software for teams" (low KD, high volume)
  □ Create comparison content for "best PM software"
  
📈 Medium-term (2-8 weeks)
  □ Build cluster around "project management tool"
  □ Develop free tool for "free project tracker"
```

### `/content-audit`

Full-site content quality scoring, duplication detection, and cannibalization report.

**Usage:**

```bash
/content-audit https://example.com
/content-audit https://example.com --scope full --depth 3
/content-audit https://example.com/blog --format detailed
```

**Options:**
- `--scope <TYPE>` — Audit scope: quick, standard, full
- `--depth <N>` — Crawl depth (1-5)
- `--format <TYPE>` — Report format: summary, detailed, technical

**Example Output:**

```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  example.com                   ║
╠══════════════════════════════════════════════════╣
║  Crawling pages …      [██████████] 100%  1 204 ✓║
║  Analyzing quality …   [████████░░]  85%    1 023║
║  Checking duplicates … [██████████] 100%      Done║
╚══════════════════════════════════════════════════╝

FINDINGS:
🔴 Critical (fix immediately)
  • 47 pages missing title tags
  • 12 pages with duplicate content (>85% match)
  • 8 keyword cannibalization clusters

🟠 High Priority (fix this sprint)
  • 203 pages missing meta descriptions
  • 89 thin content pages (<300 words)
  • 34 broken internal links

🟡 Medium Priority (fix this month)
  • 156 images missing alt text
  • 67 pages with slow load times (>3s)

CONTENT QUALITY SCORE: 67/100

TOP CANNIBALIZATION ISSUES:
/blog/seo-guide vs /resources/seo-tips vs /learn/seo-basics
  → Competing for "SEO guide" (12 mentions each)
  → Recommendation: Consolidate into single authoritative guide
```

### `/technical-seo`

Crawl budget analysis, Core Web Vitals, schema markup validation, and indexability audit.

**Usage:**

```bash
/technical-seo https://example.com
/technical-seo https://example.com --check-all
/technical-seo https://example.com --focus "core-web-vitals,schema"
```

**Options:**
- `--check-all` — Run comprehensive checks
- `--focus <AREAS>` — Comma-separated focus areas
- `--mobile-first` — Prioritize mobile issues

**Available Focus Areas:**
- `crawl-budget`
- `core-web-vitals`
- `schema-markup`
- `indexability`
- `robots-sitemap`
- `canonicalization`
- `hreflang`

**Example Integration:**

```javascript
// Programmatic usage (if exposing as API)
const seoAudit = require('seo-content-marketing-skills');

const results = await seoAudit.technicalSEO({
  url: 'https://example.com',
  checks: ['core-web-vitals', 'schema-markup'],
  mobileFirst: true
});

console.log(`CWV Score: ${results.coreWebVitals.score}`);
console.log(`Schema Issues: ${results.schema.errors.length}`);
```

### `/competitor-gap`

Backlink gap analysis, topic gap identification, and featured snippet opportunities.

**Usage:**

```bash
/competitor-gap https://mysite.com --competitors https://competitor1.com,https://competitor2.com
/competitor-gap https://mysite.com --auto-detect 5
/competitor-gap https://mysite.com --focus backlinks,topics
```

**Options:**
- `--competitors <URLS>` — Comma-separated competitor URLs
- `--auto-detect <N>` — Auto-detect N top competitors
- `--focus <TYPES>` — Analysis focus: backlinks, topics, snippets

### `/content-brief`

AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Usage:**

```bash
/content-brief "best CRM software for small business"
/content-brief "how to create a marketing plan" --format detailed
/content-brief "email marketing guide" --competitor-analysis 3
```

**Options:**
- `--format <TYPE>` — Brief format: standard, detailed, minimal
- `--competitor-analysis <N>` — Analyze top N ranking pages
- `--target-length <WORDS>` — Target word count

**Output Structure:**

```markdown
# Content Brief: "best CRM software for small business"

## Target Metrics
- **Primary Keyword**: best CRM software for small business
- **Search Volume**: 8,100/month
- **Keyword Difficulty**: 42
- **Target Word Count**: 2,800-3,200 words
- **Content Type**: Comparison/Listicle

## Search Intent
Commercial investigation — users researching CRM options before purchase

## Outline
1. Introduction (150-200 words)
   - Define CRM for small business context
   - Preview top recommendations
   
2. Top 10 CRM Solutions (2,000 words)
   - For each tool:
     • Key features
     • Pricing
     • Best for (specific use case)
     • Pros/cons
     
3. How to Choose (400 words)
   - Budget considerations
   - Must-have features
   - Integration requirements

## NLP Terms to Include
- customer relationship management
- sales pipeline
- contact management
- email integration
- mobile app
- free trial
- pricing plans
- small team
- automation features

## SERP Features to Target
✓ Featured Snippet (Comparison table)
✓ People Also Ask
  - "What is CRM software?"
  - "How much does CRM cost?"
  - "Do I need CRM for my small business?"

## Competitor Analysis
Top 3 ranking pages average:
- Word count: 2,950
- Images: 12
- Internal links: 8
- External links: 15
```

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization recommendations.

**Usage:**

```bash
/serp-monitor --keywords keywords.csv
/serp-monitor --url https://example.com --auto-extract
/serp-monitor --track "keyword1,keyword2,keyword3" --frequency daily
```

### `/link-prospecting`

Quality backlink prospect list with DA/DR filters and outreach templates.

**Usage:**

```bash
/link-prospecting "marketing automation" --min-dr 40
/link-prospecting "SaaS tools" --type "guest-post,resource-page" --limit 50
```

### `/page-speed-seo`

Render-blocking resource detection, LCP/CLS/FID diagnosis mapped to ranking impact.

**Usage:**

```bash
/page-speed-seo https://example.com/product-page
/page-speed-seo https://example.com --device mobile --priority ranking-impact
```

### `/local-seo`

NAP consistency checker, Google Business Profile optimization, and local citation audit.

**Usage:**

```bash
/local-seo "Acme Plumbing, New York, NY"
/local-seo --business-name "Joe's Pizza" --location "Brooklyn, NY" --check-citations
```

### `/content-calendar`

Data-driven editorial calendar built from search demand and seasonality data.

**Usage:**

```bash
/content-calendar --niche "fitness" --duration 90-days
/content-calendar --keywords keywords.csv --include-seasonality
/content-calendar --topics "SEO,content marketing,social media" --posts-per-week 3
```

**Output Example:**

```
╔══════════════════════════════════════════════════╗
║  Content Calendar  —  90 days, 3 posts/week     ║
╚══════════════════════════════════════════════════╝

WEEK 1 (Jun 5-11)
Mon, Jun 5  │ "Best Email Marketing Tools 2026"
            │ Target: email marketing software (8.1K vol)
            │ Type: Comparison
            
Wed, Jun 7  │ "How to Create an Email Sequence"
            │ Target: email sequence (3.6K vol)
            │ Type: How-to Guide
            
Fri, Jun 9  │ "Email Marketing Statistics 2026"
            │ Target: email marketing stats (2.2K vol)
            │ Type: Data/Listicle

SEASONAL OPPORTUNITIES:
🎃 October: Halloween marketing campaigns (+340% search volume)
🎄 November: Black Friday email templates (+520% search volume)
```

## Workflows (Multi-Step Processes)

### `full-seo-sprint`

12-step comprehensive SEO sprint from audit to implementation.

**Usage:**

```bash
/workflows:full-seo-sprint https://example.com --duration 4-weeks
```

**Workflow Steps:**

```
SPRINT OVERVIEW (4 weeks, 12 steps)

Week 1: Audit & Analysis
  □ Step 1: Technical SEO audit
  □ Step 2: Content audit
  □ Step 3: Competitor gap analysis
  
Week 2: Strategy & Planning
  □ Step 4: Keyword research & clustering
  □ Step 5: Content strategy development
  □ Step 6: Technical roadmap creation
  
Week 3: Quick Wins
  □ Step 7: On-page optimization
  □ Step 8: Content refresh (top 10 pages)
  □ Step 9: Schema markup implementation
  
Week 4: Scale & Automate
  □ Step 10: Content brief generation (10 topics)
  □ Step 11: Link prospecting campaign
  □ Step 12: Monitoring dashboard setup
```

### `launch-seo`

Pre-launch SEO checklist with canonical tags, hreflang, and sitemap validation.

**Usage:**

```bash
/workflows:launch-seo https://staging.example.com
```

### `content-refresh`

Identify and refresh underperforming pages to recover lost rankings.

**Usage:**

```bash
/workflows:content-refresh https://example.com --lookback 90-days
```

### `authority-building`

End-to-end digital PR and link-building campaign workflow.

**Usage:**

```bash
/workflows:authority-building --niche "B2B SaaS" --target-links 50
```

### `ai-content-pipeline`

Automated pipeline from keyword research to published, optimized content.

**Usage:**

```bash
/workflows:ai-content-pipeline --seed-keywords keywords.csv --output-dir ./content
```

## Configuration

### Environment Variables

```bash
# Optional: API integrations for enhanced data
export AHREFS_API_KEY=your_ahrefs_key
export SEMRUSH_API_KEY=your_semrush_key
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=path/to/credentials.json
export SCREAMING_FROG_LICENSE=your_license_key

# Output preferences
export SEO_SKILLS_OUTPUT_FORMAT=markdown  # markdown, json, csv
export SEO_SKILLS_PROGRESS_BAR=true       # Show progress bars
export SEO_SKILLS_COLOR_OUTPUT=true       # Colored terminal output
```

### Config File

Create `~/.claude/skills/seo-content-marketing/config.yaml`:

```yaml
defaults:
  output_format: markdown
  progress_display: true
  color_output: true
  
keyword_research:
  default_region: US
  default_language: en
  min_volume: 100
  max_difficulty: 60
  
content_audit:
  default_scope: standard
  crawl_depth: 3
  min_word_count: 300
  
technical_seo:
  checks:
    - core-web-vitals
    - schema-markup
    - indexability
    - robots-sitemap
  mobile_first: true
  
competitor_gap:
  auto_detect_competitors: 5
  min_dr_threshold: 30
  
content_brief:
  default_format: detailed
  competitor_analysis_depth: 3
  include_nlp_terms: true
```

## Common Usage Patterns

### Pattern 1: New Website SEO Setup

```bash
# Step 1: Pre-launch audit
/workflows:launch-seo https://staging.newsite.com

# Step 2: Initial keyword research
/keyword-research "primary product category" --scope deep

# Step 3: Create content calendar
/content-calendar --keywords keywords.csv --duration 180-days

# Step 4: Generate content briefs
/content-brief "top priority keyword 1"
/content-brief "top priority keyword 2"
```

### Pattern 2: Existing Site Optimization

```bash
# Step 1: Comprehensive audit
/workflows:full-seo-sprint https://example.com --duration 4-weeks

# Step 2: Identify quick wins
/content-refresh https://example.com --lookback 90-days

# Step 3: Fix technical issues
/technical-seo https://example.com --check-all

# Step 4: Monitor progress
/serp-monitor --url https://example.com --auto-extract
```

### Pattern 3: Competitor Analysis & Gap Exploitation

```bash
# Step 1: Identify gaps
/competitor-gap https://mysite.com --auto-detect 5

# Step 2: Extract opportunities
/keyword-research --from-gap-analysis --focus untapped

# Step 3: Create content briefs
/content-brief "gap keyword 1" --competitor-analysis 3
/content-brief "gap keyword 2" --competitor-analysis 3

# Step 4: Build authority
/workflows:authority-building --focus "topics-from-gap-analysis"
```

### Pattern 4: Local Business SEO

```bash
# Step 1: Local SEO audit
/local-seo "Business Name, City, State" --check-citations

# Step 2: Keyword research with local modifiers
/keyword-research "service + city" --region US --include-local-variants

# Step 3: Content calendar for local topics
/content-calendar --local-focus "city-name" --posts-per-week 2

# Step 4: Monitor local rankings
/serp-monitor --keywords local-keywords.csv --geo-location "City, State"
```

## Troubleshooting

### Command Not Recognized

```bash
# Verify skill is registered
ls ~/.claude/skills/seo-content-marketing/

# Re-register the skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Slow Crawl Performance

For large sites, optimize crawl settings:

```bash
# Use quick scope for initial analysis
/content-audit https://example.com --scope quick

# Limit crawl depth
/content-audit https://example.com --depth 2

# Focus on specific sections
/content-audit https://example.com/blog --depth 3
```

### API Rate Limiting

If using external SEO tool APIs:

```bash
# Reduce concurrent requests in config.yaml
rate_limiting:
  requests_per_minute: 10
  retry_after_seconds: 60

# Use cached results when available
/keyword-research "keyword" --use-cache --cache-ttl 86400
```

### Incomplete Results

For better data coverage:

```bash
# Increase analysis depth
/keyword-research "keyword" --scope deep

# Extend competitor analysis
/competitor-gap https://mysite.com --auto-detect 10

# Run comprehensive checks
/technical-seo https://example.com --check-all
```

## Advanced Integration Examples

### Scripting Workflow Automation

```bash
#!/bin/bash
# Monthly SEO maintenance script

SITE_URL="https://example.com"
DATE=$(date +%Y-%m-%d)
REPORT_DIR="./seo-reports/$DATE"

mkdir -p "$REPORT_DIR"

# Run audits
/technical-seo "$SITE_URL" --output json > "$REPORT_DIR/technical.json"
/content-audit "$SITE_URL" --output json > "$REPORT_DIR/content.json"

# Check rankings
/serp-monitor --url "$SITE_URL" --auto-extract --output csv > "$REPORT_DIR/rankings.csv"

# Generate refresh opportunities
/content-refresh "$SITE_URL" --lookback 30-days --output md > "$REPORT_DIR/refresh-opportunities.md"

echo "SEO reports generated in $REPORT_DIR"
```

### Continuous Monitoring Setup

```bash
# Set up daily rank tracking
/serp-monitor --keywords keywords.csv --frequency daily --alert-threshold 5

# Configure weekly content audits
/content-audit https://example.com --schedule weekly --email-report admin@example.com

# Monitor Core Web Vitals
/page-speed-seo https://example.com --track-vitals --alert-on-regression
```

## Output Format Reference

### Markdown (Default)

Rich formatted reports with tables, checklists, and visual progress indicators.

### JSON

Structured data for programmatic consumption:

```json
{
  "command": "keyword-research",
  "target": "project management software",
  "timestamp": "2026-05-11T10:30:00Z",
  "results": {
    "clusters": [
      {
        "name": "Commercial Intent",
        "keywords": [
          {
            "term": "best PM software",
            "volume": 8100,
            "difficulty": 38,
            "intent": "commercial",
            "opportunity_score": 8.7
          }
        ]
      }
    ],
    "total_keywords": 247,
    "avg_difficulty": 41.2
  }
}
```

### CSV

Spreadsheet-compatible format for bulk analysis:

```csv
keyword,volume,difficulty,intent,opportunity_score
"project management tool",12400,45,"commercial",8.2
"best PM software",8100,38,"commercial",8.7
```

## Tips for AI Agents

1. **Always confirm scope** before running full-site audits on large websites
2. **Start with quick wins** — use `--scope quick` for initial assessments
3. **Chain commands** — use output from one command as input to the next
4. **Cache aggressively** — keyword and SERP data changes slowly
5. **Prioritize by severity** — focus on 🔴 critical issues first
6. **Use workflows for complex tasks** — they orchestrate multiple commands efficiently
7. **Export data** — generate reports in multiple formats for different stakeholders
8. **Monitor trends** — compare audit results over time to track progress

---

**Source**: Derived from [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills)  
**License**: MIT  
**Documentation**: This skill provides command-line interface to SEO workflows. For API integration, refer to the project's programmatic usage examples above.
