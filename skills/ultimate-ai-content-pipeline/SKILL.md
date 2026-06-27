---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos from articles using Remotion
  - crawl news sources and create content automatically
  - build automated marketing content system
  - create multilingual content with AI pipeline
  - generate social media videos from written content
  - automate content research and scriptwriting
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics and crawling news sources, to generating multi-format articles in multiple languages, to rendering videos and infographics. It integrates Claude 3, OpenAI, and Remotion to transform a single keyword into publication-ready content across text and video formats.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual output (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering for social media
- Next.js web interface for easy management

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn
- API keys for:
  - OpenAI or Anthropic Claude
  - RapidAPI (for news crawling)
  - Remotion license (for video rendering)

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Configure environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env.local` file in the project root:

```env
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=postgresql://user:password@localhost:5432/content_db
```

### Run Development Server

```bash
npm run dev
# Access at http://localhost:3000
```

## Core Architecture

```
src/
├── app/              # Next.js app router pages
├── lib/
│   ├── ai/          # AI service integrations
│   ├── crawlers/    # News source crawlers
│   ├── generators/  # Content generation logic
│   └── video/       # Remotion video rendering
├── components/       # React UI components
└── types/           # TypeScript definitions
```

## Key Components & Usage

### 1. Research & Data Crawling

The system crawls multiple news sources for fresh content:

```typescript
// lib/crawlers/news-aggregator.ts
import { TechCrunchCrawler } from './sources/techcrunch';
import { TwitterCrawler } from './sources/twitter';
import { LinkedInCrawler } from './sources/linkedin';

interface NewsArticle {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

export class NewsAggregator {
  private crawlers: Array<BaseCrawler>;

  constructor(apiKey: string) {
    this.crawlers = [
      new TechCrunchCrawler(apiKey),
      new TwitterCrawler(apiKey),
      new LinkedInCrawler(apiKey)
    ];
  }

  async fetchLatestNews(keyword: string, hours: number = 24): Promise<NewsArticle[]> {
    const results = await Promise.all(
      this.crawlers.map(crawler => crawler.search(keyword, hours))
    );
    
    return results.flat().sort((a, b) => 
      b.publishedAt.getTime() - a.publishedAt.getTime()
    );
  }

  async extractInsights(articles: NewsArticle[]): Promise<string[]> {
    // Process articles to extract key insights
    const combinedContent = articles.map(a => a.content).join('\n\n');
    return this.analyzeContent(combinedContent);
  }
}
```

Usage example:

```typescript
import { NewsAggregator } from '@/lib/crawlers/news-aggregator';

const aggregator = new NewsAggregator(process.env.RAPIDAPI_KEY!);
const articles = await aggregator.fetchLatestNews('AI automation', 24);
const insights = await aggregator.extractInsights(articles);
```

### 2. AI Content Generation

Generate multi-format content using Claude or OpenAI:

```typescript
// lib/generators/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  insights: string[];
  sourceArticles: NewsArticle[];
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async generate(options: GenerateOptions): Promise<string> {
    const prompt = this.buildPrompt(options);
    
    // Use Claude for content generation
    const message = await this.claude.messages.create({
      model: 'claude-3-opus-20240229',
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

  private buildPrompt(options: GenerateOptions): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a specific perspective or viewpoint',
      'case-study': 'Analyze a real-world example in depth',
      'how-to': 'Provide step-by-step instructions'
    };

    return `
You are an expert content writer. Create ${options.language === 'vi' ? 'Vietnamese' : 'English'} content with a ${options.tone} tone.

Format: ${formatInstructions[options.format]}
Topic: ${options.keyword}

Recent insights from news sources:
${options.insights.join('\n')}

Source articles for reference:
${options.sourceArticles.map(a => `- ${a.title} (${a.source})`).join('\n')}

Requirements:
- Use recent data and examples
- Include statistics where relevant
- Make it engaging and actionable
- Optimize for SEO
`;
  }

  async generateBilingual(options: Omit<GenerateOptions, 'language'>): Promise<{
    en: string;
    vi: string;
  }> {
    const [en, vi] = await Promise.all([
      this.generate({ ...options, language: 'en' }),
      this.generate({ ...options, language: 'vi' })
    ]);

    return { en, vi };
  }
}
```

Usage example:

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';

const generator = new ContentGenerator();

const content = await generator.generateBilingual({
  keyword: 'AI automation trends 2024',
  format: 'toplist',
  tone: 'expert',
  insights: insights,
  sourceArticles: articles
});

console.log('English:', content.en);
console.log('Vietnamese:', content.vi);
```

### 3. Video Generation with Remotion

Convert written content into social media videos:

```typescript
// lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  keyPoints: string[];
  duration: number; // in seconds
  format: 'reel' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

export class VideoRenderer {
  private compositionPath: string;

  constructor() {
    this.compositionPath = path.join(process.cwd(), 'src/remotion');
  }

  async renderVideo(config: VideoConfig): Promise<string> {
    // Bundle Remotion composition
    const bundled = await bundle({
      entryPoint: path.join(this.compositionPath, 'index.ts'),
      webpackOverride: (config) => config
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundled,
      id: 'ContentVideo',
      inputProps: {
        title: config.title,
        content: config.content,
        keyPoints: config.keyPoints
      }
    });

    // Render video
    const outputPath = path.join(
      process.cwd(), 
      'output', 
      `${Date.now()}-${config.format}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundled,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        content: config.content,
        keyPoints: config.keyPoints
      }
    });

    return outputPath;
  }

  async renderMultiFormat(baseConfig: Omit<VideoConfig, 'format'>): Promise<{
    reel: string;
    tiktok: string;
    shorts: string;
  }> {
    const formats: Array<VideoConfig['format']> = ['reel', 'tiktok', 'shorts'];
    
    const results = await Promise.all(
      formats.map(format => 
        this.renderVideo({ ...baseConfig, format })
      )
    );

    return {
      reel: results[0],
      tiktok: results[1],
      shorts: results[2]
    };
  }
}
```

Remotion composition example:

```typescript
// src/remotion/ContentVideo.tsx
import React from 'react';
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  keyPoints 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{
        padding: '60px',
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        height: '100%'
      }}>
        <h1 style={{
          fontSize: '64px',
          color: '#fff',
          opacity: titleOpacity,
          marginBottom: '40px',
          fontWeight: 'bold'
        }}>
          {title}
        </h1>
        
        <div style={{ color: '#fff', fontSize: '32px', lineHeight: 1.6 }}>
          {keyPoints.map((point, index) => {
            const pointFrame = 60 + (index * 90);
            const pointOpacity = interpolate(
              frame, 
              [pointFrame, pointFrame + 20], 
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div key={index} style={{ 
                opacity: pointOpacity,
                marginBottom: '20px' 
              }}>
                • {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Orchestration

Combine all components into a single workflow:

```typescript
// lib/pipeline/content-pipeline.ts
import { NewsAggregator } from '@/lib/crawlers/news-aggregator';
import { ContentGenerator } from '@/lib/generators/content-generator';
import { VideoRenderer } from '@/lib/video/video-renderer';

interface PipelineResult {
  articles: NewsArticle[];
  content: {
    en: string;
    vi: string;
  };
  videos: {
    reel: string;
    tiktok: string;
    shorts: string;
  };
}

export class ContentPipeline {
  private aggregator: NewsAggregator;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.aggregator = new NewsAggregator(process.env.RAPIDAPI_KEY!);
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(
    keyword: string,
    format: ContentFormat,
    tone: Tone = 'expert'
  ): Promise<PipelineResult> {
    console.log(`🔍 Step 1: Researching ${keyword}...`);
    const articles = await this.aggregator.fetchLatestNews(keyword, 24);
    const insights = await this.aggregator.extractInsights(articles);

    console.log(`✍️ Step 2: Generating content...`);
    const content = await this.generator.generateBilingual({
      keyword,
      format,
      tone,
      insights,
      sourceArticles: articles
    });

    console.log(`🎬 Step 3: Rendering videos...`);
    const keyPoints = this.extractKeyPoints(content.en);
    const videos = await this.renderer.renderMultiFormat({
      title: keyword,
      content: content.en,
      keyPoints,
      duration: 60
    });

    console.log(`✅ Pipeline complete!`);
    return { articles, content, videos };
  }

  private extractKeyPoints(content: string): string[] {
    // Simple extraction - can be enhanced with AI
    const lines = content.split('\n').filter(line => 
      line.match(/^\d+\./) || line.match(/^•/) || line.match(/^-/)
    );
    return lines.slice(0, 5);
  }
}
```

Usage in API route:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone } = await request.json();

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword, format, tone);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error'
    }, { status: 500 });
  }
}
```

## Common Workflows

### Quick Content Generation

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

const pipeline = new ContentPipeline();

// Generate a toplist article with videos
const result = await pipeline.execute(
  'Top AI tools for marketing 2024',
  'toplist',
  'friendly'
);

console.log('Generated articles:', result.articles.length);
console.log('English content length:', result.content.en.length);
console.log('Video files:', result.videos);
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export class ContentScheduler {
  private pipeline: ContentPipeline;

  constructor() {
    this.pipeline = new ContentPipeline();
  }

  start() {
    // Run every day at 9 AM
    cron.schedule('0 9 * * *', async () => {
      const keywords = [
        'AI trends',
        'Marketing automation',
        'Content creation tools'
      ];

      for (const keyword of keywords) {
        try {
          await this.pipeline.execute(keyword, 'toplist', 'expert');
          console.log(`✅ Generated content for: ${keyword}`);
        } catch (error) {
          console.error(`❌ Failed for ${keyword}:`, error);
        }
      }
    });
  }
}
```

