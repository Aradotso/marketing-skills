---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate unified ROAS across marketing channels
triggers:
  - join ad data with sales data
  - calculate cross-channel ROAS
  - merge programmatic advertising with inventory
  - connect marketing spend to retail performance
  - unify ad platform data with e-commerce
  - combine Google Ads Facebook Ads with Shopify data
  - aggregate marketing attribution across channels
  - reconcile advertising costs with actual revenue
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python library that consolidates programmatic advertising data from multiple platforms (Google Ads, Facebook Ads, TikTok, etc.) with retail sales and inventory data to compute unified Return on Ad Spend (ROAS) metrics. It handles data normalization, attribution modeling, and cross-channel reconciliation to provide accurate marketing performance insights.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt

# Or install as a package
pip install genpark-marketing-joiner
```

## Core Concepts

### Data Sources
- **Programmatic Ad Platforms**: Google Ads, Facebook Ads, TikTok Ads, LinkedIn Ads
- **Retail Systems**: Shopify, WooCommerce, BigCommerce, custom e-commerce
- **Inventory Systems**: Internal SKU databases, warehouse management systems
- **Attribution Windows**: First-touch, last-touch, multi-touch attribution

### Key Metrics
- **ROAS**: Revenue / Ad Spend
- **Attributed Revenue**: Sales linked to specific campaigns
- **Cross-Channel Attribution**: Revenue distributed across multiple touchpoints

## Basic Usage

```python
from genpark_joiner import MarketingDataJoiner, AdPlatform, RetailSource

# Initialize the joiner
joiner = MarketingDataJoiner(
    attribution_window_days=7,
    attribution_model='linear'  # Options: 'first_touch', 'last_touch', 'linear', 'time_decay'
)

# Connect ad platforms
joiner.add_ad_platform(
    platform=AdPlatform.GOOGLE_ADS,
    credentials={
        'developer_token': os.getenv('GOOGLE_ADS_DEVELOPER_TOKEN'),
        'client_id': os.getenv('GOOGLE_ADS_CLIENT_ID'),
        'client_secret': os.getenv('GOOGLE_ADS_CLIENT_SECRET'),
        'refresh_token': os.getenv('GOOGLE_ADS_REFRESH_TOKEN'),
        'customer_id': os.getenv('GOOGLE_ADS_CUSTOMER_ID')
    }
)

joiner.add_ad_platform(
    platform=AdPlatform.FACEBOOK_ADS,
    credentials={
        'access_token': os.getenv('FACEBOOK_ACCESS_TOKEN'),
        'account_id': os.getenv('FACEBOOK_AD_ACCOUNT_ID')
    }
)

# Connect retail/e-commerce data
joiner.add_retail_source(
    source=RetailSource.SHOPIFY,
    credentials={
        'shop_url': os.getenv('SHOPIFY_SHOP_URL'),
        'api_key': os.getenv('SHOPIFY_API_KEY'),
        'password': os.getenv('SHOPIFY_PASSWORD')
    }
)

# Fetch and join data for date range
from datetime import datetime, timedelta

end_date = datetime.now()
start_date = end_date - timedelta(days=30)

unified_data = joiner.fetch_and_join(
    start_date=start_date,
    end_date=end_date
)

# Calculate unified ROAS
roas_report = joiner.calculate_roas(unified_data)
print(f"Overall ROAS: {roas_report.overall_roas:.2f}")
print(f"Total Ad Spend: ${roas_report.total_spend:.2f}")
print(f"Total Attributed Revenue: ${roas_report.total_revenue:.2f}")
```

## Configuration

### Attribution Models

```python
from genpark_joiner import AttributionModel

# First-touch attribution (all credit to first interaction)
joiner.set_attribution_model(AttributionModel.FIRST_TOUCH)

# Last-touch attribution (all credit to last interaction)
joiner.set_attribution_model(AttributionModel.LAST_TOUCH)

# Linear attribution (equal credit across touchpoints)
joiner.set_attribution_model(AttributionModel.LINEAR)

