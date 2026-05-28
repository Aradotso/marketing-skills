---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing content pipeline
  - generate videos from blog posts automatically
  - create multi-language content with AI scrapers
  - build content automation with Claude and OpenAI
  - implement automatic content research and publishing
  - set up AI-powered content generation pipeline
  - automate social media content with video rendering
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that creates an end-to-end pipeline from research to publication. It automatically scrapes news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates multi-format content using Claude 3 or OpenAI, and renders videos/infographics with Remotion for distribution across social media platforms.

**Key capabilities:**
- Auto-scrape trending content from major news sources (last 24h)
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Dual-language support (English/Vietnamese) with customizable tone
- Automatic video rendering optimized for Reels/TikTok/Shorts
- Next.js web interface for content management

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

### Start Development Server

```bash
npm run dev
```

Access the interface at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── lib/
│   │   ├── scrapers/       # Content scraping modules
│   │   ├── ai/             # AI content generation
│   │   ├── video/          # Remotion video rendering
│   │   └── publishers/     # Auto-publishing integrations
│   ├── pages/
│   │   ├── api/            # API routes
│   │   └── dashboard/      # Management UI
│   └── components/         # React components
├── remotion/               # Video templates
└── prisma/                 # Database schema
```

## Research & Scraping Module

### Auto-Scrape Content

```typescript
import { ContentScraper } from '@/lib/scrapers';

// Initialize scraper with multiple sources
const scraper = new ContentScraper({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Fetch and analyze trending content
async function researchTopic(topic: string) {
  const results = await scraper.scrape({
    query: topic,
    limit: 10,
    sortBy: 'relevance'
  });

  // Extract insights
  const insights = await scraper.analyzeInsights(results);
  
  return {
    articles: results,
    keyInsights: insights.topInsights,
    trendingData: insights.statistics
  };
}

// Usage
const research = await researchTopic('AI video generation');
console.log(research.keyInsights);
```

### Configure Custom Scrapers

```typescript
import { ScraperConfig } from '@/lib/scrapers/types';

const customConfig: ScraperConfig = {
  sources: [
    {
      name: 'techcrunch',
      url: 'https://techcrunch.com',
      selectors: {
        title: 'h2.post-title',
        content: 'div.article-content',
        date: 'time.timestamp'
      }
    }
  ],
  rateLimiting: {
    requestsPerMinute: 10,
    delayBetweenRequests: 2000
  },
  caching: {
    enabled: true,
    ttl: 3600 // 1 hour
  }
};
```

## AI Content Generation

### Generate Multi-Format Content

```typescript
import { ContentGenerator } from '@/lib/ai/generator';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-5-sonnet-20241022'
});

// Generate content in multiple formats
async function generateContent(research: any, format: string) {
  const prompt = {
    format: format, // 'toplist', 'pov', 'case-study', 'how-to'
    language: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous'
    length: 'medium',
    research: research.keyInsights,
    data: research.trendingData
  };

  const content = await generator.generate(prompt);
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.seo,
    socialPosts: content.snippets
  };
}

// Generate toplist article
const article = await generateContent(research, 'toplist');
```

### Claude Integration Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(insights: string[], tone: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create a comprehensive marketing article based on these insights:
        
${insights.join('\n')}

Tone: ${tone}
Format: Toplist with data-backed points
Include: Statistics, real examples, actionable takeaways`
      }
    ]
  });

  return message.content[0].text;
}
```

### OpenAI Integration Example

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(topic: string, research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating data-driven articles.'
      },
      {
        role: 'user',
        content: `Topic: ${topic}\n\nResearch:\n${JSON.stringify(research, null, 2)}\n\nCreate an engaging article with key insights and actionable advice.`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

## Video Generation with Remotion

### Render Video from Content

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      keyPoints: content.keyInsights,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  });

  const outputPath = `./public/videos/${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });

  return outputPath;
}
```

### Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  branding: any;
}> = ({ title, keyPoints, branding }) => {
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center'
        }}>
          <h1 style={{ color: 'white', fontSize: 48 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key points */}
      {keyPoints.map((point, i) => (
        <Sequence
          key={i}
          from={fps * (3 + i * 4)}
          durationInFrames={fps * 4}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            padding: 60
          }}>
            <div style={{ color: 'white', fontSize: 32 }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Multi-Platform Video Formats

```typescript
type VideoFormat = {
  name: string;
  width: number;
  height: number;
  fps: number;
};

const formats: Record<string, VideoFormat> = {
  reels: { name: 'Instagram Reels', width: 1080, height: 1920, fps: 30 },
  tiktok: { name: 'TikTok', width: 1080, height: 1920, fps: 30 },
  shorts: { name: 'YouTube Shorts', width: 1080, height: 1920, fps: 30 },
  feed: { name: 'Feed Post', width: 1080, height: 1080, fps: 30 }
};