### Custom AI Provider Selection

```typescript
// lib/generators/content-generator.ts (extended)
export class ContentGenerator {
  async generateWithProvider(
    provider: 'claude' | 'openai',
    options: GenerateOptions
  ): Promise<string> {
    if (provider === 'claude') {
      return this.generateWithClaude(options);
    } else {
      return this.generateWithOpenAI(options);
    }
  }

  private async generateWithOpenAI(options: GenerateOptions): Promise<string> {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: this.buildPrompt(options)
      }],
      max_tokens: 4096,
      temperature: 0.7
    });

    return completion.choices[0]?.message?.content || '';
  }

  private async generateWithClaude(options: GenerateOptions): Promise<string> {
    // Claude implementation (as shown above)
    const message = await this.claude.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: this.buildPrompt(options)
      }]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
}
```

## Configuration

### Content Format Templates

Create custom templates for different content formats:

```typescript
// config/templates.ts
export const contentTemplates = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  'case-study': {
    sections: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true,
    minWords: 1500
  },
  'how-to': {
    structure: 'step-by-step',
    includePrerequisites: true,
    includeExamples: true,
    minSteps: 5
  },
  pov: {
    perspective: 'first-person',
    includeOpinion: true,
    supportWithData: true
  }
};
```

### Video Rendering Settings

