---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data across channels
  - calculate total ROAS from ad and retail data
  - combine programmatic ad data with inventory
  - merge cross-channel marketing metrics
  - integrate ad spend with retail performance
  - consolidate marketing and sales data for ROAS
  - link programmatic campaigns to retail outcomes
  - unify advertising and inventory data
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that merges programmatic advertising data with retail and inventory systems to provide a unified view of Return on Ad Spend (ROAS). It's designed for marketing teams that need to connect online ad performance with offline or retail sales data.

**Key capabilities:**
- Join programmatic ad platform data (Google Ads, Facebook, etc.) with retail sales
- Correlate inventory levels with campaign performance
- Calculate true ROAS accounting for both digital and physical channels
- Handle time-series alignment across disparate data sources
- Export unified datasets for BI tools and dashboards

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

**Dependencies typically include:**
```bash
pip install pandas numpy python-dateutil requests
```

## Configuration

Set up your environment variables for API access:

```bash
export GENPARK_API_KEY=your_api_key_here
export AD_PLATFORM_API_KEY=your_ad_platform_key
export RETAIL_API_KEY=your_retail_system_key
export INVENTORY_API_KEY=your_inventory_key
```

Create a configuration file `config.json`:

```json
{
  "data_sources": {
    "programmatic_ads": {
      "type": "api",
      "endpoint": "https://api.adplatform.com/v1/campaigns"
    },
    "retail_sales": {
      "type": "csv",
      "path": "data/retail_sales.csv"
    },
    "inventory": {
      "type": "database",
      "connection_string": "postgresql://localhost/inventory"
    }
  },
  "join_keys": {
    "time_column": "date",
    "product_id": "sku",
    "campaign_id": "campaign_id"
  },
  "roas_calculation": {
    "revenue_column": "total_sales",
    "spend_column": "ad_spend",
    "attribution_window_days": 7
  }
}
```

## Basic Usage

### Importing the Library

```python
from genpark_data_joiner import DataJoiner, ROASCalculator
from genpark_data_joiner.sources import AdPlatformSource, RetailSource, InventorySource
import os

# Initialize the data joiner
joiner = DataJoiner(config_path="config.json")
```

### Loading Data Sources

```python
# Load programmatic ad data
ad_source = AdPlatformSource(
    api_key=os.getenv("AD_PLATFORM_API_KEY"),
    start_date="2026-07-01",
    end_date="2026-07-31"
)
ad_data = ad_source.fetch()

# Load retail sales data
retail_source = RetailSource(
    api_key=os.getenv("RETAIL_API_KEY"),
    start_date="2026-07-01",
    end_date="2026-07-31"
)
retail_data = retail_source.fetch()

# Load inventory data
inventory_source = InventorySource(
    api_key=os.getenv("INVENTORY_API_KEY"),
    start_date="2026-07-01",
    end_date="2026-07-31"
)
inventory_data = inventory_source.fetch()
```

### Joining Data

```python
# Join all data sources
unified_data = joiner.join(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_type="outer",  # outer, inner, left, right
    time_alignment="daily"  # daily, weekly, monthly
)

print(f"Joined {len(unified_data)} records")
print(unified_data.head())
```

### Calculating ROAS

```python
# Initialize ROAS calculator
roas_calc = ROASCalculator(attribution_window=7)

# Calculate total ROAS
results = roas_calc.calculate(
    data=unified_data,
    spend_column="ad_spend",
    revenue_column="total_sales",
    group_by=["campaign_id", "product_sku"]
)

print(f"Overall ROAS: {results['total_roas']:.2f}")
print(f"Total Spend: ${results['total_spend']:,.2f}")
print(f"Total Revenue: ${results['total_revenue']:,.2f}")

# View by campaign
campaign_roas = results['by_campaign']
print(campaign_roas.sort_values('roas', ascending=False).head(10))
```

## Advanced Patterns

### Custom Attribution Models

```python
from genpark_data_joiner.attribution import AttributionModel

# Linear attribution
linear_model = AttributionModel(
    model_type="linear",
    window_days=14
)

# Time decay attribution
time_decay_model = AttributionModel(
    model_type="time_decay",
    window_days=30,
    decay_rate=0.5
)

# Apply attribution
attributed_data = linear_model.apply(
    ad_data=ad_data,
    conversion_data=retail_data,
    match_key="customer_id"
)

# Calculate ROAS with attribution
attributed_roas = roas_calc.calculate(
    data=attributed_data,
    spend_column="attributed_spend",
    revenue_column="revenue"
)
```

### Handling Time Zones and Alignment

```python
from genpark_data_joiner.utils import TimeAligner

aligner = TimeAligner(
    source_timezone="America/New_York",
    target_timezone="UTC"
)

# Align timestamps across sources
ad_data_aligned = aligner.align(
    ad_data,
    timestamp_column="click_time"
)

retail_data_aligned = aligner.align(
    retail_data,
    timestamp_column="purchase_time"
)

# Join with time window
unified_data = joiner.join_with_window(
    ad_data=ad_data_aligned,
    retail_data=retail_data_aligned,
    window_hours=24,
    match_keys=["customer_id", "product_sku"]
)
```

### Incremental Data Processing

```python
from genpark_data_joiner import IncrementalJoiner

# Initialize incremental joiner with state persistence
incremental_joiner = IncrementalJoiner(
    state_file="data/joiner_state.json",
    checkpoint_interval=1000
)

# Process data in batches
for batch in incremental_joiner.process_batches(
    ad_source=ad_source,
    retail_source=retail_source,
    batch_size=5000
):
    # Process each batch
    joined_batch = incremental_joiner.join_batch(batch)
    
    # Calculate metrics
    batch_roas = roas_calc.calculate(joined_batch)
    
    # Save or stream results
    joined_batch.to_csv(
        f"output/joined_batch_{batch['batch_id']}.csv",
        index=False
    )
    
    print(f"Processed batch {batch['batch_id']}, ROAS: {batch_roas['total_roas']:.2f}")
```

