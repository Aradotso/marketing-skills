---
name: ai-festival-campaign-generator
description: Multi-agent AI marketing automation system that generates complete festival campaigns with n8n, Gemini, and automated delivery
triggers:
  - generate a festival marketing campaign
  - create festival campaign with AI agents
  - automate festival marketing with n8n
  - build multi-agent campaign generator
  - generate festival ads and visuals
  - create automated marketing workflow for festivals
  - set up AI campaign automation system
  - generate complete festival marketing assets
---

# AI Festival Campaign Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the AI Festival Campaign Generator, a multi-agent marketing automation system built with n8n that generates complete festival marketing campaigns including copywriting, visual assets, quality assurance, and automated delivery via email.

## What It Does

The AI Festival Campaign Generator orchestrates multiple AI agents to:

- Research festival context and consumer psychology
- Generate multi-channel marketing copy (social media, email, SMS, YouTube)
- Create AI-powered visual assets (product images, banners, stories)
- Perform campaign quality assurance
- Store assets in Google Drive
- Log campaigns in Google Sheets
- Deliver formatted campaign kits via email

## Architecture Overview

The system uses a 4-agent architecture:

1. **Festival Intelligence Agent**: Researches festival, analyzes consumer behavior, recommends offers
2. **Campaign Copywriter Agent**: Generates copy for 10+ marketing channels
3. **AI Visual Generator Agent**: Creates and uploads visual assets to Drive
4. **Campaign QA Reviewer Agent**: Reviews quality and provides deployment score

## Installation

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google Gemini API key
- Google Cloud project with Drive, Sheets, and Gmail APIs enabled
- OAuth2 credentials for Google services

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/Chaitanyaa0406/AI-Festival-Campaign-Generator.git
cd AI-Festival-Campaign-Generator
```

### n8n Workflow Import

1. Open your n8n instance
2. Navigate to Workflows > Import from File
3. Select `workflow/festival-campaign-generator.json`
4. Configure credentials for each service

### Environment Configuration

Set up the following environment variables or n8n credentials:

```bash
# Gemini API
GEMINI_API_KEY=your_gemini_api_key

# Google Drive OAuth2
GOOGLE_DRIVE_CLIENT_ID=your_client_id
GOOGLE_DRIVE_CLIENT_SECRET=your_client_secret

# Google Sheets OAuth2
GOOGLE_SHEETS_CLIENT_ID=your_client_id
GOOGLE_SHEETS_CLIENT_SECRET=your_client_secret

# Gmail OAuth2
GMAIL_CLIENT_ID=your_client_id
GMAIL_CLIENT_SECRET=your_client_secret
```

## Key Components

### 1. Festival Intelligence Agent

**Purpose**: Research festival and generate strategic insights

**Gemini Prompt Structure**:

```javascript
// n8n Code Node - Festival Research
const festivalName = $json.festival_name;
const productName = $json.product_name;
const brandName = $json.brand_name;

const prompt = `You are a Festival Marketing Intelligence Agent.

Festival: ${festivalName}
Brand: ${brandName}
Product: ${productName}

Analyze and provide:
1. Festival significance and cultural context
2. Consumer psychology during this festival
3. Emotional triggers and pain points
4. Shopping behavior patterns
5. Recommended offer types (discount %, BOGO, bundles)
6. Trending hashtags (10-15)
7. Best posting times and channels
8. Festival-specific keywords for SEO

Format as structured JSON.`;

return {
  json: {
    prompt: prompt,
    festivalName: festivalName,
    productName: productName
  }
};
```

### 2. Campaign Copywriter Agent

**Purpose**: Generate multi-channel marketing copy

**Copy Generation Pattern**:

```javascript
// n8n Code Node - Campaign Copy Generation
const festivalInsights = $json.festival_insights;
const productName = $json.product_name;
const brandName = $json.brand_name;

