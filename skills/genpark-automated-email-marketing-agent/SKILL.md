---
name: genpark-automated-email-marketing-agent
description: Goal-driven email sequence generator and A/B testing agent for automated marketing campaigns
triggers:
  - generate email marketing sequences
  - create A/B test for email campaigns
  - build automated email funnel
  - optimize email marketing performance
  - set up email sequence automation
  - design goal-driven email campaigns
  - test email subject lines and content
  - automate marketing email workflows
---

# GenPark Automated Email Marketing Agent

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

GenPark Automated Email Marketing Agent is a goal-driven email sequence generator with built-in A/B testing capabilities. It helps create, optimize, and automate email marketing campaigns by generating contextually relevant email sequences and testing variations to maximize engagement and conversion rates.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-automated-email-marketing-agent-skill.git
cd genpark-automated-email-marketing-agent-skill

# Install dependencies
pip install -r requirements.txt
```

## Configuration

Set up required environment variables:

```bash
export OPENAI_API_KEY=your_openai_api_key
export SENDGRID_API_KEY=your_sendgrid_api_key  # Optional, for sending emails
export GENPARK_DB_PATH=./data/sequences.db  # Optional, defaults to local SQLite
```

## Core Components

### Email Sequence Generator

Generate goal-driven email sequences based on campaign objectives:

```python
from genpark.sequence_generator import EmailSequenceGenerator
from genpark.models import CampaignGoal

# Initialize generator
generator = EmailSequenceGenerator(api_key=os.getenv("OPENAI_API_KEY"))

# Define campaign goal
goal = CampaignGoal(
    objective="nurture_leads",
    target_audience="SaaS developers",
    desired_action="sign_up_trial",
    tone="professional_friendly",
    sequence_length=5
)

# Generate email sequence
sequence = generator.generate_sequence(goal)

for i, email in enumerate(sequence.emails, 1):
    print(f"Email {i}: {email.subject}")
    print(f"Delay: {email.send_delay_hours}h")
    print(f"Body preview: {email.body[:100]}...")
    print("---")
```

### A/B Testing Agent

Create and manage A/B tests for email optimization:

```python
from genpark.ab_testing import ABTestManager
from genpark.models import EmailVariant

# Initialize A/B test manager
ab_manager = ABTestManager()

# Create variants
variant_a = EmailVariant(
    subject="Unlock Your Free Trial Today",
    body="Dear {first_name},\n\nStart your 14-day free trial...",
    cta_text="Start Free Trial",
    variant_name="A"
)

variant_b = EmailVariant(
    subject="Ready to Transform Your Workflow?",
    body="Hi {first_name},\n\nDiscover how our platform...",
    cta_text="Try It Free",
    variant_name="B"
)

# Set up A/B test
test = ab_manager.create_test(
    test_name="onboarding_email_1",
    variants=[variant_a, variant_b],
    split_ratio=0.5,
    success_metric="click_through_rate",
    sample_size=1000
)

print(f"Test ID: {test.test_id}")
print(f"Status: {test.status}")
```

### Campaign Execution

Execute automated email campaigns with tracking:

```python
from genpark.campaign import CampaignExecutor
from genpark.tracking import AnalyticsTracker

# Initialize campaign executor
executor = CampaignExecutor(
    sendgrid_api_key=os.getenv("SENDGRID_API_KEY")
)

# Load or create sequence
sequence = generator.generate_sequence(goal)

# Execute campaign
campaign = executor.launch_campaign(
    sequence=sequence,
    recipients=["user@example.com"],
    ab_test=test,
    from_email=os.getenv("FROM_EMAIL"),
    track_opens=True,
    track_clicks=True
)

print(f"Campaign launched: {campaign.campaign_id}")
print(f"Scheduled emails: {len(campaign.scheduled_sends)}")
```

## Common Patterns

### Multi-Stage Nurture Campaign

```python
from genpark.sequence_generator import EmailSequenceGenerator
from genpark.models import CampaignGoal, EmailTemplate

