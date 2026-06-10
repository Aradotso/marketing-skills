---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate marketing content with AI
  - create videos from text automatically
  - build automated content workflow
  - set up AI content research system
  - use marketing pipeline automation
  - generate content from keywords with AI
  - automate social media content creation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, an all-in-one AI-powered content automation system that handles research, scriptwriting, content generation, and video rendering. Built with TypeScript, Next.js, and integrations with Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation pipeline:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (Top lists, POV pieces, Case Studies, How-tos) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # Research crawlers
│   │   └── video/       # Remotion video generation
│   ├── api/             # API routes
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { ResearchService } from '@/lib/research/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Crawl content from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await researchService.crawlMultipleSources({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50,
  });
  
  return results;
}

// Example usage
const data = await gatherResearch('AI marketing automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
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
  
  return message.content[0].text;
}

function buildPrompt(research: any, format: string, language: string) {
  const formatTemplates = {
    toplist: `Create a top 10 list based on this research: ${JSON.stringify(research)}`,
    pov: `Write a point-of-view article based on: ${JSON.stringify(research)}`,
    casestudy: `Create a case study from: ${JSON.stringify(research)}`,
    howto: `Write a how-to guide using: ${JSON.stringify(research)}`,
  };
  
  const languageInstruction = language === 'vi' 
    ? 'Write in Vietnamese with a professional tone.' 
    : 'Write in English with an engaging tone.';
  
  return `${formatTemplates[format]}\n\n${languageInstruction}`;
}
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone. Create engaging marketing content.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any, platform: 'reels' | 'tiktok' | 'shorts') {
  const compositions = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
  };
  
  const config = compositions[platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...config,
    },
  });
  
  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}
```

## Complete Pipeline Example

```typescript
import { ResearchService } from '@/lib/research/research-service';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/video-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'casestudy' | 'howto';
  language: 'en' | 'vi';
  platform: 'reels' | 'tiktok' | 'shorts';
  tone: 'professional' | 'friendly' | 'humorous';
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY,
    });
    
    const researchData = await researchService.crawlMultipleSources({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
      limit: 30,
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const contentGenerator = new ContentGenerator({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
    });
    
    const content = await contentGenerator.generate({
      research: researchData,
      format: config.format,
      language: config.language,
      tone: config.tone,
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoRenderer = new VideoRenderer({
      remotionLicense: process.env.REMOTION_LICENSE_KEY,
    });
    
    const videoPath = await videoRenderer.render({
      content,
      platform: config.platform,
    });
    
    return {
      success: true,
      content,
      videoPath,
      metadata: {
        keyword: config.keyword,
        format: config.format,
        language: config.language,
      },
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI marketing trends 2026',
  format: 'toplist',
  language: 'en',
  platform: 'reels',
  tone: 'professional',
});

console.log('Content generated:', result.content);
console.log('Video saved at:', result.videoPath);
```

## API Routes (Next.js)

### Create Content API

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      platform: body.platform || 'reels',
      tone: body.tone || 'professional',
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/research-service';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();
    
    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY,
    });
    
    const data = await researchService.crawlMultipleSources({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h',
      limit: 50,
    });
    
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

## React Component Example

```typescript
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          platform: 'reels',
          tone: 'professional',
        }),
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
          <pre className="bg-gray-100 p-4 rounded overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        platform: 'reels',
        tone: 'professional',
      })
    )
  );
  
  return results;
}
```

### Multi-Language Support

```typescript
async function generateMultiLanguage(keyword: string) {
  const languages: Array<'en' | 'vi'> = ['en', 'vi'];
  
  const contents = await Promise.all(
    languages.map(language =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language,
        platform: 'reels',
        tone: 'professional',
      })
    )
  );
  
  return {
    en: contents[0],
    vi: contents[1],
  };
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys before running
function validateConfig() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute',
});

async function rateLimitedRequest(fn: () => Promise<any>) {
  await limiter.removeTokens(1);
  return fn();
}
```

### Video Rendering Errors

```typescript
async function safeVideoRender(content: any, platform: string) {
  try {
    return await generateVideo(content, platform);
  } catch (error) {
    console.error('Video rendering failed:', error);
    // Fallback: return static image
    return generateStaticImage(content);
  }
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
```

## Testing the Pipeline

```typescript
// test/pipeline.test.ts
import { runContentPipeline } from '@/lib/pipeline';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const result = await runContentPipeline({
      keyword: 'test keyword',
      format: 'toplist',
      language: 'en',
      platform: 'reels',
      tone: 'professional',
    });
    
    expect(result.success).toBe(true);
    expect(result.content).toBeDefined();
    expect(result.videoPath).toBeDefined();
  });
});
```
