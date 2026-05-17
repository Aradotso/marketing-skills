---
name: seo-content-marketing-skill-suite
description: SEO & content marketing automation commands for keyword research, content audits, technical SEO, SERP analysis, and link prospecting
triggers:
  - analyze keywords for this topic
  - audit content quality across the site
  - check technical SEO issues
  - find competitor content gaps
  - generate an SEO content brief
  - track SERP rankings and volatility
  - prospect quality backlink opportunities
  - analyze page speed impact on SEO
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

The SEO & Content Marketing Skills Suite is a specialized command collection that provides structured, actionable SEO and content marketing workflows. It delivers:

- **Keyword research** with clustering and SERP intent mapping
- **Content audits** with quality scoring and cannibalization detection
- **Technical SEO** analysis (Core Web Vitals, schema, crawlability)
- **Competitor gap** analysis for backlinks, topics, and featured snippets
- **Content briefs** with NLP term extraction and optimization targets
- **SERP monitoring** with volatility alerts and CTR recommendations
- **Link prospecting** with domain authority filtering
- **Local SEO** audits for NAP consistency and citation building
- **Content calendars** driven by search demand seasonality

All commands output structured, prioritized recommendations with visual progress tracking and time-boxed action plans.

## Installation

### Clone and Register the Skill

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills/

# Clone or copy the skill suite
cp -r /path/to/r15-shanraisshan-claude-code-best-practice-seo \
  ~/.claude/skills/seo-content-marketing-skill-suite/

# Register in your Claude Code session
/read ~/.claude/skills/seo-content-marketing-skill-suite/SKILL.md
```

### Prerequisites

The skill suite expects these tools to be available:

```bash
# Install core dependencies
npm install -g lighthouse
pip install scrapy requests beautifulsoup4 pandas

# Optional: for advanced crawling
npm install -g screaming-frog-cli
```

### Environment Variables

Set up API credentials for data sources:

```bash
export AHREFS_API_KEY=your_key_here
export SEMRUSH_API_KEY=your_key_here
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=/path/to/credentials.json
export SCREAMING_FROG_LICENSE=your_license_key
```

## Core Commands

### `/keyword-research`

Deep keyword clustering with SERP intent mapping and opportunity scoring.

**Syntax:**
```bash
/keyword-research <seed_keyword> [--output md|json|csv] [--depth shallow|deep]
```

**Example:**
```bash
/keyword-research "digital marketing tools" --output md --depth deep
```

**What It Does:**
1. Pulls keyword suggestions from multiple sources (Google Keyword Planner, Ahrefs, SEMrush)
2. Clusters keywords by semantic similarity
3. Maps SERP intent (informational, navigational, transactional, commercial)
4. Scores opportunity based on volume, difficulty, and current ranking
5. Outputs prioritized keyword groups with content recommendations

**Expected Output Structure:**
```markdown
## Keyword Research: digital marketing tools

### Cluster 1: Tool Comparison (High Priority)
- digital marketing tools comparison (Vol: 2,900 | Diff: 45 | Intent: Commercial)
- best digital marketing tools 2026 (Vol: 1,600 | Diff: 52 | Intent: Commercial)
- free digital marketing tools (Vol: 3,200 | Diff: 38 | Intent: Commercial)

**Recommended Action:** Create comparison guide with tool matrix
**Difficulty:** Medium | **Est. Traffic:** 5,400/mo | **Priority:** 🔴 High

