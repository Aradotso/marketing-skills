---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - automate content creation from research to video
  - generate blog posts with AI research automation
  - create social media videos from blog content
  - build marketing content pipeline with AI
  - scrape trending news and generate content automatically
  - convert articles to video using Remotion
  - set up automated content workflow with Claude and OpenAI
  - research and publish content automatically
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end content automation system that transforms a single keyword into fully-formatted blog posts and videos. The pipeline handles:

1. **Auto-research** - Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending content (last 24h)
2. **AI Content Generation** - Uses Claude 3/OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Output** - Generates Vietnamese and English versions simultaneously
4. **Video Rendering** - Converts content to video using Remotion for Reels/TikTok/Shorts
5. **Auto-publishing** - Schedules and posts to social platforms

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

### Environment Configuration

```env
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Content Sources
TECHCRUNCH_API_KEY=your_tc_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Publishing
FACEBOOK_PAGE_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Core Architecture

```typescript
// Typical pipeline flow
import { ResearchEngine } from './lib/research';
import { ContentGenerator } from './lib/ai-generator';
import { VideoRenderer } from './lib/video';
import { Publisher } from './lib/publisher';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await ResearchEngine.scanSources(keyword);
  
  // Step 2: Generate Content
  const content = await ContentGenerator.create(research, {
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'expert'
  });
  
  // Step 3: Render Video
  const video = await VideoRenderer.fromContent(content);
  
  // Step 4: Publish
  await Publisher.schedule(content, video, {
    platforms: ['facebook', 'linkedin', 'tiktok']
  });
}
```

## Research Module

### Auto-Scanning News Sources

```typescript
import { ResearchEngine } from '@/lib/research/engine';

const research = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  language: 'en'
});

// Scan for trending topics
const insights = await research.scan({
  keyword: 'AI automation',
  depth: 'comprehensive', // 'quick' | 'standard' | 'comprehensive'
  includeStats: true
});

console.log(insights);
// {
//   articles: [...],
//   trendingTopics: [...],
//   statistics: {...},
//   sentiment: 'positive',
//   relevanceScore: 0.92
// }
```

### Custom Source Configuration

```typescript
import { CustomSource } from '@/lib/research/sources';

const customSource = new CustomSource({
  name: 'MyBlog',
  url: 'https://myblog.com/api/posts',
  parser: async (html) => {
    // Custom parsing logic
    return {
      title: extractTitle(html),
      content: extractContent(html),
      date: extractDate(html)
    };
  }
});

research.addSource(customSource);
```

## Content Generation

### Using Claude/OpenAI

```typescript
import { ContentGenerator } from '@/lib/ai-generator';

const generator = new ContentGenerator({
  provider: 'claude', // 'claude' | 'openai'
  model: 'claude-3-opus-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate article
const article = await generator.generate({
  research: researchData,
  format: 'case-study', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  language: 'vi',
  wordCount: 1500,
  includeImages: true
});

console.log(article);
// {
//   title: "Case Study: ...",
//   content: "...",
//   metadata: {...},
//   suggestedImages: [...],
//   seo: {...}
// }
```

### Multi-Language Generation

```typescript
const bilingualContent = await generator.generateBilingual({
  research: researchData,
  languages: ['en', 'vi'],
  format: 'toplist',
  syncStructure: true // Keep same structure across languages
});

// Access by language
const englishVersion = bilingualContent.en;
const vietnameseVersion = bilingualContent.vi;
```

### Content Templates

```typescript
import { TemplateManager } from '@/lib/templates';

const template = TemplateManager.load('tech-review');

const customContent = await generator.generate({
  research: researchData,
  template: template,
  variables: {
    productName: 'ChatGPT-5',
    author: 'Tech Reviewer',
    rating: 4.5
  }
});
```

## Video Rendering with Remotion

### Basic Video Generation

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { Composition } from 'remotion';

const renderer = new VideoRenderer({
  compositionId: 'ContentVideo',
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
});

const video = await renderer.render({
  content: article,
  template: 'modern-tech', // Video template
  dimensions: {
    width: 1080,
    height: 1920 // Vertical for Reels/TikTok
  },
  music: 'upbeat-background.mp3',
  voiceover: {
    enabled: true,
    voice: 'en-US-Neural',
    speed: 1.0
  }
});

// Output: video.mp4
console.log(video.path); // ./output/video-20240614.mp4
```

### Custom Video Composition

```typescript
// components/VideoComposition.tsx
import { AbsoluteFill, Sequence, Audio } from 'remotion';

export const ContentVideoComposition: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: 'white', fontSize: 64 }}>{title}</h1>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={60 + (index * 120)}
          durationInFrames={120}
        >
          <div style={{ color: 'white', fontSize: 32 }}>
            {index + 1}. {point}
          </div>
        </Sequence>
      ))}
      
      <Audio src="/music/background.mp3" volume={0.3} />
    </AbsoluteFill>
  );
};
```

### Platform-Specific Rendering

```typescript
const platformVideos = await renderer.renderForPlatforms({
  content: article,
  platforms: {
    tiktok: {
      dimensions: { width: 1080, height: 1920 },
      maxDuration: 60
    },
    youtube: {
      dimensions: { width: 1920, height: 1080 },
      maxDuration: 300
    },
    instagram: {
      dimensions: { width: 1080, height: 1080 },
      maxDuration: 90
    }
  }
});

// Access videos by platform
const tiktokVideo = platformVideos.tiktok;
const youtubeVideo = platformVideos.youtube;
```

## Publishing Pipeline

### Auto-Scheduling

```typescript
import { Publisher } from '@/lib/publisher';

const publisher = new Publisher({
  platforms: {
    facebook: {
      pageId: process.env.FB_PAGE_ID,
      accessToken: process.env.FACEBOOK_PAGE_TOKEN
    },
    linkedin: {
      companyId: process.env.LINKEDIN_COMPANY_ID,
      accessToken: process.env.LINKEDIN_ACCESS_TOKEN
    }
  }
});

// Schedule posts
await publisher.schedule({
  content: article,
  video: video,
  scheduledTime: new Date('2024-06-15T10:00:00Z'),
  platforms: ['facebook', 'linkedin'],
  caption: article.seo.metaDescription,
  hashtags: ['#AI', '#Automation', '#Marketing']
});
```

### Batch Publishing

```typescript
const batch = await publisher.createBatch([
  { content: article1, platforms: ['facebook'] },
  { content: article2, platforms: ['linkedin', 'twitter'] },
  { content: article3, platforms: ['tiktok', 'instagram'] }
]);

await batch.publish({
  interval: '2h', // Publish every 2 hours
  startTime: new Date()
});
```

## API Routes (Next.js)

### Trigger Pipeline Endpoint

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages } = req.body;

  try {
    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      autoPublish: false
    });

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
// pages/api/pipeline/[id]/status.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { id } = req.query;
  
  const status = await getPipelineStatus(id as string);
  
  res.status(200).json(status);
  // {
  //   id: "pip_123",
  //   stage: "video_rendering", // research | content_gen | video_rendering | publishing
  //   progress: 65,
  //   startedAt: "2024-06-14T10:00:00Z",
  //   estimatedCompletion: "2024-06-14T10:15:00Z"
  // }
}
```

## CLI Commands

```bash
# Run full pipeline
npm run pipeline -- --keyword "AI automation" --format toplist --lang en,vi

