---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, script generation, and automated video creation using Claude/OpenAI
triggers:
  - automate content creation with AI research
  - generate social media videos automatically
  - create multilingual marketing content
  - build AI content pipeline workflow
  - scrape news and generate articles
  - render videos from text content
  - set up automated content publishing
  - create AI-driven content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an all-in-one TypeScript-based content automation system that handles research, scriptwriting, and video generation. The pipeline crawls fresh data from sources like TechCrunch, Twitter/X, and LinkedIn, generates multilingual content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from news sources and social media (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Multi-Platform Export**: Outputs optimized video formats for Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Source APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Application Settings
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

### Start Development Server

```bash
npm run dev
```

The application will be available at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── config/              # Configuration files
```

## Core APIs and Usage

### 1. Content Research Module

Scrape and analyze recent content from multiple sources:

```typescript
import { ContentResearcher } from '@/lib/scraper/researcher';

async function researchTopic(keyword: string) {
  const researcher = new ContentResearcher({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  const insights = await researcher.gather({
    keyword,
    limit: 20,
    language: 'en'
  });

  return insights;
}

// Usage
const data = await researchTopic('AI automation');
console.log(data.articles); // Array of scraped articles
console.log(data.insights); // Extracted insights and trends
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import { Anthropic } from '@anthropic-ai/sdk';

async function generateArticle(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const generator = new ContentGenerator(anthropic);

  const content = await generator.create({
    topic,
    format,
    tone: 'professional', // or 'friendly', 'humorous'
    languages: ['en', 'vi'],
    researchData: await researchTopic(topic)
  });

  return content;
}

// Usage
const article = await generateArticle('AI in Marketing 2026', 'toplist');
console.log(article.en.title);
console.log(article.en.body);
console.log(article.vi.title); // Vietnamese version
console.log(article.vi.body);
```

### 3. OpenAI Alternative

Switch to OpenAI instead of Claude:

```typescript
import OpenAI from 'openai';
import { ContentGenerator } from '@/lib/ai/generator';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

const generator = new ContentGenerator(openai, { provider: 'openai' });

const content = await generator.create({
  topic: 'Social Media Trends',
  format: 'how-to',
  model: 'gpt-4-turbo'
});
```

### 4. Video Rendering with Remotion

Render videos from generated content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function renderContentVideo(content: any) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 60 // seconds
    }
  });

  // Render video
  const output = `./output/${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: output,
    imageFormat: 'jpeg'
  });

  return output;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Tools for 2026',
  keyPoints: ['Tool 1: Description', 'Tool 2: Description']
});
```

### 5. Full Pipeline Execution

Run the complete automation pipeline:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'anthropic', // or 'openai'
    languages: ['en', 'vi'],
    generateVideo: true
  });

  const result = await pipeline.execute({
    keyword,
    format: 'toplist',
    videoFormat: 'reels', // or 'tiktok', 'shorts'
    autoPublish: false // set true to auto-post
  });

  return {
    articles: result.content,
    videos: result.videos,
    analytics: result.metrics
  };
}

// Usage
const output = await runFullPipeline('Marketing Automation 2026');
console.log(`Generated ${output.articles.length} articles`);
console.log(`Rendered ${output.videos.length} videos`);
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

Create a scheduled job to generate content daily:

```typescript
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline';

// Run every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Social media'];
  
  for (const keyword of keywords) {
    const pipeline = new ContentPipeline();
    await pipeline.execute({
      keyword,
      format: 'toplist',
      autoPublish: true
    });
  }
});
```

### Pattern 2: Batch Processing Multiple Topics

Process multiple topics in parallel:

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => 
      generateArticle(topic, 'pov')
    )
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  return successful;
}

// Usage
const articles = await batchGenerate([
  'AI Marketing Tools',
  'Content Automation',
  'Video Marketing Trends'
]);
```

### Pattern 3: Custom Tone Configuration

Create content with custom brand voice:

```typescript
import { ContentGenerator } from '@/lib/ai/generator';

const generator = new ContentGenerator(anthropic, {
  customPrompt: `
    You are a marketing expert writing for tech-savvy entrepreneurs.
    Use conversational tone, include data-backed insights, and focus on actionable takeaways.
    Always include 3-5 specific examples.
  `,
  maxTokens: 4000
});

const content = await generator.create({
  topic: 'AI Content Tools',
  format: 'case-study'
});
```

