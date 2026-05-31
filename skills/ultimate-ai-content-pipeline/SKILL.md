---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I use the AI content pipeline to generate articles
  - set up automatic content research and video generation
  - create social media content with AI automation
  - generate multilingual content from keywords using this pipeline
  - build an automated content workflow with Claude and OpenAI
  - render videos from blog posts using Remotion
  - automate research crawling for content creation
  - configure the marketing content pipeline system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What It Does

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms keywords into fully-rendered articles and videos. It combines:

- **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Uses Claude 3 and OpenAI to generate multi-format content (listicles, POV, case studies, how-tos)
- **Multi-language Support**: Generates simultaneous Vietnamese and English versions
- **Video Rendering**: Converts written content into videos/infographics via Remotion
- **Platform Optimization**: Outputs content for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content crawling modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Workflow

### 1. Research Phase - Auto-Crawling

```typescript
// src/lib/research/crawler.ts
import { RapidAPI } from '@/lib/research/rapidapi';

interface ResearchResult {
  title: string;
  url: string;
  snippet: string;
  publishedAt: string;
  source: string;
}

export async function crawlRecentNews(
  keyword: string,
  timeframe: string = '24h'
): Promise<ResearchResult[]> {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await rapidAPI.search({
      query: keyword,
      domain: source,
      timeRange: timeframe
    });
    
    results.push(...data.articles);
  }
  
  return results.sort((a, b) => 
    new Date(b.publishedAt).getTime() - new Date(a.publishedAt).getTime()
  );
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  research: ResearchResult[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }
  
  async generate(params: GenerateContentParams): Promise<string> {
    const prompt = this.buildPrompt(params);
    
    // Use Claude for long-form content
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }],
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  private buildPrompt(params: GenerateContentParams): string {
    const researchContext = params.research
      .map(r => `- ${r.title} (${r.source}): ${r.snippet}`)
      .join('\n');
    
    return `You are a professional content writer.

Keyword: ${params.keyword}
Format: ${params.format}
Language: ${params.language}
Tone: ${params.tone}

Recent Research:
${researchContext}

Create a comprehensive ${params.format} article in ${params.language} language with a ${params.tone} tone. 
Include data-backed insights from the research above. Structure the content with clear headings and sections.`;
  }
}
```

### 3. Dual Language Generation

```typescript
// src/lib/ai/multilingual.ts
export async function generateBilingualContent(
  keyword: string,
  research: ResearchResult[],
  format: ContentFormat,
  tone: Tone
): Promise<{ en: string; vi: string }> {
  const generator = new ContentGenerator();
  
  const [enContent, viContent] = await Promise.all([
    generator.generate({
      keyword,
      research,
      format,
      language: 'en',
      tone
    }),
    generator.generate({
      keyword,
      research,
      format,
      language: 'vi',
      tone
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const { content, title, platform } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: extractKeyPoints(content),
      ...dimensions[platform]
    },
  });
  
  // Render video
  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or headings from content
  const lines = content.split('\n');
  return lines
    .filter(line => line.startsWith('- ') || line.startsWith('## '))
    .map(line => line.replace(/^(- |## )/, ''))
    .slice(0, 5);
}
```

### 5. Complete Pipeline

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlRecentNews } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/multilingual';
import { generateVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const research = await crawlRecentNews(config.keyword);
  
  // Step 2: Generate content in both languages
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(
    config.keyword,
    research,
    config.format,
    config.tone
  );
  
  // Step 3: Generate videos (optional)
  const videos: Record<string, string[]> = { en: [], vi: [] };
  
  if (config.generateVideo && config.platforms) {
    console.log('🎬 Rendering videos...');
    
    for (const platform of config.platforms) {
      const [enVideo, viVideo] = await Promise.all([
        generateVideo({
          content: content.en,
          title: config.keyword,
          platform
        }),
        generateVideo({
          content: content.vi,
          title: config.keyword,
          platform
        })
      ]);
      
      videos.en.push(enVideo);
      videos.vi.push(viVideo);
    }
  }
  
  return {
    research,
    content,
    videos
  };
}
```

## API Routes

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      platforms: body.platforms || ['reels']
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

// Generate a toplist article
const result = await runContentPipeline({
  keyword: 'AI automation tools 2024',
  format: 'toplist',
  tone: 'expert',
  generateVideo: false
});

console.log(result.content.en); // English version
console.log(result.content.vi); // Vietnamese version
```

### Full Pipeline with Video

```typescript
// Generate content + videos for all platforms
const fullResult = await runContentPipeline({
  keyword: 'Marketing automation trends',
  format: 'case-study',
  tone: 'friendly',
  generateVideo: true,
  platforms: ['reels', 'tiktok', 'shorts']
});

console.log('Content:', fullResult.content);
console.log('Videos:', fullResult.videos);
```

### Client-Side Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function handleGenerate(keyword: string) {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        tone: 'expert',
        generateVideo: true,
        platforms: ['reels']
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  }
  
  return (
    <div>
      {/* UI implementation */}
    </div>
  );
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Type checking
npm run type-check

# Linting
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const results = await Promise.all(
  requests.map(req => limit(() => apiCall(req)))
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, maxChars: number = 2000): string[] {
  const chunks: string[] = [];
  const paragraphs = content.split('\n\n');
  
  let currentChunk = '';
  
  for (const para of paragraphs) {
    if ((currentChunk + para).length > maxChars) {
      chunks.push(currentChunk);
      currentChunk = para;
    } else {
      currentChunk += '\n\n' + para;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Error Handling Best Practices

```typescript
// Robust error handling wrapper
export async function safeExecute<T>(
  fn: () => Promise<T>,
  fallback: T
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error('Execution failed:', error);
    return fallback;
  }
}

// Usage
const content = await safeExecute(
  () => generator.generate(params),
  'Failed to generate content'
);
```
