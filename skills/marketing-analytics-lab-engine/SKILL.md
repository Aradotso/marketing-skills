---
name: marketing-analytics-lab-engine
description: Multi-platform marketing analytics engine with canonical data model for unified campaign data ingestion and KPI analysis
triggers:
  - "ingest marketing campaign data from Google Ads or Meta"
  - "validate marketing data against canonical schema"
  - "create adapter for new marketing platform"
  - "load and process campaign CSV exports"
  - "unify multi-platform marketing data"
  - "set up marketing analytics pipeline"
  - "map platform-specific fields to canonical model"
  - "validate campaign records with Pydantic"
---

# Marketing Analytics Lab Engine

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Marketing Analytics Lab is a Python-based marketing data ingestion engine that enforces a canonical schema across multiple advertising platforms (Google Ads, Meta Ads, etc.). It uses an adapter pattern to transform platform-specific CSV exports into a unified `CampaignRecord` format with strict Pydantic validation before any analytics processing.

## Installation

```bash
git clone https://github.com/shayantimes/marketing-analytics-lab.git
cd marketing-analytics-lab
pip install -r requirements.txt
```

**Core dependencies:**
- Python 3.11+
- pandas
- pydantic v2
- pytest (for testing)

## Project Architecture

The system follows a strict data pipeline:

```
Raw CSV → Adapter → Canonical Record → Validation → KPI Engine (planned)
```

**Key components:**
- `CampaignRecord`: Canonical data model (single source of truth)
- `BaseCampaignAdapter`: Abstract adapter contract
- `UnifiedLoader`: Source-aware entry point that routes to correct adapter
- Validation layer: Pydantic-based contract enforcement

## Core Data Model

All marketing data must conform to the `CampaignRecord` canonical schema:

```python
from dataclasses import dataclass
from datetime import date
from typing import Optional

@dataclass
class CampaignRecord:
    source: str                    # 'google_ads', 'meta_ads', etc.
    date: str                      # 'YYYY-MM-DD'
    platform_campaign_id: str      # Platform's internal campaign ID
    campaign_name: str             # Human-readable campaign name
    impressions: int               # Ad impressions
    clicks: int                    # Ad clicks
    cost: float                    # Total cost in USD
    conversions: Optional[float]   # Number of conversions
    revenue: Optional[float]       # Total revenue in USD
```

**Pydantic validation schema:**

```python
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

class CampaignRecord(BaseModel):
    source: str = Field(..., min_length=1)
    date: str
    platform_campaign_id: str = Field(..., min_length=1)
    campaign_name: str = Field(..., min_length=1)
    impressions: int = Field(..., ge=0)
    clicks: int = Field(..., ge=0)
    cost: float = Field(..., ge=0)
    conversions: float | None = Field(None, ge=0)
    revenue: float | None = Field(None, ge=0)

    @field_validator('date')
    @classmethod
    def validate_date(cls, v):
        try:
            datetime.strptime(v, '%Y-%m-%d')
        except ValueError:
            raise ValueError('Date must be in YYYY-MM-DD format')
        return v
```

## Loading Campaign Data

### Basic Usage

```python
from src.loaders.unified_loader import UnifiedLoader

# Load Google Ads data
records = UnifiedLoader.load(
    source='google_ads',
    filepath='data/sample_google_ads_campaign.csv'
)

# Iterate through validated canonical records
for record in records:
    print(f"Campaign: {record.campaign_name}")
    print(f"CTR: {record.clicks / record.impressions * 100:.2f}%")
    print(f"Cost: ${record.cost:.2f}")
```

### Supported Sources

Currently implemented:
- `google_ads`: Google Ads CSV exports

In progress:
- `meta_ads`: Meta (Facebook/Instagram) Ads

## Creating a New Platform Adapter

To add support for a new marketing platform:

### 1. Create Adapter Class

