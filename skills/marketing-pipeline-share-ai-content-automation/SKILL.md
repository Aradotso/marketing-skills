---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - set up AI content pipeline with Claude and OpenAI
  - create automated content workflow with Remotion
  - build content automation system with TypeScript
  - integrate AI research and video generation
  - automate social media content creation end-to-end
  - use marketing pipeline for content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use **Marketing Pipeline Share**, an end-to-end AI-powered content automation system. The project automates the entire content creation workflow: from research and data scraping, to AI-generated scripts (in multiple formats and languages), to automated video rendering with Remotion.

## What This Project Does

Marketing Pipeline Share is a complete content automation pipeline that:

- **Auto-scrapes** trending news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Supports multiple languages** (English, Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media (Reels, TikTok, Shorts)
- **Built with Next.js + TypeScript** for a smooth developer experience

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database for storing generated content
DATABASE_URL=your_database_connection_string
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to access the interface.

## Key Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research/scraping endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── page.tsx           # Main UI
├── lib/                   # Core logic
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── scraper/          # Web scraping utilities
│   └── remotion/         # Video generation
├── components/            # React components
└── public/               # Static assets
```

## Core API Patterns

### 1. Research & Data Scraping

```typescript
// lib/scraper/research.ts
import { ScraperConfig } from '@/types';

export async function scrapeNewsData(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchData[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(`https://api.rapidapi.com/${source}/search`, {
        headers: {
          'X-RapidAPI-Key': rapidApiKey!,
          'X-RapidAPI-Host': `${source}.p.rapidapi.com`
        },
        body: JSON.stringify({ query: keyword, timeframe: '24h' })
      });
      
      return response.json();
    })
  );
  
  return processAndDeduplicateResults(results);
}

// Usage in API route
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNewsData } from '@/lib/scraper/research';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  try {
    const data = await scrapeNewsData(keyword, sources);
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 2. AI Content Generation (Claude/OpenAI)

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface GenerateContentParams {
  researchData: ResearchData[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  researchData,
  format,
  language,
  tone,
  provider = 'claude'
}: GenerateContentParams): Promise<GeneratedContent> {
  
  const prompt = buildPrompt(researchData, format, language, tone);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return parseClaudeResponse(message);
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7
    });
    
    return parseOpenAIResponse(completion);
  }
}

function buildPrompt(
  data: ResearchData[],
  format: string,
  language: string,
  tone: string
): string {
  const dataContext = data.map(d => 
    `- ${d.title} (${d.source}): ${d.summary}`
  ).join('\n');
  
  return `
You are a professional content creator. Based on the following research data, create a ${format} article in ${language} with a ${tone} tone.

Research Data:
${dataContext}

Requirements:
- Format: ${format}
- Language: ${language}
- Tone: ${tone}
- Include data-backed insights
- Make it engaging and actionable
- Optimize for social media sharing

Generate the content now:
  `.trim();
}
```

### 3. Video Rendering with Remotion

```typescript
// lib/remotion/video-composer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderContentVideo({
  content,
  platform,
  duration = 30
}: VideoConfig): Promise<string> {
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[platform];
  const fps = 30;
  const durationInFrames = duration * fps;
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: content.text,
      title: content.title,
      highlights: content.keyPoints
    }
  });
  
  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: content.text,
      title: content.title,
      highlights: content.keyPoints
    }
  });
  
  return outputPath;
}
```

### 4. Complete Pipeline Workflow

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNewsData } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/remotion/video-composer';

export async function POST(req: NextRequest) {
  const {
    keyword,
    format = 'toplist',
    language = 'vi',
    tone = 'expert',
    renderVideo = true,
    platform = 'reels'
  } = await req.json();
  
  try {
    // Step 1: Research
    console.log('🔍 Starting research for:', keyword);
    const researchData = await scrapeNewsData(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      researchData,
      format,
      language,
      tone,
      provider: 'claude'
    });
    
    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (renderVideo) {
      console.log('🎬 Rendering video...');
      const videoPath = await renderContentVideo({
        content,
        platform,
        duration: 30
      });
      videoUrl = videoPath.replace(process.cwd() + '/public', '');
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        researchSources: researchData.length
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'vi',
          tone: 'expert',
          renderVideo: true,
          platform: 'reels'
        })
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result?.data && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">
            {result.data.content.title}
          </h2>
          <div className="prose">
            {result.data.content.text}
          </div>
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls className="mt-4" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple content pieces from one research
async function batchGenerate(keyword: string) {
  const researchData = await scrapeNewsData(keyword);
  
  const formats: Array<'toplist' | 'pov' | 'case-study'> = [
    'toplist',
    'pov',
    'case-study'
  ];
  
  const contents = await Promise.all(
    formats.map(format => 
      generateContent({
        researchData,
        format,
        language: 'vi',
        tone: 'expert'
      })
    )
  );
  
  return contents;
}
```

### Multi-language Content

```typescript
// Generate same content in multiple languages
async function multiLanguageContent(researchData: ResearchData[]) {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      researchData,
      format: 'toplist',
      language: 'en',
      tone: 'expert'
    }),
    generateContent({
      researchData,
      format: 'toplist',
      language: 'vi',
      tone: 'expert'
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

## Troubleshooting

### API Key Issues

```typescript
// Always validate API keys before use
function validateApiKeys() {
  const required = {
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
    RAPIDAPI_KEY: process.env.RAPIDAPI_KEY
  };
  
  const missing = Object.entries(required)
    .filter(([_, value]) => !value)
    .map(([key]) => key);
  
  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
// Implement retry logic for rate limits
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

```typescript
// Render videos in smaller chunks
async function renderLongVideo(content: GeneratedContent) {
  const chunks = splitIntoChunks(content, 30); // 30 second chunks
  
  const videos = await Promise.all(
    chunks.map((chunk, i) => 
      renderContentVideo({
        content: chunk,
        platform: 'reels',
        duration: 30
      })
    )
  );
  
  // Optionally concatenate videos
  return concatenateVideos(videos);
}
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start
```

For Vercel deployment, ensure environment variables are set in project settings and consider edge runtime for API routes handling AI requests.
