---
name: marketing-pipeline-auto-content
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the marketing content automation pipeline
  - automate content creation from research to video
  - generate articles and videos automatically with AI
  - use marketing pipeline share for content automation
  - set up AI content research and video rendering
  - configure automated content generation with Claude
  - create automated social media content pipeline
  - build content automation with Remotion video rendering
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates content creation from research through article generation to video production. The pipeline uses Claude/OpenAI for content generation and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
- **Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion License (optional, for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering server (Remotion)
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research/crawling
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core APIs and Usage

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Auto-research a topic from latest sources
const researchData = await researchTopic({
  keyword: 'AI marketing trends',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  language: 'en'
});

console.log(researchData.insights);
// Returns: { articles: [...], trends: [...], statistics: [...] }
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate content from research data
async function generateArticle(researchData: any, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Based on this research data: ${JSON.stringify(researchData)}
      
      Create a ${format} article in Vietnamese with:
      - Engaging headline
      - Data-backed insights
      - Actionable takeaways
      - SEO optimization`
    }]
  });

  return message.content[0].text;
}

// Usage
const article = await generateArticle(researchData, 'toplist');
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(topic: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert. Write in ${tone} tone.`
      },
      {
        role: 'user',
        content: `Create a comprehensive article about: ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

// Render video from article content
async function renderContentVideo(articleData: any) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: articleData.title,
      content: articleData.content,
      style: 'modern',
      duration: 60, // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleData.slug}.mp4`,
    inputProps: composition.props,
  });

  return `out/${articleData.slug}.mp4`;
}
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateArticle } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h',
      language: 'en'
    });

    // Step 2: Generate Article
    console.log('✍️ Generating article...');
    const article = await generateArticle({
      research,
      format: 'case-study',
      language: 'vi',
      tone: 'professional',
      includeStats: true
    });

    // Step 3: Create Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: article.title,
      content: article.content,
      format: 'reels', // or 'tiktok', 'youtube-shorts'
      aspectRatio: '9:16'
    });

    return {
      article,
      videoPath,
      publishReady: true
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI in marketing 2024');
console.log('✅ Content ready:', result);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  const data = await researchTopic({
    keyword,
    sources,
    timeframe,
  });

  return NextResponse.json(data);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

export async function POST(request: NextRequest) {
  const { research, format, language, tone } = await request.json();

  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create ${format} content in ${language} with ${tone} tone based on: ${JSON.stringify(research)}`
    }]
  });

  return NextResponse.json({
    content: message.content[0].text,
    usage: message.usage
  });
}
```

### Video Render Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { articleData, format } = await request.json();

  const videoPath = await renderContentVideo({
    ...articleData,
    format
  });

  return NextResponse.json({
    success: true,
    videoPath,
    downloadUrl: `/api/download/${videoPath}`
  });
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(research: any) {
  const languages = ['en', 'vi'];
  const results = {};

  for (const lang of languages) {
    results[lang] = await generateArticle({
      research,
      language: lang,
      format: 'how-to',
      tone: 'friendly'
    });
  }

  return results;
}
```

### Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const pipeline = keywords.map(async (keyword) => {
    return await runContentPipeline(keyword);
  });

  // Process in parallel with limit
  const results = await Promise.all(pipeline);
  return results;
}
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: string;
  videoPath: string;
  publishDate: Date;
  platforms: string[];
}

async function scheduleContent(keyword: string, publishDate: Date) {
  const result = await runContentPipeline(keyword);

  const scheduled: ScheduledContent = {
    content: result.article.content,
    videoPath: result.videoPath,
    publishDate,
    platforms: ['facebook', 'tiktok', 'youtube']
  };

  // Save to database or scheduling system
  await saveToScheduler(scheduled);

  return scheduled;
}
```

## Configuration Options

### Research Configuration

```typescript
interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '6h' | '12h' | '24h' | '7d';
  language: 'en' | 'vi';
  maxResults?: number;
  includeImages?: boolean;
}
```

### Content Generation Configuration

```typescript
interface ContentConfig {
  research: any;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous' | 'expert';
  wordCount?: number;
  includeStats?: boolean;
  seoOptimize?: boolean;
}
```

### Video Rendering Configuration

```typescript
interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'youtube-shorts' | 'standard';
  aspectRatio: '16:9' | '9:16' | '1:1' | '4:5';
  duration?: number;
  style?: 'modern' | 'minimal' | 'vibrant';
  includeSubtitles?: boolean;
  backgroundMusic?: string;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys before running
function validateApiKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'),
});

async function rateLimitedGenerate(content: any) {
  const { success } = await ratelimit.limit('content-gen');
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please wait.');
  }

  return await generateArticle(content);
}
```

### Video Rendering Errors

```typescript
// Handle video rendering failures gracefully
async function safeRenderVideo(articleData: any) {
  try {
    return await renderContentVideo(articleData);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback: create static image instead
    return await createStaticImage(articleData);
  }
}
```

### Memory Management for Large Batches

```typescript
// Process large batches with concurrency control
import pLimit from 'p-limit';

async function processManyKeywords(keywords: string[]) {
  const limit = pLimit(3); // Max 3 concurrent operations

  const tasks = keywords.map(keyword => 
    limit(() => runContentPipeline(keyword))
  );

  return await Promise.allSettled(tasks);
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement error handling** for all API calls
3. **Cache research data** to avoid redundant API calls
4. **Use TypeScript types** for better code safety
5. **Monitor API usage** to stay within rate limits
6. **Test video rendering** locally before production
7. **Validate input data** before passing to AI models
8. **Store generated content** in a database for reuse
