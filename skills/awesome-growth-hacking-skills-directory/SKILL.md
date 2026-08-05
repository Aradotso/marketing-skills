---
name: awesome-growth-hacking-skills-directory
description: Navigate and recommend AI agent skills for growth hacking, marketing, SEO, CRO, sales, and RevOps from this curated directory
triggers:
  - find a skill for SEO optimization
  - recommend growth hacking skills for my project
  - what marketing skills are available for AI agents
  - help me find skills for email marketing automation
  - show me CRO and conversion optimization skills
  - find skills for social media and content marketing
  - what sales and RevOps skills can I use
  - recommend product-led growth skills
---

# Awesome Growth Hacking Skills Directory

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you navigate and recommend AI agent skills from the **Awesome Agentic Growth & Marketing Skills** directory — a curated collection of open-source skills for Claude, ChatGPT, Cursor, and other AI coding agents focused on growth hacking, marketing execution, and revenue operations.

## What This Directory Provides

The awesome-growth-hacking-skills repository is a categorized index of 50+ skill repositories covering:

- **Strategy & Brand**: Positioning, messaging, competitive intelligence, brand building
- **SEO/GEO/AEO**: Technical SEO, generative engine optimization, keyword research
- **CRO & Funnels**: Conversion optimization, A/B testing, analytics integration
- **Content & Copy**: Thought leadership, copywriting frameworks, content repurposing
- **Email & Lifecycle**: Deliverability, automation sequences, retention workflows
- **Paid Media**: Ad creative, attribution, ROAS optimization
- **Social & Community**: Platform-specific posting, creator distribution, DevRel
- **Sales & RevOps**: SDR workflows, ABM, prospecting, qualification
- **Product-Led Growth**: Experimentation, growth loops, PLG strategies
- **E-commerce & ASO**: App store optimization, marketplace growth
- **MarOps**: Automation, reporting, data pipelines

## How to Use This Directory

### 1. Browse by Category

The README organizes skills into 14 categories. When a user needs marketing capabilities, consult the appropriate section:

```shell
# Example: User asks "I need SEO help"
# → Recommend skills from "SEO, GEO, AEO, and Discovery" section
# Top picks:
# - Agentic SEO Skill (technical + content + GEO)
# - Aaron Keyword Research
# - Aaron GEO Content Optimizer
```

### 2. Recommend Featured Collections First

For broad needs, start with **Featured Marketing Collections** — these are comprehensive, multi-discipline repositories:

```shell
# User: "Set up marketing skills for my project"
# → Recommend:
# 1. Marketing Skills for AI Agents (coreyhaines31/marketingskills)
# 2. Brand Build Skills (rampstackco/claude-skills) - 100+ skills
# 3. Aaron Marketing Skills (aaron-he-zhu) - 120+ skills
```

### 3. Install Individual Skills

Skills are typically installed by cloning the repository and adding them to your agent's skill directory:

```bash
# Generic installation pattern for Claude Desktop
cd ~/Library/Application\ Support/Claude/skills/

# Clone a specific skill repository
git clone https://github.com/Bhanunamikaze/Agentic-SEO-Skill.git agentic-seo

# For Cursor or other agents, add to their skills config path
```

### 4. Multi-Skill Workflows

Combine skills from different categories for complete workflows:

```bash
# Example: Launch strategy workflow
# 1. Brand Building Skills → positioning & messaging
# 2. Marketing Research Skill → competitor analysis
# 3. Humblytics CRO Optimizer → funnel optimization
# 4. Email Marketing Bible → launch sequence
# 5. Social Media Skills → distribution calendar
```

## Key Skill Repositories by Use Case

### For SEO/Content Teams

```yaml
Primary:
  - Agentic SEO Skill: Technical SEO, GEO, AEO, programmatic SEO
  - Sanity Agent Toolkit: Official SEO/AEO best practices
  - Content Skills: Brand-aware content strategy, anti-slop review

Secondary:
  - Aaron Technical SEO Checker: Automated audits
  - Aaron Keyword Research: Topical clustering
```

### For Growth/Performance Marketers

