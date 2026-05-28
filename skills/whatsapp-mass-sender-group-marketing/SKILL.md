---
name: whatsapp-mass-sender-group-marketing
description: Social media automation toolkit for TikTok and Instagram lead generation, competitor follower extraction, and automated engagement campaigns
triggers:
  - how do I extract followers from TikTok or Instagram competitors
  - automate Instagram engagement and DM campaigns
  - set up multi-platform social media marketing automation
  - scrape TikTok users by hashtag or keyword
  - build automated follower acquisition for Instagram
  - create auto-comment and auto-like campaigns on social platforms
  - extract and filter social media leads by demographics
  - set up WhatsApp mass messaging campaigns
---

# WhatsApp Mass Sender & Group Marketing System

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

This project is a multi-platform social media automation system designed for lead generation, competitor analysis, and automated engagement on TikTok, Instagram, and WhatsApp. It enables extraction of competitor followers, hashtag-based user discovery, AI-powered demographic filtering, and automated outreach campaigns.

**Primary use cases:**
- Extract followers/engaged users from competitor accounts
- Discover users by hashtag, keyword, or geolocation
- Filter leads by demographics (age, gender, location, activity)
- Automate following, liking, commenting, and direct messaging
- Drive traffic to private domains, independent sites, or WhatsApp groups

**Official resources:**
- Main site: https://www.facebook18.com
- Documentation: https://sites.google.com/view/facebook-script-custom/
- Support: https://sites.google.com/view/instagram-keyword-hashtag-lead/

## Installation

This is a service-based tool rather than a self-hosted package. Integration typically involves:

1. **API Access Setup**
```bash
# Configure environment variables
export MARKETING_API_KEY="your_api_key_here"
export MARKETING_API_ENDPOINT="https://api.facebook18.com/v1"
export PROXY_LIST_PATH="./proxies.txt"
```

2. **Account Configuration**
```javascript
// config/accounts.json
{
  "instagram": [
    {
      "username": "account1",
      "password_env": "IG_ACCOUNT1_PASS",
      "proxy": "http://proxy1:port"
    }
  ],
  "tiktok": [
    {
      "username": "account2",
      "session_token_env": "TT_ACCOUNT2_TOKEN"
    }
  ]
}
```

3. **Dependency Installation** (if using SDK)
```bash
npm install @facebook18/marketing-automation
# or
pip install marketing-automation-sdk
```

## Core Features & API

### 1. Competitor Follower Extraction

Extract all followers from a competitor's account:

```python
from marketing_automation import InstagramScraper
import os

scraper = InstagramScraper(api_key=os.getenv('MARKETING_API_KEY'))

# Extract followers from competitor
followers = scraper.get_followers(
    username='competitor_account',
    max_count=5000,
    filter_options={
        'min_followers': 100,
        'max_followers': 10000,
        'verified_only': False,
        'active_last_days': 30
    }
)

# Save to database or CSV
scraper.export_leads(followers, format='csv', output='leads.csv')
```

### 2. Hashtag & Keyword Discovery

Find users actively engaging with specific topics:

```javascript
const { TikTokLeadGen } = require('@facebook18/marketing-automation');

const tiktok = new TikTokLeadGen({
  apiKey: process.env.MARKETING_API_KEY
});

// Extract users from hashtag
const leads = await tiktok.extractByHashtag({
  hashtag: 'skincare',
  maxUsers: 3000,
  filters: {
    minEngagement: 50,
    location: ['US', 'UK', 'CA'],
    ageRange: [18, 35],
    gender: 'female'
  }
});

// Get users from keyword search
const keywordLeads = await tiktok.extractByKeyword({
  keyword: 'anti-aging cream',
  searchType: 'video_comments', // or 'bio', 'captions'
  maxUsers: 2000
});
```

### 3. AI-Powered Lead Filtering

Filter extracted leads by visual and behavioral signals:

