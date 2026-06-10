---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation from research to video
  - generate AI-powered content with automatic research
  - create marketing content pipeline with Claude and OpenAI
  - build automated content workflow with video generation
  - set up AI content automation system
  - integrate Claude for content research and generation
  - automate social media content creation with AI
  - use Remotion for automated video rendering
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from automated research and data crawling, to AI-powered script generation, and finally video rendering. It integrates Claude 3, OpenAI, and Remotion to create a complete content factory.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (listicles, POV, case studies, how-tos)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video and infographic rendering via Remotion
- Next.js frontend for easy content scheduling and publishing

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

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LAMBDA_SECRET=your_remotion_secret

# Database (if applicable)
DATABASE_URL=your_database_url

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
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Automated Research Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherTopicData(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}

// Usage
const data = await gatherTopicData('AI marketing automation');
console.log(data.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en'
) {
  const prompt = `Based on this research data: ${JSON.stringify(research)}
  
Create a ${format} article in ${language} language.
Include data-backed insights, current trends, and actionable takeaways.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Usage
const content = await generateContent(
  researchData,
  'toplist',
  'en'
);
```

### 3. OpenAI Alternative Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  research: any,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing.`
      },
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(research)}`
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
import path from 'path';

async function renderContentVideo(
  content: string,
  outputPath: string
) {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content,
      duration: 30, // seconds
      format: 'vertical' // for Reels/TikTok
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}

// Usage
await renderContentVideo(
  generatedContent,
  'output/video.mp4'
);
```

### 5. Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

async function createFullContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    includeVideo: boolean;
  }
) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent(
    research,
    options.format,
    options.language
  );

  // Step 3: Render Video (optional)
  let videoPath = null;
  if (options.includeVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(
      content,
      `output/${keyword}-${Date.now()}.mp4`
    );
  }

  return {
    content,
    research,
    videoPath,
    metadata: {
      keyword,
      format: options.format,
      language: options.language,
      createdAt: new Date().toISOString()
    }
  };
}

// Usage
const result = await createFullContentPipeline('AI content automation', {
  format: 'how-to',
  language: 'en',
  includeVideo: true
});
```

### 6. Multi-Language Content Generation

```typescript
interface BilingualContent {
  en: string;
  vi: string;
}

async function generateBilingualContent(
  research: any,
  format: string
): Promise<BilingualContent> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, format, 'en'),
    generateContent(research, format, 'vi')
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}

// Usage
const bilingualPost = await generateBilingualContent(
  researchData,
  'toplist'
);
```

### 7. Web Crawler for News Sources

```typescript
import axios from 'axios';

interface CrawlerConfig {
  source: string;
  apiEndpoint: string;
  timeframe: string;
}

async function crawlNewsSource(config: CrawlerConfig) {
  const response = await axios.get(config.apiEndpoint, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
    },
    params: {
      q: config.source,
      timeframe: config.timeframe
    }
  });

  return response.data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    summary: article.description,
    source: config.source
  }));
}

// Usage
const techCrunchNews = await crawlNewsSource({
  source: 'techcrunch',
  apiEndpoint: 'https://api.example.com/news',
  timeframe: '24h'
});
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

# Render video (Remotion)
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview
```

## API Routes (Next.js)

### Generate Content API

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: Request) {
  const { keyword, format, language } = await request.json();

  try {
    const research = await researchTopic({ keyword });
    const content = await generateContent(research, format, language);

    return NextResponse.json({
      success: true,
      content,
      research
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering API

```typescript
// app/api/render-video/route.ts
import { NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/remotion';

export async function POST(request: Request) {
  const { content, format } = await request.json();

  try {
    const videoPath = await renderContentVideo(content, format);

    return NextResponse.json({
      success: true,
      videoUrl: `/output/${videoPath}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Custom AI Provider

```typescript
// lib/ai/config.ts
interface AIConfig {
  provider: 'claude' | 'openai';
  model: string;
  temperature: number;
  maxTokens: number;
}

export const aiConfig: AIConfig = {
  provider: process.env.AI_PROVIDER || 'claude',
  model: process.env.AI_MODEL || 'claude-3-5-sonnet-20241022',
  temperature: parseFloat(process.env.AI_TEMPERATURE || '0.7'),
  maxTokens: parseInt(process.env.AI_MAX_TOKENS || '4096')
};
```

### Video Output Configuration

```typescript
// lib/video/config.ts
export const videoConfig = {
  formats: {
    horizontal: { width: 1920, height: 1080 },
    vertical: { width: 1080, height: 1920 },
    square: { width: 1080, height: 1080 }
  },
  codecs: ['h264', 'h265', 'vp8'],
  fps: 30,
  outputDir: 'public/output'
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay = 1000; // 1 second between requests

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
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    this.processing = false;
  }
}

export const rateLimiter = new RateLimiter();
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export class ContentPipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: any
  ) {
    super(message);
    this.name = 'ContentPipelineError';
  }
}

export async function withErrorHandling<T>(
  fn: () => Promise<T>,
  stage: string
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error(`Error in ${stage}:`, error);
    throw new ContentPipelineError(
      `Failed at ${stage} stage`,
      stage as any,
      error
    );
  }
}
```

### Memory Management for Large Videos

```typescript
// lib/video/optimize.ts
export async function renderWithMemoryManagement(
  content: string,
  options: any
) {
  // Split long content into chunks
  const chunks = splitIntoChunks(content, 500);
  const videos: string[] = [];

  for (const chunk of chunks) {
    const videoPath = await renderContentVideo(chunk, options);
    videos.push(videoPath);
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
  }

  return videos;
}
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** when calling external APIs
3. **Cache research results** to avoid redundant API calls
4. **Use TypeScript types** for better IDE support and error catching
5. **Test video rendering** on small samples before full production
6. **Monitor token usage** to control AI API costs
7. **Implement retry logic** for failed API calls
8. **Store generated content** with metadata for tracking and reuse
