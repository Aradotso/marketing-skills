---
name: whatsapp-mass-sender-group-marketing
description: Automate WhatsApp mass messaging, group marketing, and lead generation for TikTok/Instagram competitive audience extraction
triggers:
  - how do I set up WhatsApp mass sender for marketing
  - automate Instagram follower extraction and messaging
  - extract TikTok competitor followers for outreach
  - send bulk WhatsApp messages to target audience
  - scrape Instagram users by hashtag for marketing
  - automate social media lead generation workflow
  - build WhatsApp marketing automation system
  - extract and message competitor audiences on social platforms
---

# WhatsApp Mass Sender and Group Marketing System

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides automated tools for social media audience extraction (TikTok, Instagram) and WhatsApp mass messaging capabilities. It enables competitive intelligence gathering, precise audience targeting, and automated outreach campaigns across multiple platforms.

## What This Project Does

- **Competitive Audience Extraction**: Scrape followers, likes, and commenters from competitor accounts on TikTok and Instagram
- **Hashtag & Keyword Targeting**: Extract users posting or engaging with specific hashtags/keywords
- **AI-Powered Filtering**: Filter users by demographics, activity level, profile characteristics
- **Multi-Platform Automation**: Auto-follow, auto-like, auto-comment, and auto-DM capabilities
- **WhatsApp Mass Messaging**: Bulk message sending to extracted phone numbers/contacts
- **Group Marketing**: Automated WhatsApp group management and messaging

## Installation

This is a commercial service/system. Based on the project description, access requires:

```bash
# Contact the service provider
# Official site: https://www.facebook18.com
# Technical support: https://sites.google.com/view/facebook-script-custom/
# Documentation: https://sites.google.com/view/instagram-keyword-hashtag-lead/
```

For self-hosted implementations, typical setup would involve:

```bash
# Clone the repository
git clone https://github.com/jdodof/WhatsApp-Mass-Sender-And-Group-Marketing-System.git
cd WhatsApp-Mass-Sender-And-Group-Marketing-System

# Install dependencies (example structure)
npm install
# or
pip install -r requirements.txt
```

## Configuration

Set up environment variables for platform credentials and API access:

```bash
# .env file structure
INSTAGRAM_USERNAME=${INSTAGRAM_USERNAME}
INSTAGRAM_PASSWORD=${INSTAGRAM_PASSWORD}
TIKTOK_SESSION_ID=${TIKTOK_SESSION_ID}
WHATSAPP_API_KEY=${WHATSAPP_API_KEY}
WHATSAPP_PHONE_NUMBER=${WHATSAPP_PHONE_NUMBER}

# Rate limiting and safety
MAX_FOLLOWERS_PER_DAY=500
MAX_MESSAGES_PER_HOUR=30
AUTO_DELAY_MIN=10
AUTO_DELAY_MAX=30

# Filtering criteria
TARGET_MIN_FOLLOWERS=100
TARGET_MAX_FOLLOWERS=50000
TARGET_REGIONS=US,UK,CA,AU
TARGET_LANGUAGES=en
```

## Core Functionality Examples

### Extract Instagram Competitor Followers

```javascript
// Instagram follower extraction
const InstagramScraper = require('./lib/instagram-scraper');

const scraper = new InstagramScraper({
  username: process.env.INSTAGRAM_USERNAME,
  password: process.env.INSTAGRAM_PASSWORD
});

async function extractCompetitorFollowers() {
  await scraper.login();
  
  const targetAccount = 'competitor_username';
  const followers = await scraper.getFollowers(targetAccount, {
    maxCount: 5000,
    filters: {
      minFollowers: 100,
      maxFollowers: 50000,
      regions: ['US', 'UK', 'CA'],
      activeInDays: 30
    }
  });
  
  console.log(`Extracted ${followers.length} followers`);
  
  // Export to CSV
  await scraper.exportToCSV(followers, 'competitor_followers.csv');
  
  return followers;
}
```

### Extract Users by Hashtag

```python
# TikTok hashtag user extraction
from tiktok_scraper import TikTokAPI
import os

api = TikTokAPI(session_id=os.getenv('TIKTOK_SESSION_ID'))

def extract_hashtag_users(hashtag, max_users=1000):
    """Extract users engaging with specific hashtag"""
    
    users = api.get_hashtag_posts(
        hashtag=hashtag,
        count=500
    )
    
    engaged_users = []
    
    for post in users:
        # Get users who liked the post
        likers = api.get_post_likers(post['id'], limit=100)
        
        # Get commenters
        commenters = api.get_post_commenters(post['id'], limit=50)
        
        engaged_users.extend(likers)
        engaged_users.extend(commenters)
    
    # Deduplicate
    unique_users = {user['username']: user for user in engaged_users}
    
    # Apply filters
    filtered_users = [
        user for user in unique_users.values()
        if user['followers'] >= 100 and user['followers'] <= 50000
    ]
    
    return filtered_users[:max_users]

# Usage
users = extract_hashtag_users('fitness', max_users=2000)
print(f"Extracted {len(users)} engaged users")
```

### Automated Follow and Engagement

