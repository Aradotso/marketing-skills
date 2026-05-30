---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and OpenAI
  - generate automated content from research to video
  - create AI-powered content workflow with Remotion
  - build automated marketing content system
  - configure content automation with research and video rendering
  - implement end-to-end content generation pipeline
  - use AI to research write and render marketing content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation using Claude, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-researches** trending topics by crawling TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates multilingual content** (English & Vietnamese) in various formats (Top Lists, POV, Case Studies, How-to)
- **Renders videos automatically** using Remotion to create platform-optimized content for Reels, TikTok, Shorts
- **Integrates multiple AI providers** (Claude 3, OpenAI) for content generation
- **Provides a Next.js interface** for managing and scheduling content publication

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn/pnpm required
node --version  # Should be 18.x or higher
```

### Setup Steps

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

# Copy environment template
cp .env.example .env.local
```

### Environment Configuration

Edit `.env.local` with required API keys:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Components & API

### 1. Research Module

Automatically crawls and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research/crawler';

interface ResearchOptions {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi' | 'both';
}

async function performResearch(options: ResearchOptions) {
  const crawler = new ResearchCrawler({
    apiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await crawler.search({
    keyword: options.keyword,
    sources: options.sources,
    timeframe: options.timeframe,
  });

  return crawler.extractInsights(results);
}

// Usage
const insights = await performResearch({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h',
  language: 'both',
});
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

// Using Claude
async function generateWithClaude(request: ContentRequest) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create a ${request.format} article about ${request.topic} in ${request.language} with a ${request.tone} tone. Use this research data: ${JSON.stringify(request.researchData)}`,
      },
    ],
  });

  return message.content[0].text;
}

// Using OpenAI
async function generateWithOpenAI(request: ContentRequest) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} articles with a ${request.tone} tone.`,
      },
      {
        role: 'user',
        content: `Write about ${request.topic} in ${request.language} using this data: ${JSON.stringify(request.researchData)}`,
      },
    ],
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
  style: 'minimal' | 'dynamic' | 'infographic';
}

async function renderContentVideo(config: VideoConfig) {
  // Define dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositionId = `${config.style}-template`;
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      duration: config.duration,
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `output-${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
    },
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  content: generatedArticle,
  format: 'reels',
  duration: 30,
  style: 'infographic',
});
```

### 4. Complete Pipeline Workflow

End-to-end automation example:

```typescript
// lib/pipeline/content-automation.ts
import { performResearch } from '@/lib/research/crawler';
import { generateWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  videoFormats: ('reels' | 'tiktok' | 'shorts')[];
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const research = await performResearch({
      keyword: config.keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h',
      language: 'both',
    });

    const results = [];

    // Step 2: Generate content for each language
    for (const language of config.languages) {
      console.log(`✍️ Generating ${language} content...`);
      
      const content = await generateWithClaude({
        topic: config.keyword,
        format: config.contentFormat,
        tone: config.tone,
        language,
        researchData: research,
      });

      // Step 3: Render videos for each format
      const videos = [];
      for (const format of config.videoFormats) {
        console.log(`🎬 Rendering ${format} video for ${language}...`);
        
        const videoPath = await renderContentVideo({
          content,
          format,
          duration: 30,
          style: 'infographic',
        });

        videos.push({ format, path: videoPath });
      }

      results.push({
        language,
        content,
        videos,
      });
    }

    console.log('✅ Pipeline completed successfully!');
    return results;

  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const output = await runContentPipeline({
  keyword: 'AI marketing trends 2024',
  contentFormat: 'toplist',
  tone: 'expert',
  languages: ['en', 'vi'],
  videoFormats: ['reels', 'tiktok'],
});
```

## Next.js API Routes

Create serverless endpoints for the pipeline:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-automation';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, contentFormat, tone, languages, videoFormats } = body;

    // Validate input
    if (!keyword || !contentFormat) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const results = await runContentPipeline({
      keyword,
      contentFormat,
      tone: tone || 'expert',
      languages: languages || ['en'],
      videoFormats: videoFormats || ['reels'],
    });

    return NextResponse.json({
      success: true,
      data: results,
    });

  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron-jobs.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/content-automation';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running scheduled content generation...');
  
  const topics = ['AI trends', 'Marketing automation', 'Social media tips'];
  
  for (const topic of topics) {
    await runContentPipeline({
      keyword: topic,
      contentFormat: 'toplist',
      tone: 'friendly',
      languages: ['en', 'vi'],
      videoFormats: ['reels'],
    });
  }
});
```

### Pattern 2: Batch Processing

```typescript
// lib/batch/processor.ts
interface BatchJob {
  id: string;
  keywords: string[];
  config: Partial<PipelineConfig>;
}

async function processBatch(job: BatchJob) {
  const results = [];
  
  for (const keyword of job.keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: job.config.contentFormat || 'toplist',
      tone: job.config.tone || 'expert',
      languages: job.config.languages || ['en'],
      videoFormats: job.config.videoFormats || ['reels'],
    });
    
    results.push({ keyword, ...result });
    
    // Add delay to avoid rate limits
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

### Pattern 3: Custom Templates

```typescript
// remotion/templates/custom-template.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface TemplateProps {
  content: string;
  title: string;
  accent: string;
}

export const CustomTemplate: React.FC<TemplateProps> = ({
  content,
  title,
  accent,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <h1
        style={{
          color: accent,
          fontSize: 80,
          opacity,
          textAlign: 'center',
          padding: 40,
        }}
      >
        {title}
      </h1>
      <p
        style={{
          color: '#fff',
          fontSize: 40,
          maxWidth: 800,
          textAlign: 'center',
          padding: 40,
        }}
      >
        {content}
      </p>
    </AbsoluteFill>
  );
};
```

## Configuration Files

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;

    this.processing = true;
    const task = this.queue.shift();

    if (task) {
      await task();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }

    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute

await limiter.add(() => generateWithClaude(request));
```

### Issue: Remotion Rendering Memory Errors

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Issue: Missing Environment Variables

```typescript
// lib/utils/env-validator.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY',
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call in app startup
validateEnv();
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and retry logic
async function renderWithRetry(config: VideoConfig, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await renderContentVideo(config);
    } catch (error) {
      console.error(`Render attempt ${attempt} failed:`, error);
      
      if (attempt === maxRetries) throw error;
      
      // Wait before retry (exponential backoff)
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Use rate limiters** when working with multiple API providers
3. **Cache research results** for 24 hours to reduce API calls
4. **Process video rendering in background jobs** for better UX
5. **Monitor API costs** by logging token usage for Claude/OpenAI
6. **Test with small batches** before running large-scale automation
7. **Store generated content** in a database for reuse and analytics
