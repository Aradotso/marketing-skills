---
name: seo-content-marketing-claude-skills
description: SEO & content marketing skill suite with keyword research, content audits, technical SEO analysis, and automated workflows
triggers:
  - "help me with SEO analysis"
  - "perform a content audit"
  - "analyze keyword opportunities"
  - "check technical SEO issues"
  - "create a content brief"
  - "analyze SERP rankings"
  - "build a content calendar"
  - "find backlink opportunities"
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive Claude skill suite for SEO and content marketing, providing 10 specialized commands and 5 multi-step workflows for keyword research, content audits, SERP analysis, technical SEO, and content strategy. Derived from [jqueryscript/awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code).

## What This Project Does

This skill suite equips AI coding agents with SEO and content marketing capabilities:

- **Keyword Research** — clustering, opportunity scoring, SERP intent mapping
- **Content Audits** — quality scoring, duplication detection, cannibalization analysis
- **Technical SEO** — crawl budget, Core Web Vitals, schema markup validation
- **Competitor Analysis** — backlink gaps, topic gaps, featured snippet opportunities
- **Content Strategy** — automated briefs, calendars, and optimization workflows
- **SERP Monitoring** — rank tracking, volatility alerts, CTR optimization
- **Link Building** — prospect identification, outreach templates, DA/DR filtering
- **Local SEO** — NAP consistency, GBP optimization, citation audits

All commands use a consistent structured-output interface with progress tracking, findings tables, action plans, and next-step recommendations.

## Installation

### Method 1: Clone to Skills Directory

```bash
# Clone the repository
git clone https://github.com/FabledPackerRedeem/r05-jqueryscript-awesome-claude-code-seo.git

# Copy to Claude skills directory
mkdir -p ~/.claude/skills
cp -r r05-jqueryscript-awesome-claude-code-seo ~/.claude/skills/seo-content-marketing/

# Register in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

### Method 2: Direct Registration

In a Claude Code session:

```bash
/read /path/to/r05-jqueryscript-awesome-claude-code-seo/SKILL.md
```

## Core Commands

### Keyword Research

Perform deep keyword clustering and opportunity scoring:

```bash
/keyword-research <target_domain_or_topic>

# With options
/keyword-research "ecommerce platforms" --depth full --export csv

# Example output structure:
# ┌──────────────────────┬────────┬──────┬──────────┬─────────┐
# │ Keyword              │ Volume │ KD   │ Intent   │ Cluster │
# ├──────────────────────┼────────┼──────┼──────────┼─────────┤
# │ best ecommerce...    │ 12,100 │ 45   │ Commercial│ Top-tier│
# │ ecommerce platform...│  8,500 │ 52   │ Commercial│ Top-tier│
# └──────────────────────┴────────┴──────┴──────────┴─────────┘
```

**Use cases:**
- Initial topic research for new content
- Expanding existing keyword portfolio
- Identifying long-tail opportunities
- SERP intent mapping for content strategy

### Content Audit

Comprehensive site-wide content analysis:

```bash
/content-audit --scope full --output md

# Specific sections only
/content-audit --scope blog --check duplication,quality,cannibalization

# Export with recommendations
/content-audit --scope full --export json --include-actions
```

**Output includes:**
- Content quality scores per page
- Duplicate/thin content detection
- Keyword cannibalization matrix
- Prioritized action checklist (refresh, consolidate, remove)

### Technical SEO

Full technical SEO health check:

```bash
/technical-seo <domain>

# Specific checks
/technical-seo example.com --checks crawl,vitals,schema,mobile

# With detailed diagnostics
/technical-seo example.com --verbose --export pdf
```

**Analyzes:**
- Crawl budget and indexability
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- Mobile-friendliness
- Robots.txt and sitemap issues
- Canonical and hreflang tags

### Competitor Gap Analysis

Identify backlink and topic opportunities:

```bash
/competitor-gap <your_domain> <competitor1> <competitor2> <competitor3>

