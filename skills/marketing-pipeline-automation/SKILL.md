---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from articles automatically
  - crawl trending news and create content
  - build AI content pipeline with Remotion
  - create multi-format content with Claude
  - automate social media content production
  - set up content automation workflow
  - generate multi-language marketing content
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multi-format articles in multiple languages, to rendering videos and graphics automatically.

## What This Project Does

The Marketing Pipeline automates content production by:

- **Auto-scanning research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **AI content generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language support**: Generates parallel English and Vietnamese content
- **Video rendering**: Automatically converts written content into videos/infographics using Remotion
- **Platform optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the required API keys:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Optional configurations
CONTENT_LANGUAGE=en,vi
DEFAULT_TONE=professional
VIDEO_OUTPUT_PATH=./output/videos
```

## Key Components & Usage

### 1. Research Module (Auto-Scan)

The research module crawls trending topics from various sources:

```typescript
import { ResearchScanner } from '@/lib/research/scanner';

// Initialize scanner
const scanner = new ResearchScanner({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Scan for trending topics
const trends = await scanner.scan();

// Get insights with analysis
const insights = await scanner.analyzeTopics(trends, {
  minRelevance: 0.7,
  includeStats: true,
  language: 'en'
});

console.log(insights);
// Returns: { topic, summary, dataPoints, sources, relevanceScore }
```

### 2. Content Generation with AI

Generate multi-format content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/content/generator';
import { ContentFormat } from '@/types/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-sonnet-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate article
const article = await generator.generate({
  topic: 'AI Marketing Automation Trends 2026',
  format: ContentFormat.TOPLIST,
  tone: 'professional',
  languages: ['en', 'vi'],
  researchData: insights,
  targetAudience: 'marketers',
  wordCount: 1500
});

// Article structure
console.log(article);
// {
//   title: { en: "...", vi: "..." },
//   content: { en: "...", vi: "..." },
//   metadata: { format, tone, generatedAt },
//   outline: [...],
//   citations: [...]
// }
```

### 3. Video Rendering with Remotion

Convert articles into videos automatically:

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  composition: 'ArticleVideo',
  outputFormat: 'mp4',
  fps: 30,
  dimensions: { width: 1080, height: 1920 } // Reels/TikTok format
});

// Render video from article
const video = await renderer.render({
  articleContent: article.content.en,
  template: 'infographic',
  style: {
    theme: 'modern',
    accentColor: '#FF6B6B',
    font: 'Inter'
  },
  duration: 60, // seconds
  outputPath: './output/videos/article-video.mp4'
});

console.log(video.filePath);
```

### 4. Complete Pipeline Workflow

Chain all components together:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  },
  content: {
    provider: 'claude',
    formats: ['toplist', 'howto'],
    languages: ['en', 'vi']
  },
  video: {
    enabled: true,
    platforms: ['tiktok', 'reels', 'shorts']
  }
});

// Run full pipeline
const result = await pipeline.run({
  keyword: 'AI content automation',
  count: 5 // Generate 5 pieces of content
});

// Result contains all generated assets
result.forEach(item => {
  console.log(`Title: ${item.article.title.en}`);
  console.log(`Video: ${item.video?.filePath}`);
  console.log(`Published: ${item.publishedUrl}`);
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/content/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language } = req.body;

  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      count: 1,
      formats: [format],
      languages: [language]
    });

    res.status(200).json({
      success: true,
      data: result[0]
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research/scan.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchScanner } from '@/lib/research/scanner';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keywords, sources, timeRange } = req.query;

  const scanner = new ResearchScanner({
    sources: (sources as string).split(','),
    timeRange: timeRange as string || '24h',
    keywords: (keywords as string).split(',')
  });

  const trends = await scanner.scan();
  const insights = await scanner.analyzeTopics(trends);

  res.status(200).json({ insights });
}
```

## Common Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/types/content';

const customTemplate: ContentTemplate = {
  name: 'product-launch',
  structure: [
    { section: 'hook', wordCount: 100 },
    { section: 'problem', wordCount: 200 },
    { section: 'solution', wordCount: 300 },
    { section: 'features', wordCount: 400 },
    { section: 'cta', wordCount: 100 }
  ],
  tone: 'exciting',
  includeVisuals: true
};

const article = await generator.generate({
  topic: 'New AI Tool Launch',
  template: customTemplate
});
```

### Batch Content Generation

```typescript
const keywords = [
  'AI marketing trends',
  'Content automation tools',
  'Video marketing strategy'
];

const batchResults = await Promise.all(
  keywords.map(keyword =>
    pipeline.run({
      keyword,
      count: 1,
      formats: ['toplist']
    })
  )
);

console.log(`Generated ${batchResults.length} content pieces`);
```

### Custom Video Composition (Remotion)

```typescript
// src/video/compositions/CustomArticle.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomArticle: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{
        color: 'white',
        fontSize: 60,
        opacity: Math.min(1, frame / fps)
      }}>
        {title}
      </div>
      {points.map((point, i) => (
        <div key={i} style={{
          opacity: frame > (i + 1) * fps ? 1 : 0
        }}>
          {point}
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1
});

await limiter.throttle(async () => {
  return await generator.generate({ ... });
});
```

### Video Rendering Memory Issues

```typescript
// Use streaming for large videos
const renderer = new VideoRenderer({
  concurrency: 2, // Reduce concurrent renders
  maxMemory: '4GB',
  enableGpuAcceleration: false // Disable if causing issues
});
```

### Claude/OpenAI API Errors

```typescript
try {
  const article = await generator.generate({ ... });
} catch (error) {
  if (error.code === 'rate_limit_exceeded') {
    // Implement exponential backoff
    await new Promise(r => setTimeout(r, 60000));
    // Retry with different provider
    generator.provider = 'openai';
  } else if (error.code === 'invalid_api_key') {
    console.error('Check your API keys in .env');
  }
}
```

### Research Scanner Timeout

```typescript
const scanner = new ResearchScanner({
  timeout: 30000, // 30 seconds
  retryAttempts: 3,
  fallbackSources: ['cache'] // Use cached data if live fails
});
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render a single video (Remotion)
npm run render -- --composition=ArticleVideo --output=output.mp4

# Run research scanner standalone
npm run research:scan -- --keywords="AI,marketing"

# Generate content from CLI
npm run content:generate -- --keyword="AI trends" --format=toplist
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { ContentPipeline } from '@/lib/pipeline';

describe('Content Pipeline', () => {
  it('generates content with research data', async () => {
    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword: 'test topic',
      count: 1
    });

    expect(result).toHaveLength(1);
    expect(result[0].article.title).toBeDefined();
    expect(result[0].researchSources).toHaveLength.greaterThan(0);
  });
});
```

This system provides a complete content automation solution from research to video production, ideal for marketers, content creators, and agencies looking to scale their content operations with AI.
