---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, video creation, and social media publishing using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos automatically from blog posts
  - crawl news and create social media content automatically
  - use marketing-pipeline-share for content automation
  - create multi-language content with AI research
  - render videos from text content using Remotion
  - build automated content workflow with TypeScript
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, an end-to-end content automation system that handles research (crawling news sources), script generation (using Claude/OpenAI), and video rendering (with Remotion). The pipeline transforms keywords into publish-ready content across multiple formats and languages.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Uses Claude 3 or OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Output**: Generates English and Vietnamese versions simultaneously
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── research/     # News crawling modules
│   │   ├── ai/           # Claude/OpenAI integrations
│   │   ├── video/        # Remotion video generation
│   │   └── utils/        # Helper functions
│   ├── types/            # TypeScript type definitions
│   └── config/           # Configuration files
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key Components & Usage

### 1. Research Module (Auto-Crawling)

```typescript
import { ResearchService } from '@/lib/research/service';

// Initialize research service
const researcher = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl news by keyword
async function gatherResearch(keyword: string) {
  const results = await researcher.crawl({
    query: keyword,
    timeRange: '24h',
    maxResults: 50,
    language: ['en', 'vi']
  });

  // Filter and extract insights
  const insights = await researcher.extractInsights(results);
  
  return {
    rawData: results,
    insights: insights,
    trends: researcher.analyzeTrends(results)
  };
}

// Example usage
const research = await gatherResearch('AI automation tools');
console.log(research.insights);
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { ContentGenerator } from '@/lib/ai/generator';

// Initialize with preferred AI provider
const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content from research
async function createContent(research: ResearchData, format: ContentFormat) {
  const content = await generator.generate({
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    research: research.insights,
    tone: 'professional', // 'expert' | 'friendly' | 'humorous'
    languages: ['en', 'vi'],
    includeData: true, // Include statistics and citations
    wordCount: 1500
  });

  return content;
}

// Generate multiple formats
const formats: ContentFormat[] = ['toplist', 'pov', 'case-study'];
const articles = await Promise.all(
  formats.map(format => createContent(research, format))
);
```

### 3. Content Types & Prompts

```typescript
import { PromptBuilder } from '@/lib/ai/prompts';

const promptBuilder = new PromptBuilder();

// Toplist format
const toplistPrompt = promptBuilder.build({
  type: 'toplist',
  topic: 'AI content tools',
  items: 10,
  criteria: ['features', 'pricing', 'ease of use'],
  includeResearch: true
});

// POV (Point of View) format
const povPrompt = promptBuilder.build({
  type: 'pov',
  topic: 'Future of marketing automation',
  perspective: 'industry expert',
  arguments: research.insights.keyPoints
});

// Case study format
const caseStudyPrompt = promptBuilder.build({
  type: 'case-study',
  company: 'Example Corp',
  challenge: research.insights.problems[0],
  solution: 'AI automation',
  results: research.insights.outcomes
});
```

### 4. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Initialize video renderer
const videoRenderer = new VideoRenderer({
  composition: 'ContentVideo',
  outputFormat: 'mp4'
});

