---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with marketing pipeline
  - create video content from text automatically
  - research and write blog posts with AI
  - use remotion for content video generation
  - automate content creation workflow
  - configure Claude and OpenAI for content pipeline
  - crawl news and generate articles automatically
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, article generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Auto Content is an end-to-end content automation system that:

- **Auto-crawls** latest news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates articles** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual** content (English & Vietnamese) with customizable tone
- **Renders videos** automatically from text using Remotion
- **Exports platform-optimized** content for Reels, TikTok, Shorts

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
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling API
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Utility functions
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & News Crawling

```typescript
// src/lib/crawlers/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  source: string;
}

export async function crawlLatestNews(
  keywords: string[],
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        },
        params: {
          keywords: keywords.join(','),
          timeframe: '24h'
        }
      }
    );
    
    articles.push(...response.data.articles);
  }
  
  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  targetAudience: string;
}

export async function generateArticle(
  research: string,
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(research, keyword, config);
  
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
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(
  research: string,
  keyword: string,
  config: ContentConfig
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting top items',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze a specific example with data and insights',
    'how-to': 'Provide step-by-step actionable guidance'
  };
  
  return `
You are an expert content writer. Create a ${config.format} article about "${keyword}".

Research Data:
${research}

Requirements:
- Format: ${formatInstructions[config.format]}
- Tone: ${config.tone}
- Language: ${config.language}
- Target Audience: ${config.targetAudience}
- Include data-backed insights
- Make it SEO-friendly
- Add engaging headlines

Generate the complete article now:
  `.trim();
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(
  research: string,
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(research, keyword, config);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content writer.'
      },
      {
        role: 'user',
        content: prompt
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
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = Math.min(1, frame / 20);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          color: brandColor,
          fontSize: 60,
          fontWeight: 'bold',
          textAlign: 'center',
          padding: 40,
        }}
      >
        {title}
      </div>
      
      <div style={{ marginTop: 40 }}>
        {keyPoints.map((point, index) => {
          const pointFrame = frame - (index * fps * 2);
          const pointOpacity = Math.max(0, Math.min(1, pointFrame / 20));
          
          return (
            <div
              key={index}
              style={{
                opacity: pointOpacity,
                color: '#fff',
                fontSize: 32,
                margin: '20px 0',
                textAlign: 'center',
              }}
            >
              • {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  brandColor: string;
  outputPath: string;
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      brandColor: config.brandColor,
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      brandColor: config.brandColor,
    },
  });
  
  return config.outputPath;
}
```

## Complete Workflow Example

```typescript
// src/app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlLatestNews } from '@/lib/crawlers/news-scraper';
import { generateArticle } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    // Step 1: Research
    console.log('Crawling latest news...');
    const articles = await crawlLatestNews([keyword]);
    const research = articles
      .map(a => `${a.title}\n${a.content}`)
      .join('\n\n');
    
    // Step 2: Generate Article
    console.log('Generating article...');
    const article = await generateArticle(research, keyword, {
      format,
      tone: 'professional',
      language,
      targetAudience: 'marketers and content creators'
    });
    
    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(article);
    
    // Step 4: Render Video
    console.log('Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      keyPoints,
      brandColor: '#6366f1',
      outputPath: `public/videos/${Date.now()}.mp4`
    });
    
    return NextResponse.json({
      success: true,
      article,
      videoUrl: videoPath.replace('public', '')
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(article: string): string[] {
  // Simple extraction - in production, use AI to summarize
  const lines = article.split('\n').filter(line => 
    line.trim().startsWith('-') || 
    line.trim().startsWith('•') ||
    /^\d+\./.test(line.trim())
  );
  
  return lines.slice(0, 5).map(line => 
    line.replace(/^[-•\d.]\s*/, '').trim()
  );
}
```

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, language })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full px-4 py-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV / Opinion</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <select
          value={language}
          onChange={(e) => setLanguage(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="en">English</option>
          <option value="vi">Tiếng Việt</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 space-y-4">
          <div className="prose max-w-none">
            <h2>Generated Article</h2>
            <div dangerouslySetInnerHTML={{ __html: result.article }} />
          </div>
          
          {result.videoUrl && (
            <div>
              <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
              <video controls className="w-full">
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

## Common Patterns

### Batch Content Generation

```typescript
// src/lib/batch/batch-generator.ts
export async function generateBatchContent(
  keywords: string[],
  config: ContentConfig
): Promise<Array<{ keyword: string; article: string; videoPath: string }>> {
  const results = [];
  
  for (const keyword of keywords) {
    const articles = await crawlLatestNews([keyword]);
    const research = articles.map(a => a.content).join('\n\n');
    
    const article = await generateArticle(research, keyword, config);
    const keyPoints = extractKeyPoints(article);
    
    const videoPath = await renderContentVideo({
      title: keyword,
      keyPoints,
      brandColor: '#6366f1',
      outputPath: `public/videos/${keyword}-${Date.now()}.mp4`
    });
    
    results.push({ keyword, article, videoPath });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// remotion/templates/ReelsTemplate.tsx
export const ReelsTemplate: React.FC<{
  headline: string;
  stats: Array<{ label: string; value: string }>;
}> = ({ headline, stats }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* 9:16 vertical format optimized for Reels/TikTok */}
      <div style={{ 
        aspectRatio: '9/16',
        margin: 'auto',
        padding: 40 
      }}>
        <h1 style={{ fontSize: 48, color: '#fff' }}>
          {headline}
        </h1>
        
        {stats.map((stat, i) => (
          <div key={i} style={{ 
            marginTop: 30,
            opacity: frame > (i + 1) * 30 ? 1 : 0,
            transition: 'opacity 0.5s'
          }}>
            <div style={{ fontSize: 64, color: '#6366f1' }}>
              {stat.value}
            </div>
            <div style={{ fontSize: 24, color: '#999' }}>
              {stat.label}
            </div>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
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
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.processing = false;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce video quality for faster rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: config.outputPath,
  scale: 0.5, // Reduce to 50% for faster processing
  quality: 80, // Lower quality = smaller file
  concurrency: 1, // Limit concurrent frames
});
```

### Claude/OpenAI Timeout Handling

```typescript
export async function generateWithRetry(
  fn: () => Promise<string>,
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      
      if (error.status === 429) {
        // Rate limit - wait longer
        await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
      } else {
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

This skill provides comprehensive coverage of the marketing automation pipeline including research crawling, AI-powered content generation, and automated video rendering using TypeScript and modern frameworks.
