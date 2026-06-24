---
name: ai-content-pipeline-automation
description: Automate content creation from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i create automated content pipelines with ai
  - help me set up ai content generation workflow
  - how to use this marketing automation system
  - show me how to generate videos from content automatically
  - how do i crawl news and generate articles with ai
  - help me automate content creation with claude and openai
  - how to build an ai-powered content pipeline
  - show me how to use remotion for video generation
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers build and use an automated content pipeline that handles research, scriptwriting, article generation, and video creation using Claude, OpenAI, and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates articles** in multiple formats (toplist, POV, case study, how-to)
- **Creates bilingual content** (English & Vietnamese)
- **Renders videos** and infographics automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, Shorts

Built with TypeScript, Next.js, and integrates Claude 3, OpenAI, and RapidAPI.

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom API endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Remotion configuration
REMOTION_TIMEOUT=120000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion:render
```

## Core API Endpoints

### 1. Research & Crawl News

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeRange } = await req.json();
  
  try {
    const response = await fetch('https://news-api.rapidapi.com/search', {
      method: 'POST',
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        query: keyword,
        sources: sources || ['techcrunch', 'a16z'],
        from: timeRange || '24h'
      })
    });
    
    const data = await response.json();
    return NextResponse.json({ articles: data.results });
  } catch (error) {
    return NextResponse.json({ error: 'Research failed' }, { status: 500 });
  }
}
```

Usage:

```typescript
const research = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI startup funding',
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  })
});

const { articles } = await research.json();
```

### 2. Generate Content with AI

```typescript
// lib/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(
    prompt: string,
    format: 'toplist' | 'pov' | 'case-study' | 'how-to',
    language: 'en' | 'vi'
  ) {
    const systemPrompt = this.getSystemPrompt(format, language);
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompt,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  async generateWithOpenAI(
    prompt: string,
    format: string,
    language: string
  ) {
    const systemPrompt = this.getSystemPrompt(format, language);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: prompt }
      ],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private getSystemPrompt(format: string, language: string): string {
    const prompts = {
      'toplist': {
        'en': 'You are an expert content writer. Create an engaging top list article with data-backed insights.',
        'vi': 'Bạn là chuyên gia viết nội dung. Tạo bài viết dạng top list hấp dẫn với dữ liệu thực tế.'
      },
      'pov': {
        'en': 'You are a thought leader. Write a compelling POV article with unique perspectives.',
        'vi': 'Bạn là người dẫn dắt tư duy. Viết bài POV thuyết phục với góc nhìn độc đáo.'
      },
      'case-study': {
        'en': 'You are a business analyst. Create a detailed case study with actionable insights.',
        'vi': 'Bạn là chuyên gia phân tích. Tạo case study chi tiết với insight thực tế.'
      },
      'how-to': {
        'en': 'You are an instructional expert. Write a step-by-step how-to guide.',
        'vi': 'Bạn là chuyên gia hướng dẫn. Viết bài how-to từng bước chi tiết.'
      }
    };
    
    return prompts[format as keyof typeof prompts][language as 'en' | 'vi'];
  }
}
```

### 3. Generate Bilingual Content

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/content-generator';

export async function POST(req: NextRequest) {
  const { keyword, articles, format, provider } = await req.json();
  
  const generator = new ContentGenerator();
  
  const prompt = `
Based on these recent articles:
${articles.map((a: any) => `- ${a.title}: ${a.summary}`).join('\n')}

Create a ${format} article about "${keyword}".
`;

  try {
    const [englishContent, vietnameseContent] = await Promise.all([
      provider === 'claude' 
        ? generator.generateWithClaude(prompt, format, 'en')
        : generator.generateWithOpenAI(prompt, format, 'en'),
      provider === 'claude'
        ? generator.generateWithClaude(prompt, format, 'vi')
        : generator.generateWithOpenAI(prompt, format, 'vi')
    ]);
    
    return NextResponse.json({
      english: englishContent,
      vietnamese: vietnameseContent
    });
  } catch (error) {
    return NextResponse.json({ error: 'Generation failed' }, { status: 500 });
  }
}
```

## Remotion Video Generation

### Video Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { Composition } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  bgColor: string;
}> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60,
        opacity,
        transform: `translateY(${Math.max(0, 50 - frame)}px)`
      }}>
        <h1 style={{ 
          fontSize: 64, 
          fontWeight: 'bold',
          marginBottom: 40,
          color: 'white'
        }}>
          {title}
        </h1>
        
        {points.map((point, i) => (
          <Sequence key={i} from={fps * (i + 1)} durationInFrames={fps * 2}>
            <div style={{
              fontSize: 32,
              marginBottom: 20,
              color: 'white',
              opacity: Math.min(1, (frame - fps * (i + 1)) / fps)
            }}>
              {i + 1}. {point}
            </div>
          </Sequence>
        ))}
      </div>
    </AbsoluteFill>
  );
};

// Register composition
export const RemotionRoot: React.FC = () => {
  return (
    <Composition
      id="ContentVideo"
      component={ContentVideo}
      durationInFrames={300}
      fps={30}
      width={1080}
      height={1920}
      defaultProps={{
        title: 'Top AI Trends 2024',
        points: ['Trend 1', 'Trend 2', 'Trend 3'],
        bgColor: '#1a1a2e'
      }}
    />
  );
};
```

