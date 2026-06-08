---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - generate videos from articles automatically
  - crawl news and create content with Claude
  - use Remotion to render marketing videos
  - build automated content workflow with OpenAI
  - create multi-format content with AI research
  - automate content from research to video
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with the Ultimate AI Content Pipeline, an all-in-one automated content creation system that handles research, scriptwriting, and video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a Next.js-based TypeScript application that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter, LinkedIn) for recent news
- **Generates diverse content formats** (toplists, POV articles, case studies, how-tos) in multiple languages
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multi-platform** distribution (Reels, TikTok, Shorts)

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

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Components and Usage

### 1. Research Crawler

The crawler fetches recent news from multiple sources:

```typescript
// src/lib/crawler/research-crawler.ts
import { fetchTechCrunchNews } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function crawlResearch(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = await Promise.all([
    fetchTechCrunchNews(keyword, timeframe),
    fetchTwitterTrends(keyword),
    // Add more sources as needed
  ]);
  
  return sources.flat().sort(
    (a, b) => b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

export async function generateContent(
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(config: ContentConfig): string {
  const researchContext = config.research
    .map(r => `- ${r.title}: ${r.summary} (${r.source})`)
    .join('\n');

  return `Create a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research context from the last 24 hours:
${researchContext}

Generate engaging, data-backed content that incorporates these insights.`;
}
```

### 3. Video Rendering with Remotion

Generate videos from article content:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'reels' | 'tiktok' | 'shorts';
}

export async function renderVideo(
  config: VideoConfig
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.style,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}
```

### 4. Complete Pipeline API Route

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlResearch } from '@/lib/crawler/research-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = 
      await request.json();

    // Step 1: Research
    const research = await crawlResearch(keyword, '24h');
    
    // Step 2: Generate content
    const content = await generateContent({
      format,
      language,
      tone,
      research,
    });

    // Step 3: Render video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const videoPath = await renderVideo({
        title: keyword,
        content,
        style: 'reels',
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        research: research.slice(0, 5), // Top 5 sources
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);
    
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
    <form action={handleGenerate}>
      <input name="keyword" placeholder="Enter keyword" required />
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">POV Article</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to Guide</option>
      </select>
      <select name="language">
        <option value="en">English</option>
        <option value="vi">Vietnamese</option>
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
          <h3>Generated Content</h3>
          <div dangerouslySetInnerHTML={{ __html: result.data.content }} />
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls />
          )}
        </div>
      )}
    </form>
  );
}
```

## Remotion Video Template Example

```typescript
// remotion/Composition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface VideoProps {
  title: string;
  content: string;
}

export const ReelsComposition: React.FC<VideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 60,
            color: 'white',
            opacity: frame / 60,
          }}
        >
          <h1>{title}</h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={60} durationInFrames={180}>
        <AbsoluteFill
          style={{
            padding: 40,
            fontSize: 24,
            color: 'white',
            justifyContent: 'center',
          }}
        >
          <p>{content.slice(0, 200)}...</p>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content
async function generateBatch(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await crawlResearch(keyword);
      const content = await generateContent({
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        research,
      });
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// Use with cron or queue system
export async function scheduledPipeline() {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    const research = await crawlResearch(topic);
    const content = await generateContent({
      format: 'pov',
      language: 'en',
      tone: 'friendly',
      research,
    });
    
    // Save to database or publish
    await saveContent({ topic, content });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Crawls

```typescript
// Process in chunks
async function crawlInChunks(keywords: string[], chunkSize = 5) {
  const results = [];
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(k => crawlResearch(k))
    );
    results.push(...chunkResults.flat());
  }
  return results;
}
```

## Running the Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Building for Production

```bash
npm run build
npm run start
```
