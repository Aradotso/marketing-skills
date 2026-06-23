---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, posting, and video generation with Claude/OpenAI
triggers:
  - automate content creation with AI
  - set up marketing content pipeline
  - generate videos from articles automatically
  - crawl news and create social posts
  - build AI content automation system
  - create scripts and videos with Claude
  - automate research to video workflow
  - set up remotion video generation
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you work with the Ultimate AI Content Pipeline, an end-to-end content automation system that handles research, scriptwriting, posting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion for video generation
REMOTION_LICENSE_KEY=your_remotion_key

# Optional: Social media posting
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run video rendering
npm run render
```

## Core API Usage

### 1. Research Module - Auto News Crawling

```typescript
import { ResearchService } from '@/services/research';

const researchService = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl news from last 24 hours
const results = await researchService.crawlNews({
  keyword: 'artificial intelligence',
  timeRange: '24h',
  maxResults: 10
});

console.log(results);
// {
//   articles: [...],
//   insights: [...],
//   trending: [...]
// }
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { ContentGenerator } from '@/services/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-sonnet-20240229'
});

// Generate article from research
const article = await generator.createArticle({
  topic: 'AI in Marketing',
  format: 'toplist', // toplist | pov | case-study | how-to
  language: 'vi', // en | vi
  tone: 'professional', // professional | friendly | humorous
  researchData: results,
  wordCount: 1500
});

console.log(article);
// {
//   title: '...',
//   content: '...',
//   metadata: {...},
//   seo: {...}
// }
```

### 3. Multi-format Content Generation

```typescript
// Generate content in multiple formats simultaneously
const multiContent = await generator.generateMultiFormat({
  baseResearch: results,
  formats: ['toplist', 'how-to', 'case-study'],
  languages: ['en', 'vi']
});

// Output: 6 articles (3 formats × 2 languages)
for (const content of multiContent) {
  console.log(`${content.format} - ${content.language}: ${content.title}`);
}
```

### 4. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/services/video';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

const videoRenderer = new VideoRenderer({
  composition: 'ArticleToVideo',
  width: 1080,
  height: 1920 // 9:16 for Reels/TikTok
});

// Render video from article
const video = await videoRenderer.render({
  article: article,
  style: 'modern', // modern | minimal | vibrant
  duration: 60, // seconds
  outputFormat: 'mp4'
});

console.log(`Video saved to: ${video.outputPath}`);
```

### 5. Complete Pipeline Automation

```typescript
import { ContentPipeline } from '@/services/pipeline';

const pipeline = new ContentPipeline({
  research: researchService,
  generator: generator,
  videoRenderer: videoRenderer
});

// Run full automation
const result = await pipeline.execute({
  keyword: 'AI Marketing Tools 2024',
  contentTypes: ['article', 'video'],
  autoPost: true, // Auto-post to social media
  platforms: ['facebook', 'twitter', 'linkedin']
});

console.log(result);
// {
//   articles: [...],
//   videos: [...],
//   posts: [...],
//   analytics: {...}
// }
```

## Common Patterns

### Pattern 1: Daily Content Automation

```typescript
import { CronJob } from 'cron';

// Run daily at 6 AM
const dailyJob = new CronJob('0 6 * * *', async () => {
  const keywords = ['AI', 'Marketing', 'SaaS'];
  
  for (const keyword of keywords) {
    const research = await researchService.crawlNews({ keyword });
    
    if (research.articles.length > 0) {
      const article = await generator.createArticle({
        topic: keyword,
        format: 'toplist',
        language: 'vi',
        researchData: research
      });
      
      await pipeline.publishToSocial(article);
    }
  }
});

dailyJob.start();
```

### Pattern 2: Batch Video Generation

```typescript
// Generate videos for all articles in queue
async function batchRenderVideos(articles: Article[]) {
  const videos = await Promise.allSettled(
    articles.map(article => 
      videoRenderer.render({
        article,
        style: 'modern',
        duration: 60
      })
    )
  );
  
  return videos
    .filter(v => v.status === 'fulfilled')
    .map(v => v.value);
}
```

### Pattern 3: Custom Research Sources

