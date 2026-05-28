---
name: whatsapp-mass-sender-group-marketing
description: Multi-platform social media automation for TikTok, Instagram, and WhatsApp mass messaging and follower extraction
triggers:
  - how do I automate WhatsApp mass messaging
  - extract followers from TikTok competitor accounts
  - automate Instagram follower engagement
  - set up multi-platform social media marketing automation
  - bulk send messages on WhatsApp to groups
  - scrape TikTok user data by hashtag
  - automate Instagram comments and DMs
  - build social media lead generation funnel
---

# WhatsApp Mass Sender & Group Marketing System

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides automation tools for multi-platform social media marketing, primarily focused on TikTok, Instagram, and WhatsApp. It enables bulk follower extraction, automated engagement (follows, likes, comments), and mass messaging capabilities for lead generation and customer acquisition.

## What This Project Does

The system offers several core capabilities:

- **Competitor Follower Extraction**: Scrape follower lists and engaged users from competitor accounts on TikTok and Instagram
- **Hashtag & Keyword Targeting**: Extract users posting or engaging with specific hashtags and keywords
- **Automated Engagement**: Batch follow, like, comment, and DM operations
- **WhatsApp Mass Messaging**: Send bulk messages to groups and individual contacts
- **User Filtering**: AI-based filtering by demographics, activity level, and profile characteristics
- **Multi-Account Management**: Operate multiple social media accounts simultaneously

## Installation

Based on the project description, this appears to be a commercial service rather than an open-source tool you install locally. Access is provided through:

1. **Web Platform**: Visit the official site at `https://sites.google.com/view/facebook-script-custom/`
2. **Support/Setup**: Contact through `https://sites.google.com/view/instagram-keyword-hashtag-lead/`

For self-hosted or custom implementations, you would typically need:

```bash
# Clone the repository
git clone https://github.com/jdodof/WhatsApp-Mass-Sender-And-Group-Marketing-System.git
cd WhatsApp-Mass-Sender-And-Group-Marketing-System

# Install dependencies (assuming Node.js/Python based on typical automation tools)
npm install
# or
pip install -r requirements.txt
```

## Environment Configuration

Create a `.env` file with required credentials:

```env
# Instagram Credentials
INSTAGRAM_USERNAME=your_username
INSTAGRAM_PASSWORD=your_password

# TikTok Credentials
TIKTOK_USERNAME=your_username
TIKTOK_PASSWORD=your_password

# WhatsApp Configuration
WHATSAPP_PHONE_NUMBER=+1234567890
WHATSAPP_API_KEY=your_api_key

# Proxy Settings (recommended to avoid rate limits)
PROXY_HOST=proxy.example.com
PROXY_PORT=8080
PROXY_USERNAME=proxy_user
PROXY_PASSWORD=proxy_pass

# Rate Limiting
MAX_FOLLOWS_PER_HOUR=50
MAX_MESSAGES_PER_HOUR=30
MAX_COMMENTS_PER_HOUR=40

# Target Filters
TARGET_MIN_FOLLOWERS=100
TARGET_MAX_FOLLOWERS=50000
TARGET_COUNTRIES=US,UK,CA,AU
```

## Key Usage Patterns

### 1. Extract Competitor Followers (Instagram)

```javascript
const { InstagramScraper } = require('./lib/instagram-scraper');

async function extractCompetitorFollowers() {
  const scraper = new InstagramScraper({
    username: process.env.INSTAGRAM_USERNAME,
    password: process.env.INSTAGRAM_PASSWORD,
    proxy: {
      host: process.env.PROXY_HOST,
      port: process.env.PROXY_PORT
    }
  });

  await scraper.login();

  // Extract followers from competitor account
  const followers = await scraper.getFollowers('competitor_account', {
    maxCount: 5000,
    filters: {
      minFollowers: parseInt(process.env.TARGET_MIN_FOLLOWERS),
      maxFollowers: parseInt(process.env.TARGET_MAX_FOLLOWERS),
      activeOnly: true,
      countries: process.env.TARGET_COUNTRIES.split(',')
    }
  });

  // Save to database or file
  await scraper.saveToFile(followers, 'competitor_followers.json');
  
  console.log(`Extracted ${followers.length} followers`);
  return followers;
}

extractCompetitorFollowers().catch(console.error);
```

### 2. TikTok Hashtag User Extraction

```python
from lib.tiktok_scraper import TikTokScraper
import os
import json

def extract_hashtag_users():
    scraper = TikTokScraper(
        username=os.getenv('TIKTOK_USERNAME'),
        password=os.getenv('TIKTOK_PASSWORD'),
        proxy={
            'host': os.getenv('PROXY_HOST'),
            'port': os.getenv('PROXY_PORT')
        }
    )
    
    scraper.login()
    
    # Extract users who posted with specific hashtag
    users = scraper.get_hashtag_users(
        hashtag='fitness',
        max_videos=1000,
        filters={
            'min_followers': int(os.getenv('TARGET_MIN_FOLLOWERS')),
            'engagement_rate_min': 0.05,
            'posted_within_days': 30
        }
    )
    
    # Save results
    with open('hashtag_users.json', 'w') as f:
        json.dump(users, f, indent=2)
    
    print(f"Extracted {len(users)} users from #fitness")
    return users

if __name__ == '__main__':
    extract_hashtag_users()
```