# Focus on specific gaps
/competitor-gap example.com competitor.com --focus backlinks,topics

# With filters
/competitor-gap example.com competitor.com --min-dr 40 --topic-overlap 20
```

**Outputs:**
- Backlink gap analysis (links they have, you don't)
- Topic gap analysis (content they rank for, you don't)
- Featured snippet opportunities
- Actionable link prospects

### Content Brief Generation

AI-powered SEO content briefs:

```bash
/content-brief "<topic_or_keyword>"

# With specific parameters
/content-brief "best CRM software 2024" --word-count 2500 --competitors 5

# Include NLP terms
/content-brief "email marketing tools" --include-nlp --outline detailed
```

**Brief includes:**
- Target word count based on SERP analysis
- H2/H3 outline from top-ranking pages
- NLP terms and semantic keywords
- Internal linking opportunities
- Competitor content analysis
- User intent breakdown

### SERP Monitoring

Track rankings and identify optimization opportunities:

```bash
/serp-monitor <domain> --keywords <keyword_list>

# Daily tracking
/serp-monitor example.com --keywords keywords.csv --frequency daily

# With CTR analysis
/serp-monitor example.com --keywords keywords.csv --include-ctr --alerts
```

**Features:**
- Daily rank tracking
- SERP volatility alerts
- CTR optimization recommendations
- Featured snippet tracking
- Position change notifications

### Link Prospecting

Find quality backlink opportunities:

```bash
/link-prospecting <topic_or_niche>

# With quality filters
/link-prospecting "marketing tools" --min-da 30 --min-dr 25 --max-spam 5

# Include outreach templates
/link-prospecting "saas reviews" --include-templates --contacts
```

**Delivers:**
- Filtered prospect list (DA/DR/spam score)
- Contact information where available
- Outreach email templates
- Link placement opportunities
- Relationship-building recommendations

### Page Speed SEO

Diagnose performance issues impacting rankings:

```bash
/page-speed-seo <url>

# Multiple pages
/page-speed-seo <url_list> --batch --ranking-impact

# With fix recommendations
/page-speed-seo <url> --detailed --code-fixes
```

**Analyzes:**
- Render-blocking resources
- Largest Contentful Paint (LCP)
- Cumulative Layout Shift (CLS)
- First Input Delay (FID)
- Ranking impact assessment
- Code-level optimization recommendations

### Local SEO Audit

NAP consistency and local search optimization:

```bash
/local-seo <business_name> <location>

# Comprehensive audit
/local-seo "Acme Coffee" "New York, NY" --citations --gbp --reviews

# With competitor benchmarking
/local-seo "Acme Coffee" "New York, NY" --competitors 3
```

**Checks:**
- NAP (Name, Address, Phone) consistency
- Google Business Profile optimization
- Local citation accuracy
- Review profile analysis
- Local pack ranking factors
- Competitor benchmarking

### Content Calendar

Data-driven editorial calendar:

```bash
/content-calendar --timeframe 90 --topics <topic_list>

# With search demand data
/content-calendar --timeframe 180 --topics topics.csv --seasonality

# Export to project management tools
/content-calendar --timeframe 90 --export trello,asana
```

**Generates:**
- Content topics prioritized by search demand
- Seasonal trending opportunities
- Publication schedule optimization
- Internal linking strategy
- Content type recommendations (blog, video, infographic)

## Workflows (Multi-Step)

### Full SEO Sprint

12-step comprehensive SEO campaign:

```bash
/workflows:full-seo-sprint <domain> --scope full

# Custom sprint focus
/workflows:full-seo-sprint example.com --focus technical,content --duration 4w
```

**Workflow steps:**
1. Initial site audit
2. Keyword research and mapping
3. Competitor analysis
4. Technical SEO fixes
5. Content gap identification
6. Content brief creation
7. On-page optimization
8. Internal linking structure
9. Link building strategy
10. Local SEO (if applicable)
11. Performance monitoring setup
12. Reporting and iteration

### Launch SEO

Pre-launch SEO validation checklist:

```bash
/workflows:launch-seo <staging_url>