```python
# src/adapters/your_platform/campaign_adapter.py
import pandas as pd
from src.adapters.base import BaseCampaignAdapter
from src.core.canonical.campaign_record import CampaignRecord

class YourPlatformAdapter(BaseCampaignAdapter):
    """Adapter for YourPlatform campaign data."""
    
    def adapt(self, df: pd.DataFrame) -> list[CampaignRecord]:
        """
        Transform platform-specific DataFrame to canonical CampaignRecord list.
        
        Expected CSV columns from YourPlatform:
        - campaign_id
        - campaign_title
        - report_date
        - views
        - link_clicks
        - spend
        - purchases
        - total_revenue
        """
        records = []
        
        for _, row in df.iterrows():
            record = CampaignRecord(
                source='your_platform',
                date=self._normalize_date(row['report_date']),
                platform_campaign_id=str(row['campaign_id']),
                campaign_name=row['campaign_title'],
                impressions=int(row['views']),
                clicks=int(row['link_clicks']),
                cost=float(row['spend']),
                conversions=float(row['purchases']) if pd.notna(row.get('purchases')) else None,
                revenue=float(row['total_revenue']) if pd.notna(row.get('total_revenue')) else None
            )
            records.append(record)
        
        return records
    
    def _normalize_date(self, date_str: str) -> str:
        """Convert platform date format to YYYY-MM-DD."""
        # Example: "05/01/2026" → "2026-05-01"
        from datetime import datetime
        dt = datetime.strptime(date_str, '%m/%d/%Y')
        return dt.strftime('%Y-%m-%d')
```

### 2. Register in UnifiedLoader

```python
# src/loaders/unified_loader.py
from src.adapters.your_platform.campaign_adapter import YourPlatformAdapter

class UnifiedLoader:
    @staticmethod
    def load(source: str, filepath: str) -> list[CampaignRecord]:
        df = pd.read_csv(filepath)
        
        if source == 'google_ads':
            adapter = GoogleAdsCampaignAdapter()
        elif source == 'meta_ads':
            adapter = MetaAdsAdapter()
        elif source == 'your_platform':
            adapter = YourPlatformAdapter()
        else:
            raise ValueError(f"Unsupported source: {source}")
        
        return adapter.adapt(df)
```

### 3. Test the Adapter

```python
# tests/test_your_platform_adapter.py
import pytest
import pandas as pd
from src.adapters.your_platform.campaign_adapter import YourPlatformAdapter

def test_your_platform_adapter():
    sample_data = pd.DataFrame({
        'campaign_id': ['camp_123'],
        'campaign_title': ['Summer Sale'],
        'report_date': ['05/15/2026'],
        'views': [10000],
        'link_clicks': [500],
        'spend': [250.00],
        'purchases': [20],
        'total_revenue': [1500.00]
    })
    
    adapter = YourPlatformAdapter()
    records = adapter.adapt(sample_data)
    
    assert len(records) == 1
    assert records[0].source == 'your_platform'
    assert records[0].campaign_name == 'Summer Sale'
    assert records[0].impressions == 10000
    assert records[0].clicks == 500
```

## Validation

### Validate Individual Records

```python
from src.contracts.validator import validate_record
from src.core.canonical.campaign_record import CampaignRecord

record = CampaignRecord(
    source='google_ads',
    date='2026-05-01',
    platform_campaign_id='123',
    campaign_name='Brand Search',
    impressions=10000,
    clicks=500,
    cost=250.0,
    conversions=20.0,
    revenue=1500.0
)

is_valid, errors = validate_record(record)

if is_valid:
    print("✓ Record is valid")
else:
    print(f"✗ Validation errors: {errors}")
```

### Batch Validation

```python
from src.contracts.validator import validate_record

valid_records = []
invalid_records = []

for record in records:
    is_valid, errors = validate_record(record)
    
    if is_valid:
        valid_records.append(record)
    else:
        invalid_records.append({
            'record': record,
            'errors': errors
        })

print(f"Valid: {len(valid_records)}, Invalid: {len(invalid_records)}")

# Report errors
for item in invalid_records:
    print(f"Campaign: {item['record'].campaign_name}")
    print(f"Errors: {item['errors']}")
```

## Working with Campaign Data

