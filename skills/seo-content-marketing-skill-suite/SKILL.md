---
name: seo-content-marketing-skill-suite
description: SEO & content marketing automation with keyword research, content audits, technical SEO, competitor analysis, and workflow orchestration
triggers:
  - analyze SEO performance for this site
  - run a content audit and keyword research
  - check technical SEO issues and Core Web Vitals
  - find competitor content gaps and backlink opportunities
  - generate an SEO content brief with optimization targets
  - create a data-driven content calendar
  - audit local SEO and NAP consistency
  - track SERP rankings and volatility
---

# SEO & Content Marketing Skills Suite

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

A specialized skill suite for SEO and content marketing professionals, providing 10 commands and 5 multi-step workflows for:

- **Keyword research** with clustering and SERP intent mapping
- **Content audits** detecting quality issues, duplication, and cannibalization
- **Technical SEO** covering crawl budget, Core Web Vitals, schema markup
- **Competitor analysis** for backlink gaps, topic gaps, featured snippets
- **Content optimization** with AI-generated briefs and NLP term extraction
- **SERP monitoring** with rank tracking and CTR optimization
- **Link prospecting** with DA/DR filtering and outreach templates
- **Local SEO** including NAP consistency and GMB optimization

All commands follow a consistent 5-step interaction pattern with structured output, progress tracking, and prioritized action plans.

## Installation

```bash
# Clone the repository
git clone https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo.git

# Copy to Claude Code skills directory
mkdir -p ~/.claude/skills
cp -r r15-shanraisshan-claude-code-best-practice-seo ~/.claude/skills/seo-content-marketing/

# Register in Claude Code session
/read ~/.claude/skills/seo-content-marketing/SKILL.md
```

Or install directly in a Claude Code session:

```bash
# From within Claude Code
/install-skill https://github.com/MagicStarfishBoost/r15-shanraisshan-claude-code-best-practice-seo
```

## Core Commands

### 1. Keyword Research

Deep keyword clustering with search intent mapping and opportunity scoring.

```bash
# Basic keyword research
/keyword-research example.com

# With specific seed keywords
/keyword-research example.com --seeds "project management,task tracking"

# Export to CSV
/keyword-research example.com --output csv --min-volume 100
```

**Output structure:**
- Keyword clusters grouped by intent (informational/transactional/navigational)
- Search volume, difficulty score, and opportunity rating
- SERP feature analysis (featured snippets, PAA, video, images)
- Related keywords and LSI terms

### 2. Content Audit

Full-site content quality scoring with duplication and cannibalization detection.

```bash
# Full site audit
/content-audit example.com --scope full

# Audit specific section
/content-audit example.com/blog --scope section

# Quick audit (skip external links)
/content-audit example.com --quick --output md
```

**Checks performed:**
- Thin content detection (< 300 words)
- Duplicate/near-duplicate pages
- Keyword cannibalization
- Missing meta tags (title, description)
- Broken internal/external links
- Content freshness score

### 3. Technical SEO Audit

Comprehensive technical SEO analysis for indexability and performance.

```bash
# Full technical audit
/technical-seo example.com

# Focus on Core Web Vitals
/technical-seo example.com --focus cwv

# Check specific pages
/technical-seo example.com --urls urls.txt
```

**Audit areas:**
- Crawl budget optimization
- Core Web Vitals (LCP, FID, CLS)
- Schema markup validation
- XML sitemap structure
- Robots.txt configuration
- Canonical tag implementation
- Hreflang setup (multilingual)
- HTTPS and security headers

### 4. Competitor Gap Analysis

Identify content and backlink opportunities vs. competitors.

```bash
# Analyze competitor gaps
/competitor-gap example.com --competitors competitor1.com,competitor2.com

# Focus on backlinks only
/competitor-gap example.com --competitors competitor1.com --focus backlinks

# Featured snippet opportunities
/competitor-gap example.com --competitors competitor1.com --focus snippets
```