### Pattern 4: Multi-Platform Video Export

Generate optimized videos for different platforms:

```typescript
async function generateMultiPlatformVideos(content: any) {
  const platforms = [
    { name: 'reels', aspectRatio: '9:16', duration: 60 },
    { name: 'tiktok', aspectRatio: '9:16', duration: 60 },
    { name: 'youtube-shorts', aspectRatio: '9:16', duration: 60 },
    { name: 'linkedin', aspectRatio: '1:1', duration: 90 }
  ];

  const videos = await Promise.all(
    platforms.map(platform => 
      renderContentVideo({
        ...content,
        platform: platform.name,
        aspectRatio: platform.aspectRatio,
        maxDuration: platform.duration
      })
    )
  );

  return videos;
}
```

## Configuration

### AI Provider Settings

Configure AI model parameters in `config/ai.ts`:

```typescript
export const aiConfig = {
  anthropic: {
    model: 'claude-3-sonnet-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo',
    maxTokens: 4096,
    temperature: 0.7
  }
};
```

### Scraper Configuration

Set up data sources in `config/scraper.ts`:

```typescript
export const scraperConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      rateLimit: 10, // requests per minute
      categories: ['ai', 'startups', 'apps']
    },
    twitter: {
      enabled: true,
      hashtags: ['#AI', '#Marketing', '#ContentCreation'],
      userList: ['@elonmusk', '@sama', '@paulg']
    },
    linkedin: {
      enabled: true,
      influencers: ['andrew-ng', 'satya-nadella']
    }
  },
  cacheTimeout: 3600 // 1 hour in seconds
};
```

### Video Rendering Settings

Configure Remotion in `remotion.config.ts`:

```typescript
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

export const videoConfig = {
  fps: 30,
  dimensions: {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 }
  }
};
```

## API Endpoints

If using the Next.js API routes:

### POST /api/generate

Generate content for a topic:

```typescript
// Request
fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI Marketing',
    format: 'toplist',
    languages: ['en', 'vi']
  })
});

// Response
{
  "success": true,
  "data": {
    "en": { "title": "...", "body": "..." },
    "vi": { "title": "...", "body": "..." }
  }
}
```

### POST /api/render-video

Render video from content:

```typescript
fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: '123',
    platform: 'reels'
  })
});

// Response
{
  "success": true,
  "videoUrl": "https://your-cdn.com/video.mp4"
}
```

## Troubleshooting

### Issue: API Rate Limiting

If you hit rate limits on research APIs:

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMilliseconds: 60000 // 10 requests per minute
});

await limiter.schedule(() => researcher.gather({ keyword }));
```

### Issue: Video Rendering Timeout

For long videos, increase timeout:

```typescript
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: output,
  timeoutInMilliseconds: 300000 // 5 minutes
});
```

### Issue: Out of Memory During Video Render

Reduce concurrency and dimensions:

```typescript
Config.setConcurrency(1);

// Use lower resolution
const composition = await selectComposition({
  serveUrl: bundled,
  id: 'ContentVideo',
  inputProps: {
    ...props,
    width: 720,
    height: 1280
  }
});
```

### Issue: AI Generation Quality

Improve prompts and add context:

```typescript
const content = await generator.create({
  topic,
  format: 'toplist',
  context: {
    targetAudience: 'B2B marketers',
    industry: 'SaaS',
    tone: 'data-driven and authoritative',
    examples: true
  }
});
```

### Issue: Scraper Blocked

Rotate user agents and add delays:

```typescript
const researcher = new ContentResearcher({
  apiKey: process.env.RAPIDAPI_KEY,
  userAgent: 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  delayBetweenRequests: 2000 // 2 seconds
});
```

## Testing

Run tests with:

```bash
npm test
```

Test specific modules:

```typescript
import { describe, it, expect } from 'vitest';
import { ContentGenerator } from '@/lib/ai/generator';

describe('ContentGenerator', () => {
  it('generates toplist content', async () => {
    const generator = new ContentGenerator(mockAI);
    const result = await generator.create({
      topic: 'Test',
      format: 'toplist'
    });
    
    expect(result.en.title).toBeDefined();
    expect(result.en.body).toContain('1.');
  });
});
```

This skill provides comprehensive coverage of the marketing-pipeline-share automation system for AI coding agents to effectively assist developers in implementing automated content creation workflows.
