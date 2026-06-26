---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up marketing pipeline for auto content generation
  - create automated content workflow with Claude and video rendering
  - build AI content pipeline from research to video
  - integrate Remotion for automated video content generation
  - configure multi-language content automation system
  - use marketing pipeline share for content creation
  - generate automated social media content with AI
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an end-to-end automated content generation system that performs research, scriptwriting, content generation, and video rendering using Claude/OpenAI and Remotion.

## What This Project Does

The marketing-pipeline-share project is a TypeScript/Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case studies, how-to) in multiple languages using Claude 3 and OpenAI
3. **Video Rendering**: Automatically generates videos and infographics from written content using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Required Node.js version
node --version  # v18.0.0 or higher

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for web scraping/research
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_CONCURRENCY=4
REMOTION_QUALITY=80

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion Studio (for video editing)
npm run remotion
```

## Core Architecture

### Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── research/       # Web scraping & data collection
│   │   ├── content/        # Content generation logic
│   │   └── video/          # Remotion video rendering
│   ├── remotion/           # Remotion video compositions
│   └── types/              # TypeScript definitions
├── public/                 # Static assets
└── .env.local             # Environment variables
```

## Key Components & Usage

### 1. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

export async function generateContent(options: ContentOptions) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(options);
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return parseContentResponse(message.content);
}

function buildPrompt(options: ContentOptions): string {
  const { keyword, format, language, tone, researchData } = options;
  
  let prompt = `Create a ${format} article about "${keyword}" in ${language} with a ${tone} tone.\n\n`;
  
  if (researchData && researchData.length > 0) {
    prompt += `Use the following research data:\n${JSON.stringify(researchData, null, 2)}\n\n`;
  }
  
  prompt += `Requirements:
- Include data-backed insights
- Use engaging headlines
- Add actionable takeaways
- Format for social media sharing`;
  
  return prompt;
}
```

### 2. Research & Data Collection

```typescript
// lib/research/web-scraper.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export async function performResearch(keyword: string, sources?: ResearchSource[]) {
  const defaultSources = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search/' },
    { name: 'a16z', url: 'https://a16z.com/search/' },
  ];

  const searchSources = sources || defaultSources;
  const results = [];

  for (const source of searchSources) {
    try {
      const data = await scrapeSource(source, keyword);
      results.push({
        source: source.name,
        articles: data,
      });
    } catch (error) {
      console.error(`Failed to scrape ${source.name}:`, error);
    }
  }

  return results;
}

async function scrapeSource(source: ResearchSource, keyword: string) {
  const options = {
    method: 'GET',
    url: 'https://web-scraper-api.p.rapidapi.com/scrape',
    params: {
      url: `${source.url}${encodeURIComponent(keyword)}`,
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'web-scraper-api.p.rapidapi.com',
    },
  };

  const response = await axios.request(options);
  return parseScrapedData(response.data);
}
```

### 3. API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';
import { performResearch } from '@/lib/research/web-scraper';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, includeResearch } = body;

    // Step 1: Perform research if requested
    let researchData;
    if (includeResearch) {
      researchData = await performResearch(keyword);
    }

    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData,
    });

    return NextResponse.json({
      success: true,
      content,
      research: researchData,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 4. Remotion Video Generation

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor = '#1a1a1a',
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <TitleCard title={title} frame={frame} />
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <PointCard point={point} index={index + 1} frame={frame} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(contentData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    concurrency: parseInt(process.env.REMOTION_CONCURRENCY || '4'),
  });

  return outputLocation;
}
```

### 5. Client-Side Usage

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const generateContent = async (formData: any) => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="content-generator">
      <form onSubmit={(e) => {
        e.preventDefault();
        const formData = new FormData(e.currentTarget);
        generateContent({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          language: formData.get('language'),
          tone: formData.get('tone'),
          includeResearch: formData.get('includeResearch') === 'on',
        });
      }}>
        <input name="keyword" placeholder="Enter keyword" required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>
        <select name="language">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>
        <select name="tone">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        <label>
          <input type="checkbox" name="includeResearch" />
          Include Research
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="result">
          <h2>Generated Content</h2>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/batch-processor.ts
export async function batchGenerateContent(keywords: string[], options: Partial<ContentOptions>) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const content = await generateContent({
        keyword,
        ...options,
      });
      
      results.push({
        keyword,
        success: true,
        content,
      });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      results.push({
        keyword,
        success: false,
        error: error.message,
      });
    }
  }
  
  return results;
}
```

### Multi-Language Content Generation

```typescript
// lib/content/multi-language.ts
export async function generateMultiLanguageContent(
  keyword: string,
  languages: Array<'en' | 'vi'>,
  options: Partial<ContentOptions>
) {
  const results = await Promise.all(
    languages.map(language =>
      generateContent({
        keyword,
        language,
        ...options,
      })
    )
  );

  return languages.reduce((acc, lang, index) => {
    acc[lang] = results[index];
    return acc;
  }, {} as Record<string, any>);
}
```

### Video Rendering Queue

```typescript
// lib/video/render-queue.ts
import { renderContentVideo } from './render-video';

class RenderQueue {
  private queue: any[] = [];
  private processing = false;

  async add(videoData: any) {
    this.queue.push(videoData);
    if (!this.processing) {
      await this.process();
    }
  }

  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const videoData = this.queue.shift();
      try {
        await renderContentVideo(videoData);
      } catch (error) {
        console.error('Video render failed:', error);
      }
    }
    
    this.processing = false;
  }
}

export const renderQueue = new RenderQueue();
```

## Configuration

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setQuality(80);
Config.setCodec('h264');
```

### TypeScript Types

```typescript
// types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

export interface GeneratedContent {
  id: string;
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  title: string;
  content: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    createdAt: Date;
  };
  research?: ResearchData[];
}

export interface ResearchData {
  source: string;
  title: string;
  url: string;
  snippet: string;
  publishedDate?: Date;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private requests: number[] = [];
  private maxRequests: number;
  private timeWindow: number;

  constructor(maxRequests: number, timeWindowMs: number) {
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindowMs;
  }

  async throttle() {
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < this.timeWindow);

    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.timeWindow - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }

    this.requests.push(Date.now());
  }
}

// Usage
const limiter = new RateLimiter(10, 60000); // 10 requests per minute
await limiter.throttle();
```

### Error Handling for AI APIs

```typescript
// lib/ai/error-handler.ts
export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, i);
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

If encountering memory issues during video rendering:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev

# Reduce Remotion concurrency in .env.local
REMOTION_CONCURRENCY=2
REMOTION_QUALITY=70
```

### Missing Environment Variables

```typescript
// lib/utils/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}
```

## Performance Optimization

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
import { LRUCache } from 'lru-cache';

const researchCache = new LRUCache<string, any>({
  max: 100,
  ttl: 1000 * 60 * 60 * 24, // 24 hours
});

export async function getCachedResearch(keyword: string) {
  const cached = researchCache.get(keyword);
  if (cached) return cached;

  const data = await performResearch(keyword);
  researchCache.set(keyword, data);
  return data;
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share project for AI coding agents to effectively assist developers in implementing automated content generation workflows.
