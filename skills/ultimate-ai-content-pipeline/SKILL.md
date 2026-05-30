---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos from keywords using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content from keyword to video automatically
  - use the marketing pipeline for content creation
  - automate content research and video generation
  - build an AI content workflow with Remotion
  - set up automated content creation pipeline
  - configure Claude and OpenAI for content automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - an end-to-end content automation system that researches topics, generates multi-format content, and renders videos automatically using Claude, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a TypeScript-based Next.js application that automates the entire content creation workflow:

1. **Research Phase**: Crawls and analyzes recent data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Uses Claude 3/OpenAI to create content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese)
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-platform Output**: Exports content optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file with the required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── page.tsx           # Main page
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── research/         # Content research modules
│   └── video/            # Remotion video generation
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Render videos (if separate command exists)
npm run render
```

## Core API Usage

### 1. Research & Content Generation

```typescript
// lib/ai/research.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function researchTopic(keyword: string) {
  const crawledData = await crawlRecentNews(keyword);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Analyze this data and extract key insights for the topic "${keyword}": ${JSON.stringify(crawledData)}`
    }]
  });
  
  return message.content;
}

async function crawlRecentNews(keyword: string) {
  // RapidAPI or custom crawler implementation
  const response = await fetch('https://api.rapidapi.com/news', {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    }
  });
  
  return response.json();
}
```

### 2. Multi-Format Content Generation

```typescript
// lib/ai/content-generator.ts
import { OpenAI } from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'professional' | 'friendly' | 'humorous';

export async function generateContent(
  research: string,
  format: ContentFormat,
  language: Language,
  tone: Tone
) {
  const prompts = {
    'toplist': 'Create a top 10 list article',
    'pov': 'Write a point-of-view opinion piece',
    'case-study': 'Develop a detailed case study',
    'how-to': 'Create a step-by-step how-to guide'
  };

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Write in ${language}. Format: ${prompts[format]}.`
      },
      {
        role: 'user',
        content: `Based on this research, create content:\n\n${research}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: {
    title: string;
    points: string[];
    imageUrls?: string[];
  },
  platform: 'reels' | 'tiktok' | 'shorts' = 'reels'
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      platform,
    },
  });

  const outputLocation = `out/${content.title.replace(/\s+/g, '-')}.mp4`;

  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      platform,
    },
  });

  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  content: {
    title: string;
    points: string[];
    imageUrls?: string[];
  };
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const aspectRatios = {
    reels: '9:16',
    tiktok: '9:16',
    shorts: '9:16',
  };

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40, color: 'white' }}>
        <h1 style={{ fontSize: 48, marginBottom: 30 }}>{content.title}</h1>
        {content.points.map((point, idx) => {
          const pointFrame = frame - (idx * fps * 2);
          const pointOpacity = interpolate(pointFrame, [0, 20], [0, 1], {
            extrapolateLeft: 'clamp',
            extrapolateRight: 'clamp',
          });
          
          return (
            <p key={idx} style={{ fontSize: 24, opacity: pointOpacity, marginBottom: 20 }}>
              • {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/ai/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const research = await researchTopic(keyword);
    
    return NextResponse.json({ research });
  } catch (error) {
    console.error('Research error:', error);
    return NextResponse.json(
      { error: 'Failed to research topic' },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const { research, format, language, tone } = await request.json();
    
    const content = await generateContent(research, format, language, tone);
    
    return NextResponse.json({ content });
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const { content, platform } = await request.json();
    
    const videoPath = await renderContentVideo(content, platform);
    
    return NextResponse.json({ videoPath });
  } catch (error) {
    console.error('Render error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Complete Workflow Example

```typescript
// lib/pipeline/full-workflow.ts
import { researchTopic } from '@/lib/ai/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export async function runFullPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    tone: 'professional' | 'friendly' | 'humorous';
    platform: 'reels' | 'tiktok' | 'shorts';
  }
) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic(keyword);
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const contentText = await generateContent(
    research,
    options.format,
    options.language,
    options.tone
  );
  
  // Step 3: Parse content for video
  const content = {
    title: keyword,
    points: extractKeyPoints(contentText),
    imageUrls: [], // Optional: extract or generate images
  };
  
  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo(content, options.platform);
  
  return {
    research,
    contentText,
    videoPath,
  };
}

function extractKeyPoints(text: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = text.split('\n').filter(line => line.trim());
  return lines.slice(0, 5); // Take first 5 points
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
// lib/pipeline/batch-process.ts
export async function batchProcessKeywords(
  keywords: string[],
  options: PipelineOptions
) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runFullPipeline(keyword, options)
        .catch(error => ({
          keyword,
          error: error.message
        }))
    )
  );
  
  return results;
}
```

### Scheduling Content

```typescript
// lib/scheduling/scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  keyword: string,
  schedule: string, // cron format: '0 9 * * *'
  options: PipelineOptions
) {
  cron.schedule(schedule, async () => {
    console.log(`Generating content for: ${keyword}`);
    try {
      await runFullPipeline(keyword, options);
    } catch (error) {
      console.error(`Failed to generate content:`, error);
    }
  });
}
```

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
const cache = new Map<string, { data: any; timestamp: number }>();
const CACHE_DURATION = 24 * 60 * 60 * 1000; // 24 hours

export async function getCachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  
  if (cached && Date.now() - cached.timestamp < CACHE_DURATION) {
    return cached.data;
  }
  
  const fresh = await researchTopic(keyword);
  cache.set(keyword, { data: fresh, timestamp: Date.now() });
  
  return fresh;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function rateLimitedRequests<T>(
  tasks: (() => Promise<T>)[]
): Promise<T[]> {
  return Promise.all(tasks.map(task => limit(task)));
}
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    public stage: 'research' | 'generation' | 'rendering',
    message: string,
    public originalError?: Error
  ) {
    super(`[${stage}] ${message}`);
    this.name = 'PipelineError';
  }
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

If video rendering fails:

1. Check Remotion installation: `npm list @remotion/renderer`
2. Ensure FFmpeg is installed: `ffmpeg -version`
3. Verify input props match composition schema
4. Check available disk space for output files
5. Review Remotion logs in console

### API Key Issues

```typescript
// lib/utils/validate-env.ts
export function validateEnvironment() {
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

// Call in app initialization
validateEnvironment();
```

## Performance Optimization

### Parallel Processing

```typescript
// lib/pipeline/parallel-pipeline.ts
export async function runParallelPipeline(keyword: string, options: PipelineOptions) {
  const [research] = await Promise.all([
    researchTopic(keyword),
    // Pre-warm other services
  ]);
  
  const [contentEN, contentVI] = await Promise.all([
    generateContent(research, options.format, 'en', options.tone),
    generateContent(research, options.format, 'vi', options.tone),
  ]);
  
  return { research, contentEN, contentVI };
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline, enabling AI coding agents to assist developers in automating their entire content creation workflow from research to video rendering.