generator = EmailSequenceGenerator(api_key=os.getenv("OPENAI_API_KEY"))

# Define multi-stage nurture goal
nurture_goal = CampaignGoal(
    objective="lead_nurture_to_demo",
    target_audience="B2B decision makers",
    desired_action="book_demo",
    tone="consultative",
    sequence_length=7,
    industry="enterprise_software"
)

# Generate sequence
sequence = generator.generate_sequence(
    goal=nurture_goal,
    personalization_fields=["first_name", "company", "industry"],
    include_dynamic_content=True
)

# Add custom logic between emails
for email in sequence.emails:
    if email.position == 3:
        # Add case study after 3rd email
        email.add_attachment("case_study.pdf")
    if email.position == 5:
        # Include social proof
        email.include_testimonials = True

# Save sequence template
sequence.save("templates/enterprise_nurture.json")
```

### A/B Testing Subject Lines

```python
from genpark.ab_testing import ABTestManager
from genpark.optimization import SubjectLineOptimizer

ab_manager = ABTestManager()
optimizer = SubjectLineOptimizer()

# Generate subject line variations
base_subject = "Your exclusive invitation to our webinar"
variations = optimizer.generate_variations(
    base_subject,
    num_variations=4,
    strategies=["curiosity", "urgency", "personalization", "benefit_driven"]
)

# Create multi-variant test
variants = [
    EmailVariant(subject=var, body=email_body, variant_name=f"V{i}")
    for i, var in enumerate(variations)
]

test = ab_manager.create_test(
    test_name="webinar_subject_optimization",
    variants=variants,
    split_ratio=0.25,  # Equal 4-way split
    success_metric="open_rate",
    confidence_level=0.95
)

# Monitor test performance
results = ab_manager.get_test_results(test.test_id)
if results.is_significant:
    winner = results.winning_variant
    print(f"Winner: {winner.subject} (Open rate: {results.winner_metric:.2%})")
```

### Real-Time Performance Tracking

```python
from genpark.tracking import AnalyticsTracker
from genpark.reporting import CampaignReport

tracker = AnalyticsTracker()

# Track email events
campaign_id = "camp_12345"

# Get real-time metrics
metrics = tracker.get_campaign_metrics(campaign_id)
print(f"Sent: {metrics.sent_count}")
print(f"Opens: {metrics.open_count} ({metrics.open_rate:.2%})")
print(f"Clicks: {metrics.click_count} ({metrics.click_rate:.2%})")
print(f"Conversions: {metrics.conversion_count} ({metrics.conversion_rate:.2%})")

# Generate detailed report
report = CampaignReport(campaign_id)
report.add_metrics(metrics)
report.add_ab_test_results(test.test_id)
report.export("reports/campaign_summary.pdf")
```

### Dynamic Content Personalization

```python
from genpark.personalization import ContentPersonalizer
from genpark.models import RecipientProfile

personalizer = ContentPersonalizer()

# Define recipient profiles
recipients = [
    RecipientProfile(
        email="john@company.com",
        first_name="John",
        company="TechCorp",
        industry="fintech",
        engagement_score=85,
        previous_interactions=["clicked_pricing", "viewed_demo"]
    ),
    RecipientProfile(
        email="jane@startup.io",
        first_name="Jane",
        company="StartupIO",
        industry="saas",
        engagement_score=45,
        previous_interactions=["opened_welcome"]
    )
]

# Personalize email content
base_email = sequence.emails[0]

for recipient in recipients:
    personalized = personalizer.personalize(
        email=base_email,
        recipient=recipient,
        dynamic_sections=["intro", "product_highlight", "cta"]
    )
    
    print(f"To: {recipient.email}")
    print(f"Subject: {personalized.subject}")
    print(f"Personalized CTA: {personalized.cta_text}")
```

### Automated Segment-Based Campaigns

```python
from genpark.segmentation import AudienceSegmenter
from genpark.campaign import CampaignExecutor

