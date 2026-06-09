---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI
triggers:
  - automate content creation pipeline
  - generate content from research to video
  - set up AI content automation
  - create automated marketing content
  - build content pipeline with AI
  - generate videos from articles automatically
  - research and write content with AI
  - automate social media content generation
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (listicles, POV, case studies, how-tos) in multiple languages
3. **Video Rendering**: Automatically generates videos and infographics using Remotion
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
# or
pnpm install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Remotion configuration
REMOTION_TIMEOUT=120000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research crawling endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── page.tsx           # Main UI
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations
│   ├── crawlers/         # Web scraping logic
│   └── remotion/         # Video generation
└── components/           # React components
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
// lib/crawlers/news-scraper.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchQuery {
  keyword: string;
  sources?: string[];
  timeframe?: '24h' | '7d' | '30d';
}

export async function scrapeNewsData(query: ResearchQuery) {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const sources = query.sources || ['techcrunch', 'a16z', 'twitter'];
  const results = await Promise.all(
    sources.map(source => rapidAPI.fetchFromSource(source, query.keyword))
  );
  
  return {
    keyword: query.keyword,
    articles: results.flat(),
    timestamp: new Date().toISOString()
  };
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return {
    content: message.content[0].text,
    format: request.format,
    language: request.language
  };
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with actionable insights',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze a real case with data and outcomes',
    'how-to': 'Create a step-by-step tutorial'
  };
  
  return `
Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Language: ${request.language}
Tone: ${request.tone}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Generate comprehensive content based on the above research data.
`;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/remotion/video-composer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function generateVideo(config: VideoConfig) {
  const aspectRatios = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  };
  
  const dimensions = aspectRatios[config.format];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.tsx'),
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

## Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNewsData } from '@/lib/crawlers/news-scraper';
import { generateContent } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/remotion/video-composer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideoOutput } = await request.json();
    
    // Step 1: Research
    const researchData = await scrapeNewsData({
      keyword,
      timeframe: '24h'
    });
    
    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format,
      language,
      tone,
      researchData: researchData.articles
    });
    
    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (generateVideoOutput) {
      const videoPath = await generateVideo({
        content: content.content,
        title: keyword,
        format: 'reels'
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content: content.content,
        research: researchData.articles.length,
        videoUrl
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const runPipeline = async (formData: FormData) => {
    setLoading(true);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        generateVideoOutput: formData.get('generateVideo') === 'on'
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <form action={runPipeline} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter topic keyword"
          className="w-full p-3 border rounded"
          required
        />
        
        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <select name="language" className="w-full p-3 border rounded">
          <option value="en">English</option>
          <option value="vi">Tiếng Việt</option>
        </select>
        
        <select name="tone" className="w-full p-3 border rounded">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        
        <label className="flex items-center gap-2">
          <input type="checkbox" name="generateVideo" />
          Generate video output
        </label>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-xl font-bold mb-4">Results</h2>
          <pre className="whitespace-pre-wrap">{result.data?.content}</pre>
          {result.data?.videoUrl && (
            <video src={result.data.videoUrl} controls className="mt-4 w-full" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Project

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch-generator.ts
export async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await fetch('/api/pipeline', {
      method: 'POST',
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideoOutput: false
      })
    });
    
    results.push(await content.json());
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ color: 'white', fontSize: 48, textAlign: 'center', padding: 40 }}>
          {title}
        </div>
      </Sequence>
      
      <Sequence from={60} durationInFrames={180}>
        <div style={{ color: 'white', fontSize: 24, padding: 40 }}>
          {content}
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for API calls
- Use caching for research data
- Queue video rendering jobs

### Video Rendering Timeouts
- Increase `REMOTION_TIMEOUT` in environment variables
- Break long content into multiple shorter videos
- Use lower resolution for faster rendering

### Memory Issues
- Process content generation and video rendering separately
- Use streaming responses for large content
- Clear Remotion cache regularly: `npx remotion clear-cache`

### TypeScript Errors
```bash
# Regenerate types
npm run type-check

# Fix common import issues
npm install --save-dev @types/node
```

This skill provides comprehensive coverage of the marketing automation pipeline for AI coding agents to effectively assist developers in implementing automated content creation workflows.
