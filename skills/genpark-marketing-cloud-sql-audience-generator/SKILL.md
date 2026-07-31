---
name: genpark-marketing-cloud-sql-audience-generator
description: Natural language to Marketing Cloud SQL converter for building targeted audience segments
triggers:
  - generate marketing cloud sql query
  - create audience segment query
  - build marketing cloud audience
  - convert audience intent to sql
  - generate sfmc sql query
  - create data extension query for marketing cloud
  - build audience segment with natural language
  - translate marketing criteria to sql
---

# GenPark Marketing Cloud SQL Audience Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This project converts natural language audience intent into Marketing Cloud SQL queries, enabling marketers and developers to build targeted audience segments without deep SQL knowledge. It translates plain English descriptions of target audiences into valid Salesforce Marketing Cloud (SFMC) SQL queries for data extensions.

## Installation

```bash
git clone https://github.com/alphaparkinc/genpark-marketing-cloud-sql-audience-generator-skill.git
cd genpark-marketing-cloud-sql-audience-generator-skill
pip install -r requirements.txt
```

## Core Functionality

The skill generates SQL queries for Marketing Cloud data extensions based on natural language input. It understands common marketing segmentation criteria like demographics, behavior, engagement, and purchase history.

## Basic Usage

```python
from genpark_audience_generator import AudienceGenerator

# Initialize the generator
generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))

# Generate SQL from natural language
intent = "customers who purchased in the last 30 days and opened at least 3 emails"
sql_query = generator.generate_sql(intent)

print(sql_query)
```

Expected output:
```sql
SELECT 
    c.SubscriberKey,
    c.EmailAddress,
    c.FirstName,
    c.LastName
FROM Customers c
INNER JOIN Purchases p ON c.SubscriberKey = p.SubscriberKey
INNER JOIN EmailEngagement e ON c.SubscriberKey = e.SubscriberKey
WHERE p.PurchaseDate >= DATEADD(day, -30, GETDATE())
AND e.EventType = 'Open'
GROUP BY c.SubscriberKey, c.EmailAddress, c.FirstName, c.LastName
HAVING COUNT(DISTINCT e.EventDate) >= 3
```

## Configuration

Set up environment variables:

```bash
export GENPARK_API_KEY=your_api_key_here
export SFMC_SCHEMA=your_schema_name  # Optional: default schema for data extensions
export GENPARK_MODEL=gpt-4  # Optional: specify LLM model
```

Configuration file example (`config.yaml`):

```yaml
api:
  endpoint: https://genpark.ai/api/v1
  timeout: 30

marketing_cloud:
  default_schema: "ENT.Customers"
  data_extensions:
    - Customers
    - Purchases
    - EmailEngagement
    - WebActivity
    - Subscriptions

query_options:
  include_comments: true
  format_output: true
  max_results: 5000
```

## Common Audience Patterns

### High-Value Customer Segment

```python
from genpark_audience_generator import AudienceGenerator

generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))

# High-value customers based on purchase behavior
intent = """
Find customers who:
- Made purchases totaling over $1000 in the last 6 months
- Have purchased at least 3 times
- Are subscribed to the premium newsletter
"""

sql = generator.generate_sql(intent)
print(sql)
```

### Re-engagement Campaign

```python
# Identify customers to re-engage
intent = """
Target customers who:
- Haven't purchased in 60-90 days
- Previously purchased more than twice
- Have opened emails in the last 30 days
- Are not unsubscribed
"""

sql = generator.generate_sql(
    intent,
    output_fields=['SubscriberKey', 'EmailAddress', 'LastPurchaseDate']
)
```

### Geographic Targeting

```python
# Location-based segment
intent = """
Customers in California or New York
who have purchased outdoor gear
and have a lifetime value over $500
"""

sql = generator.generate_sql(intent, schema='ENT.RetailCustomers')
```

## Advanced Usage

### Custom Data Extension Mapping

```python
from genpark_audience_generator import AudienceGenerator, DataExtensionConfig

# Define your data extension schema
de_config = DataExtensionConfig(
    name='CustomersDE',
    fields={
        'SubscriberKey': 'Text',
        'EmailAddress': 'EmailAddress',
        'FirstName': 'Text',
        'LastName': 'Text',
        'City': 'Text',
        'State': 'Text',
        'LTV': 'Decimal',
        'LastPurchase': 'Date'
    },
    primary_key='SubscriberKey'
)

generator = AudienceGenerator(
    api_key=os.getenv('GENPARK_API_KEY'),
    data_extensions=[de_config]
)

intent = "Customers in Texas with LTV over $1000"
sql = generator.generate_sql(intent, target_de='CustomersDE')
```

### Batch Processing

```python
import json
from genpark_audience_generator import AudienceGenerator

generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))

# Process multiple audience definitions
audience_intents = [
    "Active customers in the last 30 days",
    "Abandoned cart in the last 7 days",
    "VIP customers with 10+ purchases",
    "Newsletter subscribers who never purchased"
]

results = []
for intent in audience_intents:
    try:
        sql = generator.generate_sql(intent)
        results.append({
            'intent': intent,
            'sql': sql,
            'status': 'success'
        })
    except Exception as e:
        results.append({
            'intent': intent,
            'error': str(e),
            'status': 'failed'
        })

# Save results
with open('audience_queries.json', 'w') as f:
    json.dump(results, f, indent=2)
```

