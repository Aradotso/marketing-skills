---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation and multi-platform publishing
triggers:
  - automate content creation with AI research
  - set up automated marketing content pipeline
  - generate videos from articles automatically
  - crawl news sources for content ideas
  - create multilingual content with Claude
  - automate social media content workflow
  - build AI content generation system
  - research and publish content automatically
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-driven content automation system that handles the entire content lifecycle: from researching trending topics across news sources, generating scripts in multiple formats and languages, to automatically rendering videos and publishing to social platforms. Built with TypeScript, Next.js, and integrations with Claude 3, OpenAI, and Remotion for video rendering.

**Key capabilities:**
- Auto-crawl latest news from TechCrunch, a16z, Twitter, LinkedIn (24h data)
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Dual-language output (English & Vietnamese)
- Automatic video/infographic rendering via Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

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

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
NEWSAPI_KEY=your_newsapi_key_here

# Optional: Social Platform APIs
TWITTER_API_KEY=your_twitter_key_here
LINKEDIN_API_TOKEN=your_linkedin_token_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Type checking
npm run type-check

# Linting
npm run lint
```

Access the application at `http://localhost:3000`

## Core Architecture

### 1. Research Module (Content Crawling)

Automatically scans and analyzes news sources for trending topics:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  apiEndpoint?: string;
}

export async function crawlNewsSources(
  keywords: string[],
  sources: NewsSource[] = defaultSources,
  timeframe: number = 24 // hours
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const articles = await fetchArticles(source, keywords, timeframe);
      const analyzed = await analyzeContent(articles);
      results.push({
        source: source.name,
        articles: analyzed,
        insights: extractInsights(analyzed)
      });
    } catch (error) {
      console.error(`Error crawling ${source.name}:`, error);
    }
  }
  
  return results;
}

async function fetchArticles(
  source: NewsSource,
  keywords: string[],
  hours: number
): Promise<Article[]> {
  const response = await axios.get(source.apiEndpoint, {
    params: {
      keywords: keywords.join(','),
      from: new Date(Date.now() - hours * 60 * 60 * 1000).toISOString(),
      sortBy: 'publishedAt'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
    }
  });
  
  return response.data.articles;
}
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
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
  
  async generateContent(
    researchData: ResearchResult[],
    config: ContentConfig
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(researchData, config);
    
    // Use Claude for complex analysis
    const response = await this.claude.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    const content = response.content[0].text;
    
    return {
      title: this.extractTitle(content),
      body: content,
      metadata: {
        format: config.format,
        language: config.language,
        wordCount: content.split(' ').length,
        generatedAt: new Date().toISOString()
      }
    };
  }
  
  private buildPrompt(
    data: ResearchResult[],
    config: ContentConfig
  ): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list with detailed explanations',
      'pov': 'Write from a unique perspective with strong opinions',
      'case-study': 'Analyze with data, examples, and outcomes',
      'how-to': 'Provide step-by-step actionable instructions'
    };
    
    return `
You are an expert content creator. Using the following research data, create a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research Data:
${JSON.stringify(data, null, 2)}

Instructions:
- ${formatInstructions[config.format]}
- Include specific data points and statistics
- Make it engaging and actionable
- Optimize for SEO and readability
- Length: ${config.length}

Generate the content now:
    `.trim();
  }
}
```

### 3. Dual-Language Content Generation

Generate content simultaneously in English and Vietnamese:

```typescript
// lib/ai/multilingual.ts
export async function generateMultilingualContent(
  researchData: ResearchResult[],
  baseConfig: Omit<ContentConfig, 'language'>
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const generator = new ContentGenerator();
  
  const [enContent, viContent] = await Promise.all([
    generator.generateContent(researchData, { ...baseConfig, language: 'en' }),
    generator.generateContent(researchData, { ...baseConfig, language: 'vi' })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  composition: string;
  width: number;
  height: number;
  fps: number;
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
}

export async function renderContentVideo(
  content: GeneratedContent,
  config: VideoConfig
): Promise<string> {
  const compositionId = config.composition;
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      body: content.body,
      metadata: content.metadata
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.title,
      body: content.body,
      metadata: content.metadata
    }
  });
  
  return outputLocation;
}

// Platform-specific configurations
export const platformConfigs: Record<string, VideoConfig> = {
  reels: {
    composition: 'Reels',
    width: 1080,
    height: 1920,
    fps: 30,
    platform: 'reels'
  },
  tiktok: {
    composition: 'TikTok',
    width: 1080,
    height: 1920,
    fps: 30,
    platform: 'tiktok'
  },
  shorts: {
    composition: 'Shorts',
    width: 1080,
    height: 1920,
    fps: 30,
    platform: 'shorts'
  },
  youtube: {
    composition: 'YouTube',
    width: 1920,
    height: 1080,
    fps: 60,
    platform: 'youtube'
  }
};
```

### 5. Complete Pipeline Workflow

Orchestrate the entire content creation pipeline:

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async executePipeline(keywords: string[], config: PipelineConfig): Promise<PipelineResult> {
    console.log('Starting content pipeline...');
    
    // Step 1: Research
    console.log('1. Crawling news sources...');
    const research = await crawlNewsSources(keywords, config.sources);
    
    // Step 2: Generate Content
    console.log('2. Generating content with AI...');
    const content = await generateMultilingualContent(research, {
      format: config.format,
      tone: config.tone,
      length: config.length
    });
    
    // Step 3: Render Videos
    console.log('3. Rendering videos...');
    const videos = await Promise.all(
      config.platforms.map(platform =>
        renderContentVideo(content.en, platformConfigs[platform])
      )
    );
    
    // Step 4: Save Results
    console.log('4. Saving results...');
    const result = {
      content,
      videos,
      research,
      createdAt: new Date().toISOString()
    };
    
    await this.saveToDatabase(result);
    
    console.log('Pipeline completed successfully!');
    return result;
  }
  
  private async saveToDatabase(result: PipelineResult): Promise<void> {
    // Save to your database
    // Implementation depends on your DB choice
  }
}
```

