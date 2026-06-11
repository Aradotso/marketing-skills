---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that auto-researches, scripts, and generates videos from keywords using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate video content from text automatically
  - build an AI content pipeline
  - create multi-format content with Claude and OpenAI
  - scrape trending news and generate articles
  - render videos from content using Remotion
  - set up automated marketing content workflow
  - generate social media content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline crawls real-time data from sources like TechCrunch, Twitter, and LinkedIn, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What It Does

The Marketing Pipeline Share system provides:

- **Auto-Research**: Crawls trending news from major sources (TechCrunch, a16z, Twitter, LinkedIn)
- **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
- **Video Rendering**: Automatically generates videos and infographics from written content using Remotion
- **Multi-Platform Export**: Optimized output for Reels, TikTok, Shorts

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
ANTHROPIC_API_KEY=your_anthropic_key_here

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research endpoints
│   │   ├── generate/      # Content generation
│   │   └── render/        # Video rendering
│   ├── components/        # React components
│   └── lib/              # Utility functions
├── remotion/             # Video templates
│   ├── compositions/     # Video compositions
│   └── assets/          # Media assets
├── services/            # Core services
│   ├── crawler/        # Web scraping
│   ├── ai/            # AI integrations
│   └── renderer/      # Video rendering
└── types/             # TypeScript definitions
```

## Core API Routes

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();
  
  // Crawl from multiple sources
  const researchData = await crawlSources({
    keyword,
    sources: sources || ['techcrunch', 'twitter', 'linkedin'],
    timeRange: timeRange || '24h'
  });
  
  return NextResponse.json({ data: researchData });
}
```

**Usage:**

```typescript
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  })
});

const { data } = await response.json();
```

### Content Generation API

```typescript
// app/api/generate/route.ts
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function POST(request: NextRequest) {
  const { researchData, format, language, tone, provider } = await request.json();
  
  let content;
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: generatePrompt(researchData, format, language, tone)
      }]
    });
    
    content = message.content[0].text;
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: generatePrompt(researchData, format, language, tone)
      }]
    });
    
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content });
}

function generatePrompt(research: any, format: string, language: string, tone: string) {
  return `
    Based on this research data: ${JSON.stringify(research)}
    
    Generate content in the following format: ${format}
    Language: ${language}
    Tone: ${tone}
    
    Include data-backed insights, trending topics, and actionable takeaways.
  `;
}
```

**Usage:**

```typescript
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    researchData: data,
    format: 'toplist',
    language: 'vi',
    tone: 'professional',
    provider: 'claude'
  })
});

const { content } = await response.json();
```

### Video Rendering API

```typescript
// app/api/render/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  const { content, platform, template } = await request.json();
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template || 'SocialVideo',
    inputProps: {
      content,
      platform
    }
  });
  
  // Render video
  const outputLocation = path.join(process.cwd(), 'public', 'videos', `output-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      platform
    }
  });
  
  return NextResponse.json({ 
    videoUrl: `/videos/${path.basename(outputLocation)}` 
  });
}
```

## Service Implementations

### Web Crawler Service

```typescript
// services/crawler/index.ts
import axios from 'axios';

interface CrawlerConfig {
  keyword: string;
  sources: string[];
  timeRange: string;
}

export async function crawlSources(config: CrawlerConfig) {
  const results = await Promise.all(
    config.sources.map(source => crawlSource(source, config.keyword, config.timeRange))
  );
  
  return results.flat().filter(Boolean);
}

async function crawlSource(source: string, keyword: string, timeRange: string) {
  const apiKey = process.env.RAPIDAPI_KEY;
  
  const endpoints = {
    techcrunch: 'https://techcrunch-com.p.rapidapi.com/search',
    twitter: 'https://twitter-api45.p.rapidapi.com/search',
    linkedin: 'https://linkedin-data-api.p.rapidapi.com/search'
  };
  
  try {
    const response = await axios.get(endpoints[source], {
      params: { query: keyword, time: timeRange },
      headers: {
        'X-RapidAPI-Key': apiKey,
        'X-RapidAPI-Host': source + '.p.rapidapi.com'
      }
    });
    
    return response.data.articles || response.data.posts || [];
  } catch (error) {
    console.error(`Failed to crawl ${source}:`, error);
    return [];
  }
}
```

### AI Content Service

```typescript
// services/ai/content-generator.ts
import { Anthropic } from '@anthropic-ai/sdk';

