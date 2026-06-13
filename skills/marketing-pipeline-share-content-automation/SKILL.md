---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using Claude/OpenAI APIs with auto-crawling, multi-format writing, and Remotion video rendering
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and OpenAI
  - create automated content workflow from research to video
  - build AI-powered content automation system
  - generate blog posts and videos automatically from keywords
  - implement content pipeline with auto-crawling and rendering
  - use marketing-pipeline-share for content automation
  - set up Remotion video generation with AI content
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive content automation system that handles the entire content creation workflow: from researching trending topics, generating multi-format articles in multiple languages, to rendering videos automatically using Remotion. Built with TypeScript, Next.js, and integrated with Claude 3 and OpenAI APIs.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Multi-Format Content**: Generates articles in various formats (Toplist, POV, Case Study, How-to) with customizable tone
- **Multilingual**: Automatically creates content in both English and Vietnamese
- **Video Generation**: Renders infographics and short videos from written content using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

Create a `.env.local` file with the following required variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```typescript
// Typical project structure
marketing-pineline-share/
├── src/
│   ├── app/              // Next.js app router
│   ├── components/       // React components
│   ├── lib/
│   │   ├── ai/          // AI integration (Claude, OpenAI)
│   │   ├── crawler/     // Content crawling logic
│   │   ├── generator/   // Content generation
│   │   └── video/       // Remotion video rendering
│   ├── types/           // TypeScript definitions
│   └── utils/           // Helper functions
├── remotion/            // Remotion video templates
└── public/              // Static assets
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { crawlSources } from '@/lib/crawler/sources';

// Crawl recent content from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  const results = await crawlSources({
    keyword,
    sources,
    timeframe: '24h',
    limit: 20
  });
  
  return results;
}

// Example usage
const insights = await researchTopic('AI marketing automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Generate a ${format} article about ${topic} in ${language} with a ${tone} tone.`;
  
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
  
  return message.content[0].text;
}

// Example: Generate a how-to article
const article = await generateContent(
  'AI Content Automation',
  'how-to',
  'en',
  'friendly'
);
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  researchData: any[],
  contentFormat: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and AI.'
      },
      {
        role: 'user',
        content: `Create a ${contentFormat} based on this research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { crawlSources } from '@/lib/crawler/sources';
import { generateContent } from '@/lib/ai/claude';
import { renderVideo } from '@/lib/video/remotion';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await crawlSources({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const articles = await Promise.all(
    config.languages.map(lang =>
      generateContent(
        config.keyword,
        config.format,
        lang,
        'friendly',
        research
      )
    )
  );
  
  // Step 3: Generate video if requested
  let videoUrl = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoUrl = await renderVideo({
      content: articles[0],
      format: 'reels', // or 'tiktok', 'shorts'
      duration: 60
    });
  }
  
  return {
    articles,
    videoUrl,
    research
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true
});
```

### 5. Remotion Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderVideo(contentData: {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: contentData.content,
      format: contentData.format,
    },
  });
  
  const outputLocation = `out/${compositionId}-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: contentData.content,
      format: contentData.format,
    },
  });
  
  return outputLocation;
}
```

### 6. API Route Example (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo } = body;
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en'],
      generateVideo: generateVideo || false
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### 7. Client-Side Component

```typescript
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function handleGenerate(formData: FormData) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          languages: ['en', 'vi'],
          generateVideo: formData.get('video') === 'on'
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <form action={handleGenerate}>
      <input name="keyword" placeholder="Enter keyword..." required />
      <select name="format" required>
        <option value="toplist">Top List</option>
        <option value="how-to">How-to Guide</option>
        <option value="case-study">Case Study</option>
        <option value="pov">Point of View</option>
      </select>
      <label>
        <input type="checkbox" name="video" />
        Generate Video
      </label>
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="results">
          {result.articles.map((article, i) => (
            <div key={i}>{article}</div>
          ))}
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
          )}
        </div>
      )}
    </form>
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
npm start

# Type checking
npm run type-check

# Render Remotion video locally
npx remotion render src/index.ts MyComp out/video.mp4
```

## Common Patterns

### Rate Limiting & Error Handling

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function generateWithRateLimit(prompt: string) {
  await limiter.checkLimit();
  
  try {
    return await generateContent(prompt);
  } catch (error) {
    if (error.status === 429) {
      // Handle rate limit
      await new Promise(resolve => setTimeout(resolve, 5000));
      return generateWithRateLimit(prompt);
    }
    throw error;
  }
}
```

### Caching Research Results

```typescript
import { redis } from '@/lib/cache';

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  // Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Fetch fresh data
  const research = await crawlSources({ keyword });
  
  // Cache for 6 hours
  await redis.setex(cacheKey, 21600, JSON.stringify(research));
  
  return research;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}

if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY is not set');
}
```

### Remotion Rendering Fails

```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu/Debian

# Check Remotion version compatibility
npm list @remotion/bundler @remotion/renderer
```

### Next.js Build Errors

```typescript
// next.config.js - Add webpack config for Remotion
module.exports = {
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
      };
    }
    return config;
  },
};
```

### Memory Issues with Large Crawls

```typescript
// Limit concurrent requests
import pLimit from 'p-limit';

const limit = pLimit(3);

async function crawlMultipleSources(sources: string[]) {
  return Promise.all(
    sources.map(source =>
      limit(() => crawlSource(source))
    )
  );
}
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```
