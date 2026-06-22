---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multi-format writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I use the AI content pipeline to generate articles
  - set up automated content research and video generation
  - create content from keywords using Claude and OpenAI
  - generate videos from articles with Remotion
  - automate content pipeline from research to video
  - configure the marketing content automation system
  - use the content pipeline for multi-language posts
  - troubleshoot AI content generation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content creation system that researches trending topics, generates articles in multiple formats and languages, and renders videos automatically. It integrates Claude 3, OpenAI, web scraping APIs, and Remotion to create a complete content production pipeline.

## What It Does

- **Auto-Research**: Crawls and analyzes data from TechCrunch, a16z, Twitter/X, LinkedIn for recent trending content
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese simultaneously
- **Video Generation**: Automatically renders infographics and short videos using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key

# Content Database
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

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
│   │   ├── research/    # Web scraping and research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Generate Content from Keyword

```typescript
import { ContentPipeline } from '@/lib/pipeline';
import { ResearchEngine } from '@/lib/research';
import { AIWriter } from '@/lib/ai/writer';

async function generateContent(keyword: string) {
  // Initialize the pipeline
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    format: 'toplist',
  });

  // Step 1: Research
  const researchData = await pipeline.research(keyword, {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 10,
  });

  // Step 2: Generate article
  const article = await pipeline.generateArticle({
    keyword,
    research: researchData,
    tone: 'professional', // 'friendly', 'humorous'
    format: 'toplist',
  });

  // Step 3: Generate video (optional)
  const video = await pipeline.generateVideo({
    content: article,
    platform: 'reels', // 'tiktok', 'shorts'
    aspectRatio: '9:16',
  });

  return { article, video, researchData };
}
```

### 2. Research Module

```typescript
import { ResearchEngine } from '@/lib/research';
import { analyzeInsights } from '@/lib/research/analyzer';

async function performResearch(topic: string) {
  const engine = new ResearchEngine({
    apiKey: process.env.RAPIDAPI_KEY!,
  });

  // Fetch from multiple sources
  const [techcrunch, twitter, linkedin] = await Promise.all([
    engine.fetchTechCrunch(topic, { hours: 24 }),
    engine.fetchTwitter(topic, { maxTweets: 50 }),
    engine.fetchLinkedIn(topic, { maxPosts: 30 }),
  ]);

  // Analyze and extract insights
  const insights = await analyzeInsights({
    sources: [techcrunch, twitter, linkedin],
    aiProvider: 'claude',
  });

  return {
    rawData: { techcrunch, twitter, linkedin },
    insights,
    trends: insights.trends,
    statistics: insights.stats,
  };
}
```

### 3. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { formatPrompt } from '@/lib/ai/prompts';

async function generateWithClaude(
  keyword: string,
  research: any,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    tone: string;
  }
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = formatPrompt({
    type: options.format,
    keyword,
    research,
    language: options.language,
    tone: options.tone,
  });

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

  return {
    content: message.content[0].text,
    metadata: {
      model: 'claude-3-5-sonnet',
      tokens: message.usage.input_tokens + message.usage.output_tokens,
    },
  };
}
```

### 4. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';
import { formatPrompt } from '@/lib/ai/prompts';

async function generateWithOpenAI(
  keyword: string,
  research: any,
  options: any
) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = formatPrompt({
    type: options.format,
    keyword,
    research,
    language: options.language,
    tone: options.tone,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator and marketer.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });

  return {
    content: completion.choices[0].message.content,
    metadata: {
      model: completion.model,
      tokens: completion.usage?.total_tokens,
    },
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(article: any, config: any) {
  // Prepare video data
  const videoData = {
    title: article.title,
    keyPoints: article.keyPoints || [],
    statistics: article.statistics || [],
    branding: config.branding || {},
  };

  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: videoData,
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: videoData,
  });

  return {
    path: outputPath,
    duration: composition.durationInFrames / composition.fps,
    resolution: {
      width: composition.width,
      height: composition.height,
    },
  };
}
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
  });

  try {
    // 1. Research phase
    console.log('Starting research...');
    const research = await pipeline.research('AI automation', {
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
    });

    // 2. Content generation for multiple formats
    console.log('Generating content...');
    const formats = ['toplist', 'how-to'] as const;
    const articles = await Promise.all(
      formats.map((format) =>
        pipeline.generateArticle({
          keyword: 'AI automation',
          research,
          format,
          tone: 'professional',
        })
      )
    );

    // 3. Video generation
    console.log('Generating videos...');
    const videos = await Promise.all(
      articles.map((article) =>
        pipeline.generateVideo({
          content: article,
          platform: 'reels',
          aspectRatio: '9:16',
        })
      )
    );

    // 4. Save to database
    await pipeline.saveContent({
      articles,
      videos,
      research,
      keyword: 'AI automation',
    });

    console.log('Pipeline completed successfully!');
    return { articles, videos, research };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      languages: languages || ['en', 'vi'],
    });

    const result = await pipeline.run({
      keyword,
      format: format || 'toplist',
      generateVideo: true,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error: any) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
      },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchEngine } from '@/lib/research';

export async function POST(request: NextRequest) {
  try {
    const { topic, sources, timeframe } = await request.json();

    const engine = new ResearchEngine({
      apiKey: process.env.RAPIDAPI_KEY!,
    });

    const results = await engine.research(topic, {
      sources: sources || ['techcrunch', 'twitter', 'linkedin'],
      timeframe: timeframe || '24h',
    });

    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error: any) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
      },
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

# Run Remotion studio (for video editing)
npm run remotion:studio
```