### Cluster 2: Category Specific
...
```

---

### `/content-audit`

Full-site content quality scoring with duplication and cannibalization detection.

**Syntax:**
```bash
/content-audit [--scope full|sample] [--url <site_url>] [--output md|json]
```

**Example:**
```bash
/content-audit --scope full --url https://example.com --output md
```

**What It Does:**
1. Crawls site and extracts all indexable content
2. Analyzes word count, readability, keyword density
3. Detects duplicate/thin content
4. Identifies keyword cannibalization
5. Scores each page (0-100) across quality dimensions
6. Generates prioritized fix list

**Quality Metrics Evaluated:**
- Word count (target: 1,500+ for blog posts)
- Readability (Flesch-Kincaid score)
- Keyword usage (natural density, LSI terms)
- Internal linking depth
- Meta tag completeness
- Multimedia presence
- Schema markup

---

### `/technical-seo`

Crawl budget, Core Web Vitals, schema markup, and indexability audit.

**Syntax:**
```bash
/technical-seo <site_url> [--checks all|vitals|schema|crawl] [--output md]
```

**Example:**
```bash
/technical-seo https://example.com --checks all --output md
```

**What It Does:**
1. Runs Lighthouse audit for Core Web Vitals (LCP, FID, CLS)
2. Validates schema markup with Schema.org validator
3. Checks robots.txt, XML sitemap, canonical tags
4. Analyzes crawl budget efficiency (404s, redirects, orphan pages)
5. Tests mobile-friendliness and HTTPS implementation
6. Reports indexability blockers

**Critical Checks:**
```python
# Example check: Core Web Vitals
def check_core_web_vitals(url):
    results = {
        'LCP': lighthouse_lcp(url),  # Target: < 2.5s
        'FID': lighthouse_fid(url),  # Target: < 100ms
        'CLS': lighthouse_cls(url),  # Target: < 0.1
    }
    
    for metric, value in results.items():
        status = 'PASS' if meets_threshold(metric, value) else 'FAIL'
        print(f"{metric}: {value} — {status}")
    
    return results
```

---

### `/competitor-gap`

Backlink gap, topic gap, and featured snippet opportunity analysis.

**Syntax:**
```bash
/competitor-gap <your_domain> <competitor_domain> [--gap-type links|topics|snippets]
```

**Example:**
```bash
/competitor-gap example.com competitor.com --gap-type topics
```

**What It Does:**
1. **Backlink Gap:** Identifies domains linking to competitors but not to you
2. **Topic Gap:** Finds keywords competitors rank for that you don't
3. **Featured Snippet Gap:** Lists queries where competitors own the snippet
4. Scores each opportunity by estimated traffic value
5. Provides outreach/content recommendations

**Topic Gap Analysis (Python):**
```python
import requests
import pandas as pd

def topic_gap_analysis(your_domain, competitor_domain):
    # Fetch ranking keywords for both domains
    your_keywords = fetch_keywords(your_domain, api_key=os.getenv('SEMRUSH_API_KEY'))
    comp_keywords = fetch_keywords(competitor_domain, api_key=os.getenv('SEMRUSH_API_KEY'))
    
    # Find keywords where competitor ranks but you don't
    gap = comp_keywords[~comp_keywords['keyword'].isin(your_keywords['keyword'])]
    
    # Score by volume * (1 / difficulty)
    gap['opportunity_score'] = gap['volume'] * (100 - gap['difficulty']) / 100
    gap = gap.sort_values('opportunity_score', ascending=False)
    
    return gap.head(50)

# Usage
gaps = topic_gap_analysis('example.com', 'competitor.com')
print(gaps[['keyword', 'volume', 'difficulty', 'opportunity_score']])
```

---

### `/content-brief`

AI-generated SEO content brief with outline, NLP terms, and word count targets.

**Syntax:**
```bash
/content-brief "<target_keyword>" [--format detailed|quick]
```

**Example:**
```bash
/content-brief "how to do keyword research" --format detailed
```

**What It Does:**
1. Analyzes top 10 SERP results for the keyword
2. Extracts common headings, NLP entities, and semantic terms
3. Calculates average word count and content depth
4. Identifies content gaps in existing top results
5. Generates structured brief with:
   - Target word count
   - Required H2/H3 sections
   - NLP terms to include
   - Questions to answer
   - Internal linking suggestions

**Brief Output Template:**
```markdown
## Content Brief: how to do keyword research