export interface ContentConfig {
  research: any[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

export class ContentGenerator {
  private anthropic: Anthropic;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
  }
  
  async generate(config: ContentConfig): Promise<string> {
    const systemPrompts = {
      toplist: 'You are an expert at creating engaging top 10/top 5 listicles with data-backed insights.',
      pov: 'You are a thought leader sharing unique perspectives on industry trends.',
      'case-study': 'You are a business analyst creating detailed case studies with actionable insights.',
      'how-to': 'You are a tutorial writer creating step-by-step guides.'
    };
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompts[config.format],
      messages: [{
        role: 'user',
        content: this.buildPrompt(config)
      }]
    });
    
    return message.content[0].text;
  }
  
  private buildPrompt(config: ContentConfig): string {
    const researchSummary = config.research
      .map((item, idx) => `${idx + 1}. ${item.title}: ${item.summary}`)
      .join('\n');
    
    return `
      Research Data:
      ${researchSummary}
      
      Create a ${config.format} article in ${config.language === 'vi' ? 'Vietnamese' : 'English'}.
      Tone: ${config.tone}
      
      Requirements:
      - Use data and statistics from the research
      - Include actionable insights
      - Optimize for engagement
      - Add relevant hashtags at the end
    `;
  }
}
```

## Remotion Video Templates

```typescript
// remotion/compositions/SocialVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface SocialVideoProps {
  content: {
    title: string;
    points: string[];
  };
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const SocialVideo: React.FC<SocialVideoProps> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          opacity: titleOpacity,
          color: 'white',
          fontSize: 60,
          fontWeight: 'bold',
          padding: 50,
          textAlign: 'center'
        }}>
          {content.title}
        </div>
      </Sequence>
      
      {content.points.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <div style={{
            color: 'white',
            fontSize: 40,
            padding: 50
          }}>
            {index + 1}. {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { Composition } from 'remotion';
import { SocialVideo } from './compositions/SocialVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="SocialVideo"
        component={SocialVideo}
        durationInFrames={450}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          content: {
            title: 'Sample Title',
            points: ['Point 1', 'Point 2', 'Point 3']
          },
          platform: 'reels'
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

## Common Workflows

### Full Content Pipeline

```typescript
// app/lib/pipeline.ts
import { ContentGenerator } from '@/services/ai/content-generator';
import { crawlSources } from '@/services/crawler';

export async function runFullPipeline(keyword: string) {
  // Step 1: Research
  const research = await crawlSources({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  // Step 2: Generate Content
  const generator = new ContentGenerator();
  const content = await generator.generate({
    research,
    format: 'toplist',
    language: 'vi',
    tone: 'professional'
  });
  
  // Step 3: Render Video
  const videoResponse = await fetch('/api/render', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      content: {
        title: keyword,
        points: extractPoints(content)
      },
      platform: 'reels',
      template: 'SocialVideo'
    })
  });
  
  const { videoUrl } = await videoResponse.json();
  
  return {
    content,
    videoUrl
  };
}

function extractPoints(content: string): string[] {
  // Extract key points from generated content
  const lines = content.split('\n').filter(line => 
    line.match(/^\d+\./) || line.match(/^-/)
  );
  return lines.slice(0, 5);
}
```

### Client Component Example

```typescript
// app/components/ContentCreator.tsx
'use client';

import { useState } from 'react';

export function ContentCreator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3 className="font-bold mb-2">Generated Content:</h3>
          <div className="bg-gray-100 p-4 rounded mb-4">
            {result.content}
          </div>
          
          {result.videoUrl && (
            <video src={result.videoUrl} controls className="w-full" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting errors:

```typescript
// Add retry logic with exponential backoff
async function fetchWithRetry(url: string, options: any, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url, options);
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
}
```

### Remotion Rendering Issues

Ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (use chocolatey)
choco install ffmpeg
```

### Memory Issues with Large Videos

Adjust Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Missing Environment Variables

Validate on startup:

```typescript
// app/lib/config.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

## Performance Optimization

### Caching Research Results

```typescript
// app/lib/cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached;
}

export async function cacheResearch(keyword: string, data: any) {
  await redis.set(`research:${keyword}`, data, { ex: 3600 }); // 1 hour TTL
}
```

### Parallel Processing

```typescript
async function generateMultiLanguageContent(research: any[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generator.generate({ research, format: 'toplist', language: 'en', tone: 'professional' }),
    generator.generate({ research, format: 'toplist', language: 'vi', tone: 'professional' })
  ]);
  
  return { englishContent, vietnameseContent };
}
```
