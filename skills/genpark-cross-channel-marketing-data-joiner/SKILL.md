---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data from multiple channels
  - combine ad spend with retail sales data
  - calculate cross-channel ROAS
  - merge programmatic advertising with inventory data
  - integrate marketing and sales data sources
  - unify ad performance with retail metrics
  - connect ad platforms to retail systems
  - aggregate marketing attribution data
---

# GenPark Cross-Channel Marketing Data Joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

GenPark Cross-Channel Marketing Data Joiner is a Python skill that joins programmatic advertising data with retail sales and inventory systems to calculate comprehensive Return on Ad Spend (ROAS) metrics. It enables marketers to connect disparate data sources—Google Ads, Facebook Ads, retail POS systems, inventory management—into a unified view for accurate attribution and performance analysis.

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

The data joiner works by:
1. **Extracting** data from multiple marketing and retail sources
2. **Transforming** disparate data formats into a common schema
3. **Joining** datasets on common keys (SKU, date, campaign ID, etc.)
4. **Calculating** unified ROAS metrics across all channels

## Basic Usage

```python
from genpark_data_joiner import DataJoiner, DataSource

# Initialize the joiner
joiner = DataJoiner()

# Add programmatic ad data source
ad_source = DataSource(
    name="google_ads",
    type="programmatic",
    connection_string=os.getenv("GOOGLE_ADS_CONNECTION")
)

# Add retail sales data source
retail_source = DataSource(
    name="pos_system",
    type="retail",
    connection_string=os.getenv("RETAIL_DB_CONNECTION")
)

# Add inventory data source
inventory_source = DataSource(
    name="inventory_management",
    type="inventory",
    connection_string=os.getenv("INVENTORY_API_ENDPOINT")
)

# Register sources
joiner.add_source(ad_source)
joiner.add_source(retail_source)
joiner.add_source(inventory_source)

# Execute join and calculate ROAS
results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    join_keys=["sku", "date"],
    metrics=["roas", "revenue", "ad_spend", "units_sold"]
)

print(f"Total ROAS: {results.total_roas}")
print(f"Revenue: ${results.revenue}")
print(f"Ad Spend: ${results.ad_spend}")
```

## Configuration

Create a `config.yaml` file to define your data sources and join logic:

```yaml
sources:
  - name: google_ads
    type: programmatic
    connection:
      type: api
      endpoint: https://googleads.googleapis.com/v15
      auth_env: GOOGLE_ADS_API_KEY
    fields:
      campaign_id: campaign.id
      ad_spend: metrics.cost_micros
      date: segments.date
      sku: campaign.custom_parameter.sku

  - name: facebook_ads
    type: programmatic
    connection:
      type: api
      endpoint: https://graph.facebook.com/v18.0
      auth_env: FACEBOOK_ACCESS_TOKEN
    fields:
      campaign_id: campaign_id
      ad_spend: spend
      date: date_start
      sku: custom_conversion.sku

  - name: retail_pos
    type: retail
    connection:
      type: database
      driver: postgresql
      host_env: RETAIL_DB_HOST
      database_env: RETAIL_DB_NAME
      user_env: RETAIL_DB_USER
      password_env: RETAIL_DB_PASSWORD
    fields:
      sku: product_sku
      revenue: total_sales
      units_sold: quantity
      date: sale_date

  - name: inventory_system
    type: inventory
    connection:
      type: api
      endpoint_env: INVENTORY_API_ENDPOINT
      auth_env: INVENTORY_API_KEY
    fields:
      sku: item_code
      stock_level: quantity_on_hand
      date: inventory_date

join_config:
  primary_keys:
    - sku
    - date
  join_type: inner
  date_format: "%Y-%m-%d"

metrics:
  roas:
    formula: "revenue / ad_spend"
    description: "Return on Ad Spend"
  
  revenue_per_unit:
    formula: "revenue / units_sold"
    description: "Average revenue per unit sold"
  
  inventory_turnover:
    formula: "units_sold / stock_level"
    description: "Inventory turnover rate"
```