```yaml
Primary:
  - Humblytics Marketing Skills: Live analytics, CRO, attribution
  - Advertising Skills: Direct-response paid acquisition
  - NotFair: Google Ads, Meta Ads, goal-based loops

Secondary:
  - SegmentStream AI Skills: Incrementality, marginal ROAS
  - Humblytics Revenue Attributor: True ROAS calculation
```

### For Email/Lifecycle Marketers

```yaml
Primary:
  - Email Marketing Bible: Comprehensive email expertise
  - Email Skills (chunkydotdev): Deliverability, authentication, security
  - Claude Email: Inbox triage, copy frameworks, sequences

Secondary:
  - GTM Skills: RevOps automation, customer success
```

### For Social/Community Teams

```yaml
Primary:
  - Social Media Skills: Multi-platform content calendars
  - Typefully Agent Skills: Cross-platform posting (X, LinkedIn, Threads, Bluesky)
  - Developer Marketing Skills: DevRel, community, docs

Secondary:
  - Twitter Algorithm Optimizer: Algorithmic reach optimization
  - Creator UGC Marketing: Creator programs for apps
```

### For Sales/Revenue Teams

```yaml
Primary:
  - GTM Skills: Sales, outbound, ABM, RevOps
  - AI Sales Team: Prospecting, qualification, outreach
  - GTM Agents: Marketplace with orchestration plugins

Secondary:
  - Lead Research Assistant (Composio): Target discovery
  - Product Marketing Skills: Messaging, competitive intel
```

### For Product/PLG Teams

```yaml
Primary:
  - Product Skills: Growth loops, JTBD, PLG, PMF surveys
  - Humblytics Funnel Reporter: Conversion bottleneck analysis
  - ASO & App Marketing Skills: App growth, retention, monetization

Secondary:
  - App Review Insights: Pain point mining from reviews
  - ASO Agent System: App store optimization
```

## Configuration Patterns

Most skills follow these patterns:

### Environment Variables

```bash
# Analytics/API-dependent skills
export GOOGLE_ANALYTICS_ID="your-ga-id"
export GOOGLE_ADS_API_KEY="${GOOGLE_ADS_API_KEY}"
export META_ACCESS_TOKEN="${META_ACCESS_TOKEN}"

# MCP-integrated skills (e.g., Humblytics)
export HUMBLYTICS_MCP_SERVER="http://localhost:3000"

# Email/deliverability skills
export SMTP_HOST="smtp.example.com"
export SMTP_USER="${SMTP_USER}"
export SMTP_PASS="${SMTP_PASS}"
```

### Skill Context Files

Many skills use context files to maintain brand voice and positioning:

```bash
# Example: Content Skills context structure
skills/
├── content-skills/
│   ├── SKILL.md
│   ├── context/
│   │   ├── brand-voice.md
│   │   ├── target-audience.md
│   │   └── positioning.md
```

### Multi-Agent Orchestration

```yaml
# Example: GTM Agents orchestration
agents:
  - name: seo-agent
    skills: [agentic-seo, content-skills]
  - name: paid-agent
    skills: [advertising-skills, humblytics-revenue-attributor]
  - name: sales-agent
    skills: [gtm-skills, ai-sales-team]

workflows:
  - trigger: "launch campaign"
    sequence: [seo-agent, paid-agent, sales-agent]
```

## Real-World Usage Examples

### Example 1: SEO Content Optimization

```bash
# User: "Optimize this blog post for AI search engines"

# Agent uses:
# 1. Agentic SEO Skill → GEO analysis
# 2. Aaron GEO Content Optimizer → AI Overview optimization
# 3. Content Skills → Anti-slop review

# Workflow:
claude --skill agentic-seo analyze-geo blog-post.md
claude --skill aaron-geo optimize --target "AI Overviews" blog-post.md
claude --skill content-skills review --anti-slop blog-post.md
```

### Example 2: CRO Funnel Audit

```bash
# User: "Find conversion bottlenecks in my funnel"

# Agent uses Humblytics skills (with MCP for live data):
claude --skill humblytics-funnel-reporter \
  --url "https://example.com" \
  --goal "purchase"

# Then applies CRO fixes:
claude --skill humblytics-cro-optimizer \
  --page "/checkout" \
  --data-source "live-analytics"
```