### Calculate Basic KPIs

```python
def calculate_kpis(record):
    """Calculate common marketing KPIs from a CampaignRecord."""
    kpis = {}
    
    # Click-Through Rate
    if record.impressions > 0:
        kpis['ctr'] = (record.clicks / record.impressions) * 100
    
    # Cost Per Click
    if record.clicks > 0:
        kpis['cpc'] = record.cost / record.clicks
    
    # Cost Per Acquisition
    if record.conversions and record.conversions > 0:
        kpis['cpa'] = record.cost / record.conversions
    
    # Return on Ad Spend
    if record.revenue and record.cost > 0:
        kpis['roas'] = record.revenue / record.cost
    
    # Conversion Rate
    if record.clicks > 0 and record.conversions:
        kpis['conversion_rate'] = (record.conversions / record.clicks) * 100
    
    return kpis

# Usage
for record in records:
    kpis = calculate_kpis(record)
    print(f"{record.campaign_name}:")
    print(f"  CTR: {kpis.get('ctr', 0):.2f}%")
    print(f"  CPC: ${kpis.get('cpc', 0):.2f}")
    print(f"  ROAS: {kpis.get('roas', 0):.2f}x")
```

### Aggregate by Date

```python
from collections import defaultdict
from datetime import datetime

def aggregate_by_date(records):
    """Aggregate metrics by date."""
    daily_stats = defaultdict(lambda: {
        'impressions': 0,
        'clicks': 0,
        'cost': 0.0,
        'conversions': 0.0,
        'revenue': 0.0
    })
    
    for record in records:
        date = record.date
        daily_stats[date]['impressions'] += record.impressions
        daily_stats[date]['clicks'] += record.clicks
        daily_stats[date]['cost'] += record.cost
        daily_stats[date]['conversions'] += record.conversions or 0
        daily_stats[date]['revenue'] += record.revenue or 0
    
    return dict(daily_stats)

# Usage
daily = aggregate_by_date(records)
for date, stats in sorted(daily.items()):
    print(f"{date}: ${stats['cost']:.2f} spent, {stats['clicks']} clicks")
```

### Compare Platforms

```python
def compare_platforms(records):
    """Compare performance across platforms."""
    platform_stats = defaultdict(lambda: {
        'campaigns': 0,
        'total_cost': 0.0,
        'total_revenue': 0.0,
        'total_conversions': 0.0
    })
    
    for record in records:
        source = record.source
        platform_stats[source]['campaigns'] += 1
        platform_stats[source]['total_cost'] += record.cost
        platform_stats[source]['total_revenue'] += record.revenue or 0
        platform_stats[source]['total_conversions'] += record.conversions or 0
    
    # Calculate ROAS per platform
    for source, stats in platform_stats.items():
        if stats['total_cost'] > 0:
            stats['roas'] = stats['total_revenue'] / stats['total_cost']
    
    return dict(platform_stats)

# Usage
comparison = compare_platforms(records)
for platform, stats in comparison.items():
    print(f"{platform}:")
    print(f"  Campaigns: {stats['campaigns']}")
    print(f"  ROAS: {stats.get('roas', 0):.2f}x")
```

## CSV Format Requirements

### Google Ads Expected Format

```csv
Date,Campaign ID,Campaign,Impressions,Clicks,Cost,Conversions,Conv. value
2026-05-01,123,Brand Search,10000,500,250.00,20,1500.00
2026-05-02,123,Brand Search,12000,600,300.00,25,1800.00
```

**Column mapping:**
- `Date` → `date`
- `Campaign ID` → `platform_campaign_id`
- `Campaign` → `campaign_name`
- `Impressions` → `impressions`
- `Clicks` → `clicks`
- `Cost` → `cost`
- `Conversions` → `conversions`
- `Conv. value` → `revenue`

## Testing

### Run All Tests

```bash
pytest tests/
```

### Run Specific Test

```bash
pytest tests/test_validator.py -v
```

### Test Coverage

```bash
pytest --cov=src tests/
```

## Common Patterns

### End-to-End Pipeline