### Render Video API

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(req: NextRequest) {
  const { title, points, format } = await req.json();
  
  try {
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: { title, points, bgColor: '#1a1a2e' }
    });
    
    const outputPath = path.join(
      process.cwd(), 
      'public/videos',
      `${Date.now()}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: { title, points, bgColor: '#1a1a2e' }
    });
    
    return NextResponse.json({ 
      videoUrl: outputPath.replace(process.cwd() + '/public', '')
    });
  } catch (error) {
    return NextResponse.json({ error: 'Video render failed' }, { status: 500 });
  }
}
```

## Complete Workflow Example

```typescript
// lib/content-pipeline.ts
export class ContentPipeline {
  async execute(keyword: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
    // Step 1: Research
    const researchRes = await fetch('/api/research', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ keyword, timeRange: '24h' })
    });
    const { articles } = await researchRes.json();
    
    // Step 2: Generate content
    const generateRes = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        articles,
        format,
        provider: 'claude'
      })
    });
    const { english, vietnamese } = await generateRes.json();
    
    // Step 3: Extract key points for video
    const points = this.extractKeyPoints(english);
    
    // Step 4: Render video
    const videoRes = await fetch('/api/render-video', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: keyword,
        points: points.slice(0, 5),
        format: '9:16' // TikTok/Reels format
      })
    });
    const { videoUrl } = await videoRes.json();
    
    return {
      articles: { english, vietnamese },
      videoUrl
    };
  }
  
  private extractKeyPoints(content: string): string[] {
    // Simple extraction - could be enhanced with AI
    const lines = content.split('\n').filter(line => 
      line.match(/^(\d+\.|•|-|\*)/) && line.length > 20
    );
    return lines.map(line => line.replace(/^(\d+\.|•|-|\*)\s*/, '').trim());
  }
}
```

## Usage in Client Components

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';
import { ContentPipeline } from '@/lib/content-pipeline';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const pipeline = new ContentPipeline();
      const output = await pipeline.execute(keyword, format);
      setResult(output);
    } catch (error) {
      console.error('Pipeline failed:', error);
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
          className="w-full p-4 border rounded"
        />
        
        <select 
          value={format} 
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-4 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV / Opinion</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-4 bg-blue-600 text-white rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
      </div>
      
      {result && (
        <div className="space-y-6">
          <div className="p-6 border rounded">
            <h2 className="text-2xl font-bold mb-4">English Content</h2>
            <div className="prose" dangerouslySetInnerHTML={{ __html: result.articles.english }} />
          </div>
          
          <div className="p-6 border rounded">
            <h2 className="text-2xl font-bold mb-4">Vietnamese Content</h2>
            <div className="prose" dangerouslySetInnerHTML={{ __html: result.articles.vietnamese }} />
          </div>
          
          {result.videoUrl && (
            <div className="p-6 border rounded">
              <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
              <video src={result.videoUrl} controls className="w-full max-w-md" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
// lib/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;
  
  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }
  
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
const claudeLimiter = new RateLimiter(50); // 50 requests per minute

await claudeLimiter.add(() => generator.generateWithClaude(prompt, format, 'en'));
```

### Caching Research Results

```typescript
// lib/cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached;
}

export async function setCachedResearch(keyword: string, data: any) {
  await redis.set(`research:${keyword}`, JSON.stringify(data), {
    ex: 3600 // 1 hour expiry
  });
}

// In research API
const cached = await getCachedResearch(keyword);
if (cached) return NextResponse.json(cached);

// ... fetch new data ...
await setCachedResearch(keyword, data);
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required env var: ${key}`);
    }
  }
}

validateEnv();
```

### Remotion Rendering Timeout

```typescript
// Increase timeout in remotion.config.ts
export default {
  timeout: 120000, // 2 minutes
  concurrency: 1,
  browser: 'chrome'
};
```

### Memory Issues with Large Crawls

```typescript
// Process articles in batches
async function processBatch<T>(items: T[], batchSize: number, processor: (item: T) => Promise<any>) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(processor));
    results.push(...batchResults);
  }
  
  return results;
}

const articles = await processBatch(allArticles, 10, async (article) => {
  return await summarizeArticle(article);
});
```

This skill enables AI coding agents to help developers build complete content automation pipelines from research through video generation using modern AI APIs and Remotion.
