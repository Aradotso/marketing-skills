---
name: seo-content-marketing-claude-skills
description: SEO & content marketing command suite with keyword research, audits, SERP analysis, and content strategy workflows
triggers:
  - analyze keywords for SEO
  - run a content audit
  - check technical SEO issues
  - create an SEO content brief
  - find competitor content gaps
  - build a content calendar
  - analyze page speed for SEO
  - do local SEO optimization
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A structured command and workflow suite for SEO and content marketing tasks, derived from alirezarezvani/claude-skills. Provides 10 specialized commands and 5 multi-step workflows with consistent structured output UI for keyword research, content audits, SERP analysis, technical SEO, and content strategy.

## What This Project Does

This skill suite provides AI agents with SEO and content marketing expertise through:

- **10 specialized commands** for keyword research, audits, SERP monitoring, link prospecting, and more
- **5 multi-step workflows** for complete SEO sprints, launches, content refresh, and AI content pipelines
- **Consistent UI patterns** with progress tracking, findings tables, prioritized actions, and next steps
- **Structured output** that always shows current state and actionable recommendations

All commands follow a 5-step interaction pattern: scope confirmation → live analysis → findings table → action plan → next steps.

## Installation

### Method 1: Clone to Claude Skills Directory

```bash
# Clone the repository
git clone https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo.git

# Copy to Claude skills directory
cp -r r02-alirezarezvani-claude-skills-seo ~/.claude/skills/seo-content-marketing/
```

### Method 2: Register in Claude Code Session

```bash
# In a Claude Code session, read the skill file
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Method 3: Direct Integration

Copy the skill commands directly into your Claude Code configuration or project workspace.

## Core Commands

### Keyword Research

Deep keyword clustering with opportunity scoring and SERP intent mapping.

```bash
# Basic keyword research
/keyword-research "project management software"

# With specific options
/keyword-research "email marketing" --intent transactional --volume >1000

# Export to file
/keyword-research "content marketing" --output csv --file keywords.csv
```

**Output structure:**
- Keyword clusters with search volume and difficulty
- SERP intent classification (informational, navigational, transactional)
- Opportunity score (competition vs. volume)
- Related long-tail variations
- Prioritized action plan

### Content Audit

Full-site content quality assessment with duplication and cannibalization detection.

```bash
# Full site audit
/content-audit --scope full --domain example.com

# Specific section
/content-audit --scope /blog --domain example.com

# With output format
/content-audit --scope full --output md --file audit-report.md
```

**Output includes:**
- Content quality scores per page
- Duplicate content detection
- Keyword cannibalization matrix
- Thin content identification
- Prioritized optimization opportunities

### Technical SEO Audit

Comprehensive technical SEO analysis including crawl budget, Core Web Vitals, schema markup.

```bash
# Complete technical audit
/technical-seo --domain example.com

# Focus on specific areas
/technical-seo --domain example.com --focus core-web-vitals,schema

# With recommendations
/technical-seo --domain example.com --recommendations detailed
```

**Analyzes:**
- Crawl budget efficiency
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Indexability issues
- Robots.txt and sitemap health
- Mobile-friendliness

### Competitor Gap Analysis

Backlink, topic, and featured snippet opportunity identification.

```bash
# Analyze competitor gaps
/competitor-gap --domain example.com --competitors competitor1.com,competitor2.com

# Focus on backlinks
/competitor-gap --domain example.com --competitors competitor1.com --focus backlinks

# Topic gap only
/competitor-gap --domain example.com --competitors competitor1.com --focus topics
```

**Provides:**
- Backlink gap analysis with DR/DA scores
- Topic coverage gaps
- Featured snippet opportunities
- Content format gaps (video, infographics, tools)

### SEO Content Brief Generation

AI-generated content briefs with outlines, NLP terms, and word count targets.

```bash
# Generate content brief
/content-brief "how to do keyword research"

# With specific parameters
/content-brief "email marketing best practices" --words 2000-2500 --tone professional

# Include competitor analysis
/content-brief "content strategy guide" --competitors 5 --nlp-terms true
```

**Brief includes:**
- Target keyword and variations
- Recommended word count based on SERP analysis
- H2/H3 outline structure
- NLP terms and entities to include
- Competitor content analysis
- Internal linking suggestions

### SERP Monitoring

Rank tracking with volatility alerts and CTR optimization recommendations.

```bash
# Monitor keyword rankings
/serp-monitor --keywords keywords.txt --domain example.com

# Daily report
/serp-monitor --keywords keywords.txt --frequency daily --alerts true

