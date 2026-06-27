---
name: n8n-marketing-flows
description: Open-source n8n workflow templates for marketing automation (social media, sentiment monitoring, news aggregation, ad management, AI content generation)
triggers:
  - "set up n8n marketing automation workflows"
  - "import marketing workflow templates into n8n"
  - "automate social media posting with n8n"
  - "monitor brand sentiment using n8n"
  - "create AI-powered content workflows in n8n"
  - "build marketing automation without code"
  - "connect Ollama to n8n for free AI workflows"
  - "automatically pause Facebook ads based on performance"
---

# n8n Marketing Flows Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What It Is

**n8n-marketing-flows** is a collection of 79+ ready-to-import n8n workflow templates for marketing automation. It covers:

- **Social media** publishing and hashtag generation
- **Sentiment monitoring** (PTT, social platforms)
- **News aggregation** and AI summaries
- **Ad management** (auto-pause low performers)
- **AI content generation** (posts, images, video)
- **Performance analytics** and reporting

Each workflow is a JSON file you import into [n8n](https://n8n.io) (self-hosted or cloud), connect your API keys, and activate.

**Two versions per workflow:**
- `*.json` — Standard (uses Claude/OpenAI + Email/Slack)
- `*.local-ollama.json` — **Free local** (uses local Ollama, no API keys needed)
- `*.skeleton.json` — **Credential template** (architecture ready, you fill in API keys)

## Installation

### 1. Set Up n8n

**Option A: Docker (self-hosted)**
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```
Access at `http://localhost:5678`

**Option B: n8n Cloud**
Sign up at https://n8n.io (free tier available)

**Option C: npm**
```bash
npm install -g n8n
n8n start
```

### 2. Clone the Template Repository

```bash
git clone https://github.com/YuriCrystal/n8n-marketing-flows.git
cd n8n-marketing-flows
```

### 3. Import a Workflow

1. In n8n UI: **⋯ menu** (top right) → **Import from File**
2. Select a JSON file from `templates/`
3. Configure credentials (see below)
4. Click **Save** and **Activate**

## Project Structure

```
n8n-marketing-flows/
├── templates/
│   ├── 01-weekly-ai-news-digest.json
│   ├── 01-weekly-ai-news-digest.local-ollama.json
│   ├── 02-social-hashtag-generator.json
│   ├── 04-ptt-sentiment-monitor.local-ollama.json
│   ├── 90-viral-social-post.json
│   ├── 91-webhook-bridge.json
│   └── [59 more *.skeleton.json templates]
├── docs/
│   ├── 01-weekly-ai-news-digest.md
│   └── 91-webhook-bridge.md
├── ROADMAP.md
├── CREDITS.md
└── README.md
```

## Key Workflow Types

### 1. **Free Local Workflows** (19 files, `*.local-ollama.json`)

Uses **Ollama** (free, runs locally) + public data sources (RSS, web scraping). No API keys required.

**Example: Weekly AI News Digest**
```bash
# Start Ollama locally
ollama pull llama3.2

# Import templates/01-weekly-ai-news-digest.local-ollama.json
# Configure:
# - RSS feeds: TechCrunch, Hacker News, etc.
# - Schedule: Every Monday 9am
# - Output: Discord webhook or n8n display
```

### 2. **Standard Workflows** (uses Claude/OpenAI)

Requires **Anthropic** or **OpenAI** API key.

**Setup credentials in n8n:**
```
Settings → Credentials → Add → Anthropic/OpenAI
API Key: <your key from env>
Default Model: claude-haiku-4-5 (or gpt-4o-mini)
```

### 3. **Platform Integration Skeletons** (59 files, `*.skeleton.json`)

Pre-built architecture for:
- **Meta Graph API** (Facebook, Instagram, Threads)
- **Google Sheets** (data export)
- **YouTube Data API** (analytics)
- **WordPress REST API** (auto-publish)
- **Meta Marketing API** (ad management)

**These require YOUR platform API keys** — see credential sections below.

## Core Workflows

### Social Media Posting

**Workflow 02: Generate Hashtags for Post**
```json
// Input: Post text + platform (Instagram/Facebook/Threads)
// Output: Optimized hashtags

// Usage in n8n:
Webhook → HTTP Request (LLM) → Set (format) → Respond
```

**Example prompt to LLM node:**
```javascript
// In "Prompt" field of HTTP Request node:
`Generate 5-10 hashtags for this {{ $json.platform }} post:

"{{ $json.postText }}"

Rules:
- Instagram: Mix popular + niche, under 30 chars each
- Facebook: 3-5 broad hashtags
- Threads: Conversational, 5-7 hashtags

Return only hashtags, space-separated.`
```

**Import `templates/02-social-hashtag-generator.json`** and customize the prompt.

### Sentiment Monitoring

**Workflow 04: PTT Forum Monitor**

Scrapes PTT (Taiwan's largest forum), detects mentions, sends alerts.

```javascript
// In HTTP Request node (scrape PTT):
{
  "method": "GET",
  "url": "https://www.ptt.cc/bbs/{{ $json.board }}/index.html",
  "headers": {
    "Cookie": "over18=1"  // Age verification
  }
}

// In Code node (filter mentions):
const keyword = "{{ $json.brandName }}";
const posts = $input.all();
return posts.filter(p => 
  p.json.title.includes(keyword) || 
  p.json.content.includes(keyword)
);
```

**Import `templates/04-ptt-sentiment-monitor.local-ollama.json`**

### AI Content Generation

**Workflow 90: Viral Social Post Generator**

Uses反主流 (counter-mainstream) formula from Hao's research (see CREDITS.md).

```javascript
// LLM prompt structure:
`You are a social media expert. Generate a viral post using this formula:

1. **Hook** (first line): Controversial or counter-intuitive
2. **Contrast**: "Everyone says X, but actually Y"
3. **Story**: Personal anecdote or data
4. **CTA**: "What's your experience?"

Topic: {{ $json.topic }}
Platform: {{ $json.platform }}
Tone: {{ $json.tone }}

Output 3 variations under 280 chars each.`
```

**Import `templates/90-viral-social-post.json`** or `.local-ollama.json` version.

### Webhook Bridge for External Tools

**Workflow 91: Webhook Bridge**

Allows AI agents, Zapier, Make.com, or scripts to call n8n workflows via HTTP.

```bash
# Activate workflow 91, get webhook URL:
# https://your-n8n.com/webhook/marketing-flow

# Call from code:
curl -X POST https://your-n8n.com/webhook/marketing-flow \
  -H "Content-Type: application/json" \
  -d '{
    "action": "generatePost",
    "topic": "AI automation trends",
    "platform": "linkedin"
  }'

# Returns JSON with generated content
```

**In n8n workflow:**
```javascript
// Webhook node receives POST
// Switch node routes by $json.action
// LLM node generates content
// Respond to Webhook node returns JSON
```

## Configuration Patterns

### Switching LLM Providers

**Claude (default)**
```javascript
// HTTP Request node:
{
  "url": "https://api.anthropic.com/v1/messages",
  "method": "POST",
  "headers": {
    "x-api-key": "{{ $credentials.anthropicApiKey }}",
    "anthropic-version": "2023-06-01"
  },
  "body": {
    "model": "claude-haiku-4-5",
    "messages": [{"role": "user", "content": "{{ $json.prompt }}"}]
  }
}
```

**OpenAI**
```javascript
{
  "url": "https://api.openai.com/v1/chat/completions",
  "method": "POST",
  "headers": {
    "Authorization": "Bearer {{ $credentials.openaiApiKey }}"
  },
  "body": {
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "{{ $json.prompt }}"}]
  }
}
```

**Local Ollama** (free)
```javascript
{
  "url": "http://localhost:11434/api/generate",
  "method": "POST",
  "body": {
    "model": "llama3.2",
    "prompt": "{{ $json.prompt }}",
    "stream": false
  }
}
```

### Notification Channels

**Email (SMTP)**
```javascript
// Send Email node:
To: process.env.ALERT_EMAIL
Subject: [Alert] {{ $json.title }}
Body: {{ $json.summary }}
```

**Discord Webhook**
```bash
# Get webhook URL from Discord server settings
# In HTTP Request node:
POST https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/TOKEN
{
  "content": "🚨 New mention detected",
  "embeds": [{
    "title": "{{ $json.title }}",
    "url": "{{ $json.link }}",
    "color": 15258703
  }]
}
```

**Slack**
```javascript
// Slack node (requires Slack app):
Channel: #marketing-alerts
Message: :warning: {{ $json.alert }}
```

### Scheduling

```javascript
// Cron node examples:
// Every Monday 9am:
0 9 * * 1

// Every 6 hours:
0 */6 * * *

// Weekdays at 10am:
0 10 * * 1-5
```

## Credential Setup

### Anthropic (Claude)

```bash
# Get key: https://console.anthropic.com/settings/keys
# In n8n: Credentials → Anthropic API
API_KEY=your_anthropic_key
```

### OpenAI

```bash
# Get key: https://platform.openai.com/api-keys
# In n8n: Credentials → OpenAI
API_KEY=your_openai_key
```

### Meta Graph API (Facebook/Instagram)

```bash
# Create app: https://developers.facebook.com
# Get access token with pages_read_engagement, pages_manage_posts
# In n8n: Credentials → HTTP Request (Bearer Token)
ACCESS_TOKEN=your_meta_token
```

**Use in workflow:**
```javascript
// Get Instagram posts:
GET https://graph.facebook.com/v18.0/me/media
  ?fields=id,caption,like_count,timestamp
  &access_token={{ $credentials.metaToken }}
```

### Google Sheets

```bash
# In n8n: Credentials → Google Sheets API → OAuth2
# Follow OAuth flow, authorize scopes:
# https://www.googleapis.com/auth/spreadsheets
```

## Common Use Cases

### 1. Weekly News Summary

```bash
# Import: templates/01-weekly-ai-news-digest.local-ollama.json

# Modify RSS feeds in RSS Read node:
- https://techcrunch.com/feed/
- https://news.ycombinator.com/rss
- https://www.theverge.com/rss/index.xml

# Change schedule in Cron node: 0 9 * * 1 (Monday 9am)

# Output to Discord webhook:
POST https://discord.com/api/webhooks/YOUR_ID/TOKEN
{
  "content": "📰 This week's AI news:\n\n{{ $json.summary }}"
}
```

### 2. Auto-Pause Low-Performing Ads

```bash
# Import: templates/fb-ad-auto-pause.skeleton.json

# Configure Meta Marketing API credentials
# Set threshold in IF node:
{{ $json.ctr }} < 0.01  // Click-through rate below 1%

# HTTP Request node:
POST https://graph.facebook.com/v18.0/{{ $json.adId }}
{
  "status": "PAUSED",
  "access_token": "{{ $credentials.metaToken }}"
}
```

### 3. Multi-Platform Post Scheduler

```bash
# Import: templates/05-multi-platform-post-draft.json

# Input: Topic + platforms (array: ["facebook", "instagram", "linkedin"])

# LLM generates platform-specific versions:
# - Facebook: Conversational, 280 chars
# - Instagram: Visual-first, 10 hashtags
# - LinkedIn: Professional, 500 chars

# Output: JSON array of drafts → Save to Google Sheets or send to Slack
```

## Troubleshooting

### "Missing credentials" Error

**Solution:** Click nodes with 🔑 icon, fill in API keys.

```bash
# Check required credentials for each workflow:
grep -r "credentials" templates/01-*.json

# Common ones:
- Anthropic API (Claude)
- OpenAI API
- SMTP (Email)
- Meta Graph API
- Google OAuth2
```

### Ollama Connection Failed

```bash
# Ensure Ollama is running:
ollama serve

# Test connection:
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Test"
}'

# Pull model if missing:
ollama pull llama3.2
```

### RSS Feed Not Updating

```bash
# Check feed URL in RSS Read node
# Validate feed: https://validator.w3.org/feed/

# Increase "Fetch Interval" in Cron node:
0 */2 * * *  # Every 2 hours instead of daily
```

### Rate Limits (Meta, OpenAI)

**Solution:** Add **Wait** node between API calls.

```javascript
// After HTTP Request node:
Wait node → Time: 1 second

// Or use Batch node to process in chunks:
Batch Size: 10
```

### LLM Returns Empty Response

```bash
# Check prompt in HTTP Request body:
- Ensure {{ $json.prompt }} variable exists
- Test prompt in Expression editor (=)
- Increase max_tokens in API call:
  "max_tokens": 500
```

## Advanced Patterns

### Dynamic Prompt Engineering

```javascript
// Use Code node to build context-aware prompts:
const platform = $json.platform;
const tone = $json.tone || 'professional';

const prompts = {
  instagram: `Create a visual-first post with emojis...`,
  linkedin: `Write a thought leadership piece...`,
  twitter: `Craft a controversial take under 280 chars...`
};

return {
  json: {
    prompt: prompts[platform] + `\n\nTone: ${tone}\nTopic: ${$json.topic}`
  }
};
```

### Multi-Step AI Workflows

```javascript
// Chain multiple LLM calls:
1. Generate outline (Claude)
2. Expand each point (GPT-4)
3. Add SEO keywords (local Llama)
4. Format for platform (Claude)

// Use Set node to pass data between steps:
Set node after each LLM → Store in {{ $json.step1Output }}
```

### Error Handling

```javascript
// Add Error Trigger node:
On Error → Send alert to Slack → Retry workflow

// In HTTP Request node:
{
  "continueOnFail": true,
  "retry": {
    "maxRetries": 3,
    "waitBetweenRetries": 5000
  }
}
```

## Environment Variables

**Set in n8n Docker:**
```bash
docker run -e ANTHROPIC_API_KEY=sk-xxx \
  -e OPENAI_API_KEY=sk-xxx \
  -e ALERT_EMAIL=team@company.com \
  n8nio/n8n
```

**Reference in workflows:**
```javascript
// In expressions:
{{ $env.ANTHROPIC_API_KEY }}
{{ $env.ALERT_EMAIL }}
```

## Resources

- **Full documentation:** https://github.com/YuriCrystal/n8n-marketing-flows/tree/main/docs
- **n8n docs:** https://docs.n8n.io
- **Ollama models:** https://ollama.com/library
- **Workflow examples:** `/templates` folder (79 files)

## License

MIT — Freely use, modify, and distribute. Attribution required.
