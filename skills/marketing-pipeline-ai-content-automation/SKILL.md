---
name: marketing-pipeline-ai-content-automation
description: AI-powered content pipeline that auto-researches, generates multi-format content, and renders videos using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - generate marketing content from keyword research automatically
  - create videos from blog posts using Remotion
  - build automated content workflow with AI agents
  - integrate Claude AI for content generation pipeline
  - render marketing videos automatically from text
  - crawl news sources and generate content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated system that researches topics, generates multi-format content in multiple languages, and renders videos. The pipeline combines web scraping, AI content generation (Claude 3/OpenAI), and video rendering (Remotion) into a single workflow.

## What This Project Does

The Marketing Pipeline automates the entire content creation process:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
3. **Multi-language Support**: Generates parallel English and Vietnamese content
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Variables

Create a `.env.local` file in the project root:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

### Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Key Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── scraper/        # Web scraping modules
│   │   ├── video/          # Remotion video generation
│   │   └── utils/          # Helper functions
│   └── types/              # TypeScript definitions
├── remotion/               # Remotion video templates
└── public/                 # Static assets
```

## Core API Usage

### 1. Research & Scraping

**Scrape Recent News Sources**

```typescript
import { scrapeNewsSource } from '@/lib/scraper/news-scraper';

async function researchTopic(keyword: string) {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      scrapeNewsSource({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );
  
  return results.flat();
}

// Usage
const insights = await researchTopic('AI marketing automation');
```

**Extract Insights from Raw Data**

```typescript
import { extractInsights } from '@/lib/ai/insight-extractor';

async function analyzeResearch(rawData: any[]) {
  const insights = await extractInsights({
    data: rawData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
    extractionRules: {
      includeStats: true,
      includeQuotes: true,
      minRelevanceScore: 0.7
    }
  });
  
  return insights;
}
```

### 2. Content Generation

**Generate Multi-Format Content**

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, insights: any[]) {
  const content = await generateContent({
    topic,
    insights,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly' | 'humorous'
    provider: 'claude',
    modelConfig: {
      temperature: 0.7,
      maxTokens: 4000
    }
  });
  
  return content;
}

// Returns
// {
//   en: { title: '...', content: '...', meta: {...} },
//   vi: { title: '...', content: '...', meta: {...} }
// }
```

**Customize Content with System Prompts**

```typescript
import { ClaudeClient } from '@/lib/ai/claude-client';

const claude = new ClaudeClient({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

async function generateCustomContent(context: string) {
  const response = await claude.generateCompletion({
    systemPrompt: `You are an expert marketing content writer specializing in B2B SaaS. 
    Write engaging, data-driven content that converts.`,
    userPrompt: `Create a blog post about: ${context}`,
    temperature: 0.8,
    maxTokens: 3000
  });
  
  return response.content;
}
```

### 3. Video Generation with Remotion

**Render Video from Content**

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';

async function createContentVideo(article: any) {
  const videoConfig = {
    composition: 'ContentHighlights', // Remotion composition name
    props: {
      title: article.title,
      highlights: article.keyPoints,
      stats: article.statistics,
      duration: 30, // seconds
      style: 'modern'
    },
    outputFormat: 'mp4',
    resolution: {
      width: 1080,
      height: 1920 // Vertical for Reels/TikTok
    }
  };
  
  const video = await renderVideo(videoConfig);
  
  return video.outputPath;
}
```

**Custom Remotion Composition**

```typescript
// remotion/ContentHighlights.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentHighlights: React.FC<{
  title: string;
  highlights: string[];
  duration: number;
}> = ({ title, highlights, duration }) => {
  const frame = useCurrentFrame();
  const fps = 30;
  
  const opacity = interpolate(
    frame,
    [0, fps, duration * fps - fps, duration * fps],
    [0, 1, 1, 0]
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a', opacity }}>
      <div style={{ padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 72 }}>{title}</h1>
        <ul>
          {highlights.map((item, i) => (
            <li key={i} style={{ color: 'white', fontSize: 48 }}>
              {item}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

**End-to-End Content Automation**

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function runCompleteWorkflow(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    languages: ['en', 'vi']
  });
  
  // 1. Research Phase
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });
  
  // 2. Content Generation Phase
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'how-to'],
    tone: 'professional'
  });
  
  // 3. Video Rendering Phase
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['tiktok', 'reels', 'shorts'],
    templates: ['highlights', 'statistics']
  });
  
  // 4. Export & Schedule
  const result = await pipeline.export({
    content,
    videos,
    destination: 'local', // or 's3', 'cloudinary'
    schedule: {
      enabled: true,
      platforms: ['facebook', 'instagram'],
      time: '2024-01-15T10:00:00Z'
    }
  });
  
  return result;
}