const copyPrompt = `You are an Expert Marketing Copywriter.

Using these insights:
${JSON.stringify(festivalInsights)}

Generate complete campaign copy for:

1. Facebook Ad (Primary Text, Headline, Description)
2. Instagram Ad (Caption with emojis, hashtags)
3. Google Search Ad (3 Headlines, 2 Descriptions)
4. YouTube Script (30-second hook + 2-minute full script)
5. Email Sequence (Subject, Preview, Body HTML for 3 emails)
6. WhatsApp Campaign (Opening message, follow-up, closing)
7. SMS Campaign (160 characters with link)
8. Push Notification (Title + Body under 50 chars)
9. Website Hero Section (Headline, Subheadline, CTA)
10. Influencer Brief (Key points, talking points, hashtags)
11. SEO Keywords (20 long-tail keywords)
12. Content Calendar (7-day posting schedule)

Return as structured JSON with each channel as a key.`;

return {
  json: {
    prompt: copyPrompt,
    campaign_id: `CAMP_${Date.now()}`
  }
};
```

### 3. AI Visual Generator Agent

**Purpose**: Generate and store visual assets

**Image Generation Integration**:

```javascript
// n8n Code Node - Visual Asset Prompts
const festivalName = $json.festival_name;
const productName = $json.product_name;
const brandColors = $json.brand_colors || "vibrant festive colors";

const visualPrompts = {
  product_visual: `Professional product photography of ${productName} with ${festivalName} themed decorations, ${brandColors}, high quality, marketing shot, clean background`,
  
  instagram_story: `Vertical 9:16 Instagram story design for ${festivalName}, featuring ${productName}, bold text overlay with festival offer, ${brandColors}, modern design, attention-grabbing`,
  
  ad_banner: `Horizontal advertisement banner 1200x628px for ${festivalName} sale, featuring ${productName}, professional design, ${brandColors}, clear CTA, festival elements`
};

return {
  json: {
    visuals: visualPrompts,
    campaign_id: $json.campaign_id
  }
};
```

**Google Drive Upload Pattern**:

```javascript
// n8n Google Drive Node Configuration
// After receiving generated images from Gemini/DALL-E

// Configure in n8n UI:
// - Action: Upload
// - Folder: Use expression {{$json.campaign_folder_id}}
// - File Name: {{$json.image_type}}_{{$json.campaign_id}}.png
// - Binary Data: true
// - Binary Property: data

// Post-upload processing
const driveFiles = $json.files.map(file => ({
  name: file.name,
  url: file.webViewLink,
  id: file.id
}));

return {
  json: {
    campaign_id: $json.campaign_id,
    drive_files: driveFiles
  }
};
```

### 4. Campaign QA Reviewer Agent

**Purpose**: Quality assurance and scoring

**QA Evaluation Pattern**:

```javascript
// n8n Code Node - QA Review
const campaignCopy = $json.campaign_copy;
const visualAssets = $json.visual_assets;
const festivalInsights = $json.festival_insights;

const qaPrompt = `You are a Campaign Quality Assurance Expert.

Review this campaign:
Copy: ${JSON.stringify(campaignCopy)}
Visuals: ${visualAssets.length} assets created
Festival Context: ${JSON.stringify(festivalInsights)}

Evaluate and score (0-100) on:
1. Festival Relevance (0-20 points)
2. Emotional Impact (0-20 points)
3. CTA Strength (0-15 points)
4. Offer Attractiveness (0-15 points)
5. Copy Quality (0-15 points)
6. Visual-Copy Alignment (0-15 points)

Provide:
- Total Score (0-100)
- Deployment Recommendation (YES/NO/REVISE)
- Strengths (3 bullet points)
- Improvements (3 bullet points)
- Priority Fixes (if score < 80)

Return as JSON.`;

return {
  json: {
    prompt: qaPrompt,
    campaign_id: $json.campaign_id
  }
};
```

### 5. Campaign Logging (Google Sheets)

**Sheets Integration Pattern**:

```javascript
// n8n Google Sheets Node - Append Row
// Configure in n8n UI:
// - Operation: Append
// - Sheet: Campaign Log
// - Columns: Map the following fields

