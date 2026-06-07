---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content from research to video
  - create automated content workflow with Claude and OpenAI
  - build AI-powered content generation system
  - set up automated marketing pipeline
  - generate videos from text content automatically
  - scrape and analyze news for content creation
  - create multilingual marketing content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and data scraping to scriptwriting, multilingual content generation, and automatic video rendering.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scrapes research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates AI content**: Uses Claude 3 and OpenAI to create diverse content formats (top lists, POVs, case studies, how-tos)
- **Multilingual output**: Automatically generates English and Vietnamese versions
- **Video rendering**: Uses Remotion to transform text content into videos and infographics
- **Platform optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping/research
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database connection
DATABASE_URL=your_database_connection_string

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Key Architecture

The system follows a pipeline architecture:

1. **Research Module** → Scrapes and aggregates data
2. **Content Generation Module** → AI-powered content creation
3. **Translation Module** → Multilingual adaptation
4. **Video Rendering Module** → Visual content generation
5. **Publishing Module** → Distribution to platforms

## Core Usage Patterns

### Basic Content Generation Pipeline

```typescript
import { ContentPipeline } from './lib/content-pipeline';
import { ClaudeProvider } from './lib/ai-providers/claude';
import { OpenAIProvider } from './lib/ai-providers/openai';

// Initialize the pipeline
const pipeline = new ContentPipeline({
  aiProvider: new ClaudeProvider({
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  }),
  language: 'vi', // or 'en'
  contentFormat: 'toplist' // or 'pov', 'case-study', 'how-to'
});

// Generate content from a keyword
async function generateContent(keyword: string) {
  try {
    // Step 1: Research
    const research = await pipeline.research(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: '24h',
      maxArticles: 10
    });

    // Step 2: Generate content
    const content = await pipeline.generate({
      keyword,
      research,
      tone: 'professional', // or 'friendly', 'humorous'
      targetAudience: 'marketers'
    });

    // Step 3: Create multilingual versions
    const translated = await pipeline.translate(content, ['en', 'vi']);

    return {
      original: content,
      translations: translated,
      research
    };
  } catch (error) {
    console.error('Content generation failed:', error);
    throw error;
  }
}
```

### Research and Data Scraping

```typescript
import { ResearchEngine } from './lib/research/engine';

const researchEngine = new ResearchEngine({
  rapidApiKey: process.env.RAPIDAPI_KEY
});

async function scrapeLatestNews(topic: string) {
  const results = await researchEngine.scrape({
    topic,
    sources: {
      techcrunch: {
        enabled: true,
        categories: ['ai', 'startups', 'apps']
      },
      twitter: {
        enabled: true,
        hashtags: [`#${topic}`, '#AI', '#Marketing'],
        influencers: ['@elonmusk', '@sama']
      },
      linkedin: {
        enabled: true,
        keywords: [topic, 'trends', 'innovation']
      }
    },
    timeRange: '24h',
    maxResults: 20
  });

  // Extract insights
  const insights = await researchEngine.analyzeInsights(results);

  return {
    rawData: results,
    insights,
    summary: insights.summary,
    keyStats: insights.statistics
  };
}
```

### Content Format Templates

```typescript
import { ContentFormatter } from './lib/formatters';

const formatter = new ContentFormatter();

// Top List format
async function generateTopList(data: any) {
  return await formatter.format({
    type: 'toplist',
    data,
    structure: {
      title: true,
      introduction: true,
      items: {
        count: 10,
        includeStats: true,
        includeImages: true
      },
      conclusion: true,
      cta: true
    }
  });
}

// POV (Point of View) format
async function generatePOV(data: any) {
  return await formatter.format({
    type: 'pov',
    data,
    structure: {
      hook: true,
      perspective: 'expert', // or 'consumer', 'analyst'
      argument: true,
      evidence: true,
      counterpoint: true,
      conclusion: true
    }
  });
}

// Case Study format
async function generateCaseStudy(data: any) {
  return await formatter.format({
    type: 'case-study',
    data,
    structure: {
      background: true,
      challenge: true,
      solution: true,
      results: true,
      metrics: true,
      lessons: true
    }
  });
}
```

### Video Generation with Remotion

```typescript
import { VideoRenderer } from './lib/video/renderer';
import { RemotionComposition } from './remotion/compositions';

const videoRenderer = new VideoRenderer({
  remotionRoot: './remotion',
  outputDir: './output/videos'
});

async function renderContentVideo(content: any) {
  const composition = new RemotionComposition({
    type: 'infographic',
    content: {
      title: content.title,
      points: content.keyPoints,
      stats: content.statistics,
      branding: {
        logo: './assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4', '#45B7D1']
      }
    },
    duration: 60, // seconds
    fps: 30,
    dimensions: {
      width: 1080,
      height: 1920 // 9:16 for Reels/TikTok/Shorts
    }
  });

  const video = await videoRenderer.render(composition, {
    codec: 'h264',
    quality: 'high',
    format: 'mp4'
  });

  return {
    videoPath: video.outputPath,
    thumbnail: video.thumbnailPath,
    duration: video.duration,
    size: video.fileSize
  };
}
```

### Multi-Platform Export

```typescript
import { PlatformOptimizer } from './lib/platform/optimizer';

