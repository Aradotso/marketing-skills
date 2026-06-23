---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline with video generation
  - create automated marketing content workflow
  - generate videos from articles automatically
  - build content automation system with Claude
  - scrape news and generate content with AI
  - automate social media content with Remotion
  - create AI powered content research system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a comprehensive content automation system that handles research, script writing, article generation, and video creation using Claude 3, OpenAI, and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scrapes research**: Crawls and analyzes real-time data from TechCrunch, a16z, X (Twitter), LinkedIn
- **Generates multi-format content**: Creates articles in various formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multiple languages**: Generates content in both English and Vietnamese
- **Renders videos automatically**: Uses Remotion to convert written content into infographics and short videos
- **Optimizes for platforms**: Exports videos in aspect ratios suitable for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for Research/Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities and API integrations
│   ├── ai/                # AI provider integrations (Claude, OpenAI)
│   ├── scraper/           # Web scraping utilities
│   └── video/             # Remotion video rendering
├── remotion/              # Remotion video templates
├── public/                # Static assets
└── types/                 # TypeScript type definitions
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
// lib/scraper/research.ts
import { scrapeNews } from '@/lib/scraper/news-scraper';

interface ResearchResult {
  title: string;
  summary: string;
  source: string;
  url: string;
  publishedAt: string;
  insights: string[];
}

async function conductResearch(keyword: string): Promise<ResearchResult[]> {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      scrapeNews({
        keyword,
        source,
        timeframe: '24h',
        apiKey: process.env.RAPIDAPI_KEY
      })
    )
  );
  
  return results.flat();
}

// Usage
const research = await conductResearch('AI automation');
console.log(`Found ${research.length} recent articles`);
```

### 2. Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

async function generateContent(options: ContentOptions): Promise<string> {
  const { keyword, format, language, tone, research } = options;
  
  const researchContext = research
    .map(r => `- ${r.title}: ${r.summary} (${r.source})`)
    .join('\n');
  
  const prompt = `You are an expert content creator. Generate a ${format} article about "${keyword}" in ${language}.
  
Tone: ${tone}
Recent research data:
${researchContext}

Requirements:
- Include data-backed insights
- Reference recent trends (last 24h)
- Make it engaging and actionable
- Optimize for SEO
${language === 'vi' ? '- Write in Vietnamese' : '- Write in English'}`;

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

// Usage
const article = await generateContent({
  keyword: 'AI automation',
  format: 'how-to',
  language: 'en',
  tone: 'expert',
  research: researchData
});
```

### 3. Content Generation with OpenAI

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(options: ContentOptions): Promise<string> {
  const { keyword, format, language, tone, research } = options;
  
  const researchContext = research
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert ${tone} content writer specializing in ${format} articles.`
      },
      {
        role: 'user',
        content: `Write a ${format} article about "${keyword}" in ${language}.
        
Research context:
${researchContext}

Make it engaging, data-driven, and SEO-optimized.`
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
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

async function renderVideo(config: VideoConfig): Promise<string> {
  const { title, keyPoints, duration, platform } = config;
  const { width, height } = platformDimensions[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      keyPoints,
      duration
    },
  });
  
  // Output path
  const outputPath = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      keyPoints,
      duration
    },
  });
  
  return outputPath;
}

// Usage
const videoPath = await renderVideo({
  title: 'Top 5 AI Automation Tools',
  keyPoints: [
    'Claude for content generation',
    'Remotion for video automation',
    'RapidAPI for data scraping'
  ],
  duration: 30,
  platform: 'reels'
});
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, spring } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const titleScale = spring({
    frame,
    fps,
    from: 0.8,
    to: 1,
    config: { damping: 200 }
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif'
      }}
    >
      <div
        style={{
          opacity: titleOpacity,
          transform: `scale(${titleScale})`,
          color: 'white',
          fontSize: 60,
          fontWeight: 'bold',
          textAlign: 'center',
          padding: 40,
          maxWidth: 900
        }}
      >
        {title}
      </div>
      
      <div style={{ marginTop: 60 }}>
        {keyPoints.map((point, index) => {
          const pointFrame = 60 + index * 90;
          const pointOpacity = interpolate(
            frame,
            [pointFrame, pointFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div
              key={index}
              style={{
                opacity: pointOpacity,
                color: '#00d9ff',
                fontSize: 32,
                margin: '20px 0',
                padding: '0 40px'
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Content Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
interface PipelineResult {
  article: string;
  videoPath: string;
  metadata: {
    keyword: string;
    format: string;
    language: string;
    researchSources: number;
  };
}

async function runContentPipeline(
  keyword: string,
  options: Partial<ContentOptions>
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Conducting research...');
  const research = await conductResearch(keyword);
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const article = await generateContent({
    keyword,
    format: options.format || 'how-to',
    language: options.language || 'en',
    tone: options.tone || 'expert',
    research
  });
  
  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(article);
  
  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderVideo({
    title: `${keyword}: Key Insights`,
    keyPoints,
    duration: 30,
    platform: 'reels'
  });
  
  return {
    article,
    videoPath,
    metadata: {
      keyword,
      format: options.format || 'how-to',
      language: options.language || 'en',
      researchSources: research.length
    }
  };
}

function extractKeyPoints(article: string): string[] {
  // Simple extraction - in production, use AI to extract key points
  const lines = article.split('\n').filter(line => line.trim());
  return lines
    .filter(line => /^\d+\./.test(line) || line.startsWith('-'))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.\s*|-\s*/, '').trim());
}

// Usage
const result = await runContentPipeline('AI automation tools', {
  format: 'toplist',
  language: 'en',
  tone: 'friendly'
});

console.log(`Article generated: ${result.article.length} characters`);
console.log(`Video saved to: ${result.videoPath}`);
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword, {
      format,
      language,
      tone
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  async function generateContent() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'how-to',
          language: 'en',
          tone: 'expert'
        })
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">
        AI Content Pipeline
      </h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
        
        <button
          onClick={generateContent}
          disabled={loading || !keyword}
          className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-8 space-y-4">
            <div className="border p-6 rounded">
              <h2 className="text-xl font-bold mb-4">Article</h2>
              <div className="prose max-w-none">
                {result.article}
              </div>
            </div>
            
            <div className="border p-6 rounded">
              <h2 className="text-xl font-bold mb-4">Video</h2>
              <video controls className="w-full">
                <source src={result.videoPath} type="video/mp4" />
              </video>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword, {
        format: 'toplist',
        language: 'en',
        tone: 'friendly'
      })
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic, {
      format: 'pov',
      language: 'vi',
      tone: 'expert'
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute'
});

async function safeApiCall<T>(fn: () => Promise<T>): Promise<T> {
  await limiter.removeTokens(1);
  return fn();
}
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  inputProps
});
```

### Memory Management for Large Batches

```typescript
// Process in chunks
async function processBatch(keywords: string[], chunkSize = 5) {
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    await Promise.all(chunk.map(k => runContentPipeline(k, {})));
    
    // Allow garbage collection
    if (global.gc) global.gc();
  }
}
```

### Claude API Error Handling

```typescript
async function generateWithRetry(options: ContentOptions, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(options);
    } catch (error: any) {
      if (error.status === 529 && i < maxRetries - 1) {
        // Overloaded, wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```
