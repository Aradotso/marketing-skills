---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - create marketing content from keyword to video automatically
  - use Claude and OpenAI for content automation
  - build automated content workflow with Remotion
  - generate bilingual content with AI research crawling
  - automate social media content creation with video rendering
  - set up AI content pipeline for Vietnamese and English
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A TypeScript-based automated content creation system that handles the complete workflow: research crawling from news sources, AI-powered content generation in multiple formats and languages, and automatic video/image rendering for social media platforms.

## What It Does

This pipeline automates:
- **Research crawling** from TechCrunch, a16z, Twitter/X, LinkedIn for fresh data
- **Multi-format content generation** (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Bilingual output** (English & Vietnamese) with customizable tone
- **Automatic video rendering** using Remotion for Reels, TikTok, Shorts
- **End-to-end workflow** from keyword input to publishable content

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

### Environment Variables

Create a `.env` file with the following:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Project Structure

```
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawlers/          # Research crawlers
│   ├── generators/        # Content generators
│   └── remotion/          # Video rendering
├── public/                # Static assets
└── types/                 # TypeScript definitions
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Render videos (Remotion)
npm run remotion:render
```

## Core API Usage

### 1. Research Crawler Integration

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h'
});

// Crawl for a specific topic
const researchData = await crawler.crawl({
  keyword: 'AI marketing automation',
  maxResults: 20,
  filterDuplicates: true
});

// Example output structure
interface ResearchResult {
  title: string;
  source: string;
  url: string;
  publishedAt: Date;
  content: string;
  insights: string[];
  dataPoints: string[];
}
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';

// Initialize with Claude
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const generator = new ContentGenerator({
  aiClient: anthropic,
  model: 'claude-3-5-sonnet-20241022'
});

// Generate content
const content = await generator.generate({
  keyword: 'AI content automation',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  language: 'vi', // or 'en'
  tone: 'professional', // or 'friendly', 'humorous'
  includeResearch: researchData
});

// Example output
interface GeneratedContent {
  title: string;
  introduction: string;
  mainContent: string[];
  conclusion: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    seoKeywords: string[];
  };
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';
import { ContentGenerator } from '@/lib/generators/content-generator';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const generator = new ContentGenerator({
  aiClient: openai,
  model: 'gpt-4-turbo-preview'
});

// Same API as Claude version
const content = await generator.generate({
  keyword: 'marketing automation',
  format: 'case-study',
  language: 'en',
  tone: 'expert'
});
```

### 4. Remotion Video Rendering

```typescript
import { VideoComposition } from '@/lib/remotion/compositions';
import { renderMedia } from '@remotion/renderer';

// Define video composition
const composition: VideoComposition = {
  id: 'ContentVideo',
  component: ContentVideoTemplate,
  durationInFrames: 300, // 10 seconds at 30fps
  fps: 30,
  width: 1080,
  height: 1920, // Vertical for Reels/TikTok/Shorts
  props: {
    title: content.title,
    points: content.mainContent.slice(0, 5),
    branding: {
      logo: '/logo.png',
      colors: {
        primary: '#FF6B6B',
        secondary: '#4ECDC4'
      }
    }
  }
};

// Render video
const videoPath = await renderMedia({
  composition,
  serveUrl: 'http://localhost:3000',
  outputLocation: `public/videos/${Date.now()}.mp4`,
  codec: 'h264',
});
```

## Common Patterns

### Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Run full pipeline
const result = await pipeline.execute({
  keyword: 'AI marketing trends 2026',
  outputs: {
    article: {
      formats: ['toplist', 'pov'],
      languages: ['en', 'vi'],
      tone: 'professional'
    },
    video: {
      aspectRatio: '9:16', // Vertical
      duration: 30,
      style: 'infographic'
    }
  }
});

// Result contains
interface PipelineResult {
  research: ResearchResult[];
  articles: GeneratedContent[];
  videos: {
    path: string;
    thumbnail: string;
    duration: number;
  }[];
  metadata: {
    processTime: number;
    totalWords: number;
    aiCost: number;
  };
}
```

