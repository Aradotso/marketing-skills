---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i automate content creation with ai
  - generate video content from text automatically
  - set up ai content pipeline with remotion
  - crawl news and create social media content
  - automate video generation for reels and tiktok
  - create multilingual content with claude and openai
  - build content automation workflow
  - generate infographics and videos from articles
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating multilingual articles, to rendering videos and infographics. It integrates Claude 3, OpenAI, and Remotion to create a complete content factory.

## What It Does

- **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to)
- **Multilingual Support**: Generates content in both English and Vietnamese simultaneously
- **Video Rendering**: Automatically converts text content into videos using Remotion
- **Flexible Tone**: Adjusts writing style (expert, friendly, humorous) based on audience
- **Multi-platform**: Optimizes video output for Reels, TikTok, Shorts

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

## Environment Setup

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion for video rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawler/           # News crawling logic
│   ├── content/           # Content generation
│   └── video/             # Remotion video rendering
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

async function researchTopic(keyword: string) {
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });

  return {
    articles: newsData.articles,
    insights: newsData.extractedInsights,
    trends: newsData.trendingTopics
  };
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/lib/content/types';

async function createArticle(topic: string) {
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    languages: [Language.EN, Language.VI],
    tone: Tone.EXPERT, // or FRIENDLY, HUMOROUS
    aiProvider: 'claude', // or 'openai'
    includeData: true, // Include data-backed insights
    wordCount: 1500
  });

  return content;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoFormat } from '@/lib/video/types';

async function generateVideoFromContent(content: any) {
  const video = await renderVideo({
    content: content.text,
    format: VideoFormat.REELS, // or TIKTOK, SHORTS, SQUARE
    template: 'infographic', // or 'talking-head', 'text-animation'
    duration: 30, // seconds
    resolution: '1080x1920', // vertical video
    includeSubtitles: true,
    language: 'en'
  });

  return {
    videoUrl: video.url,
    thumbnail: video.thumbnail,
    duration: video.duration
  };
}
```

## Common Patterns

### Full Pipeline: Keyword to Video

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    apiKeys: {
      openai: process.env.OPENAI_API_KEY,
      anthropic: process.env.ANTHROPIC_API_KEY,
      rapidapi: process.env.RAPIDAPI_KEY
    }
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content in multiple formats
  const articles = await pipeline.generateContent({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  });

  // Step 3: Create videos
  const videos = await pipeline.generateVideos({
    content: articles,
    platforms: ['reels', 'tiktok', 'shorts']
  });

  return {
    research,
    articles,
    videos
  };
}
```

### API Route Example (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    const content = await generateContent({
      topic: keyword,
      format: format || 'toplist',
      languages: language ? [language] : ['en', 'vi'],
      aiProvider: 'claude'
    });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Client-Side Component

```typescript
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en'
        })
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <input
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && <div>{JSON.stringify(result, null, 2)}</div>}
    </div>
  );
}
```

## Configuration

### Content Formats

```typescript
export enum ContentFormat {
  TOPLIST = 'toplist',           // Top 10 lists
  POV = 'pov',                   // Point of view articles
  CASE_STUDY = 'case_study',     // Case study format
  HOW_TO = 'how_to'              // Tutorial format
}
```

### Video Templates

```typescript
export const VideoTemplates = {
  infographic: {
    duration: 30,
    transitions: 'fade',
    includeCharts: true
  },
  talkingHead: {
    duration: 60,
    includeAvatar: true,
    backgroundMusic: true
  },
  textAnimation: {
    duration: 15,
    animationStyle: 'kinetic',
    subtitles: true
  }
};
```

## CLI Commands (if available)

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Generate content from CLI
npm run generate -- --keyword "AI trends" --format toplist

# Render video
npm run render-video -- --input content.json --template infographic
```

## Troubleshooting

### Issue: API Rate Limits

**Solution**: Implement rate limiting and caching:

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s')
});

export async function generateWithRateLimit(keyword: string) {
  const { success } = await ratelimit.limit(keyword);
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please wait.');
  }

  return await generateContent({ topic: keyword });
}
```

### Issue: Video Rendering Timeouts

**Solution**: Use background jobs or webhooks:

```typescript
import { Queue } from 'bullmq';

const videoQueue = new Queue('video-rendering', {
  connection: { host: 'localhost', port: 6379 }
});

async function queueVideoGeneration(content: any) {
  const job = await videoQueue.add('render', {
    content,
    format: 'reels'
  });

  return { jobId: job.id };
}
```

### Issue: Multilingual Generation Quality

**Solution**: Use language-specific prompts:

```typescript
const LANGUAGE_PROMPTS = {
  en: 'Write in professional American English with data-driven insights',
  vi: 'Viết bằng tiếng Việt tự nhiên, dễ hiểu, phù hợp với độc giả Việt Nam'
};

async function generateWithLanguageContext(topic: string, lang: string) {
  const systemPrompt = LANGUAGE_PROMPTS[lang] || LANGUAGE_PROMPTS.en;
  
  return await generateContent({
    topic,
    language: lang,
    systemPrompt
  });
}
```

### Issue: News Crawling Fails

**Solution**: Implement fallback sources:

```typescript
async function crawlWithFallback(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const results = [];

  for (const source of sources) {
    try {
      const data = await crawlNews({ keyword, sources: [source] });
      results.push(...data.articles);
    } catch (error) {
      console.warn(`Failed to crawl ${source}:`, error);
      continue;
    }
  }

  return results;
}
```

## Advanced Usage

### Custom Video Templates with Remotion

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{ content: string }> = ({ content }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000', opacity }}>
      <h1 style={{ color: '#fff', fontSize: 48 }}>{content}</h1>
    </AbsoluteFill>
  );
};
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      generateContent({ topic: keyword, format: 'toplist' })
    )
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

This pipeline transforms content creation from hours to minutes, enabling marketers to scale their output while maintaining quality across multiple platforms and languages.