```python
from marketing_automation import LeadFilter

filter = LeadFilter(api_key=os.getenv('MARKETING_API_KEY'))

# Apply AI filters
filtered_leads = filter.apply_filters(
    leads=raw_leads,
    filters={
        'ai_age_detection': {'min': 25, 'max': 45},
        'ai_gender_detection': 'female',
        'profile_completeness': 0.7,  # 70% complete profiles
        'engagement_score': {'min': 0.5},  # Active users
        'spam_detection': True,  # Remove spam accounts
        'location_match': ['United States', 'Canada']
    }
)

print(f"Filtered {len(raw_leads)} down to {len(filtered_leads)} qualified leads")
```

### 4. Automated Engagement Campaigns

Set up auto-follow, auto-like, and auto-comment workflows:

```javascript
const { InstagramBot } = require('@facebook18/marketing-automation');

const bot = new InstagramBot({
  apiKey: process.env.MARKETING_API_KEY,
  account: 'your_account_username',
  sessionToken: process.env.IG_SESSION_TOKEN
});

// Auto-follow campaign
await bot.createCampaign({
  type: 'follow',
  targets: leads.map(l => l.username),
  schedule: {
    dailyLimit: 150,
    hourlyLimit: 20,
    delayBetween: [30, 90] // seconds
  },
  unfollowAfterDays: 3,
  unfollowNonFollowers: true
});

// Auto-like campaign
await bot.createCampaign({
  type: 'like',
  targets: leads,
  postsPerUser: 3,
  schedule: {
    dailyLimit: 300,
    randomize: true
  }
});

// Auto-comment campaign
await bot.createCampaign({
  type: 'comment',
  targets: leads,
  commentTemplates: [
    'Love this! 💕',
    'So inspiring! ✨',
    'Amazing content! 🔥'
  ],
  spintax: true, // Randomize text
  schedule: {
    dailyLimit: 100
  }
});
```

### 5. Direct Message Automation

Send personalized DMs at scale:

```python
from marketing_automation import DMCampaign
import os

campaign = DMCampaign(
    api_key=os.getenv('MARKETING_API_KEY'),
    platform='instagram',
    account='your_account'
)

# Create DM sequence
campaign.create_sequence(
    name='Product Launch Sequence',
    messages=[
        {
            'delay_hours': 0,
            'template': 'Hi {first_name}! 👋 Noticed you love {interest}. Check out our new {product}!',
            'include_media': 'product_image.jpg'
        },
        {
            'delay_hours': 48,
            'template': 'Hey! Did you get a chance to check it out? Here\'s a special 20% off code: WELCOME20',
            'condition': 'no_reply'
        },
        {
            'delay_hours': 120,
            'template': 'Last chance! Code expires tonight 🔥',
            'condition': 'no_reply'
        }
    ]
)

# Start campaign
campaign.start(
    targets=filtered_leads,
    daily_limit=50,
    personalization_fields=['first_name', 'interest', 'product']
)
```

### 6. WhatsApp Mass Messaging

Bulk messaging for WhatsApp groups or contacts:

```javascript
const { WhatsAppSender } = require('@facebook18/marketing-automation');

const whatsapp = new WhatsAppSender({
  apiKey: process.env.MARKETING_API_KEY,
  phoneNumber: process.env.WHATSAPP_NUMBER,
  sessionPath: './whatsapp-session'
});

// Initialize session
await whatsapp.initialize();

// Send bulk messages
await whatsapp.sendBulk({
  recipients: leads.map(l => l.phoneNumber),
  message: 'Hi {name}! Special offer just for you...',
  mediaUrl: 'https://yoursite.com/promo.jpg',
  schedule: {
    batchSize: 30,
    delayBetween: [60, 120], // seconds
    dailyLimit: 200
  }
});

// Group invite automation
await whatsapp.inviteToGroup({
  groupId: 'your_group_id',
  phoneNumbers: leads.map(l => l.phoneNumber),
  inviteMessage: 'Join our exclusive community!',
  dailyLimit: 50
});
```

## Configuration Patterns

### Campaign Configuration File

