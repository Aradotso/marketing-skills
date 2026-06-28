---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "create automated content pipeline with AI"
  - "generate video content from text automatically"
  - "research and write content using AI agents"
  - "automate content creation workflow"
  - "build AI-powered marketing content system"
  - "create video content with Remotion and AI"
  - "set up content automation pipeline"
  - "generate multilingual content with Claude"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content creation pipeline: from web scraping/research, AI-powered content generation (Claude 3, OpenAI), to automated video rendering with Remotion. Supports multiple content formats, bilingual output (English/Vietnamese), and multi-platform video optimization.

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

Create a `.env.local` file in the project root:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI (for web scraping/research)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Database (if using persistence)
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities and services
│   ├── ai/               # AI service integrations
│   ├── research/         # Content research/scraping
│   └── video/            # Remotion video generation
├── remotion/             # Remotion compositions
└── public/               # Static assets
```

## Core Features & Usage

### 1. Content Research (Auto-Scan)

The system automatically crawls and analyzes fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn.

```typescript
// lib/research/scanner.ts
import { scanLatestNews } from '@/lib/research/scanner';

interface ResearchParams {
  keyword: string;
  sources?: string[];
  timeRange?: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

async function performResearch(params: ResearchParams) {
  const results = await scanLatestNews({
    keyword: params.keyword,
    sources: params.sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: params.timeRange || '24h',
  });

  return {
    articles: results.articles,
    insights: results.insights,
    trends: results.trends,
    dataPoints: results.dataPoints,
  };
}

// Example usage
const aiResearch = await performResearch({
  keyword: 'artificial intelligence',
  timeRange: '24h',
  language: 'en',
});

console.log(`Found ${aiResearch.articles.length} recent articles`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI.

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: any[];
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(config: ContentConfig) {
    const prompt = this.buildPrompt(config);

    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return this.parseResponse(message.content);
  }

  async generateWithOpenAI(config: ContentConfig) {
    const prompt = this.buildPrompt(config);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in data-backed marketing content.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
    });

    return completion.choices[0].message.content;
  }

  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with compelling points',
      'pov': 'Write from a unique perspective or viewpoint',
      'case-study': 'Analyze a specific example with data and outcomes',
      'how-to': 'Provide step-by-step instructional content',
    };

    return `
Format: ${formatInstructions[config.format]}
Tone: ${config.tone}
Language: ${config.language}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Generate comprehensive, engaging content that incorporates the latest insights and data points.
`;
  }

  private parseResponse(content: any) {
    // Parse and structure the AI response
    return {
      title: '',
      body: '',
      keyPoints: [],
      metadata: {},
    };
  }
}

// Example usage
const generator = new ContentGenerator();

const article = await generator.generateWithClaude({
  format: 'toplist',
  tone: 'expert',
  language: 'both',
  researchData: aiResearch.articles,
});
```

### 3. Bilingual Content Generation

```typescript
// lib/ai/bilingual-generator.ts
interface BilingualContent {
  en: {
    title: string;
    content: string;
    summary: string;
  };
  vi: {
    title: string;
    content: string;
    summary: string;
  };
}

async function generateBilingualContent(
  keyword: string,
  researchData: any[]
): Promise<BilingualContent> {
  const generator = new ContentGenerator();

  const [enContent, viContent] = await Promise.all([
    generator.generateWithClaude({
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData,
    }),
    generator.generateWithClaude({
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData,
    }),
  ]);

  return {
    en: enContent,
    vi: viContent,
  };
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: {
    title: string;
    points: string[];
    images?: string[];
  };
  format: 'reels' | 'tiktok' | 'shorts' | 'square';
  duration?: number;
}

const VIDEO_FORMATS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  square: { width: 1080, height: 1080, fps: 30 },
};