# Specific search engine and location
/serp-monitor --keywords keywords.txt --engine google --location "New York, US"
```

**Tracking features:**
- Daily rank positions
- Volatility alerts (unusual fluctuations)
- CTR optimization opportunities
- Featured snippet tracking
- SERP feature presence (PAA, videos, local pack)

### Link Prospecting

Quality backlink prospect identification with outreach templates.

```bash
# Find link prospects
/link-prospecting --topic "content marketing" --min-da 30

# With filters
/link-prospecting --topic "SEO tools" --min-da 40 --min-dr 35 --type guest-post

# Generate outreach templates
/link-prospecting --topic "digital marketing" --templates true
```

**Output:**
- Prospect URLs with DA/DR scores
- Contact information discovery
- Relevance scoring
- Outreach email templates
- Follow-up sequence recommendations

### Page Speed SEO Analysis

Performance diagnostics mapped to ranking impact.

```bash
# Analyze page speed
/page-speed-seo --url https://example.com/page

# Full site analysis
/page-speed-seo --domain example.com --pages 50

# Mobile focus
/page-speed-seo --url https://example.com/page --device mobile
```

**Diagnoses:**
- Render-blocking resources
- Largest Contentful Paint (LCP)
- Cumulative Layout Shift (CLS)
- First Input Delay (FID)
- SEO ranking impact assessment
- Prioritized fixes with implementation steps

### Local SEO Audit

NAP consistency, Google Business Profile optimization, and citation audit.

```bash
# Local SEO audit
/local-seo --business "Example Business" --location "New York, NY"

# Focus areas
/local-seo --business "Example Business" --focus nap,citations

# With GMB optimization
/local-seo --business "Example Business" --gmb-profile true
```

**Checks:**
- NAP (Name, Address, Phone) consistency across directories
- Google Business Profile completeness and optimization
- Local citation quality and quantity
- Review management status
- Local schema markup
- Local pack ranking factors

### Content Calendar Generation

Data-driven editorial calendar based on search demand and seasonality.

```bash
# Generate content calendar
/content-calendar --topics topics.txt --months 3

# With keyword research integration
/content-calendar --topics topics.txt --months 6 --keyword-research true

# Export calendar
/content-calendar --topics topics.txt --months 3 --output google-calendar
```

**Calendar includes:**
- Topic scheduling based on search trends
- Seasonal opportunity identification
- Content format recommendations
- Target keyword assignments
- Production timeline estimates
- Resource allocation suggestions

## Multi-Step Workflows

### Full SEO Sprint

12-step comprehensive SEO process from audit to execution.

```bash
# Run full SEO sprint
/workflows:full-seo-sprint example.com --scope full

# With specific focus areas
/workflows:full-seo-sprint example.com --focus technical,content
```

**Workflow steps:**
1. Technical audit
2. Content audit
3. Keyword research and mapping
4. Competitor analysis
5. On-page optimization plan
6. Content gap identification
7. Internal linking strategy
8. Schema markup recommendations
9. Page speed optimization
10. Backlink strategy
11. Content calendar creation
12. Measurement and KPI setup

### Launch SEO

Pre-launch SEO checklist and validation.

```bash
# Pre-launch SEO check
/workflows:launch-seo example.com

# With specific validations
/workflows:launch-seo example.com --checks canonical,hreflang,sitemap
```

**Validates:**
- Canonical tag implementation
- Hreflang configuration (multi-language sites)
- XML sitemap generation and submission
- Robots.txt configuration
- 301 redirect mapping
- Schema markup implementation
- Google Analytics and Search Console setup
- Mobile responsiveness
- Page speed baselines

### Content Refresh

Identify and optimize underperforming content to recover rankings.

```bash
# Content refresh workflow
/workflows:content-refresh example.com

# Target specific ranking drop
/workflows:content-refresh example.com --ranking-drop >10
```

**Process:**
1. Identify ranking decline pages
2. Analyze current vs. historical performance
3. Assess SERP changes and intent shifts
4. Generate refresh recommendations
5. Create updated content briefs
6. Prioritize by traffic recovery potential
7. Set measurement benchmarks

### Authority Building

End-to-end digital PR and link-building campaign.

```bash
# Authority building campaign
/workflows:authority-building example.com --niche "content marketing"

# With outreach automation
/workflows:authority-building example.com --niche "SEO" --outreach true
```

**Campaign steps:**
1. Link-worthy asset identification
2. Content upgrade recommendations
3. Linkable asset creation brief
4. Prospect list building
5. Outreach template generation
6. Follow-up sequence setup
7. Relationship management plan
8. Link acquisition tracking

### AI Content Pipeline

End-to-end automated content creation workflow.

```bash
# AI content pipeline
/workflows:ai-content-pipeline --keywords keywords.txt

