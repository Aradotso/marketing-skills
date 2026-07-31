---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - join marketing data from multiple channels
  - combine ad spend with retail sales data
  - calculate cross-channel ROAS
  - merge programmatic advertising with inventory data
  - integrate marketing performance across platforms
  - consolidate ad data with retail metrics
  - analyze total marketing return on ad spend
  - unify cross-channel marketing datasets
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python-based skill that enables AI agents to join programmatic advertising data with retail sales and inventory data to calculate comprehensive Return on Ad Spend (ROAS) metrics. This tool bridges the gap between digital marketing platforms and physical/online retail systems, providing a unified view of marketing performance.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill

# Install dependencies
pip install -r requirements.txt
```

## Key Components

### Core Functionality

The skill joins data from multiple sources:
- **Programmatic Ad Data**: Campaign spend, impressions, clicks, conversions
- **Retail Sales Data**: Transaction records, product sales, revenue
- **Inventory Data**: Stock levels, SKU information, product metadata

### Primary API

```python
from genpark_data_joiner import DataJoiner

# Initialize the data joiner
joiner = DataJoiner(
    ad_data_source="path/to/ad_data.csv",
    retail_data_source="path/to/retail_data.csv",
    inventory_data_source="path/to/inventory_data.csv"
)

# Join datasets
unified_data = joiner.join_datasets(
    join_key="product_id",
    date_range=("2026-07-01", "2026-07-31")
)

# Calculate ROAS
roas_metrics = joiner.calculate_roas(unified_data)
```

## Configuration

### Environment Variables

```bash
# Data source connections
export AD_DATA_API_KEY=your_ad_platform_api_key
export RETAIL_DB_CONNECTION=your_retail_db_connection_string
export INVENTORY_API_ENDPOINT=your_inventory_api_endpoint

# Processing options
export JOIN_STRATEGY=left  # left, inner, outer
export DATE_FORMAT="%Y-%m-%d"
export CURRENCY=USD
```

### Configuration File

Create `config.yaml`:

```yaml
data_sources:
  programmatic_ads:
    type: csv
    path: data/ad_campaigns.csv
    key_column: campaign_id
  retail_sales:
    type: database
    connection: ${RETAIL_DB_CONNECTION}
    table: sales_transactions
    key_column: product_sku
  inventory:
    type: api
    endpoint: ${INVENTORY_API_ENDPOINT}
    key_column: sku

join_config:
  primary_key: product_id
  date_column: transaction_date
  strategy: left
  
roas_calculation:
  revenue_column: total_sales
  cost_column: ad_spend
  attribution_window_days: 7
```

## Usage Examples

### Example 1: Basic Data Join

```python
from genpark_data_joiner import DataJoiner
import pandas as pd

# Initialize with data sources
joiner = DataJoiner()

# Load ad campaign data
ad_data = pd.read_csv("ad_campaigns.csv")
# Columns: campaign_id, date, product_id, ad_spend, impressions, clicks

# Load retail sales data
retail_data = pd.read_csv("retail_sales.csv")
# Columns: transaction_id, date, product_id, revenue, quantity

# Join datasets on product_id and date
joined_data = joiner.join_on_keys(
    left_df=ad_data,
    right_df=retail_data,
    left_key=["product_id", "date"],
    right_key=["product_id", "date"],
    join_type="inner"
)

print(joined_data.head())
```

### Example 2: Calculate Cross-Channel ROAS

```python
from genpark_data_joiner import DataJoiner, ROASCalculator

# Initialize components
joiner = DataJoiner()
roas_calc = ROASCalculator(attribution_window=7)

# Load and join data
unified_data = joiner.load_and_join(
    ad_source="google_ads.csv",
    retail_source="shopify_sales.csv",
    inventory_source="warehouse_inventory.csv"
)

# Calculate ROAS by channel
roas_by_channel = roas_calc.calculate_by_dimension(
    data=unified_data,
    dimension="channel",
    revenue_col="total_revenue",
    cost_col="total_ad_spend"
)

for channel, metrics in roas_by_channel.items():
    print(f"{channel}: ROAS = {metrics['roas']:.2f}, Revenue = ${metrics['revenue']:.2f}")
```

### Example 3: Multi-Source Integration

```python
from genpark_data_joiner import DataJoiner, DataSource
import os

# Configure multiple ad sources
ad_sources = [
    DataSource(
        name="google_ads",
        type="api",
        api_key=os.getenv("GOOGLE_ADS_API_KEY"),
        date_range=("2026-07-01", "2026-07-31")
    ),
    DataSource(
        name="facebook_ads",
        type="api",
        api_key=os.getenv("FACEBOOK_ADS_API_KEY"),
        date_range=("2026-07-01", "2026-07-31")
    ),
    DataSource(
        name="programmatic_dsp",
        type="csv",
        path="dsp_campaigns.csv"
    )
]

# Initialize joiner with multiple sources
joiner = DataJoiner(ad_sources=ad_sources)

# Fetch and consolidate ad data
consolidated_ads = joiner.consolidate_ad_data()

# Join with retail data
retail_source = DataSource(
    name="pos_system",
    type="database",
    connection=os.getenv("RETAIL_DB_CONNECTION"),
    query="SELECT * FROM sales WHERE date BETWEEN '2026-07-01' AND '2026-07-31'"
)

final_data = joiner.join_with_retail(
    ad_data=consolidated_ads,
    retail_source=retail_source,
    join_key="product_id"
)

# Calculate total ROAS
total_roas = joiner.calculate_total_roas(final_data)
print(f"Total Cross-Channel ROAS: {total_roas:.2f}")
```

### Example 4: Attribution Modeling

```python
from genpark_data_joiner import DataJoiner, AttributionModel

