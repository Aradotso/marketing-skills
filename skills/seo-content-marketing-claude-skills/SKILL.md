---
name: seo-content-marketing-claude-skills
description: SEO and content marketing automation suite with keyword research, technical audits, SERP analysis, and content strategy workflows for Claude AI
triggers:
  - "help me with SEO keyword research"
  - "run a technical SEO audit"
  - "analyze SERP competition"
  - "create an SEO content brief"
  - "audit my site content"
  - "build a content calendar"
  - "find backlink opportunities"
  - "optimize page speed for SEO"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Specialized Claude AI skill suite for SEO and content marketing automation. Derived from `alirezarezvani/claude-skills`, this collection provides 10 structured commands and 5 multi-step workflows for keyword research, technical SEO audits, content strategy, competitor analysis, and link building—all with consistent progress tracking and actionable output.

## What This Project Does

- **Keyword Research**: Deep clustering, opportunity scoring, SERP intent mapping
- **Technical SEO**: Crawl budget, Core Web Vitals, schema markup, indexability audits
- **Content Audits**: Quality scoring, duplication detection, cannibalization reports
- **Competitor Analysis**: Backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy**: AI-generated briefs, editorial calendars, refresh workflows
- **Link Building**: Prospect lists, outreach templates, authority-building campaigns

All commands return structured output with progress panels, prioritized findings, and time-boxed action plans.

## Installation

```bash
# Clone the repository
git clone https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo.git

# Copy to Claude skills directory
mkdir -p ~/.claude/skills
cp -r r02-alirezarezvani-claude-skills-seo ~/.claude/skills/seo-content-marketing

# Register in Claude Code session
# In Claude Code, run:
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

Alternative installation via direct copy:

```bash
# If you have the skill files in current directory
cp -r . ~/.claude/skills/seo-content-marketing/
```

## Core Commands

### `/keyword-research`

Performs keyword clustering, opportunity scoring, and SERP intent analysis.

```bash
# Basic usage
/keyword-research example.com

# With specific seed keywords
/keyword-research example.com --seeds "ai tools, automation software, workflow automation"

# With competitive analysis
/keyword-research example.com --competitors competitor1.com,competitor2.com

# Export to CSV
/keyword-research example.com --output csv --file keywords.csv
```

**Expected Output Structure:**
```
┌──────────────────────┬────────┬────────┬─────────┬──────────┐
│ Keyword              │ Volume │ Diff   │ Intent  │ Opportunity │
├──────────────────────┼────────┼────────┼─────────┼──────────┤
│ ai workflow tools    │  8 100 │     42 │ Trans   │  🟢 High │
│ automation software  │ 33 100 │     68 │ Comm    │  🟡 Med  │
│ workflow builder     │  2 900 │     38 │ Info    │  🟢 High │
└──────────────────────┴────────┴────────┴─────────┴──────────┘
```

### `/content-audit`

Full-site content quality assessment with duplication and cannibalization detection.

```bash
# Full site audit
/content-audit --scope full

# Specific URL pattern
/content-audit --pattern "/blog/*"

# With export
/content-audit --scope full --output md --file audit-report.md

# Focus on specific issues
/content-audit --check duplicates,cannibalization,thin-content
```

**Configuration Options:**
- `--scope`: `full`, `sample`, `pattern`
- `--min-words`: Minimum word count threshold (default: 300)
- `--similarity-threshold`: Duplicate detection threshold (default: 0.85)
- `--output`: `md`, `csv`, `json`

### `/technical-seo`

Comprehensive technical SEO audit covering crawl budget, performance, and indexability.

```bash
# Basic technical audit
/technical-seo example.com

# Deep crawl with rendering
/technical-seo example.com --depth full --render-js

# Focus on specific areas
/technical-seo example.com --checks core-web-vitals,schema,indexability

# Export findings
/technical-seo example.com --output json --file technical-audit.json
```

**Audit Coverage:**
- Crawl budget and bot accessibility
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Canonical tags and hreflang
- XML sitemap validation
- robots.txt analysis
- Mobile-friendliness

### `/competitor-gap`

Backlink gap, topic gap, and featured snippet opportunity analysis.

```bash
# Compare against competitors
/competitor-gap example.com --competitors competitor1.com,competitor2.com