**Primary Keyword:** how to do keyword research
**Search Intent:** Informational (tutorial)
**Target Word Count:** 2,400–2,800 words
**SERP Competition:** Medium (Difficulty: 48)

### Required Sections (H2):
1. What is Keyword Research? (200–300 words)
2. Why Keyword Research Matters for SEO (300–400 words)
3. Step-by-Step Keyword Research Process (800–1,000 words)
   - 3.1 Brainstorm Seed Keywords
   - 3.2 Use Keyword Research Tools
   - 3.3 Analyze Search Intent
   - 3.4 Assess Keyword Difficulty
4. Best Keyword Research Tools in 2026 (400–500 words)
5. Common Keyword Research Mistakes (300–400 words)

### Must-Include NLP Terms:
- search volume, keyword difficulty, long-tail keywords
- search intent, SERP analysis, keyword clustering
- semantic keywords, LSI keywords, keyword gap

### Internal Linking Opportunities:
- Link to: /guide/seo-basics
- Link to: /tools/keyword-research-tools
```

---

### `/serp-monitor`

Daily rank tracking with volatility alerts and CTR optimization tips.

**Syntax:**
```bash
/serp-monitor <domain> [--keywords keywords.csv] [--alert-threshold 3]
```

**Example:**
```bash
/serp-monitor example.com --keywords target_keywords.csv --alert-threshold 3
```

**What It Does:**
1. Tracks position changes for target keywords
2. Calculates SERP volatility score
3. Identifies ranking drops > threshold
4. Estimates CTR loss/gain from position changes
5. Suggests title/meta description optimizations

**Monitoring Script (Python):**
```python
import os
import pandas as pd
from serpapi import GoogleSearch

def track_serp_positions(domain, keywords):
    results = []
    
    for keyword in keywords:
        params = {
            "q": keyword,
            "api_key": os.getenv('SERPAPI_KEY'),
            "num": 20
        }
        
        search = GoogleSearch(params)
        serp = search.get_dict()
        
        position = None
        for idx, result in enumerate(serp.get('organic_results', []), 1):
            if domain in result.get('link', ''):
                position = idx
                break
        
        results.append({
            'keyword': keyword,
            'position': position or '>20',
            'url': result.get('link') if position else None
        })
    
    return pd.DataFrame(results)

# Track and compare to previous day
current = track_serp_positions('example.com', ['keyword 1', 'keyword 2'])
# Compare with historical data and alert on drops
```

---

### `/link-prospecting`

Quality backlink prospect list with DA/DR filters and outreach templates.

**Syntax:**
```bash
/link-prospecting "<topic>" [--min-dr 30] [--max-results 100]
```

**Example:**
```bash
/link-prospecting "digital marketing" --min-dr 40 --max-results 50
```

**What It Does:**
1. Finds relevant websites in the niche
2. Filters by Domain Rating (DR) / Domain Authority (DA)
3. Checks for link opportunities (resource pages, guest posts, broken links)
4. Extracts contact information
5. Generates personalized outreach templates

**Prospecting Workflow:**
```python
import requests
from bs4 import BeautifulSoup

def find_link_prospects(topic, min_dr=30):
    # Use Ahrefs API to find pages about topic
    api_key = os.getenv('AHREFS_API_KEY')
    
    search_query = f"intext:\"{topic}\" AND (\"write for us\" OR \"guest post\" OR \"resources\")"
    
    response = requests.get(
        f"https://api.ahrefs.com/v3/site-explorer/pages",
        params={'q': search_query, 'min_dr': min_dr},
        headers={'Authorization': f'Bearer {api_key}'}
    )
    
    prospects = response.json()['pages']
    
    # Filter for active opportunities
    opportunities = []
    for page in prospects:
        if page['dr'] >= min_dr and page['traffic'] > 100:
            opportunities.append({
                'url': page['url'],
                'dr': page['dr'],
                'traffic': page['traffic'],
                'type': detect_opportunity_type(page['url'])
            })
    
    return opportunities

