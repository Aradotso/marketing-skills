---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up marketing pipeline with auto research and posting
  - generate videos automatically from blog content
  - create content pipeline from research to social media
  - automate content workflow with Claude and Remotion
  - build AI content generation system with video rendering
  - scrape news and generate marketing content automatically
  - set up automated content research and video creation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire workflow from research to video generation. It automatically scrapes recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats (toplist, POV, case studies, how-to), supports both English and Vietnamese with customizable tones, and renders videos/infographics using Remotion.

**Key capabilities:**
- Auto-research from real-time sources (24h news)
- Multi-format content generation (Claude 3, OpenAI)
- Bilingual support (EN/VI)
- Automated video rendering (Remotion)
- Next.js dashboard interface

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

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research Module

```typescript
import { researchContent } from '@/lib/research/scraper';

// Auto-research from multiple sources
async function gatherResearch(keyword: string) {
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });

  return {
    insights: research.insights,
    dataPoints: research.dataPoints,
    trends: research.trends,
    sources: research.sources
  };
}

// Example usage
const aiResearch = await gatherResearch('artificial intelligence startups');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate content in specific format
async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `
Based on this research data: ${JSON.stringify(research)}

Create a ${format} article in ${language} with a ${tone} tone.

Requirements:
- Include data-backed insights
- Add relevant statistics
- Structure for readability
- Optimize for social media sharing
  `.trim();

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Example: Generate bilingual content
const englishContent = await generateContent(
  aiResearch,
  'toplist',
  'en',
  'expert'
);

const vietnameseContent = await generateContent(
  aiResearch,
  'toplist',
  'vi',
  'friendly'
);
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

// Render video from content
async function generateVideo(content: {
  title: string;
  keyPoints: string[];
  data: any[];
  style: 'reels' | 'tiktok' | 'shorts';
}) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content
  });

  // Render video
  const outputLocation = `public/videos/${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content
  });

  return outputLocation;
}

// Example: Create social media video
const videoPath = await generateVideo({
  title: 'Top 5 AI Trends 2024',
  keyPoints: [
    'Multimodal AI becomes mainstream',
    'Open source LLMs gain traction',
    'AI agents automate workflows',
    'Edge AI for privacy',
    'AI regulation frameworks emerge'
  ],
  data: aiResearch.dataPoints,
  style: 'reels'
});
```

### 4. Complete Pipeline Example

```typescript
import { researchContent } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { schedulePost } from '@/lib/social/scheduler';

// Full automation pipeline
async function runContentPipeline(
  keyword: string,
  platforms: ('facebook' | 'instagram' | 'tiktok')[]
) {
  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      maxResults: 15
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent(
      research,
      'toplist',
      'en',
      'expert'
    );

    // Extract key points for video
    const keyPoints = extractKeyPoints(content);

    // Step 3: Generate video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: `${keyword}: Latest Insights`,
      keyPoints,
      data: research.dataPoints,
      style: 'reels'
    });

    // Step 4: Schedule posts
    console.log('📅 Scheduling posts...');
    for (const platform of platforms) {
      await schedulePost({
        platform,
        content,
        video: videoPath,
        scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
      });
    }

    return {
      success: true,
      content,
      videoPath,
      platforms
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Helper to extract key points
function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^\d+\./))
    .map(line => line.replace(/^\d+\.\s*/, ''))
    .slice(0, 5);
}

// Execute pipeline
const result = await runContentPipeline(
  'AI automation tools',
  ['facebook', 'instagram', 'tiktok']
);
```

### 5. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, platforms, format, language, tone } = body;

    // Validate inputs
    if (!keyword || !platforms || platforms.length === 0) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline(keyword, platforms);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### 6. React Component for Dashboard

```typescript
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
        body: JSON.stringify({
          keyword,
          platforms: ['facebook', 'instagram'],
          format: 'toplist',
          language: 'en',
          tone: 'expert'
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
    <div className="p-6 max-w-2xl mx-auto">
      <h2 className="text-2xl font-bold mb-4">
        AI Content Generator
      </h2>
      
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="w-full p-3 border rounded mb-4"
      />

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content & Video'}
      </button>

      {result && (
        <div className="mt-6 p-4 bg-gray-50 rounded">
          <h3 className="font-semibold mb-2">Result:</h3>
          <p className="text-sm text-gray-600">
            Video: {result.data?.videoPath}
          </p>
          <p className="text-sm text-gray-600">
            Platforms: {result.data?.platforms.join(', ')}
          </p>
        </div>
      )}
    </div>
  );
}
```

## Common Tasks

### Custom Research Source

```typescript
// lib/research/custom-source.ts
export async function scrapeCustomSource(url: string) {
  const response = await fetch(url);
  const html = await response.text();
  
  // Parse HTML (use cheerio or similar)
  const articles = parseArticles(html);
  
  return articles.map(article => ({
    title: article.title,
    excerpt: article.excerpt,
    url: article.url,
    publishedAt: article.date,
    source: 'custom'
  }));
}
```

### Custom Video Template

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60, opacity }}>
        <h1 style={{ color: 'white', fontSize: 60, marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, i) => (
          <p key={i} style={{ color: 'white', fontSize: 30, marginBottom: 20 }}>
            • {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render Remotion video
npm run remotion:render
```

## Troubleshooting

**API Rate Limits:**
- Implement request queuing for high-volume usage
- Cache research results for 24 hours
- Use different API keys for different environments

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay = 1000; // ms between requests

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
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    this.processing = false;
  }
}
```

**Video Rendering Memory Issues:**
- Reduce video resolution for development
- Use `--max-old-space-size` flag
- Render videos in background jobs

```bash
node --max-old-space-size=4096 node_modules/.bin/remotion render
```

**Content Quality Issues:**
- Adjust AI prompt templates in `lib/ai/prompts.ts`
- Increase research data volume
- Add content validation step before publishing

This skill enables AI coding agents to effectively work with the Marketing Pipeline Share system for automated content creation, research, and video generation workflows.
