---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build marketing content pipeline with Claude and OpenAI
  - generate videos from blog posts automatically
  - scrape news and create content with AI
  - automate social media content workflow end-to-end
  - create AI content pipeline with research and rendering
  - build automated marketing content system
  - generate multi-format content from keywords automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete AI-driven content automation system that handles research (news scraping), scriptwriting (Claude/OpenAI), and video generation (Remotion). Transform a single keyword into multi-format content including blog posts, social media content, and rendered videos.

## What It Does

This TypeScript-based Next.js application creates an end-to-end content pipeline:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news (24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# News/Data Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_url_here

# Next.js Config
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # News scraping endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── components/        # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI provider integrations
│   ├── scrapers/         # News scraping modules
│   └── video/            # Remotion video generation
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key API Endpoints

### 1. Research/News Scraping

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();
  
  // Scrape news from multiple sources
  const results = await scrapeNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: timeframe || '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

**Usage:**

```typescript
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI marketing automation',
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  })
});

const { data } = await response.json();
```

### 2. Content Generation

```typescript
// app/api/generate/route.ts
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(req: NextRequest) {
  const { researchData, format, language, tone, provider } = await req.json();
  
  let content;
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: buildPrompt(researchData, format, language, tone)
      }]
    });
    content = message.content[0].text;
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: buildPrompt(researchData, format, language, tone)
      }]
    });
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content });
}

function buildPrompt(data: any, format: string, language: string, tone: string) {
  return `Create a ${format} article in ${language} with a ${tone} tone based on this research data:
  
${JSON.stringify(data, null, 2)}

Requirements:
- Format: ${format}
- Language: ${language}
- Tone: ${tone}
- Include data-backed insights
- Add actionable takeaways`;
}
```

**Usage:**

```typescript
const article = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    researchData: newsData,
    format: 'toplist',
    language: 'en',
    tone: 'professional',
    provider: 'claude'
  })
});

const { content } = await article.json();
```

### 3. Video Rendering

```typescript
// app/api/render/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(req: NextRequest) {
  const { content, template, platform } = await req.json();
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template || 'ContentVideo',
    inputProps: { content, platform },
  });
  
  // Render video
  const outputLocation = path.join(process.cwd(), 'public/videos', `output-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { content, platform },
  });
  
  return NextResponse.json({ 
    videoUrl: `/videos/${path.basename(outputLocation)}` 
  });
}
```

**Usage:**

```typescript
const video = await fetch('/api/render', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    content: articleContent,
    template: 'ContentVideo',
    platform: 'instagram' // or 'tiktok', 'youtube'
  })
});

const { videoUrl } = await video.json();
```

## Content Formats

### Available Formats

```typescript
type ContentFormat = 
  | 'toplist'      // Numbered lists with insights
  | 'pov'          // Point-of-view opinion pieces
  | 'case-study'   // Detailed analysis with examples
  | 'how-to'       // Step-by-step guides
  | 'news-roundup' // Curated news summary
  | 'comparison';  // Side-by-side comparisons

type Language = 'en' | 'vi';

type Tone = 
  | 'professional' 
  | 'friendly' 
  | 'humorous' 
  | 'authoritative'
  | 'casual';
```

### Complete Pipeline Example

```typescript
// lib/pipeline/complete-workflow.ts
import { scrapeNews } from '@/lib/scrapers';
import { generateContent } from '@/lib/ai/generator';
import { renderVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  platforms: ('instagram' | 'tiktok' | 'youtube')[];
  aiProvider: 'claude' | 'openai';
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchData = await scrapeNews({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    data: researchData,
    format: config.format,
    language: config.language,
    tone: config.tone,
    provider: config.aiProvider
  });
  
  // Step 3: Render Videos
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all(
    config.platforms.map(platform => 
      renderVideo({
        content,
        platform,
        template: 'ContentVideo'
      })
    )
  );
  
  return {
    research: researchData,
    article: content,
    videos
  };
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI marketing trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  platforms: ['instagram', 'tiktok'],
  aiProvider: 'claude'
});

