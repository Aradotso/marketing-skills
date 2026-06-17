---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, script writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate videos from articles automatically
  - crawl news and create content
  - build AI content pipeline
  - create multi-language marketing content
  - automate social media video generation
  - research and generate content with AI
  - set up automated content workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Marketing Pipeline Share project - a complete AI-powered content automation system that handles research, script writing, multi-format content generation, and automatic video rendering using Claude 3, OpenAI, and Remotion.

## Overview

Marketing Pipeline Share is an end-to-end content production pipeline that:

- **Auto-crawls** recent news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- **Generates** multi-format content (lists, POV, case studies, how-tos) in multiple languages
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** output for Reels, TikTok, Shorts, and other platforms

The system is built with Next.js (TypeScript) and integrates Claude 3, OpenAI, and RapidAPI for content intelligence.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with the following:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
```

Navigate to `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawlers/          # News crawlers
│   ├── generators/        # Content generators
│   └── video/             # Remotion video rendering
├── public/                # Static assets
├── remotion/              # Remotion video templates
└── utils/                 # Utility functions
```

## Core APIs and Usage

### 1. Research Crawler

The crawler automatically fetches recent news from multiple sources:

```typescript
// lib/crawlers/news-crawler.ts
import { crawlTechCrunch, crawlA16z, crawlTwitter } from '@/lib/crawlers';

async function researchTopic(keyword: string) {
  const sources = await Promise.all([
    crawlTechCrunch(keyword, { hours: 24 }),
    crawlA16z(keyword),
    crawlTwitter(keyword, { limit: 50 })
  ]);
  
  return {
    articles: sources.flat(),
    timestamp: new Date().toISOString()
  };
}

// Usage
const research = await researchTopic("AI automation");
console.log(`Found ${research.articles.length} articles`);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
}

async function generateContent(
  topic: string,
  research: any[],
  options: ContentOptions
) {
  const prompt = buildPrompt(topic, research, options);
  
  // Using Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(topic: string, research: any[], options: ContentOptions) {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with clear rankings',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  return `
Topic: ${topic}
Format: ${options.format}
Language: ${options.language}
Tone: ${options.tone}

Research Data:
${research.map(r => `- ${r.title}: ${r.summary}`).join('\n')}

Instructions: ${formatInstructions[options.format]}

${options.language === 'vi' ? 'Viết bằng tiếng Việt tự nhiên, chuyên nghiệp.' : 'Write in natural, professional English.'}
`;
}

// Usage
const content = await generateContent(
  "AI Content Automation Trends 2026",
  research.articles,
  {
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    length: 'medium'
  }
);
```

### 3. Multi-Language Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/ai/multi-lang-generator.ts
async function generateBilingualContent(topic: string, research: any[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, research, {
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      length: 'medium'
    }),
    generateContent(topic, research, {
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      length: 'medium'
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      topic,
      generatedAt: new Date().toISOString(),
      sourceCount: research.length
    }
  };
}
```

### 4. Video Generation with Remotion

Automatically render videos from content:

```typescript
// lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { RemotionComposition } from '@/remotion/Composition';

interface VideoConfig {
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
  duration: number;
  content: {
    title: string;
    points: string[];
    images?: string[];
  };
}

const PLATFORM_CONFIGS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 30 }
};

async function renderVideo(config: VideoConfig) {
  const platformConfig = PLATFORM_CONFIGS[config.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.content.title,
      points: config.content.points,
      images: config.content.images || []
    },
  });
  
  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: config.content,
    ...platformConfig
  });
  
  return outputLocation;
}

// Usage
const videoPath = await renderVideo({
  platform: 'reels',
  duration: 30,
  content: {
    title: "Top 5 AI Automation Tools",
    points: [
      "Claude API for content generation",
      "Remotion for video automation",
      "OpenAI for research analysis"
    ],
    images: ['/images/tool1.png', '/images/tool2.png']
  }
});

