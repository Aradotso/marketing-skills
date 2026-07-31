---
name: genpark-marketing-cloud-sql-audience-generator
description: Natural language audience intent to Marketing Cloud SQL generator for creating targeted audience segments
triggers:
  - generate Marketing Cloud SQL from audience description
  - create audience segment query for Salesforce Marketing Cloud
  - convert natural language to Marketing Cloud SQL
  - build audience filter using GenPark
  - generate SQL for marketing cloud audience
  - create targeted audience segment with natural language
  - translate audience intent to SFMC SQL
  - build marketing cloud data extension query
---

# GenPark Marketing Cloud SQL Audience Generator Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This skill enables AI agents to generate Salesforce Marketing Cloud SQL queries from natural language audience intent descriptions. It translates marketing team requests like "customers who purchased in the last 30 days" into valid Marketing Cloud SQL syntax for audience segmentation and data extension queries.

## What It Does

The GenPark Marketing Cloud SQL Audience Generator converts natural language descriptions of target audiences into properly formatted SQL queries compatible with Salesforce Marketing Cloud (SFMC). This bridges the gap between marketing intent and technical implementation, allowing marketers to define audiences in plain English while generating production-ready SQL.

**Key capabilities:**
- Natural language to Marketing Cloud SQL translation
- Audience segmentation query generation
- Data extension filtering logic
- Support for temporal, behavioral, and demographic criteria
- SFMC-specific SQL syntax compliance

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-marketing-cloud-sql-audience-generator-skill.git
cd genpark-marketing-cloud-sql-audience-generator-skill

# Install dependencies
pip install -r requirements.txt
```

**Dependencies typically include:**
- Python 3.8+
- OpenAI API or compatible LLM endpoint
- Standard libraries (requests, json, etc.)

## Configuration

Set up your environment variables for LLM API access:

```bash
# .env file
OPENAI_API_KEY=your_api_key_here
GENPARK_API_ENDPOINT=https://api.genpark.ai  # if using GenPark hosted service
MODEL_NAME=gpt-4  # or your preferred model
```

Load environment variables in your code:

```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')
```

## Basic Usage

### Running the Example

```bash
python example_usage.py
```

### Python API Usage

```python
from genpark_audience_generator import AudienceQueryGenerator

# Initialize the generator
generator = AudienceQueryGenerator(
    api_key=os.getenv('OPENAI_API_KEY'),
    model='gpt-4'
)

# Generate SQL from natural language
audience_intent = "Find all customers who opened an email in the last 7 days but didn't click"
sql_query = generator.generate_sql(audience_intent)

print(sql_query)
```

**Expected output:**
```sql
SELECT 
    s.SubscriberKey,
    s.EmailAddress,
    s.FirstName,
    s.LastName
FROM _Subscribers s
INNER JOIN _Open o ON s.SubscriberKey = o.SubscriberKey
LEFT JOIN _Click c ON s.SubscriberKey = c.SubscriberKey 
    AND c.EventDate >= DATEADD(day, -7, GETDATE())
WHERE 
    o.EventDate >= DATEADD(day, -7, GETDATE())
    AND c.SubscriberKey IS NULL
```

## Common Patterns

### Purchase Behavior Segmentation

```python
# Recent purchasers
intent = "Customers who made a purchase over $100 in the last 30 days"
query = generator.generate_sql(intent)

# Example output targets Purchase data extension
```

```sql
SELECT 
    c.SubscriberKey,
    c.EmailAddress,
    SUM(p.PurchaseAmount) as TotalSpend
FROM Customers c
INNER JOIN Purchases p ON c.CustomerID = p.CustomerID
WHERE 
    p.PurchaseDate >= DATEADD(day, -30, GETDATE())
GROUP BY 
    c.SubscriberKey,
    c.EmailAddress
HAVING 
    SUM(p.PurchaseAmount) > 100
```

### Engagement-Based Audiences

```python
# High engagement subscribers
intent = "Subscribers who clicked at least 3 emails in the past month"
query = generator.generate_sql(intent)
```

```sql
SELECT 
    s.SubscriberKey,
    s.EmailAddress,
    COUNT(DISTINCT c.JobID) as EmailsClicked
FROM _Subscribers s
INNER JOIN _Click c ON s.SubscriberKey = c.SubscriberKey
WHERE 
    c.EventDate >= DATEADD(month, -1, GETDATE())
GROUP BY 
    s.SubscriberKey,
    s.EmailAddress
HAVING 
    COUNT(DISTINCT c.JobID) >= 3
