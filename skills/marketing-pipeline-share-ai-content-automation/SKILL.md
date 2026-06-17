---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate social media content with AI pipeline
  - create videos from articles automatically
  - build automated marketing content workflow
  - use AI for content research and generation
  - set up content automation with Claude and OpenAI
  - generate multilingual marketing content
  - automate social media video creation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end content automation system that handles research, scriptwriting, article generation, and video rendering. It crawls recent news from major sources (TechCrunch, a16z, Twitter, LinkedIn), generates content in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

This TypeScript/Next.js project provides:

- **Auto Research**: Crawls and analyzes real-time data from news sources
- **AI Content Generation**: Creates articles in multiple formats (Top Lists, POV, Case Studies, How-To)
- **Multi-language Support**: Generates content in English and Vietnamese
- **Video Generation**: Automatically renders infographics and short-form videos using Remotion
- **Multi-platform Optimization**: Exports videos for Reels, TikTok, Shorts

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

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```typescript
// Typical project structure
src/
├── app/                    // Next.js app directory
├── components/             // React components
├── lib/
│   ├── ai/                // AI integration (Claude, OpenAI)
│   ├── crawler/           // News crawling logic
│   ├── video/             // Remotion video generation
│   └── utils/             // Utilities
├── remotion/              // Remotion video compositions
└── types/                 // TypeScript types
```

## Core Features & Usage

### 1. Content Research & Crawling

```typescript
// lib/crawler/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  source: string;
  title: string;
  content: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  hours: number = 24
): Promise<NewsSource[]> {
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];
  
  const results: NewsSource[] = [];
  const sinceDate = new Date(Date.now() - hours * 60 * 60 * 1000);
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/search`, {
        params: {
          q: keyword,
          domain: source,
          since: sinceDate.toISOString()
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!
        }
      });
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Error crawling ${source}:`, error);
    }
  }
  
  return results;
}

export async function extractInsights(articles: NewsSource[]): Promise<string[]> {
  const insights = articles.map(article => ({
    title: article.title,
    summary: article.content.substring(0, 500),
    source: article.source
  }));
  
  return insights.map(i => `${i.title} - ${i.source}`);
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentParams {
  keyword: string;
  research: string[];
  format: ContentFormat;
  tone: Tone;
  language: Language;
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  keyword,
  research,
  format,
  tone,
  language,
  provider = 'claude'
}: GenerateContentParams): Promise<string> {
  const researchContext = research.join('\n\n');
  
  const systemPrompt = `You are a professional content writer. Create a ${format} article about "${keyword}" based on recent research. Use a ${tone} tone and write in ${language === 'en' ? 'English' : 'Vietnamese'}.`;
  
  const userPrompt = `Research data:\n${researchContext}\n\nCreate a comprehensive ${format} article with:\n- Engaging title\n- Clear structure\n- Data-backed insights\n- Actionable takeaways`;
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: `${systemPrompt}\n\n${userPrompt}`
      }]
    });
    
    return message.content[0].type === 'text' ? message.content[0].text : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
}
```

### 3. Multi-Language Content