# Research only
npm run research -- --keyword "blockchain trends" --sources techcrunch,a16z

# Generate content from existing research
npm run generate -- --input research.json --format case-study --lang vi

# Render video
npm run video -- --input article.json --template modern --platform tiktok

# Publish content
npm run publish -- --content article.json --video video.mp4 --platforms facebook,linkedin --schedule "2024-06-15 10:00"
```

## Common Patterns

### Daily Trend Monitor

```typescript
import { CronJob } from 'cron';

// Run every day at 9 AM
const dailyTrendJob = new CronJob('0 9 * * *', async () => {
  const trends = await ResearchEngine.getTrendingTopics({
    sources: ['techcrunch', 'twitter'],
    limit: 5
  });

  for (const trend of trends) {
    await runContentPipeline({
      keyword: trend.keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      autoPublish: true
    });
  }
});

dailyTrendJob.start();
```

### Content Queue Management

```typescript
import { Queue } from '@/lib/queue';

const contentQueue = new Queue('content-generation');

// Add to queue
await contentQueue.add({
  keyword: 'AI tools',
  priority: 'high'
});

// Process queue
contentQueue.process(async (job) => {
  const result = await runContentPipeline(job.data);
  return result;
});
```

### A/B Testing Content

```typescript
const variants = await generator.generateVariants({
  research: researchData,
  count: 3,
  variations: ['tone', 'structure'],
  tones: ['professional', 'friendly', 'humorous']
});

// Test performance
const winner = await Publisher.abTest(variants, {
  platforms: ['facebook'],
  duration: '24h',
  metric: 'engagement'
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requests: 50, per: '1m' },
  anthropic: { requests: 40, per: '1m' }
});

await limiter.execute('openai', async () => {
  return await generator.generate(params);
});
```

### Video Rendering Timeouts

```typescript
// Increase timeout for long videos
const renderer = new VideoRenderer({
  timeout: 600000, // 10 minutes
  quality: 'medium', // Use 'low' for faster rendering
  concurrency: 2 // Parallel rendering
});
```

### Content Quality Validation

```typescript
import { ContentValidator } from '@/lib/validation';

const validator = new ContentValidator({
  minWordCount: 800,
  requireImages: true,
  checkPlagiarism: true,
  seoScore: 70
});

const isValid = await validator.validate(article);

if (!isValid) {
  console.log(validator.errors);
  // Regenerate with adjustments
  article = await generator.regenerate(article, {
    improvements: validator.suggestions
  });
}
```

### Error Recovery

```typescript
import { PipelineRunner } from '@/lib/pipeline';

const pipeline = new PipelineRunner({
  retryOnFailure: true,
  maxRetries: 3,
  backoff: 'exponential',
  onError: async (stage, error) => {
    console.error(`Failed at ${stage}:`, error);
    // Send notification, log to monitoring service, etc.
  }
});

await pipeline.run(config);
```

## Performance Optimization

```typescript
// Cache research results
import { CacheManager } from '@/lib/cache';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  provider: 'redis' // 'memory' | 'redis'
});

const cachedResearch = await cache.wrap('research:ai-tools', async () => {
  return await ResearchEngine.scan({ keyword: 'AI tools' });
});

// Parallel processing
const [research, template, settings] = await Promise.all([
  ResearchEngine.scan(keyword),
  TemplateManager.load(templateName),
  getUserSettings(userId)
]);
```

This skill enables AI coding agents to effectively help developers implement automated content marketing pipelines using this TypeScript-based system.
