---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, script generation, video rendering, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text with Remotion
  - crawl news and create content automatically
  - build AI-powered content workflow
  - automate social media content with Claude
  - create multilingual marketing content pipeline
  - render videos from blog posts automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms keywords into complete content packages including research, scripts, articles, and rendered videos. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (Toplist, POV, Case Study, How-to) in multiple languages, and finally renders videos using Remotion.

**Key Capabilities:**
- Auto-crawl fresh news/data from major tech sources (last 24h)
- Generate content in multiple formats and languages (EN/VI)
- Render infographics and short-form videos automatically
- Multi-platform optimization (Reels, TikTok, Shorts)
- Next.js frontend with scheduling capabilities

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

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the interface.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawlers/    # News/data crawling modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core Modules

### 1. Research & Crawling

Automatically crawl and analyze fresh content from multiple sources:

```typescript
import { crawlTechNews } from '@/lib/crawlers/tech-crawler';
import { analyzeTrends } from '@/lib/crawlers/trend-analyzer';

// Crawl news from the last 24 hours
async function gatherResearch(keyword: string) {
  const newsData = await crawlTechNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  const insights = await analyzeTrends(newsData);
  
  return {
    rawData: newsData,
    insights: insights,
    dataPoints: insights.statistics
  };
}

// Usage
const research = await gatherResearch('AI agents');
console.log(research.insights);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types';

async function createContent(
  topic: string,
  research: any,
  format: ContentFormat = 'toplist'
) {
  const content = await generateContent({
    topic,
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: [Language.EN, Language.VI],
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    model: 'claude-3-opus', // or 'gpt-4'
    includeSEO: true
  });
  
  return content;
}

// Usage
const article = await createContent(
  'Top 10 AI Marketing Tools 2026',
  research,
  'toplist'
);

// Output structure
// {
//   en: { title, intro, body, conclusion, seo },
//   vi: { title, intro, body, conclusion, seo }
// }
```

### 3. Multi-format Content Generation

```typescript
import { ContentPipeline } from '@/lib/generators/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate multiple content formats from one research
async function generateMultiFormat(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const results = await Promise.all(
    formats.map(format => 
      pipeline.generate({
        keyword,
        format,
        autoResearch: true,
        languages: ['en', 'vi']
      })
    )
  );
  
  return results;
}
```

### 4. Video Rendering with Remotion

Render videos automatically from generated content:

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { VideoTemplate } from '@/types/video';

