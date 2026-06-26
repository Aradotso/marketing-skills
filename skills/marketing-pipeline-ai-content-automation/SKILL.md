---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate video content from text using Remotion and AI
  - automate content research and article generation with Claude
  - create social media videos automatically from blog posts
  - crawl news sources and generate AI-powered content
  - build an automated content creation workflow
  - use the marketing pipeline to generate multilingual content
  - integrate OpenAI and Claude for content automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Ultimate AI Content Pipeline** is a TypeScript-based automated content creation system that handles the complete content lifecycle: from research and crawling news sources, to AI-powered scriptwriting (using Claude 3 and OpenAI), to automatic video generation with Remotion. It supports multiple content formats (toplist, POV, case study, how-to), multilingual output (English/Vietnamese), and automatic rendering of videos optimized for social media platforms.

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Navigate to `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content generation pipeline
│   ├── remotion/        # Remotion video compositions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── package.json
```

## Key Features & Usage

### 1. AI-Powered Content Research

The system automatically crawls and analyzes recent content from major sources (TechCrunch, a16z, Twitter/X, LinkedIn).

```typescript
// src/lib/crawler/news-crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsSource {
  source: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsSource[]> {
  const rapidClient = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];
  
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    const articles = await rapidClient.searchNews({
      q: keyword,
      domains: source,
      from: getTimeframeDate(timeframe),
      language: 'en',
      sortBy: 'publishedAt'
    });
    
    results.push(...articles);
  }
  
  return results;
}

function getTimeframeDate(timeframe: string): string {
  const now = new Date();
  if (timeframe === '24h') {
    now.setHours(now.getHours() - 24);
  } else if (timeframe === '7d') {
    now.setDate(now.getDate() - 7);
  }
  return now.toISOString();
}
```

### 2. AI Content Generation with Claude/OpenAI

Generate content in multiple formats with customizable tone and language.

```typescript
// src/lib/ai/content-generator.ts
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
  researchData: NewsSource[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and tech content.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096
  });
  
  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(request: ContentRequest): string {
  const researchSummary = request.researchData
    .map(item => `- ${item.title}\n  Source: ${item.source}\n  Summary: ${item.content.substring(0, 200)}...`)
    .join('\n\n');
  
  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${request.tone}
Format: ${request.format}

Research Data:
${researchSummary}

Requirements:
- Use the latest data from research sources
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for SEO
${request.format === 'toplist' ? '- Include numbered list with clear rankings' : ''}
${request.format === 'how-to' ? '- Provide step-by-step instructions' : ''}
${request.format === 'case-study' ? '- Include real-world examples and results' : ''}
`.trim();
}
```

### 3. Video Generation with Remotion

Automatically convert content into social media-ready videos.

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'youtube-shorts';
  backgroundColor?: string;
  accentColor?: string;
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      backgroundColor: config.backgroundColor || '#1a1a2e',
      accentColor: config.accentColor || '#0f3460',
    },
  });
  
  const dimensions = getVideoDimensions(config.format);
  
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.format}.mp4`
  );
  
  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
  });
  
  return outputPath;
}

function getVideoDimensions(format: VideoConfig['format']) {
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'youtube-shorts': { width: 1080, height: 1920 },
  };
  
  return dimensions[format];
}
```

### 4. Remotion Video Composition

```typescript
// src/remotion/ContentVideo.tsx
import React from 'react';
import {
  AbsoluteFill,
  useCurrentFrame,
  useVideoConfig,
  interpolate,
  spring,
} from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  backgroundColor: string;
  accentColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  backgroundColor,
  accentColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleProgress = spring({
    frame,
    fps,
    config: {
      damping: 100,
    },
  });
  
  const contentProgress = spring({
    frame: frame - 20,
    fps,
    config: {
      damping: 100,
    },
  });
  
  const titleOpacity = interpolate(titleProgress, [0, 1], [0, 1]);
  const titleTranslateY = interpolate(titleProgress, [0, 1], [-50, 0]);
  
  const contentOpacity = interpolate(contentProgress, [0, 1], [0, 1]);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor,
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div
        style={{
          opacity: titleOpacity,
          transform: `translateY(${titleTranslateY}px)`,
          color: 'white',
          fontSize: 72,
          fontWeight: 'bold',
          textAlign: 'center',
          marginBottom: 40,
          maxWidth: '90%',
        }}
      >
        {title}
      </div>
      
      <div
        style={{
          opacity: contentOpacity,
          color: 'white',
          fontSize: 36,
          textAlign: 'center',
          maxWidth: '80%',
          lineHeight: 1.6,
          borderLeft: `8px solid ${accentColor}`,
          paddingLeft: 30,
        }}
      >
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/content/pipeline.ts
import { crawlRecentNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/video-generator';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'youtube-shorts';
}

