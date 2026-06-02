---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - automate content creation from research to video
  - generate blog posts with AI research
  - create videos from content automatically
  - use Claude for content generation pipeline
  - crawl news sources and generate content
  - set up automated marketing content system
  - build AI-powered content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete automated content creation system that handles research, content generation, and video rendering. Built with Next.js, TypeScript, integrates Claude 3/OpenAI for content generation, web scraping for research, and Remotion for video rendering.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
2. **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates videos and infographics from content using Remotion
5. **Social Media Optimization**: Exports videos in formats optimized for Reels, TikTok, and Shorts

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

## Configuration

Create a `.env.local` file with the following environment variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key
```

## Core Components

### 1. Research/Crawling Module

```typescript
// services/research/crawler.ts
interface ResearchSource {
  url: string;
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeRange: '24h' | '7d' | '30d';
}

export async function crawlSources(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results: ResearchData[] = [];
  
  for (const source of sources) {
    const data = await fetchSourceData(source, keyword);
    const insights = await extractInsights(data);
    results.push({
      source: source.platform,
      data: insights,
      timestamp: new Date()
    });
  }
  
  return results;
}

// Usage
const research = await crawlSources('AI automation', [
  { url: 'https://techcrunch.com', platform: 'techcrunch', timeRange: '24h' },
  { url: 'https://a16z.com', platform: 'a16z', timeRange: '7d' }
]);
```

### 2. AI Content Generation

```typescript
// services/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  keywords: string[];
}

export async function generateContent(
  researchData: ResearchData[],
  config: ContentConfig
): Promise<GeneratedContent> {
  const prompt = buildPrompt(researchData, config);
  
  // Using Claude for content generation
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
  
  return parseContentResponse(message.content);
}

function buildPrompt(data: ResearchData[], config: ContentConfig): string {
  const insights = data.map(d => d.data).join('\n');
  
  return `
    Create a ${config.format} article based on this research:
    ${insights}
    
    Requirements:
    - Tone: ${config.tone}
    - Language: ${config.language}
    - Keywords to include: ${config.keywords.join(', ')}
    - Include data-backed insights and recent statistics
    - Format with proper headings and structure
  `;
}
```

### 3. Multi-language Content Generation

```typescript
// services/ai/multilingualContent.ts
export async function generateMultilingualContent(
  researchData: ResearchData[],
  config: Omit<ContentConfig, 'language'>
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(researchData, { ...config, language: 'en' }),
    generateContent(researchData, { ...config, language: 'vi' })
  ]);
  
  return {
    en: englishContent.text,
    vi: vietnameseContent.text
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

const formatSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(
  content: GeneratedContent,
  config: VideoConfig
): Promise<string> {
  const specs = formatSpecs[config.format];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: content.text,
      title: content.title,
      highlights: content.highlights
    },
  });
  
  // Render video
  const outputLocation = `output/${config.format}-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: content.text,
      title: content.title,
      highlights: content.highlights
    },
  });
  
  return outputLocation;
}
```

### 5. Complete Pipeline Implementation

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, generateVideo } = await request.json();
    
    // Step 1: Research
    const researchData = await crawlSources(keyword, [
      { url: 'https://techcrunch.com', platform: 'techcrunch', timeRange: '24h' },
      { url: 'https://a16z.com', platform: 'a16z', timeRange: '7d' }
    ]);
    
    // Step 2: Generate content in both languages
    const content = await generateMultilingualContent(researchData, {
      format,
      tone,
      keywords: [keyword]
    });
    
    // Step 3: Render video (optional)
    let videoUrl = null;
    if (generateVideo) {
      videoUrl = await renderContentVideo(
        { text: content.en, title: keyword, highlights: [] },
        { content: content.en, format: 'reels', duration: 60 }
      );
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      researchSources: researchData.length
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

### 6. Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const generateContent = async () => {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        tone: 'expert',
        generateVideo: true
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };
  
  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-4 border rounded-lg"
        />
        
        <button
          onClick={generateContent}
          disabled={loading}
          className="px-6 py-3 bg-blue-600 text-white rounded-lg"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-8 space-y-6">
            <div>
              <h2 className="text-2xl font-bold mb-4">English Version</h2>
              <div className="prose max-w-none">
                {result.content.en}
              </div>
            </div>
            
            <div>
              <h2 className="text-2xl font-bold mb-4">Vietnamese Version</h2>
              <div className="prose max-w-none">
                {result.content.vi}
              </div>
            </div>
            
            {result.videoUrl && (
              <div>
                <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
                <video src={result.videoUrl} controls className="w-full" />
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (if separate script)
npm run render
```

## Common Patterns

### Research Pipeline Pattern

```typescript
// lib/pipelines/research.ts
export class ResearchPipeline {
  private sources: ResearchSource[];
  
  constructor(sources: ResearchSource[]) {
    this.sources = sources;
  }
  
  async execute(keyword: string): Promise<ResearchData[]> {
    const results = await Promise.allSettled(
      this.sources.map(source => 
        this.fetchAndParse(source, keyword)
      )
    );
    
    return results
      .filter(r => r.status === 'fulfilled')
      .map(r => (r as PromiseFulfilledResult<ResearchData>).value);
  }
  
  private async fetchAndParse(
    source: ResearchSource,
    keyword: string
  ): Promise<ResearchData> {
    // Implementation
  }
}
```

### Content Generation Pattern

```typescript
// lib/pipelines/content.ts
export class ContentPipeline {
  private ai: 'claude' | 'openai';
  
  constructor(ai: 'claude' | 'openai' = 'claude') {
    this.ai = ai;
  }
  
  async generate(
    research: ResearchData[],
    config: ContentConfig
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(research, config);
    
    if (this.ai === 'claude') {
      return this.generateWithClaude(prompt);
    } else {
      return this.generateWithOpenAI(prompt);
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Promise<any>[] = [];
  
  async throttle<T>(
    fn: () => Promise<T>,
    delayMs: number = 1000
  ): Promise<T> {
    if (this.queue.length > 0) {
      await Promise.all(this.queue);
    }
    
    const promise = new Promise(resolve => 
      setTimeout(resolve, delayMs)
    );
    this.queue.push(promise);
    
    const result = await fn();
    
    this.queue = this.queue.filter(p => p !== promise);
    return result;
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.throttle(() => 
  generateContent(data, config)
);
```

### Video Rendering Errors

If Remotion rendering fails:
- Ensure sufficient disk space for video output
- Check Remotion license key is valid
- Verify ffmpeg is installed: `ffmpeg -version`
- Use lower resolution for testing

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function processLargeContent(items: string[]): Promise<void> {
  const chunkSize = 10;
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await Promise.all(chunk.map(item => processItem(item)));
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
  }
}
```

### API Key Validation

```typescript
// lib/utils/validateEnv.ts
export function validateEnvironment(): void {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnvironment();
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid repeated crawling
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video processing
3. **Error Handling**: Implement retry logic for API calls
4. **Content Versioning**: Save multiple versions of generated content
5. **Monitor API Usage**: Track API costs and usage limits
