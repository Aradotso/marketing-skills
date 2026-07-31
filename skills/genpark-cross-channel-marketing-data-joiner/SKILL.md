---
name: genpark-cross-channel-marketing-data-joiner
description: Join programmatic ad data with retail & inventory systems to calculate total ROAS across channels
triggers:
  - join marketing data from multiple channels
  - combine ad spend with sales data for ROAS
  - merge programmatic advertising with retail metrics
  - calculate cross-channel return on ad spend
  - integrate inventory data with marketing campaigns
  - unify ad performance and retail conversion data
  - reconcile marketing spend across platforms
  - analyze total ROAS from all marketing channels
---

# genpark-cross-channel-marketing-data-joiner

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

The GenPark Cross-Channel Marketing Data Joiner is a Python skill that consolidates programmatic advertising data with retail sales and inventory information to calculate comprehensive Return on Ad Spend (ROAS) metrics. It handles data from multiple marketing channels (Google Ads, Facebook, programmatic platforms) and joins it with point-of-sale, e-commerce, and inventory systems.

## Installation

```bash
git clone https://github.com/alphaparkinc/genpark-cross-channel-marketing-data-joiner-skill.git
cd genpark-cross-channel-marketing-data-joiner-skill
pip install -r requirements.txt
```

### Dependencies

Typically requires:
```bash
pip install pandas numpy requests python-dotenv
```

## Core Functionality

### Basic Usage Pattern

```python
from genpark_data_joiner import MarketingDataJoiner, DataSource

# Initialize the joiner
joiner = MarketingDataJoiner()

# Add marketing channel data
joiner.add_ad_data(
    source=DataSource.GOOGLE_ADS,
    data_path="google_ads_export.csv",
    date_column="date",
    spend_column="cost",
    campaign_id_column="campaign_id"
)

# Add retail/sales data
joiner.add_sales_data(
    source=DataSource.SHOPIFY,
    data_path="shopify_orders.csv",
    date_column="order_date",
    revenue_column="total_price",
    attribution_column="utm_campaign"
)

# Perform the join and calculate ROAS
results = joiner.calculate_roas(
    attribution_window_days=7,
    groupby=["campaign_id", "date"]
)

print(results.summary())
```

### Data Source Configuration

```python
from genpark_data_joiner import MarketingDataJoiner, ChannelConfig

joiner = MarketingDataJoiner()

# Configure Facebook Ads
fb_config = ChannelConfig(
    name="facebook_ads",
    date_field="date_start",
    spend_field="spend",
    impressions_field="impressions",
    clicks_field="clicks",
    campaign_identifier="campaign_name"
)

joiner.add_channel_config(fb_config)
```

### Multi-Channel Data Integration

```python
from genpark_data_joiner import MarketingDataJoiner
import pandas as pd

joiner = MarketingDataJoiner()

# Add multiple ad platforms
ad_sources = [
    ("google_ads.csv", "Google Ads"),
    ("facebook_ads.csv", "Facebook"),
    ("tiktok_ads.csv", "TikTok"),
    ("programmatic_dv360.csv", "DV360")
]

for filepath, platform in ad_sources:
    df = pd.read_csv(filepath)
    joiner.add_ad_platform(
        platform_name=platform,
        data=df,
        normalize_columns=True  # Auto-map common column names
    )

# Add retail data sources
joiner.add_retail_data(
    pos_data="pos_transactions.csv",
    ecommerce_data="shopify_orders.csv",
    inventory_data="inventory_levels.csv"
)

# Join all data with attribution logic
unified_data = joiner.join_all(
    attribution_model="last_click",
    lookback_window=14
)
```

### ROAS Calculation

```python
# Calculate ROAS by campaign
roas_by_campaign = joiner.calculate_roas(
    groupby="campaign_id",
    include_metrics=["impressions", "clicks", "conversions", "revenue", "spend"]
)

# Calculate ROAS by channel
roas_by_channel = joiner.calculate_roas(
    groupby="channel",
    time_period="daily"
)

# Calculate incremental ROAS (accounting for baseline sales)
incremental_roas = joiner.calculate_incremental_roas(
    baseline_sales_data="baseline_sales.csv",
    test_period_start="2026-07-01",
    test_period_end="2026-07-31"
)
```

### Attribution Models

