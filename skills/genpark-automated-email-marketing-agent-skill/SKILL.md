---
name: genpark-automated-email-marketing-agent-skill
description: AI-powered email sequence generator with A/B testing capabilities for automated marketing campaigns
triggers:
  - generate email marketing sequences
  - create A/B test email campaigns
  - automate email marketing workflows
  - build goal-driven email sequences
  - set up email campaign testing
  - optimize email marketing with AI
  - create personalized email drip campaigns
  - generate marketing email variations
---

# genpark-automated-email-marketing-agent-skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

GenPark Automated Email Marketing Agent is a goal-driven email sequence generator that leverages AI to create, optimize, and A/B test marketing email campaigns. It automates the creation of personalized email sequences based on campaign goals, audience segments, and conversion objectives.

## Installation

```bash
# Clone the repository
git clone https://github.com/alphaparkinc/genpark-automated-email-marketing-agent-skill.git
cd genpark-automated-email-marketing-agent-skill

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

Typical requirements include:
```
openai>=1.0.0
python-dotenv>=1.0.0
pydantic>=2.0.0
requests>=2.31.0
```

## Configuration

Set up environment variables in a `.env` file:

```bash
# API Keys (use your actual keys)
OPENAI_API_KEY=your_openai_api_key
GENPARK_API_KEY=your_genpark_api_key

# Email Service Provider (optional)
SENDGRID_API_KEY=your_sendgrid_key
MAILGUN_API_KEY=your_mailgun_key

# Campaign Settings
DEFAULT_SEQUENCE_LENGTH=5
AB_TEST_SPLIT_RATIO=0.5
```

## Core Concepts

### Email Sequence Generation

Generate multi-step email campaigns driven by specific goals (e.g., product launch, onboarding, re-engagement).

### A/B Testing

Automatically create variations of email content, subject lines, and CTAs to optimize performance.

### Goal-Driven Optimization

AI analyzes campaign objectives and tailors messaging, timing, and content accordingly.

## Usage Examples

### Basic Email Sequence Generation

```python
from genpark import EmailSequenceAgent, CampaignGoal

# Initialize the agent
agent = EmailSequenceAgent(
    api_key=os.getenv("GENPARK_API_KEY"),
    model="gpt-4"
)

# Define campaign goal
goal = CampaignGoal(
    objective="product_launch",
    target_audience="B2B SaaS managers",
    conversion_goal="demo_signup",
    sequence_length=5
)

# Generate email sequence
sequence = agent.generate_sequence(
    goal=goal,
    brand_voice="professional yet approachable",
    product_description="AI-powered analytics platform"
)

# Output emails
for idx, email in enumerate(sequence.emails, 1):
    print(f"\n--- Email {idx} ---")
    print(f"Subject: {email.subject}")
    print(f"Send Delay: {email.send_delay_days} days")
    print(f"Content:\n{email.body}")
```

### A/B Testing Email Variations

```python
from genpark import ABTestGenerator, TestConfig

# Initialize A/B test generator
ab_generator = ABTestGenerator(api_key=os.getenv("GENPARK_API_KEY"))

# Configure test
test_config = TestConfig(
    test_elements=["subject_line", "cta_button", "opening_line"],
    num_variations=3,
    split_ratio=0.33
)

# Generate variations
email_base = {
    "subject": "Unlock Your Team's Potential",
    "body": "Dear {first_name},\n\nDiscover how our platform can transform your workflow...",
    "cta": "Start Free Trial"
}

variations = ab_generator.create_variations(
    base_email=email_base,
    config=test_config,
    goal="maximize_click_through"
)

# Review variations
for variant_id, variant in variations.items():
    print(f"\n{variant_id}:")
    print(f"Subject: {variant['subject']}")
    print(f"CTA: {variant['cta']}")
    print(f"Hypothesis: {variant['test_hypothesis']}")
```

### Personalized Drip Campaign

```python
from genpark import DripCampaignBuilder, Segment

# Define audience segment
segment = Segment(
    name="free_trial_users",
    characteristics={
        "signup_date": "within_7_days",
        "engagement_level": "low",
        "product_usage": "minimal"
    }
)

# Build drip campaign
builder = DripCampaignBuilder(api_key=os.getenv("GENPARK_API_KEY"))

campaign = builder.create_campaign(
    segment=segment,
    goal="convert_to_paid",
    personalization_fields=["first_name", "company_name", "signup_date"],
    sequence_length=7
)

# Schedule campaign
campaign.schedule(
    start_date="2026-08-01",
    time_zone="America/New_York",
    send_time="09:00"
)

print(f"Campaign '{campaign.name}' created with {len(campaign.emails)} emails")
```

### Real-Time Performance Optimization

```python
from genpark import CampaignMonitor, OptimizationStrategy

# Monitor active campaign
monitor = CampaignMonitor(
    campaign_id="camp_abc123",
    api_key=os.getenv("GENPARK_API_KEY")
)

# Get performance metrics
metrics = monitor.get_metrics(time_period="last_7_days")
print(f"Open Rate: {metrics.open_rate}%")
print(f"Click Rate: {metrics.click_rate}%")
print(f"Conversion Rate: {metrics.conversion_rate}%")

# Auto-optimize underperforming emails
if metrics.open_rate < 20:
    strategy = OptimizationStrategy(
        focus="subject_line",
        approach="urgency_and_curiosity"
    )
    
    optimized = monitor.optimize_campaign(strategy=strategy)
    print(f"Generated {len(optimized.new_variations)} new subject line variations")
