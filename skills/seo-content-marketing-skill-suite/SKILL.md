---
name: seo-content-marketing-skill-suite
description: SEO and content marketing workflow automation with keyword research, content audits, technical SEO analysis, and multi-step optimization campaigns
triggers:
  - analyze SEO performance for this site
  - run a content audit and find optimization opportunities
  - help me do keyword research for this topic
  - check technical SEO issues on this domain
  - create an SEO content brief for this keyword
  - build a content calendar based on search demand
  - find backlink opportunities for this site
  - monitor SERP rankings and track volatility
---

# 📈 SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Skill Provides

This skill suite equips AI coding agents with SEO and content marketing expertise through **10 specialized commands** and **5 multi-step workflows**. It automates keyword research, content audits, SERP analysis, technical SEO diagnostics, and content strategy development.

Built on structured output principles with consistent UI patterns: progress tracking, severity-sorted findings, and actionable next steps.

---

## Installation

### Clone and Register

```bash
# Clone to Claude Code skills directory
mkdir -p ~/.claude/skills
cp -r . ~/.claude/skills/seo-content-marketing-skill-suite/

# Verify installation
ls -la ~/.claude/skills/seo-content-marketing-skill-suite/
```

### Register in Claude Code Session

In any Claude Code session:

```bash
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md
```

The skill is now active and available via slash commands.

---

## Core Commands

### `/keyword-research`

**Purpose:** Deep keyword clustering with opportunity scoring and SERP intent mapping.

**Syntax:**
```bash
/keyword-research <target_keyword> [--scope wide|narrow] [--output md|json]
```

**Example:**
```bash
/keyword-research "content marketing tools" --scope wide --output md
```

**What It Does:**
1. Queries search demand APIs (Google Keyword Planner, SEMrush, Ahrefs)
2. Clusters keywords by semantic similarity
3. Maps SERP intent (informational, commercial, transactional)
4. Scores keyword difficulty vs. opportunity
5. Outputs prioritized keyword targets

**Output Structure:**
```
╔══════════════════════════════════════════════════╗
║  Keyword Research  —  "content marketing tools"  ║
╠══════════════════════════════════════════════════╣
║  Fetching seed keywords …  [██████████] 100% ✓   ║
║  Clustering semantics …    [██████████] 100% ✓   ║
║  SERP intent mapping …     [██████████] 100% ✓   ║
╚══════════════════════════════════════════════════╝

┌─────────────────────────┬────────┬────────┬────────┬──────────┐
│ Keyword                 │ Volume │ Diff.  │ Intent │ Priority │
├─────────────────────────┼────────┼────────┼────────┼──────────┤
│ content marketing tools │  8 100 │ 67/100 │ Comm.  │  🟢 High │
│ best content tools      │  4 400 │ 54/100 │ Comm.  │  🟢 High │
│ free content tools      │  2 900 │ 42/100 │ Info.  │  🟡 Med  │
│ content calendar tools  │  1 800 │ 38/100 │ Trans. │  🟢 High │
└─────────────────────────┴────────┴────────┴────────┴──────────┘

Next Steps:
  → /content-brief "content marketing tools"
  → /competitor-gap --keyword "content marketing tools"
```

---

### `/content-audit`

**Purpose:** Full-site content quality scoring, duplication detection, and cannibalization reports.

**Syntax:**
```bash
/content-audit [--scope full|sample] [--domain example.com] [--output md|csv]
```

**Example:**
```bash
/content-audit --scope full --domain mysite.com --output md
```

**What It Does:**
1. Crawls all indexable pages
2. Scores content quality (uniqueness, depth, readability)
3. Detects duplicate/thin content
4. Identifies keyword cannibalization
5. Flags missing metadata (title, description, h1)

**Output Structure:**
```
╔══════════════════════════════════════════════════╗
║  Content Audit  —  mysite.com                    ║
╠══════════════════════════════════════════════════╣
║  Crawling pages …       [████████░░]  80%  1 204 ║
║  Analyzing content …    [██████████] 100%  Done ✓║
╚══════════════════════════════════════════════════╝

Quality Distribution:
  🔴 Poor (0-40):     127 pages  (10.5%)
  🟠 Fair (41-60):    382 pages  (31.7%)
  🟡 Good (61-80):    514 pages  (42.7%)
  🟢 Excellent (81+): 181 pages  (15.0%)

Top Issues:
  🔴 Thin content (<300 words):        89 pages
  🔴 Missing meta description:         302 pages
  🟠 Keyword cannibalization:          47 clusters
  🟠 Duplicate title tags:             23 instances
  🟡 Low readability (>60 Flesch):    156 pages

Action Plan:
  1. Merge/redirect 47 cannibalizing pages (2-4 hours)
  2. Rewrite 89 thin content pages (8-12 hours)
  3. Add meta descriptions to 302 pages (3-5 hours)

Next Steps:
  → /content-refresh --filter "quality:poor"
  → /technical-seo --domain mysite.com
```