# Focus on backlink gaps
/competitor-gap example.com --competitors competitor1.com --type backlinks

# Topic gap analysis
/competitor-gap example.com --competitors competitor1.com --type topics

# Featured snippet opportunities
/competitor-gap example.com --type snippets --serp-depth 20
```

### `/content-brief`

Generate SEO-optimized content briefs with outlines, NLP terms, and word count targets.

```bash
# Create brief from keyword
/content-brief "ai workflow automation"

# With specific competitors
/content-brief "ai workflow automation" --analyze competitor1.com,competitor2.com

# Custom target word count
/content-brief "ai workflow automation" --target-words 2500

# Export brief
/content-brief "ai workflow automation" --output md --file brief-ai-workflow.md
```

**Brief Includes:**
- Primary and secondary keywords
- Search intent analysis
- Recommended word count
- Outline with H2/H3 structure
- NLP terms and entities
- SERP feature opportunities
- Internal linking suggestions

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization recommendations.

```bash
# Monitor keywords
/serp-monitor --keywords "ai tools,automation software,workflow builder"

# Track specific URL
/serp-monitor --url example.com/product --keywords "ai tools"

# Set alert thresholds
/serp-monitor --keywords "ai tools" --alert-drop 3 --alert-volatility 20

# Historical comparison
/serp-monitor --keywords "ai tools" --compare 7d,30d
```

### `/link-prospecting`

Generate quality backlink prospect lists with DA/DR filters and outreach templates.

```bash
# Find prospects for niche
/link-prospecting --niche "marketing automation" --min-da 30

# Competitor backlink analysis
/link-prospecting --competitors competitor1.com,competitor2.com --min-da 40

# Specific link types
/link-prospecting --niche "saas tools" --types guest-post,resource-page,roundup

# Export with outreach templates
/link-prospecting --niche "ai tools" --output csv --include-templates
```

**Prospect Filters:**
- Domain Authority (DA)
- Domain Rating (DR)
- Traffic estimate
- Link type (guest post, resource, directory, etc.)
- Relevance score

### `/page-speed-seo`

Page speed analysis with SEO impact mapping for Core Web Vitals.

```bash
# Analyze page speed
/page-speed-seo example.com/page

# Detailed diagnostics
/page-speed-seo example.com/page --include render-blocking,lcp,cls,fid

# Mobile focus
/page-speed-seo example.com/page --device mobile

# Comparative analysis
/page-speed-seo example.com/page --compare-to competitor.com/page
```

**Metrics Analyzed:**
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)
- Render-blocking resources
- SEO ranking impact score

### `/local-seo`

NAP consistency, Google Business Profile optimization, and local citation audit.

```bash
# Full local SEO audit
/local-seo "Business Name" --location "New York, NY"

# NAP consistency check
/local-seo "Business Name" --check nap-consistency

# Citation audit
/local-seo "Business Name" --check citations --min-authority 40

# GBP optimization
/local-seo "Business Name" --check gbp-profile
```

### `/content-calendar`

Data-driven editorial calendar based on search demand and seasonality.

```bash
# Generate calendar for topic
/content-calendar --topic "marketing automation" --months 3

# With keyword research integration
/content-calendar --keywords keywords.csv --months 6

# Include seasonal trends
/content-calendar --topic "email marketing" --months 12 --include-seasonality

# Export calendar
/content-calendar --topic "seo tools" --months 3 --output csv --file calendar.csv
```

## Multi-Step Workflows

### `full-seo-sprint`

Complete 12-step SEO sprint from audit to execution.

```bash
# Run full sprint
/workflows:full-seo-sprint example.com --scope full

# With custom timeline
/workflows:full-seo-sprint example.com --scope full --duration 30d

# Resume from step
/workflows:full-seo-sprint example.com --resume-from step-5
```

**Sprint Steps:**
1. Technical SEO audit
2. Content audit
3. Keyword research
4. Competitor gap analysis
5. Priority mapping
6. Quick wins identification
7. Content brief generation
8. Technical fixes
9. On-page optimization
10. Link building strategy
11. Implementation tracking
12. Results measurement

### `launch-seo`

Pre-launch SEO checklist and validation.

```bash
# Pre-launch audit
/workflows:launch-seo example.com --type pre-launch