# With migration checks
/workflows:launch-seo <staging_url> --migration-from <old_domain>
```

**Validates:**
- Canonical tag implementation
- Hreflang tags (multi-language sites)
- XML sitemap generation
- Robots.txt configuration
- Redirect chains
- Mobile responsiveness
- Core Web Vitals
- Schema markup
- Analytics and tracking setup

### Content Refresh

Recover lost rankings by refreshing underperforming content:

```bash
/workflows:content-refresh <domain>

# Target specific decline
/workflows:content-refresh example.com --ranking-drop 5+ --timeframe 90d
```

**Process:**
1. Identify declining pages (rank drops)
2. Analyze SERP changes
3. Audit content quality vs. top rankers
4. Generate refresh recommendations
5. Update content outline
6. Re-optimize on-page elements
7. Enhance internal linking
8. Monitor recovery

### Authority Building

End-to-end digital PR and link building:

```bash
/workflows:authority-building <domain> <niche>

# With target metrics
/workflows:authority-building example.com "marketing tech" --target-dr 60 --timeframe 6m
```

**Campaign steps:**
1. Competitor backlink analysis
2. Link gap identification
3. Prospect list creation
4. Content asset planning (linkable assets)
5. Outreach template creation
6. Relationship building strategy
7. Digital PR opportunities
8. Guest posting targets
9. Broken link building
10. Progress tracking and reporting

### AI Content Pipeline

Automated keyword-to-publish workflow:

```bash
/workflows:ai-content-pipeline <keyword_list>

# With quality gates
/workflows:ai-content-pipeline keywords.csv --quality-check human --publish auto
```

**Pipeline stages:**
1. Keyword selection and clustering
2. Content brief generation
3. AI-assisted draft creation
4. SEO optimization (NLP, structure)
5. Human quality review
6. Internal linking injection
7. Media asset integration
8. Pre-publish checklist
9. Publication
10. Performance tracking

## Configuration

### Environment Variables

Store sensitive credentials as environment variables:

```bash
# Analytics and tracking
export GOOGLE_ANALYTICS_ID=your_ga_id
export GOOGLE_SEARCH_CONSOLE_PROPERTY=your_property

# SEO tool APIs
export AHREFS_API_KEY=your_ahrefs_key
export SEMRUSH_API_KEY=your_semrush_key
export MOZ_API_KEY=your_moz_key

# Content management
export WORDPRESS_URL=https://your-site.com
export WORDPRESS_USER=your_username
export WORDPRESS_APP_PASSWORD=your_app_password

# Project management
export TRELLO_API_KEY=your_trello_key
export ASANA_ACCESS_TOKEN=your_asana_token
```

### Skill Configuration File

Create `~/.claude/skills/seo-content-marketing/config.yml`:

```yaml
# Default settings
defaults:
  output_format: md
  verbosity: normal
  export_path: ./seo-reports/

# Tool integrations
integrations:
  analytics:
    - google_analytics
    - google_search_console
  seo_tools:
    - ahrefs
    - semrush
    - moz
  cms:
    - wordpress
    - webflow
  
# Command-specific settings
commands:
  keyword_research:
    default_depth: standard
    min_volume: 10
    max_difficulty: 70
  
  content_audit:
    quality_threshold: 60
    word_count_min: 300
  
  technical_seo:
    cwv_threshold:
      lcp: 2.5
      fid: 100
      cls: 0.1
  
  competitor_gap:
    max_competitors: 5
    min_dr: 30

# Workflow preferences
workflows:
  full_seo_sprint:
    duration_weeks: 4
    report_frequency: weekly
  
  content_refresh:
    ranking_drop_threshold: 5
    lookback_days: 90