### Batch Content Generation

```typescript
import { BatchProcessor } from '@/lib/processors/batch';

const processor = new BatchProcessor();

const keywords = [
  'AI marketing automation',
  'Content creation tools',
  'Social media trends'
];

// Process multiple keywords
const batchResults = await processor.processBatch({
  keywords,
  parallelLimit: 3, // Process 3 at a time
  onProgress: (progress) => {
    console.log(`Progress: ${progress.completed}/${progress.total}`);
  }
});
```

### Customizing Content Format

```typescript
import { FormatTemplate } from '@/lib/generators/templates';

// Create custom format template
const customFormat: FormatTemplate = {
  name: 'myth-busting',
  structure: {
    introduction: {
      prompt: 'Write an engaging hook about common myths in {keyword}',
      maxLength: 200
    },
    myths: {
      prompt: 'List 5 common myths about {keyword} with explanations',
      format: 'numbered-list'
    },
    truth: {
      prompt: 'Provide evidence-based truth for each myth using research data',
      includeData: true
    },
    conclusion: {
      prompt: 'Summarize key takeaways and call-to-action',
      maxLength: 150
    }
  }
};

// Use custom format
const content = await generator.generate({
  keyword: 'AI content creation',
  customFormat,
  language: 'vi'
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: Request) {
  try {
    const { keyword, format, language } = await request.json();
    
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const result = await pipeline.execute({
      keyword,
      outputs: {
        article: {
          formats: [format],
          languages: [language]
        }
      }
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { ResearchCrawler } from '@/lib/crawlers/research-crawler';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();
  
  const crawler = new ResearchCrawler({
    sources,
    timeframe,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const data = await crawler.crawl({ keyword });
  
  return NextResponse.json({ research: data });
}
```

## TypeScript Types

```typescript
// types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'professional' | 'friendly' | 'humorous' | 'expert';

export interface ContentConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  includeResearch?: boolean;
  wordCount?: number;
  seoOptimized?: boolean;
}

export interface VideoConfig {
  aspectRatio: '16:9' | '9:16' | '1:1';
  duration: number; // seconds
  style: 'infographic' | 'talking-head' | 'text-animation';
  music?: string;
  voiceover?: boolean;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  anthropic: { requestsPerMinute: 50 },
  openai: { requestsPerMinute: 60 },
  rapidapi: { requestsPerMinute: 100 }
});

// Use with automatic retry
const content = await limiter.execute('anthropic', async () => {
  return await generator.generate(config);
});
```

### Error Handling

```typescript
import { ContentGenerationError } from '@/lib/errors';

try {
  const result = await pipeline.execute(config);
} catch (error) {
  if (error instanceof ContentGenerationError) {
    console.error('Generation failed:', error.stage, error.message);
    // Retry specific stage
    if (error.retryable) {
      await pipeline.retryStage(error.stage);
    }
  }
}
```

### Video Rendering Issues

```bash
# Check Remotion dependencies
npx remotion doctor

# Test render locally
npx remotion preview

# Render with verbose logging
npx remotion render --log=verbose
```

### Memory Management for Large Batches

```typescript
import { MemoryManager } from '@/lib/utils/memory';

const manager = new MemoryManager({
  maxConcurrent: 5,
  cleanupInterval: 1000 * 60 * 5 // 5 minutes
});

await manager.processQueue(keywords, async (keyword) => {
  const result = await pipeline.execute({ keyword });
  // Result is automatically cleaned up
  return result;
});
```

## Performance Tips

- Use Claude for Vietnamese content (better language support)
- Use GPT-4 for English content requiring creativity
- Enable caching for research data to avoid redundant crawls
- Render videos in parallel with content generation
- Use CDN for static assets in Remotion compositions