# Post-migration validation
/workflows:launch-seo example.com --type post-migration --old-domain old-site.com
```

**Checklist Includes:**
- Canonical tag validation
- Hreflang configuration
- XML sitemap generation
- robots.txt validation
- Redirect mapping
- Schema markup
- Core Web Vitals check

### `content-refresh`

Identify and refresh underperforming pages.

```bash
# Find refresh opportunities
/workflows:content-refresh --min-age 180d --rank-drop 5

# Refresh specific URLs
/workflows:content-refresh --urls urls.txt

# Auto-generate briefs
/workflows:content-refresh --min-age 90d --generate-briefs
```

### `authority-building`

End-to-end digital PR and link-building campaign.

```bash
# Launch link building campaign
/workflows:authority-building --niche "marketing tools" --target-links 50

# With content creation
/workflows:authority-building --niche "ai tools" --create-assets

# Track campaign
/workflows:authority-building --campaign campaign-id --report weekly
```

### `ai-content-pipeline`

Automated content creation from keyword to publish.

```bash
# Full pipeline
/workflows:ai-content-pipeline --keywords "ai automation,workflow tools" --count 10

# Custom workflow
/workflows:ai-content-pipeline --keywords keywords.csv --steps brief,draft,optimize

# With approval gates
/workflows:ai-content-pipeline --keywords keywords.csv --require-approval draft,optimize
```

**Pipeline Steps:**
1. Keyword selection
2. Brief generation
3. Content drafting
4. SEO optimization
5. Internal linking
6. Meta data creation
7. Image suggestions
8. Final review
9. Publishing

## Configuration

Create a configuration file at `~/.claude/skills/seo-content-marketing/config.yml`:

```yaml
# API Keys (use environment variables)
serp_api_key: ${SERP_API_KEY}
ahrefs_api_key: ${AHREFS_API_KEY}
semrush_api_key: ${SEMRUSH_API_KEY}

# Default Settings
defaults:
  min_domain_authority: 30
  keyword_volume_threshold: 100
  content_min_words: 300
  audit_depth: medium
  
# Output Preferences
output:
  format: markdown
  include_visuals: true
  show_progress: true
  
# Rate Limits
rate_limits:
  serp_api: 100  # requests per hour
  ahrefs_api: 500  # requests per day
  
# Thresholds
thresholds:
  duplicate_similarity: 0.85
  keyword_difficulty: 70
  page_speed_score: 90
  core_web_vitals: "good"
```

Environment variable setup:

```bash
# Add to ~/.bashrc or ~/.zshrc
export SERP_API_KEY="your_serp_api_key"
export AHREFS_API_KEY="your_ahrefs_key"
export SEMRUSH_API_KEY="your_semrush_key"
export GOOGLE_SEARCH_CONSOLE_KEY="path/to/credentials.json"
```

## Common Patterns

### Pattern 1: Monthly SEO Health Check

```bash
# Run comprehensive monthly audit
/technical-seo example.com --output json --file tech-audit-$(date +%Y%m).json
/content-audit --scope full --output md --file content-audit-$(date +%Y%m).md
/serp-monitor --keywords keywords.txt --compare 30d
```

### Pattern 2: New Content Campaign

```bash
# Research and plan
/keyword-research example.com --seeds "topic" --output csv --file keywords.csv
/content-calendar --keywords keywords.csv --months 3 --output csv --file calendar.csv

# Create briefs
/content-brief "primary keyword" --output md --file brief-001.md

# Monitor results
/serp-monitor --url example.com/new-article --keywords "primary keyword"
```

### Pattern 3: Competitor Takedown

```bash
# Analyze competitor
/competitor-gap example.com --competitors competitor.com --type all

# Find link opportunities
/link-prospecting --competitors competitor.com --min-da 40 --output csv

# Create better content
/content-brief "target keyword" --analyze competitor.com --target-words 3000
```

### Pattern 4: Technical SEO Fix Sprint

```bash
# Identify issues
/technical-seo example.com --depth full --output json --file issues.json

