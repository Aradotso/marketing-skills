---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content workflow
  - generate videos from blog posts with Remotion integration
  - create multi-language content with Claude and OpenAI
  - crawl news sources and generate social media content
  - build content automation system with research to video
  - configure AI content pipeline with API integrations
  - automate research to script to video publishing workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the complete content lifecycle: from news research and data crawling, to AI-powered script generation in multiple formats and languages, to automated video rendering with Remotion. Integrates Claude 3, OpenAI, RapidAPI for research, and generates platform-optimized videos for TikTok, Reels, and Shorts.

## Project Overview

**Marketing Pipeline Share** is a full-stack Next.js application that automates content creation by:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Uses Claude/OpenAI to generate posts in multiple formats (Toplist, POV, Case Study, How-to)
3. **Multi-language Support**: Automatically creates Vietnamese and English versions
4. **Video Rendering**: Converts content to videos using Remotion
5. **Platform Optimization**: Outputs content optimized for social media platforms

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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

### Environment Variables

Create a `.env.local` file with the following variables:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### API Provider Setup

The system supports multiple AI providers:

```typescript
// config/ai-providers.ts
export const AIProviders = {
  OPENAI: 'openai',
  CLAUDE: 'claude',
} as const;

export const getAIClient = (provider: string) => {
  switch (provider) {
    case AIProviders.OPENAI:
      return new OpenAI({
        apiKey: process.env.OPENAI_API_KEY,
      });
    case AIProviders.CLAUDE:
      return new Anthropic({
        apiKey: process.env.ANTHROPIC_API_KEY,
      });
    default:
      throw new Error(`Unknown AI provider: ${provider}`);
  }
};
```

## Core Components

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  endpoint: string;
  parser: (data: any) => Article[];
}

export class ContentCrawler {
  private sources: NewsSource[];
  
  constructor() {
    this.sources = [
      {
        name: 'TechCrunch',
        endpoint: 'https://techcrunch.com/feed/',
        parser: this.parseTechCrunch,
      },
      // Add more sources
    ];
  }

  async crawlRecentNews(keyword: string, hours: number = 24): Promise<Article[]> {
    const promises = this.sources.map(source => 
      this.fetchFromSource(source, keyword, hours)
    );
    
    const results = await Promise.allSettled(promises);
    const articles = results
      .filter(r => r.status === 'fulfilled')
      .flatMap(r => r.value);
    
    return this.deduplicateArticles(articles);
  }

  private async fetchFromSource(
    source: NewsSource, 
    keyword: string, 
    hours: number
  ): Promise<Article[]> {
    const rapidApiKey = process.env.RAPIDAPI_KEY;
    
    const response = await axios.get(source.endpoint, {
      headers: {
        'X-RapidAPI-Key': rapidApiKey,
        'X-RapidAPI-Host': 'news-api.rapidapi.com'
      },
      params: {
        q: keyword,
        time: `${hours}h`,
      }
    });
    
    return source.parser(response.data);
  }
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: Article[];
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateContent(request: ContentRequest): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(request);
    
    // Use Claude for long-form content
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        }
      ],
    });

    const content = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    return {
      title: this.extractTitle(content),
      body: content,
      metadata: {
        format: request.format,
        language: request.language,
        generatedAt: new Date(),
      }
    };
  }

  private buildPrompt(request: ContentRequest): string {
    const researchContext = request.research
      .map(article => `- ${article.title}: ${article.summary}`)
      .join('\n');

    const formatInstructions = {
      'toplist': 'Create a numbered list article with at least 5 items',
      'pov': 'Write from a unique perspective or opinion piece',
      'case-study': 'Analyze a specific example with data and outcomes',
      'how-to': 'Create a step-by-step tutorial guide',
    };

    return `
You are an expert content creator. Generate a ${request.format} article in ${request.language} language with a ${request.tone} tone.

RESEARCH CONTEXT (last 24 hours):
${researchContext}

FORMAT: ${formatInstructions[request.format]}

REQUIREMENTS:
- Include specific data points and statistics from the research
- Make it engaging and actionable
- Optimize for social media sharing
- Include relevant hashtags at the end

Generate the complete article now:
    `.trim();
  }
}
```

### 3. Multi-Language Support

```typescript
// lib/ai/translator.ts
export class MultiLanguageGenerator {
  async generateBilingual(
    keyword: string,
    format: string,
    research: Article[]
  ): Promise<BilingualContent> {
    const generator = new ContentGenerator();

    // Generate both versions in parallel
    const [englishContent, vietnameseContent] = await Promise.all([
      generator.generateContent({
        keyword,
        format: format as any,
        language: 'en',
        tone: 'expert',
        research,
      }),
      generator.generateContent({
        keyword,
        format: format as any,
        language: 'vi',
        tone: 'expert',
        research,
      }),
    ]);

    return {
      en: englishContent,
      vi: vietnameseContent,
    };
  }
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  platform: 'tiktok' | 'reels' | 'shorts';
  style: 'minimal' | 'dynamic' | 'infographic';
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const composition = this.getCompositionForPlatform(config.platform);
    
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Select composition
    const compositionInfo = await selectComposition({
      serveUrl: bundleLocation,
      id: composition.id,
      inputProps: {
        content: config.content,
        style: config.style,
      },
    });

