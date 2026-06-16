---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - create automated content pipeline with Claude and OpenAI
  - build AI-powered content workflow
  - automate social media video generation
  - set up content research and script generation pipeline
  - use Remotion for automated video rendering
  - create multi-language marketing content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content pipeline that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a comprehensive content automation system that:

- **Auto-scans** recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Provides end-to-end pipeline** from keyword input to publishable content

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Development Setup

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start
```

## Core Components

### 1. Research Module

Automatically crawl and analyze recent content:

```typescript
// lib/research/crawler.ts
import { SearchClient } from './search-client';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  source: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const client = new SearchClient(process.env.RAPIDAPI_KEY);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await client.search({
      query: keyword,
      source,
      timeRange: '24h'
    });
    
    results.push(...data.map(item => ({
      title: item.title,
      url: item.url,
      content: item.snippet,
      source,
      publishedAt: new Date(item.published)
    })));
  }
  
  return results;
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  research: string[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }
  
  async generateWithClaude(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096,
    });
    
    return completion.choices[0]?.message?.content || '';
  }
  
  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a specific perspective with personal insights',
      'case-study': 'Analyze a real example with data and outcomes',
      'how-to': 'Provide step-by-step instructions'
    };
    
    const toneInstructions = {
      'expert': 'authoritative and professional',
      'friendly': 'conversational and approachable',
      'humorous': 'entertaining with wit'
    };
    
    return `
You are a content writer creating a ${config.format} article in ${config.language === 'en' ? 'English' : 'Vietnamese'}.

Topic: ${config.keyword}
Format: ${formatInstructions[config.format]}
Tone: ${toneInstructions[config.tone]}

Recent Research Data:
${config.research.join('\n\n')}

Requirements:
1. Use the research data to support your points
2. Include specific examples and data points
3. Write in a ${toneInstructions[config.tone]} tone
4. Make it actionable and valuable to readers
5. Target length: 1500-2000 words

Please generate the complete article now.
    `.trim();
  }
}
```

### 3. Video Generation with Remotion

Render videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from './compositions/VideoComposition';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export async function renderContentVideo(
  content: string,
  config: VideoConfig
): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/video/index.tsx',
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      aspectRatio: config.aspectRatio,
    },
  });
  
  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
  });
  
  return outputLocation;
}
```

### 4. Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = await req.json();
    
    // Step 1: Research
    const research = await crawlRecentNews(keyword);
    const researchText = research.map(r => 
      `${r.title}\n${r.content}\nSource: ${r.source}`
    );
    
    // Step 2: Generate Content
    const generator = new ContentGenerator();
    const content = await generator.generateWithClaude({
      keyword,
      format,
      language,
      tone,
      research: researchText,
    });
    
    // Step 3: Extract key points for video
    let videoUrl = null;
    if (includeVideo) {
      const keyPoints = await extractKeyPoints(content);
      videoUrl = await renderContentVideo(content, {
        title: keyword,
        keyPoints,
        duration: 60,
        aspectRatio: '9:16',
      });
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        research: research.length,
        videoUrl,
      },
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}

async function extractKeyPoints(content: string): Promise<string[]> {
  // Use AI to extract 3-5 key points from content
  const generator = new ContentGenerator();
  const response = await generator.generateWithOpenAI({
    keyword: 'Extract key points',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    research: [content],
  });
  
  return response
    .split('\n')
    .filter(line => line.match(/^\d+\./))
    .slice(0, 5);
}
```

## Frontend Usage

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function HomePage() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'en',
          tone: 'expert',
          includeVideo: true,
        }),
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full px-4 py-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-2 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-xl font-semibold mb-4">Results</h2>
          <div className="prose max-w-none">
            <p className="whitespace-pre-wrap">{result.data.content}</p>
          </div>
          {result.data.videoUrl && (
            <div className="mt-4">
              <video src={result.data.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/processor.ts
export async function batchGenerateContent(
  keywords: string[],
  config: Partial<ContentConfig>
): Promise<Map<string, string>> {
  const results = new Map<string, string>();
  const generator = new ContentGenerator();
  
  for (const keyword of keywords) {
    const research = await crawlRecentNews(keyword);
    const content = await generator.generateWithClaude({
      keyword,
      format: config.format || 'toplist',
      language: config.language || 'en',
      tone: config.tone || 'expert',
      research: research.map(r => r.content),
    });
    
    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Bilingual Content Generation

```typescript
export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat
): Promise<{ en: string; vi: string }> {
  const generator = new ContentGenerator();
  const research = await crawlRecentNews(keyword);
  const researchText = research.map(r => r.content);
  
  const [enContent, viContent] = await Promise.all([
    generator.generateWithClaude({
      keyword,
      format,
      language: 'en',
      tone: 'expert',
      research: researchText,
    }),
    generator.generateWithClaude({
      keyword,
      format,
      language: 'vi',
      tone: 'expert',
      research: researchText,
    }),
  ]);
  
  return { en: enContent, vi: viContent };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;
  
  constructor(requestsPerMinute: number) {
    this.delay = (60 * 1000) / requestsPerMinute;
  }
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
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
await limiter.execute(() => generator.generateWithClaude(config));
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  backoff = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.error(`Attempt ${i + 1} failed:`, error);
      
      if (i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, backoff * Math.pow(2, i))
        );
      }
    }
  }
  
  throw lastError!;
}
```

### Video Rendering Issues

If video rendering fails, check:

1. Remotion license is valid: `process.env.REMOTION_LICENSE_KEY`
2. FFmpeg is installed: `ffmpeg -version`
3. Output directory exists and is writable
4. Sufficient disk space for video files

```bash
# Install FFmpeg (macOS)
brew install ffmpeg

# Install FFmpeg (Ubuntu)
sudo apt-get install ffmpeg
```

## Performance Optimization

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

export async function getCachedResearch(
  keyword: string
): Promise<ResearchResult[] | null> {
  const cached = await redis.get(`research:${keyword}`);
  return cached as ResearchResult[] | null;
}

export async function cacheResearch(
  keyword: string,
  results: ResearchResult[]
): Promise<void> {
  await redis.set(`research:${keyword}`, results, {
    ex: 3600, // 1 hour TTL
  });
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share project, enabling AI coding agents to help developers implement automated content pipelines from research through video generation.
