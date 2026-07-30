---
name: genpark-marketing-banner-agent-skill
description: AI agent skill for managing GenPark seasonal promotional web banners with automated scheduling and layout matching
triggers:
  - "set up GenPark marketing banner automation"
  - "schedule seasonal promotional banners"
  - "configure banner rotation for GenPark campaigns"
  - "automate marketing banner deployment"
  - "manage promotional banner layouts"
  - "create seasonal banner schedule"
  - "update GenPark promo banners"
  - "integrate banner agent with marketing calendar"
---

# GenPark Marketing Banner Agent Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the GenPark Marketing Banner Agent, a Python-based system for automating seasonal promotional web banner deployment. The agent matches banner layouts to scheduled campaigns, handles rotation logic, and integrates with marketing calendars.

## What It Does

GenPark Marketing Banner Agent provides:
- **Automated banner scheduling** based on seasonal campaigns and promotional periods
- **Layout matching** to ensure banners fit designated webpage slots
- **Banner rotation** with priority and timing controls
- **Integration hooks** for CMS and marketing automation platforms
- **A/B testing support** for banner variants

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-marketing-banner-agent-skill.git
cd genpark-marketing-banner-agent-skill

# Install dependencies
pip install -r requirements.txt

# Or install as a package
pip install genpark-banner-agent
```

### Dependencies

Common dependencies include:
- `pyyaml` - Configuration management
- `schedule` - Job scheduling
- `pillow` - Image processing
- `requests` - API integrations
- `python-dateutil` - Date/time handling

## Quick Start

```python
from genpark_banner import BannerAgent, BannerSchedule, Layout

# Initialize the banner agent
agent = BannerAgent(
    config_path="config/banners.yaml",
    assets_dir="assets/banners"
)

# Define a seasonal campaign
schedule = BannerSchedule(
    campaign_name="summer_sale_2026",
    start_date="2026-07-01",
    end_date="2026-07-31",
    priority=1
)

# Add banner with layout matching
agent.add_banner(
    schedule=schedule,
    banner_path="assets/banners/summer_sale.png",
    layout=Layout.HERO_WIDE,
    target_pages=["home", "products"]
)

# Start the agent
agent.run()
```

## Configuration

### YAML Configuration

Create `config/banners.yaml`:

```yaml
agent:
  check_interval: 300  # seconds
  timezone: "America/New_York"
  auto_deploy: true

layouts:
  hero_wide:
    width: 1920
    height: 600
    slots: ["home_hero", "landing_hero"]
  
  sidebar:
    width: 300
    height: 600
    slots: ["product_sidebar", "blog_sidebar"]
  
  banner_strip:
    width: 1200
    height: 120
    slots: ["top_strip", "promo_strip"]

campaigns:
  - name: "summer_sale_2026"
    start: "2026-07-01T00:00:00"
    end: "2026-07-31T23:59:59"
    priority: 1
    banners:
      - asset: "summer_sale_hero.png"
        layout: "hero_wide"
        pages: ["home"]
      
      - asset: "summer_sale_sidebar.png"
        layout: "sidebar"
        pages: ["products", "categories"]
  
  - name: "back_to_school"
    start: "2026-08-15T00:00:00"
    end: "2026-09-15T23:59:59"
    priority: 2
    banners:
      - asset: "bts_hero.png"
        layout: "hero_wide"
        pages: ["home", "school-supplies"]

deployment:
  method: "api"  # or "ftp", "s3"
  endpoint: "${BANNER_API_ENDPOINT}"
  auth_token: "${BANNER_API_TOKEN}"
```

### Environment Variables

```bash
# .env file
BANNER_API_ENDPOINT=https://cms.genpark.com/api/v1/banners
BANNER_API_TOKEN=your_api_token_here
BANNER_ASSETS_DIR=/var/www/banners
BANNER_LOG_LEVEL=INFO
```

## Core API

### BannerAgent Class

```python
from genpark_banner import BannerAgent

agent = BannerAgent(
    config_path="config/banners.yaml",
    assets_dir="assets/banners",
    log_level="INFO"
)

# Schedule a new banner
agent.schedule_banner(
    campaign="holiday_sale",
    asset="holiday_hero.png",
    layout="hero_wide",
    start_date="2026-12-01",
    end_date="2026-12-25",
    pages=["home", "gifts"],
    priority=1
)

# Check active banners
active = agent.get_active_banners(page="home")
print(active)

# Force banner update
agent.update_now(page="home")

# Remove scheduled banner
agent.remove_banner(campaign="holiday_sale")
```

### Layout Matching

```python
from genpark_banner import Layout, BannerMatcher

