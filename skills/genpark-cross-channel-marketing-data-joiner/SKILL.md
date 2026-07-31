---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data from multiple channels
  - calculate cross-channel ROAS for campaigns
  - combine programmatic ad data with retail sales
  - merge advertising spend with inventory data
  - analyze total return on ad spend across platforms
  - integrate marketing attribution with sales data
  - connect ad performance to retail outcomes
  - unify marketing and sales data for ROAS calculation
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that consolidates programmatic advertising data with retail sales and inventory information to provide comprehensive Return on Ad Spend (ROAS) calculations. This tool enables marketers to understand the true effectiveness of their multi-channel campaigns by connecting online ad performance with offline and online retail outcomes.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

Typical dependencies for this type of project:
```bash
pip install pandas numpy requests python-dotenv
```

## Core Concepts

### Data Sources
- **Programmatic Ad Data**: Campaign spend, impressions, clicks, conversions from ad platforms
- **Retail Data**: Point-of-sale transactions, online orders, revenue
- **Inventory Data**: Stock levels, SKU information, product metadata

### Key Metrics
- **ROAS (Return on Ad Spend)**: Revenue generated / Ad spend
- **Cross-Channel Attribution**: Revenue attributed across multiple touchpoints
- **Inventory Impact**: How ad campaigns affect stock movement

## Basic Usage

```python
from genpark_data_joiner import CrossChannelJoiner, AdDataSource, RetailDataSource

# Initialize the joiner
joiner = CrossChannelJoiner()

# Configure ad data source
ad_data = AdDataSource(
    platform='google_ads',
    api_key=os.getenv('GOOGLE_ADS_API_KEY'),
    account_id=os.getenv('GOOGLE_ADS_ACCOUNT_ID')
)

# Configure retail data source
retail_data = RetailDataSource(
    source_type='shopify',
    api_key=os.getenv('SHOPIFY_API_KEY'),
    store_url=os.getenv('SHOPIFY_STORE_URL')
)

# Add data sources
joiner.add_source('ads', ad_data)
joiner.add_source('retail', retail_data)

# Join data for date range
results = joiner.join_data(
    start_date='2026-07-01',
    end_date='2026-07-31',
    join_key='product_sku'
)

# Calculate ROAS
roas = results.calculate_roas()
print(f"Total ROAS: {roas}")
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# Ad Platform Credentials
GOOGLE_ADS_API_KEY=your_google_ads_key
GOOGLE_ADS_ACCOUNT_ID=your_account_id
FACEBOOK_ADS_ACCESS_TOKEN=your_fb_token
FACEBOOK_ADS_ACCOUNT_ID=your_fb_account

# Retail Platform Credentials
SHOPIFY_API_KEY=your_shopify_key
SHOPIFY_STORE_URL=your_store_url
BIGCOMMERCE_API_KEY=your_bigcommerce_key

# Database (optional)
DATABASE_URL=postgresql://user:pass@localhost/marketing_data

# Attribution Window (days)
ATTRIBUTION_WINDOW=30
```

### Configuration File

Create `config.yaml` for joiner settings:

```yaml
attribution:
  window_days: 30
  model: 'last_click'  # Options: last_click, first_click, linear, time_decay

data_sources:
  ads:
    - platform: google_ads
      enabled: true
    - platform: facebook_ads
      enabled: true
  
  retail:
    - platform: shopify
      enabled: true
    - platform: pos_system
      enabled: true

join_rules:
  primary_key: product_sku
  time_tolerance_hours: 24
  match_fuzzy: false
```

## Advanced Usage Patterns

### Multi-Platform Campaign Analysis

```python
from genpark_data_joiner import CrossChannelJoiner, MultiPlatformConfig
import pandas as pd

# Initialize with config
config = MultiPlatformConfig.from_file('config.yaml')
joiner = CrossChannelJoiner(config=config)

# Add multiple ad platforms
joiner.add_ad_platform('google', {
    'api_key': os.getenv('GOOGLE_ADS_API_KEY'),
    'account_id': os.getenv('GOOGLE_ADS_ACCOUNT_ID')
})

joiner.add_ad_platform('facebook', {
    'access_token': os.getenv('FACEBOOK_ADS_ACCESS_TOKEN'),
    'account_id': os.getenv('FACEBOOK_ADS_ACCOUNT_ID')
})

# Fetch and join data
campaign_results = joiner.analyze_campaigns(
    campaigns=['summer_sale_2026', 'back_to_school'],
    start_date='2026-07-01',
    end_date='2026-07-31',
    group_by=['campaign', 'product_category']
)

# Generate ROAS report by channel
roas_by_channel = campaign_results.roas_breakdown(dimension='channel')
print(roas_by_channel)
```

### Custom Attribution Models

```python
from genpark_data_joiner import AttributionModel, TouchpointWeighting

# Define custom attribution
custom_attribution = AttributionModel(
    model_type='position_based',
    first_touch_weight=0.4,
    last_touch_weight=0.4,
    middle_touch_weight=0.2
)

# Apply to joiner
joiner.set_attribution_model(custom_attribution)

# Calculate attributed revenue
attributed_revenue = joiner.calculate_attributed_revenue(
    customer_id='CUST12345',
    conversion_date='2026-07-31'
)
```

### Inventory-Aware ROAS

```python
from genpark_data_joiner import InventoryIntegration

# Add inventory data
inventory = InventoryIntegration(
    source='warehouse_api',
    api_endpoint=os.getenv('WAREHOUSE_API_URL'),
    api_key=os.getenv('WAREHOUSE_API_KEY')
)

joiner.add_inventory_source(inventory)

# Calculate ROAS with inventory impact
results = joiner.analyze_with_inventory(
    start_date='2026-07-01',
    end_date='2026-07-31',
    include_stockouts=True
)

# Identify campaigns that drove stockouts
stockout_impact = results.stockout_analysis()
print(f"Revenue lost to stockouts: ${stockout_impact['lost_revenue']}")
```

