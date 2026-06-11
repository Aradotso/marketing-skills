---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate content from research to video automatically
  - create marketing content with Claude and OpenAI
  - automate content creation workflow
  - build AI-powered content generation system
  - use Remotion to render videos from content
  - crawl news and generate articles with AI
  - set up automated social media content pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What It Does

Ultimate AI Content Pipeline is a complete content automation system that:

- **Auto-crawls** real-time news from sources like TechCrunch, a16z, Twitter, LinkedIn
- **Generates articles** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Supports multiple languages** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Exports optimized content** for Reels, TikTok, Shorts

This is a Next.js TypeScript application that transforms a single keyword into publication-ready content across multiple formats.

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion Studio (for video editing)
npm run remotion:studio
```

## Core Architecture

### 1. Research Module (Content Crawling)

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
}

export async function crawlNews(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const sources: NewsSource[] = [
    { url: 'https://techcrunch.com/search', selector: '.post-block', platform: 'techcrunch' },
    // Add more sources
  ];

  const articles = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get(source.url, {
        params: { q: keyword, time: timeframe },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      return parseArticles(response.data, source.selector);
    })
  );

  return articles.flat();
}

function parseArticles(html: string, selector: string) {
  // Parse and extract article data
  return [];
}
```

### 2. AI Content Generation

```typescript
// lib/ai/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: any[];
}

export async function generateContent(config: ContentConfig, provider: 'claude' | 'openai' = 'claude') {
  if (provider === 'claude') {
    return generateWithClaude(config);
  }
  return generateWithOpenAI(config);
}

async function generateWithClaude(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return parseContentResponse(message.content);
}

async function generateWithOpenAI(config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });

  return parseContentResponse(completion.choices[0].message.content);
}

function buildSystemPrompt(config: ContentConfig): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining',
  };

  return `You are an expert content creator specializing in ${config.format} articles.
Write in ${config.language === 'vi' ? 'Vietnamese' : 'English'} with a ${toneMap[config.tone]} tone.
Use the provided research data to create data-backed, insightful content.`;
}

function buildUserPrompt(config: ContentConfig): string {
  return `Create a ${config.format} article about "${config.keyword}".

Research Data:
${JSON.stringify(config.research, null, 2)}

Requirements:
- Length: 1500-2000 words
- Include statistics and data from research
- Add actionable insights
- Structure with clear headings
- Include a compelling introduction and conclusion`;
}

function parseContentResponse(content: any): any {
  // Parse and structure the AI response
  return {
    title: '',
    body: '',
    metadata: {},
  };
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface VideoProps {
  title: string;
  keyPoints: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, keyPoints, brandColor }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: brandColor, fontSize: 60, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence key={index} from={fps * (2 + index * 3)} durationInFrames={fps * 3}>
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <div style={{ color: '#fff', fontSize: 40, textAlign: 'center' }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(content: any, format: 'vertical' | 'square' | 'horizontal' = 'vertical') {
  const compositions = {
    vertical: { width: 1080, height: 1920 }, // TikTok, Reels
    square: { width: 1080, height: 1080 }, // Instagram
    horizontal: { width: 1920, height: 1080 }, // YouTube
  };

  const { width, height } = compositions[format];

  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      brandColor: '#FF6B6B',
    },
  });

  const outputLocation = path.resolve(`./public/videos/${content.id}-${format}.mp4`);

  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      brandColor: '#FF6B6B',
    },
  });

  return outputLocation;
}
```

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo, videoFormat } = await request.json();

    // Step 1: Research
    const research = await crawlNews(keyword, '24h');

    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      research,
    });

    // Step 3: Render video (optional)
    let videoUrl = null;
    if (includeVideo) {
      videoUrl = await renderContentVideo(content, videoFormat);
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
      },
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Patterns

### Complete Pipeline Example

```typescript
// app/content-generator/page.tsx
'use client';

import { useState } from 'react';

export default function ContentGeneratorPage() {
  const [config, setConfig] = useState({
    keyword: '',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    includeVideo: true,
    videoFormat: 'vertical',
  });
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(config),
      });
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">AI Content Pipeline</h1>

      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={config.keyword}
          onChange={(e) => setConfig({ ...config, keyword: e.target.value })}
          className="w-full p-3 border rounded"
        />

        <select
          value={config.format}
          onChange={(e) => setConfig({ ...config, format: e.target.value })}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !config.keyword}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-4">
          <div className="bg-white p-6 rounded-lg shadow">
            <h2 className="text-2xl font-bold mb-4">{result.content.title}</h2>
            <div dangerouslySetInnerHTML={{ __html: result.content.body }} />
          </div>

          {result.videoUrl && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h3 className="text-xl font-bold mb-4">Generated Video</h3>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface BatchConfig {
  keywords: string[];
  format: string;
  language: string;
  tone: string;
}

async function batchGenerate(config: BatchConfig) {
  const results = [];

  for (const keyword of config.keywords) {
    console.log(`Processing: ${keyword}`);

    const research = await crawlNews(keyword, '24h');
    const content = await generateContent({
      keyword,
      format: config.format,
      language: config.language,
      tone: config.tone,
      research,
    });

    const videoUrl = await renderContentVideo(content, 'vertical');

    results.push({ keyword, content, videoUrl });

    // Rate limiting
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }

  return results;
}

// Usage
const config = {
  keywords: ['AI Marketing', 'Content Automation', 'Video Marketing'],
  format: 'toplist',
  language: 'en',
  tone: 'expert',
};

batchGenerate(config).then((results) => {
  console.log(`Generated ${results.length} pieces of content`);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = (60 * 1000) / requestsPerMinute;
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
      await new Promise((resolve) => setTimeout(resolve, this.delayMs));
    }
    this.processing = false;
  }
}

// Usage
const openaiLimiter = new RateLimiter(50); // 50 requests per minute
await openaiLimiter.add(() => generateContent(config));
```

### Video Rendering Issues

```typescript
// If Remotion rendering fails, check Chrome installation
import { getExecutablePath } from '@remotion/renderer';

try {
  const chromePath = getExecutablePath();
  console.log('Chrome found at:', chromePath);
} catch (error) {
  console.error('Chrome not found. Install with: npx remotion browser ensure');
}
```

### Memory Management for Large Batches

```typescript
// Use streaming for large content generation
async function streamGenerate(keyword: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const stream = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    stream: true,
    messages: [{ role: 'user', content: `Write about ${keyword}` }],
  });

  for await (const messageStreamEvent of stream) {
    if (messageStreamEvent.type === 'content_block_delta') {
      process.stdout.write(messageStreamEvent.delta.text);
    }
  }
}
```

## Advanced Configuration

### Custom Remotion Compositions

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';
import { InfographicVideo } from './compositions/InfographicVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
      <Composition
        id="InfographicVideo"
        component={InfographicVideo}
        durationInFrames={450}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

This skill enables AI agents to implement and customize the Ultimate AI Content Pipeline for automated content creation workflows.
