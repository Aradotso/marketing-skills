---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - automate content creation from research to video
  - use Claude and OpenAI for content automation
  - create AI-powered marketing content pipeline
  - generate videos from articles automatically
  - build automated content workflow with Remotion
  - scrape news and generate content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete TypeScript-based content automation system that crawls news sources, generates multi-format articles using Claude/OpenAI, and renders videos automatically with Remotion. Perfect for marketers, content creators, and agencies looking to scale content production 10x.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Research Phase**: Auto-crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
2. **Content Generation**: Uses Claude 3/OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multilingual Output**: Generates both English and Vietnamese versions simultaneously
4. **Video Rendering**: Converts articles into short-form videos using Remotion
5. **Multi-Platform Export**: Optimized for Reels, TikTok, Shorts with proper aspect ratios

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

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_CODEC=h264
REMOTION_CRF=18
```

## Core Components

### 1. Research Module

The research module crawls multiple sources for fresh content:

```typescript
import { ResearchEngine } from './lib/research';

const engine = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing automation']
});

// Fetch and analyze data
const research = await engine.scan({
  keyword: 'AI content creation',
  depth: 'comprehensive' // or 'quick'
});

console.log(research.insights); // Key insights extracted
console.log(research.dataPoints); // Statistical data
console.log(research.trends); // Trending topics
```

### 2. Content Generation

Generate content in multiple formats using AI:

```typescript
import { ContentGenerator } from './lib/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-opus-20240229'
});

// Generate article from research
const article = await generator.create({
  research: research,
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: ['en', 'vi'],
  tone: 'expert', // 'friendly', 'humorous'
  length: 'medium' // 'short', 'long'
});

// Output structure
console.log(article.en.title);
console.log(article.en.content);
console.log(article.vi.title);
console.log(article.vi.content);
console.log(article.metadata); // SEO, keywords, etc.
```

### 3. Video Rendering with Remotion

Convert articles to videos automatically:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Prepare video data from article
const videoData = {
  title: article.en.title,
  points: article.en.keyPoints,
  stats: research.dataPoints,
  branding: {
    logo: '/assets/logo.png',
    colors: ['#FF6B6B', '#4ECDC4']
  }
};

// Bundle Remotion project
const bundleLocation = await bundle({
  entryPoint: path.resolve('./remotion/index.ts'),
  webpackOverride: (config) => config,
});

// Select composition
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo', // Composition ID
  inputProps: videoData,
});

// Render video
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/${article.slug}.mp4`,
  inputProps: videoData,
});
```

### 4. Complete Pipeline Workflow

End-to-end automation example:

```typescript
import { Pipeline } from './lib/pipeline';

const pipeline = new Pipeline({
  research: {
    sources: ['techcrunch', 'twitter'],
    depth: 'comprehensive'
  },
  content: {
    provider: 'claude',
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  },
  video: {
    enabled: true,
    platforms: ['reels', 'tiktok', 'shorts']
  }
});

// Run complete pipeline
const result = await pipeline.execute({
  keyword: 'AI marketing automation',
  schedule: {
    publish: true,
    platform: 'facebook',
    date: '2024-01-15T10:00:00Z'
  }
});

// Access outputs
console.log(result.articles); // Generated articles
console.log(result.videos); // Rendered video paths
console.log(result.analytics); // Performance metrics
```

## API Routes (Next.js)

### Start Pipeline via API

```typescript
// pages/api/pipeline/start.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { Pipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, formats, languages } = req.body;

  try {
    const pipeline = new Pipeline({
      research: { sources: ['techcrunch', 'twitter'] },
      content: { provider: 'claude', formats, languages },
      video: { enabled: true }
    });

    const result = await pipeline.execute({ keyword });

    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Get Pipeline Status

```typescript
// pages/api/pipeline/status/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { getPipelineStatus } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;

  const status = await getPipelineStatus(id as string);

  res.status(200).json({
    id,
    status: status.stage, // 'research', 'generating', 'rendering', 'complete'
    progress: status.progress, // 0-100
    outputs: status.outputs
  });
}
```

## CLI Commands

If the project includes a CLI tool:

```bash
# Run research only
npm run research -- --keyword "AI marketing" --sources techcrunch,twitter

# Generate content from existing research
npm run generate -- --research-id abc123 --format toplist --lang en,vi

# Render video from article
npm run render -- --article-id xyz789 --platform reels

# Full pipeline
npm run pipeline -- --keyword "AI automation" --all

# Check pipeline status
npm run status -- --id pipeline-123
```

## Common Patterns

### Custom Content Format

```typescript
import { ContentGenerator, ContentFormat } from './lib/content';

const customFormat: ContentFormat = {
  name: 'expert-interview',
  structure: [
    { section: 'introduction', length: 150 },
    { section: 'questions', count: 5 },
    { section: 'expert-quotes', count: 3 },
    { section: 'conclusion', length: 100 }
  ],
  prompt: `Create an expert interview format discussing {topic}...`
};

const generator = new ContentGenerator({
  provider: 'claude',
  customFormats: [customFormat]
});

const article = await generator.create({
  research: research,
  format: 'expert-interview'
});
```

### Batch Processing

```typescript
import { BatchPipeline } from './lib/pipeline';

const keywords = [
  'AI content creation',
  'Marketing automation',
  'Social media tools'
];

const batch = new BatchPipeline({
  concurrency: 3, // Process 3 at a time
  retries: 2
});

const results = await batch.process(keywords, {
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  video: true
});

// Monitor progress
batch.on('progress', (status) => {
  console.log(`Completed: ${status.completed}/${status.total}`);
});
```

### Custom Video Template

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000', opacity }}>
      <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
      {points.map((point, i) => (
        <p key={i} style={{ color: '#fff', fontSize: 30 }}>{point}</p>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for research APIs
import { RateLimiter } from './lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 100,
  perMinutes: 1
});

const research = await limiter.execute(async () => {
  return await engine.scan({ keyword: 'AI marketing' });
});
```

### Claude API Timeouts

```typescript
// Add retry logic for long content generation
import { retry } from './lib/utils/retry';

const article = await retry(
  async () => await generator.create({ research, format: 'case-study' }),
  {
    retries: 3,
    delay: 2000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Memory Issues

```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render

# Or in package.json
"scripts": {
  "render": "NODE_OPTIONS='--max-old-space-size=4096' node render.js"
}
```

### Missing Environment Variables

```typescript
// Validate env vars at startup
import { z } from 'zod';

const envSchema = z.object({
  ANTHROPIC_API_KEY: z.string().min(1),
  OPENAI_API_KEY: z.string().min(1),
  RAPIDAPI_KEY: z.string().min(1),
});

try {
  envSchema.parse(process.env);
} catch (error) {
  console.error('Missing required environment variables:', error);
  process.exit(1);
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { CacheManager } from './lib/cache';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  storage: 'redis' // or 'memory'
});

const research = await cache.getOrSet(
  `research:${keyword}`,
  async () => await engine.scan({ keyword })
);
```

### Parallel Video Rendering

```typescript
import pLimit from 'p-limit';

const limit = pLimit(2); // Render 2 videos simultaneously

const videos = await Promise.all(
  articles.map(article =>
    limit(() => renderVideo(article))
  )
);
```

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through video generation, with proper error handling and optimization strategies.
