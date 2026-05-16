---
name: seo-content-marketing-claude-skills
description: SEO & content marketing command suite with keyword research, technical audits, SERP analysis, and content strategy workflows
triggers:
  - "help me with SEO optimization"
  - "perform a content audit"
  - "find keyword opportunities"
  - "analyze technical SEO issues"
  - "create a content marketing strategy"
  - "audit my site for SEO problems"
  - "generate an SEO content brief"
  - "track SERP rankings"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill suite for SEO professionals and content marketers, providing structured commands and multi-step workflows for keyword research, technical audits, competitive analysis, and content strategy. Adapted from `jqueryscript/awesome-claude-code` with specialized SEO/marketing vocabulary and visual progress tracking.

## What This Skill Does

This skill provides **10 specialized SEO commands** and **5 multi-step workflows** covering:

- **Keyword Research**: Clustering, opportunity scoring, SERP intent mapping
- **Content Audits**: Quality scoring, duplication detection, cannibalization analysis
- **Technical SEO**: Crawl budget, Core Web Vitals, schema markup, indexability
- **Competitive Analysis**: Backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy**: Brief generation, editorial calendars, refresh workflows
- **Performance Monitoring**: Rank tracking, page speed audits, local SEO

All commands follow a consistent 5-step interaction pattern with structured output.

## Installation

### Method 1: Manual Installation

```bash
# Clone into Claude skills directory
mkdir -p ~/.claude/skills/
git clone https://github.com/FabledPackerRedeem/r05-jqueryscript-awesome-claude-code-seo.git \
  ~/.claude/skills/seo-content-marketing/

# In Claude Code session, load the skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Method 2: Direct Copy

```bash
# Copy skill directory to your project
cp -r /path/to/r05-jqueryscript-awesome-claude-code-seo ./skills/seo/

# Reference in your Claude session
/read ./skills/seo/SKILL.md
```

## Core Commands

### Keyword Research

**Command**: `/keyword-research <target>`

Performs deep keyword analysis with clustering, search intent mapping, and opportunity scoring.

```bash
# Basic keyword research
/keyword-research "fitness tracking apps"

# With filters and output format
/keyword-research "fitness tracking apps" --min-volume 500 --max-difficulty 40 --output csv
```

**Output Structure**:
- Keyword clusters by search intent (informational, navigational, transactional)
- Search volume, difficulty, CPC, and opportunity scores
- SERP feature presence (featured snippets, PAA, video, etc.)
- Related questions and long-tail variations
- Prioritized action list

**Example Output**:

```
┌────────────────────────────┬────────┬────────┬─────┬───────┬──────────────┐
│ Keyword                    │ Volume │ Diff.  │ CPC │ Score │ SERP Features│
├────────────────────────────┼────────┼────────┼─────┼───────┼──────────────┤
│ best fitness tracking apps │ 12,400 │   45   │ 3.2 │  8.7  │ FS, PAA, V   │
│ fitness tracker comparison │  5,800 │   38   │ 2.9 │  9.1  │ FS, IMG      │
│ free workout tracking app  │  8,200 │   32   │ 1.4 │  9.4  │ PAA, V       │
└────────────────────────────┴────────┴────────┴─────┴───────┴──────────────┘

FS = Featured Snippet | PAA = People Also Ask | V = Video | IMG = Images
```

### Content Audit

**Command**: `/content-audit`

Analyzes site content for quality, duplication, and cannibalization issues.

```bash
# Full site audit
/content-audit --scope full --output md

# Specific subdirectory
/content-audit --scope /blog/ --min-words 300

# Focus on specific issues
/content-audit --check cannibalization,duplicates,thin-content
```

**Analysis Includes**:
- Content quality scores (readability, depth, freshness)
- Duplicate/near-duplicate detection
- Keyword cannibalization mapping
- Thin content identification (<300 words)
- Internal linking opportunities
- Metadata completeness (titles, descriptions, headers)

**Example Output**:

```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  example.com/blog              ║
╠══════════════════════════════════════════════════╣
║  Scanning pages …      [██████████] 100%  247/247 ║
║  Quality scoring …     [██████████] 100%  Done ✓  ║
║  Checking duplicates … [██████████] 100%  Done ✓  ║
╚══════════════════════════════════════════════════╝

🔴 Critical Issues (3)
  • 18 pages with duplicate title tags
  • 12 keyword cannibalization clusters
  • 5 pages with <100 words

