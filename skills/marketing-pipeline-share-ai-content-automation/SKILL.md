---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content automation pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing content from keywords
  - create automated video content pipeline
  - build AI content workflow with Claude
  - set up automated content research system
  - generate videos from blog posts automatically
  - create multi-language marketing content
  - automate content from research to video
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms a single keyword into complete, multi-format content including research, scripts, blog posts, and videos. The system automatically crawls fresh data from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats and languages, finally rendering videos using Remotion.

**Key Capabilities:**
- Auto-scan research from real-time news sources (24h fresh data)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese) with tone customization
- Automatic video rendering with Remotion integration
- Next.js frontend for workflow management

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion:render
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── research/     # Auto-scan research modules
│   │   ├── ai/           # AI content generation
│   │   └── video/        # Remotion video rendering
│   ├── api/              # API routes
│   └── types/            # TypeScript types
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── config/               # Configuration files
```

## Core Modules

### 1. Research Auto-Scan

The research module crawls and analyzes fresh content from multiple sources.

```typescript
// src/lib/research/scanner.ts
import { TechCrunchScraper } from './scrapers/techcrunch';
import { TwitterScraper } from './scrapers/twitter';
import { LinkedInScraper } from './scrapers/linkedin';

interface ResearchResult {
  source: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
  insights: string[];
}

export class ResearchScanner {
  private scrapers: Array<any>;

  constructor() {
    this.scrapers = [
      new TechCrunchScraper(process.env.RAPIDAPI_KEY!),
      new TwitterScraper(process.env.TWITTER_BEARER_TOKEN!),
      new LinkedInScraper(process.env.LINKEDIN_ACCESS_TOKEN!)
    ];
  }

  async scanKeyword(keyword: string, hours: number = 24): Promise<ResearchResult[]> {
    const results: ResearchResult[] = [];
    
    for (const scraper of this.scrapers) {
      try {
        const data = await scraper.search(keyword, hours);
        results.push(...data);
      } catch (error) {
        console.error(`Scraper ${scraper.name} failed:`, error);
      }
    }

    return this.deduplicateAndRank(results);
  }

  private deduplicateAndRank(results: ResearchResult[]): ResearchResult[] {
    // Remove duplicates and rank by relevance
    const unique = new Map();
    
    results.forEach(item => {
      if (!unique.has(item.url) || 
          item.insights.length > unique.get(item.url).insights.length) {
        unique.set(item.url, item);
      }
    });

    return Array.from(unique.values())
      .sort((a, b) => b.insights.length - a.insights.length);
  }
}
```

### 2. AI Content Generation

Generate multi-format content using Claude or OpenAI.

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async generateContent(config: ContentConfig, provider: 'claude' | 'openai' = 'claude') {
    const prompt = this.buildPrompt(config);
    
    if (provider === 'claude') {
      return this.generateWithClaude(prompt);
    } else {
      return this.generateWithOpenAI(prompt);
    }
  }

  private buildPrompt(config: ContentConfig): string {
    const researchContext = config.research
      .map(r => `[${r.source}] ${r.title}\n${r.content}\nInsights: ${r.insights.join(', ')}`)
      .join('\n\n');

    return `You are a ${config.tone} content writer creating a ${config.format} article in ${config.language === 'en' ? 'English' : 'Vietnamese'}.

Keyword: ${config.keyword}

Research Context (Last 24h):
${researchContext}

Create a comprehensive ${config.format} article that:
1. Uses the latest data and insights from the research
2. Maintains a ${config.tone} tone
3. Includes specific examples and data points
4. Is optimized for ${config.language === 'en' ? 'English' : 'Vietnamese'} readers
5. Has clear structure with headings and bullet points

Output the article in markdown format.`;
  }

  private async generateWithClaude(prompt: string) {
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  private async generateWithOpenAI(prompt: string) {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });

    return completion.choices[0].message.content || '';
  }
}
```

### 3. Video Rendering with Remotion

Transform content into videos automatically.

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'youtube-shorts';
  outputPath: string;
}

export class VideoRenderer {
  private compositionPath: string;

  constructor() {
    this.compositionPath = path.join(process.cwd(), 'remotion', 'index.ts');
  }

  async renderVideo(config: VideoConfig): Promise<string> {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: this.compositionPath,
      webpackOverride: webpackOverride
    });

    // Get composition dimensions based on format
    const dimensions = this.getFormatDimensions(config.format);

    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: {
        title: config.title,
        content: config.content,
        ...dimensions
      }
    });

    // Render video
    const outputLocation = path.join(config.outputPath, `${Date.now()}.mp4`);
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: {
        title: config.title,
        content: config.content
      }
    });

    return outputLocation;
  }

  private getFormatDimensions(format: VideoConfig['format']) {
    const formats = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'youtube-shorts': { width: 1080, height: 1920 }
    };

    return formats[format];
  }
}
```

### 4. Complete Pipeline Workflow

Orchestrate the entire content creation pipeline.

```typescript
// src/lib/pipeline/orchestrator.ts
import { ResearchScanner } from '../research/scanner';
import { ContentGenerator } from '../ai/content-generator';
import { VideoRenderer } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'en' | 'vi'>;
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'youtube-shorts';
}

