---
name: ai-marketing-kit-orchestrator
description: Orchestrate 26 battle-tested marketing skills for AI agents — SEO/GEO, analytics, ads, social automation, video editing, lead magnets, and more
triggers:
  - set up my marketing kit
  - install marketing skills for my project
  - configure ai marketing automation
  - help me with marketing using ai
  - add marketing capabilities to my agent
  - set up SEO and social media automation
  - integrate marketing tools with ai
  - prepare my ai agent for marketing tasks
---

# ai-marketing-kit-orchestrator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

**ai-marketing-kit** is a collection of 26 production-ready marketing skills for AI agents. Each skill is a SKILL.md file that teaches agents how to execute specific marketing tasks — from SEO setup and analytics configuration to video editing, paid ads, and lead generation. Built for Claude Code, Cursor, Codex, and any SKILL.md-compatible agent.

## Installation

### Claude Code (Plugin - Recommended)

```bash
/plugin marketplace add crealwork/ai-marketing-kit
/plugin install ai-marketing-kit@sundayable
```

### Claude Code (Skills Only)

```bash
git clone https://github.com/crealwork/ai-marketing-kit
cp -r ai-marketing-kit/skills/* ~/.claude/skills/
```

### Manual Installation (Any Agent)

```bash
git clone https://github.com/crealwork/ai-marketing-kit
# Copy skills/* to your agent's skills directory
```

After installation, run the **kit-onboarding** skill first:

```
User: "set up my marketing kit"
```

This creates foundational files (`DESIGN.md`, `BRAND-VOICE.md`, `CLAUDE.md`) that all other skills reference for brand consistency.

## Skill Categories

### Groundwork

**kit-onboarding** — Bootstrap brand tokens, voice guidelines, and agent configuration  
**publish-checklist** — Pre-deploy SEO head optimization (favicons, OG tags, titles, canonical)  
**seo-geo-setup** — Search engine registration (Google, Naver, Bing) + GEO for AI search citations  
**analytics-setup** — GA4 + GTM + Clarity configuration with conversion events and UTM tracking  
**crm-connect** — Universal CRM integration (HubSpot, Pipedrive, Close, Attio, Airtable)

### Content Creation

**carousel-generator** — Instagram/Threads card carousels (research → branded design → PNG)  
**ppt-slide-generator** — 16:9 presentation decks with two-stage review  
**print-design** — Posters, flyers, business cards with press-ready PDF output  
**brand-guide** — Extract brand systems from websites or logos  
**humanizer** — Remove AI patterns from English/Korean prose  
**content-repurpose** — Cross-platform content adaptation (Threads ↔ LinkedIn)  
**image-gen** — Marketing images via Higgsfield CLI (gpt-image-2), A/B variants by default  
**thumbnail-maker** — Video thumbnails with 4+ A/B variants, text overlays

### Video Production

**youtube-edit-kit** — Basic editing (silence cuts, captions, SRT, vertical formats) using ffmpeg + faster-whisper  
**longform-to-content** — Long recording → edited video + 4-8 Shorts + thumbnails  
**ad-video** — 15-60s promotional videos with motion graphics, A/B variants required

### Publishing & Distribution

**organic-social** — Multi-platform publishing/scheduling via Zernio  
**paid-ads** — Campaign management across 7 platforms with budget approval gates  
**e-blast-newsletter** — Email campaigns via Resend (3,000/month free tier)  
**b2b-cold-email** — Cold outreach sequences via Instantly.ai  
**lead-magnet** — Create lead magnets with Google Sheets database integration  
**cyrano** — Pre-meeting research briefs with cited sources

### Strategy & Coaching

**dans-advice** — Realistic marketing prescriptions with actionable next steps  
**yc-office-hours** — YC-style validation for ideas and GTM strategies  
**go-viral-or-die** — Viral/stunt marketing ideation  
**first-principles-coach** — Challenge assumptions on pricing, product, growth

## Configuration

### Environment Variables

Set these only for skills you actively use:

```bash
# Email & Outreach
export RESEND_API_KEY="re_xxxxx"              # e-blast-newsletter (free tier)
export INSTANTLY_API_KEY="inst_xxxxx"         # b2b-cold-email

# Social & Ads
export ZERNIO_API_KEY="zern_xxxxx"            # organic-social, paid-ads

# Image Generation
# Run: higgsfield auth login                   # image-gen, thumbnail-maker

# Cyrano Delivery (optional, pick one)
export CYRANO_SLACK_WEBHOOK="https://hooks.slack.com/services/xxx"
export CYRANO_TELEGRAM_TOKEN="bot_xxxxx"
export CYRANO_SMTP_PASS="smtp_password"

# CRM (varies by provider, skill guides setup)
export HUBSPOT_API_KEY="hub_xxxxx"
export PIPEDRIVE_API_TOKEN="pipe_xxxxx"
```

**Never hardcode keys in files.** The kit reads exclusively from environment variables.

