---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for multi-format content production
triggers:
  - how do I automatically generate content with AI research
  - set up marketing content pipeline with video generation
  - automate content creation from keyword to video
  - use Claude and OpenAI for automated content writing
  - generate videos from text content automatically
  - create AI-powered content pipeline with Remotion
  - build automated marketing content system
  - scrape news and generate content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a complete content production system that automates the entire workflow from research to video generation. The system:

- **Auto-crawls** real-time data from sources like TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (Top List, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

Built with Next.js and TypeScript, it provides an end-to-end solution for content creators and marketers.

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- API keys for:
  - Anthropic (Claude)
  - OpenAI
  - RapidAPI (for web scraping)

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Create environment file
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping utilities
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core Components

### 1. Content Research & Scraping

The system automatically scrapes and analyzes recent content from multiple sources.

```typescript
// src/lib/scraper/research.ts
import { RapidAPIClient } from './rapidapi';

interface ResearchOptions {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange: '24h' | '7d' | '30d';
}

export async function conductResearch(options: ResearchOptions) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    options.sources.map(source => 
      client.scrape({
        source,
        keyword: options.keyword,
        timeRange: options.timeRange
      })
    )
  );
  
  return {
    insights: extractInsights(results),
    trends: analyzeTrends(results),
    dataPoints: collectDataPoints(results)
  };
}

function extractInsights(results: any[]) {
  // Analyze and extract key insights from scraped data
  return results.flatMap(r => r.insights || []);
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats and tones.

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(options: ContentOptions): Promise<string> {
    const prompt = this.buildPrompt(options);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  async generateWithOpenAI(options: ContentOptions): Promise<string> {
    const prompt = this.buildPrompt(options);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildPrompt(options: ContentOptions): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article',
      'pov': 'Write from a specific point of view',
      'case-study': 'Analyze a real-world example',
      'how-to': 'Provide step-by-step instructions'
    };
    
    return `
      Create a ${options.language === 'vi' ? 'Vietnamese' : 'English'} article about "${options.keyword}"
      Format: ${formatInstructions[options.format]}
      Tone: ${options.tone}
      
      Use the following research data:
      ${JSON.stringify(options.researchData, null, 2)}
      
      Requirements:
      - Include data-backed insights
      - Reference recent trends (within 24h)
      - Maintain consistent ${options.tone} tone
      - Optimize for engagement
    `.trim();
  }
}
```

### 3. Video Generation with Remotion

Automatically create videos from generated content.

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  content: string;
  title: string;
  format: '16:9' | '9:16' | '1:1';
  duration: number; // seconds
}

