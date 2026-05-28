---
name: marketing-pipeline-share-ai-content
description: Automate content creation from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI
  - set up marketing content pipeline
  - create video content from text automatically
  - research and generate marketing content
  - build AI-powered content workflow
  - automate social media content creation
  - generate multilingual marketing content
  - create reels and shorts with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive system that automates content creation from research to video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** trending news from TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to)
- **Supports bilingual** content (English & Vietnamese)
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Integrates** OpenAI, Anthropic Claude, and RapidAPI

Built with TypeScript and Next.js, it transforms a single keyword into complete content ready for publishing.

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Components and API

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import { fetchNewsFromSources } from './sources';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

async function crawlTrendingTopics(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await Promise.all(
    sources.map(source => 
      fetchNewsFromSources(source, keyword, timeframe)
    )
  );
  
  return results.flat();
}

// Usage
const research = await crawlTrendingTopics('AI marketing', '24h');
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
}

async function generateContent(
  research: ResearchResult[],
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(research, config);

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

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(
  research: ResearchResult[],
  config: ContentConfig
): string {
  const researchSummary = research
    .map(r => `- ${r.title}: ${r.insights.join(', ')}`)
    .join('\n');

  return `
You are a ${config.tone} content creator writing in ${config.language}.
Create a ${config.format} article for ${config.targetAudience}.

Research data from the last 24 hours:
${researchSummary}

Requirements:
- Use data-backed insights
- Include specific examples and numbers
- Make it engaging and actionable
- Format: ${config.format}
`;
}

// Usage
const content = await generateContent(research, {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  targetAudience: 'B2B marketers',
});
```

### 3. Bilingual Content Generation

```typescript
// lib/ai/bilingual-generator.ts
async function generateBilingualContent(
  research: ResearchResult[],
  baseConfig: Omit<ContentConfig, 'language'>
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, { ...baseConfig, language: 'en' }),
    generateContent(research, { ...baseConfig, language: 'vi' }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}

// Usage
const bilingualContent = await generateBilingualContent(research, {
  format: 'how-to',
  tone: 'friendly',
  targetAudience: 'content creators',
});
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts'; // all use 9:16
  duration: number; // in frames (30fps)
}

async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      platform: config.format,
    },
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      platform: config.format,
    },
  });

  return outputPath;
}

// Usage
const videoPath = await generateVideo({
  content: bilingualContent.en,
  format: 'reels',
  duration: 900, // 30 seconds at 30fps
});
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  content,
  platform,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Parse content into scenes
  const scenes = parseContentIntoScenes(content);
  const currentScene = Math.floor(frame / (fps * 3)); // 3 seconds per scene

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          fontSize: 48,
          color: '#fff',
          textAlign: 'center',
          padding: 40,
          maxWidth: '80%',
        }}
      >
        {scenes[currentScene]?.text || ''}
      </div>
    </AbsoluteFill>
  );
};

function parseContentIntoScenes(content: string) {
  // Split content into digestible scenes
  const sentences = content.match(/[^.!?]+[.!?]+/g) || [];
  return sentences.map((text, index) => ({
    id: index,
    text: text.trim(),
  }));
}
```

### 6. Remotion Configuration

```typescript
// remotion/config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);

// For license key
Config.setLicenseKey(process.env.REMOTION_LICENSE_KEY);
```

## Complete Workflow Pattern

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, generateVideo } = await req.json();

    // Step 1: Research
    const research = await crawlTrendingTopics(keyword, '24h');

    // Step 2: Generate Content
    let content: string;
    
    if (language === 'both') {
      const bilingual = await generateBilingualContent(research, {
        format,
        tone: 'expert',
        targetAudience: 'marketers',
      });
      content = bilingual.en;
    } else {
      content = await generateContent(research, {
        format,
        language,
        tone: 'expert',
        targetAudience: 'marketers',
      });
    }

    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const videoPath = await generateVideo({
        content,
        format: 'reels',
        duration: 900,
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      research: research.length,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'both',
        generateVideo: true,
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

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
          className="px-6 py-3 bg-blue-600 text-white rounded"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result?.content && (
          <div className="mt-8 p-6 bg-gray-50 rounded">
            <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
            <div className="prose max-w-none">
              {result.content}
            </div>
            {result.videoUrl && (
              <video
                src={result.videoUrl}
                controls
                className="mt-4 w-full max-w-md"
              />
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Configuration Patterns

### Custom AI Providers

```typescript
// lib/ai/providers.ts
type AIProvider = 'claude' | 'openai';

interface ProviderConfig {
  provider: AIProvider;
  model: string;
  temperature?: number;
  maxTokens?: number;
}

async function generateWithProvider(
  prompt: string,
  config: ProviderConfig
): Promise<string> {
  switch (config.provider) {
    case 'claude':
      return generateWithClaude(prompt, config);
    case 'openai':
      return generateWithOpenAI(prompt, config);
    default:
      throw new Error(`Unknown provider: ${config.provider}`);
  }
}

async function generateWithOpenAI(
  prompt: string,
  config: ProviderConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: config.model || 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    temperature: config.temperature || 0.7,
    max_tokens: config.maxTokens || 4096,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### Content Scheduling

```typescript
// lib/scheduler/schedule.ts
interface ScheduledContent {
  content: string;
  videoUrl?: string;
  platforms: ('facebook' | 'instagram' | 'tiktok')[];
  scheduledTime: Date;
}

async function scheduleContent(
  scheduled: ScheduledContent
): Promise<void> {
  // Store in database with scheduled time
  await db.scheduledPosts.create({
    data: {
      content: scheduled.content,
      videoUrl: scheduled.videoUrl,
      platforms: scheduled.platforms,
      scheduledFor: scheduled.scheduledTime,
      status: 'pending',
    },
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
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
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
const result = await limiter.add(() => generateContent(research, config));
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: config,
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function generateLongFormContent(
  research: ResearchResult[],
  config: ContentConfig,
  sections: number = 5
): Promise<string> {
  const chunks = chunkArray(research, Math.ceil(research.length / sections));
  
  const sectionContents = await Promise.all(
    chunks.map(chunk => generateContent(chunk, config))
  );
  
  return sectionContents.join('\n\n---\n\n');
}

function chunkArray<T>(array: T[], size: number): T[][] {
  return Array.from(
    { length: Math.ceil(array.length / size) },
    (_, i) => array.slice(i * size, i * size + size)
  );
}
```

### Environment Variable Validation

```typescript
// lib/config/validate.ts
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call on startup
validateEnv();
```

## Common Use Cases

### Daily Content Automation

```typescript
// scripts/daily-automation.ts
async function runDailyContentPipeline() {
  const topics = ['AI marketing', 'content automation', 'social media trends'];
  
  for (const topic of topics) {
    const research = await crawlTrendingTopics(topic, '24h');
    
    const content = await generateBilingualContent(research, {
      format: 'toplist',
      tone: 'friendly',
      targetAudience: 'marketers',
    });
    
    const videoPath = await generateVideo({
      content: content.en,
      format: 'reels',
      duration: 900,
    });
    
    await scheduleContent({
      content: content.en,
      videoUrl: `/videos/${path.basename(videoPath)}`,
      platforms: ['instagram', 'tiktok'],
      scheduledTime: new Date(Date.now() + 86400000), // Tomorrow
    });
  }
}
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation and distribution.