```python
from src.loaders.unified_loader import UnifiedLoader
from src.contracts.validator import validate_record

# Load data from multiple sources
google_records = UnifiedLoader.load('google_ads', 'data/google_ads.csv')
meta_records = UnifiedLoader.load('meta_ads', 'data/meta_ads.csv')

# Combine all records
all_records = google_records + meta_records

# Validate and filter
valid_records = []
for record in all_records:
    is_valid, errors = validate_record(record)
    if is_valid:
        valid_records.append(record)
    else:
        print(f"Skipping invalid record: {errors}")

# Process valid records
for record in valid_records:
    # Your analytics logic here
    pass
```

### Error Handling

```python
from src.loaders.unified_loader import UnifiedLoader
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def safe_load_campaigns(source: str, filepath: str):
    """Load campaigns with error handling."""
    try:
        records = UnifiedLoader.load(source, filepath)
        logger.info(f"Loaded {len(records)} records from {source}")
        return records
    except FileNotFoundError:
        logger.error(f"File not found: {filepath}")
        return []
    except ValueError as e:
        logger.error(f"Invalid source or data: {e}")
        return []
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        return []
```

## Troubleshooting

### Issue: "Unsupported source" error

**Problem:** `UnifiedLoader` doesn't recognize the source name.

**Solution:** Ensure source name matches exactly (case-sensitive):
```python
# ✓ Correct
records = UnifiedLoader.load('google_ads', 'data.csv')

# ✗ Wrong
records = UnifiedLoader.load('Google Ads', 'data.csv')  # Space and caps
```

### Issue: Validation fails with date format error

**Problem:** Date format doesn't match `YYYY-MM-DD`.

**Solution:** Normalize dates in your adapter:
```python
from datetime import datetime

def _normalize_date(self, date_str: str) -> str:
    # Handle multiple formats
    for fmt in ['%Y-%m-%d', '%m/%d/%Y', '%d-%m-%Y']:
        try:
            dt = datetime.strptime(date_str, fmt)
            return dt.strftime('%Y-%m-%d')
        except ValueError:
            continue
    raise ValueError(f"Unrecognized date format: {date_str}")
```

### Issue: Missing required fields in CSV

**Problem:** CSV doesn't have all required columns.

**Solution:** Handle missing fields with defaults in adapter:
```python
def adapt(self, df: pd.DataFrame) -> list[CampaignRecord]:
    records = []
    for _, row in df.iterrows():
        record = CampaignRecord(
            source='platform_name',
            date=row['date'],
            platform_campaign_id=str(row['campaign_id']),
            campaign_name=row['campaign_name'],
            impressions=int(row['impressions']),
            clicks=int(row['clicks']),
            cost=float(row['cost']),
            conversions=float(row['conversions']) if 'conversions' in row and pd.notna(row['conversions']) else None,
            revenue=float(row['revenue']) if 'revenue' in row and pd.notna(row['revenue']) else None
        )
        records.append(record)
    return records
```

### Issue: Type errors during validation

**Problem:** Numeric fields contain non-numeric values.

**Solution:** Clean data in adapter before creating records:
```python
def _safe_int(self, value) -> int:
    try:
        return int(float(value))
    except (ValueError, TypeError):
        return 0

def _safe_float(self, value) -> float:
    try:
        return float(value)
    except (ValueError, TypeError):
        return 0.0
```

## Future Capabilities (Planned)

The project roadmap includes:
- **V0.2:** KPI Engine (automated CTR, CPC, CPA, ROAS, Conversion Rate calculation)
- **V0.3:** Insight Engine (best/worst campaign detection, trend analysis)
- **V0.4:** Campaign Health Monitor (anomaly detection, automated alerts)
- **V0.5:** Streamlit Dashboard (interactive KPI visualization)
- **V0.6:** Budget Optimizer (reallocation recommendations)
- **V0.7:** A/B Testing Framework (statistical significance testing)
- **V0.8:** Forecasting Engine (ARIMA, Prophet-based KPI prediction)

Check the project README for current implementation status.
