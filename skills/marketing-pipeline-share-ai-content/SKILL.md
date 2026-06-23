---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline that researches, generates scripts, and creates videos using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up an AI content pipeline with video generation
  - use marketing pipeline share to generate posts
  - create automated content from research to video
  - build an AI-powered content automation system
  - generate social media content with Claude and Remotion
  - automate content research and video creation workflow
  - set up end-to-end AI content generation pipeline
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms keywords into complete content pieces including research, script generation, and video rendering. It automatically crawls recent news from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese), finally rendering videos via Remotion.

**Key capabilities:**
- Auto-research from real-time news sources (24h data)
- Multi-format content generation (blog posts, social media)
- Bilingual support (EN/VN) with tone customization
- Automatic video/infographic rendering
- Next.js frontend for workflow management

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

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create `.env` file with required API keys:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Remotion (Video Generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
```

Access at `http://localhost:3000`

## Core Components

### 1. Research Module (Auto-Scan)

The research module crawls and aggregates content from multiple sources:

```typescript
// lib/research/scanner.ts
import { chromium } from 'playwright';

interface ResearchSource {
  url: string;
  selector: string;
  type: 'news' | 'social' | 'blog';
}

export async function scanSources(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const sources: ResearchSource[] = [
    { url: `https://techcrunch.com/search/${keyword}`, selector: '.post-block', type: 'news' },
    { url: `https://a16z.com/?s=${keyword}`, selector: 'article', type: 'blog' }
  ];

  const browser = await chromium.launch({ headless: true });
  const results = [];

  for (const source of sources) {
    const page = await browser.newPage();
    await page.goto(source.url);
    
    const articles = await page.$$(source.selector);
    
    for (const article of articles) {
      const title = await article.$eval('h2, h3', el => el.textContent);
      const excerpt = await article.$eval('p', el => el.textContent);
      const link = await article.$eval('a', el => el.href);
      
      results.push({ title, excerpt, link, source: source.type });
    }
    
    await page.close();
  }

  await browser.close();
  return results;
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' ? message.content[0].text : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective with personal insights and opinions',
    'case-study': 'Analyze a real example with data, challenges, solutions, and results',
    'how-to': 'Provide step-by-step instructions with actionable advice'
  };

  const toneGuide = {
    'expert': 'Use professional terminology and data-driven insights',
    'friendly': 'Write conversationally with relatable examples',
    'humorous': 'Include wit and entertaining elements while staying informative'
  };

  return `
You are a content creator specializing in ${request.format} articles.

Keyword: ${request.keyword}
Language: ${request.language.toUpperCase()}
Tone: ${toneGuide[request.tone]}
Format: ${formatInstructions[request.format]}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Create a comprehensive article that:
1. Uses the latest research data provided (24h news)
2. Follows the ${request.format} format exactly
3. Maintains a ${request.tone} tone throughout
4. Includes specific examples and data points from the research
5. Is optimized for social media sharing
6. Length: 800-1200 words

Output the article in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.
`;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  imageUrls: string[];
  duration: number; // in seconds
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 vertical
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      images: config.imageUrls,
    },
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
      images: config.imageUrls,
    },
  });

  return outputPath;
}
```

### 4. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Img, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
  images: string[];
}> = ({ title, content, images }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  const contentOpacity = interpolate(
    frame,
    [30, 60],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Section */}
      <div
        style={{
          position: 'absolute',
          top: '10%',
          left: '5%',
          right: '5%',
          opacity: titleOpacity,
        }}
      >
        <h1 style={{ color: 'white', fontSize: '48px', fontWeight: 'bold' }}>
          {title}
        </h1>
      </div>

      {/* Content Section */}
      <div
        style={{
          position: 'absolute',
          top: '30%',
          left: '5%',
          right: '5%',
          opacity: contentOpacity,
        }}
      >
        <p style={{ color: '#e0e0e0', fontSize: '24px', lineHeight: '1.6' }}>
          {content.substring(0, 200)}...
        </p>
      </div>

      {/* Image Carousel */}
      {images.length > 0 && (
        <div style={{ position: 'absolute', bottom: '10%', width: '100%' }}>
          {images.map((img, idx) => {
            const imageStart = 60 + (idx * 90);
            const imageOpacity = interpolate(
              frame,
              [imageStart, imageStart + 30, imageStart + 60, imageStart + 90],
              [0, 1, 1, 0],
              { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
            );

            return (
              <Img
                key={idx}
                src={img}
                style={{
                  width: '90%',
                  height: 'auto',
                  margin: '0 5%',
                  opacity: imageOpacity,
                  position: 'absolute',
                }}
              />
            );
          })}
        </div>
      )}
    </AbsoluteFill>
  );
};
```

### 5. End-to-End Pipeline

```typescript
// lib/pipeline/orchestrator.ts
import { scanSources } from '../research/scanner';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';

interface PipelineRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo: boolean;
}

export async function runContentPipeline(request: PipelineRequest) {
  console.log('🔍 Step 1: Researching...');
  const researchData = await scanSources(request.keyword, '24h');
  
  console.log(`✅ Found ${researchData.length} relevant sources`);
  
  console.log('🤖 Step 2: Generating content...');
  const content = await generateContent({
    keyword: request.keyword,
    format: request.format,
    language: request.language,
    tone: request.tone,
    researchData,
  }, 'claude');
  
  console.log('✅ Content generated');
  
  let videoPath = null;
  
  if (request.includeVideo) {
    console.log('🎬 Step 3: Rendering video...');
    
    const images = researchData
      .filter(item => item.imageUrl)
      .map(item => item.imageUrl)
      .slice(0, 3);
    
    videoPath = await renderContentVideo({
      title: request.keyword,
      content: content.substring(0, 300),
      imageUrls: images,
      duration: 30,
      format: 'reels',
    });
    
    console.log('✅ Video rendered:', videoPath);
  }
  
  return {
    content,
    videoPath,
    researchSources: researchData.length,
    timestamp: new Date().toISOString(),
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'expert',
      includeVideo: body.includeVideo || false,
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Generate a toplist article about AI tools
const result = await runContentPipeline({
  keyword: 'AI productivity tools 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  includeVideo: false,
});

console.log(result.content);
```

