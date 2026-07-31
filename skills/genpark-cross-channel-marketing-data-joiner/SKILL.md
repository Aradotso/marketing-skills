---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data from multiple channels
  - calculate cross-channel ROAS
  - combine ad spend with sales data
  - merge programmatic ads with retail inventory
  - unify marketing and sales metrics
  - analyze total return on ad spend
  - connect advertising data to revenue
  - aggregate multi-channel marketing performance
---

# GenPark Cross-Channel Marketing Data Joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that consolidates programmatic advertising data with retail sales and inventory information to provide comprehensive ROAS (Return on Ad Spend) metrics. It enables marketers to understand the full customer journey from ad impression to purchase across multiple channels.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

Or install as a package:

```bash
pip install git+https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
```

## Core Concepts

### Data Sources
- **Programmatic Ad Data**: Impressions, clicks, conversions, ad spend from platforms like Google Ads, Facebook Ads, programmatic DSPs
- **Retail Data**: Point-of-sale transactions, online orders, customer purchases
- **Inventory Data**: Stock levels, product SKUs, pricing information

### Key Metrics
- **ROAS**: Revenue / Ad Spend
- **Total ROAS**: (Online Revenue + Offline Revenue) / Total Ad Spend
- **Attribution**: Connecting ad exposure to sales events

## Basic Usage

```python
from genpark_data_joiner import DataJoiner, AdSource, RetailSource

# Initialize the data joiner
joiner = DataJoiner()

# Add programmatic ad data
ad_data = AdSource(
    platform="google_ads",
    data_path="./data/google_ads_export.csv"
)
joiner.add_source(ad_data)

# Add retail sales data
retail_data = RetailSource(
    system="shopify",
    data_path="./data/shopify_orders.csv"
)
joiner.add_source(retail_data)

# Perform the join and calculate ROAS
results = joiner.join_and_analyze(
    date_range=("2026-07-01", "2026-07-31"),
    attribution_window_days=30
)

print(f"Total ROAS: {results.total_roas}")
print(f"Online ROAS: {results.online_roas}")
print(f"Offline ROAS: {results.offline_roas}")
```

## Configuration

### Environment Variables

```bash
# API credentials for data sources
export GOOGLE_ADS_CLIENT_ID=your_client_id
export GOOGLE_ADS_CLIENT_SECRET=your_client_secret
export GOOGLE_ADS_REFRESH_TOKEN=your_refresh_token
export GOOGLE_ADS_DEVELOPER_TOKEN=your_developer_token

export FACEBOOK_ACCESS_TOKEN=your_access_token
export FACEBOOK_AD_ACCOUNT_ID=your_account_id

export SHOPIFY_API_KEY=your_api_key
export SHOPIFY_API_SECRET=your_api_secret
export SHOPIFY_SHOP_NAME=your_shop_name

# Database connection (optional, for storing results)
export DATABASE_URL=postgresql://user:password@localhost:5432/marketing_db
```

### Configuration File

Create a `config.yaml` file:

```yaml
attribution:
  window_days: 30
  model: "last_click"  # Options: last_click, first_click, linear, time_decay
  
data_sources:
  programmatic_ads:
    - platform: google_ads
      enabled: true
    - platform: facebook_ads
      enabled: true
    - platform: tiktok_ads
      enabled: false
      
  retail:
    - system: shopify
      enabled: true
    - system: square
      enabled: false
      
  inventory:
    - system: warehouse_management
      enabled: true

join_keys:
  customer_id: "email"
  product_id: "sku"
  
output:
  format: "csv"  # Options: csv, json, parquet
  destination: "./output/"
```

Load configuration:

```python
from genpark_data_joiner import DataJoiner

joiner = DataJoiner.from_config("config.yaml")
results = joiner.run()
```

## Advanced Usage Patterns

### Multi-Platform Ad Data Integration