console.log('Article:', result.article);
console.log('Videos:', result.videos);
```

## Remotion Video Templates

### Basic Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { z } from 'zod';

export const contentVideoSchema = z.object({
  content: z.object({
    title: z.string(),
    points: z.array(z.string()),
  }),
  platform: z.enum(['instagram', 'tiktok', 'youtube']),
});

export const ContentVideo: React.FC<z.infer<typeof contentVideoSchema>> = ({
  content,
  platform,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const aspectRatios = {
    instagram: '9:16',
    tiktok: '9:16',
    youtube: '16:9',
  };
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <div style={{ 
          color: 'white', 
          fontSize: 48, 
          textAlign: 'center',
          padding: 40 
        }}>
          {content.title}
        </div>
      </Sequence>
      
      {content.points.map((point, i) => (
        <Sequence 
          key={i}
          from={fps * (2 + i * 3)} 
          durationInFrames={fps * 3}
        >
          <div style={{ 
            color: 'white', 
            fontSize: 32, 
            padding: 40 
          }}>
            {i + 1}. {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Register Composition

```typescript
// remotion/index.ts
import { Composition } from 'remotion';
import { ContentVideo, contentVideoSchema } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        schema={contentVideoSchema}
        defaultProps={{
          content: {
            title: 'Top 5 AI Marketing Trends',
            points: [
              'AI-powered personalization',
              'Automated content creation',
              'Predictive analytics',
              'Chatbots and conversational AI',
              'Voice search optimization'
            ]
          },
          platform: 'instagram'
        }}
      />
    </>
  );
};
```

## News Scraping Implementation

```typescript
// lib/scrapers/multi-source.ts
import axios from 'axios';

interface ScrapeConfig {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function scrapeNews(config: ScrapeConfig) {
  const results = await Promise.all(
    config.sources.map(source => scrapeSource(source, config.keyword, config.timeframe))
  );
  
  return results.flat().sort((a, b) => 
    new Date(b.publishedAt).getTime() - new Date(a.publishedAt).getTime()
  );
}

async function scrapeSource(source: string, keyword: string, timeframe: string) {
  const scrapers = {
    techcrunch: scrapeTechCrunch,
    a16z: scrapeA16z,
    twitter: scrapeTwitter,
    linkedin: scrapeLinkedIn,
  };
  
  return scrapers[source]?.(keyword, timeframe) || [];
}

async function scrapeTechCrunch(keyword: string, timeframe: string) {
  const response = await axios.get('https://techcrunch.com/wp-json/tc/v1/magazine', {
    params: { 
      _embed: true,
      per_page: 20,
      search: keyword
    }
  });
  
  return response.data.map((article: any) => ({
    title: article.title.rendered,
    url: article.link,
    excerpt: article.excerpt.rendered,
    publishedAt: article.date,
    source: 'TechCrunch'
  }));
}

// Use RapidAPI for Twitter/LinkedIn if needed
async function scrapeTwitter(keyword: string, timeframe: string) {
  const options = {
    method: 'GET',
    url: 'https://twitter-api45.p.rapidapi.com/search',
    params: { query: keyword, count: '20' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'twitter-api45.p.rapidapi.com'
    }
  };
  
  const response = await axios.request(options);
  return response.data.timeline || [];
}
```

## Common Patterns

### Client Component with Full Pipeline

```typescript
// app/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      // Research
      const research = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
          keyword, 
          sources: ['techcrunch', 'a16z'],
          timeframe: '24h'
        })
      });
      const { data: researchData } = await research.json();
      
      // Generate
      const generate = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          researchData,
          format: 'toplist',
          language: 'en',
          tone: 'professional',
          provider: 'claude'
        })
      });
      const { content } = await generate.json();
      
      // Render
      const render = await fetch('/api/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content,
          template: 'ContentVideo',
          platform: 'instagram'
        })
      });
      const { videoUrl } = await render.json();
      
      setResult({ content, videoUrl });
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
        />
        <button type="submit" disabled={loading}>
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div>
          <h2>Generated Article</h2>
          <div>{result.content}</div>
          <h2>Video</h2>
          <video src={result.videoUrl} controls />
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### API Key Issues

```typescript
// lib/utils/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

// Call in API routes
validateEnv();
```

### Rate Limiting

```typescript
// lib/utils/rate-limit.ts
const rateLimit = new Map<string, number[]>();

export function checkRateLimit(key: string, maxRequests: number, windowMs: number): boolean {
  const now = Date.now();
  const requests = rateLimit.get(key) || [];
  
  const recentRequests = requests.filter(time => now - time < windowMs);
  
  if (recentRequests.length >= maxRequests) {
    return false;
  }
  
  recentRequests.push(now);
  rateLimit.set(key, recentRequests);
  return true;
}
```

### Video Rendering Errors

```bash
# Install ffmpeg (required for Remotion)
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

### Memory Issues with Large Renders

```typescript
// Increase Node memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "render": "NODE_OPTIONS='--max-old-space-size=8192' remotion render"
  }
}
```

## Performance Optimization

```typescript
// lib/cache/redis.ts (optional)
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;
  
  const fresh = await scrapeNews({ keyword, sources: ['techcrunch'], timeframe: '24h' });
  await redis.set(`research:${keyword}`, fresh, { ex: 3600 }); // 1 hour cache
  
  return fresh;
}
```
