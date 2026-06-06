---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that automates research, script writing, and video generation for marketing workflows
triggers:
  - automate content creation pipeline
  - generate marketing content with AI
  - scrape news and create videos automatically
  - build AI content automation workflow
  - create marketing videos from research
  - set up automated content generation system
  - use Claude and OpenAI for content pipeline
  - generate multilingual marketing content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** - an end-to-end TypeScript/Next.js system that automates content creation from research to video generation. It combines web scraping, AI content generation (Claude/OpenAI), and video rendering (Remotion) into a single automated workflow.

## What This Project Does

The Marketing Pipeline Share is a comprehensive content automation system that:

- **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn for fresh data
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Output**: Generates content in both English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Required
node >= 18.x
npm or yarn
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token_here

# Optional
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content generation logic
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core API Usage

### 1. Research Module (Web Scraping)

```typescript
// lib/scraper/news-scraper.ts
import { scrapeNewsSources } from '@/lib/scraper/news-scraper';

interface NewsResult {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  summary: string;
}

async function gatherResearch(keyword: string): Promise<NewsResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  const results = await scrapeNewsSources({
    keyword,
    sources,
    timeRange: '24h', // Last 24 hours
    maxResults: 20
  });
  
  return results;
}

// Usage
const research = await gatherResearch('AI automation');
console.log(`Found ${research.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: NewsResult[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const systemPrompt = `You are an expert content writer creating ${request.format} articles in ${request.language} with a ${request.tone} tone.`;
  
  const userPrompt = `
Topic: ${request.keyword}

Research Data:
${request.researchData.map(r => `- ${r.title}: ${r.summary}`).join('\n')}

Create a comprehensive ${request.format} article that:
1. Uses the latest data from research
2. Provides actionable insights
3. Maintains ${request.tone} tone
4. Is optimized for social media sharing
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: userPrompt }
    ],
    system: systemPrompt
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer creating ${request.format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about: ${request.keyword}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reel' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
  duration: number; // in seconds
}

async function renderVideo(config: VideoConfig): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputLocation;
}
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await gatherResearch(keyword);
    
    // Step 2: Generate English Content
    console.log('✍️ Generating English content...');
    const englishContent = await generateContent({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research
    });
    
    // Step 3: Generate Vietnamese Content
    console.log('✍️ Generating Vietnamese content...');
    const vietnameseContent = await generateContent({
      keyword,
      format: 'toplist',
      tone: 'friendly',
      language: 'vi',
      researchData: research
    });
    
    // Step 4: Create Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderVideo({
      title: keyword,
      content: englishContent.substring(0, 500), // First 500 chars
      format: 'reel',
      duration: 30
    });
    
    return {
      research,
      content: {
        en: englishContent,
        vi: vietnameseContent
      },
      video: videoPath,
      status: 'success'
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json(result);
    
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
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
    <div className="max-w-4xl mx-auto p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="w-full p-3 border rounded"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 px-6 py-3 bg-blue-600 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3>English Content</h3>
          <p>{result.content.en}</p>
          
          <h3>Vietnamese Content</h3>
          <p>{result.content.vi}</p>
          
          <video src={result.video} controls />
        </div>
      )}
    </div>
  );
}
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
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Schedule Content Generation

```typescript
// Use node-cron for scheduling
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingTopics();
  await batchGenerate(trendingKeywords);
});
```

### Custom Video Templates

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomTemplate: React.FC<{ title: string; content: string }> = ({
  title,
  content
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5)); // Fade in over 0.5s
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
        <p style={{ color: '#fff', fontSize: 24 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Options

### Content Format Templates

```typescript
// config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    tone: 'personal'
  },
  caseStudy: {
    structure: ['problem', 'solution', 'results', 'lessons'],
    dataRequired: true
  },
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    stepFormat: 'numbered'
  }
};
```

### Scraper Configuration

```typescript
// config/scraper-config.ts
export const scraperConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: '.post-block',
      rateLimit: 1000 // ms between requests
    },
    a16z: {
      url: 'https://a16z.com/posts',
      selector: '.post',
      rateLimit: 1000
    }
  },
  defaultTimeRange: '24h',
  maxResultsPerSource: 10
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Errors

```typescript
// Check Remotion dependencies
// Ensure ffmpeg is installed
import { ensureFfmpeg } from '@remotion/renderer';

async function checkVideoSetup() {
  try {
    await ensureFfmpeg();
    console.log('✅ FFmpeg is available');
  } catch (error) {
    console.error('❌ FFmpeg not found. Install with: brew install ffmpeg');
  }
}
```

### Memory Issues with Large Content

```typescript
// Stream content generation instead of loading all at once
async function generateInChunks(largeContent: string) {
  const chunkSize = 2000; // characters
  const chunks = [];
  
  for (let i = 0; i < largeContent.length; i += chunkSize) {
    const chunk = largeContent.slice(i, i + chunkSize);
    const generated = await generateContent({
      keyword: chunk,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: []
    });
    chunks.push(generated);
  }
  
  return chunks.join('\n\n');
}
```

### Environment Variable Validation

```typescript
// lib/validate-env.ts
export function validateEnvironment() {
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

This skill provides comprehensive coverage for AI coding agents to effectively use and extend the Marketing Pipeline Share automation system for content creation workflows.
