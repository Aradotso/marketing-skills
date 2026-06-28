---
name: aaron-marketing-skills
description: Use 38 AI marketing skills for SEO/GEO optimization and influencer marketing campaigns with Claude Code integration
triggers:
  - help me optimize SEO for my website
  - find influencers for my marketing campaign
  - analyze keyword opportunities for content
  - audit my page for search engine optimization
  - plan an influencer marketing campaign
  - research competitors and content gaps
  - generate schema markup for my content
  - track keyword rankings and performance
---

# Aaron Marketing Skills

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Aaron Marketing Skills is a comprehensive collection of 38 marketing skills designed for AI agents (Claude Code, Cursor, etc.) covering two main disciplines: **SEO/GEO** (Search Engine + Generative Engine Optimization) with 20 skills, and **Influencer Marketing (IMPACT)** with 18 skills. The project includes 5 slash commands, three evaluation frameworks (CORE-EEAT, CITE, C³), and supports zero-dependency Markdown skill execution.

## Installation

### Claude Code

```bash
# Register the marketplace plugin
/plugin marketplace add aaron-he-zhu/aaron-marketing-skills

# Install and enable the skills
/plugin install aaron-marketing@aaron
```

### Generic Agent Skills Hosts (skills.sh)

```bash
# Install all skills
npx skills add aaron-he-zhu/aaron-marketing-skills

# Install a single skill
npx skills add aaron-he-zhu/aaron-marketing-skills -s keyword-research
```

### Manual Installation

```bash
git clone https://github.com/aaron-he-zhu/aaron-marketing-skills
cd aaron-marketing-skills
```

## Core Commands

The project provides five main slash commands for workflow automation:

### 1. Auto Command (Intent Routing)

```bash
# Auto-detect intent and run appropriate workflow
/aaron-marketing:auto audit https://example.com/blog/post

# Deep/exhaustive analysis mode
/aaron-marketing:auto --deep analyze my SaaS competitor landscape
```

### 2. Research Command

```bash
# Keyword research and competitive analysis
/aaron-marketing:research [topic/url]

# Example: Research for a specific topic
/aaron-marketing:research "project management software for remote teams"
```

Runs: `keyword-research` → `competitor-analysis` → `serp-analysis` → `content-gap-analysis`

### 3. Create Command

```bash
# Generate content with SEO optimization
/aaron-marketing:create --brief [topic]
/aaron-marketing:create --series [topic]
/aaron-marketing:create --refresh [url]
/aaron-marketing:create --publish [content]
/aaron-marketing:create --meta [url]
/aaron-marketing:create --schema [url]

# Example: Create a content brief
/aaron-marketing:create --brief "Best practices for remote team collaboration"

# Example: Generate schema markup
/aaron-marketing:create --schema https://example.com/products/widget
```

### 4. Audit Command

```bash
# Full audit (on-page + technical + quality + authority)
/aaron-marketing:audit --full [url]

# Specific audit types
/aaron-marketing:audit --tech [url]
/aaron-marketing:audit --visibility [url]
/aaron-marketing:audit --authority [domain]

# Example: Full page audit
/aaron-marketing:audit --full https://example.com/blog/ultimate-guide
```

### 5. Track Command

```bash
# Set up ranking alerts
/aaron-marketing:track --alert [keywords]

# Generate performance report
/aaron-marketing:track --report [timeframe]

# Store project memory
/aaron-marketing:track --remember [context]

# Example: Track keyword rankings
/aaron-marketing:track --alert "saas analytics, team dashboard, project metrics"
```

## Key Skills by Category

### SEO/GEO Skills (20 skills)

#### Research Phase

```python
# keyword-research skill
# Discovers search demand, intent, and opportunity

# Natural trigger examples:
"Research keywords for my SaaS product targeting small teams"
"Find keyword opportunities in the project management niche"

# Manual invocation in Python-based hosts
from skills import run_skill

result = run_skill("keyword-research", {
    "seed_keywords": ["project management", "team collaboration"],
    "target_audience": "remote teams, 10-50 employees",
    "intent_filter": ["informational", "commercial"]
})
```