// Execute
const output = await runCompleteWorkflow('AI content automation');
console.log('Generated:', output.contentUrls);
console.log('Videos:', output.videoUrls);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/news-scraper';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await request.json();
    
    const results = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: timeRange || '24h'
    });
    
    return NextResponse.json({ success: true, data: results });
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
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, insights, format, languages } = await request.json();
  
  const content = await generateContent({
    topic,
    insights,
    format: format || 'toplist',
    languages: languages || ['en'],
    provider: 'claude'
  });
  
  return NextResponse.json({ success: true, content });
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  const { article, platform } = await request.json();
  
  const resolutions = {
    tiktok: { width: 1080, height: 1920 },
    reels: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 }
  };
  
  const videoPath = await renderVideo({
    composition: 'ContentHighlights',
    props: { title: article.title, highlights: article.keyPoints },
    resolution: resolutions[platform] || resolutions.tiktok
  });
  
  return NextResponse.json({ success: true, videoUrl: videoPath });
}
```

## Configuration Patterns

### AI Provider Configuration

```typescript
// src/config/ai-config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-sonnet-20240229',
    maxTokens: 4000,
    temperature: 0.7,
    systemPrompts: {
      toplist: 'You are a marketing expert creating engaging toplists...',
      howTo: 'You are a technical writer creating clear tutorials...',
      caseStudy: 'You are a business analyst writing compelling case studies...'
    }
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Scraper Configuration

```typescript
// src/config/scraper-config.ts
export const scraperConfig = {
  sources: {
    techcrunch: {
      baseUrl: 'https://techcrunch.com',
      apiEndpoint: '/wp-json/wp/v2/posts',
      rateLimit: 100, // requests per hour
      headers: {
        'User-Agent': 'ContentPipeline/1.0'
      }
    },
    twitter: {
      apiVersion: '2',
      endpoint: 'https://api.twitter.com/2/tweets/search/recent',
      maxResults: 100
    }
  },
  proxy: {
    enabled: false,
    url: process.env.PROXY_URL
  }
};
```

## Common Patterns

### Rate Limiting AI Requests

```typescript
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function batchGenerateContent(topics: string[]) {
  const tasks = topics.map(topic =>
    limit(() => generateContent({ topic, format: 'toplist' }))
  );
  
  return Promise.all(tasks);
}
```

### Caching Research Results

```typescript
import { cache } from '@/lib/utils/cache';

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  const cached = await cache.get(cacheKey);
  if (cached) return cached;
  
  const results = await researchTopic(keyword);
  await cache.set(cacheKey, results, 3600); // 1 hour TTL
  
  return results;
}
```

### Error Handling & Retries

```typescript
import { retry } from '@/lib/utils/retry';

async function robustContentGeneration(topic: string) {
  return retry(
    async () => {
      return await generateContent({ topic, format: 'toplist' });
    },
    {
      maxAttempts: 3,
      delayMs: 1000,
      backoff: 'exponential',
      onRetry: (error, attempt) => {
        console.log(`Retry attempt ${attempt} after error:`, error.message);
      }
    }
  );
}
```

## Troubleshooting

### API Rate Limits

**Problem**: Getting 429 errors from AI providers

**Solution**: Implement exponential backoff and request queuing

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMilliseconds: 60000 // 50 requests per minute
});

await limiter.schedule(async () => {
  return await claude.generateCompletion({...});
});
```

### Remotion Rendering Issues

**Problem**: Video rendering fails or takes too long

**Solution**: Optimize composition and use server-side rendering

```bash
# Install Remotion Lambda for faster rendering
npm install @remotion/lambda

# Configure AWS Lambda
npx remotion lambda sites create remotion/index.ts --site-name=content-videos
```

```typescript
import { renderMediaOnLambda } from '@remotion/lambda/client';

const { renderId, bucketName } = await renderMediaOnLambda({
  region: 'us-east-1',
  functionName: process.env.REMOTION_LAMBDA_FUNCTION!,
  composition: 'ContentHighlights',
  serveUrl: process.env.REMOTION_SERVE_URL!,
  codec: 'h264',
  inputProps: { title: 'My Video' }
});
```

### Scraping Blocks

**Problem**: Getting blocked by websites during scraping

**Solution**: Use rotating proxies and respect robots.txt

```typescript
import { ScraperWithProxy } from '@/lib/scraper/proxy-scraper';

const scraper = new ScraperWithProxy({
  proxyList: process.env.PROXY_LIST?.split(','),
  rotateOnError: true,
  respectRobotsTxt: true,
  delayMs: 2000
});

const data = await scraper.scrape(url);
```

### Memory Issues with Large Datasets

**Problem**: Out of memory when processing many articles

**Solution**: Use streaming and pagination

```typescript
async function* streamResearch(keyword: string) {
  const pageSize = 10;
  let offset = 0;
  
  while (true) {
    const batch = await scrapeNewsSource({
      keyword,
      limit: pageSize,
      offset
    });
    
    if (batch.length === 0) break;
    
    yield batch;
    offset += pageSize;
  }
}

// Usage
for await (const batch of streamResearch('AI')) {
  await processBatch(batch);
}
```

## Best Practices

1. **Always validate API keys on startup**
2. **Use TypeScript strict mode for type safety**
3. **Implement proper error boundaries in React components**
4. **Cache expensive AI operations**
5. **Monitor API usage and costs**
6. **Test video renders locally before production**
7. **Version control your prompts separately**
8. **Use environment-specific configs**