```typescript
// lib/ai/multilingual-generator.ts
export async function generateMultilingualContent(
  keyword: string,
  research: string[],
  format: ContentFormat,
  tone: Tone
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      research,
      format,
      tone,
      language: 'en',
      provider: 'claude'
    }),
    generateContent({
      keyword,
      research,
      format,
      tone,
      language: 'vi',
      provider: 'openai'
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  brandColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / (fps * 1));
  const pointsStartFrame = fps * 2;
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        padding: '60px',
        color: '#fff',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{
          fontSize: '72px',
          fontWeight: 'bold',
          opacity: titleOpacity,
          color: brandColor,
          marginBottom: '40px'
        }}>
          {title}
        </h1>
        
        <div>
          {keyPoints.map((point, index) => {
            const pointFrame = pointsStartFrame + (index * fps * 1.5);
            const opacity = Math.min(1, Math.max(0, (frame - pointFrame) / fps));
            
            return (
              <div
                key={index}
                style={{
                  fontSize: '48px',
                  marginBottom: '30px',
                  opacity,
                  transform: `translateX(${Math.max(0, 50 - (frame - pointFrame))}px)`
                }}
              >
                • {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderVideoParams {
  title: string;
  keyPoints: string[];
  outputPath: string;
  format?: 'vertical' | 'square' | 'horizontal';
}

const FORMAT_DIMENSIONS = {
  vertical: { width: 1080, height: 1920 },   // TikTok, Reels
  square: { width: 1080, height: 1080 },     // Instagram
  horizontal: { width: 1920, height: 1080 }  // YouTube
};

export async function renderArticleVideo({
  title,
  keyPoints,
  outputPath,
  format = 'vertical'
}: RenderVideoParams): Promise<string> {
  const compositionId = 'ArticleVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      keyPoints,
      brandColor: '#FF6B6B'
    }
  });
  
  const dimensions = FORMAT_DIMENSIONS[format];
  
  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
      durationInFrames: 30 * (2 + keyPoints.length * 1.5) // 30 fps
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      keyPoints,
      brandColor: '#FF6B6B'
    }
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline API

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews, extractInsights } from '@/lib/crawler/news-crawler';
import { generateMultilingualContent } from '@/lib/ai/multilingual-generator';
import { renderArticleVideo } from '@/lib/video/render-video';
import path from 'path';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, tone, generateVideo } = await req.json();
    
    // Step 1: Research
    const articles = await crawlRecentNews(keyword, 24);
    const insights = await extractInsights(articles);
    
    // Step 2: Generate content
    const content = await generateMultilingualContent(
      keyword,
      insights,
      format,
      tone
    );
    
    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content.en);
    
    // Step 4: Generate video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const outputPath = path.join(
        process.cwd(),
        'public/videos',
        `${Date.now()}.mp4`
      );
      
      await renderArticleVideo({
        title: keyword,
        keyPoints,
        outputPath,
        format: 'vertical'
      });
      
      videoUrl = `/videos/${path.basename(outputPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      insights: insights.length
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - improve based on content structure
  const lines = content.split('\n').filter(line => 
    line.trim().startsWith('-') || line.trim().startsWith('•')
  );
  return lines.slice(0, 5).map(line => line.replace(/^[-•]\s*/, '').trim());
}
```

### 6. Frontend Usage

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<ContentFormat>('toplist');
  const [tone, setTone] = useState<Tone>('friendly');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          tone,
          generateVideo: true
        })
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4 mb-8">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full px-4 py-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as ContentFormat)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To</option>
        </select>
        
        <select
          value={tone}
          onChange={(e) => setTone(e.target.value as Tone)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="w-full px-4 py-2 bg-blue-600 text-white rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="space-y-4">
          <div>
            <h2 className="text-2xl font-bold mb-2">English Content</h2>
            <div className="p-4 bg-gray-100 rounded whitespace-pre-wrap">
              {result.content.en}
            </div>
          </div>
          
          <div>
            <h2 className="text-2xl font-bold mb-2">Vietnamese Content</h2>
            <div className="p-4 bg-gray-100 rounded whitespace-pre-wrap">
              {result.content.vi}
            </div>
          </div>
          
          {result.videoUrl && (
            <div>
              <h2 className="text-2xl font-bold mb-2">Generated Video</h2>
              <video controls className="w-full max-w-md">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
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

# Render Remotion video (preview)
npm run remotion:preview

# Render Remotion video (production)
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/batch-generator.ts
export async function batchGenerateContent(
  keywords: string[],
  format: ContentFormat,
  tone: Tone
): Promise<Map<string, { en: string; vi: string }>> {
  const results = new Map();
  
  for (const keyword of keywords) {
    const articles = await crawlRecentNews(keyword, 24);
    const insights = await extractInsights(articles);
    const content = await generateMultilingualContent(
      keyword,
      insights,
      format,
      tone
    );
    
    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  keywords: string[],
  schedule: string = '0 9 * * *' // Daily at 9 AM
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        const articles = await crawlRecentNews(keyword, 24);
        const insights = await extractInsights(articles);
        const content = await generateMultilingualContent(
          keyword,
          insights,
          'toplist',
          'friendly'
        );
        
        // Save to database or publish
        console.log(`Generated content for: ${keyword}`);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      if (!this.processing) {
        this.process();
      }
    });
  }
  
  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(2000);
const result = await limiter.add(() => generateContent({...}));
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  retries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      console.log(`Retry ${i + 1}/${retries}...`);
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('All retries failed');
}
```

## Performance Optimization

```typescript
// Use parallel processing for multi-language generation
const [en, vi] = await Promise.all([
  generateContent({ ...params, language: 'en' }),
  generateContent({ ...params, language: 'vi' })
]);

// Cache research results
import { Redis } from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

const cacheKey = `research:${keyword}:${hours}h`;
const cached = await redis.get(cacheKey);

if (cached) {
  return JSON.parse(cached);
}

const results = await crawlRecentNews(keyword, hours);
await redis.setex(cacheKey, 3600, JSON.stringify(results));
```