---

### `/technical-seo`

**Purpose:** Crawl budget, Core Web Vitals, schema markup, and indexability audits.

**Syntax:**
```bash
/technical-seo [--domain example.com] [--depth shallow|deep]
```

**Example:**
```bash
/technical-seo --domain mysite.com --depth deep
```

**What It Does:**
1. Analyzes robots.txt and XML sitemaps
2. Checks Core Web Vitals (LCP, FID, CLS)
3. Validates structured data (schema.org)
4. Identifies crawl errors and redirect chains
5. Reports mobile usability issues

**Output Structure:**
```
╔══════════════════════════════════════════════════╗
║  Technical SEO  —  mysite.com                    ║
╠══════════════════════════════════════════════════╣
║  Indexability check …    [██████████] 100% ✓     ║
║  Core Web Vitals …       [██████████] 100% ✓     ║
║  Schema validation …     [██████████] 100% ✓     ║
╚══════════════════════════════════════════════════╝

┌──────────────────────┬──────────┬──────────┬──────────┐
│ Metric               │ Current  │ Target   │ Status   │
├──────────────────────┼──────────┼──────────┼──────────┤
│ Indexable pages      │    1 180 │    1 204 │  ⚠ 98 %  │
│ Avg. LCP             │    2.1 s │   <2.5 s │  ✓ Good  │
│ Avg. FID             │     78 ms│  <100 ms │  ✓ Good  │
│ Avg. CLS             │    0.08  │   <0.1   │  ✓ Good  │
│ Valid schema markup  │      402 │    1 180 │  ✗ 34 %  │
│ Mobile-friendly      │    1 142 │    1 180 │  ⚠ 97 %  │
└──────────────────────┴──────────┴──────────┴──────────┘

Critical Issues:
  🔴 24 pages blocked by robots.txt but in sitemap
  🔴 87 redirect chains (>3 hops)
  🟠 778 pages missing schema markup
  🟡 38 pages with slow LCP (>2.5s)

Next Steps:
  → /page-speed-seo --filter "lcp:slow"
  → Review robots.txt and sitemap.xml inconsistencies
```

---

### `/competitor-gap`

**Purpose:** Backlink gap, topic gap, and featured-snippet opportunity analysis.

**Syntax:**
```bash
/competitor-gap --domain yourdomain.com --competitors domain1.com,domain2.com
```

**Example:**
```bash
/competitor-gap --domain mysite.com --competitors competitor1.com,competitor2.com
```

**What It Does:**
1. Identifies keywords competitors rank for but you don't
2. Finds backlinks pointing to competitors but not to you
3. Discovers featured snippet opportunities
4. Compares content coverage and depth
5. Scores opportunities by traffic potential

---

### `/content-brief`

**Purpose:** AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Syntax:**
```bash
/content-brief "<target_keyword>" [--competitors domain1.com,domain2.com]
```

**Example:**
```bash
/content-brief "email marketing automation" --competitors mailchimp.com,hubspot.com
```

**What It Does:**
1. Analyzes top 10 SERP results for target keyword
2. Extracts common NLP terms and entities
3. Generates content outline with H2/H3 structure
4. Sets target word count based on top performers
5. Includes internal linking recommendations