```

### CLI Usage (if available)

```bash
# Generate email sequence
python example_usage.py --goal product_launch --audience "startup founders" --length 5

# Create A/B test
python cli.py ab-test \
  --base-email templates/welcome.json \
  --test-elements subject,cta \
  --variations 3

# Analyze campaign performance
python cli.py analyze --campaign-id camp_123 --report-type detailed
```

## Common Patterns

### Pattern: Onboarding Sequence

```python
from genpark import EmailSequenceAgent, CampaignGoal

agent = EmailSequenceAgent(api_key=os.getenv("GENPARK_API_KEY"))

onboarding = agent.generate_sequence(
    goal=CampaignGoal(
        objective="user_onboarding",
        target_audience="new signups",
        conversion_goal="feature_activation",
        sequence_length=5
    ),
    timing_strategy="progressive_nurture",  # Days 0, 2, 5, 10, 15
    content_themes=["welcome", "quick_win", "features", "best_practices", "success_story"]
)
```

### Pattern: Re-engagement Campaign

```python
from genpark import EmailSequenceAgent, CampaignGoal

reengagement = agent.generate_sequence(
    goal=CampaignGoal(
        objective="winback",
        target_audience="inactive_users_90days",
        conversion_goal="return_visit",
        sequence_length=3
    ),
    brand_voice="empathetic and value-focused",
    special_offers=["exclusive_feature", "discount_code"]
)
```

### Pattern: Multi-Variant Testing

```python
from genpark import ABTestGenerator

ab_gen = ABTestGenerator(api_key=os.getenv("GENPARK_API_KEY"))

# Test multiple elements simultaneously
multi_variant = ab_gen.create_multivariate_test(
    base_email=email_template,
    test_matrix={
        "subject_line": ["question", "benefit", "urgency"],
        "cta_position": ["top", "middle", "bottom"],
        "image_style": ["screenshot", "illustration", "none"]
    },
    sample_size=10000
)

print(f"Created {multi_variant.total_combinations} test combinations")
```

## Integration with Email Service Providers

### SendGrid Integration

```python
from genpark import EmailSequenceAgent
from genpark.integrations import SendGridConnector

# Generate sequence
sequence = agent.generate_sequence(goal=campaign_goal)

# Connect to SendGrid
sendgrid = SendGridConnector(api_key=os.getenv("SENDGRID_API_KEY"))

# Deploy campaign
sendgrid.deploy_sequence(
    sequence=sequence,
    from_email="marketing@yourcompany.com",
    reply_to="support@yourcompany.com",
    list_id="your_sendgrid_list_id"
)
```

### Mailgun Integration

```python
from genpark.integrations import MailgunConnector

mailgun = MailgunConnector(
    api_key=os.getenv("MAILGUN_API_KEY"),
    domain="mg.yourcompany.com"
)

mailgun.deploy_sequence(sequence=sequence, segment="trial_users")
```

## Troubleshooting

### Issue: API Rate Limits

```python
from genpark import EmailSequenceAgent
from genpark.utils import RateLimiter

agent = EmailSequenceAgent(
    api_key=os.getenv("GENPARK_API_KEY"),
    rate_limiter=RateLimiter(max_requests=10, time_window=60)
)
```

### Issue: Low Quality Email Generation

```python
# Provide more context and constraints
sequence = agent.generate_sequence(
    goal=goal,
    brand_voice="detailed brand voice description here",
    example_emails=["path/to/example1.txt", "path/to/example2.txt"],
    tone_constraints={"formality": "medium", "humor": "minimal"},
    word_count_range=(150, 300)
)
```

### Issue: A/B Test Not Converging

```python
# Increase sample size and test duration
test_config = TestConfig(
    test_elements=["subject_line"],
    num_variations=2,  # Start with fewer variations
    min_sample_size=1000,
    confidence_level=0.95,
    min_test_duration_hours=48
)
```

## Advanced Features

### Dynamic Content Personalization

```python
from genpark import PersonalizationEngine

personalizer = PersonalizationEngine(api_key=os.getenv("GENPARK_API_KEY"))

personalized = personalizer.apply_dynamic_content(
    email_template=email,
    user_data={
        "name": "{first_name}",
        "company": "{company_name}",
        "last_activity": "{last_login_date}",
        "recommended_feature": "{ai_recommended_feature}"
    }
)
```

### Predictive Send Time Optimization

```python
from genpark import SendTimeOptimizer

optimizer = SendTimeOptimizer(api_key=os.getenv("GENPARK_API_KEY"))

best_times = optimizer.predict_optimal_send_times(
    segment=segment,
    historical_data=campaign_history,
    timezone_aware=True
)

print(f"Optimal send time: {best_times.recommended_time}")
```

## Best Practices

1. **Always A/B test** subject lines and CTAs before full deployment
2. **Segment audiences** carefully for personalized messaging
3. **Monitor metrics** continuously and iterate based on performance
4. **Use environment variables** for all API keys and sensitive configuration
5. **Test sequences** with small sample sizes before scaling
6. **Maintain brand consistency** across all generated emails

## Resources

- Homepage: https://genpark.ai
- Repository: https://github.com/alphaparkinc/genpark-automated-email-marketing-agent-skill
- Documentation: Check repository for additional docs and examples
