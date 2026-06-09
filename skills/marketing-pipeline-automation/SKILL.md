---
name: marketing-pipeline-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion for multilingual content creation
triggers:
  - automate content creation pipeline
  - generate video from article content
  - scrape news for content research
  - create multilingual marketing content
  - build automated content workflow
  - generate social media videos automatically
  - research and write content with AI
  - setup content automation system
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a TypeScript-based automated content creation system that handles research scraping, AI content generation (Claude/OpenAI), and video rendering (Remotion) in a single workflow.

## What This Project Does

Marketing Pipeline Share automates the entire content creation process:

1. **Auto-Scan Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
2. **AI Content Generation**: Uses Claude 3 or OpenAI to generate multilingual content (English/Vietnamese) in various formats (Toplist, POV, Case Study, How-to)
3. **Video Rendering**: Automatically converts written content into infographics and short videos using Remotion
4. **Multi-Platform Export**: Generates content optimized for Reels, TikTok, Shorts

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

# Setup environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_REGION=us-east-1
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI provider integrations
│   │   ├── scraper/     # Content scraping logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript types
│   └── config/          # Configuration files
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Scraping Module

```typescript
import { ContentScraper } from '@/lib/scraper/content-scraper';

// Initialize scraper
const scraper = new ContentScraper({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Scrape content
const scrapedData = await scraper.scrape();

// Example response structure
interface ScrapedContent {
  title: string;
  url: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

// Initialize with Claude
const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content
const article = await generator.generate({
  topic: 'AI Marketing Trends 2024',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'both', // 'en' | 'vi' | 'both'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  sources: scrapedData,
  wordCount: 1500
});

// Article structure
interface GeneratedArticle {
  title: {
    en: string;
    vi: string;
  };
  content: {
    en: string;
    vi: string;
  };
  metadata: {
    format: string;
    tone: string;
    keywords: string[];
  };
}
```

### 3. Alternative AI Provider (OpenAI)

```typescript
import { OpenAIGenerator } from '@/lib/ai/openai-generator';

const openAIGen = new OpenAIGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

const content = await openAIGen.generateContent({
  prompt: 'Create a marketing case study about...',
  maxTokens: 2000,
  temperature: 0.7
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions/article-video';

// Render video from article
const videoConfig = {
  composition: VideoComposition,
  inputProps: {
    title: article.title.en,
    content: article.content.en,
    style: 'modern',
    duration: 60, // seconds
    aspectRatio: '9:16' // for Reels/TikTok
  },
  outputPath: './output/video.mp4'
};

const video = await renderVideo(videoConfig);
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Setup full pipeline
const pipeline = new ContentPipeline({
  scraper: {
    sources: ['techcrunch', 'linkedin'],
    keywords: ['SaaS', 'growth marketing']
  },
  generator: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  },
  video: {
    enabled: true,
    formats: ['reels', 'tiktok', 'shorts']
  }
});

// Execute pipeline
const result = await pipeline.execute({
  topic: 'SaaS Growth Strategies',
  format: 'how-to',
  languages: ['en', 'vi']
});

// Result structure
interface PipelineResult {
  article: GeneratedArticle;
  videos: {
    reels: string; // video URL
    tiktok: string;
    shorts: string;
  };
  metadata: {
    executionTime: number;
    tokensUsed: number;
    sources: number;
  };
}
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// src/app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { topic, format, languages } = await req.json();
  
  const pipeline = new ContentPipeline({
    scraper: { sources: ['techcrunch', 'linkedin'] },
    generator: { 
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY 
    }
  });
  
  const result = await pipeline.execute({
    topic,
    format,
    languages
  });
  
  return NextResponse.json(result);
}
```

### Video Render Endpoint

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { articleId, aspectRatio } = await req.json();
  
  const video = await renderVideo({
    articleId,
    aspectRatio,
    outputPath: `./output/${articleId}.mp4`
  });
  
  return NextResponse.json({ 
    videoUrl: video.url,
    duration: video.duration 
  });
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
import { BatchProcessor } from '@/lib/batch-processor';

