---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline with Claude and OpenAI
  - create automated content workflow from research to video
  - build AI content pipeline with automatic posting
  - generate videos from written content automatically
  - use marketing pipeline share for content automation
  - configure AI-powered content research and generation
  - set up remotion video rendering with AI content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content lifecycle: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter, LinkedIn), to generating scripts in multiple formats and languages, to rendering videos with Remotion, and auto-publishing to social platforms.

The pipeline uses Claude 3 and OpenAI for content generation, web scraping for real-time research, and Remotion for video/infographic rendering. It's designed for content creators, marketers, and businesses to reduce content production time by up to 90%.

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

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Social Media Publishing (optional)
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scraper/     # Content scraping utilities
│   │   ├── video/       # Remotion video rendering
│   │   └── publisher/   # Auto-posting services
│   └── remotion/        # Remotion video compositions
├── public/              # Static assets
└── package.json
```

## Core Functionality

### 1. Research & Data Scraping

The pipeline automatically scrapes trending content from multiple sources:

```typescript
import { researchTopic } from '@/lib/scraper/research';

// Scrape trending content from multiple sources
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data
interface ResearchResult {
  articles: Article[];
  insights: string[];
  trends: Trend[];
  statistics: Statistic[];
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/generator';

const content = await generateContent({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  language: 'both', // 'en' | 'vi' | 'both'
  researchData: researchData,
  aiProvider: 'claude' // 'claude' | 'openai'
});

// Returns bilingual content
interface ContentResult {
  en: {
    title: string;
    body: string;
    summary: string;
    keywords: string[];
  };
  vi: {
    title: string;
    body: string;
    summary: string;
    keywords: string[];
  };
  metadata: {
    format: string;
    tone: string;
    wordCount: number;
  };
}
```

### 3. Video Generation with Remotion

Transform written content into videos:

```typescript
import { renderVideo } from '@/lib/video/renderer';

const video = await renderVideo({
  content: content.en,
  template: 'infographic', // 'infographic' | 'slideshow' | 'animated'
  platform: 'reels', // 'reels' | 'tiktok' | 'shorts' | 'story'
  aspectRatio: '9:16',
  duration: 60, // seconds
  style: {
    theme: 'modern',
    colors: ['#FF6B6B', '#4ECDC4'],
    font: 'Inter'
  }
});

// Returns video file path and metadata
interface VideoResult {
  path: string;
  url: string;
  duration: number;
  size: number;
  format: string;
  thumbnail: string;
}
```

### 4. Remotion Composition Example

Create custom video compositions:

```typescript
// src/remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const InfographicVideo: React.FC<{
  title: string;
  points: string[];
  colors: string[];
}> = ({ title, points, colors }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: colors[0] }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60
      }}>
        <h1 style={{
          fontSize: 72,
          color: 'white',
          opacity: Math.min(1, frame / 30)
        }}>
          {title}
        </h1>
        
        {points.map((point, i) => {
          const startFrame = 30 + (i * 45);
          const opacity = Math.max(0, Math.min(1, (frame - startFrame) / 15));
          
          return (
            <div key={i} style={{
              fontSize: 36,
              color: 'white',
              opacity,
              marginTop: 20
            }}>
              • {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Auto-Publishing

Publish content to multiple platforms:

```typescript
import { publishContent } from '@/lib/publisher/social';

const published = await publishContent({
  content: content,
  video: video,
  platforms: ['facebook', 'linkedin', 'twitter'],
  schedule: {
    immediate: false,
    publishAt: new Date('2026-06-15T10:00:00Z')
  },
  settings: {
    facebook: {
      pageId: process.env.FACEBOOK_PAGE_ID,
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN
    },
    linkedin: {
      organizationId: process.env.LINKEDIN_ORG_ID,
      accessToken: process.env.LINKEDIN_ACCESS_TOKEN
    }
  }
});

interface PublishResult {
  platform: string;
  postId: string;
  url: string;
  status: 'published' | 'scheduled' | 'failed';
}
```

## Complete Pipeline Example

Full end-to-end automation workflow:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude',
  videoEnabled: true,
  autoPublish: true
});

// Run complete pipeline
const result = await pipeline.execute({
  keyword: 'Marketing Automation 2026',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  videoTemplates: ['infographic', 'slideshow'],
  platforms: ['facebook', 'linkedin'],
  schedule: {
    interval: 'daily',
    time: '09:00'
  }
});

// Monitor pipeline status
pipeline.on('research:complete', (data) => {
  console.log('Research completed:', data.insights.length, 'insights');
});

pipeline.on('content:generated', (content) => {
  console.log('Content generated:', content.en.title);
});

pipeline.on('video:rendered', (video) => {
  console.log('Video ready:', video.url);
});

pipeline.on('publish:success', (result) => {
  console.log('Published to', result.platform, ':', result.url);
});
```

## CLI Commands

Run the development server:

```bash
npm run dev
```

Build for production:

```bash
npm run build
npm start
```

Render Remotion videos:

```bash
# Render a single video
npx remotion render src/remotion/index.ts InfographicVideo output.mp4

# Render with custom props
npx remotion render src/remotion/index.ts InfographicVideo output.mp4 \
  --props='{"title":"AI Trends","points":["Automation","Personalization"]}'
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/research';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const results = await researchTopic({
    keyword,
    sources,
    timeframe,
    language: 'en'
  });
  
  return NextResponse.json(results);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/generator';

export async function POST(request: NextRequest) {
  const { topic, format, tone, language, researchData } = await request.json();
  
  const content = await generateContent({
    topic,
    format,
    tone,
    language,
    researchData,
    aiProvider: 'claude'
  });
  
  return NextResponse.json(content);
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, template, platform } = await request.json();
  
  const video = await renderVideo({
    content,
    template,
    platform,
    aspectRatio: '9:16',
    duration: 60
  });
  
  return NextResponse.json(video);
}
```

## Common Patterns

### Custom Content Formats

```typescript
import { ContentTemplate } from '@/lib/ai/templates';