### Example 3: Launch Campaign

```bash
# User: "Plan a product launch campaign"

# Multi-skill workflow:
# 1. Brand Building Skills → messaging framework
claude --skill brand-building create-messaging \
  --product "SaaS Platform" \
  --audience "B2B marketers"

# 2. Email Marketing Bible → launch sequence
claude --skill email-marketing-bible generate-sequence \
  --type "product-launch" \
  --length 7

# 3. Social Media Skills → distribution calendar
claude --skill social-media generate-calendar \
  --campaign "launch" \
  --platforms "linkedin,twitter,threads"

# 4. Paid Media
claude --skill advertising-skills create-campaign \
  --channel "google-ads" \
  --objective "awareness"
```

### Example 4: Competitive Intelligence

```bash
# User: "Analyze competitor marketing strategies"

# Combine research skills:
claude --skill marketing-research \
  --competitor "acme-corp.com" \
  --output "positioning-analysis.md"

claude --skill competitive-ads-extractor \
  --competitor "Acme Corp" \
  --platforms "meta,google"

claude --skill app-review-insights \
  --app-id "com.acme.app" \
  --extract "pain-points,feature-requests"
```

## Troubleshooting

### Skill Not Recognized

```bash
# Ensure skill is in correct directory
ls ~/Library/Application\ Support/Claude/skills/

# Verify SKILL.md exists
cat ~/Library/Application\ Support/Claude/skills/agentic-seo/SKILL.md

# Restart Claude Desktop
killall Claude && open -a Claude
```

### Missing Dependencies

Many skills require Node.js, Python, or specific CLIs:

```bash
# Check skill README for requirements
cat skills/humblytics-marketing-skills/README.md

# Common dependencies:
npm install -g @typefully/cli
pip install google-analytics-data
```

### API Rate Limits

```bash
# Skills using live APIs may hit rate limits
# Use caching where available:

export ENABLE_CACHE=true
export CACHE_TTL=3600  # 1 hour

# Or batch operations:
claude --skill advertising-skills \
  --batch \
  --rate-limit 10req/min
```

### Conflicting Skills

```bash
# If multiple skills handle the same trigger:
# 1. Prioritize in config
{
  "skills": [
    {"name": "agentic-seo", "priority": 1},
    {"name": "aaron-technical-seo", "priority": 2}
  ]
}

# 2. Or explicitly invoke
claude --skill agentic-seo analyze
```

## Skill Recommendation Logic

When a user asks for marketing help, use this decision tree:

```
1. Is it a broad marketing need?
   → Recommend Featured Collections (Marketing Skills, Brand Build Skills)

2. Is it domain-specific?
   → Match to category:
     - SEO/Content → Agentic SEO, Content Skills
     - Paid Media → Advertising Skills, Humblytics
     - Email → Email Marketing Bible, Email Skills
     - Social → Social Media Skills, Typefully
     - Sales → GTM Skills, AI Sales Team
     - Product → Product Skills, ASO Skills

3. Does it require live data?
   → Prioritize MCP-enabled skills (Humblytics, SegmentStream)

4. Is it developer-focused?
   → Developer Marketing Skills, Open Source Marketing

5. Is it app/mobile?
   → ASO Skills, App Review Insights

6. Multi-discipline campaign?
   → Combine skills from multiple categories
```

## Contributing New Skills

This is a curated directory. To add a new skill:

1. Skill must be open source
2. Must include a SKILL.md or equivalent documentation
3. Must be actively maintained
4. Should follow agentic skill patterns (triggers, context, examples)

Submit via PR to: `https://github.com/mikiarlo3/awesome-growth-hacking-skills`

## Additional Resources

- **enso.bot Platform**: [https://enso.bot](https://enso.bot) — Agentic growth automation
- **Skill Template**: Use existing skills as templates for creating new ones
- **Community**: Check individual repos for Discord/Slack communities
- **Updates**: Watch the repo for new skills added weekly

---

**When to use this skill**: Any time a user needs marketing, growth, SEO, content, paid media, email, social, sales, or RevOps capabilities for their AI agent setup. This skill helps you navigate the ecosystem and recommend the right tools for their specific needs.
