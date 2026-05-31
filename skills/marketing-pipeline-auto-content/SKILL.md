---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from articles automatically
  - create content with Claude and OpenAI integration
  - build AI-powered content workflow
  - automate research and scriptwriting
  - use Remotion for video generation from text
  - scrape news and generate social media content
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end automated content creation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh content within 24 hours
- **Generates diverse content formats**: Creates toplist, POV, case study, and how-to articles in multiple languages
- **Renders videos automatically**: Converts text content into infographics and short-form videos using Remotion
- **Multi-platform optimization**: Outputs video in formats optimized for Reels, TikTok, and YouTube Shorts

Built with Next.js and TypeScript, it integrates Claude (Anthropic), OpenAI, RapidAPI, and Remotion for a complete content automation workflow.

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Components & Architecture

### 1. Research Module (Auto-Scan)

The research module crawls news sources and extracts insights:

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function scanLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<NewsArticle[]> {
  const rapidApi = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const results = await rapidApi.searchNews({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    articles.push(...results);
  }
  
  return articles;
}

export async function extractInsights(articles: NewsArticle[]): Promise<string[]> {
  const insights = articles.map(article => ({
    headline: article.title,
    key_points: extractKeyPoints(article.content),
    data_points: extractDataPoints(article.content)
  }));
  
  return insights;
}
```

### 2. Content Generation with AI

Using Claude or OpenAI to generate content in various formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: string[];
}

export async function generateContentWithClaude(
  options: ContentGenerationOptions
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(options);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentWithOpenAI(
  options: ContentGenerationOptions
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildPrompt(options);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing content.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(options: ContentGenerationOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with at least 7 items',
    'pov': 'Write from a unique perspective or angle',
    'case-study': 'Analyze a real-world example with data and insights',
    'how-to': 'Create a step-by-step tutorial guide',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and light humor while staying informative',
  };

  return `
Write a ${options.format} article about "${options.keyword}" in ${options.language}.
${formatInstructions[options.format]}
${toneInstructions[options.tone]}

Use the following research data as context:
${options.researchData.join('\n\n')}

Requirements:
- Include specific data points and statistics
- Make it SEO-optimized
- Add clear headings and subheadings
- Include a compelling introduction and conclusion
`;
}
```

### 3. Video Generation with Remotion

Convert text content into videos:

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function generateVideo(config: VideoConfig): Promise<string> {
  const specs = platformSpecs[config.platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputLocation;
}
```

Remotion composition example:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const contentPoints = content.split('\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1
            style={{
              fontSize: 80,
              color: 'white',
              textAlign: 'center',
              padding: 40,
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {contentPoints.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <div
              style={{
                fontSize: 48,
                color: 'white',
                textAlign: 'center',
                backgroundColor: 'rgba(255, 255, 255, 0.1)',
                padding: 40,
                borderRadius: 20,
              }}
            >
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
import { scanLatestNews, extractInsights } from '@/lib/research/crawler';
import { generateContentWithClaude } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  platforms: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  article: string;
  videos: string[];
}> {
  // Step 1: Research
  console.log('🔍 Scanning latest news...');
  const articles = await scanLatestNews(config.keyword);
  const insights = await extractInsights(articles);

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const article = await generateContentWithClaude({
    keyword: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData: insights,
  });

  // Step 3: Generate Videos
  console.log('🎬 Rendering videos...');
  const videos: string[] = [];
  
  for (const platform of config.platforms) {
    const videoPath = await generateVideo({
      content: article,
      title: config.keyword,
      platform,
    });
    videos.push(videoPath);
  }

  return { article, videos };
}
```

## API Routes (Next.js)

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, language, tone, platforms } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'friendly',
      platforms: platforms || ['reels'],
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'friendly',
          platforms: ['reels', 'tiktok'],
        }),
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
    <main className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-4 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-3 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-8 space-y-4">
            <div className="p-4 bg-gray-100 rounded">
              <h2 className="font-bold mb-2">Article</h2>
              <pre className="whitespace-pre-wrap">{result.data.article}</pre>
            </div>
            
            <div>
              <h2 className="font-bold mb-2">Videos</h2>
              {result.data.videos.map((video: string, i: number) => (
                <div key={i}>{video}</div>
              ))}
            </div>
          </div>
        )}
      </div>
    </main>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/batch-processor.ts
export async function batchGenerateContent(
  keywords: string[],
  config: Partial<PipelineConfig>
): Promise<Map<string, any>> {
  const results = new Map();

  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        format: config.format || 'toplist',
        language: config.language || 'en',
        tone: config.tone || 'friendly',
        platforms: config.platforms || ['reels'],
      });
      
      results.set(keyword, result);
    } catch (error) {
      console.error(`Failed for keyword: ${keyword}`, error);
      results.set(keyword, { error: error.message });
    }
  }

  return results;
}
```

### Scheduling Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  schedule: string,
  config: PipelineConfig
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    
    try {
      const result = await runContentPipeline(config);
      // Save to database or publish directly
      console.log('Content generated successfully:', result);
    } catch (error) {
      console.error('Scheduled generation failed:', error);
    }
  });
}

// Usage: Run daily at 9 AM
scheduleContentGeneration('0 9 * * *', {
  keyword: 'AI trends',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  platforms: ['reels', 'tiktok'],
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
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
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute

await limiter.add(() => generateContentWithClaude(options));
```

### Issue: Video Rendering Timeout

Increase timeout for long videos:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: {
    title: config.title,
    content: config.content,
  },
});
```

### Issue: Missing Research Data

Add fallback content when research fails:

```typescript
export async function scanLatestNewsWithFallback(
  keyword: string
): Promise<NewsArticle[]> {
  try {
    return await scanLatestNews(keyword);
  } catch (error) {
    console.warn('Research failed, using cached data');
    return getCachedNews(keyword);
  }
}
```

## Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to see the application.

## Building for Production

```bash
npm run build
npm start
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to assist with content generation, video rendering, and workflow automation tasks.