🟠 High Priority (7)
  • 42 pages missing meta descriptions
  • 23 pages with low readability scores
  • 15 orphan pages (no internal links)

🟡 Medium Priority (12)
  • 67 pages not updated in 12+ months
  • 34 pages with suboptimal word count
```

### Technical SEO Audit

**Command**: `/technical-seo`

Comprehensive technical SEO analysis covering crawlability, performance, and markup.

```bash
# Full technical audit
/technical-seo --scope full

# Specific checks
/technical-seo --check indexability,schema,performance

# With Core Web Vitals
/technical-seo --cwv --mobile
```

**Checks Include**:
- Robots.txt and sitemap validation
- Indexability and crawl budget analysis
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Mobile-friendliness
- HTTPS and security headers
- Canonical tag implementation
- Hreflang configuration (if applicable)

**Example Code Pattern** (Technical Audit Checklist):

```markdown
## Technical SEO Checklist

### Crawlability
- [ ] Robots.txt accessible at /robots.txt
- [ ] XML sitemap submitted to Search Console
- [ ] No critical pages blocked by robots.txt
- [ ] Sitemap contains only canonical URLs
- [ ] Max crawl depth ≤ 3 clicks from homepage

### Indexability
- [ ] No noindex on important pages
- [ ] Canonical tags point to correct URLs
- [ ] No redirect chains (301 → 301)
- [ ] 404 pages return proper status codes
- [ ] Pagination uses rel=next/prev or canonical

### Performance (Core Web Vitals)
- [ ] LCP < 2.5s (Largest Contentful Paint)
- [ ] FID < 100ms (First Input Delay)
- [ ] CLS < 0.1 (Cumulative Layout Shift)
- [ ] Server response time < 600ms
- [ ] Images lazy-loaded and optimized

### Schema Markup
- [ ] Organization schema on homepage
- [ ] Article schema on blog posts
- [ ] Breadcrumb schema on all pages
- [ ] FAQ schema where applicable
- [ ] Review schema for products/services
```

### Competitor Gap Analysis

**Command**: `/competitor-gap`

Identifies backlink, content, and ranking opportunities by analyzing competitors.

```bash
# Basic competitor analysis
/competitor-gap <your-domain> --competitors competitor1.com,competitor2.com

# Focus on specific gap types
/competitor-gap example.com --competitors comp1.com,comp2.com --gap-type backlinks,keywords

# With filters
/competitor-gap example.com --competitors comp1.com --min-dr 30 --max-gap 10
```

**Analysis Types**:
- **Backlink Gap**: Links competitors have that you don't
- **Keyword Gap**: Keywords competitors rank for that you don't
- **Content Gap**: Topics competitors cover that you don't
- **Featured Snippet Opportunities**: Snippets competitors own
- **SERP Feature Gap**: Features competitors appear in

**Example Output**:

```
## Backlink Gap Analysis

Competitor: competitor.com (DR 68)
Your Domain: example.com (DR 52)

┌─────────────────────────────┬────┬────┬──────────┬───────────┐
│ Linking Domain              │ DR │ DA │ Link Type│ Difficulty│
├─────────────────────────────┼────┼────┼──────────┼───────────┤
│ industry-blog.com           │ 72 │ 65 │ Editorial│    Low    │
│ news-site.com               │ 81 │ 78 │ Editorial│   Medium  │
│ resource-directory.com      │ 58 │ 54 │ Directory│    Low    │
└─────────────────────────────┴────┴────┴──────────┴───────────┘

## Keyword Gap Analysis

┌──────────────────────────┬────────┬─────────┬──────────┬─────────┐
│ Keyword                  │ Volume │ Your Pos│ Comp Pos │ Priority│
├──────────────────────────┼────────┼─────────┼──────────┼─────────┤
│ best project management  │ 18,400 │   N/A   │    3     │   High  │
│ project tracking software│  9,200 │   N/A   │    7     │   High  │
│ team collaboration tools │ 12,100 │    45   │    4     │  Medium │
└──────────────────────────┴────────┴─────────┴──────────┴─────────┘
```

### Content Brief Generator

**Command**: `/content-brief`

Generates AI-powered SEO content briefs with outlines, NLP terms, and optimization targets.

```bash
# Generate brief for target keyword
/content-brief "how to choose a CRM"

# With competitive analysis
/content-brief "how to choose a CRM" --analyze-top 10