**Output Structure:**
```markdown
# Content Brief: "email marketing automation"

## Target Metrics
- **Primary Keyword:** email marketing automation
- **Search Volume:** 12,400/month
- **Keyword Difficulty:** 58/100
- **Target Word Count:** 2,400-2,800 words
- **Content Type:** Pillar page / Guide

## SERP Intent
Commercial Investigation + Informational

## Required NLP Terms (Top 30)
automation workflows, email sequences, drip campaigns, segmentation, 
personalization, A/B testing, trigger-based, behavioral targeting, 
lead nurturing, conversion funnel, marketing automation platform...

## Suggested Outline

### Introduction (200-250 words)
- Define email marketing automation
- Benefits and ROI statistics

### H2: What Is Email Marketing Automation? (300-400 words)
- Core concepts
- How it differs from broadcast emails

### H2: Key Features to Look For (500-600 words)
#### H3: Workflow Builder
#### H3: Segmentation & Personalization
#### H3: Analytics & Reporting

### H2: Top Email Automation Platforms (600-700 words)
[Compare 5-7 tools]

### H2: Best Practices (400-500 words)
...

## Internal Links
- /blog/email-segmentation-strategies
- /blog/conversion-funnel-optimization
- /products/email-marketing-tools

## Competitor Analysis
| URL | Word Count | Schema | Backlinks |
|-----|-----------|--------|-----------|
| mailchimp.com/... | 2,847 | ✓ | 342 |
| hubspot.com/...   | 3,124 | ✓ | 589 |

Next Steps:
  → Draft content following this outline
  → /content-calendar --include "email marketing automation"
```

---

### `/serp-monitor`

**Purpose:** Daily rank tracking with volatility alerts and CTR optimization tips.

**Syntax:**
```bash
/serp-monitor --keywords keyword1,keyword2 [--frequency daily|weekly]
```

---

### `/link-prospecting`

**Purpose:** Quality backlink prospect list with DA/DR filters and outreach templates.

**Syntax:**
```bash
/link-prospecting --topic "content marketing" --min-da 40
```

---

### `/page-speed-seo`

**Purpose:** Render-blocking resources, LCP, CLS, FID diagnosis mapped to ranking impact.

**Syntax:**
```bash
/page-speed-seo --url https://example.com/page [--device mobile|desktop]
```

---

### `/local-seo`

**Purpose:** NAP consistency, Google Business Profile optimization, and local citation audit.

**Syntax:**
```bash
/local-seo --business "Your Business Name" --location "City, State"
```

---

### `/content-calendar`

**Purpose:** Data-driven editorial calendar built from search demand and seasonality.

**Syntax:**
```bash
/content-calendar --months 3 --topics "content marketing,SEO"
```

**Example:**
```bash
/content-calendar --months 6 --topics "email marketing,marketing automation"
```

**What It Does:**
1. Analyzes search trends over time (Google Trends)
2. Identifies seasonal peaks in demand
3. Maps content opportunities to calendar dates
4. Suggests content types (guides, comparisons, case studies)
5. Balances evergreen vs. timely content

---

## Multi-Step Workflows

### `full-seo-sprint`

**Purpose:** Complete 12-step SEO sprint from audit to content plan to technical fixes.

**Usage:**
```bash
/workflows:full-seo-sprint --domain mysite.com --duration 2-weeks
```

**Steps:**
1. Technical SEO audit
2. Content audit
3. Keyword research (primary topics)
4. Competitor gap analysis
5. SERP monitoring setup
6. Prioritized fix list (technical)
7. Content refresh targets
8. New content briefs (top 5 opportunities)
9. Link prospecting
10. Schema markup implementation
11. Core Web Vitals optimization
12. Progress dashboard

---

### `launch-seo`

**Purpose:** Pre-launch SEO checklist with canonical, hreflang, and sitemap validation.

**Usage:**
```bash
/workflows:launch-seo --domain newsite.com --go-live-date 2026-06-01
```

**Checklist:**
- [ ] robots.txt configured
- [ ] XML sitemap generated and submitted
- [ ] Canonical tags implemented
- [ ] Hreflang tags (if multi-language)
- [ ] Schema markup on key pages
- [ ] 404 page customized
- [ ] Redirects from old URLs (if applicable)
- [ ] Page speed optimized
- [ ] Mobile-friendly verified
- [ ] Analytics and Search Console connected

---

### `content-refresh`

**Purpose:** Identify and refresh underperforming pages to recover lost rankings.

**Usage:**
```bash
/workflows:content-refresh --filter "traffic-drop:30d" --limit 20
```

**Process:**
1. Identify pages with traffic drops
2. Analyze current vs. previous rankings
3. Compare to updated SERP competition
4. Generate refresh recommendations (expand, update stats, add media)
5. Create prioritized refresh queue

---

### `authority-building`

**Purpose:** End-to-end digital PR and link-building campaign workflow.

**Usage:**
```bash
/workflows:authority-building --topic "marketing automation" --duration 3-months
```

