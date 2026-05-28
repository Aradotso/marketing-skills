---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI-powered content pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - generate videos from blog posts automatically
  - crawl trending news and create content with AI
  - build content automation system with Remotion
  - automate content from research to publication
  - integrate AI content generation pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers build and configure an automated content creation pipeline that handles research, scriptwriting, content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-crawls** trending news from sources like TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (Top Lists, POV, Case Studies, How-to) using Claude/OpenAI
- **Supports bilingual** output (English and Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager
npm --version
# or
pnpm --version
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering (Remotion)
REMOTION_WEBHOOK_SECRET=your_webhook_secret

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linter
npm run lint
```

### Remotion Video Rendering

```bash
# Render single video
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview

# Upgrade Remotion
npm run remotion:upgrade
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Content crawling modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Video rendering (Remotion)
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## API Usage Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawlers/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

export async function crawlTechNews(keyword: string): Promise<NewsSource[]> {
  const options = {
    method: 'GET',
    url: 'https://news-api.p.rapidapi.com/v1/search',
    params: {
      q: keyword,
      lang: 'en',
      sort_by: 'published_at',
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(), // Last 24h
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.slice(0, 10);
  } catch (error) {
    console.error('News crawling error:', error);
    return [];
  }
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = `You are an expert content creator specializing in ${request.format} format. 
  Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'} with a ${request.tone} tone.
  Use the provided research data to create data-backed, trending content.`;

  const userPrompt = `Create a ${request.format} article about "${request.keyword}" using this research:
  
${request.researchData}

Requirements:
- Include real data and insights from the research
- Make it engaging and trend-focused
- Optimize for social media sharing
- Add relevant statistics and quotes`;

  const message = await anthropic.messages.create({
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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} format.`,
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about "${request.keyword}" using this research:\n\n${request.researchData}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1'; // TikTok, YouTube, Instagram
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio,
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
    inputProps: composition.props,
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Implementation

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlTechNews } from '../crawlers/news-crawler';
import { generateContentWithClaude } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

export interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research & Crawl
  console.log('Step 1: Crawling news sources...');
  const newsArticles = await crawlTechNews(config.keyword);
  const researchData = newsArticles
    .map(
      (article) =>
        `Title: ${article.title}\nURL: ${article.url}\nContent: ${article.content}\n---`
    )
    .join('\n\n');

  // Step 2: Generate Content
  console.log('Step 2: Generating content with AI...');
  const generatedContent = await generateContentWithClaude({
    keyword: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData,
  });

  // Step 3: Render Video (if enabled)
  let videoPath: string | null = null;
  if (config.generateVideo) {
    console.log('Step 3: Rendering video...');
    videoPath = await renderContentVideo({
      title: config.keyword,
      content: generatedContent,
      aspectRatio: config.videoAspectRatio || '16:9',
    });
  }

  return {
    content: generatedContent,
    videoPath,
    sources: newsArticles,
    timestamp: new Date().toISOString(),
  };
}
```

## Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      videoAspectRatio: body.videoAspectRatio || '16:9',
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          generateVideo: true,
          videoAspectRatio: '16:9',
        }),
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 bg-blue-600 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3 className="font-bold">Generated Content:</h3>
          <div className="prose">{result.content}</div>
          {result.videoPath && (
            <video src={result.videoPath} controls className="mt-4" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: '#fff', fontSize: '48px' }}>{title}</h1>
        <p style={{ color: '#fff', fontSize: '24px', marginTop: '20px' }}>
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false,
      })
    )
  );
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// Use with cron or task scheduler
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  // Run daily at 9 AM
  const trendingKeywords = ['AI', 'Web3', 'Marketing Automation'];
  await generateBatchContent(trendingKeywords);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map((keyword) =>
    limit(() => runContentPipeline({ keyword, /* ... */ }))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout in Remotion config
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Claude/OpenAI Error Handling

```typescript
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContentWithClaude(request);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((resolve) => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

### Environment Variable Validation

```typescript
// src/lib/utils/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```