const optimizer = new PlatformOptimizer();

async function exportToPlatforms(content: any, video: any) {
  // Optimize for different platforms
  const exports = await Promise.all([
    // TikTok/Reels/Shorts (9:16)
    optimizer.optimize({
      platform: 'shorts',
      content,
      video,
      aspectRatio: '9:16',
      maxDuration: 60,
      captions: true
    }),

    // Instagram Feed (1:1)
    optimizer.optimize({
      platform: 'instagram-feed',
      content,
      video,
      aspectRatio: '1:1',
      maxDuration: 60,
      overlay: true
    }),

    // YouTube (16:9)
    optimizer.optimize({
      platform: 'youtube',
      content,
      video,
      aspectRatio: '16:9',
      maxDuration: 300,
      chapters: true
    })
  ]);

  return exports;
}
```

### Full Pipeline Example

```typescript
import { FullContentPipeline } from './lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new FullContentPipeline({
    aiProviders: {
      primary: new ClaudeProvider({ apiKey: process.env.ANTHROPIC_API_KEY }),
      fallback: new OpenAIProvider({ apiKey: process.env.OPENAI_API_KEY })
    },
    research: {
      rapidApiKey: process.env.RAPIDAPI_KEY
    },
    video: {
      enabled: true,
      platforms: ['tiktok', 'reels', 'shorts', 'youtube']
    },
    languages: ['en', 'vi']
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    contentTypes: ['toplist', 'pov', 'case-study'],
    generateVideo: true,
    autoPublish: false // Set to true for automatic publishing
  });

  console.log('Pipeline completed:', {
    articlesGenerated: result.articles.length,
    videosCreated: result.videos.length,
    languages: result.languages,
    totalAssets: result.assets.length
  });

  return result;
}

// Usage
runFullPipeline('AI Marketing Automation')
  .then(result => console.log('Success:', result))
  .catch(error => console.error('Pipeline error:', error));
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering locally
npm run render

# Test pipeline
npm run test:pipeline

# Lint and format
npm run lint
npm run format
```

## API Configuration

### Claude AI Provider

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, context: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    temperature: 0.7,
    messages: [
      {
        role: 'user',
        content: `${context}\n\n${prompt}`
      }
    ]
  });

  return message.content[0].text;
}
```

### OpenAI Provider

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: context },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

## Common Patterns

### Error Handling and Retry Logic

```typescript
import { retry } from './lib/utils/retry';

async function resilientContentGeneration(keyword: string) {
  return await retry(
    async () => {
      return await pipeline.generate({ keyword });
    },
    {
      maxRetries: 3,
      backoff: 'exponential',
      onRetry: (error, attempt) => {
        console.log(`Retry attempt ${attempt} after error:`, error.message);
      }
    }
  );
}
```

### Caching Research Results

```typescript
import { CacheManager } from './lib/cache';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  storage: 'redis' // or 'memory', 'file'
});

async function cachedResearch(topic: string) {
  const cacheKey = `research:${topic}`;
  
  // Check cache first
  const cached = await cache.get(cacheKey);
  if (cached) return cached;

  // Perform research
  const research = await researchEngine.scrape({ topic });

  // Store in cache
  await cache.set(cacheKey, research);

  return research;
}
```

### Batch Processing

```typescript
import { BatchProcessor } from './lib/batch';

async function batchGenerateContent(keywords: string[]) {
  const processor = new BatchProcessor({
    concurrency: 3,
    retryFailed: true
  });

  const results = await processor.process(keywords, async (keyword) => {
    return await generateContent(keyword);
  });

  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function rateLimitedGeneration(prompt: string) {
  await limiter.wait();
  return await anthropic.messages.create({ /* ... */ });
}
```

### Video Rendering Issues

If video rendering fails:

```bash
# Check Remotion installation
npm ls remotion

# Verify FFmpeg is installed
ffmpeg -version

# Test rendering with sample composition
npx remotion render ./remotion/index.ts HelloWorld ./output/test.mp4
```

### Memory Management for Large Batches

```typescript
import { MemoryManager } from './lib/utils/memory';

const memoryManager = new MemoryManager({
  maxMemoryUsage: 0.8, // 80% of available memory
  gcInterval: 60000 // Run garbage collection every minute
});

async function processLargeBatch(items: any[]) {
  for (const chunk of chunkArray(items, 10)) {
    await memoryManager.checkAndOptimize();
    await processBatch(chunk);
  }
}
```

### Debugging Pipeline Steps

```typescript
import { PipelineDebugger } from './lib/debug';

const debugger = new PipelineDebugger({
  logLevel: 'verbose',
  saveIntermediateResults: true,
  outputDir: './debug'
});

pipeline.use(debugger.middleware());

// Run with debugging
await pipeline.execute({ keyword: 'AI Marketing' });

// Check debug logs
console.log(debugger.getLogs());
console.log(debugger.getTimings());
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI coding agents to effectively assist developers in implementing automated content creation workflows from research through video generation.