```javascript
// Auto-follow and engage workflow
const SocialAutomation = require('./lib/automation');

const automation = new SocialAutomation({
  platform: 'instagram',
  credentials: {
    username: process.env.INSTAGRAM_USERNAME,
    password: process.env.INSTAGRAM_PASSWORD
  }
});

async function automatedEngagement(userList) {
  await automation.login();
  
  for (const user of userList) {
    try {
      // Follow user
      await automation.follow(user.username);
      console.log(`Followed: ${user.username}`);
      
      // Like recent posts
      const posts = await automation.getUserPosts(user.username, 3);
      for (const post of posts) {
        await automation.like(post.id);
        await automation.delay(5, 10); // Random delay 5-10 seconds
      }
      
      // Leave comment
      const comments = [
        "Great content! 🔥",
        "Love this! 👍",
        "Amazing work! ✨"
      ];
      const randomComment = comments[Math.floor(Math.random() * comments.length)];
      await automation.comment(posts[0].id, randomComment);
      
      // Send DM (optional)
      if (user.followers > 500) {
        await automation.sendDM(user.username, 
          `Hey! Love your content. Check out my page for similar vibes! 🚀`
        );
      }
      
      // Rate limiting delay
      await automation.delay(30, 60);
      
    } catch (error) {
      console.error(`Error engaging with ${user.username}:`, error.message);
      continue;
    }
  }
}
```

### WhatsApp Mass Messaging

```python
# WhatsApp bulk messaging
from whatsapp_api import WhatsAppClient
import pandas as pd
import os
import time

client = WhatsAppClient(
    api_key=os.getenv('WHATSAPP_API_KEY'),
    phone_number=os.getenv('WHATSAPP_PHONE_NUMBER')
)

def send_bulk_messages(contacts_csv, message_template):
    """Send personalized messages to contact list"""
    
    # Load contacts
    contacts = pd.read_csv(contacts_csv)
    
    successful = 0
    failed = 0
    
    for index, contact in contacts.iterrows():
        try:
            # Personalize message
            message = message_template.format(
                name=contact.get('name', 'there'),
                custom_field=contact.get('custom_field', '')
            )
            
            # Send message
            response = client.send_message(
                to=contact['phone'],
                message=message
            )
            
            if response['status'] == 'sent':
                successful += 1
                print(f"✓ Sent to {contact['phone']}")
            else:
                failed += 1
                print(f"✗ Failed to send to {contact['phone']}")
            
            # Rate limiting (30 messages per hour = 2 minutes delay)
            time.sleep(120)
            
        except Exception as e:
            failed += 1
            print(f"Error sending to {contact['phone']}: {e}")
    
    return {
        'successful': successful,
        'failed': failed,
        'total': len(contacts)
    }

# Usage
message = """
Hi {name}! 👋

We noticed you're interested in [PRODUCT CATEGORY]. 

We're offering exclusive deals this week - check it out: [LINK]

Reply STOP to unsubscribe.
"""

results = send_bulk_messages('extracted_contacts.csv', message)
print(f"Campaign complete: {results['successful']} sent, {results['failed']} failed")
```

### AI-Powered User Filtering

```javascript
// Filter users with AI criteria
const UserFilter = require('./lib/filters');

async function filterUsers(rawUsers) {
  const filter = new UserFilter({
    minFollowers: 100,
    maxFollowers: 50000,
    minEngagementRate: 2.0,
    targetRegions: ['US', 'UK', 'CA', 'AU'],
    targetLanguages: ['en'],
    excludeBusinessAccounts: false,
    requireProfilePicture: true,
    activeInDays: 30
  });
  
  // Apply basic filters
  let filtered = filter.applyBasicFilters(rawUsers);
  
  // AI-based filtering (age, gender estimation from profile)
  if (process.env.AI_VISION_API_KEY) {
    filtered = await filter.applyAIFilters(filtered, {
      estimateAge: true,
      estimateGender: true,
      detectBots: true,
      targetAgeRange: [18, 45],
      targetGender: 'any' // 'male', 'female', 'any'
    });
  }
  
  // Engagement quality check
  filtered = await filter.checkEngagementQuality(filtered, {
    minCommentLength: 10,
    excludeGenericComments: true,
    requireRecentActivity: true
  });
  
  return filtered;
}
```

## Common Workflow Patterns

### End-to-End Campaign

```python
# Complete marketing campaign workflow
import json
from campaign_manager import CampaignManager

campaign = CampaignManager(config_file='config.json')

# Step 1: Extract target audience
competitors = ['competitor1', 'competitor2', 'competitor3']
hashtags = ['#fitness', '#workout', '#health']

audience = campaign.extract_audience(
    instagram_competitors=competitors,
    tiktok_hashtags=hashtags,
    max_users=10000
)

# Step 2: Filter and clean
filtered_audience = campaign.filter_audience(
    audience,
    min_followers=100,
    max_followers=50000,
    regions=['US', 'UK'],
    active_days=30
)

# Step 3: Engage on social media
campaign.auto_engage(
    users=filtered_audience[:1000],
    actions=['follow', 'like', 'comment'],
    daily_limit=500
)

# Step 4: Extract contact info (if available)
contacts = campaign.extract_contact_info(filtered_audience)

# Step 5: WhatsApp outreach
whatsapp_contacts = [u for u in contacts if 'phone' in u]
campaign.send_whatsapp_campaign(
    contacts=whatsapp_contacts,
    message_template='templates/intro_message.txt',
    schedule='spread_24h'
)

# Step 6: Track results
results = campaign.get_campaign_stats()
print(json.dumps(results, indent=2))
```

