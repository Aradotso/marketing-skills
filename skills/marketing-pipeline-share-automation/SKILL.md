---
name: marketing-pipeline-share-automation
description: Ultimate AI content pipeline for automated research, script generation, video creation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from blog posts automatically
  - crawl news and generate content
  - build automated content workflow
  - setup AI content generation system
  - generate multilingual marketing content
  - create social media videos from articles
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, a complete TypeScript-based content automation system that handles everything from research and script generation to automated video rendering and multi-platform publishing. The pipeline leverages Claude 3, OpenAI, and Remotion to transform keywords into fully-formed content pieces.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates English and Vietnamese versions simultaneously
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Create environment variables file
cp .env.example .env
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

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/newsScanner.ts
import { NewsSource } from '@/types/content';

interface CrawlConfig {
  sources: NewsSource[];
  timeRange: number; // hours
  keywords?: string[];
}

export async function scanRecentNews(config: CrawlConfig) {
  const { sources, timeRange, keywords } = config;
  
  const newsData = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(
        `https://api.rapidapi.com/news/${source}`,
        {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          },
        }
      );
      return response.json();
    })
  );

  return filterAndAnalyze(newsData, keywords, timeRange);
}

// Usage example
const insights = await scanRecentNews({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: 24,
  keywords: ['AI', 'marketing automation'],
});
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any;
}

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContent(request: ContentRequest) {
  const { keyword, format, language, tone, researchData } = request;

  // Build context-aware prompt
  const systemPrompt = buildSystemPrompt(format, language, tone);
  const userPrompt = buildUserPrompt(keyword, researchData);

  // Use Claude for long-form content
  const response = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return {
    content: response.content[0].text,
    metadata: extractMetadata(response),
  };
}

function buildSystemPrompt(
  format: string,
  language: string,
  tone: string
): string {
  const formatInstructions = {
    toplist: 'Create a numbered list article with clear rankings',
    pov: 'Write from a strong personal perspective',
    'case-study': 'Analyze a real-world example with data',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  return `You are a ${tone} ${language === 'vi' ? 'Vietnamese' : 'English'} content writer.
Format: ${formatInstructions[format]}
Use research data provided to back claims with evidence.
Include relevant statistics and examples.`;
}
```

### 3. Multi-language Content Generation

```typescript
// src/lib/ai/multilingualGenerator.ts
import { generateContent } from './contentGenerator';

export async function generateMultilingualContent(
  keyword: string,
  format: string,
  tone: string,
  researchData: any
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format,
      language: 'en',
      tone,
      researchData,
    }),
    generateContent({
      keyword,
      format,
      language: 'vi',
      tone,
      researchData,
    }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/videoGenerator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  style: 'infographic' | 'text-overlay' | 'animated';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
};

export async function generateVideo(config: VideoConfig) {
  const { title, content, platform, style } = config;
  const { width, height } = platformDimensions[platform];

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: style,
    inputProps: {
      title,
      content: parseContentForVideo(content),
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}

function parseContentForVideo(content: string) {
  // Extract key points for video scenes
  const points = content
    .split('\n')
    .filter((line) => line.trim().length > 0)
    .slice(0, 5); // Take first 5 key points

  return points.map((point, index) => ({
    scene: index + 1,
    text: point.trim(),
    duration: 3, // seconds per scene
  }));
}
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { scanRecentNews } from '../crawler/newsScanner';
import { generateMultilingualContent } from '../ai/multilingualGenerator';
import { generateVideo } from '../video/videoGenerator';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  const {
    keyword,
    format,
    tone,
    includeVideo,
    platforms = ['reels'],
  } = config;

  // Step 1: Research
  console.log('🔍 Scanning recent news...');
  const researchData = await scanRecentNews({
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: 24,
    keywords: [keyword],
  });

  // Step 2: Generate content
  console.log('✍️ Generating multilingual content...');
  const content = await generateMultilingualContent(
    keyword,
    format,
    tone,
    researchData
  );

  // Step 3: Generate videos (optional)
  let videos: { [key: string]: string } = {};
  if (includeVideo) {
    console.log('🎬 Rendering videos...');
    videos = await generateVideosForPlatforms(
      content.en.content,
      platforms
    );
  }

  return {
    content,
    research: researchData,
    videos,
    metadata: {
      keyword,
      format,
      generatedAt: new Date().toISOString(),
    },
  };
}

async function generateVideosForPlatforms(
  content: string,
  platforms: string[]
) {
  const videoPromises = platforms.map((platform) =>
    generateVideo({
      title: extractTitle(content),
      content,
      platform: platform as any,
      style: 'infographic',
    })
  );

  const videoPaths = await Promise.all(videoPromises);

  return platforms.reduce((acc, platform, index) => {
    acc[platform] = videoPaths[index];
    return acc;
  }, {} as { [key: string]: string });
}