# With quality gates
/workflows:ai-content-pipeline --keywords keywords.txt --review-gates true
```

**Pipeline stages:**
1. Keyword selection and clustering
2. Content brief generation
3. Outline creation
4. AI draft generation
5. SEO optimization (NLP terms, headings)
6. Fact-checking and quality review
7. Internal linking insertion
8. Meta data creation
9. Image optimization recommendations
10. Publishing preparation

## Configuration

### Environment Variables

Commands may use these environment variables for API integrations:

```bash
# SEO tool APIs
export SEMRUSH_API_KEY="your-semrush-key"
export AHREFS_API_KEY="your-ahrefs-key"
export MOZ_API_KEY="your-moz-key"

# Search Console and Analytics
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS="path/to/credentials.json"
export GOOGLE_ANALYTICS_CREDENTIALS="path/to/ga-credentials.json"

# Page speed APIs
export GOOGLE_PAGESPEED_API_KEY="your-pagespeed-key"

# Crawling
export SCREAMING_FROG_LICENSE="your-license-key"
```

### Command Options

Global options available for most commands:

```bash
--output [md|csv|json|html]  # Output format
--file <path>                 # Export to file
--verbose                     # Detailed logging
--domain <domain>             # Target domain
--scope [full|section|page]   # Analysis scope
```

## Common Patterns

### Integration with Existing Workflows

```bash
# Morning SEO check routine
/serp-monitor --keywords priority-keywords.txt --domain example.com --frequency daily
/technical-seo --domain example.com --focus core-web-vitals --quick true

# Weekly content planning
/keyword-research --input content-ideas.txt --output csv --file weekly-keywords.csv
/content-calendar --keywords weekly-keywords.csv --months 1

# Monthly comprehensive audit
/workflows:full-seo-sprint example.com --scope full --output html --file monthly-report.html
```

### Combining Commands

```bash
# Research → Brief → Calendar pipeline
/keyword-research "email marketing" --output json --file keywords.json
/content-brief --keywords keywords.json --output md --file briefs/
/content-calendar --briefs briefs/ --months 3 --output google-calendar

# Technical fix → Monitor workflow
/technical-seo --domain example.com --recommendations detailed
# (implement fixes)
/page-speed-seo --domain example.com --baseline true
# (wait 7 days)
/page-speed-seo --domain example.com --compare baseline
```

### Automation Scripts

```bash
#!/bin/bash
# Daily SEO monitoring script

DOMAIN="example.com"
DATE=$(date +%Y-%m-%d)

# Check rankings
/serp-monitor --keywords keywords.txt --domain $DOMAIN --output csv --file "reports/rankings-$DATE.csv"

# Check page speed
/page-speed-seo --domain $DOMAIN --pages top-10.txt --output json --file "reports/speed-$DATE.json"

# Alert on issues
/technical-seo --domain $DOMAIN --quick true --alerts-only true
```

## Output Format Examples

### Progress Display

All commands show real-time progress:

```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "content marketing"        ║
╠══════════════════════════════════════════════════╣
║  Fetching volume …     [██████████] 100%  Done ✓ ║
║  Clustering …          [████████░░]  80%  324/405║
║  Scoring difficulty …  [███░░░░░░░]  30%  122/405║
╚══════════════════════════════════════════════════╝
```

### Findings Table

Results presented in prioritized tables:

```
┌──────────────────────────┬────────┬────────┬──────────┬────────┐
│ Keyword                  │ Volume │ Diff.  │ Intent   │ Score  │
├──────────────────────────┼────────┼────────┼──────────┼────────┤
│ content marketing tips   │ 🔴 22K │ 🟢 32  │ Info     │ ⭐⭐⭐⭐ │
│ what is content marketing│ 🟠 18K │ 🟡 45  │ Info     │ ⭐⭐⭐  │
│ content marketing tools  │ 🟡 12K │ 🔴 68  │ Trans    │ ⭐⭐    │
│ content marketing guide  │ 🟢  8K │ 🟢 28  │ Info     │ ⭐⭐⭐⭐ │
└──────────────────────────┴────────┴────────┴──────────┴────────┘
```

### Action Plan

Prioritized recommendations with time estimates:

```
═══════════════════════════════════════════════════
ACTION PLAN — Prioritized by Impact
═══════════════════════════════════════════════════

🔥 QUICK WINS (This Week)
  ☐ Target "content marketing tips" (Est: 2h)
  ☐ Optimize existing guide for "content marketing guide" (Est: 3h)
  ☐ Add FAQ schema to top 5 pages (Est: 1h)

⚡ MEDIUM-TERM (This Month)
  ☐ Create comprehensive guide for "what is content marketing" (Est: 8h)
  ☐ Build topic cluster around "content marketing strategy" (Est: 20h)
  ☐ Launch link building campaign (Est: 40h)