```python
from genpark_data_joiner import AttributionModel

# Last-click attribution
last_click = joiner.calculate_roas(
    attribution_model=AttributionModel.LAST_CLICK,
    window_days=7
)

# First-click attribution
first_click = joiner.calculate_roas(
    attribution_model=AttributionModel.FIRST_CLICK,
    window_days=7
)

# Linear attribution (equal credit to all touchpoints)
linear = joiner.calculate_roas(
    attribution_model=AttributionModel.LINEAR,
    window_days=14
)

# Time-decay attribution
time_decay = joiner.calculate_roas(
    attribution_model=AttributionModel.TIME_DECAY,
    window_days=30,
    decay_rate=0.5  # Half-life in days
)
```

## API Reference

### MarketingDataJoiner Class

```python
class MarketingDataJoiner:
    def __init__(self, config_path: str = None):
        """Initialize the data joiner with optional config file"""
        pass
    
    def add_ad_data(self, source: str, data: Union[str, pd.DataFrame], **kwargs):
        """Add advertising data from a platform"""
        pass
    
    def add_sales_data(self, source: str, data: Union[str, pd.DataFrame], **kwargs):
        """Add retail/sales data"""
        pass
    
    def add_inventory_data(self, data: Union[str, pd.DataFrame], product_id_column: str):
        """Add inventory level data"""
        pass
    
    def join_all(self, attribution_model: str = "last_click", lookback_window: int = 7):
        """Join all data sources with specified attribution logic"""
        pass
    
    def calculate_roas(self, groupby: Union[str, list] = None, **kwargs):
        """Calculate ROAS metrics"""
        pass
    
    def export_results(self, filepath: str, format: str = "csv"):
        """Export joined data to file"""
        pass
```

### Configuration File

Create a `config.yaml` for complex setups:

```yaml
data_sources:
  ad_platforms:
    - name: google_ads
      file: data/google_ads.csv
      date_column: date
      spend_column: cost
      campaign_column: campaign_id
    
    - name: facebook
      file: data/facebook_ads.csv
      date_column: date_start
      spend_column: spend
      campaign_column: campaign_name
  
  retail:
    - name: shopify
      file: data/shopify_orders.csv
      date_column: created_at
      revenue_column: total_price
      attribution_column: utm_campaign
    
    - name: pos_system
      file: data/pos_transactions.csv
      date_column: transaction_date
      revenue_column: amount
      store_column: store_id

attribution:
  model: last_click
  window_days: 7
  deduplication: true

output:
  format: csv
  path: results/roas_analysis.csv
```

Load configuration:

```python
joiner = MarketingDataJoiner(config_path="config.yaml")
results = joiner.run()
```

## Common Patterns

### Complete ROAS Analysis Pipeline

```python
from genpark_data_joiner import MarketingDataJoiner
import os

def run_roas_analysis():
    # Initialize
    joiner = MarketingDataJoiner()
    
    # Load ad data
    ad_files = {
        "Google Ads": "data/google_ads_2026_07.csv",
        "Facebook": "data/facebook_ads_2026_07.csv",
        "Programmatic": "data/dv360_2026_07.csv"
    }
    
    for platform, filepath in ad_files.items():
        joiner.add_ad_platform(platform, filepath)
    
    # Load sales data
    joiner.add_sales_data("ecommerce", "data/online_sales.csv")
    joiner.add_sales_data("retail", "data/store_sales.csv")
    
    # Load inventory for product-level analysis
    joiner.add_inventory_data("data/inventory.csv", product_id_column="sku")
    
    # Join and calculate
    unified = joiner.join_all(attribution_model="time_decay", lookback_window=14)
    
    # Calculate various ROAS views
    overall_roas = joiner.calculate_roas()
    campaign_roas = joiner.calculate_roas(groupby="campaign_id")
    daily_roas = joiner.calculate_roas(groupby=["date", "channel"])
    product_roas = joiner.calculate_roas(groupby="product_sku")
    
    # Export results
    joiner.export_results("results/roas_report.csv")
    
    return {
        "overall": overall_roas,
        "by_campaign": campaign_roas,
        "by_day": daily_roas,
        "by_product": product_roas
    }

if __name__ == "__main__":
    results = run_roas_analysis()
    print(f"Overall ROAS: {results['overall']['roas']:.2f}")
```

### Custom Data Preprocessing

