---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude and OpenAI
triggers:
  - automate content creation with AI research and video generation
  - set up AI-powered marketing content pipeline
  - generate videos from blog posts automatically
  - create multi-language content with AI agents
  - build automated content workflow from research to video
  - integrate Claude and OpenAI for content automation
  - schedule and publish AI-generated content automatically
  - crawl news sources and generate marketing content
---

# Marketing Pipeline Share AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system that transforms keywords into published content and videos. It combines web scraping for research, AI-powered content generation (Claude 3/OpenAI), and video rendering (Remotion) into a single pipeline.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Dual-language output (English & Vietnamese)
- Auto-render videos and infographics via Remotion
- Multi-platform publishing (Reels, TikTok, Shorts)

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Content Publishing
FACEBOOK_PAGE_TOKEN=your_fb_page_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion Rendering
REMOTION_LICENSE_KEY=your_remotion_license
```

## Core Workflow

### 1. Research & Data Collection

The pipeline starts by crawling recent content from configured sources:

```typescript
import { crawlNewsSources } from './services/research';

// Crawl news for a specific topic
const researchData = await crawlNewsSources({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  limit: 20
});

// Returns structured data
// {
//   articles: [...],
//   insights: [...],
//   trends: [...],
//   statistics: [...]
// }
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from './services/content-generator';

const content = await generateContent({
  topic: 'AI automation trends 2026',
  format: 'toplist', // 'pov' | 'case-study' | 'how-to'
  language: 'both', // 'en' | 'vi' | 'both'
  tone: 'expert', // 'friendly' | 'humorous' | 'expert'
  researchData: researchData,
  aiProvider: 'claude' // or 'openai'
});

// Returns
// {
//   title: { en: '...', vi: '...' },
//   body: { en: '...', vi: '...' },
//   metadata: { ... },
//   mediaAssets: [...]
// }
```

### 3. Video Rendering with Remotion

Transform content into videos automatically:

```typescript
import { renderVideo } from './services/video-renderer';

const videoPath = await renderVideo({
  content: content,
  template: 'infographic', // 'talking-head' | 'text-animation' | 'infographic'
  platform: 'reels', // 'tiktok' | 'shorts' | 'reels'
  aspectRatio: '9:16',
  duration: 60, // seconds
  style: {
    theme: 'modern',
    colors: ['#FF6B6B', '#4ECDC4'],
    font: 'Inter'
  }
});

// Returns path to rendered video
// './output/video_123456.mp4'
```

### 4. Publishing & Scheduling

```typescript
import { publishContent } from './services/publisher';

await publishContent({
  content: content,
  video: videoPath,
  platforms: ['facebook', 'linkedin'],
  schedule: new Date('2026-06-15T10:00:00Z'),
  options: {
    autoHashtags: true,
    crossPost: true
  }
});
```

## Full Pipeline Example

```typescript
import { ContentPipeline } from './pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude',
  defaultLanguages: ['en', 'vi'],
  autoPublish: false
});

// Run complete pipeline
const result = await pipeline.run({
  keyword: 'Marketing automation AI',
  contentFormats: ['toplist', 'how-to'],
  videoEnabled: true,
  publishSchedule: {
    facebook: '2026-06-15T09:00:00Z',
    linkedin: '2026-06-15T14:00:00Z'
  }
});

// Monitor pipeline status
pipeline.on('research:complete', (data) => {
  console.log(`Found ${data.articles.length} articles`);
});

pipeline.on('content:generated', (content) => {
  console.log(`Generated: ${content.title.en}`);
});

pipeline.on('video:rendered', (path) => {
  console.log(`Video ready: ${path}`);
});
```

## API Service Integration

### Claude 3 Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, context: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ]
  });

  return message.content[0].text;
}
```

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

## Configuration

Create a `pipeline.config.ts` file:

