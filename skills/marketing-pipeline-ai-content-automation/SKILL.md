---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing content pipeline
  - generate videos from blog posts automatically
  - crawl news and create content with Claude
  - build AI-powered content automation workflow
  - create multi-format content with OpenAI and Remotion
  - automate social media content research and publishing
  - generate TikTok and Reels videos from articles
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that handles the complete workflow from research to video generation. It automatically crawls fresh news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, uses Claude 3 or OpenAI to generate multiple content formats (toplist, POV, case study, how-to), and renders videos using Remotion for social media platforms.

**Key capabilities:**
- Auto-scan research from real-time news sources
- Multi-format content generation (English + Vietnamese)
- Automatic video/infographic rendering for Reels, TikTok, Shorts
- Next.js frontend with API integrations
- Extensible architecture with Claude, OpenAI, and RapidAPI

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NODE_ENV=development
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utilities
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env                 # Environment variables
```

## Key Features & Usage

### 1. News Crawling & Research

```typescript
// src/lib/crawlers/news-scanner.ts
import { NewsScanner } from '@/lib/crawlers/news-scanner';

interface NewsSource {
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  keywords: string[];
  timeframe: '24h' | '7d' | '30d';
}

async function crawlNews(config: NewsSource) {
  const scanner = new NewsScanner({
    apiKey: process.env.RAPIDAPI_KEY!,
    sources: [config.source]
  });

  const articles = await scanner.scan({
    keywords: config.keywords,
    since: config.timeframe,
    limit: 20
  });

  return articles;
}

// Usage example
const techNews = await crawlNews({
  source: 'techcrunch',
  keywords: ['AI', 'marketing automation'],
  timeframe: '24h'
});
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
  research: any[];
}

export async function generateContent(request: ContentRequest) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const researchContext = request.research
    .map(r => `Title: ${r.title}\nSummary: ${r.summary}\nURL: ${r.url}`)
    .join('\n\n');

  const prompt = `Based on the following recent research:

${researchContext}

Create a ${request.format} article about "${request.topic}" in ${request.language} with a ${request.tone} tone.

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Format appropriately for ${request.format}
- Length: 800-1200 words
- Add engaging headlines and subheadings`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(request: ContentRequest) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} format with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: buildPrompt(request)
      }
    ],
    temperature: 0.7,
    max_tokens: 2500
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/generate-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reel' | 'tiktok' | 'short';
  thumbnail?: string;
}

export async function generateVideo(config: VideoConfig) {
  const dimensions = {
    reel: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    short: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: parseContentToScenes(config.content),
      thumbnail: config.thumbnail
    }
  });

  // Render video
  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });

  return outputPath;
}

function parseContentToScenes(content: string) {
  // Split content into scenes for video
  const paragraphs = content.split('\n\n').filter(p => p.trim());
  return paragraphs.slice(0, 5).map((text, index) => ({
    id: index,
    text: text.substring(0, 200),
    duration: 3 // seconds
  }));
}
```

### 5. Complete Pipeline Example

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlNews } from '@/lib/crawlers/news-scanner';
import { generateContent } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/generate-video';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormats?: ('reel' | 'tiktok' | 'short')[];
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  const research = await crawlNews({
    source: 'techcrunch',
    keywords: [config.keyword],
    timeframe: '24h'
  });

  console.log(`Found ${research.length} articles`);

  // Step 2: Generate content for each language
  const articles = await Promise.all(
    config.languages.map(async (language) => {
      const content = await generateContent({
        topic: config.keyword,
        format: config.contentFormat,
        language,
        tone: 'expert',
        research
      });

      return { language, content };
    })
  );

  // Step 3: Generate videos (optional)
  let videos = [];
  if (config.generateVideo && config.videoFormats) {
    for (const article of articles) {
      for (const format of config.videoFormats) {
        const videoPath = await generateVideo({
          title: config.keyword,
          content: article.content,
          format
        });
        
        videos.push({
          language: article.language,
          format,
          path: videoPath
        });
      }
    }
  }

  return {
    research,
    articles,
    videos
  };
}
```

## API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo || false,
      videoFormats: body.videoFormats || ['reel']
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// Frontend component example
async function generateContentClick() {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: 'AI marketing automation',
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      videoFormats: ['reel', 'tiktok']
    })
  });

  const result = await response.json();
  console.log(result.data);
}
```

## Remotion Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: Array<{ id: number; text: string; duration: number }>;
  thumbnail?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  thumbnail
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const currentSceneIndex = Math.floor(
    content.reduce((acc, scene, idx) => {
      const sceneEnd = acc + scene.duration * fps;
      return frame < sceneEnd ? acc : sceneEnd;
    }, 0) / (content[0]?.duration * fps || 1)
  );

  const currentScene = content[currentSceneIndex] || content[0];

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        color: 'white',
        fontSize: 48,
        textAlign: 'center',
        padding: 60
      }}>
        <h1>{title}</h1>
        <p style={{ fontSize: 32, marginTop: 40 }}>
          {currentScene?.text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Examples

### Multi-Source Research Config

```typescript
// config/research-sources.ts
export const researchConfig = {
  sources: [
    {
      name: 'techcrunch',
      endpoint: 'https://techcrunch-api.p.rapidapi.com/articles',
      priority: 1
    },
    {
      name: 'a16z',
      endpoint: 'https://a16z.com/feed',
      priority: 2
    }
  ],
  refreshInterval: 3600000, // 1 hour
  maxArticlesPerSource: 50
};
```

### Content Format Templates

```typescript
// config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    tone: 'opinionated'
  },
  'case-study': {
    structure: ['problem', 'solution', 'results', 'lessons'],
    requireStats: true
  },
  'how-to': {
    structure: ['overview', 'steps', 'tips', 'conclusion'],
    minSteps: 3
  }
};
```

## Common Patterns

### Rate Limiting for AI APIs

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      this.processQueue();
    });
  }

  private async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const task = this.queue.shift()!;
    
    await task();
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.processQueue();
  }
}

// Usage
const claudeLimiter = new RateLimiter(50); // 50 requests per minute

const content = await claudeLimiter.execute(() =>
  generateContent(request)
);
```

### Caching Research Results

```typescript
// src/lib/cache/research-cache.ts
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, any>({
  max: 100,
  ttl: 1000 * 60 * 60 // 1 hour
});

export async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  const research = await crawlNews({
    source: 'techcrunch',
    keywords: [keyword],
    timeframe: '24h'
  });

  cache.set(cacheKey, research);
  return research;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Video Rendering Failures

- Ensure Remotion license key is valid
- Check available disk space for output
- Verify ffmpeg is installed: `npx remotion doctor`
- Increase memory limit: `NODE_OPTIONS=--max-old-space-size=4096 npm run dev`

### Content Generation Timeout

```typescript
// Add timeout wrapper
async function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number
): Promise<T> {
  return Promise.race([
    promise,
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), timeoutMs)
    )
  ]);
}

// Usage
const content = await withTimeout(
  generateContent(request),
  30000 // 30 seconds
);
```

### Rate Limit Errors

- Implement exponential backoff
- Use the RateLimiter class shown above
- Monitor API usage quotas
- Consider using multiple API keys in rotation

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
vercel --prod
```

### Environment Variables for Production

Ensure all API keys are set in your deployment platform's environment configuration, not in code or `.env` files committed to git.