// Render video from content
async function renderContentVideo(content: GeneratedContent) {
  const videoConfig = {
    title: content.title,
    script: content.body,
    voiceOver: content.audioScript,
    visuals: await generateVisuals(content),
    duration: 60, // seconds
    platform: 'instagram-reels' // 'tiktok' | 'youtube-shorts'
  };

  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  const video = await renderMedia({
    composition: videoConfig,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/${content.slug}.mp4`,
    inputProps: videoConfig
  });

  return video;
}
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation pipeline
const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  },
  generation: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    formats: ['toplist', 'pov'],
    languages: ['en', 'vi']
  },
  video: {
    enabled: true,
    platforms: ['instagram-reels', 'tiktok'],
    outputDir: './output/videos'
  }
});

// Execute complete pipeline
async function automateContent(keyword: string) {
  try {
    const result = await pipeline.execute({
      keyword: keyword,
      generateVideo: true,
      autoPublish: false // Set to true to auto-post
    });

    console.log('Pipeline Results:');
    console.log('- Articles generated:', result.articles.length);
    console.log('- Videos rendered:', result.videos.length);
    console.log('- Languages:', result.languages);
    
    return result;
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run pipeline
const output = await automateContent('AI marketing tools 2024');
```

## API Routes (Next.js)

### Create Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/service';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await request.json();
    
    const researcher = new ResearchService({
      apiKey: process.env.RAPIDAPI_KEY,
      sources: sources
    });

    const results = await researcher.crawl({
      query: keyword,
      timeRange: timeRange || '24h',
      maxResults: 50
    });

    return NextResponse.json({
      success: true,
      data: results
    });
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
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/ai/generator';

export async function POST(request: NextRequest) {
  try {
    const { research, format, tone, languages } = await request.json();
    
    const generator = new ContentGenerator({
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    const content = await generator.generate({
      format: format,
      research: research,
      tone: tone || 'professional',
      languages: languages || ['en', 'vi']
    });

    return NextResponse.json({
      success: true,
      content: content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { content, platform, duration } = await request.json();
    
    const renderer = new VideoRenderer({
      composition: 'ContentVideo',
      outputFormat: 'mp4'
    });

    const video = await renderer.render({
      title: content.title,
      script: content.body,
      platform: platform || 'instagram-reels',
      duration: duration || 60
    });

    return NextResponse.json({
      success: true,
      videoUrl: video.url,
      videoPath: video.path
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Type Definitions

```typescript
// src/types/content.ts
export interface ResearchData {
  rawData: NewsArticle[];
  insights: Insight[];
  trends: Trend[];
  timestamp: Date;
}

export interface NewsArticle {
  title: string;
  source: string;
  url: string;
  publishedAt: Date;
  content: string;
  author?: string;
}

export interface Insight {
  topic: string;
  keyPoints: string[];
  statistics: Statistic[];
  quotes: Quote[];
  relevanceScore: number;
}

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

export interface GeneratedContent {
  id: string;
  title: string;
  slug: string;
  format: ContentFormat;
  body: string;
  language: string;
  metadata: ContentMetadata;
  audioScript?: string;
  createdAt: Date;
}

export interface ContentMetadata {
  keywords: string[];
  readTime: number;
  wordCount: number;
  tone: 'expert' | 'friendly' | 'humorous';
  sources: string[];
}

export interface VideoConfig {
  title: string;
  script: string;
  voiceOver?: string;
  visuals: Visual[];
  duration: number;
  platform: 'instagram-reels' | 'tiktok' | 'youtube-shorts';
  aspectRatio?: '9:16' | '16:9' | '1:1';
}
```

## Configuration Files

### Pipeline Configuration

```typescript
// src/config/pipeline.ts
export const pipelineConfig = {
  research: {
    defaultSources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    defaultTimeRange: '24h',
    maxResultsPerSource: 20,
    cacheEnabled: true,
    cacheTTL: 3600 // seconds
  },
  
  generation: {
    defaultProvider: 'claude',
    models: {
      claude: 'claude-3-opus-20240229',
      openai: 'gpt-4-turbo-preview'
    },
    defaultTone: 'professional',
    defaultLanguages: ['en', 'vi'],
    maxRetries: 3,
    timeout: 60000 // ms
  },
  
  video: {
    defaultPlatform: 'instagram-reels',
    defaultDuration: 60,
    defaultFps: 30,
    quality: 'high',
    formats: {
      'instagram-reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'youtube-shorts': { width: 1080, height: 1920 }
    }
  }
};
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import { CronJob } from 'cron';
import { ContentPipeline } from '@/lib/pipeline';

// Run pipeline daily at 9 AM
const dailyContentJob = new CronJob(
  '0 9 * * *',
  async () => {
    const pipeline = new ContentPipeline(config);
    
    const keywords = [
      'AI marketing trends',
      'content automation tools',
      'social media strategy'
    ];
    
    for (const keyword of keywords) {
      await pipeline.execute({
        keyword: keyword,
        generateVideo: true,
        autoPublish: false
      });
    }
  },
  null,
  true,
  'America/New_York'
);

dailyContentJob.start();
```

### Pattern 2: Batch Processing with Queue

```typescript
import Queue from 'bull';
import { ContentPipeline } from '@/lib/pipeline';

const contentQueue = new Queue('content-generation', {
  redis: {
    host: process.env.REDIS_HOST,
    port: parseInt(process.env.REDIS_PORT || '6379')
  }
});

// Add jobs to queue
async function queueContentGeneration(keywords: string[]) {
  for (const keyword of keywords) {
    await contentQueue.add('generate', {
      keyword: keyword,
      priority: 'normal'
    });
  }
}

// Process queue
contentQueue.process('generate', async (job) => {
  const { keyword } = job.data;
  const pipeline = new ContentPipeline(config);
  
  const result = await pipeline.execute({
    keyword: keyword,
    generateVideo: true
  });
  
  return result;
});
```

### Pattern 3: Multi-language Content Variants

```typescript
async function generateMultiLanguageContent(keyword: string) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const research = await gatherResearch(keyword);
  
  // Generate for each language
  const languages = ['en', 'vi', 'es', 'fr'];
  const variants = await Promise.all(
    languages.map(async (lang) => {
      const content = await generator.generate({
        format: 'toplist',
        research: research.insights,
        tone: 'professional',
        languages: [lang],
        wordCount: 1500
      });
      
      return {
        language: lang,
        content: content
      };
    })
  );
  
  return variants;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const contentWithRetry = await retry(
  () => generator.generate(params),
  {
    maxAttempts: 5,
    delayMs: 1000,
    backoffMultiplier: 2,
    onRetry: (attempt, error) => {
      console.log(`Retry attempt ${attempt}:`, error.message);
    }
  }
);
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and fallbacks
try {
  const video = await videoRenderer.render(videoConfig);
} catch (error) {
  if (error.code === 'TIMEOUT') {
    // Reduce video quality or duration
    videoConfig.quality = 'medium';
    videoConfig.duration = 30;
    return await videoRenderer.render(videoConfig);
  }
  
  if (error.code === 'OUT_OF_MEMORY') {
    // Process in smaller chunks
    return await videoRenderer.renderChunked(videoConfig);
  }
  
  throw error;
}
```

### Issue: Research Returns No Results

```typescript
// Implement fallback sources and broader queries
async function resilientResearch(keyword: string) {
  let results = await researcher.crawl({
    query: keyword,
    timeRange: '24h'
  });
  
  if (results.length === 0) {
    // Try broader time range
    results = await researcher.crawl({
      query: keyword,
      timeRange: '7d'
    });
  }
  
  if (results.length === 0) {
    // Use related keywords
    const relatedKeywords = await findRelatedKeywords(keyword);
    results = await researcher.crawl({
      query: relatedKeywords[0],
      timeRange: '24h'
    });
  }
  
  return results;
}
```

### Issue: Claude/OpenAI Token Limits

```typescript
// Split content generation into chunks
async function generateLongContent(research: ResearchData, targetWords: number) {
  const sections = splitIntoSections(research, targetWords);
  
  const generatedSections = await Promise.all(
    sections.map(async (section) => {
      return await generator.generate({
        format: 'partial',
        research: section,
        wordCount: section.targetWords
      });
    })
  );
  
  // Combine sections
  return combineContent(generatedSections);
}
```

## Performance Optimization

```typescript
// Cache research results
import { LRUCache } from 'lru-cache';

const researchCache = new LRUCache<string, ResearchData>({
  max: 100,
  ttl: 1000 * 60 * 60 // 1 hour
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  if (researchCache.has(cacheKey)) {
    return researchCache.get(cacheKey);
  }
  
  const research = await gatherResearch(keyword);
  researchCache.set(cacheKey, research);
  
  return research;
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share automation system, enabling AI coding agents to assist developers in building and customizing automated content workflows from research through video generation.
