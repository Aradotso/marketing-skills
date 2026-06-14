---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, auto-posting, and video generation
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing pipeline with Claude and OpenAI
  - create auto-posting content system with video rendering
  - build AI content pipeline from research to publication
  - configure automated content workflow with Remotion video
  - integrate multi-platform content automation system
  - setup AI-driven content research and scriptwriting
  - deploy end-to-end marketing automation pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and data crawling, through AI-powered scriptwriting, to automated video generation and multi-platform publishing.

## What This Project Does

The Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-crawls** real-time data from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Supports bilingual output** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for Reels, TikTok, Shorts
- **Auto-posts** to multiple platforms with scheduling capabilities

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Core Architecture

The pipeline consists of four main modules:

1. **Research Module** - Data crawling and analysis
2. **Content Generation Module** - AI-powered writing
3. **Video Rendering Module** - Remotion integration
4. **Publishing Module** - Multi-platform posting

## Key API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { ResearchCrawler } from '@/lib/research/crawler';
import { DataAnalyzer } from '@/lib/research/analyzer';

// Initialize crawler
const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing automation']
});

// Crawl data
const rawData = await crawler.crawl();

// Analyze and extract insights
const analyzer = new DataAnalyzer();
const insights = await analyzer.extractInsights(rawData, {
  minRelevance: 0.7,
  includeStats: true,
  language: 'both' // en + vi
});

console.log(insights);
```

### 2. AI Content Generation with Claude

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';
import { ContentFormat } from '@/types/content';

// Initialize Claude generator
const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: 'claude-3-opus-20240229'
});

// Generate content
const content = await generator.generate({
  topic: 'AI Marketing Automation Trends 2024',
  format: ContentFormat.TOPLIST,
  language: 'vi',
  tone: 'professional',
  insights: insights, // from research
  wordCount: 1500,
  includeStats: true
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. OpenAI Alternative

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai-generator';

const generator = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY!,
  model: 'gpt-4-turbo-preview'
});

const content = await generator.generate({
  topic: 'Case Study: AI Content Pipeline Success',
  format: ContentFormat.CASE_STUDY,
  language: 'en',
  tone: 'friendly',
  insights: insights,
  customPrompt: 'Focus on ROI and time savings'
});
```

### 4. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { VideoTemplate } from '@/lib/video/templates';

// Initialize renderer
const renderer = new VideoRenderer({
  licenseKey: process.env.REMOTION_LICENSE_KEY!,
  outputDir: './output/videos'
});

// Render video from content
const video = await renderer.render({
  template: VideoTemplate.REELS,
  content: {
    title: content.title,
    keyPoints: content.keyPoints,
    stats: content.stats
  },
  style: {
    aspectRatio: '9:16', // For Reels/TikTok
    duration: 30,
    theme: 'modern',
    brandColors: ['#FF6B6B', '#4ECDC4']
  }
});

console.log(`Video rendered: ${video.path}`);
```

### 5. Multi-Platform Publishing

```typescript
import { ContentPublisher } from '@/lib/publisher/publisher';
import { Platform } from '@/types/platform';

const publisher = new ContentPublisher({
  platforms: {
    facebook: {
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN!,
      pageId: process.env.FACEBOOK_PAGE_ID!
    },
    linkedin: {
      accessToken: process.env.LINKEDIN_ACCESS_TOKEN!,
      organizationId: process.env.LINKEDIN_ORG_ID!
    }
  }
});

// Schedule or immediate publish
await publisher.publish({
  content: content.body,
  platforms: [Platform.FACEBOOK, Platform.LINKEDIN],
  media: [video.path],
  schedule: new Date('2024-06-15T10:00:00Z'), // Optional
  tags: ['AI', 'Marketing', 'Automation']
});
```

## Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize full pipeline
const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  },
  ai: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  },
  video: {
    enabled: true,
    templates: ['reels', 'shorts']
  },
  publishing: {
    platforms: ['facebook', 'linkedin', 'tiktok'],
    autoSchedule: true
  }
});

// Run complete automation
const result = await pipeline.run({
  keywords: ['AI marketing', 'automation tools'],
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  outputCount: 3 // Generate 3 pieces of content
});

console.log(`Generated ${result.contents.length} contents`);
console.log(`Rendered ${result.videos.length} videos`);
console.log(`Published to ${result.publishedCount} platforms`);
```

