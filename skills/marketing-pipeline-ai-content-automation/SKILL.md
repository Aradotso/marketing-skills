---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and social posting using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered content workflow from research to video"
  - "automate social media content creation with AI"
  - "build content automation system with Claude and OpenAI"
  - "generate videos from text content automatically"
  - "scrape trending topics and create content"
  - "set up multi-format AI content generator"
  - "create automated marketing content pipeline"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables you to work with **Ultimate AI Content Pipeline**, a complete TypeScript-based automation system that transforms keywords into fully-formatted content and videos. The pipeline handles: research/scraping from major sources (TechCrunch, Twitter, LinkedIn), AI content generation in multiple formats (toplist, POV, case study, how-to), bilingual output (EN/VI), and automatic video rendering via Remotion.

## What This Project Does

The marketing-pipeline-share project is an end-to-end content automation factory:

1. **Auto-Research**: Crawls fresh data from news sources, social platforms
2. **AI Content Generation**: Uses Claude 3/OpenAI to create articles in various formats and tones
3. **Video Rendering**: Converts text content to infographics and short videos using Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Platform APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Content scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Patterns

### 1. Content Research & Scraping

```typescript
// lib/scraper/research.ts
import axios from 'axios';

interface ResearchSource {
  platform: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
  query: string;
  timeframe: '24h' | '7d' | '30d';
}

export async function scrapeContent(source: ResearchSource) {
  const { platform, query, timeframe } = source;
  
  const config = {
    method: 'get',
    url: `https://api.rapidapi.com/${platform}/search`,
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': `${platform}.rapidapi.com`
    },
    params: {
      q: query,
      timeframe: timeframe
    }
  };

  try {
    const response = await axios.request(config);
    return response.data;
  } catch (error) {
    console.error('Scraping error:', error);
    throw new Error(`Failed to scrape ${platform}`);
  }
}

