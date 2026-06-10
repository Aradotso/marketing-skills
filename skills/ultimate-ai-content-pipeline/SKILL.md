---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up ultimate ai content pipeline with claude and openai
  - create videos automatically from blog content
  - automate content creation from research to video
  - use remotion to generate video from articles
  - configure ai content pipeline with auto-research
  - build automated marketing content system
  - generate multilingual content with ai pipeline
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
2. **AI Content Generation**: Creates articles in multiple formats using Claude 3 or OpenAI
3. **Multi-Language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Exports video formats for Reels, TikTok, and Shorts

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

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
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
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research Module

```typescript
import { ResearchEngine } from '@/lib/research/engine';

// Initialize research engine
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h'
});

// Conduct research on a topic
const insights = await researcher.research({
  keyword: 'AI automation',
  depth: 'deep',
  includeStats: true
});

console.log(insights);
// {
//   articles: [...],
//   insights: [...],
//   statistics: {...},
//   trends: [...]
// }
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-opus-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate content with research data
const article = await generator.generate({
  topic: 'AI in Marketing',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  tone: 'expert', // 'friendly', 'humorous'
  languages: ['en', 'vi'],
  researchData: insights
});

console.log(article);
// {
//   english: { title: '...', content: '...' },
//   vietnamese: { title: '...', content: '...' }
// }
```

### 3. Multi-Format Content

```typescript
import { FormatAdapter } from '@/lib/content/format-adapter';

const adapter = new FormatAdapter();

// Generate different formats
const formats = await adapter.generateMultiFormat({
  baseContent: article,
  formats: ['blog-post', 'linkedin-post', 'twitter-thread', 'video-script']
});

// Each format optimized for its platform
formats.forEach(({ platform, content }) => {
  console.log(`${platform}:`, content);
});
```

### 4. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer();

// Render video from article content
const video = await renderer.render({
  composition: 'ArticleVideo',
  props: {
    title: article.english.title,
    keyPoints: article.english.keyPoints,
    duration: 60, // seconds
    aspectRatio: '9:16' // for Reels/TikTok
  },
  outputPath: './output/video.mp4'
});

console.log('Video rendered:', video.outputPath);
```

### 5. Complete Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  research: { enabled: true, depth: 'deep' },
  ai: { provider: 'claude' },
  video: { enabled: true, platforms: ['reels', 'tiktok', 'shorts'] }
});

// Run full automation
const result = await pipeline.execute({
  keyword: 'AI Marketing Tools 2024',
  outputFormats: ['blog', 'video', 'social'],
  schedule: true // Auto-schedule posting
});

console.log('Pipeline completed:', result);
// {
//   articles: [...],
//   videos: [...],
//   scheduled: [...],
//   analytics: {...}
// }
```

## Next.js API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchEngine } from '@/lib/research/engine';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  const researcher = new ResearchEngine({ sources });
  const data = await researcher.research({ keyword });
  
  return NextResponse.json(data);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  const { topic, format, researchData } = await req.json();
  
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const content = await generator.generate({
    topic,
    format,
    researchData
  });
  
  return NextResponse.json(content);
}
```

### Video Render Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { content, config } = await req.json();
  
  const renderer = new VideoRenderer();
  const video = await renderer.render({
    composition: 'ArticleVideo',
    props: content,
    ...config
  });
  
  return NextResponse.json({ 
    success: true, 
    videoUrl: video.outputPath 
  });
}
```

## Common Patterns

### Pattern 1: Research-First Workflow

```typescript
import { pipeline } from '@/lib/pipeline';

async function researchAndCreate(keyword: string) {
  // Step 1: Research
  const research = await pipeline.research.gather({
    keyword,
    sources: ['techcrunch', 'a16z'],
    minArticles: 10
  });
  
  // Step 2: Generate content variants
  const variants = await Promise.all([
    pipeline.ai.generate({ format: 'toplist', research }),
    pipeline.ai.generate({ format: 'how-to', research }),
    pipeline.ai.generate({ format: 'case-study', research })
  ]);
  
  // Step 3: Create videos for each variant
  const videos = await Promise.all(
    variants.map(variant => 
      pipeline.video.render({ content: variant, platform: 'reels' })
    )
  );
  
  return { variants, videos };
}
```