```

## Common Patterns

### Pattern 1: New Content Strategy

Research and plan content from scratch:

```bash
# Step 1: Keyword research
/keyword-research "target niche" --depth full --export csv

# Step 2: Competitor gap analysis
/competitor-gap your-domain.com competitor1.com competitor2.com --focus topics

# Step 3: Generate content calendar
/content-calendar --timeframe 90 --topics research-output.csv --seasonality

# Step 4: Create briefs for top priorities
/content-brief "high priority keyword 1" --word-count 2500 --outline detailed
/content-brief "high priority keyword 2" --word-count 1800 --outline detailed
```

### Pattern 2: Site Migration SEO

Ensure SEO preservation during site redesign:

```bash
# Pre-migration audit
/technical-seo old-domain.com --export baseline-report.json

# Create redirect map
/workflows:launch-seo staging-url.com --migration-from old-domain.com

# Post-migration validation
/technical-seo new-domain.com --compare baseline-report.json

# Monitor rankings
/serp-monitor new-domain.com --keywords all-keywords.csv --frequency daily --alerts
```

### Pattern 3: Recover Lost Rankings

Fix pages that have dropped in rankings:

```bash
# Run content refresh workflow
/workflows:content-refresh your-domain.com --ranking-drop 5+ --timeframe 90d

# For each identified page:
/content-brief "page target keyword" --competitors 10 --include-nlp

# Technical check
/page-speed-seo https://your-domain.com/page-url --ranking-impact

# Re-optimize and monitor
/serp-monitor your-domain.com --keywords page-keywords.csv --frequency daily
```

### Pattern 4: Link Building Campaign

Build authority through strategic link acquisition:

```bash
# Start authority building workflow
/workflows:authority-building your-domain.com "your niche" --target-dr 60

# Find specific opportunities
/link-prospecting "niche keywords" --min-da 40 --include-templates

# Analyze competitor backlinks
/competitor-gap your-domain.com top-competitor.com --focus backlinks

# Track progress
# (Manual tracking or integrate with CRM)
```

### Pattern 5: Monthly SEO Reporting

Automated monthly performance review:

```bash
# Technical health check
/technical-seo your-domain.com --export monthly-technical-$(date +%Y-%m).pdf

# Content performance
/content-audit --scope blog --check quality,performance --export json

# Ranking changes
/serp-monitor your-domain.com --keywords all-keywords.csv --report monthly

# Competitor benchmarking
/competitor-gap your-domain.com competitor1.com competitor2.com --report monthly
```

## Code Examples

### Example 1: Custom Keyword Research Script

```python
#!/usr/bin/env python3
"""
Custom keyword research automation
Integrates with SEO skill suite
"""

import os
import subprocess
import json

