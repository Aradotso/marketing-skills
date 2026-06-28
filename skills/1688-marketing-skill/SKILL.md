---
name: 1688-marketing-skill
description: Expert in 1688 platform marketing operations including merchant enrollment, activity registration, product pricing suggestions, and business opportunity recommendations.
triggers:
  - "How do I register for 1688 merchant activities?"
  - "Help me find business opportunities on 1688"
  - "Check suggested pricing for my 1688 products"
  - "Submit activity enrollment on 1688"
  - "Query available merchant activities on 1688"
  - "Show me business opportunity recommendations"
  - "How to use 1688 marketing tools?"
  - "Get market opportunities on 1688 platform"
---

# 1688 Marketing Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Expert skill for helping merchants perform marketing operations on the 1688 platform, including querying merchant activities, checking product suggested pricing, submitting activity registrations, and viewing business opportunity recommendations.

## What This Project Does

1688-marketing is a marketing skill designed to help merchants on the 1688 platform (Alibaba's Chinese B2B marketplace) perform key marketing operations:

- **Activity Query**: Search and browse available merchant enrollment activities
- **Suggested Pricing**: Query recommended pricing for products based on market data
- **Activity Registration**: Submit applications to participate in merchant activities
- **Business Opportunities**: Discover and track market opportunities and recommendations

## Installation

```bash
# Clone the repository
git clone https://github.com/next-1688/1688-marketing.git
cd 1688-marketing

# Install dependencies
pip install -r requirements.txt
```

## Configuration

Set up environment variables for 1688 API credentials:

```bash
export ALIBABA_1688_APP_KEY=your_app_key
export ALIBABA_1688_APP_SECRET=your_app_secret
export ALIBABA_1688_ACCESS_TOKEN=your_access_token
```

Create a configuration file `config.py`:

```python
import os

CONFIG = {
    'app_key': os.getenv('ALIBABA_1688_APP_KEY'),
    'app_secret': os.getenv('ALIBABA_1688_APP_SECRET'),
    'access_token': os.getenv('ALIBABA_1688_ACCESS_TOKEN'),
    'api_base_url': 'https://gw.open.1688.com/openapi',
    'timeout': 30
}
```

## Core Components

### 1. Activity Query Module

Query available merchant enrollment activities:

```python
from marketing_1688.activity_query import ActivityQueryClient

# Initialize client
client = ActivityQueryClient(
    app_key=CONFIG['app_key'],
    app_secret=CONFIG['app_secret'],
    access_token=CONFIG['access_token']
)

# Query all available activities
activities = client.query_activities(
    status='ongoing',  # ongoing, upcoming, ended
    category='招商活动',
    page_size=20
)

for activity in activities['data']:
    print(f"Activity: {activity['name']}")
    print(f"Start: {activity['start_date']} - End: {activity['end_date']}")
    print(f"Enrollment deadline: {activity['enrollment_deadline']}")
    print(f"Requirements: {activity['requirements']}")
    print("---")
```

### 2. Suggested Pricing Query

Check market-based suggested pricing for products:

```python
from marketing_1688.pricing import PricingClient

pricing_client = PricingClient(
    app_key=CONFIG['app_key'],
    app_secret=CONFIG['app_secret'],
    access_token=CONFIG['access_token']
)

# Query suggested price for a product
product_id = "123456789"
price_suggestion = pricing_client.get_suggested_price(
    product_id=product_id,
    include_competitor_analysis=True
)

print(f"Product ID: {price_suggestion['product_id']}")
print(f"Suggested Price Range: {price_suggestion['min_price']} - {price_suggestion['max_price']}")
print(f"Market Average: {price_suggestion['market_average']}")
print(f"Competitor Count: {price_suggestion['competitor_count']}")

# Batch query for multiple products
product_ids = ["123456789", "987654321", "456789123"]
batch_suggestions = pricing_client.batch_get_suggested_prices(product_ids)

for suggestion in batch_suggestions:
    print(f"{suggestion['product_id']}: ¥{suggestion['recommended_price']}")
```

### 3. Activity Registration

Submit enrollment applications for merchant activities:

```python
from marketing_1688.enrollment import EnrollmentClient

enrollment_client = EnrollmentClient(
    app_key=CONFIG['app_key'],
    app_secret=CONFIG['app_secret'],
    access_token=CONFIG['access_token']
)

# Submit activity enrollment
enrollment_data = {
    'activity_id': 'ACT20260619001',
    'product_ids': ['123456789', '987654321'],
    'shop_id': 'SHOP123456',
    'contact_name': '张三',
    'contact_phone': '13800138000',
    'remarks': '希望参加此次招商活动'
}

result = enrollment_client.submit_enrollment(enrollment_data)

if result['success']:
    print(f"Enrollment successful! ID: {result['enrollment_id']}")
    print(f"Status: {result['status']}")
    print(f"Review time: {result['estimated_review_time']}")
else:
    print(f"Enrollment failed: {result['error_message']}")

# Check enrollment status
enrollment_status = enrollment_client.check_enrollment_status(
    enrollment_id=result['enrollment_id']
)
print(f"Current status: {enrollment_status['status']}")
print(f"Review notes: {enrollment_status.get('review_notes', 'N/A')}")
```

### 4. Business Opportunity Recommendations

Query and track business opportunities:

```python
from marketing_1688.opportunities import OpportunityClient

opp_client = OpportunityClient(
    app_key=CONFIG['app_key'],
    app_secret=CONFIG['app_secret'],
    access_token=CONFIG['access_token']
)

# Get personalized business opportunities
opportunities = opp_client.get_recommendations(
    shop_id='SHOP123456',
    category='电子产品',
    min_score=0.7  # Relevance score threshold
)

for opp in opportunities['data']:
    print(f"Opportunity: {opp['title']}")
    print(f"Category: {opp['category']}")
    print(f"Relevance Score: {opp['score']}")
    print(f"Potential Revenue: ¥{opp['estimated_revenue']}")
    print(f"Competition Level: {opp['competition_level']}")
    print(f"Description: {opp['description']}")
    print("---")

# Track opportunity performance
opp_client.track_opportunity(
    opportunity_id='OPP20260619001',
    action='viewed'  # viewed, interested, applied
)
```

## Common Patterns

### Complete Activity Enrollment Workflow

```python
from marketing_1688 import Marketing1688Client

# Initialize unified client
client = Marketing1688Client(config=CONFIG)

# Step 1: Search for relevant activities
activities = client.activities.search(
    keywords='夏季促销',
    category='服装',
    status='ongoing'
)

# Step 2: Get detailed activity information
activity_detail = client.activities.get_detail(
    activity_id=activities[0]['id']
)

# Step 3: Check product pricing suggestions
products_to_submit = ['123456789', '987654321']
pricing_check = client.pricing.batch_check(products_to_submit)

# Step 4: Submit enrollment with optimal pricing
enrollment = client.enrollment.submit({
    'activity_id': activity_detail['id'],
    'product_ids': products_to_submit,
    'pricing_strategy': pricing_check,
    'auto_adjust_price': True
})

# Step 5: Monitor enrollment status
status = client.enrollment.monitor(enrollment['id'], auto_refresh=True)
```

### Business Opportunity Pipeline

```python
from marketing_1688 import Marketing1688Client

client = Marketing1688Client(config=CONFIG)

# Build opportunity pipeline
pipeline = client.opportunities.create_pipeline(
    filters={
        'categories': ['电子产品', '家居用品'],
        'min_score': 0.75,
        'competition_level': ['low', 'medium']
    }
)

# Daily opportunity digest
daily_opportunities = pipeline.get_daily_digest()

for opp in daily_opportunities:
    # Check if products match
    matching_products = client.products.find_matching(
        opportunity_requirements=opp['requirements']
    )
    
    if matching_products:
        # Auto-apply if high relevance
        if opp['score'] > 0.9:
            client.opportunities.auto_apply(
                opportunity_id=opp['id'],
                product_ids=[p['id'] for p in matching_products]
            )
```

### Price Optimization Strategy

```python
from marketing_1688.pricing import PricingOptimizer

optimizer = PricingOptimizer(config=CONFIG)

# Analyze product pricing
product_id = "123456789"
analysis = optimizer.analyze_product(product_id)

# Get optimization recommendations
recommendations = optimizer.get_recommendations(
    current_price=analysis['current_price'],
    market_data=analysis['market_data'],
    goals=['maximize_profit', 'increase_sales_volume']
)

# Apply dynamic pricing
optimizer.apply_dynamic_pricing(
    product_id=product_id,
    strategy=recommendations['optimal_strategy'],
    price_range=(recommendations['min_safe_price'], recommendations['max_safe_price']),
    auto_adjust=True,
    adjustment_frequency='daily'
)
```

## Error Handling

```python
from marketing_1688.exceptions import (
    AuthenticationError,
    EnrollmentError,
    QuotaExceededError
)

try:
    result = client.enrollment.submit(enrollment_data)
except AuthenticationError as e:
    print(f"Auth failed: {e.message}")
    # Refresh access token
    client.refresh_token()
except EnrollmentError as e:
    print(f"Enrollment error: {e.message}")
    print(f"Error code: {e.code}")
    # Check eligibility requirements
    eligibility = client.activities.check_eligibility(activity_id)
except QuotaExceededError as e:
    print(f"API quota exceeded: {e.message}")
    print(f"Reset time: {e.reset_time}")
    # Implement retry with backoff
except Exception as e:
    print(f"Unexpected error: {str(e)}")
```

## Troubleshooting

### Authentication Issues

```python
# Verify credentials
from marketing_1688.auth import verify_credentials

is_valid = verify_credentials(
    app_key=CONFIG['app_key'],
    app_secret=CONFIG['app_secret'],
    access_token=CONFIG['access_token']
)

if not is_valid:
    # Regenerate access token
    from marketing_1688.auth import refresh_access_token
    new_token = refresh_access_token(
        app_key=CONFIG['app_key'],
        app_secret=CONFIG['app_secret']
    )
```

### Rate Limiting

```python
# Implement rate limiting
from marketing_1688.utils import RateLimiter

limiter = RateLimiter(max_requests=100, time_window=60)

with limiter:
    activities = client.activities.query_all()
```

### Data Validation

```python
# Validate enrollment data before submission
from marketing_1688.validators import EnrollmentValidator

validator = EnrollmentValidator()
is_valid, errors = validator.validate(enrollment_data)

if not is_valid:
    for error in errors:
        print(f"Validation error: {error['field']} - {error['message']}")
```

## Best Practices

1. **Always use environment variables** for sensitive credentials
2. **Implement retry logic** for API calls with exponential backoff
3. **Cache activity and pricing data** to reduce API calls
4. **Monitor enrollment status** regularly for timely responses
5. **Batch operations** when possible to optimize API quota usage
6. **Validate data** before submission to avoid rejection
