---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and Remotion
  - create automated content workflow from research to video
  - generate AI content with automatic research crawling
  - build content automation pipeline with OpenAI and video rendering
  - use ultimate AI content pipeline for marketing automation
  - automate social media content from research to video
  - create multilingual content with AI and auto-generated videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research and crawling to script generation and video rendering. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content automation.

## What It Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls news sources** (TechCrunch, a16z, X/Twitter, LinkedIn) for real-time data
- **Generates multi-format content** (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)
- **Provides Next.js UI** for easy content management and scheduling

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

```env
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Web Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core Architecture

### 1. Research & Data Crawling

The pipeline starts by crawling recent articles from configured sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface CrawlResult {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<CrawlResult[]> {
  const results: CrawlResult[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        },
        params: {
          q: keyword,
          from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
        },
      }
    );
    
    results.push(...response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      content: article.content,
      publishedAt: new Date(article.publishedAt),
      source,
    })));
  }
  
  return results;
}
```

### 2. Content Generation with AI

Generate structured content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: CrawlResult[];
}

export async function generateContent(
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const researchContext = config.researchData
    .map((r) => `[${r.source}] ${r.title}\n${r.content.slice(0, 500)}`)
    .join('\n\n');
  
  const prompt = `You are a ${config.tone} content creator. Create a ${config.format} article in ${config.language} about "${config.keyword}".

Research context from last 24h:
${researchContext}

Requirements:
- Use real data and insights from the research
- ${config.format === 'toplist' ? 'Create a numbered list with detailed explanations' : ''}
- ${config.format === 'case-study' ? 'Include specific examples and outcomes' : ''}
- Write in ${config.tone} tone
- Make it engaging and actionable`;

  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4000,
    });
    
    return completion.choices[0]?.message?.content || '';
  }
}
```

### 3. Video Generation with Remotion

Create videos from generated content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  bgColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  bgColor = '#1a1a1a',
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{
        padding: '60px',
        color: 'white',
        fontFamily: 'Inter, sans-serif',
      }}>
        <h1 style={{
          fontSize: '64px',
          marginBottom: '40px',
          opacity: titleOpacity,
        }}>
          {title}
        </h1>
        
        <div style={{ fontSize: '32px', lineHeight: 1.6 }}>
          {points.map((point, index) => {
            const pointStart = 60 + index * 60;
            const opacity = interpolate(
              frame,
              [pointStart, pointStart + 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div key={index} style={{ opacity, marginBottom: '20px' }}>
                {index + 1}. {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

Register the composition:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Top 5 AI Trends',
          points: [
            'AI-powered content automation',
            'Real-time data integration',
            'Multi-format generation',
            'Video automation',
            'Cross-platform optimization',
          ],
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### 4. Complete Pipeline Orchestration

Combine all steps into a unified workflow:

```typescript
// lib/pipeline/orchestrator.ts
import { crawlRecentNews } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  generateVideo: boolean;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  content: string;
  videoPath?: string;
}> {
  // Step 1: Research
  console.log('🔍 Crawling research data...');
  const researchData = await crawlRecentNews(config.keyword);
  
  // Step 2: Generate content
  console.log('✍️ Generating content with AI...');
  const content = await generateContent({
    keyword: config.keyword,
    format: config.format,
    tone: 'expert',
    language: config.language,
    researchData,
  });
  
  // Step 3: Generate video (optional)
  let videoPath: string | undefined;
  
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    
    // Extract title and points from content
    const lines = content.split('\n').filter(l => l.trim());
    const title = lines[0].replace(/^#+\s*/, '');
    const points = lines
      .filter(l => /^\d+\./.test(l.trim()))
      .map(l => l.replace(/^\d+\.\s*/, ''))
      .slice(0, 5);
    
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: { title, points },
    });
    
    const outputLocation = path.join(
      process.cwd(),
      'public/videos',
      `${config.keyword.replace(/\s+/g, '-')}-${Date.now()}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: { title, points },
    });
    
    videoPath = outputLocation;
    console.log(`✅ Video saved to: ${videoPath}`);
  }
  
  return { content, videoPath };
}
```

## API Routes (Next.js)

Create API endpoints for the pipeline:

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideo } = body;
    
    if (!keyword || !format || !language) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      language,
      generateVideo: generateVideo ?? false,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

Create a form to trigger the pipeline:

```typescript
// app/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [generateVideo, setGenerateVideo] = useState(false);
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, language, generateVideo }),
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
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block mb-2">Keyword</label>
          <input
            type="text"
            value={keyword}
            onChange={(e) => setKeyword(e.target.value)}
            className="w-full p-2 border rounded"
            placeholder="e.g., AI Marketing Automation"
            required
          />
        </div>
        
        <div>
          <label className="block mb-2">Format</label>
          <select
            value={format}
            onChange={(e) => setFormat(e.target.value as any)}
            className="w-full p-2 border rounded"
          >
            <option value="toplist">Top List</option>
            <option value="pov">POV (Point of View)</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>
        
        <div>
          <label className="block mb-2">Language</label>
          <select
            value={language}
            onChange={(e) => setLanguage(e.target.value as any)}
            className="w-full p-2 border rounded"
          >
            <option value="en">English</option>
            <option value="vi">Vietnamese</option>
          </select>
        </div>
        
        <div>
          <label className="flex items-center">
            <input
              type="checkbox"
              checked={generateVideo}
              onChange={(e) => setGenerateVideo(e.target.checked)}
              className="mr-2"
            />
            Generate Video
          </label>
        </div>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-6 p-4 border rounded">
          <h3 className="font-bold mb-2">Generated Content:</h3>
          <div className="prose max-w-none">
            {result.content}
          </div>
          {result.videoPath && (
            <div className="mt-4">
              <h4 className="font-bold">Video:</h4>
              <video controls className="w-full mt-2">
                <source src={result.videoPath} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
// lib/pipeline/batch-processor.ts
export async function batchProcessKeywords(
  keywords: string[],
  baseConfig: Omit<PipelineConfig, 'keyword'>
) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline({ ...baseConfig, keyword })
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

### Caching Research Data

```typescript
// lib/research/cache.ts
import { createClient } from 'redis';

const redis = createClient({
  url: process.env.REDIS_URL,
});

export async function getCachedResearch(keyword: string) {
  await redis.connect();
  const cached = await redis.get(`research:${keyword}`);
  await redis.disconnect();
  return cached ? JSON.parse(cached) : null;
}

export async function cacheResearch(keyword: string, data: any) {
  await redis.connect();
  await redis.setEx(`research:${keyword}`, 3600, JSON.stringify(data));
  await redis.disconnect();
}
```

## Troubleshooting

### API Rate Limits

If hitting rate limits, implement retry logic:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large videos, use headless rendering:

```bash
npm run remotion:render -- --concurrency=1 --memory=4096
```

### TypeScript Errors

Ensure all type definitions are installed:

```bash
npm install -D @types/node @types/react @types/react-dom
```