async function generateVideo(config: VideoConfig) {
  const format = VIDEO_FORMATS[config.format];
  const compositionId = 'ContentVideo';

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      points: config.content.points,
      images: config.content.images || [],
    },
  });

  // Render video
  const outputLocation = path.resolve(`./output/video-${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.content.title,
      points: config.content.points,
    },
  });

  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  content: {
    title: 'Top 5 AI Trends in 2024',
    points: [
      'Multimodal AI integration',
      'AI agents in enterprise',
      'Open source LLM growth',
      'AI regulation frameworks',
      'Edge AI deployment',
    ],
  },
  format: 'reels',
});
```

### 5. Remotion Video Composition

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  images?: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  images,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const titleScale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

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
          opacity: titleOpacity,
          transform: `scale(${titleScale})`,
          color: '#fff',
          fontSize: 80,
          fontWeight: 'bold',
          textAlign: 'center',
          padding: 40,
        }}
      >
        {title}
      </div>

      {/* Render points with animation */}
      {points.map((point, index) => {
        const startFrame = 60 + index * 90;
        const opacity = interpolate(
          frame,
          [startFrame, startFrame + 20],
          [0, 1],
          { extrapolateRight: 'clamp' }
        );

        return (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: `${30 + index * 15}%`,
              opacity,
              color: '#fff',
              fontSize: 40,
              padding: 20,
            }}
          >
            {index + 1}. {point}
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

### 6. Complete Content Pipeline

```typescript
// lib/pipeline/content-pipeline.ts
import { performResearch } from '@/lib/research/scanner';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/generator';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`🔍 Step 1: Researching "${config.keyword}"...`);
  const research = await performResearch({
    keyword: config.keyword,
    timeRange: '24h',
  });

  console.log(`✍️ Step 2: Generating content...`);
  const generator = new ContentGenerator();
  const content = await generator.generateWithClaude({
    format: config.format,
    tone: config.tone,
    language: 'both',
    researchData: research.articles,
  });

  let videoPath = null;
  if (config.includeVideo) {
    console.log(`🎬 Step 3: Rendering video...`);
    videoPath = await generateVideo({
      content: {
        title: content.title,
        points: content.keyPoints,
      },
      format: config.videoFormat || 'reels',
    });
  }

  return {
    research: {
      articlesFound: research.articles.length,
      insights: research.insights,
    },
    content: {
      en: content.en,
      vi: content.vi,
    },
    video: videoPath,
    metadata: {
      generatedAt: new Date().toISOString(),
      keyword: config.keyword,
    },
  };
}

// Example: Complete pipeline execution
const result = await runContentPipeline({
  keyword: 'AI automation tools 2024',
  format: 'toplist',
  tone: 'expert',
  includeVideo: true,
  videoFormat: 'reels',
});

console.log('Pipeline complete:', result);
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, includeVideo, videoFormat } = body;

    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      includeVideo,
      videoFormat,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video (if separate)
npm run remotion:render
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron-job.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

// Run daily at 8 AM
cron.schedule('0 8 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content creation'];

  for (const keyword of keywords) {
    await runContentPipeline({
      keyword,
      format: 'toplist',
      tone: 'expert',
      includeVideo: true,
      videoFormat: 'reels',
    });
  }
});
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'expert',
        includeVideo: false,
      })
    )
  );

  const successful = results.filter((r) => r.status === 'fulfilled');
  const failed = results.filter((r) => r.status === 'rejected');

  return { successful, failed };
}
```

### Pattern 3: Custom Content Templates

```typescript
// lib/templates/custom-template.ts
interface ContentTemplate {
  sections: {
    intro: string;
    mainPoints: string[];
    conclusion: string;
  };
  style: {
    headingFormat: string;
    bulletStyle: string;
  };
}

function applyTemplate(content: any, template: ContentTemplate) {
  return {
    formatted: `
${template.style.headingFormat}${content.title}

${template.sections.intro}

${content.keyPoints.map((p, i) => `${template.style.bulletStyle} ${p}`).join('\n')}

${template.sections.conclusion}
    `,
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
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
    const task = this.queue.shift();

    if (task) {
      await task();
      await new Promise((resolve) => setTimeout(resolve, 1000)); // 1s delay
    }

    this.processing = false;
    this.process();
  }
}
```

### Video Rendering Issues

If Remotion fails to render:

```bash
# Install required system dependencies
sudo apt-get install -y chromium-browser

# Set Chromium path
export PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
```

### Memory Issues with Large Content

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

### Claude API Errors

```typescript
// lib/utils/retry.ts
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise((resolve) => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generator.generateWithClaude(config)
);
```

## Performance Optimization

```typescript
// Enable caching for research results
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const results = await performResearch({ keyword });
  await redis.setex(cacheKey, 3600, JSON.stringify(results)); // 1 hour cache

  return results;
}
```
