---
name: ultimate-ai-content-pipeline
description: Automated content creation system that researches, generates multi-format content in multiple languages, and renders videos using AI (Claude/OpenAI) and Remotion
triggers:
  - "create automated content pipeline with AI"
  - "generate video content from text automatically"
  - "research and write articles with Claude"
  - "build multi-language content automation"
  - "setup remotion video rendering for content"
  - "automate social media content creation"
  - "create AI-powered marketing content system"
  - "generate infographic videos from articles"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An automated content creation pipeline that handles the entire workflow from research to video generation. The system crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates multi-format content (toplist, POV, case study, how-to) in multiple languages using Claude/OpenAI, and automatically renders videos and infographics using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (last 24h)
- **AI Content Generation**: Creates diverse content formats in English and Vietnamese with customizable tone
- **Video Rendering**: Automatically generates infographics and short-form videos from articles
- **Multi-Platform**: Exports videos optimized for Reels, TikTok, Shorts with proper aspect ratios
- **Next.js Frontend**: Clean UI for managing content creation and scheduling

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

# Setup environment variables
cp .env.example .env.local
```

## Configuration

Create `.env.local` with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling utilities
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsArticles } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function researchTopic(keyword: string) {
  // Crawl recent articles
  const articles = await crawlNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Analyze and extract insights
  const insights = await analyzeContent(articles, {
    model: 'claude-3-opus',
    extractDataPoints: true
  });

  return {
    articles,
    insights,
    timestamp: new Date().toISOString()
  };
}
```

### 2. Content Generation with AI

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import { generateContent } from '@/lib/ai/content-generator';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function createArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const content = await generateContent({
    client: anthropic,
    topic,
    format,
    language,
    tone: 'professional', // or 'friendly', 'humorous'
    includeDataBacking: true,
    researchData: await researchTopic(topic)
  });

  return content;
}

// Generate bilingual content
async function generateBilingualContent(topic: string) {
  const [englishVersion, vietnameseVersion] = await Promise.all([
    createArticle(topic, 'toplist', 'en'),
    createArticle(topic, 'toplist', 'vi')
  ]);

  return { englishVersion, vietnameseVersion };
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { InfoGraphicVideo } from '@/remotion/InfoGraphic';

async function renderContentVideo(
  content: any,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: './remotion/index.tsx',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'InfoGraphic',
    inputProps: {
      content,
      theme: 'modern',
      duration: 30
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}-${platform}.mp4`,
    ...dimensions[platform]
  });
}
```

### 5. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true,
    platforms: ['reels', 'tiktok']
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    steps: [
      'research',      // Crawl news
      'analyze',       // Extract insights
      'generate',      // Create content
      'render-video',  // Generate videos
      'optimize'       // SEO optimization
    ]
  });

  return {
    articles: result.articles,
    videos: result.videos,
    metadata: result.metadata
  };
}
```

## Common Patterns

### News Crawler Setup

```typescript
import axios from 'axios';

interface CrawlerConfig {
  rapidApiKey: string;
  sources: string[];
  maxArticles: number;
}

class NewsCrawler {
  private config: CrawlerConfig;

  constructor(config: CrawlerConfig) {
    this.config = config;
  }

  async crawl(query: string, timeRange: string = '24h') {
    const options = {
      method: 'GET',
      url: 'https://news-api14.p.rapidapi.com/search',
      params: {
        q: query,
        language: 'en',
        sortBy: 'publishedAt',
        pageSize: this.config.maxArticles,
        from: this.getTimeRangeDate(timeRange)
      },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
      }
    };