```python
from genpark_data_joiner import MarketingDataJoiner
import pandas as pd

def preprocess_ad_data(df, platform):
    """Custom preprocessing for ad platform data"""
    # Normalize column names
    column_mapping = {
        "Cost": "spend",
        "Campaign": "campaign_id",
        "Date": "date"
    }
    df = df.rename(columns=column_mapping)
    
    # Add platform identifier
    df["platform"] = platform
    
    # Convert date formats
    df["date"] = pd.to_datetime(df["date"])
    
    # Remove test campaigns
    df = df[~df["campaign_id"].str.contains("test", case=False)]
    
    return df

joiner = MarketingDataJoiner()

# Load and preprocess
google_df = pd.read_csv("google_ads.csv")
google_df = preprocess_ad_data(google_df, "Google Ads")

joiner.add_ad_data("google", data=google_df)
```

### Handling Missing Attribution Data

```python
from genpark_data_joiner import MarketingDataJoiner

joiner = MarketingDataJoiner()

# Configure how to handle unattributed sales
joiner.configure_attribution(
    unattributed_strategy="proportional",  # Distribute based on spend ratio
    minimum_confidence=0.7,  # Only attribute if confidence > 70%
    fallback_to_organic=True  # Treat low-confidence as organic
)

# Calculate ROAS with attribution quality metrics
results = joiner.calculate_roas(
    include_attribution_quality=True
)

# Filter for high-confidence results
high_conf = results[results["attribution_confidence"] > 0.8]
```

## Environment Variables

```bash
# Optional API keys if pulling data directly from platforms
GOOGLE_ADS_API_KEY=your_key_here
FACEBOOK_ACCESS_TOKEN=your_token_here
SHOPIFY_API_KEY=your_key_here
SHOPIFY_API_SECRET=your_secret_here

# Database connection if storing results
DATABASE_URL=postgresql://user:pass@localhost/marketing_db
```

## Troubleshooting

### Date Format Mismatches

```python
# Explicitly set date format for each source
joiner.add_ad_data(
    "google",
    data="google_ads.csv",
    date_column="date",
    date_format="%Y-%m-%d"  # Specify format
)

joiner.add_sales_data(
    "shopify",
    data="orders.csv",
    date_column="created_at",
    date_format="%Y-%m-%d %H:%M:%S"  # With timestamp
)
```

### Campaign ID Mismatches

```python
# Use fuzzy matching for campaign names
joiner.configure_matching(
    fuzzy_match=True,
    similarity_threshold=0.85,
    normalize_campaign_names=True
)

# Or provide explicit mapping
campaign_map = {
    "summer_sale_v1": "Summer Sale 2026",
    "summer_sale_v2": "Summer Sale 2026",
    "back_to_school": "BTS Campaign"
}

joiner.add_campaign_mapping(campaign_map)
```

### Memory Issues with Large Datasets

```python
# Process data in chunks
joiner = MarketingDataJoiner(chunk_size=10000)

# Or use lazy loading
joiner.enable_lazy_loading(True)

# Calculate ROAS incrementally
for chunk in joiner.iterate_date_ranges(start="2026-01-01", end="2026-07-31", freq="W"):
    chunk_roas = joiner.calculate_roas(date_range=chunk)
    chunk_roas.to_csv(f"results/roas_{chunk[0]}.csv")
```

### Currency Conversion

```python
# Handle multi-currency data
joiner.add_currency_config(
    base_currency="USD",
    conversion_rates={
        "EUR": 1.10,
        "GBP": 1.27,
        "CAD": 0.74
    }
)

# Or use live rates
joiner.enable_live_currency_conversion(api_key=os.getenv("EXCHANGE_RATE_API_KEY"))
```

## Advanced Usage

### Custom Attribution Models

```python
def custom_attribution_logic(touchpoints, conversion_value):
    """
    Custom attribution function
    touchpoints: list of (timestamp, channel, spend) tuples
    conversion_value: float
    Returns: dict of {channel: attributed_value}
    """
    # Example: Give 50% to first touch, 50% to last touch
    if len(touchpoints) == 0:
        return {}
    
    first_channel = touchpoints[0][1]
    last_channel = touchpoints[-1][1]
    
    attribution = {}
    attribution[first_channel] = conversion_value * 0.5
    attribution[last_channel] = attribution.get(last_channel, 0) + conversion_value * 0.5
    
    return attribution

joiner.set_custom_attribution(custom_attribution_logic)
results = joiner.calculate_roas()
```

### Integration with BI Tools

```python
# Export to database for Tableau/PowerBI
joiner.export_to_database(
    connection_string=os.getenv("DATABASE_URL"),
    table_name="marketing_roas",
    if_exists="replace"
)

# Or export to Google Sheets
joiner.export_to_sheets(
    credentials_file="credentials.json",
    spreadsheet_id="your_sheet_id",
    worksheet_name="ROAS Analysis"
)
```