def detect_opportunity_type(url):
    if 'guest-post' in url or 'write-for-us' in url:
        return 'Guest Post'
    elif 'resources' in url or 'links' in url:
        return 'Resource Page'
    else:
        return 'General Outreach'
```

---

### `/page-speed-seo`

Render-blocking resources, LCP, CLS, FID diagnosis mapped to ranking impact.

**Syntax:**
```bash
/page-speed-seo <url> [--device mobile|desktop]
```

**Example:**
```bash
/page-speed-seo https://example.com/blog/post --device mobile
```

**What It Does:**
1. Runs Lighthouse performance audit
2. Identifies render-blocking CSS/JS
3. Measures Largest Contentful Paint (LCP)
4. Detects layout shift sources (CLS)
5. Maps each issue to estimated ranking impact
6. Provides fix code snippets

**Speed Analysis Output:**
```markdown
## Page Speed SEO Analysis: example.com/blog/post

### Core Web Vitals Status
- **LCP:** 3.2s ❌ (Target: <2.5s) — High ranking impact
- **FID:** 85ms ✅ (Target: <100ms) — Passing
- **CLS:** 0.18 ❌ (Target: <0.1) — Medium ranking impact

### Critical Issues (Fix First)
1. **Render-blocking CSS (3 files)**
   - Impact: Delays LCP by ~1.2s
   - Fix: Inline critical CSS, defer non-critical
   ```html
   <link rel="preload" href="critical.css" as="style" onload="this.rel='stylesheet'">
   ```

2. **Unoptimized images**
   - Impact: LCP element is 1.2MB PNG
   - Fix: Convert to WebP, add responsive images
   ```html
   <picture>
     <source srcset="hero.webp" type="image/webp">
     <img src="hero.jpg" alt="..." loading="eager" fetchpriority="high">
   </picture>
   ```
```

---

### `/local-seo`

NAP consistency, Google Business Profile optimization, and local citation audit.

**Syntax:**
```bash
/local-seo "<business_name>" "<city>" [--check citations|gbp|nap]
```

**Example:**
```bash
/local-seo "Acme Plumbing" "Austin" --check all
```

**What It Does:**
1. Checks NAP (Name, Address, Phone) consistency across directories
2. Audits Google Business Profile completeness
3. Finds citation opportunities (Yelp, YellowPages, local directories)
4. Analyzes local pack rankings
5. Reviews schema markup for LocalBusiness

---

### `/content-calendar`

Data-driven editorial calendar built from search demand and seasonality.

**Syntax:**
```bash
/content-calendar [--months 6] [--topics topics.csv] [--output calendar.csv]
```

**Example:**
```bash
/content-calendar --months 12 --topics seed_keywords.csv --output editorial-plan.csv
```

**What It Does:**
1. Analyzes search volume trends over time (Google Trends)
2. Identifies seasonal peaks for each topic
3. Maps content types to funnel stages
4. Schedules posts to align with demand curves
5. Balances quick wins vs. long-term authority content

---

## Workflows (Multi-Step)

### `full-seo-sprint`

Complete 12-step SEO sprint from audit to implementation.

**Syntax:**
```bash
/workflows:full-seo-sprint <domain> [--scope full|quick]
```

**Steps:**
1. Technical SEO audit
2. Keyword research
3. Content audit
4. Competitor gap analysis
5. On-page optimization plan
6. Content brief generation
7. Internal linking map
8. Schema markup plan
9. Link prospecting
10. Page speed fixes
11. Local SEO setup (if applicable)
12. Tracking setup (GSC, Analytics)

---

### `launch-seo`

Pre-launch SEO checklist with canonical, hreflang, and sitemap validation.

**Syntax:**
```bash
/workflows:launch-seo <staging_url>
```

**Checklist:**
- [ ] Robots.txt configured correctly
- [ ] XML sitemap generated and submitted
- [ ] Canonical tags on all pages
- [ ] Hreflang for multi-language (if applicable)
- [ ] Schema markup validated
- [ ] Core Web Vitals passing
- [ ] Mobile-friendly test passed
- [ ] Google Search Console verified
- [ ] Google Analytics installed
- [ ] 301 redirects for old URLs mapped

---

### `content-refresh`

Identify and refresh underperforming pages to recover lost rankings.

**Syntax:**
```bash
/workflows:content-refresh <domain>
```

**Process:**
1. Identify pages with declining traffic (GSC data)
2. Analyze why rankings dropped (content decay, new competitors)
3. Generate refresh briefs (new sections, updated stats)
4. Update publish dates and re-index
5. Monitor recovery

---

### `authority-building`

End-to-end digital PR and link-building campaign.

**Steps:**
1. Identify linkable assets (data studies, tools, guides)
2. Prospect high-DR targets
3. Craft outreach campaigns
4. Track mentions and backlinks
5. Measure domain authority growth

---

### `ai-content-pipeline`

Keyword → brief → draft → optimize → publish automation.

**Pipeline:**
```bash
# Step 1: Research keywords
/keyword-research "topic" --output keywords.json

