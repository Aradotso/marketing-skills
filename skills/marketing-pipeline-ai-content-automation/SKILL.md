---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate videos from text content automatically
  - set up AI marketing content pipeline
  - create automated content workflows with Claude and OpenAI
  - build AI-powered content research and generation system
  - automate social media content and video creation
  - integrate Remotion for automated video rendering
  - create multi-language content with AI automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls fresh data from news sources, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-scans research data** from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates AI content** in multiple formats (toplist, POV, case study, how-to)
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for Reels, TikTok, Shorts
- **Provides Next.js interface** for easy content scheduling and management

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

### Required Environment Variables

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### 1. Research Module (Data Crawling)

The research module crawls fresh content from various sources:

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchSource {
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  keywords: string[];
  timeRange: '24h' | '7d' | '30d';
}

export async function crawlResearchData(config: ResearchSource) {
  const apiClient = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await apiClient.search({
    query: config.keywords.join(' OR '),
    sources: [config.source],
    from: new Date(Date.now() - 24 * 60 * 60 * 1000), // 24h ago
    to: new Date(),
  });
  
  return results.map(item => ({
    title: item.title,
    content: item.content,
    url: item.url,
    publishedAt: item.publishedAt,
    insights: extractInsights(item.content),
  }));
}

function extractInsights(content: string): string[] {
  // Extract key insights, statistics, and data points
  const insights: string[] = [];
  
  // Look for numbers and percentages
  const stats = content.match(/\d+%|\$\d+[MBK]?/g);
  if (stats) insights.push(...stats);
  
  return insights;
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  tone: Tone;
  language: 'en' | 'vi';
  researchData: any[];
}

export class AIContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }
  
  async generateWithClaude(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
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
  
  async generateWithOpenAI(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing content.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
      max_tokens: 4096,
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a specific point of view with personal insights',
      'case-study': 'Analyze a real-world example with data and outcomes',
      'how-to': 'Provide step-by-step instructions with actionable tips',
    };
    
    const toneInstructions = {
      'expert': 'Use professional, authoritative language',
      'friendly': 'Use conversational, approachable language',
      'humorous': 'Use engaging, witty language with occasional humor',
    };
    
    return `
Create a ${config.format} article in ${config.language === 'en' ? 'English' : 'Vietnamese'}.

Format: ${formatInstructions[config.format]}
Tone: ${toneInstructions[config.tone]}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Requirements:
- Include specific data points and statistics from the research
- Make it engaging and valuable for the target audience
- Optimize for SEO and social sharing
- Include a compelling headline and introduction
${config.language === 'vi' ? '- Write naturally in Vietnamese, avoiding awkward translations' : ''}
    `.trim();
  }
}
```

### 3. Bilingual Content Generation

Generate content in both languages simultaneously:

```typescript
// lib/ai/bilingual-generator.ts
import { AIContentGenerator, ContentConfig } from './content-generator';

