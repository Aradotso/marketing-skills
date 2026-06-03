---
name: ultimate-ai-content-pipeline
description: Vietnamese marketing content automation system with AI research, multi-format scriptwriting, and auto video generation using Claude/OpenAI and Remotion
triggers:
  - set up automated content pipeline with AI
  - generate marketing content from research to video
  - create Vietnamese and English content automatically
  - automate content research and video generation
  - build AI-powered content creation workflow
  - integrate Claude and OpenAI for content automation
  - create auto-posting marketing pipeline
  - generate videos from text content with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that transforms a single keyword into fully-researched, multi-format content with auto-generated videos. The pipeline crawls real-time news from TechCrunch, a16z, Twitter/X, and LinkedIn, generates bilingual content (English/Vietnamese) using Claude 3 or OpenAI, and renders videos using Remotion.

## What It Does

This project automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from news sources and social media within 24 hours
2. **AI Content Generation**: Creates multiple content formats (Toplist, POV, Case Study, How-to) in multiple languages and tones
3. **Video Rendering**: Automatically generates infographics and short-form videos optimized for Reels, TikTok, and Shorts
4. **Multi-platform Publishing**: Prepares content for automatic posting to various platforms

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

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if used)
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs for auto-posting
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawlers/    # News/social media scrapers
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Workflow

### 1. Research Phase - Auto-Crawling

```typescript
// lib/crawlers/news-aggregator.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  timeframe: number; // hours
}

export async function crawlRecentNews(keyword: string): Promise<Article[]> {
  const sources = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search/', timeframe: 24 },
    { name: 'a16z', url: 'https://a16z.com/posts/', timeframe: 48 }
  ];

  const articles: Article[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      });

      const filtered = response.data.articles.filter((article: any) => {
        const hoursOld = (Date.now() - new Date(article.publishedAt).getTime()) / 3600000;
        return hoursOld <= source.timeframe;
      });

      articles.push(...filtered);
    } catch (error) {
      console.error(`Error crawling ${source.name}:`, error);
    }
  }

  return articles;
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: Article[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(request.format, request.tone, request.language);
  const userPrompt = buildUserPrompt(request.keyword, request.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

function buildSystemPrompt(format: ContentFormat, tone: Tone, language: Language): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining'
  };

  const formatMap = {
    'toplist': 'a numbered list article with clear rankings',
    'pov': 'an opinion piece with strong personal perspective',
    'case-study': 'a detailed analysis of a specific example',
    'how-to': 'a step-by-step tutorial guide'
  };

  return `You are an expert ${language === 'vi' ? 'Vietnamese' : 'English'} content writer.
Write ${formatMap[format]} in a ${toneMap[tone]} tone.
Use data and insights from provided research to support your points.
Make content engaging, SEO-optimized, and platform-ready.`;
}

function buildUserPrompt(keyword: string, research: Article[]): string {
  const researchSummary = research.map(article => 
    `- ${article.title} (${article.source}): ${article.summary}`
  ).join('\n');

  return `Topic: ${keyword}\n\nRecent Research:\n${researchSummary}\n\nCreate comprehensive content based on this research.`;
}
```

### 3. Bilingual Content Generation

```typescript
// lib/generators/bilingual-content.ts
export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  researchData: Article[]
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format,
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      keyword,
      format,
      language: 'vi',
      tone: 'friendly',
      researchData
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render-content-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: ContentFormat;
  aspectRatio: '9:16' | '16:9' | '1:1'; // Reels/TikTok, YouTube, Instagram
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = `${config.format}-${config.aspectRatio}`;
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${compositionId}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Template Example

```typescript
// remotion/compositions/ToplistVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ToplistVideoProps {
  content: string;
  items: string[];
}