# Step 2: Generate briefs
for kw in keywords.json:
  /content-brief "$kw" --format detailed > briefs/$kw.md

# Step 3: Draft content (AI-assisted)
# (Human or AI writer creates draft)

# Step 4: Optimize
/technical-seo <draft_url> --checks all

# Step 5: Publish and monitor
/serp-monitor <domain> --keywords keywords.csv
```

---

## Configuration

### Skill Settings File

Create `~/.seo-skills/config.yaml`:

```yaml
default_output: md
api_keys:
  ahrefs: ${AHREFS_API_KEY}
  semrush: ${SEMRUSH_API_KEY}
  serpapi: ${SERPAPI_KEY}
  
thresholds:
  keyword_difficulty_max: 60
  min_domain_rating: 30
  content_word_count_min: 1500
  core_web_vitals:
    lcp_max: 2.5
    fid_max: 100
    cls_max: 0.1

workflows:
  full_seo_sprint:
    scope: full
    parallel_execution: true
  
  content_refresh:
    lookback_days: 90
    min_traffic_drop_pct: 25
```

---

## Common Patterns

### Pattern 1: Monthly SEO Health Check

```bash
# Run comprehensive monthly audit
/technical-seo https://example.com --checks all --output monthly-report.md
/content-audit --scope full --url https://example.com
/serp-monitor example.com --keywords top-keywords.csv

# Review and prioritize action items
# Track progress month-over-month
```

### Pattern 2: New Content Creation Flow

```bash
# 1. Research
/keyword-research "target topic" --depth deep --output keywords.json

# 2. Competitive analysis
/competitor-gap example.com competitor.com --gap-type topics

# 3. Create brief
/content-brief "chosen keyword" --format detailed > brief.md

# 4. After publishing, monitor
/serp-monitor example.com --keywords new-keywords.csv
```

### Pattern 3: Ranking Recovery

```bash
# 1. Identify drops
/serp-monitor example.com --alert-threshold 5

# 2. Audit affected pages
/content-audit --scope sample --url https://example.com/affected-page

# 3. Refresh content
/content-brief "target keyword" --format quick

# 4. Fix technical issues
/page-speed-seo https://example.com/affected-page
/technical-seo https://example.com/affected-page --checks schema
```

---

## Troubleshooting

### API Rate Limits

If you hit rate limits:

```bash
# Add delays between requests in config
echo "rate_limit_delay: 2" >> ~/.seo-skills/config.yaml

# Or use rotating API keys
export AHREFS_API_KEY_POOL="key1,key2,key3"
```

### Crawl Failures

For large sites causing timeouts:

```bash
# Use sample scope first
/content-audit --scope sample --url https://example.com