const customTemplate = new ContentTemplate({
  name: 'product-launch',
  structure: [
    { section: 'hook', words: 50 },
    { section: 'problem', words: 150 },
    { section: 'solution', words: 200 },
    { section: 'features', words: 300 },
    { section: 'cta', words: 100 }
  ],
  tone: 'persuasive',
  includeStatistics: true,
  includeExamples: true
});

const content = await generateContent({
  topic: 'New AI Tool Launch',
  template: customTemplate,
  researchData: researchData
});
```

### Batch Processing

```typescript
import { BatchProcessor } from '@/lib/pipeline/batch';

const processor = new BatchProcessor({
  concurrency: 3,
  retryAttempts: 2
});

const keywords = [
  'AI Marketing',
  'Content Automation',
  'Video Marketing'
];

const results = await processor.processKeywords(keywords, async (keyword) => {
  const research = await researchTopic({ keyword, sources: ['all'] });
  const content = await generateContent({ topic: keyword, researchData: research });
  const video = await renderVideo({ content, template: 'infographic' });
  return { keyword, content, video };
});
```

### Scheduled Publishing

```typescript
import { Scheduler } from '@/lib/publisher/scheduler';

const scheduler = new Scheduler();

// Schedule daily posts
scheduler.schedule('daily-content', '0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  const result = await pipeline.execute({
    keyword: 'trending-topic',
    autoPublish: true
  });
  console.log('Daily content published');
});

// Schedule weekly video series
scheduler.schedule('weekly-video', '0 10 * * 1', async () => {
  const topics = await getTrendingTopics('week');
  for (const topic of topics) {
    await pipeline.execute({
      keyword: topic,
      videoEnabled: true,
      platforms: ['youtube', 'linkedin']
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requests: 50, window: '1m' },
  anthropic: { requests: 40, window: '1m' },
  rapidapi: { requests: 100, window: '1h' }
});

// Use with automatic retry
const content = await limiter.execute('openai', () => 
  generateContent({ topic: 'AI', aiProvider: 'openai' })
);
```

### Video Rendering Failures

```typescript
// Handle rendering errors gracefully
try {
  const video = await renderVideo({
    content,
    template: 'infographic',
    timeout: 300000 // 5 minutes
  });
} catch (error) {
  if (error.code === 'TIMEOUT') {
    // Fallback to simpler template
    video = await renderVideo({
      content,
      template: 'simple-slideshow',
      timeout: 120000
    });
  } else if (error.code === 'MEMORY_ERROR') {
    // Reduce video quality
    video = await renderVideo({
      content,
      template: 'infographic',
      quality: 'medium'
    });
  }
}
```

### Research Data Quality

```typescript
import { validateResearch } from '@/lib/scraper/validator';

const research = await researchTopic({ keyword: 'AI' });

// Validate data quality
const validation = validateResearch(research, {
  minArticles: 5,
  minInsights: 3,
  requireStatistics: true,
  maxAge: '48h'
});

if (!validation.isValid) {
  console.warn('Research quality issues:', validation.issues);
  // Expand search or use fallback sources
  const expandedResearch = await researchTopic({
    keyword: 'AI',
    sources: ['all'],
    timeframe: '7d'
  });
}
```

### Missing Environment Variables

```typescript
import { validateEnv } from '@/lib/utils/env-validator';

// Validate required environment variables at startup
const requiredVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

const envCheck = validateEnv(requiredVars);

if (!envCheck.isValid) {
  throw new Error(`Missing environment variables: ${envCheck.missing.join(', ')}`);
}
```

## Performance Optimization

### Caching Research Data

```typescript
import { CacheManager } from '@/lib/cache';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  storage: 'redis' // or 'memory'
});

// Cache research results
const research = await cache.getOrSet(
  `research:${keyword}`,
  () => researchTopic({ keyword }),
  { ttl: 7200 } // 2 hours for research
);
```

### Parallel Processing

```typescript
// Process multiple content pieces simultaneously
const results = await Promise.all([
  generateContent({ topic: 'AI', format: 'toplist' }),
  generateContent({ topic: 'AI', format: 'how-to' }),
  generateContent({ topic: 'AI', format: 'case-study' })
]);
```

This skill enables AI coding agents to effectively implement and customize the Marketing Pipeline Share automation system for content research, generation, video rendering, and multi-platform publishing.
