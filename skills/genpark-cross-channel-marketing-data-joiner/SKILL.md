---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join ad data with sales data
  - calculate cross-channel ROAS
  - merge programmatic ads with inventory
  - connect marketing data to retail systems
  - combine ad spend with revenue data
  - analyze total return on ad spend
  - integrate cross-channel marketing metrics
  - unify ad platform and sales data
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that unifies programmatic advertising data with retail sales and inventory systems to calculate accurate Return on Ad Spend (ROAS) across multiple marketing channels. It handles data normalization, matching, and aggregation from disparate sources.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

## Core Concepts

This skill performs three primary operations:

1. **Data Ingestion** - Imports data from ad platforms (Google Ads, Facebook, programmatic DSPs) and retail systems
2. **Data Joining** - Matches transactions to ad campaigns using timestamps, user IDs, or attribution windows
3. **ROAS Calculation** - Computes return metrics across all channels

## Basic Usage

```python
from genpark_data_joiner import DataJoiner, AdSource, RetailSource

# Initialize the joiner
joiner = DataJoiner()

# Add programmatic ad data
ad_data = AdSource(
    platform="google_ads",
    data_path="data/google_ads_export.csv",
    api_key=os.environ.get("GOOGLE_ADS_API_KEY")
)

# Add retail/sales data
retail_data = RetailSource(
    platform="shopify",
    data_path="data/sales_export.csv",
    api_key=os.environ.get("SHOPIFY_API_KEY")
)

# Join data with 7-day attribution window
results = joiner.join(
    ad_sources=[ad_data],
    retail_sources=[retail_data],
    attribution_window_days=7,
    match_key="user_id"
)

# Calculate ROAS
roas = results.calculate_roas()
print(f"Total ROAS: {roas['total_roas']}")
print(f"Channel breakdown: {roas['by_channel']}")
```

## Configuration

Create a `config.yaml` file for persistent settings:

```yaml
attribution:
  window_days: 7
  model: "last_click"  # Options: last_click, first_click, linear, time_decay
  
data_sources:
  ad_platforms:
    - name: "google_ads"
      enabled: true
      api_key_env: "GOOGLE_ADS_API_KEY"
    - name: "facebook_ads"
      enabled: true
      api_key_env: "FACEBOOK_ADS_API_KEY"
    - name: "dv360"
      enabled: false
      
  retail_platforms:
    - name: "shopify"
      enabled: true
      api_key_env: "SHOPIFY_API_KEY"
    - name: "square"
      enabled: false
      
matching:
  keys: ["user_id", "email", "device_id"]
  fuzzy_match: true
  confidence_threshold: 0.85
  
output:
  format: "csv"  # Options: csv, json, parquet
  include_unmatched: false
```

Load configuration:

```python
from genpark_data_joiner import DataJoiner

joiner = DataJoiner.from_config("config.yaml")
results = joiner.run()
```

## API Reference

### DataJoiner Class

```python
from genpark_data_joiner import DataJoiner

# Initialize with custom settings
joiner = DataJoiner(
    attribution_window_days=7,
    attribution_model="last_click",
    match_keys=["user_id", "email"]
)

# Add data sources programmatically
joiner.add_ad_source(platform, data_source)
joiner.add_retail_source(platform, data_source)

# Execute join
results = joiner.join()

# Access results
results.get_matched_transactions()
results.get_unmatched_ad_spend()
results.get_channel_metrics()
results.calculate_roas(by_channel=True)
results.export("output.csv")
```

### AdSource Class

```python
from genpark_data_joiner import AdSource

# From CSV file
ad_source = AdSource.from_csv(
    path="ads.csv",
    platform="google_ads",
    date_column="date",
    spend_column="cost",
    campaign_column="campaign_name"
)

# From API
ad_source = AdSource.from_api(
    platform="facebook_ads",
    api_key=os.environ.get("FACEBOOK_ADS_API_KEY"),
    account_id="123456789",
    date_range=("2026-07-01", "2026-07-31")
)

# From DataFrame
import pandas as pd
df = pd.read_csv("custom_data.csv")
ad_source = AdSource.from_dataframe(df, platform="custom")
```

### RetailSource Class

```python
from genpark_data_joiner import RetailSource

# From Shopify API
retail_source = RetailSource.from_shopify(
    api_key=os.environ.get("SHOPIFY_API_KEY"),
    shop_url="mystore.myshopify.com",
    date_range=("2026-07-01", "2026-07-31")
)

# From CSV
retail_source = RetailSource.from_csv(
    path="sales.csv",
    date_column="order_date",
    revenue_column="total_price",
    user_id_column="customer_id"
)

# From database
retail_source = RetailSource.from_database(
    connection_string=os.environ.get("DB_CONNECTION_STRING"),
    query="SELECT * FROM orders WHERE date >= '2026-07-01'"
)
```

## Common Patterns

### Multi-Channel ROAS Analysis

