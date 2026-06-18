---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate video content from articles automatically
  - build automated marketing pipeline with Claude
  - create AI content workflow from research to video
  - set up content automation with Remotion rendering
  - integrate OpenAI and Claude for content generation
  - crawl news and generate marketing content
  - automate social media content pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, a complete AI-powered content automation system that handles research, scriptwriting, article generation, and video rendering. The pipeline crawls real-time data from news sources, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

Marketing Pipeline Share is a TypeScript-based automation pipeline that:

- **Auto-crawls** recent content from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates articles** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically from text content using Remotion
- **Optimizes for platforms** (Reels, TikTok, Shorts) with proper aspect ratios

## Installation

### Prerequisites

```bash
# Required
node >= 18.x
npm or yarn
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Configure environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following keys:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if needed)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Remotion Configuration
REMOTION_RENDER_CONCURRENCY=4
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── crawlers/          # News/content crawlers
│   ├── generators/        # AI content generation
│   ├── renderers/         # Remotion video rendering
│   ├── api/              # Next.js API routes
│   ├── components/       # React components
│   └── utils/            # Helper functions
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── HUONG_DAN_CAI_DAT.md # Detailed installation guide
```

## Key APIs and Usage Patterns

### 1. Content Research/Crawling

```typescript
// src/crawlers/newsResearch.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: string;
  url: string;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        params: { q: keyword, timeframe: '24h' },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      }
    );
    
    results.push(...response.data.articles);
  }
  
  return results;
}

// Usage
const insights = await crawlRecentNews('AI marketing', ['techcrunch']);
```

### 2. AI Content Generation with Claude

```typescript
// src/generators/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(config: ContentConfig): Promise<string> {
  const systemPrompt = buildSystemPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Generate a ${config.format} article based on this research:\n\n${config.researchData}`,
      },
    ],
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(config: ContentConfig): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining',
  };
  
  return `You are a ${toneMap[config.tone]} content writer creating ${config.format} articles in ${config.language}. Use data-backed insights and maintain high quality.`;
}

// Usage
const article = await generateContent({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: insights.map(i => i.content).join('\n\n'),
});
```

### 3. OpenAI Alternative

```typescript
// src/generators/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(
  prompt: string,
  systemMessage: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemMessage },
      { role: 'user', content: prompt },
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Remotion Video Rendering

```typescript
// src/renderers/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  outputPath: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
  });
  
  return config.outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Marketing Trends 2024',
  keyPoints: [
    'AI-powered personalization',
    'Automated content generation',
    'Predictive analytics',
  ],
  outputPath: './output/video.mp4',
  format: 'reels',
});
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
          }}
        >
          <h1
            style={{
              fontSize: 60,
              color: 'white',
              opacity: Math.min(1, frame / 30),
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40,
            }}
          >
            <div
              style={{
                fontSize: 40,
                color: 'white',
                textAlign: 'center',
              }}
            >
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// src/pipeline/contentPipeline.ts
import { crawlRecentNews } from '../crawlers/newsResearch';
import { generateContent } from '../generators/claudeGenerator';
import { renderContentVideo } from '../renderers/videoRenderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Researching content...');
  const research = await crawlRecentNews(config.keyword);
  
  console.log('✍️ Step 2: Generating article...');
  const article = await generateContent({
    format: config.contentFormat,
    language: config.language,
    tone: 'expert',
    researchData: research.map(r => r.content).join('\n\n'),
  });
  
  let videoPath = null;
  if (config.generateVideo) {
    console.log('🎬 Step 3: Rendering video...');
    
    // Extract key points from article
    const keyPoints = extractKeyPoints(article);
    
    videoPath = await renderContentVideo({
      title: `${config.keyword} - Key Insights`,
      keyPoints,
      outputPath: `./output/${Date.now()}.mp4`,
      format: 'reels',
    });
  }
  
  console.log('✅ Pipeline complete!');
  return {
    article,
    videoPath,
    research,
  };
}

function extractKeyPoints(article: string): string[] {
  // Simple extraction - in production use AI to extract
  const lines = article.split('\n').filter(l => l.trim().startsWith('-'));
  return lines.slice(0, 5).map(l => l.replace(/^-\s*/, ''));
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  contentFormat: 'toplist',
  language: 'en',
  generateVideo: true,
});
```

## Next.js API Routes

```typescript
// src/pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/pipeline/contentPipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, generateVideo } = req.body;
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: format,
      language,
      generateVideo: generateVideo ?? false,
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline failed',
      message: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Running the Project

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video only (if separate script)
npm run render-video
```

## Common Patterns

### Bilingual Content Generation

```typescript
async function generateBilingualContent(keyword: string) {
  const research = await crawlRecentNews(keyword);
  
  const [englishArticle, vietnameseArticle] = await Promise.all([
    generateContent({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research.map(r => r.content).join('\n\n'),
    }),
    generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData: research.map(r => r.content).join('\n\n'),
    }),
  ]);
  
  return { englishArticle, vietnameseArticle };
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      language: 'en',
      generateVideo: true,
    });
    
    results.push({ keyword, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/utils/rateLimiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async enqueue<T>(fn: () => Promise<T>): Promise<T> {
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
    const fn = this.queue.shift()!;
    
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    
    this.processing = false;
    this.process();
  }
}

export const rateLimiter = new RateLimiter();
```

### Error Handling

```typescript
async function safeGenerateContent(config: ContentConfig): Promise<string | null> {
  try {
    return await generateContent(config);
  } catch (error) {
    if (error instanceof Anthropic.APIError) {
      if (error.status === 429) {
        console.error('Rate limit exceeded, waiting...');
        await new Promise(resolve => setTimeout(resolve, 60000));
        return safeGenerateContent(config);
      }
    }
    
    console.error('Content generation failed:', error);
    return null;
  }
}
```

### Video Rendering Issues

```bash
# If Remotion fails, check Chrome installation
npx remotion browser ensure

# Clear Remotion cache
rm -rf node_modules/.cache/remotion

# Test render with verbose logging
npx remotion render remotion/index.ts ContentVideo output.mp4 --log=verbose
```

### Memory Issues with Large Batches

```typescript
// Process in smaller chunks
async function processBatches<T>(
  items: T[],
  batchSize: number,
  processor: (item: T) => Promise<any>
) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await Promise.all(batch.map(processor));
    
    // Force garbage collection if available
    if (global.gc) global.gc();
  }
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI agents to help developers build end-to-end content automation workflows.
