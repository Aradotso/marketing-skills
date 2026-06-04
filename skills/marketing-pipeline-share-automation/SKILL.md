---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with this pipeline
  - set up the marketing content automation system
  - generate video content automatically with AI
  - create automated social media posts with research
  - use the content pipeline for multi-language posts
  - configure the AI content generation workflow
  - crawl news sources and generate content automatically
  - render videos from AI-generated content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the **Ultimate AI Content Pipeline** (marketing-pipeline-share), an end-to-end content automation system that performs research, generates scripts, and creates videos automatically using Claude 3, OpenAI, and Remotion.

## What This Project Does

The marketing-pipeline-share automates the entire content creation workflow:

1. **Auto-Scan Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-Language Output**: Generates parallel English and Vietnamese content
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
# Ensure Node.js 18+ is installed
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

### Install Dependencies

```typescript
// Install all required packages
npm install

// Or using yarn
yarn install

// Or using pnpm
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion License (if applicable)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Database (if used)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Key Architecture

The system is built on Next.js with the following structure:

```
/app                 # Next.js app directory
  /api              # API routes
    /research       # News crawling endpoints
    /generate       # Content generation endpoints
    /render         # Video rendering endpoints
/components         # React components
/lib                # Core utilities
  /ai              # AI integration (Claude, OpenAI)
  /crawler         # Web scraping logic
  /video           # Remotion video generation
/public            # Static assets
/remotion          # Remotion compositions
```

## Core API Usage

### 1. Research & Crawl News

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const crawlResults = await fetch('https://rapidapi.com/...', {
    method: 'POST',
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': 'news-api.rapidapi.com',
    },
    body: JSON.stringify({
      query: keyword,
      sources: sources || ['techcrunch', 'a16z', 'twitter'],
      from: timeframe || '24h',
    }),
  });
  
  const data = await crawlResults.json();
  
  return NextResponse.json({
    articles: data.articles,
    insights: extractInsights(data),
  });
}

function extractInsights(data: any) {
  // Extract key data points, trends, and statistics
  return {
    trends: data.articles.map((a: any) => a.title),
    statistics: [],
    keyQuotes: [],
  };
}
```

### 2. Generate Content with AI

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: any;
}