const logEntry = {
  campaign_id: $json.campaign_id,
  timestamp: new Date().toISOString(),
  festival_name: $json.festival_name,
  product_name: $json.product_name,
  brand_name: $json.brand_name,
  qa_score: $json.qa_score,
  deployment_status: $json.deployment_recommendation,
  drive_folder: $json.drive_folder_url,
  email_sent: 'YES',
  channels_covered: 'FB,IG,GG,YT,Email,WA,SMS',
  created_by: $json.user_email
};

return { json: logEntry };
```

### 6. Email Delivery

**HTML Email Template Pattern**:

```javascript
// n8n Code Node - Generate HTML Email
const campaignData = $json;

const htmlEmail = `
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
    .header { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); 
              color: white; padding: 30px; text-align: center; }
    .content { padding: 30px; max-width: 800px; margin: 0 auto; }
    .section { margin: 30px 0; padding: 20px; background: #f9f9f9; border-radius: 8px; }
    .score { font-size: 48px; font-weight: bold; color: #667eea; text-align: center; }
    .channel { background: white; padding: 15px; margin: 10px 0; border-left: 4px solid #667eea; }
    .cta-button { background: #667eea; color: white; padding: 15px 30px; 
                  text-decoration: none; border-radius: 5px; display: inline-block; }
  </style>
</head>
<body>
  <div class="header">
    <h1>🎉 ${campaignData.festival_name} Campaign Ready!</h1>
    <p>Campaign ID: ${campaignData.campaign_id}</p>
  </div>
  
  <div class="content">
    <div class="section">
      <h2>📊 Campaign Quality Score</h2>
      <div class="score">${campaignData.qa_score}/100</div>
      <p><strong>Status:</strong> ${campaignData.deployment_recommendation}</p>
    </div>
    
    <div class="section">
      <h2>✍️ Generated Copy</h2>
      
      <div class="channel">
        <h3>📘 Facebook Ad</h3>
        <p><strong>Headline:</strong> ${campaignData.copy.facebook.headline}</p>
        <p>${campaignData.copy.facebook.primary_text}</p>
      </div>
      
      <div class="channel">
        <h3>📸 Instagram Ad</h3>
        <p>${campaignData.copy.instagram.caption}</p>
        <p><small>${campaignData.copy.instagram.hashtags}</small></p>
      </div>
      
      <div class="channel">
        <h3>📧 Email Subject Lines</h3>
        <ul>
          ${campaignData.copy.email_sequence.map(e => `<li>${e.subject}</li>`).join('')}
        </ul>
      </div>
    </div>
    
    <div class="section">
      <h2>🎨 Visual Assets</h2>
      <p>All assets uploaded to Google Drive:</p>
      <a href="${campaignData.drive_folder_url}" class="cta-button">
        View Campaign Assets →
      </a>
    </div>
    
    <div class="section">
      <h2>💡 Strategic Insights</h2>
      <p><strong>Best Posting Times:</strong> ${campaignData.insights.best_times}</p>
      <p><strong>Recommended Hashtags:</strong></p>
      <p>${campaignData.insights.hashtags.join(' ')}</p>
    </div>
    
    <div class="section">
      <h2>📅 Content Calendar</h2>
      <pre>${JSON.stringify(campaignData.content_calendar, null, 2)}</pre>
    </div>
  </div>
</body>
</html>
`;