    // Render video
    const outputLocation = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition: compositionInfo,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: {
        content: config.content,
        style: config.style,
      },
    });

    return outputLocation;
  }

  private getCompositionForPlatform(platform: string) {
    const dimensions = {
      tiktok: { width: 1080, height: 1920, fps: 30 },
      reels: { width: 1080, height: 1920, fps: 30 },
      shorts: { width: 1080, height: 1920, fps: 30 },
    };

    return {
      id: platform,
      ...dimensions[platform],
      durationInFrames: 900, // 30 seconds at 30fps
    };
  }
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  private crawler: ContentCrawler;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.crawler = new ContentCrawler();
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async runFullPipeline(keyword: string): Promise<PipelineResult> {
    try {
      // Step 1: Research
      console.log('📡 Starting research...');
      const articles = await this.crawler.crawlRecentNews(keyword, 24);
      
      // Step 2: Generate content
      console.log('🧠 Generating content...');
      const content = await this.generator.generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        research: articles,
      });

      // Step 3: Render video
      console.log('🎬 Rendering video...');
      const videoPath = await this.renderer.renderContentVideo({
        content,
        platform: 'tiktok',
        style: 'infographic',
      });

      return {
        success: true,
        content,
        videoPath,
        researchCount: articles.length,
      };
    } catch (error) {
      console.error('Pipeline error:', error);
      throw error;
    }
  }
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language, includeVideo } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.runFullPipeline(keyword);

    res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({
      error: 'Failed to generate content',
      message: error.message,
    });
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { ContentPipeline } from './lib/pipeline/orchestrator';

async function generateMarketingContent() {
  const pipeline = new ContentPipeline();
  
  const result = await pipeline.runFullPipeline('AI Marketing Tools 2026');
  
  console.log('Generated content:', result.content.title);
  console.log('Video saved to:', result.videoPath);
  console.log('Based on', result.researchCount, 'recent articles');
}
```

### Custom Format & Language

```typescript
const generator = new ContentGenerator();

const content = await generator.generateContent({
  keyword: 'Social Media Automation',
  format: 'how-to',
  language: 'vi',
  tone: 'friendly',
  research: articles,
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword => pipeline.runFullPipeline(keyword))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  console.log(`Generated ${successful.length}/${keywords.length} pieces of content`);
}

// Usage
batchGenerateContent([
  'AI Content Marketing',
  'Video Marketing Trends',
  'Social Media Automation'
]);
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion Studio (for video editing)
npm run remotion:studio

# Render a specific video composition
npm run remotion:render
```

## Common Patterns

### Error Handling with Retry Logic

```typescript
async function generateWithRetry(
  request: ContentRequest,
  maxRetries: number = 3
): Promise<GeneratedContent> {
  const generator = new ContentGenerator();
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await generator.generateContent(request);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string): Promise<Article[] | null> {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

async function cacheResearch(keyword: string, articles: Article[]): Promise<void> {
  await redis.setex(
    `research:${keyword}`,
    3600, // 1 hour TTL
    JSON.stringify(articles)
  );
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with AI providers:

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const results = await Promise.all(
  requests.map(req => limit(() => generator.generateContent(req)))
);
```

### Video Rendering Memory Issues

For large video renders, increase Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

### Debugging Research Crawling

```typescript
// Enable verbose logging
const crawler = new ContentCrawler();
crawler.setDebug(true);

const articles = await crawler.crawlRecentNews('keyword', 24);
console.log('Crawled articles:', articles.map(a => a.title));
```

## Integration Tips

- **Scheduled Jobs**: Use cron jobs or services like Vercel Cron to automate content generation
- **Database Storage**: Store generated content in PostgreSQL or MongoDB for publishing queue
- **Social Media APIs**: Integrate with Facebook, Twitter, LinkedIn APIs for auto-posting
- **Analytics**: Track content performance and use data to optimize prompts
- **Content Calendar**: Build a scheduling system to space out posts across platforms
