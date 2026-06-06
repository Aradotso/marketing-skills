---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate content with automatic research and video
  - create automated marketing content with AI
  - build content from research to video automatically
  - use the marketing content automation pipeline
  - generate articles and videos with AI research
  - set up automated content creation workflow
  - configure the content pipeline with Claude and OpenAI
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from automatic research and data crawling to AI-powered writing and video generation using Claude, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research** from TechCrunch, a16z, X (Twitter), LinkedIn for fresh insights
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multilingual output** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Provides end-to-end automation** from keyword input to publishable content

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
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Additional APIs for enhanced research
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Auto-research crawlers
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API Patterns

### 1. Research Automation

```typescript
import { autoResearch } from '@/lib/research/crawler';

interface ResearchOptions {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange?: '24h' | '7d' | '30d';
  maxResults?: number;
}

async function gatherResearch(options: ResearchOptions) {
  const { keyword, sources, timeRange = '24h', maxResults = 10 } = options;
  
  const researchData = await autoResearch({
    query: keyword,
    sources,
    filters: {
      publishedAfter: timeRange,
      limit: maxResults,
    },
  });
  
  // Returns structured data with insights, sources, and statistics
  return {
    insights: researchData.insights,
    sources: researchData.sources,
    statistics: researchData.stats,
    trendingTopics: researchData.trending,
  };
}

// Usage example
const data = await gatherResearch({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 15,
});
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, ContentTone, Language } from '@/types/content';

interface ContentGenerationParams {
  keyword: string;
  researchData: any;
  format: ContentFormat;
  language: Language;
  tone: ContentTone;
  targetLength?: number;
}

async function createContent(params: ContentGenerationParams) {
  const {
    keyword,
    researchData,
    format = 'article',
    language = 'en',
    tone = 'professional',
    targetLength = 1500,
  } = params;
  
  // Choose AI provider based on requirements
  const provider = format === 'case-study' ? 'claude' : 'openai';
  
  const content = await generateContent({
    provider,
    prompt: {
      keyword,
      research: researchData,
      format,
      tone,
      language,
      wordCount: targetLength,
    },
  });
  
  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    summary: content.summary,
    keywords: content.keywords,
  };
}

// Usage with Claude
const article = await createContent({
  keyword: 'AI content automation',
  researchData: data,
  format: 'how-to',
  language: 'en',
  tone: 'friendly',
  targetLength: 2000,
});
```

### 3. Bilingual Content Generation

```typescript
import { generateBilingual } from '@/lib/ai/bilingual-generator';

async function createBilingualContent(keyword: string, researchData: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      researchData,
      format: 'toplist',
      language: 'en',
      tone: 'professional',
    }),
    generateContent({
      keyword,
      researchData,
      format: 'toplist',
      language: 'vi',
      tone: 'professional',
    }),
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    // Optionally sync metadata across languages
    sharedMetadata: {
      keywords: [...englishContent.keywords, ...vietnameseContent.keywords],
      createdAt: new Date().toISOString(),
    },
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

interface VideoRenderOptions {
  content: any;
  template: VideoTemplate;
  platform: 'reels' | 'tiktok' | 'shorts' | 'landscape';
  duration?: number;
}

async function generateVideoFromContent(options: VideoRenderOptions) {
  const { content, template = 'infographic', platform = 'reels', duration = 60 } = options;
  
  // Platform-specific aspect ratios
  const aspectRatios = {
    reels: { width: 1080, height: 1920 },    // 9:16
    tiktok: { width: 1080, height: 1920 },   // 9:16
    shorts: { width: 1080, height: 1920 },   // 9:16
    landscape: { width: 1920, height: 1080 }, // 16:9
  };
  
  const videoConfig = {
    composition: template,
    inputProps: {
      title: content.title,
      points: content.body.split('\n').filter(Boolean).slice(0, 5),
      theme: 'modern',
      background: 'gradient',
    },
    dimensions: aspectRatios[platform],
    fps: 30,
    durationInFrames: duration * 30,
  };
  
  const renderedVideo = await renderVideo(videoConfig);
  
  return {
    videoPath: renderedVideo.path,
    thumbnail: renderedVideo.thumbnail,
    duration: renderedVideo.duration,
    size: renderedVideo.fileSize,
  };
}

// Usage
const video = await generateVideoFromContent({
  content: article,
  template: 'infographic',
  platform: 'reels',
  duration: 45,
});
```

### 5. Complete Pipeline Integration

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    enableVideo: true,
    enableBilingual: true,
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
  });
  
  // Step 2: Generate Content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional',
  });
  
  // Step 3: Create Videos
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok'],
    templates: ['infographic', 'animated-text'],
  });
  
  return {
    research,
    content,
    videos,
    metadata: {
      keyword,
      generatedAt: new Date().toISOString(),
      pipeline: pipeline.getStats(),
    },
  };
}

// Execute complete pipeline
const result = await runFullContentPipeline('AI marketing trends 2026');
```

## Configuration

### AI Provider Configuration

```typescript
// src/lib/ai/config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7,
    apiKey: process.env.ANTHROPIC_API_KEY,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
    apiKey: process.env.OPENAI_API_KEY,
  },
};

