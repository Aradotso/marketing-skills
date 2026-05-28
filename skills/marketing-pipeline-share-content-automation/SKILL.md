---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, auto-posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and Remotion
  - create automated content workflow from research to video
  - build AI content generation system with auto-posting
  - implement content automation with research crawling and video rendering
  - use marketing pipeline for automated social media content
  - generate videos and articles automatically from keywords
  - set up content research and scriptwriting automation
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms a single keyword into fully-formed content including research, articles, scripts, and videos. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion video rendering to create a complete content factory.

**Key capabilities:**
- Auto-crawl recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate articles in multiple formats (toplist, POV, case study, how-to)
- Bilingual support (English & Vietnamese) with customizable tone
- Automatic video and infographic rendering via Remotion
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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPID_API_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Auto-posting
FACEBOOK_PAGE_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Core Architecture

The pipeline follows this flow:

```typescript
// Typical pipeline workflow
Keyword Input → Research Scraping → Content Generation → Video Rendering → Auto-Post
```

## Key Components

### 1. Research Module (Auto-Scan)

Crawl and aggregate recent content from multiple sources:

```typescript
// lib/research/scraper.ts
import { researchService } from '@/services/research';

async function performResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const research = await researchService.crawl({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 20
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.stats
  };
}

// Usage
const data = await performResearch('AI automation tools');
console.log(`Found ${data.articles.length} articles with ${data.insights.length} insights`);
```

### 2. Content Generation with AI

Generate articles using Claude or OpenAI with various formats:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateArticle(
  research: ResearchData,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = buildPrompt(research, format, language, tone);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return {
    title: extractTitle(message.content),
    body: extractBody(message.content),
    metadata: {
      format,
      language,
      tone,
      wordCount: countWords(message.content)
    }
  };
}

// Example usage
const article = await generateArticle(
  researchData,
  'toplist',
  'en',
  'expert'
);
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/content/bilingual.ts
async function generateBilingualContent(
  research: ResearchData,
  format: ContentFormat
) {
  const [englishArticle, vietnameseArticle] = await Promise.all([
    generateArticle(research, format, 'en', 'expert'),
    generateArticle(research, format, 'vi', 'friendly')
  ]);
  
  return {
    english: englishArticle,
    vietnamese: vietnameseArticle,
    metadata: {
      generatedAt: new Date().toISOString(),
      format
    }
  };
}
```

### 4. Video Rendering with Remotion

Transform content into video format:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  article: Article,
  videoFormat: 'reel' | 'tiktok' | 'shorts'
) {
  const aspectRatio = getAspectRatio(videoFormat);
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: article.title,
      content: article.body,
      format: videoFormat,
      aspectRatio
    }
  });
  
  // Render video
  const outputPath = `./output/${article.id}-${videoFormat}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });
  
  return outputPath;
}

function getAspectRatio(format: string) {
  const ratios = {
    reel: '9:16',
    tiktok: '9:16',
    shorts: '9:16',
    feed: '1:1'
  };
  return ratios[format] || '16:9';
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { performResearch } from '@/lib/research/scraper';
import { generateBilingualContent } from '@/lib/content/bilingual';
import { renderContentVideo } from '@/lib/video/renderer';

export async function runContentPipeline(
  keyword: string,
  format: ContentFormat,
  videoFormats: VideoFormat[]
) {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  console.log('Step 1: Performing research...');
  const research = await performResearch(keyword);
  
  // Step 2: Generate content
  console.log('Step 2: Generating bilingual content...');
  const content = await generateBilingualContent(research, format);
  
  // Step 3: Render videos
  console.log('Step 3: Rendering videos...');
  const videos = await Promise.all(
    videoFormats.map(vf => renderContentVideo(content.english, vf))
  );
  
  // Step 4: (Optional) Auto-post
  console.log('Step 4: Auto-posting content...');
  await autoPost(content, videos);
  
  return {
    research,
    content,
    videos,
    completedAt: new Date().toISOString()
  };
}

// Usage
const result = await runContentPipeline(
  'AI marketing automation',
  'toplist',
  ['reel', 'tiktok', 'shorts']
);
```

## API Routes (Next.js)

### Create Content Pipeline Endpoint

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, videoFormats } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(
      keyword,
      format || 'toplist',
      videoFormats || ['reel']
    );
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### Research Only Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const research = await performResearch(keyword);
  
  return NextResponse.json({
    keyword,
    articlesFound: research.articles.length,
    insights: research.insights,
    sources: research.sources
  });
}
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Generate a single article quickly
import { generateArticle } from '@/lib/content/generator';
import { performResearch } from '@/lib/research/scraper';

async function quickContent(keyword: string) {
  const research = await performResearch(keyword);
  const article = await generateArticle(research, 'how-to', 'en', 'friendly');
  return article;
}
```

### Pattern 2: Batch Processing

```typescript
// Process multiple keywords in batch
async function batchProcess(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(
      keyword,
      'toplist',
      ['reel']
    );
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}

// Usage
const keywords = ['AI tools', 'marketing automation', 'content strategy'];
const outputs = await batchProcess(keywords);
```

### Pattern 3: Scheduled Content Creation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export function scheduleDailyContent(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const todayKeyword = keywords[new Date().getDay() % keywords.length];
    
    console.log(`Daily content generation: ${todayKeyword}`);
    await runContentPipeline(todayKeyword, 'toplist', ['reel', 'shorts']);
  });
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Add retry logic with exponential backoff
async function apiCallWithRetry(apiCall: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Issue: Video Rendering Memory

```typescript
// Use chunks for large content
async function renderLargeContent(article: Article) {
  const chunks = splitArticle(article, 500); // 500 words per chunk
  
  const videoChunks = await Promise.all(
    chunks.map((chunk, i) => 
      renderContentVideo({ ...article, body: chunk }, 'reel')
    )
  );
  
  return concatenateVideos(videoChunks);
}
```

### Issue: Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPID_API_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

// Call in your main entry point
validateEnv();
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the application
# http://localhost:3000
```

## Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm run start
```

## Testing Pipeline Components

```typescript
// test/pipeline.test.ts
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

describe('Content Pipeline', () => {
  it('should complete full pipeline', async () => {
    const result = await runContentPipeline(
      'test keyword',
      'toplist',
      ['reel']
    );
    
    expect(result.content).toBeDefined();
    expect(result.videos.length).toBeGreaterThan(0);
  }, 60000); // 60s timeout for full pipeline
});
```

This skill enables AI coding agents to effectively utilize the Marketing Pipeline Share system for automated content creation, from research through video generation and distribution.