async function renderMultiPlatform(content: any) {
  const videos: Record<string, string> = {};

  for (const [platform, format] of Object.entries(formats)) {
    videos[platform] = await renderMedia({
      composition: {
        id: 'ContentVideo',
        width: format.width,
        height: format.height,
        fps: format.fps,
        durationInFrames: 300
      },
      inputProps: content,
      outputLocation: `./output/${platform}-${Date.now()}.mp4`
    });
  }

  return videos;
}
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize the full pipeline
const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  },
  ai: {
    provider: 'claude',
    model: 'claude-3-5-sonnet-20241022'
  },
  video: {
    platforms: ['reels', 'tiktok', 'shorts'],
    branding: {
      logo: '/assets/logo.png',
      colors: ['#FF6B6B', '#4ECDC4']
    }
  }
});

// Run complete automation
async function automateContentCreation(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await pipeline.research(keyword);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  });

  // Step 3: Create videos
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.generateVideos(content);

  // Step 4: Schedule publishing
  console.log('📅 Scheduling posts...');
  await pipeline.schedule({
    content,
    videos,
    platforms: ['facebook', 'linkedin', 'twitter'],
    publishTime: new Date('2024-01-01T10:00:00Z')
  });

  return {
    articles: content,
    videos: videos,
    scheduled: true
  };
}

// Execute pipeline
const result = await automateContentCreation('AI marketing tools 2024');
```

## API Routes

### POST /api/pipeline/run

Trigger the full content pipeline:

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, formats, languages, platforms } = req.body;

  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      formats,
      languages,
      platforms
    });

    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### GET /api/content/:id

Retrieve generated content:

```typescript
// pages/api/content/[id].ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;

  const content = await prisma.content.findUnique({
    where: { id: String(id) },
    include: {
      videos: true,
      research: true
    }
  });

  res.status(200).json(content);
}
```

## Database Schema

```typescript
// prisma/schema.prisma
model Content {
  id          String   @id @default(cuid())
  title       String
  format      String
  language    String
  body        String   @db.Text
  seo         Json
  research    Research @relation(fields: [researchId], references: [id])
  researchId  String
  videos      Video[]
  createdAt   DateTime @default(now())
}

model Research {
  id        String    @id @default(cuid())
  keyword   String
  sources   Json
  insights  Json
  data      Json
  contents  Content[]
  createdAt DateTime  @default(now())
}

model Video {
  id         String   @id @default(cuid())
  platform   String
  url        String
  format     Json
  content    Content  @relation(fields: [contentId], references: [id])
  contentId  String
  createdAt  DateTime @default(now())
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await pipeline.research(keyword);
      const content = await pipeline.generateContent({
        research,
        formats: ['toplist']
      });
      return { keyword, content };
    })
  );

  return results;
}
```

### Custom Content Workflow

```typescript
import { ResearchEngine } from '@/lib/research';
import { AIWriter } from '@/lib/ai/writer';
import { VideoRenderer } from '@/lib/video';

async function customWorkflow(topic: string) {
  // Custom research with specific sources
  const researcher = new ResearchEngine({
    sources: ['techcrunch', 'a16z']
  });
  const data = await researcher.analyze(topic);

  // Generate with specific AI settings
  const writer = new AIWriter({
    provider: 'openai',
    temperature: 0.8,
    maxTokens: 2000
  });
  const article = await writer.write({
    topic,
    data,
    style: 'conversational'
  });

  // Render only for specific platforms
  const renderer = new VideoRenderer();
  const video = await renderer.create({
    content: article,
    platforms: ['reels'],
    customTemplate: './templates/custom.tsx'
  });

  return { article, video };
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

await limiter.execute(async () => {
  return await scraper.scrape(query);
});
```

### Video Rendering Errors

```typescript
try {
  await renderMedia(composition);
} catch (error) {
  if (error.message.includes('ENOMEM')) {
    console.error('Not enough memory. Reduce video quality or duration.');
  }
  
  // Retry with lower quality
  await renderMedia({
    ...composition,
    scale: 0.5,
    codec: 'h264',
    crf: 28
  });
}
```

### AI Generation Timeout

```typescript
const generateWithTimeout = async (prompt: string, timeout = 30000) => {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      signal: controller.signal,
      // ... other options
    });
    return response;
  } finally {
    clearTimeout(timeoutId);
  }
};
```

### Database Connection Issues

```typescript
// Add to prisma/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient({
  log: ['query', 'error', 'warn'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  }
});

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

## Performance Optimization

### Caching Strategy

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const result = await scraper.scrape(keyword);
  await redis.setex(cacheKey, 3600, JSON.stringify(result));

  return result;
}
```

### Parallel Processing

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent operations

async function processMultipleTopics(topics: string[]) {
  const tasks = topics.map(topic => 
    limit(() => automateContentCreation(topic))
  );

  return await Promise.all(tasks);
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share project, enabling AI coding agents to assist developers in implementing automated content research, generation, and video rendering workflows.