```

### Churn Prevention

```python
# At-risk customers
intent = "Active customers who haven't purchased in 60 days but were previously monthly buyers"
query = generator.generate_sql(intent)
```

### Multi-Criteria Segmentation

```python
# Complex audience definition
intent = """
Find subscribers who:
- Live in California, New York, or Texas
- Have lifetime value over $500
- Opened at least one email in the last 14 days
- Are subscribed to the newsletter
"""
query = generator.generate_sql(intent)
```

## Advanced Usage

### Custom Data Extensions

```python
# Specify your data extension schema
context = {
    "data_extensions": {
        "CustomerProfile": ["SubscriberKey", "Email", "State", "LTV", "MemberSince"],
        "EmailEngagement": ["SubscriberKey", "JobID", "EventDate", "EventType"],
        "Preferences": ["SubscriberKey", "NewsletterOptIn", "Category"]
    }
}

generator = AudienceQueryGenerator(
    api_key=os.getenv('OPENAI_API_KEY'),
    context=context
)

intent = "Newsletter subscribers in California with LTV over $500"
query = generator.generate_sql(intent)
```

### Query Validation

```python
# Validate generated SQL before use
from genpark_audience_generator import validate_query

sql = generator.generate_sql(intent)
is_valid, errors = validate_query(sql)

if is_valid:
    print("Query is valid")
else:
    print(f"Validation errors: {errors}")
```

### Batch Generation

```python
# Generate multiple audience queries
audience_definitions = [
    "VIP customers with purchases over $1000 this year",
    "Engaged subscribers who opened 5+ emails this month",
    "Inactive subscribers with no opens in 90 days"
]

queries = {}
for intent in audience_definitions:
    queries[intent] = generator.generate_sql(intent)
    
# Export queries
import json
with open('audience_queries.json', 'w') as f:
    json.dump(queries, f, indent=2)
```

## Integration with Marketing Cloud

### Using Generated SQL in SFMC

```python
# Generate and format for SFMC Query Activity
intent = "Customers who abandoned cart in last 24 hours"
sql = generator.generate_sql(intent)

# Add target data extension reference
sfmc_query = f"""
-- Target DE: AbandonedCartAudience
{sql}
"""

print("Copy this SQL into Marketing Cloud Query Activity:")
print(sfmc_query)
```

### Automation Studio Integration

```python
# Generate queries for multiple audience steps
workflow = {
    "step_1_engaged": "Opened email in last 7 days",
    "step_2_not_purchased": "No purchases in last 30 days from step 1",
    "step_3_high_value": "Lifetime value over $200 from step 2"
}

for step_name, intent in workflow.items():
    print(f"\n--- {step_name} ---")
    print(generator.generate_sql(intent))
```

## Troubleshooting

### Common Issues

**API Key Errors**
```python
# Check environment variable is set
import os
if not os.getenv('OPENAI_API_KEY'):
    raise ValueError("OPENAI_API_KEY not set in environment")
```

**Invalid SQL Output**
- Ensure your intent is specific and includes timeframes
- Provide data extension context for better accuracy
- Validate generated SQL before deploying to SFMC

**Model Timeout**
```python
# Increase timeout for complex queries
generator = AudienceQueryGenerator(
    api_key=os.getenv('OPENAI_API_KEY'),
    timeout=60  # seconds
)
```

**SFMC System Table References**
- Use `_Subscribers`, `_Open`, `_Click`, `_Bounce`, `_Sent` for engagement data
- Ensure date functions use `GETDATE()` and `DATEADD()` (SFMC syntax)
- Avoid unsupported SQL functions

### Debugging Generated Queries

```python
# Enable verbose output
generator = AudienceQueryGenerator(
    api_key=os.getenv('OPENAI_API_KEY'),
    verbose=True
)

# See the full prompt and response
sql = generator.generate_sql(intent, debug=True)
```

## Best Practices

1. **Be specific with intent**: Include timeframes, metrics, and thresholds
2. **Validate before deployment**: Always test queries in SFMC Query Studio
3. **Provide schema context**: Better results with data extension definitions
4. **Use system tables correctly**: Reference SFMC system views appropriately
5. **Test with sample data**: Verify logic with small data sets first
6. **Document generated queries**: Keep track of audience definitions and SQL

## Example Workflow

```python
import os
from genpark_audience_generator import AudienceQueryGenerator

# Initialize
generator = AudienceQueryGenerator(api_key=os.getenv('OPENAI_API_KEY'))

# Define audience
intent = "Email subscribers who clicked a product link in the last 14 days but haven't purchased"

# Generate SQL
sql = generator.generate_sql(intent)

# Review and save
print("Generated SQL:")
print(sql)

with open('product_interest_audience.sql', 'w') as f:
    f.write(f"-- Audience: {intent}\n")
    f.write(f"-- Generated: {datetime.now()}\n\n")
    f.write(sql)

print("\nReady to import into Marketing Cloud Query Activity")
```