```python
from genpark_data_joiner import DataJoiner, GoogleAdsSource, FacebookAdsSource, TikTokAdsSource

joiner = DataJoiner()

# Add multiple ad platforms
joiner.add_source(GoogleAdsSource(
    customer_id=os.getenv("GOOGLE_ADS_CUSTOMER_ID"),
    date_range=("2026-07-01", "2026-07-31")
))

joiner.add_source(FacebookAdsSource(
    account_id=os.getenv("FACEBOOK_AD_ACCOUNT_ID"),
    date_range=("2026-07-01", "2026-07-31")
))

joiner.add_source(TikTokAdsSource(
    advertiser_id=os.getenv("TIKTOK_ADVERTISER_ID"),
    date_range=("2026-07-01", "2026-07-31")
))

# Join with retail data
joiner.add_source(RetailSource(
    system="shopify",
    store_url=os.getenv("SHOPIFY_SHOP_NAME")
))

# Run analysis
results = joiner.analyze(
    attribution_model="time_decay",
    attribution_window=30,
    group_by=["platform", "campaign", "product_category"]
)

# Export results
results.to_csv("./output/cross_channel_roas.csv")
```

### Custom Attribution Models

```python
from genpark_data_joiner import DataJoiner, AttributionModel

# Define custom attribution logic
class CustomAttributionModel(AttributionModel):
    def calculate_credit(self, touchpoints):
        """
        Assign 40% to first touch, 40% to last touch, 20% distributed evenly
        """
        if len(touchpoints) == 1:
            return {touchpoints[0]: 1.0}
        
        credits = {}
        credits[touchpoints[0]] = 0.4
        credits[touchpoints[-1]] = credits.get(touchpoints[-1], 0) + 0.4
        
        middle_credit = 0.2 / len(touchpoints)
        for touchpoint in touchpoints:
            credits[touchpoint] = credits.get(touchpoint, 0) + middle_credit
        
        return credits

joiner = DataJoiner()
joiner.set_attribution_model(CustomAttributionModel())

# Add data sources and run
results = joiner.join_and_analyze()
```

### Inventory-Aware ROAS

```python
from genpark_data_joiner import DataJoiner, InventorySource

joiner = DataJoiner()

# Add inventory data to optimize for in-stock products
joiner.add_source(InventorySource(
    system="warehouse",
    api_endpoint=os.getenv("WAREHOUSE_API_URL"),
    api_key=os.getenv("WAREHOUSE_API_KEY")
))

# Calculate ROAS with inventory context
results = joiner.analyze(
    include_inventory_metrics=True,
    filter_out_of_stock=True
)

# See which products had high ROAS but inventory issues
for product in results.products:
    if product.roas > 5.0 and product.stockout_days > 7:
        print(f"High ROAS product {product.sku} had {product.stockout_days} days out of stock")
```

### Real-Time Data Streaming

```python
from genpark_data_joiner import StreamingDataJoiner
import asyncio

async def process_real_time_data():
    joiner = StreamingDataJoiner()
    
    # Connect to real-time data streams
    await joiner.connect_stream("google_ads", os.getenv("GOOGLE_ADS_STREAM_URL"))
    await joiner.connect_stream("shopify_webhooks", os.getenv("SHOPIFY_WEBHOOK_URL"))
    
    # Process events as they arrive
    async for event in joiner.stream():
        if event.type == "purchase":
            # Update ROAS calculations in real-time
            updated_roas = await joiner.calculate_incremental_roas(event)
            print(f"Updated ROAS: {updated_roas}")

asyncio.run(process_real_time_data())
```

## Data Schema

### Expected Ad Data Format

```python
# CSV or DataFrame with columns:
ad_data_schema = {
    "date": "YYYY-MM-DD",
    "platform": "google_ads | facebook_ads | tiktok_ads",
    "campaign_id": "string",
    "campaign_name": "string",
    "ad_group_id": "string",
    "impressions": "integer",
    "clicks": "integer",
    "spend": "float",
    "conversions": "integer",
    "user_id": "string (optional)",
    "click_id": "string (optional)"
}
```

### Expected Retail Data Format

```python
# CSV or DataFrame with columns:
retail_data_schema = {
    "order_id": "string",
    "date": "YYYY-MM-DD HH:MM:SS",
    "customer_id": "string",
    "customer_email": "string",
    "product_sku": "string",
    "product_name": "string",
    "quantity": "integer",
    "revenue": "float",
    "channel": "online | offline",
    "click_id": "string (optional, for direct attribution)"
}
```

## Common Patterns

### Campaign Performance Analysis