```typescript
// Add custom RSS feeds or APIs
researchService.addSource({
  name: 'custom-blog',
  type: 'rss',
  url: 'https://yourblog.com/feed',
  parser: async (feed) => {
    return feed.items.map(item => ({
      title: item.title,
      url: item.link,
      publishedAt: item.pubDate,
      content: item.content
    }));
  }
});
```

### Pattern 4: A/B Testing Content Variations

```typescript
// Generate multiple versions for testing
const variations = await generator.createVariations({
  topic: 'AI Marketing',
  count: 3,
  vary: ['tone', 'hook', 'cta']
});

// Track performance
for (const variant of variations) {
  await pipeline.publishWithTracking(variant, {
    experimentId: 'test-001',
    variantId: variant.id
  });
}
```

## API Endpoints (Next.js)

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/services/research';

export async function POST(req: NextRequest) {
  const { keyword, timeRange } = await req.json();
  
  const service = new ResearchService({
    apiKey: process.env.RAPIDAPI_KEY
  });
  
  const results = await service.crawlNews({ keyword, timeRange });
  
  return NextResponse.json(results);
}
```

### POST /api/generate

```typescript
// app/api/generate/route.ts
export async function POST(req: NextRequest) {
  const { topic, format, language, researchData } = await req.json();
  
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const article = await generator.createArticle({
    topic,
    format,
    language,
    researchData
  });
  
  return NextResponse.json(article);
}
```

### POST /api/render

```typescript
// app/api/render/route.ts
export async function POST(req: NextRequest) {
  const { articleId, style } = await req.json();
  
  const renderer = new VideoRenderer();
  const video = await renderer.render({
    articleId,
    style,
    duration: 60
  });
  
  return NextResponse.json({ videoUrl: video.url });
}
```

## Remotion Video Components

### Basic Video Composition

```typescript
// remotion/ArticleToVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ArticleToVideo: React.FC<{
  title: string;
  points: string[];
  style: 'modern' | 'minimal';
}> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'modern' ? '#1a1a1a' : '#ffffff',
        padding: 60
      }}
    >
      <h1 style={{ opacity, fontSize: 72, fontWeight: 'bold' }}>
        {title}
      </h1>
      
      {points.map((point, i) => {
        const pointFrame = frame - (i + 1) * fps * 2;
        const pointOpacity = Math.max(0, Math.min(1, pointFrame / 30));
        
        return (
          <p key={i} style={{ opacity: pointOpacity, fontSize: 48 }}>
            {point}
          </p>
        );
      })}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Rate Limit Errors

```typescript
// Add retry logic with exponential backoff
import pRetry from 'p-retry';

const generateWithRetry = async (params) => {
  return pRetry(
    () => generator.createArticle(params),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`);
      }
    }
  );
};
```

### Video Rendering Memory Issues

```typescript
// Render videos sequentially instead of parallel
for (const article of articles) {
  await videoRenderer.render({ article });
  // Allow memory cleanup between renders
  await new Promise(resolve => setTimeout(resolve, 2000));
}
```

### News Crawling Blocked

```typescript
// Use rotating proxies
researchService.setProxyRotation({
  enabled: true,
  proxyList: process.env.PROXY_LIST?.split(',') || []
});
```

### Claude API Timeouts

```typescript
// Increase timeout for long-form content
const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 120000, // 2 minutes
  maxTokens: 4096
});
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** jobs to manage server resources
3. **Store generated content** in a database for reuse and analytics
4. **Monitor API usage** to stay within rate limits
5. **Use webhooks** for long-running operations instead of polling
6. **Validate content** before auto-posting to social media
7. **Implement error tracking** (Sentry, LogRocket) for production

## Testing

```typescript
// tests/pipeline.test.ts
import { ContentPipeline } from '@/services/pipeline';

describe('Content Pipeline', () => {
  it('should generate article from research', async () => {
    const mockResearch = {
      articles: [{ title: 'AI News', content: '...' }]
    };
    
    const result = await pipeline.execute({
      keyword: 'AI',
      contentTypes: ['article'],
      autoPost: false
    });
    
    expect(result.articles).toHaveLength(1);
    expect(result.articles[0].title).toBeDefined();
  });
});
```