### 3. Automated Engagement Campaign

```javascript
const { InstagramBot } = require('./lib/instagram-bot');

async function runEngagementCampaign() {
  const bot = new InstagramBot({
    username: process.env.INSTAGRAM_USERNAME,
    password: process.env.INSTAGRAM_PASSWORD,
    rateLimits: {
      followsPerHour: parseInt(process.env.MAX_FOLLOWS_PER_HOUR),
      likesPerHour: parseInt(process.env.MAX_LIKES_PER_HOUR),
      commentsPerHour: parseInt(process.env.MAX_COMMENTS_PER_HOUR)
    }
  });

  await bot.login();

  // Load target users
  const targetUsers = require('./competitor_followers.json');

  // Automated engagement loop
  for (const user of targetUsers) {
    try {
      // Follow user
      await bot.follow(user.username);
      
      // Like recent posts
      await bot.likeRecentPosts(user.username, { count: 3 });
      
      // Leave comment on latest post
      const comments = [
        "Great content! 🔥",
        "Love this! 💯",
        "Awesome post! 👏"
      ];
      await bot.commentOnLatestPost(
        user.username, 
        comments[Math.floor(Math.random() * comments.length)]
      );
      
      // Wait between actions (human-like behavior)
      await bot.randomDelay(30000, 60000);
      
    } catch (error) {
      console.error(`Error processing ${user.username}:`, error.message);
    }
  }

  console.log('Engagement campaign completed');
}

runEngagementCampaign().catch(console.error);
```

### 4. WhatsApp Mass Messaging

```python
from lib.whatsapp_sender import WhatsAppSender
import os
import time

def send_mass_messages():
    sender = WhatsAppSender(
        phone_number=os.getenv('WHATSAPP_PHONE_NUMBER'),
        api_key=os.getenv('WHATSAPP_API_KEY')
    )
    
    # Connect to WhatsApp
    sender.connect()
    
    # Load contact list
    contacts = sender.load_contacts('contacts.csv')
    
    # Message template
    message_template = """
    Hi {name}! 👋
    
    I noticed you're interested in {interest}. 
    
    We have a special offer running this week that I think you'd love!
    
    Check it out: {link}
    """
    
    # Send messages with rate limiting
    for contact in contacts:
        try:
            message = message_template.format(
                name=contact['name'],
                interest=contact['interest'],
                link=contact['custom_link']
            )
            
            sender.send_message(
                phone=contact['phone'],
                message=message
            )
            
            print(f"Sent to {contact['name']}")
            
            # Rate limit: 30 messages per hour
            time.sleep(120)  # 2 minutes between messages
            
        except Exception as e:
            print(f"Failed to send to {contact['name']}: {e}")
    
    sender.disconnect()

if __name__ == '__main__':
    send_mass_messages()
```

### 5. Multi-Platform Lead Generation Pipeline

```javascript
const { TikTokScraper } = require('./lib/tiktok-scraper');
const { InstagramBot } = require('./lib/instagram-bot');
const { WhatsAppSender } = require('./lib/whatsapp-sender');
const { LeadFilter } = require('./lib/lead-filter');

async function leadGenerationPipeline() {
  // Step 1: Extract leads from TikTok
  const tiktokScraper = new TikTokScraper({
    username: process.env.TIKTOK_USERNAME,
    password: process.env.TIKTOK_PASSWORD
  });
  
  const rawLeads = await tiktokScraper.getHashtagUsers('ecommerce', {
    maxCount: 10000
  });
  
  // Step 2: Filter and enrich leads
  const filter = new LeadFilter();
  const qualifiedLeads = await filter.process(rawLeads, {
    minFollowers: 500,
    maxFollowers: 50000,
    engagementRate: 0.03,
    countries: ['US', 'UK', 'CA'],
    ageRange: [25, 45]
  });
  
  // Step 3: Engage on Instagram
  const instagramBot = new InstagramBot({
    username: process.env.INSTAGRAM_USERNAME,
    password: process.env.INSTAGRAM_PASSWORD
  });
  
  for (const lead of qualifiedLeads.slice(0, 100)) {
    if (lead.instagramUsername) {
      await instagramBot.follow(lead.instagramUsername);
      await instagramBot.sendDM(lead.instagramUsername, 
        "Hey! Loved your TikTok content. Following you here too! 🎉"
      );
      await instagramBot.randomDelay(60000, 120000);
    }
  }
  
  // Step 4: WhatsApp follow-up (for converted leads)
  const whatsappSender = new WhatsAppSender({
    phoneNumber: process.env.WHATSAPP_PHONE_NUMBER,
    apiKey: process.env.WHATSAPP_API_KEY
  });
  
  const convertedLeads = qualifiedLeads.filter(l => l.contactInfo?.phone);
  
  for (const lead of convertedLeads) {
    await whatsappSender.sendMessage(
      lead.contactInfo.phone,
      `Hi ${lead.name}! Thanks for connecting on Instagram. 
      I'd love to share some exclusive offers with you! 🎁`
    );
    
    await new Promise(resolve => setTimeout(resolve, 150000)); // 2.5 min delay
  }
  
  console.log(`Pipeline complete: ${convertedLeads.length} leads contacted`);
}

