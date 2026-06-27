---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - automate content creation from research to video
  - use Claude and OpenAI for content automation
  - create AI-powered marketing content pipeline
  - generate videos from AI-written content automatically
  - build automated content workflow with Remotion
  - scrape news and generate content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end automated content creation system that researches trending topics, generates content in multiple formats (articles, scripts), and renders videos/graphics automatically. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion for video rendering.

## What It Does

This project automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in various formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates Vietnamese and English content simultaneously
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # News crawling & research
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research News Sources

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function getLatestNews(topic: string) {
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    dataSources: research.sources
  };
}
```

### 2. Generate Content with AI

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    provider: 'claude', // 'claude' | 'openai'
    includeResearch: true
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    translations: content.translations
  };
}
```

### 3. Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    system: 'You are an expert content creator specializing in marketing and tech trends.'
  });

  return message.content[0].text;
}
```

### 4. Using OpenAI API

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech trends.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });

  return completion.choices[0].message.content;
}
```

### 5. Generate Video with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function generateVideo(content: any) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      body: content.body,
      style: 'modern' // 'modern' | 'minimal' | 'dynamic'
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.inputProps,
  });

  return `out/${content.id}.mp4`;
}
```

### 6. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    languages: ['en', 'vi']
  });

  // Step 1: Research
  const research = await pipeline.research(keyword, {
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  });

  // Step 2: Generate content
  const content = await pipeline.generateContent(research, {
    format: 'toplist',
    tone: 'professional'
  });

  // Step 3: Create video
  const video = await pipeline.generateVideo(content, {
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    aspectRatio: '9:16'
  });

  return {
    content,
    video,
    research
  };
}
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { topic, sources } = await request.json();

  const results = await researchTopic({
    keyword: topic,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: '24h'
  });

  return NextResponse.json(results);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, provider } = await request.json();

  const content = await generateContent({
    topic,
    format: format || 'toplist',
    language: language || ['en'],
    provider: provider || 'claude'
  });

  return NextResponse.json(content);
}
```

### Video Generation Endpoint

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { contentId, platform } = await request.json();

  const videoUrl = await renderVideo({
    contentId,
    platform: platform || 'reels',
    aspectRatio: platform === 'youtube' ? '16:9' : '9:16'
  });

  return NextResponse.json({ videoUrl });
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion preview (for video templates)
npm run remotion

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      const research = await researchTopic({ keyword: topic });
      const content = await generateContent({
        topic,
        format: 'toplist',
        language: ['en', 'vi']
      });
      return { topic, content, research };
    })
  );

  return results;
}
```

### Custom Content Format

```typescript
interface CustomFormat {
  structure: string[];
  tone: string;
  wordCount: number;
}

async function generateCustomFormat(
  topic: string,
  customFormat: CustomFormat
) {
  const prompt = `
    Create content about ${topic} with the following structure:
    ${customFormat.structure.join('\n')}
    
    Tone: ${customFormat.tone}
    Target word count: ${customFormat.wordCount}
  `;

  return await generateWithClaude(prompt);
}
```

### Multi-Platform Video Export

```typescript
async function exportMultiPlatform(contentId: string) {
  const platforms = [
    { name: 'reels', ratio: '9:16', duration: 60 },
    { name: 'youtube', ratio: '16:9', duration: 120 },
    { name: 'tiktok', ratio: '9:16', duration: 60 }
  ];

  const videos = await Promise.all(
    platforms.map(async (platform) => ({
      platform: platform.name,
      url: await renderVideo({
        contentId,
        aspectRatio: platform.ratio,
        maxDuration: platform.duration
      })
    }))
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for large videos
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(composition: any) {
  const { assetsInfo } = await renderFrames({
    composition,
    onStart: () => console.log('Rendering started'),
    onFrameUpdate: (frame) => {
      if (frame % 10 === 0) {
        console.log(`Rendered frame ${frame}`);
      }
    },
    concurrency: 2, // Limit concurrent rendering
  });

  return assetsInfo;
}
```

### Handling Research Failures

```typescript
async function safeResearch(topic: string) {
  try {
    return await researchTopic({ keyword: topic });
  } catch (error) {
    console.error('Research failed:', error);
    // Fallback to AI-only content generation
    return {
      articles: [],
      insights: `Generated without real-time data for: ${topic}`,
      sources: []
    };
  }
}
```

### Environment Variable Validation

```typescript
function validateConfig() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call on app initialization
validateConfig();
```
