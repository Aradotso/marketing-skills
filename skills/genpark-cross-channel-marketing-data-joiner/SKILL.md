---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate unified ROAS metrics
triggers:
  - join marketing data across channels
  - combine ad spend with sales data
  - calculate total ROAS from multiple sources
  - merge programmatic ads with retail inventory
  - unify marketing attribution data
  - connect advertising platforms with point of sale
  - aggregate cross-channel marketing performance
  - blend online ads with offline sales
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python library that unifies data from programmatic advertising platforms with retail sales and inventory systems. It enables marketers to calculate accurate Return on Ad Spend (ROAS) by joining disparate data sources into a single analytical view.

**Key capabilities:**
- Join ad platform data (Google Ads, Facebook Ads, etc.) with POS/retail systems
- Reconcile online advertising spend with offline conversions
- Calculate unified ROAS across channels
- Handle SKU-level attribution and inventory correlation
- Support time-series alignment for delayed conversions

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt

# Or install directly if packaged
pip install genpark-marketing-joiner
```

## Quick Start

```python
from genpark_joiner import DataJoiner, ROASCalculator
from genpark_joiner.sources import AdPlatformSource, RetailSource

# Initialize the data joiner
joiner = DataJoiner()

# Configure ad platform source
ad_source = AdPlatformSource(
    platform='google_ads',
    credentials_path='./config/google_ads_creds.json'
)

# Configure retail/POS source
retail_source = RetailSource(
    system='shopify',
    api_key=os.environ.get('SHOPIFY_API_KEY'),
    store_url=os.environ.get('SHOPIFY_STORE_URL')
)

# Add sources to joiner
joiner.add_source('ads', ad_source)
joiner.add_source('retail', retail_source)

# Perform join and calculate ROAS
result = joiner.join_and_calculate(
    start_date='2026-07-01',
    end_date='2026-07-31',
    join_key='campaign_id',
    attribution_window=7  # days
)

print(f"Total ROAS: {result.total_roas}")
print(f"Revenue: ${result.total_revenue}")
print(f"Ad Spend: ${result.total_spend}")
```

## Core Components

### DataJoiner

The main orchestration class that handles data source registration and joining logic.

```python
from genpark_joiner import DataJoiner

joiner = DataJoiner(
    cache_enabled=True,
    cache_dir='./cache',
    parallel_fetch=True
)

# Register multiple sources
joiner.add_source('google_ads', google_ads_source)
joiner.add_source('facebook_ads', facebook_ads_source)
joiner.add_source('retail_pos', pos_source)
joiner.add_source('inventory', inventory_source)

# Execute join with custom mapping
result = joiner.join(
    sources=['google_ads', 'facebook_ads', 'retail_pos'],
    join_strategy='left',
    join_keys={
        'google_ads': 'campaign_id',
        'facebook_ads': 'campaign_name',
        'retail_pos': 'promo_code'
    },
    mapping_rules='./config/field_mappings.yaml'
)
```

### ROASCalculator

Calculate ROAS metrics with various attribution models.

```python
from genpark_joiner import ROASCalculator

calculator = ROASCalculator(
    attribution_model='last_touch',  # or 'first_touch', 'linear', 'time_decay'
    conversion_window_days=30,
    include_view_through=True
)

# Calculate from joined data
roas_metrics = calculator.calculate(
    joined_data=result.dataframe,
    spend_column='ad_spend',
    revenue_column='sales_revenue',
    group_by=['campaign_id', 'channel']
)

# Access detailed metrics
for channel, metrics in roas_metrics.by_channel.items():
    print(f"{channel}: ROAS {metrics.roas:.2f}, CAC ${metrics.cac:.2f}")
```

### Data Sources

#### AdPlatformSource

Connect to programmatic advertising platforms.

```python
from genpark_joiner.sources import AdPlatformSource

