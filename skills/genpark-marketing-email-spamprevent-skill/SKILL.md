---
name: genpark-marketing-email-spamprevent-skill
description: Scan marketing email drafts for spam trigger words and calculate deliverability scores to prevent blocklisting
triggers:
  - check this email for spam triggers
  - analyze my marketing email for deliverability
  - scan this newsletter draft for spam words
  - calculate spam score for this email copy
  - help me avoid email spam filters
  - check if my email will be blocked
  - validate my marketing email content
  - prevent my email from going to spam
---

# genpark-marketing-email-spamprevent-skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

GenPark Marketing Email Spam Prevent is a Python-based AI skill that analyzes marketing email content (subject lines and body copy) to identify spam trigger words and calculate deliverability scores. It helps prevent legitimate marketing emails from being flagged by spam filters or causing domain blocklisting.

The tool evaluates content against known spam patterns, aggressive marketing language, and suspicious formatting to provide actionable feedback before sending campaigns.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-marketing-email-spamprevent-skill.git
cd genpark-marketing-email-spamprevent-skill

# Install dependencies
pip install -r requirements.txt
```

If no requirements.txt exists, the core dependencies are typically:

```bash
pip install requests python-dotenv
```

## Basic Usage

### Python Client API

```python
from client import MarketingEmailSpamPreventClient

# Initialize the client
client = MarketingEmailSpamPreventClient()

# Analyze email content
result = client.analyze_copy(
    subject="Special Promotion Just For You!",
    body="Buy now and save 50%! Click here immediately to claim your prize!"
)

# Check the spam score (0-100, higher = more likely to be flagged)
print(f"Spam Score: {result['spam_score']}")
print(f"Risk Level: {result['risk_level']}")
print(f"Trigger Words Found: {result['trigger_words']}")
```

### Detailed Analysis Response

```python
result = client.analyze_copy(
    subject="Weekly Newsletter - Marketing Tips",
    body="Hello! Here are this week's top marketing insights for your business."
)

# Expected response structure
{
    "spam_score": 15,  # 0-100 scale
    "risk_level": "low",  # low, medium, high, critical
    "trigger_words": [],
    "recommendations": [
        "Good use of personalized greeting",
        "Subject line is clear and professional"
    ],
    "issues": [],
    "deliverability_estimate": 0.95  # 0-1 probability
}
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# API Configuration (if using remote analysis)
GENPARK_API_KEY=your_api_key_here
GENPARK_API_URL=https://api.genpark.ai

# Scoring Thresholds
SPAM_SCORE_LOW=30
SPAM_SCORE_MEDIUM=60
SPAM_SCORE_HIGH=80

# Analysis Options
STRICT_MODE=false
INCLUDE_SUGGESTIONS=true
```

### Configuration File

Create `config.json` for custom trigger word lists:

```json
{
  "trigger_words": {
    "high_risk": ["free money", "guaranteed income", "click here now"],
    "medium_risk": ["limited time", "act now", "special promotion"],
    "low_risk": ["discount", "sale", "offer"]
  },
  "scoring_weights": {
    "subject_line": 0.4,
    "body_content": 0.4,
    "formatting": 0.2
  }
}
```

## Key API Methods

### analyze_copy()

Main method for analyzing email content:

```python
result = client.analyze_copy(
    subject="Your subject line",
    body="Email body content",
    options={
        "strict_mode": False,
        "include_html_analysis": True,
        "check_links": True
    }
)
```

### batch_analyze()

Analyze multiple emails at once:

```python
emails = [
    {"subject": "Newsletter #1", "body": "Content 1"},
    {"subject": "Newsletter #2", "body": "Content 2"},
    {"subject": "Newsletter #3", "body": "Content 3"}
]

results = client.batch_analyze(emails)
for idx, result in enumerate(results):
    print(f"Email {idx+1} - Score: {result['spam_score']}")
```

### get_suggestions()

Get improvement suggestions for flagged content:

```python
suggestions = client.get_suggestions(
    subject="FREE MONEY NOW!!!",
    body="Click here to claim your prize immediately!"
)

print("Suggested Changes:")
for suggestion in suggestions:
    print(f"- {suggestion['issue']}: {suggestion['fix']}")
```

## Common Patterns

### Pre-Send Email Validation

```python
from client import MarketingEmailSpamPreventClient

def validate_campaign_email(subject, body, threshold=50):
    """Validate email before sending campaign"""
    client = MarketingEmailSpamPreventClient()
    result = client.analyze_copy(subject, body)
    
    if result['spam_score'] > threshold:
        print(f"⚠️  Warning: High spam score ({result['spam_score']})")
        print("Trigger words found:", result['trigger_words'])
        return False
    
    print(f"✓ Email passed validation (score: {result['spam_score']})")
    return True

# Usage
is_safe = validate_campaign_email(
    subject="Weekly Marketing Insights",
    body="Here are this week's top strategies..."
)
```

### Newsletter Template Testing

```python
def test_newsletter_template(template_path):
    """Test a newsletter template for spam triggers"""
    client = MarketingEmailSpamPreventClient()
    
    with open(template_path, 'r') as f:
        content = f.read()
    
    # Extract subject from template (example)
    subject = content.split('<subject>')[1].split('</subject>')[0]
    body = content.split('<body>')[1].split('</body>')[0]
    
    result = client.analyze_copy(subject, body)
    
    return {
        'template': template_path,
        'score': result['spam_score'],
        'safe_to_use': result['spam_score'] < 40
    }

