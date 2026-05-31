---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - generate blog posts and videos automatically
  - set up an AI content pipeline
  - create automated marketing content workflow
  - build content from research to video
  - automate social media content generation
  - use Claude and OpenAI for content automation
  - implement AI-powered content research and writing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content creation pipeline that handles research, scriptwriting, content generation, and video rendering using AI models (Claude 3, OpenAI) and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes fresh content from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates diverse content formats (Toplists, POV, Case Studies, How-tos) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Output**: Exports content optimized for Reels, TikTok, Shorts, and blog posts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Model APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Content Sources
TWITTER_BEARER_TOKEN=your_twitter_token_here
LINKEDIN_API_KEY=your_linkedin_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_CONCURRENCY=4
REMOTION_QUALITY=80
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Core Architecture

The pipeline consists of several key modules:

```
/app           - Next.js application routes
/lib           - Core utilities and API clients
  /ai          - AI model integrations (Claude, OpenAI)
  /research    - Content scraping and analysis
  /generators  - Content format generators
/remotion      - Video composition templates
/public        - Static assets
```

## Key API Patterns

### 1. Research Module

Automatically scrape and analyze fresh content:

```typescript
// lib/research/scraper.ts
import { ResearchClient } from './client';

interface ResearchQuery {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function performResearch(query: ResearchQuery) {
  const client = new ResearchClient({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await client.scrape({
    keyword: query.keyword,
    sources: query.sources,
    timeframe: query.timeframe,
  });

  // Analyze and extract insights
  const insights = await analyzeData(results);
  
  return {
    rawData: results,
    insights,
    trends: extractTrends(results),
    dataPoints: extractDataPoints(results),
  };
}

async function analyzeData(results: any[]) {
  // Process scraped content for key insights
  return results.map(item => ({
    title: item.title,
    summary: item.summary,
    url: item.url,
    publishedAt: item.publishedAt,
    sentiment: analyzeSentiment(item.content),
  }));
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

export async function generateContent(request: ContentRequest) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });

  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.topic, request.researchData);

  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  const content = message.content[0].text;

  // If bilingual, generate second language
  if (request.language === 'vi') {
    const translated = await translateContent(content, 'vi');
    return { en: content, vi: translated };
  }

  return { [request.language]: content };
}

function buildSystemPrompt(format: string, tone: string): string {
  const formats = {
    'toplist': 'You are an expert content creator specializing in curated top lists with data-backed insights.',
    'pov': 'You are a thought leader sharing unique perspectives and expert opinions.',
    'case-study': 'You are a business analyst creating detailed, actionable case studies.',
    'how-to': 'You are an instructional designer creating clear, step-by-step guides.',
  };

  const tones = {
    'expert': 'Use professional, authoritative language with industry terminology.',
    'friendly': 'Use conversational, approachable language.',
    'humorous': 'Inject wit and humor while maintaining credibility.',
  };

  return `${formats[format]} ${tones[tone]}`;
}

function buildUserPrompt(topic: string, researchData?: any[]): string {
  let prompt = `Create comprehensive content about: ${topic}\n\n`;
  
  if (researchData && researchData.length > 0) {
    prompt += 'Use these recent insights and data points:\n';
    researchData.forEach(item => {
      prompt += `- ${item.title}: ${item.summary}\n`;
    });
  }
  
  return prompt;
}
```

### 3. Video Generation with Remotion

Create videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRequest {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  style: 'infographic' | 'text-overlay' | 'slides';
}

export async function generateVideo(request: VideoRequest) {
  const compositionId = getCompositionId(request.format, request.style);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: request.content,
      format: request.format,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: request.content,
      format: request.format,
    },
    concurrency: parseInt(process.env.REMOTION_CONCURRENCY || '4'),
  });

  return {
    videoPath: outputLocation,
    duration: composition.durationInFrames / composition.fps,
  };
}

