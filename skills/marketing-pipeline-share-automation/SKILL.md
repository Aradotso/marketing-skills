---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline with video generation
  - create automated marketing content from research to video
  - build a content automation system with Claude and OpenAI
  - generate videos automatically from AI-written content
  - automate social media content research and posting
  - create AI-driven content workflow with Remotion
  - set up automated content pipeline for marketing
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that transforms a single keyword into complete content pieces including research, written content, and rendered videos. It integrates Claude 3, OpenAI, and Remotion to create a fully automated content production pipeline.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, X (Twitter), LinkedIn within 24h
- Generate multi-format content (Toplist, POV, Case Study, How-to)
- Support bilingual output (English & Vietnamese)
- Auto-render videos and infographics via Remotion
- Optimize content for Reels, TikTok, Shorts

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

# Copy environment variables
cp .env.example .env.local
```

## Configuration

### Required Environment Variables

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Sources
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Rendering)
REMOTION_STUDIO_PORT=3001

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### API Configuration

Create a `config/api.ts` file:

```typescript
export const apiConfig = {
  anthropic: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    temperature: 0.7,
  },
  rapidApi: {
    baseUrl: 'https://api.rapid.com',
    endpoints: {
      news: '/news/search',
      twitter: '/twitter/trends',
    },
  },
};
```

## Core Components

### 1. Research Module (Auto-Scan)

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    date: string;
  }>;
  insights: string[];
  keywords: string[];
}

async function autoResearch(keyword: string): Promise<ResearchResult> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Fetch news from multiple sources
  const newsData = await fetchNewsFromSources(keyword);
  
  // Analyze with Claude
  const message = await client.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Analyze these articles about "${keyword}" and extract key insights, trends, and data points:\n\n${JSON.stringify(newsData)}`,
      },
    ],
  });

  return parseResearchResults(message.content);
}

async function fetchNewsFromSources(keyword: string) {
  const sources = [
    { name: 'TechCrunch', api: '/techcrunch' },
    { name: 'a16z', api: '/a16z' },
    { name: 'Twitter', api: '/twitter' },
  ];

  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(
        `${process.env.RAPIDAPI_URL}${source.api}?q=${keyword}`,
        {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          },
        }
      );
      return response.json();
    })
  );

  return results.flat();
}
```

### 2. Content Generation Module

```typescript
import OpenAI from 'openai';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

async function generateContent(
  research: ResearchResult,
  options: ContentOptions
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildContentPrompt(research, options);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content creator specializing in ${options.format} format. Write in ${options.language} with a ${options.tone} tone.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content || '';
}

function buildContentPrompt(
  research: ResearchResult,
  options: ContentOptions
): string {
  return `
Based on this research:
${JSON.stringify(research, null, 2)}

Create a ${options.format} article for ${options.targetAudience}.
Include:
- Compelling headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
`;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

interface VideoScript {
  title: string;
  scenes: Array<{
    text: string;
    duration: number;
    background?: string;
  }>;
  music?: string;
}

async function generateVideo(
  content: string,
  format: 'reel' | 'tiktok' | 'short'
): Promise<string> {
  // Convert content to video script
  const script = await contentToScript(content, format);

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './src/video/index.ts',
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: format,
    inputProps: script,
  });

  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

async function contentToScript(
  content: string,
  format: string
): Promise<VideoScript> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await client.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 2048,
    messages: [
      {
        role: 'user',
        content: `Convert this content into a ${format} video script with 5-7 scenes (max 60 seconds total):\n\n${content}`,
      },
    ],
  });

  return JSON.parse(message.content[0].text);
}
```

### 4. Remotion Video Component

```typescript
// src/video/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

export const ContentVideo: React.FC<{
  title: string;
  scenes: Array<{ text: string; duration: number }>;
}> = ({ title, scenes }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 80,
            color: 'white',
            fontWeight: 'bold',
          }}
        >
          {title}
        </AbsoluteFill>
      </Sequence>

      {scenes.map((scene, index) => {
        const startFrame = 90 + scenes.slice(0, index).reduce((acc, s) => acc + s.duration, 0);
        return (
          <Sequence
            key={index}
            from={startFrame}
            durationInFrames={scene.duration}
          >
            <AbsoluteFill
              style={{
                justifyContent: 'center',
                alignItems: 'center',
                fontSize: 60,
                color: 'white',
                padding: 60,
                textAlign: 'center',
              }}
            >
              {scene.text}
            </AbsoluteFill>
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Integration

```typescript
// src/pipeline/index.ts
interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  videoFormat: 'reel' | 'tiktok' | 'short';
  targetAudience: string;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  content: string;
  videoPath: string;
  research: ResearchResult;
}> {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Researching...');
  const research = await autoResearch(config.keyword);

  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const content = await generateContent(research, {
    format: config.contentFormat,
    language: config.language,
    tone: 'expert',
    targetAudience: config.targetAudience,
  });

  // Step 3: Generate Video
  console.log('🎬 Rendering video...');
  const videoPath = await generateVideo(content, config.videoFormat);

  console.log('✅ Pipeline complete!');

  return {
    content,
    videoPath,
    research,
  };
}
```

## API Routes (Next.js)

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const config = req.body;

    const result = await runContentPipeline(config);

    res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
}
```

## CLI Usage

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start Remotion Studio (video preview)
npm run remotion:studio

# Render a specific composition
npm run remotion:render -- ContentVideo output.mp4

# Run the pipeline from CLI
npm run pipeline -- --keyword "AI trends 2024" --format toplist --lang en
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      language: 'en',
      videoFormat: 'reel',
      targetAudience: 'marketers',
    });

    results.push(result);

    // Rate limiting
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Pattern 2: Custom Video Templates

```typescript
// Register custom video composition
import { registerRoot } from 'remotion';
import { ContentVideo } from './video/ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="reel"
        component={ContentVideo}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1920}
      />
      <Composition
        id="tiktok"
        component={ContentVideo}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

### Pattern 3: Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedResearch(keyword: string): Promise<ResearchResult> {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const result = await autoResearch(keyword);

  // Cache for 24 hours
  await redis.setex(cacheKey, 86400, JSON.stringify(result));

  return result;
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
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((resolve) =>
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Errors

```bash
# Ensure ffmpeg is installed
brew install ffmpeg # macOS
apt-get install ffmpeg # Ubuntu

# Check Remotion configuration
npx remotion versions

# Clear Remotion cache
rm -rf node_modules/.cache/remotion
```

### Memory Issues

```typescript
// Limit concurrent operations
import pLimit from 'p-limit';

const limit = pLimit(3);

const results = await Promise.all(
  keywords.map((keyword) =>
    limit(() => runContentPipeline({ keyword, /* ... */ }))
  )
);
```

### API Key Issues

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```

## Best Practices

1. **Rate Limiting**: Implement delays between API calls to avoid hitting rate limits
2. **Error Handling**: Always wrap API calls in try-catch blocks
3. **Caching**: Cache research results to reduce API costs
4. **Monitoring**: Log pipeline progress and errors for debugging
5. **Resource Management**: Clean up temporary video files after processing