```python
# competitor-analysis skill
# Analyzes competitor content, backlinks, and rankings

result = run_skill("competitor-analysis", {
    "competitors": ["asana.com", "monday.com", "clickup.com"],
    "focus_urls": ["/blog", "/features"],
    "metrics": ["content_gaps", "backlink_sources", "keyword_overlap"]
})
```

#### Build Phase

```python
# seo-content-writer skill
# Creates optimized content following EEAT principles

result = run_skill("seo-content-writer", {
    "topic": "How to improve team productivity with async communication",
    "target_keywords": ["async communication", "team productivity"],
    "content_type": "pillar_post",
    "word_count": 2500,
    "tone": "professional, helpful"
})
```

```python
# schema-markup-generator skill
# Generates structured data for search engines

result = run_skill("schema-markup-generator", {
    "url": "https://example.com/products/enterprise-plan",
    "schema_types": ["Product", "Offer", "AggregateRating"],
    "product_data": {
        "name": "Enterprise Plan",
        "price": "299",
        "currency": "USD"
    }
})
```

#### Optimize Phase

```python
# content-quality-auditor skill
# 80-item CORE-EEAT quality gate

result = run_skill("content-quality-auditor", {
    "content_url": "https://example.com/blog/guide",
    "benchmark": "CORE-EEAT",
    "threshold": 75  # Minimum score to pass
})
```

```python
# on-page-seo-auditor skill
# Checks title, meta, headings, internal links, images

result = run_skill("on-page-seo-auditor", {
    "url": "https://example.com/blog/post",
    "target_keyword": "project management best practices",
    "check_items": ["title", "meta_description", "h1", "keyword_density", "internal_links"]
})
```

### Influencer Marketing Skills (18 skills)

#### Insight Phase

```python
# audience-analyzer skill
# Analyzes target audience demographics, psychographics, behavior

result = run_skill("audience-analyzer", {
    "product": "Organic skincare line for Gen Z",
    "platforms": ["TikTok", "Instagram"],
    "analysis_depth": "deep"  # Includes psychographics and pain points
})
```

#### Map Phase

```python
# influencer-discovery skill
# Finds creators matching campaign criteria

result = run_skill("influencer-discovery", {
    "niche": "sustainable fashion",
    "platforms": ["Instagram", "YouTube"],
    "follower_range": [10000, 100000],
    "engagement_min": 3.5,
    "location": "US, UK, Canada"
})
```

```python
# fit-scorer skill
# Scores influencers on C³ ACE framework (Audience, Content, Engagement)

result = run_skill("fit-scorer", {
    "influencers": ["@creator1", "@creator2"],
    "campaign_brief": {
        "product": "eco-friendly water bottle",
        "target_age": "18-35",
        "values": ["sustainability", "active lifestyle"]
    },
    "framework": "C3_ACE"
})
```

#### Plan Phase

```python
# campaign-planner skill
# Creates end-to-end influencer campaign strategy

result = run_skill("campaign-planner", {
    "goal": "Launch new product line with 50K reach",
    "budget": 15000,
    "duration": "60 days",
    "platforms": ["Instagram", "TikTok"],
    "deliverables": ["feed_posts", "stories", "reels"]
})
```

```python
# brief-generator skill
# Generates creator briefs with guidelines and requirements

result = run_skill("brief-generator", {
    "campaign_id": "summer_launch_2024",
    "creator_tier": "micro",
    "content_type": "Instagram Reel",
    "key_messages": ["sustainability", "convenience", "style"],
    "compliance": ["FTC_disclosure", "brand_safety"]
})
```

#### Activate Phase

```python
# outreach-manager skill
# Manages influencer outreach sequences

result = run_skill("outreach-manager", {
    "influencer_list": ["creator1@email.com", "creator2@email.com"],
    "template": "collaboration_invite",
    "personalization": {
        "creator1@email.com": {"recent_post": "Your reel about sustainable living"}
    },
    "follow_up_days": [3, 7]
})
```

#### Track Phase

```python
# performance-analyzer skill
# Tracks campaign metrics across platforms

result = run_skill("performance-analyzer", {
    "campaign_id": "summer_launch_2024",
    "metrics": ["reach", "engagement_rate", "conversions", "ROI"],
    "breakdown": ["by_creator", "by_platform", "by_content_type"]
})
```