export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
  },
  pov: {
    structure: 'opinion-based',
    includeSources: true,
  },
  'case-study': {
    structure: 'problem-solution',
    includeData: true,
  },
  'how-to': {
    structure: 'step-by-step',
    includeExamples: true,
  },
};
```

### Research Crawler Configuration

```typescript
// src/lib/research/config.ts
export const researchConfig = {
  sources: {
    techcrunch: {
      endpoint: 'https://techcrunch.com/wp-json/tc/v1/magazine',
      rateLimit: 10, // requests per minute
    },
    twitter: {
      apiVersion: 'v2',
      maxTweets: 100,
      rateLimit: 15,
    },
    linkedin: {
      apiVersion: 'v2',
      maxPosts: 50,
      rateLimit: 10,
    },
  },
  defaultFilters: {
    language: ['en', 'vi'],
    minEngagement: 10,
    excludeRetweets: true,
  },
};
```

### Remotion Video Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setCodec('h264');
Config.setCrf(18);
Config.setPixelFormat('yuv420p');

export const videoTemplates = {
  infographic: {
    composition: 'Infographic',
    defaultProps: {
      theme: 'modern',
      accentColor: '#3b82f6',
    },
  },
  'animated-text': {
    composition: 'AnimatedText',
    defaultProps: {
      fontFamily: 'Inter',
      animation: 'fade-slide',
    },
  },
};
```

## Common Workflows

### Workflow 1: Quick Article Generation

```typescript
import { quickGenerate } from '@/lib/workflows/quick-generate';

async function generateQuickArticle(topic: string) {
  const result = await quickGenerate({
    keyword: topic,
    format: 'article',
    language: 'en',
    autoPublish: false,
  });
  
  console.log('Generated:', result.content.title);
  console.log('Word count:', result.content.wordCount);
  console.log('Research sources:', result.research.sources.length);
  
  return result;
}
```

### Workflow 2: Multi-Platform Content Creation

```typescript
async function createMultiPlatformContent(keyword: string) {
  const pipeline = new ContentPipeline();
  
  // Research once
  const research = await pipeline.research({ keyword });
  
  // Generate content variants
  const variants = await Promise.all([
    // Long-form blog post
    pipeline.generateContent({
      research,
      format: 'case-study',
      language: 'en',
      tone: 'professional',
      targetLength: 2500,
    }),
    // Social media post
    pipeline.generateContent({
      research,
      format: 'toplist',
      language: 'en',
      tone: 'friendly',
      targetLength: 500,
    }),
    // Vietnamese version
    pipeline.generateContent({
      research,
      format: 'how-to',
      language: 'vi',
      tone: 'professional',
      targetLength: 1500,
    }),
  ]);
  
  // Generate videos for social variants
  const videos = await Promise.all(
    variants.slice(1).map(content =>
      pipeline.renderVideos({
        content,
        platforms: ['reels', 'tiktok', 'shorts'],
      })
    )
  );
  
  return { variants, videos };
}
```

### Workflow 3: Scheduled Content Generation

```typescript
import { scheduleContentGeneration } from '@/lib/workflows/scheduler';

async function setupContentSchedule() {
  const schedule = await scheduleContentGeneration({
    keywords: [
      'AI automation trends',
      'marketing technology',
      'content creation tools',
    ],
    frequency: 'daily',
    time: '09:00',
    timezone: 'Asia/Ho_Chi_Minh',
    pipeline: {
      research: true,
      generateContent: true,
      renderVideo: true,
      languages: ['en', 'vi'],
    },
  });
  
  return schedule;
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const pipeline = new ContentPipeline();
    const research = await pipeline.research({ keyword });
    const content = await pipeline.generateContent({
      research,
      format: format || 'article',
      language: language || 'en',
      tone: tone || 'professional',
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Render Video Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { contentId, platform, template } = body;
    
    // Fetch content from database or cache
    const content = await getContentById(contentId);
    
    const video = await renderVideo({
      content,
      template: template || 'infographic',
      platform: platform || 'reels',
    });
    
    return NextResponse.json({
      success: true,
      video: {
        url: video.path,
        thumbnail: video.thumbnail,
        duration: video.duration,
      },
    });
  } catch (error) {
    console.error('Video render error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting with exponential backoff
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000, // 1 minute
  retryStrategy: 'exponential',
});

async function rateLimitedRequest(fn: () => Promise<any>) {
  return await limiter.execute(fn);
}
```

### Issue: Video Rendering Timeouts

```typescript
// Increase timeout for video rendering
import { renderMedia } from '@remotion/renderer';

const video = await renderMedia({
  composition,
  inputProps,
  codec: 'h264',
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
  },
});
```

### Issue: Research Data Quality

```typescript
// Filter and validate research data
function validateResearchData(data: any) {
  return data.filter(item => {
    return (
      item.content.length > 100 &&
      item.publishedAt &&
      item.engagement > 10 &&
      !item.isSpam
    );
  });
}
```

### Issue: Memory Usage with Large Content

```typescript
// Stream large content processing
import { Transform } from 'stream';

async function processLargeContent(content: string) {
  const chunks = content.match(/.{1,1000}/g) || [];
  const processed = [];
  
  for (const chunk of chunks) {
    const result = await processChunk(chunk);
    processed.push(result);
    // Allow garbage collection
    await new Promise(resolve => setImmediate(resolve));
  }
  
  return processed.join('');
}
```

## Performance Optimization

```typescript
// Cache research data
import { cache } from '@/lib/utils/cache';

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await cache.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const fresh = await autoResearch({ query: keyword });
  await cache.set(cacheKey, fresh, 3600); // 1 hour TTL
  
  return fresh;
}

// Parallel processing
async function parallelContentGeneration(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runFullContentPipeline(keyword))
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

This skill provides comprehensive guidance for AI agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research to video generation.