### Data Quality and Validation

```python
from genpark_data_joiner.validation import DataValidator

validator = DataValidator()

# Validate data quality before joining
ad_validation = validator.validate(
    ad_data,
    required_columns=["campaign_id", "date", "spend", "impressions"],
    date_column="date",
    numeric_columns=["spend", "impressions"]
)

if not ad_validation.is_valid:
    print("Ad data validation errors:")
    for error in ad_validation.errors:
        print(f"  - {error}")

# Check for join key overlap
overlap_report = validator.check_join_overlap(
    left_data=ad_data,
    right_data=retail_data,
    join_keys=["date", "product_sku"]
)

print(f"Join overlap: {overlap_report['overlap_percentage']:.1f}%")
print(f"Unmatched ad records: {overlap_report['left_unmatched']}")
print(f"Unmatched retail records: {overlap_report['right_unmatched']}")
```

### Export and Visualization

```python
from genpark_data_joiner.export import DataExporter

exporter = DataExporter()

# Export to multiple formats
exporter.to_csv(unified_data, "output/unified_data.csv")
exporter.to_parquet(unified_data, "output/unified_data.parquet")
exporter.to_json(unified_data, "output/unified_data.json", orient="records")

# Export ROAS summary for BI tools
exporter.to_bi_format(
    results,
    output_path="output/roas_dashboard.csv",
    format="tableau"  # tableau, powerbi, looker
)

# Generate HTML report
from genpark_data_joiner.reporting import HTMLReporter

reporter = HTMLReporter()
report_html = reporter.generate(
    unified_data=unified_data,
    roas_results=results,
    include_charts=True,
    template="executive_summary"
)

with open("output/marketing_report.html", "w") as f:
    f.write(report_html)
```

## Common Use Cases

### E-commerce ROAS Tracking

```python
# Join online ads with e-commerce sales
ecommerce_joiner = DataJoiner(config_path="config_ecommerce.json")

# Fetch data
shopify_data = RetailSource(type="shopify", api_key=os.getenv("SHOPIFY_API_KEY")).fetch()
facebook_ads = AdPlatformSource(type="facebook", api_key=os.getenv("FACEBOOK_API_KEY")).fetch()
google_ads = AdPlatformSource(type="google", api_key=os.getenv("GOOGLE_ADS_API_KEY")).fetch()

# Combine ad sources
all_ads = ecommerce_joiner.combine_ad_sources([facebook_ads, google_ads])

# Join with sales
unified = ecommerce_joiner.join(
    ad_data=all_ads,
    retail_data=shopify_data,
    match_on=["customer_email", "order_id"],
    attribution_window=7
)

# Calculate channel-specific ROAS
channel_roas = roas_calc.calculate_by_channel(unified)
print(channel_roas)
```

### Retail Store Performance

```python
# Join digital ads with in-store purchases
retail_joiner = DataJoiner(config_path="config_retail.json")

# Use geolocation to match campaigns with stores
unified_retail = retail_joiner.join_with_geo(
    ad_data=ad_data,
    retail_data=pos_data,
    inventory_data=inventory_data,
    geo_match_radius_miles=25
)

# Calculate ROAS by store location
store_roas = roas_calc.calculate(
    unified_retail,
    group_by=["store_id", "campaign_id"]
)

# Identify best-performing store/campaign combinations
top_performers = store_roas.nlargest(10, "roas")
print(top_performers)
```

## Troubleshooting

### Data Not Joining

```python
# Debug join issues
from genpark_data_joiner.debug import JoinDebugger

debugger = JoinDebugger()
debug_report = debugger.analyze_join(
    left_data=ad_data,
    right_data=retail_data,
    join_keys=["date", "product_sku"]
)

print(debug_report.summary())
# Shows: key mismatches, type inconsistencies, null values, etc.
```

### Performance Optimization

```python
# For large datasets, use chunked processing
joiner.set_chunk_size(10000)
joiner.enable_parallel_processing(n_workers=4)

# Use columnar storage
joiner.use_parquet_backend()

# Optimize memory usage
joiner.set_memory_limit_mb(4096)
```

### Handling Missing Data

```python
# Configure missing data handling
joiner.set_missing_data_strategy(
    strategy="interpolate",  # interpolate, forward_fill, drop
    columns=["ad_spend", "impressions"]
)

# Impute missing values before calculation
from genpark_data_joiner.preprocessing import MissingDataHandler

handler = MissingDataHandler()
cleaned_data = handler.handle_missing(
    unified_data,
    numeric_strategy="median",
    categorical_strategy="mode"
)
```

## API Reference Summary

**Core Classes:**
- `DataJoiner`: Main class for joining data sources
- `ROASCalculator`: Calculate ROAS metrics
- `AdPlatformSource`: Connect to ad platforms
- `RetailSource`: Connect to retail/POS systems
- `InventorySource`: Connect to inventory systems
- `AttributionModel`: Apply attribution logic
- `DataValidator`: Validate data quality
- `DataExporter`: Export results

**Key Methods:**
- `joiner.join()`: Join multiple data sources
- `roas_calc.calculate()`: Calculate ROAS metrics
- `source.fetch()`: Retrieve data from source
- `validator.validate()`: Check data quality
- `exporter.to_csv()`: Export results

For complete API documentation, see the project repository.
