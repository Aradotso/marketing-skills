---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from articles automatically
  - set up AI content automation system
  - build automated marketing content workflow
  - research and generate content with Claude
  - create social media videos with Remotion
  - automated content pipeline setup
---

# Marketing Pipeline Share AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from research (crawling TechCrunch, a16z, Twitter, LinkedIn), to AI-powered scriptwriting (using Claude 3 or OpenAI), to automatic video generation (via Remotion). It's designed to reduce content creation time by 90% through intelligent automation.

**Key Capabilities:**
- Auto-scan and research current news/trends from multiple sources
- Generate multilingual content (English/Vietnamese) in various formats
- Automatically render videos and infographics from written content
- Optimize output for multiple platforms (Reels, TikTok, Shorts)

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
cp .env.example .env
```

## Configuration

Create a `.env` file in the project root with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Generation)
REMOTION_REGION=us-east-1
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & research
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion compositions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── .env                 # Environment configuration
```

## Core Usage Patterns

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { analyzeContent } from '@/lib/research/analyzer';

// Auto-scan recent news on a topic
async function gatherResearch(keyword: string) {
  const sources = await researchTopic(keyword, {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  // Analyze and extract insights
  const insights = await analyzeContent(sources, {
    extractStats: true,
    identifyTrends: true,
    summarize: true
  });
  
  return insights;
}

// Usage
const aiInsights = await gatherResearch('AI automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  researchData: any,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto',
  language: 'en' | 'vi'
) {
  const prompt = `
Based on this research data: ${JSON.stringify(researchData)}

Create a ${format} article in ${language} that:
- Uses data-backed insights
- Maintains a professional yet engaging tone
- Includes specific examples and statistics
- Is optimized for social media sharing
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}

// Generate bilingual content
const enContent = await generateContent(insights, 'toplist', 'en');
const viContent = await generateContent(insights, 'toplist', 'vi');
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing and technology.`
      },
      {
        role: 'user',
        content: `Write a comprehensive article about ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  articleContent: string,
  aspectRatio: '9:16' | '16:9' | '1:1' = '9:16'
) {
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
      content: articleContent,
      aspectRatio,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `output-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: articleContent,
      aspectRatio,
    },
  });

  return outputLocation;
}

// Generate for multiple platforms
const reelsVideo = await generateVideo(content, '9:16');
const youtubeVideo = await generateVideo(content, '16:9');
```

### 5. Remotion Video Composition Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  content,
  aspectRatio,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30, durationInFrames - 30, durationInFrames],
    [0, 1, 1, 0]
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <div
        style={{
          color: 'white',
          fontSize: 48,
          textAlign: 'center',
          padding: 40,
        }}
      >
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const article = await generateContent(research, 'toplist', 'en');
    const articleVi = await generateContent(research, 'toplist', 'vi');

    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      generateVideo(article, '9:16'),  // Reels/TikTok
      generateVideo(article, '16:9'),  // YouTube
      generateVideo(article, '1:1'),   // Instagram
    ]);

    return {
      articles: {
        en: article,
        vi: articleVi,
      },
      videos,
      success: true,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI Marketing Tools 2024');
```

### 7. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json(result);
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### 8. React Component for UI

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
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
        placeholder="Enter topic keyword..."
        className="w-full p-3 border rounded"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 px-6 py-3 bg-blue-500 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3>Generated Content:</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
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
npm run start

# Render Remotion videos (standalone)
npm run remotion:render
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

### Scheduling Content Generation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  await batchGenerate(trendingTopics);
});
```

## Troubleshooting

**API Rate Limits:**
```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
}
```

**Video Rendering Issues:**
```typescript
// Ensure proper composition registration
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

**Memory Issues with Large Batches:**
```typescript
// Process in chunks
async function processInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
  }
  return results;
}
```
