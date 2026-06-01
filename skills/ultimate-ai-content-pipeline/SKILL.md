---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content pipeline with AI research, script generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up the marketing pipeline for content generation
  - create AI-powered content from research to video
  - use Claude and OpenAI for automated content pipeline
  - generate videos automatically from content scripts
  - build automated content workflow with Remotion
  - crawl news sources and generate marketing content
  - automate multilingual content creation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive content automation system that combines web crawling, AI-powered content generation (Claude/OpenAI), and automated video rendering (Remotion). It transforms keywords into complete content pieces with research, scripts, and videos in both English and Vietnamese.

## What It Does

- **Auto-Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multilingual Output**: Generates parallel Vietnamese and English versions
- **Video Rendering**: Automatically creates infographics and short videos optimized for Reels/TikTok/Shorts
- **Next.js Interface**: Web UI for easy content creation and scheduling

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
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video rendering
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
│   │   ├── crawler/     # Web scraping modules
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/           # Remotion video templates
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

### 1. Research & Crawling

```typescript
// src/lib/crawler/news-crawler.ts
import { CrawlerService } from './crawler-service';

interface CrawlConfig {
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  keywords: string[];
}

async function crawlContent(config: CrawlConfig) {
  const crawler = new CrawlerService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await crawler.fetchFromMultipleSources({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    keywords: config.keywords,
    timeRange: config.timeRange,
  });

  return results;
}

// Usage
const newsData = await crawlContent({
  sources: ['techcrunch', 'a16z'],
  timeRange: '24h',
  keywords: ['AI', 'marketing automation'],
});
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });

  const systemPrompt = buildSystemPrompt(config.format, config.tone);
  const researchContext = formatResearchData(config.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Language: ${config.language}\n\nResearch Data:\n${researchContext}\n\nGenerate content based on this research.`,
      },
    ],
  });

  return message.content[0].text;
}

function buildSystemPrompt(format: string, tone: string): string {
  const prompts = {
    toplist: 'You are a content creator specializing in top 10 lists and rankings.',
    pov: 'You are an industry thought leader sharing unique perspectives.',
    'case-study': 'You are a business analyst creating detailed case studies.',
    'how-to': 'You are an educational content creator writing step-by-step guides.',
  };

  return `${prompts[format]} Write in a ${tone} tone with data-backed insights.`;
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

async function generateContentOpenAI(config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(config.format, config.tone),
      },
      {
        role: 'user',
        content: `Language: ${config.language}\n\nCreate content from: ${JSON.stringify(config.researchData)}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
// src/lib/video/render-service.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'infographic';
}

async function renderContentVideo(config: VideoConfig) {
  const compositionId = `${config.style}-${config.platform}`;
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      platform: config.platform,
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
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      platform: config.platform,
    },
  });

  return outputPath;
}
```

### 5. Complete Pipeline

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlContent } from '../crawler/news-crawler';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-service';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('vi' | 'en')[];
  generateVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Crawling research data...');
  const research = await crawlContent({
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    keywords: [config.keyword],
  });

  const results = [];

  // Step 2: Generate content for each language
  for (const lang of config.languages) {
    console.log(`✍️ Generating ${lang.toUpperCase()} content...`);
    
    const content = await generateContent({
      format: config.format,
      language: lang,
      tone: 'expert',
      researchData: research,
    });

    const result: any = {
      language: lang,
      content,
      research: research.slice(0, 5), // Top 5 sources
    };

    // Step 3: Generate video if requested
    if (config.generateVideo) {
      console.log(`🎬 Rendering video for ${lang.toUpperCase()}...`);
      
      const videoPath = await renderContentVideo({
        content,
        platform: 'reels',
        style: 'infographic',
      });

      result.video = videoPath;
    }

    results.push(result);
  }

  return results;
}

// Usage example
const output = await runContentPipeline({
  keyword: 'AI Marketing Automation',
  format: 'toplist',
  languages: ['vi', 'en'],
  generateVideo: true,
});
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, languages, generateVideo } = body;

    const results = await runContentPipeline({
      keyword,
      format,
      languages,
      generateVideo,
    });

    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error) {
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

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await runContentPipeline({
          keyword,
          format: 'toplist',
          languages: ['vi', 'en'],
          generateVideo: false,
        });
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return null;
      }
    })
  );

  return results.filter(Boolean);
}
```

### Scheduled Content Generation

```typescript
// Using node-cron or similar
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      format: 'pov',
      languages: ['vi'],
      generateVideo: true,
    });
  }
});
```

### Custom Remotion Video Template

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import { z } from 'zod';

export const infographicSchema = z.object({
  content: z.string(),
  platform: z.enum(['reels', 'tiktok', 'shorts']),
});

export const Infographic: React.FC<z.infer<typeof infographicSchema>> = ({
  content,
  platform,
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <h1 style={{ color: 'white', fontSize: 48, textAlign: 'center' }}>
        {content}
      </h1>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

  async add<T>(fn: () => Promise<T>, delayMs = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
        await new Promise((r) => setTimeout(r, delayMs));
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
    }
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.add(() => generateContent(config), 2000);
```

### Error Handling for AI APIs

```typescript
async function generateContentWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (error.status === 429) {
        // Rate limit - wait exponentially
        await new Promise((r) => setTimeout(r, Math.pow(2, i) * 1000));
        continue;
      }
      if (i === maxRetries - 1) throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Render videos sequentially to avoid memory issues
async function renderMultipleVideos(contents: string[]) {
  const videos = [];
  
  for (const content of contents) {
    const video = await renderContentVideo({
      content,
      platform: 'reels',
      style: 'minimal',
    });
    
    videos.push(video);
    
    // Clear cache between renders
    if (global.gc) {
      global.gc();
    }
  }
  
  return videos;
}
```

## Key Configuration Files

**next.config.js**:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverActions: true,
  },
  env: {
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
    RAPIDAPI_KEY: process.env.RAPIDAPI_KEY,
  },
};

module.exports = nextConfig;
```

**tsconfig.json** path aliases:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/components/*": ["./src/components/*"]
    }
  }
}
```