```python
from genpark_data_joiner import DataJoiner, AdSource, RetailSource

joiner = DataJoiner(attribution_window_days=7)

# Add multiple ad platforms
platforms = [
    ("google_ads", "GOOGLE_ADS_API_KEY"),
    ("facebook_ads", "FACEBOOK_ADS_API_KEY"),
    ("tiktok_ads", "TIKTOK_ADS_API_KEY")
]

for platform, env_key in platforms:
    ad_source = AdSource.from_api(
        platform=platform,
        api_key=os.environ.get(env_key),
        date_range=("2026-07-01", "2026-07-31")
    )
    joiner.add_ad_source(ad_source)

# Add retail data
retail = RetailSource.from_shopify(
    api_key=os.environ.get("SHOPIFY_API_KEY"),
    shop_url=os.environ.get("SHOPIFY_SHOP_URL")
)
joiner.add_retail_source(retail)

# Calculate
results = joiner.join()
channel_roas = results.calculate_roas(by_channel=True)

for channel, metrics in channel_roas.items():
    print(f"{channel}: ROAS {metrics['roas']:.2f}, Spend ${metrics['spend']:.2f}, Revenue ${metrics['revenue']:.2f}")
```

### Custom Attribution Model

```python
from genpark_data_joiner import DataJoiner, AttributionModel

# Define custom time-decay attribution
def custom_time_decay(days_since_interaction, total_days=7):
    return (total_days - days_since_interaction) / sum(range(1, total_days + 1))

joiner = DataJoiner(
    attribution_model=AttributionModel.custom(custom_time_decay),
    attribution_window_days=7
)

results = joiner.join()
```

### Incremental Data Processing

```python
from genpark_data_joiner import DataJoiner
from datetime import datetime, timedelta

joiner = DataJoiner()

# Process data in daily increments
start_date = datetime(2026, 7, 1)
end_date = datetime(2026, 7, 31)
current_date = start_date

all_results = []

while current_date <= end_date:
    date_str = current_date.strftime("%Y-%m-%d")
    
    # Load day's data
    ad_source = AdSource.from_csv(f"data/ads_{date_str}.csv")
    retail_source = RetailSource.from_csv(f"data/sales_{date_str}.csv")
    
    # Join
    daily_results = joiner.join(
        ad_sources=[ad_source],
        retail_sources=[retail_source]
    )
    
    all_results.append(daily_results)
    current_date += timedelta(days=1)

# Combine results
combined = DataJoiner.merge_results(all_results)
total_roas = combined.calculate_roas()
```

### Exporting Results

```python
results = joiner.join()

# Export to CSV
results.export("output/marketing_roas.csv", format="csv")

# Export to JSON with metadata
results.export("output/marketing_roas.json", format="json", include_metadata=True)

# Export to Parquet for analytics
results.export("output/marketing_roas.parquet", format="parquet")

# Get as DataFrame for further analysis
df = results.to_dataframe()
import matplotlib.pyplot as plt
df.groupby("channel")["roas"].plot(kind="bar")
plt.savefig("roas_by_channel.png")
```

## Troubleshooting

### Issue: Low Match Rate

```python
# Check match diagnostics
results = joiner.join()
diagnostics = results.get_match_diagnostics()

print(f"Match rate: {diagnostics['match_rate']:.2%}")
print(f"Unmatched ad records: {diagnostics['unmatched_ads']}")
print(f"Unmatched sales: {diagnostics['unmatched_sales']}")

# Try fuzzy matching
joiner = DataJoiner(
    match_keys=["email"],
    fuzzy_match=True,
    confidence_threshold=0.8
)
```

### Issue: Attribution Window Too Short

```python
# Analyze conversion lag
from genpark_data_joiner.analysis import conversion_lag_analysis

lag_report = conversion_lag_analysis(ad_data, retail_data)
print(f"Average days to conversion: {lag_report['avg_days']}")
print(f"90th percentile: {lag_report['p90_days']}")

# Adjust attribution window based on analysis
optimal_window = lag_report['p90_days']
joiner = DataJoiner(attribution_window_days=optimal_window)
```

### Issue: Missing API Credentials

```python
import os

required_vars = [
    "GOOGLE_ADS_API_KEY",
    "SHOPIFY_API_KEY",
    "FACEBOOK_ADS_API_KEY"
]

missing = [var for var in required_vars if not os.environ.get(var)]

if missing:
    raise EnvironmentError(f"Missing environment variables: {', '.join(missing)}")
```

### Issue: Data Format Inconsistencies

```python
from genpark_data_joiner import DataNormalizer

# Normalize before joining
normalizer = DataNormalizer()

ad_data_normalized = normalizer.normalize_ad_data(
    ad_source,
    date_format="%Y-%m-%d",
    currency="USD",
    timezone="America/New_York"
)

retail_data_normalized = normalizer.normalize_retail_data(
    retail_source,
    date_format="%m/%d/%Y",
    currency="USD",
    timezone="America/New_York"
)

results = joiner.join(
    ad_sources=[ad_data_normalized],
    retail_sources=[retail_data_normalized]
)
```

## Environment Variables

Required environment variables:

- `GOOGLE_ADS_API_KEY` - Google Ads API credentials
- `FACEBOOK_ADS_API_KEY` - Facebook Ads API token
- `SHOPIFY_API_KEY` - Shopify API key
- `SHOPIFY_SHOP_URL` - Your Shopify store URL
- `DB_CONNECTION_STRING` - Database connection (if using database sources)

Optional:

- `GENPARK_LOG_LEVEL` - Logging level (DEBUG, INFO, WARN, ERROR)
- `GENPARK_CACHE_DIR` - Directory for caching API responses