function getCompositionId(format: string, style: string): string {
  const aspectRatios = {
    'reels': '9:16',
    'tiktok': '9:16',
    'shorts': '9:16',
  };
  
  return `${style}-${aspectRatios[format]}`;
}
```

### 4. Remotion Composition Example

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface InfographicProps {
  content: string;
  format: string;
}

export const Infographic: React.FC<InfographicProps> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30, durationInFrames - 30, durationInFrames],
    [0, 1, 1, 0]
  );

  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: 40,
          color: 'white',
          fontSize: format === 'reels' ? 48 : 36,
          textAlign: 'center',
          maxWidth: '80%',
        }}
      >
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, outputTypes } = await request.json();

    // Step 1: Research
    const research = await performResearch({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
      language,
    });

    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format,
      language,
      tone: 'expert',
      researchData: research.insights,
    });

    // Step 3: Generate Video (if requested)
    let video = null;
    if (outputTypes.includes('video')) {
      video = await generateVideo({
        content: content[language],
        format: 'reels',
        style: 'infographic',
      });
    }

    return NextResponse.json({
      success: true,
      data: {
        research: research.insights.slice(0, 5), // Top 5 insights
        content,
        video: video?.videoPath,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// app/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export default function PipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    const payload = {
      keyword: formData.get('keyword'),
      format: formData.get('format'),
      language: formData.get('language'),
      outputTypes: formData.getAll('outputTypes'),
    };

    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        name="keyword"
        placeholder="Enter topic or keyword"
        required
        className="w-full px-4 py-2 border rounded"
      />
      
      <select name="format" className="w-full px-4 py-2 border rounded">
        <option value="toplist">Top List</option>
        <option value="pov">POV / Opinion</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to Guide</option>
      </select>

      <select name="language" className="w-full px-4 py-2 border rounded">
        <option value="en">English</option>
        <option value="vi">Vietnamese</option>
      </select>

      <div className="flex gap-4">
        <label>
          <input type="checkbox" name="outputTypes" value="text" defaultChecked />
          Text Content
        </label>
        <label>
          <input type="checkbox" name="outputTypes" value="video" />
          Video
        </label>
      </div>

      <button
        type="submit"
        disabled={loading}
        className="w-full px-4 py-2 bg-blue-600 text-white rounded"
      >
        {loading ? 'Processing...' : 'Generate Content'}
      </button>

      {result?.success && (
        <div className="mt-4 p-4 bg-green-50 rounded">
          <h3 className="font-bold">Generated Successfully!</h3>
          <pre className="mt-2 text-sm overflow-auto">
            {JSON.stringify(result.data, null, 2)}
          </pre>
        </div>
      )}
    </form>
  );
}
```

## Common Usage Patterns

### Pattern 1: Batch Content Generation

```typescript
// scripts/batch-generate.ts
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      const research = await performResearch({
        keyword: topic,
        sources: ['techcrunch', 'twitter'],
        timeframe: '24h',
      });

      return await generateContent({
        topic,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: research.insights,
      });
    })
  );

  return results;
}
```

### Pattern 2: Multi-Language Content

```typescript
async function generateBilingual(topic: string) {
  const research = await performResearch({
    keyword: topic,
    sources: ['techcrunch'],
    timeframe: '24h',
  });

  const [enContent, viContent] = await Promise.all([
    generateContent({
      topic,
      format: 'how-to',
      language: 'en',
      tone: 'friendly',
      researchData: research.insights,
    }),
    generateContent({
      topic,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
      researchData: research.insights,
    }),
  ]);

  return { en: enContent, vi: viContent };
}
```

## Troubleshooting

### API Rate Limits

If you hit API rate limits:

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number = 3, private delayMs: number = 1000) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          this.running++;
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        } finally {
          this.running--;
          await new Promise(r => setTimeout(r, this.delayMs));
          this.process();
        }
      });
      this.process();
    });
  }

  private process() {
    if (this.running < this.maxConcurrent && this.queue.length > 0) {
      const fn = this.queue.shift();
      fn?.();
    }
  }
}

// Usage
const limiter = new RateLimiter(3, 1000);
const result = await limiter.add(() => generateContent(params));
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion configuration for large videos
const outputLocation = await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps,
  concurrency: 1, // Reduce concurrency
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage'],
  },
  timeoutInMilliseconds: 120000,
});
```

### Content Quality Issues

```typescript
// Add validation and retry logic
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const content = await generateContent(request);
    
    if (isQualityContent(content)) {
      return content;
    }
    
    console.log(`Retry ${i + 1}/${maxRetries}: Quality check failed`);
  }
  
  throw new Error('Failed to generate quality content after retries');
}

function isQualityContent(content: any): boolean {
  const text = content.en || content.vi || '';
  return text.length > 500 && text.split(' ').length > 100;
}
```

## Production Deployment

Build for production:

```bash
# Build Next.js app
npm run build

# Build Remotion compositions
npx remotion bundle remotion/index.ts public/remotion-bundle

# Start production server
npm start
```

Environment variables for production:

```bash
NODE_ENV=production
OPENAI_API_KEY=your_production_key
ANTHROPIC_API_KEY=your_production_key
RAPIDAPI_KEY=your_production_key
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

This skill provides comprehensive guidance for integrating and using the Ultimate AI Content Pipeline for automated content creation workflows.
