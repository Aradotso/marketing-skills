---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for marketing workflows
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - set up automated content pipeline with Claude and OpenAI
  - create videos automatically from blog posts
  - build AI-powered content automation system
  - crawl news and generate social media content
  - use Remotion to render marketing videos
  - automate content research and scriptwriting
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete AI-powered content automation pipeline that transforms keywords into fully-fledged content pieces including articles, scripts, and videos. It crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, uses Claude/OpenAI to generate content in multiple formats and languages, and renders videos automatically using Remotion.

## What This Project Does

- **Auto-Research**: Crawls and analyzes real-time news data from major tech publications
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Generation**: Automatically renders infographics and short-form videos using Remotion
- **Platform Optimization**: Outputs video in formats optimized for Reels, TikTok, and Shorts

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

Create a `.env.local` file in the root directory:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion for video rendering
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render videos with Remotion
npm run remotion:render
```

## Core API Usage

### 1. News Research & Crawling

```typescript
// src/lib/crawler/newsService.ts
import { fetchNewsData } from '@/lib/crawler/newsService';

interface NewsSource {
  source: string;
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

async function getLatestNews(keyword: string): Promise<NewsSource[]> {
  const news = await fetchNewsData({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  return news;
}

// Usage
const articles = await getLatestNews('AI automation');
console.log(`Found ${articles.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claudeService.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
You are a content marketing expert. Based on the following research data, 
create a ${request.format} article in ${request.language} with a ${request.tone} tone.

Keyword: ${request.keyword}

Research Data:
${request.researchData}

Requirements:
- Use data-backed insights
- Include specific examples and statistics
- Optimize for engagement and shareability
- Length: 1000-1500 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage
const content = await generateContent({
  keyword: 'AI Marketing Tools',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: JSON.stringify(articles)
});
```

### 3. OpenAI Integration Alternative

```typescript
// src/lib/ai/openaiService.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert specializing in ${request.format} content.`
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about "${request.keyword}" using this research: ${request.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}-${config.style}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  title: '5 AI Marketing Tools You Need',
  content: [
    'Tool 1: AI Content Generator',
    'Tool 2: Social Media Scheduler',
    'Tool 3: Analytics Dashboard'
  ],
  style: 'reels'
});
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/contentPipeline.ts
import { fetchNewsData } from '@/lib/crawler/newsService';
import { generateContent } from '@/lib/ai/claudeService';
import { renderContentVideo } from '@/lib/video/renderVideo';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoStyle?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Crawling news data...');
  const newsData = await fetchNewsData({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  });

  const results = [];

  // Step 2: Generate content for each language
  for (const language of config.languages) {
    console.log(`✍️ Generating ${language} content...`);
    const content = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language,
      tone: 'expert',
      researchData: JSON.stringify(newsData)
    });

    results.push({
      language,
      content,
      wordCount: content.split(' ').length
    });
  }

  // Step 3: Generate video if requested
  let videoPath;
  if (config.generateVideo && config.videoStyle) {
    console.log('🎬 Rendering video...');
    
    // Extract key points from content
    const keyPoints = extractKeyPoints(results[0].content);
    
    videoPath = await renderContentVideo({
      title: config.keyword,
      content: keyPoints,
      style: config.videoStyle
    });
  }

  return {
    articles: results,
    video: videoPath,
    researchSources: newsData.length
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - in production, use AI to extract key points
  const sentences = content.split(/[.!?]+/).filter(s => s.trim().length > 20);
  return sentences.slice(0, 5).map(s => s.trim());
}

// Usage Example
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoStyle: 'reels'
});

console.log(`✅ Generated ${result.articles.length} articles`);
console.log(`✅ Used ${result.researchSources} research sources`);
if (result.video) {
  console.log(`✅ Video saved to: ${result.video}`);
}
```

## API Routes (Next.js)

### Create Content API Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, languages, generateVideo, videoStyle } = body;

    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
      videoStyle: videoStyle || 'reels'
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const generateContent = async (formData: any) => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="content-generator">
      <form onSubmit={(e) => {
        e.preventDefault();
        const formData = new FormData(e.currentTarget);
        generateContent({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          languages: ['en', 'vi'],
          generateVideo: true,
          videoStyle: 'reels'
        });
      }}>
        <input name="keyword" placeholder="Enter keyword" required />
        <select name="format" required>
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="results">
          <h3>Generated Content</h3>
          {/* Display results */}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private running = 0;
  
  constructor(
    private maxConcurrent: number = 3,
    private delayMs: number = 1000
  ) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    this.running++;
    
    try {
      const result = await fn();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
      return result;
    } finally {
      this.running--;
    }
  }
}

// Usage
const limiter = new RateLimiter(3, 1000);

const results = await Promise.all(
  keywords.map(keyword => 
    limiter.add(() => generateContent({ keyword, /* ... */ }))
  )
);
```

### Caching Research Data

```typescript
// src/lib/cache/researchCache.ts
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

export async function cacheResearch(keyword: string, data: any) {
  await redis.setex(
    `research:${keyword}`,
    3600, // 1 hour TTL
    JSON.stringify(data)
  );
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with Claude or OpenAI:

```typescript
// Implement exponential backoff
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Remotion Rendering Issues

If video rendering fails:

```typescript
// Increase memory limit for Node.js
// package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### News Crawling Timeout

```typescript
// Add timeout to fetch requests
async function fetchWithTimeout(url: string, timeoutMs = 10000) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, { signal: controller.signal });
    return response;
  } finally {
    clearTimeout(timeout);
  }
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the dev server:

```bash
# Kill existing process
pkill -f "next dev"

# Restart
npm run dev
```

## Performance Optimization

### Parallel Processing

```typescript
// Generate content in parallel for multiple languages
async function generateMultiLanguage(keyword: string) {
  const languages: ('en' | 'vi')[] = ['en', 'vi'];
  
  const contents = await Promise.all(
    languages.map(language =>
      generateContent({
        keyword,
        format: 'toplist',
        language,
        tone: 'expert',
        researchData: '...'
      })
    )
  );

  return languages.reduce((acc, lang, idx) => {
    acc[lang] = contents[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

This skill covers the essential functionality of the Ultimate AI Content Pipeline, enabling AI agents to help developers automate marketing content creation from research through video generation.