# Or increase timeout in config
echo "crawl_timeout: 300" >> ~/.seo-skills/config.yaml
```

### Missing Dependencies

```bash
# Install all required tools
npm install -g lighthouse cli-table3
pip install scrapy pandas requests beautifulsoup4 lxml google-api-python-client

# Verify installations
lighthouse --version
python -c "import scrapy; print(scrapy.__version__)"
```

### Google API Authentication

For Search Console integration:

```bash
# 1. Create service account in Google Cloud Console
# 2. Download credentials JSON
# 3. Set environment variable
export GOOGLE_SEARCH_CONSOLE_CREDENTIALS=/path/to/credentials.json

# 4. Share GSC property with service account email
# 5. Test connection
/serp-monitor example.com --source gsc
```

---

## Output Format Standards

All commands follow this structure:

```markdown
## [Command Name]: [Target]

### Summary
- Key metric 1: value
- Key metric 2: value

### Findings (Priority Order)
🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

### Action Plan
**Quick Wins (0-2 weeks):**
- [ ] Action item with time estimate

**Medium-term (2-8 weeks):**
- [ ] Action item with time estimate

**Strategic (2-6 months):**
- [ ] Action item with time estimate

### Next Steps
Suggested follow-up commands or workflows
```

---

## Integration Examples

### With CI/CD Pipeline

```yaml
# .github/workflows/seo-check.yml
name: SEO Health Check
on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday

jobs:
  seo_audit:
    runs-on: ubuntu-latest
    steps:
      - name: Technical SEO Audit
        run: |
          /technical-seo ${{ secrets.SITE_URL }} --output report.md
      
      - name: Upload Report
        uses: actions/upload-artifact@v2
        with:
          name: seo-report
          path: report.md
```

### With Notion for Tracking

```python
# Export content calendar to Notion
import os
from notion_client import Client

notion = Client(auth=os.getenv('NOTION_API_KEY'))

# Run content calendar command and parse output
calendar_data = run_command('/content-calendar --months 6 --output calendar.json')

# Create Notion database entries
for item in calendar_data:
    notion.pages.create(
        parent={"database_id": os.getenv('NOTION_DB_ID')},
        properties={
            "Title": {"title": [{"text": {"content": item['topic']}}]},
            "Publish Date": {"date": {"start": item['date']}},
            "Keyword": {"rich_text": [{"text": {"content": item['keyword']}}]},
            "Priority": {"select": {"name": item['priority']}}
        }
    )
```

---

## Advanced Usage

### Custom Scoring Algorithms

Override default opportunity scoring:

```python
# ~/.seo-skills/custom_scoring.py
def custom_keyword_score(keyword_data):
    volume = keyword_data['volume']
    difficulty = keyword_data['difficulty']
    current_rank = keyword_data.get('current_position', 100)
    
    # Custom formula: prioritize low-hanging fruit
    if current_rank <= 20:
        rank_multiplier = 2.0  # Already ranking, easier to improve
    else:
        rank_multiplier = 1.0
    
    score = (volume / difficulty) * rank_multiplier
    return score

# Use in config
# scoring_function: custom_scoring.custom_keyword_score
```

### Batch Processing

Process multiple domains:

```bash
#!/bin/bash
# batch-audit.sh

for domain in $(cat domains.txt); do
  echo "Auditing $domain..."
  /technical-seo $domain --output reports/$domain-technical.md
  /content-audit --url https://$domain --output reports/$domain-content.md
  sleep 60  # Rate limiting
done

echo "All audits complete. Reports in ./reports/"
```

---

## Further Resources

- **Source Project:** [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- **SEO Documentation:** Each command includes inline help via `/command --help`
- **Community Workflows:** See `~/.seo-skills/examples/` for contributed patterns

---

**License:** MIT  
**Maintained by:** MagicStarfishBoost  
**Skill Version:** 1.0.0
