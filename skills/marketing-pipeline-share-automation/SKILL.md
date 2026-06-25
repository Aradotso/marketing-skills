---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI integration
triggers:
  - "help me automate content creation with AI"
  - "how do I use the marketing pipeline for content generation"
  - "set up automated content research and video generation"
  - "generate AI content from research to video"
  - "configure the content automation pipeline"
  - "create automated marketing content with Claude"
  - "build an AI content workflow from scratch"
  - "integrate OpenAI for content generation pipeline"
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the **Ultimate AI Content Pipeline** - a complete content automation system that handles everything from research, scriptwriting, to video generation. The system leverages Claude 3, OpenAI, and Remotion to create a fully automated content production pipeline.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-crawls research data** from sources like TechCrunch, a16z, X (Twitter), LinkedIn within the last 24 hours
- **Generates multi-format content** (Toplist, POV, Case Study, How-to) in multiple languages (English, Vietnamese)
- **Renders videos and infographics automatically** using Remotion
- **Optimizes for multi-platform distribution** (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Config
REMOTION_BUNDLE_TIMEOUT=120000

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key Architecture & Components

### 1. Research Module

The research module automatically crawls and analyzes content from various sources.

```typescript
// lib/research/crawler.ts
import { type ResearchSource } from '@/types/research';

interface CrawlOptions {
  keywords: string[];
  timeframe: '24h' | '7d' | '30d';
  sources: ResearchSource[];
  language?: 'en' | 'vi';
}

export async function crawlResearch(options: CrawlOptions) {
  const { keywords, timeframe, sources } = options;
  
  const results = await Promise.all(
    sources.map(source => {
      switch(source) {
        case 'techcrunch':
          return crawlTechCrunch(keywords, timeframe);
        case 'twitter':
          return crawlTwitter(keywords, timeframe);
        case 'linkedin':
          return crawlLinkedIn(keywords, timeframe);
        default:
          return null;
      }
    })
  );
  
  return results.filter(Boolean);
}

async function crawlTechCrunch(keywords: string[], timeframe: string) {
  const response = await fetch('https://techcrunch-api.p.rapidapi.com/search', {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
    }
  });
  
  const data = await response.json();
  return data.articles;
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI with customizable formats and tones.

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  research: any[];
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
  provider: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions) {
  const { research, format, tone, language, provider } = options;
  
  const prompt = buildPrompt(research, format, tone, language);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
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

async function generateWithOpenAI(prompt: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are an expert content creator.' },
      { role: 'user', content: prompt }
    ],
    max_tokens: 4096,
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(
  research: any[], 
  format: ContentFormat, 
  tone: ToneOfVoice, 
  language: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips'
  };
  
  return `
    Based on the following research data, create ${language === 'en' ? 'an English' : 'a Vietnamese'} article in ${format} format with a ${tone} tone.
    
    Format: ${formatInstructions[format]}
    
    Research Data:
    ${JSON.stringify(research, null, 2)}
    
    Requirements:
    - Include relevant statistics and data points
    - Make it engaging and actionable
    - Optimize for SEO
    - Length: 1500-2000 words
  `;
}
```

### 3. Video Generation with Remotion

Automatically generate videos from content using Remotion.

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration?: number;
}

export async function renderVideo(config: VideoConfig) {
  const { content, title, platform, duration = 60 } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: parseContentForVideo(content),
      ...dimensions[platform]
    },
  });
  
  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps,
  });
  
  return outputLocation;
}

function parseContentForVideo(content: string) {
  // Break content into scenes/frames
  const paragraphs = content.split('\n\n');
  return paragraphs.slice(0, 5).map((text, index) => ({
    id: `scene-${index}`,
    text: text.trim(),
    duration: 30 // frames
  }));
}
```

### 4. Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { 
      keywords, 
      format, 
      tone, 
      language, 
      generateVideo: shouldGenerateVideo,
      platform 
    } = body;
    
    // Step 1: Research
    const research = await crawlResearch({
      keywords,
      timeframe: '24h',
      sources: ['techcrunch', 'twitter', 'linkedin'],
      language
    });
    
    // Step 2: Generate Content
    const content = await generateContent({
      research,
      format,
      tone,
      language,
      provider: 'claude' // or 'openai'
    });
    
    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (shouldGenerateVideo) {
      const videoPath = await renderVideo({
        content,
        title: keywords[0],
        platform,
        duration: 60
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        research: research.length
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

### Using the Pipeline from React Components

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const runPipeline = async (config: any) => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(config)
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="pipeline-container">
      <button 
        onClick={() => runPipeline({
          keywords: ['AI marketing', 'automation'],
          format: 'toplist',
          tone: 'expert',
          language: 'en',
          generateVideo: true,
          platform: 'reels'
        })}
        disabled={loading}
      >
        {loading ? 'Processing...' : 'Run Pipeline'}
      </button>
      
      {result && (
        <div className="result">
          <h3>Generated Content</h3>
          <pre>{result.data.content}</pre>
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// lib/batch/processor.ts
export async function batchGenerateContent(topics: string[]) {
  const results = [];
  
  for (const topic of topics) {
    const research = await crawlResearch({
      keywords: [topic],
      timeframe: '24h',
      sources: ['techcrunch', 'twitter']
    });
    
    const content = await generateContent({
      research,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      provider: 'claude'
    });
    
    results.push({ topic, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 2: Multi-Language Content

```typescript
export async function generateMultiLanguageContent(config: any) {
  const languages = ['en', 'vi'];
  const contents: Record<string, string> = {};
  
  for (const language of languages) {
    contents[language] = await generateContent({
      ...config,
      language
    });
  }
  
  return contents;
}
```

### Pattern 3: Content Scheduling

```typescript
// lib/scheduler/content-scheduler.ts
interface ScheduledContent {
  id: string;
  content: string;
  publishAt: Date;
  platform: string;
}

export async function scheduleContent(scheduled: ScheduledContent) {
  // Save to database
  await db.scheduledContent.create({
    data: {
      id: scheduled.id,
      content: scheduled.content,
      publishAt: scheduled.publishAt,
      platform: scheduled.platform,
      status: 'pending'
    }
  });
  
  // Set up cron job or queue
  return scheduled.id;
}
```

## Configuration

### Remotion Video Configuration

```typescript
// remotion/config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2);
Config.setCodec('h264');
```

### AI Provider Configuration

```typescript
// lib/config/ai-providers.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
} as const;
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
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
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}
```

### Issue: Video Rendering Timeout

Increase timeout in `.env.local`:

```env
REMOTION_BUNDLE_TIMEOUT=300000
REMOTION_TIMEOUT_IN_MILLISECONDS=300000
```

### Issue: Memory Issues with Large Content

```typescript
// Use streaming for large content generation
export async function generateContentStream(options: GenerateContentOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: options.prompt }]
  });
  
  let fullContent = '';
  
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && 
        chunk.delta.type === 'text_delta') {
      fullContent += chunk.delta.text;
    }
  }
  
  return fullContent;
}
```

## Best Practices

1. **Always use environment variables** for API keys - never hardcode them
2. **Implement rate limiting** when calling external APIs
3. **Cache research results** to avoid redundant API calls
4. **Use queue systems** (Bull, BullMQ) for video rendering in production
5. **Monitor API usage** to stay within quota limits
6. **Validate content** before publishing to ensure quality
7. **Use TypeScript strictly** for type safety across the pipeline

This skill enables AI coding agents to effectively use and extend the Marketing Pipeline Share automation system for complete content workflow automation.
