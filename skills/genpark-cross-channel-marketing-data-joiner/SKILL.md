---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory sources to calculate unified ROAS metrics across channels
triggers:
  - join marketing data from multiple channels
  - calculate cross-channel ROAS for my campaigns
  - merge programmatic ad data with retail sales
  - combine advertising spend with inventory data
  - unify marketing attribution across platforms
  - consolidate ad performance with revenue metrics
  - integrate cross-channel marketing analytics
  - merge ad platform data with sales results
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python-based skill that consolidates programmatic advertising data with retail sales and inventory information to provide unified Return on Ad Spend (ROAS) calculations. This tool enables marketers to understand true campaign performance by connecting ad spend across platforms (Google Ads, Facebook, programmatic DSPs) with actual sales outcomes and inventory availability.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

**Common dependencies include:**
```
pandas>=2.0.0
numpy>=1.24.0
sqlalchemy>=2.0.0
requests>=2.31.0
python-dotenv>=1.0.0
```

## Configuration

Create a `.env` file in the project root with your data source credentials:

```bash
# Ad Platform API Keys
GOOGLE_ADS_DEVELOPER_TOKEN=your_token_here
GOOGLE_ADS_CLIENT_ID=your_client_id
GOOGLE_ADS_CLIENT_SECRET=your_secret
FACEBOOK_ACCESS_TOKEN=your_fb_token

# Retail/Sales Database
RETAIL_DB_HOST=localhost
RETAIL_DB_PORT=5432
RETAIL_DB_NAME=retail_data
RETAIL_DB_USER=your_username
RETAIL_DB_PASSWORD=your_password

# Inventory System API
INVENTORY_API_URL=https://api.inventory.example.com
INVENTORY_API_KEY=your_inventory_key
```

## Core Components

### 1. Data Source Connectors

**Ad Platform Connector:**
```python
from genpark_joiner.connectors import AdPlatformConnector

# Initialize connector
ad_connector = AdPlatformConnector(
    platform='google_ads',
    credentials={
        'developer_token': os.getenv('GOOGLE_ADS_DEVELOPER_TOKEN'),
        'client_id': os.getenv('GOOGLE_ADS_CLIENT_ID'),
        'client_secret': os.getenv('GOOGLE_ADS_CLIENT_SECRET')
    }
)

# Fetch ad performance data
ad_data = ad_connector.fetch_campaign_data(
    start_date='2026-07-01',
    end_date='2026-07-31',
    metrics=['spend', 'impressions', 'clicks', 'conversions']
)
```

**Retail Sales Connector:**
```python
from genpark_joiner.connectors import RetailConnector

retail_connector = RetailConnector(
    db_type='postgresql',
    host=os.getenv('RETAIL_DB_HOST'),
    port=os.getenv('RETAIL_DB_PORT'),
    database=os.getenv('RETAIL_DB_NAME'),
    user=os.getenv('RETAIL_DB_USER'),
    password=os.getenv('RETAIL_DB_PASSWORD')
)

# Fetch sales data
sales_data = retail_connector.fetch_sales(
    start_date='2026-07-01',
    end_date='2026-07-31',
    groupby=['product_id', 'date']
)
```

### 2. Data Joiner

**Basic Join Operation:**
```python
from genpark_joiner import DataJoiner

joiner = DataJoiner()

# Join ad data with sales data
unified_data = joiner.join(
    ad_data=ad_data,
    sales_data=sales_data,
    join_keys=['product_id', 'date'],
    join_type='left'
)
```

**Multi-Source Join:**
```python
from genpark_joiner.connectors import InventoryConnector

inventory_connector = InventoryConnector(
    api_url=os.getenv('INVENTORY_API_URL'),
    api_key=os.getenv('INVENTORY_API_KEY')
)

inventory_data = inventory_connector.fetch_inventory(
    start_date='2026-07-01',
    end_date='2026-07-31'
)

# Join all three data sources
complete_data = joiner.join_multiple([
    {'data': ad_data, 'name': 'ads'},
    {'data': sales_data, 'name': 'sales'},
    {'data': inventory_data, 'name': 'inventory'}
], join_keys=['product_id', 'date'])
```