### Brand Configuration Files

After running `kit-onboarding`, these files drive brand consistency:

**DESIGN.md** — Brand tokens (colors, fonts, spacing, logo usage)  
**BRAND-VOICE.md** — Tone, vocabulary, content guidelines  
**CLAUDE.md** — Agent configuration and workflow preferences

Skills automatically reference these files. Example from `carousel-generator`:

```python
def load_brand_tokens():
    """Read color palette and fonts from DESIGN.md"""
    with open("DESIGN.md") as f:
        design = f.read()
    
    colors = extract_colors(design)  # {"primary": "#FF5733", ...}
    fonts = extract_fonts(design)    # {"heading": "Inter Bold", ...}
    return colors, fonts
```

## Key Workflows

### First-Time Setup

```python
# 1. Install kit
# 2. User says: "set up my marketing kit"
# 3. kit-onboarding skill activates:

def onboard_marketing_kit():
    questions = [
        "What's your brand name?",
        "Primary brand color (hex)?",
        "Who is your target audience?",
        "Tone: formal, casual, or playful?",
    ]
    
    answers = interview_user(questions)
    
    # Generate foundational files
    create_design_md(answers)
    create_brand_voice_md(answers)
    create_claude_md()
    
    confirm("✓ Marketing kit ready. Try: 'create an Instagram carousel about X'")
```

### Creating Social Content

```python
# User: "create a 5-slide carousel about our new feature"
# carousel-generator activates:

def generate_carousel(topic, slide_count=5):
    # 1. Research phase
    research = gather_research(topic)
    
    # 2. Load brand
    colors, fonts = load_brand_tokens()
    voice = load_brand_voice()
    
    # 3. Generate slides
    slides = []
    for i in range(slide_count):
        content = write_slide_copy(research, i, voice)
        design = design_slide(content, colors, fonts)
        slides.append(design)
    
    # 4. Review gate
    show_preview(slides)
    if user_approves():
        export_slides_to_png(slides)
        return "✓ Carousel saved to outputs/carousel_*.png"
```

### Setting Up Analytics

```python
# User: "set up analytics for my site"
# analytics-setup activates:

def setup_analytics(domain):
    steps = []
    
    # GA4
    ga4_id = create_ga4_property(domain)
    ga4_snippet = generate_ga4_code(ga4_id)
    steps.append(f"Add to <head>:\n{ga4_snippet}")
    
    # GTM
    gtm_id = create_gtm_container(domain)
    gtm_snippet = generate_gtm_code(gtm_id)
    steps.append(f"Add GTM:\n{gtm_snippet}")
    
    # Clarity
    clarity_id = setup_clarity(domain)
    steps.append(f"Clarity project: {clarity_id}")
    
    # Conversion events
    events = ["sign_up", "purchase", "trial_start"]
    configure_ga4_events(ga4_id, events)
    
    return format_installation_guide(steps)
```

### Publishing Content (with approval gates)

```python
# User: "post this to Instagram and LinkedIn"
# organic-social activates:

def publish_multi_platform(content, platforms):
    import os
    import requests
    
    # 1. Show preview
    preview = format_preview(content, platforms)
    show(preview)
    
    # 2. APPROVAL GATE (required for anything that leaves the machine)
    approval = ask_user("Publish to Instagram and LinkedIn? (yes/no)")
    if approval.lower() != "yes":
        return "❌ Publish cancelled"
    
    # 3. Execute via Zernio
    api_key = os.getenv("ZERNIO_API_KEY")
    headers = {"Authorization": f"Bearer {api_key}"}
    
    for platform in platforms:
        payload = {
            "platform": platform,
            "content": content["text"],
            "media_urls": content.get("images", []),
            "schedule": content.get("schedule")  # None = publish now
        }
        
        response = requests.post(
            "https://api.zernio.com/v1/publish",
            json=payload,
            headers=headers
        )
        
        if response.status_code != 200:
            return f"❌ {platform} failed: {response.text}"
    
    return "✓ Published to Instagram, LinkedIn"
```

### Running Paid Ads (with budget approval)

```python
# User: "run a Facebook ad for our landing page"
# paid-ads activates:

def create_paid_ad_campaign(platform, objective, target_url):
    # 1. Generate A/B creative variants (required)
    creatives = generate_ad_creatives(objective, count=2)
    
    # 2. Show preview + budget
    preview = format_ad_preview(creatives, platform)
    show(preview)
    
    # 3. BUDGET APPROVAL GATE (required for spend actions)
    budget_info = ask_user("Budget per day (USD)? Duration (days)?")
    daily_budget = budget_info["daily"]
    duration = budget_info["duration"]
    total_spend = daily_budget * duration
    
    confirmation = ask_user(
        f"Spend ${total_spend} on {platform} over {duration} days? (yes/no)"
    )
    if confirmation.lower() != "yes":
        return "❌ Campaign cancelled"
    
    # 4. Create campaign via Zernio
    api_key = os.getenv("ZERNIO_API_KEY")
    campaign_id = create_campaign_via_api(
        api_key, platform, creatives, daily_budget, duration, target_url
    )
    
    return f"✓ Campaign {campaign_id} live on {platform}"
```

