---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate my content creation workflow
  - set up AI content pipeline with video generation
  - create automated marketing content system
  - build content from research to video automatically
  - use Claude and OpenAI for content automation
  - generate videos from written content automatically
  - set up automated content research and publishing
  - create AI-powered content generation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete automated content creation system that handles everything from research and scriptwriting to video generation. The system crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-researches** trending topics from multiple news sources and social media
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using AI
- **Supports multi-language** output (English and Vietnamese by default)
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_API_KEY=your_linkedin_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```typescript
// Start the Next.js development server
npm run dev
// or
yarn dev

// Server runs on http://localhost:3000
```

## Key Architecture & Components

### 1. Content Research Module

```typescript
// lib/research/crawler.ts
import { AutoResearcher } from '@/lib/research';

interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
}

async function conductResearch(config: ResearchConfig) {
  const researcher = new AutoResearcher({
    apiKey: process.env.RAPIDAPI_KEY,
    twitterToken: process.env.TWITTER_BEARER_TOKEN,
  });

  const results = await researcher.scan({
    keyword: config.keyword,
    sources: config.sources,
    timeframe: config.timeframe,
    limit: config.maxResults,
  });

  return {
    articles: results.articles,
    insights: results.extractedInsights,
    trends: results.trendingTopics,
    statistics: results.dataPoints,
  };
}

// Usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h',
  maxResults: 20,
});
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'casestudy' | 'howto';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4000,
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      toplist: 'Create a numbered list article with detailed explanations',
      pov: 'Write from a personal perspective with strong opinions',
      casestudy: 'Analyze a real-world example with data and outcomes',
      howto: 'Provide step-by-step instructions with actionable tips',
    };

    return `
You are a ${config.tone} content writer. 
Format: ${formatInstructions[config.format]}
Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create compelling, data-backed content that engages readers.
Include statistics, quotes, and actionable insights.
`;
  }
}

// Usage
const generator = new ContentGenerator();
const content = await generator.generateWithClaude({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research,
});
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'infographic';
}

class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const dimensions = this.getPlatformDimensions(config.platform);
    const compositionId = `ContentVideo-${config.style}`;

    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        content: config.content,
        title: config.title,
        ...dimensions,
      },
    });

    // Render video
    const outputPath = path.resolve(`./output/${Date.now()}.mp4`);
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        content: config.content,
        title: config.title,
      },
    });

    return outputPath;
  }

  private getPlatformDimensions(platform: string) {
    const dimensions = {
      reels: { width: 1080, height: 1920, fps: 30 },
      tiktok: { width: 1080, height: 1920, fps: 30 },
      shorts: { width: 1080, height: 1920, fps: 30 },
    };
    return dimensions[platform as keyof typeof dimensions];
  }
}

// Usage
const renderer = new VideoRenderer();
const videoPath = await renderer.renderContentVideo({
  content: content,
  title: 'Top 5 AI Automation Tools',
  platform: 'reels',
  style: 'dynamic',
});
```