# Define custom layout
custom_layout = Layout(
    name="custom_hero",
    width=1440,
    height=500,
    aspect_ratio="16:9",
    allowed_formats=["png", "jpg", "webp"]
)

# Match banner to layout
matcher = BannerMatcher()
is_match = matcher.validate(
    banner_path="assets/promo.png",
    layout=custom_layout,
    auto_resize=True  # Resize if dimensions don't match
)

if not is_match:
    # Get resized version
    resized_path = matcher.resize_to_layout(
        banner_path="assets/promo.png",
        layout=custom_layout,
        output_path="assets/promo_resized.png"
    )
```

### Scheduling Patterns

```python
from genpark_banner import BannerSchedule, RecurrenceRule

# One-time campaign
schedule = BannerSchedule(
    campaign_name="flash_sale",
    start_date="2026-07-15T10:00:00",
    end_date="2026-07-15T18:00:00",
    priority=1
)

# Recurring weekly banner
weekly_schedule = BannerSchedule(
    campaign_name="weekend_deals",
    recurrence=RecurrenceRule(
        frequency="weekly",
        days=["saturday", "sunday"],
        start_time="00:00:00",
        end_time="23:59:59"
    ),
    start_date="2026-07-01",
    end_date="2026-12-31"
)

agent.add_schedule(weekly_schedule)
```

### A/B Testing

```python
from genpark_banner import ABTest

# Create A/B test variants
ab_test = ABTest(
    campaign="summer_sale",
    variants=[
        {
            "name": "variant_a",
            "asset": "summer_hero_a.png",
            "weight": 50
        },
        {
            "name": "variant_b",
            "asset": "summer_hero_b.png",
            "weight": 50
        }
    ],
    tracking_param="banner_variant"
)

agent.add_ab_test(
    test=ab_test,
    layout="hero_wide",
    pages=["home"],
    start_date="2026-07-01",
    end_date="2026-07-07"
)

# Get variant performance
stats = agent.get_ab_stats(campaign="summer_sale")
print(f"Variant A CTR: {stats['variant_a']['ctr']}")
print(f"Variant B CTR: {stats['variant_b']['ctr']}")
```

## Deployment Integration

### API Deployment

```python
from genpark_banner import APIDeployer

deployer = APIDeployer(
    endpoint=os.getenv("BANNER_API_ENDPOINT"),
    auth_token=os.getenv("BANNER_API_TOKEN")
)

# Deploy banner to CMS
deployer.deploy(
    banner_path="assets/summer_sale.png",
    slot="home_hero",
    metadata={
        "campaign": "summer_sale_2026",
        "start": "2026-07-01T00:00:00",
        "end": "2026-07-31T23:59:59",
        "link_url": "https://genpark.com/summer-sale"
    }
)
```

### S3 Deployment

```python
from genpark_banner import S3Deployer

s3_deployer = S3Deployer(
    bucket=os.getenv("S3_BANNER_BUCKET"),
    region=os.getenv("AWS_REGION"),
    access_key=os.getenv("AWS_ACCESS_KEY_ID"),
    secret_key=os.getenv("AWS_SECRET_ACCESS_KEY")
)

s3_deployer.upload(
    banner_path="assets/summer_sale.png",
    s3_key="banners/active/home_hero.png",
    cache_control="max-age=3600"
)
```

## Common Patterns

### Automated Seasonal Rotation

```python
from genpark_banner import BannerAgent, SeasonalTemplate

agent = BannerAgent(config_path="config/banners.yaml")

# Define seasonal templates
seasons = SeasonalTemplate(
    spring={"start": "03-20", "end": "06-19", "asset": "spring_banner.png"},
    summer={"start": "06-20", "end": "09-21", "asset": "summer_banner.png"},
    fall={"start": "09-22", "end": "12-20", "asset": "fall_banner.png"},
    winter={"start": "12-21", "end": "03-19", "asset": "winter_banner.png"}
)

agent.apply_seasonal_template(
    template=seasons,
    layout="hero_wide",
    pages=["home"]
)

agent.run()
```

### Priority Override

```python
# Normal campaign (priority 3)
agent.schedule_banner(
    campaign="standard_promo",
    asset="standard.png",
    priority=3,
    start_date="2026-07-01",
    end_date="2026-07-31"
)

# High priority flash sale (overrides lower priority)
agent.schedule_banner(
    campaign="flash_sale",
    asset="flash_sale.png",
    priority=1,  # Lower number = higher priority
    start_date="2026-07-15T10:00:00",
    end_date="2026-07-15T18:00:00"
)
```

### Banner Preview & Testing

```python
from genpark_banner import BannerPreview