# Specify content type
/content-brief "CRM comparison" --type comparison-guide --target-words 2500
```

**Brief Includes**:
- Primary and secondary keywords
- Search intent analysis
- Target word count and reading level
- Recommended headings (H1-H4 structure)
- NLP terms to include (entities, co-occurring phrases)
- Questions to answer (from PAA)
- Competitive content analysis
- Internal linking opportunities
- Image/media recommendations

**Example Output**:

```markdown
# Content Brief: "How to Choose a CRM"

## Overview
- **Primary Keyword**: how to choose a CRM
- **Search Volume**: 3,600/mo
- **Keyword Difficulty**: 42
- **Search Intent**: Informational/Commercial Investigation
- **Target Word Count**: 2,200-2,500 words
- **Reading Level**: Grade 8-10

## Recommended Structure

### H1: How to Choose the Right CRM for Your Business (2024 Guide)

### Introduction (150-200 words)
- Hook: CRM selection statistics
- Preview the decision framework
- Mention key factors to consider

### H2: What is a CRM? (300 words)
- Define CRM software
- Explain core functions
- Differentiate from other tools

### H2: 7 Key Factors When Choosing a CRM (1,200 words)

#### H3: 1. Identify Your Business Needs
- Sales pipeline management
- Contact management
- Marketing automation
- Customer service

#### H3: 2. Evaluate Ease of Use
- User interface considerations
- Learning curve
- Mobile accessibility

#### H3: 3. Integration Capabilities
- Email platforms
- Marketing tools
- Accounting software

[... additional sections ...]

## NLP Terms to Include
- customer relationship management
- sales pipeline
- contact management
- lead nurturing
- sales automation
- customer data
- CRM features
- implementation process

## Questions to Answer (from PAA)
1. What are the different types of CRM?
2. How much does CRM software cost?
3. What CRM is best for small businesses?
4. Do I need technical knowledge to use a CRM?

## Competitive Analysis
- Top 3 ranking pages average 2,340 words
- 85% include comparison tables
- 60% have embedded videos
- All include downloadable resources
```

### SERP Monitoring

**Command**: `/serp-monitor`

Track rankings, volatility, and CTR optimization opportunities.

```bash
# Monitor specific keywords
/serp-monitor --keywords "keyword1,keyword2,keyword3"

# Daily tracking report
/serp-monitor --daily --alert-threshold 5

# CTR analysis
/serp-monitor --analyze-ctr --suggest-improvements
```

**Provides**:
- Current rankings and position changes
- SERP volatility alerts
- CTR benchmarks vs. actual CTR
- Title/meta description optimization suggestions
- Featured snippet opportunities
- SERP feature presence tracking

### Link Prospecting

**Command**: `/link-prospecting`

Generate qualified backlink prospect lists with outreach templates.

```bash
# Find link prospects by topic
/link-prospecting "project management" --min-dr 30

# Specific link types
/link-prospecting "saas tools" --type guest-post,resource-page

# With outreach templates
/link-prospecting "productivity" --include-templates
```

**Output**:
- Qualified prospect list (domain, DR/DA, contact info)
- Link opportunity type (guest post, resource page, broken link, etc.)
- Outreach email templates
- Prioritization by acquisition difficulty

### Page Speed SEO Audit

**Command**: `/page-speed-seo`

Diagnose performance issues mapped to SEO ranking impact.

```bash
# Audit specific page
/page-speed-seo https://example.com/page

# Mobile focus
/page-speed-seo https://example.com --mobile --cwv

# Generate fix recommendations
/page-speed-seo https://example.com --recommendations
```

**Analysis**:
- Core Web Vitals (LCP, FID, CLS)
- Render-blocking resources
- Image optimization opportunities
- JavaScript/CSS minification
- Server response time
- Caching configuration
- SEO impact scoring for each issue

### Local SEO Audit

**Command**: `/local-seo`

Optimize for local search with NAP consistency, GBP, and citation audits.

```bash
# Full local SEO audit
/local-seo --business "Coffee Shop Name" --location "Portland, OR"

# NAP consistency check
/local-seo --check nap-consistency

# Citation audit
/local-seo --audit-citations --target-count 50
```

**Includes**:
- NAP (Name, Address, Phone) consistency across web
- Google Business Profile optimization
- Local citation audit (directories, listings)
- Review management recommendations
- Local keyword opportunities
- Schema markup for local business

### Content Calendar

**Command**: `/content-calendar`

Data-driven editorial calendar based on search demand and seasonality.

```bash
# Generate 3-month calendar
/content-calendar --duration 3months --niche "digital marketing"

