---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, script generation, video rendering, and multi-platform publishing
triggers:
  - automate content creation with AI research
  - generate marketing content from trending topics
  - create automated video content pipeline
  - build AI content workflow with Claude and OpenAI
  - setup marketing automation with video generation
  - scrape news and generate social media content
  - automate content research and video production
  - build end-to-end content pipeline with Remotion
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content creation pipeline: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter, LinkedIn), to generating multi-format articles with Claude/OpenAI, to rendering videos with Remotion. It's built with TypeScript and Next.js, designed for marketers and content creators to automate 90% of their workflow.

**Key capabilities:**
- Auto-scrape trending news from major tech publications (last 24h)
- Generate content in multiple formats (listicles, POV, case studies, how-to)
- Bilingual support (English & Vietnamese)
- Automatic video rendering from written content using Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

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

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for data scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video rendering
REMOTION_RENDERER_API_KEY=your_remotion_key_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping utilities
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Workflows

### 1. Research & Scraping Pipeline

```typescript
import { scrapeLatestNews } from '@/lib/scraper/news-aggregator';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

async function researchTopic(keyword: string) {
  // Scrape from multiple sources
  const rawData = await scrapeLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
  });

  // AI-powered insight extraction
  const insights = await analyzeInsights(rawData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
  });

  return {
    articles: rawData.articles,
    trends: insights.trends,
    dataPoints: insights.statistics,
    sources: rawData.sources,
  };
}
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/types/content';

async function createArticle(researchData: any) {
  const content = await generateContent({
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    language: Language.BILINGUAL, // EN, VI, or BILINGUAL
    tone: Tone.EXPERT, // FRIENDLY, HUMOROUS, EXPERT
    research: researchData,
    aiProvider: 'anthropic', // Uses Claude
    model: 'claude-3-opus-20240229',
  });

  return {
    title: content.title,
    body: content.body,
    metadata: {
      wordCount: content.wordCount,
      readingTime: content.readingTime,
      seoScore: content.seoScore,
    },
    translations: content.translations, // If bilingual
  };
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(article: any) {
  const videoConfig = {
    template: VideoTemplate.INFOGRAPHIC, // or SHORT_FORM, EXPLAINER
    content: {
      title: article.title,
      keyPoints: article.body.keyPoints,
      statistics: article.metadata.statistics,
    },
    style: {
      aspectRatio: '9:16', // For Reels/TikTok/Shorts
      duration: 60, // seconds
      branding: {
        colors: ['#FF6B6B', '#4ECDC4'],
        logo: '/assets/logo.png',
      },
    },
  };

  const video = await renderVideo(videoConfig);

  return {
    url: video.url,
    thumbnail: video.thumbnail,
    duration: video.duration,
    platform: 'multi', // Optimized for all platforms
  };
}
```

## API Routes

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/news-aggregator';
import { generateContent } from '@/lib/content/generator';