def run_keyword_research(topic, depth="full", min_volume=100):
    """
    Execute keyword research command and parse results
    """
    cmd = [
        "/keyword-research",
        topic,
        f"--depth={depth}",
        f"--min-volume={min_volume}",
        "--export=json"
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    if result.returncode == 0:
        return json.loads(result.stdout)
    else:
        raise Exception(f"Keyword research failed: {result.stderr}")

def cluster_keywords(keywords, max_clusters=10):
    """
    Further cluster keywords into content groups
    """
    clusters = {}
    
    for kw in keywords:
        intent = kw.get('intent', 'informational')
        if intent not in clusters:
            clusters[intent] = []
        clusters[intent].append(kw)
    
    return clusters

def main():
    topic = "email marketing automation"
    
    # Run research
    print(f"Researching: {topic}")
    keywords = run_keyword_research(topic, depth="full", min_volume=50)
    
    # Cluster by intent
    clustered = cluster_keywords(keywords)
    
    # Export for content calendar
    with open("keyword-clusters.json", "w") as f:
        json.dump(clustered, f, indent=2)
    
    print(f"Found {len(keywords)} keywords in {len(clustered)} clusters")

if __name__ == "__main__":
    main()
```

### Example 2: Content Audit Automation

```javascript
#!/usr/bin/env node
/**
 * Automated content audit with Slack notifications
 * Requires: SLACK_WEBHOOK_URL environment variable
 */

const { execSync } = require('child_process');
const https = require('https');
const url = require('url');

async function runContentAudit(domain, scope = 'full') {
  const cmd = `/content-audit --scope ${scope} --export json`;
  
  try {
    const output = execSync(cmd, { encoding: 'utf8' });
    return JSON.parse(output);
  } catch (error) {
    throw new Error(`Audit failed: ${error.message}`);
  }
}

async function sendSlackNotification(auditResults) {
  const webhookUrl = process.env.SLACK_WEBHOOK_URL;
  
  if (!webhookUrl) {
    console.log('No Slack webhook configured, skipping notification');
    return;
  }
  
  const issues = auditResults.findings.filter(f => f.severity === 'high');
  
  const message = {
    text: `📊 Content Audit Complete`,
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `*Content Audit Results for ${auditResults.domain}*`
        }
      },
      {
        type: 'section',
        fields: [
          { type: 'mrkdwn', text: `*Total Pages:*\n${auditResults.total_pages}` },
          { type: 'mrkdwn', text: `*High-Priority Issues:*\n${issues.length}` },
          { type: 'mrkdwn', text: `*Avg Quality Score:*\n${auditResults.avg_quality}` }
        ]
      }
    ]
  };
  
  return new Promise((resolve, reject) => {
    const options = url.parse(webhookUrl);
    options.method = 'POST';
    options.headers = {
      'Content-Type': 'application/json',
    };
    
    const req = https.request(options, (res) => {
      res.on('data', () => {});
      res.on('end', () => resolve());
    });
    
    req.on('error', reject);
    req.write(JSON.stringify(message));
    req.end();
  });
}

async function main() {
  const domain = process.argv[2] || 'your-domain.com';
  
  console.log(`Running content audit for ${domain}...`);
  const results = await runContentAudit(domain);
  
  console.log(`Audit complete. Sending notification...`);
  await sendSlackNotification(results);
  
  console.log('Done!');
}

main().catch(console.error);
```

### Example 3: Automated Content Brief Generation

```bash
#!/bin/bash
# Generate content briefs for a list of keywords

KEYWORDS_FILE="${1:-keywords.txt}"
OUTPUT_DIR="${2:-./content-briefs}"

if [ ! -f "$KEYWORDS_FILE" ]; then
  echo "Keywords file not found: $KEYWORDS_FILE"
  exit 1
fi

mkdir -p "$OUTPUT_DIR"

while IFS= read -r keyword; do
  # Skip empty lines
  [ -z "$keyword" ] && continue
  
  echo "Generating brief for: $keyword"
  
  # Create filename-safe version
  filename=$(echo "$keyword" | tr ' ' '-' | tr -cd '[:alnum:]-')
  
  # Generate brief
  /content-brief "$keyword" \
    --word-count 2000 \
    --competitors 5 \
    --include-nlp \
    --outline detailed \
    --export md > "$OUTPUT_DIR/$filename.md"
  
  echo "✓ Saved to $OUTPUT_DIR/$filename.md"
  
  # Rate limiting
  sleep 2
done < "$KEYWORDS_FILE"

echo ""
echo "All briefs generated in $OUTPUT_DIR/"
```

## Troubleshooting

### Command Not Found

**Problem:** `/keyword-research` or other commands not recognized

**Solution:**
```bash
# Ensure skill is properly loaded
/read ~/.claude/skills/seo-content-marketing/SKILL.md

# Verify skill directory structure
ls -la ~/.claude/skills/seo-content-marketing/