console.log(`Video rendered: ${videoPath}`);
```

### 5. Complete Pipeline Orchestration

Run the entire pipeline from keyword to video:

```typescript
// lib/pipeline/orchestrator.ts
import { researchTopic } from '@/lib/crawlers/news-crawler';
import { generateBilingualContent } from '@/lib/ai/multi-lang-generator';
import { renderVideo } from '@/lib/video/video-renderer';
import { extractKeyPoints } from '@/lib/utils/content-parser';

interface PipelineResult {
  research: any;
  content: {
    en: string;
    vi: string;
  };
  videos: {
    reels: string;
    tiktok: string;
  };
  metadata: any;
}

async function runContentPipeline(keyword: string): Promise<PipelineResult> {
  console.log(`🚀 Starting pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Crawling news sources...');
  const research = await researchTopic(keyword);
  
  // Step 2: Generate Content
  console.log('🧠 Generating bilingual content...');
  const content = await generateBilingualContent(keyword, research.articles);
  
  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content.en);
  
  // Step 4: Render videos for multiple platforms
  console.log('🎬 Rendering videos...');
  const [reelsVideo, tiktokVideo] = await Promise.all([
    renderVideo({
      platform: 'reels',
      duration: 30,
      content: {
        title: keyword,
        points: keyPoints.slice(0, 5)
      }
    }),
    renderVideo({
      platform: 'tiktok',
      duration: 30,
      content: {
        title: keyword,
        points: keyPoints.slice(0, 5)
      }
    })
  ]);
  
  console.log('✅ Pipeline complete!');
  
  return {
    research,
    content,
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo
    },
    metadata: {
      keyword,
      completedAt: new Date().toISOString(),
      sourceCount: research.articles.length
    }
  };
}

// Usage
const result = await runContentPipeline("AI Marketing Automation 2026");
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, options } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Content Generated!</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Error Handling in Pipeline

```typescript
// lib/pipeline/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: string,
    public originalError?: any
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function safeExecutePipeline(keyword: string) {
  try {
    return await runContentPipeline(keyword);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Failed at ${error.stage}: ${error.message}`);
      // Implement retry logic or fallback
    }
    throw error;
  }
}
```

### Caching Research Results

```typescript
// lib/utils/cache.ts
const researchCache = new Map<string, { data: any; timestamp: number }>();
const CACHE_TTL = 3600000; // 1 hour

export async function getCachedResearch(keyword: string) {
  const cached = researchCache.get(keyword);
  
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    console.log('Using cached research');
    return cached.data;
  }
  
  const fresh = await researchTopic(keyword);
  researchCache.set(keyword, {
    data: fresh,
    timestamp: Date.now()
  });
  
  return fresh;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number, private delayMs: number) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

const claudeLimiter = new RateLimiter(5, 1000);

// Use in content generation
const content = await claudeLimiter.execute(() =>
  generateContent(topic, research, options)
);
```

### Video Rendering Memory Issues

If Remotion rendering fails due to memory:

```typescript
// Render in chunks for long videos
async function renderLongVideo(config: VideoConfig) {
  const chunkDuration = 10; // seconds
  const chunks = Math.ceil(config.duration / chunkDuration);
  
  const renderedChunks = [];
  
  for (let i = 0; i < chunks; i++) {
    const chunk = await renderVideo({
      ...config,
      duration: chunkDuration,
      content: {
        ...config.content,
        points: config.content.points.slice(i * 3, (i + 1) * 3)
      }
    });
    renderedChunks.push(chunk);
  }
  
  // Concatenate chunks (requires ffmpeg)
  return concatenateVideos(renderedChunks);
}
```

### Missing Environment Variables

```typescript
// lib/config/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnvironment() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnvironment();
```

This skill provides comprehensive guidance for AI agents to work with the Marketing Pipeline Share project, covering installation, core APIs, real-world usage patterns, and troubleshooting common issues.
