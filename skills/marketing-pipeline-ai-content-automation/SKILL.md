---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate video content from text using this marketing automation tool
  - automate content research and article generation with AI
  - create social media videos automatically from blog posts
  - integrate Claude and OpenAI for content automation
  - crawl news sources and generate multilingual content
  - use remotion to render marketing videos from scripts
  - build an automated content creation workflow
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls news sources, generates multilingual articles using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for recent data
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts content to videos/infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn
- API keys for Claude (Anthropic) or OpenAI
- RapidAPI key for news crawling (optional)

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Remotion Config
REMOTION_QUALITY=high
REMOTION_FPS=30

# App Config
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Content Research and Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

async function researchTopic(keyword: string) {
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 50
  });

  return {
    articles: newsData.articles,
    insights: newsData.extractedInsights,
    trends: newsData.trendingTopics
  };
}

// Example usage
const research = await researchTopic('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Create a ${format} article about "${topic}" in ${language} with a ${tone} tone.
  
  Use the following research data: ${JSON.stringify(researchData)}
  
  Requirements:
  - Include data-backed insights
  - Add relevant statistics
  - Structure with clear headings
  - Optimize for SEO`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}

// Generate bilingual content
const englishArticle = await generateArticle('AI Marketing', 'toplist', 'en', 'expert');
const vietnameseArticle = await generateArticle('AI Marketing', 'toplist', 'vi', 'friendly');
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(topic: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: `Create a comprehensive article about "${topic}". Context: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

async function generateVideo(
  articleContent: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Define video dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content: articleContent,
      ...dimensions[platform]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    fps: parseInt(process.env.REMOTION_FPS || '30'),
  });

  return outputPath;
}

// Example usage
const videoPath = await generateVideo(
  englishArticle,
  './output/marketing-video.mp4',
  'reels'
);
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    language: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true,
    videoPlatforms: ['reels', 'tiktok', 'shorts']
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const articles = await pipeline.generateContent({
    topic: keyword,
    research,
    tone: 'expert'
  });

  // Step 3: Generate videos
  const videos = await pipeline.generateVideos(articles);

  // Step 4: Save and schedule
  await pipeline.save({
    articles,
    videos,
    scheduledDate: new Date('2024-03-15')
  });

  return {
    articles,
    videos,
    research
  };
}

// Execute pipeline
const results = await runFullPipeline('AI Content Automation');
```

## API Endpoints (Next.js Routes)

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const data = await crawlNews({ keyword, sources, timeRange });
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/generate-content

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, tone, researchData } = await request.json();

  try {
    const article = await generateArticle({
      topic,
      format,
      language,
      tone,
      researchData
    });

    return NextResponse.json({ success: true, article });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/generate-video

```typescript
// app/api/generate-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/video-generator';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();

  try {
    const videoUrl = await generateVideo(content, platform);
    return NextResponse.json({ success: true, videoUrl });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        const research = await crawlNews({ keyword, timeRange: '24h' });
        const article = await generateArticle(
          keyword,
          'toplist',
          'en',
          'expert'
        );
        return { keyword, article, success: true };
      } catch (error) {
        return { keyword, error: error.message, success: false };
      }
    })
  );

  return results;
}

// Generate 10 articles at once
const keywords = [
  'AI Marketing',
  'Content Automation',
  'Video SEO',
  // ... more keywords
];
const articles = await batchGenerateContent(keywords);
```

### Pattern 2: Multilingual Content Strategy

```typescript
async function createMultilingualCampaign(topic: string) {
  const languages = ['en', 'vi'] as const;
  const formats = ['toplist', 'how-to', 'case-study'] as const;

  const campaign = await Promise.all(
    languages.flatMap(lang =>
      formats.map(async format => ({
        language: lang,
        format,
        content: await generateArticle(topic, format, lang, 'expert')
      }))
    )
  );

  return campaign;
}
```

### Pattern 3: Video Optimization Pipeline

```typescript
async function optimizeVideoForPlatforms(content: string) {
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  
  const videos = await Promise.all(
    platforms.map(async platform => {
      const videoPath = `./output/${platform}-${Date.now()}.mp4`;
      await generateVideo(content, videoPath, platform);
      
      return {
        platform,
        path: videoPath,
        metadata: {
          duration: 60, // seconds
          resolution: '1080x1920',
          format: 'mp4'
        }
      };
    })
  );

  return videos;
}
```

### Pattern 4: Scheduled Content Publishing

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function scheduleContent(
  content: string,
  publishDate: Date,
  platforms: string[]
) {
  const scheduledPost = await prisma.scheduledContent.create({
    data: {
      content,
      publishDate,
      platforms,
      status: 'pending',
      metadata: {
        generated: new Date(),
        autoPosted: false
      }
    }
  });

  return scheduledPost;
}

// Schedule for next week
const nextWeek = new Date();
nextWeek.setDate(nextWeek.getDate() + 7);

await scheduleContent(
  article,
  nextWeek,
  ['facebook', 'linkedin', 'twitter']
);
```

## TypeScript Types

```typescript
// types/content.ts
export interface ResearchData {
  keyword: string;
  articles: Article[];
  insights: string[];
  trends: Trend[];
  sources: Source[];
}

export interface Article {
  title: string;
  content: string;
  language: 'en' | 'vi';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  metadata: {
    wordCount: number;
    readingTime: number;
    seoScore: number;
  };
}

export interface VideoConfig {
  platform: 'reels' | 'tiktok' | 'shorts';
  width: number;
  height: number;
  fps: number;
  duration: number;
  quality: 'low' | 'medium' | 'high';
}

export interface PipelineConfig {
  aiProvider: 'claude' | 'openai';
  language: ('en' | 'vi')[];
  formats: ('toplist' | 'pov' | 'case-study' | 'how-to')[];
  generateVideo: boolean;
  videoPlatforms: ('reels' | 'tiktok' | 'shorts')[];
}
```

## Running the Application

### Development Mode

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

### Production Build

```bash
# Build the application
npm run build

# Start production server
npm start
```

### Running Remotion Studio

```bash
# Open Remotion preview
npm run remotion:studio

# Render a specific composition
npm run remotion:render
```

## Configuration Files

### next.config.js

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  env: {
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
  },
  webpack: (config) => {
    config.externals.push({
      'utf-8-validate': 'commonjs utf-8-validate',
      'bufferutil': 'commonjs bufferutil',
    });
    return config;
  },
};

module.exports = nextConfig;
```

### remotion.config.ts

```typescript
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setCodec('h264');
Config.setPixelFormat('yuv420p');
Config.setConcurrency(4);
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateWithRateLimit(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateArticle(topic, 'toplist', 'en', 'expert'))
  );
  
  return Promise.all(promises);
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for video rendering
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
  },
});
```

### Issue: Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatch<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize: number = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(processor)
    );
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Issue: Claude API Errors

```typescript
// Retry logic for AI requests
async function generateWithRetry(
  prompt: string,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const message = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
      return message.content[0].text;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}
```

## Best Practices

1. **Always validate API keys** before starting the pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use TypeScript strict mode** for type safety
4. **Implement proper error handling** at each pipeline stage
5. **Monitor API usage** to stay within rate limits
6. **Store generated content** in a database for future reference
7. **Use environment variables** for all sensitive configuration
8. **Test video rendering** with short samples before full production

## Additional Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed Vietnamese setup guide
- Refer to Anthropic/OpenAI documentation for advanced AI features
- Explore Remotion docs for custom video templates
- Review Next.js 14 App Router documentation for API routes