# Re-register skill in current session
/refresh-skills
```

### API Rate Limiting

**Problem:** "Rate limit exceeded" errors from SEO tool APIs

**Solution:**
```bash
# Add delays between commands in scripts
sleep 5

# Use batch mode with throttling
/keyword-research topics.csv --batch --throttle 2000

# Configure rate limits in config.yml
# rate_limits:
#   ahrefs: 1000/day
#   semrush: 500/day
```

### Export Failures

**Problem:** `--export` flag produces errors or empty files

**Solution:**
```bash
# Ensure export directory exists
mkdir -p ./seo-reports

# Specify absolute path
/content-audit --export json --output /absolute/path/to/reports/audit.json

# Check permissions
chmod 755 ./seo-reports

# Verify disk space
df -h
```

### Incomplete Audit Results

**Problem:** Content audit or technical SEO returning partial results

**Solution:**
```bash
# Increase timeout for large sites
/content-audit --scope full --timeout 300

# Break into smaller scopes
/content-audit --scope /blog/
/content-audit --scope /products/

# Use verbose mode to see progress
/technical-seo example.com --verbose

# Check for robots.txt blocking
curl https://example.com/robots.txt
```

### Integration Authentication Issues

**Problem:** Google Analytics, Search Console, or SEO tools not connecting

**Solution:**
```bash
# Verify environment variables
echo $GOOGLE_ANALYTICS_ID
echo $AHREFS_API_KEY

# Re-authenticate
/auth:google-search-console
/auth:ahrefs

# Check API key validity
curl -H "Authorization: Bearer $AHREFS_API_KEY" \
  https://api.ahrefs.com/v3/account

# Update credentials in config
nano ~/.claude/skills/seo-content-marketing/config.yml
```

### Workflow Interruptions

**Problem:** Multi-step workflow stops mid-process

**Solution:**
```bash
# Resume from checkpoint
/workflows:full-seo-sprint example.com --resume step-5

# View workflow state
/workflows:status full-seo-sprint

# Save progress manually
/workflows:checkpoint

# Start with smaller scope
/workflows:full-seo-sprint example.com --scope minimal
```

## Advanced Usage

### Custom Workflow Creation

Create domain-specific workflows by chaining commands:

```bash
# E-commerce SEO workflow
/technical-seo example.com --checks schema,structured-data
/keyword-research "product categories" --intent commercial
/content-audit --scope /products/ --check quality,thin
/competitor-gap example.com competitor.com --focus featured-snippets
```

### Scheduled Monitoring

Set up recurring SEO monitoring:

```bash
# Add to crontab
0 9 * * 1 /serp-monitor example.com --keywords all-keywords.csv --report weekly --email team@example.com
0 1 1 * * /content-audit --scope full --export json --archive
```

### Integration with CI/CD

Pre-deployment SEO validation:

```yaml
# .github/workflows/seo-check.yml
name: SEO Pre-Deploy Check

on:
  pull_request:
    branches: [main]

jobs:
  seo-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run Technical SEO Check
        run: |
          /technical-seo ${{ env.STAGING_URL }} \
            --checks canonical,mobile,vitals \
            --fail-on error
      
      - name: Validate Schema Markup
        run: |
          /technical-seo ${{ env.STAGING_URL }} \
            --checks schema \
            --export json > schema-validation.json
      
      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: seo-audit
          path: schema-validation.json
```

## Resources

- **Source Repository:** [jqueryscript/awesome-claude-code](https://github.com/jqueryscript/awesome-claude-code)
- **Project Repository:** [r05-jqueryscript-awesome-claude-code-seo](https://github.com/FabledPackerRedeem/r05-jqueryscript-awesome-claude-code-seo)
- **Documentation:** See README.md in project root
- **Issues:** Report bugs via GitHub Issues
- **License:** MIT

---

**Version:** 1.0.0  
**Last Updated:** 2026-05-11  
**Maintained by:** FabledPackerRedeem
