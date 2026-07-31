---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate unified ROAS across marketing channels
triggers:
  - combine marketing data from different channels
  - join programmatic ad data with sales data
  - calculate cross-channel ROAS
  - merge advertising and retail data
  - unify marketing performance metrics
  - integrate ad spend with inventory data
  - calculate total return on ad spend
  - connect programmatic ads to retail performance
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python-based skill that consolidates programmatic advertising data with retail sales and inventory information to provide unified Return on Ad Spend (ROAS) metrics. This enables marketers to understand the true impact of their advertising across multiple channels and platforms.

## Installation

```bash
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill
pip install -r requirements.txt
```

## Core Functionality

The skill provides data joining capabilities for:

- **Programmatic Ad Platforms**: Google Ads, Facebook Ads, Display & Video 360, Trade Desk
- **Retail Systems**: Point-of-sale data, e-commerce platforms, CRM systems
- **Inventory Management**: Stock levels, product catalogs, SKU mapping

## Basic Usage

```python
from genpark_data_joiner import MarketingDataJoiner, DataSource

# Initialize the joiner
joiner = MarketingDataJoiner()

# Add programmatic ad data source
ad_source = DataSource(
    name="google_ads",
    type="programmatic",
    data_path="data/google_ads_export.csv"
)
joiner.add_source(ad_source)

# Add retail data source
retail_source = DataSource(
    name="shopify_sales",
    type="retail",
    data_path="data/shopify_sales.csv"
)
joiner.add_source(retail_source)

# Join data on common keys
result = joiner.join_data(
    join_keys=["date", "product_id"],
    metrics=["ad_spend", "impressions", "clicks", "revenue", "units_sold"]
)

# Calculate ROAS
roas = joiner.calculate_roas(result)
print(f"Total ROAS: {roas}")
```

## Configuration

Create a `config.json` file to define data sources and join strategies:

```json
{
  "sources": [
    {
      "name": "programmatic_ads",
      "type": "csv",
      "path": "data/ads.csv",
      "date_column": "campaign_date",
      "id_columns": ["campaign_id", "product_sku"]
    },
    {
      "name": "retail_sales",
      "type": "csv",
      "path": "data/sales.csv",
      "date_column": "transaction_date",
      "id_columns": ["sku"]
    },
    {
      "name": "inventory",
      "type": "api",
      "endpoint": "${INVENTORY_API_ENDPOINT}",
      "api_key_env": "INVENTORY_API_KEY"
    }
  ],
  "join_strategy": {
    "time_window": "7d",
    "attribution_model": "last_touch",
    "deduplication": true
  },
  "metrics": {
    "roas_formula": "(revenue - cogs) / ad_spend",
    "include_organic": false
  }
}
```

Load configuration:

```python
from genpark_data_joiner import MarketingDataJoiner

joiner = MarketingDataJoiner.from_config("config.json")
results = joiner.run()
```

## Advanced Data Joining

### Multi-Touch Attribution

```python
from genpark_data_joiner import AttributionModel

# Configure multi-touch attribution
joiner.set_attribution_model(
    AttributionModel.LINEAR,
    lookback_window_days=30
)

# Join with attribution
attributed_results = joiner.join_with_attribution(
    ad_sources=["google_ads", "facebook_ads", "display_360"],
    conversion_source="retail_sales",
    touchpoint_key="user_id"
)

# Calculate channel-specific ROAS
channel_roas = joiner.calculate_channel_roas(attributed_results)
for channel, roas in channel_roas.items():
    print(f"{channel}: {roas:.2f}")
```

### Time-Series Analysis

```python
from genpark_data_joiner import TimeSeriesAggregator

# Aggregate by time periods
aggregator = TimeSeriesAggregator(joiner)

daily_roas = aggregator.calculate_roas_by_period(
    period="daily",
    start_date="2026-07-01",
    end_date="2026-07-31"
)

weekly_roas = aggregator.calculate_roas_by_period(
    period="weekly",
    rolling_window=4
)
```

### Inventory Integration

```python
from genpark_data_joiner import InventoryEnricher

# Enrich with inventory data
enricher = InventoryEnricher(
    inventory_source="inventory_api",
    api_key_env="INVENTORY_API_KEY"
)

enriched_data = enricher.add_inventory_metrics(
    joined_data=result,
    metrics=["stock_level", "restock_date", "margin_pct"]
)

# Filter campaigns by inventory availability
high_stock_campaigns = enriched_data[enriched_data["stock_level"] > 100]
roas_high_stock = joiner.calculate_roas(high_stock_campaigns)
```

## Data Export and Reporting

