---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with Claude and OpenAI
  - create automated content workflow from research to video
  - build AI content generation system with Remotion
  - configure marketing automation pipeline with TypeScript
  - generate videos automatically from written content
  - crawl news sources and create content with AI
  - setup end-to-end content production pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using **marketing-pipeline-share**, an end-to-end AI-powered content automation system that handles research, scriptwriting, content generation, and video rendering. The pipeline crawls news sources (TechCrunch, Twitter, LinkedIn), generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What It Does

The marketing-pipeline-share project is a TypeScript/Next.js application that:

- **Auto-crawls news sources** for fresh data and insights
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using AI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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

### Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── crawlers/    # News crawling logic
│   │   ├── ai/          # AI content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Components and Usage

### 1. Content Research & Crawling

The system crawls multiple sources for fresh content:

```typescript
// src/lib/crawlers/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export async function crawlNewsSources(
  keyword: string,
  sources: NewsSource[]
): Promise<CrawledData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          params: { q: keyword },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          }
        });
        
        return {
          source: source.name,
          data: response.data,
          timestamp: new Date()
        };
      } catch (error) {
        console.error(`Failed to crawl ${source.name}:`, error);
        return null;
      }
    })
  );
  
  return results.filter(Boolean);
}

// Usage
const sources = [
  { name: 'TechCrunch', url: 'https://api.techcrunch.com/search', selector: '.article' },
  { name: 'Twitter', url: 'https://twitter-api.rapidapi.com/search', selector: '.tweet' }
];

const data = await crawlNewsSources('AI marketing', sources);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researched_data: string;
}

export async function generateContentWithClaude(
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `
You are a professional content writer. Generate a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research Data:
${config.researched_data}

Requirements:
- Use data-backed insights
- Include relevant statistics
- Optimize for social media engagement
- Format: ${config.format}
`;

  const message = await anthropic.messages.create({
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

export async function generateContentWithOpenAI(
  config: ContentConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${config.tone} content writer creating ${config.format} articles in ${config.language}.`
      },
      {
        role: 'user',
        content: `Create content based on: ${config.researched_data}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

const DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
};

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  const outputLocation = path.resolve(
    `./output/${config.format}-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    ...DIMENSIONS[config.format],
  });

  return outputLocation;
}
```

### 4. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { crawlNewsSources } from '../crawlers/news-crawler';
import { generateContentWithClaude } from '../ai/content-generator';
import { renderContentVideo } from '../video/render-video';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  videoFormat: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  content: string;
  videoPath: string;
}> {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const sources = [
    { name: 'TechCrunch', url: 'https://api.techcrunch.com/search', selector: '.article' },
    { name: 'Twitter', url: 'https://twitter-api.rapidapi.com/search', selector: '.tweet' }
  ];
  
  const researchData = await crawlNewsSources(config.keyword, sources);
  const researched_data = JSON.stringify(researchData);

  // Step 2: Generate Content
  console.log('✍️ Generating content with AI...');
  const content = await generateContentWithClaude({
    format: config.format,
    language: config.language,
    tone: config.tone,
    researched_data,
  });

  // Step 3: Render Video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    content,
    title: config.keyword,
    format: config.videoFormat,
  });

  console.log('✅ Pipeline complete!');
  return { content, videoPath };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  videoFormat: 'reels',
});
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, videoFormat } = body;

    // Validate inputs
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      videoFormat: videoFormat || 'reels',
    });

    return NextResponse.json({
      success: true,
      content: result.content,
      videoUrl: `/videos/${path.basename(result.videoPath)}`,
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

### Usage from Frontend

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        videoFormat: formData.get('videoFormat'),
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <form action={handleGenerate}>
      <input name="keyword" placeholder="Enter keyword..." required />
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to</option>
      </select>
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Generated Content:</h3>
          <p>{result.content}</p>
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
          )}
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Bilingual Content Generation

```typescript
export async function generateBilingualContent(
  keyword: string,
  format: string
): Promise<{ en: string; vi: string }> {
  const researchData = await crawlNewsSources(keyword, sources);
  const researched_data = JSON.stringify(researchData);

  const [enContent, viContent] = await Promise.all([
    generateContentWithClaude({
      format,
      language: 'en',
      tone: 'expert',
      researched_data,
    }),
    generateContentWithClaude({
      format,
      language: 'vi',
      tone: 'expert',
      researched_data,
    }),
  ]);

  return { en: enContent, vi: viContent };
}
```

### Batch Video Generation

```typescript
export async function batchGenerateVideos(
  contents: Array<{ title: string; content: string }>,
  format: 'reels' | 'tiktok' | 'shorts'
): Promise<string[]> {
  const videoPaths = await Promise.all(
    contents.map((item) =>
      renderContentVideo({
        content: item.content,
        title: item.title,
        format,
      })
    )
  );

  return videoPaths;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Timeout

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Crawls

```typescript
// Process crawled data in chunks
async function processCrawledDataInChunks(
  data: CrawledData[],
  chunkSize = 10
) {
  const results = [];
  for (let i = 0; i < data.length; i += chunkSize) {
    const chunk = data.slice(i, i + chunkSize);
    const processed = await processChunk(chunk);
    results.push(...processed);
  }
  return results;
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering locally
npm run remotion:preview
```

This skill enables AI coding agents to help developers build and customize automated content pipelines with research, AI generation, and video rendering capabilities.