### 3. ROAS Calculator

**Calculate Unified ROAS:**
```python
from genpark_joiner.metrics import ROASCalculator

calculator = ROASCalculator()

# Calculate ROAS from unified data
roas_results = calculator.calculate_roas(
    data=unified_data,
    spend_column='ad_spend',
    revenue_column='sales_revenue',
    groupby=['campaign_id', 'product_category']
)

print(roas_results)
# Output:
# campaign_id  product_category  ad_spend  revenue  ROAS
# 12345       Electronics        5000.00  25000.00  5.00
# 12346       Apparel           3000.00   9000.00  3.00
```

**Advanced ROAS with Attribution:**
```python
# Apply attribution model before ROAS calculation
attributed_data = calculator.apply_attribution(
    data=unified_data,
    model='linear',  # Options: 'first_touch', 'last_touch', 'linear', 'time_decay'
    lookback_window=30  # days
)

roas_with_attribution = calculator.calculate_roas(
    data=attributed_data,
    spend_column='attributed_spend',
    revenue_column='attributed_revenue'
)
```

## Complete Example Usage

```python
import os
from dotenv import load_dotenv
from genpark_joiner import DataJoiner
from genpark_joiner.connectors import AdPlatformConnector, RetailConnector, InventoryConnector
from genpark_joiner.metrics import ROASCalculator
import pandas as pd

# Load environment variables
load_dotenv()

def main():
    # 1. Fetch ad platform data
    print("Fetching ad data...")
    ad_connector = AdPlatformConnector(platform='google_ads')
    ad_data = ad_connector.fetch_campaign_data(
        start_date='2026-07-01',
        end_date='2026-07-31'
    )
    
    # 2. Fetch retail sales data
    print("Fetching sales data...")
    retail_connector = RetailConnector(db_type='postgresql')
    sales_data = retail_connector.fetch_sales(
        start_date='2026-07-01',
        end_date='2026-07-31'
    )
    
    # 3. Fetch inventory data
    print("Fetching inventory data...")
    inventory_connector = InventoryConnector()
    inventory_data = inventory_connector.fetch_inventory(
        start_date='2026-07-01',
        end_date='2026-07-31'
    )
    
    # 4. Join all data sources
    print("Joining data sources...")
    joiner = DataJoiner()
    unified_data = joiner.join_multiple([
        {'data': ad_data, 'name': 'ads'},
        {'data': sales_data, 'name': 'sales'},
        {'data': inventory_data, 'name': 'inventory'}
    ], join_keys=['product_id', 'date'])
    
    # 5. Calculate ROAS
    print("Calculating ROAS...")
    calculator = ROASCalculator()
    roas_results = calculator.calculate_roas(
        data=unified_data,
        spend_column='ad_spend',
        revenue_column='sales_revenue',
        groupby=['campaign_id', 'product_category']
    )
    
    # 6. Export results
    roas_results.to_csv('roas_report.csv', index=False)
    print(f"ROAS report saved. Total campaigns: {len(roas_results)}")
    print(f"Average ROAS: {roas_results['ROAS'].mean():.2f}")
    
    return roas_results

if __name__ == '__main__':
    results = main()
```

## Common Patterns

### Pattern 1: Daily ROAS Dashboard

```python
from genpark_joiner.pipelines import DailyROASPipeline

pipeline = DailyROASPipeline(
    ad_sources=['google_ads', 'facebook_ads', 'dv360'],
    retail_source='postgresql',
    output_format='dashboard'
)

# Run daily pipeline
daily_results = pipeline.run(date='2026-07-31')
pipeline.export_to_dashboard(daily_results)
```

### Pattern 2: Product-Level Attribution

```python
from genpark_joiner.attribution import ProductAttributor

attributor = ProductAttributor()

# Attribute sales to specific ad touchpoints
product_attribution = attributor.attribute(
    ad_data=ad_data,
    sales_data=sales_data,
    product_id='SKU-12345',
    attribution_model='time_decay',
    lookback_days=30
)

# Get touchpoint contribution
touchpoint_value = attributor.get_touchpoint_value(product_attribution)
```

### Pattern 3: Cross-Channel Budget Optimization

