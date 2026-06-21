---
name: marketing-pipeline-share-ai-content-automation
description: Automated content pipeline using Claude/OpenAI to research, generate multi-format content, and render videos with Remotion
triggers:
  - how do I automate content creation with AI research
  - generate blog posts and videos from keywords automatically
  - use marketing pipeline to create content from trends
  - set up AI content automation with Claude and OpenAI
  - create video content from text using Remotion
  - build automated content research and generation pipeline
  - configure AI content workflow with multi-language support
  - automate content from research to video rendering
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that transforms keywords into published content. It crawls recent news from TechCrunch, a16z, Twitter/X, and LinkedIn, generates multi-format articles in multiple languages using Claude/OpenAI, and automatically renders videos using Remotion.

**Key capabilities:**
- Auto-scan and research trending topics (last 24h)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Multi-language support (English & Vietnamese)
- Automatic video rendering from text content
- Next.js frontend with API integrations

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

Create `.env.local` with the following variables:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if needed)
DATABASE_URL=your_database_url_here

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Utility libraries
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── research/          # Content research crawlers
│   └── video/             # Remotion video generation
├── remotion/              # Video templates
└── public/                # Static assets
```

## Core Usage Patterns

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import { searchTechCrunch, searchTwitter } from './sources';

interface ResearchResult {
  title: string;
  summary: string;
  url: string;
  publishedAt: Date;
  source: string;
}

export async function researchTopic(
  keyword: string,
  timeRange: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = await Promise.all([
    searchTechCrunch(keyword, timeRange),
    searchTwitter(keyword, timeRange),
    // Add more sources as needed
  ]);
  
  return sources.flat().sort(
    (a, b) => b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

// Usage
const results = await researchTopic('AI automation', '24h');
console.log(`Found ${results.length} recent articles`);
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

export async function generateContent(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(topic, options);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(topic: string, options: ContentOptions): string {
  const researchContext = options.research
    .map(r => `- ${r.title} (${r.source}): ${r.summary}`)
    .join('\n');
    
  return `Create a ${options.format} article about "${topic}" in ${options.language}.
Tone: ${options.tone}

Recent research:
${researchContext}

Generate comprehensive, data-backed content with clear structure.`;
}
```

### 3. OpenAI Alternative Generator

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(topic, options);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in tech and marketing.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  highlights: string[];
  backgroundColor: string;
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });
  
  return outputPath;
}
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  highlights: string[];
  backgroundColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  highlights,
  backgroundColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor,
        justifyContent: 'center',
        alignItems: 'center',
        fontSize: 60,
        color: 'white',
        padding: 60,
      }}
    >
      <div style={{ opacity, textAlign: 'center' }}>
        <h1>{title}</h1>
        <ul style={{ fontSize: 40, marginTop: 40 }}>
          {highlights.map((highlight, i) => {
            const itemFrame = frame - (fps * (i + 1));
            const itemOpacity = Math.min(1, Math.max(0, itemFrame / fps));
            return (
              <li key={i} style={{ opacity: itemOpacity, marginBottom: 20 }}>
                {highlight}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline API Route

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();
    
    // Step 1: Research
    console.log('Researching topic...');
    const research = await researchTopic(keyword, '24h');
    
    // Step 2: Generate content
    console.log('Generating content...');
    const content = await generateContent(keyword, {
      format,
      language,
      tone,
      research,
    });
    
    // Step 3: Extract highlights for video
    const highlights = extractHighlights(content);
    
    // Step 4: Render video
    console.log('Rendering video...');
    const videoPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}.mp4`
    );
    
    await renderContentVideo(
      {
        title: keyword,
        highlights,
        backgroundColor: '#1a1a2e',
      },
      videoPath
    );
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl: `/videos/${path.basename(videoPath)}`,
      researchCount: research.length,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}

function extractHighlights(content: string): string[] {
  // Simple extraction - improve based on your content structure
  const lines = content.split('\n').filter(line => 
    line.trim().startsWith('-') || line.trim().startsWith('•')
  );
  return lines.slice(0, 5).map(line => line.replace(/^[-•]\s*/, ''));
}
```

### 7. Frontend Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
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
        className="border p-2 w-full mb-4"
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
          <h3 className="font-bold">Generated Content:</h3>
          <div className="whitespace-pre-wrap">{result.content}</div>
          {result.videoUrl && (
            <video controls className="mt-4 w-full max-w-2xl">
              <source src={result.videoUrl} type="video/mp4" />
            </video>
          )}
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
npm start

# Render video only
npm run remotion:render
```

## Common Patterns

### Multi-language Content Generation

```typescript
export async function generateMultiLanguage(
  topic: string,
  research: ResearchResult[]
): Promise<{ en: string; vi: string }> {
  const [enContent, viContent] = await Promise.all([
    generateContent(topic, {
      format: 'toplist',
      language: 'en',
      tone: 'professional',
      research,
    }),
    generateContent(topic, {
      format: 'toplist',
      language: 'vi',
      tone: 'professional',
      research,
    }),
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### Batch Content Generation

```typescript
export async function generateBatchContent(
  keywords: string[]
): Promise<Map<string, string>> {
  const results = new Map<string, string>();
  
  for (const keyword of keywords) {
    const research = await researchTopic(keyword, '24h');
    const content = await generateContent(keyword, {
      format: 'toplist',
      language: 'en',
      tone: 'professional',
      research,
    });
    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private lastCall = 0;
  private minInterval: number;
  
  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }
  
  async wait(): Promise<void> {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

// Usage
const claudeLimiter = new RateLimiter(50); // 50 calls/minute
await claudeLimiter.wait();
const content = await generateContent(topic, options);
```

### Video Rendering Issues

If Remotion fails to render:

```typescript
// Check Remotion configuration
import { Config } from '@remotion/cli/config';

Config.setOverwriteOutput(true);
Config.setCodec('h264');
Config.setConcurrency(2); // Reduce if memory issues

// For large videos, use cloud rendering
import { renderMediaOnLambda } from '@remotion/lambda';
```

### Memory Management for Large Research

```typescript
// Limit concurrent requests
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function researchMultipleTopics(
  topics: string[]
): Promise<Map<string, ResearchResult[]>> {
  const results = new Map();
  
  await Promise.all(
    topics.map(topic =>
      limit(async () => {
        const research = await researchTopic(topic, '24h');
        results.set(topic, research);
      })
    )
  );
  
  return results;
}
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use environment-specific configs** for development vs production
3. **Implement retry logic** for API failures
4. **Monitor AI token usage** to control costs
5. **Pre-render video templates** for faster generation
6. **Validate input keywords** before starting the pipeline
7. **Log each pipeline step** for debugging

This skill enables AI agents to help developers implement automated content pipelines with research, AI generation, and video rendering capabilities.
