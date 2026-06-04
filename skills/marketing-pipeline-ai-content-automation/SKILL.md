---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, auto-posting and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline content automation
  - generate social media posts with AI research
  - create automated content workflow with video rendering
  - build AI content pipeline from research to publishing
  - integrate Claude and Remotion for content automation
  - automate blog posts and video generation
  - schedule automated AI content posting
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The Marketing Pipeline Share project is an end-to-end AI content automation system that:

- **Auto-researches** trending topics from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** and infographics automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, Shorts

Built with TypeScript, Next.js, and integrated with OpenAI, Anthropic Claude, and RapidAPI.

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if used)
DATABASE_URL=your_database_url
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration modules
│   │   ├── claude.ts      # Claude API wrapper
│   │   ├── openai.ts      # OpenAI API wrapper
│   │   └── prompts.ts     # Prompt templates
│   ├── research/          # Content research modules
│   │   ├── crawler.ts     # Web scraping logic
│   │   └── sources.ts     # News source configs
│   └── video/             # Remotion video generation
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Key Features & Usage

### 1. Auto-Research Content

```typescript
// lib/research/crawler.ts
import { fetchTrendingTopics } from './sources';

interface ResearchResult {
  title: string;
  summary: string;
  sources: string[];
  insights: string[];
  publishedAt: Date;
}

export async function researchTopic(keyword: string): Promise<ResearchResult> {
  const topics = await fetchTrendingTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  const insights = topics.map(topic => ({
    title: topic.title,
    summary: topic.description,
    url: topic.url,
    engagement: topic.metrics
  }));
  
  return {
    title: keyword,
    summary: aggregateSummary(insights),
    sources: insights.map(i => i.url),
    insights: extractKeyInsights(insights),
    publishedAt: new Date()
  };
}
```

### 2. Generate Content with AI

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchResult;
}

export async function generateContent(params: ContentParams): Promise<string> {
  const prompt = buildPrompt(params);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(params: ContentParams): string {
  return `
You are a professional content writer creating a ${params.format} article.

Topic: ${params.topic}
Tone: ${params.tone}
Language: ${params.language}

Research Data:
${JSON.stringify(params.researchData, null, 2)}

Create an engaging, data-backed article that:
- Uses the latest insights from research
- Follows the ${params.format} format
- Maintains a ${params.tone} tone
- Is optimized for social media engagement
`;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(params: ContentParams): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${params.format} articles.`
      },
      {
        role: 'user',
        content: buildPrompt(params)
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function generateVideo(config: VideoConfig): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  const spec = PLATFORM_SPECS[config.platform];
  const outputLocation = `out/video-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    ...spec,
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Workflow

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, tone, language, platform } = await req.json();
    
    // Step 1: Research
    const research = await researchTopic(keyword);
    
    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format,
      tone,
      language,
      researchData: research
    });
    
    // Step 3: Create Video
    const videoPath = await generateVideo({
      content,
      title: research.title,
      platform
    });
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        research,
        video: videoPath
      }
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function generateContent(e: React.FormEvent) {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.target as HTMLFormElement);
    
    const response = await fetch('/api/content/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        tone: formData.get('tone'),
        language: formData.get('language'),
        platform: formData.get('platform')
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <form onSubmit={generateContent}>
      <input name="keyword" placeholder="Enter topic keyword" required />
      
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>
      
      <select name="tone">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      
      <select name="language">
        <option value="en">English</option>
        <option value="vi">Vietnamese</option>
      </select>
      
      <select name="platform">
        <option value="reels">Instagram Reels</option>
        <option value="tiktok">TikTok</option>
        <option value="shorts">YouTube Shorts</option>
      </select>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Generated Content:</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Bilingual Content Generation

```typescript
async function generateBilingualContent(params: Omit<ContentParams, 'language'>) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ ...params, language: 'en' }),
    generateContent({ ...params, language: 'vi' })
  ]);
  
  return { en: englishContent, vi: vietnameseContent };
}
```

### Batch Processing

```typescript
async function processBatchTopics(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      researchTopic(keyword).then(research =>
        generateContent({
          topic: keyword,
          format: 'toplist',
          tone: 'expert',
          language: 'en',
          researchData: research
        })
      )
    )
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Scheduled Content Pipeline

```typescript
// lib/scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const trendingTopics = await fetchTrendingTopics({
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h'
    });
    
    for (const topic of trendingTopics.slice(0, 3)) {
      const research = await researchTopic(topic.keyword);
      const content = await generateContent({
        topic: topic.keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        researchData: research
      });
      
      // Auto-publish or save to CMS
      await publishContent(content);
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>, delay = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          await new Promise(r => setTimeout(r, delay));
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      await fn();
    }
    this.processing = false;
  }
}

export const rateLimiter = new RateLimiter();
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries === 0) throw error;
    
    await new Promise(r => setTimeout(r, delay));
    return withRetry(fn, retries - 1, delay * 2);
  }
}

// Usage
const content = await withRetry(() => 
  generateContent(params)
);
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large videos
export async function renderWithOptimization(config: VideoConfig) {
  const outputLocation = await generateVideo(config);
  
  // Clean up temporary files
  await fs.promises.rm(bundleLocation, { recursive: true });
  
  return outputLocation;
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run type checking
npm run type-check

# Lint code
npm run lint
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid repeated API calls
2. **Queue Video Rendering**: Use a job queue for video generation to prevent memory issues
3. **Monitor API Usage**: Track OpenAI/Claude token usage to manage costs
4. **Validate Content**: Implement content moderation before auto-publishing
5. **Store Generated Assets**: Save videos and content to cloud storage (S3, Cloudinary)
6. **A/B Test Formats**: Track engagement metrics per content format and tone