# Seasonal content planning
/content-calendar --seasonal --year 2024

# Topic clustering
/content-calendar --cluster-topics --output csv
```

**Generates**:
- Topic ideas prioritized by search volume and opportunity
- Publishing schedule aligned with search trends
- Seasonal content recommendations
- Topic clusters for pillar/cluster strategy
- Keyword assignments per topic
- Content type recommendations (guide, listicle, video, etc.)

## Multi-Step Workflows

### Full SEO Sprint

**Workflow**: `full-seo-sprint`

12-step comprehensive SEO process from audit to execution.

```bash
/workflows:full-seo-sprint <domain> --scope full
```

**Steps**:
1. Technical SEO audit
2. Content audit
3. Backlink profile analysis
4. Keyword research and mapping
5. Competitor gap analysis
6. On-page optimization priorities
7. Content refresh opportunities
8. New content recommendations
9. Link building strategy
10. Local SEO optimization (if applicable)
11. Performance optimization
12. Measurement and tracking setup

**Outputs**: Comprehensive SEO roadmap with prioritized tasks, timelines, and success metrics.

### Launch SEO Workflow

**Workflow**: `launch-seo`

Pre-launch SEO checklist ensuring technical foundations are solid.

```bash
/workflows:launch-seo --site <staging-url>
```

**Checklist**:
- Robots.txt configuration
- XML sitemap generation and submission
- Canonical tag implementation
- Hreflang setup (multi-language sites)
- Meta tags (titles, descriptions)
- Schema markup deployment
- 301 redirect mapping (if migrating)
- Page speed baseline
- Mobile-friendliness verification
- Analytics and Search Console setup

### Content Refresh Workflow

**Workflow**: `content-refresh`

Identify and update underperforming content to recover rankings.

```bash
/workflows:content-refresh --min-age 6months --traffic-drop 20%
```

**Process**:
1. Identify declining pages (traffic/ranking drops)
2. Analyze why rankings dropped (freshness, comprehensiveness, competition)
3. Research current SERP landscape
4. Generate updated content brief
5. Refresh content (update stats, add sections, improve depth)
6. Optimize on-page elements
7. Re-promote via internal links and social
8. Monitor recovery

### Authority Building Campaign

**Workflow**: `authority-building`

End-to-end digital PR and link-building campaign.

```bash
/workflows:authority-building --niche "B2B SaaS" --duration 6months
```

**Campaign Components**:
1. Asset ideation (linkable content)
2. Asset creation plan
3. Prospect research and list building
4. Outreach template creation
5. Outreach execution timeline
6. Follow-up sequences
7. Relationship nurturing
8. Link acquisition tracking
9. Impact measurement

### AI Content Pipeline

**Workflow**: `ai-content-pipeline`

Automated content production from keyword to publication.

```bash
/workflows:ai-content-pipeline --topic "email marketing best practices"
```

**Pipeline Steps**:
1. Keyword research
2. Content brief generation
3. AI-assisted draft creation
4. SEO optimization (keywords, structure, internal links)
5. Quality review checklist
6. Meta data creation
7. Image/media sourcing
8. Publication preparation
9. Post-publish promotion

## Configuration

### Environment Variables

```bash
# API keys (if integrating with external tools)
export SEO_TOOL_API_KEY="your-api-key-here"
export SEARCH_CONSOLE_API_KEY="your-api-key-here"
export ANALYTICS_API_KEY="your-api-key-here"

# Default settings
export SEO_DEFAULT_SCOPE="full"
export SEO_OUTPUT_FORMAT="markdown"
export SEO_MIN_KEYWORD_VOLUME=100
export SEO_MAX_KEYWORD_DIFFICULTY=50
```

### Skill Configuration File

Create `~/.claude/skills/seo-config.yml`:

```yaml
defaults:
  output_format: markdown
  scope: full
  min_keyword_volume: 100
  max_keyword_difficulty: 50

integrations:
  search_console: enabled
  analytics: enabled
  
thresholds:
  content_quality_min: 70
  page_speed_min: 85
  mobile_friendly_min: 90
  
workflows:
  full_seo_sprint:
    duration_weeks: 12
    checkpoint_frequency: weekly
