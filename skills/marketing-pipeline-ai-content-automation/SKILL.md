---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) with TypeScript
triggers:
  - how do I automate content creation with AI
  - generate blog posts and videos automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build AI-powered content generation system
  - automate research to video content creation
  - use marketing pipeline for content automation
  - integrate AI content generation in TypeScript
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a TypeScript-based system that automates the entire content creation workflow: from research and script writing to automated posting and video generation. It integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

**Key capabilities:**
- Auto-crawl fresh news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese)
- Auto-generate videos and infographics using Remotion
- Optimize content for multiple platforms (Reels, TikTok, Shorts)

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
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Data Collection
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Video Generation
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── generation/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function collectResearch(topic: string) {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });

  return {
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.statistics
  };
}

// Example usage
const aiResearch = await collectResearch('artificial intelligence trends');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  researchData: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `
    Based on this research data: ${JSON.stringify(researchData)}
    
    Create a ${format} article in ${language} language.
    Make it engaging, data-backed, and optimized for social media.
  `;

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

  return message.content[0].text;
}
```

### 3. Multi-Format Content Generation

```typescript
import { generateContent } from '@/lib/generation/content-generator';

interface ContentOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
}

async function createMultiFormatContent(options: ContentOptions) {
  const results = [];

  for (const language of options.languages) {
    const content = await generateContent({
      topic: options.topic,
      format: options.format,
      tone: options.tone,
      language: language,
      aiProvider: 'claude' // or 'openai'
    });

    results.push({
      language,
      title: content.title,
      body: content.body,
      metadata: content.metadata
    });
  }

  if (options.includeVideo) {
    const videoData = await generateVideoScript(results[0]);
    results[0].video = videoData;
  }

  return results;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(
  contentData: {
    title: string;
    points: string[];
    images: string[];
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const compositionId = 'ContentVideo';
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      images: contentData.images,
      ...dimensions[platform]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${platform}-${Date.now()}.mp4`,
    inputProps: composition.defaultProps,
  });
}
```

### 5. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    autoPost: false,
    generateVideo: true
  });

  // Step 1: Research
  const research = await pipeline.research(keyword, {
    sources: ['techcrunch', 'a16z', 'twitter'],
    depth: 'deep'
  });

  // Step 2: Generate content in multiple formats
  const content = await pipeline.generateContent(research, {
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    tone: 'professional'
  });

  // Step 3: Generate videos
  const videos = await pipeline.generateVideos(content, {
    platforms: ['reels', 'tiktok', 'shorts'],
    style: 'modern'
  });

  // Step 4: Optional auto-post
  if (pipeline.config.autoPost) {
    await pipeline.publish(content, videos);
  }

  return {
    research,
    content,
    videos
  };
}
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const research = await researchTopic({
      keyword,
      sources,
      timeRange
    });

    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/generation/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, tone } = await request.json();

  try {
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      aiProvider: 'claude'
    });

    return NextResponse.json({ success: true, data: content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Workflows

### Workflow 1: Daily Trend Analysis & Content

```typescript
async function dailyTrendContent() {
  const topics = ['AI', 'Web3', 'SaaS', 'Marketing'];
  
  for (const topic of topics) {
    const research = await researchTopic({
      keyword: topic,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    if (research.trendScore > 0.7) {
      await createMultiFormatContent({
        topic: research.topHeadline,
        format: 'pov',
        tone: 'professional',
        languages: ['en', 'vi'],
        includeVideo: true
      });
    }
  }
}
```

### Workflow 2: Batch Video Generation

```typescript
async function batchVideoGeneration(contentList: any[]) {
  const videos = await Promise.all(
    contentList.map(async (content) => {
      return await generateContentVideo(
        {
          title: content.title,
          points: content.keyPoints,
          images: content.images
        },
        'reels'
      );
    })
  );

  return videos;
}
```

## Type Definitions

```typescript
// types/content.ts
export interface ResearchData {
  keyword: string;
  insights: Insight[];
  trends: Trend[];
  statistics: Statistic[];
  sources: Source[];
}

export interface Insight {
  text: string;
  relevance: number;
  source: string;
  timestamp: Date;
}

export interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  structure: string[];
  minLength: number;
  maxLength: number;
}

export interface GeneratedContent {
  title: string;
  body: string;
  metadata: {
    format: string;
    language: string;
    wordCount: number;
    readingTime: number;
  };
  seo: {
    keywords: string[];
    metaDescription: string;
  };
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start with specific port
npm run dev -- -p 3001

# Build for production
npm run build

# Start production server
npm start
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

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
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Generation Memory Issues

If Remotion runs out of memory:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

```typescript
// Render videos sequentially instead of parallel
for (const content of contentList) {
  await generateContentVideo(content, 'reels');
}
```

## Best Practices

1. **Cache research data** to avoid repeated API calls
2. **Use rate limiters** for external APIs
3. **Implement retry logic** for AI generation failures
4. **Store generated content** in a database before publishing
5. **Test video rendering** locally before batch generation
6. **Monitor API costs** for Claude/OpenAI usage
7. **Use environment-specific configs** for development vs production
