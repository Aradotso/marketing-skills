---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across marketing channels
triggers:
  - combine ad spend with sales data
  - join marketing data with inventory
  - calculate cross-channel ROAS
  - merge programmatic ads and retail data
  - integrate advertising and sales metrics
  - unify marketing and retail attribution
  - connect ad platforms with point of sale
  - aggregate multi-channel marketing performance
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that unifies programmatic advertising data with retail and inventory systems to provide comprehensive ROAS (Return on Ad Spend) analytics. It bridges the gap between digital marketing campaigns and physical/online sales outcomes, enabling marketers to understand true campaign effectiveness across channels.

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

## Quick Start

```python
from genpark_data_joiner import CrossChannelJoiner

# Initialize the joiner
joiner = CrossChannelJoiner(
    ad_data_source="google_ads",
    retail_data_source="shopify",
    inventory_data_source="warehouse_api"
)

# Join data and calculate ROAS
result = joiner.join_and_calculate(
    date_range=("2026-07-01", "2026-07-31"),
    attribution_window=7  # days
)

print(f"Total ROAS: {result.total_roas}")
print(f"Channel breakdown: {result.channel_roas}")
```

## Core Components

### 1. Data Source Connectors

Connect to various ad platforms, retail systems, and inventory databases:

```python
from genpark_data_joiner import AdConnector, RetailConnector, InventoryConnector

# Programmatic ad sources
ad_connector = AdConnector()
ad_connector.add_source("google_ads", api_key=os.getenv("GOOGLE_ADS_API_KEY"))
ad_connector.add_source("facebook_ads", api_key=os.getenv("FB_ADS_API_KEY"))
ad_connector.add_source("dv360", credentials=os.getenv("DV360_CREDENTIALS"))

# Retail data sources
retail_connector = RetailConnector()
retail_connector.add_source("shopify", api_key=os.getenv("SHOPIFY_API_KEY"))
retail_connector.add_source("square", access_token=os.getenv("SQUARE_ACCESS_TOKEN"))

# Inventory systems
inventory_connector = InventoryConnector()
inventory_connector.add_source("warehouse_api", endpoint=os.getenv("WAREHOUSE_ENDPOINT"))
```

### 2. Data Joining

Join datasets using multiple attribution models:

```python
from genpark_data_joiner import DataJoiner, AttributionModel

joiner = DataJoiner(
    ad_data=ad_connector.fetch_data(start_date, end_date),
    retail_data=retail_connector.fetch_data(start_date, end_date),
    inventory_data=inventory_connector.fetch_data(start_date, end_date)
)

# Join with last-click attribution
last_click = joiner.join(
    attribution_model=AttributionModel.LAST_CLICK,
    attribution_window_days=7
)

# Join with multi-touch attribution
multi_touch = joiner.join(
    attribution_model=AttributionModel.LINEAR,
    attribution_window_days=30
)

# Join with position-based attribution
position_based = joiner.join(
    attribution_model=AttributionModel.POSITION_BASED,
    attribution_window_days=14,
    first_touch_weight=0.4,
    last_touch_weight=0.4,
    middle_weight=0.2
)
```

### 3. ROAS Calculation

Calculate return on ad spend across channels:

```python
from genpark_data_joiner import ROASCalculator

calculator = ROASCalculator(joined_data=last_click)

# Total ROAS
total_roas = calculator.calculate_total_roas()
print(f"Total ROAS: {total_roas:.2f}")

# Channel-specific ROAS
channel_roas = calculator.calculate_by_channel()
for channel, roas in channel_roas.items():
    print(f"{channel}: {roas:.2f}")

# Campaign-level ROAS
campaign_roas = calculator.calculate_by_campaign()

# Product-level ROAS
product_roas = calculator.calculate_by_product()

# With inventory margins
roas_with_margin = calculator.calculate_with_margins(
    include_cogs=True,
    include_shipping=True
)
```

## Configuration

### Configuration File

Create a `config.yaml` file:

```yaml
data_sources:
  ads:
    - name: google_ads
      api_key_env: GOOGLE_ADS_API_KEY
      customer_id_env: GOOGLE_ADS_CUSTOMER_ID
    - name: facebook_ads
      api_key_env: FB_ADS_API_KEY
      ad_account_id_env: FB_AD_ACCOUNT_ID
  
  retail:
    - name: shopify
      api_key_env: SHOPIFY_API_KEY
      store_url_env: SHOPIFY_STORE_URL
    - name: square
      access_token_env: SQUARE_ACCESS_TOKEN
  
  inventory:
    - name: warehouse_api
      endpoint_env: WAREHOUSE_ENDPOINT
      auth_token_env: WAREHOUSE_AUTH_TOKEN

attribution:
  default_model: last_click
  default_window_days: 7
  
roas:
  include_returns: true
  include_cogs: true
  include_shipping_costs: true

output:
  format: csv
  include_raw_data: false
```

Load configuration:

```python
from genpark_data_joiner import load_config

config = load_config("config.yaml")
joiner = CrossChannelJoiner.from_config(config)
```

### Environment Variables

```bash
# Ad Platform Credentials
export GOOGLE_ADS_API_KEY="your_key"
export GOOGLE_ADS_CUSTOMER_ID="your_customer_id"
export FB_ADS_API_KEY="your_key"
export FB_AD_ACCOUNT_ID="your_account_id"
export DV360_CREDENTIALS="path/to/credentials.json"

# Retail Platform Credentials
export SHOPIFY_API_KEY="your_key"
export SHOPIFY_STORE_URL="your-store.myshopify.com"
export SQUARE_ACCESS_TOKEN="your_token"

# Inventory System Credentials
export WAREHOUSE_ENDPOINT="https://api.warehouse.example.com"
export WAREHOUSE_AUTH_TOKEN="your_token"
```

## Common Patterns

### Pattern 1: Weekly ROAS Report

```python
from genpark_data_joiner import CrossChannelJoiner, ReportGenerator
from datetime import datetime, timedelta

def generate_weekly_roas_report():
    # Get last 7 days
    end_date = datetime.now()
    start_date = end_date - timedelta(days=7)
    
    # Initialize and join data
    joiner = CrossChannelJoiner.from_config("config.yaml")
    result = joiner.join_and_calculate(
        date_range=(start_date, end_date),
        attribution_window=7
    )
    
    # Generate report
    report = ReportGenerator(result)
    report.add_summary()
    report.add_channel_breakdown()
    report.add_top_campaigns(limit=10)
    report.add_product_performance()
    
    # Export
    report.export_csv("weekly_roas_report.csv")
    report.export_pdf("weekly_roas_report.pdf")
    
    return result

if __name__ == "__main__":
    generate_weekly_roas_report()
```

### Pattern 2: Multi-Attribution Comparison

```python
from genpark_data_joiner import CrossChannelJoiner, AttributionModel

def compare_attribution_models(start_date, end_date):
    joiner = CrossChannelJoiner.from_config("config.yaml")
    
    models = [
        AttributionModel.LAST_CLICK,
        AttributionModel.FIRST_CLICK,
        AttributionModel.LINEAR,
        AttributionModel.TIME_DECAY,
        AttributionModel.POSITION_BASED
    ]
    
    results = {}
    for model in models:
        result = joiner.join_and_calculate(
            date_range=(start_date, end_date),
            attribution_model=model,
            attribution_window=14
        )
        results[model.name] = {
            "total_roas": result.total_roas,
            "channel_roas": result.channel_roas
        }
    
    # Compare results
    for model_name, data in results.items():
        print(f"\n{model_name}:")
        print(f"  Total ROAS: {data['total_roas']:.2f}")
        for channel, roas in data['channel_roas'].items():
            print(f"  {channel}: {roas:.2f}")
    
    return results
```

### Pattern 3: Real-Time ROAS Dashboard