```python
from genpark_data_joiner import DataJoiner

joiner = DataJoiner.from_config("config.yaml")
results = joiner.join_and_analyze()

# Group by campaign
campaign_performance = results.group_by("campaign_name").agg({
    "spend": "sum",
    "revenue": "sum",
    "orders": "count",
    "roas": "mean"
})

# Sort by ROAS
top_campaigns = campaign_performance.sort_values("roas", ascending=False).head(10)
print(top_campaigns)
```

### Cross-Channel Attribution Report

```python
# Compare platform performance
platform_comparison = results.group_by("platform").agg({
    "spend": "sum",
    "revenue": "sum",
    "roas": "mean",
    "attributed_orders": "count"
})

print(platform_comparison)
```

### Cohort Analysis

```python
# Analyze ROAS by customer acquisition cohort
cohort_analysis = joiner.analyze_cohorts(
    cohort_field="first_purchase_date",
    cohort_period="month",
    metrics=["roas", "ltv", "repeat_purchase_rate"]
)

print(cohort_analysis)
```

## Troubleshooting

### Data Not Joining Properly

```python
# Debug join keys
joiner = DataJoiner(debug=True)
joiner.validate_join_keys()  # Shows which keys are missing or mismatched

# Check data quality
quality_report = joiner.data_quality_report()
print(quality_report)
```

### Attribution Window Issues

```python
# If ROAS seems too low, try extending attribution window
results_7day = joiner.analyze(attribution_window=7)
results_30day = joiner.analyze(attribution_window=30)
results_90day = joiner.analyze(attribution_window=90)

print(f"7-day ROAS: {results_7day.total_roas}")
print(f"30-day ROAS: {results_30day.total_roas}")
print(f"90-day ROAS: {results_90day.total_roas}")
```

### Missing Revenue Data

```python
# Check for unattributed revenue
unattributed = joiner.get_unattributed_revenue()
print(f"Unattributed revenue: ${unattributed.total}")
print(f"Percentage of total: {unattributed.percentage}%")

# Investigate reasons
for reason, amount in unattributed.breakdown.items():
    print(f"{reason}: ${amount}")
```

### Performance Optimization

```python
# For large datasets, use chunked processing
joiner = DataJoiner(chunk_size=10000)

# Enable caching
joiner.enable_cache(cache_dir="./cache")

# Use parallel processing
results = joiner.analyze(n_workers=4)
```

## API Reference

### Core Classes

- `DataJoiner`: Main class for joining and analyzing data
- `AdSource`: Base class for ad platform data sources
- `RetailSource`: Base class for retail system data sources
- `InventorySource`: Base class for inventory data sources
- `AttributionModel`: Base class for custom attribution logic

### Key Methods

- `DataJoiner.add_source(source)`: Add a data source
- `DataJoiner.join_and_analyze()`: Perform join and calculate metrics
- `DataJoiner.analyze_cohorts()`: Run cohort analysis
- `DataJoiner.export(format, path)`: Export results

## Example Workflow

```python
#!/usr/bin/env python3
from genpark_data_joiner import DataJoiner, GoogleAdsSource, ShopifySource
import os

def main():
    # Initialize joiner
    joiner = DataJoiner()
    
    # Configure data sources
    joiner.add_source(GoogleAdsSource(
        customer_id=os.getenv("GOOGLE_ADS_CUSTOMER_ID"),
        date_range=("2026-07-01", "2026-07-31")
    ))
    
    joiner.add_source(ShopifySource(
        shop_name=os.getenv("SHOPIFY_SHOP_NAME"),
        api_key=os.getenv("SHOPIFY_API_KEY")
    ))
    
    # Run analysis
    print("Joining data sources...")
    results = joiner.join_and_analyze(
        attribution_window=30,
        attribution_model="time_decay"
    )
    
    # Display results
    print(f"\n=== Cross-Channel ROAS Report ===")
    print(f"Total Ad Spend: ${results.total_spend:,.2f}")
    print(f"Total Revenue: ${results.total_revenue:,.2f}")
    print(f"Total ROAS: {results.total_roas:.2f}x")
    print(f"Attributed Orders: {results.attributed_orders}")
    
    # Export detailed report
    results.to_csv("./output/roas_report.csv")
    print("\nDetailed report exported to ./output/roas_report.csv")

if __name__ == "__main__":
    main()
```
