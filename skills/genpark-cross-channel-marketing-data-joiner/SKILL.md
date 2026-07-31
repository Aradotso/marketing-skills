---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate unified ROAS across marketing channels
triggers:
  - join my ad data with sales data
  - calculate cross-channel ROAS
  - merge programmatic advertising with inventory data
  - combine marketing spend with retail performance
  - unify ad platform data with store sales
  - calculate total return on ad spend across channels
  - integrate advertising data with e-commerce metrics
  - consolidate marketing attribution data
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python-based skill that combines programmatic advertising data (Google Ads, Facebook Ads, etc.) with retail sales and inventory data to calculate unified Return on Ad Spend (ROAS) metrics. This enables marketers to understand the true impact of their advertising across all channels and touchpoints.

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

The skill operates on three primary data sources:

1. **Programmatic Ad Data**: Campaign spend, impressions, clicks from ad platforms
2. **Retail/E-commerce Data**: Sales transactions, revenue, customer IDs
3. **Inventory Data**: Product availability, SKUs, stock levels

These are joined on common keys (timestamps, product IDs, campaign IDs) to produce unified attribution reports.

## Basic Usage

```python
from genpark_data_joiner import CrossChannelJoiner

# Initialize the joiner
joiner = CrossChannelJoiner(
    ad_platform_api_key=os.environ.get('AD_PLATFORM_API_KEY'),
    retail_db_conn=os.environ.get('RETAIL_DB_CONNECTION_STRING')
)

# Load ad data
ad_data = joiner.load_ad_data(
    platforms=['google_ads', 'facebook_ads', 'tiktok_ads'],
    date_range=('2026-07-01', '2026-07-31')
)

# Load retail sales data
sales_data = joiner.load_sales_data(
    source='shopify',  # or 'bigcommerce', 'custom_db', etc.
    date_range=('2026-07-01', '2026-07-31')
)

# Join data and calculate ROAS
unified_report = joiner.join_and_calculate(
    ad_data=ad_data,
    sales_data=sales_data,
    attribution_window_days=7,
    join_keys=['timestamp', 'product_id', 'utm_campaign']
)

# Access results
print(f"Total ROAS: {unified_report.total_roas}")
print(f"Revenue: ${unified_report.total_revenue}")
print(f"Ad Spend: ${unified_report.total_spend}")
```

## Configuration

Create a `config.json` file for persistent configuration:

```json
{
  "ad_platforms": {
    "google_ads": {
      "customer_id": "123-456-7890",
      "api_version": "v14"
    },
    "facebook_ads": {
      "account_id": "act_1234567890"
    }
  },
  "retail_source": {
    "type": "shopify",
    "store_url": "https://mystore.myshopify.com"
  },
  "attribution": {
    "window_days": 7,
    "model": "last_click"
  },
  "join_strategy": "left",
  "currency": "USD"
}
```

Load configuration:

```python
from genpark_data_joiner import CrossChannelJoiner

joiner = CrossChannelJoiner.from_config('config.json')
```

## Data Loading Methods

### Loading Ad Platform Data

```python
# Google Ads
google_data = joiner.load_google_ads(
    customer_id=os.environ.get('GOOGLE_ADS_CUSTOMER_ID'),
    date_range=('2026-07-01', '2026-07-31'),
    metrics=['cost', 'clicks', 'impressions', 'conversions']
)

# Facebook Ads
fb_data = joiner.load_facebook_ads(
    account_id=os.environ.get('FB_ADS_ACCOUNT_ID'),
    date_range=('2026-07-01', '2026-07-31'),
    fields=['spend', 'clicks', 'impressions', 'actions']
)

# Multiple platforms at once
all_ad_data = joiner.load_ad_data(
    platforms=['google_ads', 'facebook_ads', 'tiktok_ads'],
    date_range=('2026-07-01', '2026-07-31')
)
```

### Loading Retail/Sales Data

```python
# Shopify
sales_data = joiner.load_shopify_sales(
    api_key=os.environ.get('SHOPIFY_API_KEY'),
    store_url=os.environ.get('SHOPIFY_STORE_URL'),
    date_range=('2026-07-01', '2026-07-31')
)

# Custom database
sales_data = joiner.load_from_db(
    connection_string=os.environ.get('SALES_DB_CONNECTION'),
    query="""
        SELECT order_id, product_id, revenue, timestamp, utm_campaign
        FROM sales
        WHERE timestamp BETWEEN '2026-07-01' AND '2026-07-31'
    """
)

# CSV file
sales_data = joiner.load_from_csv(
    filepath='sales_data.csv',
    date_column='order_date',
    revenue_column='total_revenue'
)
```

### Loading Inventory Data

```python
# Load inventory levels
inventory_data = joiner.load_inventory(
    source='warehouse_db',
    connection_string=os.environ.get('INVENTORY_DB_CONNECTION'),
    product_ids=['SKU-001', 'SKU-002', 'SKU-003']
)
```