export async function generateContent(request: ContentRequest) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const prompt = buildPrompt(request);
  
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
  
  return {
    content: message.content[0].text,
    metadata: {
      model: 'claude-3.5-sonnet',
      format: request.format,
      language: request.language,
    },
  };
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with at least 5 items',
    'pov': 'Write a personal perspective opinion piece',
    'case-study': 'Analyze with data, problem, solution, results structure',
    'how-to': 'Create step-by-step tutorial with actionable advice',
  };
  
  return `
You are a professional content writer. Create ${request.format} content about "${request.keyword}".

Tone: ${request.tone}
Language: ${request.language === 'both' ? 'English and Vietnamese (separate sections)' : request.language}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Instructions:
${formatInstructions[request.format]}

- Use data from research to back claims
- Include statistics and quotes
- Make it engaging and actionable
- Optimize for social media sharing
${request.language === 'both' ? '- Provide both English and Vietnamese versions' : ''}
  `.trim();
}
```

### 3. Alternative: OpenAI Integration

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(
  keyword: string,
  researchData: any
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media.',
      },
      {
        role: 'user',
        content: `Create engaging content about ${keyword} using this research: ${JSON.stringify(researchData)}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });
  
  return completion.choices[0].message.content || '';
}
```

## Video Generation with Remotion

### Setup Remotion Composition

```typescript
// remotion/Composition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: 'white',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${0.8 + opacity * 0.2})`,
        }}
      >
        <h1
          style={{
            color: brandColor,
            fontSize: 60,
            marginBottom: 40,
            textAlign: 'center',
          }}
        >
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointFrame = frame - (index + 1) * 30;
          const pointOpacity = Math.max(0, Math.min(1, pointFrame / 20));
          
          return (
            <div
              key={index}
              style={{
                opacity: pointOpacity,
                marginBottom: 20,
                fontSize: 24,
                transform: `translateX(${Math.max(0, 50 - pointFrame)}px)`,
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

### Register Composition

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { Composition } from 'remotion';
import { ContentVideo } from './Composition';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
        defaultProps={{
          title: 'Top 5 AI Trends',
          points: ['Trend 1', 'Trend 2', 'Trend 3', 'Trend 4', 'Trend 5'],
          brandColor: '#FF6B6B',
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### Render Video API

```typescript
// app/api/render/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { compositionId, inputProps } = await request.json();
  
  try {
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });
    
    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps,
    });
    
    // Render video
    const outputLocation = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps,
    });
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(outputLocation)}`,
    });
  } catch (error) {
    console.error('Render error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Complete Workflow Example

```typescript
// lib/pipeline/content-pipeline.ts
interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchResponse = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
    }),
  });
  const researchData = await researchResponse.json();
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const contentResponse = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      researchData: researchData.insights,
    }),
  });
  const content = await contentResponse.json();
  
  // Step 3: Generate Video (if requested)
  let videoUrl = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const videoResponse = await fetch('/api/render', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        compositionId: 'ContentVideo',
        inputProps: {
          title: config.keyword,
          points: extractKeyPoints(content.content),
          brandColor: '#FF6B6B',
        },
      }),
    });
    const videoData = await videoResponse.json();
    videoUrl = videoData.videoUrl;
  }
  
  return {
    research: researchData,
    content: content.content,
    videoUrl,
    timestamp: new Date().toISOString(),
  };
}

function extractKeyPoints(content: string): string[] {
  // Extract numbered points or main ideas from content
  const lines = content.split('\n');
  return lines
    .filter(line => /^\d+\./.test(line.trim()))
    .map(line => line.replace(/^\d+\.\s*/, ''))
    .slice(0, 5);
}
```

## Frontend Component Example

```typescript
// components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'both',
          generateVideo: true,
        }),
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">
            Keyword / Topic
          </label>
          <input
            type="text"
            value={keyword}
            onChange={(e) => setKeyword(e.target.value)}
            className="w-full px-4 py-2 border rounded"
            placeholder="e.g., AI in Marketing 2024"
            required
          />
        </div>
        
        <div>
          <label className="block text-sm font-medium mb-2">
            Content Format
          </label>
          <select
            value={format}
            onChange={(e) => setFormat(e.target.value as any)}
            className="w-full px-4 py-2 border rounded"
          >
            <option value="toplist">Top List</option>
            <option value="pov">POV / Opinion</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white py-3 rounded font-medium hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Start Pipeline 🚀'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 space-y-4">
          <div className="border rounded p-4">
            <h3 className="font-bold mb-2">Generated Content</h3>
            <pre className="whitespace-pre-wrap text-sm">{result.content}</pre>
          </div>
          
          {result.videoUrl && (
            <div className="border rounded p-4">
              <h3 className="font-bold mb-2">Video</h3>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## CLI Commands

```bash
# Development
npm run dev              # Start dev server
npm run build           # Build for production
npm run start           # Start production server

# Remotion specific
npm run remotion        # Open Remotion Studio
npm run render          # Render all compositions

# Type checking
npm run type-check      # Run TypeScript compiler

# Linting
npm run lint            # Run ESLint
```

## Configuration Options

### Customize AI Models

```typescript
// lib/config/ai-models.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2000,
    temperature: 0.7,
  },
};
```

### Video Export Presets

```typescript
// lib/config/video-presets.ts
export const VIDEO_PRESETS = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264',
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30,
    codec: 'h264',
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264',
  },
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'both',
        generateVideo: true,
      })
    )
  );
  
  return results;
}
```

### Schedule Publishing

```typescript
// lib/scheduler/publish-scheduler.ts
interface ScheduledPost {
  content: string;
  publishAt: Date;
  platforms: ('facebook' | 'instagram' | 'twitter')[];
}

export async function schedulePost(post: ScheduledPost) {
  // Store in database with scheduled time
  // Use cron job or queue system to publish
  console.log(`Scheduled post for ${post.publishAt}`);
}
```

## Troubleshooting

### API Rate Limits
```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Timeout
```bash
# Increase timeout in render config
# remotion.config.ts
export default {
  timeout: 300000, // 5 minutes
};
```

### Memory Issues with Large Videos
```typescript
// Use streaming for large renders
import { renderFrames } from '@remotion/renderer';

// Render frame by frame instead of full video at once
```

This skill provides comprehensive guidance for using the marketing-pipeline-share automation system to create AI-powered content workflows from research through video generation.
