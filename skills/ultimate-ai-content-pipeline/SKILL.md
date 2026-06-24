---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI
  - set up content pipeline with research and video
  - create automated marketing content workflow
  - use remotion for video generation from content
  - build AI content automation system
  - integrate Claude and OpenAI for content creation
  - automate content research and script generation
  - generate videos from AI written content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow:

1. **Research Phase**: Automatically crawls and analyzes data from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **Content Generation**: Uses Claude 3 and OpenAI to create diverse content formats (toplists, POV articles, case studies, how-tos) in multiple languages
3. **Video Rendering**: Converts written content into videos and infographics using Remotion
4. **Multi-platform Optimization**: Outputs content ready for Reels, TikTok, Shorts, and blog posts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── pages/             # Page components
├── lib/                   # Core utilities
│   ├── ai/               # AI service integrations
│   ├── research/         # Web scraping & research
│   └── video/            # Remotion video generation
├── remotion/             # Remotion video templates
└── types/                # TypeScript type definitions
```

## Core Workflows

### 1. Research & Data Collection

```typescript
import { ResearchService } from '@/lib/research/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Crawl recent content on a topic
async function gatherResearch(keyword: string) {
  const results = await researchService.crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  // Extract insights
  const insights = await researchService.extractInsights(results);
  
  return {
    rawData: results,
    insights,
    sources: results.map(r => r.url)
  };
}
```

### 2. AI Content Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

// Initialize AI clients
const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Generate content with Claude
async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Based on the following research data, create a ${format} article in ${language}:
  
Research: ${JSON.stringify(research.insights)}
Sources: ${research.sources.join(', ')}

Requirements:
- Use data-backed insights
- Include specific examples and statistics
- Write in an engaging, ${language === 'vi' ? 'Vietnamese' : 'English'} style
- Format as ${format}`;

  const message = await claude.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate content with OpenAI
async function generateContentWithOpenAI(
  research: any,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer creating engaging marketing content.`
      },
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(research.insights)}`
      }
    ],
    temperature: tone === 'humorous' ? 0.9 : 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  background: string;
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: config,
  });

  // Render video
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
    inputProps: config,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Trends 2024',
  content: [
    'AI Automation is booming',
    '90% time saved in content creation',
    'Multi-platform optimization'
  ],
  background: '#1a1a2e',
  duration: 30
});
```

### 4. Complete Pipeline API Route

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/research-service';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, includeVideo } = await request.json();

    // Step 1: Research
    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });
    
    const research = await researchService.crawlSources({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
    });

    const insights = await researchService.extractInsights(research);

    // Step 2: Generate Content
    const content = await generateContentWithClaude(
      { insights, sources: research.map(r => r.url) },
      format,
      language
    );

    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const videoPath = await generateVideo({
        title: keyword,
        content: insights.slice(0, 5),
        background: '#1a1a2e',
        duration: 30
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        sources: research.map(r => r.url),
        generatedAt: new Date().toISOString()
      }
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render Remotion video (standalone)
npm run remotion:render
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(research: any, format: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude(research, format, 'en'),
    generateContentWithClaude(research, format, 'vi')
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### Batch Video Generation

```typescript
async function generateMultiplePlatformVideos(content: string) {
  const configs = [
    { platform: 'tiktok', aspectRatio: '9:16', duration: 15 },
    { platform: 'youtube', aspectRatio: '16:9', duration: 60 },
    { platform: 'instagram', aspectRatio: '1:1', duration: 30 }
  ];

  const videos = await Promise.all(
    configs.map(config => 
      generateVideo({
        title: content.split('\n')[0],
        content: content.split('\n').slice(1, 6),
        background: '#1a1a2e',
        duration: config.duration
      })
    )
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially to avoid memory issues
async function renderVideosSequentially(configs: VideoConfig[]) {
  const results = [];
  for (const config of configs) {
    const video = await generateVideo(config);
    results.push(video);
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  return results;
}
```

### Missing Dependencies

If you encounter module errors:

```bash
# Install all peer dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli
npm install @anthropic-ai/sdk openai
npm install axios cheerio
```

## Performance Optimization

```typescript
// Cache research results
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return JSON.parse(cached);

  const research = await researchService.crawlSources({ keyword });
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(research));
  
  return research;
}
```
