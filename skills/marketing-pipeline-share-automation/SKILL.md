---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content automation pipeline
  - generate videos from content automatically
  - research and create social media posts with AI
  - use marketing pipeline share for content
  - automate content from research to video
  - create multilingual content with AI pipeline
  - build automated content workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that handles the complete content lifecycle: research → scriptwriting → publishing → video generation. It crawls fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

**Key Capabilities:**
- Auto-crawl research from news sources (last 24h)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual output (English/Vietnamese)
- Automated video/infographic rendering with Remotion
- Next.js web interface for content management

## Installation

### Prerequisites

```bash
# Required tools
node >= 18.0.0
npm >= 9.0.0
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Development Server

```bash
# Start the development server
npm run dev

# Open browser to http://localhost:3000
```

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

## Core Architecture

### Project Structure

```
marketing-pipeline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/
│   ├── ai/               # AI service integrations
│   ├── crawler/          # Research crawlers
│   ├── content/          # Content generation
│   └── video/            # Remotion video generation
├── public/               # Static assets
└── remotion/             # Video templates
```

## Content Generation Pipeline

### 1. Research & Crawling

```typescript
import { crawlRecentNews } from '@/lib/crawler/news-scraper';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

async function gatherResearch(keyword: string) {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.article' },
    { name: 'a16z', url: 'https://a16z.com/blog', selector: '.post' }
  ];

  const articles = await crawlRecentNews({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 10
  });

  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

function buildPrompt(request: ContentRequest): string {
  const formatTemplates = {
    'toplist': `Create a top 10 list about ${request.topic}...`,
    'pov': `Write a POV perspective on ${request.topic}...`,
    'case-study': `Analyze a case study about ${request.topic}...`,
    'how-to': `Create a step-by-step guide on ${request.topic}...`
  };

  return `
    Format: ${formatTemplates[request.format]}
    Language: ${request.language}
    Tone: ${request.tone}
    Research Data: ${JSON.stringify(request.researchData)}
    
    Include data-backed insights and recent trends.
  `;
}
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(request: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.tone} content.`
      },
      {
        role: 'user',
        content: buildPrompt(request)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

## Video Generation with Remotion

### Video Composition Setup

```typescript
// remotion/ContentVideo.tsx
import { Composition } from 'remotion';
import { ContentSlide } from './ContentSlide';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentSlide}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Video Title',
          content: 'Video content...',
          style: 'modern'
        }}
      />
    </>
  );
};
```

### Rendering Videos

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(options: VideoRenderOptions) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: options.title,
      content: options.content,
    },
  });

  const dimensions = getFormatDimensions(options.format);

  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `public/videos/${options.format}-${Date.now()}.mp4`,
    inputProps: {
      title: options.title,
      content: options.content,
    },
  });
}

function getFormatDimensions(format: string) {
  const formats = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 },
  };
  return formats[format] || formats.reels;
}
```

## Complete Workflow Example

```typescript
// lib/pipeline/content-pipeline.ts

export class ContentPipeline {
  async execute(keyword: string, options: PipelineOptions) {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await this.gatherResearch(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await this.generateContent({
      topic: keyword,
      format: options.format,
      language: options.language,
      tone: options.tone,
      researchData: research
    });
    
    // Step 3: Create bilingual versions
    console.log('🌐 Creating translations...');
    const translations = await this.createTranslations(content);
    
    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const video = await this.renderVideo({
      content: content,
      title: keyword,
      format: options.videoFormat
    });
    
    return {
      content,
      translations,
      video,
      research
    };
  }

  private async gatherResearch(keyword: string) {
    return await crawlRecentNews({
      keyword,
      sources: this.getDefaultSources(),
      timeframe: '24h'
    });
  }

  private async generateContent(request: ContentRequest) {
    if (process.env.ANTHROPIC_API_KEY) {
      return await generateContent(request);
    }
    return await generateWithOpenAI(request);
  }

  private async createTranslations(content: string) {
    const languages = ['en', 'vi'];
    const translations = {};
    
    for (const lang of languages) {
      translations[lang] = await this.translate(content, lang);
    }
    
    return translations;
  }

  private async renderVideo(options: VideoRenderOptions) {
    return await renderContentVideo(options);
  }
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: Request) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, videoFormat } = body;

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword, {
      format,
      language,
      tone,
      videoFormat
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

### Usage from Frontend

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: 'AI automation',
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        videoFormat: 'reels'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div>
          <h3>Generated Content</h3>
          <p>{result.content}</p>
          <video src={result.video} controls />
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Custom Content Formats

```typescript
interface CustomFormat {
  name: string;
  structure: string[];
  minWords: number;
  includeSections: string[];
}

const customFormats: Record<string, CustomFormat> = {
  'deep-dive': {
    name: 'Deep Dive Analysis',
    structure: ['intro', 'background', 'analysis', 'implications', 'conclusion'],
    minWords: 2000,
    includeSections: ['data-points', 'expert-quotes']
  },
  'quick-tips': {
    name: 'Quick Tips',
    structure: ['hook', 'tips-list', 'call-to-action'],
    minWords: 500,
    includeSections: ['actionable-steps']
  }
};
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    const result = await pipeline.execute(keyword, {
      format: 'toplist',
      language: 'en',
      tone: 'friendly',
      videoFormat: 'reels'
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const trendingTopics = await fetchTrendingTopics();
    const pipeline = new ContentPipeline();
    
    for (const topic of trendingTopics) {
      await pipeline.execute(topic, {
        format: 'pov',
        language: 'en',
        tone: 'expert',
        videoFormat: 'shorts'
      });
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Promise<any>[] = [];
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    const promise = this.queue[this.queue.length - 1] || Promise.resolve();
    
    const next = promise.then(() => 
      new Promise(resolve => setTimeout(resolve, this.delayMs))
    ).then(() => fn());
    
    this.queue.push(next);
    return next;
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
await limiter.execute(() => anthropic.messages.create({...}));
```

### Video Rendering Errors

```typescript
// Check Remotion installation
try {
  await renderContentVideo(options);
} catch (error) {
  if (error.message.includes('bundle')) {
    console.error('Remotion bundler error. Run: npm install @remotion/bundler');
  } else if (error.message.includes('renderer')) {
    console.error('Remotion renderer error. Check REMOTION_LICENSE_KEY');
  }
  throw error;
}
```

### Missing Research Data

```typescript
async function safeGatherResearch(keyword: string) {
  try {
    const research = await crawlRecentNews({ keyword });
    
    if (!research || research.length === 0) {
      console.warn('No recent research found, using fallback data');
      return await getFallbackData(keyword);
    }
    
    return research;
  } catch (error) {
    console.error('Research gathering failed:', error);
    return await getFallbackData(keyword);
  }
}
```

### Environment Variables Not Loading

```bash
# Verify .env file exists
ls -la .env

# Check Next.js env loading
# Restart dev server after .env changes
npm run dev
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const research = await crawlRecentNews({ keyword });
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(research));
  
  return research;
}
```

### Parallel Processing

```typescript
async function parallelPipeline(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      new ContentPipeline().execute(keyword, options)
    )
  );
  
  return results;
}
```