### Full Pipeline with Video

```typescript
// Generate complete content package with video
const fullPackage = await runContentPipeline({
  keyword: 'marketing automation trends',
  format: 'case-study',
  language: 'vi',
  tone: 'friendly',
  includeVideo: true,
});

console.log('Content:', fullPackage.content);
console.log('Video:', fullPackage.videoPath);
console.log('Sources used:', fullPackage.researchSources);
```

### Frontend Integration

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        includeVideo: formData.get('includeVideo') === 'on',
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <form onSubmit={handleGenerate}>
      <input name="keyword" placeholder="Enter keyword..." required />
      
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>
      
      <select name="language">
        <option value="en">English</option>
        <option value="vi">Tiếng Việt</option>
      </select>
      
      <select name="tone">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      
      <label>
        <input type="checkbox" name="includeVideo" />
        Generate Video
      </label>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Content:</h3>
          <div dangerouslySetInnerHTML={{ __html: result.data.content }} />
          
          {result.data.videoPath && (
            <video src={result.data.videoPath} controls />
          )}
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple formats for the same keyword
async function generateMultiFormat(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  
  const results = await Promise.all(
    formats.map(format =>
      runContentPipeline({
        keyword,
        format,
        language: 'en',
        tone: 'expert',
        includeVideo: false,
      })
    )
  );
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// Using node-cron for scheduled generation
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI news', 'tech trends', 'startup funding'];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      includeVideo: true,
    });
    
    // Save to database or publish directly
    await saveToDatabase(result);
  }
});
```

### Custom Research Sources

```typescript
// Add custom research sources
import axios from 'axios';

async function customResearch(keyword: string) {
  const sources = [
    `https://api.example.com/news?q=${keyword}`,
    `https://api.another.com/posts?search=${keyword}`,
  ];
  
  const results = await Promise.all(
    sources.map(url =>
      axios.get(url, {
        headers: { 'X-API-Key': process.env.CUSTOM_API_KEY }
      })
    )
  );
  
  return results.flatMap(r => r.data.items);
}
```

## Troubleshooting

### Issue: Research returns no results

```typescript
// Add error handling and fallback sources
async function robustScanSources(keyword: string) {
  try {
    const results = await scanSources(keyword, '24h');
    
    if (results.length === 0) {
      console.warn('No results found, trying 7-day timeframe');
      return await scanSources(keyword, '7d');
    }
    
    return results;
  } catch (error) {
    console.error('Research failed:', error);
    // Use fallback: search Google News API or RSS feeds
    return await fallbackResearch(keyword);
  }
}
```

### Issue: AI generation timeout

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request, 'claude');
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Issue: Video rendering fails

```typescript
// Check Remotion configuration and add error handling
async function safeRenderVideo(config: VideoConfig) {
  try {
    // Validate inputs
    if (!config.imageUrls.every(url => url.startsWith('http'))) {
      throw new Error('Invalid image URLs');
    }
    
    return await renderContentVideo(config);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback: generate static image instead
    return await generateStaticImage(config);
  }
}
```

### Issue: Rate limiting

```typescript
// Implement rate limiting with bottleneck
import Bottleneck from 'bottleneck';

const limiter = new Bottleneck({
  minTime: 1000, // 1 request per second
  maxConcurrent: 1,
});

const rateLimitedGenerate = limiter.wrap(generateContent);

// Use rate-limited version
const content = await rateLimitedGenerate(request, 'claude');
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedScanSources(keyword: string) {
  const cacheKey = `research:${keyword}:24h`;
  
  // Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Fetch fresh data
  const results = await scanSources(keyword, '24h');
  
  // Cache for 1 hour
  await redis.setex(cacheKey, 3600, JSON.stringify(results));
  
  return results;
}
```

### Parallel Processing

```typescript
// Process multiple keywords in parallel
async function batchGenerate(keywords: string[]) {
  const chunks = chunkArray(keywords, 5); // Process 5 at a time
  
  for (const chunk of chunks) {
    await Promise.all(
      chunk.map(keyword =>
        runContentPipeline({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          includeVideo: false,
        })
      )
    );
  }
}

function chunkArray<T>(array: T[], size: number): T[][] {
  return Array.from({ length: Math.ceil(array.length / size) }, (_, i) =>
    array.slice(i * size, i * size + size)
  );
}
```

## Building for Production

```bash
# Build Next.js app
npm run build

# Start production server
npm run start

# Build Remotion compositions
npm run remotion:build

# Deploy to Vercel
vercel --prod
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { runContentPipeline } from '../lib/pipeline/orchestrator';

describe('Content Pipeline', () => {
  it('should generate content successfully', async () => {
    const result = await runContentPipeline({
      keyword: 'test keyword',
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      includeVideo: false,
    });
    
    expect(result.content).toBeTruthy();
    expect(result.researchSources).toBeGreaterThan(0);
  }, 30000); // 30s timeout
});
```
