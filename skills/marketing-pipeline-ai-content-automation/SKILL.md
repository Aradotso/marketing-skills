---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content research and generation
  - set up an AI content pipeline with video rendering
  - create automated marketing content with Claude API
  - build a content automation system with AI research
  - generate videos from AI-written content automatically
  - configure an end-to-end content marketing pipeline
  - automate content from research to video with TypeScript
  - use Remotion for automated video content generation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a TypeScript-based automated content generation system that handles research, scriptwriting, and video generation end-to-end using Claude/OpenAI APIs and Remotion.

## What This Project Does

The marketing-pipeline-share project is a complete content automation pipeline that:

- **Auto-crawls** news and insights from TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (listicles, POV pieces, case studies, how-tos) using Claude 3 or OpenAI
- **Supports bilingual output** (English and Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn/pnpm required
node --version  # Should be 18+
```

### Clone and Install

```bash
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

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video rendering settings
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js settings
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Key Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Research Module

```typescript
// src/lib/research/crawler.ts
import { fetchLatestNews } from '@/lib/research/sources';

interface ResearchConfig {
  sources: string[];
  keywords: string[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

async function researchContent(config: ResearchConfig) {
  const { sources, keywords, timeRange, language = 'en' } = config;
  
  const newsData = await fetchLatestNews({
    sources: sources,
    keywords: keywords,
    since: timeRange,
  });
  
  // Filter and rank by relevance
  const insights = newsData
    .filter(item => item.relevanceScore > 0.7)
    .sort((a, b) => b.relevanceScore - a.relevanceScore);
  
  return insights;
}

// Usage
const research = await researchContent({
  sources: ['techcrunch', 'a16z', 'twitter'],
  keywords: ['AI', 'marketing automation'],
  timeRange: '24h',
  language: 'en'
});
```

### 2. AI Content Generation

```typescript
// src/lib/ai/generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

async function generateContent(params: ContentGenerationParams) {
  const { topic, format, tone, language, researchData } = params;
  
  const systemPrompt = `You are an expert content creator specializing in ${format} articles.
Tone: ${tone}. Language: ${language}.
Use the following research data to create an engaging, data-backed article.`;

  const researchContext = researchData
    .map(item => `- ${item.title}: ${item.summary}`)
    .join('\n');

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Topic: ${topic}\n\nResearch Data:\n${researchContext}\n\nCreate a ${format} article.`
    }],
    system: systemPrompt,
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage
const article = await generateContent({
  topic: 'AI in Marketing 2026',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  researchData: research
});
```

### 3. Alternative: OpenAI Integration

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(params: ContentGenerationParams) {
  const { topic, format, tone, language, researchData } = params;
  
  const researchContext = researchData
    .map(item => `${item.title}: ${item.summary}`)
    .join('\n\n');

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Expert ${format} content creator. Tone: ${tone}. Language: ${language}.`
      },
      {
        role: 'user',
        content: `Topic: ${topic}\n\nResearch:\n${researchContext}\n\nWrite a ${format} article.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function renderVideo(config: VideoConfig) {
  const { content, title, format, duration } = config;
  
  // Video dimensions by platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      content,
      duration,
    },
  });

  const outputPath = path.resolve(`./output/${title.replace(/\s+/g, '-')}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
      duration,
    },
  });

  return outputPath;
}

// Usage
const videoPath = await renderVideo({
  content: article,
  title: 'AI Marketing Trends 2026',
  format: 'reels',
  duration: 30
});
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
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
        padding: 60,
      }}
    >
      <div style={{ opacity, transform: `scale(${scale})` }}>
        <h1 style={{ color: 'white', fontSize: 72, textAlign: 'center', marginBottom: 40 }}>
          {title}
        </h1>
        <p style={{ color: '#cccccc', fontSize: 36, textAlign: 'center', lineHeight: 1.6 }}>
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/orchestrator.ts
import { researchContent } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  topic: string;
  keywords: string[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Researching content...');
  const research = await researchContent({
    sources: ['techcrunch', 'a16z', 'twitter'],
    keywords: config.keywords,
    timeRange: '24h',
    language: config.language,
  });

  console.log(`✅ Found ${research.length} relevant articles`);

  console.log('✍️ Step 2: Generating content...');
  const article = await generateContent({
    topic: config.topic,
    format: config.format,
    tone: config.tone,
    language: config.language,
    researchData: research,
  });

  console.log('✅ Content generated successfully');

  let videoPath: string | null = null;

  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Step 3: Rendering video...');
    videoPath = await renderVideo({
      content: article,
      title: config.topic,
      format: config.videoFormat,
      duration: 30,
    });
    console.log(`✅ Video rendered: ${videoPath}`);
  }

  return {
    article,
    videoPath,
    research: research.slice(0, 5), // Top 5 sources
  };
}

// Usage
const result = await runContentPipeline({
  topic: '5 AI Marketing Trends You Need to Know in 2026',
  keywords: ['AI marketing', 'automation', 'trends 2026'],
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  generateVideo: true,
  videoFormat: 'reels',
});

console.log('Article:', result.article);
console.log('Video:', result.videoPath);
console.log('Sources:', result.research);
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      topic: body.topic,
      keywords: body.keywords || [body.topic],
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      language: body.language || 'en',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reels',
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple articles in parallel
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      runContentPipeline({
        topic,
        keywords: [topic],
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        generateVideo: false,
      })
    )
  );
  
  return results;
}
```

### Bilingual Content

```typescript
// Generate content in both languages
async function generateBilingual(config: Omit<PipelineConfig, 'language'>) {
  const [english, vietnamese] = await Promise.all([
    runContentPipeline({ ...config, language: 'en' }),
    runContentPipeline({ ...config, language: 'vi' }),
  ]);
  
  return { english, vietnamese };
}
```

### Content Scheduling

```typescript
// Schedule content generation
import { scheduleJob } from 'node-schedule';

// Run every day at 9 AM
scheduleJob('0 9 * * *', async () => {
  const result = await runContentPipeline({
    topic: 'Daily AI News Roundup',
    keywords: ['AI', 'technology', 'news'],
    format: 'toplist',
    tone: 'friendly',
    language: 'en',
    generateVideo: true,
    videoFormat: 'shorts',
  });
  
  console.log('Scheduled content generated:', result);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
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
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const article = await retryWithBackoff(() =>
  generateContent(params)
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: {
    title,
    content,
    duration,
  },
});
```

### Memory Issues with Large Content

```typescript
// Chunk large content for processing
function chunkContent(content: string, chunkSize = 500) {
  const words = content.split(' ');
  const chunks = [];
  
  for (let i = 0; i < words.length; i += chunkSize) {
    chunks.push(words.slice(i, i + chunkSize).join(' '));
  }
  
  return chunks;
}
```

### Environment Variable Validation

```typescript
// src/lib/config/validate.ts
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateEnv();
```

## Performance Optimization

### Caching Research Results

```typescript
// Use Redis or in-memory cache
import { LRUCache } from 'lru-cache';

const researchCache = new LRUCache<string, any>({
  max: 100,
  ttl: 1000 * 60 * 60, // 1 hour
});

async function cachedResearch(config: ResearchConfig) {
  const cacheKey = JSON.stringify(config);
  const cached = researchCache.get(cacheKey);
  
  if (cached) {
    console.log('Using cached research data');
    return cached;
  }
  
  const result = await researchContent(config);
  researchCache.set(cacheKey, result);
  
  return result;
}
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to effectively assist developers in building automated content generation workflows.