# Test all templates
templates = ['template1.html', 'template2.html', 'template3.html']
for template in templates:
    result = test_newsletter_template(template)
    print(f"{result['template']}: {'✓' if result['safe_to_use'] else '✗'} ({result['score']})")
```

### A/B Testing Subject Lines

```python
def compare_subject_lines(subjects, body):
    """Compare multiple subject line variants"""
    client = MarketingEmailSpamPreventClient()
    results = []
    
    for subject in subjects:
        result = client.analyze_copy(subject, body)
        results.append({
            'subject': subject,
            'score': result['spam_score'],
            'risk': result['risk_level']
        })
    
    # Sort by best (lowest) spam score
    results.sort(key=lambda x: x['score'])
    return results

# Usage
variants = [
    "🎉 HUGE SALE - Buy Now!",
    "Weekly Special Offer Inside",
    "Your Personalized Recommendations"
]

body = "Check out these products selected for you..."
rankings = compare_subject_lines(variants, body)

print("Subject Line Rankings (best to worst):")
for i, r in enumerate(rankings, 1):
    print(f"{i}. {r['subject']} - Score: {r['score']} ({r['risk']})")
```

### Integration with Email Service

```python
import os
from client import MarketingEmailSpamPreventClient

def send_safe_email(to_address, subject, body, email_service):
    """Only send email if it passes spam check"""
    client = MarketingEmailSpamPreventClient()
    
    # Analyze before sending
    result = client.analyze_copy(subject, body)
    
    if result['spam_score'] > 70:
        print(f"❌ Email blocked - spam score too high: {result['spam_score']}")
        print("Issues found:", result['issues'])
        return None
    
    if result['spam_score'] > 40:
        print(f"⚠️  Warning: Medium spam score ({result['spam_score']})")
        user_confirm = input("Send anyway? (y/n): ")
        if user_confirm.lower() != 'y':
            return None
    
    # Send through your email service
    return email_service.send(to=to_address, subject=subject, body=body)
```

## Troubleshooting

### High Spam Scores on Legitimate Content

If legitimate emails score too high:

```python
# Use strict_mode=False for more lenient scoring
result = client.analyze_copy(
    subject="Your subject",
    body="Your body",
    options={"strict_mode": False}
)

# Check which specific words triggered
print("Triggers:", result['trigger_words'])
# Replace or rephrase trigger words
```

### Missing Dependencies

```python
# If MarketingEmailSpamPreventClient import fails
import sys
sys.path.append('./src')  # Adjust to actual source directory
from client import MarketingEmailSpamPreventClient
```

### API Connection Issues

```python
import os

# Verify environment variables are loaded
from dotenv import load_dotenv
load_dotenv()

print("API Key set:", bool(os.getenv('GENPARK_API_KEY')))
print("API URL:", os.getenv('GENPARK_API_URL', 'Not set'))

# Use local mode if API unavailable
client = MarketingEmailSpamPreventClient(local_mode=True)
```

### Custom Trigger Word Lists

```python
# Override default trigger words
custom_triggers = {
    "high_risk": ["your custom", "high risk words"],
    "medium_risk": ["medium risk", "trigger words"]
}

client = MarketingEmailSpamPreventClient(
    custom_triggers=custom_triggers
)
```

## Best Practices

1. **Test Early**: Analyze copy before finalizing email designs
2. **Monitor Scores**: Track spam scores across campaigns to identify patterns
3. **Iterate Subject Lines**: Test multiple variants before sending
4. **Keep Scores Low**: Aim for spam scores below 30 for best deliverability
5. **Review Triggers**: Regularly review and update custom trigger word lists
6. **Use Environment Variables**: Never hardcode API keys in scripts

## Example Workflow

```python
from client import MarketingEmailSpamPreventClient
import os

def email_validation_workflow():
    client = MarketingEmailSpamPreventClient()
    
    # Draft content
    subject = "New Product Launch - Exclusive Preview"
    body = """
    Hello,
    
    We're excited to share our new product with you.
    Get early access and 20% off during launch week.
    
    View products: https://example.com/launch
    
    Best regards,
    Marketing Team
    """
    
    # Step 1: Initial analysis
    result = client.analyze_copy(subject, body)
    print(f"Initial Score: {result['spam_score']}")
    
    # Step 2: If score is high, get suggestions
    if result['spam_score'] > 40:
        suggestions = client.get_suggestions(subject, body)
        print("\nSuggested improvements:")
        for s in suggestions:
            print(f"- {s}")
    
    # Step 3: Make adjustments and re-test
    improved_body = body.replace("20% off", "a special discount")
    result2 = client.analyze_copy(subject, improved_body)
    print(f"\nImproved Score: {result2['spam_score']}")
    
    return result2['spam_score'] < 30

if __name__ == "__main__":
    email_validation_workflow()
```
