---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline using Claude/OpenAI for research, scriptwriting, video generation with Remotion
triggers:
  - how do I use the marketing pipeline content generator
  - set up automated content creation with AI
  - generate blog posts and videos from keywords automatically
  - configure Claude and OpenAI for content automation
  - create AI-powered content research pipeline
  - render videos from blog content with Remotion
  - automate social media content generation
  - build end-to-end AI content workflow
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms a single keyword into complete content packages including:

- **Automated Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates blog posts in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video/Infographic Rendering**: Automatically creates video content using Remotion
- **Platform Optimization**: Exports video in formats suitable for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Providers (use at least one)
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Workflows

### 1. Content Research Pipeline

```typescript
// src/lib/research/crawler.ts
import { fetchTechCrunchNews } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: Date;
  }>;
  insights: string[];
  keywords: string[];
}

export async function conductResearch(
  keyword: string,
  hours: number = 24
): Promise<ResearchResult> {
  const sources = await Promise.all([
    fetchTechCrunchNews(keyword, hours),
    fetchTwitterTrends(keyword, hours),
    fetchLinkedInPosts(keyword, hours),
  ]);

  const allSources = sources.flat();
  
  return {
    sources: allSources,
    insights: extractInsights(allSources),
    keywords: extractKeywords(allSources),
  };
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchResult;
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
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
      max_tokens: 4096,
    });
    
    return completion.choices[0]?.message?.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with engaging items',
    'pov': 'Write from a specific perspective or viewpoint',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructions',
  };
  
  return `
You are a professional content writer specializing in ${request.format} articles.

Keyword: ${request.keyword}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${request.tone}

Recent Research Data:
${request.researchData.sources.map(s => `- ${s.title}: ${s.summary}`).join('\n')}

Key Insights:
${request.researchData.insights.join('\n')}

Write a comprehensive article that:
1. Uses recent, data-backed information from the research
2. Maintains the specified tone consistently
3. Follows the ${request.format} structure
4. Is optimized for engagement and SEO
5. Includes specific examples and statistics
`.trim();
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '../../remotion/Composition';

interface VideoConfig {
  title: string;
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function renderVideo(config: VideoConfig): Promise<string> {
  const { width, height, fps } = platformDimensions[config.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });
  
  // Render video
  const outputPath = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });
  
  return outputPath;
}
```

### 4. Complete Pipeline Example

```typescript
// src/lib/pipeline/orchestrator.ts
import { conductResearch } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  languages: Language[];
  tone: Tone;
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('Conducting research...');
  const researchData = await conductResearch(config.keyword, 24);
  
  // Step 2: Generate content for each language
  console.log('Generating content...');
  const articles: Record<Language, string> = {} as any;
  
  for (const lang of config.languages) {
    articles[lang] = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language: lang,
      tone: config.tone,
      researchData,
    });
  }
  
  // Step 3: Generate video if requested
  let videoPath: string | null = null;
  if (config.generateVideo && config.platform) {
    console.log('Rendering video...');
    videoPath = await renderVideo({
      title: config.keyword,
      content: articles['en'] || articles['vi'],
      platform: config.platform,
    });
  }
  
  return {
    research: researchData,
    articles,
    videoPath,
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en', 'vi'],
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      platform: body.platform,
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
    }, { status: 500 });
  }
}
```

## Frontend Usage

```typescript
// src/app/page.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          tone: 'friendly',
          generateVideo: true,
          platform: 'reels',
        }),
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
      <h1 className="text-3xl font-bold mb-6">
        AI Content Pipeline
      </h1>
      
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result?.data && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          
          {/* Display articles */}
          {Object.entries(result.data.articles).map(([lang, content]) => (
            <div key={lang} className="mb-6">
              <h3 className="text-xl font-semibold mb-2">
                {lang.toUpperCase()}
              </h3>
              <div className="prose max-w-none">
                {String(content)}
              </div>
            </div>
          ))}
          
          {/* Display video */}
          {result.data.videoPath && (
            <div className="mb-6">
              <h3 className="text-xl font-semibold mb-2">Video</h3>
              <p>Video generated: {result.data.videoPath}</p>
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
npm run start

# Type checking
npm run type-check

# Linting
npm run lint

# Remotion preview (if using Remotion Studio)
npm run remotion:preview
```

## Common Patterns

### Rate Limiting API Calls

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(
    private maxConcurrent: number = 3,
    private delayMs: number = 1000
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    this.running++;
    
    try {
      const result = await fn();
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
      return result;
    } finally {
      this.running--;
    }
  }
}

// Usage
const limiter = new RateLimiter(3, 1000);
const results = await Promise.all(
  keywords.map(kw => limiter.execute(() => generateContent({...})))
);
```

### Caching Research Results

```typescript
// src/lib/research/cache.ts
import { createHash } from 'crypto';

const cache = new Map<string, { data: any; timestamp: number }>();
const CACHE_TTL = 3600000; // 1 hour

export function getCachedResearch(keyword: string): ResearchResult | null {
  const key = createHash('md5').update(keyword).digest('hex');
  const cached = cache.get(key);
  
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data;
  }
  
  return null;
}

export function setCachedResearch(keyword: string, data: ResearchResult) {
  const key = createHash('md5').update(keyword).digest('hex');
  cache.set(key, { data, timestamp: Date.now() });
}
```

## Troubleshooting

### API Key Issues

```typescript
// Check API keys are loaded
if (!process.env.ANTHROPIC_API_KEY && !process.env.OPENAI_API_KEY) {
  throw new Error('At least one AI provider API key must be set');
}
```

### Remotion Rendering Fails

- Ensure `@remotion/bundler` and `@remotion/renderer` versions match
- Check disk space for video output
- Verify composition ID matches in both code and config

### Research Crawler Returns Empty

- Check RapidAPI quota and subscription status
- Verify network access to external APIs
- Implement fallback to cached or sample data for development

### TypeScript Errors

```bash
# Clear Next.js cache
rm -rf .next

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

### Memory Issues During Video Rendering

```typescript
// Adjust Node.js memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev"
  }
}
```