```python
from genpark_data_joiner import ReportExporter

exporter = ReportExporter(joiner)

# Export to CSV
exporter.to_csv(
    data=result,
    output_path="reports/cross_channel_roas.csv"
)

# Export to dashboard format
exporter.to_dashboard_json(
    data=result,
    metrics=["roas", "cpa", "conversion_rate"],
    dimensions=["channel", "campaign", "product_category"],
    output_path="reports/dashboard_data.json"
)

# Generate summary report
summary = exporter.create_summary_report(
    data=result,
    include_charts=True,
    output_format="html",
    output_path="reports/marketing_summary.html"
)
```

## Common Patterns

### Pattern 1: Daily ROAS Monitoring

```python
import schedule
from genpark_data_joiner import MarketingDataJoiner

def calculate_daily_roas():
    joiner = MarketingDataJoiner.from_config("config.json")
    result = joiner.join_data(date_range="yesterday")
    roas = joiner.calculate_roas(result)
    
    # Send alert if ROAS drops
    if roas < 2.0:
        send_alert(f"ROAS Alert: {roas:.2f}")
    
    return roas

# Schedule daily
schedule.every().day.at("09:00").do(calculate_daily_roas)
```

### Pattern 2: Campaign Optimization

```python
from genpark_data_joiner import CampaignOptimizer

optimizer = CampaignOptimizer(joiner)

# Find underperforming campaigns
recommendations = optimizer.analyze_campaigns(
    min_roas=1.5,
    min_spend=1000,
    time_period="last_30_days"
)

for rec in recommendations:
    print(f"Campaign: {rec['campaign_name']}")
    print(f"Current ROAS: {rec['roas']}")
    print(f"Recommendation: {rec['action']}")
    print(f"Expected Impact: +{rec['expected_roas_increase']:.2f}")
```

### Pattern 3: Product-Level Analysis

```python
# Join at product SKU level
product_performance = joiner.join_data(
    join_keys=["date", "sku"],
    aggregate_by="sku"
)

# Calculate product-level metrics
product_metrics = joiner.calculate_metrics(
    data=product_performance,
    metrics={
        "roas": "(revenue - cogs) / ad_spend",
        "profit_margin": "(revenue - cogs - ad_spend) / revenue",
        "ad_efficiency": "revenue / impressions"
    }
)

# Sort by profitability
top_products = product_metrics.sort_values("profit_margin", ascending=False).head(20)
```

## Environment Variables

Set required environment variables for API connections:

```bash
# Inventory API
export INVENTORY_API_KEY=your_inventory_api_key
export INVENTORY_API_ENDPOINT=https://api.inventory.example.com

# Programmatic Ad Platforms
export GOOGLE_ADS_DEVELOPER_TOKEN=your_token
export FACEBOOK_ADS_ACCESS_TOKEN=your_token
export DV360_API_KEY=your_key

# Retail Platforms
export SHOPIFY_API_KEY=your_key
export SHOPIFY_API_SECRET=your_secret
```

## Troubleshooting

### Issue: Data Join Produces No Results

**Cause**: Mismatched join keys or date formats

**Solution**:
```python
# Inspect data schemas
joiner.inspect_source_schemas()

# Normalize date formats
joiner.normalize_dates(date_format="%Y-%m-%d")

# Fuzzy matching for product IDs
joiner.enable_fuzzy_matching(
    columns=["product_id", "sku"],
    threshold=0.85
)
```

### Issue: ROAS Calculation Seems Incorrect

**Cause**: Missing cost data or duplicate transactions

**Solution**:
```python
# Enable deduplication
joiner.set_deduplication(
    enabled=True,
    keys=["transaction_id", "order_id"]
)

# Validate cost data completeness
validation = joiner.validate_data(
    required_fields=["ad_spend", "revenue", "date"]
)

if not validation.is_complete:
    print(f"Missing data: {validation.missing_fields}")
```

### Issue: Slow Performance with Large Datasets

**Cause**: In-memory processing of large files

**Solution**:
```python
# Enable chunked processing
joiner.set_processing_mode(
    mode="chunked",
    chunk_size=10000
)

# Use database backend
joiner.set_backend(
    backend="postgresql",
    connection_string="postgresql://user:pass@localhost/marketing_db"
)
```

## API Reference

### Core Methods

- `MarketingDataJoiner.add_source(source)` - Add a data source
- `MarketingDataJoiner.join_data(join_keys, metrics)` - Join datasets
- `MarketingDataJoiner.calculate_roas(data)` - Calculate ROAS
- `MarketingDataJoiner.set_attribution_model(model, **kwargs)` - Configure attribution
- `MarketingDataJoiner.validate_data(required_fields)` - Validate completeness

### Data Source Types

- `csv` - CSV file
- `json` - JSON file
- `api` - REST API endpoint
- `database` - SQL database connection
- `bigquery` - Google BigQuery
- `s3` - AWS S3 bucket

## Best Practices

1. **Always deduplicate** transaction data to avoid inflated ROAS
2. **Set appropriate attribution windows** (typically 7-30 days)
3. **Normalize product identifiers** across systems before joining
4. **Monitor data freshness** to ensure timely insights
5. **Use incremental loading** for large historical datasets