## Next.js API Route Example

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { topic, format, language, generateVideo } = req.body;

    const pipeline = new ContentPipeline({
      research: { sources: ['techcrunch', 'a16z'] },
      ai: { provider: 'claude' },
      video: { enabled: generateVideo }
    });

    const result = await pipeline.run({
      keywords: [topic],
      contentFormats: [format],
      languages: [language],
      outputCount: 1
    });

    res.status(200).json({
      success: true,
      content: result.contents[0],
      video: result.videos[0] || null
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Content generation failed',
      message: error.message 
    });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
import { BatchProcessor } from '@/lib/batch-processor';

const batch = new BatchProcessor({
  concurrency: 3, // Process 3 at a time
  retryOnError: true,
  maxRetries: 2
});

const topics = [
  'AI Trends 2024',
  'Marketing Automation ROI',
  'Content Strategy Guide'
];

const results = await batch.process(topics, async (topic) => {
  return await pipeline.run({
    keywords: [topic],
    contentFormats: ['toplist'],
    languages: ['en'],
    outputCount: 1
  });
});
```

### Custom Content Templates

```typescript
import { TemplateEngine } from '@/lib/templates/engine';

const templateEngine = new TemplateEngine();

// Register custom template
templateEngine.register('product-launch', {
  structure: [
    'hook',
    'problem-statement',
    'solution-intro',
    'features-benefits',
    'social-proof',
    'cta'
  ],
  tone: 'exciting',
  length: 'medium'
});

// Use custom template
const content = await generator.generate({
  topic: 'New AI Tool Launch',
  template: 'product-launch',
  customData: {
    productName: 'ContentAI Pro',
    features: ['Auto-research', 'Video gen', 'Multi-platform']
  }
});
```

### Webhook Integration for Publishing

```typescript
import { WebhookPublisher } from '@/lib/publisher/webhook';

const webhookPublisher = new WebhookPublisher({
  endpoints: {
    wordpress: process.env.WORDPRESS_WEBHOOK_URL!,
    zapier: process.env.ZAPIER_WEBHOOK_URL!
  }
});

await webhookPublisher.send('wordpress', {
  title: content.title,
  body: content.body,
  status: 'publish',
  categories: ['AI', 'Marketing']
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1,
  provider: 'claude'
});

// Wrap API calls
await limiter.execute(async () => {
  return await generator.generate(config);
});
```

### Error Handling

```typescript
import { PipelineError, ErrorType } from '@/lib/errors';

try {
  await pipeline.run(config);
} catch (error) {
  if (error instanceof PipelineError) {
    switch (error.type) {
      case ErrorType.API_LIMIT:
        console.log('Rate limit hit, retry in 60s');
        break;
      case ErrorType.INVALID_CONTENT:
        console.log('Content validation failed');
        break;
      case ErrorType.RENDER_FAILED:
        console.log('Video rendering error');
        break;
      default:
        console.error('Unknown error:', error.message);
    }
  }
}
```

### Debug Mode

```typescript
const pipeline = new ContentPipeline({
  debug: true,
  logLevel: 'verbose',
  onProgress: (stage, progress) => {
    console.log(`[${stage}] ${progress}%`);
  }
});
```

### Cache Management

```typescript
import { CacheManager } from '@/lib/cache';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  provider: 'redis'
});

// Cache research results
const insights = await cache.get(`insights:${topic}`, async () => {
  return await analyzer.extractInsights(rawData);
});
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run tests
npm run test

# Lint code
npm run lint

# Type check
npm run type-check

# Render video (Remotion CLI)
npm run render -- --props='{"title":"Test Video"}'
```

## Configuration Files

### `pipeline.config.ts`

```typescript
export const pipelineConfig = {
  research: {
    defaultSources: ['techcrunch', 'a16z'],
    crawlDepth: 2,
    maxArticles: 50
  },
  ai: {
    defaultProvider: 'claude',
    fallbackProvider: 'openai',
    temperature: 0.7,
    maxTokens: 4000
  },
  video: {
    defaultTemplate: 'reels',
    fps: 30,
    quality: 'high'
  },
  publishing: {
    autoSchedule: false,
    timezone: 'Asia/Ho_Chi_Minh'
  }
};
```

This skill enables AI coding agents to effectively utilize the Marketing Pipeline Share system for comprehensive content automation workflows.