export class ContentPipeline {
  private scanner: ResearchScanner;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.scanner = new ResearchScanner();
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(config: PipelineConfig) {
    console.log(`Starting pipeline for keyword: ${config.keyword}`);

    // Step 1: Research
    console.log('Step 1: Scanning research...');
    const research = await this.scanner.scanKeyword(config.keyword, 24);
    console.log(`Found ${research.length} relevant sources`);

    // Step 2: Generate content for each format and language
    const contents = [];
    
    for (const format of config.formats) {
      for (const language of config.languages) {
        console.log(`Generating ${format} content in ${language}...`);
        
        const content = await this.generator.generateContent({
          keyword: config.keyword,
          format,
          language,
          tone: config.tone,
          research
        });

        contents.push({
          format,
          language,
          content,
          title: this.extractTitle(content)
        });
      }
    }

    // Step 3: Generate videos if requested
    const videos = [];
    
    if (config.generateVideo && config.videoFormat) {
      for (const contentItem of contents) {
        console.log(`Rendering video for ${contentItem.format} (${contentItem.language})...`);
        
        const videoPath = await this.renderer.renderVideo({
          title: contentItem.title,
          content: contentItem.content,
          format: config.videoFormat,
          outputPath: './output/videos'
        });

        videos.push({
          ...contentItem,
          videoPath
        });
      }
    }

    return {
      research,
      contents,
      videos
    };
  }

  private extractTitle(markdown: string): string {
    const match = markdown.match(/^#\s+(.+)$/m);
    return match ? match[1] : 'Untitled';
  }
}
```

## API Routes

### Create Content Pipeline

```typescript
// src/app/api/pipeline/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword: body.keyword,
      formats: body.formats || ['toplist'],
      languages: body.languages || ['en'],
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error'
    }, { status: 500 });
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

const pipeline = new ContentPipeline();

const result = await pipeline.execute({
  keyword: 'AI automation trends 2024',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  tone: 'expert',
  generateVideo: false
});

console.log(`Generated ${result.contents.length} articles`);
```

### Full Pipeline with Video

```typescript
const result = await pipeline.execute({
  keyword: 'Marketing automation with AI',
  formats: ['case-study'],
  languages: ['en'],
  tone: 'friendly',
  generateVideo: true,
  videoFormat: 'reels'
});

result.videos.forEach(video => {
  console.log(`Video created: ${video.videoPath}`);
});
```

### Custom Research Timeframe

```typescript
import { ResearchScanner } from '@/lib/research/scanner';

const scanner = new ResearchScanner();
const research = await scanner.scanKeyword('blockchain technology', 48); // Last 48 hours

console.log(`Found ${research.length} articles from the last 48 hours`);
```

## Common Patterns

### Batch Processing

```typescript
const keywords = ['AI marketing', 'Content automation', 'Video generation'];
const pipeline = new ContentPipeline();

const results = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({
      keyword,
      formats: ['toplist'],
      languages: ['en'],
      tone: 'expert',
      generateVideo: false
    })
  )
);
```

### Scheduled Content Generation

```typescript
// Using node-cron or similar
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  
  await pipeline.execute({
    keyword: 'daily marketing insights',
    formats: ['toplist'],
    languages: ['en', 'vi'],
    tone: 'friendly',
    generateVideo: true,
    videoFormat: 'reels'
  });
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Errors

If Remotion fails to render:

```bash
# Check Remotion installation
npm ls @remotion/renderer

# Clear cache
rm -rf node_modules/.cache

# Reinstall dependencies
npm install
```

### Research Scraper Issues

```typescript
// Add timeout and error handling
const results = await Promise.allSettled(
  scrapers.map(scraper => 
    Promise.race([
      scraper.search(keyword, hours),
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Timeout')), 30000)
      )
    ])
  )
);

const successful = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value);
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
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
  }
  
  return results;
}
```

## Configuration Best Practices

1. **Use environment variables** for all API keys and sensitive data
2. **Set rate limits** when calling external APIs
3. **Cache research results** to avoid redundant API calls
4. **Monitor token usage** for AI providers (Claude/OpenAI)
5. **Optimize video rendering** settings based on target platform
6. **Log pipeline execution** for debugging and monitoring