**Analysis includes:**
- Backlink gap (links they have, you don't)
- Topic gap (keywords they rank for, you don't)
- Featured snippet opportunities
- Content format analysis
- Domain authority comparison

### 5. SEO Content Brief Generator

AI-generated content briefs with optimization targets.

```bash
# Generate brief for target keyword
/content-brief "project management software" --domain example.com

# Include competitor analysis
/content-brief "task tracking tools" --analyze-competitors 5

# Export as template
/content-brief "agile methodology" --template md --output brief.md
```

**Brief components:**
- Target keyword and variants
- Recommended word count (based on top 10 SERP)
- H2/H3 outline structure
- NLP terms to include
- Questions to answer (from PAA)
- Internal linking opportunities
- Image/media recommendations

### 6. SERP Monitor

Daily rank tracking with volatility alerts and CTR optimization.

```bash
# Track keywords
/serp-monitor example.com --keywords keywords.txt

# Get volatility report
/serp-monitor example.com --report volatility --period 30d

# CTR optimization suggestions
/serp-monitor example.com --optimize-ctr
```

**Monitoring features:**
- Daily rank checking (desktop + mobile)
- SERP feature tracking
- Ranking volatility alerts
- CTR vs. position analysis
- Title/meta description A/B test suggestions

### 7. Link Prospecting

Generate quality backlink prospect lists with outreach templates.

```bash
# Find link prospects
/link-prospecting example.com --niche "project management"

# Filter by domain metrics
/link-prospecting example.com --min-da 30 --min-dr 25

# Generate outreach emails
/link-prospecting example.com --niche "saas" --outreach-templates
```

**Prospecting methods:**
- Competitor backlink sources
- Guest post opportunities
- Resource page listings
- Broken link building
- HARO/journalist requests
- Niche directory submissions

### 8. Page Speed SEO Audit

Performance analysis mapped to ranking impact.

```bash
# Analyze page speed
/page-speed-seo example.com/landing-page

# Bulk URL analysis
/page-speed-seo example.com --urls urls.txt

# Mobile-first audit
/page-speed-seo example.com --device mobile --focus lcp
```

**Performance metrics:**
- Render-blocking resources
- Largest Contentful Paint (LCP)
- Cumulative Layout Shift (CLS)
- First Input Delay (FID)
- Time to Interactive (TTI)
- SEO ranking impact score

### 9. Local SEO Audit

NAP consistency, GMB optimization, and local citation audit.

```bash
# Local SEO audit
/local-seo "Business Name" --location "New York, NY"

# Check NAP consistency
/local-seo "Business Name" --check-citations

# GMB optimization
/local-seo "Business Name" --optimize-gmb
```

**Local signals:**
- NAP (Name, Address, Phone) consistency
- Google Business Profile completeness
- Local citation audit (directories)
- Review signals (volume, recency, rating)
- Local keyword rankings
- On-page local optimization

### 10. Content Calendar

Data-driven editorial calendar from search demand and seasonality.

```bash
# Generate content calendar
/content-calendar example.com --niche "project management" --months 3

# Include seasonal trends
/content-calendar example.com --seasonal --region US

# Export to Google Sheets
/content-calendar example.com --output gsheet --share team@example.com
```

**Calendar features:**
- Search volume trends
- Seasonal demand patterns
- Content type recommendations
- Publishing frequency optimization
- Topic clustering
- Internal linking plan

## Multi-Step Workflows

### Full SEO Sprint (12 steps)

Comprehensive SEO audit to implementation.

```bash
/workflows:full-seo-sprint example.com --scope full
```

**Workflow steps:**
1. Technical SEO audit
2. Content quality analysis
3. Keyword research and mapping
4. Competitor gap analysis
5. Backlink profile audit
6. On-page optimization plan
7. Content creation briefs (top 10)
8. Internal linking strategy
9. Schema markup implementation
10. Core Web Vitals fixes
11. Local SEO optimization (if applicable)
12. Monitoring and tracking setup

### Launch SEO Checklist

Pre-launch SEO validation workflow.

```bash
/workflows:launch-seo example.com --pre-launch
```

**Checklist items:**
- Canonical tags configured
- Hreflang setup (if multilingual)
- XML sitemap generated and submitted
- Robots.txt validation
- 301 redirects mapped
- HTTPS enabled
- Core Web Vitals passing
- Schema markup implemented
- Analytics/Search Console connected
- Meta tags populated

### Content Refresh Workflow

Recover lost rankings by refreshing underperforming content.

```bash
/workflows:content-refresh example.com --min-decline 10
```

**Process:**
1. Identify declined pages (position drop > threshold)
2. Analyze SERP changes
3. Content gap analysis vs. current top 10
4. Generate refresh brief
5. Update content
6. Re-optimize on-page elements
7. Add internal links
8. Request re-indexing

### Authority Building Campaign

End-to-end digital PR and link building.

```bash
/workflows:authority-building example.com --duration 90d
```

**Campaign phases:**
1. Link-worthy asset ideation
2. Content creation (linkable assets)
3. Prospect list generation
4. Outreach template creation
5. Email campaign execution
6. Follow-up automation
7. Link acquisition tracking
8. ROI measurement

### AI Content Pipeline

Automated keyword-to-publish workflow.

```bash
/workflows:ai-content-pipeline example.com --keywords keywords.txt
```

**Pipeline stages:**
1. Keyword prioritization
2. Content brief generation
3. AI draft creation
4. SEO optimization
5. Fact-checking and editing
6. Image/media addition
7. Internal linking
8. Schema markup
9. Pre-publish checklist
10. Publication and indexing

## Configuration

Create a `.seoconfig.json` file in your project root:

```json
{
  "domain": "example.com",
  "api_keys": {
    "search_console": "${GOOGLE_SEARCH_CONSOLE_KEY}",
    "ahrefs": "${AHREFS_API_KEY}",
    "semrush": "${SEMRUSH_API_KEY}",
    "screaming_frog": "${SCREAMING_FROG_LICENSE}"
  },
  "preferences": {
    "default_location": "US",
    "default_language": "en",
    "min_keyword_volume": 50,
    "max_keyword_difficulty": 70,
    "min_domain_authority": 30
  },
  "workflows": {
    "full_seo_sprint": {
      "notification_email": "${TEAM_EMAIL}",
      "report_format": "pdf",
      "auto_create_tickets": true
    }
  },
  "integrations": {
    "google_analytics": "${GA_PROPERTY_ID}",
    "google_search_console": "${GSC_PROPERTY_URL}",
    "slack_webhook": "${SLACK_WEBHOOK_URL}"
  }
}
```

Environment variables required:

```bash
# API credentials
export GOOGLE_SEARCH_CONSOLE_KEY="your-key"
export AHREFS_API_KEY="your-key"
export SEMRUSH_API_KEY="your-key"
export SCREAMING_FROG_LICENSE="your-license"

# Notification settings
export TEAM_EMAIL="team@example.com"
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."

# Analytics
export GA_PROPERTY_ID="UA-XXXXXXXX-X"
export GSC_PROPERTY_URL="https://example.com"
```

## Common Patterns

### Pattern 1: New Site SEO Foundation

```bash
# Step 1: Technical foundation
/technical-seo example.com --pre-launch
# Fix critical issues before launch

# Step 2: Keyword research
/keyword-research example.com --competitors competitor1.com,competitor2.com
# Build initial keyword map

# Step 3: Content planning
/content-calendar example.com --months 6 --keywords keyword-map.csv
# Create 6-month editorial calendar

# Step 4: Launch checklist
/workflows:launch-seo example.com
# Validate all pre-launch items
```

### Pattern 2: Traffic Recovery

```bash
# Step 1: Identify problem pages
/serp-monitor example.com --report volatility --period 90d
# Find pages with ranking drops

# Step 2: Content audit
/content-audit example.com --focus declined-pages.txt
# Analyze quality issues

# Step 3: Refresh workflow
/workflows:content-refresh example.com --pages declined-pages.txt
# Systematic refresh process

# Step 4: Re-index
# Request indexing via Search Console
```

### Pattern 3: Competitor Outranking

```bash
# Step 1: Gap analysis
/competitor-gap example.com --competitors top-competitor.com --focus all

# Step 2: Content brief
/content-brief "target keyword" --analyze-competitors 10 --domain example.com

# Step 3: Link prospecting
/link-prospecting example.com --competitor-sources top-competitor.com

# Step 4: Implementation
# Create content, build links, optimize on-page
```

### Pattern 4: Local Business Optimization

```bash
# Step 1: Local audit
/local-seo "Business Name" --location "City, State" --check-citations

# Step 2: GMB optimization
/local-seo "Business Name" --optimize-gmb --generate-posts

# Step 3: Local content
/content-calendar example.com --local --region "City, State" --months 3

# Step 4: Local links
/link-prospecting example.com --local --niche "industry" --location "City, State"
```

### Pattern 5: Enterprise Content Audit

```bash
# Step 1: Full site crawl
/content-audit example.com --scope full --output csv

# Step 2: Technical SEO
/technical-seo example.com --urls all-pages.csv

# Step 3: Page speed
/page-speed-seo example.com --urls high-traffic.csv

# Step 4: Action plan
/workflows:full-seo-sprint example.com --scope full --priority-fixes
```

## Code Examples

### Integrating with Google Search Console

```python
import os
from google.oauth2 import service_account
from googleapiclient.discovery import build

# Authenticate
credentials = service_account.Credentials.from_service_account_file(
    os.getenv('GOOGLE_SERVICE_ACCOUNT_FILE'),
    scopes=['https://www.googleapis.com/auth/webmasters.readonly']
)

# Build Search Console client
webmasters = build('searchconsole', 'v1', credentials=credentials)

# Get search analytics data
site_url = os.getenv('GSC_PROPERTY_URL')
request = {
    'startDate': '2026-04-01',
    'endDate': '2026-04-30',
    'dimensions': ['query', 'page'],
    'rowLimit': 1000
}

response = webmasters.searchanalytics().query(
    siteUrl=site_url,
    body=request
).execute()

# Process ranking data
for row in response.get('rows', []):
    query = row['keys'][0]
    page = row['keys'][1]
    clicks = row['clicks']
    impressions = row['impressions']
    ctr = row['ctr']
    position = row['position']
    
    print(f"{query}: pos {position:.1f}, CTR {ctr:.2%}, {clicks} clicks")
```

### Keyword Clustering Algorithm

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.cluster import KMeans
import pandas as pd

def cluster_keywords(keywords, n_clusters=10):
    """
    Cluster keywords by semantic similarity
    """
    # Vectorize keywords
    vectorizer = TfidfVectorizer(ngram_range=(1, 3))
    X = vectorizer.fit_transform(keywords)
    
    # Cluster
    kmeans = KMeans(n_clusters=n_clusters, random_state=42)
    clusters = kmeans.fit_predict(X)
    
    # Create dataframe
    df = pd.DataFrame({
        'keyword': keywords,
        'cluster': clusters
    })
    
    # Sort by cluster
    return df.sort_values('cluster')

# Usage
keywords = ["project management", "task tracking", "team collaboration", ...]
clustered = cluster_keywords(keywords, n_clusters=5)
print(clustered.groupby('cluster')['keyword'].apply(list))
```

### Content Quality Scoring

```python
import re
from textstat import flesch_reading_ease, flesch_kincaid_grade

def score_content_quality(html_content, target_keyword):
    """
    Score content quality on 100-point scale
    """
    score = 0
    feedback = []
    
    # Extract text
    text = re.sub('<[^<]+?>', '', html_content)
    word_count = len(text.split())
    
    # Word count (20 points)
    if word_count >= 1500:
        score += 20
    elif word_count >= 1000:
        score += 15
        feedback.append("Consider expanding to 1500+ words")
    else:
        score += 10
        feedback.append(f"Thin content: {word_count} words")
    
    # Readability (20 points)
    readability = flesch_reading_ease(text)
    if 60 <= readability <= 70:
        score += 20
    elif 50 <= readability <= 80:
        score += 15
    else:
        score += 10
        feedback.append(f"Readability score: {readability:.1f} (target 60-70)")
    
    # Keyword usage (20 points)
    keyword_count = text.lower().count(target_keyword.lower())
    keyword_density = (keyword_count / word_count) * 100
    if 0.5 <= keyword_density <= 2.0:
        score += 20
    else:
        score += 10
        feedback.append(f"Keyword density: {keyword_density:.2f}% (target 0.5-2.0%)")
    
    # Headers (20 points)
    h2_count = html_content.count('<h2')
    if h2_count >= 3:
        score += 20
    else:
        score += 10
        feedback.append(f"Add more H2 headers (found {h2_count})")
    
    # Images (20 points)
    img_count = html_content.count('<img')
    if img_count >= 3:
        score += 20
    elif img_count >= 1:
        score += 15
    else:
        score += 5
        feedback.append("Add images for better engagement")
    
    return {
        'score': score,
        'grade': 'A' if score >= 90 else 'B' if score >= 75 else 'C' if score >= 60 else 'D',
        'feedback': feedback
    }
```

### SERP Feature Detection

```python
from bs4 import BeautifulSoup
import requests

def analyze_serp_features(keyword, location='US'):
    """
    Detect SERP features for a keyword
    """
    # Use a SERP API (e.g., SerpApi, DataForSEO)
    api_key = os.getenv('SERPAPI_KEY')
    url = f"https://serpapi.com/search.json?q={keyword}&location={location}&api_key={api_key}"
    
    response = requests.get(url)
    data = response.json()
    
    features = {
        'featured_snippet': False,
        'people_also_ask': False,
        'local_pack': False,
        'knowledge_panel': False,
        'images': False,
        'videos': False,
        'news': False,
        'shopping': False
    }
    
    # Check for featured snippet
    if 'featured_snippet' in data:
        features['featured_snippet'] = True
    
    # Check for People Also Ask
    if 'related_questions' in data:
        features['people_also_ask'] = True
    
    # Check for local pack
    if 'local_results' in data:
        features['local_pack'] = True
    
    # Check for knowledge panel
    if 'knowledge_graph' in data:
        features['knowledge_panel'] = True
    
    # Check for image pack
    if 'inline_images' in data:
        features['images'] = True
    
    # Check for video carousel
    if 'inline_videos' in data:
        features['videos'] = True
    
    return features
```

## Troubleshooting

### Issue: Rate limiting from search APIs

**Symptoms:** 429 errors or quota exceeded messages

**Solution:**
```bash
# Configure rate limiting in .seoconfig.json
{
  "api_rate_limits": {
    "requests_per_minute": 10,
    "concurrent_requests": 2
  }
}

# Use batch mode for large audits
/content-audit example.com --batch-size 50 --delay 5
```

### Issue: Incomplete crawl data

**Symptoms:** Missing pages in audit results

**Solution:**
```bash
# Increase crawl depth
/content-audit example.com --depth 5 --follow-external

# Provide sitemap for complete coverage
/content-audit example.com --sitemap https://example.com/sitemap.xml

# Check robots.txt isn't blocking
/technical-seo example.com --validate-robots
```

### Issue: Keyword data not showing search volume

**Symptoms:** Zero or missing search volume

**Solution:**
```bash
# Ensure API keys are configured
export SEMRUSH_API_KEY="your-key"

# Use alternative data source
/keyword-research example.com --data-source ahrefs

# Fall back to trends data
/keyword-research example.com --use-trends
```

### Issue: Slow page speed audits

**Symptoms:** Timeouts or very long execution

**Solution:**
```bash
# Use cached Lighthouse results
/page-speed-seo example.com --use-cache

# Audit fewer URLs
/page-speed-seo example.com --limit 20 --priority high-traffic

# Run in background
/page-speed-seo example.com --async --notify-email ${TEAM_EMAIL}
```

### Issue: Competitor data not available

**Symptoms:** Empty competitor analysis results

**Solution:**
```bash
# Verify competitor domains are accessible
curl -I https://competitor.com

# Use archived data
/competitor-gap example.com --competitors competitor.com --use-archive

# Broaden analysis
/competitor-gap example.com --auto-discover-competitors 10
```

## Best Practices

1. **Run technical audits monthly** to catch crawl errors early
2. **Monitor Core Web Vitals weekly** as they impact rankings
3. **Refresh top-performing content quarterly** to maintain relevance
4. **Track competitors continuously** to identify new opportunities
5. **Validate data sources** by cross-referencing GSC with third-party tools
6. **Document all changes** for correlation with ranking movements
7. **Test meta changes** in low-stakes pages before rolling out site-wide
8. **Build links consistently** rather than in bursts
9. **Prioritize user intent** over exact keyword matching
10. **Measure business outcomes** (conversions, revenue) not just rankings

## Resources

- Source project: [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- Google Search Console API: [developers.google.com/webmaster-tools](https://developers.google.com/webmaster-tools)
- Lighthouse API: [developers.google.com/web/tools/lighthouse](https://developers.google.com/web/tools/lighthouse)
- Schema.org: [schema.org](https://schema.org)