### Data Export and Reporting

```python
# Export joined data to CSV
joiner.export_results(
    results=campaign_results,
    output_file='campaign_roas_july_2026.csv',
    format='csv'
)

# Export to data warehouse
joiner.export_to_warehouse(
    results=campaign_results,
    destination='bigquery',
    project_id=os.getenv('GCP_PROJECT_ID'),
    dataset='marketing_analytics',
    table='cross_channel_roas'
)

# Generate visual report
from genpark_data_joiner.reporting import ROASReport

report = ROASReport(campaign_results)
report.generate_dashboard(output='roas_dashboard.html')
```

## Common Patterns

### Daily ROAS Calculation Pipeline

```python
from datetime import datetime, timedelta
from genpark_data_joiner import CrossChannelJoiner

def daily_roas_pipeline():
    """Run daily ROAS calculation and store results"""
    joiner = CrossChannelJoiner.from_env()
    
    # Calculate for yesterday
    yesterday = datetime.now() - timedelta(days=1)
    date_str = yesterday.strftime('%Y-%m-%d')
    
    # Join data
    results = joiner.join_data(
        start_date=date_str,
        end_date=date_str,
        join_key='product_sku'
    )
    
    # Calculate metrics
    metrics = {
        'total_spend': results.total_ad_spend(),
        'total_revenue': results.total_revenue(),
        'roas': results.calculate_roas(),
        'conversions': results.conversion_count()
    }
    
    # Store in database
    joiner.store_metrics(metrics, date=date_str)
    
    return metrics

if __name__ == '__main__':
    metrics = daily_roas_pipeline()
    print(f"Daily ROAS: {metrics['roas']:.2f}")
```

### Product-Level Attribution

```python
def analyze_product_performance(sku, date_range_days=30):
    """Analyze individual product marketing performance"""
    joiner = CrossChannelJoiner.from_env()
    
    end_date = datetime.now()
    start_date = end_date - timedelta(days=date_range_days)
    
    # Filter by SKU
    product_data = joiner.join_data(
        start_date=start_date.strftime('%Y-%m-%d'),
        end_date=end_date.strftime('%Y-%m-%d'),
        filters={'product_sku': sku}
    )
    
    return {
        'sku': sku,
        'ad_spend': product_data.total_ad_spend(),
        'revenue': product_data.total_revenue(),
        'roas': product_data.calculate_roas(),
        'top_channel': product_data.top_performing_channel(),
        'units_sold': product_data.units_sold()
    }
```

### Budget Optimization

```python
from genpark_data_joiner.optimization import BudgetOptimizer

def optimize_channel_budgets(total_budget, constraints=None):
    """Recommend budget allocation across channels"""
    joiner = CrossChannelJoiner.from_env()
    
    # Get historical performance
    historical = joiner.join_data(
        start_date='2026-01-01',
        end_date='2026-07-31'
    )
    
    # Initialize optimizer
    optimizer = BudgetOptimizer(historical_data=historical)
    
    # Calculate optimal allocation
    recommendations = optimizer.optimize(
        total_budget=total_budget,
        objective='maximize_roas',
        constraints=constraints or {}
    )
    
    return recommendations
```

## Troubleshooting

### Data Mismatch Issues

```python
# Enable debug mode for detailed join logs
joiner = CrossChannelJoiner(debug=True)

# Validate data before joining
validation = joiner.validate_sources()
if not validation.is_valid:
    print(f"Data issues found: {validation.errors}")

# Check join key coverage
coverage = joiner.analyze_join_coverage(
    ad_data=ad_data,
    retail_data=retail_data,
    join_key='product_sku'
)
print(f"Join coverage: {coverage.match_rate * 100}%")
```

### Handling Missing Data

```python
# Configure fallback strategies
joiner.set_fallback_strategy(
    missing_ad_data='use_previous',
    missing_retail_data='exclude',
    missing_join_key='fuzzy_match'
)

# Impute missing values
from genpark_data_joiner.preprocessing import DataImputer

imputer = DataImputer(strategy='forward_fill')
cleaned_data = imputer.fit_transform(results.raw_data)
```

### Attribution Window Adjustments

```python
# Test different attribution windows
for window_days in [7, 14, 30, 60]:
    joiner.set_attribution_window(window_days)
    roas = joiner.calculate_roas(
        start_date='2026-07-01',
        end_date='2026-07-31'
    )
    print(f"{window_days}-day window ROAS: {roas:.2f}")
```

## API Reference

### CrossChannelJoiner Methods

- `add_source(name, source)` - Add data source
- `join_data(start_date, end_date, join_key)` - Execute data join
- `calculate_roas()` - Calculate return on ad spend
- `set_attribution_model(model)` - Configure attribution logic
- `export_results(results, output_file, format)` - Export data

### Results Object Methods

- `total_ad_spend()` - Sum of all ad spend
- `total_revenue()` - Sum of attributed revenue
- `roas_breakdown(dimension)` - ROAS by dimension
- `top_performing_channel()` - Highest ROAS channel
- `conversion_count()` - Number of conversions

## Best Practices

1. **Always use environment variables** for credentials
2. **Set appropriate attribution windows** based on your sales cycle
3. **Regularly validate** join key coverage and data quality
4. **Test attribution models** with historical data before production use
5. **Monitor data freshness** to ensure timely ROAS calculations
6. **Document custom join rules** for team consistency