## API Routes (Next.js)

### Create Content Pipeline Endpoint

```typescript
// pages/api/pipeline/create.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keywords, format, platforms, tone } = req.body;
    
    if (!keywords || !Array.isArray(keywords)) {
      return res.status(400).json({ error: 'Keywords array required' });
    }
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.executePipeline(keywords, {
      format: format || 'toplist',
      platforms: platforms || ['reels', 'tiktok'],
      tone: tone || 'friendly',
      length: 'medium',
      sources: [] // Use default sources
    });
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Get Pipeline Status

```typescript
// pages/api/pipeline/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;
  
  // Fetch from your database
  const pipeline = await fetchPipelineById(id as string);
  
  if (!pipeline) {
    return res.status(404).json({ error: 'Pipeline not found' });
  }
  
  res.status(200).json(pipeline);
}
```

## Common Usage Patterns

### 1. Quick Content Generation

```typescript
// Generate content for a trending topic
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

const pipeline = new ContentPipeline();

const result = await pipeline.executePipeline(
  ['AI automation', 'marketing tools'],
  {
    format: 'toplist',
    platforms: ['reels', 'tiktok', 'shorts'],
    tone: 'expert',
    length: 'medium',
    sources: []
  }
);

console.log('Generated content:', result.content.en.title);
console.log('Videos:', result.videos);
```

### 2. Scheduled Content Creation

```typescript
// Schedule daily content generation
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  const todayKeywords = await getTrendingKeywords();
  const pipeline = new ContentPipeline();
  
  await pipeline.executePipeline(todayKeywords, {
    format: 'pov',
    platforms: ['youtube', 'reels'],
    tone: 'friendly',
    length: 'long',
    sources: []
  });
});
```

### 3. Batch Processing Multiple Topics

```typescript
// Generate content for multiple topics
const topics = [
  ['AI marketing', 'automation'],
  ['social media trends', '2024'],
  ['content creation', 'video']
];

const results = await Promise.all(
  topics.map(keywords =>
    pipeline.executePipeline(keywords, defaultConfig)
  )
);
```

## TypeScript Type Definitions

```typescript
// types/pipeline.ts
export interface ResearchResult {
  source: string;
  articles: Article[];
  insights: Insight[];
}

export interface Article {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
  author?: string;
}

export interface Insight {
  topic: string;
  sentiment: 'positive' | 'negative' | 'neutral';
  dataPoints: string[];
  relevanceScore: number;
}

export interface GeneratedContent {
  title: string;
  body: string;
  metadata: {
    format: string;
    language: string;
    wordCount: number;
    generatedAt: string;
  };
}

export interface PipelineConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  platforms: ('reels' | 'tiktok' | 'shorts' | 'youtube')[];
  tone: 'expert' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
  sources: NewsSource[];
}

export interface PipelineResult {
  content: {
    en: GeneratedContent;
    vi: GeneratedContent;
  };
  videos: string[];
  research: ResearchResult[];
  createdAt: string;
}
```

## Configuration Files

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

### Next.js Config

```typescript
// next.config.js
module.exports = {
  reactStrictMode: true,
  webpack: (config) => {
    config.resolve.fallback = {
      ...config.resolve.fallback,
      fs: false,
      net: false,
      tls: false,
    };
    return config;
  },
  env: {
    REMOTION_LICENSE_KEY: process.env.REMOTION_LICENSE_KEY,
  },
};
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with AI providers:

```typescript
// lib/utils/rate-limiter.ts
import pQueue from 'p-queue';

const queue = new pQueue({
  concurrency: 2,
  interval: 1000,
  intervalCap: 2
});

export async function rateLimitedRequest<T>(
  fn: () => Promise<T>
): Promise<T> {
  return queue.add(fn);
}

// Usage
const content = await rateLimitedRequest(() =>
  generator.generateContent(research, config)
);
```

### Video Rendering Failures

Check Remotion logs and ensure sufficient memory:

```typescript
// Increase memory for rendering
export async function renderWithRetry(
  content: GeneratedContent,
  config: VideoConfig,
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(content, config);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### Missing Environment Variables

```typescript
// lib/utils/env-check.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnvironment(): void {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnvironment();
```

### Database Connection Issues

```typescript
// Implement connection pooling and retry logic
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  log: ['error', 'warn'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

export async function connectWithRetry(maxRetries: number = 5): Promise<void> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await prisma.$connect();
      console.log('Database connected successfully');
      return;
    } catch (error) {
      console.error(`Connection attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
}
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use caching** for frequently accessed research results
3. **Implement queue systems** for video rendering to prevent server overload
4. **Monitor API costs** across all providers
5. **Store generated content** with proper version control
6. **Test multilingual output** for translation accuracy
7. **Optimize video assets** for target platform requirements