Load configuration:

```python
from genpark_data_joiner import DataJoiner

joiner = DataJoiner.from_config("config.yaml")
results = joiner.execute()
```

## Advanced Usage

### Custom Data Transformers

Apply custom transformations before joining:

```python
from genpark_data_joiner import DataJoiner, Transformer

def normalize_sku(sku):
    """Normalize SKU formats across systems"""
    return sku.upper().replace("-", "").strip()

def convert_currency(amount, from_currency, to_currency):
    """Convert currency values"""
    # Implementation here
    return converted_amount

# Create custom transformer
transformer = Transformer()
transformer.add_field_transform("sku", normalize_sku)
transformer.add_field_transform("ad_spend", 
    lambda x: convert_currency(x, "EUR", "USD"))

joiner.add_transformer("google_ads", transformer)
```

### Multi-Touch Attribution

Calculate attribution across multiple touchpoints:

```python
from genpark_data_joiner import AttributionModel

# Define attribution model
attribution = AttributionModel(
    model_type="time_decay",
    lookback_window=30,  # days
    decay_rate=0.5
)

joiner.set_attribution_model(attribution)

results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    attribution=True
)

# Access attribution data
for touchpoint in results.attribution_path:
    print(f"Channel: {touchpoint.channel}, Credit: {touchpoint.credit}%")
```

### Batch Processing

Process large datasets in batches:

```python
from genpark_data_joiner import DataJoiner, BatchProcessor

joiner = DataJoiner.from_config("config.yaml")
processor = BatchProcessor(joiner, batch_size=10000)

# Process in chunks
for batch_results in processor.process_date_range(
    start_date="2026-01-01",
    end_date="2026-07-31"
):
    print(f"Batch ROAS: {batch_results.roas}")
    batch_results.save_to_csv(f"output_{batch_results.batch_id}.csv")
```

### Export Results

```python
# Export to CSV
results.to_csv("marketing_roas_report.csv")

# Export to JSON
results.to_json("marketing_roas_report.json")

# Export to database
results.to_database(
    connection_string=os.getenv("OUTPUT_DB_CONNECTION"),
    table_name="marketing_roas"
)

# Export to data warehouse
results.to_bigquery(
    project_id=os.getenv("GCP_PROJECT_ID"),
    dataset_id="marketing_analytics",
    table_id="cross_channel_roas",
    credentials_path=os.getenv("GCP_CREDENTIALS_PATH")
)
```

## Common Patterns

### Daily ROAS Calculation

```python
from genpark_data_joiner import DataJoiner
from datetime import datetime, timedelta

joiner = DataJoiner.from_config("config.yaml")

# Calculate yesterday's ROAS
yesterday = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")
results = joiner.join_and_calculate(
    date_range=(yesterday, yesterday),
    join_keys=["sku", "date"]
)

print(f"Yesterday's ROAS: {results.total_roas:.2f}")
```

### Campaign-Level Analysis

```python
# Join data by campaign
results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    join_keys=["campaign_id", "date"],
    group_by=["campaign_id", "campaign_name"]
)

# Analyze each campaign
for campaign in results.groups:
    print(f"Campaign: {campaign.campaign_name}")
    print(f"  ROAS: {campaign.roas:.2f}")
    print(f"  Revenue: ${campaign.revenue:,.2f}")
    print(f"  Ad Spend: ${campaign.ad_spend:,.2f}")
```

### SKU-Level Performance

```python
# Join at SKU level
results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    join_keys=["sku"],
    group_by=["sku", "product_name"]
)

# Find top performing SKUs
top_skus = results.sort_by("roas", ascending=False).head(10)

for sku in top_skus:
    print(f"SKU: {sku.sku}, ROAS: {sku.roas:.2f}, Units Sold: {sku.units_sold}")
```

### Incremental ROAS

Calculate incremental impact of advertising:

```python
from genpark_data_joiner import IncrementalAnalyzer

analyzer = IncrementalAnalyzer(joiner)

incremental_results = analyzer.calculate_incremental_roas(
    test_campaigns=["campaign_123", "campaign_456"],
    control_group="no_ads",
    date_range=("2026-07-01", "2026-07-31")
)

print(f"Incremental ROAS: {incremental_results.incremental_roas:.2f}")
print(f"Lift: {incremental_results.lift_percentage:.1f}%")
```

## Environment Variables

Set these environment variables for authentication and configuration:

```bash
# Google Ads
export GOOGLE_ADS_API_KEY="your_google_ads_api_key"
export GOOGLE_ADS_CONNECTION="your_connection_string"

# Facebook Ads
export FACEBOOK_ACCESS_TOKEN="your_facebook_access_token"

# Retail Database
export RETAIL_DB_HOST="your_db_host"
export RETAIL_DB_NAME="your_db_name"
export RETAIL_DB_USER="your_db_user"
export RETAIL_DB_PASSWORD="your_db_password"
export RETAIL_DB_CONNECTION="postgresql://user:pass@host:5432/dbname"

# Inventory System
export INVENTORY_API_ENDPOINT="https://api.inventory.example.com"
export INVENTORY_API_KEY="your_inventory_api_key"

# Output/Warehouse
export OUTPUT_DB_CONNECTION="your_output_connection"
export GCP_PROJECT_ID="your_gcp_project"
export GCP_CREDENTIALS_PATH="/path/to/credentials.json"
```

## Troubleshooting

### Data Source Connection Failures

```python
# Test individual source connections
from genpark_data_joiner import DataJoiner

joiner = DataJoiner.from_config("config.yaml")

for source in joiner.sources:
    try:
        connection_status = source.test_connection()
        print(f"{source.name}: {'✓' if connection_status else '✗'}")
    except Exception as e:
        print(f"{source.name}: Error - {str(e)}")
```

### Join Key Mismatches

```python
# Validate join keys before processing
validation = joiner.validate_join_keys(
    date_range=("2026-07-01", "2026-07-31"),
    join_keys=["sku", "date"]
)

print(f"Matched records: {validation.matched_count}")
print(f"Unmatched from source 1: {validation.unmatched_source1}")
print(f"Unmatched from source 2: {validation.unmatched_source2}")

# Inspect unmatched records
for record in validation.unmatched_records[:10]:
    print(f"SKU: {record.sku}, Date: {record.date}, Source: {record.source}")
```

### Data Type Inconsistencies

```python
# Enable automatic type coercion
joiner.set_option("auto_coerce_types", True)

# Or define explicit type mappings
type_mapping = {
    "ad_spend": float,
    "revenue": float,
    "units_sold": int,
    "date": "datetime64",
    "sku": str
}

joiner.set_type_mapping(type_mapping)
```

### Performance Optimization

```python
# Enable caching for repeated queries
joiner.enable_cache(ttl=3600)  # 1 hour TTL

# Use parallel processing
joiner.set_option("parallel_processing", True)
joiner.set_option("max_workers", 4)

# Sample data for testing
results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    sample_size=1000  # Process only 1000 records for testing
)
```

### Debugging

```python
# Enable debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

joiner.set_option("debug", True)
joiner.set_option("verbose", True)

# Inspect intermediate results
results = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    debug=True
)

# Access raw data before joining
print("Ad Data Sample:", results.debug_info.ad_data.head())
print("Retail Data Sample:", results.debug_info.retail_data.head())
print("Join Statistics:", results.debug_info.join_stats)
```

## API Reference

Key classes and methods:

- `DataJoiner`: Main class for joining data sources
- `DataSource`: Represents a data source (ads, retail, inventory)
- `Transformer`: Apply transformations to data fields
- `AttributionModel`: Define attribution logic
- `BatchProcessor`: Process large datasets in batches
- `IncrementalAnalyzer`: Calculate incremental impact

For complete API documentation, visit https://genpark.ai