```python
from genpark_data_joiner import CrossChannelJoiner, StreamingConnector
import time

def realtime_roas_monitor(refresh_interval=300):
    """Monitor ROAS every 5 minutes"""
    joiner = CrossChannelJoiner.from_config("config.yaml")
    
    while True:
        try:
            # Get today's data
            result = joiner.join_and_calculate(
                date_range="today",
                attribution_window=1
            )
            
            print(f"\n[{datetime.now()}] Real-time ROAS:")
            print(f"Total ROAS: {result.total_roas:.2f}")
            print(f"Total Spend: ${result.total_spend:,.2f}")
            print(f"Total Revenue: ${result.total_revenue:,.2f}")
            
            # Alert if ROAS drops below threshold
            if result.total_roas < 2.0:
                send_alert(f"ROAS Alert: {result.total_roas:.2f}")
            
            time.sleep(refresh_interval)
            
        except Exception as e:
            print(f"Error: {e}")
            time.sleep(60)
```

### Pattern 4: Inventory-Aware Campaign Optimization

```python
from genpark_data_joiner import CrossChannelJoiner, InventoryOptimizer

def optimize_campaigns_by_inventory():
    joiner = CrossChannelJoiner.from_config("config.yaml")
    optimizer = InventoryOptimizer(joiner)
    
    # Get current campaign performance with inventory levels
    analysis = optimizer.analyze(
        include_stock_levels=True,
        include_margins=True,
        include_velocity=True
    )
    
    recommendations = []
    
    for campaign in analysis.campaigns:
        if campaign.inventory_level == "low" and campaign.roas > 3.0:
            recommendations.append({
                "campaign_id": campaign.id,
                "action": "pause",
                "reason": "Low inventory, high ROAS - avoid stockout"
            })
        elif campaign.inventory_level == "high" and campaign.roas < 1.5:
            recommendations.append({
                "campaign_id": campaign.id,
                "action": "increase_budget",
                "reason": "High inventory, low ROAS - clear stock"
            })
    
    return recommendations
```

## Troubleshooting

### Connection Issues

```python
# Test individual connections
from genpark_data_joiner import test_connections

results = test_connections("config.yaml")
for source, status in results.items():
    if not status["connected"]:
        print(f"Failed to connect to {source}: {status['error']}")
```

### Data Matching Problems

```python
# Debug data matching
from genpark_data_joiner import DataJoiner

joiner = DataJoiner(ad_data, retail_data, inventory_data)
match_report = joiner.diagnose_matching()

print(f"Ad records: {match_report.ad_record_count}")
print(f"Retail records: {match_report.retail_record_count}")
print(f"Matched records: {match_report.matched_count}")
print(f"Unmatched ad records: {match_report.unmatched_ads}")
print(f"Unmatched retail records: {match_report.unmatched_retail}")
```

### Attribution Window Issues

```python
# Experiment with different attribution windows
from genpark_data_joiner import AttributionAnalyzer

analyzer = AttributionAnalyzer(joiner)
window_analysis = analyzer.test_windows(
    windows=[1, 3, 7, 14, 30],
    date_range=(start_date, end_date)
)

# Find optimal window
optimal_window = window_analysis.recommend_window()
print(f"Recommended attribution window: {optimal_window} days")
```

### Performance Optimization

```python
# For large datasets, use batch processing
from genpark_data_joiner import BatchProcessor

processor = BatchProcessor(
    batch_size=10000,
    parallel_workers=4
)

result = processor.process_large_dataset(
    ad_data_path="large_ad_data.csv",
    retail_data_path="large_retail_data.csv",
    inventory_data_path="large_inventory_data.csv"
)
```

## API Reference

### Key Methods

- `CrossChannelJoiner.join_and_calculate()` - Main method to join data and calculate ROAS
- `DataJoiner.join()` - Join datasets with specified attribution model
- `ROASCalculator.calculate_total_roas()` - Calculate overall ROAS
- `ROASCalculator.calculate_by_channel()` - Get per-channel ROAS
- `AttributionModel` - Enum of available attribution models
- `ReportGenerator.export_csv()` - Export results to CSV
- `test_connections()` - Validate all data source connections

## Additional Resources

- Homepage: https://genpark.ai
- Repository: https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill
- Example usage: Run `python example_usage.py` in the repository