## Joining Strategies

### Time-Based Attribution

```python
# Last-click attribution with 7-day window
result = joiner.join_with_attribution(
    ad_data=ad_data,
    sales_data=sales_data,
    model='last_click',
    window_days=7
)

# Multi-touch attribution
result = joiner.join_with_attribution(
    ad_data=ad_data,
    sales_data=sales_data,
    model='linear',  # or 'time_decay', 'position_based'
    window_days=14
)
```

### Key-Based Joining

```python
# Join on specific keys
result = joiner.join_on_keys(
    ad_data=ad_data,
    sales_data=sales_data,
    keys=['utm_campaign', 'product_id'],
    join_type='inner'  # or 'left', 'right', 'outer'
)

# Join with custom matching logic
result = joiner.join_with_matcher(
    ad_data=ad_data,
    sales_data=sales_data,
    matcher=lambda ad_row, sale_row: (
        ad_row['campaign_id'] == sale_row['utm_campaign'] and
        abs((ad_row['timestamp'] - sale_row['timestamp']).days) <= 7
    )
)
```

## ROAS Calculation

```python
# Basic ROAS calculation
roas = joiner.calculate_roas(
    revenue=sales_data['revenue'].sum(),
    spend=ad_data['cost'].sum()
)

# ROAS by campaign
campaign_roas = joiner.calculate_roas_by_dimension(
    joined_data=result,
    dimension='campaign_name'
)

# ROAS by product category
category_roas = joiner.calculate_roas_by_dimension(
    joined_data=result,
    dimension='product_category'
)

# Time-series ROAS (daily)
daily_roas = joiner.calculate_roas_timeseries(
    joined_data=result,
    frequency='daily'
)
```

## Advanced Features

### Multi-Touch Attribution Models

```python
from genpark_data_joiner.attribution import AttributionModel

# Define custom attribution model
custom_model = AttributionModel(
    name='custom_decay',
    weight_function=lambda days_since_touch: 1.0 / (1.0 + days_since_touch)
)

result = joiner.join_with_attribution(
    ad_data=ad_data,
    sales_data=sales_data,
    model=custom_model,
    window_days=30
)
```

### Data Enrichment

```python
# Enrich with product catalog data
enriched_result = joiner.enrich_with_catalog(
    joined_data=result,
    catalog_source='product_catalog.csv',
    join_key='product_id',
    fields=['category', 'brand', 'margin']
)

# Add customer segment data
enriched_result = joiner.enrich_with_segments(
    joined_data=result,
    segment_source='customer_segments',
    customer_id_field='customer_id'
)
```

### Filtering and Aggregation

```python
# Filter by date range
filtered = joiner.filter_by_date(
    data=result,
    start_date='2026-07-15',
    end_date='2026-07-31'
)

# Aggregate by multiple dimensions
aggregated = joiner.aggregate_by(
    data=result,
    dimensions=['campaign_name', 'product_category'],
    metrics=['revenue', 'cost', 'roas']
)
```

## Exporting Results

```python
# Export to CSV
joiner.export_to_csv(
    data=unified_report,
    filepath='unified_roas_report.csv'
)

# Export to Excel with multiple sheets
joiner.export_to_excel(
    data=unified_report,
    filepath='unified_report.xlsx',
    sheets={
        'Summary': unified_report.summary,
        'By Campaign': unified_report.by_campaign,
        'By Product': unified_report.by_product
    }
)

# Export to data warehouse
joiner.export_to_warehouse(
    data=unified_report,
    destination='bigquery',
    project_id=os.environ.get('GCP_PROJECT_ID'),
    dataset='marketing_analytics',
    table='unified_roas'
)
```

## Common Patterns

### Daily ROAS Report Pipeline

```python
from datetime import datetime, timedelta
from genpark_data_joiner import CrossChannelJoiner

def generate_daily_roas_report():
    joiner = CrossChannelJoiner.from_config('config.json')
    
    yesterday = (datetime.now() - timedelta(days=1)).strftime('%Y-%m-%d')
    
    # Load data
    ad_data = joiner.load_ad_data(
        platforms=['google_ads', 'facebook_ads'],
        date_range=(yesterday, yesterday)
    )
    
    sales_data = joiner.load_sales_data(
        source='shopify',
        date_range=(yesterday, yesterday)
    )
    
    # Join and calculate
    report = joiner.join_and_calculate(
        ad_data=ad_data,
        sales_data=sales_data,
        attribution_window_days=7
    )
    
    # Export
    joiner.export_to_csv(
        data=report,
        filepath=f'reports/roas_{yesterday}.csv'
    )
    
    return report

if __name__ == '__main__':
    report = generate_daily_roas_report()
    print(f"ROAS: {report.total_roas:.2f}")
```

### Multi-Channel Campaign Analysis