segmenter = AudienceSegmenter()
executor = CampaignExecutor(sendgrid_api_key=os.getenv("SENDGRID_API_KEY"))

# Load audience
audience = segmenter.load_from_csv("data/contacts.csv")

# Create segments
segments = segmenter.segment_by_criteria(
    audience,
    criteria=[
        {"field": "engagement_score", "operator": ">=", "value": 70},
        {"field": "industry", "operator": "in", "value": ["tech", "saas"]},
        {"field": "last_interaction_days", "operator": "<=", "value": 30}
    ]
)

print(f"High-engagement segment: {len(segments['high_engagement'])} contacts")

# Generate and launch targeted campaigns
for segment_name, contacts in segments.items():
    goal = CampaignGoal(
        objective=f"reactivate_{segment_name}",
        target_audience=segment_name,
        desired_action="product_trial"
    )
    
    sequence = generator.generate_sequence(goal)
    
    campaign = executor.launch_campaign(
        sequence=sequence,
        recipients=[c.email for c in contacts],
        segment_name=segment_name
    )
    
    print(f"Launched campaign for {segment_name}: {campaign.campaign_id}")
```

## Troubleshooting

### API Rate Limits

```python
from genpark.utils import RateLimiter

# Wrap API calls with rate limiting
rate_limiter = RateLimiter(max_calls=100, time_window=60)

with rate_limiter:
    sequence = generator.generate_sequence(goal)
```

### Email Deliverability Issues

```python
from genpark.validation import EmailValidator

validator = EmailValidator()

# Validate email list before sending
valid_emails = []
for email in recipient_list:
    if validator.is_valid(email) and not validator.is_disposable(email):
        valid_emails.append(email)
    else:
        print(f"Skipping invalid email: {email}")

# Use validated list
campaign = executor.launch_campaign(
    sequence=sequence,
    recipients=valid_emails
)
```

### Test Results Not Significant

```python
# Check test status and sample size
test_status = ab_manager.get_test_status(test.test_id)

if not test_status.has_minimum_sample:
    print(f"Need {test_status.required_sample - test_status.current_sample} more samples")
    
if not test_status.is_significant:
    print(f"Current confidence: {test_status.confidence_level:.2%}")
    print("Continue test or increase sample size")
```

### Database Connection Issues

```python
import os
from genpark.database import DatabaseManager

# Custom database path
db_manager = DatabaseManager(db_path=os.getenv("GENPARK_DB_PATH", "./data/campaigns.db"))

# Test connection
if db_manager.test_connection():
    print("Database connected successfully")
else:
    print("Database connection failed - check path and permissions")
```

## Example Usage Script

```python
import os
from genpark.sequence_generator import EmailSequenceGenerator
from genpark.ab_testing import ABTestManager
from genpark.campaign import CampaignExecutor
from genpark.models import CampaignGoal, EmailVariant

def main():
    # Initialize components
    generator = EmailSequenceGenerator(api_key=os.getenv("OPENAI_API_KEY"))
    ab_manager = ABTestManager()
    executor = CampaignExecutor(sendgrid_api_key=os.getenv("SENDGRID_API_KEY"))
    
    # Define goal
    goal = CampaignGoal(
        objective="product_launch",
        target_audience="early_adopters",
        desired_action="purchase",
        sequence_length=5
    )
    
    # Generate sequence
    sequence = generator.generate_sequence(goal)
    
    # Create A/B test for first email
    test = ab_manager.create_test(
        test_name="launch_email_test",
        variants=[
            EmailVariant(subject=sequence.emails[0].subject, body=sequence.emails[0].body, variant_name="A"),
            EmailVariant(subject="Alternative: " + sequence.emails[0].subject, body=sequence.emails[0].body, variant_name="B")
        ],
        split_ratio=0.5
    )
    
    # Launch campaign
    campaign = executor.launch_campaign(
        sequence=sequence,
        recipients=["test@example.com"],
        ab_test=test
    )
    
    print(f"Campaign {campaign.campaign_id} launched successfully!")

if __name__ == "__main__":
    main()
```