async function createVideoFromContent(content: any) {
  const video = await renderVideo({
    template: VideoTemplate.INFOGRAPHIC,
    content: {
      title: content.en.title,
      keyPoints: content.en.body.sections.map(s => s.heading),
      statistics: content.research.dataPoints,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    dimensions: {
      width: 1080,
      height: 1920 // 9:16 for Reels/TikTok/Shorts
    },
    duration: 30, // seconds
    outputPath: './public/videos'
  });
  
  return video;
}

// Usage
const videoPath = await createVideoFromContent(article);
console.log(`Video rendered: ${videoPath}`);
```

### 5. Complete Pipeline Example

Full end-to-end content creation workflow:

```typescript
import { ContentPipeline } from '@/lib/generators/pipeline';
import { publishToSchedule } from '@/lib/publishing/scheduler';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline();
  
  try {
    // Step 1: Research
    console.log('🔍 Crawling research data...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      keyword,
      research,
      format: 'toplist',
      languages: ['en', 'vi']
    });
    
    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo(content);
    
    // Step 4: Schedule publishing
    console.log('📅 Scheduling posts...');
    await publishToSchedule({
      content: content.en,
      video,
      platforms: ['facebook', 'instagram', 'tiktok'],
      scheduleTime: new Date(Date.now() + 3600000) // 1 hour from now
    });
    
    return {
      content,
      video,
      status: 'scheduled'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute
runFullPipeline('AI Marketing Automation 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## API Routes (Next.js)

### POST /api/content/generate

Generate content from a keyword:

```typescript
// File: src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/generators/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();
  
  const pipeline = new ContentPipeline();
  
  const result = await pipeline.generate({
    keyword,
    format: format || 'toplist',
    languages: languages || ['en', 'vi'],
    autoResearch: true
  });
  
  return NextResponse.json(result);
}
```

**Usage:**
```bash
curl -X POST http://localhost:3000/api/content/generate \
  -H "Content-Type: application/json" \
  -d '{"keyword": "AI Agents", "format": "toplist", "languages": ["en", "vi"]}'
```

### POST /api/video/render

Render video from content:

```typescript
// File: src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  const { contentId, template, dimensions } = await request.json();
  
  // Fetch content from database
  const content = await getContentById(contentId);
  
  const videoPath = await renderVideo({
    template,
    content,
    dimensions: dimensions || { width: 1080, height: 1920 }
  });
  
  return NextResponse.json({ videoPath });
}
```

## Configuration

### Content Generation Config

```typescript
// File: src/config/content.ts
export const contentConfig = {
  ai: {
    defaultModel: 'claude-3-opus',
    fallbackModel: 'gpt-4',
    maxTokens: 4000,
    temperature: 0.7
  },
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxArticles: 20
  },
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 15,
      includeRatings: true
    },
    pov: {
      perspective: 'expert',
      includeCounterpoints: true
    },
    caseStudy: {
      includeMetrics: true,
      includeTimeline: true
    }
  },
  languages: {
    default: ['en', 'vi'],
    tones: ['expert', 'friendly', 'humorous']
  }
};
```

### Video Rendering Config

```typescript
// File: remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setPixelFormat('yuv420p');
Config.setConcurrency(4);

export const videoTemplates = {
  infographic: {
    fps: 30,
    durationInFrames: 900, // 30 seconds
    width: 1080,
    height: 1920
  },
  slideshow: {
    fps: 30,
    durationInFrames: 1500, // 50 seconds
    width: 1920,
    height: 1080
  }
};
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
import { ContentPipeline } from '@/lib/generators/pipeline';

async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];
  
  for (const keyword of keywords) {
    const content = await pipeline.generate({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi']
    });
    
    results.push(content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 2: Content Variation Testing

```typescript
async function generateVariations(keyword: string) {
  const pipeline = new ContentPipeline();
  const tones = ['expert', 'friendly', 'humorous'];
  
  const variations = await Promise.all(
    tones.map(tone => 
      pipeline.generate({
        keyword,
        format: 'pov',
        tone,
        languages: ['en']
      })
    )
  );
  
  return variations;
}
```

### Pattern 3: Multi-platform Video Export

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';

async function renderMultiPlatform(content: any) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'youtube', width: 1920, height: 1080 },
    { name: 'tiktok', width: 1080, height: 1920 }
  ];
  
  const videos = await Promise.all(
    platforms.map(platform =>
      renderVideo({
        content,
        dimensions: { width: platform.width, height: platform.height },
        outputPath: `./public/videos/${platform.name}`
      })
    )
  );
  
  return videos;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(keyword: string) {
  return pRetry(
    async () => {
      const pipeline = new ContentPipeline();
      return await pipeline.generate({ keyword });
    },
    {
      retries: 3,
      minTimeout: 1000,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Memory Issues

For large video renders, use streaming:

```typescript
import { renderMedia } from '@remotion/renderer';

async function renderLargeVideo(content: any) {
  await renderMedia({
    composition: content,
    serveUrl: 'http://localhost:3000',
    codec: 'h264',
    outputLocation: './output.mp4',
    concurrency: 2, // Reduce concurrency
    chromiumOptions: {
      headless: true,
      args: ['--no-sandbox', '--disable-dev-shm-usage']
    }
  });
}
```

### Crawler Blocking

If crawlers are being blocked, add delays and user agents:

```typescript
import { crawlTechNews } from '@/lib/crawlers/tech-crawler';

async function crawlWithStealth(keyword: string) {
  return await crawlTechNews({
    keyword,
    options: {
      userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
      delay: 3000, // 3 second delay between requests
      maxRetries: 3
    }
  });
}
```

### Database Connection Issues

Handle database errors gracefully:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  log: ['error'],
  errorFormat: 'minimal'
});

async function saveContent(content: any) {
  try {
    return await prisma.content.create({ data: content });
  } catch (error) {
    console.error('Database error:', error);
    // Fallback to file system
    const fs = require('fs');
    fs.writeFileSync(
      `./backup/${Date.now()}.json`,
      JSON.stringify(content)
    );
  }
}
```

## Build for Production

```bash
# Build Next.js app
npm run build

# Start production server
npm start

# Build Remotion videos (if needed separately)
npx remotion render src/index.ts --bundle-cache=./remotion-cache
```

## Key Dependencies

- `next` - React framework
- `@anthropic-ai/sdk` - Claude AI integration
- `openai` - OpenAI API client
- `remotion` - Video rendering
- `@remotion/renderer` - Server-side rendering
- `axios` / `node-fetch` - HTTP requests for crawling
- `cheerio` - HTML parsing
- `prisma` - Database ORM (optional)
