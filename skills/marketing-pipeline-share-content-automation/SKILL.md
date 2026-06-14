---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content from research to video
  - create automated content workflow with Claude
  - build AI-powered content generation system
  - set up automatic blog and video creation
  - implement content automation with Remotion
  - use marketing pipeline for content research
  - automate social media content generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from research and scripting to automatic posting and video generation using Claude 3, OpenAI, and Remotion.

## What It Does

Marketing Pipeline Share automates:
- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (24h)
- **AI Content Generation**: Creates multiple formats (Toplist, POV, Case Study, How-to) in multiple languages using Claude/OpenAI
- **Video Rendering**: Generates infographics and short videos using Remotion
- **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

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
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Research crawlers
│   │   └── video/       # Remotion video generation
│   ├── services/         # Business logic services
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion:render
```

## Core API Usage

### 1. Research Module

Auto-crawl and analyze content from multiple sources:

```typescript
import { ResearchService } from '@/services/research';

const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Crawl fresh content by keyword
const researchData = await researchService.crawlByKeyword({
  keyword: 'AI Marketing',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h',
  limit: 50,
});

// Extract insights
const insights = await researchService.extractInsights(researchData);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const generator = new ContentGenerator(anthropic);

// Generate content in multiple formats
const content = await generator.generate({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  language: 'vi', // or 'en'
  tone: 'expert', // or 'friendly', 'humorous'
  researchData: insights,
  wordCount: 1500,
});

// Dual language generation
const bilingualContent = await generator.generateBilingual({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist',
  researchData: insights,
});
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';
import { ContentGenerator } from '@/lib/ai/generator';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const generator = new ContentGenerator(openai, { provider: 'openai' });

const content = await generator.generate({
  topic: 'Social Media Automation',
  format: 'how-to',
  language: 'en',
  model: 'gpt-4-turbo-preview',
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from content
async function renderContentVideo(content: GeneratedContent) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      style: 'modern',
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.defaultProps,
  });
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/services/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
  remotionLicense: process.env.REMOTION_LICENSE_KEY,
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI Content Marketing',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPost: false, // Set to true for automatic posting
});

console.log('Generated content:', result.content);
console.log('Video output:', result.videoPath);
console.log('Insights used:', result.insights);
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';
import { ContentPipeline } from '@/services/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI Marketing', 'Content Automation', 'Social Media Trends'];
  
  for (const keyword of keywords) {
    await pipeline.execute({
      keyword,
      formats: ['toplist'],
      languages: ['en'],
      generateVideo: true,
      autoPost: true,
    });
  }
});
```

### Pattern 2: Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/ai/templates';

const customTemplate = new ContentTemplate({
  name: 'product-launch',
  structure: {
    sections: [
      { type: 'hook', tone: 'exciting' },
      { type: 'problem', tone: 'empathetic' },
      { type: 'solution', tone: 'confident' },
      { type: 'features', format: 'bullets' },
      { type: 'cta', tone: 'urgent' },
    ],
  },
});

const content = await generator.generate({
  topic: 'New AI Tool Launch',
  template: customTemplate,
  researchData: insights,
});
```

### Pattern 3: Multi-Platform Video Export

```typescript
import { VideoExporter } from '@/lib/video/exporter';

const exporter = new VideoExporter();

// Export for multiple platforms
await exporter.exportMultiPlatform({
  sourceVideo: './out/content-video.mp4',
  platforms: [
    { name: 'tiktok', aspectRatio: '9:16', maxDuration: 60 },
    { name: 'youtube-shorts', aspectRatio: '9:16', maxDuration: 60 },
    { name: 'instagram-reels', aspectRatio: '9:16', maxDuration: 90 },
    { name: 'linkedin', aspectRatio: '1:1', maxDuration: 120 },
  ],
  outputDir: './out/multi-platform/',
});
```

### Pattern 4: Batch Content Processing

```typescript
import { BatchProcessor } from '@/services/batch';

const processor = new BatchProcessor({
  concurrency: 3,
  retryAttempts: 2,
});

const topics = [
  'AI in Healthcare',
  'Future of E-commerce',
  'Sustainable Marketing',
];

const results = await processor.processBatch(topics, async (topic) => {
  return await pipeline.execute({
    keyword: topic,
    formats: ['pov', 'case-study'],
    languages: ['en', 'vi'],
  });
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/services/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();

  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  try {
    const result = await pipeline.execute({
      keyword,
      formats: [format],
      languages: [language],
      generateVideo: false,
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Options

### Content Generator Config

```typescript
interface GeneratorConfig {
  provider: 'anthropic' | 'openai';
  model?: string; // 'claude-3-opus-20240229' | 'gpt-4-turbo-preview'
  temperature?: number; // 0-1, default 0.7
  maxTokens?: number; // default 4000
  systemPrompt?: string;
}
```

### Research Config

```typescript
interface ResearchConfig {
  sources: ('techcrunch' | 'twitter' | 'linkedin' | 'a16z')[];
  timeRange: '24h' | '7d' | '30d';
  limit: number;
  includeMetrics?: boolean;
  filterKeywords?: string[];
}
```

### Video Config

```typescript
interface VideoConfig {
  resolution: '1080p' | '720p' | '480p';
  fps: 30 | 60;
  codec: 'h264' | 'h265';
  bitrate?: string; // e.g., '5M'
  aspectRatio: '16:9' | '9:16' | '1:1';
  duration?: number; // max seconds
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 60,
  retryAfter: 5000,
});

const content = await limiter.execute(() =>
  generator.generate({ topic: 'AI Marketing' })
);
```

### Video Rendering Errors

```typescript
// Check Remotion license
if (!process.env.REMOTION_LICENSE_KEY) {
  console.warn('Remotion license not found, using free tier');
}

// Handle rendering timeouts
const renderWithTimeout = async (timeout = 300000) => {
  return Promise.race([
    renderContentVideo(content),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Render timeout')), timeout)
    ),
  ]);
};
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatchInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Claude/OpenAI Connection Issues

```typescript
import { retry } from '@/lib/utils/retry';

const generateWithRetry = async (config: GenerateConfig) => {
  return retry(
    async () => await generator.generate(config),
    {
      maxAttempts: 3,
      delay: 2000,
      backoff: 'exponential',
    }
  );
};
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Use environment variables** for all sensitive data
3. **Implement rate limiting** for API calls
4. **Cache research data** to avoid redundant crawls
5. **Monitor token usage** to control costs
6. **Queue video rendering** for better resource management
7. **Version control your templates** for consistent output
8. **Log all pipeline executions** for debugging

This skill enables AI agents to effectively assist developers in building automated content marketing systems using this comprehensive pipeline.
