---
name: ai-content-pipeline-automation
description: Automated content pipeline using Claude/OpenAI for research, scriptwriting, and video generation with Remotion
triggers:
  - automate content creation with AI pipeline
  - research and generate content automatically
  - create videos from text with Remotion
  - scrape news and generate blog posts
  - build content automation workflow
  - generate multilingual marketing content
  - setup AI content pipeline system
  - crawl trending topics and create videos
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The AI Content Pipeline is a comprehensive content automation system that handles the entire content lifecycle: from scraping trending news across platforms (TechCrunch, Twitter, LinkedIn), to AI-powered content generation (using Claude 3 and OpenAI), to automatic video rendering with Remotion. Built with Next.js and TypeScript, it enables marketers to produce high-quality multilingual content (English/Vietnamese) at scale.

**Key capabilities:**
- Auto-crawl news from multiple sources in real-time
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Produce bilingual content with customizable tone
- Render videos and infographics automatically
- Export optimized content for social platforms

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Setup environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Start Development Server

```bash
npm run dev
# Access at http://localhost:3000
```

## Core Architecture

### 1. Research/Scraping Module

Automatically crawls and aggregates content from news sources:

```typescript
// lib/scraper/news-aggregator.ts
import { RapidAPIClient } from './rapid-api-client';

interface NewsSource {
  platform: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
  query: string;
  timeframe: '24h' | '7d' | '30d';
}

export class NewsAggregator {
  private apiClient: RapidAPIClient;

  constructor(apiKey: string) {
    this.apiClient = new RapidAPIClient(apiKey);
  }

  async fetchTrendingNews(sources: NewsSource[]): Promise<Article[]> {
    const results = await Promise.all(
      sources.map(source => this.apiClient.search({
        platform: source.platform,
        query: source.query,
        timeframe: source.timeframe
      }))
    );

    return this.aggregateAndRank(results.flat());
  }

  private aggregateAndRank(articles: RawArticle[]): Article[] {
    // Remove duplicates and rank by engagement
    const unique = this.deduplicateArticles(articles);
    return unique.sort((a, b) => b.engagement - a.engagement);
  }
}

// Usage
const aggregator = new NewsAggregator(process.env.RAPIDAPI_KEY!);
const trendingTopics = await aggregator.fetchTrendingNews([
  { platform: 'techcrunch', query: 'AI startup', timeframe: '24h' },
  { platform: 'twitter', query: '#marketing', timeframe: '24h' }
]);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
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

  async generateArticle(
    research: Article[],
    config: ContentConfig
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(research, config);

    const response = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return this.parseResponse(response);
  }

  private buildPrompt(research: Article[], config: ContentConfig): string {
    const formatInstructions = this.getFormatInstructions(config.format);
    const toneGuidelines = this.getToneGuidelines(config.tone);
    const researchSummary = this.summarizeResearch(research);

    return `
You are a ${config.tone} content writer creating a ${config.format} article in ${config.language}.

Research Data:
${researchSummary}

${formatInstructions}
${toneGuidelines}

Generate a compelling article that:
- Uses data-backed insights from the research
- Maintains ${config.tone} tone throughout
- Follows ${config.format} structure
- Is optimized for ${config.language} audience
`;
  }

  async generateBilingual(research: Article[]): Promise<{
    en: GeneratedContent;
    vi: GeneratedContent;
  }> {
    const [english, vietnamese] = await Promise.all([
      this.generateArticle(research, { 
        format: 'toplist', 
        tone: 'expert', 
        language: 'en', 
        length: 'medium' 
      }),
      this.generateArticle(research, { 
        format: 'toplist', 
        tone: 'expert', 
        language: 'vi', 
        length: 'medium' 
      })
    ]);

    return { en: english, vi: vietnamese };
  }
}

// Usage in API route
// pages/api/generate-content.ts
import { ContentGenerator } from '@/lib/ai/content-generator';

export default async function handler(req, res) {
  const { keyword, format, tone } = req.body;

  const generator = new ContentGenerator();
  const content = await generator.generateArticle(researchData, {
    format,
    tone,
    language: 'en',
    length: 'medium'
  });

  res.json(content);
}
```

### 3. Video Generation with Remotion

Transform text content into engaging videos:

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
  duration: number;
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const composition = this.selectCompositionByPlatform(config.platform);
    const bundleLocation = await bundle(
      path.join(process.cwd(), 'remotion/index.ts')
    );

    const inputProps = {
      title: config.content.title,
      points: config.content.keyPoints,
      images: config.content.images,
      duration: config.duration
    };

    const outputLocation = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps
    });

    return outputLocation;
  }

  private selectCompositionByPlatform(platform: string) {
    const dimensions = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 },
      youtube: { width: 1920, height: 1080 }
    };

    return {
      id: platform,
      ...dimensions[platform],
      fps: 30,
      durationInFrames: 30 * 60 // 60 seconds
    };
  }
}