# Focus on speed
/page-speed-seo example.com --include render-blocking,lcp,cls

# Validate fixes
/workflows:launch-seo example.com --type validation
```

### Pattern 5: Content Refresh Campaign

```bash
# Find opportunities
/workflows:content-refresh --min-age 180d --rank-drop 5 --generate-briefs

# Re-optimize
/content-audit --pattern "/blog/*" --check thin-content,duplicates

# Track recovery
/serp-monitor --urls refreshed-urls.txt --compare 7d,30d
```

## Structured Output Format

All commands follow this consistent structure:

```
╔══════════════════════════════════════════════════╗
║  [Command Name]  —  [Target]                     ║
╠══════════════════════════════════════════════════╣
║  [Progress bars with percentages]                ║
╚══════════════════════════════════════════════════╝

┌─────────────────────┬──────────┬──────────┬──────────┐
│ [Metric tables with current/target/status]       │
└─────────────────────┴──────────┴──────────┴──────────┘

🔴 Critical Issues (fix immediately)
  • Issue 1 [estimated time: 2h]
  • Issue 2 [estimated time: 4h]

🟡 Opportunities (medium priority)
  • Opportunity 1 [estimated impact: +15% traffic]
  • Opportunity 2 [estimated impact: +10% rankings]

✅ Strengths (maintain/expand)
  • Strength 1
  • Strength 2

📋 Next Steps
  1. [Recommended next command]
  2. [Alternative action]
```

## Troubleshooting

### API Rate Limits

```bash
# Check current rate limit status
/status --api-limits

# Reduce request frequency
/keyword-research example.com --rate-limit 10  # 10 requests per minute
```

### Large Site Audits Timeout

```bash
# Use sampling instead of full crawl
/content-audit --scope sample --sample-size 500

# Split audit by section
/content-audit --pattern "/blog/*"
/content-audit --pattern "/products/*"
```

### Missing Dependencies

```bash
# Verify skill installation
ls -la ~/.claude/skills/seo-content-marketing/

# Re-register skill
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Environment Variables Not Loading

```bash
# Verify environment variables
echo $SERP_API_KEY

# Reload shell configuration
source ~/.bashrc  # or ~/.zshrc

# Test API connection
/test-connection --api serp
```

### Slow Performance

```bash
# Use cache for repeated queries
/keyword-research example.com --use-cache

# Reduce audit depth
/technical-seo example.com --depth shallow

# Limit concurrent requests
/competitor-gap example.com --competitors comp.com --max-concurrent 5
```

### Export Failures

```bash
# Ensure output directory exists
mkdir -p reports/

# Use absolute paths
/content-audit --output md --file ~/reports/audit-$(date +%Y%m%d).md

# Check disk space
df -h ~
```

## Integration with Other Tools

### Google Search Console

```bash
# Import GSC data
/import-gsc --property example.com --date-range 90d

# Combine with keyword research
/keyword-research example.com --include-gsc-data
```

### Analytics Platforms

```bash
# Import GA4 data for content audit
/content-audit --include-analytics --source ga4

# Traffic-weighted priority
/keyword-research example.com --weight-by-traffic
```

### CMS Integration

```bash
# Export for WordPress
/content-calendar --months 3 --format wordpress-csv

# Shopify product optimization
/technical-seo example.com --platform shopify
```

## Best Practices

1. **Start with technical foundation**: Run `/technical-seo` before content work
2. **Use workflows for complex projects**: Workflows ensure nothing is missed
3. **Export all reports**: Always use `--output` flags for documentation
4. **Set up monitoring**: Use `/serp-monitor` to track progress
5. **Batch similar tasks**: Group keyword research, audits, and briefs together
6. **Review competitor data regularly**: Run `/competitor-gap` monthly
7. **Maintain content calendar**: Update `/content-calendar` quarterly
8. **Track fixes**: Document all changes for future reference

---

**License**: MIT  
**Source**: https://github.com/AgentTestingClamp/r02-alirezarezvani-claude-skills-seo  
**Parent Project**: https://github.com/alirezarezvani/claude-skills