```yaml
# config/campaign.yml
campaign:
  name: "Competitor Follower Acquisition Q1"
  platforms:
    - instagram
    - tiktok
  
  extraction:
    instagram:
      competitors:
        - username: "competitor1"
          max_followers: 5000
        - username: "competitor2"
          max_followers: 3000
      hashtags:
        - "#skincare"
        - "#antiaging"
      keywords:
        - "best moisturizer"
    
    tiktok:
      creators:
        - "@beauty_guru1"
        - "@skincare_expert"
      hashtags:
        - "#skincareroutine"
  
  filters:
    location: ["US", "UK", "CA", "AU"]
    age_range: [25, 45]
    gender: "female"
    min_engagement_rate: 0.02
    exclude_business_accounts: false
    
  engagement:
    follow:
      enabled: true
      daily_limit: 150
      unfollow_after_days: 3
    
    like:
      enabled: true
      posts_per_user: 2
      daily_limit: 300
    
    comment:
      enabled: true
      daily_limit: 80
      templates:
        - "Love this! 💕"
        - "So helpful! 🙌"
    
    dm:
      enabled: true
      delay_after_follow_hours: 24
      daily_limit: 50
      sequence: "product_launch"
  
  schedule:
    active_hours: [9, 22]  # 9 AM to 10 PM
    timezone: "America/New_York"
    days_active: ["mon", "tue", "wed", "thu", "fri", "sat"]
```

### Load and Execute Campaign

```python
from marketing_automation import CampaignManager
import yaml
import os

# Load configuration
with open('config/campaign.yml') as f:
    config = yaml.safe_load(f)

# Initialize campaign
manager = CampaignManager(api_key=os.getenv('MARKETING_API_KEY'))
campaign = manager.create_from_config(config)

# Start campaign
campaign.start()

# Monitor progress
stats = campaign.get_stats()
print(f"Leads extracted: {stats['leads_extracted']}")
print(f"Follows sent: {stats['follows_sent']}")
print(f"DMs sent: {stats['dms_sent']}")
print(f"Conversion rate: {stats['conversion_rate']}")

# Pause/resume
campaign.pause()
campaign.resume()

# Export results
campaign.export_results('campaign_results.csv')
```

## Advanced Patterns

### Multi-Account Rotation

Distribute actions across multiple accounts to avoid rate limits:

```javascript
const { AccountRotator } = require('@facebook18/marketing-automation');

const rotator = new AccountRotator({
  apiKey: process.env.MARKETING_API_KEY,
  accounts: [
    { username: 'account1', sessionToken: process.env.ACCOUNT1_TOKEN },
    { username: 'account2', sessionToken: process.env.ACCOUNT2_TOKEN },
    { username: 'account3', sessionToken: process.env.ACCOUNT3_TOKEN }
  ],
  strategy: 'round_robin' // or 'random', 'least_used'
});

// Actions automatically rotate across accounts
await rotator.followUsers(leads, {
  dailyLimitPerAccount: 100,
  smartRotation: true // Switch on rate limit warnings
});
```

### Proxy Management

Route requests through rotating proxies:

```python
from marketing_automation import ProxyManager

proxy_mgr = ProxyManager()
proxy_mgr.load_from_file('proxies.txt')  # Format: ip:port:user:pass

# Assign proxies to accounts
scraper = InstagramScraper(
    api_key=os.getenv('MARKETING_API_KEY'),
    proxy_manager=proxy_mgr,
    proxy_rotation='per_request'  # or 'per_session', 'per_account'
)

# Test proxies
working_proxies = proxy_mgr.test_all()
print(f"{len(working_proxies)} working proxies")
```

### Webhook Notifications

Get real-time updates on campaign events:

