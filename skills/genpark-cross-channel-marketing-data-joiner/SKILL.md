---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join ad data with sales data
  - calculate cross-channel ROAS
  - merge programmatic advertising with retail data
  - combine marketing data from multiple sources
  - integrate ad spend with inventory metrics
  - analyze total return on ad spend
  - unify cross-channel marketing performance
  - consolidate ad and retail analytics
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner combines programmatic advertising data with retail and inventory systems to provide unified ROAS (Return on Ad Spend) calculations. This skill enables marketing analysts to correlate ad spend across channels with actual sales and inventory movement.

## Installation

```bash
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill
pip install -r requirements.txt
```

Or install as a package:

```bash
pip install genpark-cross-channel-marketing-data-joiner
```

## Core Functionality

### Key Features
- **Multi-source data joining**: Combine data from programmatic ad platforms, retail systems, and inventory databases
- **ROAS calculation**: Compute total return on ad spend across all channels
- **Attribution modeling**: Map ad impressions/clicks to sales conversions
- **Inventory correlation**: Link marketing campaigns to inventory movement
- **Time-series alignment**: Synchronize data across different time zones and reporting periods

## Basic Usage

```python
from genpark_data_joiner import DataJoiner, ROASCalculator

# Initialize the data joiner
joiner = DataJoiner(
    ad_platform_api_key=os.environ.get('AD_PLATFORM_API_KEY'),
    retail_api_key=os.environ.get('RETAIL_API_KEY'),
    inventory_api_key=os.environ.get('INVENTORY_API_KEY')
)

# Load data sources
ad_data = joiner.load_ad_data(
    start_date='2026-07-01',
    end_date='2026-07-31',
    channels=['google_ads', 'facebook_ads', 'programmatic']
)

retail_data = joiner.load_retail_data(
    start_date='2026-07-01',
    end_date='2026-07-31'
)

inventory_data = joiner.load_inventory_data(
    start_date='2026-07-01',
    end_date='2026-07-31'
)

# Join datasets
unified_data = joiner.join_datasets(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_key='product_id',
    attribution_window=7  # days
)

# Calculate ROAS
calculator = ROASCalculator(unified_data)
total_roas = calculator.calculate_total_roas()
channel_roas = calculator.calculate_channel_roas()

print(f"Total ROAS: {total_roas}")
print(f"Channel breakdown: {channel_roas}")
```

## Configuration

### Environment Variables

```bash
# Ad platform credentials
export AD_PLATFORM_API_KEY="your_ad_platform_key"
export AD_PLATFORM_ACCOUNT_ID="your_account_id"

# Retail system credentials
export RETAIL_API_KEY="your_retail_api_key"
export RETAIL_ENDPOINT="https://api.retail-system.com/v1"

# Inventory system credentials
export INVENTORY_API_KEY="your_inventory_key"
export INVENTORY_DATABASE_URL="postgresql://user:pass@host:5432/inventory"

# GenPark configuration
export GENPARK_API_ENDPOINT="https://genpark.ai/api"
```

### Configuration File

Create a `config.yaml`:

```yaml
data_sources:
  ad_platforms:
    - name: google_ads
      enabled: true
      attribution_model: last_click
    - name: facebook_ads
      enabled: true
      attribution_model: data_driven
    - name: programmatic
      enabled: true
      attribution_model: linear
  
  retail:
    sync_interval: 3600  # seconds
    currency: USD
  
  inventory:
    sync_interval: 1800
    warehouse_ids: [1, 2, 3]

roas_calculation:
  attribution_window_days: 7
  include_returns: true
  cost_basis: clicks  # or impressions
  
output:
  format: json  # or csv, parquet
  destination: s3://your-bucket/output/
```

Load configuration:

```python
from genpark_data_joiner import load_config

config = load_config('config.yaml')
joiner = DataJoiner(config=config)
```

## Advanced Patterns

### Custom Attribution Models

```python
from genpark_data_joiner import AttributionModel

# Define custom attribution logic
class CustomAttribution(AttributionModel):
    def calculate_credit(self, touchpoints):
        # First touch gets 40%, last touch gets 40%, others split 20%
        if len(touchpoints) == 1:
            return {touchpoints[0]: 1.0}
        
        credits = {}
        middle_credit = 0.2 / (len(touchpoints) - 2) if len(touchpoints) > 2 else 0
        
        for i, touchpoint in enumerate(touchpoints):
            if i == 0:
                credits[touchpoint] = 0.4
            elif i == len(touchpoints) - 1:
                credits[touchpoint] = 0.4
            else:
                credits[touchpoint] = middle_credit
        
        return credits

# Use custom attribution
joiner = DataJoiner()
joiner.set_attribution_model(CustomAttribution())
unified_data = joiner.join_datasets(ad_data, retail_data, inventory_data)
```

### Filtering and Segmentation

```python
# Filter by product category
filtered_data = unified_data.filter_by_category('electronics')

# Segment by customer cohort
cohort_analysis = calculator.calculate_roas_by_cohort(
    cohort_field='customer_acquisition_date',
    cohort_periods=['2026-01', '2026-02', '2026-03']
)

# Geographic segmentation
geo_roas = calculator.calculate_roas_by_geography(
    geographic_level='state'  # or city, zip, country
)
```

### Batch Processing