leadGenerationPipeline().catch(console.error);
```

## Advanced Configuration

### Proxy Rotation

```javascript
const { ProxyManager } = require('./lib/proxy-manager');

const proxyManager = new ProxyManager({
  proxies: [
    { host: 'proxy1.example.com', port: 8080, user: 'user1', pass: 'pass1' },
    { host: 'proxy2.example.com', port: 8080, user: 'user2', pass: 'pass2' },
    { host: 'proxy3.example.com', port: 8080, user: 'user3', pass: 'pass3' }
  ],
  rotationStrategy: 'round-robin' // or 'random', 'least-used'
});

const scraper = new InstagramScraper({
  username: process.env.INSTAGRAM_USERNAME,
  password: process.env.INSTAGRAM_PASSWORD,
  proxyManager: proxyManager
});
```

### Session Management

```python
from lib.session_manager import SessionManager

session_mgr = SessionManager(
    storage_path='./sessions',
    max_session_age_hours=24
)

# Reuse existing session to avoid re-login
scraper = TikTokScraper(
    username=os.getenv('TIKTOK_USERNAME'),
    password=os.getenv('TIKTOK_PASSWORD'),
    session=session_mgr.get_session('tiktok_main')
)

# Session will be auto-saved after operations
scraper.login()
session_mgr.save_session('tiktok_main', scraper.get_session())
```

## Troubleshooting

### Rate Limiting / Account Blocks

**Problem**: Instagram/TikTok blocking or limiting your account

**Solutions**:
```javascript
// Reduce action rates
const bot = new InstagramBot({
  rateLimits: {
    followsPerHour: 20,  // Reduce from 50
    likesPerHour: 40,     // Reduce from 100
    commentsPerHour: 15   // Reduce from 40
  },
  humanBehavior: {
    enabled: true,
    minDelay: 30000,     // 30 seconds min
    maxDelay: 180000,    // 3 minutes max
    randomPauses: true   // Random breaks
  }
});

// Use residential proxies instead of datacenter
// Add random delays between sessions
// Warm up new accounts gradually
```

### Login Failures

**Problem**: Cannot authenticate to social platforms

**Solutions**:
```python
# Use session cookies instead of password login
scraper = InstagramScraper(
    session_file='instagram_session.json'
)

# Handle 2FA if enabled
scraper.login(
    username=os.getenv('INSTAGRAM_USERNAME'),
    password=os.getenv('INSTAGRAM_PASSWORD'),
    two_factor_code=input("Enter 2FA code: ")
)

# For WhatsApp, ensure QR code scanning works
whatsapp = WhatsAppSender()
whatsapp.connect(headless=False)  # Show browser for QR scan
```

### Data Extraction Errors

**Problem**: Scraping returns incomplete or no data

**Solutions**:
```javascript
// Add error handling and retries
async function safeExtract(scraper, username, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const data = await scraper.getFollowers(username);
      if (data && data.length > 0) return data;
    } catch (error) {
      console.log(`Attempt ${i + 1} failed: ${error.message}`);
      await new Promise(r => setTimeout(r, 5000 * (i + 1)));
    }
  }
  throw new Error(`Failed after ${retries} attempts`);
}

// Check for platform UI changes
// Update selectors if DOM structure changed
// Use API endpoints instead of scraping when available
```

### Memory Issues with Large Datasets

**Problem**: Script crashes when processing thousands of users

**Solutions**:
```python
# Process in batches
def process_in_batches(users, batch_size=100):
    for i in range(0, len(users), batch_size):
        batch = users[i:i + batch_size]
        process_batch(batch)
        # Free memory
        del batch
        import gc
        gc.collect()

# Stream data instead of loading all at once
def stream_followers(username):
    for follower_batch in scraper.get_followers_stream(username, batch_size=100):
        yield follower_batch
```

## Best Practices

1. **Start Slow**: New accounts should build activity gradually over days/weeks
2. **Use Proxies**: Residential proxies are essential for multi-account operations
3. **Rotate Sessions**: Don't use the same account 24/7
4. **Respect Rate Limits**: Platform limits exist for a reason
5. **Backup Data**: Save extracted data regularly
6. **Monitor Accounts**: Check for warnings or restrictions daily
7. **Comply with ToS**: Understand risks of automation on these platforms

## Legal and Ethical Considerations

⚠️ **Warning**: Automated scraping and mass messaging may violate platform Terms of Service and local laws (GDPR, CAN-SPAM, TCPA). Use responsibly and ensure compliance with:

- Platform Terms of Service
- Data protection regulations (GDPR, CCPA)
- Anti-spam laws
- User consent requirements

This skill documentation is for educational purposes. Users are responsible for compliance with all applicable laws and platform policies.
