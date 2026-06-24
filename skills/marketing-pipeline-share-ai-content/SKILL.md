---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline that researches, generates scripts, and creates videos using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI marketing content pipeline
  - generate blog posts and videos automatically
  - use Claude and OpenAI for content automation
  - create automated content research and video generation
  - build a content pipeline with AI and Remotion
  - automate social media content from research to video
  - set up marketing content automation workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive automation system that handles the entire content creation workflow: from researching trending topics, generating multi-format articles, to rendering videos automatically. Built with Next.js, TypeScript, and integrates Claude 3, OpenAI, and Remotion for video generation.

## What It Does

- **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
- **Multi-Format Content**: Generates articles in various formats (Top Lists, POV, Case Studies, How-tos)
- **Bilingual Output**: Creates content in both English and Vietnamese
- **Automatic Video Generation**: Converts written content to videos using Remotion
- **Voice Customization**: Adjusts tone for different audiences (expert, friendly, humorous)
- **Platform Optimization**: Renders videos for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file with the following environment variables:

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video generation
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```typescript
// Typical structure
/src
  /app                 // Next.js app directory
  /components          // React components
  /lib
    /ai               // AI integration (Claude, OpenAI)
    /research         // Content research/crawling
    /video            // Remotion video generation
  /types              // TypeScript types
  /utils              // Helper functions
```

## Core Usage Patterns

### 1. Research & Content Scraping

```typescript
// lib/research/scraper.ts
import { fetchNewsArticles } from '@/lib/research/sources';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

async function conductResearch(config: ResearchConfig) {
  const articles = await fetchNewsArticles({
    query: config.keyword,
    sources: config.sources,
    since: config.timeRange,
  });

  const insights = articles.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.excerpt,
    publishedAt: article.publishedAt,
  }));

  return insights;
}

// Usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  language: 'en',
});
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
// lib/ai/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData?: any[];
}

async function generateContentWithClaude(request: ContentRequest) {
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

  return message.content[0].text;
}

async function generateContentWithOpenAI(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with clear rankings',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  const researchContext = request.researchData
    ? `\n\nRecent research data:\n${JSON.stringify(request.researchData, null, 2)}`
    : '';

  return `
Create a ${request.format} article about "${request.topic}" in ${request.language}.
Tone: ${request.tone}
${formatInstructions[request.format]}
${researchContext}

Requirements:
- Use recent data and trends from research
- Include specific examples and statistics
- Make it actionable and engaging
- Optimize for social media sharing
`;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'youtube-short';
  style?: 'minimal' | 'dynamic' | 'professional';
}

const VIDEO_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  'youtube-short': { width: 1080, height: 1920 },
};

async function generateVideo(config: VideoConfig) {
  const { width, height } = VIDEO_DIMENSIONS[config.format];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/video/index.tsx'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style || 'minimal',
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style,
    },
  });

  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// src/video/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const contentParts = content.split('\n\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1
            style={{
              fontSize: 80,
              fontWeight: 'bold',
              color: '#fff',
              textAlign: 'center',
              padding: '0 100px',
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {contentParts.map((part, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 100px',
            }}
          >
            <p
              style={{
                fontSize: 48,
                color: '#fff',
                lineHeight: 1.5,
                textAlign: 'center',
              }}
            >
              {part}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline API Route

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { conductResearch } from '@/lib/research/scraper';
import { generateContentWithClaude } from '@/lib/ai/generator';
import { generateVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideoOutput } = body;

    // Step 1: Research
    console.log('Starting research...');
    const researchData = await conductResearch({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      language,
    });

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContentWithClaude({
      topic: keyword,
      format,
      tone: 'expert',
      language,
      researchData,
    });

    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (generateVideoOutput) {
      console.log('Generating video...');
      const videoPath = await generateVideo({
        content,
        title: keyword,
        format: 'reels',
        style: 'dynamic',
      });
      videoUrl = videoPath.replace(process.cwd() + '/public', '');
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        research: researchData,
      },
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

### 6. Frontend Integration

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const generateContent = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'en',
          generateVideoOutput: true,
        }),
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword or topic"
          className="w-full p-3 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <button
          onClick={generateContent}
          disabled={loading || !keyword}
          className="w-full p-3 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div className="p-6 bg-gray-50 rounded">
            <h2 className="text-xl font-bold mb-4">Generated Content</h2>
            <div className="whitespace-pre-wrap">{result.content}</div>
          </div>

          {result.videoUrl && (
            <div className="p-6 bg-gray-50 rounded">
              <h2 className="text-xl font-bold mb-4">Generated Video</h2>
              <video controls className="w-full max-w-md mx-auto">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion studio (for video editing)
npm run remotion:studio
```

## Common Workflows

### Full Content Pipeline

```typescript
// Example: Complete workflow
async function runFullPipeline(keyword: string) {
  // 1. Research phase
  const research = await conductResearch({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h',
    language: 'en',
  });

  // 2. Generate English content
  const englishContent = await generateContentWithClaude({
    topic: keyword,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    researchData: research,
  });

  // 3. Generate Vietnamese content
  const vietnameseContent = await generateContentWithClaude({
    topic: keyword,
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    researchData: research,
  });

  // 4. Generate videos for both languages
  const englishVideo = await generateVideo({
    content: englishContent,
    title: keyword,
    format: 'reels',
  });

  const vietnameseVideo = await generateVideo({
    content: vietnameseContent,
    title: keyword,
    format: 'tiktok',
  });

  return {
    research,
    content: { en: englishContent, vi: vietnameseContent },
    videos: { en: englishVideo, vi: vietnameseVideo },
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = (60 * 1000) / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise((resolve) => setTimeout(resolve, this.delayMs));
    }
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
const result = await limiter.add(() => generateContentWithClaude(config));
```

### Video Rendering Memory Issues

Increase Node.js memory limit:

```bash
# package.json scripts
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Environment Variable Loading

```typescript
// lib/config/env.ts
export const config = {
  openai: {
    apiKey: process.env.OPENAI_API_KEY,
    model: process.env.OPENAI_MODEL || 'gpt-4-turbo-preview',
  },
  anthropic: {
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: process.env.ANTHROPIC_MODEL || 'claude-3-5-sonnet-20241022',
  },
  rapidapi: {
    key: process.env.RAPIDAPI_KEY,
  },
};

// Validation
Object.entries(config).forEach(([service, settings]) => {
  Object.entries(settings).forEach(([key, value]) => {
    if (!value && key === 'apiKey') {
      throw new Error(`Missing ${service}.${key} in environment variables`);
    }
  });
});
```

This skill provides comprehensive guidance for using the Marketing Pipeline Share project to automate content creation from research through video generation.