```typescript
// config/video.ts
export const videoSettings = {
  reel: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;

  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }

  async throttle(): Promise<void> {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 calls per minute
await limiter.throttle();
await apiCall();
```

### Failed Video Renders

```typescript
// Add retry logic
async function renderWithRetry(
  renderer: VideoRenderer, 
  config: VideoConfig, 
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderer.renderVideo(config);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('All retry attempts failed');
}
```

### Memory Issues with Large Content

```typescript
// Process in chunks
async function processLargeArticleSet(
  articles: NewsArticle[], 
  chunkSize: number = 10
): Promise<string[]> {
  const results: string[] = [];
  
  for (let i = 0; i < articles.length; i += chunkSize) {
    const chunk = articles.slice(i, i + chunkSize);
    const insights = await aggregator.extractInsights(chunk);
    results.push(...insights);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return results;
}
```

### Environment Variable Validation

```typescript
// lib/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  OPENAI_API_KEY: z.string().optional(),
  ANTHROPIC_API_KEY: z.string().optional(),
  RAPIDAPI_KEY: z.string().min(1),
  REMOTION_LICENSE_KEY: z.string().min(1),
  DATABASE_URL: z.string().url()
}).refine(data => data.OPENAI_API_KEY || data.ANTHROPIC_API_KEY, {
  message: 'Either OPENAI_API_KEY or ANTHROPIC_API_KEY must be provided'
});

export function validateEnv() {
  try {
    envSchema.parse(process.env);
  } catch (error) {
    console.error('❌ Invalid environment variables:', error);
    process.exit(1);
  }
}
```

## Best Practices

1. **Use environment-specific configs**: Separate development and production settings
2. **Implement caching**: Cache news articles and AI responses to reduce API costs
3. **Monitor API usage**: Track token usage for OpenAI/Claude to optimize costs
4. **Queue video renders**: Use a job queue (Bull, BullMQ) for video processing
5. **Error handling**: Always wrap AI calls in try-catch with fallbacks
6. **Content validation**: Validate generated content for quality before publishing
7. **Version control outputs**: Store generated content with metadata for tracking

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
vercel deploy --prod
```

For video rendering at scale, consider using:
- Remotion Lambda for serverless rendering
- Separate worker processes for video generation
- Cloud storage (S3, Cloudflare R2) for output files