# Time-decay attribution (more credit to recent touchpoints)
joiner.set_attribution_model(
    AttributionModel.TIME_DECAY,
    decay_rate=0.5  # Half-life in days
)

# Position-based attribution (40% first, 40% last, 20% middle)
joiner.set_attribution_model(AttributionModel.POSITION_BASED)
```

### Custom Attribution Windows

```python
# Set different windows for different platforms
joiner.set_platform_window(AdPlatform.GOOGLE_ADS, days=7)
joiner.set_platform_window(AdPlatform.FACEBOOK_ADS, days=28)
joiner.set_platform_window(AdPlatform.TIKTOK_ADS, days=7)
```

## Advanced Features

### Custom Data Mapping

```python
from genpark_joiner import FieldMapper

# Map custom fields from your retail system
custom_mapper = FieldMapper()
custom_mapper.add_mapping(
    source_field='order_total',
    target_field='revenue',
    transform=lambda x: float(x) * 1.0  # Convert to float
)

custom_mapper.add_mapping(
    source_field='promo_code',
    target_field='campaign_id',
    transform=lambda code: extract_campaign_from_promo(code)
)

joiner.set_field_mapper(custom_mapper)
```

### Inventory Integration

```python
from genpark_joiner import InventorySource

# Add inventory data for stock-level analysis
joiner.add_inventory_source(
    source=InventorySource.CUSTOM,
    connection_string=os.getenv('INVENTORY_DB_CONNECTION'),
    query_template="""
        SELECT sku, stock_level, reorder_point, last_updated
        FROM inventory
        WHERE last_updated >= :start_date
    """
)

# Generate report with inventory context
report = joiner.generate_inventory_aware_report(
    start_date=start_date,
    end_date=end_date,
    include_out_of_stock=True
)

# Identify campaigns promoting out-of-stock items
wasted_spend = report.campaigns_with_stock_issues()
```

### Multi-Touch Attribution Journey

```python
# Get customer journey data
journeys = joiner.get_customer_journeys(
    start_date=start_date,
    end_date=end_date,
    min_touchpoints=2
)

for journey in journeys:
    print(f"Customer: {journey.customer_id}")
    print(f"Touchpoints: {len(journey.touchpoints)}")
    for tp in journey.touchpoints:
        print(f"  - {tp.platform} / {tp.campaign} / {tp.timestamp}")
    print(f"Conversion Value: ${journey.conversion_value}")
    print(f"Attribution: {journey.attribution_breakdown}")
```

### Export and Reporting

```python
# Export to CSV
joiner.export_to_csv(
    unified_data,
    filename='cross_channel_roas.csv',
    include_metadata=True
)

# Export to Google Sheets
joiner.export_to_google_sheets(
    unified_data,
    spreadsheet_id=os.getenv('GOOGLE_SHEETS_ID'),
    worksheet_name='ROAS Dashboard'
)

# Generate PDF report
from genpark_joiner import ReportGenerator

report_gen = ReportGenerator(template='executive_summary')
report_gen.generate_pdf(
    data=unified_data,
    output_path='monthly_roas_report.pdf',
    include_charts=True
)
```

## Campaign-Level Analysis

```python
# Get ROAS by campaign
campaign_roas = joiner.get_campaign_roas(unified_data)

for campaign in campaign_roas:
    print(f"Campaign: {campaign.name}")
    print(f"  Platform: {campaign.platform}")
    print(f"  Spend: ${campaign.spend:.2f}")
    print(f"  Revenue: ${campaign.attributed_revenue:.2f}")
    print(f"  ROAS: {campaign.roas:.2f}")
    print(f"  Conversions: {campaign.conversion_count}")
    print()

# Filter underperforming campaigns
low_roas = [c for c in campaign_roas if c.roas < 2.0]
print(f"Found {len(low_roas)} campaigns with ROAS < 2.0")
```

## UTM Parameter Tracking

```python
from genpark_joiner import UTMTracker