## Configuration Options

### Content Formats

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

const formatConfig = {
  toplist: {
    structure: 'numbered list',
    minItems: 5,
    maxItems: 10,
  },
  pov: {
    structure: 'opinion-based',
    perspective: 'first-person',
  },
  'case-study': {
    structure: 'problem-solution',
    includeStats: true,
  },
  'how-to': {
    structure: 'step-by-step',
    includeVisuals: true,
  },
};
```

### Tone Options

```typescript
type ToneOption = 'professional' | 'friendly' | 'humorous' | 'expert';

const toneSettings = {
  professional: {
    formality: 'high',
    jargon: 'moderate',
  },
  friendly: {
    formality: 'low',
    jargon: 'minimal',
  },
  humorous: {
    formality: 'low',
    jargon: 'minimal',
    includeJokes: true,
  },
  expert: {
    formality: 'high',
    jargon: 'high',
    citations: true,
  },
};
```

### Video Platform Settings

```typescript
const platformSettings = {
  reels: {
    aspectRatio: '9:16',
    maxDuration: 90,
    fps: 30,
  },
  tiktok: {
    aspectRatio: '9:16',
    maxDuration: 60,
    fps: 30,
  },
  shorts: {
    aspectRatio: '9:16',
    maxDuration: 60,
    fps: 30,
  },
  landscape: {
    aspectRatio: '16:9',
    maxDuration: 300,
    fps: 30,
  },
};
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(keyword: string) {
  const languages = ['en', 'vi'];
  const results = {};

  for (const lang of languages) {
    const content = await generateWithClaude(keyword, research, {
      format: 'toplist',
      language: lang,
      tone: 'professional',
    });

    results[lang] = content;
  }

  return results;
}
```

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
  });

  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await pipeline.run({
        keyword,
        format: 'toplist',
        generateVideo: true,
      });
      results.push({ keyword, success: true, data: result });
    } catch (error: any) {
      results.push({ keyword, success: false, error: error.message });
    }

    // Rate limiting
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Custom Remotion Composition

```typescript
// remotion/compositions/CustomVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomVideo: React.FC<{
  title: string;
  keyPoints: string[];
}> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#000', padding: 60 }}>
      <h1 style={{ color: '#fff', opacity }}>{title}</h1>
      <ul>
        {keyPoints.map((point, i) => (
          <li
            key={i}
            style={{
              color: '#fff',
              opacity: frame > (i + 1) * fps ? 1 : 0,
              transition: 'opacity 0.3s',
            }}
          >
            {point}
          </li>
        ))}
      </ul>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError: any;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      lastError = error;

      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }

  throw lastError;
}
```

### Video Rendering Errors

```typescript
// Handle video rendering failures gracefully
async function safeRenderVideo(article: any, config: any) {
  try {
    return await generateVideo(article, config);
  } catch (error: any) {
    console.error('Video rendering failed:', error);

    // Fallback to image generation
    return await generateStaticImage(article, config);
  }
}
```

### Memory Management for Large Batches

```typescript
// Process in chunks to avoid memory issues
async function processInChunks<T, R>(
  items: T[],
  processor: (item: T) => Promise<R>,
  chunkSize = 5
): Promise<R[]> {
  const results: R[] = [];

  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);

    // Clear memory between chunks
    if (global.gc) {
      global.gc();
    }
  }

  return results;
}
```

### Invalid API Keys

Always check environment variables are loaded:

```typescript
function validateEnv() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

// Call at app startup
validateEnv();
```
