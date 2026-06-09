---
name: ultimate-ai-content-pipeline
description: TypeScript-based automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I generate automated content with AI pipeline
  - set up content automation from research to video
  - use Claude and OpenAI for content generation workflow
  - create automated marketing content with Remotion
  - build AI-powered content pipeline with TypeScript
  - automate content research and video creation
  - generate multi-language content with AI
  - set up automated content publishing pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based system that automates the entire content creation workflow: from real-time research and data scraping, to AI-powered content generation (using Claude 3 and OpenAI), to automatic video rendering with Remotion. The pipeline supports multi-language output (English/Vietnamese), multiple content formats, and platform-optimized video exports.

## What It Does

This project creates a fully automated content production pipeline:

1. **Auto-Research**: Crawls and analyzes fresh data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates diverse formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multi-Language Support**: Generates content simultaneously in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically renders infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_BUCKET=your_s3_bucket
REMOTION_REGION=us-east-1

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research and data scraping
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

// Automatically crawl and analyze recent data
async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 50
  });

  return {
    insights: research.insights,
    dataPoints: research.dataPoints,
    trends: research.trends,
    sources: research.sourcesUsed
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'professional' | 'friendly' | 'humorous'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about "${topic}" in ${language} with a ${tone} tone. 
      Include data-backed insights and recent trends.`
    }],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  researchData: any,
  contentParams: {
    format: string;
    language: string;
    tone: string;
  }
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${contentParams.format} format.`
      },
      {
        role: 'user',
        content: `Based on this research data: ${JSON.stringify(researchData)}, 
        create content in ${contentParams.language} with ${contentParams.tone} tone.`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';
import { renderVideo } from '@/lib/video/remotion';

async function executeContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate Content (both languages)
  const contentEN = await generateContent(
    keyword,
    'toplist',
    'en',
    'professional'
  );

  const contentVI = await generateContent(
    keyword,
    'toplist',
    'vi',
    'professional'
  );

  // Step 3: Generate Video
  const videoUrl = await renderVideo({
    content: contentEN,
    format: 'reels', // or 'tiktok', 'shorts'
    duration: 60,
    style: 'infographic'
  });

  return {
    research,
    content: {
      en: contentEN,
      vi: contentVI
    },
    video: videoUrl
  };
}
```

### 5. Remotion Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderVideo(contentData: {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
  style: string;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: contentData.content,
      style: contentData.style
    }
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: contentData.content,
      style: contentData.style
    }
  });

  return outputLocation;
}
```

### 6. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages, includeVideo } = await request.json();

    const result = await executeContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { 
        success: false, 
        error: error instanceof Error ? error.message : 'Unknown error'
      },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Component Example

```typescript
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          includeVideo: true
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="content-generator">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div className="results">
          <h3>English Content</h3>
          <div>{result.data.content.en}</div>
          <h3>Vietnamese Content</h3>
          <div>{result.data.content.vi}</div>
          {result.data.video && (
            <video src={result.data.video} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video (if CLI available)
npx remotion render src/index.tsx ContentVideo out/video.mp4
```

## Configuration Options

### Content Generation Config

```typescript
interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
  includeImages: boolean;
  includeDataPoints: boolean;
}
```

### Research Config

```typescript
interface ResearchConfig {
  keyword: string;
  sources: Array<'techcrunch' | 'a16z' | 'twitter' | 'linkedin'>;
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
  language?: 'en' | 'vi';
}
```

### Video Config

```typescript
interface VideoConfig {
  format: 'reels' | 'tiktok' | 'shorts' | 'square';
  duration: number; // seconds
  fps: number;
  resolution: {
    width: number;
    height: number;
  };
  style: 'infographic' | 'talking-head' | 'slideshow';
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

async function generateWithRateLimit(userId: string, prompt: string) {
  const { success } = await ratelimit.limit(userId);
  
  if (!success) {
    throw new Error('Rate limit exceeded');
  }
  
  return await generateContent(prompt);
}
```

### Error Handling

```typescript
async function safeContentGeneration(params: any) {
  try {
    return await executeContentPipeline(params.keyword);
  } catch (error) {
    if (error instanceof Anthropic.APIError) {
      console.error('Claude API Error:', error.status, error.message);
      // Fallback to OpenAI
      return await generateWithGPT(params);
    }
    
    if (error instanceof Error && error.message.includes('rate limit')) {
      console.error('Rate limit hit, retrying after delay...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await executeContentPipeline(params.keyword);
    }
    
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Handle Remotion rendering errors
async function safeRenderVideo(config: VideoConfig) {
  try {
    return await renderVideo(config);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Try with lower quality settings
    return await renderVideo({
      ...config,
      resolution: {
        width: config.resolution.width / 2,
        height: config.resolution.height / 2
      },
      fps: 30
    });
  }
}
```

## Best Practices

1. **Cache Research Results**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for heavy Remotion rendering tasks
3. **Validate AI Output**: Always validate and sanitize AI-generated content
4. **Monitor Costs**: Track API usage for Claude, OpenAI, and RapidAPI
5. **Optimize Video Settings**: Balance quality with rendering time and file size
6. **Multi-Language Testing**: Verify content quality in both languages before publishing
7. **Error Recovery**: Implement fallback strategies for each pipeline stage