const topics = [
  'AI in Marketing',
  'Content Automation',
  'Video Marketing Trends'
];

const batchProcessor = new BatchProcessor({
  generator: contentGenerator,
  scraper: contentScraper,
  concurrency: 3
});

const results = await batchProcessor.processTopics(topics, {
  format: 'toplist',
  languages: ['en', 'vi']
});

// Process results
results.forEach((result, index) => {
  console.log(`Topic: ${topics[index]}`);
  console.log(`Articles generated: ${result.success}`);
});
```

### Pattern 2: Custom Video Templates

```typescript
// remotion/compositions/custom-template.tsx
import { Composition } from 'remotion';

export const CustomVideoTemplate: React.FC<{
  title: string;
  points: string[];
  brand: {
    logo: string;
    colors: string[];
  };
}> = ({ title, points, brand }) => {
  return (
    <div style={{ 
      width: 1080, 
      height: 1920,
      background: brand.colors[0]
    }}>
      <h1>{title}</h1>
      {points.map((point, i) => (
        <div key={i}>{point}</div>
      ))}
    </div>
  );
};
```

### Pattern 3: Content Scheduling

```typescript
import { ScheduleManager } from '@/lib/schedule-manager';

const scheduler = new ScheduleManager();

// Schedule content generation
await scheduler.schedule({
  topic: 'Weekly Marketing Tips',
  frequency: 'weekly',
  dayOfWeek: 'monday',
  time: '09:00',
  config: {
    format: 'toplist',
    languages: ['en', 'vi'],
    generateVideo: true
  }
});
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# In another terminal, start Remotion studio
npm run remotion

# Build for production
npm run build
npm start
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  timeWindow: 60000 // 1 minute
});

const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
  rateLimiter: limiter
});
```

### Issue: Scraping Failures

```typescript
import { ScraperWithRetry } from '@/lib/scraper/retry-scraper';

const scraper = new ScraperWithRetry({
  maxRetries: 3,
  backoff: 'exponential',
  onError: (error, attempt) => {
    console.log(`Retry ${attempt} failed:`, error.message);
  }
});
```

### Issue: Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering
const video = await renderVideo({
  composition: VideoComposition,
  inputProps: videoProps,
  chromiumOptions: {
    headless: true,
    args: ['--disable-web-security', '--no-sandbox']
  },
  concurrency: 2, // Reduce for lower memory
  frameRange: [0, 300] // Limit frames if needed
});
```

### Issue: TypeScript Type Errors

```typescript
// Define custom types
import type { ContentFormat, Language, Tone } from '@/types';

interface ContentConfig {
  topic: string;
  format: ContentFormat;
  language: Language | Language[];
  tone: Tone;
  sources?: ScrapedContent[];
}

// Use type guards
function isValidFormat(format: string): format is ContentFormat {
  return ['toplist', 'pov', 'case-study', 'how-to'].includes(format);
}
```

## Advanced Configuration

### Multi-Provider AI Setup

```typescript
import { AIProviderRouter } from '@/lib/ai/router';

const router = new AIProviderRouter({
  providers: [
    { 
      name: 'claude', 
      apiKey: process.env.ANTHROPIC_API_KEY,
      priority: 1 
    },
    { 
      name: 'openai', 
      apiKey: process.env.OPENAI_API_KEY,
      priority: 2 
    }
  ],
  fallback: true
});

// Automatically uses Claude, falls back to OpenAI if needed
const content = await router.generate({ topic: '...' });
```

### Custom Scraping Sources

```typescript
import { CustomScraper } from '@/lib/scraper/custom';

const scraper = new CustomScraper();

scraper.addSource({
  name: 'custom-blog',
  url: 'https://example.com/blog',
  selectors: {
    title: '.post-title',
    content: '.post-content',
    date: '.publish-date'
  },
  rateLimit: 1000 // ms between requests
});
```

This skill covers the essential functionality for automating content creation pipelines with AI-powered research, generation, and video rendering capabilities.