preview = BannerPreview(agent=agent)

# Preview what will be active on a specific date
upcoming = preview.preview_date(
    date="2026-07-15",
    page="home"
)

print(f"Active banner: {upcoming['campaign']}")
print(f"Asset: {upcoming['asset']}")
print(f"Priority: {upcoming['priority']}")

# Test schedule conflicts
conflicts = preview.check_conflicts(
    start_date="2026-07-01",
    end_date="2026-07-31",
    page="home",
    layout="hero_wide"
)

if conflicts:
    print(f"Warning: {len(conflicts)} scheduling conflicts detected")
    for conflict in conflicts:
        print(f"  - {conflict['campaign']} (priority {conflict['priority']})")
```

## CLI Commands

### Start the Agent

```bash
# Run with default config
python -m genpark_banner run

# Specify config file
python -m genpark_banner run --config config/production.yaml

# Dry run (no deployment)
python -m genpark_banner run --dry-run

# Run once (don't loop)
python -m genpark_banner run --once
```

### Schedule Management

```bash
# List active schedules
python -m genpark_banner list --active

# List all schedules
python -m genpark_banner list --all

# Add banner from CLI
python -m genpark_banner add \
  --campaign "summer_sale" \
  --asset "assets/summer.png" \
  --layout "hero_wide" \
  --start "2026-07-01" \
  --end "2026-07-31" \
  --pages "home,products"

# Remove schedule
python -m genpark_banner remove --campaign "summer_sale"

# Preview schedule
python -m genpark_banner preview --date "2026-07-15" --page "home"
```

### Validation

```bash
# Validate configuration
python -m genpark_banner validate --config config/banners.yaml

# Check banner assets
python -m genpark_banner check-assets

# Test deployment connection
python -m genpark_banner test-deploy
```

## Troubleshooting

### Banner Not Deploying

```python
# Enable debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

agent = BannerAgent(config_path="config/banners.yaml", log_level="DEBUG")

# Check deployment status
status = agent.get_deployment_status()
print(f"Last deployment: {status['last_deployment']}")
print(f"Status: {status['status']}")
print(f"Errors: {status['errors']}")
```

### Layout Mismatch

```python
from genpark_banner import BannerMatcher

matcher = BannerMatcher()

# Detailed validation
result = matcher.validate_detailed(
    banner_path="assets/banner.png",
    layout=Layout.HERO_WIDE
)

if not result['valid']:
    print("Validation failed:")
    for error in result['errors']:
        print(f"  - {error}")
    
    # Auto-fix if possible
    if result['can_auto_fix']:
        fixed_path = matcher.auto_fix(
            banner_path="assets/banner.png",
            layout=Layout.HERO_WIDE,
            output_path="assets/banner_fixed.png"
        )
        print(f"Fixed banner saved to: {fixed_path}")
```

### Schedule Conflicts

```python
# Find overlapping schedules
conflicts = agent.find_schedule_conflicts(
    page="home",
    layout="hero_wide"
)

for conflict in conflicts:
    print(f"Conflict between:")
    print(f"  {conflict['schedule_a']['campaign']} (priority {conflict['schedule_a']['priority']})")
    print(f"  {conflict['schedule_b']['campaign']} (priority {conflict['schedule_b']['priority']})")
    print(f"  Overlap: {conflict['overlap_start']} to {conflict['overlap_end']}")
```

### API Connection Issues

```python
from genpark_banner import APIDeployer

deployer = APIDeployer(
    endpoint=os.getenv("BANNER_API_ENDPOINT"),
    auth_token=os.getenv("BANNER_API_TOKEN"),
    timeout=30,
    retry_attempts=3
)

# Test connection
try:
    deployer.test_connection()
    print("API connection successful")
except Exception as e:
    print(f"Connection failed: {e}")
    # Check endpoint and credentials
```

## Best Practices

1. **Use priority levels wisely**: Reserve priority 1 for urgent/flash campaigns
2. **Test layouts before deployment**: Use `BannerMatcher` to validate dimensions
3. **Monitor deployment logs**: Enable INFO or DEBUG logging in production
4. **Use environment variables**: Never hardcode API tokens or endpoints
5. **Preview schedules**: Check for conflicts before adding new campaigns
6. **Implement rollback**: Keep previous banner versions for quick rollback
7. **Cache assets**: Use CDN or caching for frequently accessed banners

---

This skill enables AI coding agents to effectively work with GenPark's marketing banner automation system, handling scheduling, layout matching, deployment, and troubleshooting for seasonal promotional campaigns.