```python
# roi-calculator skill
# Calculates influencer marketing ROI

result = run_skill("roi-calculator", {
    "campaign_spend": 15000,
    "revenue_generated": 47500,
    "attribution_model": "last_click",
    "include_metrics": ["CPM", "CPC", "CPA", "ROAS"]
})
```

## Cross-Cutting Protocol Skills

Four skills form the protocol layer across all marketing activities:

```python
# content-quality-auditor
# 80-item CORE-EEAT benchmark
result = run_skill("content-quality-auditor", {
    "content_url": "https://example.com/article",
    "audit_type": "CORE-EEAT",
    "output_format": "scorecard"
})

# domain-authority-auditor
# 40-item CITE trust benchmark
result = run_skill("domain-authority-auditor", {
    "domain": "example.com",
    "audit_type": "CITE",
    "compare_to": ["competitor1.com", "competitor2.com"]
})

# entity-optimizer
# Manages canonical entity profiles (brand, author, organization)
result = run_skill("entity-optimizer", {
    "entity_type": "Organization",
    "entity_name": "Acme Corp",
    "attributes": {
        "sameAs": ["https://twitter.com/acmecorp", "https://linkedin.com/company/acme"],
        "expertise": ["B2B SaaS", "Project Management"]
    }
})

# memory-management
# HOT/WARM/COLD project memory storage
result = run_skill("memory-management", {
    "action": "capture",
    "tier": "HOT",
    "context": {
        "project": "Q4_content_campaign",
        "data": {"keywords": [...], "competitors": [...]}
    }
})
```

## Configuration

Skills use environment variables for optional tool connectors:

```bash
# API keys for enhanced data sources (Tier 2+ connectors)
export SERP_API_KEY="your_serpapi_key"
export AHREFS_API_KEY="your_ahrefs_key"
export SEMRUSH_API_KEY="your_semrush_key"

# Social platform APIs for influencer data
export INSTAGRAM_ACCESS_TOKEN="your_instagram_token"
export TIKTOK_API_KEY="your_tiktok_key"
export YOUTUBE_API_KEY="your_youtube_key"

# Analytics integration
export GOOGLE_ANALYTICS_CREDENTIALS="path/to/credentials.json"
```

**Important**: All skills work at **Tier 1 (user-provided data)** without any API keys. Connectors enhance automation but are optional.

## Common Patterns

### Pattern 1: Full SEO Audit Workflow

```python
# Step 1: Research keywords
keywords = run_skill("keyword-research", {
    "seed": ["project management software"]
})

# Step 2: Analyze competitors
competitors = run_skill("competitor-analysis", {
    "keywords": keywords["top_keywords"]
})

# Step 3: Find content gaps
gaps = run_skill("content-gap-analysis", {
    "your_domain": "example.com",
    "competitors": competitors["top_competitors"]
})

# Step 4: Create optimized content
content = run_skill("seo-content-writer", {
    "topic": gaps["priority_topics"][0],
    "target_keywords": gaps["opportunity_keywords"]
})

# Step 5: Run quality audit (80-item CORE-EEAT)
audit = run_skill("content-quality-auditor", {
    "content": content["draft"],
    "benchmark": "CORE-EEAT",
    "threshold": 75
})

# Step 6: Generate schema markup
schema = run_skill("schema-markup-generator", {
    "content": content["final"],
    "schema_types": ["Article", "FAQPage"]
})
```

### Pattern 2: Influencer Campaign Setup

