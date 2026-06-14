---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multi-format writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate articles and videos from keywords automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated content workflow with video rendering
  - build content automation system with Remotion
  - configure AI content research and generation pipeline
  - automate content from research to video generation
  - use ultimate AI content pipeline for marketing automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end content automation system that transforms keywords into complete content packages including research, multi-format articles (in multiple languages), and rendered videos. Built with TypeScript, Next.js, and integrating Claude 3, OpenAI, and Remotion for video generation.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
2. **AI Writing**: Generates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language**: Produces simultaneous English and Vietnamese versions
4. **Video Generation**: Renders infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scrapers/    # Content scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { scrapeLatestNews } from '@/lib/scrapers/news-scraper';
import { analyzeResearch } from '@/lib/ai/research-analyzer';

async function gatherResearch(keyword: string) {
  // Scrape latest news from multiple sources
  const newsData = await scrapeLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  // Analyze and extract insights using AI
  const insights = await analyzeResearch({
    keyword,
    rawData: newsData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    newsData,
    insights,
    sources: insights.citedSources
  };
}
```

### 2. Multi-Format Content Generation

```typescript
import { generateContent } from '@/lib/generators/content-generator';

async function createArticle(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const content = await generateContent({
    keyword,
    format,
    language,
    tone: 'professional', // 'friendly', 'humorous'
    research: await gatherResearch(keyword),
    aiProvider: process.env.OPENAI_API_KEY ? 'openai' : 'claude',
    includeDataPoints: true,
    wordCount: 1500
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    images: content.suggestedImages,
    cta: content.callToAction
  };
}
```

### 3. Dual-Language Generation

```typescript
import { generateBilingualContent } from '@/lib/generators/bilingual-generator';

async function createBilingualArticle(keyword: string) {
  const { en, vi } = await generateBilingualContent({
    keyword,
    format: 'toplist',
    tone: 'expert',
    ensureConsistency: true, // Keeps structure aligned between languages
    aiProvider: 'claude'
  });

  return {
    english: {
      title: en.title,
      content: en.body,
      meta: en.metadata
    },
    vietnamese: {
      title: vi.title,
      content: vi.body,
      meta: vi.metadata
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/video-renderer';
import { generateVideoScript } from '@/lib/generators/video-script-generator';

async function createContentVideo(article: any) {
  // Generate video script from article
  const script = await generateVideoScript({
    title: article.title,
    keyPoints: article.keyPoints,
    duration: 60, // seconds
    platform: 'reels' // 'tiktok', 'shorts'
  });

  // Render video using Remotion
  const video = await renderVideo({
    composition: 'InfoGraphic', // Template name
    script,
    props: {
      title: article.title,
      points: script.scenes,
      brandColors: {
        primary: '#0066FF',
        secondary: '#00D9FF'
      },
      aspectRatio: '9:16' // Vertical for Reels/TikTok
    },
    outputFormat: 'mp4',
    fps: 30
  });

  return {
    videoPath: video.outputPath,
    duration: video.duration,
    size: video.fileSize
  };
}
```

## Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

async function automateContentCreation(keyword: string) {
  const pipeline = await runContentPipeline({
    keyword,
    steps: {
      research: {
        enabled: true,
        sources: ['techcrunch', 'a16z', 'twitter'],
        timeframe: '24h'
      },
      writing: {
        enabled: true,
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi'],
        tone: 'professional',
        aiProvider: 'claude'
      },
      video: {
        enabled: true,
        platforms: ['reels', 'tiktok', 'shorts'],
        duration: 60,
        includeSubtitles: true
      }
    },
    autoPublish: false // Set to true to auto-post to platforms
  });

  return {
    researchSummary: pipeline.research.summary,
    articles: pipeline.articles, // Array of generated content
    videos: pipeline.videos, // Array of rendered videos
    analytics: pipeline.metadata
  };
}

// Usage
const result = await automateContentCreation('AI automation tools 2026');
console.log(`Generated ${result.articles.length} articles`);
console.log(`Rendered ${result.videos.length} videos`);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      steps: {
        research: { enabled: true },
        writing: {
          enabled: true,
          formats: [format || 'toplist'],
          languages: [language || 'en']
        },
        video: { enabled: false }
      }
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/video-renderer';

export async function POST(request: NextRequest) {
  const { articleId, platform } = await request.json();

  const video = await renderVideo({
    composition: 'InfoGraphic',
    props: {
      articleId,
      aspectRatio: platform === 'youtube' ? '16:9' : '9:16'
    }
  });

  return NextResponse.json({
    videoUrl: video.outputPath,
    duration: video.duration
  });
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run development server
npm run dev

# Generate content from CLI
npm run generate -- --keyword "AI trends" --format toplist --lang en

# Render video
npm run render-video -- --article-id abc123 --platform reels

# Build for production
npm run build

# Start production server
npm run start
```

## Common Patterns

### Custom Content Template

```typescript
import { ContentTemplate } from '@/types/content';

const customTemplate: ContentTemplate = {
  name: 'product-launch',
  structure: {
    hook: { maxWords: 50, tone: 'exciting' },
    problemStatement: { maxWords: 150 },
    solution: { maxWords: 200 },
    features: { count: 5, wordsPerFeature: 50 },
    socialProof: { includeStats: true },
    cta: { type: 'action-oriented' }
  },
  aiInstructions: `
    Write a compelling product launch announcement.
    Focus on benefits over features.
    Include specific data points and user testimonials.
  `
};

const content = await generateContent({
  keyword: 'New AI Tool Launch',
  customTemplate,
  aiProvider: 'claude'
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        steps: {
          research: { enabled: true, timeframe: '24h' },
          writing: { enabled: true, formats: ['toplist'] },
          video: { enabled: false }
        }
      })
    )
  );

  return results.map((result, i) => ({
    keyword: keywords[i],
    article: result.articles[0],
    researchSources: result.research.sources.length
  }));
}
```

### Webhook Integration

```typescript
import { scheduleContentPublish } from '@/lib/publishing/scheduler';

async function publishToWebhook(content: any, webhookUrl: string) {
  await scheduleContentPublish({
    content,
    destination: {
      type: 'webhook',
      url: webhookUrl,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.WEBHOOK_TOKEN}`
      }
    },
    scheduleTime: new Date(Date.now() + 3600000) // 1 hour from now
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function safeAPICall(fn: () => Promise<any>) {
  await limiter.waitForSlot();
  return await fn();
}
```

### Claude/OpenAI Errors

```typescript
async function generateWithFallback(prompt: string) {
  try {
    return await generateWithClaude(prompt);
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI:', error);
    return await generateWithOpenAI(prompt);
  }
}
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion configuration for large videos
const video = await renderVideo({
  composition: 'InfoGraphic',
  props: { /* ... */ },
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  concurrency: 2, // Reduce concurrent rendering
  frameRange: [0, 1800] // Limit frames if needed
});
```

### Missing Research Data

```typescript
async function gatherResearchWithRetry(keyword: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const data = await scrapeLatestNews({ keyword });
      if (data.length > 0) return data;
    } catch (error) {
      console.error(`Retry ${i + 1}/${maxRetries}:`, error);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Failed to gather research after retries');
}
```

## Environment Variable Reference

```bash
# Required
ANTHROPIC_API_KEY=      # Claude API key from console.anthropic.com
OPENAI_API_KEY=         # OpenAI API key from platform.openai.com
RAPIDAPI_KEY=           # RapidAPI key for content scraping

# Optional
REMOTION_LICENSE_KEY=   # Remotion license for commercial use
NEXT_PUBLIC_APP_URL=    # Public URL of your deployment
WEBHOOK_TOKEN=          # Token for webhook authentication
DATABASE_URL=           # If using database for content storage
```