```

## Common Patterns

### Pattern 1: Monthly SEO Health Check

```bash
# Run monthly audit sequence
/technical-seo --scope full --output monthly-report.md
/content-audit --check quality,cannibalization
/serp-monitor --daily --alert-threshold 5
/page-speed-seo --cwv --mobile
```

### Pattern 2: New Content Creation

```bash
# Research → Brief → Create → Optimize
/keyword-research "target topic" --cluster
/content-brief "primary keyword" --analyze-top 10
# [Create content]
/technical-seo --check schema,metadata
/link-prospecting "topic" --include-templates
```

### Pattern 3: Competitive Conquest

```bash
# Identify and capture competitor opportunities
/competitor-gap <your-domain> --competitors <competitor>
/keyword-research <competitor-keyword> --intent transactional
/content-brief <target-keyword> --type pillar-page
# [Create superior content]
/link-prospecting <topic> --min-dr 40
```

### Pattern 4: Traffic Recovery

```bash
# Diagnose and fix traffic drops
/serp-monitor --analyze-volatility
/content-audit --scope declining-pages
/technical-seo --check indexability,performance
/workflows:content-refresh --traffic-drop 30%
```

## Troubleshooting

### Issue: Command Not Found

```bash
# Ensure skill is loaded
/read ~/.claude/skills/seo-content-marketing/SKILL.md

# Verify path
ls ~/.claude/skills/seo-content-marketing/
```

### Issue: Incomplete Analysis

**Symptom**: Partial results or missing data sections

**Solutions**:
```bash
# Increase scope
/technical-seo --scope full --depth comprehensive

# Check rate limits (if using APIs)
# Verify API keys are set
echo $SEO_TOOL_API_KEY

# Run with verbose logging
/content-audit --verbose --log audit.log
```

### Issue: Slow Performance

**Symptom**: Commands take too long to execute

**Solutions**:
```bash
# Narrow scope
/content-audit --scope /blog/ --limit 100

# Use sampling
/keyword-research "topic" --sample 500

# Run specific checks only
/technical-seo --check indexability,schema
```

### Issue: No Keyword Data

**Symptom**: Keyword research returns empty results

**Solutions**:
```bash
# Lower filters
/keyword-research "topic" --min-volume 10 --max-difficulty 80

# Check for typos in target keyword
/keyword-research "correct spelling"

# Broaden topic
/keyword-research "broader category term"
```

## Integration Examples

### Export to CSV for Further Analysis

```bash
/keyword-research "topic" --output csv --file keywords.csv
/content-audit --output csv --file audit.csv
/competitor-gap <domain> --output csv --file gaps.csv
```

### Pipe to Project Management

```bash
# Generate action items
/technical-seo --scope full --output tasks.md

# Format for Jira/Asana/Linear
/workflows:full-seo-sprint --export-tasks --format jira
```

### Integrate with Analytics

```bash
# Combine with GA4 data
export ANALYTICS_API_KEY="..."
/serp-monitor --combine-analytics --date-range 30days
```

## Best Practices

1. **Run audits regularly**: Monthly technical SEO and content audits catch issues early
2. **Prioritize by impact**: Focus on high-severity issues (🔴) before medium (🟠)
3. **Track changes**: Export reports to track progress over time
4. **Combine commands**: Use multiple commands together for comprehensive analysis
5. **Automate workflows**: Use multi-step workflows for repeatable processes
6. **Document findings**: Export results to markdown/CSV for stakeholder reports

## Example: Complete SEO Project

```bash
# Step 1: Initial audit
/technical-seo --scope full --output reports/technical-audit.md
/content-audit --scope full --output reports/content-audit.md

# Step 2: Competitive research
/competitor-gap example.com --competitors comp1.com,comp2.com --output reports/gap-analysis.md

# Step 3: Keyword strategy
/keyword-research "primary topic" --cluster --output reports/keywords.csv

# Step 4: Content planning
/content-calendar --duration 3months --niche "your niche" --output reports/calendar.csv

# Step 5: Create content briefs
/content-brief "keyword 1" --analyze-top 10 --output briefs/brief-1.md
/content-brief "keyword 2" --analyze-top 10 --output briefs/brief-2.md

# Step 6: Link building
/link-prospecting "topic" --min-dr 30 --include-templates --output outreach/prospects.csv

# Step 7: Monitor results
/serp-monitor --daily --keywords "kw1,kw2,kw3" --alert-threshold 5
```

---

**For more information**: See the [project repository](https://github.com/FabledPackerRedeem/r05-jqueryscript-awesome-claude-code-seo)
