---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content from research to video
  - set up automated content workflow
  - create video content from text automatically
  - build AI-powered content generation system
  - integrate Claude and OpenAI for content automation
  - configure Remotion video rendering pipeline
  - automate social media content creation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an end-to-end content automation system that handles research, scriptwriting, content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a comprehensive content automation system that:

- **Auto-scans and researches** trending topics from TechCrunch, a16z, X (Twitter), LinkedIn
- **Generates multi-format content** (toplist, POV, case studies, how-to) in multiple languages
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multi-platform** distribution (Reels, TikTok, Shorts)
- **Provides end-to-end automation** from keyword input to published content

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
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Components and Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';

// Scrape recent content about a topic
async function gatherResearch(topic: string) {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const research = await gatherResearch(topic);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about ${topic} using this research data:
      
      ${JSON.stringify(research, null, 2)}
      
      Requirements:
      - Use data-backed insights
      - Include statistics and sources
      - Make it engaging and actionable
      - Format for both Vietnamese and English versions`
    }]
  });

  return message.content;
}
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, tone: 'professional' | 'friendly' | 'humorous') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone. Create engaging marketing content.`
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

interface VideoConfig {
  title: string;
  content: string[];
  theme: 'dark' | 'light';
  aspectRatio: '9:16' | '16:9' | '1:1';
}

async function generateVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
      theme: config.theme,
    },
  });

  const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      slides: config.content,
      theme: config.theme,
    },
  });

  return outputPath;
}
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { schedulePost } from '@/lib/social/scheduler';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
      limit: 15
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent(keyword, 'toplist');

    // Step 3: Extract key points for video
    const videoSlides = extractKeyPoints(content);

    // Step 4: Generate Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: keyword,
      content: videoSlides,
      theme: 'dark',
      aspectRatio: '9:16'
    });

    // Step 5: Schedule posts
    console.log('📅 Scheduling posts...');
    await schedulePost({
      content: content,
      media: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok'],
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      content,
      videoPath,
      research
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Helper function
function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sections
  const lines = content.split('\n');
  return lines
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, '').trim())
    .slice(0, 5); // Top 5 points
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

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
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const topic = searchParams.get('topic');
  const timeframe = searchParams.get('timeframe') || '24h';

  if (!topic) {
    return NextResponse.json(
      { error: 'Topic parameter is required' },
      { status: 400 }
    );
  }

  const research = await researchTopic({
    keyword: topic,
    timeframe,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
  });

  return NextResponse.json({ data: research });
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Type checking
npm run type-check

# Lint code
npm run lint

# Render video (Remotion)
npm run remotion:render
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(topic: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, 'toplist', 'en'),
    generateContent(topic, 'toplist', 'vi')
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Content Scheduling with Retry Logic

```typescript
async function scheduleWithRetry(post: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await schedulePost(post);
      return { success: true };
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateMultipleContents(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => generateContent(topic, 'toplist'))
    )
  );
}
```

### Video Rendering Issues

```typescript
// Handle Remotion rendering errors
try {
  await generateVideo(config);
} catch (error) {
  if (error.message.includes('ENOSPC')) {
    console.error('Not enough disk space');
    // Cleanup old videos
  } else if (error.message.includes('timeout')) {
    console.error('Rendering timeout - try reducing video length');
  }
  throw error;
}
```

### Memory Management for Large Batches

```typescript
async function processLargeBatch(items: any[], batchSize = 5) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => runContentPipeline(item))
    );
    results.push(...batchResults);
    
    // Allow garbage collection between batches
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return results;
}
```

## TypeScript Type Definitions

```typescript
// src/types/index.ts
export interface ResearchData {
  keyword: string;
  articles: Article[];
  insights: string[];
  statistics: Statistic[];
  trends: Trend[];
}

export interface Article {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  summary: string;
}

export interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

export interface VideoOutput {
  path: string;
  duration: number;
  aspectRatio: '9:16' | '16:9' | '1:1';
  format: 'mp4' | 'webm';
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI agents to effectively assist developers in building and customizing automated content generation workflows.
