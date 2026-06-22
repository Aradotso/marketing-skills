---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion for marketers and content creators
triggers:
  - how do I use the AI content pipeline to generate articles
  - set up automated content research and video generation
  - configure Claude and OpenAI for content automation
  - create videos from articles using Remotion integration
  - automate content workflow from research to publishing
  - generate multilingual content with AI pipeline
  - build automated marketing content system
  - use content pipeline for social media automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles everything from research and content generation to video rendering. This pipeline uses Claude 3, OpenAI, and Remotion to transform keywords into finished articles and videos automatically.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes recent data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Multi-Platform Output**: Exports content optimized for Reels, TikTok, and YouTube Shorts

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

Create a `.env` file with the following required variables:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom settings
CONTENT_LANGUAGE=en,vi
DEFAULT_TONE=professional
VIDEO_ASPECT_RATIO=9:16
```

## Core Components

### Research Engine

The research module crawls and analyzes real-time data:

```typescript
import { ResearchEngine } from './lib/research';

const engine = new ResearchEngine({
  apiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Fetch research data for a topic
const research = await engine.fetchResearch('AI marketing tools');

// Returns structured data with insights
console.log(research.insights);
console.log(research.sources);
console.log(research.keyPoints);
```

### Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from './lib/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: 'claude-3-opus-20240229'
});

// Generate article from research
const article = await generator.generate({
  topic: 'AI Marketing Trends 2024',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: 'en',
  tone: 'professional',
  researchData: research
});

console.log(article.title);
console.log(article.content);
console.log(article.metadata);
```

### Multilingual Content

Generate content in multiple languages simultaneously:

```typescript
import { MultilingualGenerator } from './lib/content';

const mlGenerator = new MultilingualGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!
});

const multiContent = await mlGenerator.generateMultilingual({
  topic: 'Content Marketing Automation',
  languages: ['en', 'vi'],
  format: 'how-to'
});

// Access content by language
console.log(multiContent.en.title);
console.log(multiContent.vi.title);
```

### Video Generation with Remotion

Transform articles into videos:

```typescript
import { VideoRenderer } from './lib/video';
import { Composition } from 'remotion';

const renderer = new VideoRenderer({
  fps: 30,
  duration: 60, // seconds
  width: 1080,
  height: 1920 // 9:16 for vertical video
});

// Render video from article
const video = await renderer.render({
  article: article,
  template: 'infographic', // 'talking-head', 'slideshow'
  outputPath: './output/video.mp4',
  backgroundColor: '#000000',
  fontFamily: 'Inter'
});

console.log(`Video rendered: ${video.path}`);
console.log(`Duration: ${video.duration}s`);
```

## Complete Workflow Example

End-to-end pipeline from keyword to video:

```typescript
import { ContentPipeline } from './lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    apiKey: process.env.RAPIDAPI_KEY!,
    sources: ['techcrunch', 'twitter']
  },
  content: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY!
  },
  video: {
    enabled: true,
    aspectRatio: '9:16'
  }
});

// Run full pipeline
const result = await pipeline.execute({
  keyword: 'AI content automation',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['tiktok', 'reels', 'shorts']
});

// Access outputs
result.articles.forEach(article => {
  console.log(`${article.language}: ${article.title}`);
});

result.videos.forEach(video => {
  console.log(`Video for ${video.platform}: ${video.path}`);
});
```

## API Routes (Next.js Integration)

### Generate Content API

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();

  const pipeline = new ContentPipeline({
    research: { apiKey: process.env.RAPIDAPI_KEY! },
    content: { 
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY! 
    }
  });

  try {
    const result = await pipeline.execute({
      keyword,
      contentFormat: format,
      languages,
      generateVideo: false
    });

    return NextResponse.json({ 
      success: true, 
      articles: result.articles 
    });
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

### Video Generation API

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video';

export async function POST(req: NextRequest) {
  const { articleId, template, platform } = await req.json();

  const renderer = new VideoRenderer({
    fps: 30,
    duration: platform === 'tiktok' ? 15 : 60,
    width: 1080,
    height: 1920
  });

  try {
    const video = await renderer.render({
      articleId,
      template,
      outputPath: `./output/${platform}-${Date.now()}.mp4`
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: video.url,
      duration: video.duration 
    });
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
const keywords = [
  'AI marketing automation',
  'Content creation tools',
  'Social media strategy'
];

const batchResults = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({
      keyword,
      contentFormat: 'toplist',
      languages: ['en'],
      generateVideo: true
    })
  )
);

batchResults.forEach((result, index) => {
  console.log(`Generated ${result.articles.length} articles for: ${keywords[index]}`);
});
```

### Custom Tone Configuration

```typescript
const tonePresets = {
  expert: {
    style: 'authoritative',
    vocabulary: 'technical',
    formality: 'high'
  },
  friendly: {
    style: 'conversational',
    vocabulary: 'simple',
    formality: 'low'
  },
  humorous: {
    style: 'entertaining',
    vocabulary: 'casual',
    formality: 'low'
  }
};

const article = await generator.generate({
  topic: 'Marketing Tips',
  tone: tonePresets.friendly,
  format: 'how-to'
});
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const todayTopic = await getTrendingTopic();
  
  const result = await pipeline.execute({
    keyword: todayTopic,
    contentFormat: 'pov',
    languages: ['en', 'vi'],
    generateVideo: true
  });

  await publishToPlatforms(result);
  console.log('Daily content published successfully');
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './lib/utils';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1
});

const safeGenerate = limiter.wrap(async (topic: string) => {
  return await generator.generate({ topic });
});

// Automatically handles rate limiting
const article = await safeGenerate('AI trends');
```

### Video Rendering Errors

```typescript
try {
  const video = await renderer.render({ article });
} catch (error) {
  if (error.code === 'REMOTION_TIMEOUT') {
    // Retry with lower quality
    const video = await renderer.render({ 
      article, 
      quality: 'medium',
      fps: 24 
    });
  } else if (error.code === 'OUT_OF_MEMORY') {
    // Reduce video duration
    const video = await renderer.render({ 
      article, 
      duration: 30 
    });
  }
}
```

### Research Data Quality

```typescript
const research = await engine.fetchResearch('topic');

// Validate research quality
if (research.sources.length < 3) {
  console.warn('Low source count, expanding search');
  const expandedResearch = await engine.fetchResearch('topic', {
    sources: ['all'],
    timeRange: '48h'
  });
}

// Filter out low-quality sources
const qualityResearch = {
  ...research,
  insights: research.insights.filter(i => i.confidence > 0.7)
};
```

## Performance Optimization

### Caching Research Results

```typescript
import { CacheManager } from './lib/cache';

const cache = new CacheManager({ ttl: 3600 }); // 1 hour

async function getCachedResearch(topic: string) {
  const cached = await cache.get(`research:${topic}`);
  if (cached) return cached;

  const research = await engine.fetchResearch(topic);
  await cache.set(`research:${topic}`, research);
  return research;
}
```

### Parallel Video Rendering

```typescript
const videos = await Promise.all(
  articles.map(article => 
    renderer.render({ article, template: 'infographic' })
  )
);
```

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through video generation, with full TypeScript support and Next.js integration.