# Initialize with attribution settings
joiner = DataJoiner()
attribution = AttributionModel(
    model_type="last_click",  # Options: last_click, first_click, linear, time_decay
    lookback_window=14
)

# Load customer journey data
journey_data = joiner.load_customer_journeys(
    ad_data="ad_interactions.csv",
    conversion_data="conversions.csv"
)

# Apply attribution model
attributed_data = attribution.apply(journey_data)

# Join with retail and inventory
full_dataset = joiner.join_all(
    attributed_ads=attributed_data,
    retail_data="sales.csv",
    inventory_data="inventory.csv"
)

# Calculate attributed ROAS
attributed_roas = joiner.calculate_attributed_roas(full_dataset)

for touchpoint, roas in attributed_roas.items():
    print(f"{touchpoint}: {roas:.2f}")
```

## Common Patterns

### Pattern 1: Daily ROAS Report

```python
from genpark_data_joiner import DataJoiner
from datetime import datetime, timedelta

def generate_daily_roas_report(date=None):
    if date is None:
        date = datetime.now().date() - timedelta(days=1)
    
    joiner = DataJoiner()
    
    # Load yesterday's data
    data = joiner.join_datasets(
        ad_data_filter={"date": date},
        retail_data_filter={"date": date}
    )
    
    # Calculate metrics
    metrics = {
        "total_spend": data["ad_spend"].sum(),
        "total_revenue": data["revenue"].sum(),
        "roas": data["revenue"].sum() / data["ad_spend"].sum() if data["ad_spend"].sum() > 0 else 0,
        "transactions": len(data),
        "avg_order_value": data["revenue"].mean()
    }
    
    return metrics

# Usage
report = generate_daily_roas_report()
print(f"ROAS: {report['roas']:.2f}")
```

### Pattern 2: Product-Level Performance

```python
from genpark_data_joiner import DataJoiner
import pandas as pd

def analyze_product_performance(top_n=20):
    joiner = DataJoiner()
    
    # Join all data sources
    data = joiner.join_datasets()
    
    # Group by product
    product_metrics = data.groupby("product_id").agg({
        "ad_spend": "sum",
        "revenue": "sum",
        "impressions": "sum",
        "clicks": "sum",
        "quantity_sold": "sum"
    }).reset_index()
    
    # Calculate ROAS per product
    product_metrics["roas"] = product_metrics["revenue"] / product_metrics["ad_spend"]
    product_metrics["ctr"] = product_metrics["clicks"] / product_metrics["impressions"]
    
    # Get top performers
    top_products = product_metrics.nlargest(top_n, "roas")
    
    return top_products

# Usage
top_performers = analyze_product_performance(top_n=10)
print(top_performers)
```

### Pattern 3: Inventory-Aware Campaign Optimization

```python
from genpark_data_joiner import DataJoiner

def optimize_campaigns_by_inventory():
    joiner = DataJoiner()
    
    # Join ad performance with current inventory
    data = joiner.join_with_inventory_check()
    
    # Identify high ROAS products with low inventory
    optimization_opportunities = data[
        (data["roas"] > 3.0) & 
        (data["inventory_level"] < data["inventory_threshold"])
    ]
    
    # Identify low ROAS products with high inventory
    reduce_spend = data[
        (data["roas"] < 1.0) & 
        (data["inventory_level"] > data["inventory_max"])
    ]
    
    return {
        "increase_budget": optimization_opportunities["product_id"].tolist(),
        "decrease_budget": reduce_spend["product_id"].tolist()
    }

# Usage
recommendations = optimize_campaigns_by_inventory()
print(f"Increase budget for: {recommendations['increase_budget']}")
print(f"Decrease budget for: {recommendations['decrease_budget']}")
```

## Troubleshooting

### Issue: Data Sources Not Aligning

**Problem**: Joined datasets have mismatched dates or product IDs

**Solution**:
```python
from genpark_data_joiner import DataJoiner, DataNormalizer

# Use normalizer to standardize formats
normalizer = DataNormalizer()

# Normalize date formats
normalized_ads = normalizer.normalize_dates(
    ad_data, 
    date_column="date",
    input_format="%d/%m/%Y",
    output_format="%Y-%m-%d"
)

# Normalize product IDs
normalized_retail = normalizer.normalize_product_ids(
    retail_data,
    id_column="sku",
    mapping_file="product_id_mapping.csv"
)

# Now join
joiner = DataJoiner()
joined = joiner.join_datasets(normalized_ads, normalized_retail)
```

### Issue: Missing Revenue Attribution

**Problem**: Sales data doesn't link back to ad campaigns

**Solution**:
```python
from genpark_data_joiner import AttributionLinker

linker = AttributionLinker(attribution_window=7)

# Link conversions to ad touchpoints
linked_data = linker.link_conversions(
    ad_interactions="ad_clicks.csv",
    conversions="sales.csv",
    user_id_column="customer_id",
    timestamp_column="timestamp"
)
```

### Issue: Performance with Large Datasets

**Problem**: Slow processing with millions of rows

**Solution**:
```python
from genpark_data_joiner import DataJoiner

# Use chunked processing
joiner = DataJoiner(chunk_size=100000)

results = []
for chunk in joiner.process_in_chunks(
    ad_data="large_ad_data.csv",
    retail_data="large_retail_data.csv"
):
    chunk_roas = joiner.calculate_roas(chunk)
    results.append(chunk_roas)

# Aggregate results
final_roas = sum(r["revenue"] for r in results) / sum(r["cost"] for r in results)
```

## Running the Example

```bash
# Run the example usage script
python example_usage.py
```

This will demonstrate a complete workflow of joining ad data with retail and inventory data to calculate cross-channel ROAS.