```python
# Step 1: Analyze target audience
audience = run_skill("audience-analyzer", {
    "product": "Sustainable fashion brand",
    "platforms": ["Instagram", "TikTok"]
})

# Step 2: Discover influencers
creators = run_skill("influencer-discovery", {
    "niche": "sustainable fashion",
    "audience_match": audience["demographics"],
    "follower_range": [10000, 100000]
})

# Step 3: Score creator fit (C³ ACE framework)
scored = run_skill("fit-scorer", {
    "influencers": creators["matches"],
    "campaign_criteria": audience["psychographics"],
    "framework": "C3_ACE"
})

# Step 4: Plan campaign
campaign = run_skill("campaign-planner", {
    "selected_creators": scored["top_10"],
    "budget": 20000,
    "goal": "brand_awareness"
})

# Step 5: Generate creator briefs
briefs = run_skill("brief-generator", {
    "campaign": campaign["plan"],
    "per_creator": True,
    "include_compliance": ["FTC_disclosure"]
})

# Step 6: Track performance
performance = run_skill("performance-analyzer", {
    "campaign_id": campaign["id"],
    "metrics": ["reach", "engagement", "conversions"]
})
```

### Pattern 3: Memory-Backed Multi-Phase Project

```python
# Capture research in HOT memory
run_skill("memory-management", {
    "action": "capture",
    "tier": "HOT",
    "context": {
        "project": "Q1_SEO_Campaign",
        "phase": "research",
        "data": {
            "keywords": [...],
            "competitors": [...]
        }
    }
})

# Later: Query memory for context
memory = run_skill("memory-management", {
    "action": "query",
    "project": "Q1_SEO_Campaign",
    "phase": "research"
})

# Promote important items to WARM for long-term retention
run_skill("memory-management", {
    "action": "promote",
    "from_tier": "HOT",
    "to_tier": "WARM",
    "filter": {"importance": "high"}
})
```

## Evaluation Frameworks

### CORE-EEAT (80-item content quality)

```python
# Run comprehensive content quality audit
audit = run_skill("content-quality-auditor", {
    "content_url": "https://example.com/article",
    "framework": "CORE-EEAT",
    "categories": [
        "Core_Content",      # 20 items
        "Originality",       # 15 items
        "Relevance",         # 15 items
        "Expertise",         # 10 items
        "Experience",        # 10 items
        "Authoritativeness", # 5 items
        "Trustworthiness"    # 5 items
    ],
    "pass_threshold": 75
})

# Output includes per-category scores, blockers, and recommendations
```

### CITE (40-item domain authority)

```python
# Audit domain trust signals
domain_audit = run_skill("domain-authority-auditor", {
    "domain": "example.com",
    "framework": "CITE",
    "categories": [
        "Credibility",       # 10 items - author expertise, citations
        "Infrastructure",    # 10 items - technical health, security
        "Transparency",      # 10 items - about pages, contact info
        "Engagement"         # 10 items - social proof, user signals
    ],
    "compare_to": ["competitor.com"]
})
```

### C³ (Influencer Creator/Content/Campaign)

```python
# Score influencer fit using C³ framework
c3_score = run_skill("fit-scorer", {
    "influencer": "@creator_handle",
    "framework": "C3",
    "dimensions": {
        "ACE": {  # Creator fit
            "Audience_alignment": True,
            "Content_quality": True,
            "Engagement_authenticity": True
        },
        "ART": {  # Content quality
            "Authenticity": True,
            "Relevance": True,
            "Technique": True
        },
        "ROI": {  # Campaign performance
            "Reach": True,
            "Outcomes": True,
            "Investment_efficiency": True
        }
    }
})
```

## Troubleshooting

### Skills not loading in Claude Code

```bash
# Verify plugin is registered
/plugin list

# Re-install if needed
/plugin uninstall aaron-marketing@aaron
/plugin install aaron-marketing@aaron

# Check marketplace registration
/plugin marketplace list
```

### Command not found

```bash
# Ensure you're using the new namespace
# Old: /seo:audit or /aaron-seo-geo:audit
# New: /aaron-marketing:audit

# The auto command can recover old syntax
/aaron-marketing:auto /seo:audit https://example.com
```

### Skill returns incomplete data

```bash
# Most skills work at Tier 1 (user-provided data)
# If you need automated data fetching, configure connectors:

export SERP_API_KEY="your_key"
export AHREFS_API_KEY="your_key"

# Check CONNECTORS.md for tier documentation
```

### Memory queries return empty results

```python
# Verify memory tier and project name
result = run_skill("memory-management", {
    "action": "query",
    "tier": "HOT",  # Try "WARM" or "COLD" if not in HOT
    "project": "exact_project_name"  # Case-sensitive
})

# List all stored memory contexts
result = run_skill("memory-management", {
    "action": "list",
    "tier": "all"
})
```

