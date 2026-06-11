---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing content pipeline with Claude and OpenAI
  - generate videos automatically from content
  - create AI-powered content automation system
  - research and generate content with AI agents
  - build automated marketing pipeline with TypeScript
  - how to use Remotion for video generation in content pipeline
  - automate content research and video creation
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content creation workflow from research to video generation. It:

- **Auto-crawls** trending news from sources like TechCrunch, a16z, X (Twitter), and LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Supports multi-language** output (English and Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following required variables:

```bash
# AI Service APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Key API Usage Patterns

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import { fetchTrendingNews } from './api/research';

interface ResearchParams {
  keyword: string;
  sources?: string[];
  timeframe?: '24h' | '7d' | '30d';
}

async function performResearch(params: ResearchParams) {
  const { keyword, sources = ['techcrunch', 'a16z', 'twitter'], timeframe = '24h' } = params;
  
  const newsData = await fetchTrendingNews({
    query: keyword,
    sources,
    timeframe,
    apiKey: process.env.RAPIDAPI_KEY
  });
  
  return newsData;
}

// Usage
const research = await performResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'linkedin'],
  timeframe: '24h'
});
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const { topic, format, language, tone, researchData } = request;
  
  const systemPrompt = `You are an expert content creator. Generate a ${format} article about ${topic} in ${language} language with a ${tone} tone.`;
  
  const userPrompt = `
    Topic: ${topic}
    Format: ${format}
    Research Data: ${JSON.stringify(researchData, null, 2)}
    
    Create a comprehensive article that:
    1. Uses the latest research data provided
    2. Includes specific examples and statistics
    3. Follows the ${format} structure
    4. Maintains a ${tone} tone throughout
  `;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: userPrompt }
    ],
    system: systemPrompt
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const article = await generateContent({
  topic: 'AI Content Automation Trends',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research
});
```

### 3. OpenAI Alternative Pattern

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, systemRole: string): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemRole },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const { content, title, format } = config;
  
  // Video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion', 'index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      ...dimensions[format]
    },
  });
  
  // Render video
  const outputLocation = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content,
    },
  });
  
  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  content: article,
  title: 'AI Content Automation Trends',
  format: 'reels'
});
```

### 5. Complete Pipeline Integration

```typescript
// lib/pipeline/content-pipeline.ts
import { performResearch } from './research/crawler';
import { generateContent } from './ai/content-generator';
import { renderContentVideo } from './video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const researchData = await performResearch({
      keyword: config.keyword,
      timeframe: '24h'
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: config.keyword,
      format: config.contentFormat,
      language: config.language,
      tone: config.tone,
      researchData
    });
    
    // Step 3: Generate Video (optional)
    let videoPath = null;
    if (config.generateVideo && config.videoFormat) {
      console.log('🎬 Rendering video...');
      videoPath = await renderContentVideo({
        content,
        title: config.keyword,
        format: config.videoFormat
      });
    }
    
    return {
      success: true,
      content,
      videoPath,
      research: researchData
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2026',
  contentFormat: 'toplist',
  language: 'en',
  tone: 'expert',
  generateVideo: true,
  videoFormat: 'reels'
});
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, tone, generateVideo, videoFormat } = req.body;
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: format,
      language,
      tone,
      generateVideo,
      videoFormat
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Common Patterns & Best Practices

### 1. Error Handling with Retry Logic

```typescript
// lib/utils/retry.ts
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => generateContent(config), 3, 2000);
```

### 2. Rate Limiting for API Calls

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;
  
  constructor(requestsPerMinute: number) {
    this.delay = (60 * 1000) / requestsPerMinute;
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
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
const result = await limiter.add(() => generateContent(config));
```

### 3. Caching Research Results

```typescript
// lib/cache/research-cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached;
}

async function cacheResearch(keyword: string, data: any, ttl: number = 3600) {
  await redis.setex(`research:${keyword}`, ttl, JSON.stringify(data));
}

// Usage in research function
async function performResearchWithCache(keyword: string) {
  const cached = await getCachedResearch(keyword);
  if (cached) return JSON.parse(cached as string);
  
  const fresh = await performResearch({ keyword });
  await cacheResearch(keyword, fresh);
  return fresh;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement exponential backoff
async function generateWithBackoff(config: ContentRequest) {
  let delay = 1000;
  const maxDelay = 32000;
  
  while (delay <= maxDelay) {
    try {
      return await generateContent(config);
    } catch (error: any) {
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2;
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retry delay exceeded');
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for long renders
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    gl: 'angle',
  },
});
```

### Issue: Memory Issues with Large Content

```typescript
// Stream large content instead of loading all at once
import { createWriteStream } from 'fs';

async function streamContentGeneration(config: ContentRequest, outputPath: string) {
  const stream = createWriteStream(outputPath);
  
  const content = await generateContent(config);
  stream.write(content);
  stream.end();
  
  return outputPath;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview (if applicable)
npm run remotion:preview
```

## Project Structure

```
marketing-pineline-share/
├── pages/
│   ├── api/
│   │   ├── generate-content.ts
│   │   └── research.ts
│   └── index.tsx
├── lib/
│   ├── ai/
│   │   ├── content-generator.ts
│   │   └── openai-generator.ts
│   ├── research/
│   │   └── crawler.ts
│   ├── video/
│   │   └── renderer.ts
│   └── pipeline/
│       └── content-pipeline.ts
├── remotion/
│   ├── index.ts
│   └── compositions/
│       └── ContentVideo.tsx
└── .env.local
```