// Remotion composition
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  images: string[];
}> = ({ title, points, images }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleScene title={title} />
      </Sequence>
      {points.map((point, idx) => (
        <Sequence
          key={idx}
          from={90 + idx * 120}
          durationInFrames={120}
        >
          <ContentPoint point={point} image={images[idx]} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. End-to-End Pipeline

Orchestrate the entire workflow:

```typescript
// lib/pipeline/content-pipeline.ts
import { NewsAggregator } from '@/lib/scraper/news-aggregator';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/remotion-renderer';

export class ContentPipeline {
  private aggregator: NewsAggregator;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.aggregator = new NewsAggregator(process.env.RAPIDAPI_KEY!);
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async executeFullPipeline(keyword: string): Promise<ContentPackage> {
    // Step 1: Research
    console.log('🔍 Researching trending topics...');
    const research = await this.aggregator.fetchTrendingNews([
      { platform: 'techcrunch', query: keyword, timeframe: '24h' },
      { platform: 'twitter', query: `#${keyword}`, timeframe: '24h' }
    ]);

    // Step 2: Generate Content
    console.log('✍️ Generating bilingual content...');
    const content = await this.generator.generateBilingual(research);

    // Step 3: Render Videos
    console.log('🎬 Rendering videos...');
    const [videoEN, videoVI] = await Promise.all([
      this.renderer.renderContentVideo({
        content: content.en,
        platform: 'reels',
        duration: 60
      }),
      this.renderer.renderContentVideo({
        content: content.vi,
        platform: 'reels',
        duration: 60
      })
    ]);

    console.log('✅ Pipeline complete!');
    return {
      articles: content,
      videos: { en: videoEN, vi: videoVI },
      research
    };
  }
}

// Usage in Next.js API
// pages/api/pipeline/execute.ts
export default async function handler(req, res) {
  const { keyword } = req.body;

  const pipeline = new ContentPipeline();
  const result = await pipeline.executeFullPipeline(keyword);

  res.json(result);
}
```

## Frontend Integration

React component for triggering the pipeline:

```typescript
// components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline/execute', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI marketing)"
          className="w-full p-3 border rounded"
          required
        />
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded"
        >
          {loading ? '⏳ Generating...' : '🚀 Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8 space-y-4">
          <h2 className="text-2xl font-bold">Generated Content</h2>
          <div className="grid grid-cols-2 gap-4">
            <div>
              <h3>English Version</h3>
              <p>{result.articles.en.title}</p>
              <video controls src={result.videos.en} />
            </div>
            <div>
              <h3>Vietnamese Version</h3>
              <p>{result.articles.vi.title}</p>
              <video controls src={result.videos.vi} />
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom Content Formats

Create reusable content format templates:

```typescript
// config/content-formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: [
      'Introduction with hook',
      '5-10 numbered points with explanations',
      'Data/examples for each point',
      'Conclusion with CTA'
    ],
    seoOptimized: true,
    minWords: 1500
  },
  pov: {
    structure: [
      'Personal perspective opening',
      'Main argument with supporting evidence',
      'Counterarguments addressed',
      'Strong opinion-based conclusion'
    ],
    seoOptimized: false,
    minWords: 1000
  },
  'case-study': {
    structure: [
      'Problem statement',
      'Solution approach',
      'Implementation details',
      'Results and metrics',
      'Key takeaways'
    ],
    seoOptimized: true,
    minWords: 2000
  }
};
```

### AI Model Selection

```typescript
// lib/ai/model-selector.ts
export function selectOptimalModel(task: string, config: ContentConfig) {
  // Use Claude for creative, nuanced content
  if (config.tone === 'humorous' || config.format === 'pov') {
    return {
      provider: 'anthropic',
      model: 'claude-3-5-sonnet-20241022'
    };
  }

  // Use GPT-4 for data-heavy, structured content
  if (config.format === 'case-study' || config.format === 'toplist') {
    return {
      provider: 'openai',
      model: 'gpt-4-turbo-preview'
    };
  }

  return { provider: 'anthropic', model: 'claude-3-5-sonnet-20241022' };
}
```

## Troubleshooting

### Common Issues

**API Rate Limits**
```typescript
// Implement retry logic with exponential backoff
import pRetry from 'p-retry';

async function callAIWithRetry(prompt: string) {
  return pRetry(
    () => claude.messages.create({ /* ... */ }),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

**Remotion Rendering Timeout**
```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps,
  timeoutInMilliseconds: 300000 // 5 minutes
});
```

**Memory Issues with Large Content**
```typescript
// Process content in chunks
async function processLargeContent(articles: Article[]) {
  const chunkSize = 10;
  const chunks = [];
  
  for (let i = 0; i < articles.length; i += chunkSize) {
    const chunk = articles.slice(i, i + chunkSize);
    const processed = await processChunk(chunk);
    chunks.push(processed);
  }
  
  return chunks.flat();
}
```

**Environment Variable Loading**
```typescript
// Validate all required env vars at startup
// lib/config/validate-env.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

requiredEnvVars.forEach(key => {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
});
```

## Performance Optimization

```typescript
// Use caching for research data
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;

  const fresh = await aggregator.fetchTrendingNews([/* ... */]);
  await redis.set(`research:${keyword}`, fresh, { ex: 3600 }); // 1 hour cache
  return fresh;
}
```

## Deployment

```bash
# Build for production
npm run build

# Deploy to Vercel
vercel --prod

# Or Docker
docker build -t content-pipeline .
docker run -p 3000:3000 --env-file .env content-pipeline
```