export async function POST(req: NextRequest) {
  const { keyword, format, language } = await req.json();

  try {
    // Step 1: Research
    const research = await researchTopic(keyword);

    // Step 2: Generate content
    const content = await generateContent({
      format,
      language,
      research,
      aiProvider: process.env.DEFAULT_AI_PROVIDER || 'anthropic',
    });

    return NextResponse.json({
      success: true,
      data: content,
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
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { contentId, template, aspectRatio } = await req.json();

  try {
    const video = await renderVideo({
      contentId,
      template,
      style: { aspectRatio },
    });

    return NextResponse.json({
      success: true,
      videoUrl: video.url,
      thumbnail: video.thumbnail,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Client-Side Usage

```typescript
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate(keyword: string) {
    setLoading(true);

    try {
      // Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'bilingual',
        }),
      });

      const content = await contentRes.json();

      // Render video
      const videoRes = await fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contentId: content.data.id,
          template: 'infographic',
          aspectRatio: '9:16',
        }),
      });

      const video = await videoRes.json();

      setResult({
        article: content.data,
        video: video.videoUrl,
      });
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      <input
        type="text"
        placeholder="Enter keyword..."
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            handleGenerate(e.currentTarget.value);
          }
        }}
      />
      {loading && <p>Generating content...</p>}
      {result && (
        <div>
          <h2>{result.article.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.article.body }} />
          <video src={result.video} controls />
        </div>
      )}
    </div>
  );
}
```

## Configuration Examples

### Custom AI Provider Configuration

```typescript
// src/lib/ai/config.ts
export const AI_CONFIG = {
  providers: {
    anthropic: {
      models: {
        fast: 'claude-3-haiku-20240307',
        balanced: 'claude-3-sonnet-20240229',
        powerful: 'claude-3-opus-20240229',
      },
      maxTokens: 4096,
    },
    openai: {
      models: {
        fast: 'gpt-3.5-turbo',
        balanced: 'gpt-4',
        powerful: 'gpt-4-turbo',
      },
      maxTokens: 8192,
    },
  },
  defaults: {
    temperature: 0.7,
    topP: 1,
    frequencyPenalty: 0,
    presencePenalty: 0,
  },
};
```

### Scraper Configuration

```typescript
// src/lib/scraper/config.ts
export const SCRAPER_CONFIG = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com/feed/',
      type: 'rss',
      weight: 1.0,
    },
    a16z: {
      url: 'https://a16z.com/feed/',
      type: 'rss',
      weight: 0.9,
    },
    twitter: {
      apiEndpoint: '/api/twitter-trends',
      type: 'api',
      weight: 0.8,
    },
  },
  filters: {
    minEngagement: 100,
    maxAge: '24h',
    excludeKeywords: ['sponsored', 'ad'],
  },
};
```

## Common Patterns

### Full Pipeline Execution

```typescript
import { runFullPipeline } from '@/lib/pipeline/orchestrator';

async function automateContentCreation(keyword: string) {
  const result = await runFullPipeline({
    input: { keyword },
    steps: [
      'research',      // Scrape trending news
      'analyze',       // Extract insights with AI
      'generate',      // Create article content
      'translate',     // Generate bilingual versions
      'render_video',  // Create video from content
      'optimize_seo',  // SEO optimization
    ],
    config: {
      aiProvider: 'anthropic',
      videoTemplate: 'infographic',
      outputFormats: ['markdown', 'html', 'json'],
    },
  });

  return result;
}
```

### Batch Processing

```typescript
async function processMultipleKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchTopic(keyword);
      const content = await generateContent({
        format: 'toplist',
        language: 'bilingual',
        research,
      });
      return { keyword, content };
    })
  );

  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => r.value);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generateContent(config),
  {
    retries: 3,
    minTimeout: 1000,
    factor: 2,
  }
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
const video = await renderVideo(config, {
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  },
});
```

### Memory Issues with Large Datasets

```typescript
// Stream processing for large scrapes
import { streamScrape } from '@/lib/scraper/stream';

for await (const article of streamScrape(keyword)) {
  await processArticle(article);
  // Process incrementally instead of loading all at once
}
```

### AI Provider Fallback

```typescript
async function generateWithFallback(config: any) {
  try {
    return await generateContent({ ...config, aiProvider: 'anthropic' });
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI');
    return await generateContent({ ...config, aiProvider: 'openai' });
  }
}
```

## Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the web interface.

## Building for Production

```bash
npm run build
npm run start
```

## Key Dependencies

- **Next.js 14+**: App router and API routes
- **@anthropic-ai/sdk**: Claude AI integration
- **openai**: OpenAI GPT integration
- **remotion**: Video rendering engine
- **cheerio**: HTML parsing for web scraping
- **zod**: Schema validation

Refer to `HUONG_DAN_CAI_DAT.md` in the repository for detailed Vietnamese setup instructions.