**Campaign Steps:**
1. Identify linkable assets (studies, tools, guides)
2. Prospect high-quality link targets
3. Generate outreach templates
4. Track outreach progress
5. Monitor acquired backlinks
6. Measure domain authority growth

---

### `ai-content-pipeline`

**Purpose:** Keyword → brief → draft → optimize → publish automation pipeline.

**Usage:**
```bash
/workflows:ai-content-pipeline --topic "email marketing" --quantity 10
```

**Pipeline:**
1. **Keyword research:** Identify 10 content opportunities
2. **Brief generation:** Auto-generate content briefs
3. **Outline creation:** Structure each piece
4. **Draft (optional):** Generate AI-assisted first draft
5. **SEO optimization:** Add NLP terms, internal links, meta tags
6. **Quality check:** Readability, uniqueness, completeness
7. **Publish-ready package:** Export markdown/HTML + metadata

---

## Configuration

### Environment Variables

All API integrations require environment variables:

```bash
# Search APIs
export GOOGLE_KEYWORD_PLANNER_API_KEY=your_key_here
export SEMRUSH_API_KEY=your_key_here
export AHREFS_API_KEY=your_key_here

# Analytics
export GOOGLE_ANALYTICS_CREDENTIALS=/path/to/credentials.json
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=/path/to/credentials.json

# Page Speed
export PAGESPEED_INSIGHTS_API_KEY=your_key_here

# Optional: AI content generation
export OPENAI_API_KEY=your_key_here
```

### Config File

Create `~/.claude/skills/seo-content-marketing-skill-suite/config.yaml`:

```yaml
defaults:
  keyword_research:
    scope: wide
    output: md
  content_audit:
    scope: full
    min_quality_score: 60
  technical_seo:
    depth: deep
    check_mobile: true
  serp_monitor:
    frequency: daily

integrations:
  google_keyword_planner: true
  semrush: true
  ahrefs: false  # Set to true if API key available
  screaming_frog: false  # Local crawl tool

output:
  format: markdown
  save_reports: true
  report_dir: ./seo-reports/
```

---

## Common Patterns

### Pattern 1: New Site SEO Setup

```bash
# Step 1: Baseline audit
/technical-seo --domain newsite.com --depth deep

# Step 2: Keyword research
/keyword-research "primary topic" --scope wide

# Step 3: Content brief for top keyword
/content-brief "top keyword from research"

# Step 4: Editorial calendar
/content-calendar --months 3 --topics "topic1,topic2"

# Step 5: Pre-launch checklist
/workflows:launch-seo --domain newsite.com
```

---

### Pattern 2: Traffic Recovery

```bash
# Step 1: Identify drops
/content-audit --scope full --domain mysite.com

# Step 2: Competitor analysis
/competitor-gap --domain mysite.com --competitors comp1.com,comp2.com

# Step 3: Refresh workflow
/workflows:content-refresh --filter "traffic-drop:30d" --limit 20

# Step 4: Monitor rankings
/serp-monitor --keywords "key1,key2,key3" --frequency daily
```

---

### Pattern 3: Content-First Campaign

```bash
# Step 1: Keyword opportunities
/keyword-research "campaign topic" --scope wide

# Step 2: Automated pipeline
/workflows:ai-content-pipeline --topic "campaign topic" --quantity 10

# Step 3: Link building support
/link-prospecting --topic "campaign topic" --min-da 40

# Step 4: Monitor performance
/serp-monitor --keywords "campaign keywords"
```

---

## Troubleshooting

### Issue: API Rate Limits

**Symptom:** `429 Too Many Requests` errors

**Solution:**
```yaml
# In config.yaml, add rate limiting
rate_limits:
  google_keyword_planner: 100/hour
  semrush: 50/hour
  ahrefs: 30/hour
```

---

### Issue: Incomplete Crawls

**Symptom:** Content audit stops at 80%

**Solution:**
```bash
# Increase timeout and retry settings
/content-audit --domain mysite.com --timeout 600 --max-retries 3
```

---

### Issue: Missing Search Volume Data

**Symptom:** Keyword research returns `N/A` for search volume

**Solution:**
1. Verify `GOOGLE_KEYWORD_PLANNER_API_KEY` is set
2. Check API quota in Google Cloud Console
3. Fallback to alternative APIs:

```bash
# Use SEMrush as fallback
/keyword-research "topic" --api semrush
```

---

### Issue: Schema Validation Errors