```typescript
export default {
  research: {
    sources: [
      {
        name: 'techcrunch',
        enabled: true,
        priority: 1
      },
      {
        name: 'a16z',
        enabled: true,
        priority: 2
      }
    ],
    crawlInterval: '24h',
    maxArticles: 50
  },
  
  content: {
    defaultFormat: 'toplist',
    supportedFormats: ['toplist', 'pov', 'case-study', 'how-to'],
    defaultTone: 'expert',
    languages: ['en', 'vi'],
    aiProvider: 'claude', // or 'openai'
    wordCount: {
      min: 800,
      max: 2000
    }
  },
  
  video: {
    enabled: true,
    defaultTemplate: 'infographic',
    platforms: {
      reels: { aspectRatio: '9:16', maxDuration: 90 },
      tiktok: { aspectRatio: '9:16', maxDuration: 60 },
      shorts: { aspectRatio: '9:16', maxDuration: 60 }
    },
    renderQuality: 'high'
  },
  
  publishing: {
    autoPublish: false,
    platforms: ['facebook', 'linkedin'],
    scheduling: {
      timezone: 'Asia/Ho_Chi_Minh',
      optimalTimes: ['09:00', '14:00', '19:00']
    }
  }
};
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the UI
# http://localhost:3000
```

## CLI Commands

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist

# Render video from existing content
npm run render-video -- --content-id 12345 --template infographic

# Crawl news sources
npm run research -- --sources techcrunch,a16z --hours 24

# Publish scheduled content
npm run publish -- --platform facebook --schedule "2026-06-15 09:00"
```

## Common Patterns

### Batch Content Generation

```typescript
const keywords = [
  'AI marketing automation',
  'Content creation tools',
  'Social media trends 2026'
];

const batchResults = await Promise.all(
  keywords.map(keyword => 
    pipeline.run({
      keyword,
      contentFormats: ['toplist'],
      videoEnabled: true
    })
  )
);
```

### Custom Content Templates

```typescript
import { ContentTemplate } from './types';

const customTemplate: ContentTemplate = {
  name: 'product-review',
  structure: {
    sections: [
      { type: 'introduction', wordCount: 150 },
      { type: 'pros-cons', format: 'list' },
      { type: 'use-cases', wordCount: 300 },
      { type: 'conclusion', wordCount: 100 }
    ]
  },
  tone: 'balanced',
  aiPrompt: `Create a balanced product review...`
};

const content = await generateContent({
  topic: 'Review of Tool X',
  template: customTemplate,
  researchData: researchData
});
```

### Video Customization

```typescript
import { VideoComposition } from './remotion/compositions';

const customVideo = await renderVideo({
  content: content,
  composition: {
    scenes: [
      { type: 'intro', duration: 3, text: content.title.en },
      { type: 'points', duration: 45, items: content.keyPoints },
      { type: 'cta', duration: 5, message: 'Follow for more' }
    ],
    transitions: 'fade',
    bgMusic: './assets/background.mp3',
    voiceover: {
      enabled: true,
      provider: 'elevenlabs',
      voice: 'professional-male'
    }
  }
});
```

## Troubleshooting

### Research Crawling Issues

```typescript
// Handle rate limiting
const researchData = await crawlNewsSources({
  keyword: 'AI',
  sources: ['techcrunch'],
  retryOptions: {
    maxRetries: 3,
    backoff: 'exponential',
    initialDelay: 1000
  }
});
```

### AI Generation Errors

```typescript
try {
  const content = await generateContent({ ... });
} catch (error) {
  if (error.code === 'rate_limit_exceeded') {
    // Switch to backup AI provider
    const content = await generateContent({
      ...options,
      aiProvider: 'openai' // fallback
    });
  }
  if (error.code === 'content_policy_violation') {
    // Adjust prompt or content
    console.error('Content flagged by AI safety filters');
  }
}
```

### Video Rendering Failures

```typescript
// Monitor rendering progress
const videoPath = await renderVideo({
  content: content,
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  },
  timeout: 300000, // 5 minutes
  fallbackTemplate: 'simple' // Use if main template fails
});
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processLargeKeywordList(keywords: string[]) {
  const chunkSize = 5;
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    await Promise.all(chunk.map(k => pipeline.run({ keyword: k })));
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
}
```

## Best Practices

1. **Research Quality**: Use multiple sources and recent data (< 24h) for trending topics
2. **AI Provider Selection**: Use Claude for creative/nuanced content, OpenAI for structured formats
3. **Video Optimization**: Match aspect ratio and duration to target platform
4. **Scheduling**: Publish during optimal engagement times for each platform
5. **Monitoring**: Track pipeline metrics (success rate, generation time, engagement)

## Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese setup guide
- API documentation in `/docs` folder
- Example templates in `/templates` directory
- Video compositions in `/remotion/compositions`
