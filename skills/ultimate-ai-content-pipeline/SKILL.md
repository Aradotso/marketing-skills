---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated video generation from text
  - create content pipeline with Claude and OpenAI
  - generate videos automatically from research
  - build AI content workflow with Remotion
  - automate content research and video rendering
  - create multilingual content with AI pipeline
  - set up automated content publishing system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to crawl news sources, generate content in multiple formats and languages, and render videos automatically.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** news from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Supports multilingual** output (English and Vietnamese)
- **Renders videos** and infographics automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, and Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# pnpm (recommended) or npm
npm install -g pnpm
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```typescript
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
// src/lib/crawler/news-crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsSource {
  title: string;
  url: string;
  content: string;
  publishedAt: string;
  source: string;
}

export async function crawlRecentNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsSource[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      client.search({
        query: keyword,
        source,
        timeframe,
        limit: 10
      })
    )
  );
  
  return results.flat();
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });

  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.topic, request.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(format: string, tone: string): string {
  const prompts = {
    toplist: `You are an expert content writer creating engaging top-list articles.`,
    pov: `You are a thought leader sharing unique perspectives and opinions.`,
    'case-study': `You are a business analyst creating data-driven case studies.`,
    'how-to': `You are an instructor creating clear, actionable how-to guides.`,
  };

  const tones = {
    expert: 'Use professional, authoritative language with industry jargon.',
    friendly: 'Use conversational, approachable language.',
    humorous: 'Use witty, entertaining language while maintaining credibility.',
  };

  return `${prompts[format as keyof typeof prompts]} ${tones[tone as keyof typeof tones]}`;
}

function buildUserPrompt(topic: string, researchData: string[]): string {
  return `
Topic: ${topic}

Recent Research Data:
${researchData.map((data, i) => `${i + 1}. ${data}`).join('\n')}

Create a comprehensive article based on the topic and research data provided. 
Include data-backed insights, statistics, and real-world examples.
  `.trim();
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(request.format, request.tone),
      },
      {
        role: 'user',
        content: buildUserPrompt(request.topic, request.researchData),
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  bgColor: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
      bgColor: config.bgColor,
    },
  });

  const { width, height } = platformDimensions[config.platform];

  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      slides: config.content,
      bgColor: config.bgColor,
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  slides: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  slides,
  bgColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const slideDuration = fps * 3; // 3 seconds per slide

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence durationInFrames={fps * 2}>
        <TitleSlide title={title} />
      </Sequence>
      
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={(index + 1) * slideDuration}
          durationInFrames={slideDuration}
        >
          <ContentSlide content={slide} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleSlide: React.FC<{ title: string }> = ({ title }) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <h1 style={{ fontSize: 80, color: 'white', textAlign: 'center', padding: 40 }}>
        {title}
      </h1>
    </AbsoluteFill>
  );
};

const ContentSlide: React.FC<{ content: string; index: number }> = ({
  content,
  index,
}) => {
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div style={{ fontSize: 48, color: 'white', lineHeight: 1.5 }}>
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

## Full Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlRecentNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render-video';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  content: string;
  videoPath?: string;
}> {
  // Step 1: Research
  console.log(`🔍 Crawling news for keyword: ${config.keyword}`);
  const newsData = await crawlRecentNews(config.keyword, '24h');
  const researchData = newsData.map(news => 
    `${news.title}: ${news.content.substring(0, 200)}...`
  );

  // Step 2: Generate Content
  console.log(`✍️ Generating ${config.format} content in ${config.language}`);
  const content = await generateContent({
    topic: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData,
  });

  // Step 3: Generate Video (optional)
  let videoPath: string | undefined;
  if (config.generateVideo && config.platform) {
    console.log(`🎬 Rendering video for ${config.platform}`);
    
    const slides = extractKeyPoints(content, 5);
    videoPath = `./output/video-${Date.now()}.mp4`;
    
    await renderContentVideo(
      {
        title: config.keyword,
        content: slides,
        bgColor: '#1a1a1a',
        platform: config.platform,
      },
      videoPath
    );
  }

  return {
    content,
    videoPath,
  };
}

function extractKeyPoints(content: string, count: number): string[] {
  const paragraphs = content.split('\n\n').filter(p => p.trim().length > 0);
  return paragraphs.slice(0, count).map(p => p.trim());
}
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      platform: body.platform || 'reels',
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      {
        success: false,
        error: error instanceof Error ? error.message : 'Unknown error',
      },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development
pnpm dev                 # Start Next.js dev server on localhost:3000

# Build
pnpm build              # Build for production
pnpm start              # Start production server

# Remotion
pnpm remotion preview   # Preview Remotion video compositions
pnpm remotion render    # Render video manually

# Type checking
pnpm type-check         # Run TypeScript compiler
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false,
      })
    )
  );
  
  return results;
}
```

### Multilingual Output

```typescript
// Generate same content in both languages
async function generateBilingual(keyword: string) {
  const [english, vietnamese] = await Promise.all([
    generateContent({
      topic: keyword,
      format: 'how-to',
      language: 'en',
      tone: 'friendly',
      researchData: [],
    }),
    generateContent({
      topic: keyword,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
      researchData: [],
    }),
  ]);

  return { english, vietnamese };
}
```

### Custom Video Templates

```typescript
// src/remotion/index.ts - Register compositions
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Sample Title',
          slides: ['Slide 1', 'Slide 2', 'Slide 3'],
          bgColor: '#000000',
        }}
      />
    </>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic for API calls
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos in chunks for large batches
async function renderInBatches(
  configs: VideoConfig[],
  batchSize = 3
) {
  const results: string[] = [];
  
  for (let i = 0; i < configs.length; i += batchSize) {
    const batch = configs.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((config, idx) =>
        renderContentVideo(config, `./output/video-${i + idx}.mp4`)
      )
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Environment Variables Not Loading

```typescript
// Validate environment on startup
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
```

## Performance Optimization

```typescript
// Cache news data to reduce API calls
import NodeCache from 'node-cache';

const newsCache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedNews(keyword: string) {
  const cacheKey = `news:${keyword}`;
  const cached = newsCache.get<NewsSource[]>(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const news = await crawlRecentNews(keyword);
  newsCache.set(cacheKey, news);
  
  return news;
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline, enabling AI agents to help developers automate content research, generation, and video rendering with Claude, OpenAI, and Remotion.
