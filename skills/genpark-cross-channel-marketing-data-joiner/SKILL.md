---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data across channels
  - calculate total ROAS from ad spend
  - merge programmatic ads with retail data
  - combine inventory data with marketing metrics
  - analyze cross-channel marketing performance
  - integrate ad campaigns with sales data
  - connect programmatic advertising to retail results
  - unify marketing and inventory datasets
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that unifies programmatic advertising data with retail sales and inventory information to calculate accurate Return on Ad Spend (ROAS) across multiple marketing channels. It enables marketers to understand the true performance of their campaigns by connecting ad impressions, clicks, and spend with actual sales outcomes and inventory levels.

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

1. **Programmatic Ad Data**: Campaign metrics (impressions, clicks, spend, conversions)
2. **Retail Sales Data**: Transaction records with product SKUs, revenue, timestamps
3. **Inventory Data**: Stock levels, product metadata, pricing information

The joiner matches these datasets using common keys (SKU, timestamp, campaign ID) to produce unified analytics.

## Basic Usage

```python
from genpark_data_joiner import CrossChannelJoiner, DataSource

# Initialize the joiner
joiner = CrossChannelJoiner()

# Load data sources
ad_data = DataSource.from_csv('programmatic_ads.csv')
retail_data = DataSource.from_csv('retail_sales.csv')
inventory_data = DataSource.from_csv('inventory.csv')

# Join datasets
unified_data = joiner.join(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_keys=['sku', 'date'],
    attribution_window=7  # days
)

# Calculate ROAS
roas_metrics = joiner.calculate_roas(unified_data)
print(f"Total ROAS: {roas_metrics['total_roas']:.2f}")
```

## Data Source Configuration

### CSV Format

```python
from genpark_data_joiner import DataSource

# Load from CSV with custom configuration
ad_data = DataSource.from_csv(
    'ads.csv',
    date_column='timestamp',
    date_format='%Y-%m-%d',
    encoding='utf-8'
)
```

### Database Connection

```python
import os
from genpark_data_joiner import DataSource

# Load from database using environment variables
ad_data = DataSource.from_database(
    connection_string=os.getenv('DATABASE_URL'),
    query="SELECT * FROM programmatic_ads WHERE date >= '2026-01-01'"
)
```

### API Integration

```python
import os
from genpark_data_joiner import DataSource

# Pull data from marketing API
ad_data = DataSource.from_api(
    endpoint='https://api.adplatform.com/v1/campaigns',
    auth_token=os.getenv('AD_PLATFORM_API_KEY'),
    params={'start_date': '2026-01-01', 'end_date': '2026-01-31'}
)
```

## Advanced Joining Strategies

### Multi-Key Joins

```python
# Join on multiple dimensions
unified_data = joiner.join(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_keys=['sku', 'date', 'region'],
    join_type='left'  # Options: 'inner', 'left', 'right', 'outer'
)
```

### Attribution Windows

```python
# Configure attribution logic
unified_data = joiner.join(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_keys=['sku'],
    attribution_window=14,  # days
    attribution_model='last_touch'  # Options: 'last_touch', 'first_touch', 'linear', 'time_decay'
)
```

### Custom Matching Logic

```python
from genpark_data_joiner import MatchingStrategy

# Define custom matching function
def fuzzy_sku_match(ad_sku, retail_sku):
    return ad_sku.lower().strip() == retail_sku.lower().strip()

strategy = MatchingStrategy(
    match_function=fuzzy_sku_match,
    threshold=0.9
)

unified_data = joiner.join(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_keys=['sku'],
    matching_strategy=strategy
)
```

## ROAS Calculations

### Basic ROAS

```python
# Calculate overall ROAS
roas_metrics = joiner.calculate_roas(unified_data)
print(f"Total Revenue: ${roas_metrics['revenue']:,.2f}")
print(f"Total Spend: ${roas_metrics['spend']:,.2f}")
print(f"ROAS: {roas_metrics['total_roas']:.2f}x")
```

### Channel-Specific ROAS

```python
# Calculate ROAS by channel
channel_roas = joiner.calculate_roas(
    unified_data,
    group_by='channel'
)

for channel, metrics in channel_roas.items():
    print(f"{channel}: ROAS = {metrics['roas']:.2f}x")
```

### Time-Series ROAS

```python
# Calculate ROAS over time
time_series_roas = joiner.calculate_roas(
    unified_data,
    group_by='date',
    resample='W'  # Weekly aggregation: 'D' (daily), 'W' (weekly), 'M' (monthly)
)

# Export to visualization
time_series_roas.to_csv('roas_time_series.csv')
```

## Filtering and Preprocessing

### Data Validation

```python
from genpark_data_joiner import DataValidator

validator = DataValidator()

# Validate data before joining
ad_data_validated = validator.validate(
    ad_data,
    required_columns=['sku', 'date', 'spend', 'impressions'],
    drop_nulls=True,
    drop_duplicates=True
)
```

### Data Transformation

```python
from genpark_data_joiner import DataTransformer

transformer = DataTransformer()

# Normalize SKU formats
ad_data = transformer.normalize_column(
    ad_data,
    column='sku',
    transformation='uppercase'
)

# Add calculated fields
ad_data = transformer.add_calculated_field(
    ad_data,
    new_column='cpc',
    formula=lambda row: row['spend'] / row['clicks'] if row['clicks'] > 0 else 0
)
```

## Export and Reporting

### Export Unified Data

```python
# Export to CSV
joiner.export(unified_data, 'unified_marketing_data.csv', format='csv')

# Export to JSON
joiner.export(unified_data, 'unified_marketing_data.json', format='json')

# Export to Parquet (compressed)
joiner.export(unified_data, 'unified_marketing_data.parquet', format='parquet')
```