```python
def analyze_campaign_performance(campaign_name):
    joiner = CrossChannelJoiner.from_config('config.json')
    
    # Load last 30 days
    date_range = (
        (datetime.now() - timedelta(days=30)).strftime('%Y-%m-%d'),
        datetime.now().strftime('%Y-%m-%d')
    )
    
    ad_data = joiner.load_ad_data(
        platforms=['google_ads', 'facebook_ads', 'tiktok_ads'],
        date_range=date_range,
        filters={'campaign_name': campaign_name}
    )
    
    sales_data = joiner.load_sales_data(
        source='shopify',
        date_range=date_range,
        filters={'utm_campaign': campaign_name}
    )
    
    # Calculate metrics
    result = joiner.join_and_calculate(
        ad_data=ad_data,
        sales_data=sales_data,
        attribution_window_days=7
    )
    
    # Get insights
    insights = {
        'total_spend': result.total_spend,
        'total_revenue': result.total_revenue,
        'roas': result.total_roas,
        'by_platform': result.by_platform,
        'top_products': result.top_products(limit=10)
    }
    
    return insights
```

### Inventory-Aware ROAS Optimization

```python
def optimize_spend_with_inventory():
    joiner = CrossChannelJoiner.from_config('config.json')
    
    # Load current inventory
    inventory = joiner.load_inventory(source='warehouse_db')
    
    # Load recent campaign data
    ad_data = joiner.load_ad_data(
        platforms=['google_ads', 'facebook_ads'],
        date_range=('2026-07-01', '2026-07-31')
    )
    
    sales_data = joiner.load_sales_data(
        source='shopify',
        date_range=('2026-07-01', '2026-07-31')
    )
    
    # Join all three data sources
    unified = joiner.join_multi(
        ad_data=ad_data,
        sales_data=sales_data,
        inventory_data=inventory,
        join_keys=['product_id']
    )
    
    # Calculate ROAS by product
    product_roas = joiner.calculate_roas_by_dimension(
        joined_data=unified,
        dimension='product_id'
    )
    
    # Filter for products with high ROAS and high inventory
    recommendations = unified[
        (unified['roas'] > 3.0) & 
        (unified['inventory_level'] > 100)
    ].sort_values('roas', ascending=False)
    
    return recommendations
```

## Troubleshooting

### Common Issues

**Issue: Attribution window not capturing conversions**
```python
# Increase attribution window
result = joiner.join_with_attribution(
    ad_data=ad_data,
    sales_data=sales_data,
    window_days=14  # Increase from 7 to 14 days
)
```

**Issue: Data type mismatches when joining**
```python
# Normalize data types before joining
ad_data = joiner.normalize_data_types(
    data=ad_data,
    schema={'product_id': 'string', 'timestamp': 'datetime'}
)

sales_data = joiner.normalize_data_types(
    data=sales_data,
    schema={'product_id': 'string', 'timestamp': 'datetime'}
)
```

**Issue: Missing UTM parameters in sales data**
```python
# Use fuzzy matching on product and time
result = joiner.join_fuzzy(
    ad_data=ad_data,
    sales_data=sales_data,
    match_fields=['product_id'],
    time_window_days=7
)
```

**Issue: Large datasets causing memory issues**
```python
# Process in batches
results = []
for batch in joiner.batch_process(
    ad_data=ad_data,
    sales_data=sales_data,
    batch_size=10000
):
    results.append(batch)

final_result = joiner.combine_batches(results)
```

## Environment Variables

Required environment variables for API access:

- `AD_PLATFORM_API_KEY`: Generic ad platform API key
- `GOOGLE_ADS_CUSTOMER_ID`: Google Ads customer ID
- `GOOGLE_ADS_DEVELOPER_TOKEN`: Google Ads developer token
- `FB_ADS_ACCOUNT_ID`: Facebook Ads account ID
- `FB_ADS_ACCESS_TOKEN`: Facebook Ads access token
- `SHOPIFY_API_KEY`: Shopify API key
- `SHOPIFY_STORE_URL`: Shopify store URL
- `RETAIL_DB_CONNECTION_STRING`: Retail database connection string
- `INVENTORY_DB_CONNECTION`: Inventory database connection string
- `GCP_PROJECT_ID`: Google Cloud Platform project ID (for BigQuery export)

## Best Practices

1. **Use consistent date ranges**: Ensure ad data and sales data cover the same periods plus attribution window
2. **Validate join keys**: Check that join keys exist and match between datasets
3. **Handle timezone differences**: Normalize all timestamps to UTC before joining
4. **Regular data validation**: Verify data quality before calculating ROAS
5. **Cache API responses**: Reduce API calls by caching ad platform data
6. **Monitor attribution gaps**: Track sales that don't match to any ad touchpoints

## Additional Resources

- Homepage: https://genpark.ai
- Documentation: Check the repository wiki for detailed API documentation
- Issues: Report bugs or request features on GitHub