export const ToplistVideo: React.FC<ToplistVideoProps> = ({ content, items }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {items.map((item, index) => (
        <Sequence
          key={index}
          from={index * 90}
          durationInFrames={90}
        >
          <ToplistItem item={item} index={index + 1} frame={frame} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const ToplistItem: React.FC<{ item: string; index: number; frame: number }> = ({
  item,
  index,
  frame
}) => {
  const opacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });
  const scale = interpolate(frame, [0, 20], [0.8, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
      <div style={{ opacity, transform: `scale(${scale})` }}>
        <h1 style={{ color: '#fff', fontSize: '4rem' }}>#{index}</h1>
        <p style={{ color: '#ccc', fontSize: '2rem', maxWidth: '80%' }}>{item}</p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
export async function runContentPipeline(keyword: string): Promise<PipelineResult> {
  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  const research = await crawlRecentNews(keyword);
  console.log(`Found ${research.length} articles`);

  // Step 2: Generate content in multiple formats
  const formats: ContentFormat[] = ['toplist', 'how-to', 'pov'];
  const contentResults = await Promise.all(
    formats.map(async (format) => {
      const bilingualContent = await generateBilingualContent(keyword, format, research);
      return { format, ...bilingualContent };
    })
  );

  // Step 3: Render videos for each content piece
  const videoResults = await Promise.all(
    contentResults.map(async (content) => {
      const aspectRatios: Array<'9:16' | '16:9' | '1:1'> = ['9:16', '16:9', '1:1'];
      
      const videos = await Promise.all(
        aspectRatios.map((aspectRatio) =>
          renderContentVideo({
            content: content.en,
            format: content.format,
            aspectRatio,
          })
        )
      );

      return { format: content.format, videos };
    })
  );

  return {
    keyword,
    research,
    content: contentResults,
    videos: videoResults,
    timestamp: new Date().toISOString(),
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the application
# Navigate to http://localhost:3000
```

## Common Usage Patterns

### Pattern 1: Single Keyword to Full Content Suite

```typescript
// Example usage in a component or API route
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

async function createContentSuite() {
  const result = await runContentPipeline('AI Marketing Automation');
  
  // Returns:
  // - 20+ research articles
  // - 3 content formats x 2 languages = 6 articles
  // - 9 videos (3 formats x 3 aspect ratios)
}
```

### Pattern 2: Custom Format with Specific Tone

```typescript
const customContent = await generateContent({
  keyword: 'Content Marketing Trends 2024',
  format: 'case-study',
  language: 'vi',
  tone: 'humorous',
  researchData: await crawlRecentNews('content marketing')
});
```

### Pattern 3: Batch Video Rendering

```typescript
// lib/video/batch-renderer.ts
export async function renderMultipleVideos(
  contents: string[],
  format: ContentFormat
): Promise<string[]> {
  const aspectRatios: Array<'9:16' | '16:9' | '1:1'> = ['9:16', '16:9', '1:1'];
  
  const allVideos = await Promise.all(
    contents.flatMap((content) =>
      aspectRatios.map((aspectRatio) =>
        renderContentVideo({ content, format, aspectRatio })
      )
    )
  );

  return allVideos;
}
```

## Configuration Options

### AI Provider Selection

```typescript
// lib/ai/provider-config.ts
export const AI_PROVIDERS = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    bestFor: ['long-form', 'analysis', 'bilingual']
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    bestFor: ['creative', 'concise', 'technical']
  }
} as const;

// Switch providers dynamically
export function getAIProvider(preference: 'claude' | 'openai') {
  return preference === 'claude' 
    ? new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })
    : new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
}
```

### Video Output Configuration

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  durationInFrames: 270, // 9 seconds at 30fps
  width: {
    '9:16': 1080,
    '16:9': 1920,
    '1:1': 1080
  },
  height: {
    '9:16': 1920,
    '16:9': 1080,
    '1:1': 1080
  }
};
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

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

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
      }
    }
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.add(() => generateContent(request));
```

### Issue: Video Rendering Memory Issues

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(configs: VideoConfig[]): Promise<string[]> {
  const results: string[] = [];
  
  for (const config of configs) {
    const videoPath = await renderContentVideo(config);
    results.push(videoPath);
    
    // Clear memory between renders
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Issue: Crawling Blocked by Rate Limits

```typescript
// Use rotating proxies or add delays
async function crawlWithRetry(url: string, maxRetries = 3): Promise<any> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await axios.get(url, {
        headers: { 'X-RapidAPI-Key': process.env.RAPIDAPI_KEY }
      });
      return response.data;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Issue: Content Quality Inconsistency

```typescript
// Add validation and regeneration logic
async function generateValidatedContent(request: ContentRequest): Promise<string> {
  let attempts = 0;
  const maxAttempts = 3;

  while (attempts < maxAttempts) {
    const content = await generateContent(request);
    
    if (validateContent(content, request)) {
      return content;
    }
    
    attempts++;
    console.log(`Content validation failed, retrying (${attempts}/${maxAttempts})`);
  }

  throw new Error('Failed to generate valid content after maximum attempts');
}

function validateContent(content: string, request: ContentRequest): boolean {
  const minLength = { 'toplist': 1000, 'how-to': 1500, 'pov': 800, 'case-study': 2000 };
  return content.length >= minLength[request.format];
}
```

## Performance Optimization

```typescript
// Cache research data to avoid redundant crawls
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string): Promise<Article[] | null> {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

async function cacheResearch(keyword: string, articles: Article[]): Promise<void> {
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(articles)); // 1 hour cache
}

export async function crawlRecentNewsWithCache(keyword: string): Promise<Article[]> {
  const cached = await getCachedResearch(keyword);
  if (cached) return cached;

  const articles = await crawlRecentNews(keyword);
  await cacheResearch(keyword, articles);
  return articles;
}
```

This pipeline transforms raw keywords into production-ready, multi-platform content with minimal human intervention, perfect for marketing teams and content creators scaling their output.