## Safety Rules (Kit-Wide)

All skills enforce these constraints:

1. **Spend approval** — Any action that costs money (ads, budget changes) requires explicit user approval with platform + budget + duration
2. **Publish approval** — Any action that leaves the machine (sends, posts, activations) needs an explicit "go"
3. **No blind retries** — On timeouts, list results first; never auto-retry (prevents double-charges, double-posts)
4. **Image policy** — All image/video generation uses Higgsfield CLI (gpt-image-2); no silent fallbacks; A/B variants for performance visuals

Example enforcement:

```python
def enforce_approval_gate(action_type, details):
    if action_type in ["spend", "publish", "send"]:
        preview = format_preview(details)
        show(preview)
        
        response = ask_user(f"Proceed with {action_type}? (yes/no)")
        if response.lower() != "yes":
            raise UserCancelled(f"{action_type} cancelled by user")
    
    return True
```

## Common Patterns

### Reading Brand Files

```python
def load_brand_context():
    """Load brand tokens and voice for consistent output"""
    design = read_file("DESIGN.md")
    voice = read_file("BRAND-VOICE.md")
    
    return {
        "colors": extract_colors(design),
        "fonts": extract_fonts(design),
        "tone": extract_tone(voice),
        "vocabulary": extract_vocabulary(voice),
    }
```

### Generating A/B Variants

```python
def generate_ab_variants(base_content, count=2):
    """Performance content (ads, thumbnails) always ships as A/B sets"""
    variants = []
    
    for i in range(count):
        variant = create_variant(base_content, variation_seed=i)
        variants.append(variant)
    
    show_comparison(variants)
    return variants
```

### External API Calls (with error handling)

```python
import os
import requests

def call_external_api(endpoint, payload):
    api_key = os.getenv("REQUIRED_API_KEY")
    if not api_key:
        return "❌ Missing REQUIRED_API_KEY environment variable"
    
    try:
        response = requests.post(
            endpoint,
            json=payload,
            headers={"Authorization": f"Bearer {api_key}"},
            timeout=30
        )
        response.raise_for_status()
        return response.json()
    
    except requests.Timeout:
        return "❌ API timeout — list current state, do not retry blindly"
    except requests.HTTPError as e:
        return f"❌ API error: {e.response.status_code} {e.response.text}"
```

## Troubleshooting

### Skills Not Loading

```bash
# Verify skills directory
ls ~/.claude/skills/

# Should see: kit-onboarding.md, carousel-generator.md, etc.

# Re-copy if missing
cp -r ai-marketing-kit/skills/* ~/.claude/skills/
```

### Missing Environment Variables

```python
# Skills fail gracefully with clear errors:
# "❌ Missing RESEND_API_KEY — set via: export RESEND_API_KEY=re_xxxxx"

# Check loaded vars
echo $RESEND_API_KEY
echo $ZERNIO_API_KEY
```

### Brand Files Not Found

```bash
# Run onboarding first
# User: "set up my marketing kit"

# Or create manually:
touch DESIGN.md BRAND-VOICE.md CLAUDE.md
```

### Image Generation Failures

```bash
# Authenticate Higgsfield CLI
higgsfield auth login

# Test generation
higgsfield generate "test image" --model gpt-image-2

# Skills will report: "❌ Higgsfield CLI failed: [error details]"
# No silent fallback to other image sources
```

### Approval Gates Not Triggering

Skills must explicitly call approval for spend/publish actions. If bypassed:

```python
# Correct pattern:
enforce_approval_gate("publish", preview_data)
execute_publish()

# Incorrect (will be caught in review):
execute_publish()  # ❌ Missing approval gate
```

## Extending the Kit

### Adding Custom Skills

Create `custom-skill.md` in `~/.claude/skills/`:

```markdown
---
name: custom-marketing-automation
description: Your custom marketing task
triggers:
  - do my custom task
---

# custom-marketing-automation

## Implementation

```python
def custom_task():
    # Load brand context like other skills
    brand = load_brand_context()
    
    # Your logic here
    ...
    
    # Enforce approval for external actions
    enforce_approval_gate("publish", preview)
```
```

### Overriding Brand Tokens

Edit `DESIGN.md` directly:

```markdown
# DESIGN.md

## Colors
- Primary: #FF5733
- Secondary: #3498DB

## Fonts
- Heading: Inter Bold
- Body: Inter Regular
```

Skills auto-reload on file changes.

## License

MIT — use, fork, or hand to your agent.

---

**Built by [Sundayable](https://www.sundayable.com)** — AI + Revenue Growth Team for Small Business