```python
from genpark_joiner.optimization import BudgetOptimizer

optimizer = BudgetOptimizer()

# Analyze current ROAS across channels
channel_performance = optimizer.analyze_channels(unified_data)

# Get budget reallocation recommendations
recommendations = optimizer.optimize_budget(
    current_allocation={'google_ads': 50000, 'facebook_ads': 30000, 'dv360': 20000},
    target_roas=4.0,
    constraints={'min_spend_per_channel': 10000}
)

print(recommendations)
# Output: {'google_ads': 55000, 'facebook_ads': 35000, 'dv360': 10000}
```

### Pattern 4: Inventory-Aware Campaign Pausing

```python
from genpark_joiner.automation import InventoryAutomation

automation = InventoryAutomation()

# Automatically pause campaigns for out-of-stock products
automation.pause_campaigns_for_oos(
    unified_data=unified_data,
    inventory_threshold=10,  # units
    ad_platforms=['google_ads', 'facebook_ads']
)
```

## Troubleshooting

### Issue: Data Join Returns Empty Results

**Problem:** Join operation returns no rows despite data in both sources.

**Solution:**
```python
# Check data types of join keys
print(ad_data['product_id'].dtype)  # Should match
print(sales_data['product_id'].dtype)

# Normalize join keys before joining
ad_data['product_id'] = ad_data['product_id'].astype(str).str.strip()
sales_data['product_id'] = sales_data['product_id'].astype(str).str.strip()

# Use fuzzy matching if needed
joiner = DataJoiner(fuzzy_match=True, match_threshold=0.85)
```

### Issue: ROAS Calculation Shows Null Values

**Problem:** ROAS column contains NaN or None values.

**Solution:**
```python
# Handle missing or zero spend
calculator = ROASCalculator(handle_zero_spend='exclude')  # or 'fill_with_zero'

# Filter out incomplete records before calculation
complete_data = unified_data.dropna(subset=['ad_spend', 'sales_revenue'])
roas_results = calculator.calculate_roas(complete_data)
```

### Issue: API Rate Limiting

**Problem:** Ad platform API calls fail with rate limit errors.

**Solution:**
```python
# Enable automatic retry with backoff
ad_connector = AdPlatformConnector(
    platform='google_ads',
    retry_on_rate_limit=True,
    max_retries=5,
    backoff_factor=2
)

# Or batch requests
ad_data = ad_connector.fetch_campaign_data_batch(
    date_ranges=[
        ('2026-07-01', '2026-07-10'),
        ('2026-07-11', '2026-07-20'),
        ('2026-07-21', '2026-07-31')
    ],
    batch_delay=2  # seconds between batches
)
```

### Issue: Memory Error with Large Datasets

**Problem:** Out of memory when joining large datasets.

**Solution:**
```python
# Use chunked processing
joiner = DataJoiner(chunk_size=10000)

unified_data_generator = joiner.join_chunked(
    ad_data=ad_data,
    sales_data=sales_data,
    join_keys=['product_id', 'date']
)

# Process chunks iteratively
for chunk in unified_data_generator:
    roas_chunk = calculator.calculate_roas(chunk)
    roas_chunk.to_csv('roas_report.csv', mode='a', header=False)
```

### Issue: Date Timezone Mismatches

**Problem:** Dates don't align due to timezone differences.

**Solution:**
```python
# Normalize all dates to UTC before joining
joiner = DataJoiner(normalize_timezone='UTC')

# Or specify timezone for each source
ad_data = ad_connector.fetch_campaign_data(
    start_date='2026-07-01',
    end_date='2026-07-31',
    timezone='America/New_York'
)

sales_data = retail_connector.fetch_sales(
    start_date='2026-07-01',
    end_date='2026-07-31',
    timezone='America/New_York'
)
```

## Best Practices

1. **Always validate join keys** before performing joins to ensure data type and format consistency
2. **Use attribution models** appropriate to your customer journey length
3. **Set up automated alerts** for ROAS drops below threshold
4. **Cache API responses** to reduce redundant calls and improve performance
5. **Log all data transformations** for debugging and audit trails
6. **Test with sample data** before processing full production datasets