# Configure UTM parameter extraction
utm_tracker = UTMTracker()
joiner.set_utm_tracker(utm_tracker)

# Analyze by UTM source/medium/campaign
utm_breakdown = joiner.analyze_by_utm(
    unified_data,
    group_by=['utm_source', 'utm_medium', 'utm_campaign']
)

for group, metrics in utm_breakdown.items():
    print(f"{group}: ROAS = {metrics.roas:.2f}, Spend = ${metrics.spend:.2f}")
```

## Scheduled Data Syncing

```python
from genpark_joiner import Scheduler

# Set up automatic daily sync
scheduler = Scheduler(joiner)

scheduler.schedule_daily_sync(
    time='02:00',  # 2 AM
    timezone='America/New_York',
    callback=lambda data: send_slack_notification(data)
)

scheduler.start()
```

## Error Handling

```python
from genpark_joiner.exceptions import (
    PlatformConnectionError,
    DataMismatchError,
    AttributionError
)

try:
    unified_data = joiner.fetch_and_join(start_date, end_date)
except PlatformConnectionError as e:
    print(f"Failed to connect to {e.platform}: {e.message}")
    # Retry logic or fallback to cached data
except DataMismatchError as e:
    print(f"Data inconsistency: {e.details}")
    # Log for manual review
except AttributionError as e:
    print(f"Attribution calculation failed: {e.reason}")
    # Fall back to simpler attribution model
```

## Common Patterns

### Daily ROAS Monitoring

```python
import schedule
import time

def daily_roas_check():
    yesterday = datetime.now() - timedelta(days=1)
    data = joiner.fetch_and_join(yesterday, yesterday)
    roas = joiner.calculate_roas(data)
    
    if roas.overall_roas < 2.0:
        send_alert(f"Low ROAS detected: {roas.overall_roas:.2f}")

schedule.every().day.at("09:00").do(daily_roas_check)

while True:
    schedule.run_pending()
    time.sleep(60)
```

### Budget Allocation Recommendations

```python
# Get platform performance
platform_performance = joiner.get_platform_roas(unified_data)

# Calculate optimal budget distribution
from genpark_joiner import BudgetOptimizer

optimizer = BudgetOptimizer(platform_performance)
recommended_allocation = optimizer.optimize(
    total_budget=50000,
    min_roas_threshold=1.5
)

print("Recommended Budget Allocation:")
for platform, budget in recommended_allocation.items():
    print(f"  {platform}: ${budget:.2f}")
```

## Troubleshooting

### API Rate Limits
```python
joiner.set_rate_limit(
    platform=AdPlatform.GOOGLE_ADS,
    requests_per_minute=50
)
```

### Missing Conversion Data
```python
# Enable conversion import from Google Analytics
joiner.enable_ga_conversion_import(
    view_id=os.getenv('GA_VIEW_ID'),
    credentials_path='ga_credentials.json'
)
```

### Data Sync Issues
```python
# Check sync status
status = joiner.get_sync_status()
print(f"Last successful sync: {status.last_sync}")
print(f"Failed syncs: {status.failed_platforms}")

# Force resync for specific platform
joiner.force_resync(AdPlatform.FACEBOOK_ADS, days_back=7)
```

## Environment Variables

```bash
# Google Ads
GOOGLE_ADS_DEVELOPER_TOKEN=your_token
GOOGLE_ADS_CLIENT_ID=your_client_id
GOOGLE_ADS_CLIENT_SECRET=your_secret
GOOGLE_ADS_REFRESH_TOKEN=your_refresh_token
GOOGLE_ADS_CUSTOMER_ID=your_customer_id

# Facebook Ads
FACEBOOK_ACCESS_TOKEN=your_access_token
FACEBOOK_AD_ACCOUNT_ID=act_123456789

# Shopify
SHOPIFY_SHOP_URL=yourstore.myshopify.com
SHOPIFY_API_KEY=your_api_key
SHOPIFY_PASSWORD=your_password

# Database
INVENTORY_DB_CONNECTION=postgresql://user:pass@host:5432/inventory
```