return {
  json: {
    html: htmlEmail,
    subject: `🎉 ${campaignData.festival_name} Campaign Ready - Score: ${campaignData.qa_score}/100`,
    to: campaignData.client_email
  }
};
```

## Common Workflow Patterns

### Pattern 1: Campaign Trigger via Webhook

```javascript
// n8n Webhook Node - POST /generate-campaign
{
  "festival_name": "Diwali",
  "product_name": "Premium Smartphones",
  "brand_name": "TechBrand",
  "brand_colors": "blue and white",
  "target_audience": "young professionals",
  "budget_range": "mid-premium",
  "client_email": "client@example.com",
  "user_email": "agent@agency.com"
}
```

### Pattern 2: Campaign ID Generation

```javascript
// n8n Code Node - Generate Unique Campaign ID
const timestamp = Date.now();
const festivalCode = $json.festival_name.substring(0, 3).toUpperCase();
const productCode = $json.product_name.substring(0, 3).toUpperCase();

const campaignId = `${festivalCode}_${productCode}_${timestamp}`;

return {
  json: {
    ...$json,
    campaign_id: campaignId,
    created_at: new Date().toISOString()
  }
};
```

### Pattern 3: Gemini API Call Configuration

```javascript
// n8n HTTP Request Node - Gemini API
// Method: POST
// URL: https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent

const requestBody = {
  contents: [{
    parts: [{
      text: $json.prompt
    }]
  }],
  generationConfig: {
    temperature: 0.7,
    topK: 40,
    topP: 0.95,
    maxOutputTokens: 8192,
  },
  safetySettings: [
    {
      category: "HARM_CATEGORY_HARASSMENT",
      threshold: "BLOCK_MEDIUM_AND_ABOVE"
    }
  ]
};

return {
  json: requestBody
};
```

### Pattern 4: Error Handling and Retry Logic

```javascript
// n8n Code Node - Error Handler
try {
  const response = $json;
  
  if (!response.candidates || response.candidates.length === 0) {
    throw new Error('No content generated from Gemini API');
  }
  
  const generatedText = response.candidates[0].content.parts[0].text;
  const parsedContent = JSON.parse(generatedText);
  
  return {
    json: {
      success: true,
      content: parsedContent,
      campaign_id: $json.campaign_id
    }
  };
  
} catch (error) {
  return {
    json: {
      success: false,
      error: error.message,
      campaign_id: $json.campaign_id,
      retry_needed: true
    }
  };
}
```

### Pattern 5: Conditional Campaign Deployment

```javascript
// n8n Switch Node Logic
// Route based on QA score

const qaScore = $json.qa_score;

if (qaScore >= 85) {
  return {
    json: {
      ...$json,
      deployment_path: 'auto_deploy',
      send_email: true,
      approval_needed: false
    }
  };
} else if (qaScore >= 70) {
  return {
    json: {
      ...$json,
      deployment_path: 'human_review',
      send_email: true,
      approval_needed: true
    }
  };
} else {
  return {
    json: {
      ...$json,
      deployment_path: 'revise',
      send_email: false,
      approval_needed: true,
      revision_priority: 'high'
    }
  };
}
```

## Configuration Best Practices

### Google Drive Folder Structure

Create this folder structure in your Google Drive:

```
Festival Campaigns/
├── 2024/
│   ├── Diwali/
│   │   ├── CAMP_12345/
│   │   │   ├── visuals/
│   │   │   ├── copy/
│   │   │   └── reports/
```

### Google Sheets Campaign Log Schema

```javascript
// Recommended Sheets columns
const sheetColumns = [
  'campaign_id',
  'timestamp',
  'festival_name',
  'product_name',
  'brand_name',
  'target_audience',
  'qa_score',
  'deployment_status',
  'drive_folder_url',
  'email_sent_to',
  'channels_covered',
  'estimated_reach',
  'created_by',
  'approved_by',
  'launch_date'
];
```

### Gemini Model Selection

```javascript
// Choose model based on task complexity
const modelMapping = {
  festival_research: 'gemini-pro',
  copy_generation: 'gemini-pro',
  visual_prompts: 'gemini-pro',
  qa_review: 'gemini-pro',
  image_generation: 'gemini-pro-vision' // if using vision capabilities
};
```

## Troubleshooting

### Issue: Gemini API Rate Limits

```javascript
// n8n Code Node - Rate Limit Handler
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function callGeminiWithRetry(prompt, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      // API call here
      return response;
    } catch (error) {
      if (error.status === 429) {
        await delay(2000 * (i + 1)); // Exponential backoff
        continue;
      }
      throw error;
    }
  }
}
```

### Issue: Google Drive Upload Failures

```javascript
// Check binary data format
if (!$binary.data) {
  throw new Error('No binary data found for upload');
}