### Query Validation

```python
from genpark_audience_generator import AudienceGenerator, QueryValidator

generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))
validator = QueryValidator()

intent = "Customers who clicked on Black Friday email"
sql = generator.generate_sql(intent)

# Validate before using in Marketing Cloud
validation = validator.validate(sql)
if validation.is_valid:
    print(f"Valid SQL: {sql}")
else:
    print(f"Validation errors: {validation.errors}")
```

## API Reference

### AudienceGenerator Class

```python
class AudienceGenerator:
    def __init__(
        self,
        api_key: str,
        endpoint: str = None,
        data_extensions: list = None,
        model: str = 'gpt-4'
    ):
        """Initialize the audience generator"""
        pass
    
    def generate_sql(
        self,
        intent: str,
        schema: str = None,
        target_de: str = None,
        output_fields: list = None,
        validate: bool = True
    ) -> str:
        """Generate Marketing Cloud SQL from natural language"""
        pass
    
    def explain_query(self, sql: str) -> dict:
        """Get natural language explanation of SQL query"""
        pass
```

### Key Methods

- `generate_sql()`: Converts natural language to SQL
- `explain_query()`: Reverse operation - SQL to natural language
- `validate_schema()`: Check data extension schema compatibility
- `optimize_query()`: Suggest query optimizations

## CLI Usage

```bash
# Generate SQL from command line
python genpark_audience_generator.py --intent "customers who purchased last week"

# With custom schema
python genpark_audience_generator.py \
  --intent "high-value customers" \
  --schema "ENT.RetailCustomers" \
  --output output.sql

# Batch mode
python genpark_audience_generator.py --batch intents.txt --output-dir ./queries/

# Validate existing query
python genpark_audience_generator.py --validate existing_query.sql
```

## Example: Complete Workflow

```python
import os
from genpark_audience_generator import AudienceGenerator
from sfmc_client import MarketingCloudClient  # Hypothetical SFMC client

# Initialize
generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))
mc_client = MarketingCloudClient(
    client_id=os.getenv('SFMC_CLIENT_ID'),
    client_secret=os.getenv('SFMC_CLIENT_SECRET')
)

# Define audience intent
intent = """
Create a segment of customers who:
- Purchased women's apparel in Q4 2025
- Have an email open rate above 25%
- Live in metro areas
- Are not currently in any active campaign
"""

# Generate SQL
sql_query = generator.generate_sql(
    intent,
    schema='ENT.CustomerMaster',
    output_fields=['SubscriberKey', 'EmailAddress', 'FirstName', 'City']
)

# Create data extension and execute query
de_name = 'Q4_WomensApparel_Engaged_Metro'
mc_client.create_data_extension(de_name, sql_query)
mc_client.execute_query(sql_query, target_de=de_name)

print(f"Audience segment created: {de_name}")
```

## Troubleshooting

### API Authentication Errors

```python
# Check API key configuration
import os
if not os.getenv('GENPARK_API_KEY'):
    raise ValueError("GENPARK_API_KEY environment variable not set")

# Test connection
from genpark_audience_generator import AudienceGenerator
generator = AudienceGenerator(api_key=os.getenv('GENPARK_API_KEY'))
generator.test_connection()
```

### Invalid SQL Generated

```python
# Enable debug mode for detailed logging
generator = AudienceGenerator(
    api_key=os.getenv('GENPARK_API_KEY'),
    debug=True
)

# Add more context to intent
intent = """
Data extension: Customers (fields: SubscriberKey, EmailAddress, City, State, LTV)
Find: Customers in California with LTV > 1000
"""
sql = generator.generate_sql(intent)
```

### Query Performance Issues

```python
# Request optimized query
sql = generator.generate_sql(
    intent,
    optimize=True,
    max_complexity='medium'
)

# Or optimize existing query
optimized_sql = generator.optimize_query(sql)
```

### Schema Mismatch

```python
# Explicitly define schema
from genpark_audience_generator import DataExtensionConfig

schema = DataExtensionConfig.from_file('my_schema.json')
generator = AudienceGenerator(
    api_key=os.getenv('GENPARK_API_KEY'),
    data_extensions=[schema]
)
```

## Best Practices

1. **Be specific**: More detailed intents produce better SQL
2. **Define schema**: Provide data extension schema for accuracy
3. **Validate queries**: Always validate before executing in production
4. **Use environment variables**: Never hardcode API keys
5. **Test with small datasets**: Validate logic before full execution
6. **Monitor performance**: Track query execution times in Marketing Cloud

## Resources

- Homepage: https://genpark.ai
- Repository: https://github.com/alphaparkinc/genpark-marketing-cloud-sql-audience-generator-skill
- Marketing Cloud SQL Reference: Salesforce Marketing Cloud SQL documentation
