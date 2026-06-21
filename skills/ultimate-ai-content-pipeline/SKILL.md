---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multi-format writing, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from articles automatically
  - create content pipeline with Claude and OpenAI
  - build automated marketing content system
  - set up AI content research and writing
  - implement content to video automation
  - create multi-format content with AI
  - automate social media content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

The Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from real-time research scraping, AI-powered article generation in multiple formats, to automatic video rendering. It integrates Claude 3, OpenAI, and Remotion to transform keywords into publish-ready content across multiple platforms.

## What It Does

This pipeline automates:
- **Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn
- **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Generation**: Automatically renders videos and infographics using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env` file with the following required variables:

```env
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key

# Remotion License (if using)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

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
│   │   ├── scraper/     # Content scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Content Scraping

```typescript
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Scrape recent news from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeRecentNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent({
    data: newsData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return insights;
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types/content';

// Generate multi-format content
async function createArticle(topic: string, research: any) {
  const contentConfig = {
    topic,
    research,
    format: 'toplist' as ContentFormat, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en' as Language, // 'en' | 'vi'
    tone: 'expert' as Tone, // 'expert' | 'friendly' | 'humorous'
    targetAudience: 'marketers',
    includeStats: true
  };

  const article = await generateContent({
    ...contentConfig,
    provider: 'claude',
    model: 'claude-3-sonnet-20240229'
  });

  return article;
}
```

### 3. Multi-language Generation

```typescript
import { generateBilingual } from '@/lib/ai/bilingual-generator';

// Generate content in both English and Vietnamese
async function createBilingualContent(topic: string, research: any) {
  const bilingualContent = await generateBilingual({
    topic,
    research,
    format: 'case-study',
    tone: 'expert',
    primaryLanguage: 'en',
    secondaryLanguage: 'vi'
  });

  return {
    english: bilingualContent.en,
    vietnamese: bilingualContent.vi,
    metadata: bilingualContent.metadata
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { VideoTemplate } from '@/types/video';

// Render video from article content
async function generateVideoFromArticle(article: any) {
  const videoConfig = {
    template: 'infographic' as VideoTemplate, // 'infographic' | 'talking-points' | 'shorts'
    content: {
      title: article.title,
      keyPoints: article.keyPoints,
      statistics: article.stats,
      brandColors: ['#FF6B6B', '#4ECDC4']
    },
    format: {
      width: 1080,
      height: 1920, // 9:16 for Reels/TikTok/Shorts
      fps: 30,
      duration: 60 // seconds
    },
    output: {
      format: 'mp4',
      quality: 'high'
    }
  };

  const video = await renderVideo(videoConfig);
  
  return {
    url: video.url,
    thumbnail: video.thumbnail,
    duration: video.duration
  };
}
```