**Symptom:** `/technical-seo` reports invalid schema markup

**Solution:**
```bash
# Run detailed schema validation
/technical-seo --domain mysite.com --validate-schema --verbose

# Cross-check with Google Rich Results Test
# https://search.google.com/test/rich-results
```

---

## Advanced Usage

### Custom Workflow Orchestration

Create custom workflow sequences:

```bash
# Define in custom-workflows.yaml
workflows:
  monthly_seo_review:
    steps:
      - command: /technical-seo
        args: --domain $DOMAIN --depth shallow
      - command: /content-audit
        args: --scope sample --size 100
      - command: /serp-monitor
        args: --keywords $TRACKED_KEYWORDS --report monthly
      - command: /competitor-gap
        args: --domain $DOMAIN --competitors $COMPETITORS

# Execute
/workflows:monthly_seo_review --domain mysite.com
```

---

### Batch Processing

Process multiple domains or keywords:

```bash
# Batch keyword research
cat keywords.txt | xargs -I {} /keyword-research "{}" --output json > results.json

# Batch page speed analysis
cat urls.txt | xargs -I {} /page-speed-seo --url "{}" --device mobile
```

---

## Output Formats

### Markdown Reports

Default output includes structured markdown:

```markdown
# SEO Audit Report — mysite.com
Date: 2026-05-11

## Summary
- Total pages crawled: 1,204
- Indexable pages: 1,180 (98%)
- Critical issues: 24
- Warnings: 127

## Critical Issues
1. **Robots.txt blocking 24 pages in sitemap**
   - Impact: Pages won't be indexed
   - Fix: Remove disallow rules or update sitemap
   
...
```

---

### JSON Export

For programmatic processing:

```bash
/content-audit --domain mysite.com --output json > audit.json
```

```json
{
  "domain": "mysite.com",
  "timestamp": "2026-05-11T14:30:00Z",
  "pages_crawled": 1204,
  "pages_indexable": 1180,
  "issues": {
    "critical": [
      {
        "type": "robots_sitemap_conflict",
        "count": 24,
        "severity": "critical",
        "urls": ["...", "..."]
      }
    ],
    "warnings": [...]
  },
  "quality_distribution": {
    "poor": 127,
    "fair": 382,
    "good": 514,
    "excellent": 181
  }
}
```

---

### CSV for Spreadsheet Analysis

```bash
/keyword-research "topic" --output csv > keywords.csv
```

Opens in Excel/Google Sheets with columns:
```
Keyword, Volume, Difficulty, Intent, CPC, Priority
```

---

## Integration with Development Workflows

### Pre-Commit Hook for Metadata Validation

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Validate meta tags in changed HTML files
git diff --cached --name-only | grep "\.html$" | while read file; do
  if ! grep -q "<meta name=\"description\"" "$file"; then
    echo "❌ Missing meta description: $file"
    exit 1
  fi
done
```

---

### CI/CD Pipeline Integration

```yaml
# .github/workflows/seo-check.yml
name: SEO Validation

on: [push, pull_request]

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install skill suite
        run: |
          cp -r . ~/.claude/skills/seo-content-marketing-skill-suite/
      
      - name: Run technical SEO check
        run: |
          /technical-seo --domain ${{ secrets.STAGING_URL }} --depth shallow
        env:
          PAGESPEED_INSIGHTS_API_KEY: ${{ secrets.PAGESPEED_API_KEY }}
      
      - name: Fail on critical issues
        run: |
          # Parse output and fail if critical issues > 0
          if [ "$CRITICAL_ISSUES" -gt 0 ]; then exit 1; fi
```

---

## Best Practices

1. **Always verify API quotas** before running large-scale audits
2. **Use `--scope sample` for initial tests** to avoid rate limits
3. **Export reports regularly** for historical comparison
4. **Combine workflows** for end-to-end campaigns
5. **Monitor SERP volatility** before making large-scale changes
6. **Test schema markup** with Google Rich Results Test before deployment
7. **Track Core Web Vitals** as part of every technical audit

---

## Support & Documentation

- **Source Repository:** [MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo](https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo)
- **Derived From:** [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- **License:** MIT
- **Issues:** Report bugs or request features via GitHub Issues

---

## Version

**Skill Suite Version:** 1.0.0  
**Last Updated:** 2026-05-11  
**Compatibility:** Claude Code, Cursor, GitHub Copilot, Codex