# Google Ads
google_source = AdPlatformSource(
    platform='google_ads',
    customer_id=os.environ.get('GOOGLE_ADS_CUSTOMER_ID'),
    credentials_path='./config/google_ads_service_account.json',
    metrics=['impressions', 'clicks', 'cost', 'conversions']
)

# Facebook Ads
facebook_source = AdPlatformSource(
    platform='facebook_ads',
    access_token=os.environ.get('FACEBOOK_ACCESS_TOKEN'),
    ad_account_id=os.environ.get('FACEBOOK_AD_ACCOUNT_ID'),
    metrics=['spend', 'impressions', 'reach', 'actions']
)

# Fetch data
ad_data = google_source.fetch(
    start_date='2026-07-01',
    end_date='2026-07-31',
    breakdown=['campaign', 'adgroup', 'date']
)
```

#### RetailSource

Connect to retail and POS systems.

```python
from genpark_joiner.sources import RetailSource

# Shopify
shopify_source = RetailSource(
    system='shopify',
    api_key=os.environ.get('SHOPIFY_API_KEY'),
    api_secret=os.environ.get('SHOPIFY_API_SECRET'),
    store_url=os.environ.get('SHOPIFY_STORE_URL')
)

# Square
square_source = RetailSource(
    system='square',
    access_token=os.environ.get('SQUARE_ACCESS_TOKEN'),
    location_ids=os.environ.get('SQUARE_LOCATION_IDS').split(',')
)

# Custom API
custom_source = RetailSource(
    system='custom',
    base_url='https://api.yourretail.com',
    auth_token=os.environ.get('RETAIL_API_TOKEN'),
    endpoint_config='./config/retail_endpoints.yaml'
)

# Fetch sales data
sales_data = shopify_source.fetch(
    start_date='2026-07-01',
    end_date='2026-07-31',
    include_fields=['order_id', 'total', 'items', 'utm_source', 'utm_campaign']
)
```

#### InventorySource

Track inventory changes correlated with ad campaigns.

```python
from genpark_joiner.sources import InventorySource

inventory_source = InventorySource(
    system='warehouse_management',
    api_endpoint=os.environ.get('WMS_API_ENDPOINT'),
    api_key=os.environ.get('WMS_API_KEY')
)

# Fetch inventory movements
inventory_data = inventory_source.fetch(
    start_date='2026-07-01',
    end_date='2026-07-31',
    sku_list=['SKU-001', 'SKU-002', 'SKU-003'],
    include_transfers=True
)
```

## Configuration

### Field Mapping

Define how fields from different sources map to unified schema:

```yaml
# config/field_mappings.yaml
unified_schema:
  campaign_id:
    google_ads: campaign.id
    facebook_ads: campaign_name
    retail_pos: utm_campaign
  
  date:
    google_ads: segments.date
    facebook_ads: date_start
    retail_pos: created_at
  
  spend:
    google_ads: metrics.cost_micros
    facebook_ads: spend
    transform: divide_by_million  # for google_ads
  
  revenue:
    retail_pos: total_price
    inventory: calculated_cogs
```

Load and use mappings:

```python
from genpark_joiner import FieldMapper

mapper = FieldMapper.from_yaml('./config/field_mappings.yaml')
unified_df = mapper.apply(joined_data)
```

### Attribution Configuration

```python
# config/attribution.py
attribution_config = {
    'model': 'time_decay',
    'decay_rate': 0.5,  # half-life in days
    'conversion_window': 30,
    'view_through_window': 1,
    'cross_device': True,
    'deduplication': {
        'enabled': True,
        'priority': ['google_ads', 'facebook_ads', 'organic']
    }
}

calculator = ROASCalculator(**attribution_config)
```

## Common Patterns

### Multi-Channel ROAS Analysis

```python
from genpark_joiner import DataJoiner, ROASCalculator
import pandas as pd

