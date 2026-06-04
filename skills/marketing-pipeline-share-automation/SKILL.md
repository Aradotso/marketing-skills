---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate content with automatic research crawling
  - create videos from articles using remotion
  - set up AI content pipeline with Claude and OpenAI
  - automate content from research to video generation
  - build automated marketing content workflow
  - use marketing pipeline for content automation
  - generate multilingual content with AI research
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive content automation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

- **Auto-Research**: Crawls and analyzes data from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically renders infographics and short videos using Remotion
- **Multi-Platform Export**: Optimized video output for Reels, TikTok, Shorts

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
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_STUDIO_PORT=3001
```

## Project Structure

```typescript
// Typical project structure
src/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI provider integrations
│   ├── research/          # Research/crawling logic
│   └── video/             # Remotion video generation
├── remotion/              # Remotion video templates
└── types/                 # TypeScript definitions
```

## Core API Usage

### Research & Data Crawling

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchResult {
  source: string;
  title: string;
  content: string;
  publishedAt: Date;
  insights: string[];
}

export async function crawlTechNews(
  keyword: string,
  timeRange: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      client.search({
        source,
        query: keyword,
        timeRange,
        limit: 10
      })
    )
  );
  
  return results.flat();
}

// Extract insights from research
export async function extractInsights(
  results: ResearchResult[]
): Promise<string[]> {
  const combinedContent = results
    .map(r => r.content)
    .join('\n\n');
    
  // Use AI to extract key insights
  const insights = await analyzeWithAI(combinedContent);
  return insights;
}
```

### AI Content Generation

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

export async function generateContent(
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(config);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }],
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096,
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list format',
    'pov': 'Write from a unique point of view',
    'case-study': 'Analyze as a detailed case study',
    'how-to': 'Create a step-by-step guide'
  };
  
  return `
You are a ${config.tone} content writer.
Create content in ${config.language === 'en' ? 'English' : 'Vietnamese'}.

Topic: ${config.keyword}
Format: ${formatInstructions[config.format]}

Research insights:
${config.research.join('\n')}

Requirements:
- Use data-backed insights from the research
- Make it engaging and actionable
- Include relevant statistics and examples
- Optimize for social media sharing
`;
}
```

### Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number; // in frames
  fps: number;
  aspectRatio: '16:9' | '9:16' | '1:1';
}

export async function renderContentVideo(
  content: string,
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
      keyPoints: config.keyPoints,
      content: content.slice(0, 500), // Preview
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

### Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  content,
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        padding: 60,
        fontFamily: 'Inter, sans-serif',
      }}
    >
      <div style={{ opacity }}>
        <h1
          style={{
            fontSize: 72,
            color: '#fff',
            marginBottom: 40,
            fontWeight: 'bold',
          }}
        >
          {title}
        </h1>
        
        <div style={{ fontSize: 32, color: '#eee', lineHeight: 1.6 }}>
          {keyPoints.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [30 + index * 20, 50 + index * 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div
                key={index}
                style={{
                  opacity: pointOpacity,
                  marginBottom: 20,
                  display: 'flex',
                  alignItems: 'flex-start',
                }}
              >
                <span style={{ color: '#00d4ff', marginRight: 15 }}>
                  {index + 1}.
                </span>
                <span>{point}</span>
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/workflow.ts
import { crawlTechNews, extractInsights } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Crawling research data...');
  const research = await crawlTechNews(keyword, '24h');
  const insights = await extractInsights(research);
  
  // Step 2: Generate content (bilingual)
  console.log('✍️ Generating content...');
  const [contentEN, contentVI] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      research: insights,
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      research: insights,
    }),
  ]);
  
  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(contentEN);
  
  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo(contentEN, {
    title: keyword,
    keyPoints,
    duration: 300, // 10 seconds at 30fps
    fps: 30,
    aspectRatio: '9:16', // TikTok/Reels format
  });
  
  return {
    content: {
      en: contentEN,
      vi: contentVI,
    },
    video: videoPath,
    research: insights,
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - could be enhanced with AI
  const sentences = content.split(/[.!?]+/);
  return sentences
    .filter(s => s.trim().length > 20)
    .slice(0, 5)
    .map(s => s.trim());
}
```

## Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/workflow';

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
    
    return NextResponse.json({
      success: true,
      data: result,
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

## CLI Usage (if available)

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video
npm run remotion:render

# Open Remotion studio
npm run remotion:studio
```

## Common Patterns

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
import { kv } from '@vercel/kv'; // or your preferred cache

export async function getCachedResearch(
  keyword: string
): Promise<ResearchResult[] | null> {
  const cacheKey = `research:${keyword}:24h`;
  return await kv.get(cacheKey);
}

export async function setCachedResearch(
  keyword: string,
  results: ResearchResult[]
): Promise<void> {
  const cacheKey = `research:${keyword}:24h`;
  // Cache for 1 hour
  await kv.set(cacheKey, results, { ex: 3600 });
}
```

### Error Handling with Retry

```typescript
// lib/utils/retry.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      if (i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, delay * Math.pow(2, i))
        );
      }
    }
  }
  
  throw lastError!;
}

// Usage
const content = await withRetry(() => 
  generateContent(config, 'claude')
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // limit each IP to 10 requests per windowMs
});

// Apply to API routes
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '1mb',
    },
  },
};
```

### Video Rendering Issues

If Remotion rendering fails:

1. Check Node.js version (requires 18+)
2. Ensure FFmpeg is installed
3. Verify sufficient disk space
4. Check memory limits for large videos

```bash
# Install FFmpeg on Ubuntu/Debian
sudo apt-get install ffmpeg

# Install FFmpeg on macOS
brew install ffmpeg
```

### AI Provider Timeouts

```typescript
// Increase timeout for long-running AI requests
const completion = await openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [{
    role: 'user',
    content: prompt
  }],
  max_tokens: 4096,
}, {
  timeout: 60000, // 60 seconds
});
```

### Memory Management for Large Batches

```typescript
// Process in batches to avoid memory issues
async function processInBatches<T, R>(
  items: T[],
  batchSize: number,
  processor: (batch: T[]) => Promise<R[]>
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await processor(batch);
    results.push(...batchResults);
  }
  
  return results;
}
```