```python
from genpark_data_joiner import BatchProcessor

processor = BatchProcessor(
    config=config,
    batch_size=10000,
    parallel_workers=4
)

# Process large datasets in batches
results = processor.process_date_range(
    start_date='2026-01-01',
    end_date='2026-07-31',
    output_path='s3://your-bucket/results/'
)

for batch_result in results:
    print(f"Batch {batch_result.batch_id}: ROAS = {batch_result.roas}")
```

### Real-time Data Streaming

```python
from genpark_data_joiner import StreamingJoiner

# Set up streaming pipeline
streaming_joiner = StreamingJoiner(
    ad_stream_endpoint=os.environ.get('AD_STREAM_ENDPOINT'),
    retail_stream_endpoint=os.environ.get('RETAIL_STREAM_ENDPOINT')
)

# Process streaming data
@streaming_joiner.on_data_received
def process_streaming_data(event):
    # Join streaming ad event with retail data
    joined_event = streaming_joiner.join_event(event)
    
    # Update running ROAS calculation
    current_roas = streaming_joiner.get_current_roas()
    print(f"Current ROAS: {current_roas}")

streaming_joiner.start()
```

## Data Export

```python
# Export to CSV
unified_data.to_csv('marketing_data_joined.csv')

# Export to Parquet
unified_data.to_parquet('marketing_data_joined.parquet')

# Export to BigQuery
unified_data.to_bigquery(
    project_id=os.environ.get('GCP_PROJECT_ID'),
    dataset='marketing_analytics',
    table='unified_roas'
)

# Export to data warehouse
unified_data.to_warehouse(
    connection_string=os.environ.get('WAREHOUSE_CONNECTION_STRING'),
    table_name='marketing.unified_data',
    if_exists='replace'  # or append
)
```

## Reporting and Visualization

```python
from genpark_data_joiner import ReportGenerator

generator = ReportGenerator(unified_data)

# Generate executive summary
summary = generator.create_executive_summary(
    metrics=['roas', 'cpa', 'conversion_rate', 'revenue'],
    time_period='monthly'
)

# Create channel comparison report
channel_report = generator.compare_channels(
    channels=['google_ads', 'facebook_ads', 'programmatic'],
    metrics=['spend', 'conversions', 'roas']
)

# Export report
generator.export_report(
    format='pdf',
    output_path='reports/monthly_roas_report.pdf',
    include_charts=True
)
```

## Troubleshooting

### Data Sync Issues

```python
# Validate data sources before joining
validator = joiner.validate_data_sources()

if not validator.ad_data_valid:
    print(f"Ad data issues: {validator.ad_data_errors}")

if not validator.retail_data_valid:
    print(f"Retail data issues: {validator.retail_data_errors}")

# Manual sync if needed
joiner.force_sync_ad_data()
joiner.force_sync_retail_data()
```

### Handling Missing Data

```python
# Configure missing data handling
joiner.set_missing_data_strategy(
    strategy='interpolate',  # or 'drop', 'forward_fill', 'zero'
    threshold=0.05  # drop if >5% missing
)

# Check data completeness
completeness_report = joiner.check_data_completeness()
print(f"Ad data completeness: {completeness_report['ad_data']}%")
print(f"Retail data completeness: {completeness_report['retail_data']}%")
```

### Performance Optimization

```python
# Enable caching for repeated queries
joiner.enable_cache(
    cache_backend='redis',
    cache_ttl=3600,
    redis_url=os.environ.get('REDIS_URL')
)

# Use query optimization
joiner.optimize_queries(
    use_indexes=True,
    batch_size=5000,
    prefetch_related=['products', 'campaigns']
)
```

### Debugging Attribution

```python
# Enable detailed attribution logging
joiner.set_log_level('DEBUG')

# Trace specific conversion
conversion_trace = joiner.trace_conversion(
    order_id='ORDER-12345',
    include_all_touchpoints=True
)

print(f"Touchpoints: {conversion_trace.touchpoints}")
print(f"Attribution credits: {conversion_trace.credits}")
print(f"Total attributed spend: {conversion_trace.total_spend}")
```

## Common Use Cases

### Multi-touch Attribution Analysis

```python
# Compare different attribution models
models = ['first_click', 'last_click', 'linear', 'time_decay', 'data_driven']
model_comparison = {}

for model in models:
    joiner.set_attribution_model(model)
    unified_data = joiner.join_datasets(ad_data, retail_data, inventory_data)
    calculator = ROASCalculator(unified_data)
    model_comparison[model] = calculator.calculate_total_roas()

print("ROAS by attribution model:")
for model, roas in model_comparison.items():
    print(f"  {model}: {roas:.2f}")
```

### Inventory-Aware Campaign Optimization

```python
# Find campaigns driving sales of overstocked items
overstocked_products = inventory_data[inventory_data['stock_level'] > inventory_data['optimal_stock']]

campaign_performance = unified_data[
    unified_data['product_id'].isin(overstocked_products['product_id'])
].groupby('campaign_id').agg({
    'spend': 'sum',
    'revenue': 'sum',
    'conversions': 'sum'
})

campaign_performance['roas'] = campaign_performance['revenue'] / campaign_performance['spend']
top_campaigns = campaign_performance.nlargest(10, 'roas')

print("Top campaigns for moving overstock:")
print(top_campaigns)
```

This skill enables AI agents to help developers integrate, analyze, and optimize cross-channel marketing performance with unified ROAS calculations.
