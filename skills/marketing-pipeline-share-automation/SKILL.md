---
name: marketing-pipeline-share-automation
description: Automate content research, scriptwriting, and video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content pipeline with research and video generation
  - use marketing pipeline share to generate automated content
  - create automated content workflow with Claude and OpenAI
  - generate videos from content using Remotion integration
  - configure AI content research and script generation
  - build automated marketing content pipeline
  - set up content automation from research to video
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

An end-to-end AI-powered content automation system that handles research, scriptwriting, and video generation. This TypeScript-based pipeline automatically crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:
- **Auto-Research**: Crawls TechCrunch, a16z, X (Twitter), LinkedIn for fresh data
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
- **Video Generation**: Automatically renders infographics and short videos using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render videos with Remotion
npm run remotion:render
```

## Key Components

### 1. Content Research Module

Automatically crawl and analyze recent content from news sources:

```typescript
import { ResearchCrawler } from './lib/research/crawler';

const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Execute research
const research = await crawler.scan();

console.log(research.insights); // Array of insights
console.log(research.data); // Raw data with sources
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from './lib/ai/generator';

const generator = new ContentGenerator({
  model: 'claude-3', // or 'gpt-4'
  apiKey: process.env.ANTHROPIC_API_KEY
});

const content = await generator.create({
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  topic: 'AI Marketing Tools 2024',
  language: 'en', // 'en' | 'vi'
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  research: research.insights,
  length: 'medium' // 'short' | 'medium' | 'long'
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Multi-Format Content Creation

Create content in different formats:

```typescript
import { MultiFormatGenerator } from './lib/ai/multi-format';

const multiGen = new MultiFormatGenerator();

// Generate toplist
const toplist = await multiGen.generateToplist({
  topic: 'Top 10 AI Tools for Content Creators',
  items: 10,
  research: research.insights
});

// Generate POV article
const pov = await multiGen.generatePOV({
  topic: 'Why AI Will Transform Marketing',
  perspective: 'industry-expert',
  arguments: research.data
});

// Generate case study
const caseStudy = await multiGen.generateCaseStudy({
  company: 'Example Corp',
  challenge: 'Content production at scale',
  solution: 'AI automation pipeline',
  results: research.data
});

// Generate how-to guide
const howTo = await multiGen.generateHowTo({
  topic: 'How to Set Up AI Content Pipeline',
  steps: 5,
  difficulty: 'intermediate'
});
```

### 4. Video Generation with Remotion

Render videos from generated content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from './remotion/Composition';

// Bundle Remotion project
const bundleLocation = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config,
});

// Render video
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
});

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/${content.title}.mp4`,
  inputProps: {
    title: content.title,
    body: content.body,
    format: 'reel', // 'reel' | 'tiktok' | 'shorts'
  },
});
```

### 5. Remotion Video Composition

Create video composition for content:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  body: string;
  format: 'reel' | 'tiktok' | 'shorts';
}> = ({ title, body, format }) => {
  const frame = useCurrentFrame();

  const dimensions = {
    reel: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <div style={{
          fontSize: 48,
          color: 'white',
          textAlign: 'center',
          padding: 40,
        }}>
          {title}
        </div>
      </Sequence>
      <Sequence from={90} durationInFrames={210}>
        <div style={{
          fontSize: 32,
          color: 'white',
          padding: 60,
        }}>
          {body}
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

Run the entire pipeline from research to video:

```typescript
import { ContentPipeline } from './lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI Marketing Automation',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reel', 'shorts'],
});

console.log(result.research); // Research data
console.log(result.content); // Generated content array
console.log(result.videos); // Rendered video paths
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language, generateVideo } = req.body;

  try {
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY,
    });

    const result = await pipeline.execute({
      keyword,
      formats: [format],
      languages: [language],
      generateVideo,
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchCrawler } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keywords, sources, timeRange } = req.query;

  try {
    const crawler = new ResearchCrawler({
      sources: (sources as string).split(','),
      timeRange: timeRange as string || '24h',
      keywords: (keywords as string).split(','),
    });

    const research = await crawler.scan();
    res.status(200).json(research);
  } catch (error) {
    console.error('Research error:', error);
    res.status(500).json({ error: 'Research failed' });
  }
}
```

## Common Patterns

### Batch Content Generation

Generate multiple pieces of content at once:

```typescript
import { BatchGenerator } from './lib/ai/batch';

const batchGen = new BatchGenerator();

const contents = await batchGen.generateBatch({
  topics: [
    'AI Marketing Tools',
    'Content Automation Best Practices',
    'Video Marketing Trends 2024',
  ],
  format: 'toplist',
  languages: ['en', 'vi'],
  parallelProcessing: true,
});

contents.forEach((content, index) => {
  console.log(`Content ${index + 1}:`, content.title);
});
```

### Scheduled Content Generation

Set up automated content generation:

```typescript
import { Scheduler } from './lib/scheduler';

const scheduler = new Scheduler();

// Schedule daily content generation
scheduler.daily('08:00', async () => {
  const pipeline = new ContentPipeline(config);
  
  const result = await pipeline.execute({
    keyword: 'trending AI news',
    formats: ['toplist', 'pov'],
    languages: ['en'],
    generateVideo: true,
  });
  
  // Auto-publish or save to database
  await publishContent(result.content);
});
```

### Custom Video Templates

Create custom video templates:

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, interpolate, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  data: any;
}> = ({ data }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity }}>
        {/* Custom video layout */}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

If you hit rate limits:

```typescript
import { RateLimiter } from './lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1,
});

await limiter.throttle(async () => {
  return await generator.create(options);
});
```

### Video Rendering Fails

Check Remotion configuration and ensure FFmpeg is installed:

```bash
# Install FFmpeg
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from ffmpeg.org
```

Verify Remotion setup:

```typescript
import { getCompositions } from '@remotion/renderer';

try {
  const compositions = await getCompositions(bundleLocation);
  console.log('Available compositions:', compositions);
} catch (error) {
  console.error('Remotion error:', error);
}
```

### AI Generation Timeout

Increase timeout for long-form content:

```typescript
const generator = new ContentGenerator({
  model: 'claude-3',
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 120000, // 2 minutes
  maxRetries: 3,
});
```

### Memory Issues with Large Batches

Process in smaller chunks:

```typescript
import { chunk } from 'lodash';

const topics = [...]; // Large array
const chunks = chunk(topics, 5); // Process 5 at a time

for (const topicChunk of chunks) {
  await batchGen.generateBatch({ topics: topicChunk });
  // Add delay between chunks
  await new Promise(resolve => setTimeout(resolve, 5000));
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific configs** for different deployment stages
4. **Monitor API usage** to stay within quotas
5. **Version control video templates** separately from main codebase
6. **Test video renders locally** before production deployment
7. **Implement proper error handling** for each pipeline stage
8. **Use TypeScript strict mode** for type safety