### Generate Reports

```python
from genpark_data_joiner import ReportGenerator

reporter = ReportGenerator()

# Generate summary report
report = reporter.generate_summary(
    unified_data,
    metrics=['roas', 'revenue', 'spend', 'conversions'],
    dimensions=['channel', 'campaign', 'product_category']
)

# Export report
reporter.save_report(report, 'marketing_performance_report.html')
```

## Complete Example

```python
import os
from genpark_data_joiner import (
    CrossChannelJoiner,
    DataSource,
    DataValidator,
    ReportGenerator
)

def analyze_marketing_performance():
    # Initialize components
    joiner = CrossChannelJoiner()
    validator = DataValidator()
    reporter = ReportGenerator()
    
    # Load data sources
    print("Loading data sources...")
    ad_data = DataSource.from_csv('data/programmatic_ads.csv')
    retail_data = DataSource.from_database(
        connection_string=os.getenv('RETAIL_DATABASE_URL'),
        query="SELECT * FROM sales WHERE date >= CURRENT_DATE - INTERVAL '30 days'"
    )
    inventory_data = DataSource.from_api(
        endpoint=os.getenv('INVENTORY_API_ENDPOINT'),
        auth_token=os.getenv('INVENTORY_API_TOKEN')
    )
    
    # Validate data
    print("Validating data...")
    ad_data = validator.validate(
        ad_data,
        required_columns=['sku', 'date', 'spend', 'impressions', 'clicks'],
        drop_nulls=True
    )
    
    # Join datasets
    print("Joining datasets...")
    unified_data = joiner.join(
        ad_data=ad_data,
        retail_data=retail_data,
        inventory_data=inventory_data,
        join_keys=['sku', 'date'],
        attribution_window=7,
        attribution_model='last_touch'
    )
    
    # Calculate ROAS metrics
    print("Calculating ROAS...")
    overall_roas = joiner.calculate_roas(unified_data)
    channel_roas = joiner.calculate_roas(unified_data, group_by='channel')
    
    # Generate report
    print("Generating report...")
    report = reporter.generate_summary(
        unified_data,
        metrics=['roas', 'revenue', 'spend', 'conversions', 'inventory_turns'],
        dimensions=['channel', 'campaign', 'product_category']
    )
    
    # Export results
    joiner.export(unified_data, 'output/unified_data.csv', format='csv')
    reporter.save_report(report, 'output/performance_report.html')
    
    print(f"Analysis complete!")
    print(f"Overall ROAS: {overall_roas['total_roas']:.2f}x")
    print(f"Total Revenue: ${overall_roas['revenue']:,.2f}")
    print(f"Total Spend: ${overall_roas['spend']:,.2f}")
    
    return unified_data, report

if __name__ == '__main__':
    unified_data, report = analyze_marketing_performance()
```

## Configuration File

Create a `config.yaml` for reusable settings:

```yaml
data_sources:
  ad_platform:
    type: api
    endpoint: https://api.adplatform.com/v1/campaigns
    auth_token_env: AD_PLATFORM_API_KEY
  
  retail:
    type: database
    connection_string_env: RETAIL_DATABASE_URL
    query: "SELECT * FROM sales WHERE date >= :start_date"
  
  inventory:
    type: csv
    path: data/inventory.csv

join_config:
  keys: [sku, date]
  attribution_window: 7
  attribution_model: last_touch
  join_type: left

output:
  format: csv
  path: output/unified_marketing_data.csv
```

Load configuration:

```python
from genpark_data_joiner import CrossChannelJoiner

joiner = CrossChannelJoiner.from_config('config.yaml')
unified_data = joiner.run()
```

## Troubleshooting

### Missing Join Keys

```python
# Check for missing keys before joining
from genpark_data_joiner import DataAnalyzer

analyzer = DataAnalyzer()
missing_keys = analyzer.check_missing_keys(ad_data, retail_data, keys=['sku', 'date'])

if missing_keys:
    print(f"Warning: Missing keys found: {missing_keys}")
    # Fill or filter as needed
```

### Date Parsing Issues

```python
# Explicitly set date format
ad_data = DataSource.from_csv(
    'ads.csv',
    date_column='timestamp',
    date_format='%Y-%m-%d %H:%M:%S',
    timezone='UTC'
)
```

### Memory Issues with Large Datasets

```python
# Use chunked processing
unified_data = joiner.join_chunked(
    ad_data=ad_data,
    retail_data=retail_data,
    inventory_data=inventory_data,
    join_keys=['sku', 'date'],
    chunk_size=10000  # Process 10k rows at a time
)
```

### Attribution Mismatches

```python
# Debug attribution matching
debug_report = joiner.debug_attribution(
    ad_data=ad_data,
    retail_data=retail_data,
    join_keys=['sku'],
    attribution_window=7
)

# Review unmatched records
print(f"Unmatched ad records: {debug_report['unmatched_ads']}")
print(f"Unmatched retail records: {debug_report['unmatched_retail']}")
```

## Environment Variables

Set these environment variables for production use:

- `AD_PLATFORM_API_KEY`: Authentication token for programmatic ad platform
- `RETAIL_DATABASE_URL`: Connection string for retail sales database
- `INVENTORY_API_ENDPOINT`: URL for inventory management API
- `INVENTORY_API_TOKEN`: Authentication token for inventory API

Example `.env` file:

```
AD_PLATFORM_API_KEY=your_ad_platform_key_here
RETAIL_DATABASE_URL=postgresql://user:pass@host:5432/retail_db
INVENTORY_API_ENDPOINT=https://inventory.example.com/api/v1
INVENTORY_API_TOKEN=your_inventory_token_here
```

Load with:

```python
from dotenv import load_dotenv
load_dotenv()
```