// Verify file size
const maxSize = 10 * 1024 * 1024; // 10MB
if ($binary.data.fileSize > maxSize) {
  throw new Error('File size exceeds limit');
}
```

### Issue: Malformed JSON from Gemini

```javascript
// n8n Code Node - JSON Parser with Fallback
function parseGeminiResponse(text) {
  try {
    return JSON.parse(text);
  } catch (e) {
    // Try to extract JSON from markdown code blocks
    const jsonMatch = text.match(/```json\n([\s\S]*?)\n```/);
    if (jsonMatch) {
      return JSON.parse(jsonMatch[1]);
    }
    
    // Try to find JSON object in text
    const objMatch = text.match(/\{[\s\S]*\}/);
    if (objMatch) {
      return JSON.parse(objMatch[0]);
    }
    
    throw new Error('Could not parse JSON from Gemini response');
  }
}
```

### Issue: Email Delivery Failures

```javascript
// Gmail API error handling
if ($json.error) {
  console.error('Gmail API Error:', $json.error);
  
  // Log to Sheets for follow-up
  return {
    json: {
      campaign_id: $json.campaign_id,
      email_status: 'FAILED',
      error_message: $json.error.message,
      retry_at: new Date(Date.now() + 3600000).toISOString()
    }
  };
}
```

### Issue: Incomplete Campaign Data

```javascript
// n8n Code Node - Data Validation
function validateCampaignData(data) {
  const required = ['festival_name', 'product_name', 'brand_name', 'client_email'];
  const missing = required.filter(field => !data[field]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required fields: ${missing.join(', ')}`);
  }
  
  // Validate email format
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(data.client_email)) {
    throw new Error('Invalid client email format');
  }
  
  return true;
}
```

## Advanced Features

### Memory Integration (Future Enhancement)

```javascript
// Pattern for adding campaign memory
const campaignMemory = {
  brand_guidelines: $('Get Brand Guidelines').all(),
  past_campaigns: $('Query Past Campaigns').all(),
  performance_data: $('Get Performance Metrics').all()
};

const enhancedPrompt = `
${basePrompt}

Historical Context:
- Brand Guidelines: ${JSON.stringify(campaignMemory.brand_guidelines)}
- Top Performing Campaigns: ${JSON.stringify(campaignMemory.past_campaigns.slice(0, 3))}

Generate copy consistent with brand voice and proven patterns.
`;
```

### Multi-Language Support

```javascript
// Language detection and translation
const supportedLanguages = ['en', 'hi', 'es', 'fr'];
const targetLanguage = $json.target_language || 'en';

if (supportedLanguages.includes(targetLanguage)) {
  const translationPrompt = `
  Translate the following campaign copy to ${targetLanguage}:
  ${JSON.stringify($json.campaign_copy)}
  
  Maintain cultural relevance and marketing impact.
  `;
}
```

### Analytics Integration

```javascript
// Track campaign performance
const analyticsPayload = {
  campaign_id: $json.campaign_id,
  event: 'campaign_created',
  properties: {
    festival: $json.festival_name,
    channels: $json.channels_covered,
    qa_score: $json.qa_score,
    timestamp: new Date().toISOString()
  }
};

// Send to analytics platform
return { json: analyticsPayload };
```

This skill provides comprehensive coverage of the AI Festival Campaign Generator workflow, enabling AI coding agents to help developers implement, customize, and troubleshoot multi-agent marketing automation systems.
