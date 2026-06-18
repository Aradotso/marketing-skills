---
name: marketing-pipeline-ai-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, and video generation using Claude/OpenAI
triggers:
  - automate content creation with AI research and video
  - generate marketing content from trending topics
  - create social media videos automatically
  - build content pipeline with Claude and OpenAI
  - scrape news and generate content automatically
  - setup AI content automation system
  - use remotion for automated video rendering
  - build marketing content generation workflow
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline is an end-to-end AI content automation system that transforms keywords into finished marketing content. It combines web scraping, AI content generation (Claude 3/OpenAI), and automated video rendering (Remotion) into a single pipeline.

**Key capabilities:**
- Auto-scrape trending news from TechCrunch, a16z, Twitter, LinkedIn
- Generate multi-format content (toplist, POV, case study, how-to)
- Bilingual output (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering for social media
- Next.js interface for managing content workflow

## Installation

```bash
# Clone repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install

# Setup environment variables
cp .env.example .env.local
```

## Configuration

Create `.env.local` with required API keys:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion
```

## Core Architecture

### 1. Research Pipeline

The research module scrapes and analyzes trending content:

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
}

export async function fetchTrendingContent(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get('/api/scrape', {
        params: {
          url: source.url,
          keyword,
          platform: source.platform,
          timeframe: source.timeframe
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      return {
        platform: source.platform,
        articles: response.data.results,
        insights: extractInsights(response.data.results)
      };
    })
  );
  
  return results;
}

function extractInsights(articles: Article[]): Insight[] {
  return articles.map(article => ({
    headline: article.title,
    summary: article.description,
    metrics: article.engagement,
    url: article.url,
    publishedAt: article.publishedAt
  }));
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  wordCount: number;
}

export async function generateContent(
  research: ResearchData[],
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<GeneratedContent> {
  const prompt = buildPrompt(research, config);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return parseContent(message.content[0].text, config);
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return parseContent(completion.choices[0].message.content, config);
  }
}

function buildPrompt(research: ResearchData[], config: ContentConfig): string {
  const researchSummary = research
    .flatMap(r => r.insights)
    .map(i => `- ${i.headline}: ${i.summary}`)
    .join('\n');
  
  return `
You are an expert content creator. Generate a ${config.format} article based on this research:

${researchSummary}

Requirements:
- Format: ${config.format}
- Tone: ${config.tone}
- Language: ${config.language}
- Target length: ${config.wordCount} words
- Include data-backed insights
- Make it engaging and actionable

Output as JSON with: title, introduction, sections[], conclusion, cta
`;
}
```

### 3. Video Rendering with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  sections: Array<{ headline: string; content: string }>;
  branding: {
    logo: string;
    primaryColor: string;
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  sections,
  branding
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  return (
    <AbsoluteFill style={{ backgroundColor: branding.primaryColor }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <TitleSlide title={title} logo={branding.logo} />
      </Sequence>
      
      {sections.map((section, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 5)}
          durationInFrames={fps * 5}
        >
          <ContentSlide headline={section.headline} content={section.content} />
        </Sequence>
      ))}
      
      <Sequence from={fps * (3 + sections.length * 5)} durationInFrames={fps * 2}>
        <CTASlide />
      </Sequence>
    </AbsoluteFill>
  );
};

// remotion/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle(path.join(__dirname, './index.ts'));
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      branding: {
        logo: '/assets/logo.png',
        primaryColor: '#FF6B6B'
      }
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });
  
  return outputPath;
}
```

### 4. API Routes

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { fetchTrendingContent } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/remotion/render';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, renderVideo } = req.body;
    
    // Step 1: Research
    const research = await fetchTrendingContent(keyword, [
      { url: 'techcrunch.com', platform: 'techcrunch', timeframe: '24h' },
      { url: 'a16z.com/blog', platform: 'a16z', timeframe: '24h' }
    ]);
    
    // Step 2: Generate content
    const content = await generateContent(research, {
      format,
      tone: 'professional',
      language,
      wordCount: 1500
    });
    
    // Step 3: Render video (optional)
    let videoUrl = null;
    if (renderVideo) {
      const videoPath = await renderContentVideo(
        content,
        `/tmp/video-${Date.now()}.mp4`
      );
      videoUrl = await uploadToStorage(videoPath);
    }
    
    res.status(200).json({
      content,
      videoUrl,
      research: research.map(r => ({
        platform: r.platform,
        articleCount: r.articles.length
      }))
    });
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Usage Patterns

### Full Pipeline Example

```typescript
// app/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async (keyword: string) => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          renderVideo: true
        })
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
    <div>
      <input
        type="text"
        placeholder="Enter keyword..."
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            handleGenerate(e.currentTarget.value);
          }
        }}
      />
      
      {loading && <div>Generating content...</div>}
      
      {result && (
        <div>
          <h2>{result.content.title}</h2>
          <article>{result.content.introduction}</article>
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

### Batch Processing

```typescript
// scripts/batch-generate.ts
import { fetchTrendingContent } from '../lib/research/scraper';
import { generateContent } from '../lib/ai/content-generator';

const keywords = ['AI marketing', 'content automation', 'video marketing'];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await fetchTrendingContent(keyword, [
      { url: 'techcrunch.com', platform: 'techcrunch', timeframe: '24h' }
    ]);
    
    const content = await generateContent(research, {
      format: 'how-to',
      tone: 'friendly',
      language: 'en',
      wordCount: 1200
    });
    
    // Save to database or file
    await saveContent(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

batchGenerate();
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.processQueue();
  }
}

// Usage
const limiter = new RateLimiter(2000);
const content = await limiter.execute(() => generateContent(research, config));
```

### Video Rendering Memory Issues

If Remotion fails with memory errors:

```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion
```

Or split rendering into chunks:

```typescript
export async function renderInChunks(content: GeneratedContent) {
  const chunks = splitIntoChunks(content.sections, 3);
  const videos = [];
  
  for (const chunk of chunks) {
    const videoPath = await renderContentVideo(
      { ...content, sections: chunk },
      `/tmp/chunk-${videos.length}.mp4`
    );
    videos.push(videoPath);
  }
  
  return concatenateVideos(videos);
}
```

### Claude/OpenAI Timeout Handling

```typescript
export async function generateWithRetry(
  research: ResearchData[],
  config: ContentConfig,
  maxRetries = 3
): Promise<GeneratedContent> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(research, config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  
  throw new Error('Max retries exceeded');
}
```