# Initialize joiner with multiple ad platforms
joiner = DataJoiner()
joiner.add_source('google', google_ads_source)
joiner.add_source('facebook', facebook_ads_source)
joiner.add_source('tiktok', tiktok_ads_source)
joiner.add_source('sales', shopify_source)

# Join all sources
joined = joiner.join_all(
    date_range=('2026-07-01', '2026-07-31'),
    primary_key='order_id',
    attribution_window=7
)

# Calculate channel-specific ROAS
calculator = ROASCalculator(attribution_model='linear')
channel_roas = calculator.calculate_by_channel(joined)

# Export results
channel_roas.to_csv('roas_by_channel.csv')
channel_roas.plot(kind='bar', x='channel', y='roas', title='ROAS by Channel')
```

### SKU-Level Performance

```python
# Join ad campaigns with product-level sales
sku_performance = joiner.join(
    sources=['google_ads', 'retail_pos', 'inventory'],
    join_keys={
        'google_ads': ['campaign_id', 'product_id'],
        'retail_pos': ['utm_campaign', 'sku'],
        'inventory': ['product_sku']
    },
    granularity='sku'
)

# Calculate product ROAS
product_roas = calculator.calculate(
    sku_performance,
    group_by='sku',
    include_metrics=['units_sold', 'inventory_turns', 'margin']
)

# Find top performers
top_skus = product_roas.sort_values('roas', ascending=False).head(10)
print(top_skus[['sku', 'roas', 'revenue', 'spend', 'units_sold']])
```

### Time-Series Analysis

```python
# Daily ROAS trends
daily_roas = joiner.join_and_calculate(
    start_date='2026-01-01',
    end_date='2026-07-31',
    time_grain='daily',
    rolling_window=7
)

# Plot trends
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 6))
plt.plot(daily_roas.date, daily_roas.roas, label='Daily ROAS')
plt.plot(daily_roas.date, daily_roas.rolling_roas, label='7-Day Rolling Avg')
plt.xlabel('Date')
plt.ylabel('ROAS')
plt.legend()
plt.title('ROAS Trend Over Time')
plt.savefig('roas_trend.png')
```

### Custom Attribution Windows

```python
# Compare different attribution windows
windows = [1, 3, 7, 14, 30]
results = {}

for window in windows:
    calculator = ROASCalculator(conversion_window_days=window)
    result = joiner.join_and_calculate(
        start_date='2026-07-01',
        end_date='2026-07-31',
        calculator=calculator
    )
    results[window] = result.total_roas

# Analyze impact of attribution window
for window, roas in results.items():
    print(f"{window}-day window: ROAS {roas:.2f}")
```

## Troubleshooting

### Data Mismatch Between Sources

**Issue:** Timestamps don't align between ad platforms and retail systems.

```python
# Use timezone normalization
joiner = DataJoiner(
    normalize_timezones=True,
    target_timezone='America/New_York'
)

# Apply time bucket alignment
result = joiner.join(
    time_alignment='hour',  # bucket to nearest hour
    grace_period_minutes=15  # allow 15-min offset
)
```

### Missing Join Keys

**Issue:** Not all records have matching campaign identifiers.

```python
# Use fuzzy matching for campaign names
from genpark_joiner.matching import FuzzyMatcher

matcher = FuzzyMatcher(
    algorithm='levenshtein',
    threshold=0.85
)

joiner.set_matcher(matcher)
result = joiner.join(
    fallback_strategy='fuzzy',
    unmatched_handling='create_organic_bucket'
)

# Inspect unmatched records
print(f"Unmatched records: {result.unmatched_count}")
result.unmatched_records.to_csv('unmatched.csv')
```

### API Rate Limiting

**Issue:** Hitting rate limits when fetching data from multiple platforms.

```python
# Enable rate limiting and retry logic
ad_source = AdPlatformSource(
    platform='google_ads',
    credentials_path='./config/google_ads_creds.json',
    rate_limit={'requests_per_minute': 50},
    retry_config={
        'max_retries': 3,
        'backoff_factor': 2,
        'retry_on': [429, 503]
    }
)