🎯 STRATEGIC (This Quarter)
  ☐ Develop content marketing tools/calculator (Est: 80h)
  ☐ Build authority with digital PR campaign (Est: 60h)
  ☐ Video content series for YouTube optimization (Est: 100h)
```

## Troubleshooting

### Command Not Found

If commands aren't recognized:

```bash
# Verify installation
ls ~/.claude/skills/seo-content-marketing/

# Re-read skill in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md

# Check skill registration
/skills list
```

### API Rate Limits

When hitting API limits:

```bash
# Use cache flag to reuse previous data
/keyword-research "topic" --cache true

# Reduce scope
/content-audit --scope /blog --pages 100

# Batch process with delays
/serp-monitor --keywords keywords.txt --batch-size 50 --delay 5
```

### Incomplete Results

If analysis seems incomplete:

```bash
# Increase timeout
/technical-seo --domain example.com --timeout 300

# Enable verbose logging
/content-audit --scope full --verbose true

# Check specific areas separately
/technical-seo --domain example.com --focus schema
/technical-seo --domain example.com --focus core-web-vitals
```

### Export Issues

When exports fail:

```bash
# Verify output directory exists
mkdir -p reports/

# Use absolute paths
/keyword-research "topic" --output csv --file /absolute/path/to/keywords.csv

# Try different format
/content-audit --output json # instead of csv or md
```

### Large Site Performance

For large sites (>10,000 pages):

```bash
# Use sampling
/content-audit --domain example.com --sample 1000

# Process in sections
/content-audit --scope /blog --domain example.com
/content-audit --scope /products --domain example.com

# Use quick mode
/technical-seo --domain example.com --quick true
```

## Best Practices

### Regular Monitoring Schedule

```bash
# Daily: Rankings and critical metrics
/serp-monitor --keywords priority.txt --domain example.com

# Weekly: Technical health check
/technical-seo --domain example.com --quick true

# Monthly: Comprehensive audit
/workflows:full-seo-sprint example.com --scope full

# Quarterly: Competitor analysis
/competitor-gap --domain example.com --competitors competitors.txt
```

### Content Production Workflow

```bash
# 1. Research phase
/keyword-research --input content-ideas.txt --output csv --file keywords.csv

# 2. Planning phase
/content-brief --keywords keywords.csv --output md --file briefs/
/content-calendar --briefs briefs/ --months 3

# 3. Production phase
/workflows:ai-content-pipeline --briefs briefs/ --review-gates true

# 4. Optimization phase
/content-audit --scope /new-content --domain example.com
```

### Link Building Campaign

```bash
# 1. Identify opportunities
/competitor-gap --domain example.com --focus backlinks

# 2. Find prospects
/link-prospecting --topic "your niche" --min-da 30 --templates true

# 3. Launch campaign
/workflows:authority-building example.com --niche "your niche"

# 4. Monitor progress
# (track manually or integrate with outreach tools)
```

## Integration Examples

### With CI/CD Pipelines

```yaml
# .github/workflows/seo-check.yml
name: SEO Health Check
on:
  schedule:
    - cron: '0 9 * * *'  # Daily at 9 AM
jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Technical SEO Audit
        run: |
          /technical-seo --domain ${{ secrets.DOMAIN }} --quick true --alerts-only true
      - name: Check Page Speed
        run: |
          /page-speed-seo --domain ${{ secrets.DOMAIN }} --pages top-10.txt --threshold 90
```

### With Content Management Systems

```javascript
// WordPress/Headless CMS integration example
const generateContentBrief = async (keyword) => {
  // Use command to generate brief
  const brief = await executeCommand(`/content-brief "${keyword}" --output json`);
  
  // Create draft post in CMS
  await cms.createDraft({
    title: brief.title,
    outline: brief.outline,
    keywords: brief.keywords,
    wordCount: brief.wordCount
  });
};
```

### With Analytics Platforms

```python
# Python script for automated reporting
import subprocess
import json

def daily_seo_report():
    # Run SERP monitoring
    result = subprocess.run(
        ['/serp-monitor', '--keywords', 'keywords.txt', '--output', 'json'],
        capture_output=True
    )
    
    data = json.loads(result.stdout)
    
    # Send to analytics platform
    analytics.track_rankings(data)
    
    # Alert on significant changes
    for keyword in data['keywords']:
        if abs(keyword['position_change']) > 5:
            alert.send(f"Ranking change: {keyword['term']} moved {keyword['position_change']} positions")
```

## Support and Resources

- **Original Project:** [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills)
- **This Adaptation:** [r02-alirezarezvani-claude-skills-seo](https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo)
- **Issues:** Report issues specific to SEO commands in this repository
- **License:** MIT