export async function generateBilingualContent(
  config: Omit<ContentConfig, 'language'>
): Promise<{ en: string; vi: string }> {
  const generator = new AIContentGenerator();
  
  const [enContent, viContent] = await Promise.all([
    generator.generateWithClaude({ ...config, language: 'en' }),
    generator.generateWithClaude({ ...config, language: 'vi' }),
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

Create videos from generated content:

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export class VideoRenderer {
  private dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  async renderContentVideo(config: VideoConfig): Promise<string> {
    // Bundle the Remotion project
    const bundled = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });
    
    // Get composition
    const composition = await selectComposition({
      serveUrl: bundled,
      id: 'ContentVideo',
      inputProps: {
        content: config.content,
        title: config.title,
      },
    });
    
    // Render video
    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}-${config.format}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundled,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        content: config.content,
        title: config.title,
      },
    });
    
    return outputPath;
  }
}
```

### 5. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: '60px',
          maxWidth: '900px',
        }}
      >
        <h1
          style={{
            color: '#fff',
            fontSize: '72px',
            fontWeight: 'bold',
            marginBottom: '40px',
            textAlign: 'center',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            color: '#e0e0e0',
            fontSize: '36px',
            lineHeight: '1.6',
            textAlign: 'center',
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
import { crawlResearchData } from '../research/crawler';
import { AIContentGenerator } from '../ai/content-generator';
import { generateBilingualContent } from '../ai/bilingual-generator';
import { VideoRenderer } from '../video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormats?: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research & Data Crawling
  console.log('📡 Crawling research data...');
  const researchData = await crawlResearchData({
    source: 'techcrunch',
    keywords: [config.keyword],
    timeRange: '24h',
  });
  
  // Step 2: Generate Bilingual Content
  console.log('🧠 Generating content with AI...');
  const content = await generateBilingualContent({
    format: config.format,
    tone: config.tone,
    researchData,
  });
  
  // Step 3: Generate Videos (if requested)
  let videos: string[] = [];
  if (config.generateVideo && config.videoFormats) {
    console.log('🎬 Rendering videos...');
    const renderer = new VideoRenderer();
    
    videos = await Promise.all(
      config.videoFormats.map(format =>
        renderer.renderContentVideo({
          content: content.en,
          title: `${config.keyword} - ${config.format}`,
          format,
        })
      )
    );
  }
  
  return {
    content,
    videos,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      tone: config.tone,
      researchSources: researchData.length,
      generatedAt: new Date().toISOString(),
    },
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      videoFormats: body.videoFormats || ['reels'],
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'expert',
          generateVideo: true,
          videoFormats: ['reels', 'tiktok'],
        }),
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-4 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="px-6 py-3 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-8 space-y-4">
            <h2 className="text-2xl font-bold">Results</h2>
            <div className="grid grid-cols-2 gap-4">
              <div>
                <h3 className="font-bold">English Content</h3>
                <p className="whitespace-pre-wrap">{result.content.en}</p>
              </div>
              <div>
                <h3 className="font-bold">Vietnamese Content</h3>
                <p className="whitespace-pre-wrap">{result.content.vi}</p>
              </div>
            </div>
            {result.videos.length > 0 && (
              <div>
                <h3 className="font-bold">Generated Videos</h3>
                <ul>
                  {result.videos.map((video: string, i: number) => (
                    <li key={i}>
                      <a href={video} className="text-blue-600">
                        Video {i + 1}
                      </a>
                    </li>
                  ))}
                </ul>
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron-jobs.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/content-pipeline';

export function scheduleContentGeneration() {
  // Run daily at 8 AM
  cron.schedule('0 8 * * *', async () => {
    console.log('Running scheduled content generation...');
    
    const keywords = ['AI trends', 'marketing automation', 'content creation'];
    
    for (const keyword of keywords) {
      await runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'expert',
        generateVideo: true,
        videoFormats: ['reels', 'shorts'],
      });
    }
  });
}
```

### Pattern 2: Batch Processing

```typescript
// lib/batch/batch-processor.ts
export async function processBatchKeywords(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'expert',
        generateVideo: false,
      });
      
      results.push({ keyword, success: true, result });
    } catch (error) {
      results.push({ keyword, success: false, error });
    }
  }
  
  return results;
}
```

### Pattern 3: Content Variation Generation

```typescript
// lib/variations/content-variations.ts
export async function generateContentVariations(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  const tones = ['expert', 'friendly', 'humorous'] as const;
  
  const variations = [];
  
  for (const format of formats) {
    for (const tone of tones) {
      const result = await runContentPipeline({
        keyword,
        format,
        tone,
        generateVideo: false,
      });
      
      variations.push({ format, tone, content: result.content });
    }
  }
  
  return variations;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;
  
  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }
  
  async throttle() {
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
const limiter = new RateLimiter(10); // 10 calls per minute
await limiter.throttle();
await apiCall();
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  
  throw lastError!;
}
```

### Video Rendering Memory Issues

If video rendering fails due to memory:

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Content Quality Validation

```typescript
// lib/validation/content-validator.ts
export function validateContent(content: string): {
  valid: boolean;
  issues: string[];
} {
  const issues: string[] = [];
  
  if (content.length < 500) {
    issues.push('Content too short (minimum 500 characters)');
  }
  
  if (!/\d+%|\$\d+/.test(content)) {
    issues.push('Missing statistics or data points');
  }
  
  if (!content.includes('\n')) {
    issues.push('Missing paragraph breaks');
  }
  
  return {
    valid: issues.length === 0,
    issues,
  };
}
```

This skill enables AI agents to effectively use the Marketing Pipeline for automated content creation, from research through video generation, with proper error handling and optimization strategies.