# Use batch fetching
joiner = DataJoiner(
    batch_size=1000,
    parallel_fetch=True,
    max_workers=3
)
```

### Memory Issues with Large Datasets

**Issue:** Running out of memory when joining large date ranges.

```python
# Use chunked processing
joiner = DataJoiner(
    chunk_size=10000,
    streaming_mode=True
)

# Process in date chunks
from datetime import datetime, timedelta

start = datetime(2026, 1, 1)
end = datetime(2026, 7, 31)
chunk_days = 7

results = []
current = start
while current < end:
    chunk_end = min(current + timedelta(days=chunk_days), end)
    chunk_result = joiner.join_and_calculate(
        start_date=current.strftime('%Y-%m-%d'),
        end_date=chunk_end.strftime('%Y-%m-%d')
    )
    results.append(chunk_result)
    current = chunk_end

# Aggregate results
total_roas = sum(r.total_revenue for r in results) / sum(r.total_spend for r in results)
```

### Inconsistent Currency Handling

**Issue:** Ad spend in USD but sales in local currency.

```python
from genpark_joiner.currency import CurrencyConverter

# Initialize with exchange rate API
converter = CurrencyConverter(
    api_key=os.environ.get('EXCHANGE_RATE_API_KEY'),
    base_currency='USD'
)

joiner.set_currency_converter(converter)

# Auto-convert during join
result = joiner.join_and_calculate(
    start_date='2026-07-01',
    end_date='2026-07-31',
    currency_config={
        'google_ads': 'USD',
        'retail_pos': 'EUR',
        'target_currency': 'USD'
    }
)
```

## Environment Variables

Required environment variables for typical setup:

```bash
# Google Ads
GOOGLE_ADS_CUSTOMER_ID=123-456-7890
GOOGLE_ADS_DEVELOPER_TOKEN=your_dev_token

# Facebook Ads
FACEBOOK_ACCESS_TOKEN=your_access_token
FACEBOOK_AD_ACCOUNT_ID=act_123456789

# Shopify
SHOPIFY_API_KEY=your_api_key
SHOPIFY_API_SECRET=your_api_secret
SHOPIFY_STORE_URL=your-store.myshopify.com

# Square
SQUARE_ACCESS_TOKEN=your_access_token
SQUARE_LOCATION_IDS=loc1,loc2,loc3

# Currency conversion (optional)
EXCHANGE_RATE_API_KEY=your_exchange_rate_key

# Custom retail API
RETAIL_API_TOKEN=your_retail_token
WMS_API_ENDPOINT=https://api.yourwms.com
WMS_API_KEY=your_wms_key
```

## Advanced Usage

### Custom Data Source

Implement a custom data source connector:

```python
from genpark_joiner.sources import BaseSource

class CustomPlatformSource(BaseSource):
    def __init__(self, api_key, **kwargs):
        super().__init__(**kwargs)
        self.api_key = api_key
        self.base_url = "https://api.customplatform.com"
    
    def fetch(self, start_date, end_date, **params):
        import requests
        
        response = requests.get(
            f"{self.base_url}/campaigns",
            headers={"Authorization": f"Bearer {self.api_key}"},
            params={
                "start_date": start_date,
                "end_date": end_date,
                **params
            }
        )
        
        data = response.json()
        return self.normalize(data)
    
    def normalize(self, raw_data):
        # Transform to standard format
        return [{
            'campaign_id': item['id'],
            'date': item['reporting_date'],
            'spend': item['cost'],
            'impressions': item['views'],
            'clicks': item['clicks']
        } for item in raw_data['campaigns']]

# Use custom source
custom_source = CustomPlatformSource(api_key=os.environ.get('CUSTOM_API_KEY'))
joiner.add_source('custom_platform', custom_source)
```