export async function renderContentVideo(options: VideoOptions) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: options.content,
      title: options.title
    }
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
    inputProps: {
      content: options.content,
      title: options.title
    }
  });
  
  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = Math.min(1, frame / 20);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: 60,
          color: 'white'
        }}
      >
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>{title}</h1>
        <p style={{ fontSize: 32, lineHeight: 1.6 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

## API Usage Patterns

### Complete Pipeline Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { conductResearch } from '@/lib/scraper/research';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, generateVideo } = await request.json();
    
    // Step 1: Research
    const researchData = await conductResearch({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeRange: '24h'
    });
    
    // Step 2: Generate Content
    const generator = new ContentGenerator();
    const content = await generator.generateWithClaude({
      keyword,
      format,
      tone: 'expert',
      language,
      researchData
    });
    
    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const videoPath = await renderContentVideo({
        content: content.substring(0, 500), // Excerpt for video
        title: keyword,
        format: '16:9',
        duration: 30
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      research: researchData
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

### Frontend Component Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGeneratorForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  async function handleGenerate() {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'en',
          generateVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div className="space-y-4">
      <input
        type="text"
        value={keyword}
        onChange={e => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="w-full px-4 py-2 border rounded"
      />
      
      <select
        value={format}
        onChange={e => setFormat(e.target.value as any)}
        className="w-full px-4 py-2 border rounded"
      >
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to Guide</option>
      </select>
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="px-6 py-2 bg-blue-600 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6 space-y-4">
          <div className="prose max-w-none">
            <h3>Generated Content:</h3>
            <div dangerouslySetInnerHTML={{ __html: result.content }} />
          </div>
          
          {result.videoUrl && (
            <div>
              <h3>Video:</h3>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Customizing Content Formats

```typescript
// src/config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'narrative',
    perspective: 'first-person',
    includePersonalExperience: true
  },
  'case-study': {
    structure: 'problem-solution',
    includeDataPoints: true,
    includeResults: true
  },
  'how-to': {
    structure: 'step-by-step',
    includeImages: true,
    includeExamples: true
  }
};
```

### Video Templates Configuration

```typescript
// remotion/remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2);

export const videoConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  landscape: { width: 1920, height: 1080, fps: 30 },
  square: { width: 1080, height: 1080, fps: 30 }
};
```

## Common Workflows

### Batch Content Generation

```typescript
// src/lib/batch/generator.ts
export async function generateBatchContent(keywords: string[]) {
  const generator = new ContentGenerator();
  
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await conductResearch({
        keyword,
        sources: ['techcrunch', 'twitter'],
        timeRange: '24h'
      });
      
      const content = await generator.generateWithClaude({
        keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        researchData: research
      });
      
      return { keyword, content, research };
    })
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Pipeline

```typescript
// src/lib/scheduler/pipeline.ts
import cron from 'node-cron';

export function scheduleDailyContent(keywords: string[]) {
  // Run daily at 6 AM
  cron.schedule('0 6 * * *', async () => {
    console.log('Starting daily content generation...');
    
    for (const keyword of keywords) {
      try {
        const research = await conductResearch({
          keyword,
          sources: ['techcrunch', 'a16z', 'twitter'],
          timeRange: '24h'
        });
        
        const generator = new ContentGenerator();
        const content = await generator.generateWithClaude({
          keyword,
          format: 'toplist',
          tone: 'expert',
          language: 'en',
          researchData: research
        });
        
        // Save to database or publish
        await saveContent({ keyword, content });
        
      } catch (error) {
        console.error(`Failed to generate content for ${keyword}:`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  private maxConcurrent = 3;
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

const limiter = new RateLimiter();

export async function rateLimitedRequest<T>(fn: () => Promise<T>): Promise<T> {
  return limiter.execute(fn);
}
```

### Error Handling for AI APIs

```typescript
// src/lib/ai/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      lastError = error;
      
      if (error.status === 429) {
        // Rate limit - wait and retry
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      
      if (error.status >= 500) {
        // Server error - retry
        await new Promise(resolve => setTimeout(resolve, 1000));
        continue;
      }
      
      // Client error - don't retry
      throw error;
    }
  }
  
  throw lastError!;
}
```

### Video Rendering Issues

If video rendering fails:

1. Check Remotion license key is set
2. Ensure sufficient disk space
3. Verify ffmpeg is installed
4. Check memory limits

```bash
# Install ffmpeg if missing
# Ubuntu/Debian
sudo apt-get install ffmpeg

# macOS
brew install ffmpeg

# Increase Node.js memory
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

### Research Data Quality

```typescript
// src/lib/scraper/validator.ts
export function validateResearchData(data: any) {
  if (!data || !data.insights || data.insights.length === 0) {
    throw new Error('Insufficient research data');
  }
  
  const recentData = data.insights.filter((item: any) => {
    const age = Date.now() - new Date(item.publishedAt).getTime();
    return age < 24 * 60 * 60 * 1000; // Within 24 hours
  });
  
  if (recentData.length < 3) {
    console.warn('Limited recent data available');
  }
  
  return data;
}
```

## Performance Optimization

### Caching Research Results

```typescript
// src/lib/cache/research-cache.ts
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, any>({
  max: 100,
  ttl: 1000 * 60 * 60 // 1 hour
});

export async function getCachedResearch(
  keyword: string,
  fetcher: () => Promise<any>
) {
  const cached = cache.get(keyword);
  if (cached) return cached;
  
  const data = await fetcher();
  cache.set(keyword, data);
  return data;
}
```

This skill enables AI coding agents to effectively use the Marketing Pipeline AI Content Automation system for building automated content creation workflows with AI-powered research, generation, and video rendering capabilities.