### Pattern 2: Scheduled Content Creation

```typescript
import { CronJob } from 'cron';
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline();

// Daily content generation at 9 AM
const dailyJob = new CronJob('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content tools'];
  
  for (const keyword of keywords) {
    await pipeline.execute({
      keyword,
      schedule: true,
      platforms: ['facebook', 'linkedin', 'twitter']
    });
  }
});

dailyJob.start();
```

### Pattern 3: Multi-Language Content

```typescript
import { MultiLangGenerator } from '@/lib/ai/multilang';

async function generateMultiLang(topic: string) {
  const generator = new MultiLangGenerator({
    languages: ['en', 'vi'],
    maintainTone: true
  });
  
  const content = await generator.generate({
    topic,
    format: 'toplist',
    customizePerLanguage: {
      en: { tone: 'professional', examples: 'global' },
      vi: { tone: 'friendly', examples: 'local' }
    }
  });
  
  // Content optimized per language culture
  return content;
}
```

### Pattern 4: Video Template Customization

```typescript
// remotion/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  keyPoints: string[];
  duration: number;
}> = ({ title, keyPoints, duration }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleScreen title={title} />
      </Sequence>
      
      {keyPoints.map((point, i) => (
        <Sequence 
          key={i} 
          from={90 + i * 120} 
          durationInFrames={120}
        >
          <KeyPointScreen point={point} index={i + 1} />
        </Sequence>
      ))}
      
      <Sequence from={duration * 30 - 60} durationInFrames={60}>
        <CTAScreen />
      </Sequence>
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run research only
npm run research -- --keyword "AI tools" --depth deep

# Generate content from existing research
npm run generate -- --input ./research-data.json --format toplist

# Render video
npm run render-video -- --composition ArticleVideo --props ./props.json

# Full pipeline execution
npm run pipeline -- --keyword "Marketing automation" --all-formats
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  anthropic: { requestsPerMinute: 50 },
  openai: { requestsPerMinute: 60 }
});

// Automatically handles rate limiting
await limiter.execute('anthropic', async () => {
  return await generator.generate(params);
});
```

### Video Rendering Timeout

```typescript
import { VideoRenderer } from '@/lib/video/renderer';

const renderer = new VideoRenderer({
  timeout: 300000, // 5 minutes
  concurrency: 2,  // Render 2 videos at once
  retries: 3
});

try {
  await renderer.render(config);
} catch (error) {
  if (error.code === 'TIMEOUT') {
    // Reduce video quality or duration
    await renderer.render({ ...config, quality: 'medium' });
  }
}
```

### Research Data Quality

```typescript
import { DataValidator } from '@/lib/research/validator';

const validator = new DataValidator({
  minArticles: 5,
  minInsights: 3,
  requireStats: true
});

const research = await researcher.research({ keyword });

if (!validator.validate(research)) {
  // Retry with different sources
  const fallbackResearch = await researcher.research({
    keyword,
    sources: ['alternative-source']
  });
}
```

### Memory Management for Large Pipelines

```typescript
import { PipelineOptimizer } from '@/lib/pipeline/optimizer';

const optimizer = new PipelineOptimizer({
  chunkSize: 5, // Process 5 keywords at a time
  clearCache: true,
  streamResults: true
});

await optimizer.processBatch(keywords, async (keyword) => {
  return await pipeline.execute({ keyword });
});
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys
3. **Implement caching** for research results to avoid redundant API calls
4. **Monitor API quotas** across all providers
5. **Test video renders** with short durations first
6. **Schedule heavy operations** during off-peak hours
7. **Keep video assets** under 50MB for optimal rendering

## Advanced Configuration

```typescript
// lib/config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: {
      techcrunch: { weight: 1.0, priority: 'high' },
      a16z: { weight: 0.9, priority: 'high' },
      twitter: { weight: 0.7, priority: 'medium' }
    },
    cache: {
      enabled: true,
      ttl: 3600 // 1 hour
    }
  },
  ai: {
    fallback: ['claude', 'openai'],
    temperature: 0.7,
    maxTokens: 4000
  },
  video: {
    defaultFps: 30,
    defaultFormat: 'mp4',
    quality: 'high'
  }
};
```