### 4. Complete Pipeline API

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { AutoResearcher } from '@/lib/research';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, platform, generateVideo } = req.body;

    // Step 1: Research
    const researcher = new AutoResearcher({
      apiKey: process.env.RAPIDAPI_KEY!,
    });
    
    const research = await researcher.scan({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      limit: 15,
    });

    // Step 2: Generate Content
    const generator = new ContentGenerator();
    const content = await generator.generateWithClaude({
      format,
      language,
      tone: 'expert',
      researchData: research,
    });

    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const renderer = new VideoRenderer();
      const videoPath = await renderer.renderContentVideo({
        content,
        title: keyword,
        platform,
        style: 'dynamic',
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return res.status(200).json({
      success: true,
      data: {
        content,
        research: {
          articles: research.articles.length,
          insights: research.extractedInsights,
        },
        videoUrl,
      },
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({
      error: 'Content generation failed',
      message: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Common Usage Patterns

### Full Pipeline Execution

```typescript
// app/pipeline.ts
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline() {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const result = await pipeline.execute({
    keyword: 'AI marketing automation',
    contentFormats: ['toplist', 'howto'],
    languages: ['en', 'vi'],
    generateVideos: true,
    platforms: ['reels', 'tiktok'],
  });

  console.log('Generated content:', result.content);
  console.log('Video URLs:', result.videos);
  console.log('Research insights:', result.insights);
}
```

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
async function batchGenerate(keywords: string[]) {
  const generator = new ContentGenerator();
  const researcher = new AutoResearcher({
    apiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researcher.scan({
        keyword,
        sources: ['techcrunch'],
        timeframe: '24h',
        limit: 10,
      });

      const content = await generator.generateWithClaude({
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: research,
      });

      return { keyword, content, research };
    })
  );

  return results;
}
```

### Scheduling Automated Posts

```typescript
// lib/scheduler.ts
import cron from 'node-cron';

class ContentScheduler {
  startAutoPipeline() {
    // Run every day at 9 AM
    cron.schedule('0 9 * * *', async () => {
      const pipeline = new ContentPipeline({
        anthropicKey: process.env.ANTHROPIC_API_KEY!,
        openaiKey: process.env.OPENAI_API_KEY!,
        rapidApiKey: process.env.RAPIDAPI_KEY!,
      });

      const trending = await this.getTrendingTopics();
      
      for (const topic of trending) {
        await pipeline.execute({
          keyword: topic,
          contentFormats: ['toplist'],
          languages: ['en', 'vi'],
          generateVideos: true,
          platforms: ['reels'],
        });
      }
    });
  }

  private async getTrendingTopics(): Promise<string[]> {
    // Implement trending topic detection
    return ['AI automation', 'Marketing trends', 'Content creation'];
  }
}
```

## Remotion Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  title: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, title }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        <div style={{ color: '#e0e0e0', fontSize: 36, lineHeight: 1.6 }}>
          {content.split('\n').map((line, i) => (
            <p key={i}>{line}</p>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: Map<string, number[]> = new Map();

  async throttle(key: string, maxRequests: number, windowMs: number) {
    const now = Date.now();
    const requests = this.requests.get(key) || [];
    
    const validRequests = requests.filter(time => now - time < windowMs);
    
    if (validRequests.length >= maxRequests) {
      const oldestRequest = validRequests[0];
      const waitTime = windowMs - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    validRequests.push(now);
    this.requests.set(key, validRequests);
  }
}

// Usage
const limiter = new RateLimiter();
await limiter.throttle('anthropic', 50, 60000); // 50 requests per minute
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safeExecute<T>(
  fn: () => Promise<T>,
  stage: PipelineError['stage']
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    throw new PipelineError(
      `Failed at ${stage} stage`,
      stage,
      error instanceof Error ? error : undefined
    );
  }
}
```

### Video Rendering Issues

```typescript
// Common fix for Remotion bundling errors
// Ensure webpack config in next.config.js:

/** @type {import('next').NextConfig} */
const nextConfig = {
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
        path: false,
      };
    }
    return config;
  },
};

module.exports = nextConfig;
```

## Performance Optimization

```typescript
// lib/cache/content-cache.ts
import { Redis } from 'ioredis';

class ContentCache {
  private redis: Redis;

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async cacheResearch(keyword: string, data: any, ttl = 3600) {
    await this.redis.setex(
      `research:${keyword}`,
      ttl,
      JSON.stringify(data)
    );
  }

  async getResearch(keyword: string) {
    const cached = await this.redis.get(`research:${keyword}`);
    return cached ? JSON.parse(cached) : null;
  }
}
```

This skill provides comprehensive guidance for AI agents to effectively use the Marketing Pipeline AI Content Automation system for automated content creation, research, and video generation workflows.