// Extract insights from raw data
export function extractInsights(rawData: any[]): string[] {
  return rawData
    .filter(item => item.engagement > 1000)
    .map(item => item.summary || item.text)
    .slice(0, 10);
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContent(config: ContentConfig) {
  const { keyword, format, language, tone, researchData } = config;
  
  const systemPrompt = `You are an expert content creator specializing in ${format} format.
Write in ${language} with a ${tone} tone.
Use the following research insights: ${researchData.join('\n')}`;

  const userPrompt = `Create a comprehensive ${format} article about "${keyword}".
Include data-backed insights, real examples, and actionable takeaways.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. OpenAI Alternative Implementation

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(config: ContentConfig) {
  const { keyword, format, language, tone, researchData } = config;
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Expert ${format} content writer. Language: ${language}. Tone: ${tone}.
Research data: ${researchData.join('\n')}`
      },
      {
        role: 'user',
        content: `Write a ${format} article about "${keyword}" with real insights and data.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Remotion Video Generation

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

const DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderContentVideo(config: VideoConfig) {
  const { title, keyPoints, stats, platform } = config;
  const { width, height } = DIMENSIONS[platform];

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      keyPoints,
      stats
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      keyPoints,
      stats
    },
  });

  return outputLocation;
}
```

### 5. End-to-End Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { scrapeContent, extractInsights } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function runContentPipeline(keyword: string) {
  console.log('🔍 Starting content pipeline for:', keyword);

  // Step 1: Research
  const sources = ['techcrunch', 'twitter', 'linkedin'] as const;
  const researchResults = await Promise.all(
    sources.map(platform => 
      scrapeContent({ platform, query: keyword, timeframe: '24h' })
    )
  );
  
  const insights = extractInsights(researchResults.flat());
  console.log('✅ Research complete. Found', insights.length, 'insights');

  // Step 2: Generate Content (English)
  const contentEN = await generateContent({
    keyword,
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    researchData: insights
  });
  console.log('✅ English content generated');

  // Step 3: Generate Content (Vietnamese)
  const contentVI = await generateContent({
    keyword,
    format: 'toplist',
    language: 'vi',
    tone: 'friendly',
    researchData: insights
  });
  console.log('✅ Vietnamese content generated');

  // Step 4: Extract key points for video
  const keyPoints = extractKeyPoints(contentEN);
  const stats = extractStats(contentEN);

  // Step 5: Render Videos
  const videoReels = await renderContentVideo({
    title: keyword,
    keyPoints,
    stats,
    platform: 'reels'
  });
  console.log('✅ Video rendered:', videoReels);

  return {
    contentEN,
    contentVI,
    videoPath: videoReels,
    insights
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  return content
    .split('\n')
    .filter(line => line.match(/^[\d•-]/))
    .slice(0, 5)
    .map(line => line.replace(/^[\d•-]\s*/, ''));
}

function extractStats(content: string): { label: string; value: string }[] {
  const regex = /(\d+%|\$\d+[MBK]?|\d+x)/g;
  const matches = content.match(regex) || [];
  return matches.slice(0, 3).map((value, i) => ({
    label: `Metric ${i + 1}`,
    value
  }));
}
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
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
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="flex gap-4 mb-6">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI Marketing)"
          className="flex-1 px-4 py-2 border rounded"
        />
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate'}
        </button>
      </div>

      {result && (
        <div className="space-y-6">
          <div className="p-4 bg-gray-50 rounded">
            <h2 className="font-bold mb-2">English Content</h2>
            <p className="whitespace-pre-wrap">{result.contentEN}</p>
          </div>
          
          <div className="p-4 bg-gray-50 rounded">
            <h2 className="font-bold mb-2">Vietnamese Content</h2>
            <p className="whitespace-pre-wrap">{result.contentVI}</p>
          </div>

          <div className="p-4 bg-gray-50 rounded">
            <h2 className="font-bold mb-2">Generated Video</h2>
            <video src={result.videoPath} controls className="w-full" />
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Workflows

### Full Automation Workflow

```typescript
// scripts/auto-publish.ts
import { runContentPipeline } from '@/lib/pipeline/orchestrator';
import { publishToFacebook } from '@/lib/social/facebook';

const TRENDING_KEYWORDS = [
  'AI Marketing Trends 2026',
  'Content Automation Tools',
  'Video Marketing Strategy'
];

async function autoPublish() {
  for (const keyword of TRENDING_KEYWORDS) {
    console.log(`Processing: ${keyword}`);
    
    const { contentEN, videoPath } = await runContentPipeline(keyword);
    
    await publishToFacebook({
      message: contentEN.slice(0, 500),
      videoPath: videoPath
    });
    
    // Wait between posts
    await new Promise(resolve => setTimeout(resolve, 60000));
  }
}

autoPublish();
```

### Batch Content Generation

```typescript
// lib/batch/processor.ts
export async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline(keyword);
      results.push({ keyword, ...result, status: 'success' });
    } catch (error) {
      results.push({ keyword, status: 'failed', error: error.message });
    }
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private lastCall = 0;
  private minInterval: number;

  constructor(requestsPerMinute: number) {
    this.minInterval = (60 * 1000) / requestsPerMinute;
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
await limiter.throttle();
await scrapeContent(source);
```

### Video Rendering Issues

If Remotion fails to render:
- Ensure FFmpeg is installed: `brew install ffmpeg` (macOS) or `apt-get install ffmpeg` (Linux)
- Check license key validity
- Verify composition ID matches your Remotion config

### AI Model Fallback

```typescript
export async function generateWithFallback(config: ContentConfig) {
  try {
    return await generateContent(config); // Claude
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI');
    return await generateWithGPT(config); // OpenAI
  }
}
```

## Running the Development Server

```bash
npm run dev
# Access at http://localhost:3000
```

## Building for Production

```bash
npm run build
npm start
```

This skill equips AI agents to help developers implement, customize, and troubleshoot the complete marketing content automation pipeline.