### Quality audit fails with low scores

```python
# Review detailed breakdown
audit = run_skill("content-quality-auditor", {
    "content_url": "url",
    "framework": "CORE-EEAT",
    "verbose": True  # Shows per-item scores and failures
})

# Focus on blockers first (items scoring < 2)
blockers = audit["blockers"]

# Use content-refresher skill to address specific issues
refreshed = run_skill("content-refresher", {
    "url": "url",
    "focus_areas": blockers["categories"]
})
```

### Influencer discovery returns no matches

```python
# Widen search criteria
result = run_skill("influencer-discovery", {
    "niche": "broader_category",
    "follower_range": [5000, 500000],  # Wider range
    "engagement_min": 2.0,  # Lower threshold
    "location": "any"  # Remove geo restrictions
})

# Try alternative platforms
result = run_skill("influencer-discovery", {
    "platforms": ["Instagram", "YouTube", "TikTok"],  # Expand platforms
    "niche": "related_niche"
})
```

## Advanced Usage

### Custom Skill Workflows

Create Python scripts to chain skills:

```python
#!/usr/bin/env python3
# custom_seo_workflow.py

from skills import run_skill

def full_seo_workflow(topic, domain):
    """Complete SEO workflow from research to publish."""
    
    # Research
    kw = run_skill("keyword-research", {"seed": [topic]})
    comp = run_skill("competitor-analysis", {"keywords": kw["top_keywords"]})
    gaps = run_skill("content-gap-analysis", {
        "your_domain": domain,
        "competitors": comp["domains"]
    })
    
    # Create
    content = run_skill("seo-content-writer", {
        "topic": gaps["priority_topics"][0],
        "keywords": gaps["opportunity_keywords"]
    })
    
    # Quality gate
    audit = run_skill("content-quality-auditor", {
        "content": content["draft"],
        "threshold": 75
    })
    
    if audit["score"] < 75:
        # Refresh until passing
        content = run_skill("content-refresher", {
            "content": content["draft"],
            "focus": audit["low_scoring_areas"]
        })
    
    # Optimize
    meta = run_skill("meta-tags-optimizer", {"content": content["final"]})
    schema = run_skill("schema-markup-generator", {"content": content["final"]})
    
    return {
        "content": content["final"],
        "meta": meta,
        "schema": schema,
        "quality_score": audit["score"]
    }

# Run workflow
result = full_seo_workflow("remote work tools", "example.com")
print(result)
```

### Bash Command Wrapper

Commands in `commands/` are Bash scripts calling skill runners:

```bash
#!/usr/bin/env bash
# commands/aaron-marketing-audit.sh

# Full audit mode
if [[ "$1" == "--full" ]]; then
    URL="$2"
    
    # On-page audit
    ./skills/optimize/on-page-seo-auditor/run.sh "$URL"
    
    # Quality audit (CORE-EEAT)
    ./skills/cross-cutting/content-quality-auditor/run.sh "$URL" --framework CORE-EEAT
    
    # Technical audit
    ./skills/optimize/technical-seo-checker/run.sh "$URL"
    
    # Domain authority (CITE)
    DOMAIN=$(echo "$URL" | awk -F/ '{print $3}')
    ./skills/cross-cutting/domain-authority-auditor/run.sh "$DOMAIN" --framework CITE
fi
```

## Reference Documentation

- **CORE-EEAT**: `references/core-eeat-benchmark.md` — 80-item content quality framework
- **CITE**: `references/cite-domain-rating.md` — 40-item domain authority framework
- **C³**: `references/c3-benchmark.md` — Influencer scoring (ACE/ART/ROI)
- **Skill Contract**: `references/skill-contract.md` — Standard skill structure
- **State Model**: `references/state-model.md` — Memory tier semantics
- **Connectors**: `CONNECTORS.md` — Optional API integration tiers

## License

Apache License 2.0 — See [LICENSE](https://github.com/aaron-he-zhu/aaron-marketing-skills/blob/main/LICENSE)