interface PipelineResult {
  content: string;
  videoPath?: string;
  researchSources: NewsSource[];
  generatedAt: Date;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Crawl and research
  console.log('🔍 Researching content...');
  const researchData = await crawlRecentNews(config.keyword, '24h');
  
  // Step 2: Generate content with AI
  console.log('🧠 Generating content with AI...');
  const content = await generateContent({
    keyword: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData,
  });
  
  // Step 3: Generate video (optional)
  let videoPath: string | undefined;
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Generating video...');
    videoPath = await generateVideo({
      content: content.substring(0, 300), // Excerpt for video
      title: config.keyword,
      format: config.videoFormat,
    });
  }
  
  return {
    content,
    videoPath,
    researchSources: researchData,
    generatedAt: new Date(),
  };
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/content/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat,
    });
    
    return NextResponse.json({
      success: true,
      data: result,
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

### Usage Example

```bash
curl -X POST http://localhost:3000/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "keyword": "AI automation trends 2024",
    "format": "toplist",
    "language": "en",
    "tone": "expert",
    "generateVideo": true,
    "videoFormat": "reels"
  }'
```

## Common Patterns

### Batch Content Generation

```typescript
// src/scripts/batch-generate.ts
import { runContentPipeline } from '@/lib/content/pipeline';

const keywords = [
  'AI automation',
  'Marketing trends 2024',
  'Content strategy',
];

async function batchGenerate() {
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      generateVideo: true,
      videoFormat: 'reels',
    });
    
    console.log(`✅ Generated content for: ${keyword}`);
    console.log(`📝 Content length: ${result.content.length} chars`);
    if (result.videoPath) {
      console.log(`🎬 Video saved to: ${result.videoPath}`);
    }
  }
}

batchGenerate();
```

### Multilingual Content Generation

```typescript
async function generateMultilingual(keyword: string) {
  const languages: Language[] = ['en', 'vi'];
  const results = [];
  
  for (const language of languages) {
    const result = await runContentPipeline({
      keyword,
      format: 'how-to',
      language,
      tone: 'friendly',
      generateVideo: false,
    });
    
    results.push({
      language,
      content: result.content,
    });
  }
  
  return results;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys before running pipeline
function validateAPIKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

export async function checkRateLimit(identifier: string) {
  const { success, limit, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error(`Rate limit exceeded. Try again later.`);
  }
  
  return { remaining, limit };
}
```

### Video Rendering Errors

```typescript
// Handle Remotion rendering errors
export async function safeVideoGeneration(config: VideoConfig) {
  try {
    const videoPath = await generateVideo(config);
    return { success: true, videoPath };
  } catch (error) {
    console.error('Video generation failed:', error);
    
    // Fallback: save content as image instead
    const imagePath = await generateStaticImage(config);
    return { success: false, fallbackPath: imagePath };
  }
}
```

### Memory Management for Large Content

```typescript
// Stream large content instead of loading all at once
import { Transform } from 'stream';

export function processLargeContent(content: string) {
  const chunks = content.match(/.{1,1000}/g) || [];
  
  return chunks.map((chunk, index) => ({
    id: index,
    content: chunk,
    timestamp: new Date(),
  }));
}
```

## CLI Commands

```bash
# Development
npm run dev                  # Start dev server
npm run build               # Build for production
npm run start               # Start production server

# Remotion Video
npm run remotion:preview    # Preview Remotion compositions
npm run remotion:render     # Render video from CLI

# Database
npm run db:push             # Push schema changes
npm run db:migrate          # Run migrations

# Testing
npm run test                # Run tests
npm run type-check          # TypeScript type checking
```

## Performance Optimization

### Caching Research Data

```typescript
import { Redis } from '@upstash/redis';

const redis = Redis.fromEnv();

export async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  const fresh = await crawlRecentNews(keyword);
  await redis.set(cacheKey, JSON.stringify(fresh), { ex: 3600 });
  
  return fresh;
}
```

This skill provides comprehensive guidance for using the Marketing Pipeline AI Content Automation system to create automated content workflows with AI-powered research, writing, and video generation.