    const response = await axios.request(options);
    return response.data.articles;
  }

  private getTimeRangeDate(range: string): string {
    const now = new Date();
    const hours = parseInt(range);
    now.setHours(now.getHours() - hours);
    return now.toISOString();
  }
}
```

### Content Format Templates

```typescript
interface ContentTemplate {
  format: string;
  structure: string[];
  aiPrompt: string;
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    format: 'toplist',
    structure: ['intro', 'items', 'conclusion'],
    aiPrompt: 'Create a ranked list article with data-backed rankings'
  },
  pov: {
    format: 'pov',
    structure: ['hook', 'argument', 'evidence', 'counterpoint', 'conclusion'],
    aiPrompt: 'Write an opinion piece with strong perspective and evidence'
  },
  caseStudy: {
    format: 'case-study',
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    aiPrompt: 'Analyze a real-world example with metrics and outcomes'
  },
  howTo: {
    format: 'how-to',
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    aiPrompt: 'Create a step-by-step guide with actionable advice'
  }
};
```

### Batch Content Generation

```typescript
async function batchGenerateContent(
  keywords: string[],
  options: {
    formats: string[];
    languages: string[];
    concurrent?: number;
  }
) {
  const { formats, languages, concurrent = 3 } = options;

  const tasks = keywords.flatMap(keyword =>
    formats.flatMap(format =>
      languages.map(language => ({
        keyword,
        format,
        language
      }))
    )
  );

  // Process in batches to avoid rate limits
  const results = [];
  for (let i = 0; i < tasks.length; i += concurrent) {
    const batch = tasks.slice(i, i + concurrent);
    const batchResults = await Promise.all(
      batch.map(task => 
        createArticle(task.keyword, task.format as any, task.language as any)
      )
    );
    results.push(...batchResults);
    
    // Wait between batches
    if (i + concurrent < tasks.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }

  return results;
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;

    const content = await generateContent({
      topic: keyword,
      format,
      language,
      tone
    });

    return NextResponse.json({
      success: true,
      content,
      generatedAt: new Date().toISOString()
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { contentId, platform, content } = body;

    const videoPath = await renderContentVideo(content, platform);

    return NextResponse.json({
      success: true,
      videoUrl: videoPath,
      platform
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video rendering failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Usage (if available)

```bash
# Generate content
npm run generate -- --keyword "AI trends" --format toplist --lang en

# Render video
npm run render -- --input content.json --platform reels

# Run full pipeline
npm run pipeline -- --keyword "Web3" --all-formats --video
```

## Remotion Video Templates

```typescript
// remotion/InfoGraphic.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const InfoGraphicVideo: React.FC<{
  content: any;
  theme: string;
  duration: number;
}> = ({ content, theme, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      <Sequence from={0} durationInFrames={90}>
        <Title text={content.title} />
      </Sequence>
      
      <Sequence from={90} durationInFrames={180}>
        <DataPoints points={content.keyPoints} />
      </Sequence>
      
      <Sequence from={270} durationInFrames={90}>
        <CallToAction text={content.cta} />
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function robustAPICall<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  return pRetry(fn, {
    retries: maxRetries,
    onFailedAttempt: (error) => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    },
    minTimeout: 2000,
    maxTimeout: 10000
  });
}
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion configuration for large videos
export const remotionConfig = {
  composition: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900
  },
  rendering: {
    concurrency: 2, // Reduce if running out of memory
    imageFormat: 'jpeg',
    jpegQuality: 80
  }
};
```

### Claude/OpenAI Timeout Handling

```typescript
async function generateWithTimeout(
  generateFn: () => Promise<any>,
  timeoutMs: number = 60000
) {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Generation timeout')), timeoutMs)
  );

  return Promise.race([generateFn(), timeoutPromise]);
}
```

### Error Recovery in Pipeline

```typescript
class Pipeline {
  async executeWithRecovery(steps: string[]) {
    const results = { completed: [], failed: [] };
    
    for (const step of steps) {
      try {
        const result = await this.executeStep(step);
        results.completed.push({ step, result });
      } catch (error) {
        console.error(`Step ${step} failed:`, error);
        results.failed.push({ step, error: error.message });
        
        // Continue with remaining steps unless critical
        if (this.isCriticalStep(step)) {
          throw new Error(`Critical step ${step} failed: ${error.message}`);
        }
      }
    }
    
    return results;
  }
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Lint code
npm run lint
```

## Performance Tips

- Use batch processing for multiple content pieces
- Implement caching for research data (24h TTL)
- Queue video rendering jobs to avoid memory spikes
- Use streaming responses for long AI generations
- Implement webhook callbacks for async operations
