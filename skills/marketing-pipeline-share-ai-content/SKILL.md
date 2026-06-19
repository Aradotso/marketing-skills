---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - generate automated content with marketing pipeline
  - create AI-powered video content from research
  - automate content creation from keyword to video
  - use marketing pipeline for content automation
  - set up AI content generation pipeline
  - create social media content with AI pipeline
  - build automated marketing content workflow
  - generate videos from text with remotion pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that transforms keywords into complete content packages including research, scripts, articles, and videos. It combines web scraping, AI writing (Claude/OpenAI), and video rendering (Remotion) into a single automated pipeline.

**Key Features:**
- Automated web research and news scraping from sources like TechCrunch, a16z, Twitter, LinkedIn
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual output (English & Vietnamese)
- Automatic video and infographic rendering via Remotion
- Next.js-based web interface

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Services
OPENAI_API_KEY=your_openai_api_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# RapidAPI (for web scraping/research)
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

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
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core Usage Patterns

### 1. Content Research Pipeline

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return {
    insights: research.insights,
    dataPoints: research.dataPoints,
    trending: research.trendingTopics,
    sources: research.sourcesUsed
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function createArticleWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  research: any
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about "${topic}" using this research data: ${JSON.stringify(research)}`
    }]
  });
  
  return message.content;
}

// Using OpenAI
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function createArticleWithOpenAI(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous',
  language: 'en' | 'vi'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content writer with ${tone} tone, writing in ${language === 'en' ? 'English' : 'Vietnamese'}.`
      },
      {
        role: 'user',
        content: `Write an article about ${topic}`
      }
    ],
    temperature: 0.7
  });
  
  return completion.choices[0].message.content;
}
```

### 3. Bilingual Content Generation

```typescript
interface BilingualContent {
  en: string;
  vi: string;
  metadata: {
    format: string;
    tone: string;
    generatedAt: Date;
  };
}

async function generateBilingualContent(
  topic: string,
  format: string,
  research: any
): Promise<BilingualContent> {
  const [englishContent, vietnameseContent] = await Promise.all([
    createArticleWithOpenAI(topic, 'expert', 'en'),
    createArticleWithOpenAI(topic, 'expert', 'vi')
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      format,
      tone: 'expert',
      generatedAt: new Date()
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideoFromContent(
  content: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const aspectRatios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...aspectRatios[platform]
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content
    }
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateBilingualContent } from '@/lib/ai/content-generator';
import { generateVideoFromContent } from '@/lib/video/renderer';

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Starting research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });
    
    // Step 2: Generate Content
    console.log('🧠 Generating content...');
    const content = await generateBilingualContent(
      keyword,
      'toplist',
      research
    );
    
    // Step 3: Create Videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      generateVideoFromContent(content.en, './output/video-en.mp4', 'reels'),
      generateVideoFromContent(content.vi, './output/video-vi.mp4', 'tiktok')
    ]);
    
    return {
      research,
      content,
      videos,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}

// Usage
const result = await runFullPipeline('AI Marketing Automation 2024');
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();
    
    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h'
    });
    
    return NextResponse.json({ success: true, data: research });
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
  try {
    const { topic, format, tone, language } = await request.json();
    
    const content = await generateContent({
      topic,
      format: format || 'toplist',
      tone: tone || 'expert',
      language: language || 'en'
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// src/app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideoFromContent } from '@/lib/video/renderer';
import { v4 as uuidv4 } from 'uuid';

export async function POST(request: NextRequest) {
  try {
    const { content, platform } = await request.json();
    const videoId = uuidv4();
    const outputPath = `./public/videos/${videoId}.mp4`;
    
    await generateVideoFromContent(content, outputPath, platform);
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${videoId}.mp4`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio for video template editing
npm run remotion:studio

# Render a specific video composition
npm run remotion:render
```

## Common Patterns

### Error Handling with Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent({ topic: 'AI Marketing', format: 'toplist' })
);
```

### Content Caching

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN
});

async function getCachedContent(key: string) {
  const cached = await redis.get(`content:${key}`);
  if (cached) return cached;
  
  const content = await generateContent({ topic: key });
  await redis.set(`content:${key}`, content, { ex: 3600 }); // 1 hour
  
  return content;
}
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
// Implement rate limiting
import { Ratelimit } from '@upstash/ratelimit';

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 m')
});

async function rateLimitedGenerate(userId: string, topic: string) {
  const { success } = await ratelimit.limit(userId);
  if (!success) throw new Error('Rate limit exceeded');
  
  return await generateContent({ topic });
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for video rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true
  }
});
```

### Issue: Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

### Issue: Research Data Quality

```typescript
// Validate and filter research results
function validateResearchData(research: any) {
  return {
    insights: research.insights.filter(i => i.relevanceScore > 0.7),
    dataPoints: research.dataPoints.filter(d => d.source && d.date),
    trending: research.trending.slice(0, 5) // Top 5 only
  };
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for expensive AI operations
3. **Add rate limiting** to prevent API quota exhaustion
4. **Validate inputs** before sending to AI APIs
5. **Use TypeScript types** for all content structures
6. **Monitor API usage** and costs
7. **Implement proper error handling** with user-friendly messages
8. **Cache research data** to avoid redundant scraping