### Scheduled Automation

```javascript
// Schedule recurring campaigns
const cron = require('node-cron');
const CampaignRunner = require('./lib/campaign-runner');

const runner = new CampaignRunner({
  configPath: './campaigns/config.json'
});

// Daily follower extraction at 2 AM
cron.schedule('0 2 * * *', async () => {
  console.log('Running daily follower extraction...');
  await runner.extractFollowers({
    platform: 'instagram',
    targets: ['competitor1', 'competitor2'],
    maxPerTarget: 500
  });
});

// Engagement automation every 4 hours
cron.schedule('0 */4 * * *', async () => {
  console.log('Running engagement automation...');
  await runner.autoEngage({
    platform: 'instagram',
    actionsPerCycle: 50,
    actions: ['follow', 'like']
  });
});

// Weekly WhatsApp campaign on Mondays at 10 AM
cron.schedule('0 10 * * 1', async () => {
  console.log('Running weekly WhatsApp campaign...');
  await runner.whatsappCampaign({
    templateId: 'weekly_promo',
    audienceSegment: 'engaged_users'
  });
});
```

## Troubleshooting

### Rate Limiting Issues

```javascript
// Handle platform rate limits gracefully
const RateLimiter = require('./lib/rate-limiter');

const limiter = new RateLimiter({
  maxActionsPerHour: 30,
  maxActionsPerDay: 500,
  cooldownOnLimit: 3600000 // 1 hour in ms
});

async function safeAction(actionFn, actionName) {
  try {
    if (!limiter.canPerformAction(actionName)) {
      const waitTime = limiter.getWaitTime(actionName);
      console.log(`Rate limit reached. Waiting ${waitTime}ms...`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    const result = await actionFn();
    limiter.recordAction(actionName);
    return result;
    
  } catch (error) {
    if (error.message.includes('rate limit') || error.status === 429) {
      console.log('Platform rate limit detected. Sleeping for 1 hour...');
      await new Promise(resolve => setTimeout(resolve, 3600000));
      return safeAction(actionFn, actionName);
    }
    throw error;
  }
}
```

### Account Security

```python
# Implement security best practices
from security_manager import SecurityManager
import random

security = SecurityManager()

# Rotate proxies
proxy = security.get_random_proxy()

# Use realistic delays
def human_delay(min_sec=5, max_sec=30):
    delay = random.uniform(min_sec, max_sec)
    time.sleep(delay)

# Vary user agent
user_agent = security.get_random_user_agent()

# Session management
session = security.create_session(
    proxy=proxy,
    user_agent=user_agent,
    cookies_file='sessions/instagram_session.json'
)

# Activity patterns
security.simulate_human_behavior(
    login_time=random.choice(['morning', 'afternoon', 'evening']),
    session_duration_minutes=random.randint(10, 45),
    actions_per_session=random.randint(20, 50)
)
```

### Data Export and Integration

```javascript
// Export extracted data in multiple formats
const DataExporter = require('./lib/exporter');

const exporter = new DataExporter();

// Export to CSV
await exporter.toCSV(users, 'exports/users.csv', {
  columns: ['username', 'followers', 'engagement_rate', 'phone', 'email']
});

// Export to Google Sheets
await exporter.toGoogleSheets(users, {
  spreadsheetId: process.env.GOOGLE_SHEET_ID,
  sheetName: 'Extracted Users',
  credentials: process.env.GOOGLE_CREDENTIALS_PATH
});

// Export to CRM (HubSpot example)
await exporter.toCRM(users, {
  platform: 'hubspot',
  apiKey: process.env.HUBSPOT_API_KEY,
  listId: 'target_audience_list'
});

// Export to email marketing platform
await exporter.toEmailPlatform(users, {
  platform: 'mailchimp',
  apiKey: process.env.MAILCHIMP_API_KEY,
  listId: process.env.MAILCHIMP_LIST_ID
});
```

## Best Practices

1. **Respect Rate Limits**: Always implement delays and respect platform limits to avoid account bans
2. **Use Proxies**: Rotate IP addresses, especially for high-volume operations
3. **Comply with Terms**: Review platform ToS and messaging regulations (GDPR, CAN-SPAM, etc.)
4. **Provide Opt-Out**: Always include unsubscribe options in bulk messages
5. **Monitor Performance**: Track metrics like response rate, conversion, and account health
6. **Segment Audiences**: Don't blast everyone - segment and personalize for better results
7. **Test Small First**: Run small test campaigns before scaling up
8. **Backup Sessions**: Save authentication sessions to avoid repeated logins
