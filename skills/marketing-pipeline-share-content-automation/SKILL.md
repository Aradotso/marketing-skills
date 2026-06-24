---
name: marketing-pipeline-share-content-automation
description: Ultimate AI Content Pipeline for automated research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate articles from web research automatically
  - create videos from content using Remotion
  - build AI-powered marketing content pipeline
  - scrape news and generate social media content
  - set up automated content workflow with Claude
  - generate multilingual marketing content with AI
  - build content automation system with TypeScript
---

# Marketing Pipeline Share - Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an all-in-one AI content automation system that handles the complete content creation lifecycle: web research, content generation, and video rendering. Built with TypeScript, Next.js, and integrated with Claude 3, OpenAI, and Remotion for video generation.

**Key Capabilities:**
- Automated web scraping from TechCrunch, a16z, Twitter, LinkedIn
- AI-powered content generation (articles, POV pieces, case studies, how-tos)
- Multi-language support (English/Vietnamese)
- Automatic video/infographic rendering with Remotion
- Multiple content formats and voice tones

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

Create a `.env.local` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js App Router pages
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Web Research & Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Scrape recent news on a topic
async function researchTopic(keyword: string) {
  const articles = await scrapeNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze and extract insights
  const insights = await analyzeContent({
    articles,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return insights;
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, VoiceTone } from '@/types/content';

// Generate article with specific format and tone
async function createArticle(topic: string) {
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    tone: VoiceTone.EXPERT, // or FRIENDLY, HUMOROUS
    languages: ['en', 'vi'],
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY!,
    includeResearch: true // Auto-fetch recent data
  });

  return content;
}
```

### 3. Multi-Language Generation

```typescript
import { generateMultiLanguageContent } from '@/lib/content/multilingual';

async function createBilingualPost(keyword: string) {
  const result = await generateMultiLanguageContent({
    keyword,
    languages: ['en', 'vi'],
    format: 'social-post',
    maxLength: 500
  });

  // Returns: { en: "...", vi: "..." }
  return result;
}
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(content: any) {
  const video = await renderVideo({
    template: VideoTemplate.INFOGRAPHIC,
    content: {
      title: content.title,
      points: content.keyPoints,
      stats: content.statistics
    },
    duration: 30, // seconds
    format: 'reels', // or 'tiktok', 'shorts'
    resolution: '1080x1920'
  });

  return video.outputPath;
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const content = await generateContent({
      topic: keyword,
      format,
      tone: 'expert',
      languages: [language],
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY!
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews } from '@/lib/scraper/news-scraper';

export async function POST(request: NextRequest) {
  const { keyword } = await request.json();

  const articles = await scrapeNews({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  });

  return NextResponse.json({ articles });
}
```

## Common Patterns

### Full Content Pipeline

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

async function automateContentCreation(keyword: string) {
  // Complete pipeline: research → generate → render video
  const result = await runContentPipeline({
    keyword,
    steps: ['research', 'generate', 'video'],
    config: {
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      videoTemplate: 'infographic',
      autoPublish: false
    }
  });

  return {
    article: result.content,
    video: result.videoUrl,
    research: result.sources
  };
}
```

### Custom AI Provider Configuration

```typescript
import { ClaudeProvider } from '@/lib/ai/providers/claude';
import { OpenAIProvider } from '@/lib/ai/providers/openai';

// Use Claude for research, OpenAI for generation
const aiConfig = {
  research: new ClaudeProvider({
    apiKey: process.env.ANTHROPIC_API_KEY!,
    model: 'claude-3-sonnet-20240229'
  }),
  generation: new OpenAIProvider({
    apiKey: process.env.OPENAI_API_KEY!,
    model: 'gpt-4-turbo-preview'
  })
};
```

### Batch Content Generation

```typescript
async function generateBulkContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      return await generateContent({
        topic: keyword,
        format: 'article',
        tone: 'expert',
        languages: ['en'],
        provider: 'claude',
        apiKey: process.env.ANTHROPIC_API_KEY!
      });
    })
  );

  return results;
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video (if separate)
npm run video:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Troubleshooting

### API Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

async function safeApiCall() {
  await limiter.waitForSlot();
  return await generateContent({...});
}
```

### Web Scraping Failures

```typescript
// Add retry logic
import { retry } from '@/lib/utils/retry';

const articles = await retry(
  () => scrapeNews({ keyword: 'AI' }),
  {
    maxAttempts: 3,
    delayMs: 2000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for complex videos
const video = await renderVideo({
  template: 'complex-infographic',
  timeout: 300000, // 5 minutes
  quality: 'high'
});
```

### Memory Issues with Large Content

```typescript
// Stream content generation
import { streamContent } from '@/lib/content/stream-generator';

const stream = await streamContent({
  topic: 'Long article',
  onChunk: (chunk) => {
    // Process chunks incrementally
    console.log(chunk);
  }
});
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Cache research results** to avoid redundant scraping
3. **Implement rate limiting** for AI API calls
4. **Queue video rendering** for resource-intensive operations
5. **Validate content** before publishing
6. **Store generated content** in database for reuse
7. **Monitor API usage** to control costs

## Integration Example

```typescript
// Complete workflow example
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude',
  scraperConfig: {
    sources: ['techcrunch', 'a16z'],
    limit: 10
  }
});

// Run automated pipeline
const result = await pipeline.execute({
  keyword: 'AI Marketing',
  outputFormats: ['article', 'video', 'social-posts'],
  languages: ['en', 'vi'],
  schedule: {
    publishAt: new Date('2026-07-01'),
    platforms: ['facebook', 'linkedin']
  }
});

console.log('Content ready:', result);
```