function extractTitle(content: string): string {
  const firstLine = content.split('\n')[0];
  return firstLine.replace(/^#+\s*/, '').trim();
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, includeVideo, platforms } = body;

    // Validate input
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      includeVideo: includeVideo ?? false,
      platforms: platforms ?? ['reels'],
    });

    return NextResponse.json(result, { status: 200 });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: any) => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
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
    <div className="content-generator">
      {/* Form UI here */}
      {loading && <p>⏳ Generating content...</p>}
      {result && (
        <div>
          <h3>English Version:</h3>
          <div>{result.content.en.content}</div>
          <h3>Vietnamese Version:</h3>
          <div>{result.content.vi.content}</div>
          {result.videos && (
            <div>
              <h3>Generated Videos:</h3>
              {Object.entries(result.videos).map(([platform, path]) => (
                <video key={platform} src={path} controls />
              ))}
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Remotion Video Template Example

```typescript
// remotion/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface Props {
  title: string;
  content: Array<{ scene: number; text: string; duration: number }>;
}

export const Infographic: React.FC<Props> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Calculate which scene to show
  let currentScene = 0;
  let elapsedTime = 0;
  for (let i = 0; i < content.length; i++) {
    const sceneDuration = content[i].duration * fps;
    if (frame < elapsedTime + sceneDuration) {
      currentScene = i;
      break;
    }
    elapsedTime += sceneDuration;
  }

  const scene = content[currentScene];
  const sceneProgress =
    (frame - elapsedTime) / (scene.duration * fps);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <h1
        style={{
          color: 'white',
          fontSize: 80,
          textAlign: 'center',
          marginBottom: 40,
          opacity: Math.min(sceneProgress * 2, 1),
        }}
      >
        {title}
      </h1>
      <p
        style={{
          color: '#e0e0e0',
          fontSize: 48,
          textAlign: 'center',
          lineHeight: 1.5,
          transform: `translateY(${(1 - sceneProgress) * 50}px)`,
          opacity: sceneProgress,
        }}
      >
        {scene.text}
      </p>
      <div
        style={{
          position: 'absolute',
          bottom: 40,
          color: '#888',
          fontSize: 36,
        }}
      >
        {currentScene + 1} / {content.length}
      </div>
    </AbsoluteFill>
  );
}
```

## Configuration Examples

### Custom Tone Presets

```typescript
// src/config/tones.ts
export const tonePresets = {
  expert: {
    vocabulary: 'professional',
    sentenceStructure: 'complex',
    examples: 'industry-specific',
    citations: true,
  },
  friendly: {
    vocabulary: 'conversational',
    sentenceStructure: 'simple',
    examples: 'relatable',
    citations: false,
  },
  humorous: {
    vocabulary: 'casual',
    sentenceStructure: 'varied',
    examples: 'funny analogies',
    citations: false,
  },
};
```

### Content Format Templates

```typescript
// src/config/formats.ts
export const formatTemplates = {
  toplist: {
    structure: 'numbered',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true,
  },
  pov: {
    structure: 'opinion-driven',
    includeCounterarguments: true,
    personalExamples: true,
  },
  'case-study': {
    structure: 'problem-solution-results',
    requiresData: true,
    sections: ['background', 'challenge', 'solution', 'results'],
  },
  'how-to': {
    structure: 'step-by-step',
    minSteps: 3,
    includePrerequisites: true,
    includeTips: true,
  },
};
```

## Common Patterns

### Error Handling with Retry Logic

```typescript
// src/utils/retry.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((resolve) =>
        setTimeout(resolve, delay * (i + 1))
      );
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generateContent({ keyword: 'AI', format: 'toplist', language: 'en', tone: 'expert' })
);
```

### Caching Research Data

```typescript
// src/lib/cache/researchCache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached;
}

export async function cacheResearch(
  keyword: string,
  data: any,
  ttl = 86400
) {
  await redis.setex(`research:${keyword}`, ttl, JSON.stringify(data));
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add rate limiting middleware
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
});

// Apply to API routes
app.use('/api/', limiter);
```

### Video Rendering Failures

If Remotion videos fail to render:

```bash
# Ensure ffmpeg is installed
npm install -g @remotion/lambda

# Check Remotion CLI
npx remotion --version

# Test render locally
npx remotion render remotion/index.ts Infographic out.mp4
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, chunkSize = 2000) {
  const chunks = [];
  for (let i = 0; i < content.length; i += chunkSize) {
    chunks.push(content.slice(i, i + chunkSize));
  }
  return chunks;
}

// Generate content in batches
const chunks = chunkContent(largeContent);
const results = [];
for (const chunk of chunks) {
  const result = await processChunk(chunk);
  results.push(result);
}
```

### Environment Variable Issues

```bash
# Verify all required env vars are set
node -e "console.log(process.env.ANTHROPIC_API_KEY ? '✓ Claude API' : '✗ Missing Claude API')"
node -e "console.log(process.env.OPENAI_API_KEY ? '✓ OpenAI API' : '✗ Missing OpenAI API')"
```

## Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm run start

# Or deploy to Vercel
vercel deploy --prod
```

This skill provides comprehensive guidance for working with the Marketing Pipeline Share automation system, enabling AI agents to help developers implement complete content generation workflows.