### 5. Complete Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Execute the full pipeline
async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    apiKeys: {
      openai: process.env.OPENAI_API_KEY,
      anthropic: process.env.ANTHROPIC_API_KEY,
      rapidapi: process.env.RAPIDAPI_KEY
    }
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate content in multiple formats
  const contents = await pipeline.generateMultiFormat({
    research,
    formats: ['toplist', 'case-study', 'how-to'],
    languages: ['en', 'vi']
  });

  // Step 3: Generate videos
  const videos = await pipeline.renderVideos({
    contents,
    templates: ['infographic', 'shorts'],
    platforms: ['tiktok', 'reels', 'youtube-shorts']
  });

  return {
    research,
    contents,
    videos
  };
}
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, formats, languages } = await req.json();

    const pipeline = new ContentPipeline({
      apiKeys: {
        openai: process.env.OPENAI_API_KEY,
        anthropic: process.env.ANTHROPIC_API_KEY,
        rapidapi: process.env.RAPIDAPI_KEY
      }
    });

    const result = await pipeline.execute({
      keyword,
      formats: formats || ['toplist'],
      languages: languages || ['en']
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(req: NextRequest) {
  try {
    const { content, template, format } = await req.json();

    const video = await renderVideo({
      template,
      content,
      format: format || {
        width: 1080,
        height: 1920,
        fps: 30
      }
    });

    return NextResponse.json({ video });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Workflows

### Workflow 1: Daily Trend Content

```typescript
import { scheduleDailyContent } from '@/lib/scheduler';

// Automate daily content creation
async function setupDailyTrends() {
  await scheduleDailyContent({
    keywords: ['AI trends', 'Marketing automation', 'Social media'],
    formats: ['toplist', 'pov'],
    languages: ['en', 'vi'],
    schedule: '0 8 * * *', // Every day at 8 AM
    autoPost: true,
    platforms: ['facebook', 'linkedin', 'twitter']
  });
}
```

### Workflow 2: Competitive Analysis

```typescript
import { analyzeCompetitors } from '@/lib/ai/competitor-analyzer';

async function createCompetitorContent(competitor: string) {
  // Scrape competitor content
  const competitorData = await analyzeCompetitors({
    competitor,
    sources: ['website', 'social-media', 'blog'],
    analysisDepth: 'deep'
  });

  // Generate competitive response content
  const responseContent = await generateContent({
    topic: `How we compare to ${competitor}`,
    research: competitorData,
    format: 'case-study',
    tone: 'expert',
    includeComparison: true
  });

  return responseContent;
}
```

### Workflow 3: Multi-Platform Campaign

```typescript
async function createCampaign(topic: string) {
  const pipeline = new ContentPipeline({
    apiKeys: {
      openai: process.env.OPENAI_API_KEY,
      anthropic: process.env.ANTHROPIC_API_KEY,
      rapidapi: process.env.RAPIDAPI_KEY
    }
  });

  // Generate content for all platforms
  const campaign = await pipeline.createMultiPlatformCampaign({
    topic,
    platforms: {
      blog: { format: 'how-to', length: 'long' },
      linkedin: { format: 'pov', length: 'medium' },
      twitter: { format: 'toplist', length: 'short' },
      tiktok: { type: 'video', template: 'shorts' },
      instagram: { type: 'video', template: 'reels' }
    },
    languages: ['en', 'vi']
  });

  return campaign;
}
```

## TypeScript Types

```typescript
// src/types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type AIProvider = 'claude' | 'openai';

export interface ContentConfig {
  topic: string;
  research?: any;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience?: string;
  includeStats?: boolean;
  wordCount?: number;
}

export interface Article {
  title: string;
  content: string;
  keyPoints: string[];
  stats?: Array<{ label: string; value: string }>;
  metadata: {
    format: ContentFormat;
    language: Language;
    wordCount: number;
    generatedAt: Date;
  };
}

// src/types/video.ts
export type VideoTemplate = 'infographic' | 'talking-points' | 'shorts';
export type Platform = 'tiktok' | 'reels' | 'youtube-shorts';

export interface VideoConfig {
  template: VideoTemplate;
  content: {
    title: string;
    keyPoints: string[];
    statistics?: Array<{ label: string; value: string }>;
    brandColors?: string[];
  };
  format: {
    width: number;
    height: number;
    fps: number;
    duration?: number;
  };
  output?: {
    format: 'mp4' | 'webm';
    quality: 'low' | 'medium' | 'high';
  };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion studio (for video template editing)
npm run remotion:studio

# Render individual video
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requestsPerMinute: 50 },
  claude: { requestsPerMinute: 40 },
  rapidapi: { requestsPerMinute: 100 }
});

// Use with automatic retry
const content = await limiter.execute('claude', async () => {
  return await generateContent(config);
});
```

### Error Handling

```typescript
import { ContentPipelineError } from '@/lib/errors';

try {
  const result = await pipeline.execute(config);
} catch (error) {
  if (error instanceof ContentPipelineError) {
    console.error(`Pipeline failed at stage: ${error.stage}`);
    console.error(`Reason: ${error.message}`);
    
    // Retry from last successful stage
    const result = await pipeline.retryFrom(error.stage);
  }
}
```

### Video Rendering Issues

```typescript
import { validateVideoConfig } from '@/lib/video/validator';

// Validate config before rendering
const config = {
  template: 'infographic',
  content: { /* ... */ },
  format: { width: 1080, height: 1920, fps: 30 }
};

const validation = validateVideoConfig(config);
if (!validation.valid) {
  console.error('Invalid config:', validation.errors);
  // Fix issues before rendering
}
```

### Memory Management for Large Campaigns

```typescript
import { BatchProcessor } from '@/lib/utils/batch-processor';

// Process large campaigns in batches
const processor = new BatchProcessor({
  batchSize: 5,
  concurrency: 2,
  retryAttempts: 3
});

const results = await processor.process(largeCampaignItems, async (item) => {
  return await pipeline.execute(item);
});
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid API quota issues
3. **Cache research data** to reduce redundant API calls
4. **Validate content** before video generation
5. **Use batch processing** for large-scale campaigns
6. **Monitor API costs** with usage tracking
7. **Test video templates** in Remotion Studio before automation
