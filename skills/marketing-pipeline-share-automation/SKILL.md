---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated content pipeline with research and video
  - generate content from keyword to video automatically
  - use Claude and OpenAI for content automation
  - create automated marketing content workflow
  - build AI-powered content generation system
  - automate video generation from text content
  - set up Remotion video rendering pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms keywords into fully-researched articles and videos. It crawls fresh data from news sources (TechCrunch, a16z, Twitter/X, LinkedIn), generates content in multiple formats using Claude 3 or OpenAI, and automatically renders videos using Remotion.

**Key capabilities:**
- Auto-research from real-time data sources (last 24h)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual output (English/Vietnamese)
- Automatic video rendering for social media
- Next.js-based UI for workflow management

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI model integrations
│   │   ├── crawlers/    # News source crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript types
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Initialize AI Client

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Claude client
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// OpenAI client
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});
```

### 2. Research & Content Crawling

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research';

interface ResearchParams {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
}

async function conductResearch(params: ResearchParams) {
  const crawler = new ResearchCrawler({
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await crawler.scan({
    keyword: params.keyword,
    sources: params.sources,
    timeframe: params.timeframe,
  });

  // Returns aggregated insights
  return {
    articles: results.articles,
    insights: results.insights,
    dataPoints: results.dataPoints,
    trends: results.trends,
  };
}

// Usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z'],
  timeframe: '24h',
});
```

### 3. Generate Content with AI

```typescript
import { ContentGenerator } from '@/lib/generators/content';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(config: ContentConfig) {
  const generator = new ContentGenerator({
    model: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const content = await generator.create({
    format: config.format,
    language: config.language,
    tone: config.tone,
    context: config.researchData,
  });

  return content;
}

// Example: Generate a toplist in English
const article = await generateContent({
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research,
});
```

### 4. Claude Integration (Detailed)

```typescript
async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Context: ${context}\n\nTask: ${prompt}`,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate bilingual content
async function generateBilingual(researchData: any) {
  const englishPrompt = `
    Create a professional article about ${researchData.keyword}.
    Format: ${researchData.format}
    Include: ${researchData.insights.join(', ')}
  `;

  const vietnamesePrompt = `
    Tạo bài viết chuyên nghiệp về ${researchData.keyword}.
    Định dạng: ${researchData.format}
    Bao gồm: ${researchData.insights.join(', ')}
  `;

  const [english, vietnamese] = await Promise.all([
    generateWithClaude(englishPrompt, JSON.stringify(researchData)),
    generateWithClaude(vietnamesePrompt, JSON.stringify(researchData)),
  ]);

  return { english, vietnamese };
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/Composition';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const outputPath = `./output/${config.title.replace(/\s+/g, '-')}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    ...dimensions[config.format],
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  content: article.english,
  title: 'Top 5 AI Automation Tools',
  format: 'reels',
});
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline();

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
  });

  // Step 2: Generate content (both languages)
  const content = await pipeline.generate({
    format: 'toplist',
    researchData: research,
    languages: ['en', 'vi'],
    tone: 'expert',
  });

  // Step 3: Render videos
  const videos = await pipeline.renderVideos({
    content: content.english,
    formats: ['reels', 'tiktok', 'shorts'],
  });

  // Step 4: Save to database (optional)
  await pipeline.save({
    keyword,
    research,
    content,
    videos,
  });

  return {
    articles: content,
    videoUrls: videos,
    insights: research.insights,
  };
}

// Execute pipeline
const result = await runContentPipeline('AI content automation');
console.log('Articles generated:', result.articles);
console.log('Videos rendered:', result.videoUrls);
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video
npm run render:video
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  const crawler = new ResearchCrawler({
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await crawler.scan({
    keyword,
    sources,
    timeframe,
  });

  return NextResponse.json(results);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { format, language, tone, researchData } = await request.json();

  const generator = new ContentGenerator({
    model: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const content = await generator.create({
    format,
    language,
    tone,
    context: researchData,
  });

  return NextResponse.json({ content });
}
```

## TypeScript Types

```typescript
// types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type VideoFormat = 'reels' | 'tiktok' | 'shorts';

export interface ResearchResult {
  keyword: string;
  articles: Article[];
  insights: string[];
  dataPoints: DataPoint[];
  trends: Trend[];
}

export interface Article {
  title: string;
  content: string;
  language: Language;
  format: ContentFormat;
  metadata: {
    wordCount: number;
    readingTime: number;
    tone: Tone;
  };
}

export interface VideoOutput {
  path: string;
  format: VideoFormat;
  duration: number;
  dimensions: {
    width: number;
    height: number;
  };
}
```

## Troubleshooting

### AI API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function batchGenerate(prompts: string[]) {
  const tasks = prompts.map(prompt =>
    limit(() => generateWithClaude(prompt, ''))
  );
  
  return Promise.all(tasks);
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrent renders
const renderLimit = pLimit(1); // Render one video at a time

async function renderVideosSequentially(configs: VideoConfig[]) {
  const tasks = configs.map(config =>
    renderLimit(() => renderContentVideo(config))
  );
  
  return Promise.all(tasks);
}
```

### Crawler Timeout Handling

```typescript
async function crawlWithRetry(params: ResearchParams, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await conductResearch(params);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Environment Variable Validation

```typescript
// lib/config/validate.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use streaming responses** for long-form content generation
4. **Implement retry logic** for all external API calls
5. **Monitor token usage** to stay within budget limits
6. **Queue video rendering** to prevent memory exhaustion
7. **Store generated content** with metadata for analytics
