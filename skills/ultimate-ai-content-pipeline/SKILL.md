---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script writing to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build AI-powered content generation system
  - automate research and video creation pipeline
  - use Remotion for automated video rendering
  - implement end-to-end content automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A TypeScript-based automated content generation system that handles the complete content lifecycle: automated research from news sources, AI-powered script generation in multiple formats and languages, and automatic video/infographic rendering. Perfect for content creators, marketers, and businesses looking to scale content production.

## What This Project Does

The Ultimate AI Content Pipeline automates:

1. **Research Phase**: Auto-crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
2. **Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports video in ratios suitable for Reels, TikTok, Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Database (if using)
DATABASE_URL=your_database_url_here

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Modules

### 1. Research Module

Automated news crawling and insight extraction:

```typescript
import { ResearchService } from '@/lib/research/ResearchService';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch latest news on a topic
async function researchTopic(keyword: string) {
  const results = await researchService.searchNews({
    keyword,
    timeRange: '24h',
    maxResults: 50
  });

  // Extract insights
  const insights = await researchService.extractInsights(results);
  
  return {
    articles: results,
    insights,
    sources: results.map(r => r.source)
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(`Found ${research.articles.length} articles`);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/ai/ContentGenerator';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: 'claude-3-opus-20240229'
});

// Generate article
async function generateArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  research: any
) {
  const prompt = {
    topic,
    format,
    tone: 'professional', // 'friendly', 'humorous'
    language: 'vi', // or 'en'
    researchData: research.insights,
    sources: research.sources
  };

  const article = await generator.generate(prompt);
  
  return {
    title: article.title,
    content: article.content,
    language: article.language,
    metadata: {
      format,
      wordCount: article.wordCount,
      readingTime: article.readingTime
    }
  };
}

// Multi-language generation
async function generateMultiLanguage(topic: string, research: any) {
  const [viArticle, enArticle] = await Promise.all([
    generateArticle(topic, 'toplist', research),
    generator.generate({ ...research, language: 'en' })
  ]);

  return { vi: viArticle, en: enArticle };
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Video rendering service
class VideoRenderService {
  async renderVideo(content: {
    title: string;
    points: string[];
    style: 'infographic' | 'short-form';
    platform: 'reels' | 'tiktok' | 'youtube-shorts';
  }) {
    // Determine aspect ratio
    const dimensions = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      'youtube-shorts': { width: 1080, height: 1920 }
    };

    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: content.style === 'infographic' ? 'Infographic' : 'ShortForm',
      inputProps: {
        title: content.title,
        points: content.points,
        ...dimensions[content.platform]
      },
    });

    // Render video
    const outputLocation = `./output/${Date.now()}.mp4`;
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
    });

    return outputLocation;
  }
}

// Usage
const videoService = new VideoRenderService();
const videoPath = await videoService.renderVideo({
  title: 'Top 5 AI Tools for 2024',
  points: ['Tool 1', 'Tool 2', 'Tool 3', 'Tool 4', 'Tool 5'],
  style: 'infographic',
  platform: 'reels'
});
```

### 4. Complete Pipeline

Full end-to-end workflow:

```typescript
import { ContentPipeline } from '@/lib/ContentPipeline';

const pipeline = new ContentPipeline({
  researchService,
  contentGenerator: generator,
  videoRenderer: videoService
});

// Run complete pipeline
async function runPipeline(keyword: string) {
  console.log('Starting pipeline for:', keyword);

  // Step 1: Research
  console.log('🔍 Researching...');
  const research = await pipeline.research(keyword);

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'pov'],
    languages: ['vi', 'en']
  });

  // Step 3: Create Videos
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.renderVideos({
    content: content.vi.toplist,
    platforms: ['reels', 'tiktok']
  });

  // Step 4: Export results
  return {
    research: {
      articlesFound: research.articles.length,
      insights: research.insights
    },
    content: {
      vi: content.vi,
      en: content.en
    },
    videos: videos.map(v => v.path)
  };
}

// Execute
const result = await runPipeline('AI Marketing Automation');
console.log('Pipeline complete!', result);
```

## API Routes (Next.js)

### Create Content API

```typescript
// src/app/api/content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/ContentPipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const pipeline = new ContentPipeline({
      // ... configuration from env vars
    });

    const result = await pipeline.run({
      keyword,
      format,
      language
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Research API

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/ResearchService';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');

  if (!keyword) {
    return NextResponse.json({ error: 'Keyword required' }, { status: 400 });
  }

  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const results = await researchService.searchNews({
    keyword,
    timeRange: '24h'
  });

  return NextResponse.json({ results });
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion studio (for video template editing)
npm run remotion:studio
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Usage
const keywords = ['AI tools', 'Marketing automation', 'SEO trends'];
const results = await batchGenerate(keywords);
```

### Pattern 2: Scheduled Pipeline with Cron

```typescript
import { CronJob } from 'cron';

// Run pipeline daily at 8 AM
const job = new CronJob('0 8 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    try {
      await runPipeline(topic);
      console.log(`✅ Completed: ${topic}`);
    } catch (error) {
      console.error(`❌ Failed: ${topic}`, error);
    }
  }
});

job.start();
```

### Pattern 3: Custom Content Templates

```typescript
interface ContentTemplate {
  format: string;
  structure: string[];
  tone: string;
  targetAudience: string;
}

const templates: Record<string, ContentTemplate> = {
  'viral-thread': {
    format: 'twitter-thread',
    structure: ['hook', 'problem', 'solution', 'cta'],
    tone: 'engaging',
    targetAudience: 'entrepreneurs'
  },
  'linkedin-post': {
    format: 'professional-insight',
    structure: ['observation', 'analysis', 'takeaway'],
    tone: 'professional',
    targetAudience: 'business-leaders'
  }
};

async function generateFromTemplate(keyword: string, templateName: string) {
  const template = templates[templateName];
  const research = await researchTopic(keyword);
  
  return await generator.generate({
    ...template,
    keyword,
    researchData: research
  });
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function callWithRetry(fn: () => Promise<any>) {
  return pRetry(fn, {
    retries: 3,
    onFailedAttempt: error => {
      console.log(`Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`);
    }
  });
}

// Usage
const research = await callWithRetry(() => researchService.searchNews({ keyword }));
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout and add progress tracking
const videoPath = await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 120000, // 2 minutes
  onProgress: ({ progress }) => {
    console.log(`Rendering progress: ${Math.round(progress * 100)}%`);
  }
});
```

### Issue: Memory Issues with Large Batches

```typescript
import pLimit from 'p-limit';

// Limit concurrent operations
const limit = pLimit(3);

async function processBatch(items: string[]) {
  return Promise.all(
    items.map(item => limit(() => processItem(item)))
  );
}
```

### Issue: Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

## Testing

```typescript
// Example test for research service
import { describe, it, expect, vi } from 'vitest';

describe('ResearchService', () => {
  it('should fetch and parse news articles', async () => {
    const service = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!
    });

    const results = await service.searchNews({
      keyword: 'AI',
      timeRange: '24h',
      maxResults: 10
    });

    expect(results).toBeInstanceOf(Array);
    expect(results.length).toBeGreaterThan(0);
    expect(results[0]).toHaveProperty('title');
    expect(results[0]).toHaveProperty('source');
  });
});
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = cache.get(cacheKey);

  if (cached) {
    console.log('Cache hit:', keyword);
    return cached;
  }

  const research = await researchTopic(keyword);
  cache.set(cacheKey, research);
  return research;
}
```

This skill enables AI coding agents to effectively implement and customize the Ultimate AI Content Pipeline for automated content generation, from research through video creation.
