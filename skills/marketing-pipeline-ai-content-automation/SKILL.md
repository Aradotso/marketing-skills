---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and generation
  - set up AI content pipeline with video rendering
  - create automated marketing content workflow
  - generate videos from articles automatically
  - build content automation with Claude and OpenAI
  - crawl news sources and create content with AI
  - automate social media content creation pipeline
  - research to video content generation system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline that takes a keyword, researches current news/trends, generates articles in multiple formats, and renders them as videos. Built with Next.js, TypeScript, Claude 3, OpenAI, and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically converts articles to videos using Remotion
5. **Social Media Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web crawling & research
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Features & Usage

### 1. Research & Data Collection

```typescript
// src/lib/research/crawler.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function crawlTechNews(keyword: string): Promise<NewsArticle[]> {
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];

  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/search`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        },
        params: {
          q: keyword,
          source: source,
          from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Error crawling ${source}:`, error);
    }
  }
  
  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateArticle(request: ContentRequest): Promise<string> {
  const systemPrompt = `You are an expert content writer. Create a ${request.format} article about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'} with a ${request.tone} tone.`;
  
  const userPrompt = `Based on this research data:\n\n${request.researchData}\n\nCreate a comprehensive ${request.format} article that:
- Uses latest data and insights
- Includes specific examples and statistics
- Is optimized for social media engagement
- Follows ${request.format} structure`;

  const message = await client.messages.create({
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

### 3. OpenAI Integration (Alternative)

```typescript
// src/lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(
  keyword: string,
  researchData: string,
  format: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert specializing in ${format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${format} article about "${keyword}" using this research:\n\n${researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

export interface VideoConfig {
  title: string;
  content: string[];
  format: 'reels' | 'tiktok' | 'youtube-shorts';
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config,
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: config,
  });

  return outputLocation;
}
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

interface Props {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<Props> = ({ title, content }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60
        }}>
          <h1 style={{
            fontSize: 72,
            color: 'white',
            textAlign: 'center',
            opacity: Math.min(1, frame / 30)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {content.map((text, index) => (
        <Sequence key={index} from={90 + index * 120} durationInFrames={120}>
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60
          }}>
            <p style={{
              fontSize: 48,
              color: 'white',
              textAlign: 'center',
              lineHeight: 1.5
            }}>
              {text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline API Route

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechNews } from '@/lib/research/crawler';
import { generateArticle } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    // Step 1: Research
    const articles = await crawlTechNews(keyword);
    const researchData = articles
      .map(a => `${a.title}\n${a.content}`)
      .join('\n\n---\n\n');

    // Step 2: Generate Content
    const article = await generateArticle({
      keyword,
      format,
      language,
      tone,
      researchData
    });

    // Step 3: Parse content for video
    const lines = article.split('\n').filter(line => line.trim());
    const title = lines[0];
    const content = lines.slice(1, 6); // First 5 key points

    // Step 4: Render Video
    const videoPath = await renderContentVideo({
      title,
      content,
      format: 'reels'
    });

    return NextResponse.json({
      success: true,
      article,
      videoUrl: videoPath.replace(process.cwd() + '/public', '')
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Component

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'vi',
          tone: 'expert'
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
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-3 bg-blue-600 text-white rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-4">
          <div className="p-4 bg-gray-50 rounded">
            <h2 className="font-bold mb-2">Generated Article:</h2>
            <pre className="whitespace-pre-wrap">{result.article}</pre>
          </div>

          {result.videoUrl && (
            <div>
              <h2 className="font-bold mb-2">Generated Video:</h2>
              <video controls className="w-full rounded">
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

## Running the Project

```bash
# Development
npm run dev

# Build
npm run build

# Start production
npm start

# Render videos (if using Remotion CLI)
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await crawlTechNews(keyword);
      const researchData = articles.map(a => a.content).join('\n');
      
      return await generateArticle({
        keyword,
        format: 'toplist',
        language: 'vi',
        tone: 'expert',
        researchData
      });
    })
  );
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// Use with cron job or background worker
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await generateAndPublish(topic);
  }
});
```

## Troubleshooting

**API Rate Limits**
- Implement request queuing and rate limiting
- Use Redis for distributed rate limiting across instances

**Video Rendering Timeouts**
- Increase serverless function timeout limits
- Consider offloading to background workers
- Use Remotion Lambda for cloud rendering

**Memory Issues with Large Research Data**
- Implement pagination for news crawling
- Summarize research data before passing to AI
- Stream responses when possible

**Claude/OpenAI API Errors**
- Implement retry logic with exponential backoff
- Fallback between Claude and OpenAI
- Cache generated content to avoid regeneration

```typescript
// Retry helper
async function withRetry<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
    }
  }
  throw new Error('Max retries exceeded');
}
```