```javascript
const { WebhookServer } = require('@facebook18/marketing-automation');

const webhook = new WebhookServer({
  port: 3000,
  secret: process.env.WEBHOOK_SECRET
});

webhook.on('lead_extracted', (data) => {
  console.log(`New lead: ${data.username} from ${data.source}`);
  // Send to CRM, database, etc.
});

webhook.on('dm_sent', (data) => {
  console.log(`DM sent to ${data.recipient}`);
});

webhook.on('rate_limit', (data) => {
  console.warn(`Rate limit hit on ${data.account} - pausing for ${data.cooldown_minutes} minutes`);
});

webhook.on('campaign_complete', (data) => {
  console.log(`Campaign ${data.campaign_id} completed: ${data.total_leads} leads generated`);
});

webhook.start();
```

## Troubleshooting

### Rate Limit Issues

```python
# Handle rate limits gracefully
from marketing_automation.exceptions import RateLimitError
import time

try:
    followers = scraper.get_followers(username='competitor', max_count=10000)
except RateLimitError as e:
    print(f"Rate limited. Retry after {e.retry_after} seconds")
    time.sleep(e.retry_after)
    followers = scraper.get_followers(username='competitor', max_count=10000)
```

### Session Expiration

```javascript
// Auto-refresh sessions
bot.on('session_expired', async (account) => {
  console.log(`Session expired for ${account.username}, refreshing...`);
  await bot.refreshSession(account);
});

// Manual session refresh
try {
  await bot.followUser('target_user');
} catch (error) {
  if (error.code === 'SESSION_EXPIRED') {
    await bot.login(account.username, process.env.ACCOUNT_PASSWORD);
    await bot.followUser('target_user');
  }
}
```

### Account Shadowban Detection

```python
from marketing_automation import HealthChecker

checker = HealthChecker(api_key=os.getenv('MARKETING_API_KEY'))

# Check account health
health = checker.check_account(platform='instagram', username='your_account')

if health['shadowbanned']:
    print("Account is shadowbanned - reduce activity")
    campaign.set_limits(daily_follows=50, daily_likes=100)
elif health['action_blocked']:
    print("Temporary action block - pause for 24h")
    campaign.pause(hours=24)
else:
    print("Account healthy")
```

### Data Quality Issues

```python
# Remove duplicates and invalid leads
from marketing_automation import LeadCleaner

cleaner = LeadCleaner()
leads = cleaner.clean(
    raw_leads,
    remove_duplicates=True,
    validate_usernames=True,
    remove_inactive=True,  # No posts in 90 days
    remove_private=True    # Private accounts
)

print(f"Cleaned {len(raw_leads)} down to {len(leads)} quality leads")
```

## Best Practices

1. **Start slow**: Begin with low daily limits (50-100 actions/day) and gradually increase
2. **Rotate accounts**: Use 3-5 accounts per campaign to distribute activity
3. **Humanize timing**: Add random delays (30-90s) between actions
4. **Monitor health**: Check account status daily for shadowbans or blocks
5. **Segment leads**: Filter leads into tiers (hot/warm/cold) for targeted messaging
6. **Test messages**: A/B test DM templates before scaling
7. **Comply with ToS**: Review platform policies and use responsibly
8. **Use proxies**: Residential proxies for accounts, datacenter for scraping

## Environment Variables Reference

```bash
# API Authentication
MARKETING_API_KEY=your_api_key
MARKETING_API_ENDPOINT=https://api.facebook18.com/v1

# Account Credentials (store securely)
IG_ACCOUNT1_USER=username1
IG_ACCOUNT1_PASS=password1
TT_ACCOUNT1_TOKEN=session_token1

# WhatsApp
WHATSAPP_NUMBER=+1234567890
WHATSAPP_SESSION_PATH=./sessions/whatsapp

# Proxy Configuration
PROXY_LIST_PATH=./proxies.txt
PROXY_ROTATION=per_request

# Webhook
WEBHOOK_URL=https://yoursite.com/webhook
WEBHOOK_SECRET=your_webhook_secret

# Limits (optional overrides)
DAILY_FOLLOW_LIMIT=150
DAILY_DM_LIMIT=50
HOURLY_ACTION_LIMIT=30
```

This skill covers the core automation capabilities for social media marketing campaigns across TikTok, Instagram, and WhatsApp platforms.
