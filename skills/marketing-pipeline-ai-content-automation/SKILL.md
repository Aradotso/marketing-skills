---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - automate content research and video generation
  - create automated marketing content with AI
  - generate videos from written content automatically
  - set up Claude and OpenAI for content automation
  - build an AI-powered content creation workflow
  - automate social media content and video rendering
  - use Remotion to render marketing videos
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, a TypeScript-based automated content pipeline that handles everything from research and scriptwriting to video generation. The system crawls news sources, generates multi-format content with AI, and renders videos automatically.

## What This Project Does

The marketing pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
2. **AI Content Generation**: Uses Claude 3 and OpenAI to create content in multiple formats (toplist, POV, case study, how-to)
3. **Multi-Language Support**: Generates parallel English and Vietnamese content
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
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

Create a `.env.local` file in the project root:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

Reference environment variables in code:

```typescript
const anthropicKey = process.env.ANTHROPIC_API_KEY;
const openaiKey = process.env.OPENAI_API_KEY;
const rapidApiKey = process.env.RAPIDAPI_KEY;
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web crawling and research
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlNewsSource } from '@/lib/research/crawler';

// Crawl recent news from a source
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await Promise.all(
    sources.map(source => 
      crawlNewsSource({
        source,
        keyword,
        timeframe: '24h',
        limit: 10
      })
    )
  );
  
  return results.flat();
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
  language: 'en' | 'vi'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Write a ${format} article about ${topic} in ${language}. 
                Include data-backed insights and current trends.`
    }],
  });
  
  return message.content[0].text;
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  content: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...dimensions[platform],
    },
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${platform}-video.mp4`,
    inputProps: { content },
  });
  
  return `out/${platform}-video.mp4`;
}
```

## Common Workflow Patterns

### Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const researchData = await researchTopic(keyword);
    
    // Step 2: Generate content in multiple languages
    console.log('✍️ Generating content...');
    const [contentEN, contentVI] = await Promise.all([
      generateContent(keyword, 'toplist', 'en'),
      generateContent(keyword, 'toplist', 'vi'),
    ]);
    
    // Step 3: Generate videos for different platforms
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      generateVideo(contentEN, 'reels'),
      generateVideo(contentEN, 'tiktok'),
      generateVideo(contentEN, 'shorts'),
    ]);
    
    return {
      content: { en: contentEN, vi: contentVI },
      videos,
      research: researchData,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### Multi-Format Content Generation

```typescript
async function generateMultiFormatContent(topic: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  const tones = ['expert', 'friendly', 'humorous'];
  
  const contentVariations = await Promise.all(
    formats.map(async (format) => ({
      format,
      content: await generateContent(topic, format, 'en'),
    }))
  );
  
  return contentVariations;
}
```

### Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Render Remotion video (example)
npm run remotion:render -- ContentVideo out/video.mp4

# Preview Remotion composition
npm run remotion:preview
```

## API Route Examples

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research';

export async function POST(req: NextRequest) {
  const { keyword } = await req.json();
  
  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }
  
  const data = await researchTopic(keyword);
  return NextResponse.json({ data });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(req: NextRequest) {
  const { topic, format, language } = await req.json();
  
  const content = await generateContent(topic, format, language);
  return NextResponse.json({ content });
}
```

## TypeScript Type Definitions

```typescript
// types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Platform = 'reels' | 'tiktok' | 'shorts';
export type Tone = 'expert' | 'friendly' | 'humorous';

export interface ResearchResult {
  source: string;
  title: string;
  url: string;
  publishedAt: Date;
  snippet: string;
}

export interface ContentConfig {
  topic: string;
  format: ContentFormat;
  language: Language;
  tone?: Tone;
  research?: ResearchResult[];
}

export interface VideoConfig {
  content: string;
  platform: Platform;
  dimensions: {
    width: number;
    height: number;
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

### Remotion Rendering Issues

```typescript
// Ensure proper codec and browser setup
import { BrowserExecutable } from '@remotion/renderer';

const renderConfig = {
  codec: 'h264',
  chromiumOptions: {
    executablePath: process.env.CHROMIUM_PATH,
  },
  envVariables: {
    NODE_ENV: 'production',
  },
};
```

## Performance Optimization

### Parallel Processing

```typescript
async function parallelContentGeneration(topics: string[]) {
  // Process in batches to avoid overwhelming APIs
  const batchSize = 3;
  const results = [];
  
  for (let i = 0; i < topics.length; i += batchSize) {
    const batch = topics.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(topic => runContentPipeline(topic))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Caching Research Data

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;
  
  const data = await researchTopic(keyword);
  await redis.set(`research:${keyword}`, data, { ex: 3600 }); // 1 hour
  return data;
}
```
